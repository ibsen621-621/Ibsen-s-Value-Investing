# DOCTRINE-001 — Edge over Logic

> **Status**: Foundational (Tier-0)
> **Scope**: Binding on all VIAIOS-P versions ≥ V5.4
> **Origin**: Distilled from a working dialogue between the maintainer and a long-term investing partner, 2026-05.
> **Principle**: Doctrine precedes implementation. Any future module, agent, or scoring rule that contradicts this document MUST be rejected or trigger an explicit doctrine amendment (DOCTRINE-00X-amend).

---

## 0. Why this document exists

Every prior version of VIAIOS-P (V4.x → V5.3) optimized along a **logic axis**: better templates, better calibration, better multi-agent decomposition, better Damodaran integration. These are necessary but **not sufficient** to generate excess returns.

The hard truth, surfaced in dialogue:

> *"Quants have been doing exactly what you are doing for 10+ years, with API access, leading multiples from sell-side, and TTM data combined. If your edge is 'better analysis on public information,' you are — at best — playing a probability game with no informational advantage. The expected outcome is being the exit liquidity."*

This doctrine resets the framework's purpose:

**The system's job is NOT to find good companies. The system's job is to allocate scarce human attention to the few situations where (a) the payoff is grossly asymmetric AND (b) the operator possesses a non-replicable information edge.**

Everything else is overhead.

---

## 1. The Five Root Propositions

These are non-negotiable axioms. All scoring, ranking, and recommendation logic in downstream versions must be derivable from — or at minimum compatible with — these five.

### P1 — Asymmetry over Prediction
> *"10x returns are never predicted; they are forged in the process. What you can control is risking $1 to make $100."*

- The framework optimizes for **payoff geometry**, not for point estimates of intrinsic value.
- Target: Asymmetric Payoff Ratio (APR) ≥ 50:1 between modeled downside and a credible (not maximal) upside scenario.
- "Fair value" is a sanity check, not a thesis.

### P2 — Edge over Analysis
> *"You are already at a disadvantage in any direction where AI/quant can replicate your work. Advance only where they cannot."*

- Every actionable recommendation must be tagged with an **Alpha Source** that is *not* replicable by a generic API + LLM pipeline.
- Acceptable edge types: primary expert calls, management access, supply-chain contacts, frontier paper tracking, on-the-ground operational signals.
- Unacceptable as sole edge: factor screens, ratio comparisons, sell-side consensus deltas, public news synthesis.

### P3 — Cheap is not Safe
> *"Tencent gets more expensive as it falls if Capex regime shifts and historical FCF is invalidated."*

- Low multiples are **suspect by default** when capital allocation regime is changing.
- Any thesis that relies on historical FCF/EPS during a Capex inflection must be downgraded until the new reinvestment rate × ROIC produces a credible forward FCF.
- The Value Trap is the modal failure mode of "value investing done logically." It must be modeled, not assumed away.

### P4 — Industries Ignite Discretely
> *"PCB and optical-module companies looked dead for years, then dozens lit up simultaneously within months. Ignition is a discrete event, not a continuous valuation function."*

- Cycle position scoring (current ADVANCED-MFG / CYCLICAL templates) is **necessary but insufficient**.
- The framework must separately detect **ignition** — a coordinated, multi-signal, narrow-window state change across an industry.
- Pre-ignition monitoring beats post-ignition valuation. Valuation is for sizing, not for discovery, in ignition cases.

### P5 — Concentration and Patience are Assets
> *"30 names tracked deeply is enough. Waiting 10 years for one real opportunity, in exchange for 10%/year, is a trade I would gladly take."*

- Breadth is a liability; depth is the asset.
- Hard cap: ≤ 30 names in the Active Conviction Pool at any time. Adding requires evicting.
- `WAIT` is a first-class recommendation. Idle cash carries an explicit option value that the system must price.

---

## 2. The Human / Machine Charter

The framework's self-understanding of its own division of labor. Binding.

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

**Corollary**: any future agent that attempts to automate a "Human" row of this table is a doctrine violation and must be rejected unless DOCTRINE-001 is formally amended.

---

## 3. The Output Contract

All future versions must produce, for every monitored name, at minimum:

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

A name without a populated `alpha_source` and `verification_ledger` **cannot** receive a recommendation higher than `TRACK`.

---

## 4. The "WAIT" Default

The default state of the system, in absence of evidence, is `WAIT`.

- A version that produces a non-`WAIT` recommendation in the absence of `(APR ≥ 50) AND (edge_score ≥ 4)` is in violation.
- The system must report a `WAIT` decision with the same rigor as a `BUY-CANDIDATE` decision: what was scanned, what failed, and what would change the answer.
- The patience premium (the optionality of cash awaiting a once-per-decade dislocation) is to be modeled explicitly, not implicitly assumed away.

---

## 5. Failure Modes this Doctrine Exists to Prevent

1. **The Probability Trap** — Producing a long list of "statistically attractive" names with no edge, then watching them perform like an ETF.
2. **The Value Trap** — Recommending falling-multiple names whose Capex regime has invalidated their historical cash generation.
3. **The Logic Trap** — Mistaking a well-structured argument for an investable advantage. Logic is necessary; it is not the source of returns.
4. **The Breadth Trap** — Diluting attention across hundreds of names so that no single name is known well enough to act decisively when ignition occurs.
5. **The Automation Trap** — Building agents to replace the irreducibly human edge (relationships, taste, judgement), thereby destroying the only remaining source of alpha.

---

## 6. Amendment Protocol

This doctrine may be amended only by:

1. A new file `doctrine/DOCTRINE-001-amend-NNN.md` that explicitly names the clause being changed.
2. A written rationale referencing either (a) empirical track-record evidence over ≥ 3 years, or (b) a structural change in the AI/quant landscape that invalidates an axiom.
3. No silent overrides in code, prompts, or downstream version specs.

Versions V5.4 and onward must include, at the top of their spec, the line:

> *"Compliant with DOCTRINE-001-Edge-over-Logic."*

If a future version cannot make that claim, it must declare the deviation and reference an amendment file.

---

## 7. One-Sentence Summary

> **Stop trying to be a smarter analyst. Start being the only person in the room who knows something specific, about a small number of situations, where being right pays 50× and being wrong costs 1×.**

Everything in this repository — every template, every agent, every score — exists to serve that sentence.

---

*End of DOCTRINE-001.*
