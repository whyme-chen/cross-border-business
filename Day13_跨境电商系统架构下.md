# Day13 — 跨境电商系统架构（下）

---

## 一、库存架构

### 统一库存服务

```
┌─────────────────────────────────────────────────────┐
│                  统一库存服务                         │
│                                                     │
│  ┌───────────────────────────────────────┐          │
│  │           库存池 (Stock Pool)          │          │
│  │                                       │          │
│  │  SKU-A: 总量1000                      │          │
│  │    ├── Amazon US FBA: 300             │          │
│  │    ├── 海外仓(美西): 200              │          │
│  │    ├── 海外仓(美东): 150              │          │
│  │    ├── 国内仓: 300                    │          │
│  │    └── 在途(头程): 50                 │          │
│  └───────────────────────────────────────┘          │
│                                                     │
│  库存操作:                                           │
│  ├── 预占(Reserve): 下单时锁定                       │
│  ├── 扣减(Deduct): 发货时扣减                        │
│  ├── 释放(Release): 取消/超时时释放                   │
│  ├── 入库(Receive): 收货入库                         │
│  └── 调整(Adjust): 盘点/校正                         │
└─────────────────────────────────────────────────────┘
```

### 锁库存/预占/释放机制

```
时序:

下单 ──→ 预占(reserved_qty += n) ──→ 可用库存减少
  │                                       │
  │ 超时未付/取消                          │ 发货
  │                                       │
  ▼                                       ▼
释放(reserved_qty -= n)              扣减(available_qty -= n,
     available_qty += n                    reserved_qty -= n)
```

### Redis 库存预扣

```java
@Service
public class InventoryReserveService {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    private static final String STOCK_KEY = "stock:available:%d"; // stock:available:{skuId}
    
    /**
     * 预占库存 - 原子操作
     * @return true=预占成功, false=库存不足
     */
    public boolean reserve(Long skuId, Integer quantity) {
        String key = String.format(STOCK_KEY, skuId);
        // Lua脚本保证原子性: 检查+扣减
        String script = 
            "local stock = redis.call('GET', KEYS[1]) " +
            "if not stock or tonumber(stock) < tonumber(ARGV[1]) then " +
            "  return 0 " +
            "end " +
            "redis.call('DECRBY', KEYS[1], ARGV[1]) " +
            "return 1";
        
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Collections.singletonList(key),
            String.valueOf(quantity)
        );
        
        if (result == 1) {
            // 异步写DB
            asyncUpdateDatabase(skuId, quantity, "RESERVE");
            return true;
        }
        return false;
    }
    
    /**
     * 释放预占
     */
    public void release(Long skuId, Integer quantity) {
        String key = String.format(STOCK_KEY, skuId);
        redisTemplate.opsForValue().increment(key, quantity);
        asyncUpdateDatabase(skuId, quantity, "RELEASE");
    }
}
```

### 多渠道库存分配

```
总可用库存: 1000
  ├── Amazon渠道配额: 500 (50%)
  ├── Shopee渠道配额: 200 (20%)
  ├── 独立站配额: 150 (15%)
  └── 安全库存: 150 (15%, 不对外)

Amazon可售数量 = min(Amazon配额, 总可用-安全库存) 
              = min(500, 850) = 500

当Amazon用完500后:
  ├── 方案1: 不超卖，显示缺货
  ├── 方案2: 从安全库存中释放(需审批)
  └── 方案3: 动态调整配额(从其他渠道借)
```

---

## 二、订单状态机实现

### 状态机 + 事件驱动

```java
@Service
public class OrderStateMachine {
    
    // 合法状态转换定义
    private static final Map<OrderStatus, Set<OrderEvent>> TRANSITIONS = Map.of(
        OrderStatus.PENDING_PAYMENT, Set.of(OrderEvent.PAY, OrderEvent.CANCEL),
        OrderStatus.PAID, Set.of(OrderEvent.APPROVE, OrderEvent.CANCEL),
        OrderStatus.PENDING_SHIPMENT, Set.of(OrderEvent.SHIP, OrderEvent.CANCEL),
        OrderStatus.SHIPPED, Set.of(OrderEvent.DELIVER, OrderEvent.PARTIAL_DELIVER),
        OrderStatus.COMPLETED, Set.of(OrderEvent.RETURN)
    );
    
    @Transactional
    public void fireEvent(Long orderId, OrderEvent event) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new BusinessException("订单不存在"));
        
        // 校验状态转换合法性
        if (!TRANSITIONS.getOrDefault(order.getStatus(), Set.of()).contains(event)) {
            throw new BusinessException(
                String.format("非法状态转换: %s + %s", order.getStatus(), event));
        }
        
        // 执行状态转换
        OrderStatus newStatus = resolveNewStatus(order.getStatus(), event);
        order.setStatus(newStatus);
        orderRepository.save(order);
        
        // 记录状态变更日志
        orderStatusLogRepository.save(
            OrderStatusLog.builder()
                .orderId(orderId)
                .fromStatus(order.getStatus())
                .toStatus(newStatus)
                .event(event)
                .build()
        );
        
        // 发布领域事件
        eventPublisher.publish(orderId, event, order);
    }
}
```

