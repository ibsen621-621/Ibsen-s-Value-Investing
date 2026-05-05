---
version: 5.4-Gem
role: 输出模板全集（IC Memo、投资备忘录、Watchlist、Portfolio Review、WAIT Memo）
load_priority: 8
---

# 08 OUTPUT TEMPLATES — VIAIOS-P V5.4 Complete Output Specification

> Cross-ref: [[01_CORE_PROTOCOL.md#输出契约]], [[03_DOCTRINE_PRINCIPLES.md#4-输出契约完整-schema]], [[07_RISK_FRAMEWORK.md]]

---

## 目录 (TOC)

1. [评级体系与置信度标准](#1-评级体系与置信度标准)
2. [IC Memo 格式（核心输出）](#2-ic-memo-格式核心输出)
3. [完整投资分析报告结构](#3-完整投资分析报告结构)
4. [WAIT Memo 格式](#4-wait-memo-格式)
5. [Holding Thesis Block（持仓论题块）](#5-holding-thesis-block持仓论题块)
6. [Watchlist Entry Format](#6-watchlist-entry-format)
7. [Portfolio Review Format](#7-portfolio-review-format)
8. [数据质量标注规范](#8-数据质量标注规范)
9. [置信度评分规范](#9-置信度评分规范)

---

## 1. 评级体系与置信度标准

### 推荐评级体系（V5.4 修正）

| 评级 | 含义 | 必要条件 |
|------|------|---------|
| **WAIT** | 耐心等待（第一类推荐） | 默认状态；无名称满足 APR ≥ 50 AND edge_score ≥ 4.0 |
| **TRACK** | 监视，不持仓 | edge_score < 4.0 OR ai_replicable: true |
| **BUY-CANDIDATE** | 可买入候选 | APR ≥ 50, edge_score ≥ 2.0, MoS ≥ 30%, ≥2 checkpoints |
| **SIZE-UP** | 加仓现有持仓 | APR ≥ 50, edge_score ≥ 4.0, MoS ≥ 40%, thesis 完整 |
| **HOLD** | 持有 | MoS 10%-30%, thesis 未失效 |
| **TRIM** | 减仓 | MoS < 10%, 2+ checkpoints FAILED, 或 edge_score 衰减至 < 2.0 |
| **EXIT** | 退出 | Kill Criteria 触发，或全部 checkpoints FAILED，或 MoS < 0% |
| **PASS** | 主动拒绝（不进入追踪列表） | 护城河无法识别，或 MoS < 0，或能力圈之外 |

### 不允许的评级（V5.4 删除）

| 删除的评级 | 替代 |
|---------|------|
| STRONG BUY | SIZE-UP（需要 edge_score ≥ 4.0 门控） |
| HOLD（无置信度区间） | HOLD + confidence_score 必须填写 |
| 单纯"BUY"（无 APR 计算） | BUY-CANDIDATE（需要 APR ≥ 50） |

---

## 2. IC Memo 格式（核心输出）

> IC Memo = 投资委员会格式的简洁决策摘要。所有 BUY-CANDIDATE/SIZE-UP 推荐必须产生 IC Memo。

```markdown
# IC MEMO — [公司名] ([TICKER]) [分析日期]
**评级**: [BUY-CANDIDATE / SIZE-UP / TRIM / EXIT / WAIT]
**置信度**: [0-100] / [Low/Med/High]
**安全边际**: [X]%（当前价 [P] vs 保守内在价值 [V]）

---

## 论题核心（一段话）
[用一段话描述为什么这是一个好的投资机会：核心优势 + 价格错位来源 + 催化剂]

---

## 不对称收益分析（APR 计算）

| 情景 | 内在价值估算 | 概率权重 | 加权贡献 |
|------|------------|---------|---------|
| 悲观（P10-P25） | [V_bear] | [P_bear%] | [X] |
| 基准（P50） | [V_base] | [P_base%] | [Y] |
| 乐观（P75-P90） | [V_bull] | [P_bull%] | [Z] |
| **期望值** | **[E_V]** | 100% | — |

**当前价**: [P]
**隐含上行**: [E_V/P - 1]% （可信上行，非最大情景）
**隐含下行**: [MoS 缺失情景，如护城河失效] = -[X]%
**APR（不对称收益比）**: [上行%] / [下行%] = [APR]:1
→ APR ≥ 50 门控：[通过 / 未通过]

---

## Alpha Source（优势来源）

```yaml
alpha_source:
  type: [PROBABILISTIC | INFORMATIONAL | CATALYST]
  edge_evidence: "[具体的一句话，命名特定非公开信号来源]"
  ai_replicable: [true | false]
  edge_score: [0-10]
```
→ 仓位层级：[Heavy / Standard / Tracking]（基于 edge_score 门控）

---

## 估值三角验证

| 方法 | 结果 | 主要假设 |
|------|------|---------|
| Reverse DCF 难度分 | [1-10] | 隐含增长 [X]%/年持续 [Y]年 |
| 蒙卡分布 P50 | [V] | 核心变量概率分布 |
| SOTP（若触发 VR-02） | [V] | 各业务线分解 |
| 相对估值 EV/EBITDA | [NTM] vs 同业 [X]x | 护城河调整后 ±[Z]% |

---

## 风险矩阵（Pre-Mortem 摘要）

| 风险 | 概率 | 影响 | 已覆盖 |
|------|------|------|--------|
| [最高概率失败路径1] | [X%] | -[Y]% | [是/否] |
| [最高概率失败路径2] | [X%] | -[Y]% | [是/否] |
| [核心论题失效] | [X%] | -[Y]% | [是/否] |

---

## 验证检查点（Patch 5）

```yaml
holding_thesis:
  core_assumption: "[一句话，含可测量结果]"
  falsification_checkpoints:
    - checkpoint_id: CP-001
      at: T+6M
      due_date: [YYYY-MM-DD]
      metric: "[可观测的具体指标]"
      fail_threshold: "[明确阈值]"
      status: PENDING
    - checkpoint_id: CP-002
      at: T+12M
      due_date: [YYYY-MM-DD]
      metric: "[可观测的具体指标]"
      fail_threshold: "[明确阈值]"
      status: PENDING
  exit_trigger: "[明确引用 checkpoint 结果的量化退出规则]"
```

---

## VR 规则状态

| VR 规则 | 状态 | 说明 |
|---------|------|------|
| VR-01（价格 > P75） | [触发/未触发] | [当前价格分位数] |
| VR-02（SOTP） | [触发/未触发/不适用] | [业务线数量] |
| VR-03（双轨分离） | [触发/未触发] | [净负债/EBITDA] |
| VR-04（周期常态化） | [触发/未触发/不适用] | [行业模板] |
| VR-05（定价体操） | [触发/未触发/不适用] | [研报检测] |

---

## 推荐结论

**最终推荐**: [评级]
**置信度分数**: [0-100]
**置信度区间**: [Low/Med/High]
**最大可接受买入价**: [P_max_entry]（仅 BUY-CANDIDATE/SIZE-UP 有效）
**数据截止日**: [YYYY-MM-DD]

> ⚠️ 免责声明：本分析仅为辅助参考，不构成投资建议。所有最终决策由人类操作者独立做出。
```

---

## 3. 完整投资分析报告结构

> 适用于 `MODE_FULL_DD`。以下为各 Agent 输出的整合结构。

### 报告各部分

```
1. 摘要（Executive Summary）
   - IC Memo（见格式第2节）
   - 3 句话论题摘要

2. 第一部分：法证合规（Agent-0）
   - Beneish M-Score 及 8 个分量
   - Sloan Accrual Ratio
   - Altman Z-Score（或行业替代）
   - 法证三联检结论：[PASS / FAIL]
   - 数据质量等级综合：[A/B/C/D]

3. 第二部分：基本面分析（Agent-1）
   - 5 年财务趋势（营收/EBIT/EBITDA/净利润）
   - ROIC 分析（含 [[05_BASE_RATES.md#roic_persistence]] 对比）
   - FCF Yield 计算
   - 净负债/EBITDA、利息保障倍数
   - V5.4 Patch 4：capex_ratio 跟踪结果

4. 第三部分：商业分析（Agent-2，ADVISORY-ONLY）
   - 商业模式与护城河评估
   - 单位经济模型（Unit Economics）
   - 管理层可观测代理指标（非判断性）
   - 注明：advisory_status: ADVISORY-ONLY

5. 第四部分：宏观分析（Agent-5）
   - 利率/汇率/监管环境
   - 行业周期定位
   - 黑天鹅事件标记（M-013 状态）

6. 第五部分：估值分析（Agent-6 + Agent-9）
   - Reverse DCF 及难度分
   - Damodaran 7 大工具激活状态
   - 蒙卡分布（P10/P25/P50/P75/P90）
   - SOTP 分解（若 VR-02 触发）
   - 扩张期权（若触发）
   - 双轨分离（若 VR-03 触发）
   - 动态 WACC 计算明细
   - 估值三角汇总

7. 第六部分：风险评估（Agent-7）
   - Pre-Mortem（3-5 条失败路径）
   - Kill Criteria 检查
   - VR 规则状态
   - 评级及置信度

8. 附录
   - 数据来源与质量等级明细
   - 方法论注释
   - 历史预测记录（Agent-8 格式）
   - 免责声明
```

---

## 4. WAIT Memo 格式

> 当全局 WAIT 门控触发时（Patch 7），必须产生此格式的输出。

```yaml
wait_memo:
  scan_date: <ISO-8601>
  universe_scanned: <int>            # 本次扫描的 ticker 总数
  active_pool_size: <int>            # 当前 active_conviction_pool 大小
  
  gate_failures:
    apr_below_50_count: <int>        # APR < 50 的名称数量
    edge_score_below_4_count: <int>  # edge_score < 4.0 的名称数量
    both_gates_failed_count: <int>   # 两个门控都失败的名称数量
    
  closest_to_gate:
    - ticker: <symbol>
      current_apr: <float>
      current_edge_score: <float>
      gap_to_apr_gate: <float>       # 50 - current_apr（负数表示已过）
      gap_to_edge_gate: <float>      # 4.0 - current_edge_score（负数表示已过）
      what_would_change: <string>    # "Q3 供应链确认 + 一个新渠道联系人，可能使 edge_score 至 4.2"
    
  cash_option_value:
    risk_free_rate: <float>          # 当前 10Y 国债
    implied_erp: <float>             # 当前市场 ERP
    patience_premium_note: >
      现金不是闲置的；它持有一个面向下一个十年一次错位的期权。
      这个 WAIT 是深思熟虑的立场，而非工作缺失。
  
  next_scan_date: <ISO-8601>         # 推荐下次扫描时间
  
  recommendation: WAIT              # 字面量；不得替换为其他评级
```

---

## 5. Holding Thesis Block（持仓论题块）

> 所有 Active Pool 中的名称必须有此块（Patch 5 强制）。

```yaml
holding_thesis:
  ticker: <symbol>
  established_date: <ISO-8601>        # 加入池子的日期
  
  core_assumption: >
    [一句话，现时时态，包含可测量结果。
     禁止使用纯叙事语言（"secular growth trend", "AI opportunity"）作为唯一内容。
     示例："工厂 A 产能利用率在 Q3 2027 前恢复至 ≥80%，驱动 EBIT margin 恢复至 15%+"]
  
  falsification_checkpoints:
    - checkpoint_id: CP-001
      at: T+6M
      due_date: <ISO-8601>
      metric: <string>                # 可公开获取的、可观测指标
      fail_threshold: <string>        # 明确的失败阈值（如 "<12%"）
      status: PENDING                 # PENDING | PASSED | FAILED
      actual_value: null              # 到期后填写
      
    - checkpoint_id: CP-002
      at: T+12M
      due_date: <ISO-8601>
      metric: <string>
      fail_threshold: <string>
      status: PENDING
      actual_value: null
      
    # 可选（建议）：T+18M, T+24M
      
  exit_trigger: >
    [明确引用 checkpoint 结果的量化退出规则。
     示例："若 CP-001 FAILED AND CP-002 FAILED → 在 10 个交易日内减仓 50%。
            若 CP-001 PASSED 但 Q2 2027 收入增长 < 5% → 重新评估论题。"]
            
  last_thesis_review: <ISO-8601>     # 最后一次人工审查持仓论题的日期
  next_scheduled_review: <ISO-8601>  # 下一个 checkpoint 到期日
```

---

## 6. Watchlist Entry Format

```yaml
# 监视列表条目（Tracking 层级，edge_score 0-2，APR < 50 或未满足其他门控条件）

watchlist_entry:
  ticker: <symbol>
  company_name: <string>
  added_date: <ISO-8601>
  current_recommendation: TRACK
  
  tracking_reason: <string>         # 为什么关注此标的（一句话）
  gate_status:
    apr: <float>                    # 当前 APR
    apr_gate_50_met: <bool>
    edge_score: <float>
    edge_score_gate_4_met: <bool>
    
  what_would_trigger_promotion: >
    [什么条件会将此标的从 TRACK 升级为 BUY-CANDIDATE？
     示例："若 Q2 出货量超预期 + 主要客户双源化 edge_score 超过 4.0"]
    
  ignition_state:
    signals_active: <list>          # 当前活跃的 IgnitionScanner 信号
    ignition_alert: <bool>
    
  next_review: <ISO-8601>
```

---

## 7. Portfolio Review Format

```yaml
# 组合定期审查（建议每季度执行）

portfolio_review:
  review_date: <ISO-8601>
  reviewer: <string>               # 人类审查者（非 AI）
  
  pool_summary:
    current_size: <int>            # ≤ 30
    heavy_positions: <list>        # edge_score ≥ 4.0 的名称
    standard_positions: <list>     # edge_score 2.0-4.0
    tracking_positions: <list>     # edge_score < 2.0
    wait_status: <bool>            # 当前 WAIT 门控状态
    
  checkpoint_review:
    due_this_quarter: <list>       # 本季度到期的所有 checkpoints
    passed: <list>
    failed: <list>
    actions_triggered: <list>      # 失败触发的 TRIM/EXIT 行动
    
  regime_shift_monitoring:
    names_with_flag_true: <list>   # regime_shift_flag = true 的名称
    regime_shift_redcf_required: <list>
    
  edge_decay_alerts:
    channels_expired: <list>       # current_contribution = 0 的渠道
    edge_scores_dropped: <list>    # edge_score 下降超过 2.0 的名称
    names_now_below_heavy_threshold: <list>  # edge_score 降至 < 4.0 的重仓名称
    
  wait_gate_review:
    global_wait_triggered: <bool>
    closest_candidates: <list>
    
  exit_pool_log:
    evicted_this_quarter: <list>
    eviction_reasons: <list>
```

---

## 8. 数据质量标注规范

每个关键数据点旁必须标注来源和质量等级（M-006 强制规则）：

| 等级 | 含义 | 置信度影响 |
|------|------|----------|
| **A** | 经审计财务报告数据 | 不扣分 |
| **B** | 未经审计但官方披露数据 | 扣 5 分 |
| **C** | 第三方估算/分析师一致预期 | 扣 10 分 |
| **D** | 系统推断/行业均值替代 | 扣 20 分（必须标注"降级处理"） |

**数据缺失降级决策树**：
```
STEP 1：判断缺失数据重要性
  关键指标缺失（法证三联检/ROIC/FCF/营收）
    → 触发红旗；若 Agent-0 判断无法替代 → 建议暂缓投资
  非关键指标缺失
    → 存在替代指标：标注"替代指标"
    → 无替代：使用行业中位数，标注"行业中位数替代，降级处理"

STEP 2：置信度调整
  关键数据为 D 级 → 置信度扣 20 分，输出中明确提示局限性
```

---

## 9. 置信度评分规范

```
起始分: 100
扣分项:
  - 关键数据 B 级: -5
  - 关键数据 C 级: -10
  - 关键数据 D 级: -20
  - alpha_source.ai_replicable = true: -15（降至最高 TRACK）
  - 内部人持续净卖出 > 3 季度（C-010）: -10
  - 大股东质押率 > 50%（C-015）: -10
  - Pre-Mortem 发现未覆盖重大风险: -5 至 -15
  - 数据截止 > 90 天: -10（> 180 天：BUY 评级熔断）

加分项:
  - edge_score ≥ 6.0（强优势）: +5（上限 95）
  - 多个估值方法结果一致（差异 < 10%）: +5
  - 法证三联检全部绿灯: +5

输出标准:
  High 置信度: > 70（可出现在最终报告中）
  Med 置信度: 50-70（须额外标注不确定性来源）
  Low 置信度: < 50（必须附带详细局限性说明）
```

---

*End of 08_OUTPUT_TEMPLATES.md — VIAIOS-P V5.4-Gem*
