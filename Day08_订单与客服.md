# Day08 — 订单与客服

---

## 一、订单生命周期与状态机

### 订单状态流转

```
待付款(PENDING_PAYMENT)
    │
    ├── 超时未付 ──→ 已取消(CANCELLED)
    │
    ▼ 付款成功
已付款(PAID)
    │
    ▼ 审核通过
待发货(PENDING_SHIPMENT)
    │
    ├── 缺货 ──→ 缺货等待(OUT_OF_STOCK) → 补货后 → 待发货
    │
    ▼ 创建发货单
已发货(SHIPPED)
    │
    ├── 部分妥投 ──→ 部分完成(PARTIAL_COMPLETED)
    │
    ▼ 全部妥投
已完成(COMPLETED)
    │
    ├── 退货申请 ──→ 退货中(RETURNING) → 退货完成(REFUNDED)
    │
    ▼ 超过售后期
已关闭(CLOSED)
```

### 状态机设计原则

1. **状态不可回退**：只能向前流转或取消，不允许从"已发货"回到"待付款"
2. **触发条件明确**：每个状态转换有明确的触发事件
3. **幂等性**：同一事件重复触发不产生副作用
4. **可追溯**：每次状态变更记录日志（谁、何时、什么操作）

---

## 二、多平台订单汇聚与标准化

### 问题

各平台订单数据结构不同：

```
Amazon Order:
  amazon_order_id, order_status, buyer_email, 
  order_total(amount, currency), 
  order_items(asin, quantity_ordered, item_price)

Shopee Order:
  ordersn, order_status, buyer_userid,
  total_amount, currency,
  item_list(item_id, model_id, quantity, item_price)

TikTok Order:
  order_id, order_status, buyer_info,
  payment_info(total_amount, currency),
  line_items(sku_id, quantity, sale_price)
```

### 标准化方案

```
┌────────┐  ┌────────┐  ┌────────┐
│Amazon  │  │Shopee  │  │TikTok  │
│Adapter │  │Adapter │  │Adapter │
└───┬────┘  └───┬────┘  └───┬────┘
    │           │           │
    ▼           ▼           ▼
┌─────────────────────────────────┐
│       统一订单模型(OrderDTO)      │
│  orderId / platformOrderNo      │
│  platform / shopId              │
│  buyerInfo / shippingAddress    │
│  items[] / paymentInfo          │
│  status / createdAt             │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│         本地订单系统              │
│  入库、状态映射、后续处理         │
└─────────────────────────────────┘
```

### 状态映射

| 平台状态 | Amazon | Shopee | TikTok | 本地统一状态 |
|---------|--------|--------|--------|------------|
| 待付款 | Pending | UNPAID | UNPAID | PENDING_PAYMENT |
| 已付款 | Unshipped | READY_TO_SHIP | AWAITING_SHIPMENT | PAID |
| 已发货 | Shipped | SHIPPED | SHIPPED | SHIPPED |
| 已完成 | Delivered | COMPLETED | COMPLETED | COMPLETED |
| 已取消 | Cancelled | CANCELLED | CANCELLED | CANCELLED |

---

## 三、订单拆合规则

### 拆单场景

```
场景1: 一单多SKU分布在不同仓库
  订单: SKU-A(美国仓) + SKU-B(英国仓)
  → 拆为2个子单，分别从不同仓库发货

场景2: 一单含FBA商品和FBM商品
  订单: SKU-A(FBA库存) + SKU-B(海外仓库存)
  → 拆为FBA发货 + 海外仓发货

场景3: 大件+小件混单
  订单: SKU-A(大件需海运) + SKU-B(小件走快递)
  → 拆为不同物流渠道发货
```

### 合单场景

```
场景1: 同一买家同地址多单
  买家在短时间内下了3单，收货地址相同
  → 合并为一个包裹发货，节省运费

场景2: 补发单与原单合并
  买家原订单缺货部分，补货后与后续订单合并
```

