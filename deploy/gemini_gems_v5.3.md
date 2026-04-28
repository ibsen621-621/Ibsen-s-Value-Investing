# Gemini Gems 部署清单 — VIAIOS-P V5.3 Damodaran-Calibrated

> 本文档为 V5.3 的 Gemini Gems 部署操作手册，提供**两种部署方案**（最小集 / 完整集），
> 适用于希望将 VIAIOS-P V5.3 作为专属 Gemini Gem 使用的研究团队或个人投资者。

---

## 一、Gemini Gems 部署约束（重要）

| 约束项 | 限制 |
|--------|------|
| Instructions（系统提示词） | 1 个文件 |
| Knowledge 附件（知识库） | 最多 10 个 |
| Instructions 是否计入附件数 | ⚠️ 平台差异：若 Instructions 计入附件，则实际 Knowledge 最多 9 个 |
| 文件格式支持 | .md、.pdf、.txt、.yaml/.yml、.json 等 |
| 单文件大小建议 | < 500KB（超过可能影响加载） |

---

## 二、方案A：完整部署模式（推荐）

> **适用场景**：机构级深度研究，需要完整的 Damodaran 7 大工具 + 动态校准能力。  
> **总文件数**：1（Instructions）+ 9（Knowledge）= 10 个（满足 ≤10 限制）。

### Instructions 文件（粘贴到 Gem 的 Instructions 框）

| 编号 | 路径 | 用途 | 估计大小 |
|------|------|------|---------|
| **Instructions** | `VIAIOS-P-V5.3-Damodaran-Calibrated.md` | 主系统提示词，包含全部分析协议、7大工具、5条硬规则 | ~100-120 KB |

### Knowledge 附件清单（9 个，上传至 Knowledge 框）

| # | 路径 | 用途 | 调用时机 | 估计大小 |
|---|------|------|---------|---------|
| 1 | `agents/agent_8_calibration_auditor.md` | Agent-8 校准审计师详细协议；Agent-O 归档后触发 | 每份报告完成后；校准回测时 | ~5 KB |
| 2 | `agents/agent_9_damodaran_engine.md` | Agent-9 达莫达兰引擎详细协议；7大工具执行规程 | 所有涉及估值的分析（尤其 MODE_HARD_TO_VALUE） | ~16 KB |
| 3 | `base_rates/bond_implied_default_rates.yml` | 债券隐含违约率基准；P_distress 反推锚点 | §12.3 双轨分离触发时（净负债/EBITDA > 4 或 ICR < 3） | ~5 KB |
| 4 | `base_rates/implied_erp_history.yml` | 月度隐含ERP历史；动态WACC校准基准 | §14.3 动态ERP校准；黑天鹅事件后 M-013 触发时 | ~4 KB |
| 5 | `base_rates/sales_to_capital_by_industry.yml` | 行业销售资本比；SOTP 再投资率锚定 | §10.5 SOTP 执行时（多业务线公司） | ~7 KB |
| 6 | `base_rates/cyclical_midcycle_margins.yml` | 周期行业中周期利润率；VR-04 合规依据 | §6 CYCLICAL 模板；周期股 DCF 正常化 | ~7 KB |
| 7 | `base_rates/sovereign_cds_snapshot.yml` | 主权CDS利差快照；跨国资产 CRP 实时更新 | §14.3(b) 跨国资产估值；地缘危机触发时 | ~7 KB |
| 8 | `base_rates/roic_persistence.yml` | 高 ROIC 维持概率；护城河持久性校准 | §8 Financial Quality；Agent-1 ROIC 分析时 | ~1 KB |
| 9 | `base_rates/margin_reversion.yml` | 超额利润率回归半衰期；均值回归压力测试 | §工具2 外部视角；估值利润率假设校准时 | ~1 KB |

**总计：1 + 9 = 10 个文件，满足 Gemini Gems ≤10 附件限制。**

---

## 三、方案B：最小部署模式

