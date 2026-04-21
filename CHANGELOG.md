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
