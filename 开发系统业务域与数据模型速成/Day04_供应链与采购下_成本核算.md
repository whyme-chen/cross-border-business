# Day04 — 供应链与采购（下）+ 成本核算

---

## 一、成本构成全解析

跨境电商一件商品的完整成本远不止采购价，这是新手最容易忽略的：

```
┌───────────────────────────────────────────────────────┐
│               跨境电商商品成本构成                      │
│                                                       │
│  ┌─────────┐  ┌─────────┐  ┌──────┐  ┌───────────┐  │
│  │ 采购成本 │  │ 头程物流 │  │ 关税  │  │ 平台佣金  │  │
│  │  40-55%  │  │  5-15%  │  │ 2-8% │  │  8-15%   │  │
│  └─────────┘  └─────────┘  └──────┘  └───────────┘  │
│                                                       │
│  ┌─────────┐  ┌─────────┐  ┌──────┐  ┌───────────┐  │
│  │ FBA费用  │  │ 广告费用 │  │退货率 │  │  汇率损益  │  │
│  │  10-20%  │  │  5-15%  │  │2-10% │  │  0.5-3%  │  │
│  └─────────┘  └─────────┘  └──────┘  └───────────┘  │
└───────────────────────────────────────────────────────┘
注：百分比为占售价的大致比例，因品类和运营策略差异较大
```

### 各项成本详解

| 成本项 | 计算方式 | 备注 |
|--------|---------|------|
| 采购成本 | 单价 × 数量 + 运费 | 含税需计算可抵扣税额 |
| 头程物流 | 按重量/体积重 × 运费单价 | 体积重 = 长×宽×高/5000(空运)/6000(海运) |
| 关税 | 申报价值 × 关税率 | 按HS编码查税率，各国不同 |
| 增值税(VAT) | (申报价值 + 关税 + 运费) × VAT率 | 欧盟/英国进口时缴纳 |
| 平台佣金 | 售价 × 佣金率 | Amazon 8-15%，Shopee 2-8% |
| FBA配送费 | 按重量/尺寸段收费 | 分标准/大件/超大件 |
| FBA仓储费 | 按体积 × 天数 × 费率 | 长期仓储(>365天)费率飙升 |
| 广告费 | CPC × 点击次数 | ACOS = 广告花费/广告销售额 |
| 退货成本 | 退货率 × (配送费 + 退件处理费 + 商品折损) | 退货商品可能无法二次销售 |
| 汇率损益 | (结算汇率 - 采购时汇率) × 金额 | 汇率波动直接影响利润 |
| 包装费 | 包材 + 人工 | FBA有包装要求 |
| 其他 | 付款手续费、订阅费、仓储杂费 | — |

---

## 二、头程物流方式对比

| 方式 | 时效 | 成本(/kg) | 适合场景 | 起步量 |
|------|------|----------|---------|--------|
| 国际快递(DHL/UPS/FedEx) | 3-7天 | ¥60-120 | 紧急补货、样品 | 1kg+ |
| 空运专线 | 5-12天 | ¥25-50 | 中小件、时效要求较高 | 21kg+ |
| 海运快船 | 15-25天 | ¥5-12 | 大批量备货、大件商品 | 1CBM+ |
| 海运慢船 | 25-45天 | ¥3-8 | 不急的大批量备货 | 1CBM+ |
| 铁联(中欧班列) | 18-25天 | ¥8-15 | 欧洲方向适中时效 | 1CBM+ |

### 体积重计算

```
空运: 体积重(kg) = 长(cm) × 宽(cm) × 高(cm) / 5000
海运: 体积重(kg) = 长(cm) × 宽(cm) × 高(cm) / 6000

计费重量 = MAX(实际重量, 体积重)

示例:
  实重 8kg，尺寸 40×30×25cm
  体积重 = 40×30×25/5000 = 6kg
  计费重量 = MAX(8, 6) = 8kg
```

---

## 三、HS 编码与关税

### HS编码（Harmonized System）

```
HS编码结构（6位国际通用 + 各国扩展）:
  6203.42.31.00
  │    │  │  └── 本国子目(2位)
  │    │  └───── 本国子目(2位)  
  │    └──────── HS子目(2位)
  └───────────── HS章(4位)

示例:
  6203 - 西装、裤子等男装
  6203.42 - 棉制男式长裤
  6203.42.31 - 棉制男式牛仔裤
```