### 拆合规则引擎

```java
public class OrderSplitStrategy {
    
    public List<SubOrder> split(Order order) {
        // 按仓库分组
        Map<Warehouse, List<OrderItem>> warehouseGroups = order.getItems().stream()
            .collect(Collectors.groupingBy(item -> inventoryService
                .getAvailableWarehouse(item.getSkuId(), item.getQuantity())));
        
        // 按物流渠道分组（同仓内可能还需按渠道拆）
        // ...
        
        return warehouseGroups.entrySet().stream()
            .map(e -> SubOrder.builder()
                .parentOrderId(order.getId())
                .warehouseId(e.getKey().getId())
                .items(e.getValue())
                .build())
            .collect(Collectors.toList());
    }
}
```

---

## 四、订单路由算法

### 路由决策因素

```
┌────────────────────────────────────────┐
│           订单路由决策树                │
│                                        │
│  1. 哪个仓有货？                       │
│     └── 优先选择有足够库存的仓库        │
│                                        │
│  2. 哪个渠道最快？                     │
│     └── 按目的国+仓位置匹配最优渠道     │
│                                        │
│  3. 哪个方案最便宜？                    │
│     └── 多个可用方案中选运费最低的       │
│                                        │
│  4. 平台要求？                         │
│     └── FBA订单必须从FBA仓发           │
│                                        │
│  综合打分 = 时效分×0.4 + 成本分×0.3    │
│           + 服务分×0.2 + 库存分×0.1    │
└────────────────────────────────────────┘
```

### 路由算法示例

```java
public class OrderRouter {
    
    public RouteResult route(Order order) {
        // 1. 查找有库存的仓库
        List<Warehouse> availableWarehouses = inventoryService
            .findAvailableWarehouses(order.getItems());
        
        if (availableWarehouses.isEmpty()) {
            return RouteResult.outOfStock();
        }
        
        // 2. 对每个仓库+渠道组合打分
        List<RouteCandidate> candidates = new ArrayList<>();
        for (Warehouse wh : availableWarehouses) {
            List<LogisticsChannel> channels = logisticsService
                .getAvailableChannels(wh, order.getDestCountry());
            for (LogisticsChannel ch : channels) {
                double score = calculateScore(wh, ch, order);
                candidates.add(new RouteCandidate(wh, ch, score));
            }
        }
        
        // 3. 选择最高分方案
        return candidates.stream()
            .max(Comparator.comparingDouble(RouteCandidate::getScore))
            .map(c -> RouteResult.of(c.getWarehouse(), c.getChannel()))
            .orElse(RouteResult.noAvailableRoute());
    }
    
    private double calculateScore(Warehouse wh, LogisticsChannel ch, Order order) {
        double timeScore = 100.0 / ch.getEstimatedDays();  // 越快越高
        double costScore = 100.0 / ch.getEstimatedFee();    // 越便宜越高
        double invScore = inventoryService.getStockRatio(wh, order); // 库存充足度
        return timeScore * 0.4 + costScore * 0.3 + invScore * 0.3;
    }
}
```

---

## 五、客服场景与工单系统

### 常见客服场景

| 场景 | 触发 | 处理 |
|------|------|------|
| 物流查询 | 买家问"包裹到哪了" | 查追踪号→返回轨迹 |
| 退款申请 | 商品问题/未收到 | 审核→退款/拒绝 |
| 退货申请 | 不满意/质量问题 | 审核→提供退货地址→收货→退款 |
| 纠纷/Claim | 买家开纠纷 | 举证→协商/平台仲裁 |
| 差评跟进 | 买家留差评 | 联系买家→协商解决→修改评价 |
| 修改地址 | 买家要求改地址 | 订单未发货→修改；已发货→联系物流 |
| 开发票 | 买家要发票 | 生成商业发票→发送 |

