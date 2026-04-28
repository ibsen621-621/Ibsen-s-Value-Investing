# V5.3 Damodaran-Calibrated — 红队测试用例

> 本文件包含 3 个红队测试场景，用于验证 VIAIOS-P V5.3 新增的 Damodaran 7 大工具和 5 条硬规则（VR-01~VR-05）是否正确激活。
> 测试应在部署完成后执行，所有"预期输出关键字段"均应出现在系统输出中。

---

## 测试场景1：Hard-to-Value 独角兽 — SpaceX 类商业航天公司

### 1.1 测试背景

**标的类型**：早期颠覆性独角兽，无上市历史，跨越多个前沿领域。  
**主要挑战**：缺乏可比公司、无盈利历史、多业务线估值逻辑截然不同。  
**此测试验证**：工具1（SOTP）、工具2（蒙卡）、工具3（扩张期权）、工具5（动态ERP）。

### 1.2 测试 Input Prompt

```
分析模式：MODE_HARD_TO_VALUE

请对以下商业航天公司进行完整估值分析：

公司概况（仅供测试，example_only）：
- 名称：[假设公司 SpaceTech Alpha]
- 成立：2015年
- 业务板块：
  (1) 火箭发射服务：当前年营收 45亿美元，全球发射市场占有率约35%
  (2) 卫星宽带网络（类Starlink）：当前年营收 28亿美元，已部署1,200颗卫星
  (3) AI推演/国防数据服务：当前年营收 8亿美元
  (4) 太空采矿业务：尚未商业化，有概念验证项目
  (5) 火星载人探测计划：研发阶段，预计2035年后可能商业化

- 当前轮次估值：1.75万亿美元（IPO路演定价）
- 融资历史：已融资300亿美元（私募轮次）
- 卖方研报：花旗给出2035年EV/Sales = 5x的目标估值（基于2035年预期营收）

问题：
1. 请执行 SOTP 分析，将5个业务板块按叙事逻辑分类
2. 请评估 1.75万亿美元 IPO 估值是否合理
3. 太空采矿和火星计划应如何处理？
4. 请检测花旗研报是否存在"定价体操"
```

### 1.3 预期触发的硬规则

| 规则 | 预期触发原因 | 预期输出 |
|------|------------|---------|
| **VR-01** | 若当前 IPO 价格 > 蒙卡P75 | `vr01_triggered: true`；BUY 评级被否决 |
| **VR-02** | 5个业务板块，单一板块占比 < 60% | SOTP 强制执行，禁止整体 DCF |
| **VR-05** | 花旗使用 2035 年远期 EV/Sales | 标记 `PRICING_GYMNASTICS_FAR_FORWARD` |

### 1.4 预期输出关键字段

**§10.5 SOTP 分解**（必须出现）：
```
SOTP 业务线拆分：
  业务线1：火箭发射（成熟低成本垄断者）→ 独立DCF
  业务线2：卫星宽带（颠覆性宽带基建）→ TAM渗透率分析 + 独立DCF
  业务线3：AI/国防数据服务 → SaaS模板DCF
  扩张期权1：太空采矿 → 工具3（期权估值，独立列示）
  扩张期权2：火星载人 → 工具3（远期期权，贡献占比须 ≤15%）

叙事-数字一致性检验表：
  [✓] 业务线1：营收假设与全球发射市场TAM×35%市占率一致
  [?] 业务线2：第10年营收需与全球宽带用户×ARPU×渗透率一致
  [！] 扩张期权贡献占比：[X]%（若>15%须说明）
```

**§11.5 蒙卡分布**（必须出现）：
```
估值分布（10,000次模拟）：
  P10：[X]亿美元
  P25：[X]亿美元
  P50（中位数）：[X]亿美元（预计约1.29万亿美元附近，远低于1.75万亿）
  P75：[X]亿美元
  P90：[X]亿美元
  
  IPO定价1.75万亿美元所处分位数：[>P75预期]
  VR-01状态：触发（若股价>P75）
```

**§11.6 扩张期权**（必须出现）：
```
扩张期权1：太空采矿
  触发年限：2032+（估计）
  爆发概率：5-15%
  期权价值贡献占总估值：[X]%（强制 ≤15%）
  
扩张期权2：火星载人旅行
  触发年限：2035+
  爆发概率：3-8%
  期权价值贡献占总估值：[X]%（强制 ≤15%）
  
注意：以上期权均独立列示，不得揉进主营DCF折现率
```