### 关税查询

| 工具 | 网址 | 说明 |
|------|------|------|
| 中国海关税则 | customs.gov.cn | 出口退税查询 |
| US HTS | hts.usitc.gov | 美国进口关税 |
| EU TARIC | ec.europa.eu/taxation_customs | 欧盟关税 |
| UK Trade Tariff | gov.uk/trade-tariff | 英国关税 |

### 关税计算示例

```
商品: 蓝牙耳机
HS编码: 8518.30 (美国)
关税率: 3.9%
申报价值: $10,000
关税 = $10,000 × 3.9% = $390
```

---

## 四、汇率与多币种

### 常见币种对

| 币种对 | 场景 |
|--------|------|
| CNY → USD | 采购(国内) → 销售(美国站) |
| CNY → EUR | 采购(国内) → 销售(欧洲站) |
| CNY → GBP | 采购(国内) → 销售(英国站) |
| USD → CNY | 回款(美元) → 结汇(人民币) |

### 汇率处理策略

| 策略 | 说明 | 适用场景 |
|------|------|---------|
| 实时汇率 | 每次交易取实时汇率 | 小额、低频 |
| 锁定汇率 | 采购时锁定，结算时使用 | 大额采购 |
| 周期均价 | 使用周/月平均汇率 | 财务报表 |
| 远期结汇 | 与银行签订远期合约 | 大额资金避险 |

### 汇兑损益

```
采购时: ¥100,000 @ 7.20 = $13,888.89
结算时: ¥100,000 @ 7.10 = $14,084.51

汇兑收益 = $14,084.51 - $13,888.89 = $195.62 (人民币贬值，美元回款更值钱)

反之亦然：
采购时: ¥100,000 @ 7.10 = $14,084.51
结算时: ¥100,000 @ 7.20 = $13,888.89
汇兑损失 = $195.62 (人民币升值，利润被吃掉)
```

---

## 五、BOM 与组合商品成本卷积

### BOM（Bill of Materials）

```
组合商品: "手机套装" (SKU-BUNDLE-001)
├── 手机壳 ×1    → 采购价 ¥8.00
├── 钢化膜 ×2    → 采购价 ¥1.50 × 2 = ¥3.00
├── 数据线 ×1    → 采购价 ¥3.00
└── 包装盒 ×1    → 采购价 ¥2.00

BOM总成本 = ¥8 + ¥3 + ¥3 + ¥2 = ¥16.00
```

### 成本卷积（Cost Roll-up）

将子项成本逐层向上汇总到成品的过程：

```
步骤1: 计算最底层原材料成本
步骤2: 加上加工/组装人工费
步骤3: 加上制造费用分摊
步骤4: 汇总得到成品成本

代码示例:
```

```java
// 成本卷积计算
public class CostRollUp {
    
    public BigDecimal calculateCost(BomNode node) {
        if (node.isLeaf()) {
            // 叶子节点：直接取采购价
            return node.getPurchasePrice()
                       .multiply(BigDecimal.valueOf(node.getQuantity()));
        }
        // 非叶子节点：递归计算子项成本 + 自身加工费
        BigDecimal childrenCost = node.getChildren().stream()
            .map(child -> calculateCost(child))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        return childrenCost.add(node.getAssemblyCost());
    }
}
```

---

## 六、利润测算模型

### 单SKU利润公式

```
毛利润 = 售价 - 采购成本 - 头程运费 - 关税 - 平台佣金 - FBA配送费 - FBA仓储费
净利润 = 毛利润 - 广告费 - 退货损失 - 汇率损益 - 包装费 - 其他费用
利润率 = 净利润 / 售价 × 100%
```

### 利润测算示例

```
商品: 蓝牙耳机
售价: $29.99 (Amazon US站)

成本拆解:
  采购成本:     ¥35.00 → $4.86  (@7.20)
  头程空运:     ¥8.00  → $1.11
  关税(3.9%):            $1.17
  Amazon佣金(15%):       $4.50
  FBA配送费:             $3.48
  FBA仓储费(月):         $0.30
  广告费(ACOS 20%):      $6.00
  退货率(5%):            $1.50  (退货成本分摊)
  包装费:                $0.42
  
总成本:                  $23.34
净利润: $29.99 - $23.34 = $6.65
利润率: $6.65 / $29.99 = 22.2%
```

