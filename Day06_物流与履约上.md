# Day06 — 物流与履约（上）

---

## 一、物流模式详解

### FBA（Fulfillment by Amazon）

```
卖家 → 将商品发至Amazon仓库 → Amazon负责存储/拣货/打包/配送/客服

流程:
  创建发货计划 → 备货贴标 → 头程发往FBA → Amazon接收上架 → 客户下单 → Amazon发货

优点: Prime标识、Buy Box加权、客服外包、退换货处理
缺点: 仓储费高、库容限制、无法多平台发货、贴标要求严格
```

### FBM（Fulfillment by Merchant）

```
卖家自存货 → 客户下单 → 卖家自行打包发货

流程:
  客户下单 → 订单拉取 → 仓库拣货打包 → 选择物流渠道 → 获取面单 → 发货 → 上传追踪号

优点: 灵活、费用可控、可多平台发货
缺点: 无Prime标识、需自建客服、时效依赖物流渠道
```

### 海外仓发货

```
卖家 → 头程批量发货至海外仓 → 客户下单 → 海外仓当地发货

流程:
  备货 → 头程发至海外仓 → 入库上架 → 订单推送海外仓 → 仓库拣货 → 尾程派送

优点: 时效接近FBA、可多平台发货、退货可当地处理
缺点: 需要备货资金、三方仓服务费、库存管理复杂
```

### 直邮（Drop Shipping from China）

```
客户下单 → 国内直接发货 → 国际物流 → 清关 → 尾程派送

优点: 无需海外仓、低库存风险
缺点: 时效长(7-30天)、物流追踪不稳定、退换货困难
```

| 对比 | FBA | FBM(海外仓) | FBM(直邮) |
|------|-----|-------------|----------|
| 时效 | 1-2天 | 2-5天 | 7-30天 |
| 成本 | 高 | 中 | 低 |
| 灵活性 | 低 | 高 | 最高 |
| Prime | ✅ | ❌ | ❌ |
| 多平台 | ❌ | ✅ | ✅ |
| 退货 | Amazon处理 | 海外仓处理 | 困难 |

---

## 二、头程 vs 尾程

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   头程 (First Leg / Inbound)                            │
│   国内仓库 ═════════════════════► 海外仓库/平台仓         │
│   海运/空运/铁联/快递                 入库上架             │
│                                                         │
│   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │
│                                                         │
│   尾程 (Last Mile / Outbound)                           │
│   海外仓库 ═════════════════════► 消费者                 │
│   当地快递(USPS/Royal Mail/DHL)    签收                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 头程关注点

| 维度 | 关注点 |
|------|--------|
| 时效 | 各渠道标准时效，影响补货周期 |
| 成本 | 运费+关税+清关费，影响商品成本 |
| 合规 | 出口报关、目的国清关、认证要求 |
| 包装 | FBA贴标要求、托盘规格、外箱标签 |

### 尾程关注点

| 维度 | 关注点 |
|------|--------|
| 时效 | 妥投时效，影响客户满意度 |
| 追踪 | 全程轨迹可查，减少客诉 |
| 妥投率 | 成功投递比例，影响退款率 |
| 退件 | 退货处理流程和成本 |

---

## 三、常用物流渠道对比

### 国际快递

| 渠道 | 时效 | 价格(/kg) | 优势 | 劣势 |
|------|------|----------|------|------|
| DHL | 3-5天 | ¥80-150 | 全球覆盖、清关强 | 贵 |
| UPS | 3-7天 | ¥70-130 | 美国强势 | 偏远附加费 |
| FedEx | 3-7天 | ¥70-130 | 东南亚快 | 欧洲偏弱 |

### 专线物流

| 渠道 | 时效 | 价格(/kg) | 优势 |
|------|------|----------|------|
| 燕文物流 | 7-15天 | ¥30-60 | Amazon官方合作 |
| 云途物流 | 7-12天 | ¥35-65 | 覆盖广、清关好 |
| 递四方 | 7-15天 | ¥30-55 | 阿里系、东南亚强 |

### 邮政小包

| 渠道 | 时效 | 价格(/kg) | 优势 | 劣势 |
|------|------|----------|------|------|
| ePacket | 15-30天 | ¥15-30 | 便宜 | 慢、追踪不全 |
| 中国邮政挂号 | 20-40天 | ¥10-25 | 最便宜 | 非常慢 |

### 海运

| 类型 | 时效 | 价格(/CBM) | 优势 |
|------|------|-----------|------|
| 海运快船(美森) | 12-18天 | ¥800-1500 | 海运中最快 |
| 海运普船 | 25-40天 | ¥400-800 | 最便宜 |

---

## 四、物流服务商对接

### 典型API能力

```
┌──────────────────────────────────────┐
│          物流服务商 API               │
├──────────────────────────────────────┤
│  1. 运费查询 (Rate Query)            │
│     输入: 始发地/目的国/重量/尺寸     │
│     输出: 渠道列表+运费+时效          │
│                                      │
│  2. 创建物流单 (Create Shipment)      │
│     输入: 订单信息/收件人/商品信息     │
│     输出: 物流单号+面单URL            │
│                                      │
│  3. 获取面单 (Get Label)             │
│     输入: 物流单号                    │
│     输出: 面单PDF/URL                │
│                                      │
│  4. 轨迹查询 (Track)                 │
│     输入: 追踪号                      │
│     输出: 节点列表(时间+状态+地点)     │
│                                      │
│  5. 费用账单 (Billing)               │
│     输入: 时间范围                    │
│     输出: 费用明细+差异              │
│                                      │
│  6. 取消物流单 (Cancel Shipment)      │
│     输入: 物流单号                    │
│     输出: 取消结果                    │
└──────────────────────────────────────┘
```

