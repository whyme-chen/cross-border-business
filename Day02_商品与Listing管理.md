# Day02 — 商品与 Listing 管理

---

## 一、商品模型：SPU / SKU / 变体

### 核心概念

```
SPU (Standard Product Unit) — 标准产品单元
  └── 抽象的商品类型，如 "iPhone 15"
  └── 包含公共属性：品牌、型号、描述、图片

SKU (Stock Keeping Unit) — 最小库存单位
  └── 具体的可售卖规格，如 "iPhone 15 黑色 128G"
  └── 每个SKU有独立库存、独立价格

变体 (Variation) — 同一SPU下的不同SKU组合
  └── 如：颜色 × 存储 = 6个SKU
```

### 示例

```
SPU: iPhone 15
├── SKU-001: iPhone 15 黑色 128G   → ¥5999  库存: 100
├── SKU-002: iPhone 15 黑色 256G   → ¥6999  库存: 80
├── SKU-003: iPhone 15 黑色 512G   → ¥8999  库存: 30
├── SKU-004: iPhone 15 蓝色 128G   → ¥5999  库存: 120
├── SKU-005: iPhone 15 蓝色 256G   → ¥6999  库存: 60
└── SKU-006: iPhone 15 蓝色 512G   → ¥8999  库存: 20

变体维度: 颜色(黑/蓝) × 存储(128/256/512) = 6个SKU
```

### 多平台映射

同一商品在不同平台上的表示不同：

```
本地商品库                    平台侧
┌──────────┐              ┌───────────────────┐
│ SPU-1001 │──── Amazon ──→│ ASIN: B0CXXXXXX   │
│ iPhone15 │              │ 变体: 6个Child ASIN│
└──────────┘              └───────────────────┘
     │                    ┌───────────────────┐
     ├──── Shopee ────→  │ Item ID: 12345678 │
     │                    │ Model: 6个型号     │
     │                    └───────────────────┘
     │                    ┌───────────────────┐
     └──── TikTok ────→  │ Product ID: 9...  │
                          │ SKU: 6个          │
                          └───────────────────┘
```

**映射策略：**
- 本地 SPU ↔ 平台 Listing（1:N，一个SPU可能对应多平台多个Listing）
- 本地 SKU ↔ 平台 SKU/Child ASIN/Model（1:1 或 1:N）
- 维护映射表记录对应关系

---

## 二、Listing 结构详解（以 Amazon 为例）

### Listing 组成

```
┌────────────────────────────────────────┐
│              Listing 页面               │
├────────────────────────────────────────┤
│  标题 (Title)                          │
│  └── 核心关键词 + 品牌名 + 核心卖点     │
├────────────────────────────────────────┤
│  图片 (Images)                         │
│  └── 主图(白底) + 副图(场景/细节/尺寸)  │
├────────────────────────────────────────┤
│  变体选择 (Variations)                  │
│  └── 颜色/尺码等选择器                  │
├────────────────────────────────────────┤
│  价格 (Price)                          │
│  └── 各SKU独立定价                      │
├────────────────────────────────────────┤
│  五点描述 (Bullet Points)               │
│  └── 5个核心卖点，每点一行              │
├────────────────────────────────────────┤
│  A+页面 (A+ Content)                   │
│  └── 富文本/图文排版，品牌卖家专属       │
├────────────────────────────────────────┤
│  产品描述 (Product Description)         │
│  └── 详细文字描述（有A+时此部分被替代）  │
├────────────────────────────────────────┤
│  技术规格 (Technical Specifications)    │
│  └── 属性键值对列表                     │
└────────────────────────────────────────┘
```

### 多语言多币种

```
同一商品在不同站点的差异化：

US站: Title(英文) / Price($29.99) / 合规属性(FCC)
UK站: Title(英文) / Price(£24.99) / 合规属性(CE)
DE站: Title(德文) / Price(€27.99) / 合规属性(CE+EU)
JP站: Title(日文) / Price(¥3999) / 合规属性(PSE)
```

---

## 三、分类与属性

### 类目树

```
Root
├── 电子产品 (Electronics)
│   ├── 手机 (Cell Phones)
│   ├── 电脑 (Computers)
│   └── 耳机 (Headphones)
├── 家居 (Home & Kitchen)
│   ├── 厨具 (Kitchenware)
│   └── 家纺 (Bedding)
└── 服装 (Clothing)
    ├── 男装 (Men's)
    └── 女装 (Women's)
```

- 各平台有独立类目体系，需做类目映射
- 不同类目有不同属性模板（必填/选填属性不同）

### 属性模板

| 属性类型 | 示例 | 说明 |
|---------|------|------|
| 基础属性 | 品牌、型号、颜色 | 通用属性 |
| 类目属性 | 电池容量(手机)、面料(服装) | 类目特有 |
| 合规属性 | CE认证号、FDA注册号 | 法规要求 |
| 物流属性 | 重量、尺寸、是否带电 | 影响运费 |
| 销售属性 | 颜色/尺码（变体维度） | 决定变体结构 |

