---
version: 5.4-Gem
role: 估值模型全集（DCF、Reverse DCF、Agent-9 七大工具、Damodaran 方法论）
load_priority: 6
---

# 06 VALUATION MODELS — VIAIOS-P V5.4 Complete Valuation Toolkit

> Cross-ref: [[02_AGENTS_ROSTER.md#agent-9]], [[05_BASE_RATES.md]], [[07_RISK_FRAMEWORK.md#vr-01vr-05]]

---

## 目录 (TOC)

1. [估值层架构概览](#1-估值层架构概览)
2. [Reverse DCF — Agent-6 核心工具](#2-reverse-dcf--agent-6-核心工具)
3. [Agent-9 Damodaran Engine](#3-agent-9-damodaran-engine)
4. [工具1：SOTP 叙事分解与分类加总](#4-工具1sotp-叙事分解与分类加总)
5. [工具2：Monte Carlo 蒙特卡洛估值分布](#5-工具2monte-carlo-蒙特卡洛估值分布)
6. [工具3：Expansion Option 扩张期权](#6-工具3expansion-option-扩张期权)
7. [工具4：Distress-Adjusted Dual-Track 困境双轨](#7-工具4distress-adjusted-dual-track-困境双轨)
8. [工具5：Dynamic ERP & CRP 动态风险校准](#8-工具5dynamic-erp--crp-动态风险校准)
9. [工具6：Restated Financials 财务报表外科手术](#9-工具6restated-financials-财务报表外科手术)
10. [工具7：Cyclical Mid-Cycle Normalization 周期常态化](#10-工具7cyclical-mid-cycle-normalization-周期常态化)
11. [Pricing Gymnastics Detector 定价体操检测](#11-pricing-gymnastics-detector-定价体操检测)
12. [相对估值（Relative Valuation）辅助工具](#12-相对估值relative-valuation辅助工具)
13. [V5.4 Patch 4：Regime Shift Re-DCF](#13-v54-patch-4regime-shift-re-dcf)

---

## 1. 估值层架构概览

```
Layer 4（与 Agent-1/2/5 并行）

  Agent-6 (Valuation Expert)          Agent-9 (Damodaran Engine)
  ─────────────────────────            ─────────────────────────
  • Reverse DCF（从价格反推）           • SOTP（多业务线分解）
  • 隐含增长率计算                       • Monte Carlo（分布替代三点情景）
  • Difficulty Score 量化              • Expansion Option（期权独立计量）
  • alpha_source 标注（Patch 1）        • Distress Dual-Track（VR-03）
  • edge_score 检查（Patch 6）          • Dynamic ERP/CRP（WACC 动态校准）
                                        • Restated Financials（ROIC 重述）
                                        • Cyclical Normalization（VR-04）
                                        • Pricing Gymnastics Detection（VR-05）
                                        • Capex Regime Check（V5.4 Patch 4）
```

**执行前提**：Agent-9 必须等待 `agent_0_forensic_pass = true` 方可执行。

---

## 2. Reverse DCF — Agent-6 核心工具

### 核心逻辑

从当前市场价格**反向推导**市场隐含的增长和利润率假设，然后评估这些假设实现的难度。

### 执行步骤

```
Step 1: 收集输入
  - 当前股价 (P_current)
  - WACC（从 Agent-9 工具5 获取动态值）
  - 当前 NOPAT 或 FCFF
  - 净负债/现金（资产负债表）
  - 稀释股数

Step 2: 反解隐含增长率
  假设 P_current = DCF(Revenue_growth, Terminal_margin, Sales/Capital, WACC)
  迭代求解使 DCF = P_current 的参数组合
  
  常用近似：
  Implied_FCFF_Growth = WACC - (FCFF_current / Enterprise_Value)

Step 3: 难度评分（Difficulty Score，1-10）
  1-3：隐含增长假设保守，低于历史基率 → BUY 候选
  4-6：中性假设，隐含增长接近历史基率中位数 → 中性
  7-9：乐观假设，需要高于历史基率的增长 → 警惕
  10：极度乐观，历史上 <5% 的公司达到 → SELL/AVOID

  锚定 [[05_BASE_RATES.md#growth_persistence]]：
    隐含增长 > 20%/年持续 10 年 → Difficulty = 9-10（历史基率仅 4%）

Step 4: 输出
  - implied_revenue_growth: <float>
  - implied_terminal_margin: <float>
  - difficulty_score: <int, 1-10>
  - historical_base_rate_for_implied_growth: <float>
  - verdict: "市场已定价了 [X]% 年增长持续 [Y] 年，历史上只有 [Z]% 的公司实现了这一目标"
```

---

## 3. Agent-9 Damodaran Engine

### 7 大工具调用决策树

```
接收输入后，按以下顺序路由：

1. Capex Regime Check (V5.4 Patch 4)
   → capex_ratio > 1.5 连续 ≥2 季度?
   → YES: regime_shift_flag = true; 禁用历史 FCF; 使用 Regime-Shift Re-DCF

2. SOTP 触发检查
   → 业务线 > 2 AND 单一业务 < 60%?
   → YES: 工具1 SOTP（VR-02 强制）

3. Monte Carlo（全部分析必须执行）
   → 始终激活工具2

4. 扩张期权检查
   → 有远期低概率高赔率叙事?
   → YES: 工具3

5. 困境/破产风险检查
   → 净负债/EBITDA > 4 OR ICR < 3?
   → YES: 工具4 + VR-03 强制

6. 宏观动态校准（全部分析必须执行）
   → 始终激活工具5

7. 财务报表重述检查
   → R&D/Revenue > 8% OR 获客成本/Revenue > 15%?
   → YES: 工具6

8. 周期股常态化检查
   → Industry_Template == CYCLICAL?
   → YES: 工具7 + VR-04 强制

特殊：MODE_HARD_TO_VALUE
   → 工具1+2+3+4+5 全部强制激活（无条件）
```

---

## 4. 工具1：SOTP 叙事分解与分类加总

**触发条件**：业务线 > 2，且单一业务占比 < 60%  
**硬规则**：[VR-02 MANDATORY] 满足条件必须执行，禁止单一整体 DCF

### 执行步骤

```
STEP 1：叙事拆解
  - 识别公司核心独立业务叙事（每条叙事对应一个独立市场和竞争逻辑）
  - 示例（SpaceX 类）：发射服务 + 卫星宽带 + AI/算力 + 扩张期权
  - 输出：叙事清单 + 每条叙事的市场定位描述

STEP 2：各业务线独立终局假设（第 10 年目标）
  每条业务线必须独立设定：
  - 第10年营收（Revenue_Y10）
  - 第10年营业利润率（EBIT_Margin_Y10）
  - 销售资本比（Sales-to-Capital）← 锚定 [[05_BASE_RATES.md#sales_to_capital_by_industry]]
  - 终局市占率（Terminal Market Share）
  - 终值增长率（Terminal Growth Rate）

STEP 3：叙事-数字一致性检验（3 项强制检验）
  □ 第10年营收是否与 TAM × 终局市占率一致?
  □ 利润率假设是否与行业基准和竞争格局一致?（参照 cyclical_midcycle_margins.yml）
  □ Sales/Capital 是否与行业轻重资产特征一致?（参照 sales_to_capital_by_industry.yml）
  □ 终局市占率假设是否已说明"谁将失去这些市场份额"?
  输出：narrative_numbers_consistency_table（通过/未通过/待说明）

STEP 4：各业务线分别构建 DCF
  - 各业务线独立折现（可使用差异化 WACC/折现率）
  - 扩张期权部分单独列示（工具3处理）
  
  FCFF 计算（正常情景）：
    Reinvestment_Rate = Revenue_Growth / Sales_to_Capital
    FCFF_t = EBIT_t × (1 - tax_rate) × (1 - Reinvestment_Rate)
  
STEP 5：SOTP 汇总
  Total_Value = Σ(Business_Line_Values) + Expansion_Option_Value + Cash - Net_Debt
  输出：SOTP_breakdown_table（含各业务线贡献占比%）
```

---

## 5. 工具2：Monte Carlo 蒙特卡洛估值分布

**覆盖范围**：全部分析（取代 Bear/Base/Bull 三点情景，不可跳过）  
**模拟规模**：≥ 10,000 次

### 核心变量概率分布设定

```
典型变量选择（4-5 个核心变量）：
  1. 营收复合增长率（Revenue CAGR）→ 三角分布（左偏）
  2. 终局营业利润率（Terminal EBIT Margin）→ 截断正态分布
  3. TAM 渗透率（Terminal Market Share）→ 对数正态分布
  4. 销售资本比（Sales-to-Capital）→ 正态分布
  5. 终值倍数/永续增长率→ 正态分布

分布参数确定：
  - 三角分布：设定 最低/最可能/最高 三点
  - 截断正态：设定 均值/标准差/上下截断点
  参数须从 [[05_BASE_RATES.md]] 历史数据和行业基准锚定，不得凭空设置
```

### 强制输出指标

```
P10（10th percentile）：极悲观情景价值
P25（25th percentile）：保守情景价值
P50（50th percentile / 中位数）：基准情景价值（核心参考）
P75（75th percentile）：乐观情景价值
P90（90th percentile）：极乐观情景价值
Current_Price_Percentile：当前股价在分布中的分位数
```

### VR-01 硬规则

```
[VR-01 MANDATORY]
当前股价 > P75 → 一票否决
  vr01_triggered: true
  vr01_message: "当前股价[X]超过估值分布P75[Y]，处于乐观情景上方，
                 风险收益不对称，BUY/STRONG BUY 评级被否决"
```

---

## 6. 工具3：Expansion Option 扩张期权

**适用场景**：远期低概率高赔率叙事（太空采矿、火星旅行、颠覆性新市场进入）  
**关键原则**：扩张期权**必须单独列示**，不得揉进主营业务 DCF 的折现率

### 执行步骤

```
STEP 1：识别扩张期权
  - 明确列示所有远期颠覆性叙事（第6年之后）
  - 每个期权独立设定：
    · 触发年限（Option_Trigger_Year）
    · 爆发概率：0-30%（高于 30% 应视为主营业务，不应作为期权处理）
    · 条件营收（若实现，第N年营收规模）
    · 成熟后利润率假设
    · Sales/Capital（参照 [[05_BASE_RATES.md#sales_to_capital_by_industry]]）

STEP 2：期权价值计算
  Option_Value = P(realization) × DCF(conditional_cash_flows)
  折现率 = 主营 WACC + 5-10pp（反映期权的不确定性）

STEP 3：贡献度检验（强制）
  Option_Value_Contribution = Option_Value / (Core_DCF_Value + Option_Value)
  [MANDATORY] 期权价值贡献占总估值建议上限 15%
  若超过 15%，须特别说明理由并在报告中显著标注
```

---

## 7. 工具4：Distress-Adjusted Dual-Track 困境双轨

**触发条件**：净负债/EBITDA > 4 OR 利息保障倍数 < 3  
**硬规则**：[VR-03 MANDATORY] 禁止仅靠调高 WACC 处理破产风险

### 双轨期望值模型

```
公式（刚性，禁止变更）：
  V_total = V_going_concern × (1 − P_distress) + V_liquidation × P_distress

P_distress 计算（必须从市场数据反推，禁止主观估算）：
  方法1：从公司垃圾债价格反推
    P_distress_annual ≈ Bond_Spread / (1 - Recovery_Rate)
  方法2：若公司债未上市，使用 5 年期 CDS 利差
    P_distress_annual ≈ CDS_5Y_bps / 10000 / (1 - Recovery_Rate)
  方法3：若两者均不可得，使用同评级/行业均值（降级处理，标注"行业违约率替代"）
  合理性校验：与 [[05_BASE_RATES.md#bond_implied_default_rates]] 对应评级区间比较

V_going_concern：正常 DCF（无困境调整，假设持续经营实体）
V_liquidation：
  - 流动资产（通常接近账面价值）
  - 固定资产（重置成本打折，通常 50-80%）
  - 无形资产（困境处置通常折扣 60-90%）
  - 减：优先债务清偿
```

**V5.4 Patch 4 修正**：当 `regime_shift_flag: true` 时，`V_going_concern` 必须使用 Regime-Shift Re-DCF，而非历史 FCF DCF。

---

## 8. 工具5：Dynamic ERP & CRP 动态风险校准

**覆盖范围**：全部分析必须执行（任何分析都须验证当期 ERP 是否需要更新）

### WACC 计算标准

```
WACC = Ke × (E/V) + Kd × (1-t) × (D/V)

Ke = Rf + β × ERP + CRP + 特定风险溢价

各组成部分：
  Rf：当期 10Y 国债
    - 美元资产：美国 10Y 国债（当前 ~4.40%，参照 implied_erp_history.yml）
    - 人民币/港元资产：中国 10Y 国债收益率（触发 C-013）
  
  ERP：当月隐含 ERP（非历史均值！）
    - 从 [[05_BASE_RATES.md#implied_erp_history]] 取最新值
    - 黑天鹅事件触发 M-013 后必须使用最新月份数据
    - 禁止使用历史算术均值 4.6% 作为替代（M-013 触发后）
  
  CRP：国家风险溢价（海外营收 > 30% 时触发 C-014）
    - 从 [[05_BASE_RATES.md#sovereign_cds_snapshot]] 取对应国家 CDS 数据
    - 危机期间禁止使用评级机构静态评级估算
  
  特定风险溢价：小公司溢价/流动性折扣（1-3%，需说明理由）
```

### 动态 ERP 更新触发条件（M-013）

以下任一情况发生后 14 天内必须重算 WACC：
- 重大宏观事件（战争/流动性冲击）
- ERP 单月跳升 > 30bp
- S&P 500 单月下跌 > 5%

---

## 9. 工具6：Restated Financials 财务报表外科手术

**触发条件**：R&D/Revenue > 8% OR 获客成本（S&M）/Revenue > 15%

### 执行步骤

```
STEP 1：研发费用资本化
  Capitalized_RD = R&D_Expense (视为资产)
  Amortization = Capitalized_RD / Research_Asset_Life (通常 3-5 年)
  EBIT_Restated = EBIT_GAAP + R&D_Expense - Amortization

STEP 2：获客成本资本化
  Customer_Acquisition_Asset = Σ(S&M spend over avg customer lifetime)
  Amortization_CAC = Customer_Acquisition_Asset / Avg_Customer_Life
  EBIT_Restated += S&M_Expense - Amortization_CAC

STEP 3：ROIC 重述
  Invested_Capital_Restated = Invested_Capital_GAAP
                              + Capitalized_RD_Net
                              + Customer_Acquisition_Asset_Net
  ROIC_Restated = NOPAT_Restated / Invested_Capital_Restated

STEP 4：输出对比
  旧 ROIC（GAAP）: [X]%
  重述 ROIC: [Y]%
  差异: [Y-X]pp（通常正向，即真实 ROIC 高于 GAAP ROIC）
```

---

## 10. 工具7：Cyclical Mid-Cycle Normalization 周期常态化

**触发条件**：Industry_Template == CYCLICAL  
**硬规则**：[VR-04 MANDATORY] 禁止使用 TTM 利润率做 DCF

### 执行步骤

```
方法 A（10 年滚动均值）：
  1. 确定周期窗口（通常 7-10 年，覆盖完整商品周期）
  2. 从 [[05_BASE_RATES.md#cyclical_midcycle_margins]] 获取行业中周期均值
  3. 计算当前 TTM 利润率 vs 中周期均值的差异（pp）
  4. 若差异 > 10pp → 强制使用中周期均值做 DCF 终值假设

方法 B（大宗商品价格回归）：
  建立回归方程：EBIT_Margin = α + β × Commodity_Price
  使用 IEA/行业长期均衡价格（非当期现货价格）
  计算正常化 EBIT_Margin

强制披露（必须出现在输出中）：
  □ 使用的周期窗口年份区间
  □ 中点取法（算术均值 vs 调和均值）
  □ TTM 利润率 vs 正常化利润率的差异（pp）
  □ 若 TTM > 正常化 + 10pp → "景气顶部风险警告"
```

---

## 11. Pricing Gymnastics Detector 定价体操检测

**触发条件**：提供了卖方研究报告（可选输入）  
**硬规则**：[VR-05 MANDATORY] 触发 → 研报不得作为唯一估值依据

### 检测的 4 类体操手法

| 类型 | 描述 | 标记代码 |
|------|------|---------|
| 远期倍数陷阱 | 用 5-10 年后的预期收益/营收计算"当前"倍数 | `PRICING_GYMNASTICS_FAR_FORWARD` |
| 调整后指标滥用 | "调整后 EBITDA" = 剔除了所有不利因素 | `PRICING_GYMNASTICS_ADJUSTED_METRICS` |
| 对标池操纵 | 选择性选取高倍数公司作为"同业" | `PRICING_GYMNASTICS_PEER_SELECTION` |
| 峰值利润 P/E | 用历史最高盈利年份的 P/E 看起来"便宜" | `PRICING_GYMNASTICS_PEAK_PE` |

---

## 12. 相对估值（Relative Valuation）辅助工具

作为 DCF 的交叉验证（B-001 最佳实践：至少使用 3 种方法）

### 主要倍数框架

```
EV/EBITDA：适用于资本密集型企业（剔除折旧差异）
EV/EBIT：适用于盈利稳定企业（反映营业利润）
P/FCF：最直接的现金生成能力衡量
P/E：简单但受会计政策影响
EV/Sales：适用于早期/亏损企业（危险，需谨慎）
EV/NOPAT：Damodaran 偏好，含资本成本概念

使用规则：
1. 同业比较必须使用"正常化"指标（非 TTM 峰值）
2. 需要说明为何被选中的同业集合代表性强
3. 溢价/折价必须有结构性解释（护城河强弱、增长差异、资本密集度）
4. 不得将相对估值作为唯一推荐依据
```

---

## 13. V5.4 Patch 4：Regime Shift Re-DCF

**触发条件**：`regime_shift_flag: true`（capex_ratio > 1.5 连续 ≥2 季度）

### Re-DCF 公式

```yaml
# 强制的 Re-DCF 路径（禁止使用历史 FCF 外推）

输入：
  forward_ebit: <float>           # 共识或内部建模的前向 EBIT
  tax_rate: <float>               # 有效税率
  new_reinvestment_rate: <float>  # 从 capex_ttm 计算：
                                  # = capex_ttm / (NOPAT + D&A)
                                  # 其中 NOPAT = forward_ebit × (1 - tax_rate)
  forward_roic: <float>           # 分析师/模型估计；必须标注为 ESTIMATE
  wacc: <float>                   # 从 Agent-9 工具5 动态计算

计算：
  forward_nopat = forward_ebit × (1 - tax_rate)
  forward_fcff = forward_nopat × (1 - new_reinvestment_rate)
  terminal_growth = min(wacc - 0.01, 0.03)  # 保守：不超过 3% 或 WACC - 1%
  dcf_value = forward_fcff / (wacc - terminal_growth)  # 简化永续增长
  dcf_status: "REGIME-SHIFT-REDCF"
```

**额外影响**：
- 任何使用历史 FCF 的 DCF 输出必须标记为 `dcf_status: INVALID-REGIME-SHIFT`
- 安全边际评分中的 `low_pe_safety_bonus` 下调一级（HIGH → MEDIUM，MEDIUM → LOW，LOW → NONE）
- `capex_ratio` 降至 < 1.5x 时，`consecutive_quarters_above_1_5x` 计数器重置为 0

---

*End of 06_VALUATION_MODELS.md — VIAIOS-P V5.4-Gem*
