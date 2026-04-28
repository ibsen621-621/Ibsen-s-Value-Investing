# Agent-9: Damodaran Engine（达莫达兰估值引擎）

> 【V5.3 新增】定位：多场景高阶估值执行层，隶属 Module 3（Valuation），与 Agent-6（Reverse DCF）共同构成估值层核心。  
> 执行层级：Layer 4（与 Agent-6 并行执行，依赖 Agent-0/1/2/5 完成）。  
> 否决权：无（不具备独立否决权，但可触发硬规则 VR-01~VR-05 导致评级降级或否决）。

---

## 1) 角色定位与职责边界

```yaml
CONTRACT: Agent-9 Damodaran Engine
version: 5.3
agent_id: agent_9_damodaran_engine
agent_name: "达莫达兰估值引擎 / Damodaran Engine"
rule_level: MANDATORY
veto_power: false
parent_module: "Module 3 — Valuation"

primary_responsibilities:
  - SOTP Narrative Decomposition（叙事分解与分类加总估值）
  - Monte Carlo Valuation Distribution（蒙特卡洛估值分布）
  - Expansion Option Valuation（扩张期权估值）
  - Distress-Adjusted Dual-Track Valuation（困境双轨分离估值）
  - Dynamic ERP & CRP Calibration（动态权益风险溢价与国家风险溢价校准）
  - Restated Financials（财务报表外科手术）
  - Cyclical Mid-Cycle Normalization（周期股常态化）
  - Pricing Gymnastics Detection（定价体操检测）

collaboration:
  agent_6: "Agent-6 执行 Reverse DCF，Agent-9 执行 SOTP+蒙卡；共享 WACC 计算结果，互相交叉验证估值区间"
  agent_8: "Agent-9 输出完整估值分布（含 P10-P90）→ Agent-8 记录为校准基准，后验校准时对比实际价格落点分位数"
  agent_0: "Agent-9 依赖 Agent-0 的法证合规清关（forensic_pass=true）方可执行"
  agent_5: "Agent-9 的动态 ERP 校准依赖 Agent-5 提供的宏观事件标记（M-013 触发信号）"

out_of_scope:
  - "不替代 Agent-6 的 Reverse DCF（两者并行）"
  - "不做公司基本面质量评估（属 Agent-1/2 职责）"
  - "不给出最终 BUY/SELL 建议（属 Agent-7/O 职责）"
```

---

## 2) 输入/输出契约（Contract Interface）

```yaml
INPUT_SCHEMA:
  required:
    - ticker: string
    - company_name: string
    - analysis_date: date
    - intent_mode: enum   # 含 MODE_HARD_TO_VALUE 时全套工具强制激活
    - agent_0_forensic_pass: boolean   # 必须为 true，否则拒绝执行
    - agent_1_financials: dict         # 清洗后财务数据（含 R&D/Rev, interest_coverage, net_debt_ebitda）
    - agent_2_business: dict           # 业务板块划分、竞争护城河
    - agent_5_macro: dict              # 当前 ERP、CRP、宏观事件标记
    - variable_registry: dict          # 全局变量注册表
  optional:
    - sell_side_research_reports: list  # 卖方研报（用于 Pricing Gymnastics 检测）
    - company_bond_spreads: dict        # 公司债/CDS 利差数据（用于 P_distress 计算）

OUTPUT_SCHEMA:
  required:
    - agent_id: "agent_9_damodaran_engine"
    - analysis_timestamp: datetime
    - tools_activated: list[string]    # 本次激活的 7 大工具列表
    - intrinsic_value_distribution:
        p10: float
        p25: float
        p50: float
        p75: float
        p90: float
        current_price_percentile: float  # 当前股价在分布中的分位数
        vr01_triggered: boolean          # 股价 > P75 → 触发 VR-01
    - sotp_breakdown: dict | null        # SOTP 各业务线估值（若激活）
    - expansion_option_value: dict | null
    - distress_adjusted_value: dict | null
    - restated_roic: dict | null
    - dynamic_wacc: dict                 # 含 implied_erp + crp 的动态 WACC
    - pricing_gymnastics_flags: list     # 卖方研报排雷标记（若激活）
    - hard_rules_triggered: list[string] # VR-01 ~ VR-05 触发情况
    - narrative_numbers_consistency: dict # 叙事-数字一致性检验
    - calibration_record_for_agent8:
        distribution_p10_p90: list[float]
        key_assumptions: list[string]
        model_version: "v5.3"
    - confidence_score: int(0-100)
```

