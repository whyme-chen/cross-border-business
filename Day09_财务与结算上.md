# Day09 — 财务与结算（上）

---

## 一、收入确认规则

### 收入确认时点

```
跨境电商收入确认的关键问题：什么时候算"赚到这笔钱"？

┌──────────────────────────────────────────────────────┐
│  方案1: 发货时确认（权责发生制）                       │
│  └── 商品发出即确认收入，风险已转移                    │
│  └── 优点: 及时反映经营状况                            │
│  └── 缺点: 退货需冲回                                 │
│                                                       │
│  方案2: 妥投时确认                                     │
│  └── 买家签收后确认收入                               │
│  └── 优点: 更保守、退货影响小                         │
│  └── 缺点: 妥投确认有延迟                             │
│                                                       │
│  方案3: 结算时确认（收付实现制）                       │
│  └── 平台打款到账后确认                               │
│  └── 优点: 确定性强                                   │
│  └── 缺点: 滞后，不利于经营决策                       │
└──────────────────────────────────────────────────────┘

多数跨境电商ERP采用: 发货确认收入 + 退货冲回
```

---

## 二、平台费用详解

### Amazon 费用结构

| 费用类型 | 计算方式 | 典型比例 |
|---------|---------|---------|
| 销售佣金(Referral Fee) | 售价 × 佣金率 | 8%-15% |
| FBA配送费 | 按重量/尺寸段 | $2.50-$5.0+/件 |
| FBA月仓储费 | 体积 × 天数 × 费率 | $0.87/立方英尺/月(标准) |
| FBA长期仓储费 | 超龄库存额外收费 | $6.90/立方英尺/半年 |
| 订阅费 | 固定月费 | $39.99/月(专业卖家) |
| 广告费 | CPC × 点击量 | 按实际 |
| 退货处理费 | 按件收费 | $1.00-$5.00/件 |
| 移除/弃置费 | 按件收费 | $0.50-$0.80/件 |
| 高量listing费 | 超过活跃ASIN阈值 | $0.005/ASIN/月 |

### FBA配送费分段示例（美国站）

```
小标准尺寸(最长边≤15"):
  0-4 oz:  $2.50
  4-8 oz:  $2.80
  8-12 oz: $3.10
  12-16 oz: $3.38

大标准尺寸(最长边≤18"):
  0-8 oz:  $3.10
  8-16 oz: $3.38
  1-2 lb:  $4.36
  2-3 lb:  $4.90

小大件(最长边≤60"):
  0-2 lb:  $8.05
  2-4 lb:  $8.92
  每增1lb: +$0.42
```

### Shopee 费用结构

| 费用 | 说明 | 比例 |
|------|------|------|
| 交易佣金 | 按订单金额 | 2%-8% |
| 交易手续费 | 支付通道费 | 1%-2% |
| 免运费活动费 | 参与免邮补贴 | 2%-5% |
| Shopee广告 | CPC竞价 | 按实际 |

---

## 三、结算周期与结算报告

### 各平台结算周期

| 平台 | 结算周期 | 最低结算额 | 币种 |
|------|---------|-----------|------|
| Amazon US | 每14天 | 无 | USD |
| Amazon EU | 每14天 | 无 | EUR/GBP |
| Shopee | 每周/双周 | 各站点不同 | 当地币种 |
| TikTok Shop | 每周 | 各站点不同 | 当地币种 |
| Lazada | 每周 | 各站点不同 | 当地币种 |

### Amazon Settlement Report 解析

```
结算报告(Date Range Report)关键字段:

settlement-id:          结算批次号
settlement-start-date:  结算起始日
settlement-end-date:    结算截止日
deposit-date:           打款日期
total-amount:           本次结算总金额

交易明细(Transaction)类型:
  Order:          订单收入
  Refund:         退款
  Commission:     佣金扣除
  FBA Fee:        FBA费用
  Shipping:       运费收入/扣除
  Advertising:    广告费
  Subscription:   月租费
  Storage Fee:    仓储费
  Adjustment:     调整项
  Reimbursement:  赔偿
```

### 结算数据拉取

```java
// 伪代码：拉取Amazon结算报告
public class AmazonSettlementService {
    
    public void syncSettlement(String shopId, LocalDate start, LocalDate end) {
        // 1. 调用SP-API获取结算报告
        SettlementReport report = spApiClient.getSettlementReport(
            shopId, start, end);
        
        // 2. 解析交易明细
        for (Transaction txn : report.getTransactions()) {
            // 3. 按类型分别处理
            switch (txn.getType()) {
                case "Order":
                    // 匹配本地订单，确认收入
                    matchOrderAndConfirmRevenue(txn);
                    break;
                case "Refund":
                    // 匹配退款单，冲回收入
                    matchRefundAndReverse(txn);
                    break;
                case "Commission":
                case "FBA Fee":
                case "Advertising":
                    // 记录费用，分摊到SKU
                    recordFeeAndAllocate(txn);
                    break;
            }
        }
        
        // 4. 生成对账摘要
        generateReconciliationSummary(report);
    }
}
```

---

## 四、多币种处理

### 核心概念

```
┌──────────────────────────────────────────────┐
│              多币种处理框架                    │
│                                              │
│  本位币 (Base Currency)                      │
│  └── 公司记账使用的基准币种，通常为CNY或USD   │
│                                              │
│  交易币 (Transaction Currency)               │
│  └── 每笔交易实际发生的币种                   │
│                                              │
│  汇率来源:                                    │
│  ├── 银行中间价（权威，但有延迟）             │
│  ├── 平台结算汇率（Amazon自行换算）           │
│  └── 合约汇率（远期结汇锁定）                 │
│                                              │
│  汇兑差额:                                    │
│  └── 交易时汇率 ≠ 结算时汇率 产生的差异       │
│  └── 计入"汇兑损益"科目                      │
└──────────────────────────────────────────────┘
```

