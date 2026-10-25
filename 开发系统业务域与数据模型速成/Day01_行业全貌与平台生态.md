# Day01 — 行业全貌与平台生态

---

## 一、跨境电商是什么

跨境电商（Cross-border E-commerce）是指分属不同关境的交易主体，通过电子商务平台达成交易、进行支付结算，并通过跨境物流送达商品、完成交易的国际商业活动。

### 发展历程

| 阶段 | 时间 | 特征 |
|------|------|------|
| 萌芽期 | 1999-2003 | 阿里巴巴B2B起步，信息展示为主 |
| 成长期 | 2004-2012 | eBay/Amazon向中国卖家开放，速卖通上线 |
| 爆发期 | 2013-2019 | 平台百花齐放，独立站崛起（Shopify） |
| 成熟期 | 2020-至今 | TikTok Shop/Temu/Shein新模式，全托管/半托管 |

### 中国跨境电商规模

- 2024年出口跨境电商交易额超 2.6 万亿元人民币
- 年均增速 15%+，远超传统外贸增速
- 卖家群体超百万，深圳/广州/义乌为主要聚集地

---

## 二、跨境电商模式对比

### 按交易主体分

```
B2B (Business to Business)
  └── 批量出口，如阿里巴巴国际站
  └── 订单金额大，频次低，合规要求高

B2C (Business to Consumer)
  └── 零售出口，如Amazon/Shopee/速卖通
  └── 订单量多，单笔金额小，重运营效率

DTC (Direct to Consumer)
  └── 品牌直达消费者，如Shein/独立站
  └── 自建渠道，掌控用户数据，品牌溢价高
```

### 按方向分

| 维度 | 跨境出口 | 跨境进口 |
|------|---------|---------|
| 方向 | 中国 → 海外 | 海外 → 中国 |
| 典型平台 | Amazon/Shopee/Temu | 天猫国际/京东国际/考拉 |
| 税务 | 出口退税、目的国VAT | 行邮税/跨境电商综合税 |
| 物流 | 头程出境+尾程派送 | 保税仓/海外直邮 |
| **本文档侧重** | **✅ 跨境出口** | ❌ |

---

## 三、全链路概览

这是跨境电商从选品到回款的完整链路，后续所有专题都围绕此展开：

```
┌─────────────────────────────────────────────────────────────────┐
│                     跨境电商全链路                                │
│                                                                 │
│  选品 → 采购 → 头程物流 → 入仓 → 上架 → 营销 → 接单            │
│   │      │       │        │      │       │       │              │
│   ▼      ▼       ▼        ▼      ▼       ▼       ▼              │
│ 市场    供应    国际      仓储   Listing  广告   订单             │
│ 调研    商管理  运输      管理   管理    投放   处理             │
│                                                                 │
│  → 履约发货 → 尾程物流 → 妥投 → 结算回款 → 利润核算             │
│       │          │        │        │          │                  │
│       ▼          ▼        ▼        ▼          ▼                  │
│     物流      物流追踪   签收   平台结算    财务                 │
│     执行      客服      售后    资金归集    分析                 │
└─────────────────────────────────────────────────────────────────┘
```

**各环节对应系统：**

| 环节 | 核心系统 | Day 专题 |
|------|---------|---------|
| 选品/商品 | PIM（商品信息管理） | Day02 |
| 采购/供应链 | SRM/采购管理 | Day03-04 |
| 仓储/库存 | WMS | Day05 |
| 物流/履约 | TMS/OMS | Day06-07 |
| 订单/客服 | OMS/CRM | Day08 |
| 财务/结算 | FMS | Day09-10 |
| 合规/风控 | BIS | Day11 |

---

## 四、主流平台对比

### Amazon

| 维度 | 说明 |
|------|------|
| 总部 | 美国 |
| 主要市场 | 北美、欧洲、日本、澳洲 |
| 模式 | B2C为主，第三方卖家+自营 |
| 月租 | 专业卖家 $39.99/月 |
| 佣金 | 8%-15%（按类目） |
| 物流 | FBA（平台仓发货，主流） / FBM（自发货） |
| 特点 | 重产品轻店铺，A9算法驱动，品牌化趋势明显 |
| 卖家门槛 | 较高，需企业资质+品牌备案 |
| API | SP-API (Selling Partner API) |

