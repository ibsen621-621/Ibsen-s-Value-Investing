# Agent-8: Calibration Auditor（校准审计师）

> 【V5.2 新增】定位：系统级元评估，不参与单次分析评分，不具备否决权。  
> 【V5.2 新增】执行层级：Layer 6（Agent-O 之后，跨报告周期异步执行）。

## 1) Contract Interface

```yaml
CONTRACT: Agent-8 Calibration Auditor
version: 5.2
agent_id: agent_8_calibration_auditor
agent_role: 预测校准与历史回测
rule_level: MANDATORY
veto_power: false

INPUT_SCHEMA:
  required:
    - predictions_log: "predictions_log.jsonl"
    - realized_outcomes: dict
    - evaluation_horizons_months: [6, 12, 24, 36]
  optional:
    - segment_filters:
        market: [US, CN_A, HK]
        sector: string
        rating_bucket: [BUY, HOLD, SELL, TERMINATE]

OUTPUT_SCHEMA:
  required:
    - calibration_report_id: string
    - window_start: date
    - window_end: date
    - brier_score: dict
    - calibration_curve: dict
    - hit_rate_by_decile: list
    - confidence_drift: dict
    - recommended_shrinkage_factor: float
    - threshold_adjustments: dict
    - counterfactual_review_summary: dict
```

## 2) Mandatory Computations

```yaml
MANDATORY_COMPUTATIONS:
  - compute_brier_score_by_horizon
  - compute_calibration_curve_10_bins
  - compute_hit_rate_by_decile
  - compute_kill_criteria_precision_recall
  - classify_terminate_avoid_counterfactuals
  - generate_dynamic_shrinkage_recommendation
```

## 3) Prediction Record Spec（报告端必须输出）

```yaml
prediction_record:
  report_id: string
  ticker: string
  report_date: date
  rating: string
  confidence_score: 0-100
  intrinsic_value_range:
    bear: float
    base: float
    bull: float
  probabilities:
    bear: 0-1
    base: 0-1
    bull: 0-1
  kill_criteria_thresholds:
    - code: string
      metric: string
      threshold: string
      frequency: string
  expected_horizon_months: 36
  key_assumptions: [string, string, string]
```

## 4) Persistence Protocol

```text
file: predictions_log.jsonl
append_mode: true
record_format: one-json-per-line
idempotency_key: report_id
```

示例：
```json
{"report_id":"rpt-2026-0001","ticker":"AAPL","report_date":"2026-04-21","prediction_record":{"rating":"BUY","confidence_score":68}}
```

## 5) Metrics Definitions

- **Brier Score**：衡量概率预测误差，越低越好。  
- **Calibration Curve**：比较预测概率与实际命中率的偏离。  
- **Hit Rate by Decile**：按置信度十分位统计命中率，验证排序有效性。  

## 6) Feedback to Tool 6（动态收缩）

```text
shrinkage_t = clamp(0.05, 0.35, 0.10 + 0.8 × calibration_error_t)
p_calibrated = 0.5 + (p_raw - 0.5) × (1 - shrinkage_t)
```

说明：`calibration_error_t` 默认取滚动窗口 Brier 相对基准误差（待校准）。

## 7) Counterfactual Review for TERMINATE/Avoid

```yaml
counterfactual_review:
  horizons_months: [6, 12]
  labels:
    - TRUE_NEGATIVE
    - FALSE_NEGATIVE
  output:
    - action_recommendation: [loosen, maintain, tighten]
```