### 对接架构

```java
// 适配器模式对接多物流商
public interface LogisticsAdapter {
    RateResult queryRate(RateRequest request);
    ShipmentResult createShipment(ShipmentRequest request);
    LabelResult getLabel(String trackingNo);
    TrackResult track(String trackingNo);
}

// 云途实现
@Component("yuntu")
public class YuntuAdapter implements LogisticsAdapter { ... }

// 递四方实现
@Component("4px")
public class FourPxAdapter implements LogisticsAdapter { ... }

// 工厂获取
public class LogisticsAdapterFactory {
    public LogisticsAdapter getAdapter(String provider) {
        return context.getBean(provider + "Adapter", LogisticsAdapter.class);
    }
}
```

---

## 五、运费模板设计

### 运费计算矩阵

```
运费 = 基础运费 + 重量附加费 + 偏远附加费 + 燃油附加费 + 特殊处理费

基础运费: 按目的国+渠道的起步价
重量附加费: (计费重 - 起步重量) × 续重单价
偏远附加费: 目的地区为偏远区时加收
燃油附加费: 按月调整的比例
特殊处理费: 带电/超长/超重/液体等
```

### 运费模板示例

```
模板: 美国标准专线
  ┌────────────────────────────────────────┐
  │ 起步价(0-50g):  ¥25.00               │
  │ 续重(每50g):    ¥3.50                 │
  │ 处理费:         ¥5.00/单              │
  │ 偏远附加:       ¥15.00(阿拉斯加/夏威夷)│
  │ 燃油附加:       基础运费 × 12%        │
  │ 带电附加:       ¥8.00/单              │
  │ 时效:           7-12工作日             │
  │ 体积重除数:     5000                   │
  └────────────────────────────────────────┘
```

### 运费计算代码示例

```java
public class ShippingCalculator {
    
    public BigDecimal calculate(ShippingCalcRequest req) {
        // 1. 计算计费重量
        BigDecimal volumetricWeight = req.getLength()
            .multiply(req.getWidth())
            .multiply(req.getHeight())
            .divide(BigDecimal.valueOf(req.getDivisor()), 2, RoundingMode.HALF_UP);
        BigDecimal chargeableWeight = req.getActualWeight().max(volumetricWeight);
        
        // 2. 基础运费
        ShippingTemplate template = getTemplate(req.getChannel(), req.getCountry());
        BigDecimal baseFee = template.getBaseFee(); // 起步价
        
        // 3. 续重费
        BigDecimal extraWeight = chargeableWeight.subtract(template.getBaseWeight())
            .max(BigDecimal.ZERO);
        BigDecimal extraFee = extraWeight.divide(template.getWeightUnit(), 0, RoundingMode.UP)
            .multiply(template.getExtraUnitPrice());
        
        // 4. 附加费
        BigDecimal surcharge = BigDecimal.ZERO;
        if (req.isRemoteArea()) surcharge = surcharge.add(template.getRemoteFee());
        if (req.isBattery()) surcharge = surcharge.add(template.getBatteryFee());
        BigDecimal fuelSurcharge = baseFee.add(extraFee)
            .multiply(template.getFuelRate());
        
        // 5. 总运费
        return baseFee.add(extraFee).add(surcharge).add(fuelSurcharge)
            .add(template.getHandlingFee())
            .setScale(2, RoundingMode.HALF_UP);
    }
}
```

---

## 六、核心术语表

| 术语 | 含义 |
|------|------|
| 头程 | First Leg，国内到海外仓的运输 |
| 尾程 | Last Mile，海外仓到消费者的派送 |
| FBA | Fulfillment by Amazon，亚马逊平台仓发货 |
| FBM | Fulfillment by Merchant，卖家自发货 |
| 追踪号 | Tracking Number，物流轨迹追踪编号 |
| 面单 | Shipping Label，贴在包裹上的运单标签 |
| 计费重 | Chargeable Weight，取实重和体积重的较大值 |
| 妥投 | Delivered，成功投递到买家 |
| ASN | Advanced Shipping Notice，预到货通知 |
| FBA发货计划 | Amazon发货计划，指定发往哪个FBA仓 |
| 贴标 | FBA要求每个商品贴FNSKU条码 |
| 合仓 | FBA将库存集中在指定仓库(收费) |
| 分仓 | FBA自动分配到多个仓库(免费) |
| 托盘 | Pallet，海运中整托发货的单位 |

---

## 七、面试/实战高频问题

1. **FBA和海外仓怎么选？**
   - 单平台(Amazon)主力SKU用FBA，多平台发货/退货处理/中转备货用海外仓。通常两者搭配使用。

2. **头程物流怎么选？**
   - 紧急补货用空运/快递，常规补货用海运快船，大件重货必走海运。综合考量时效需求×库存水平×成本预算。

3. **运费计算为什么用体积重？**
   - 轻抛货（如毛绒玩具）实际重量轻但占用空间大，物流商按空间收费，所以用体积重。包装设计应尽量紧凑减少体积重。

4. **物流服务商对接的难点是什么？**
   - 各家API标准不统一、接口文档质量参差、错误码不一致、网络不稳定需重试机制、面单格式各异。需要适配器模式+统一抽象层。

5. **如何保证运费计算准确？**
   - 建立运费模板配置化、定期与物流商账单对账、设置差异阈值告警、体积重参数可配置。

---

## 八、推荐资源

| 资源 | 说明 |
|------|------|
| 云途物流 Open API | 典型跨境物流商接口文档 |
| 递四方 API 文档 | 另一家主流物流商 |
| Amazon FBA 文档 | FBA发货流程和费用说明 |
| 17Track API | 轨迹追踪聚合平台 |
| AfterShip API | 另一个轨迹追踪服务 |