### Shopee

| 维度 | 说明 |
|------|------|
| 总部 | 新加坡（腾讯投资） |
| 主要市场 | 东南亚（印尼/菲律宾/越南/泰国/马来/新加坡）、巴西 |
| 模式 | B2C+C2C |
| 佣金 | 2%-8%（新卖家免佣期） |
| 物流 | SLS（Shopee Logistics Service）+ 自选物流 |
| 特点 | 移动端为主，社交属性强，免邮补贴 |
| 卖家门槛 | 较低，个人/企业均可 |
| API | Shopee Open API |

### Lazada

| 维度 | 说明 |
|------|------|
| 总部 | 新加坡（阿里控股） |
| 主要市场 | 东南亚六国 |
| 佣金 | 1%-5% |
| 物流 | LGS（Lazada Global Shipping） |
| 特点 | 阿里生态，与淘宝/天猫体系打通 |
| API | Lazada Open API |

### TikTok Shop

| 维度 | 说明 |
|------|------|
| 总部 | 中国（字节跳动） |
| 主要市场 | 东南亚、英国、美国 |
| 模式 | 内容电商（直播+短视频带货） |
| 佣金 | 1%-5%（新商家免佣期） |
| 特点 | 兴趣电商，流量爆发力强，运营门槛不同 |
| API | TikTok Shop API |

### Temu

| 维度 | 说明 |
|------|------|
| 总部 | 中国（拼多多旗下） |
| 主要市场 | 北美、欧洲 |
| 模式 | 全托管（卖家供货，平台定价销售） |
| 特点 | 极致低价，卖家无定价权，供货商角色 |
| 门槛 | 供货商模式，适合工厂型卖家 |

### Shein

| 维度 | 说明 |
|------|------|
| 总部 | 中国 |
| 主要市场 | 全球（欧美为主） |
| 模式 | DTC品牌+平台化（第三方卖家入驻） |
| 特点 | 小单快反，柔性供应链，快时尚标杆 |

### 速卖通（AliExpress）

| 维度 | 说明 |
|------|------|
| 总部 | 中国（阿里旗下） |
| 主要市场 | 俄罗斯、欧洲、拉美 |
| 佣金 | 5%-8% |
| 物流 | 菜鸟物流 |
| 特点 | 阿里体系，俄罗斯市场强势 |

---

## 五、平台开放 API 能力对比

| 能力 | Amazon SP-API | Shopee Open API | TikTok Shop API | Lazada Open API |
|------|--------------|-----------------|-----------------|-----------------|
| 商品/Listing | ✅ Catalog/CatalogItems | ✅ Product | ✅ Product | ✅ Product |
| 订单 | ✅ Orders | ✅ Order | ✅ Order | ✅ Order |
| 物流/发货 | ✅ Fulfillment | ✅ Logistics | ✅ Fulfillment | ✅ Fulfillment |
| 财务/结算 | ✅ Finances/Reports | ✅ Payment | ✅ Finance | ✅ Finance |
| 广告 | ✅ Advertising API | ✅ Ads | ❌ | ❌ |
| 消息/客服 | ✅ Messaging | ✅ Chat | ✅ Conversation | ✅ IM |
| 报告 | ✅ Reports | ✅ | ✅ | ✅ |
| Webhook | ✅ Notifications | ✅ Push | ✅ Webhook | ✅ Callback |

---

## 六、核心术语速查表

### 通用术语

