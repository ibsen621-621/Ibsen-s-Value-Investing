## V5.3-Damodaran-Calibrated (2026-04-28)

### 新增文件
- `VIAIOS-P-V5.3-Damodaran-Calibrated.md` — 主文档（基于 V5.2 全量升级，可直接作为 Gemini Gem 系统提示词）
- `agents/agent_9_damodaran_engine.md` — Agent-9 达莫达兰引擎（7大工具执行规程）
- `base_rates/bond_implied_default_rates.yml` — 债券隐含违约率（按评级/行业/久期）
- `base_rates/implied_erp_history.yml` — 月度隐含股权风险溢价历史
- `base_rates/sales_to_capital_by_industry.yml` — 行业销售资本比基准
- `base_rates/cyclical_midcycle_margins.yml` — 周期行业中周期常态利润率
- `base_rates/sovereign_cds_snapshot.yml` — 主要国家主权CDS利差快照
- `deploy/gemini_gems_v5.3.md` — Gemini Gems 部署清单（完整集/最小集两种方案）
- `tests/v5_3_damodaran_cases.md` — V5.3 红队测试（SpaceX / LVS / Exxon 三场景）

### 新增模块（主文档内）
- **§0 SYSTEM IDENTITY** — 版本标识更新至 V5.3 Damodaran-Calibrated
- **§3 INTENT ROUTER** — 新增 `MODE_HARD_TO_VALUE`（独角兽/困境/战时/早期颠覆者）
- **§5.2.8 CYCLICAL 强制指令** — VR-04：禁止 TTM 利润率做 DCF，强制执行 mid-cycle 常态化
- **§8.4 Restated Financials** — R&D/获客成本资本化，双口径 ROIC 强制披露
- **§10.5 Narrative Decomposition** — SOTP 强制路由（多业务线公司），VR-02
- **§11.5 Monte Carlo Valuation Distribution** — 取代三点情景，P10-P90 分布，VR-01
- **§11.6 Expansion Option Module** — 远期低概率高赔率叙事独立计量
- **§11.7 Pricing Gymnastics Detector** — 卖方研报三类定价体操检测，VR-05
- **§12.3 Distress-Adjusted Valuation** — 双轨期望值模型，P_distress 从债市反推，VR-03
- **§14.3 Dynamic Risk Calibration Engine** — Implied ERP + 主权 CDS + 黑天鹅触发机制
- **§15.1 一票否决规则集中归档** — VR-01～VR-05 + V5.2 已有否决项完整档案
- **§Gemini Gems 部署指引** — 两种部署方案（完整集9附件/最小集4附件），总附件≤10

### 新增硬规则
- **VR-01**：当前股价 > P75（蒙卡估值分布）→ 一票否决 BUY/STRONG BUY
- **VR-02**：多业务线公司未执行 SOTP → 估值结论 INVALID
- **VR-03**：高负债公司（净负债/EBITDA > 4 或 ICR < 3）未执行双轨分离 → INVALID
- **VR-04**：CYCLICAL 行业用 TTM 利润率做 DCF → INVALID，强制常态化
- **VR-05**：相对估值对标池未披露筛选标准 → 标记 "Pricing Gymnastics"，不得作为唯一依据

### 升级 Agent 架构
- **Agent-9 Damodaran Engine**（新增）：隶属 Layer 4（与 Agent-6 并行）
  - 工具1：SOTP Narrative Decomposition
  - 工具2：Monte Carlo Valuation Distribution
  - 工具3：Expansion Option Module
  - 工具4：Distress-Adjusted Dual-Track
  - 工具5：Dynamic ERP & CRP Calibration
  - 工具6：Restated Financials
  - 工具7：Cyclical Mid-Cycle Normalization
- **Agent-9 → Agent-8 闭环**：Agent-9 输出分布 → Agent-8 后验校准

### 不变项（V5.2 完整保留）
- V5.2 全部模块、硬规则、Agent 架构均完整保留
- 所有 base_rates/ V5.2 YAML 文件（6个）保持不变
- agents/agent_8_calibration_auditor.md 保持不变
- 所有 V4.x 和 V5.0 文档不修改

# CHANGELOG

## V5.2-Calibrated (2026-04-21)

- 新增 `VIAIOS-P-V5.2-Calibrated.md`（保留 V5.1 内容并扩展）
- 新增 Calibration Tracking Layer 与 Agent-8（Layer 6）
- 强制每份报告输出 `prediction_record`，并定义 `predictions_log.jsonl` 协议
- 增加校准指标：Brier Score / Calibration Curve / Hit Rate by Decile
- 将概率校准由固定 10% 收缩升级为动态收缩公式
- 扩展 Agent-0 法证二阶检验（M-Score 分量一致性、审计师降级、Benford 等）
- 强制 Agent-6 执行 Reverse DCF，并输出隐含预期难度分（>=4 自动降级）
- 新增 `base_rates/` 基础概率知识库（6个 YAML）
- 扩展 Agent-2 叙事-行动一致性检查并输出管理层诚信指数
- 在 7 套行业模板引入颠覆性技术风险评估输出块
- 新增数据时效性硬熔断规则与 M-011/M-012/M-013
- 增加一致预期 `[V5.2 锚定校正]` 与分歧度不确定性溢价并入 WACC
- 强制机会成本评分（无风险利率 / 同业Top3 / 大盘CAPE）
- 新增第十一部分执行纪律协议（分批建仓 + 头寸大小建议）
- 升级 MODE_PORTFOLIO_MONITOR（alpha 分解、行为一致性、延迟统计）
- 扩展 Agent-7 反共识打分（Variant Perception Score）
- 要求 TERMINATE/Avoid 在 6/12 个月做反事实复盘（TN/FN）
- 新增 `agents/agent_8_calibration_auditor.md`
- 新增 `examples/README.md`
- 新增 `tests/red_team_cases.md`
- 新增根目录 `README.md`

## V5.1-MAS

- 多智能体工程化版本，含 ADVANCED-MFG 模板、模板混合机制、A股/港股市场适配层。

## V5.0-MAS

- 引入多智能体 DAG、数据中介、意图路由与规则层级体系。

## V4.3-Lite

- 轻量化分析流程与快速筛选能力。

## V4.2-Pro-Final

- 强化认知纪律与审慎表达框架。

## V4.1-Gemini-Ultimate

- Gemini 场景优化版，提供分阶段输出与完整性约束。
