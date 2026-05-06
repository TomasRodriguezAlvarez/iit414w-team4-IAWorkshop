1. Decision Context

Who are we supporting?: The Chief Strategy Officer for a mid-to-top tier team (e.g., Mercedes or Aston Martin).

When is the decision made?: Sunday morning before the race, with real-time adjustments during the first 15–20 laps.

Decision focus: Optimization of the number of pit stops (n_stops) and the tire sequence (compound_sequence) to maximize the probability of a points-scoring finish.

2. Target & Primary Metric
Target: is_top10.

Primary Metric: Brier Score.

Justification: In F1 strategy, binary classification (Yes/No) is insufficient for high-stakes risk management. We need precise probability estimates to weigh the "gamble" of an extra pit stop. The Brier Score penalizes overconfident yet incorrect predictions, which is critical: predicting a 95% chance of a Top 10 finish and failing is a catastrophic failure for a strategist. We also track Log Loss and ROC-AUC for calibration and discrimination monitoring.

3. Baseline Plan
Model: Ensemble of a Random Forest and a Logistic Regression. The Random Forest and Logistic components are trained on pre-race features (2019–2021), raw ensemble scores are combined (70% RF + 30% LR), and the combined score is Platt-calibrated using the 2022 calibration block. We also report individually Platt-calibrated RF and LR results for reference.

Features: a set of pre-race observables including `grid_position` (and `qualifying_position` when available), engineered grid features (`grid_rank_inv`, `grid_x_tier`), recent form (`driver_prior3_avg_finish`, `constructor_prior3_avg_finish`), `driver_circuit_prior_avg`, and categorical features like `constructor_tier`, `circuit_type`, and `constructor_name`.

F1 Rationale: Starting position is the single strongest pre-race predictor, but combining it with short-term form and team-tier information improves calibration and discrimination while remaining leakage-safe.

Performance: On the untouched 2023–2024 test set the notebook reports:
- Grid-rule baseline (assignment floor): Brier = 0.208
- Logistic baseline (uncalibrated): Brier ≈ 0.134
- Logistic (Platt-calibrated): Brier ≈ 0.135
- Random Forest (Platt-calibrated): Brier ≈ 0.132
- RF + LR ensemble (Platt-calibrated): Brier ≈ 0.132 (best by Brier)

The ensemble matches the calibrated docent reference Brier (0.132) while using only pre-race features and the locked temporal split.

4. What-if Comparison Plan
To validate the model's utility as a strategy tool, we will simulate two specific scenarios:

Scenario A (Stop Count Strategy): Lewis Hamilton (Mercedes) at the British GP (Silverstone) starting P7. Compare P(is_top10) for a 1-stop strategy (M-H) versus a 2-stop strategy (M-M-H).

Scenario B (Compound Choice): Charles Leclerc (Ferrari) at the Monaco GP starting P4. Compare P(is_top10) for an aggressive "Soft-start" stint versus a conservative "Hard-start" long stint.

5. Acknowledgment of Dataset Limitations
We identify and account for the following limitations in our framing:

Strategy Features as Scenario Inputs (Leakage): Variables like n_stops and stint_lengths are post-race observations. We declare them as scenario inputs (user-controlled "what-if" variables) rather than pre-race signals to avoid invalid model leakage.

Incomplete Qualifying Data: Some detailed qualifying telemetry (e.g., `qualifying_time_s`) is missing in this build. The notebook uses `qualifying_position` when available and falls back to `grid_position` where needed (and the grid-rule baseline similarly falls back). This proxying may omit nuances such as late grid penalties or post-qualifying penalties not reflected in the raw starting order.

Temporal Scope: Data starts in 2019, which captures the current turbo-hybrid era but limits the model's exposure to historical shifts in tire degradation patterns.

6. Experiments Planned (Hito 2)
Non-Linear Modeling (XGBoost): Hypothesis: Gradient-boosted trees will better capture the non-linear interaction between track temperature (avg_track_temp) and tire life compared to a linear baseline.

Momentum Engineering: Hypothesis: Including the constructor_prior3_avg_finish will help the model account for mid-season technical upgrades and team momentum.

Weather Sensitivity Audit: Hypothesis: The weight of grid_position decreases in wet weather conditions (weather_actual = 'wet'), increasing the relative importance of strategy choice.

7. Team Workflow
Mon – Tue: Joaquín implemented the data loading, temporal split logic (2019–2021 / 2022 / 2023–2024), and the Platt calibration mapping in the notebook. [Partner Name] researched metric justifications and drafted the decision context.

Wed (Studio Session): Final integration of the notebook, execution of the leakage audit cell, and documentation of AI interactions in PROMPTS.md. Final verification that the repository runs on a clean clone before the 16:20 deadline.