# VIAIOS-P V5.4 — Edge-Driven

> Compliant with DOCTRINE-001-Edge-over-Logic.
> Inherits all of VIAIOS-P-V5.3-Damodaran-Calibrated unless explicitly overridden below.
> Status: Specification (implementation pending).

---

## Document Overview

This document is an **incremental specification** on top of `VIAIOS-P-V5.3-Damodaran-Calibrated.md`. It does not replace or rewrite V5.3. Every module, agent, rule, and output format in V5.3 remains in effect unless a patch below explicitly overrides or wraps it. Read V5.3 first.

**Why V5.4 exists**: V5.3 was optimized along the *logic axis* — better calibration, better Damodaran tools, better agent decomposition. DOCTRINE-001 establishes that logic is necessary but not sufficient. V5.4 implements the eight structural changes the doctrine requires: edge tagging, pool concentration, ignition detection, regime alarms, falsifiable theses, edge scoring, patience pricing, and the human/machine boundary.

**Scope**: Specification only. No Python, no agent implementations under `agents/`. The `edge_registry/` directory is created with schema documentation only; no real ticker data.

**Binding order**: DOCTRINE-001 > V5.4 patches > V5.3 base modules.

---

## AI / Human Boundary Charter (Operations Preamble)

The following table is reproduced verbatim from DOCTRINE-001 §2 and is binding on all agents operating under V5.4. It appears here at the top of the operations section per Patch 8's requirement.

| Task | Owner | Rationale |
|---|---|---|
| Mass scanning of public filings, ratios, multiples | **AI** | Speed, objectivity, no fatigue |
| Anomaly detection across thousands of tickers | **AI** | Bandwidth |
| Templated valuation (Damodaran-style DCF, EV/EBITDA bands) | **AI** | Mechanical |
| Detection of *candidate* ignition signals from public data | **AI** | Pattern recognition |
| **Verification of ignition** (calls, channel checks, expert dialogue) | **Human** | Non-replicable |
| Judgement on management quality and capital allocation intent | **Human** | Requires relationship + context |
| Frontier-paper / pre-commercial tech leading indicators | **Human** (with AI assist) | Requires domain taste |
| **Final buy / hold / exit decision** | **Human** | Accountability + edge integration |
| Position sizing | **Human**, AI provides structured memo | Edge × asymmetry × conviction is irreducibly judgmental |

**Runtime enforcement**: If any agent produces output for a task marked **Human** in the Owner column above (e.g., final buy/exit decision, management quality judgement), the system MUST:
1. Mark the output block `advisory_status: ADVISORY-ONLY`.
2. Leave the `recommendation` field unpopulated (null / omitted).
3. Emit a `charter_violation_warning` citing the offending task row.

---

## Patch 1 — Alpha Source Tagging

### Rationale (P2)

DOCTRINE-001 P2 states: *"Every actionable recommendation must be tagged with an Alpha Source that is not replicable by a generic API + LLM pipeline."* V5.3 produces recommendations without any structural attestation of edge provenance. This patch makes edge attestation a hard gate: without a valid `alpha_source` block, the maximum permissible recommendation is `TRACK`.

### Spec

Every output record for every monitored name must carry an `alpha_source` block conforming to the following schema:

```yaml
alpha_source:
  type: <PROBABILISTIC | INFORMATIONAL | CATALYST>
  edge_evidence: <string>       # One concrete sentence. Must name a specific, non-public signal source.
                                # Acceptable: "Supply chain contact at upstream tier-2 supplier confirmed order restart Q3."
                                # NOT acceptable: "Valuation screen shows EV/EBITDA below 5-year average."
  ai_replicable: <bool>         # true = a generic LLM + API pipeline could produce this same signal.
```

**Hard cap rule**: If `ai_replicable: true`, the system MUST downgrade `recommendation` to `TRACK` at most, regardless of APR or any other score. This cap is a one-way ratchet: no downstream module may override it.

**Type definitions**:
- `PROBABILISTIC`: Edge is a base-rate or structural insight that most participants are not applying (e.g., informed use of Damodaran reinvestment models). Weakest type; `ai_replicable` is usually `true` for this type, which typically caps at `TRACK`.
- `INFORMATIONAL`: Edge derives from a specific information source unavailable to the general market (expert network call, management meeting, supply-chain channel check). Strongest type; `ai_replicable` must be `false`.
- `CATALYST`: Edge is a known but un-priced upcoming event (regulatory decision, patent expiry, contract renewal). Medium type; `ai_replicable` may be `false` if the catalyst interpretation is non-obvious.

### Integration Point

Amends V5.3 **§9 最终输出格式** (Final Output Format). The `alpha_source` block is appended to every output record produced by Agent-6 and Agent-7. Agent-7 (综合评级引擎) is responsible for enforcing the `ai_replicable: true → TRACK` cap before emitting the final recommendation.

### Acceptance Criteria

- No output record with `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}` exists where `alpha_source.ai_replicable = true`.
- Every output record above `WAIT` carries a non-null `alpha_source.edge_evidence` string of ≥ 10 words.
- Removing the `alpha_source` block from any record and re-running Agent-7 must produce `TRACK` as the maximum recommendation.
- `alpha_source.type` is always one of the three enumerated values; no free-form strings accepted.

