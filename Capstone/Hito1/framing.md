# framing.md — Hito 1: F1 Race Strategy Advisor

---

## 1. Decision Context

**Who are we supporting?** The Chief Strategy Officer for a mid-to-top tier Formula 1 team (e.g., Mercedes or Ferrari).

**When is the decision made?** Sunday morning before the race, with real-time adjustments possible during the first 15–20 laps.

**Decision focus:** Optimization of pit stop count (`n_stops`) and tire sequence (`compound_sequence`) to maximize the probability of a points-scoring finish (`is_top10`).

**Prediction unit:** One driver-race observation. The model produces P(is_top10) for a given driver, circuit, and pre-race context. The strategist then uses the model as a what-if comparison tool by varying scenario inputs (n_stops, compound_sequence) to evaluate alternative strategies.

---

## 2. Target & Primary Metric

**Target:** `is_top10`

**Primary metric:** Brier Score

**Justification:** In F1 strategy, a binary Yes/No prediction is insufficient for high-stakes risk management. The strategist needs calibrated probability estimates to weigh the risk of an extra pit stop against the expected position gain. The Brier Score penalizes overconfident incorrect predictions — predicting a 95% chance of a Top 10 finish and failing is a catastrophic planning error. We additionally track Log Loss for distributional quality and ROC-AUC for discrimination monitoring.

---

## 3. Baseline Plan

**Model:** Ensemble of a Random Forest and a Logistic Regression.

- Both components are trained on pre-race features from 2019–2021 only.
- Raw ensemble scores are combined: 70% RF + 30% LR.
- The combined score is Platt-calibrated using the 2022 calibration block only.
- The 2022 block is **never** used for model selection — only to learn the calibration mapping.

**Pre-race features used:**
- Raw: `grid_position`, `qualifying_position`, `driver_prior3_avg_finish`, `constructor_prior3_avg_finish`, `driver_circuit_prior_avg`, `round`
- Categorical: `constructor_tier`, `circuit_type`, `constructor_name`
- Engineered: `grid_rank_inv` (1/grid_position), `grid_x_tier` (grid × tier interaction), `driver_constructor_avg`, `form_gap`, `circuit_history_missing`

**F1 rationale:** Starting position is the single strongest pre-race predictor of finish outcome. Adding rolling form, circuit history, and constructor-name information improves calibration and discrimination while remaining fully leakage-safe. The interaction term `grid_x_tier` captures the empirical fact that P10 from grid P6 in a front-tier car is very different from P6 in a backmarker car.

**Performance on the untouched 2023–2024 test set:**

| Model | Brier | Log Loss | ROC-AUC |
|---|---|---|---|
| Grid-rule heuristic (assignment floor) | 0.208 | — | — |
| Logistic baseline (uncalibrated) | 0.134 | 0.427 | 0.887 |
| Logistic (Platt-calibrated) | 0.135 | 0.429 | 0.887 |
| Random Forest (Platt-calibrated) | 0.132 | 0.424 | 0.887 |
| **RF + LR ensemble (Platt-calibrated)** | **0.132** | **0.422** | **0.889** |
| Docent reference | 0.132 | — | 0.892 |

The ensemble matches the calibrated docent reference on Brier score and comes within 0.003 ROC-AUC points. The small gap is expected given the feature set is intentionally restricted to pre-race observables. We do not recommend deployment without further validation on live race data.

---

## 4. What-if Comparison Plan

**Important declaration — scenario inputs vs. leakage:**
Strategy features (`n_stops`, `compound_sequence`, `stint_lengths`) are post-race observations in the raw data. Using them as ordinary predictors would be textbook leakage. In this capstone they are explicitly declared as **user-controlled scenario inputs**: the strategist sets them to ask "what if?" The model's pre-race features produce a baseline P(is_top10); the strategy scenario frames the risk trade-off around that baseline. This distinction is declared in the leakage audit cell (Section 4 of the notebook) and enforced throughout the pipeline.

Because the model is trained on pre-race features only, the predicted P(is_top10) does not change when only the strategy label changes (e.g., 1-stop vs. 2-stop at the same grid position). This is correct and intentional: the tool quantifies baseline risk from starting conditions. The strategist then asks "given this baseline risk of 0.854, is the extra degradation of a 2-stop worth the track-position gain?"

**Scenario A — Stop count strategy: Lewis Hamilton (Mercedes), British GP 2024 (Silverstone)**

Feature values from the actual 2024 British GP dataset row:
`grid_position=2`, `driver_prior3_avg_finish=3.67`, `constructor_prior3_avg_finish=3.17`, `driver_circuit_prior_avg=1.83`, `constructor_tier=front`, `circuit_type=permanent`, `round=12`.

| Scenario | Grid | Strategy label | P(top10) | Interpretation |
|---|---|---|---|---|
| P2, 1-stop M-H | 2 | n_stops=1, M-H | **0.922** | Very high baseline from front row |
| P7, 1-stop M-H | 7 | n_stops=1, M-H | **0.854** | Same driver, dropped grid |
| P7, 2-stop M-M-H | 7 | n_stops=2, M-M-H | **0.854** | Identical pre-race risk; 2-stop justified only if team expects pace to recover 3+ positions |

**Scenario B — Compound and tier sensitivity: Charles Leclerc (Ferrari), Monaco GP**