**§Pricing Gymnastics**（必须出现）：
```
花旗研报检测：
  检测到：PRICING_GYMNASTICS_FAR_FORWARD
  原因：用2035年预期营收（约900亿美元）压低EV/Sales倍数至5x，
        但同组对标的波音/洛马仍使用当期乘数
  反向对标池：若同等折算至2035年，传统航天同业也将呈现低倍数
  结论：研报不得作为唯一估值依据（VR-05触发）
```

### 1.5 失败判定（以下情况视为测试失败）

- ❌ 系统执行整体 DCF 而未执行 SOTP
- ❌ 太空采矿/火星计划被揉进主营 DCF 折现率中
- ❌ 蒙卡分布缺失（仍使用 Bear/Base/Bull 三点情景）
- ❌ 花旗研报未被检测到定价体操
- ❌ 未输出 P10/P25/P50/P75/P90 分位数

---

## 测试场景2：Distressed 高杠杆困境 — Las Vegas Sands 类博彩/地产

### 2.1 测试背景

**标的类型**：高杠杆、资产密集、面临地缘政治风险（澳门/新加坡业务）。  
**主要挑战**：高负债率、新冠冲击后恢复中、多国监管风险。  
**此测试验证**：工具4（双轨分离）、工具5（动态ERP/CRP）、§12.3 VR-03。

### 2.2 测试 Input Prompt

```
分析模式：MODE_FULL_DD

请对以下高杠杆博彩公司进行估值分析（仅供测试，example_only）：

公司概况：
- 名称：[假设公司 Grand Casino Holdings]
- 业务：澳门博彩（65%营收）+ 新加坡综合度假村（25%营收）+ 美国本土（10%营收）
- 财务指标（TTM）：
  - 营收：75亿美元
  - EBITDA：18亿美元
  - EBIT：8亿美元（含折旧22亿）
  - 净负债：92亿美元
  - 净负债/EBITDA：5.1x
  - 利息支出：4.2亿美元
  - 利息保障倍数（ICR）：EBIT/利息 = 8/4.2 = 1.9x
  - 公司债BB评级，近期债券交易利差约780bps
  - 澳门特区续牌许可证需在2028年到期续期

主要风险：
- 澳门营业执照续期不确定性
- 地缘政治（中美关系）对客流量影响
- 高杠杆在利率上行环境中的再融资风险

问题：
1. 请评估该公司的内在价值
2. 高杠杆风险应如何处理？
3. 澳门续牌不确定性如何量化？
```

### 2.3 预期触发的硬规则

| 规则 | 预期触发原因 | 预期输出 |
|------|------------|---------|
| **VR-03** | 净负债/EBITDA = 5.1x > 4 AND ICR = 1.9x < 3 | 强制执行双轨分离，标注禁止仅靠调高WACC |
| **C-014** | 澳门营收占比 65% > 30% + 地缘敏感区域 | 叠加澳门/中国CRP |
| **M-013（潜在）** | 若分析时2026年3月中东战争仍在持续 | 动态ERP须用最新值（4.77%而非历史均值） |

### 2.4 预期输出关键字段

**§12.3 双轨期望值**（必须出现）：
```
VR-03 触发：净负债/EBITDA = 5.1x > 4 AND ICR = 1.9x < 3
[MANDATORY] 执行双轨期望值模型

P_distress 计算（从市场数据反推，非主观估算）：
  BB债券交易利差：780bps
  假设回收率：40%（按bond_implied_default_rates.yml BB级别）
  P_distress_annual ≈ 780bps / (1 - 40%) = 13.0% / 年
  P_distress_cumulative_5y ≈ 1 - (1 - 13%)^5 = 49%
  合理性校验：参照 bond_implied_default_rates.yml，BB级别5年累计违约率~6.4%
  注意：当前利差大幅高于BB均值（意味着市场对该公司的评级有所怀疑，需说明）

V_going_concern：持续经营DCF价值 = [X]亿美元
V_liquidation：博彩物业清算价值（按重置成本55-65折） = [X]亿美元

V_total = V_going_concern × (1 - 49%) + V_liquidation × 49%
       = [X] × 51% + [X] × 49%
       = [X] 亿美元

禁止声明：本分析禁止仅靠将 WACC 从8%调高至18%来处理破产风险（VR-03合规）
```