---

## Patch 2 — 30-Name Active Conviction Pool Cap

### Rationale (P5)

DOCTRINE-001 P5: *"30 names tracked deeply is enough. Adding requires evicting."* V5.3 has no structural enforcement of a pool ceiling. Attention dilution is a first-order failure mode; the spec must make exceeding 30 names mechanically impossible.

### Spec

```yaml
active_conviction_pool:
  max_size: 30
  current_size: <int>           # Must be ≤ 30 at all times.
  names: <list[ticker]>

pool_event:
  event_type: <ADD | EVICT | PROMOTE | DEMOTE>
  ticker: <symbol>
  timestamp: <ISO-8601>
  reason: <string>
  eviction_candidate: <symbol>  # Required when event_type = ADD
```

**Eviction protocol** (when `current_size = 30` and an ADD is requested):

The eviction candidate is selected by the following priority rule, applied in order:

1. **Lowest `edge_score`** (see Patch 6). If tie, proceed to rule 2.
2. **Lowest `APR`** (Asymmetric Payoff Ratio from the Output Contract). If tie, proceed to rule 3.
3. **Oldest unverified falsification checkpoint** (see Patch 5). The name with the longest elapsed time since its most recent passed checkpoint is evicted.

An eviction event must be logged as a `pool_event` record with `event_type: EVICT` before the ADD event is logged.

**Replacement memo**: Each ADD event must include a one-paragraph `replacement_rationale` field explaining why the incoming name's edge is superior to the evicted name's edge at the time of replacement.

### Integration Point

Wraps V5.3 **§4 意图路由器** (Intent Router) Universe discovery module and the shortlist construction logic in Agent-5. The pool ceiling check is a pre-condition gate: if `current_size = 30`, the system MUST run the eviction protocol before routing any new name to Agent-1 for full analysis. Passing a 31st name to the analysis pipeline without eviction is a protocol violation.

### Acceptance Criteria

- `active_conviction_pool.current_size` is never > 30 in any system state snapshot.
- Every ADD event in the `pool_event` log is preceded by an EVICT event in the same or prior transaction when `current_size` was 30 before the ADD.
- `replacement_rationale` field is non-null for every ADD event.
- Eviction candidates are always selected per the three-rule priority order; no manual override without a logged `charter_violation_warning`.

---

## Patch 3 — IgnitionScanner Agent

### Rationale (P4)

DOCTRINE-001 P4: *"Industries Ignite Discretely... Pre-ignition monitoring beats post-ignition valuation."* V5.3 provides cycle-position scoring (ADVANCED-MFG, CYCLICAL templates) but no mechanism for detecting the discrete ignition event itself. This patch introduces a dedicated agent that watches for coordinated multi-signal state changes across an industry, before valuation becomes relevant.

### Spec

**Agent name**: `IgnitionScanner` (operates as a parallel watchlist agent; not part of the standard 9-agent V5.3 DAG for individual name analysis).

**Inputs**:

```yaml
ignition_scanner_input:
  industry_id: <string>           # e.g., "optical_module", "pcb_tier1", "solid_state_battery"
  universe_top_n: 20              # Top 20 names in the industry by market cap or coverage density
  silence_index_window_years: <int>  # Rolling N years for silence baseline (recommend 3)

  silence_index_signals:          # Coverage and volume proxies (all normalized 0–1 vs. 5y baseline)
    - sell_side_note_count_zscore: <float>   # Z-score of trailing-3M sell-side note volume
    - equity_volume_zscore: <float>          # Z-score of average daily trading volume
    - job_posting_count_zscore: <float>      # Z-score of hiring JD count (industry-wide, LinkedIn/Indeed)
    - media_mention_zscore: <float>          # Z-score of news/media mentions (public sources only)

  live_signals:                   # Binary flags; each independently observable
    - order_book_delta_positive: <bool>      # Net order book imbalance turned positive vs. trailing avg
    - spot_price_inflection: <bool>          # Spot price crossed above 200-day moving avg
    - capex_guidance_raised: <bool>          # ≥ 2 industry names raised capex guidance in trailing 90d
    - hiring_surge: <bool>                   # job_posting_count_zscore > 2.0
    - paper_citation_surge: <bool>           # Academic paper citation volume in sector > 2σ above baseline
    - supply_chain_lead_time_extension: <bool>  # Lead-time data (if available) extended vs. baseline
```

**Trigger rule**:

```
ignition_alert = true  IFF  count(live_signals where value == true) >= 3
                        AND all triggering signals occurred within a rolling 90-day window
```

**Output**:

```yaml
ignition_scanner_output:
  industry_id: <string>
  scan_date: <ISO-8601>
  ignition_alert: <bool>
  signals_active: <list[string]>       # Names of live_signals that are true
  signals_active_count: <int>
  signal_window_days: <int>            # Days between earliest and latest active signal
  watchlist_event:
    type: <PRE_IGNITION_MONITOR | IGNITION_CONFIRMED | FALSE_POSITIVE_REVIEW>
    priority: <HIGH | MEDIUM | LOW>
    human_action_required: "Conduct channel checks to verify ignition. AI signals are candidate only."
  valuation_note: >
    This output is a watchlist event only. No valuation has been performed.
    Per DOCTRINE-001 P4, pre-ignition tracking takes priority over post-ignition valuation.
    Route to human for verification before initiating full V5.3 analysis pipeline.
```

