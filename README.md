# Ibsen's Value Investing Framework

VIAIOS-P 是一个面向价值投资研究的多智能体分析框架，强调：
- 先排雷（法证与风险）再谈增长
- 概率思维与反证机制
- 结构化输出与可追溯复盘

## 使用方式（快速上手）

### Gemini Gems 部署（推荐 V5.3）

1. 打开 [Google Gemini](https://gemini.google.com) → 点击"Gems" → "新建 Gem"
2. 将 `VIAIOS-P-V5.3-Damodaran-Calibrated.md` 全文粘贴到 **Instructions** 框
3. 按 [`deploy/gemini_gems_v5.3.md`](deploy/gemini_gems_v5.3.md) 清单上传知识库附件（≤10 个）
4. 命名 Gem：`VIAIOS-P V5.3 Damodaran-Calibrated`
5. 运行测试验证（详见 `tests/v5_3_damodaran_cases.md`）

### 其他平台部署

将对应版本 Markdown 作为系统提示词粘贴到：
- ChatGPT（Custom GPT / Custom Instructions）
- Claude Projects

建议同时提供：标的代码（Ticker）、分析模式（如 `MODE_FULL_DD` / `MODE_HARD_TO_VALUE`）、数据截止日与市场（US/A股/HK）

---

## 版本差异矩阵

| 版本 | 核心特性 | 适用场景 | Gemini Gems |
|---|---|---|---|
| V4.1 | Gemini 深度适配、分阶段输出、完整性检查 | 需要稳定长文分析输出 | ✅ |
| V4.2 Pro | 强化认知纪律与诚实表达（低置信度优先）| 强调"不过度自信"的深度研究 | ✅ |
| V4.3 Lite | 轻量化流程、较低执行成本 | 快速筛选与高频初筛 | ✅ |
| V5.0 / V5.1 MAS | 多智能体 DAG、数据中介、意图路由、行业模板（含先进制造/A股适配）| 机构级完整尽调 | ⚠️ |
| V5.2 Calibrated | V5.1 + 校准回测闭环、基础概率库、执行纪律协议、反事实追踪 | 追求分析准确性与赚钱概率的长期迭代 | ✅ |
| **V5.3 Damodaran-Calibrated** | **V5.2 + Damodaran 7大估值工具 + VR-01~05 硬规则 + Agent-9 + 蒙卡分布** | **独角兽/困境/周期/跨国资产的高阶估值** | **✅ 专属部署指引** |

## V5.3 重点升级（摘要）

1. **Agent-9 Damodaran Engine**（新增，隶属 Module 3 估值层）
2. **§10.5 Narrative Decomposition + SOTP 强制路由**（VR-02）
3. **§11.5 蒙特卡洛估值分布**（取代三点情景，≥10,000次，P10-P90）
4. **§11.6 Expansion Option Module**（扩张期权显性化，贡献上限 15%）
5. **§11.7 Pricing Gymnastics Detector**（卖方研报排雷，VR-05）
6. **§12.3 Distress-Adjusted Valuation**（双轨分离，P_distress 从债市反推，VR-03）
7. **§14.3 Dynamic Risk Calibration**（Implied ERP + 主权 CDS + 黑天鹅触发）
8. **§8.4 Restated Financials**（R&D/获客成本资本化，双口径 ROIC）
9. **§5.2.8 CYCLICAL 常态化强制规则**（VR-04，禁止 TTM 利润率）
10. **MODE_HARD_TO_VALUE**（新增意图路由模式，独角兽/困境/战时）
11. **§15.1 一票否决规则集中归档**（VR-01～VR-05 完整档案）
12. **5 个新增 base_rates YAML**
13. **`deploy/gemini_gems_v5.3.md`**（两种部署方案）
14. **`tests/v5_3_damodaran_cases.md`**（SpaceX/LVS/Exxon 三个红队测试）

---

## 文件结构

```
/
├── VIAIOS-P-V4.1-Gemini-Ultimate.md
├── VIAIOS-P-V4.2-Pro-Final.md
├── VIAIOS-P-V4.3-Lite.md
├── VIAIOS-P-V5.0-MAS.md
├── VIAIOS-P-V5.2-Calibrated.md
├── VIAIOS-P-V5.3-Damodaran-Calibrated.md  ← 最新版本
├── README.md
├── CHANGELOG.md
│
├── agents/
│   ├── agent_8_calibration_auditor.md
│   └── agent_9_damodaran_engine.md         ← V5.3 新增
│
├── base_rates/
│   ├── roic_persistence.yml
│   ├── margin_reversion.yml
│   ├── growth_persistence.yml
│   ├── ipo_long_term_returns.yml
│   ├── m_and_a_value_creation.yml
│   ├── turnaround_success.yml
│   ├── bond_implied_default_rates.yml      ← V5.3 新增
│   ├── implied_erp_history.yml             ← V5.3 新增
│   ├── sales_to_capital_by_industry.yml    ← V5.3 新增
│   ├── cyclical_midcycle_margins.yml       ← V5.3 新增
│   └── sovereign_cds_snapshot.yml          ← V5.3 新增
│
├── deploy/
│   └── gemini_gems_v5.3.md                 ← V5.3 新增
│
├── examples/
│   └── README.md
│
└── tests/
    ├── red_team_cases.md
    └── v5_3_damodaran_cases.md             ← V5.3 新增
```

## 免责声明

本项目用于研究与教育目的，不构成任何投资建议。市场有风险，决策需独立完成并自担风险。