**动态ERP/CRP**（必须出现）：
```
§14.3 动态风险校准：
  当期 Implied ERP（from implied_erp_history.yml）：4.65%（2026-04）
  澳门/中国 CRP（from sovereign_cds_snapshot.yml）：
    中国主权CDS：72bps → CRP ≈ 0.7%
  
  WACC 动态计算：
    无风险利率：4.40%（美国10Y国债）
    Beta：1.1（博彩行业）
    Implied ERP：4.65%
    中国CRP：0.7%（叠加C-014触发）
    特定风险溢价（续牌不确定）：0.8%
    Ke = 4.40% + 1.1 × 4.65% + 0.7% + 0.8% = 11.02%
```

### 2.5 失败判定

- ❌ 系统未触发 VR-03，仍使用单一 DCF（调高 WACC 处理破产风险）
- ❌ P_distress 使用主观估算而非债券利差反推
- ❌ 未引用 bond_implied_default_rates.yml 做合理性校验
- ❌ 动态 ERP 使用历史均值（4.6%）而非最新实时值

---

## 测试场景3：Cyclical 周期股常态化 — Exxon Mobil 类大型石油公司

### 3.1 测试背景

**标的类型**：成熟大型周期性大宗商品企业，利润随油价剧烈波动。  
**主要挑战**：景气周期末段，TTM 利润率处于历史高位，新兴市场能源转型压力。  
**此测试验证**：工具7（周期常态化）、VR-04、CYCLICAL模板、base_rates/cyclical_midcycle_margins.yml。

### 3.2 测试 Input Prompt

```
分析模式：MODE_VALUATION_REVIEW

请对以下大型石油公司进行估值分析（仅供测试，example_only）：

公司概况：
- 名称：[假设公司 Pacific Energy Corp]
- 业务：综合石油公司（上游E&P + 炼油 + 化工）
- 主营业务地区：北美（60%）+ 中东（25%）+ 亚太（15%）
- 财务指标（TTM，当前高油价景气期）：
  - 营收：4,200亿美元
  - EBIT：890亿美元（EBIT利润率 21.2%，近5年最高）
  - EBITDA：1,100亿美元
  - 净利润：650亿美元
  - 净负债：320亿美元（净负债/EBITDA = 0.29x）
  - 当前布伦特油价：95美元/桶（高于历史均值约40%）

分析师一致预期：
  - 卖方普遍给出 15x TTM P/E 的目标价
  - 依据当前 21% EBIT 利润率的 DCF 模型

中东业务风险：
  - 2026年3月中东战争爆发，对中东区块有直接地缘风险

问题：
1. 当前 21% EBIT 利润率是否可用于 DCF？
2. 正常化估值应该是多少？
3. 中东战争对 WACC 有何影响？
4. 卖方 15x TTM P/E 是否合理？
```

### 3.3 预期触发的硬规则

| 规则 | 预期触发原因 | 预期输出 |
|------|------------|---------|
| **VR-04** | CYCLICAL 模板 + TTM利润率21.2% vs 中周期均值约14% | 禁止使用TTM利润率；强制正常化 |
| **M-013** | 中东战争爆发后14天内（2026-03事件） | 动态ERP重算（4.37%→4.77%跳升） |
| **C-014** | 中东营收25% > 30%接近触发（分析师应核查实际占比） | 叠加中东地区CRP |

### 3.4 预期输出关键字段

