# Day14 — 核心数据模型与设计模式

---

## 一、商品模型 ER 图

```
┌──────────────┐     ┌──────────────┐
│  product_spu │     │ product_sku  │
├──────────────┤     ├──────────────┤
│ id (PK)      │←──┐ │ id (PK)      │
│ spu_code     │   │ │ spu_id (FK)  │──→
│ name         │   └─│ sku_code     │
│ brand_id     │     │ barcode      │
│ category_id  │     │ purchase_price│
│ status       │     │ cost_price   │
│ description  │     │ weight       │
└──────┬───────┘     │ attrs_json   │
       │             │ status       │
       │             └──────┬───────┘
       │                    │
       ▼                    ▼
┌──────────────┐     ┌──────────────────┐
│spu_i18n      │     │platform_mapping  │
├──────────────┤     ├──────────────────┤
│ id (PK)      │     │ id (PK)          │
│ spu_id (FK)  │     │ sku_id (FK)      │
│ locale       │     │ platform         │
│ name         │     │ shop_id          │
│ title        │     │ platform_sku_id  │
│ bullet_points│     │ sync_status      │
│ description  │     │ last_sync_at     │
└──────────────┘     └──────────────────┘

┌──────────────┐     ┌──────────────┐
│ brand        │     │ category     │
├──────────────┤     ├──────────────┤
│ id (PK)      │     │ id (PK)      │
│ name         │     │ name         │
│ name_en      │     │ parent_id    │
│ status       │     │ level        │
└──────────────┘     │ path         │
                     └──────────────┘
```

---

## 二、订单模型 ER 图

```
┌──────────────┐     ┌──────────────┐
│   orders     │     │  order_item  │
├──────────────┤     ├──────────────┤
│ id (PK)      │←──┐ │ id (PK)      │
│ order_no     │   └─│ order_id (FK)│
│ platform     │     │ sku_id       │
│ platform_no  │     │ product_name │
│ shop_id      │     │ quantity     │
│ status       │     │ unit_price   │
│ currency     │     │ total_price  │
│ total_amount │     └──────────────┘
│ item_amount  │
│ ship_amount  │     ┌──────────────────┐
│ discount_amt │     │order_ship_address│
│ ordered_at   │     ├──────────────────┤
│ paid_at      │     │ id (PK)          │
│ shipped_at   │     │ order_id (FK)    │
│ delivered_at │     │ name             │
└──────┬───────┘     │ phone            │
       │             │ address1/2       │
       │             │ city/state/zip   │
       │             │ country          │
       │             └──────────────────┘
       │
       │     ┌──────────────┐
       └────→│ order_refund │
             ├──────────────┤
             │ id (PK)      │
             │ order_id(FK) │
             │ item_id (FK) │
             │ type         │
             │ reason       │
             │ amount       │
             │ status       │
             └──────────────┘
```

---

## 三、库存模型 ER 图

```
┌──────────────┐     ┌──────────────┐
│  warehouse   │     │ wh_location  │
├──────────────┤     ├──────────────┤
│ id (PK)      │←──┐ │ id (PK)      │
│ wh_code      │   └─│ warehouse_id │
│ name         │     │ location_code│
│ type         │     │ zone         │
│ country      │     │ type         │
│ status       │     │ status       │
└──────────────┘     └──────────────┘

┌──────────────────┐     ┌──────────────────┐
│inventory_balance │     │inventory_txn     │
├──────────────────┤     ├──────────────────┤
│ id (PK)          │     │ id (PK)          │
│ warehouse_id     │     │ warehouse_id     │
│ sku_id           │     │ sku_id           │
│ available_qty    │     │ txn_type         │
│ reserved_qty     │     │ qty              │
│ defective_qty    │     │ ref_type         │
│ frozen_qty       │     │ ref_id           │
│ in_transit_qty   │     │ balance_after    │
│ updated_at       │     │ created_at       │
└──────────────────┘     └──────────────────┘
     UK: (warehouse_id, sku_id)

┌──────────────────┐
│inv_transfer      │
├──────────────────┤
│ id (PK)          │
│ transfer_no      │
│ from_wh_id       │
│ to_wh_id         │
│ status           │
└──────────────────┘
```

