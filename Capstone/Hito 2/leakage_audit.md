# leakage_audit.md — Hito 2

This document extends the Hito 1 leakage audit to cover both targets (`is_top10` and `is_top5`) and addresses the confounding limitation explicitly.

---

## Column classification (unchanged from Hito 1)

| Category                         | Columns                                                                                                                                                                                                                     | Status                                                 |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| **Pre-race features (used)**     | `grid_position`, `qualifying_position`, `constructor_tier`, `circuit_type`, `constructor_name`, `driver_prior3_avg_finish`, `constructor_prior3_avg_finish`, `driver_circuit_prior_avg`, `round`                            | Used as predictors                                     |
| **Engineered pre-race features** | `grid_rank_inv`, `grid_x_tier`, `driver_constructor_avg`, `form_gap`, `circuit_history_missing`, `tier_num`                                                                                                                 | Derived from pre-race inputs only                      |
| **Scenario inputs**              | `n_stops`, `strategy_type`, `compound_sequence`, `stint_lengths`, `stint1_length`–`stint5_length`                                                                                                                           | What-if labels only — never in the predictive pipeline |
| **Audit / post-race**            | `safety_car_periods`, `safety_car_laps`, `vsc_laps`, `weather_actual`, `wet_laps`, `avg_track_temp`, `avg_air_temp`, `avg_pit_stop_duration_s`, `total_pit_time_s`, `first_pit_lap`, `last_pit_lap`, `track_status_summary` | Excluded from model; used only in error slicing        |
| **Target leakage — never use**   | `finish_position`, `points`, `positions_gained`, `is_top3`, `dnf`, `status`                                                                                                                                                 | Never used as predictors                               |

---

## Target-specific leakage check

### `is_top10` (carry-over)

- `is_top10` is the target — excluded from features.
- `is_top5`, `is_top3`, `points`, `finish_position`, `positions_gained` are all correlated with `is_top10` but are also post-race outcomes — none appear in the feature set.
- `dnf` is a post-race outcome (we do not know pre-race if a driver will retire) — excluded.

### `is_top5` (expansion)

- `is_top5` is the target — excluded from features.
- `is_top10` is post-race and correlated with `is_top5` — excluded.
- The same feature set used for `is_top10` is applied unchanged. No new feature was added for `is_top5` that could create leakage.
- The Platt calibration for `is_top5` uses only the 2022 calibration block — never the test set.

---

## Scenario input declaration — holds under both targets

The assignment states: "Strategy features are observed post-race. They are scenario inputs in this capstone, not pre-race signals."

This declaration holds identically for `is_top5`. The argument is symmetric:

- We do not know n_stops or compound_sequence before the race for either target.
- The what-if comparison uses strategy labels to frame the risk trade-off, not as model inputs.
- Changing the strategy label in the what-if table does not change the model output — only changing pre-race features (grid position, form, tier) changes the probability.

---

## Confounding limitation — explicit treatment

**Statement:** Strategy choice is not independent of car pace, driver skill, weather, or race incidents. Teams with faster cars run more aggressive strategies, and those strategies tend to succeed partly because of car pace rather than strategy quality alone.

**Consequence for `is_top10`:** A model trained on observed (strategy, outcome) pairs will conflate the signal of an aggressive strategy (associated with fast cars) with the underlying car pace signal. Even though strategy variables are excluded as predictors, their correlates — constructor tier, driver form — carry some of this confounded signal.

**Consequence for `is_top5`:** The confounding is more severe at the top-5 boundary. Podium strategies (e.g., undercut attempts) are overwhelmingly used by front-tier cars. The `is_top5` model's strong ROC-AUC (0.940) partly reflects this: front-tier cars dominate the top 5, and constructor_tier + form features predict that accurately. But this means the model is capturing "which tier of car tends to finish top 5" more than "which strategy leads to top 5."

**What this means for recommendations:** All what-if probabilities should be interpreted as **baseline probability estimates given pre-race conditions**, not as causal strategy-outcome probabilities. A recommendation such as "2-stop M-M-H increases your P(top5)" is not supported by this model — only "starting from P7 in a front-tier car, the baseline P(top5) is 0.678."

**Mitigation attempted:** The error analysis (Slice 3: constructor tier) explicitly quantifies where this confounding is most visible — the front-tier slice has the highest Brier for `is_top5` (0.177), which is where intra-tier car-pace differences the model cannot see are largest.

---

## Temporal split compliance — both targets

| Split       | Seasons   | Used for                                                                |
| ----------- | --------- | ----------------------------------------------------------------------- |
| Train       | 2019–2021 | `model.fit()` for both `is_top10` and `is_top5` models                  |
| Calibration | 2022      | Platt scaling only for both models — never for model selection          |
| Test        | 2023–2024 | One-time final evaluation — not accessed during training or calibration |

No data from 2022, 2023, or 2024 was used during feature engineering decisions, model architecture choices, or hyperparameter selection.

---

## Qualifying data note

`qualifying_position` and `grid_position` are identical in this dataset (Pearson correlation = 1.000 across all rows). The assignment notes that `qualifying_time_s` is empty. This means grid-penalty scenarios (driver takes engine penalty, drops 5 places) are not reflected in the features. Any what-if scenario that involves a post-qualifying penalty must carry this caveat explicitly.