---

## 四、商品同步策略

### 同步方向

```
┌────────────┐   Push(上架/更新)    ┌────────────┐
│            │ ──────────────────→  │            │
│  本地商品库 │                      │  平台Listing│
│            │ ←────────────────── │            │
└────────────┘   Pull(拉取订单/状态) └────────────┘
```

### 同步冲突处理

| 场景 | 冲突 | 处理策略 |
|------|------|---------|
| 本地改价 + 平台改价 | 价格不一致 | 最后修改时间优先 / 人工审核 |
| 本地库存扣减 + 平台卖出 | 超卖 | 库存预留 + 预占机制 |
| 本地下架 + 平台在售 | 状态不一致 | 定时全量对账 |
| 平台删除Listing | 本地有映射无对应 | 标记映射失效，告警 |

### 同步频率

| 数据类型 | 频率 | 方式 |
|---------|------|------|
| 商品信息 | 变更时推送 / 每日全量 | Push + Pull |
| 库存数量 | 实时（秒级） | 事件驱动Push |
| 价格 | 变更时推送 | Push |
| Listing状态 | 定时拉取 | Pull（5-15分钟） |

---

## 五、数据模型设计

### 商品主表 (product_spu)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| spu_code | VARCHAR(64) | SPU编码，唯一 |
| name | VARCHAR(256) | 商品名称 |
| brand_id | BIGINT | 品牌ID |
| category_id | BIGINT | 本地类目ID |
| status | TINYINT | 状态：0草稿/1上架/2下架 |
| description | TEXT | 商品描述 |
| created_at | DATETIME | 创建时间 |
| updated_at | DATETIME | 更新时间 |

### SKU表 (product_sku)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| spu_id | BIGINT | 关联SPU |
| sku_code | VARCHAR(64) | SKU编码，唯一 |
| barcode | VARCHAR(64) | 条码(UPC/EAN) |
| weight | DECIMAL(10,3) | 重量(kg) |
| length/width/height | DECIMAL(10,3) | 尺寸(cm) |
| purchase_price | DECIMAL(12,2) | 采购价 |
| cost_price | DECIMAL(12,2) | 成本价(含分摊) |
| status | TINYINT | 状态 |
| attrs_json | JSON | 变体属性 {"color":"黑","storage":"128G"} |

### 多语言表 (product_spu_i18n)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| spu_id | BIGINT | 关联SPU |
| locale | VARCHAR(10) | 语言代码(zh/en/de/ja) |
| name | VARCHAR(256) | 本地化名称 |
| title | VARCHAR(512) | Listing标题 |
| bullet_points | JSON | 五点描述 |
| description | TEXT | 详细描述 |

### 平台映射表 (product_platform_mapping)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| spu_id | BIGINT | 关联SPU |
| sku_id | BIGINT | 关联SKU |
| platform | VARCHAR(32) | 平台标识(amazon/shopee) |
| shop_id | BIGINT | 店铺ID |
| platform_listing_id | VARCHAR(64) | 平台Listing ID |
| platform_sku_id | VARCHAR(64) | 平台SKU ID(如ASIN) |
| sync_status | TINYINT | 同步状态 |
| last_sync_at | DATETIME | 最后同步时间 |

### 属性表 (product_attribute)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| name | VARCHAR(64) | 属性名 |
| attr_type | TINYINT | 0基础/1类目/2合规/3物流/4销售 |
| input_type | TINYINT | 0文本/1数字/2下拉/3多选 |
| required | TINYINT | 是否必填 |
| options_json | JSON | 下拉选项值 |

---

## 六、面试/实战高频问题

1. **SPU和SKU的区别？为什么需要两层？**
   - SPU是商品抽象，SKU是具体可售规格。两层设计好处：公共信息在SPU层维护避免重复，SKU只存储差异化信息（规格/价格/库存），查询和统计更灵活。

2. **多平台商品映射怎么设计？**
   - 维护平台映射表，本地SKU → 平台SKU是一对多关系。同步时根据映射关系推送/拉取，同时处理各平台属性差异。

3. **如何防止超卖？**
   - 下单时先锁库存（预占），发货后扣减。多渠道共享库存需预留安全库存，或按渠道分配库存配额。

4. **商品信息变更多平台同步如何保证一致性？**
   - 采用最终一致性：本地变更→消息队列→各平台适配器异步推送。失败时重试+对账补偿，不做强一致性事务。

5. **一个新平台对接，商品模块需要做什么？**
   - 实现该平台的适配器（商品CRUD API）、类目映射配置、属性模板映射、Listing结构转换、同步策略配置。

---

## 七、推荐资源

| 资源 | 说明 |
|------|------|
| Amazon Seller Central → Listing 帮助 | Listing结构官方说明 |
| SP-API Catalog Items API | 商品接口文档 |
| Shopee Product API | Shopee商品接口 |
| Odoo Product 模块源码 | 参考数据模型设计 |
| ERPNext Item 模块 | 另一个ERP数据模型参考 |