> **适用场景**：个人研究者或存储限制环境，优先保留核心 Damodaran 工具。  
> **总文件数**：1（Instructions）+ 4（Knowledge）= 5 个。

### Instructions 文件（同方案A）

`VIAIOS-P-V5.3-Damodaran-Calibrated.md`

### Knowledge 附件清单（4 个核心 base_rates）

| # | 路径 | 保留原因 |
|---|------|---------|
| 1 | `base_rates/implied_erp_history.yml` | 动态ERP是V5.3最核心的新增工具，必须保留 |
| 2 | `base_rates/bond_implied_default_rates.yml` | 双轨分离法（高杠杆公司分析）必需 |
| 3 | `base_rates/sales_to_capital_by_industry.yml` | SOTP（多业务线估值）必需 |
| 4 | `base_rates/cyclical_midcycle_margins.yml` | 周期股分析必需 |

> **注意**：最小模式下，sovereign_cds_snapshot.yml 不上传。
> 跨国资产分析时，Agent-9 将使用降级处理（评级均值替代），
> 在输出中自动标注"最小部署模式，CRP 使用评级均值替代，建议补充 sovereign_cds_snapshot.yml"。

---

## 四、部署步骤（Gemini Gems）

### 步骤 1：进入 Gemini Gems 创建界面