### 工单数据模型

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| ticket_no | VARCHAR(32) | 工单号 |
| order_id | BIGINT | 关联订单 |
| platform | VARCHAR(32) | 来源平台 |
| type | VARCHAR(32) | 类型(REFUND/RETURN/COMPLAINT/INQUIRY) |
| priority | TINYINT | 优先级(1-5) |
| status | VARCHAR(32) | 状态(OPEN/IN_PROGRESS/RESOLVED/CLOSED) |
| assignee_id | BIGINT | 处理人 |
| content | TEXT | 内容 |
| resolution | TEXT | 解决方案 |

---

## 六、退款退货流程

### 退款类型

| 类型 | 说明 | 库存影响 |
|------|------|---------|
| 仅退款 | 不退货直接退款 | 无影响 |
| 退货退款 | 退回商品并退款 | 退回后质检→入库/残次 |
| 换货 | 换一个新商品 | 新商品扣库存 |
| 部分退款 | 退部分金额，保留商品 | 无影响 |

### 退款流程

```
买家申请退款
     │
     ▼
系统自动审核规则:
  ├── 金额 < $5 且 签收 < 7天 → 自动同意(仅退款)
  ├── 金额 > $50 → 人工审核
  └── 其他 → 自动审核/人工审核
     │
     ▼
卖家审核
   ┌────┴────┐
   ▼         ▼
 同意       拒绝(需理由)
   │         │
   ▼         ▼
 执行退款   买家可升级纠纷
   │
   ├── 仅退款 → 调用平台退款API → 更新订单状态
   └── 退货退款 → 买家寄回 → 仓库收货 → 质检 → 退款
```

---

## 七、订单状态机代码示例

### Spring Statemachine 配置

```java
@Configuration
@EnableStateMachineFactory
public class OrderStateMachineConfig 
    extends EnumStateMachineConfigurerAdapter<OrderStatus, OrderEvent> {
    
    @Override
    public void configure(StateMachineStateConfigurer<OrderStatus, OrderEvent> states) 
        throws Exception {
        states.withStates()
            .initial(OrderStatus.PENDING_PAYMENT)
            .states(EnumSet.allOf(OrderStatus.class))
            .end(OrderStatus.CANCELLED)
            .end(OrderStatus.CLOSED);
    }
    
    @Override
    public void configure(StateMachineTransitionConfigurer<OrderStatus, OrderEvent> transitions) 
        throws Exception {
        transitions
            // 待付款 → 已付款
            .withExternal()
                .source(OrderStatus.PENDING_PAYMENT)
                .target(OrderStatus.PAID)
                .event(OrderEvent.PAY)
            // 已付款 → 待发货
            .and()
            .withExternal()
                .source(OrderStatus.PAID)
                .target(OrderStatus.PENDING_SHIPMENT)
                .event(OrderEvent.APPROVE)
            // 待发货 → 已发货
            .and()
            .withExternal()
                .source(OrderStatus.PENDING_SHIPMENT)
                .target(OrderStatus.SHIPPED)
                .event(OrderEvent.SHIP)
            // 已发货 → 已完成
            .and()
            .withExternal()
                .source(OrderStatus.SHIPPED)
                .target(OrderStatus.COMPLETED)
                .event(OrderEvent.DELIVER)
            // 任何状态 → 已取消
            .and()
            .withExternal()
                .source(OrderStatus.PENDING_PAYMENT)
                .target(OrderStatus.CANCELLED)
                .event(OrderEvent.CANCEL);
    }
}

// 状态枚举
public enum OrderStatus {
    PENDING_PAYMENT, PAID, PENDING_SHIPMENT, 
    OUT_OF_STOCK, SHIPPED, PARTIAL_COMPLETED, 
    COMPLETED, RETURNING, REFUNDED, 
    CANCELLED, CLOSED
}

// 事件枚举
public enum OrderEvent {
    PAY, APPROVE, SHIP, DELIVER, 
    CANCEL, RETURN, REFUND
}
```

