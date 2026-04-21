# Ibsen's Value Investing Framework

VIAIOS-P 是一个面向价值投资研究的多智能体分析框架，强调：
- 先排雷（法证与风险）再谈增长
- 概率思维与反证机制
- 结构化输出与可追溯复盘

## 版本差异矩阵

| 版本 | 核心特性 | 适用场景 |
|---|---|---|
| V4.1 | Gemini 深度适配、分阶段输出、完整性检查 | 需要稳定长文分析输出 |
| V4.2 Pro | 强化认知纪律与诚实表达（低置信度优先） | 强调“不过度自信”的深度研究 |
| V4.3 Lite | 轻量化流程、较低执行成本 | 快速筛选与高频初筛 |
| V5.0 / V5.1 MAS | 多智能体 DAG、数据中介、意图路由、行业模板（含先进制造/A股适配） | 机构级完整尽调 |
| V5.2 Calibrated | 在 V5.1 上新增校准回测闭环、基础概率库、执行纪律协议、反事实追踪 | 追求分析准确性与赚钱概率的长期迭代 |

## V5.2 重点升级（摘要）

1. Calibration Tracking Layer + Agent-8
2. 法证二阶检验
3. Reverse DCF 强制化 + 隐含预期难度分
4. Base Rate Library（`base_rates/`）
5. 管理层“叙事 vs 行动”一致性
6. 行业模板新增颠覆性技术风险评估
7. 数据时效性硬熔断（M-011/012/013）
8. 分析师一致预期锚定校正
9. 机会成本强制项
10. 执行纪律（建仓节奏）
11. 已实现 vs 应得回报归因
12. 头寸大小（½ Kelly）
13. 反共识打分
14. TERMINATE/Avoid 反事实记录
15. 目录工程化（agents/examples/tests）
16. README + CHANGELOG

## 使用方式

将对应版本 Markdown 作为系统提示词粘贴到：
- Gemini Gems / AI Studio
- ChatGPT（Custom GPT / Instructions）
- Claude Projects

建议同时提供：
- 标的代码（Ticker）
- 分析模式（如 `MODE_FULL_DD`）
- 数据截止日与市场（US/A股/HK）

## 免责声明

本项目用于研究与教育目的，不构成任何投资建议。市场有风险，决策需独立完成并自担风险。
