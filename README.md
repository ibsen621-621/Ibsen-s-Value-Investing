# Ibsen's Value Investing 决策工作台

这是一个**投资决策工作台，不是信息归档库**。

核心理念：**停止让 AI 给你信息，开始让 AI 帮你证伪假设。**

## 使用原则

- 停止产出周报式信息汇总。
- 每次使用 AI，必须以一条可执行判断收尾。
- 每次使用 AI 后，必须在 `decisions/` 产出或更新一条带复盘机制的决策记录。

## 三条触发工作流（独立运行）

### 1) 风险雷达（Risk Radar）
- 触发条件：持仓或观察池出现业绩异动、政策变化、估值突破阈值、行业重大事件。
- 目标：快速判断要不要减仓/清仓。
- 输出位置：`risk-radar/`

### 2) 机会挖掘（Opportunity Hunt）
- 触发条件：你提出一个可证伪、可执行的具体假设。
- 目标：形成候选清单，并强制构造反方（steelman）论证。
- 输出位置：`opportunity-hunt/`

### 3) 跟踪机会（Watchlist Tracking）
- 触发条件：财报/关键数据更新，或达到预设阈值；也可按固定节奏复核。
- 目标：只更新“判断变化（delta）”，不重复背景叙事。
- 输出位置：`watchlist/` 与 `decisions/`

## 目录导航

```text
Ibsen-s-Value-Investing/
├── README.md
├── prompts/
│   ├── README.md
│   ├── company-deep-dive.md
│   ├── macro-trigger.md
│   ├── industry-shift.md
│   └── risk-check.md
├── watchlist/
│   ├── README.md
│   └── TEMPLATE.md
├── decisions/
│   ├── README.md
│   └── TEMPLATE.md
├── risk-radar/
│   └── README.md
├── opportunity-hunt/
│   └── README.md
└── postmortems/
    ├── README.md
    └── TEMPLATE.md
```

## 禁止事项（硬约束）

- 禁止产出“宏观/行业/公司周报式综述”。
- 禁止以“需要进一步观察”作为结尾。
- 禁止提出“帮我分析一下 XX 公司”这类开放式问题。
- 禁止 AI 使用后不产出/不更新 `decisions/` 决策记录。

## 历史资料说明

仓库原有 VIAIOS-P 版本文档、方法论与测试样例仍保留，用于历史参考；本仓库的当前主用途已切换为“投资决策工作台”。
