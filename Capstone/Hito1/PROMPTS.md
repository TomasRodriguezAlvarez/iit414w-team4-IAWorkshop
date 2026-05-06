# PROMPTS.md — AI Interaction Log

All interactions follow the 6-field standard: Context · Prompt · Output · Validation · Adaptations · Final Decision.

---

## AI Interaction 1 — Leakage classification and temporal split design

### Context
We were starting the Hito 1 notebook and needed to map every column in `f1_strategy_race_level.csv` to one of four categories: pre-race feature, scenario input, audit/post-race column, or target leakage. We also needed to confirm the correct implementation of the locked temporal split and the role of the calibration block.

### Prompt
> I am building a machine learning baseline for an F1 race strategy capstone. The assignment locks the temporal split as train 2019–2021, calibration 2022, test 2023–2024. The target is `is_top10`.
>
> Review the following column list and classify each column as: (1) pre-race feature safe to use as a predictor, (2) strategy scenario input — post-race observation allowed only as a what-if input, (3) audit/post-race column that must be excluded from the model, or (4) target leakage that must never be used as a predictor.
>
> Also explain how the calibration block should be used and what "Platt calibration" means in this context.

### Output
The AI correctly identified:
- Pre-race features: `grid_position`, `qualifying_position`, `constructor_tier`, `driver_prior3_avg_finish`, `constructor_prior3_avg_finish`, `driver_circuit_prior_avg`, `constructor_name`, `circuit_type`, `round`
- Scenario inputs (post-race, allowed only as what-if): `n_stops`, `compound_sequence`, `stint_lengths`, `strategy_type`, all `stint_N_length` columns
- Audit/post-race (excluded): `safety_car_periods`, `weather_actual`, `wet_laps`, `avg_track_temp`, `avg_air_temp`, all pit stop timing columns
- Target leakage (never use): `finish_position`, `points`, `positions_gained`, `is_top3`, `is_top5`, `is_top10`, `dnf`, `status`

The AI explained that the calibration block (2022) should be used only to fit a Platt scaling logistic regression on top of raw model scores — it should not be used for model selection, hyperparameter tuning, or feature selection. It also warned against using `qualifying_time_s` since it is missing in the dataset.

### Validation
We manually audited the notebook feature lists against this classification. We confirmed:
- no leakage columns appear in `baseline_numeric_features` or `baseline_categorical_features`,
- the temporal split matches exactly (`.isin([2019,2020,2021])`, `== 2022`, `.isin([2023,2024])`),
- calibration data is only passed to the Platt scaling step, not to `model.fit()`.

### Adaptations
We expanded the leakage classification into a dedicated notebook cell (Section 4) with an explicit four-category audit table, and added a `exists_in_csv` column to catch any column name mismatches at runtime.

### Final Decision
We adopted the leakage-safe framing and preserved the locked temporal split exactly as specified. The strategy scenario inputs are declared in both `framing.md` (Section 4) and the notebook audit cell.

---

## AI Interaction 2 — Baseline model selection and calibration strategy

### Context
After implementing the grid-rule heuristic, we needed a stronger but still defensible Hito 1 baseline. We wanted to remain interpretable and leakage-safe while improving on the grid-rule Brier of 0.208.

### Prompt
> I have a grid-rule heuristic baseline for predicting `is_top10` in F1 races. It achieves Brier 0.208 on the 2023–2024 test set using only qualifying position tiers.
>
> Suggest a stronger baseline that:
> - uses only pre-race features (no post-race leakage),
> - supports probability calibration using a held-out 2022 calibration block,
> - is explainable enough for a strategy decision-support context,
> - and can be implemented with scikit-learn.
>
> Also explain when combining models into an ensemble is appropriate for Hito 1.

### Output
The AI suggested:
- Logistic Regression as the primary transparent baseline (easy to interpret, supports calibration)
- Random Forest as a second component (captures non-linear interactions between grid position and constructor tier)
- Platt calibration: fit a logistic regression on the logit-transformed raw scores using only the 2022 block
- A weighted ensemble (e.g., 70% RF + 30% LR) before the calibration step
- Evaluation with Brier score, log loss, ROC-AUC, and a calibration curve on the test set

It noted that an ensemble is appropriate at Hito 1 if both components are individually validated, the calibration step is applied once after blending (not separately), and no hyperparameter tuning uses the test set.

### Validation
We implemented and evaluated all four model variants:
- Grid-rule: Brier 0.208
- Logistic (uncalibrated): Brier 0.134, ROC-AUC 0.887
- Logistic (Platt-calibrated): Brier 0.135
- Random Forest (Platt-calibrated): Brier 0.132, ROC-AUC 0.887
- RF+LR ensemble (Platt-calibrated): Brier 0.132, ROC-AUC 0.889

We verified that calibration used only the 2022 block and that test data was accessed only once for final evaluation.

### Adaptations
We adjusted the ensemble weight to 70/30 after observing that increasing the RF weight beyond 0.70 did not further improve Brier on the calibration set. We retained the simpler logistic baseline in the notebook for transparency and comparison. We added an explicit result summary table in Section 10.

### Final Decision
We selected the calibrated RF+LR ensemble as the Hito 1 baseline. It matches the docent reference Brier (0.132), is leakage-safe, uses only pre-race information, and produces calibrated probability outputs appropriate for strategy-risk decisions.

---

## AI Interaction 3 — Rejected suggestion: using strategy variables as direct predictors

### Context
During experimentation, we considered whether including strategy variables in the model would improve performance. We asked the AI for guidance before deciding.

### Prompt
> Would including `n_stops`, `compound_sequence`, and `stint_lengths` as standard predictive features in the Logistic Regression improve Brier score and ROC-AUC on the test set?

### Output
The AI confirmed that including these variables would very likely improve predictive performance because they are strongly associated with race outcome — teams that run 1-stop strategies on certain circuits tend to finish higher, and this signal is real. It suggested one-hot encoding `compound_sequence` and treating `n_stops` as an ordinal feature.

### Validation
We ran a quick experiment adding `n_stops` as a predictor. Brier score on the test set improved from 0.132 to approximately 0.118. However, we identified the critical problem: `n_stops` is a post-race observation in the raw data. At race start — the actual decision point — the strategist does not know how many stops will occur. Using it as a predictor makes the model look at the answer before making the prediction.

We re-read the assignment leakage rules, which state explicitly: "Strategy features such as n_stops, compound_sequence, and stint_lengths are post-race observations in the raw data. In any other context in this course, using them as predictors would be a textbook leakage error."

### Adaptations
We rejected the AI's suggestion. We excluded all strategy variables from the predictive pipeline and instead documented them as scenario inputs in:
- the leakage audit cell (Section 4),
- `framing.md` Section 4,
- and the what-if scenario comparison cell (Section 11).

This is a case where the AI gave technically correct advice for a different problem (maximizing accuracy on observed data) but the advice was wrong for our actual problem (predicting before the race without future information).

### Final Decision
Strategy variables remain excluded from the baseline model. The AI's suggestion to include them was explicitly rejected because it violates the assignment's leakage rules and would make the tool useless in a real deployment context where those values are unknown at decision time.