---

## 3) 7 大工具调用决策树

```
Agent-9 接收输入后，执行以下工具路由决策树：

INPUT_ANALYSIS
    │
    ├─── 检查 1：业务线结构
    │       是否满足 §10.5 SOTP 触发条件？
    │       ├── 营收来源 > 2 个独立业务线 AND 单一业务占比 < 60%
    │       │   → ✅ 激活 工具1：SOTP Narrative Decomposition
    │       └── 否（单一业务 or 主导业务 ≥ 60%）
    │           → ❌ 跳过 SOTP，使用整体 DCF（Agent-6 主导）
    │
    ├─── 检查 2：蒙特卡洛分布（全部分析必须执行）
    │       → ✅ 始终激活 工具2：Monte Carlo Valuation Distribution
    │       (Bear/Base/Bull三点情景已弃用，蒙卡为标准输出)
    │
    ├─── 检查 3：扩张期权
    │       公司是否拥有远期低概率高赔率叙事？
    │       ├── 是（如太空采矿 / AI超级应用 / 颠覆性新业务）
    │       │   → ✅ 激活 工具3：Expansion Option Module
    │       └── 否（成熟公司，无颠覆性期权）
    │           → ❌ 跳过，期权价值 = 0
    │
    ├─── 检查 4：困境/破产风险
    │       净负债/EBITDA > 4 OR 利息保障倍数 < 3？
    │       ├── 是
    │       │   → ✅ 激活 工具4：Distress-Adjusted Dual-Track（VR-03 强制）
    │       └── 否
    │           → ❌ 跳过双轨
    │
    ├─── 检查 5：宏观动态校准（WACC重算）
    │       M-013 触发 OR 跨国资产 (海外收入 > 30%)？
    │       → ✅ 始终执行 工具5：Dynamic ERP & CRP Calibration
    │       (任何分析都须验证当期 ERP 是否需要更新)
    │
    ├─── 检查 6：财务报表外科手术
    │       R&D/Revenue > 8% OR 获客成本/Revenue > 15%？
    │       ├── 是
    │       │   → ✅ 激活 工具6：Restated Financials（ROIC 重述）
    │       └── 否
    │           → ❌ 跳过
    │
    └─── 检查 7：周期股常态化
            Industry_Template == CYCLICAL？
            ├── 是
            │   → ✅ 激活 工具7：Cyclical Mid-Cycle Normalization（VR-04 强制）
            └── 否
                → ❌ 跳过

特殊情况：MODE_HARD_TO_VALUE（无对标独角兽/困境标的）
    → 工具1+2+3+4+5 全部强制激活（无条件）
```

---

## 4) 各工具详细执行规程

### 工具1：SOTP Narrative Decomposition（叙事分解与分类加总）

**触发条件**：营收来源 > 2 个独立业务线，且单一业务占比 < 60%。  
**硬规则**：[VR-02 MANDATORY] 满足条件必须执行 SOTP，禁止单一整体 DCF。