---

## 三、平台对接层

### 适配器模式

```
┌──────────────────────────────────────────┐
│            PlatformAdapter               │
│              (接口层)                     │
├──────────────────────────────────────────┤
│  + syncProducts()                        │
│  + syncOrders()                          │
│  + pushInventory()                       │
│  + createShipment()                      │
│  + getSettlement()                       │
└──────────────┬───────────────────────────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│Amazon  │ │Shopee  │ │TikTok  │
│Adapter │ │Adapter │ │Adapter │
└────────┘ └────────┘ └────────┘
```

### 限流/重试/幂等

```java
// 1. 限流 - 防止超出平台API调用限制
@RateLimiter(name = "amazonApi", fallbackMethod = "fallback")
public List<Order> fetchAmazonOrders(String shopId, LocalDateTime since) {
    return amazonApiClient.getOrders(shopId, since);
}

// 2. 重试 - 网络异常自动重试
@Retryable(value = {ApiException.class}, 
           maxAttempts = 3,
           backoff = @Backoff(delay = 1000, multiplier = 2))
public Product pushProduct(String platform, ProductDTO dto) {
    return getAdapter(platform).createProduct(dto);
}

// 3. 幂等 - 防止重复处理
public void processOrderEvent(PlatformOrderEvent event) {
    // 以平台订单号+事件ID做幂等键
    String idempotentKey = event.getPlatform() + ":" + event.getOrderId();
    if (idempotentRepository.existsByKey(idempotentKey)) {
        log.info("重复事件,跳过: {}", idempotentKey);
        return;
    }
    // 处理业务...
    idempotentRepository.save(new IdempotentRecord(idempotentKey));
}
```

### 各平台 API 限流

| 平台 | 限流规则 | 应对 |
|------|---------|------|
| Amazon SP-API | 按API分组，如5请求/秒 | 令牌桶 + 队列排队 |
| Shopee | 1000次/分钟/店铺 | 计数器限流 |
| TikTok | 按API不同，通常100次/分钟 | 令牌桶 |

---

## 四、数据同步

### 拉取（Pull）— 平台数据到本地

```
┌─────────────┐                    ┌─────────────┐
│   平台API    │ ──── 增量拉取 ────→ │   本地系统   │
│ (数据源)     │                    │             │
└─────────────┘                    └─────────────┘

策略:
  ├── 定时全量: 每日全量同步(兜底)
  ├── 增量拉取: 按更新时间增量拉取(主力)
  └── Webhook: 平台主动推送(实时性最高)
```

### 推送（Push）— 本地数据到平台

```
┌─────────────┐                    ┌─────────────┐
│   本地系统   │ ──── 主动推送 ────→ │   平台API    │
│ (数据源)     │                    │             │
└─────────────┘                    └─────────────┘

场景:
  ├── 商品上架/更新 → 推送到平台
  ├── 库存变更 → 推送库存更新
  ├── 价格变更 → 推送价格更新
  ├── 发货信息 → 推送追踪号
```

### 增量同步策略

```java
@Service
public class OrderSyncService {
    
    /**
     * 增量拉取订单
     */
    @Scheduled(fixedRate = 300000) // 每5分钟
    public void incrementalSync() {
        for (Shop shop : shopRepository.findAllActive()) {
            // 获取上次同步时间
            LocalDateTime lastSyncTime = syncRecordRepository
                .getLastSyncTime(shop.getId(), "ORDER");
            
            // 增量拉取
            PlatformAdapter adapter = adapterFactory.getAdapter(shop.getPlatform());
            List<OrderDTO> orders = adapter.fetchOrders(
                shop.getShopId(), lastSyncTime, LocalDateTime.now());
            
            // 逐单处理(幂等)
            for (OrderDTO dto : orders) {
                processOrder(shop, dto);
            }
            
            // 更新同步时间
            syncRecordRepository.updateLastSyncTime(
                shop.getId(), "ORDER", LocalDateTime.now());
        }
    }
}
```

---

## 五、消息与异步

### 最终一致性

```
场景: 订单付款 → 扣库存 → 创建发货单 → 物流下单

强一致性(2PC): 不适合，跨服务事务太重
最终一致性: 

  1. 订单服务: 更新订单状态为PAID + 发送事件
  2. 库存服务: 消费事件 → 扣减库存 + 发送事件
  3. 物流服务: 消费事件 → 创建发货单

  如果库存扣减失败:
  ├── 重试3次
  ├── 仍失败 → 进入死信队列 → 人工处理
  └── 或: 回滚订单状态(补偿)
```

### 本地消息表