### 汇率处理策略

| 场景 | 策略 | 说明 |
|------|------|------|
| 采购下单 | 锁定采购日汇率 | 采购成本确定 |
| 平台销售 | 使用平台结算汇率 | 以平台实际结算为准 |
| 资金归集 | 使用结汇日汇率 | 银行实际汇率 |
| 财务报表 | 月末汇率/月均汇率 | 折算为本位币 |

### 数据模型

```sql
-- 汇率表
CREATE TABLE exchange_rate (
    id BIGINT PRIMARY KEY,
    from_currency VARCHAR(3),     -- 源币种
    to_currency VARCHAR(3),       -- 目标币种
    rate DECIMAL(12,6),           -- 汇率
    rate_date DATE,               -- 汇率日期
    source VARCHAR(32),           -- 来源(PBOC/PLATFORM/CUSTOM)
    created_at DATETIME
);
```

---

## 五、费用分摊算法

### 为什么要分摊到SKU

```
平台结算报告给出的是整笔订单的费用，利润核算需要精确到每个SKU:

订单 #123:
  SKU-A ×2  $15.00 × 2 = $30.00
  SKU-B ×1  $20.00 × 1 = $20.00
  订单总额 = $50.00
  
  FBA配送费总计 = $8.00
  佣金(15%) = $7.50
  
  如何将$8.00分配给SKU-A和SKU-B？
```

### 分摊方法

**方法1: 按金额比例分摊**

```
SKU-A占比 = $30 / $50 = 60%
SKU-B占比 = $20 / $50 = 40%

SKU-A分摊FBA费 = $8.00 × 60% = $4.80
SKU-B分摊FBA费 = $8.00 × 40% = $3.20
```

**方法2: 按数量均摊**

```
总数量 = 2 + 1 = 3

SKU-A分摊FBA费 = $8.00 × 2/3 = $5.33
SKU-B分摊FBA费 = $8.00 × 1/3 = $2.67
```

**方法3: 按重量分摊（FBA配送费推荐）**

```
SKU-A重量 = 0.2kg × 2 = 0.4kg
SKU-B重量 = 0.3kg × 1 = 0.3kg
总重量 = 0.7kg

SKU-A分摊FBA费 = $8.00 × 0.4/0.7 = $4.57
SKU-B分摊FBA费 = $8.00 × 0.3/0.7 = $3.43
```

### 分摊规则建议

| 费用类型 | 推荐分摊方式 | 原因 |
|---------|------------|------|
| FBA配送费 | 按重量 | 配送费本身按重量计费 |
| 佣金 | 按金额 | 佣金按售价百分比 |
| 广告费 | 按金额 | 与销售额关联 |
| 仓储费 | 按体积 | 仓储费按体积计费 |
| 退货处理费 | 按件 | 直接归属该SKU |

---

## 六、结算数据模型

### 结算批次表 (finance_settlement)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| settlement_no | VARCHAR(32) | 结算批次号 |
| platform | VARCHAR(32) | 平台 |
| shop_id | BIGINT | 店铺 |
| start_date | DATE | 结算起始日 |
| end_date | DATE | 结算截止日 |
| deposit_date | DATE | 打款日期 |
| currency | VARCHAR(3) | 币种 |
| total_amount | DECIMAL(12,2) | 结算总额 |
| total_fee | DECIMAL(12,2) | 费用总额 |
| net_amount | DECIMAL(12,2) | 净打款额 |
| status | VARCHAR(16) | 状态(PENDING/CONFIRMED/RECONCILED) |

### 费用明细表 (finance_fee_detail)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| settlement_id | BIGINT | 关联结算批次 |
| order_id | BIGINT | 关联订单 |
| order_item_id | BIGINT | 关联订单明细(SKU级) |
| sku_id | BIGINT | SKU ID |
| fee_type | VARCHAR(32) | 费用类型(COMMISSION/FBA/AD/SHIPPING) |
| amount | DECIMAL(12,2) | 金额 |
| currency | VARCHAR(3) | 币种 |
| allocation_method | VARCHAR(16) | 分摊方式 |

---

## 七、面试/实战高频问题

1. **平台结算报告的数据什么时候拉取？**
   - 打款后拉取，确保数据完整。Amazon每14天结算一次，通常在deposit_date后1-2天拉取。

2. **为什么佣金分摊按金额而不是按数量？**
   - 佣金按售价百分比收取，售价高的SKU承担更多佣金，按金额比例分摊最合理。

3. **汇兑损益如何记账？**
   - 交易时按交易汇率记应收/应付，结算时按实际汇率入账，差额计入"财务费用-汇兑损益"。

4. **一笔订单费用如何精确到SKU级别？**
   - 根据费用类型选择合适的分摊规则，先整单记录，再按规则拆分到SKU明细。分摊结果写入fee_detail表。

5. **FBA费用为什么难精确预估？**
   - FBA配送费按重量和尺寸段分级，仓储费按体积按时长计算，还有长期仓储费、退货处理费等。需要维护准确的商品重量/尺寸数据。

---

## 八、推荐资源

| 资源 | 说明 |
|------|------|
| Amazon Seller Central → Payments | 结算报告官方说明 |
| SP-API Finances API | 财务接口文档 |
| FBA Revenue Calculator | Amazon利润计算器 |
| 《跨境电商财务实战》 | 财务知识专项 |
| 领星/积加 ERP 财务模块 | 产品视角理解 |
