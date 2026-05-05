---
version: 5.4-Gem
role: Edge Registry 优势注册表全集（Schema、衰减公式、仓位门控、Alpha Source 分类）
load_priority: 4
---

# 04 EDGE REGISTRY — VIAIOS-P V5.4 Information Edge System

> Cross-ref: [[01_CORE_PROTOCOL.md#patch-1]], [[01_CORE_PROTOCOL.md#patch-6]], [[03_DOCTRINE_PRINCIPLES.md#p2]]

---

## 目录 (TOC)

1. [Edge Registry 设计哲学](#1-edge-registry-设计哲学)
2. [文件命名与目录结构](#2-文件命名与目录结构)
3. [完整 YAML Schema](#3-完整-yaml-schema)
4. [允许的渠道类型（Channel Types）](#4-允许的渠道类型channel-types)
5. [Edge Score 衰减公式](#5-edge-score-衰减公式)
6. [Edge Score 计算方法](#6-edge-score-计算方法)
7. [仓位层级门控（Position Tier Gate）](#7-仓位层级门控position-tier-gate)
8. [Alpha Source 标注规范（Patch 1）](#8-alpha-source-标注规范patch-1)
9. [有效 vs 无效 Edge Evidence 示例](#9-有效-vs-无效-edge-evidence-示例)
10. [Edge Registry 操作规程](#10-edge-registry-操作规程)

---

## 1. Edge Registry 设计哲学

**来自 DOCTRINE-001 P2**：
> *"你在 AI/量化可以复制你工作的任何方向已经处于劣势。只在他们无法做到的地方前进。"*

Edge Registry 是将 P2 原则转化为可执行的静态资产注册表：

- 追踪操作者为每个标的/行业维护的**具体、非公开信息渠道**
- 通过时间衰减公式强制要求渠道保持活跃
- 通过 edge_score 门控将仓位规模与真实优势强度绑定
- 防止**分析表象替代真实优势**（即防止仅因估值便宜就进入重仓）

---

## 2. 文件命名与目录结构

```
edge_registry/
  README.md                    # Schema 文档（参考本文件）
  {TICKER}.yaml                # 单个 ticker（如 NVDA.yaml, 0700.HK.yaml）
  {INDUSTRY_ID}.yaml           # 行业级优势（如 optical_module.yaml, pcb_tier1.yaml）
```

**命名约定**：
- 单 ticker：`{EXCHANGE}:{TICKER}.yaml`（推荐格式，如 `HKG:0700.yaml`）
- 行业：`{INDUSTRY_ID}.yaml`（使用小写，下划线分隔）

**重要**：此目录由**人工操作者**维护；不得由任何 AI 代理自动填充。AI 代理只能**读取**，不能**写入**。

---

## 3. 完整 YAML Schema

```yaml
# edge_registry/{TICKER}.yaml

ticker: <symbol>                    # 交易所:ticker 格式（如 HKG:0700）
last_registry_update: <ISO-8601>    # 此文件最后一次人工更新的日期

edge_channels:
  - channel_id: EC-001              # 在此文件内顺序唯一
    type: <channel_type>            # 见下方允许值
    description: <string>           # 一句话。不命名具体联系人。
                                    # 示例："基板材料的二级供应商；每季度提供
                                    #        交货期和订单量信息。"
    last_contact: <ISO-8601>        # 此渠道最近一次信号的日期
    freshness_decay_days: <int>     # 贡献衰减至零的天数
                                    # 除非有理由覆盖，否则使用下方默认值
    current_contribution: <float>   # 计算值：base_contribution × decay_factor（0.0–1.0）
                                    # 每次更新时重新计算

  - channel_id: EC-002
    # ...（更多渠道）

edge_score: <float>                 # 0.0–10.0。从 current_contribution 值计算
                                    # 硬门控：edge_score < 4.0 → 仓位权重 ≤ 5%
                                    # （重仓层级被阻止）

edge_score_floor_for_heavy_position: 4.0   # 硬常量；不得在没有 patch 说明的情况下修改
```

---

## 4. 允许的渠道类型（Channel Types）

| `type` 值 | 含义 | 基础贡献 | 默认衰减天数 |
|---|---|---|---|
| `management_access` | 直接访问 C-suite 或 IR | 1.0 | 90 |
| `supply_chain_contact` | 供应商、客户或分销商渠道 | 0.9 | 60 |
| `industry_expert_call` | 专家网络电话或独立行业专家 | 0.8 | 90 |
| `channel_check` | 经销商、零售商或终端客户调查 | 0.7 | 45 |
| `regulatory_contact` | 监管机构、标准机构或政府内部人士 | 0.7 | 120 |
| `paper_tracking` | 学术或商业化前 R&D 论文监控 | 0.5 | 180 |
| `other` | 不适合上述任何类别；需要描述 | 0.3 | 60 |

**渠道类型选择指南**：
- 信息越接近一手、越非公开 → 基础贡献越高
- 信息越容易通过公开资料复制 → 应归类为 `PROBABILISTIC` Alpha Source（较弱）
- `paper_tracking` 虽然是公开的，但领域品味和早期追踪是人类独有能力

---

## 5. Edge Score 衰减公式

```
days_since_contact = (today - last_contact).days
decay_factor = max(0.0, 1.0 - (days_since_contact / freshness_decay_days))
current_contribution = base_contribution × decay_factor
```

**数值示例**：

| 渠道类型 | last_contact | freshness_decay_days | decay_factor | current_contribution |
|---------|-------------|---------------------|-------------|---------------------|
| supply_chain_contact | 30天前 | 60 | 1.0 - 30/60 = 0.5 | 0.9 × 0.5 = **0.45** |
| management_access | 45天前 | 90 | 1.0 - 45/90 = 0.5 | 1.0 × 0.5 = **0.50** |
| paper_tracking | 90天前 | 180 | 1.0 - 90/180 = 0.5 | 0.5 × 0.5 = **0.25** |
| industry_expert_call | 100天前 | 90 | max(0, 1.0 - 100/90) = **0.0** | 0.8 × 0.0 = **0.00** |
| channel_check | 50天前 | 45 | max(0, 1.0 - 50/45) = **0.0** | 0.7 × 0.0 = **0.00** |

**衰减完成后**：`current_contribution = 0.0`。这意味着超期渠道对 edge_score **没有任何贡献**。

---

## 6. Edge Score 计算方法

```
raw_score  = sum(current_contribution for all channels)
edge_score = min(10.0, raw_score × normalization_factor)
```

**归一化因子（normalization_factor）**：
- 参考默认值：`3.0`
- 含义：具有 4 个多样化、完全新鲜渠道的注册表（raw_score ≈ 3.4）映射到大约 8-9/10 的 edge_score
- 精确校准因子在 V5.5 的实施层延迟；V5.4 规范使用 `3.0` 作为参考默认值

**典型 edge_score 范围示例**：

| 情景 | 描述 | 大约 edge_score |
|------|------|---------------|
| 没有注册表文件 | 默认 | 0.0（上限 TRACK） |
| 1 个过期渠道 | current_contribution = 0.0 | 0.0（上限 TRACK） |
| 1 个新鲜 paper_tracking | raw_score = 0.5 → × 3.0 | 1.5（上限 Standard） |
| 1 个新鲜 supply_chain_contact | raw_score = 0.9 → × 3.0 | 2.7（上限 Standard） |
| 2 个新鲜渠道（supply + expert） | raw_score = 1.7 → × 3.0 | 5.1（允许 Heavy） |
| 4 个新鲜多样化渠道 | raw_score ≈ 3.4 → × 3.0 | 约 8-9（允许 Heavy） |

---

## 7. 仓位层级门控（Position Tier Gate）

| 仓位层级 | 最大组合权重 | 最低 edge_score 要求 |
|---------|------------|---------------------|
| **Heavy（重仓）** | > 5% | **≥ 4.0**（硬门控） |
| Standard（标准） | 2%–5% | ≥ 2.0 |
| Tracking（跟踪） | < 2% | ≥ 0.0（无门控） |

**执行规则**：
- Agent-7 在发出任何 `SIZE-UP` 推荐之前，必须读取相关注册表文件中的 `edge_score`
- 没有注册表文件默认 `edge_score = 0.0` → 上限 `TRACK`，阻止 `SIZE-UP`
- `edge_score < 4.0` 时，即使 APR 非常高，也不得进入重仓层级

**相关 V5.4 验收测试**：
- `SIZE-UP` 绝不会出现在 `edge_score < 4.0` 的名称
- 只有 `paper_tracking` 渠道（衰减 180 天），90 天前最后联系：`current_contribution = 0.5 × (1 − 90/180) = 0.25`，产生降低的 `edge_score`
- `last_contact` 设置为超过 `freshness_decay_days` 之前的日期，必须为该渠道设置 `current_contribution = 0.0`

---

## 8. Alpha Source 标注规范（Patch 1）

每个输出记录必须携带 `alpha_source` 块：

```yaml
alpha_source:
  type: <PROBABILISTIC | INFORMATIONAL | CATALYST>
  edge_evidence: <string>       # 一句具体话。必须命名特定的非公开信号来源。
  ai_replicable: <bool>         # true = 通用 LLM + API 管道可产生相同信号
```

**类型定义与门控逻辑**：

| Alpha Source 类型 | 典型 ai_replicable | 推荐上限 | 说明 |
|---|---|---|---|
| `PROBABILISTIC` | 通常 `true` | TRACK | 基础率/结构洞察。最弱类型 |
| `INFORMATIONAL` | 必须 `false` | BUY-CANDIDATE / SIZE-UP | 特定不可复制信息源 |
| `CATALYST` | 可能 `false` | BUY-CANDIDATE（若解读非显而易见） | 已知但未定价的即将到来的事件 |

**硬上限规则**：若 `ai_replicable: true`，系统必须将 `recommendation` 降级为最高 `TRACK`。此上限是单向棘轮：任何下游模块都不得覆盖它。

---

## 9. 有效 vs 无效 Edge Evidence 示例

### ✅ 有效（Acceptable）Edge Evidence

```
"基板材料的二级供应商联系人确认 Q3 订单重启。"
"直接访问 CFO 时，CFO 表示当前产能利用率为 83%，与公开电话的'约 80%'不一致。"
"在 Semicon Japan 2025 展会上，3 家不同组件供应商独立提到交货期延长至 18 周。"
"预印本论文（arXiv:2502.xxxxx）显示固态电池关键瓶颈有了解决方案，会议前 30 天。"
"独立经销商调查（N=15）显示终端需求提前复苏——早于任何卖方预测。"
```

### ❌ 无效（NOT Acceptable）作为唯一 Edge Evidence

```
"估值屏幕显示 EV/EBITDA 低于 5 年平均值。"  # 纯公开数据
"公司低于行业平均 P/E。"  # 纯因子筛选
"卖方共识已下调 15%，可能存在超卖。"  # 公开共识增量
"管理层在最近的电话中表示信心十足。"  # 公开信息综合
"新闻报道显示该行业有强劲需求。"  # 公开新闻
"历史上低 P/B 后产生高回报。"  # 因子回测
```

**关键区别**：`edge_evidence` 必须命名**特定**、**非公开**的信号来源。通用公开数据分析不是优势。

---

## 10. Edge Registry 操作规程

### 10.1 创建新注册表文件

1. 确定 ticker/行业在 Active Conviction Pool 中（不超过 30 个名称）
2. 在 `edge_registry/` 下创建 `{TICKER}.yaml` 文件
3. 列出所有当前活跃的信息渠道，每个渠道填写 `channel_id`、`type`、`description`（无具体姓名）、`last_contact`
4. 计算 `current_contribution`（使用衰减公式）
5. 计算 `edge_score`
6. 验证：若 `edge_score < 4.0`，则当前仓位层级不能是 Heavy

### 10.2 更新注册表文件

- 每次与渠道有新的接触/信号后：更新 `last_contact`，重新计算 `current_contribution` 和 `edge_score`
- 每季度审查：检查是否有渠道已衰减至零
- 驱逐评估时：`edge_score` 是池驱逐优先级的第一排序键

### 10.3 禁止操作

- **不记录**联系人姓名或联系方式。文件仅存储渠道类型、描述和时间元数据
- **不记录**交易决策、价格目标或推荐。这是推荐引擎的输入，而非输出
- **不创建**含真实 ticker 数据的示例文件。Schema 文档本身已足够用于实施；真实条目由人工操作者创建
- **不允许**任何 AI 代理写入此目录

### 10.4 没有注册表文件时的默认行为

```yaml
# 若 edge_registry/{TICKER}.yaml 不存在：
edge_score: 0.0         # 默认
recommendation_cap: TRACK  # 推荐上限
heavy_position_allowed: false
size_up_allowed: false
```

---

*End of 04_EDGE_REGISTRY.md — VIAIOS-P V5.4-Gem*