| 术语 | 全称 | 含义 |
|------|------|------|
| SKU | Stock Keeping Unit | 最小库存单位，一个可售卖的具体商品规格 |
| SPU | Standard Product Unit | 标准产品单元，一类商品的抽象（如"iPhone 15"） |
| ASIN | Amazon Standard ID Number | Amazon 平台唯一商品标识 |
| Listing | — | 商品在平台上的展示页面 |
| Variation | — | 变体，同一SPU下不同SKU的组合（如不同颜色/尺码） |
| FBA | Fulfillment by Amazon | 亚马逊平台仓发货 |
| FBM | Fulfillment by Merchant | 卖家自发货 |
| MOQ | Minimum Order Quantity | 最小起订量 |
| GMV | Gross Merchandise Volume | 成交总额（含未付款） |
| ROI | Return on Investment | 投资回报率 |
| ROAS | Return on Ad Spend | 广告支出回报率 |
| ACOS | Advertising Cost of Sale | 广告花费占销售额比（Amazon核心指标） |
| BSR | Best Sellers Rank | 畅销排名 |
| Buy Box | — | 购物车（Amazon黄金购物车，决定谁获得订单） |
| Listing变体 | — | 同一商品的不同规格组合展示 |
| 跟卖 | — | 在他人Listing下挂自己的SKU售卖（Amazon特色） |
| UPC/EAN | Universal Product Code / European Article Number | 商品条码 |
| FNSKU | Fulfillment Network SKU | FBA商品专用标识 |
| HTS/HS | Harmonized Tariff Schedule | 海关编码 |
| OEM | Original Equipment Manufacturer | 代工生产 |
| ODM | Original Design Manufacturer | 贴牌生产 |
| 白牌 | — | 无品牌商品 |
| 独立站 | — | 自建电商网站（Shopify/自研） |
| DTC | Direct to Consumer | 品牌直达消费者 |
| 头程 | — | 国内仓库 → 海外仓库/平台仓 |
| 尾程 | Last Mile | 海外仓库 → 消费者 |
| 妥投 | Delivered | 成功投递 |
| VAT | Value Added Tax | 增值税（欧盟/英国） |
| GST | Goods and Services Tax | 商品及服务税（澳洲/印度） |
| 清关 | Customs Clearance | 货物进出境时向海关申报 |
| 退换标 | — | 退回商品重新贴标后二次销售 |
| 库容 | — | 仓库可用的存储容量（FBA有库容限制） |
| 长期仓储费 | Long-term Storage Fee | 超龄库存的额外仓储费用 |
| 移除费 | Removal Fee | 从平台仓移出商品的费用 |
| 赔偿 | Reimbursement | 平台对丢失/损坏库存的赔偿 |

---

## 七、面试/实战高频问题

1. **跨境电商和国内电商最大的区别是什么？**
   - 答：涉及跨境物流（时效长/成本高）、多币种结算、关税与清关、多平台多站点运营、不同国家合规要求。

2. **FBA和FBM分别适合什么场景？**
   - 答：FBA适合销量稳定、周转快的商品（时效好、有Buy Box权重加成）；FBM适合测款期、大件商品、低频商品。

3. **为什么跨境电商需要ERP系统？**
   - 答：多平台多店铺运营时，手动操作效率低、易出错。ERP统一管理商品/订单/库存/物流/财务，实现自动化和规模化。

4. **Amazon的Buy Box机制是什么？**
   - 答：一个Listing可能有多个卖家售卖，Buy Box决定哪个卖家获得"添加到购物车"按钮。影响因素：价格、发货方式（FBA优先）、卖家评级、库存等。

5. **跨境出口的核心痛点有哪些？**
   - 答：库存管理（海外仓备货风险）、物流时效与成本、多币种汇率波动、合规（VAT/知识产权）、平台政策变化、退货处理成本高。

---

## 八、推荐资源

| 类型 | 资源 | 说明 |
|------|------|------|
| 书籍 | 《跨境电商运营实战》 | 全貌入门 |
| 网站 | 雨果网 (cifnews.com) | 行业资讯 |
| 网站 | 亿邦动力 (ebrun.com) | 行业报告 |
| 文档 | Amazon SP-API 官方文档 | developer.amazonservices.com |
| 文档 | Shopee Open API | open.shopee.com |
| 公众号 | 雨果网 | 每日行业动态 |
| 公众号 | 跨境电商头条 | 政策与趋势 |
| 视频 | B站"跨境电商全链路讲解" | 可视化理解 |
| 报告 | 艾瑞咨询跨境电商报告 | 市场数据 |