---

## 四、财务模型 ER 图

```
┌──────────────────┐     ┌──────────────────┐
│finance_settlement │     │ finance_fee_detail│
├──────────────────┤     ├──────────────────┤
│ id (PK)          │←──┐ │ id (PK)          │
│ settlement_no    │   └─│ settlement_id(FK)│
│ platform         │     │ order_id         │
│ shop_id          │     │ order_item_id    │
│ start_date       │     │ sku_id           │
│ end_date         │     │ fee_type         │
│ deposit_date     │     │ amount           │
│ currency         │     │ currency         │
│ total_amount     │     │ alloc_method     │
│ net_amount       │     └──────────────────┘
│ status           │
└──────────────────┘     ┌──────────────────┐
                         │finance_transaction│
┌──────────────────┐     ├──────────────────┤
│finance_reconcili │     │ id (PK)          │
│ation             │     │ txn_no           │
├──────────────────┤     │ txn_type         │
│ id (PK)          │     │ amount           │
│ recon_no         │     │ currency         │
│ type             │     │ base_amount      │
│ counterparty     │     │ order_id         │
│ system_amount    │     │ sku_id           │
│ cp_amount        │     │ account_type     │
│ diff_amount      │     │ txn_time         │
│ status           │     └──────────────────┘
└──────────────────┘
```

---

## 五、常用设计模式在电商中的应用

### 策略模式 — 物流路由

```java
// 物流路由策略接口
public interface ShippingStrategy {
    RouteResult route(Order order);
    boolean supports(OrderContext context);
}

// 最快到达策略
@Component
public class FastestStrategy implements ShippingStrategy {
    public RouteResult route(Order order) {
        // 选择时效最短的渠道
    }
    public boolean supports(OrderContext ctx) {
        return ctx.getPriority() == Priority.URGENT;
    }
}

// 最低成本策略
@Component
public class CheapestStrategy implements ShippingStrategy {
    public RouteResult route(Order order) {
        // 选择运费最低的渠道
    }
    public boolean supports(OrderContext ctx) {
        return ctx.getPriority() == Priority.ECONOMY;
    }
}

// 策略选择器
@Service
public class ShippingStrategySelector {
    @Autowired private List<ShippingStrategy> strategies;
    
    public ShippingStrategy select(OrderContext ctx) {
        return strategies.stream()
            .filter(s -> s.supports(ctx))
            .findFirst()
            .orElseThrow();
    }
}
```

### 工厂模式 — 平台适配器

```java
// 适配器工厂
@Component
public class PlatformAdapterFactory {
    
    @Autowired
    private Map<String, PlatformAdapter> adapterMap;
    
    public PlatformAdapter getAdapter(String platform) {
        PlatformAdapter adapter = adapterMap.get(platform + "Adapter");
        if (adapter == null) {
            throw new BusinessException("不支持的平台: " + platform);
        }
        return adapter;
    }
}

// 使用
PlatformAdapter adapter = factory.getAdapter("amazon");
List<OrderDTO> orders = adapter.fetchOrders(shopId, since);
```

### 状态模式 — 订单状态机

```java
// 状态接口
public interface OrderState {
    OrderState pay();
    OrderState ship();
    OrderState deliver();
    OrderState cancel();
}

// 具体状态
public class PendingPaymentState implements OrderState {
    public OrderState pay() { return new PaidState(); }
    public OrderState ship() { throw new IllegalStateException(); }
    public OrderState deliver() { throw new IllegalStateException(); }
    public OrderState cancel() { return new CancelledState(); }
}

public class PaidState implements OrderState {
    public OrderState pay() { throw new IllegalStateException(); }
    public OrderState ship() { return new ShippedState(); }
    public OrderState deliver() { throw new IllegalStateException(); }
    public OrderState cancel() { return new CancelledState(); }
}
```

### 观察者模式 — 事件通知