**Explicit constraint**: `IgnitionScanner` output MUST NOT contain a `recommendation` field, a DCF estimate, or any price target. It is a routing and alerting mechanism only. The human operator decides whether to route an ignition alert into the standard V5.3 pipeline.

### Integration Point

New agent; does not replace any existing V5.3 agent. Operates as a scheduled background watchlist scanner (e.g., weekly sweep of industry universes). When `ignition_alert: true`, the output feeds into the pool eviction decision in Patch 2: the alerted industry becomes a high-priority ADD candidate, subject to the 30-name cap.

### Acceptance Criteria

- `IgnitionScanner` never emits a `recommendation` field.
- `ignition_alert: true` requires exactly ≥ 3 live signals, all within a 90-day window; if only 2 signals are active, `ignition_alert` must be `false` regardless of their magnitude.
- Output always contains `human_action_required` text directing verification to a human.
- `signal_window_days` must be ≤ 90 when `ignition_alert: true`.

---

## Patch 4 — Reinvestment Regime Shift Alarm

### Rationale (P3)

DOCTRINE-001 P3: *"Tencent gets more expensive as it falls if Capex regime shifts and historical FCF is invalidated."* V5.3's Damodaran DCF module (Agent-9, §11.5 Monte Carlo) operates on reinvestment assumptions but does not actively detect when the historical reinvestment baseline has been structurally broken by current behavior. This patch adds a continuous monitoring alarm that invalidates any DCF extrapolating historical FCF when a regime shift is detected.

### Spec

