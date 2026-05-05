---
version: 5.4-Gem
role: 风险框架（Pre-Mortem、Kelly Sizing、五大一票否决规则、V5.4 八项 Patch 风险控制）
load_priority: 7
---

# 07 RISK FRAMEWORK — VIAIOS-P V5.4 Complete Risk Control System

> Cross-ref: [[01_CORE_PROTOCOL.md]], [[03_DOCTRINE_PRINCIPLES.md]], [[06_VALUATION_MODELS.md]]

---

## 目录 (TOC)

1. [风险框架总览](#1-风险框架总览)
2. [五大一票否决规则（VR-01~VR-05）](#2-五大一票否决规则vr-01vr-05)
3. [Kill Criteria（终止标准）](#3-kill-criteria终止标准)
4. [Margin of Safety（安全边际）规则](#4-margin-of-safety安全边际规则)
5. [Pre-Mortem 事前验尸流程](#5-pre-mortem-事前验尸流程)
6. [Kelly Criterion 仓位管理](#6-kelly-criterion-仓位管理)
7. [Drawdown 控制规则](#7-drawdown-控制规则)
8. [V5.4 Patch 2：Active Conviction Pool 上限](#8-v54-patch-2active-conviction-pool-上限)
9. [V5.4 Patch 3：IgnitionScanner 信号协议](#9-v54-patch-3ignitionscanner-信号协议)
10. [V5.4 Patch 4：Reinvestment Regime Shift Alarm](#10-v54-patch-4reinvestment-regime-shift-alarm)
11. [V5.4 Patch 5：Verification Checkpoint Ledger](#11-v54-patch-5verification-checkpoint-ledger)
12. [V5.4 Patch 7：Patience Premium & WAIT Gate](#12-v54-patch-7patience-premium--wait-gate)
13. [V5.4 Patch 8：Human/Machine Boundary Charter](#13-v54-patch-8humanmachine-boundary-charter)
14. [V5.4 验收测试清单](#14-v54-验收测试清单)

---

## 1. 风险框架总览

VIAIOS-P V5.4 的风险框架由三层组成：

**第一层：系统性保护（硬规则）**
- 五大一票否决规则（VR-01~VR-05）
- Kill Criteria（法证/流动性/治理红旗）
- 边界宪章（AI 不能做的事）

**第二层：结构性约束（条件规则）**
- 安全边际要求（≥30%）
- APR ≥ 50 门控
- edge_score 门控
- 30 名仓位上限

**第三层：行为纪律（最佳实践）**
- Pre-Mortem 强制执行
- Kelly 仓位管理
- Verification Checkpoint（持仓中的可证伪性）

---

## 2. 五大一票否决规则（VR-01~VR-05）

### VR-01：估值过高否决

```
[VR-01 MANDATORY — 由 Agent-9 工具2 执行]
条件：当前股价 > 蒙卡分布 P75
后果：禁止 BUY / STRONG BUY 评级（无例外）

vr01_triggered: true
vr01_message: "当前股价[X]超过估值分布P75[Y]，处于乐观情景上方，
               风险收益不对称，BUY/STRONG BUY 评级被否决"
```

### VR-02：SOTP 强制执行

```
[VR-02 MANDATORY — 由 Agent-9 工具1 执行]
条件：业务线 > 2 AND 任意单一业务占营收 < 60%
后果：禁止单一整体 DCF；必须执行 SOTP 分解

vr02_triggered: true
vr02_message: "公司存在[N]个独立业务叙事，单一业务占比最高[X]%，
               低于 60%，强制执行 SOTP 分解，禁止整体 DCF"
```

### VR-03：困境双轨强制执行

```
[VR-03 MANDATORY — 由 Agent-9 工具4 执行]
条件：净负债/EBITDA > 4 OR 利息保障倍数 (ICR) < 3
后果：禁止仅靠调高 WACC 处理破产风险；必须执行双轨分离

vr03_triggered: true
禁止声明：本分析禁止仅靠将 WACC 从[X]%调高至[Y]%来处理破产风险（VR-03 合规）
```

### VR-04：周期股常态化强制执行

```
[VR-04 MANDATORY — 由 Agent-9 工具7 执行]
条件：Industry_Template == CYCLICAL AND 使用了 TTM 利润率
后果：禁止使用当前 TTM 利润率做 DCF；必须使用中周期均值

vr04_triggered: true
vr04_message: "CYCLICAL 模板已激活，TTM 利润率[X]%与中周期均值[Y]%偏差[Z]pp，
               强制使用中周期均值[Y]%作为 DCF 终值假设"
```

### VR-05：定价体操否决

```
[VR-05 MANDATORY — 由 Agent-9 Pricing Gymnastics Detector 执行]
条件：检测到卖方研报中的定价体操手法
后果：该研报不得作为唯一估值依据；需进行独立估值

触发标记：
  PRICING_GYMNASTICS_FAR_FORWARD：用远期（5-10年后）收益计算"当前"估值
  PRICING_GYMNASTICS_ADJUSTED_METRICS：过度使用"调整后"指标剔除不利因素
  PRICING_GYMNASTICS_PEER_SELECTION：选择性选取高倍数同业
  PRICING_GYMNASTICS_PEAK_PE：用历史峰值盈利年份的 P/E 掩盖高估
```

---

## 3. Kill Criteria（终止标准）

以下任一条件触发，立即停止分析并输出 `TERMINATE`：

```yaml
kill_criteria:
  # 法证红旗（三重并发）
  forensic_triple_red:
    - condition: Beneish M-Score > -1.78
    - condition: Sloan Accrual Ratio > 10%
    - condition: Altman Z-Score < 1.81
    - logic: AND（三者同时触发）
    - action: TERMINATE

  # 会计诚信
  audit_integrity:
    - non_standard_audit_opinion: true
    - action: TERMINATE
    - note: "包括：否定意见、无法表示意见、保留意见（重大事项）"

  # 现金流舞弊
  cashflow_fraud_signal:
    - condition: "经营现金流/净利润 < 0.3 连续 3 年"
    - condition: "应收账款增长率 - 营收增长率 > 30pp 连续 2 年"
    - logic: OR（任一触发）
    - action: TERMINATE

  # 债务危机
  debt_crisis:
    - condition: "利息保障倍数 (ICR) < 1.0"
    - condition: "净负债/EBITDA > 8x AND 短期到期债务 > 流动资产 × 1.5"
    - logic: OR（任一触发）
    - action: TERMINATE
    
  # 管理层欺诈记录
  management_integrity:
    - condition: "监管机构（SEC/SFC/CSRC）已确认欺诈调查"
    - action: TERMINATE
```

---

## 4. Margin of Safety（安全边际）规则

```
安全边际 = 1 - (当前股价 / 保守内在价值)

最低安全边际要求：
  BUY-CANDIDATE: MoS ≥ 30%（即当前价格 ≤ 保守内在价值的 70%）
  SIZE-UP: MoS ≥ 40%（更高确信度，更大折扣）
  TRIM 信号: MoS < 10%（价格接近内在价值）

保守内在价值 = 蒙卡 P25（第25百分位，偏保守情景）
              或 SOTP 熊案总和（取其较低者）

安全边际 < 30%：无论其他指标多好，必须输出 PASS / 不投资
```

---

## 5. Pre-Mortem 事前验尸流程

**激活时机**：分析接近完成、准备给出最终评级之前（强制）

### 执行步骤

```
Step 1：假设失败
  设定：这项投资在 [3年] 后出现了 [-50%] 的亏损

Step 2：逆向工程失败路径（构建 3-5 条最合理的失败路径）
  每条失败路径包含：
  - 描述：具体的宏观/行业/公司级别失败机制
  - 触发概率：P(path) = [X%]
  - 已在哪个 Agent 中分析：[Agent-N] / 未覆盖

Step 3：覆盖检查
  □ 最高概率失败路径是否在现有分析中充分覆盖？
  □ 若未覆盖，是否调整置信度分数？
  □ 是否存在"分析盲点"（尚未建模的失败机制）？

Step 4：输出标准格式
```

```yaml
pre_mortem:
  hypothetical_loss: "-50% in 3 years"
  failure_paths:
    - path_id: FP-001
      description: <string>
      probability_pct: <float>
      covered_by_agent: <Agent-N | "未覆盖">
      coverage_quality: <充分 | 部分 | 未覆盖>
    - path_id: FP-002
      # ...
  uncovered_risk_blindspots: <list>
  impact_on_final_rating: <调整 | 维持>
  adjusted_confidence_delta: <int>  # 若调整评级，置信度变化幅度
```

---

## 6. Kelly Criterion 仓位管理

### Kelly 公式

```
Kelly_Fraction = (p × b - q) / b

其中：
  p = 成功概率（BUY thesis 实现概率）
  q = 1 - p = 失败概率
  b = 收益/损失比 ≈ APR（不对称收益比）
    b = credible_upside_pct / modeled_downside_pct

Full Kelly = Kelly_Fraction × portfolio_size（不推荐，波动过大）
Half Kelly = 0.5 × Kelly_Fraction × portfolio_size（推荐基准）
Quarter Kelly = 0.25 × Kelly_Fraction × portfolio_size（高度谨慎时）
```

### V5.4 仓位管理规则

```yaml
position_sizing_rules:
  # 系统提供 structured memo，人类最终决定（Patch 8）
  
  max_single_position: 10%      # 无论 Kelly 计算多大，不超过 10%
  max_sector_concentration: 30% # 任何单一行业不超过 30%
  
  # edge_score 门控（Patch 6）
  heavy_position_requires:
    edge_score_min: 4.0         # 硬门控，不可绕过
    mos_min: 40%                # 安全边际要求更高
    conviction_level: HIGH      # confidence_score > 70
  
  # APR 门控（Patch 7）
  buy_candidate_requires:
    apr_min: 50                 # 不对称比至少 50:1（上行/下行）
    edge_score_min: 2.0         # 至少 Standard 层级
    
  # 减仓信号
  trim_signals:
    mos_below: 10%              # 价格接近内在价值
    failed_checkpoints: 2       # 两个验证检查点 FAILED（Patch 5）
    edge_score_drops_below: 2.0 # 优势衰减
```

**注意**：`position_size_recommendation`（具体百分比建议）属于 Human 边界任务（Patch 8），系统只提供 `position_size_memo`（结构化分析备忘录），不直接输出具体仓位百分比。

---

## 7. Drawdown 控制规则

```yaml
drawdown_controls:
  # 组合级别
  portfolio_level:
    max_drawdown_alert: 15%     # 组合回撤 > 15% → 生成风险审查报告
    max_drawdown_hard_stop: 25% # 组合回撤 > 25% → 强制生成完整再评估
    
  # 单个持仓级别
  position_level:
    stop_loss_memo_trigger: 20% # 单持仓回撤 > 20% → 生成 re-evaluation memo
    holding_thesis_review: 15%  # 单持仓回撤 > 15% → 检查 verification checkpoints
    
  # 禁止行为
  prohibited:
    - "不得因股价下跌单纯加仓（平均成本），除非 holding_thesis 未发生变化"
    - "不得设置固定止损价格（价格本身不是 thesis failure 的证据）"
    - "止损应基于 thesis 失效，而非价格变动"
```

---

## 8. V5.4 Patch 2：Active Conviction Pool 上限

```yaml
active_conviction_pool:
  max_size: 30                  # 硬上限，不可例外
  
  # 当 current_size = 30 时，添加新名称必须先驱逐
  eviction_protocol:
    priority_sort:
      1st: lowest edge_score    # 最低优势分数首先驱逐
      2nd: lowest apr           # 其次最低不对称比
      3rd: oldest_unverified_checkpoint  # 最后：最旧未验证检查点
    
    required_before_add: true   # 驱逐必须先于 ADD 事件
    event_log_required: true    # 所有 ADD/EVICT 事件必须记录在 pool_event_log

验收测试：
  - active_conviction_pool.current_size ≤ 30（任何时间点的系统状态快照）
  - 每个 ADD 事件（当 current_size = 30 时）必须紧接着一个 EVICT 事件
  - 驱逐候选按序排列：最低 edge_score → 最低 APR → 最旧未验证检查点
```

---

## 9. V5.4 Patch 3：IgnitionScanner 信号协议

```yaml
ignition_scanner:
  trigger_rule: "count(live_signals == true) >= 3 in rolling 90-day window"
  
  monitored_signals:
    - signal_id: S-001
      name: order_book_delta_positive
      description: "行业净订单簿失衡从负转正（可从公司财报/会议转为正向声明）"
    - signal_id: S-002
      name: spot_price_inflection
      description: "关键原材料/产品的现货价格穿越 200 日均线"
    - signal_id: S-003
      name: capex_guidance_raised
      description: "过去 90 天，行业内 ≥2 家名称上调 capex 指引"
    - signal_id: S-004
      name: hiring_surge
      description: "行业目标职位招聘量 Z-Score > 2.0（vs 12个月基线）"
    - signal_id: S-005
      name: paper_citation_surge
      description: "行业学术论文引用量 > 2σ 基准（预先商业化信号）"
    - signal_id: S-006
      name: supply_chain_lead_time_extension
      description: "交货期延长（来自供应链联系人，非公开来源）"
  
  constraints:
    no_recommendation_field: true     # 绝对禁止 recommendation 字段
    no_price_target: true             # 禁止价格目标
    no_dcf: true                      # 禁止 DCF 估值
    human_action_required: true       # 必须包含人工核实指示
    signal_window_days_max: 90        # ignition_alert = true 时，信号窗口 ≤ 90 天
```

---

## 10. V5.4 Patch 4：Reinvestment Regime Shift Alarm

```yaml
reinvestment_regime_tracking:
  # 每个名称的数据记录
  capex_ttm: <float>                  # 过去12个月资本支出（百万美元）
  capex_5y_avg: <float>               # 5年平均年资本支出（百万美元）
  capex_ratio: <float>                # = capex_ttm / capex_5y_avg
  consecutive_quarters_above_1_5x: <int>  # 连续 capex_ratio > 1.5 的季度数
  regime_shift_flag: <bool>           # true if consecutive_quarters >= 2
  last_updated: <ISO-8601>

  trigger_conditions:
    - "capex_ratio > 1.5 连续 ≥2 季度 → regime_shift_flag = true"
    
  consequences:
    dcf_invalidation:
      - "所有使用历史 FCF 外推的 DCF 标记为 dcf_status: INVALID-REGIME-SHIFT"
      - "这些 DCF 不得作为任何高于 TRACK 推荐的估值锚点"
    
    forced_redcf:
      - "强制使用 Regime-Shift Re-DCF（见 [[06_VALUATION_MODELS.md#13]]）"
      - "new_reinvestment_rate = capex_ttm / (NOPAT + D&A)"
      
    safety_bonus_downgrade:
      - "low_pe_safety_bonus: HIGH → MEDIUM（若当前为 HIGH）"
      - "low_pe_safety_bonus: MEDIUM → LOW（若当前为 MEDIUM）"
      - "low_pe_safety_bonus: LOW → NONE（若当前为 LOW）"
    
    counter_reset:
      - "capex_ratio 降至 < 1.5x 的任何季度 → consecutive_quarters 重置为 0"
```

---

## 11. V5.4 Patch 5：Verification Checkpoint Ledger

```yaml
holding_thesis_requirements:
  # 所有 Active Pool 名称必须有此块（无例外）
  core_assumption:
    format: "一句话，现时时态，包含可测量结果"
    prohibited_terms: ["narrative", "story", "theme", "secular growth"]  # 作为唯一证据时
    example_valid: "Fab A 产能利用率在 Q3 2027 前恢复至 ≥80%"
    example_invalid: "AI 长期增长趋势支撑需求"
  
  falsification_checkpoints:
    minimum_count: 2              # 至少 2 个检查点（无例外）
    first_checkpoint_max: "T+6M" # 第一个检查点不得晚于 T+6M
    required_fields:
      - checkpoint_id             # CP-001, CP-002, ...
      - at                        # T+3M | T+6M | T+12M | T+18M | T+24M
      - due_date                  # 基于仓位建立日的绝对日期（ISO-8601）
      - metric                    # 可公开获取的、可观测指标
      - fail_threshold            # 明确的失败阈值
      - status                    # PENDING | PASSED | FAILED
      - actual_value              # 到期日填写；之前为 null
  
  exit_trigger:
    must_reference_checkpoints: true
    example: "若 CP-001 FAILED AND CP-002 FAILED → 50% 减仓。若 3 个检查点均 FAILED → 5个交易日内完全退出"
  
  consequences:
    no_holding_thesis: "无法获得 BUY-CANDIDATE 或 SIZE-UP 推荐"
    two_checkpoints_failed: "recommendation 自动降级至 TRIM 或 EXIT（按 exit_trigger 规则）"
```

---

## 12. V5.4 Patch 7：Patience Premium & WAIT Gate

```yaml
wait_gate:
  global_condition: "NOT EXISTS (name IN active_conviction_pool WHERE apr >= 50 AND edge_score >= 4.0)"
  
  when_triggered:
    mandatory_output: "WAIT"
    must_include_wait_memo: true
    
  agent_penalty_rule:
    "任何代理发出 BUY-CANDIDATE/SIZE-UP 而未满足门控条件（APR ≥ 50 AND edge_score ≥ 4）的输出，
     必须标记 gate_violation: true，推荐被 Agent-7 覆盖为 TRACK"
  
  cash_as_option:
    modeling_required: true
    components:
      - risk_free_rate: "当前 10Y 国债收益率"
      - implied_erp: "当前市场 ERP（参照 [[05_BASE_RATES.md#implied_erp_history]]）"
      - opportunity_set: "closest_to_gate 名称距离通过两个门控的距离"

wait_memo_validation:
  gate_failures_must_be_accurate: true
  closest_to_gate_min_entries: 1
  next_scan_date_required: true
```

---

## 13. V5.4 Patch 8：Human/Machine Boundary Charter

```yaml
human_boundary_triggers:
  # 输出中出现以下任一字段，触发边界检查
  - "final_buy_decision"
  - "final_exit_decision"
  - "final_hold_decision"
  - "management_quality_score"
  - "capital_allocation_intent_score"
  - "position_size_recommendation"
  - "ignition_verification_confirmed"

enforcement_action:
  advisory_status: "ADVISORY-ONLY"
  recommendation: null                 # 必须为 null，不得填写
  charter_violation_warning: >
    "此输出触及 DOCTRINE-001 人机边界宪章中标记为 Human-owner 的任务行。
     系统仅提供顾问性背景。推荐字段未填写。人类操作者必须做出此判断。"
  triggering_field: <string>           # 触发边界检查的字段名称
  triggering_charter_row: <string>     # 被违反的宪章行逐字文本

advisory_memo_structure:
  # AI 可以为 Human 行提供的内容
  topic: <string>
  observable_proxies:
    - proxy: <string>
      data: <string>
      interpretation_note: "代理指标仅供参考。最终判断需要关系和背景（Human）"
  structured_data_for_human: <any>    # 标记为输入（供人使用），非输出
```

---

## 14. V5.4 验收测试清单

### Pool 完整性（Patch 2）

- [ ] `active_conviction_pool.current_size` ≤ 30（所有系统状态快照）
- [ ] 每个 ADD 事件（current_size = 30 时）紧接着一个 EVICT 事件
- [ ] 驱逐候选按序排列：最低 edge_score → 最低 APR → 最旧未验证检查点

### Alpha Source 门控（Patch 1）

- [ ] 没有输出记录同时存在 `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}` AND `alpha_source.ai_replicable = true`
- [ ] 没有高于 WAIT 的输出记录有 null 或缺失的 `alpha_source.edge_evidence`

### 制度转变（Patch 4）

- [ ] 没有 `dcf_status = INVALID-REGIME-SHIFT` 的 DCF 输出作为任何高于 TRACK 推荐的估值锚点
- [ ] `regime_shift_flag: true` 的名称在输出中包含非 null 的 `new_reinvestment_rate` 的 `regime_shift_redcf` 块
- [ ] `regime_shift_flag: true` 时，`low_pe_safety_bonus` 下调（而非省略）一级

### 验证检查点（Patch 5）

- [ ] 没有 BUY-CANDIDATE 或 SIZE-UP 推荐存在于 `holding_thesis` 缺失或有少于 2 个 `falsification_checkpoints` 的名称
- [ ] 2 个 `status: FAILED` 检查点触发 `recommendation ∈ {TRIM, EXIT}`（按 exit_trigger 规则）

### WAIT 门控（Patch 7）

- [ ] 当 Active Pool 没有名称同时满足 `apr ≥ 50 AND edge_score ≥ 4.0` 时，现金分配推荐为 WAIT + 非 null 的 `wait_memo`
- [ ] `gate_violation: true` 出现在任何在门控未满足时发出 `BUY-CANDIDATE/SIZE-UP` 的代理输出

### Edge Registry（Patch 6）

- [ ] `SIZE-UP` 从不出现在 `edge_score < 4.0` 的名称
- [ ] 只有 `paper_tracking` 渠道（decay 180天），90天前联系：`current_contribution = 0.5 × (1 − 90/180) = 0.25`
- [ ] 没有 `edge_registry/{TICKER}.yaml` 文件 → `edge_score = 0.0`

### IgnitionScanner（Patch 3）

- [ ] `ignition_alert: true` 要求 ≥3 个 `live_signals = true` 在 90 天窗口内
- [ ] IgnitionScanner 输出从不包含 `recommendation` 字段或价格目标
- [ ] `signal_window_days ≤ 90`（当 `ignition_alert = true` 时）

### 人机边界宪章（Patch 8）

- [ ] Agent-2 输出从不包含 `recommendation` 字段；只包含 `advisory_memo` 和 `observable_proxies`
- [ ] 在 Agent-7 输出中注入 `management_quality_score: 9` 导致输出 `recommendation: null` 和 `charter_violation_warning` 被填写
- [ ] 没有输出包含硬性仓位建议（如"买入8%投资组合"）而没有人工审查标志

---

*End of 07_RISK_FRAMEWORK.md — VIAIOS-P V5.4-Gem*