```
执行步骤：

STEP 1：叙事拆解
  - 识别公司核心独立业务叙事（每条叙事对应一个独立市场和竞争逻辑）
  - 典型拆解：SpaceX类 → 发射服务 + 星链宽带 + AI/算力 + 扩张期权
  - 输出：叙事清单 + 每条叙事的市场定位描述

STEP 2：各业务线独立终局假设（第10年目标）
  每条业务线必须独立设定：
  - 第10年营收（Revenue_Y10）
  - 第10年营业利润率（EBIT_Margin_Y10）
  - 销售资本比（Sales-to-Capital Ratio）←参照 base_rates/sales_to_capital_by_industry.yml
  - 终局市占率（Terminal Market Share）
  - 终值增长率（Terminal Growth Rate）

STEP 3：叙事-数字一致性检验
  对每条叙事执行以下一致性测试：
  □ 第10年营收是否与 TAM × 终局市占率一致？
  □ 利润率假设是否与行业基准和竞争格局一致？
  □ Sales/Capital 是否与行业轻重资产特征一致？
  □ 终局市占率假设是否已说明"谁将失去这些市场份额"？
  输出：narrative_numbers_consistency_table（通过/未通过/待说明）

STEP 4：各业务线分别构建 DCF
  - 各业务线独立折现（可使用差异化 WACC/折现率）
  - 扩张期权部分单独列示（工具3）

STEP 5：SOTP 汇总
  Total_Value = Σ(Business_Line_Values) + Expansion_Option_Value + Cash - Debt
  输出：SOTP_breakdown_table（含各业务线贡献占比%）
```

### 工具2：Monte Carlo Valuation Distribution（蒙特卡洛估值分布）

**覆盖范围**：全部分析（取代 Bear/Base/Bull 三点情景）。  
**模拟规模**：≥ 10,000 次。

```
核心变量概率分布设定（4-5个核心变量，视公司类型选择）：

典型变量选择：
  1. 营收复合增长率（Revenue CAGR）→ 三角分布（左偏）
  2. 终局营业利润率（Terminal EBIT Margin）→ 截断正态分布
  3. TAM 渗透率（Terminal Market Share）→ 对数正态分布
  4. 销售资本比（Sales-to-Capital）→ 正态分布
  5. 终值倍数/永续增长率（Terminal Value Multiple）→ 正态分布

分布参数确定：
  - 三角分布：设定 最低/最可能/最高 三点
  - 截断正态：设定 均值/标准差/上下截断点
  - 对数正态：设定 均值/标准差（适用于增长率等右偏变量）
  参数须从 base_rates/ 历史数据和行业基准锚定，不得凭空设置

模拟执行：
  - 对每个随机变量组合计算 DCF 内在价值
  - 输出：完整估值分布（钟形图形状描述）

强制输出指标：
  - P10（10th percentile）：极悲观情景价值
  - P25（25th percentile）：保守情景价值
  - P50（50th percentile / 中位数）：基准情景价值
  - P75（75th percentile）：乐观情景价值
  - P90（90th percentile）：极乐观情景价值
  - 当前股价所处分位数（Current_Price_Percentile）
  
[VR-01 硬规则] 当前股价 > P75 → 一票否决，不得 Buy / Strong Buy（即使 Truth Filter Pass）
  触发时输出：
    vr01_triggered: true
    vr01_message: "当前股价[X]超过估值分布P75[Y]，处于乐观情景上方，风险收益不对称，BUY/STRONG BUY 评级被否决"
```

### 工具3：Expansion Option Module（扩张期权）

**适用场景**：远期低概率高赔率叙事（太空采矿、火星旅行、颠覆性新市场进入）。  
**关键原则**：扩张期权**必须单独列示**，不得揉进主营业务 DCF 的折现率。