**Per-name tracking** (appended to each name's data record):

```yaml
reinvestment_regime:
  ticker: <symbol>
  capex_ttm: <float>                    # Trailing-12-month capex (USD millions)
  capex_5y_avg: <float>                 # 5-year average annual capex (USD millions)
  capex_ratio: <float>                  # = capex_ttm / capex_5y_avg
  consecutive_quarters_above_1_5x: <int>  # Quarters where capex_ratio > 1.5
  regime_shift_flag: <bool>             # true if consecutive_quarters_above_1_5x >= 2
  last_updated: <ISO-8601>
```

**Trigger conditions and consequences**:

| Condition | Action |
|---|---|
| `capex_ratio > 1.5` for ≥ 2 consecutive quarters | Set `regime_shift_flag: true` |
| `regime_shift_flag: true` | Invalidate any DCF using historical FCF extrapolation; mark those DCF outputs as `dcf_status: INVALID-REGIME-SHIFT` |
| `regime_shift_flag: true` | Force re-DCF using: `Forward FCF = (NOPAT × (1 − new_reinvestment_rate)) where NOPAT = forward EBIT × (1 − tax_rate)` and `new_reinvestment_rate = capex_ttm / (NOPAT + D&A)` |
| `regime_shift_flag: true` | Downgrade the safety-margin score component labeled `low_pe_safety_bonus` by one tier (HIGH → MEDIUM, MEDIUM → LOW, LOW → NONE) |

**Re-DCF required inputs** (Damodaran style, referencing V5.3 Agent-9):

```yaml
regime_shift_redcf:
  forward_ebit: <float>           # Consensus or internally modeled
  tax_rate: <float>
  new_reinvestment_rate: <float>  # Computed from capex_ttm as above
  forward_roic: <float>           # Analyst/model estimate; must be flagged as ESTIMATE
  wacc: <float>                   # From V5.3 Agent-9 §14.3 Dynamic ERP calibration
  dcf_status: REGIME-SHIFT-REDCF  # Distinguishes from normal V5.3 DCF
```

### Integration Point

Amends V5.3 **Agent-9 Damodaran Engine** (§11.5 Monte Carlo Valuation Distribution and §8.4 Restated Financials). The regime check runs as a pre-step before any DCF is initiated. If `regime_shift_flag: true`, the historical FCF path in the Monte Carlo is disabled; only the re-DCF path using `new_reinvestment_rate × forward_ROIC` is valid.

Also amends V5.3 **§12.3 Distress-Adjusted Valuation**: when `regime_shift_flag: true`, the going-concern value (`V_going_concern`) in the dual-track model must be sourced from the regime-shift re-DCF, not the historical FCF DCF.

### Acceptance Criteria

- No output record has `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}` where `regime_shift_flag: true` and the DCF was computed from historical FCF (i.e., `dcf_status = INVALID-REGIME-SHIFT` records must not be used as valuation anchors).
- Re-DCF using `new_reinvestment_rate` is present in the output whenever `regime_shift_flag: true`.
- `low_pe_safety_bonus` in the safety-margin score is downgraded when `regime_shift_flag: true`; spot-check with a name where PE is low but capex ratio > 1.5x for 3 consecutive quarters.
- `consecutive_quarters_above_1_5x` counter resets to 0 when `capex_ratio` drops below 1.5x for any quarter.

---

## Patch 5 — Verification Checkpoint Ledger

### Rationale (P1, P3)

DOCTRINE-001 P1: *"What you can control is risking $1 to make $100."* The asymmetry only holds if the thesis has a defined failure mode. P3 adds that cheap is not safe; regime shifts must be caught. Both propositions require that every holding thesis be expressed as a falsifiable hypothesis, not a narrative. This patch mandates a structured checkpoint ledger that forces the operator to pre-commit to observable exit signals.

### Spec

Every name in the Active Conviction Pool must carry a `holding_thesis` block:

```yaml
holding_thesis:
  ticker: <symbol>
  core_assumption: <string>     # One sentence. The single most critical thing that must be true
                                # for the investment to work. Not a story. Not a trend description.
                                # Example: "Utilization rate at Fab A returns to ≥ 80% by Q3 2027."
  falsification_checkpoints:
    - checkpoint_id: CP-001
      at: T+3M                  # One of: T+3M, T+6M, T+12M, T+18M, T+24M
      due_date: <ISO-8601>      # Absolute date derived from position initiation date
      metric: <string>          # Observable, publicly reportable metric.
                                # Example: "Fab A quarterly utilization rate (from earnings call)"
      fail_threshold: <string>  # Value at which this checkpoint is failed.
                                # Example: "< 70% utilization for two consecutive quarters"
      status: <PENDING | PASSED | FAILED>
      actual_value: <string>    # Populated at due_date; null until then

    - checkpoint_id: CP-002     # Minimum 2 checkpoints required per thesis
      at: T+6M
      due_date: <ISO-8601>
      metric: <string>
      fail_threshold: <string>
      status: <PENDING | PASSED | FAILED>
      actual_value: <string>

  exit_trigger: <string>        # Explicit rule. Example: "Any 2 checkpoints FAILED → reduce position 50%.
                                # All 3 checkpoints FAILED → full exit within 5 trading days."

  narrative_prohibition: >
    Stories are not allowed in this block. Only time-stamped, falsifiable hypotheses with
    observable metrics and pre-committed fail thresholds. Violation of this rule invalidates
    the entire holding_thesis block and triggers a re-evaluation request.
```

**Minimum requirements**:
- `core_assumption`: 1 sentence, present-tense, contains a measurable outcome.
- `falsification_checkpoints`: ≥ 2 entries; first checkpoint at T+3M or T+6M maximum.
- `exit_trigger`: explicit quantitative rule referencing checkpoint outcomes.

**Prohibition**: The words "narrative," "story," "theme," "trend," and "secular growth" used as sole evidence in `core_assumption` or `fail_threshold` trigger a validation error. They may appear in supporting context only.

### Integration Point

New data structure appended to each Active Pool name record. Wraps V5.3 **§9 最终输出格式**: the final output for any name in the Active Pool must include the `holding_thesis` block. Agent-7 (综合评级引擎) validates that both `core_assumption` and ≥ 2 `falsification_checkpoints` are populated before emitting `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}`.

Also integrates with V5.3 **§8 置信度校准与回测**: failed checkpoints feed into the prediction record (`prediction_record` in V5.3) as logged outcomes for Brier Score calibration.

### Acceptance Criteria

- No `BUY-CANDIDATE` or `SIZE-UP` recommendation exists for any name without a fully populated `holding_thesis` block.
- Every `holding_thesis` has ≥ 2 `falsification_checkpoints` with non-null `due_date`, `metric`, and `fail_threshold`.
- `exit_trigger` references at least one checkpoint outcome (e.g., "If CP-001 FAILED AND CP-002 FAILED").
- A name where 2 checkpoints have `status: FAILED` must have its `recommendation` automatically downgraded to `TRIM` or `EXIT` per the `exit_trigger` rule.
- System rejects any `holding_thesis` where `core_assumption` consists solely of narrative language without a measurable outcome.

---

## Patch 6 — Information Edge Registry

### Rationale (P2)

DOCTRINE-001 P2: *"Advance only where [AI] cannot."* The edge types enumerated in the doctrine (primary expert calls, management access, supply-chain contacts, frontier paper tracking) must be tracked as a persistent, queryable asset — not self-reported per analysis. This patch creates a static registry that decays edge scores with staleness, gating heavy positions behind demonstrated active edge.

### Spec

**Registry location**: `edge_registry/` directory at the repository root. One YAML file per ticker or industry (`edge_registry/{TICKER}.yaml` or `edge_registry/{INDUSTRY_ID}.yaml`). Schema documentation: `edge_registry/README.md`.

**File schema**:

```yaml
# edge_registry/{TICKER}.yaml
ticker: <symbol>
last_registry_update: <ISO-8601>

edge_channels:
  - channel_id: EC-001
    type: <industry_expert_call | management_access | paper_tracking |
           supply_chain_contact | channel_check | regulatory_contact | other>
    description: <string>       # Non-public; one sentence on the nature of the contact/source.
    last_contact: <ISO-8601>    # Date of most recent signal from this channel.
    freshness_decay_days: <int> # Number of days after which this channel's contribution to
                                # edge_score decays to zero (see decay formula below).
    current_contribution: <float>  # 0.0–1.0; computed from decay formula.

edge_score: <float>             # Computed field: sum of current_contribution × channel_weight,
                                # normalized to 0–10 scale. Recomputed on every registry update.

edge_score_floor_for_heavy_position: 4.0   # Hard constant; do not modify without a patch note.
```

**Edge score decay formula**:

```
days_since_contact = (today - last_contact).days
decay_factor = max(0.0, 1.0 - (days_since_contact / freshness_decay_days))
current_contribution = base_contribution × decay_factor
```

**Channel weight defaults** (can be overridden per registry file with justification):

| Channel Type | Base Contribution | Default Freshness Decay (days) |
|---|---|---|
| `management_access` | 1.0 | 90 |
| `supply_chain_contact` | 0.9 | 60 |
| `industry_expert_call` | 0.8 | 90 |
| `channel_check` | 0.7 | 45 |
| `regulatory_contact` | 0.7 | 120 |
| `paper_tracking` | 0.5 | 180 |
| `other` | 0.3 | 60 |

**Hard rule**: `edge_score < 4.0` → the name CANNOT enter the heavy-position tier (portfolio weight > 5%). Agent-7 enforces this gate before emitting any `SIZE-UP` recommendation.

**Position tier → edge_score gate**:

| Position Tier | Max Portfolio Weight | Minimum edge_score |
|---|---|---|
| Heavy | > 5% | ≥ 4.0 |
| Standard | 2%–5% | ≥ 2.0 |
| Tracking | < 2% | ≥ 0.0 (no gate) |

### Integration Point

New directory and static asset. Integrates with V5.3 **Agent-6** (估值综合) and **Agent-7** (综合评级引擎): before Agent-7 emits a recommendation, it reads the `edge_score` from the relevant registry file. If the file does not exist, `edge_score` defaults to 0.0, which caps the recommendation at `TRACK` and blocks `SIZE-UP`.

Also integrates with **Patch 2** (eviction protocol): `edge_score` is the primary eviction sort key.

### Acceptance Criteria

- Every name in the Active Pool has a corresponding `edge_registry/{TICKER}.yaml` file; missing file → `edge_score = 0.0` → no recommendation above `TRACK`.
- `SIZE-UP` never appears in output for a name with `edge_score < 4.0`.
- Edge score decays correctly: a name with only `paper_tracking` contact (decay 180 days), last contacted 90 days ago, must show `current_contribution = 0.5 × (1 − 90/180) = 0.25`, yielding a reduced `edge_score`.
- Modifying `last_contact` to a date > `freshness_decay_days` ago must set `current_contribution = 0.0` for that channel.

---

## Patch 7 — Patience Premium / Idle Cash Legitimization

### Rationale (P5, WAIT default)

DOCTRINE-001 P5 and §4: *"WAIT is a first-class recommendation. Idle cash carries an explicit option value that the system must price."* V5.3 does not model the `WAIT` state as a structured output with its own memo format. This patch makes `WAIT` as rigorous as `BUY-CANDIDATE`: it must document what was scanned, what failed the gate, and what would change the answer.

### Spec

**Global gate condition**:

```
WAIT_DEFAULT = true  IFF  NOT EXISTS (name IN active_conviction_pool
                           WHERE apr >= 50 AND edge_score >= 4.0)
```

When `WAIT_DEFAULT = true`, the system output MUST be `WAIT` accompanied by a structured `wait_memo`:

```yaml
system_output:
  recommendation: WAIT
  wait_memo:
    scan_date: <ISO-8601>
    universe_scanned: <int>          # Number of names reviewed in this scan cycle
    active_pool_size: <int>
    gate_failures:
      apr_below_50_count: <int>      # Names that failed APR ≥ 50 gate
      edge_score_below_4_count: <int>  # Names that failed edge_score ≥ 4 gate
      both_gates_failed_count: <int>   # Names failing both gates simultaneously
    closest_to_gate:
      - ticker: <symbol>
        apr: <float>
        edge_score: <float>
        gap_to_apr_gate: <float>      # = 50 - apr (positive = how far below gate)
        gap_to_edge_gate: <float>     # = 4.0 - edge_score (positive = how far below gate)
        what_would_change: <string>   # One sentence on the single event most likely to
                                      # flip this name through both gates.
    patience_premium_note: >
      Cash is not idle; it holds an option on the next once-per-decade dislocation.
      This WAIT is a considered position, not an absence of work. The above names are
      tracked and will be re-evaluated on each scan cycle.
    next_scan_date: <ISO-8601>
```

**Agent penalty rule**: Any agent that emits `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}` without the global gate condition being satisfied (APR ≥ 50 AND edge_score ≥ 4) must have its output flagged with `gate_violation: true` and the recommendation overridden to `TRACK` by Agent-7.

**Modeling the option value of cash** (qualitative, to be formalized in V5.5):

The `wait_memo` may include an optional `cash_option_value_note` field describing the current opportunity cost structure: risk-free rate, implied ERP (from V5.3 §14.3), and whether any name in the `closest_to_gate` list is within one catalyst event of crossing the APR gate.

### Integration Point

Amends V5.3 **§9 最终输出格式** and **Agent-7**: the final output pipeline must evaluate the global gate before routing to a buy/sell recommendation. If the gate is unmet, `WAIT` with the structured memo is the mandatory output.

Also amends V5.3 **§4 意图路由器** `MODE_PORTFOLIO_MONITOR`: when running in portfolio monitor mode, the global gate check is executed first; a portfolio with no name meeting both gates generates a `WAIT` signal for the undeployed cash sleeve.

### Acceptance Criteria

- When all names in the Active Pool have `apr < 50` OR `edge_score < 4`, system output for the cash allocation recommendation is `WAIT` with a populated `wait_memo`.
- `wait_memo.gate_failures` fields are non-null and accurate (sum correctly across the pool).
- `wait_memo.closest_to_gate` contains ≥ 1 entry (the name closest to passing both gates).
- Any agent output with `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}` that was produced without gate satisfaction is marked `gate_violation: true` and overridden in Agent-7's consolidation step.
- `WAIT` outputs contain a `next_scan_date` field.

---

## Patch 8 — AI vs Human Boundary Charter

### Rationale (Section 2 of DOCTRINE-001)

DOCTRINE-001 §2 establishes a binding Human/Machine Charter defining which tasks belong to AI and which to human judgment. The charter is reproduced at the top of this document (Operations Preamble). This patch adds the runtime enforcement mechanism so that it is not merely a statement of intent.

### Spec

**Runtime boundary check**: Before any agent emits a final output, it MUST evaluate whether the output includes content for a task row marked **Human** in the charter table.

**Human-row task identifiers** (these string patterns trigger the check):

```yaml
human_boundary_triggers:
  - "final_buy_decision"
  - "final_exit_decision"
  - "final_hold_decision"
  - "management_quality_score"          # Any automated scoring of management quality
  - "capital_allocation_intent_score"   # Any automated scoring of management intent
  - "position_size_recommendation"      # Hard sizing (as opposed to a structured memo)
  - "ignition_verification_confirmed"   # Verification (not candidate detection) of ignition
```

**Enforcement action**: When a human-boundary trigger is detected in an agent's output:

```yaml
boundary_enforcement:
  advisory_status: ADVISORY-ONLY
  recommendation: null              # Field must be omitted or explicitly null; never populated
  charter_violation_warning: >
    This output touches a task row designated as Human-owner in the DOCTRINE-001
    Human/Machine Charter. The system is providing advisory context only.
    No recommendation has been populated. The human operator must make this determination.
  triggering_field: <string>        # Name of the field that triggered the boundary check
  triggering_charter_row: <string>  # Verbatim text of the charter row that was violated
```

**Structured advisory memo** (what the agent CAN produce for Human rows):

```yaml
advisory_memo:
  topic: <string>              # e.g., "Management quality indicators"
  observable_proxies:          # AI-observable proxies only; no inferred intent
    - proxy: <string>          # e.g., "Insider buying in trailing 12M"
      data: <string>
      interpretation_note: "Proxy only. Final judgement requires relationship and context (Human)."
  structured_data_for_human: <any>   # Structured data the human can use; labelled as input, not output
```

### Integration Point

Amends all V5.3 agents, particularly **Agent-2** (管理层尽调, management diligence) and **Agent-6** (估值综合, which feeds the final recommendation). Agent-2's management quality outputs must all carry `advisory_status: ADVISORY-ONLY` and must not contribute a numeric score to `recommendation` directly; they provide `observable_proxies` for human review only.

Agent-7 (综合评级引擎) must implement the boundary check as a final filter before emitting its output: scan output for `human_boundary_triggers`; apply enforcement action if any trigger is found.

### Acceptance Criteria

- Agent-2 outputs do not contain a `recommendation` field; they contain only `advisory_memo` with `observable_proxies`.
- Agent-7 output for any name includes `advisory_status: ADVISORY-ONLY` wherever management quality or position sizing inputs from human rows were referenced.
- Injecting a test payload that sets `management_quality_score: 9` into Agent-7's input causes the output `recommendation` field to be null and `charter_violation_warning` to be populated.
- No output contains a hard position size (e.g., "Buy 8% of portfolio") without the human-review flag; at most, a `position_size_memo` with scenario analysis is permitted.

---

## Compatibility Matrix

The following table maps every named V5.3 module to its V5.4 status. Modules not listed here are inherited unchanged.

| V5.3 Module / Section | V5.4 Status | Patch Reference |
|---|---|---|
| §1 系统核心协议 (System Core Protocol) | Inherited unchanged | — |
| §2 多智能体架构 Agent-0–Agent-9 DAG | Inherited unchanged | — |
| §3 数据中介协议 (Data Broker) | Inherited unchanged | — |
| §4 意图路由器 (Intent Router) — Universe/shortlist | **Wrapped by V5.4 Patch 2** | Pool cap enforced pre-routing |
| §4 意图路由器 — MODE_PORTFOLIO_MONITOR | **Amended by V5.4 Patch 7** | Global WAIT gate added |
| §5 行业适配引擎 (Industry Templates) | Inherited unchanged | — |
| §6 Agent执行手册 Agent-0 (Forensic) | Inherited unchanged | — |
| §6 Agent执行手册 Agent-1 (Data Intake) | Inherited unchanged | — |
| §6 Agent执行手册 Agent-2 (Management Diligence) | **Amended by V5.4 Patch 8** | Outputs ADVISORY-ONLY; no recommendation |
| §6 Agent执行手册 Agent-3 (Industry/Competitive) | Inherited unchanged | — |
| §6 Agent执行手册 Agent-4 (Bear Case) | Inherited unchanged | — |
| §6 Agent执行手册 Agent-5 (Shortlist Constructor) | **Wrapped by V5.4 Patch 2** | Pre-ADD pool cap check |
| §6 Agent执行手册 Agent-6 (Valuation Synthesis) | **Amended by V5.4 Patches 1, 6** | alpha_source gate; edge_score gate |
| §6 Agent执行手册 Agent-7 (综合评级引擎) | **Amended by V5.4 Patches 1, 7, 8** | ai_replicable cap; WAIT gate; boundary check |
| §6 Agent执行手册 Agent-8 (Calibration Auditor) | Inherited unchanged | — |
| §6 Agent执行手册 Agent-9 (Damodaran Engine) | **Amended by V5.4 Patch 4** | Regime shift pre-check; historical FCF invalidation |
| §8.4 Restated Financials | **Amended by V5.4 Patch 4** | Capex ratio tracking added |
| §9 最终输出格式 (Final Output Format) | **Amended by V5.4 Patches 1, 5, 7** | alpha_source block; holding_thesis block; WAIT memo |
| §10 系统约束与安全过滤器 | Inherited unchanged | — |
| §11.5 Monte Carlo Valuation Distribution | **Amended by V5.4 Patch 4** | Historical FCF path disabled when regime_shift_flag |
| §11.6 Expansion Option Module | Inherited unchanged | — |
| §11.7 Pricing Gymnastics Detector | Inherited unchanged | — |
| §12.3 Distress-Adjusted Dual-Track | **Amended by V5.4 Patch 4** | V_going_concern sourced from regime-shift re-DCF when flagged |
| §14.3 Dynamic Risk Calibration Engine | Inherited unchanged | — |
| §15.1 一票否决规则集中归档 (VR-01–VR-05) | Inherited unchanged | — |
| Gemini Gems 部署指引 | Inherited unchanged | — |
| **IgnitionScanner** (new) | **New: V5.4 Patch 3** | No V5.3 equivalent |
| **edge_registry/** (new) | **New: V5.4 Patch 6** | No V5.3 equivalent |
| **active_conviction_pool** (new) | **New: V5.4 Patch 2** | No V5.3 equivalent (enforcement) |

---

## Output Contract

### Base Schema (DOCTRINE-001 §3 verbatim)

All outputs must conform to the following schema. This is reproduced verbatim from DOCTRINE-001 §3:

```yaml
ticker: <symbol>
recommendation: [WAIT | TRACK | BUY-CANDIDATE | SIZE-UP | TRIM | EXIT]

asymmetric_payoff:
  modeled_downside_pct: <number>     # required, conservative
  credible_upside_pct: <number>      # required, NOT max scenario
  apr: <upside / downside>           # must be ≥ 50 to be BUY-CANDIDATE

alpha_source:
  type: [PROBABILISTIC | INFORMATIONAL | CATALYST]
  edge_evidence: <one sentence, concrete>
  ai_replicable: <true | false>      # if true, recommendation downgraded to TRACK at most

reinvestment_regime:
  capex_ttm_vs_5y_avg: <ratio>
  regime_shift_flag: <bool>          # if true, historical FCF cannot be extrapolated

ignition_state:
  signals_active: <list>
  ignition_alert: <bool>

verification_ledger:
  core_assumption: <string>
  falsification_checkpoints:
    - at: <T+3M | T+6M | T+12M>
      metric: <observable>
      fail_threshold: <value>
  exit_trigger: <rule>

edge_score: <0-10>                   # gates max position size
```

### V5.4 Extensions

The following fields are added to the base schema for V5.4. They do not replace any base fields.

```yaml
# Extension 1: Full holding thesis (Patch 5 expansion of verification_ledger)
holding_thesis:
  ticker: <symbol>
  core_assumption: <string>
  falsification_checkpoints:
    - checkpoint_id: <string>
      at: <T+3M | T+6M | T+12M | T+18M | T+24M>
      due_date: <ISO-8601>
      metric: <string>
      fail_threshold: <string>
      status: <PENDING | PASSED | FAILED>
      actual_value: <string | null>
  exit_trigger: <string>

# Extension 2: Pool event log (Patch 2)
pool_event_log:
  - event_type: <ADD | EVICT | PROMOTE | DEMOTE>
    ticker: <symbol>
    timestamp: <ISO-8601>
    reason: <string>
    eviction_candidate: <symbol | null>
    replacement_rationale: <string | null>

# Extension 3: Advisory status (Patch 8)
advisory_status: <ADVISORY-ONLY | null>
charter_violation_warning: <string | null>

# Extension 4: Gate violation flag (Patch 7)
gate_violation: <bool | null>

# Extension 5: Regime shift re-DCF (Patch 4)
regime_shift_redcf:
  forward_ebit: <float | null>
  tax_rate: <float | null>
  new_reinvestment_rate: <float | null>
  forward_roic: <float | null>
  wacc: <float | null>
  dcf_status: <REGIME-SHIFT-REDCF | INVALID-REGIME-SHIFT | NORMAL | null>
```

---

## Acceptance Tests

The following assertions constitute a minimum test suite that any implementation under `tests/` must verify. Each assertion references one or more patches.

**Pool integrity (Patch 2)**:
- `active_conviction_pool.current_size` ≤ 30 in all system state snapshots at any point in time.
- Every ADD event in the pool event log when `current_size` was 30 is immediately preceded by an EVICT event.
- Eviction candidates are ordered by: lowest `edge_score` first, then lowest `APR`, then oldest unverified checkpoint; no arbitrary eviction is accepted.

**Alpha source gate (Patch 1)**:
- No output record exists where `recommendation ∈ {BUY-CANDIDATE, SIZE-UP}` and `alpha_source.ai_replicable = true`.
- No output record above `WAIT` has a null or missing `alpha_source.edge_evidence`.

**Regime shift (Patch 4)**:
- No DCF output with `dcf_status = INVALID-REGIME-SHIFT` is used as a valuation anchor for any recommendation above `TRACK`.
- For any name with `regime_shift_flag: true`, the output contains a `regime_shift_redcf` block with non-null `new_reinvestment_rate`.
- `low_pe_safety_bonus` is downgraded (not omitted, but lowered one tier) for any name with `regime_shift_flag: true`.

**Verification checkpoints (Patch 5)**:
- No `BUY-CANDIDATE` or `SIZE-UP` recommendation exists for any name where `holding_thesis` is absent or has fewer than 2 `falsification_checkpoints`.
- A name with 2 `falsification_checkpoints` both having `status: FAILED` triggers `recommendation ∈ {TRIM, EXIT}` per its `exit_trigger` rule.

**WAIT gate (Patch 7)**:
- When no name satisfies `apr ≥ 50 AND edge_score ≥ 4.0` simultaneously, system cash allocation output is `WAIT` with a non-null `wait_memo`.
- `wait_memo.gate_failures.apr_below_50_count + wait_memo.gate_failures.edge_score_below_4_count` is consistent with the active pool data.

**Edge registry (Patch 6)**:
- `SIZE-UP` never appears for a name with `edge_score < 4.0`.
- A registry file with only `paper_tracking` channel, last_contact = 200 days ago (freshness_decay_days = 180), yields `current_contribution = 0.0` for that channel.
- A name with no `edge_registry/{TICKER}.yaml` file defaults to `edge_score = 0.0`.

**IgnitionScanner (Patch 3)**:
- `ignition_alert: true` requires ≥ 3 `live_signals = true` within a 90-day window; 2 signals never triggers the alert.
- `IgnitionScanner` output never contains a `recommendation` field or any price target.
- `signal_window_days ≤ 90` whenever `ignition_alert = true`.

**Human/Machine charter (Patch 8)**:
- Agent-2 output never contains a `recommendation` field; it contains only `advisory_memo`.
- Agent-7 output containing any field matching `human_boundary_triggers` has `recommendation: null` and `charter_violation_warning` populated.
- Injecting `management_quality_score: 9` directly into Agent-7 input causes `recommendation: null` in Agent-7 output.

---

## Open Questions / Deferred to V5.5

The following items are explicitly deferred. They are not solved by this specification. V5.5 must address or formally defer them again.

1. **Silence index data source**: Patch 3 (IgnitionScanner) specifies `sell_side_note_count_zscore`, `equity_volume_zscore`, `job_posting_count_zscore`, and `media_mention_zscore` as inputs. The data pipeline for these inputs is not specified. Sources may include Refinitiv, Bloomberg, LinkedIn Talent Insights, and public web scraping. Data licensing, refresh cadence, and normalization methodology are deferred to V5.5.

2. **Edge score decay calibration**: Patch 6 specifies default `freshness_decay_days` per channel type. These defaults are heuristic. No empirical calibration has been performed. The question of whether management access decays faster or slower than paper tracking in practice is deferred to V5.5 pending track-record data.

3. **APR model for PROBABILISTIC alpha sources**: Patch 1 allows `type: PROBABILISTIC` as a valid alpha source (though it typically caps at `TRACK`). The question of whether any `PROBABILISTIC` source can ever justify `BUY-CANDIDATE` — and under what conditions — is deferred to V5.5. The conservative default is: `PROBABILISTIC` → `TRACK` always.

4. **Cash option value quantification**: Patch 7 references "the option value of cash awaiting a once-per-decade dislocation" as a concept. Formal modeling (e.g., using expected-value of the best opportunity in a rolling 10-year window, discounted by the cost of holding) is deferred to V5.5. The current spec requires the `wait_memo` to acknowledge the option value but does not require a numerical estimate.

5. **IgnitionScanner false-positive rate**: Patch 3's trigger rule (≥ 3 signals in 90 days) has an unknown false-positive rate. Calibration against historical industry ignition events (PCB/optical modules 2023–2024, battery materials 2020–2021) is deferred to V5.5.

6. **Inter-checkpoint regime**: Patch 5 defines checkpoints at T+3M, T+6M, T+12M. For multi-year positions, T+18M and T+24M checkpoints are listed as allowed values but their mandatory inclusion criteria are not specified. V5.5 should define when long-tenor checkpoints are required (e.g., positions held > 12 months must add T+18M automatically).

7. **edge_registry write access control**: Patch 6 creates `edge_registry/*.yaml` as static assets. The question of who is authorized to update these files (and with what audit trail) is a process question deferred to V5.5.

---

*End of VIAIOS-P-V5.4-Edge-Driven.*