```java
// 事件发布
@Service
public class OrderEventPublisher {
    @Autowired private ApplicationEventPublisher publisher;
    
    public void publishOrderCreated(Order order) {
        publisher.publishEvent(new OrderCreatedEvent(order));
    }
}

// 事件监听
@Component
public class InventoryEventListener {
    @EventListener
    @Async
    public void onOrderCreated(OrderCreatedEvent event) {
        inventoryService.reserveStock(event.getOrder());
    }
}

@Component
public class NotificationEventListener {
    @EventListener
    @Async
    public void onOrderCreated(OrderCreatedEvent event) {
        notificationService.notifyNewOrder(event.getOrder());
    }
}
```

### 适配器模式 — 多物流商对接

```java
// 统一接口
public interface LogisticsAdapter {
    ShipmentResult createShipment(ShipmentRequest req);
    LabelResult getLabel(String trackingNo);
    TrackResult track(String trackingNo);
}

// 各物流商实现
@Component("yuntuAdapter")
public class YuntuAdapter implements LogisticsAdapter { ... }

@Component("fourPxAdapter")  
public class FourPxAdapter implements LogisticsAdapter { ... }
```

### 模板方法模式 — 同步流程

```java
// 同步模板
public abstract class AbstractSyncService<T> {
    
    public final void sync(LocalDateTime since) {
        // 1. 拉取数据(子类实现)
        List<T> dataList = fetchData(since);
        
        // 2. 数据转换(子类实现)
        List<UnifiedData> unifiedList = convert(dataList);
        
        // 3. 幂等处理(通用逻辑)
        for (UnifiedData data : unifiedList) {
            if (!exists(data.getId())) {
                save(data);
            } else {
                update(data);
            }
        }
        
        // 4. 后处理(子类可选实现)
        postProcess(unifiedList);
    }
    
    protected abstract List<T> fetchData(LocalDateTime since);
    protected abstract List<UnifiedData> convert(List<T> dataList);
    protected void postProcess(List<UnifiedData> list) { /* 默认空实现 */ }
}
```

---

## 六、分库分表策略

### 分表维度选择

| 维度 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| 按店铺(shop_id) | 多租户 | 隔离性好 | 数据倾斜 |
| 按时间(月) | 订单/流水 | 冷热分离 | 跨月查询复杂 |
| 按平台(platform) | 平台隔离 | 简单 | 数据倾斜 |

### 推荐方案

```
订单表: 按月分表
  ├── order_202401, order_202402, ...
  └── 冷热分离: 近3个月热表 + 历史归档

库存表: 不分表(数据量可控)
  └── 按仓库+SKU维度，量级: 万级SKU × 十级仓库 = 十万级

财务流水表: 按月分表
  ├── finance_txn_202401, ...
  └── 数据量大且有时效性

商品表: 不分表
  └── SKU量级通常在万-百万级，单表可承载
```

### ShardingSphere 配置

```yaml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          orders:
            actual-data-nodes: ds0.orders_$->{202401..202412}
            table-strategy:
              standard:
                sharding-column: ordered_at
                sharding-algorithm-name: orders-inline
        sharding-algorithms:
          orders-inline:
            type: INLINE
            props:
              algorithm-expression: orders_$->{202401..202412}[ordered_at.format('yyyyMM')]
```

---

## 七、建表 SQL 示例