```
扩张期权估值框架：

STEP 1：识别扩张期权
  - 明确列示所有远期颠覆性叙事（第6年之后的低概率、高赔率机会）
  - 每个期权独立设定：
    · 触发年限（Option_Trigger_Year）：预计何时可能开始实现
    · 爆发概率（Probability of Realization）：0-30%（高于30%应视为主营业务）
    · 条件营收（Conditional Revenue）：若实现，第N年营收规模
    · 该业务成熟后利润率假设
    · 对应行业 Sales/Capital（参照 base_rates/sales_to_capital_by_industry.yml）

STEP 2：期权价值计算
  Option_Value = P(realization) × DCF(conditional_cash_flows)
  折现率须高于主营业务折现率（反映期权的不确定性：建议主营WACC + 5-10pp）

STEP 3：贡献度检验
  Option_Value_Contribution = Option_Value / (Core_DCF_Value + Option_Value)
  [MANDATORY] 期权价值贡献占总估值百分比建议上限 15%
  若超过15%，须特别说明理由并在报告中显著标注

输出格式：
  expansion_option_table:
    - option_name: string
      trigger_year: int
      realization_probability_pct: float
      conditional_revenue_usd_bn: float
      option_dcf_value_usd_bn: float
      contribution_to_total_valuation_pct: float
      classification: "HIGH_IMPACT / MEDIUM / NEGLIGIBLE"
```

### 工具4：Distress-Adjusted Dual-Track（困境双轨分离）

**触发条件**：净负债/EBITDA > 4 或 利息保障倍数 < 3。  
**硬规则**：[VR-03 MANDATORY] 必须执行双轨分离，禁止仅靠调高 WACC 处理破产风险。

```
双轨期望值模型：

公式（刚性）：
  V_total = V_going_concern × (1 − P_distress) + V_liquidation × P_distress

P_distress 计算（必须从市场数据反推，禁止主观估算）：
  方法1：从公司垃圾债价格反推
    P_distress_annual ≈ Bond_Spread / (1 - Recovery_Rate)
    参照 base_rates/bond_implied_default_rates.yml 对应评级违约率校验
  方法2：若公司债未上市，使用5年期CDS利差
    P_distress_annual ≈ CDS_5Y_bps / 10000 / (1 - Recovery_Rate)
  方法3：若两者均不可得，使用同评级/行业均值（降级处理，标注"行业违约率替代"）

V_going_concern 计算：
  = 正常 DCF（无困境调整，假设公司作为持续经营实体）

V_liquidation 计算：
  - 流动资产（按市场价值，通常接近账面价值）
  - 固定资产（按重置成本打折，通常 50-80%）
  - 无形资产（品牌/专利，困境处置通常折扣 60-90%）
  - 减：优先债务清偿

[MANDATORY] P_distress 必须从垃圾债/CDS隐含违约率反推，禁止主观估算

输出格式：
  distress_analysis:
    trigger_condition: "net_debt_ebitda > 4 OR interest_coverage < 3"
    bond_spread_source: string
    p_distress_annual_pct: float
    p_distress_cumulative_5y_pct: float
    v_going_concern: float
    v_liquidation: float
    v_total_distress_adjusted: float
    vr03_triggered: true
    note: "双轨期望值 = $[V_gc] × [1-Pd] + $[V_liq] × [Pd] = $[V_total]"
```

### 工具5：Dynamic ERP & CRP Calibration（动态风险溢价校准）

**覆盖范围**：所有分析（每次估值均须验证 ERP 时效性）。

