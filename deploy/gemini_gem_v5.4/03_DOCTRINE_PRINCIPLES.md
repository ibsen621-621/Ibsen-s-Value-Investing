---
version: 5.4-Gem
role: 投资准则（DOCTRINE-001）、巴菲特/芒格/Damodaran 思想精要、失败模式防范
load_priority: 3
---

# 03 DOCTRINE & PRINCIPLES — VIAIOS-P V5.4 Investment Doctrine

> Cross-ref: [[01_CORE_PROTOCOL.md]], [[04_EDGE_REGISTRY.md]]

---

## 目录 (TOC)

1. [DOCTRINE-001 全文](#1-doctrine-001全文)
2. [五大根本命题](#2-五大根本命题)
3. [人机边界宪章](#3-人机边界宪章)
4. [输出契约（完整 Schema）](#4-输出契约完整-schema)
5. ["WAIT" 默认状态](#5-wait-默认状态)
6. [此准则存在以防止的失败模式](#6-此准则存在以防止的失败模式)
7. [巴菲特/芒格思想精要](#7-巴菲特芒格思想精要)
8. [Damodaran 思想精要](#8-damodaran-思想精要)
9. [DOCTRINE-001 修正协议](#9-doctrine-001-修正协议)
10. [一句话总结](#10-一句话总结)

---

## 1. DOCTRINE-001 全文

> **状态**：基础性（Tier-0）  
> **范围**：对所有 VIAIOS-P 版本 ≥ V5.4 具有约束力  
> **原则**：准则先于实施。任何与本文件矛盾的未来模块、代理或评分规则必须被拒绝，或触发明确的准则修正（DOCTRINE-00X-amend）

### 0. 为什么存在此文件

每一个之前的 VIAIOS-P 版本（V4.x → V5.3）都沿着**逻辑轴**优化：更好的模板、更好的校准、更好的多代理分解、更好的达莫达兰整合。这些都是必要的，但**不足以**产生超额回报。

对话中浮现的残酷真相：

> *"量化基金做的正是你所做的，拥有 API 访问、卖方领先倍数和 TTM 数据组合，已经做了 10 年以上。如果你的优势是'对公开信息进行更好的分析'，你充其量只是在没有信息优势的概率博弈中——预期结果是成为退出流动性。"*

此准则重置了框架的目的：

**系统的工作不是寻找好公司。系统的工作是将稀缺的人类注意力分配到少数情况，在这些情况中 (a) 收益是严重不对称的，AND (b) 操作者拥有不可复制的信息优势。**

其他一切都是开销。

---

## 2. 五大根本命题

这些是不可谈判的公理。所有下游版本中的评分、排名和推荐逻辑必须从这五个命题推导出来，或至少与之兼容。

### P1 — 不对称胜于预测

> *"10 倍回报从不被预测；它们在过程中被锻造。你能控制的是以 1 美元换 100 美元的风险。"*

- 框架优化的是**收益几何形状**，而非内在价值的点估计
- 目标：不对称收益比（APR）≥ 50:1（模拟下行损失 vs 可信（非最大）上行情景）
- "公平价值"是合理性检查，不是论题

### P2 — 优势胜于分析

> *"在 AI/量化可以复制你工作的任何方向，你已经处于劣势。只在他们无法做到的地方前进。"*

- 每个可操作的推荐必须标记一个*不可被通用 API + LLM 管道复制*的 **Alpha Source**
- 可接受的优势类型：初级专家电话、管理层访问、供应链联系人、前沿论文跟踪、实地运营信号
- 不可作为唯一优势：因子筛选、比率比较、卖方共识增量、公开新闻综合

### P3 — 便宜不等于安全

> *"如果资本支出制度转变使历史 FCF 失效，腾讯在下跌时变得更贵。"*

- 低倍数在资本配置制度改变时**默认可疑**
- 任何在 Capex 拐点期间依赖历史 FCF/EPS 的论题必须降级，直到新的再投资率 × ROIC 产生可信的远期 FCF
- 价值陷阱是"逻辑上做价值投资"的典型失败模式；必须建模，而非假设不存在

### P4 — 行业离散点火

> *"PCB 和光模块公司看起来死气沉沉多年，然后数十家在几个月内同时点亮。点火是离散事件，不是连续估值函数。"*

- 周期位置评分（当前 ADVANCED-MFG/CYCLICAL 模板）是必要但不充分的
- 框架必须单独检测**点火**——跨行业的协调、多信号、窄窗口状态变化
- 预点火监控胜过点火后估值。估值用于规模，而非发现（在点火情况下）

### P5 — 集中和耐心是资产

> *"深度跟踪 30 个名称就足够了。为了 10%/年的交换，等待 10 年等一个真正的机会，是我乐意接受的交易。"*

- 广度是负债；深度是资产
- 硬上限：随时在 Active Conviction Pool 中最多 ≤30 个名称。添加需要驱逐
- `WAIT` 是第一类推荐。闲置现金承载着系统必须定价的明确期权价值

---

## 3. 人机边界宪章

框架对自身分工的自我理解。具有约束力。

| 任务 | Owner | 理由 |
|------|-------|------|
| 公开申报、比率、倍数的大规模扫描 | **AI** | 速度、客观性、无疲劳 |
| 数千个 ticker 的异常检测 | **AI** | 带宽 |
| 模板化估值（Damodaran 式 DCF、EV/EBITDA 区间） | **AI** | 机械性 |
| 从公开数据检测*候选*点火信号 | **AI** | 模式识别 |
| **核实点火**（电话、渠道检查、专家对话） | **Human** | 不可复制 |
| 对管理层质量和资本配置意图的判断 | **Human** | 需要关系+背景 |
| 前沿论文/商业化前科技领先指标 | **Human**（AI 辅助） | 需要领域品味 |
| **最终买/持/退出决策** | **Human** | 问责+优势整合 |
| 仓位规模 | **Human**，AI 提供结构化备忘录 | 不可约简的判断性 |

**推论**：任何试图自动化此表中"Human"行的未来代理都是准则违规，必须被拒绝，除非 DOCTRINE-001 被正式修订。

---

## 4. 输出契约（完整 Schema）

所有未来版本必须为每个被监控的名称至少产生：

```yaml
# 基础架构（DOCTRINE-001 §3 逐字）
ticker: <symbol>
recommendation: [WAIT | TRACK | BUY-CANDIDATE | SIZE-UP | TRIM | EXIT]

asymmetric_payoff:
  modeled_downside_pct: <number>     # 必填，保守
  credible_upside_pct: <number>      # 必填，非最大情景
  apr: <upside / downside>           # 必须 ≥ 50 才能是 BUY-CANDIDATE

alpha_source:
  type: [PROBABILISTIC | INFORMATIONAL | CATALYST]
  edge_evidence: <一句话，具体>
  ai_replicable: <true | false>      # 若为 true，推荐降级为最高 TRACK

reinvestment_regime:
  capex_ttm_vs_5y_avg: <ratio>
  regime_shift_flag: <bool>          # 若为 true，历史 FCF 不能外推

ignition_state:
  signals_active: <list>
  ignition_alert: <bool>

verification_ledger:
  core_assumption: <string>
  falsification_checkpoints:
    - at: <T+3M | T+6M | T+12M>
      metric: <可观测指标>
      fail_threshold: <值>
  exit_trigger: <规则>

edge_score: <0-10>                   # 门控最大仓位
```

```yaml
# V5.4 扩展字段（Patch 2, 4, 5, 7, 8）
holding_thesis:
  ticker: <symbol>
  core_assumption: <string>
  falsification_checkpoints:
    - checkpoint_id: <string>
      at: <T+3M | T+6M | T+12M | T+18M | T+24M>
      due_date: <ISO-8601>
      metric: <string>
      fail_threshold: <string>
      status: <PENDING | PASSED | FAILED>
      actual_value: <string | null>
  exit_trigger: <string>

pool_event_log:
  - event_type: <ADD | EVICT | PROMOTE | DEMOTE>
    ticker: <symbol>
    timestamp: <ISO-8601>
    reason: <string>
    eviction_candidate: <symbol | null>
    replacement_rationale: <string | null>

advisory_status: <ADVISORY-ONLY | null>
charter_violation_warning: <string | null>
gate_violation: <bool | null>

regime_shift_redcf:
  forward_ebit: <float | null>
  tax_rate: <float | null>
  new_reinvestment_rate: <float | null>
  forward_roic: <float | null>
  wacc: <float | null>
  dcf_status: <REGIME-SHIFT-REDCF | INVALID-REGIME-SHIFT | NORMAL | null>
```

**关键规则**：没有填充的 `alpha_source` 和 `verification_ledger` 的名称**不能**获得高于 `TRACK` 的推荐。

---

## 5. "WAIT" 默认状态

系统在缺乏证据的情况下，默认状态是 `WAIT`。

- 在没有满足 `(APR ≥ 50) AND (edge_score ≥ 4)` 的情况下产生非 `WAIT` 推荐的版本是违规的
- 系统必须以与 `BUY-CANDIDATE` 决策同等的严格程度报告 `WAIT` 决策：扫描了什么，什么失败了，什么会改变答案
- 耐心溢价（等待每十年一次错位的现金期权性）要明确建模，而非隐含地假设

**WAIT Memo 结构**（当 `WAIT_DEFAULT = true` 时必须产生）：

```yaml
wait_memo:
  scan_date: <ISO-8601>
  universe_scanned: <int>
  active_pool_size: <int>
  gate_failures:
    apr_below_50_count: <int>
    edge_score_below_4_count: <int>
    both_gates_failed_count: <int>
  closest_to_gate:
    - ticker: <symbol>
      apr: <float>
      edge_score: <float>
      gap_to_apr_gate: <float>
      gap_to_edge_gate: <float>
      what_would_change: <string>
  patience_premium_note: >
    Cash is not idle; it holds an option on the next once-per-decade dislocation.
    This WAIT is a considered position, not an absence of work.
  next_scan_date: <ISO-8601>
```

---

## 6. 此准则存在以防止的失败模式

1. **概率陷阱** — 产生一长串"统计上有吸引力"的名称，但没有优势，然后看着它们表现得像 ETF
2. **价值陷阱** — 推荐下跌倍数名称，其 Capex 制度已使其历史现金生成失效
3. **逻辑陷阱** — 将结构良好的论点误认为可投资优势。逻辑是必要的；它不是回报来源
4. **广度陷阱** — 将注意力分散在数百个名称上，使得点火发生时没有任何名称被充分了解以果断行动
5. **自动化陷阱** — 构建代理来替代不可约简的人类优势（关系、品味、判断），从而摧毁唯一剩余的 Alpha 来源

---

## 7. 巴菲特/芒格思想精要

这些思想融入 VIAIOS-P 框架各处，以下为关键操作化原则：

### 巴菲特核心操作化

**护城河优先**：
- 护城河类型：品牌忠诚度、转换成本、网络效应、成本优势、规模经济、监管壁垒
- 护城河持久性：高 ROIC 持续 10 年以上是护城河的最强证明（见 [[05_BASE_RATES.md#roic_persistence]]）
- 护城河与定价权的关系：能否在通货膨胀下维持利润率是真正护城河的测试

**能力圈**：只分析真正理解的企业。对于"太难"的企业，明确输出 `PASS`，而非强行给出评级。

**安全边际（Margin of Safety）**：
- 最小安全边际：当前价格 vs 保守内在价值 ≥ 30%
- 安全边际 < 30%：必须输出 `PASS / 不投资`，无论其他指标多好
- 安全边际越大，持仓规模可以越大（但需要 edge_score 支撑）

**集中持仓**：Top 5 持仓可以占投资组合 50-70%（对非常高确信度的 ≥ edge_score 8.0 的持仓）

### 芒格心智模型操作化

**多学科思维**：任何分析都应激活至少 3 种不同角度（财务/竞争/心理/行业/宏观）

**反转**：问"这个投资如何让我亏光？"（即事前验尸，Pre-Mortem）

**激励分析**：分析管理层激励机制和股东利益一致性（Agent-2 observable_proxies）

**心理偏差防范**：
- 确认偏误：Reflexion Loop 强制自我批评
- 近期偏误：使用 5-10 年历史基础率（见 [[05_BASE_RATES.md]]）
- 过度自信：概率向 50% 方向收缩 10%

---

## 8. Damodaran 思想精要

### 核心估值哲学

- **价格 vs 价值**：定价（市场愿意支付什么）与估值（基本面应该支付什么）是两件事。两者都有必要理解
- **叙事与数字**：好的估值必须将叙事与数字紧密链接（Narrative Numbers Consistency Test）
- **估值的谦逊**：任何精确的估值都是虚假精度。正确是使用分布，而非点估计

### 7 大工具的设计逻辑

- **SOTP**：将复杂公司分解为可独立估值的叙事，防止跨业务线混淆资本密集度和利润率
- **Monte Carlo**：将估值不确定性可视化为分布，而非假装单一情景是真相
- **Expansion Option**：不要丢弃远期低概率高赔率叙事，但必须独立计量（≤15% 贡献）
- **Distress Dual-Track**：调高 WACC 是危险的单一工具——它同时影响继续经营价值和破产概率，混淆了两者
- **Dynamic ERP**：用市场当前定价的风险溢价（而非历史均值）——危机期间这一差距可达 5-6pp
- **Restated Financials**：研发是资产，不是费用。错误的会计处理导致系统性低估 ROIC
- **Cyclical Normalization**：峰值盈利 × 历史倍数 = 峰值估值 × 峰值倍数 = 双重错误

### WACC 计算标准（Damodaran 方法）

```
WACC = Ke × (E/V) + Kd × (1-t) × (D/V)
Ke = Rf + β × ERP + CRP + 特定风险溢价
  Rf: 当期 10Y 国债（美国/中国，对应 C-013）
  ERP: 当月隐含 ERP from implied_erp_history.yml（非历史均值）
  CRP: Sovereign CDS from sovereign_cds_snapshot.yml（海外营收 > 30% 时）
  特定风险溢价: 小公司溢价/流动性折扣（1-3%，需说明）
```

---

## 9. DOCTRINE-001 修正协议

此准则只能通过以下方式修订：

1. 一个新文件 `doctrine/DOCTRINE-001-amend-NNN.md`，明确命名被修改的条款
2. 书面理由，引用 (a) ≥3 年实证追踪记录证据，或 (b) AI/量化格局的结构性变化，使某个公理失效
3. 代码、提示词或下游版本规范中不得有静默覆盖

V5.4 及以后版本必须在其规范顶部包含：

> *"Compliant with DOCTRINE-001-Edge-over-Logic."*

---

## 10. 一句话总结

> **停止试图成为更聪明的分析师。开始成为房间里唯一了解特定情况的人，在少数情况中，正确的回报是 50 倍，错误的成本是 1 倍。**

这个存储库中的一切——每个模板、每个代理、每个评分——都是为了服务这句话。

---

*End of 03_DOCTRINE_PRINCIPLES.md — VIAIOS-P V5.4-Gem*