### 利润测算代码示例

```java
@Data
@Builder
public class ProfitCalculator {
    
    private BigDecimal sellingPrice;    // 售价(USD)
    private BigDecimal purchasePrice;   // 采购价(CNY)
    private BigDecimal exchangeRate;    // 汇率 CNY→USD
    private BigDecimal firstLegCost;    // 头程(USD)
    private BigDecimal tariffRate;      // 关税率
    private BigDecimal commissionRate;  // 佣金率
    private BigDecimal fbaFee;          // FBA配送费
    private BigDecimal storageFee;      // 月仓储费
    private BigDecimal adAcos;          // 广告ACOS
    private BigDecimal returnRate;      // 退货率
    
    public ProfitResult calculate() {
        BigDecimal purchaseInUsd = purchasePrice.divide(exchangeRate, 2, RoundingMode.HALF_UP);
        BigDecimal tariff = sellingPrice.multiply(tariffRate);
        BigDecimal commission = sellingPrice.multiply(commissionRate);
        BigDecimal adCost = sellingPrice.multiply(adAcos);
        BigDecimal returnCost = sellingPrice.multiply(returnRate)
            .add(fbaFee.multiply(returnRate)); // 退货损失=退款+运费
        
        BigDecimal totalCost = purchaseInUsd
            .add(firstLegCost).add(tariff).add(commission)
            .add(fbaFee).add(storageFee).add(adCost).add(returnCost);
        
        BigDecimal netProfit = sellingPrice.subtract(totalCost);
        BigDecimal profitRate = netProfit.divide(sellingPrice, 4, RoundingMode.HALF_UP)
            .multiply(BigDecimal.valueOf(100));
        
        return ProfitResult.builder()
            .totalCost(totalCost)
            .netProfit(netProfit)
            .profitRate(profitRate)
            .build();
    }
}
```

---

## 七、核心术语表

| 术语 | 含义 |
|------|------|
| 头程 | First Leg，国内仓到海外仓的运输 |
| 体积重 | Volumetric Weight，按体积折算的重量 |
| 计费重 | Chargeable Weight，取实重和体积重的较大值 |
| CBM | Cubic Meter，立方米(海运计费单位) |
| HS编码 | 海关商品编码，确定关税率 |
| 关税 | Import Duty，进口时缴纳的税 |
| VAT | Value Added Tax，增值税 |
| ACOS | Advertising Cost of Sale，广告花费占销售比 |
| BOM | Bill of Materials，物料清单 |
| 成本卷积 | Cost Roll-up，子项成本逐层汇总 |
| 汇兑损益 | Exchange Gain/Loss，汇率变动产生的损益 |
| 远期结汇 | Forward Exchange，锁定未来汇率 |
| 申报价值 | Declared Value，报关时的货值 |

---

## 八、面试/实战高频问题

1. **为什么跨境电商经常"看着赚钱实际亏钱"？**
   - 隐性成本多：退货、汇损、长期仓储、广告、FBA杂费。建议做完整成本测算而非只看毛利。

2. **体积重和实重哪个用于计费？**
   - 取较大值。轻抛货（如毛绒玩具）体积重远大于实重，运费被体积重主导，设计包装时应优化体积。

3. **如何选择头程物流方式？**
   - 紧急补货→快递/空运；常规备货→海运快船；大件重货→海运。综合考虑时效×成本×库存周转。

4. **BOM成本卷积如何处理多层级？**
   - 递归遍历BOM树，从叶子节点（原材料）向上汇总，每层加上本层的加工/组装费用。

5. **汇率波动对利润影响有多大？**
   - 人民币每波动1%，利润率可能波动2-5%（因为采购用CNY、售价用USD，杠杆效应）。

---

## 九、推荐资源

| 资源 | 说明 |
|------|------|
| FBA Revenue Calculator | Amazon官方利润计算器 |
| hts.usitc.gov | 美国关税查询 |
| 中国人民银行汇率中间价 | 汇率参考 |
| B站"跨境电商成本核算" | 视频教程 |
| 《供应链成本管理》 | 成本核算理论 |