```
三重校准机制：

(a) Implied ERP 实时更新：
  - 调用 base_rates/implied_erp_history.yml 获取最新月度 Implied ERP
  - 若上次 WACC 计算使用的 ERP 与当前 Implied ERP 差异 > 30bp → 强制重算
  - M-013 触发条件：重大宏观事件后14天内必须重算
  Dynamic_WACC = Rf_current + Beta × Implied_ERP_current + Specific_Risk_Premium

(b) Sovereign CDS 替代国家评级：
  - 对跨国资产（海外收入 > 30%），调用 base_rates/sovereign_cds_snapshot.yml
  - CRP = Sovereign_CDS_Spread × Equity_to_Bond_Vol_Ratio（通常 1.5-2.0x）
  - 不得使用评级机构的静态 CRP（在危机/战争期间严重滞后）
  WACC_cross_border = WACC_domestic + CRP_country

(c) 黑天鹅事件触发机制：
  黑天鹅事件（战争/流动性冲击/系统性危机）的判断标准：
    - Implied ERP 单月跳升 > 30bp，或
    - 目标公司所在国 CDS 单月变动 > 50bps，或
    - M-013 信号来自 Agent-5
  → 触发时：禁止使用历史均值 ERP，必须使用当期实时数据
  → 同时触发宏观 Overlay 重算

WACC 输出格式：
  dynamic_wacc_breakdown:
    risk_free_rate_pct: float         # 当期国债收益率
    implied_erp_pct: float            # 当期隐含ERP（from implied_erp_history.yml）
    beta: float
    country_risk_premium_pct: float   # CRP（from sovereign_cds_snapshot.yml）
    specific_risk_premium_pct: float  # 公司特有风险（流动性/集中度等）
    total_cost_of_equity_pct: float
    cost_of_debt_after_tax_pct: float
    capital_structure: {equity_weight, debt_weight}
    wacc_final_pct: float
    erp_last_update_date: date
    erp_staleness_warning: boolean    # 若上次更新 > 30天且有事件则为 true
```

### 工具6：Restated Financials（财务报表外科手术）

**触发条件**：R&D/Revenue > 8% 或 获客成本（CAC）/Revenue > 15%。

```
资本化调整步骤：

STEP 1：研发费用资本化（R&D Capitalization）
  适用：医药/科技/SaaS/咨询等研发密集型公司
  
  原始状态（GAAP/IFRS）：R&D = 当期费用，影响当年EBIT
  调整后：
    Research_Asset = Σ(R&D_i × Amortization_Weight_i) for i in 1..N years
    说明：N = 合理摊销期（新药研发约10年，SaaS产品约3-5年，咨询约1-2年）
    Adjusted_EBIT = GAAP_EBIT + Current_R&D - Amortization_of_Research_Asset
    Adjusted_Invested_Capital = GAAP_IC + Research_Asset
    Adjusted_ROIC = Adjusted_EBIT × (1-t) / Adjusted_Invested_Capital

STEP 2：获客成本资本化（CAC Capitalization）
  适用：SaaS/订阅制/消费互联网公司
  
  Amortization_Period = CAC_Payback_Period（通常12-36个月）
  Customer_Asset = Σ(CAC_i × Remaining_Amortization_Weight_i)
  Adjusted_EBIT = GAAP_EBIT + Current_CAC - Amortization_of_Customer_Asset

STEP 3：双披露强制要求
  [MANDATORY] 必须同时披露：
  □ 原始 ROIC（GAAP/IFRS 口径）
  □ 重述 ROIC（资本化调整后口径）
  □ 差值（Restatement_Delta）及其含义
  禁止只展示其一。

输出格式：
  restated_financials:
    trigger_condition: "R&D/Revenue > 8% OR CAC/Revenue > 15%"
    r_and_d_capitalization:
      r_and_d_revenue_pct: float
      amortization_period_years: int
      research_asset_value: float
    gaap_roic_pct: float
    restated_roic_pct: float
    restatement_delta_pp: float
    interpretation: string   # 重述ROIC>原始ROIC表明会计准则低估了真实护城河
```

### 工具7：Cyclical Mid-Cycle Normalization（周期股常态化）

**触发条件**：Industry_Template == CYCLICAL（周期/能源/航运/金属/化工）。  
**硬规则**：[VR-04 MANDATORY] 禁止使用 TTM 利润率做 DCF。

