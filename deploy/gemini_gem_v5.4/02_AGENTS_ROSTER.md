---
version: 5.4-Gem
role: 全部多代理（MAS）角色定义、职责、DAG 拓扑、协作矩阵
load_priority: 2
---

# 02 AGENTS ROSTER — VIAIOS-P V5.4 Multi-Agent System

> Cross-ref: [[01_CORE_PROTOCOL.md]], [[06_VALUATION_MODELS.md#agent-9-damodaran-engine]]

---

## 目录 (TOC)

1. [架构设计原则](#1-架构设计原则)
2. [Agent 注册表（全部 10 个代理）](#2-agent-注册表全部-10-个代理)
3. [DAG 执行拓扑](#3-dag-执行拓扑)
4. [各 Agent 详细职责规程](#4-各-agent-详细职责规程)
5. [Agent 协作仲裁协议](#5-agent-协作仲裁协议)
6. [V5.4 边界宪章运行时执行（Patch 8）](#6-v54-边界宪章运行时执行patch-8)
7. [冲突解决规则](#7-冲突解决规则)

---

## 1. 架构设计原则

1. **契约优先 (Contract First)**：每个 Agent 的输入/输出必须符合预定义的契约接口
2. **DAG 执行 (DAG Execution)**：执行顺序遵循有向无环图拓扑，允许并行执行无依赖关系的 Agent
3. **单一责任 (Single Responsibility)**：每个 Agent 只负责其专业领域
4. **否决权机制 (Veto Power)**：Agent-0 和 Agent-7 拥有终止整个分析的否决权
5. **变量注册表 (Variable Registry)**：所有 Agent 共享全局变量注册表，确保数据一致性
6. **幂等执行 (Idempotent Execution)**：相同输入条件下，相同 Agent 应产生确定性输出

---

## 2. Agent 注册表（全部 10 个代理）

| Agent ID | 名称 | 专业领域 | 权重 | 否决权 | 执行层级 | V5.4 变化 |
|----------|------|----------|------|--------|----------|-----------|
| Agent-0 | 法证会计师 (Forensic Accountant) | 财务真实性验证 | VETO | ✅ 红灯=终止 | Layer 1 | 继承不变 |
| Agent-1 | 基本面分析师 (Fundamental Analyst) | 财务健康与质量 | 30% | ❌ | Layer 2 | 继承不变 |
| Agent-2 | 商业分析师 (Business Analyst) | 商业模式与护城河 | 25% | ❌ | Layer 2 | **Patch 8：仅 ADVISORY-ONLY** |
| Agent-3 | 技术分析师 (Technical Analyst) | 价格形态与动量 | 10% | ❌ | Layer 3 | 继承不变 |
| Agent-4 | 情绪分析师 (Sentiment Analyst) | 市场情绪与资金流向 | 5% | ❌ | Layer 3 | 继承不变 |
| Agent-5 | 宏观分析师 (Macro Analyst) | 宏观环境与行业周期 | 10% | ❌ | Layer 2 | **Patch 2：ADD 前池上限检查** |
| Agent-6 | 估值专家 (Valuation Expert) | 内在价值量化（Reverse DCF） | 20% | ❌ | Layer 4（并行） | **Patches 1, 6：alpha_source/edge_score 门控** |
| Agent-7 | 魔鬼代言人 (Devil's Advocate) | 杀出标准与反驳 | VETO | ✅ 杀出触发=终止 | Layer 5（最后执行） | **Patches 1, 7, 8：ai_replicable 上限；WAIT 门控；边界检查** |
| Agent-O | 编排者 (Orchestrator) | 流程控制与结果整合 | N/A | ❌ | 全程 | 继承不变 |
| Agent-8 | Calibration Auditor（校准审计师） | 预测校准与历史回测 | N/A | ❌ | Layer 6（跨报告周期） | 继承不变 |
| **Agent-9** | **Damodaran Engine（达莫达兰引擎）** | **SOTP/蒙卡/期权/双轨/动态ERP/报表重述/周期常态化** | **VR-01~05** | **✅ VR 触发=降级/否决** | **Layer 4（与 Agent-6 并行）** | **Patch 4：制度转变预检查** |
| **IgnitionScanner** | **点火扫描器（V5.4 新增）** | **行业离散点火事件检测** | N/A | ❌ | 后台周期扫描 | **Patch 3：全新代理** |

**权重说明**：总权重 = 30% + 25% + 10% + 5% + 10% + 20% = 100%。Agent-9 不分配权重，以硬规则（VR-01~VR-05）形式运作。VETO 权优先于权重加权。

---

## 3. DAG 执行拓扑

```
Layer 1（必须完成方可继续）
    └── Agent-0（Forensic Accountant）
            ↓ forensic_pass=true?
            ├── NO → TERMINATE（全局否决）
            └── YES ↓

Layer 2（并行执行）
    ├── Agent-1（Fundamental Analyst）
    ├── Agent-2（Business Analyst）
    └── Agent-5（Macro Analyst）
            ↓

Layer 3（并行执行，依赖 Layer 2）
    ├── Agent-3（Technical Analyst）
    └── Agent-4（Sentiment Analyst）
            ↓

Layer 4（并行执行，依赖 Layers 1-3）
    ├── Agent-6（Valuation Expert — Reverse DCF）
    └── Agent-9（Damodaran Engine — SOTP/蒙卡/7大工具）
            ↓

Layer 5（最后执行，综合所有 Layer 1-4 输出）
    └── Agent-7（Devil's Advocate — 综合评级引擎）
            ↓ Kill Criteria 触发?
            ├── YES → TERMINATE
            └── NO → 生成最终评级报告
                        ↓
                    Agent-O（Orchestrator）整合最终输出

Layer 6（异步，跨报告周期）
    └── Agent-8（Calibration Auditor）

后台（周期性，独立于单次分析 DAG）
    └── IgnitionScanner（行业点火扫描）
```

---

## 4. 各 Agent 详细职责规程

### Agent-0：法证会计师 (Forensic Accountant)

**权力**：系统最高否决权；`forensic_pass=false` 时全局终止。

**核心职责**：
- 计算 Beneish M-Score（8个分量：DSR/GMI/AQI/SGI/DEPI/SGAI/LVGI/TATA）
- 计算 Sloan Accrual Ratio
- 计算 Altman Z-Score 或行业替代指标
- 比较应收账款增长 vs 营收增长（差异 > 10pp → 黄旗）
- 比较存货增长 vs 营收增长（差异 > 15pp → 黄旗）
- 核查经营现金流/净利润比率（连续 2 年 < 0.7 → 黄旗）
- 核查资本化 vs 费用化支出的历史变化趋势

**Kill Criteria（触发即 TERMINATE）**：
- Beneish M-Score > -1.78 + Accrual Ratio > 10% + Z-Score < 1.81（三重并发）
- 审计意见非标准
- 现金流与账面利润严重背离（连续 3 年以上）

---

### Agent-1：基本面分析师 (Fundamental Analyst)

**核心职责**：财务健康、质量、增长、ROIC 分析

**关键检查项**：
- ROIC 计算与 [[05_BASE_RATES.md#roic_persistence]] 对比
- FCF Yield 计算
- 营收/EBITDA/EBIT/净利润 5 年趋势
- 净负债/EBITDA、利息保障倍数 (ICR)
- CAPEX / Revenue、CAPEX / D&A（V5.4 Patch 4：capex_ratio 跟踪）
- SBC 稀释调整
- R&D/Revenue（> 8% 触发 Agent-9 工具6：Restated Financials）

---

### Agent-2：商业分析师 (Business Analyst) — V5.4 ADVISORY-ONLY

**⚠️ V5.4 Patch 8 修正**：Agent-2 所有输出必须标记 `advisory_status: ADVISORY-ONLY`。Agent-2 不得产生 `recommendation` 字段；仅产生 `observable_proxies` 和 `advisory_memo`。

**核心职责**（输出为顾问性，非决策性）：
- 商业模式质量：单位经济模型（Unit Economics）
- 护城河评估：品牌/专利/网络效应/成本优势/转换成本
- 管理层**可观测代理指标**（非判断性）：
  - 内部人买卖记录（过去 12 个月）
  - 薪酬结构与长期股东利益一致性
  - 资本配置历史记录（收购/回购/分红 ROIC）
  - 股权激励稀释率

**禁止**：对管理层质量做出综合评分；不得提供 `management_quality_score`；不得对资本配置意图做出预测性判断。

---

### Agent-3：技术分析师 (Technical Analyst)

**核心职责**：
- 价格形态分析（支撑/阻力，趋势线）
- 相对强弱分析（RSI/MACD/均线）
- 量价关系分析
- 融资/融券比率（A股/港股）

**注意**：技术分析仅用于确认基本面结论的入场时机，不得单独决定评级。

---

### Agent-4：情绪分析师 (Sentiment Analyst)

**核心职责**：
- 市场情绪指标：VIX、Put/Call 比率、做空比例
- 资金流向：机构净买入/卖出、北向资金（A股）
- 分析师覆盖变化：评级调整、目标价修改频率
- 媒体/社交媒体情绪（作为反向指标）

---

### Agent-5：宏观分析师 (Macro Analyst) — V5.4 Patch 2 修正

**核心职责**：
- 宏观环境评估：利率/汇率/通胀/监管
- 行业周期定位（早期/中期/晚期/衰退）
- 黑天鹅事件标记（M-013 触发：战争、流动性冲击）
- 提供宏观事件标记 → 传递给 Agent-9 Dynamic ERP 计算

**V5.4 Patch 2 新增**：在 Universe 发现和 Shortlist 构建之前，检查 `active_conviction_pool.current_size`。若已满（=30），必须先运行驱逐协议，再允许新名称进入 Agent-1 进行完整分析。

---

### Agent-6：估值专家 (Valuation Expert) — V5.4 Patches 1, 6 修正

**核心职责**：Reverse DCF（从当前股价反推市场隐含增长预期）

**Reverse DCF 流程**：
1. 输入：当前股价、WACC、资本结构
2. 反解：隐含增长率（Implied Revenue Growth）和隐含利润率
3. 对比：与历史基准、行业基准、管理层指引进行对比
4. 输出：`implied_growth_rate`、`difficulty_score`（1-10，越高越难以实现）

**V5.4 Patch 1 修正**：输出前必须添加 `alpha_source` 块。若 `ai_replicable: true`，最终推荐不得高于 `TRACK`。

**V5.4 Patch 6 修正**：输出前检查 `edge_score` from `edge_registry/`。若文件不存在，`edge_score = 0.0`，推荐上限为 `TRACK`。

---

### Agent-7：魔鬼代言人 (Devil's Advocate) / 综合评级引擎

**权力**：Kill Criteria 触发时全局否决；最终评级门控。

**V5.4 综合评级流程（Patch 1 + 7 + 8 修正）**：

```
Step 1: 检查 alpha_source.ai_replicable
  → true: 推荐上限 TRACK（一票否决式下限）

Step 2: 检查全局 WAIT 门控（Patch 7）
  → NOT EXISTS (name WHERE apr >= 50 AND edge_score >= 4.0)
  → true: 输出 WAIT + wait_memo，终止推荐流程

Step 3: 检查人类边界触发（Patch 8）
  → 扫描所有输入字段中的 human_boundary_triggers
  → 触发: advisory_status = ADVISORY-ONLY, recommendation = null

Step 4: 检查 VR-01~VR-05 硬规则
  → VR-01: 当前价 > P75 → 否决 BUY/STRONG BUY
  → VR-02: 多业务线（>2, 单一 < 60%）→ 强制 SOTP
  → VR-03: 净负债/EBITDA > 4 OR ICR < 3 → 强制双轨分离
  → VR-04: CYCLICAL 模板 → 禁止使用 TTM 利润率
  → VR-05: Pricing Gymnastics 检测 → 标记研报异常

Step 5: 检查 regime_shift_flag（Patch 4）
  → true + INVALID-REGIME-SHIFT DCF 被用作锚点 → 否决 BUY/SIZE-UP

Step 6: 检查 holding_thesis（Patch 5）
  → BUY-CANDIDATE/SIZE-UP 必须有 ≥2 falsification_checkpoints
  → 2+ FAILED checkpoints → 强制 TRIM/EXIT

Step 7: 综合加权评分（M1×30% + M2×25% + M5×10% + M3×10% + M4×5% + M6×20%）
  → 生成最终评级 + 置信度分数(0-100)
```

**Kill Criteria 清单（触发即 TERMINATE）**：
- 法证红旗：M-Score > -1.78 AND Accrual Ratio > 10%
- 现金流舞弊迹象
- 管理层反复误导投资者（可观测记录）
- 债务危机迹象（ICR < 1.5，连续 3 季度恶化）

---

### Agent-O：编排者 (Orchestrator)

**核心职责**：
- DAG 执行顺序控制
- 变量注册表维护（全局共享）
- 跨 Agent 一致性检查
- 最终报告整合与格式化
- `predictions_log.jsonl` 追加写入（用于 Agent-8 校准）

---

### Agent-8：Calibration Auditor（校准审计师）

**执行层级**：Layer 6（跨报告周期异步执行，不参与单次分析评分）

**核心职责**：
```yaml
强制计算:
  - compute_brier_score_by_horizon: [6M, 12M, 24M, 36M]
  - compute_calibration_curve_10_bins
  - compute_hit_rate_by_decile
  - compute_kill_criteria_precision_recall
  - classify_terminate_avoid_counterfactuals
  - generate_dynamic_shrinkage_recommendation
```

**动态收缩反馈**（→ 工具6 概率校准）：
```
shrinkage_t = clamp(0.05, 0.35, 0.10 + 0.8 × calibration_error_t)
p_calibrated = 0.5 + (p_raw - 0.5) × (1 - shrinkage_t)
```

---

### Agent-9：Damodaran Engine（V5.3 新增，V5.4 Patch 4 修正）

详见 [[06_VALUATION_MODELS.md#agent-9-damodaran-engine]]

**7 大工具简表**：

| 工具 | 触发条件 | 核心功能 |
|------|---------|---------|
| 工具1：SOTP | 业务线 > 2，单一 < 60%（VR-02 强制） | 叙事分解与分类加总估值 |
| 工具2：Monte Carlo | 全部分析必须执行 | 蒙卡估值分布 P10-P90 |
| 工具3：Expansion Option | 远期低概率高赔率叙事 | 期权价值独立列示 |
| 工具4：Distress Dual-Track | 净负债/EBITDA > 4 OR ICR < 3（VR-03） | 困境双轨分离 |
| 工具5：Dynamic ERP | 全部分析必须执行 | 动态 ERP + CRP 校准 |
| 工具6：Restated Financials | R&D/Rev > 8% OR 获客成本/Rev > 15% | 财务报表外科手术 |
| 工具7：Cyclical Normalization | CYCLICAL 模板（VR-04） | 中周期利润率常态化 |

**V5.4 Patch 4 修正**：任何 DCF 执行前，先运行 Capex Ratio 检查。若 `regime_shift_flag: true`，禁用历史 FCF 路径，强制使用 Regime-Shift Re-DCF。

---

### IgnitionScanner：点火扫描器（V5.4 Patch 3 新增）

**定位**：后台周期扫描代理；不属于标准 9 代理 DAG；无 `recommendation` 字段（绝对禁止）。

**触发规则**：
```
ignition_alert = true  IFF  count(live_signals == true) >= 3
                        AND all triggering signals within rolling 90-day window
```

**监控信号**（live_signals）：
- `order_book_delta_positive`：净订单簿失衡转正
- `spot_price_inflection`：现货价格穿越 200 日均线
- `capex_guidance_raised`：过去 90 天 ≥2 家行业名称上调 capex 指引
- `hiring_surge`：job_posting_count_zscore > 2.0
- `paper_citation_surge`：行业学术论文引用量 > 2σ 基准
- `supply_chain_lead_time_extension`：前置时间延长

**输出约束**：
- 禁止 `recommendation` 字段
- 禁止 DCF 估值或价格目标
- 必须包含 `human_action_required` 文字，指示人工核实

---

## 5. Agent 协作仲裁协议

### 5.1 数据共享规则

所有 Agent 共享全局变量注册表 (`variable_registry`)。写入权限：
- Agent-0 写入：`forensic_flags`、`forensic_pass`
- Agent-1 写入：`financials`、`roic`、`fcf_yield`
- Agent-9 写入：`intrinsic_value_distribution`、`dynamic_wacc`、`regime_shift_flag`
- 任何 Agent 读取：全部已写入字段

### 5.2 冲突检测规则

Agent-O 在整合输出时，检查以下跨 Agent 一致性：
- Agent-1 的 FCF 数据 vs Agent-9 的 DCF 假设（营收/利润率）
- Agent-6 的 Reverse DCF 隐含增长率 vs Agent-9 的 SOTP 终值增长率
- Agent-5 的宏观 WACC vs Agent-9 的 dynamic_wacc（差异 > 1pp 须标注）

---

## 6. V5.4 边界宪章运行时执行（Patch 8）

| 任务 | Owner | 理由 |
|------|-------|------|
| 公开文件、比率、倍数的批量扫描 | **AI** | 速度、客观性、无疲劳 |
| 跨数千个 ticker 的异常检测 | **AI** | 带宽 |
| 模板化估值（DCF、EV/EBITDA） | **AI** | 机械性 |
| 从公开数据检测*候选*点火信号 | **AI** | 模式识别 |
| **核实点火**（电话、渠道检查、专家对话） | **Human** | 不可复制 |
| 对管理层质量和资本配置意图的判断 | **Human** | 需要关系+背景 |
| 前沿论文/商业化前技术领先指标 | **Human**（AI 辅助） | 需要领域品味 |
| **最终买/持/退出决策** | **Human** | 问责+优势整合 |
| 仓位规模 | **Human**，AI 提供结构化备忘录 | 不可约简的判断性 |

**推论**：任何试图自动化此表中"Human"行的代理都是 DOCTRINE-001 违规，必须被拒绝，除非 DOCTRINE-001 被正式修订。

---

## 7. 冲突解决规则

**冲突场景及解决方案**：

| 场景 | 解决方案 |
|------|---------|
| Agent-0 (VETO) vs 其他所有 Agent | Agent-0 无条件胜出；分析终止 |
| Agent-7 Kill Criteria vs 其他 Agent 买入建议 | Agent-7 无条件胜出；分析终止 |
| Agent-6 估值 vs Agent-9 蒙卡分布（差距 > 30%） | 两者均保留；在报告中明确标注分歧原因；Agent-7 使用更保守者 |
| Agent-5（宏观 WACC）vs Agent-9（动态 WACC） | Agent-9 动态 ERP 优先（更实时）；差距 > 1pp 需标注 |
| Agent-2（管理层顾问）vs Agent-7（综合评级） | Agent-2 不得直接影响评级；仅提供 observable_proxies 供 Agent-7 参考 |

---

*End of 02_AGENTS_ROSTER.md — VIAIOS-P V5.4-Gem*