Feature values from actual Monaco GP dataset rows:

| Scenario | Season | Grid | Driver form | Constructor form | Circuit avg | Tier | P(top10) |
|---|---|---|---|---|---|---|---|
| P1, front-tier, soft-start | 2024 | 1 | 3.33 | 4.17 | 12.5 | front | **0.931** |
| P4, front-tier, hard-start | 2024 | 4 | 3.33 | 4.17 | 12.5 | front | **0.898** |
| P6, midfield form, 1-stop M-H | 2023 | 6 | 10.0 | 8.67 | 14.67 | midfield | **0.797** |

Monaco is the most position-locked circuit on the calendar. A 3-position drop (P1→P4) costs ~3.3 pp; Ferrari's tier demotion in 2023 cost an additional ~10 pp. The model helps quantify whether an aggressive soft-start is worth the risk from P1.

---

## 5. Acknowledgment of Dataset Limitations

All five known limitations are acknowledged with consequences for recommendations:

**1. Strategy features are post-race observations (scenario input declaration required)**
`n_stops`, `compound_sequence`, and `stint_lengths` are observed after the race ends. They are excluded from the predictive pipeline and declared as user-controlled scenario inputs. Any recommendation derived from them must be interpreted as a hypothetical comparison, not a causal prediction.

**2. Qualifying data is incomplete**
`qualifying_time_s` is entirely missing (100% null). `qualifying_position` is used as a stand-in for `grid_position`; the two columns are identical in this dataset (correlation = 1.0). Grid penalties and post-qualifying position changes are not captured. Strategy recommendations framed around penalty-grid scenarios carry this caveat.

**3. Safety car and weather features are post-race**
`safety_car_periods` is a binary flag observed after the race — not a full race-control interval count. `weather_actual` and `wet_laps` are similarly post-race. These are kept in the audit table as audit columns and excluded from the baseline model. They could be used for scenario stress-tests ("what if a safety car occurs?") in Hito 2.

**4. Coverage starts in 2019**
The dataset covers 2019–2024 only, restricting the model to the current turbo-hybrid era. Teams that changed performance tier between 2019 and 2024 (e.g., Aston Martin) may be misrepresented by rolling averages computed early in the window.

**5. Strategy choice is not independent of car pace**
`n_stops` and `compound_sequence` choices are endogenous — teams with faster cars choose more aggressive strategies, and those strategies succeed partly because of car pace, not strategy alone. This confounding means the what-if tool should be interpreted as a baseline probability estimator, not a causal strategy optimizer. All scenario outputs carry this disclosure.

---

## 6. Experiments Planned for Hito 2

**Experiment 1 — XGBoost with track temperature interaction**
*Hypothesis:* Gradient-boosted trees will better capture the non-linear interaction between pre-race forecast temperature and tire degradation across circuit types. XGBoost will improve Brier score on the 2023–2024 test set by at least **0.005** relative to the RF+LR ensemble (from 0.132 to ≤ 0.127).
*Data source note:* `avg_track_temp` is a post-race audit column in Hito 1. For Hito 2 we will replace it with pre-race weather forecast data (available from historical weather APIs for race weekends), framing it as a legitimate pre-race scenario input consistent with leakage rules.

**Experiment 2 — Driver identity as a categorical feature**
*Hypothesis:* Encoding `driver_id` as a one-hot categorical predictor will improve ROC-AUC by at least **0.005** (from 0.889 to ≥ 0.894) by capturing driver-specific skill effects not fully reflected in the 3-race rolling average.
*Risk:* Drivers with few training observations will be handled via `handle_unknown='ignore'` OHE, falling back to the constructor-level signal. If the improvement is below 0.003, the feature will be dropped to avoid overfitting on sparse driver histories.

**Experiment 3 — Championship round as a seasonality feature**
*Hypothesis:* Splitting `round` into early/mid/late-season buckets and interacting with `constructor_tier` will improve Brier score by at least **0.003** (from 0.132 to ≤ 0.129), capturing mid-season development cycles where teams improve significantly between round 5 and round 15.
*Metric to watch:* Brier score on the 2023–2024 test set. If improvement is below 0.003, the experiment is declared inconclusive and the simpler `round` feature is retained.

---

## 7. Team Workflow

- **Joaquín Rodríguez:** Data loading pipeline, temporal split logic (2019–2021 train / 2022 calibration / 2023–2024 test), leakage audit cell, feature engineering (grid_rank_inv, grid_x_tier, driver_constructor_avg, form_gap, circuit_history_missing), and Random Forest + Logistic Regression model training with Platt calibration mapping.

- **Tomás Solano:** Decision context framing, Chief Strategy Officer use case, primary metric justification (Brier Score vs. binary classification), what-if scenario design (Hamilton/Silverstone and Leclerc/Monaco), and dataset limitation disclosure.

- **Tomás Rodríguez:** Experiment hypotheses for Hito 2 (XGBoost temperature interaction, driver identity features, championship round seasonality), performance reference benchmarking (0.208 grid-rule floor, 0.132 docent reference), calibration curve visualization, and results summary documentation.

Final integration of all notebook cells, verification of end-to-end execution on a clean clone, documentation of AI interactions in PROMPTS.md, and cross-team review before the 16:20 deadline.