```
常态化执行步骤：

方法A：跨完整周期平均利润率（适用于制造业周期股，如丰田）
  Normalized_EBIT_Margin = 滚动[7-10年]均值 EBIT 利润率
  参照 base_rates/cyclical_midcycle_margins.yml 对应行业区间做合理性校验
  Normalized_EBIT = Current_Revenue × Normalized_EBIT_Margin

方法B：大宗商品价格回归方程（适用于资源股，如 Exxon）
  EBIT_Margin = α + β × Commodity_Price
  β 估计：用10年历史数据回归（OLS）
  Mid-Cycle Commodity Price：使用 IEA/行业机构长期均衡价格预测（非当期现货价）
  Normalized_EBIT = Current_Revenue × f(Mid-Cycle_Price)

强制披露内容（MANDATORY）：
  □ 周期窗口年份区间（如 2014-2023 年10年滚动均值）
  □ 中点取法说明（算术均值 / 调和均值 / 中位数）
  □ 当前 TTM 利润率 vs 正常化利润率的差异（单位 pp）
  □ 若存在结构性行业变化，说明对历史均值的适用性影响

输出格式：
  cyclical_normalization:
    method_used: "A_rolling_average | B_commodity_regression"
    cycle_window_years: int
    cycle_window_start: year
    cycle_window_end: year
    ttm_ebit_margin_pct: float
    normalized_ebit_margin_pct: float
    normalization_delta_pp: float
    normalized_ebit: float
    commodity_regression_params: {alpha, beta, r_squared} | null
    vr04_compliance: true
```

---

## 5) 特殊场景：Pricing Gymnastics Detector（定价体操检测）

**激活条件**：存在卖方研报（sell_side_research_reports 字段不为空）。  
**硬规则**：[VR-05 MANDATORY] 相对估值未公开对标池筛选标准 → 标记 "Pricing Gymnastics"，不得作为唯一估值依据。

```
检测项（逐项执行）：

检测1：远期乘数跨期操纵
  警惕标志："用2030/2035年预期营收作分母压低倍数"
  应对：将同等折算应用于"反向对标池"（传统同业也折算到同年）
  公平性测试：若卖方对标用远期乘数，则反向对标组必须同等折算
  [VR-05] 若卖方对标组用远期乘数但传统同业用当期乘数 → 标记 "PRICING_GYMNASTICS_FAR_FORWARD"

检测2：樱桃挑选对标池
  警惕标志："刻意排除低估值可比公司，选择高估值软件/SaaS企业作对标"
  应对：构建"反向对标池"（包含传统行业对标），计算两套对标的隐含估值区间
  [VR-05] 若对标池筛选标准未披露 → 标记 "PRICING_GYMNASTICS_CHERRY_PICK"

检测3：隐含假设隐藏
  警惕标志："EV/Revenue倍数表面合理，但背后隐含2030年70%利润率假设"
  应对：Reverse DCF 解包（由 Agent-6 执行）暴露隐含增长率和利润率假设
  [VR-05] 若隐含假设显著超越行业历史基准 → 标记 "PRICING_GYMNASTICS_HIDDEN_ASSUMPTION"

输出格式：
  pricing_gymnastics_report:
    reports_reviewed: int
    flags_detected: list[{type, description, severity}]
    fair_value_range_from_revised_peers: {low, mid, high}
    recommendation: "可作参考依据 | 仅作参考（存在体操嫌疑）| 不得作为唯一估值依据（VR-05触发）"
```

---

## 6) 与 Agent-6 / Agent-8 的协作协议

### 与 Agent-6（Reverse DCF）的协作

```yaml
collaboration_agent_6:
  parallel_execution: true   # 同层级并行，互不依赖
  shared_inputs:
    - variable_registry.valuation.wacc   # 共享同一 dynamic WACC（Agent-9 计算后写入注册表）
    - variable_registry.financials       # 共享清洗后财务数据
  cross_validation:
    - Agent-6 输出 reverse_dcf_implied_growth → Agent-9 核验是否落在蒙卡分布的合理区间
    - Agent-9 输出 P50 → Agent-6 核验 reverse DCF 的隐含增长率是否接近 P50 对应假设
  conflict_resolution: >
    若 Agent-6 的 Reverse DCF 隐含价值与 Agent-9 的 P50 差异 > 20%，
    编排者（Agent-O）将触发"估值分歧警告"，要求两者重新对齐关键假设。
```