1. 打开 [Google Gemini](https://gemini.google.com)
2. 点击左侧"Gems"→"新建 Gem"（或进入 [aistudio.google.com](https://aistudio.google.com) 的 System Instruction 功能）

```
[截图位置 - Gem 创建界面入口]
```

### 步骤 2：粘贴 Instructions

1. 打开 `VIAIOS-P-V5.3-Damodaran-Calibrated.md` 文件
2. 全选内容（Ctrl+A）并复制
3. 粘贴到 Gem 的 **"Instructions"** 文本框中
4. 确认字符数在 Gemini 的提示词限制以内

```
[截图位置 - Instructions 粘贴完成后的预览]
```

### 步骤 3：上传 Knowledge 附件

1. 点击 **"Knowledge"** → **"Add files"**
2. 按方案A或方案B的清单依次上传文件
3. 确认所有文件显示"已上传"状态

```
[截图位置 - Knowledge 附件上传完成后的清单视图]
```

### 步骤 4：设置 Gem 名称和描述

- **名称建议**：`VIAIOS-P V5.3 Damodaran-Calibrated`
- **描述建议**：`AI 价值投资分析框架 V5.3，集成达莫达兰7大估值工具，适用于独角兽/周期/困境标的的高阶估值分析`

### 步骤 5：保存并测试

1. 点击"保存" / "Save"
2. 执行以下验证测试（见第五节）

---

## 五、部署验证清单（必须通过以下测试）

部署完成后，依次输入以下测试 Prompt，验证 V5.3 关键功能是否正常激活：

### 验证测试 1：SpaceX 类独角兽（Hard-to-Value）

**测试 Prompt**：
```
请对一家类似 SpaceX 的商业航天公司进行估值分析，该公司业务包括：
火箭发射服务（当前营收50亿美元）、卫星互联网（Starlink类，当前营收30亿美元）、
火星探测（早期阶段，无营收）。分析模式：MODE_HARD_TO_VALUE
```

**预期验证点**：
- [ ] 系统自动路由至 `MODE_HARD_TO_VALUE`
- [ ] 触发 `Agent-9` 全套工具（SOTP + 蒙卡 + 期权）
- [ ] 输出 SOTP 业务线拆分（3条叙事）
- [ ] 输出蒙卡估值分布（P10/P25/P50/P75/P90）
- [ ] 火星探测业务单独列示为扩张期权（贡献占比 ≤15%）
- [ ] 引用 `sales_to_capital_by_industry.yml` 对应行业销售资本比
- [ ] 叙事-数字一致性检验表出现在输出中

### 验证测试 2：Las Vegas Sands 高杠杆困境（Distressed）

**测试 Prompt**：
```
请分析 Las Vegas Sands（LVS）的估值，当前该公司净负债/EBITDA = 5.2x，
利息保障倍数 = 2.1x，CDS利差约850bps。分析模式：MODE_FULL_DD
```

**预期验证点**：
- [ ] 触发 `VR-03` 硬规则（净负债/EBITDA > 4 且 ICR < 3）
- [ ] 执行双轨分离：V_total = V_going_concern × (1-P_distress) + V_liquidation × P_distress
- [ ] P_distress 从 CDS 利差反推（约 850bps/(1-0.40) = 14%/年），非主观估算
- [ ] 引用 `bond_implied_default_rates.yml` 做合理性校验
- [ ] 输出中明确显示"禁止仅靠调高 WACC 处理破产风险"
- [ ] 蒙卡分布包含 P10/P50/P90

### 验证测试 3：Exxon 周期股常态化（Cyclical）

**测试 Prompt**：
```
请分析埃克森美孚（Exxon Mobil, XOM）的估值。
当前 TTM EBIT 利润率为 22%，但过去10年平均约 13%，当前布伦特油价 95 美元/桶。
分析模式：MODE_VALUATION_REVIEW
```

**预期验证点**：
- [ ] 触发 `VR-04` 硬规则（CYCLICAL 模板，禁止使用 TTM 利润率 22%）
- [ ] 使用正常化利润率（约 13-14% 中周期均值）而非 TTM 22%
- [ ] 引用 `cyclical_midcycle_margins.yml` 石油行业区间
- [ ] 披露周期窗口年份（如 2014-2023）和中点取法
- [ ] 输出正常化 EBIT vs TTM EBIT 的差异（约 -9pp）
- [ ] 建立大宗商品价格回归方程（可选，若数据充足）

---

## 六、常见问题排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| Agent-9 未激活 SOTP | 公司业务线数据未清晰说明 | 在 Prompt 中明确列出各业务线营收占比 |
| 蒙卡分布过宽（P90/P10 > 20x） | 核心变量假设区间过大 | 提供更具体的行业数据缩窄分布 |
| VR-01 误触发 | 股价与估值分布数据不一致 | 确认当前股价是分析时最新价格 |
| 动态ERP未更新 | implied_erp_history.yml 未上传或过时 | 检查 Knowledge 附件是否包含该文件；更新月度数据 |
| Pricing Gymnastics 未检测 | 未提供卖方研报内容 | 在 Prompt 中粘贴卖方研报关键假设供系统检测 |
| 中文输出被英文替代 | 意图路由器识别为英文模式 | 在 Prompt 开头明确"请使用中文输出分析报告" |

---

## 七、版本管理与更新建议

| 更新项目 | 建议频率 | 操作 |
|---------|---------|------|
| `implied_erp_history.yml` | 月度更新 | 从 [damodaran.com](http://pages.stern.nyu.edu/~adamodar/) 获取最新月度数据，替换 Knowledge 附件 |
| `sovereign_cds_snapshot.yml` | 季度更新（危机期间即时更新）| 从 Bloomberg/Markit 获取最新 CDS 数据 |
| `bond_implied_default_rates.yml` | 年度更新 | 从 Moody's/S&P Annual Default Study 更新 |
| 主 Instructions（V5.3 主文档） | 发布新版本时替换 | 遵循版本迭代规范 |

---

## 八、Gemini Gems 附件数量方案对比

| 方案 | Instructions | Knowledge 附件数 | 总文件数 | 适用场景 |
|------|-------------|-----------------|---------|---------|
| **完整集（方案A）** | 1 | 9 | **10** ✅ | 机构级完整尽调 |
| **最小集（方案B）** | 1 | 4 | **5** ✅ | 个人研究/快速筛选 |
| **若 Instructions 计入附件** | 1（计入） | 8（不含主文档） | **9** ✅ | 仍满足 ≤10 限制 |

> 无论哪种方案，均满足 Gemini Gems **≤10 附件**的硬性限制。

---

*最后更新：2026-04-28 | VIAIOS-P V5.3 Damodaran-Calibrated*
