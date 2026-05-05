# edge_registry/ — Information Edge Registry Schema

This directory stores one YAML file per tracked ticker or industry, recording the information edge channels the operator maintains for that name. It is a **static asset** maintained by human operators; it is not auto-populated by any AI agent.

Spec: `VIAIOS-P-V5.4-Edge-Driven.md` Patch 6.

---

## File naming

| Scope | Filename |
|---|---|
| Single ticker | `{TICKER}.yaml` (e.g., `0700.HK.yaml`, `NVDA.yaml`) |
| Industry-level edge | `{INDUSTRY_ID}.yaml` (e.g., `optical_module.yaml`, `pcb_tier1.yaml`) |

---

## Schema

```yaml
# edge_registry/{TICKER}.yaml

ticker: <symbol>                    # Exchange:ticker format recommended (e.g., HKG:0700)
last_registry_update: <ISO-8601>    # Date this file was last manually updated

edge_channels:
  - channel_id: EC-001              # Sequential, unique within this file
    type: <channel_type>            # See allowed values below
    description: <string>           # One sentence. Does NOT name the individual contact.
                                    # Example: "Tier-2 supplier of substrate material; provides
                                    #           lead-time and order volume colour quarterly."
    last_contact: <ISO-8601>        # Date of most recent signal from this channel.
    freshness_decay_days: <int>     # Days after which contribution decays to zero.
                                    # Use defaults below unless overriding with justification.
    current_contribution: <float>   # Computed: base_contribution × decay_factor (0.0–1.0).
                                    # Recompute on each registry update.

edge_score: <float>                 # 0.0–10.0. Computed from current_contribution values.
                                    # Hard gate: edge_score < 4.0 → position weight ≤ 5%
                                    # (heavy-position tier blocked).
```

---

## Allowed channel types

| `type` value | Meaning | Base Contribution | Default Decay (days) |
|---|---|---|---|
| `management_access` | Direct access to C-suite or IR | 1.0 | 90 |
| `supply_chain_contact` | Supplier, customer, or distributor channel | 0.9 | 60 |
| `industry_expert_call` | Expert network call or independent industry expert | 0.8 | 90 |
| `channel_check` | Reseller, retailer, or end-customer survey | 0.7 | 45 |
| `regulatory_contact` | Regulatory agency, standards body, or government insider | 0.7 | 120 |
| `paper_tracking` | Academic or pre-commercial R&D paper monitoring | 0.5 | 180 |
| `other` | Any channel not fitting the above; requires description | 0.3 | 60 |

---

## Decay formula

```
days_since_contact = (today - last_contact).days
decay_factor       = max(0.0, 1.0 - (days_since_contact / freshness_decay_days))
current_contribution = base_contribution × decay_factor
```

Example: `supply_chain_contact`, `last_contact` = 30 days ago, `freshness_decay_days` = 60:
```
decay_factor         = 1.0 − (30 / 60) = 0.5
current_contribution = 0.9 × 0.5       = 0.45
```

---

## edge_score computation

```
raw_score  = sum(current_contribution for all channels)
edge_score = min(10.0, raw_score × normalization_factor)
```

`normalization_factor` scales the raw sum to the 0–10 range. With a single maximum-weight channel fully fresh, raw_score = 1.0; with four diverse fresh channels, raw_score ≈ 3.4. The normalization maps a "four-channel, fully-fresh" registry to approximately 8–9 out of 10. The exact factor is calibrated at the implementation layer (deferred to V5.5); for the spec, `normalization_factor = 3.0` is the reference default.

---

## Position tier gate (from Patch 6)

| Position Tier | Max Portfolio Weight | Minimum edge_score Required |
|---|---|---|
| Heavy | > 5% | ≥ 4.0 |
| Standard | 2%–5% | ≥ 2.0 |
| Tracking | < 2% | None (0.0 acceptable) |

A missing registry file defaults to `edge_score = 0.0`, which caps the name at the Tracking tier.

---

## What NOT to store here

- Do not record the names or contact details of sources. This file stores channel type, description, and timing metadata only.
- Do not record trading decisions, price targets, or recommendations. This is an input to the recommendation engine, not an output.
- Do not create example files with real ticker data. The schema above is sufficient for implementation; real entries are created by the human operator.

---

*Schema version: V5.4. Any schema change requires a patch note in the next VIAIOS-P version.*
