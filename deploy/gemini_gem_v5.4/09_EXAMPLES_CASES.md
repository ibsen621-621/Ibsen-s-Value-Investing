---
version: 5.4-Gem
role: 经典案例与示例（V5.3 测试场景、红队失败案例、V5.4 验收测试）
load_priority: 9
---

# 09 EXAMPLES & CASES — VIAIOS-P V5.4 Representative Cases

> Cross-ref: [[02_AGENTS_ROSTER.md]], [[06_VALUATION_MODELS.md]], [[07_RISK_FRAMEWORK.md]]

---

## 目录 (TOC)

1. [使用说明](#1-使用说明)
2. [V5.3 测试场景一：Hard-to-Value 独角兽（SpaceTech Alpha）](#2-v53-测试场景一hard-to-value-独角兽)
3. [V5.3 测试场景二：高杠杆困境企业（Grand Casino Holdings）](#3-v53-测试场景二高杠杆困境企业)
4. [V5.3 测试场景三：周期股常态化（Pacific Energy Corp）](#4-v53-测试场景三周期股常态化)
5. [红队失败案例库（V5.2 核心集）](#5-红队失败案例库)
6. [V5.4 Patch 特定验收案例](#6-v54-patch-特定验收案例)
7. [案例记录模板](#7-案例记录模板)

---

## 1. 使用说明

本文件包含：
- **V5.3 标准测试场景**：用于验证 Damodaran 7 大工具和 VR-01~VR-05 是否正确激活
- **红队失败案例**：用于验证系统能否在历史"已知失败案例"上触发正确红旗
- **V5.4 Patch 验收案例**：用于验证 8 个 Patch 是否按规范运作

**示例公司声明**：所有案例中的公司名称（如 SpaceTech Alpha、Grand Casino Holdings、Pacific Energy Corp）均为测试目的的假设公司，数据仅供格式参考（`example_only: true`）。

---

## 2. V5.3 测试场景一：Hard-to-Value 独角兽

### 场景背景

**标的类型**：早期颠覆性独角兽，无上市历史，跨越多个前沿领域  
**主要挑战**：缺乏可比公司、无盈利历史、多业务线估值逻辑截然不同  
**验证目标**：工具1（SOTP）、工具2（蒙卡）、工具3（扩张期权）、工具5（动态ERP）

### 输入提示

```
分析模式：MODE_HARD_TO_VALUE

公司概况（example_only）：
- 名称：SpaceTech Alpha
- 业务板块：
  1. 火箭发射服务：年营收 45亿美元，全球发射市场占有率约35%
  2. 卫星宽带网络（类Starlink）：年营收 28亿美元，已部署1,200颗卫星
  3. AI推演/国防数据服务：年营收 8亿美元
  4. 太空采矿业务：尚未商业化
  5. 火星载人探测计划：研发阶段，预计2035年后可能商业化
- IPO路演估值：1.75万亿美元
- 卖方研报：花旗给出 2035年 EV/Sales = 5x 的目标估值
```

### 预期触发规则

| 规则 | 触发原因 | 预期输出 |
|------|---------|---------|
| **VR-01** | IPO 价格 > 蒙卡 P75 | `vr01_triggered: true`；BUY 评级被否决 |
| **VR-02** | 5个业务板块，单一板块 < 60% | 强制 SOTP，禁止整体 DCF |
| **VR-05** | 花旗使用 2035年远期 EV/Sales | `PRICING_GYMNASTICS_FAR_FORWARD` |

### 预期关键输出字段

**SOTP 分解（必须出现）**：
```
业务线1：火箭发射（成熟低成本垄断者）→ 独立DCF
  Sales/Capital（参照 base_rates）: 航天发射 中值 0.65
业务线2：卫星宽带→ TAM渗透率分析 + 独立DCF
  Sales/Capital（成熟期）: 卫星宽带 中值 0.40→0.70
业务线3：AI/国防数据服务 → SaaS模板 DCF
  Sales/Capital: 软件 SaaS 中值 2.20
扩张期权1：太空采矿 → 工具3（独立列示）
扩张期权2：火星载人 → 工具3（远期期权，贡献占比须 ≤15%）

叙事-数字一致性检验：
  [✓] 业务线1：营收假设与全球发射市场 TAM×35%市占率一致
  [?] 业务线2：第10年营收需与全球宽带用户×ARPU×渗透率一致
  [!] 扩张期权贡献占比：[X]%（若>15%须说明）
```

**蒙卡分布（必须出现）**：
```
估值分布（10,000次模拟）：
  P10: [X]亿美元 | P25: [X] | P50（中位数）: [X] | P75: [X] | P90: [X]
  
IPO定价1.75万亿美元所处分位数：预期 > P75
VR-01状态：触发（若 IPO价格 > P75）
```

**定价体操检测（必须出现）**：
```
花旗研报检测：
  检测到：PRICING_GYMNASTICS_FAR_FORWARD
  原因：用2035年预期营收（约900亿美元）压低EV/Sales倍数至5x，
        但同组对标的波音/洛马仍使用当期乘数
  结论：研报不得作为唯一估值依据（VR-05触发）
```

### 失败判定（以下情况视为测试失败）

- ❌ 系统执行整体 DCF 而未执行 SOTP（VR-02 违规）
- ❌ 太空采矿/火星计划被揉进主营 DCF 折现率中
- ❌ 蒙卡分布缺失（仍使用 Bear/Base/Bull 三点情景）
- ❌ 花旗研报未被检测到定价体操
- ❌ 未输出 P10/P25/P50/P75/P90 分位数

---

## 3. V5.3 测试场景二：高杠杆困境企业

### 场景背景

**标的类型**：高杠杆、资产密集、面临地缘政治风险  
**验证目标**：工具4（双轨分离）、工具5（动态ERP/CRP）、VR-03

### 输入提示

```
分析模式：MODE_FULL_DD

公司概况（example_only）：
- 名称：Grand Casino Holdings
- 业务：澳门博彩（65%）+ 新加坡综合度假村（25%）+ 美国本土（10%）
- 财务指标（TTM）：
  - EBITDA: 18亿美元；净负债: 92亿美元
  - 净负债/EBITDA: 5.1x；利息保障倍数: 1.9x
  - 公司债 BB 评级，债券利差约 780bps
  - 澳门特区营业执照 2028年到期续期
```

### 预期触发规则

| 规则 | 触发原因 |
|------|---------|
| **VR-03** | 净负债/EBITDA = 5.1x > 4 AND ICR = 1.9x < 3 |
| **C-014** | 澳门营收 65% > 30% + 地缘敏感区域 |
| **M-013**（潜在） | 中东战争背景下 ERP 须用最新值 |

### 预期关键输出字段

**双轨期望值（必须出现）**：
```
[VR-03 MANDATORY 触发]
P_distress 计算（从市场数据反推，禁止主观估算）：
  BB 债券交易利差：780bps
  假设回收率：40%（参照 bond_implied_default_rates.yml BB级别）
  P_distress_annual ≈ 780bps / (1 - 40%) ≈ 13.0%/年
  P_distress_5y ≈ 1 - (1 - 13%)^5 = 49%
  合理性校验：BB级别历史5年累计违约率~6.4%，当前780bps明显高于均值320bps，
             市场对该公司风险评估高于评级隐含水平（须说明）

V_total = V_going_concern × (1 - 49%) + V_liquidation × 49%

禁止声明：本分析禁止仅靠将 WACC 调高来处理破产风险（VR-03 合规）
```

**动态WACC（必须出现）**：
```
当期 Implied ERP（from implied_erp_history.yml）：4.65%（2026-04）
澳门/中国 CRP（from sovereign_cds_snapshot.yml）：中国 CDS 72bps → CRP ≈ 0.7%
澳门续牌特定风险溢价：0.8%（标注 "estimate"）

Ke = 4.40% + 1.1 × 4.65% + 0.7% + 0.8% ≈ 11.0%
```

### 失败判定

- ❌ 未触发 VR-03，使用单一 DCF 调高 WACC 处理破产风险
- ❌ P_distress 使用主观估算而非债券利差反推
- ❌ 动态 ERP 使用历史均值（4.6%）而非最新值（4.65%）

---

## 4. V5.3 测试场景三：周期股常态化

### 场景背景

**标的类型**：成熟大型周期性大宗商品企业  
**主要挑战**：景气周期末段，TTM 利润率处于历史高位  
**验证目标**：工具7（周期常态化）、VR-04、M-013

### 输入提示

```
分析模式：MODE_VALUATION_REVIEW

公司概况（example_only）：
- 名称：Pacific Energy Corp
- 业务：综合石油公司（上游 E&P + 炼油 + 化工）
- 财务指标（TTM，当前高油价景气期）：
  - 营收: 4,200亿美元；EBIT: 890亿美元
  - TTM EBIT 利润率：21.2%（近5年最高）
  - 当前布伦特油价：95美元/桶（高于历史均值约40%）
- 卖方普遍给出 15x TTM P/E 的目标价
- 中东战争背景（2026-03 M-013 触发）
```

### 预期触发规则

| 规则 | 触发原因 |
|------|---------|
| **VR-04** | TTM 利润率 21.2% vs 中周期均值约 14% | 
| **M-013** | 中东战争爆发后 14 天内 |
| **C-014** | 中东营收 25% 接近触发阈值 |

### 预期关键输出字段

**周期常态化（必须出现）**：
```
[VR-04 MANDATORY 触发]
CYCLICAL 模板（Integrated Oil & Gas）：禁止使用 TTM 利润率 21.2% 做 DCF

方法A（10年滚动均值，2014-2023）：
  来源：cyclical_midcycle_margins.yml，Integrated Oil & Gas
  中周期 EBIT 利润率中值：14.0%（区间 8-20%）

方法B（布伦特价格回归）：
  中周期布伦特假设：65美元/桶（IEA长期均衡价格）
  β ≈ 0.35 → 正常化 EBIT 利润率约 13-15%

TTM：21.2% vs 正常化：14.0% → 差值 -7.2pp（景气顶部风险警告）
正常化 EBIT = 4,200亿 × 14.0% = 588亿美元（vs TTM 890亿）

必须披露：
  □ 周期窗口：2014-2023（10年）
  □ 中点取法：算术均值
  □ 当前处于景气晚期，触发 C-009（高利润+高资本支出+高估值）
```

**动态ERP校准（必须出现）**：
```
M-013 触发：2026年3月中东战争
  2026-02 ERP: 4.37%（战前）→ 2026-03 ERP: 4.77%（战时，+40bp）→ 2026-04: 4.65%
  禁止使用历史均值 4.60%（M-013 触发后不适用）
```

### 失败判定

- ❌ 使用 TTM 利润率 21.2% 做 DCF 终值（VR-04 违规）
- ❌ WACC 使用 2026-02 前的历史均值 ERP（未执行 M-013 重算）
- ❌ 未引用 cyclical_midcycle_margins.yml 对应行业区间

---

## 5. 红队失败案例库

> 目标：验证系统能否在历史"已知失败案例"上触发正确红旗。

| Case ID | 标的 | 时点 | 应触发 Agent | 关键红旗 | 期望系统行为 |
|---------|------|------|------------|---------|------------|
| **RT-001** | 瑞幸咖啡 Luckin Coffee | 2020Q1前后 | Agent-0, Agent-7 | 收入真实性异常、审计/披露问题 | 触发法证红旗，TERMINATE/AVOID |
| **RT-002** | 康美药业 | 2018-2019 | Agent-0 | 货币资金与现金流矛盾、关联交易可疑 | 强制 TERMINATE（Beneish M + 应计率双触发） |
| **RT-003** | Wirecard | 2019-2020 | Agent-0, Agent-2 | 现金余额真实性、审计异常、管理层叙事失真 | 触发红旗并降级/终止 |
| **RT-004** | Enron | 2000-2001 | Agent-0, Agent-2, Agent-7 | 复杂表外结构、利润质量异常、治理失效 | 触发多重红旗并终止 |
| **RT-005** | 雷曼兄弟 | 2007-2008 | Agent-5, Agent-7 | 流动性脆弱、宏观金融压力、杠杆失控 | 风险扫描转为高危，VR-03 触发 |
| **RT-006** | 思科 Cisco（泡沫期） | 2000 | Agent-6, Agent-7 | 隐含增长预期极高、估值脱离基本面 | Reverse DCF 难度分 = 9-10；评级下调 |

### 红队案例操作标准

```
对于每个案例，系统应：
1. 在第几步（哪个 Agent）触发哪条阈值
2. 输出的具体红旗标记和 TERMINATE/AVOID 信号
3. 若未触发，记录为"模型漏检"并进入 Agent-8 校准回路

扩展建议：
- 恒大（高杠杆+地产商）：应触发 VR-03、C-014
- FTX：审计机构异常、现金流与营收严重背离、Agent-0 即终止
```

---

## 6. V5.4 Patch 特定验收案例

### 案例 P4-001：Regime Shift（Patch 4）

```
场景：某 A 股先进制造企业
  当前 capex_ttm: 280亿元
  5年平均 capex: 160亿元
  capex_ratio: 280/160 = 1.75（> 1.5 阈值）
  连续超过 1.5x 的季度数: 3（≥ 2，触发）

预期系统行为：
  regime_shift_flag: true
  dcf_status: INVALID-REGIME-SHIFT（任何使用历史 FCF 外推的 DCF）
  forced_redcf 块必须出现
  low_pe_safety_bonus 下调一级
  
验收点：
  - 报告不得将历史 FCF DCF 作为 BUY-CANDIDATE 的价格锚点
  - 必须包含 regime_shift_redcf 块（含 new_reinvestment_rate 计算）
```

### 案例 P2-001：Pool 上限驱逐（Patch 2）

```
场景：Active Conviction Pool current_size = 30，新名称想进入

预期系统行为：
  Agent-5 在 ADD 之前运行驱逐协议
  按序排列驱逐候选：最低 edge_score → 最低 APR → 最旧未验证 checkpoint
  EVICT 事件 + ADD 事件均记录在 pool_event_log
  
验收点：
  - current_size 在任何时间点均 ≤ 30
  - pool_event_log 中每个 EVICT 事件有明确的驱逐原因
```

### 案例 P7-001：WAIT 门控（Patch 7）

```
场景：Active Pool 中所有名称的 APR < 50 或 edge_score < 4.0

预期系统行为：
  WAIT_DEFAULT = true
  系统输出 WAIT + 完整 wait_memo
  wait_memo 包含：
    - universe_scanned: [N]
    - gate_failures 统计
    - closest_to_gate 名称及距离两个门控的差距
    - patience_premium_note

验收点：
  - 没有 BUY-CANDIDATE/SIZE-UP 输出（任何 Agent）
  - wait_memo.next_scan_date 已填写
```

### 案例 P8-001：边界宪章（Patch 8）

```
场景：注入 management_quality_score: 9.0 到 Agent-7 输入中

预期系统行为：
  advisory_status: "ADVISORY-ONLY"
  recommendation: null
  charter_violation_warning 被填写（非 null）
  triggering_field: "management_quality_score"

验收点：
  - recommendation 字段值为 null（不得为 BUY 或任何其他评级）
  - charter_violation_warning 明确引用 DOCTRINE-001 人机边界宪章
```

---

## 7. 案例记录模板

```yaml
# 预测记录（用于 Agent-8 校准审计）
prediction_record:
  report_id: "rpt-[YEAR]-[NNNN]"
  ticker: <symbol>
  report_date: <YYYY-MM-DD>
  mode: <MODE_FULL_DD | MODE_VALUATION_REVIEW | ...>
  rating: <BUY-CANDIDATE | SIZE-UP | HOLD | TRIM | EXIT | WAIT | PASS>
  confidence_score: <0-100>
  
  intrinsic_value_distribution:
    p10: <float>
    p25: <float>
    p50: <float>
    p75: <float>
    p90: <float>
    current_price_percentile: <float>
  
  probabilities:
    bear: <float>
    base: <float>
    bull: <float>
  
  expected_horizon_months: 36
  
  key_assumptions:
    - <string>
    - <string>
    - <string>
  
  kill_criteria_thresholds:
    - code: <string>
      metric: <string>
      threshold: <string>
      frequency: <quarterly | annual>
  
  alpha_source:
    type: <PROBABILISTIC | INFORMATIONAL | CATALYST>
    edge_evidence: <string>
    ai_replicable: <bool>

# 后验记录（Agent-8 校准用，6/12个月后填写）
realized_outcomes_6m:
  actual_price_change_pct: <float>
  narrative_held: <bool>
  checkpoints_passed: <int>
  checkpoints_failed: <int>
  agent_8_classification: <TRUE_POSITIVE | TRUE_NEGATIVE | FALSE_POSITIVE | FALSE_NEGATIVE>
```

---

*End of 09_EXAMPLES_CASES.md — VIAIOS-P V5.4-Gem*