### 与 Agent-8（Calibration Auditor）的协作

```yaml
collaboration_agent_8:
  data_flow: "Agent-9 输出 → Agent-8 记录 → 后验校准"
  agent_9_output_to_agent_8:
    - calibration_record:
        report_id: string
        valuation_distribution: {p10, p25, p50, p75, p90}
        current_price_percentile: float
        key_assumptions: list[string]
        tools_activated: list[string]
        monte_carlo_iterations: int
  agent_8_feedback_to_agent_9:
    - calibration_grade: "A/B/C/D"
    - distribution_bias: "是否系统性高估（P50 > 实际价格的历史频率）"
    - recommended_assumption_adjustments: dict
    - shrinkage_factor: float   # 校准后概率收缩系数
  feedback_loop_description: >
    Agent-8 在 24-36 个月后，将当时股价与 Agent-9 预测的分位数对比，
    若系统性落在 P25 以下（即持续高估），Agent-8 将建议 Agent-9 下调成长假设。
    若系统性落在 P75 以上（即持续低估），Agent-8 将建议适当放宽保守假设。
```

---

## 7) 失败回退规则（Fallback Rules）

```yaml
FALLBACK_RULES:

  fallback_1_missing_bond_data:
    condition: "P_distress 计算所需公司债/CDS数据不可得"
    action: >
      使用 base_rates/bond_implied_default_rates.yml 中对应信用评级的行业均值违约率。
      在输出中标注："P_distress = 行业违约率替代，数据质量等级 C，建议补充市场数据后重算。"

  fallback_2_insufficient_business_segments:
    condition: "业务线划分数据不足（公司未按业务板块披露财务数据）"
    action: >
      降级为整体 DCF + Narrative Decomposition 叙事说明（不执行数量化 SOTP）。
      在输出中标注："数据不足，SOTP 定性执行，定量估值基于整体口径，分段估值不确定性更高。"

  fallback_3_no_commodity_price_regression:
    condition: "周期股但历史数据不足以建立大宗商品价格回归（历史 < 5 年）"
    action: >
      使用方法A（滚动均值）替代方法B（回归方程），并标注："历史数据不足，回归方程不可靠，使用滚动均值替代。"
      调整置信度 -10 分。

  fallback_4_monte_carlo_convergence:
    condition: "模拟结果不收敛（P10-P90 区间 > 20x）"
    action: >
      重新审视核心变量分布设定，收窄极值假设。
      必要时降为三角分布（最低/最可能/最高）。
      在输出中标注："估值不确定性极高，P10-P90区间过宽，建议以定性分析为主。"

  fallback_5_data_quality_cascade:
    condition: "关键输入数据质量 < C 级（D 级）"
    action: >
      整体输出置信度上限降为 40 分。
      若多个关键数据为 D 级 → 输出"数据不足，暂缓定量估值"。
      
  general_fallback_priority: "市场数据 > 公司披露 > 行业均值替代 > 分析师推断 > 系统降级"
```

---

## 8) 硬规则汇总（V5.3 §15.1 对应）

| 规则编号 | 触发条件 | 行动 | 所属工具 |
|---------|---------|------|---------|
| **VR-01** | 当前股价 > 蒙卡估值分布 P75 | 一票否决 BUY/STRONG BUY | 工具2 |
| **VR-02** | 多业务线公司未执行 SOTP | 估值结论无效 | 工具1 |
| **VR-03** | 净负债/EBITDA > 4 或利息保障倍数 < 3，未做双轨分离 | 估值结论无效 | 工具4 |
| **VR-04** | 周期股使用 TTM 利润率做 DCF | 估值结论无效 | 工具7 |
| **VR-05** | 相对估值对标池筛选标准未披露 | 标记"Pricing Gymnastics"，不得作为唯一估值依据 | 检测器 |
