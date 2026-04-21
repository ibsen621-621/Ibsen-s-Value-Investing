# examples 目录说明

> 【V5.2 新增】本目录用于沉淀可复用的完整分析样本（含后验验证）。

## 建议提交内容

每个样本建议包含：

1. `input/`：输入问题、分析模式、数据截止日
2. `report/`：完整输出（含 Agent 分段结果）
3. `prediction_record.yml`：结构化预测记录
4. `review_6m.md` / `review_12m.md`：6/12个月后回顾

## 最小字段规范

```yaml
report_id: string
ticker: string
report_date: YYYY-MM-DD
mode: MODE_FULL_DD | MODE_VALUATION_REVIEW | MODE_PORTFOLIO_MONITOR
rating: string
confidence_score: 0-100
intrinsic_value_range: {bear: float, base: float, bull: float}
probabilities: {bear: float, base: float, bull: float}
expected_horizon_months: 36
```

## 贡献要求

- 保留原始输入与输出，不做事后“美化”
- 必须注明数据来源与数据截止日
- 若发现误判，鼓励附上复盘与阈值修订建议