---

## 八、订单数据模型

### 订单主表 (orders)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| order_no | VARCHAR(32) | 系统订单号 |
| platform | VARCHAR(32) | 平台 |
| platform_order_no | VARCHAR(64) | 平台订单号 |
| shop_id | BIGINT | 店铺ID |
| buyer_id | VARCHAR(64) | 买家ID |
| status | VARCHAR(32) | 订单状态 |
| currency | VARCHAR(3) | 币种 |
| total_amount | DECIMAL(12,2) | 订单总金额 |
| item_amount | DECIMAL(12,2) | 商品金额 |
| shipping_amount | DECIMAL(12,2) | 运费 |
| discount_amount | DECIMAL(12,2) | 优惠金额 |
| ordered_at | DATETIME | 下单时间(平台) |
| paid_at | DATETIME | 付款时间 |
| shipped_at | DATETIME | 发货时间 |
| delivered_at | DATETIME | 妥投时间 |

### 订单明细表 (order_item)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| order_id | BIGINT | 关联订单 |
| sku_id | BIGINT | 本地SKU ID |
| platform_sku_id | VARCHAR(64) | 平台SKU ID |
| product_name | VARCHAR(256) | 商品名称 |
| quantity | INT | 数量 |
| unit_price | DECIMAL(12,2) | 单价 |
| total_price | DECIMAL(12,2) | 小计 |

### 收货地址表 (order_shipping_address)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| order_id | BIGINT | 关联订单 |
| name | VARCHAR(128) | 收件人 |
| phone | VARCHAR(32) | 电话 |
| address1 | VARCHAR(256) | 地址行1 |
| address2 | VARCHAR(256) | 地址行2 |
| city | VARCHAR(64) | 城市 |
| state | VARCHAR(64) | 州/省 |
| postal_code | VARCHAR(16) | 邮编 |
| country | VARCHAR(3) | 国家代码 |

### 退款单表 (order_refund)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| refund_no | VARCHAR(32) | 退款单号 |
| order_id | BIGINT | 关联订单 |
| order_item_id | BIGINT | 关联明细(部分退款) |
| type | VARCHAR(16) | 类型(REFUND_ONLY/RETURN_REFUND/EXCHANGE) |
| reason | VARCHAR(256) | 退款原因 |
| amount | DECIMAL(12,2) | 退款金额 |
| status | VARCHAR(32) | 状态 |
| approved_by | BIGINT | 审批人 |
| refunded_at | DATETIME | 退款完成时间 |

---

## 九、面试/实战高频问题

1. **多平台订单如何保证不重复拉取？**
   - 以平台订单号为唯一键做幂等。拉取前查询本地是否已存在该platform_order_no，存在则更新而非新增。

2. **订单拆单后如何关联？**
   - 父子订单关系：主订单(parent_order_id=NULL) → 子订单(parent_order_id=主订单ID)。财务统计按主订单维度。

3. **如何设计高并发下的库存扣减？**
   - Redis预扣(原子操作DECR) + 数据库最终一致性。下单时Redis预扣，支付后异步写DB。Redis与DB定时对账。

4. **状态机为什么比if-else好？**
   - 状态机将合法转换明确定义，非法转换直接拒绝，不会出现"已发货→待付款"这种bug。新增状态只需添加转换规则，无需修改业务代码。

5. **退款退货如何影响库存和财务？**
   - 仅退款：库存不变，财务记退款支出；退货退款：库存回补(质检后)，财务记退款+退货运费。需注意退货商品质检结果决定是否可重新上架。

---

## 十、推荐资源

| 资源 | 说明 |
|------|------|
| Spring Statemachine 文档 | 状态机框架 |
| Amazon Orders API | 订单接口参考 |
| Shopee Order API | Shopee订单接口 |
| 《领域驱动设计》 | 状态机+领域建模 |
| B站"电商订单系统设计" | 视频教程 |
