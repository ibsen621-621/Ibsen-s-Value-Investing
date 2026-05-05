---
version: 5.4-Gem
role: Base Rates 基础率数据库（11 个数据集）—— 行业/公司类型/事件类型的历史概率
load_priority: 5
---

# 05 BASE RATES — VIAIOS-P V5.4 Historical Probability Database

> Cross-ref: [[01_CORE_PROTOCOL.md#工具2外部视角]], [[06_VALUATION_MODELS.md]]

---

## 目录 (TOC)

1. [使用规则与反幻觉声明](#1-使用规则与反幻觉声明)
2. [roic_persistence — ROIC 持续性](#2-roic_persistence--roic-持续性)
3. [margin_reversion — 利润率回归均值](#3-margin_reversion--利润率回归均值)
4. [growth_persistence — 营收增长持续性](#4-growth_persistence--营收增长持续性)
5. [ipo_long_term_returns — IPO 长期回报](#5-ipo_long_term_returns--ipo-长期回报)
6. [m_and_a_value_creation — 并购价值创造](#6-m_and_a_value_creation--并购价值创造)
7. [turnaround_success — 困境反转成功率](#7-turnaround_success--困境反转成功率)
8. [bond_implied_default_rates — 债券隐含违约率](#8-bond_implied_default_rates--债券隐含违约率)
9. [implied_erp_history — 隐含 ERP 历史序列](#9-implied_erp_history--隐含-erp-历史序列)
10. [sales_to_capital_by_industry — 行业销售资本比](#10-sales_to_capital_by_industry--行业销售资本比)
11. [cyclical_midcycle_margins — 周期行业中周期利润率](#11-cyclical_midcycle_margins--周期行业中周期利润率)
12. [sovereign_cds_snapshot — 主权 CDS 快照](#12-sovereign_cds_snapshot--主权-cds-快照)

---

## 1. 使用规则与反幻觉声明

⚠️ **重要**：本文件中的数值为统计均值/示例占位数据。实际部署时：
- `implied_erp_history`：每月从达莫达兰网站 (pages.stern.nyu.edu/~adamodar) 更新
- `sovereign_cds_snapshot`：每季度（危机期间每月）从 Bloomberg/Markit 更新
- `bond_implied_default_rates`：每年从 Moody's/S&P 年度违约报告更新
- 其他基础率数据集：年度更新

**禁止**：当基础率数据与用户描述的情景不符时，禁止编造数字。必须明确标注数据来源和适用条件。

**调用时机**（Agent-9 强制调用）：
- 工具2 外部视角：`growth_persistence`、`margin_reversion`、`roic_persistence`、`m_and_a_value_creation`、`ipo_long_term_returns`、`turnaround_success`
- 工具4 双轨分离：`bond_implied_default_rates`（VR-03 触发）
- 工具5 动态 ERP：`implied_erp_history`、`sovereign_cds_snapshot`
- 工具1 SOTP：`sales_to_capital_by_industry`
- 工具7 周期常态化：`cyclical_midcycle_margins`（VR-04 强制）

---

## 2. roic_persistence — ROIC 持续性

**用途**：护城河持久性校准；Agent-1 ROIC 分析

```yaml
library: roic_persistence
statistical_window: "1995-2024"
sample_size: 9320 上市公司
applicable_markets: [US, CN_A, HK]

metrics:
  持续高 ROIC 概率:
    roic_gt_15pct_for_5y:
      probability: 0.24    # 仅24%的公司能维持ROIC>15%达5年
      ci_95: [0.22, 0.26]
    roic_gt_20pct_for_5y:
      probability: 0.14    # 仅14%能维持ROIC>20%达5年
      ci_95: [0.12, 0.16]
    roic_gt_20pct_for_10y:
      probability: 0.06    # 仅6%能维持ROIC>20%达10年（护城河真正的稀缺性）
      ci_95: [0.05, 0.07]
  fade_to_industry_median_years:
    median: 6.0  # 大多数高ROIC公司在6年内回归行业中位数
    ci_95: [5.4, 6.8]
```

**操作含义**：如果管理层预测 ROIC > 20% 持续 10 年，历史基率仅 6%，必须有极强的护城河证据支撑，否则下调假设。

---

## 3. margin_reversion — 利润率回归均值

**用途**：估值利润率假设校准；工具7 周期常态化

```yaml
library: margin_reversion
statistical_window: "1985-2024"
sample_size: 86521 公司年度观测

metrics:
  excess_margin_half_life_years:
    all_industries:
      median: 5.2  # 超额利润率在约5.2年内回归到行业均值的一半
      ci_95: [4.7, 5.8]
    asset_light_sectors:
      median: 6.4  # 轻资产行业衰减更慢
      ci_95: [5.7, 7.1]
    heavy_asset_sectors:
      median: 4.1  # 重资产行业衰减更快
      ci_95: [3.7, 4.6]
  reversion_speed_lambda:
    baseline: 0.13  # 每年超额利润率衰减约13%
    ci_95: [0.10, 0.16]
```

**操作含义**：DCF 中的利润率假设必须加入均值回归假设（除非有具体护城河证据）。行业景气期估计会低估回归速度。

---

## 4. growth_persistence — 营收增长持续性

**用途**：外部视角；管理层高增长预测的隐含概率锚定

```yaml
library: growth_persistence
statistical_window: "1990-2024"
sample_size: 12436 全球上市公司

metrics:
  cagr_persistence:
    gt_10pct_for_5y:
      probability: 0.27    # 仅27%的公司能维持>10% CAGR达5年
    gt_10pct_for_10y:
      probability: 0.18    # 仅18%能维持10年
    gt_15pct_for_5y:
      probability: 0.15
    gt_15pct_for_10y:
      probability: 0.09
    gt_20pct_for_5y:
      probability: 0.08
    gt_20pct_for_10y:
      probability: 0.04    # 仅4%能维持>20% CAGR达10年
```

**操作含义**：当管理层预测"未来10年20%年增长"时，历史基率仅4%。无特别强护城河证据的情况下，假设此概率是高度乐观的。

---

## 5. ipo_long_term_returns — IPO 长期回报

**用途**：新股估值；IPO 期间市场定价理性校验

```yaml
library: ipo_long_term_returns
statistical_window: "1980-2024"
sample_size: 14612 IPO

metrics:
  underperform_benchmark_frequency:
    at_3_years: {probability: 0.62}   # 62%的IPO在3年内跑输基准
    at_5_years: {probability: 0.66}
    at_10_years: {probability: 0.71}  # 71%的IPO在10年内跑输基准
  median_relative_return:
    at_5_years: -0.18    # 中位数5年相对回报为-18%
    at_10_years: -0.31
```

**操作含义**：任何 IPO 定价分析应明确标注此基率。高发行热度年份的长期超额回报通常更差。IPO 中的解禁与再融资因素必须纳入。

---

## 6. m_and_a_value_creation — 并购价值创造

**用途**：含并购计划的公司估值；管理层资本配置质量评估

```yaml
library: m_and_a_value_creation
statistical_window: "1990-2024"
sample_size: 10244 个交易

metrics:
  success_probability:
    positive_excess_return_1y: {probability: 0.46}    # 仅46%在1年后产生正超额回报
    positive_economic_profit_3y: {probability: 0.38}  # 仅38%在3年后创造经济利润
    positive_economic_profit_5y: {probability: 0.34}
  failure_modes:
    overpayment_rate: {probability: 0.41}             # 41%的并购属于溢价过高
    integration_failure_rate: {probability: 0.29}     # 29%整合失败
```

**操作含义**：当公司宣布大型并购时，默认从负面影响开始，要求管理层提供具体整合计划和协同效应来源才能升级。

---

## 7. turnaround_success — 困境反转成功率

**用途**：困境投资分析；VR-03 双轨分离补充参考

```yaml
library: turnaround_success
statistical_window: "1990-2024"
sample_size: 3890 困境案例

metrics:
  successful_turnaround_probability:
    fcf_positive_within_3y: {probability: 0.33}        # 仅33%在3年内实现正FCF
    leverage_normalized_within_5y: {probability: 0.28} # 仅28%在5年内正常化杠杆
    equity_outperformance_5y: {probability: 0.24}      # 仅24%在5年内股票超额表现
  permanent_capital_impairment_probability:
    probability: 0.41    # 41%的困境案例导致永久资本减损
```

**操作含义**：财务重组成功不等于股东回报成功。困境反转案例对融资环境高度敏感。

---

## 8. bond_implied_default_rates — 债券隐含违约率

**用途**：§12.3 双轨分离（VR-03 触发时强制调用）；P_distress 参数反推

### 投资级违约率（5年累计，示例均值）

| 评级 | 典型利差(bps) | 5年累计违约率(%) | 回收率(%) |
|------|-------------|----------------|---------|
| AAA | 30 | 0.01 | 65.0 |
| AA | 55 | 0.04 | 62.0 |
| A | 90 | 0.14 | 58.0 |
| BBB | 180 | 0.72 | 53.0 |

### 高收益/投机级违约率（5年累计，示例均值）

| 评级 | 典型利差(bps) | 5年累计违约率(%) | 回收率(%) |
|------|-------------|----------------|---------|
| BB | 320 | 6.40 | 46.0 |
| B | 600 | 16.80 | 38.0 |
| CCC | 1200 | 35.70 | 28.0 |
| CC | 2500 | 55.00 | 20.0 |
| C/D | 5000 | 80.00 | 10.0 |

### 行业利差调整（叠加于评级基准，单位 bps）

| 行业 | 调整 |
|------|------|
| Technology Software | -20 |
| Energy Oil & Gas | +40 |
| Real Estate Developer | +80 |
| Financials (Bank) | -15 |
| Consumer Staples | -15 |
| Utilities | -25 |
| Mining & Metals | +50 |

### P_distress 反推公式

```
P_distress_annual ≈ Bond_Spread / (1 - Recovery_Rate)
P_distress_cumulative_5y ≈ 1 - (1 - P_distress_annual)^5

示例：公司债利差 850bps，回收率 40%
  P_distress_annual = 850bps / (1 - 0.40) = 14.2% / 年
  P_distress_5y = 1 - (1 - 0.142)^5 ≈ 51%
  合理性校验：对比 B 级别 5年 16.8%（当前高于均值，说明市场评级更悲观）
```

**V5.3 硬规则 VR-03**：净负债/EBITDA > 4 或利息保障倍数 < 3 时，必须执行双轨分离，禁止仅靠调高 WACC 处理破产风险。P_distress 必须从市场数据反推，**禁止主观估算**。

---

## 9. implied_erp_history — 隐含 ERP 历史序列

**用途**：§14.3 动态风险校准引擎（Agent-9 工具5）；任何 WACC 计算

### 月度序列（示例，部署时须替换为 damodaran.com 最新数据）

| 日期 | S&P 500 | 10Y 国债(%) | 隐含 ERP(%) | 事件标注 |
|------|---------|------------|------------|---------|
| 2024-01 | 4742 | 3.97 | 4.62 | — |
| 2024-06 | 5460 | 4.36 | 4.28 | — |
| 2024-12 | 5881 | 4.58 | 4.12 | — |
| 2025-06 | 5570 | 4.50 | 4.38 | — |
| 2025-12 | 5780 | 4.62 | 4.31 | — |
| 2026-02 | 5650 | 4.48 | 4.37 | — |
| **2026-03** | **5290** | **4.35** | **4.77** | **★ 中东战争爆发 — ERP 单月跳升 40bp；M-013 触发** |
| 2026-04 | 5390 | 4.40 | 4.65 | 地缘局势略有缓和，但 ERP 仍高于战前 |

**历史统计摘要（背景参考）**：
- 1962-2023 算术均值：4.60%
- 中位数：4.40%
- 危机峰值：2008-10 = 10.60%；2020-03 = 7.80%；2022-09 = 5.90%

### 强制刷新触发条件（M-013）

```
以下任一情况发生后 14 天内，必须重新计算 WACC：
- 重大宏观事件（战争/流动性冲击）
- ERP 单月跳升 > 30bp
- S&P 500 单月下跌 > 5%
- 地缘军事冲突升级
```

**禁止**：在上述触发条件满足后继续使用历史均值 ERP（如静态 4.6% 假设）。

---

## 10. sales_to_capital_by_industry — 行业销售资本比

**用途**：§10.5 SOTP；Agent-9 工具1；无历史财报公司的再投资率锚定

**使用公式**：
```
Reinvestment_Rate = Revenue_Growth / Sales_to_Capital
FCFF = EBIT × (1-t) × (1 - Reinvestment_Rate)
```

### 主要行业参考值（示例，部署时须从 damodaran.com 更新）

| 行业 | 子行业 | Sales/Capital 低 | 中值 | 高 | 资本特征 |
|------|-------|----------------|------|-----|---------|
| Technology | Software (SaaS) | 1.50 | 2.20 | 3.50 | 极轻资产 |
| Technology | Semiconductors (IDM/Foundry) | 0.30 | 0.55 | 0.90 | 极度资本密集 |
| Technology | Internet Platform | 1.80 | 2.50 | 4.00 | 轻资产 |
| Technology | Semiconductors (Fabless) | 1.00 | 1.50 | 2.20 | 研发密集 |
| Aerospace | Launch Services | 0.40 | 0.65 | 1.00 | 资本密集 |
| Aerospace | Satellite Internet | 0.20 | 0.40 | 0.70 | 建设期极低 |
| Energy | Integrated Oil & Gas | 0.40 | 0.65 | 0.95 | 资本密集 |
| Advanced Mfg | EV Battery | 0.35 | 0.60 | 0.90 | 扩产期偏低 |
| Advanced Mfg | Solar PV | 0.45 | 0.75 | 1.10 | 中等 |
| Healthcare | Pharma (Big Pharma) | 0.70 | 1.10 | 1.60 | 研发密集 |
| Healthcare | Biotech (Early Stage) | 0.10 | 0.25 | 0.50 | 早期无营收 |
| Consumer | Luxury Goods | 1.20 | 1.80 | 2.80 | 轻资产（品牌） |
| Consumer | Gaming/Casinos | 0.30 | 0.55 | 0.85 | 物业密集 |
| Telecom | Mobile Network | 0.35 | 0.55 | 0.85 | 网络密集 |
| Financials | Commercial Banking | 0.08 | 0.15 | 0.25 | 金融杠杆，不适合本比率 |

**强制注意**：对无历史财务数据的独角兽/早期公司，禁止凭空猜测 CapEx，必须从本表行业中值出发，结合公司特有因素做 ±20% 调整。

---

## 11. cyclical_midcycle_margins — 周期行业中周期利润率

**用途**：VR-04 硬规则（CYCLICAL 模板强制调用）；禁止使用 TTM 利润率做 DCF

**VR-04 强制执行指引**：
```
[VR-04 MANDATORY]
CYCLICAL 模板：禁止使用 TTM 利润率直接做 DCF 终值假设
必须使用下表对应行业的 7-10 年滚动中周期均值
必须同时披露：
  1. 使用的周期窗口年份区间（如 2014-2024 年 10 年滚动均值）
  2. 中点取法（算术均值 vs 调和均值）
  3. 当前 TTM 利润率 vs 正常化利润率的差异（单位 pp）
  4. 若差异 > 10pp → 标注"景气顶部风险"或"景气低谷修复预期"
```

### 主要周期行业参考值（示例）

| 行业 | 周期窗口 | EBIT 利润率 低/中/高(%) | 顶峰预警(%) | 中周期商品价格假设 |
|------|---------|----------------------|-----------|-----------------|
| Integrated Oil & Gas | 10年 | 8.0 / **14.0** / 20.0 | **28.0** | 布伦特 65$/桶 |
| Exploration & Production | 10年 | 5.0 / **18.0** / 35.0 | 45.0 | 布伦特 65$/桶 |
| Diversified Metals & Mining | 10年 | 12.0 / **20.0** / 30.0 | 38.0 | 铜/铁矿石综合 |
| Steel (Integrated) | 8年 | 3.0 / **7.0** / 12.0 | 18.0 | 钢价 β ≈ 0.25-0.35 |
| Coal Mining | 10年 | 8.0 / **16.0** / 28.0 | 35.0 | 结构性衰退折扣 2-4pp |
| Container Shipping | 10年 | 2.0 / **8.0** / 15.0 | **35.0** | 疫情超级周期不可用 |
| Automobile Manufacturing | 8年 | 3.0 / **6.5** / 10.0 | 13.0 | 全球汽车销量回归 |
| EV Battery Manufacturing | 7年 | 8.0 / **14.0** / 20.0 | 25.0 | 行业尚无完整周期 |

**煤炭特殊注意**：能源转型导致永久性需求下移风险。历史中周期均值须额外加"结构性衰退折扣"（建议 2-4pp），并明确标注"历史中值 vs 新均衡假设"差异。

---

## 12. sovereign_cds_snapshot — 主权 CDS 快照

**用途**：跨国资产的国家风险溢价（CRP）实时更新（Agent-9 工具5；C-014 触发时必须调用）

### CRP 计算公式

```
方法1（Damodaran 方法，精确分析）：
  CRP = (Sovereign_CDS_Spread_bps / 10000) × (σ_equity / σ_bond)
  σ_equity / σ_bond 经验值：新兴市场 ≈ 1.5-2.0x；发达市场 ≈ 1.2-1.5x

方法2（简化方法，快速估算）：
  CRP ≈ Sovereign_CDS_Spread_bps / 10000

WACC 整合：
  WACC_adjusted = WACC_domestic + CRP_country
```

### 主要国家 CDS 快照（示例，2026-04-01）

| 国家 | 地区 | 5Y CDS(bps) | 评级 | CRP(%) | 战时标志 |
|------|------|------------|------|-------|---------|
| United States | 北美 | 35 | AA+ | 0.0 | — |
| Germany | 欧洲 | 18 | AAA | 0.0 | — |
| Japan | 亚太 | 28 | A+ | 0.1 | — |
| China (PRC) | 亚太 | 72 | A+ | **0.7** | — |
| South Korea | 亚太 | 38 | AA | 0.3 | — |
| India | 亚太 | 85 | BBB- | 1.0 | — |
| Vietnam | 亚太 | 85 | BB+ | 1.2 | — |
| UAE (Dubai) | 中东 | 55 | AA- | 0.5 | — |
| Saudi Arabia | 中东 | 68 | A | 0.8 | **战时↑** |
| Israel | 中东 | 145 | A+ | **2.0** | **战时↑↑** |
| Iraq | 中东 | 480 | B- | **6.5** | **战时↑↑↑** |
| Brazil | 拉美 | 175 | BB | 2.8 | — |
| Mexico | 拉美 | 130 | BBB | 1.8 | — |
| Turkey | 新兴欧洲 | 280 | B+ | 4.2 | — |
| Ukraine | 东欧 | 2800 | CC | 22.0 | **战时** |
| Russia | 东欧 | 9999 | SD | 20.0 | **制裁/不可交易** |

**禁止**：在黑天鹅事件（战争/危机）爆发后继续使用评级机构的静态国家评级来估算 CRP。评级机构下调通常滞后实际事件 3-12 个月；CDS 市场是实时的。

---

*End of 05_BASE_RATES.md — VIAIOS-P V5.4-Gem*