```java
/**
 * 保证消息一定发出的方案: 本地消息表
 * 
 * 1. 业务操作 + 写消息表 在同一个事务中
 * 2. 定时任务扫描消息表，发送未发出的消息
 * 3. 发送成功后标记已发送
 */
@Transactional
public void createOrder(OrderDTO dto) {
    // 1. 业务操作
    Order order = orderRepository.save(dto.toOrder());
    
    // 2. 写本地消息表(同一事务)
    OutboxMessage message = OutboxMessage.builder()
        .aggregateId(order.getId())
        .aggregateType("Order")
        .eventType("order.created")
        .payload(JsonUtils.toJson(order))
        .status("PENDING")
        .createdAt(LocalDateTime.now())
        .build();
    outboxRepository.save(message);
}

// 定时扫描发送
@Scheduled(fixedRate = 5000)
public void sendPendingMessages() {
    List<OutboxMessage> messages = outboxRepository
        .findByStatus("PENDING");
    for (OutboxMessage msg : messages) {
        try {
            mqTemplate.send(msg.getEventType(), msg.getPayload());
            msg.setStatus("SENT");
        } catch (Exception e) {
            msg.setRetryCount(msg.getRetryCount() + 1);
            if (msg.getRetryCount() > 3) {
                msg.setStatus("FAILED");
            }
        }
        outboxRepository.save(msg);
    }
}
```

### TCC (Try-Confirm-Cancel)

```
场景: 下单扣库存(跨服务)

Try阶段:
  ├── 订单服务: 创建订单(待确认)
  └── 库存服务: 冻结库存(frozen_qty += n)

Confirm阶段(全部Try成功):
  ├── 订单服务: 确认订单
  └── 库存服务: 扣减库存(available -= n, frozen -= n)

Cancel阶段(任一Try失败):
  ├── 订单服务: 取消订单
  └── 库存服务: 释放冻结(frozen -= n, available += n)
```

---

## 六、报表与 BI

### 技术架构

```
┌──────────┐   CDC    ┌──────────┐   同步    ┌──────────────┐
│ MySQL    │────────→ │  Kafka   │────────→ │ ClickHouse   │
│ (OLTP)   │  (binlog)│          │          │ (OLAP)       │
└──────────┘          └──────────┘          └──────┬───────┘
                                                   │
                                                   ▼
                                            ┌──────────────┐
                                            │  Metabase/   │
                                            │  Grafana     │
                                            │  (BI看板)     │
                                            └──────────────┘
```

### ClickHouse 建表示例

```sql
-- 订单事实表(宽表)
CREATE TABLE fact_order (
    order_id UInt64,
    order_date Date,
    platform String,
    shop_id UInt64,
    sku_id UInt64,
    country String,
    quantity UInt32,
    sale_amount Decimal(12,2),
    cost_amount Decimal(12,2),
    commission Decimal(12,2),
    fba_fee Decimal(12,2),
    shipping_fee Decimal(12,2),
    ad_fee Decimal(12,2),
    net_profit Decimal(12,2),
    profit_rate Decimal(5,2)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(order_date)
ORDER BY (platform, shop_id, order_date, sku_id);

-- 利润查询示例
SELECT 
    platform,
    country,
    SUM(sale_amount) as total_sales,
    SUM(net_profit) as total_profit,
    AVG(profit_rate) as avg_profit_rate
FROM fact_order
WHERE order_date BETWEEN '2024-06-01' AND '2024-06-30'
GROUP BY platform, country
ORDER BY total_profit DESC;
```

---

## 七、关键技术难点与解决方案

| 难点 | 解决方案 |
|------|---------|
| 多平台API限流 | 令牌桶限流 + 请求队列 + 优先级调度 |
| 库存超卖 | Redis原子预扣 + DB乐观锁 + 定期对账 |
| 订单状态一致性 | 状态机 + 事件驱动 + 补偿事务 |
| 跨服务事务 | 本地消息表 + 最终一致性 |
| 海量数据查询 | MySQL分库分表 + ClickHouse OLAP |
| 实时数据同步 | Canal/Debezium CDC + Kafka |
| 面单PDF生成 | 模板引擎 + 异步生成 + OSS存储 |
| 多币种计算 | 统一汇率服务 + 本位币折算 |

---

## 八、面试/实战高频问题

1. **库存预扣用Redis还是MySQL？**
   - Redis做实时预扣（高性能），MySQL做最终持久化（可靠）。Redis和MySQL定时对账保证一致性。

2. **平台API经常超时怎么处理？**
   - 超时重试（指数退避）+ 降级（返回缓存数据）+ 告警 + 异步化（不阻塞主流程）。

3. **如何保证跨服务数据一致性？**
   - 优先用最终一致性（本地消息表+MQ）。强一致需求用TCC/Saga，但复杂度高，慎用。

4. **ClickHouse和MySQL怎么分工？**
   - MySQL存业务数据(OLTP)，ClickHouse存分析数据(OLAP)。通过CDC同步。报表查询走ClickHouse，业务操作走MySQL。

5. **如何设计高可用的平台对接层？**
   - 熔断(Sentinel)防雪崩 + 降级(返回缓存) + 限流(令牌桶) + 重试(指数退避) + 幂等(防重复) + 监控告警。

---

## 九、推荐资源

| 资源 | 说明 |
|------|------|
| Seata 官方文档 | 分布式事务 |
| Canal/Debezium | MySQL CDC工具 |
| ClickHouse 文档 | OLAP引擎 |
| Spring Cloud Gateway | API网关 |
| 《微服务架构设计模式》 | 微服务设计 |