**§VR-04 周期常态化**（必须出现）：
```
[VR-04 MANDATORY 触发]
CYCLICAL 模板：禁止使用 TTM 利润率 21.2% 做 DCF

周期常态化执行：
  方法：A（滚动均值）+ B（大宗商品价格回归）双验证
  
  方法A（10年滚动均值）：
    周期窗口：2014-2023年（10年）
    来源：cyclical_midcycle_margins.yml，Integrated Oil & Gas
    中周期EBIT利润率中值：14.0%（区间 8-20%）
    中点取法：算术均值
    
  方法B（布伦特价格回归）：
    回归方程：EBIT_Margin = α + β × Brent_Price
    β（估计）≈ 0.35（即布伦特每变动10美元，利润率变动约3.5pp）
    中周期布伦特假设：65美元/桶（IEA长期均衡价格）
    对应正常化EBIT利润率 = [基线] + 0.35 × (65 - 55) = 约 13-15%
    
  当前TTM利润率：21.2%
  正常化利润率（双方法均值）：约 13.5-14.5%
  差异：约 -7pp（当前明显处于景气顶部）
  
  正常化EBIT = 4,200亿 × 14.0% = 588亿美元（vs TTM 890亿）
  估值调整：按正常化EBIT的DCF结论将显著低于卖方的TTM P/E估值

必须披露：
  □ 周期窗口：2014-2023（10年）
  □ 中点取法：算术均值
  □ TTM 21.2% vs 正常化 14.0%，差值 -7.2pp
  □ 当前处于景气晚期（高利润+高油价+高Capex），触发C-009
```

**§14.3 动态ERP校准**（必须出现）：
```
M-013 触发：2026年3月中东战争，WACC须重算

Implied ERP 更新：
  2026-02 ERP：4.37%（战前）
  2026-03 ERP：4.77%（战时，+40bp，来自 implied_erp_history.yml）
  2026-04 ERP：4.65%（当前）
  
  禁止使用历史均值 ERP = 4.60%（M-013触发后不适用）

中东 CRP（若中东营收 > 30%触发 C-014）：
  Saudi Arabia CDS：68bps（战时）→ CRP ≈ 0.8%
  Iraq CDS（直接交战区）：480bps → CRP ≈ 6.5%
  加权CRP（按中东业务地区分布）：[X]%
  
  WACC_adjusted = WACC_base + CRP_middle_east
```

**§卖方研报检测**（推荐出现）：
```
Pricing Gymnastics 检测（卖方 15x TTM P/E）：
  问题：使用当前景气顶峰的 TTM P/E（65.0/4.42 ≈ 14.7x）看似合理
  但实质上锁定了顶峰盈利作为分母，属于"峰值P/E陷阱"
  公平性测试：若使用正常化EPS（约590亿×(1-21%)÷股数），则P/E将升至约21-25x
  结论：卖方 15x TTM P/E 实质上是用顶峰盈利掩盖了高估，建议标记为潜在定价体操
```

### 3.5 失败判定

- ❌ 系统使用 TTM 利润率 21.2% 做 DCF 终值假设（VR-04 违反）
- ❌ WACC 使用2026年2月前的历史均值 ERP（未执行 M-013 重算）
- ❌ 未引用 cyclical_midcycle_margins.yml 对应行业区间
- ❌ 未披露周期窗口年份和中点取法
- ❌ 卖方 TTM P/E 估值被系统直接采纳（未检测峰值P/E陷阱）

---

## 附：测试结果记录模板

```yaml
# V5.3 测试记录模板
test_execution_log:
  deployment_date: "YYYY-MM-DD"
  gem_version: "V5.3 Damodaran-Calibrated"
  deployment_mode: "完整集（方案A）| 最小集（方案B）"
  
  test_1_spacex:
    status: "PASS | FAIL | PARTIAL"
    vr01_triggered: boolean
    vr02_triggered: boolean
    vr05_triggered: boolean
    sotp_lines_identified: int   # 预期 ≥3
    monte_carlo_percentiles_present: boolean
    expansion_option_separately_listed: boolean
    issues_found: string
    
  test_2_lvs_distressed:
    status: "PASS | FAIL | PARTIAL"
    vr03_triggered: boolean
    p_distress_from_market_data: boolean   # 非主观估算
    dual_track_formula_applied: boolean
    dynamic_erp_used: boolean
    issues_found: string
    
  test_3_exxon_cyclical:
    status: "PASS | FAIL | PARTIAL"
    vr04_triggered: boolean
    normalized_margin_used: boolean
    cycle_window_disclosed: boolean
    m013_erp_recalculated: boolean
    issues_found: string
    
  overall_result:
    all_tests_passed: boolean
    critical_failures: list[string]
    recommendations: string
```

---

*V5.3 Damodaran-Calibrated 测试用例文档 | 创建日期：2026-04-28*