```sql
-- ============ 商品域 ============

CREATE TABLE product_spu (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    spu_code VARCHAR(64) NOT NULL UNIQUE,
    name VARCHAR(256) NOT NULL,
    brand_id BIGINT,
    category_id BIGINT,
    status TINYINT NOT NULL DEFAULT 0 COMMENT '0草稿/1上架/2下架',
    description TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    tenant_id BIGINT NOT NULL,
    INDEX idx_brand (brand_id),
    INDEX idx_category (category_id),
    INDEX idx_tenant (tenant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE product_sku (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    spu_id BIGINT NOT NULL,
    sku_code VARCHAR(64) NOT NULL UNIQUE,
    barcode VARCHAR(64),
    purchase_price DECIMAL(12,2),
    cost_price DECIMAL(12,2),
    weight DECIMAL(10,3),
    length DECIMAL(10,3),
    width DECIMAL(10,3),
    height DECIMAL(10,3),
    attrs_json JSON,
    status TINYINT NOT NULL DEFAULT 0,
    tenant_id BIGINT NOT NULL,
    INDEX idx_spu (spu_id),
    INDEX idx_tenant (tenant_id),
    FOREIGN KEY (spu_id) REFERENCES product_spu(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ============ 订单域 ============

CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_no VARCHAR(32) NOT NULL UNIQUE,
    platform VARCHAR(32) NOT NULL,
    platform_order_no VARCHAR(64) NOT NULL,
    shop_id BIGINT NOT NULL,
    status VARCHAR(32) NOT NULL DEFAULT 'PENDING_PAYMENT',
    currency VARCHAR(3) NOT NULL,
    total_amount DECIMAL(12,2) NOT NULL,
    item_amount DECIMAL(12,2),
    shipping_amount DECIMAL(12,2),
    discount_amount DECIMAL(12,2) DEFAULT 0,
    ordered_at DATETIME,
    paid_at DATETIME,
    shipped_at DATETIME,
    delivered_at DATETIME,
    tenant_id BIGINT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_platform_order (platform, platform_order_no),
    INDEX idx_shop (shop_id),
    INDEX idx_status (status),
    INDEX idx_ordered (ordered_at),
    INDEX idx_tenant (tenant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ============ 库存域 ============

CREATE TABLE inventory_balance (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    warehouse_id BIGINT NOT NULL,
    sku_id BIGINT NOT NULL,
    available_qty INT NOT NULL DEFAULT 0,
    reserved_qty INT NOT NULL DEFAULT 0,
    defective_qty INT NOT NULL DEFAULT 0,
    frozen_qty INT NOT NULL DEFAULT 0,
    in_transit_qty INT NOT NULL DEFAULT 0,
    version INT NOT NULL DEFAULT 0 COMMENT '乐观锁版本号',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    tenant_id BIGINT NOT NULL,
    UNIQUE KEY uk_wh_sku (warehouse_id, sku_id),
    INDEX idx_tenant (tenant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ============ 财务域 ============

CREATE TABLE finance_transaction (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    txn_no VARCHAR(32) NOT NULL UNIQUE,
    txn_type VARCHAR(32) NOT NULL COMMENT 'REVENUE/REFUND/FEE/PAYMENT/RECEIPT',
    amount DECIMAL(12,2) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    exchange_rate DECIMAL(10,4),
    base_amount DECIMAL(12,2) COMMENT '本位币金额',
    order_id BIGINT,
    sku_id BIGINT,
    settlement_id BIGINT,
    account_type VARCHAR(16),
    counterparty VARCHAR(128),
    txn_time DATETIME NOT NULL,
    tenant_id BIGINT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_type (txn_type),
    INDEX idx_order (order_id),
    INDEX idx_txn_time (txn_time),
    INDEX idx_tenant (tenant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 八、面试/实战高频问题

1. **商品表为什么分SPU和SKU？**
   - SPU存公共信息（品牌/描述/图片），SKU存差异化信息（规格/价格/库存），避免数据冗余，查询更灵活。

2. **inventory_balance 为什么用乐观锁？**
   - 库存是高并发写场景（下单/发货/退货都改库存），乐观锁(version)比悲观锁性能更好，冲突时重试即可。

3. **订单表为什么按月分表？**
   - 订单数据量大且持续增长，按月分表可控制单表数据量，冷热分离（近3月热表+历史归档），查询性能稳定。

4. **策略模式和工厂模式怎么选？**
   - 策略模式：关注"算法可替换"（如物流路由策略）；工厂模式：关注"创建对象"（如平台适配器创建）。两者常配合使用。

5. **多租户数据如何保证隔离？**
   - 每张表加tenant_id字段+索引，查询时自动加条件(MyBatis-Plus租户插件)。关键：所有查询必须带tenant_id，否则数据泄露。

---

## 九、推荐资源

| 资源 | 说明 |
|------|------|
| ShardingSphere 文档 | 分库分表 |
| MyBatis-Plus 多租户插件 | 租户隔离 |
| 《高性能MySQL》 | 数据库设计优化 |
| 《实现领域驱动设计》 | 领域建模 |
| 《Head First 设计模式》 | 设计模式入门 |
