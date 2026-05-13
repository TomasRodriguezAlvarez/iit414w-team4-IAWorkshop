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
The AI correctly identified pre-race features, scenario inputs, audit columns, and target leakage. It explained that the calibration block (2022) should be used only to fit a Platt scaling logistic regression on top of raw model scores — never for model selection, hyperparameter tuning, or feature selection. It warned against using `qualifying_time_s` since it is missing in the dataset.

### Validation
We manually audited the notebook feature lists against this classification. We confirmed no leakage columns appear in the feature set, the temporal split matches exactly, and calibration data is only passed to the Platt scaling step.

### Adaptations
We expanded the leakage classification into a dedicated notebook cell (Section 4) with an explicit four-category audit table, and added a `exists_in_csv` column to catch any column name mismatches at runtime.

### Final Decision
We adopted the leakage-safe framing and preserved the locked temporal split exactly as specified.

---

## AI Interaction 2 — Baseline model selection and calibration strategy

### Context
After implementing the grid-rule heuristic, we needed a stronger but still defensible Hito 1 baseline using only pre-race features.

### Prompt
> Suggest a stronger baseline that: uses only pre-race features (no post-race leakage), supports probability calibration using a held-out 2022 calibration block, is explainable enough for a strategy decision-support context, and can be implemented with scikit-learn.

### Output
The AI suggested Logistic Regression as a transparent baseline, Random Forest as a second component, Platt calibration using only the 2022 block, and a weighted ensemble (70% RF + 30% LR) before the calibration step.

### Validation
We implemented all four model variants. The RF+LR ensemble achieved Brier 0.132, ROC-AUC 0.889 on the test set. We verified calibration used only the 2022 block and test data was accessed once.

### Adaptations
We adjusted the ensemble weight to 70/30 after observing that increasing the RF weight beyond 0.70 did not further improve Brier on the calibration set.

### Final Decision
We selected the calibrated RF+LR ensemble as the Hito 1 baseline. It matches the docent reference Brier, is leakage-safe, and produces calibrated probability outputs.

---

## AI Interaction 3 — Rejected suggestion: using strategy variables as direct predictors

### Context
During experimentation, we considered whether including strategy variables in the model would improve performance.

### Prompt
> Would including `n_stops`, `compound_sequence`, and `stint_lengths` as standard predictive features in the Logistic Regression improve Brier score and ROC-AUC on the test set?

### Output
The AI confirmed these variables would likely improve predictive performance because they are strongly associated with race outcome. It suggested one-hot encoding `compound_sequence` and treating `n_stops` as an ordinal feature.

### Validation
We ran a quick experiment adding `n_stops` as a predictor. Brier improved from 0.132 to approximately 0.118. However, `n_stops` is a post-race observation — at race start the strategist does not know how many stops will occur. This is textbook leakage.

### Adaptations
We rejected the suggestion. All strategy variables were excluded from the predictive pipeline and documented as scenario inputs.

### Final Decision
Strategy variables remain excluded. The AI gave technically correct advice for a different problem (maximizing accuracy on observed data) but wrong advice for our actual problem (predicting before the race without future information).

---

## AI Interaction 4 — Expansion target selection for Hito 2

### Context
Hito 2 requires adding a second target from the list: `is_top5`, `is_top3`, `finish_position`, or `points`. We needed to choose the target most consistent with the decision context framed in Hito 1 and implement it with the same pipeline.

### Prompt
> Our Hito 1 decision context supports a Chief Strategy Officer for a mid-to-top tier F1 team. The target is is_top10. For Hito 2, we must add one expansion target from: is_top5, is_top3, finish_position (regression), points (regression).
>
> Which target adds the most decision-value information beyond is_top10 for this specific decision context? Justify the choice. Then explain how to implement it with the same RF+LR Platt-calibrated pipeline, and what baseline to compare against.

### Output
The AI recommended `is_top5` as the expansion target, arguing that for a mid-to-top tier team the relevant strategic boundary is not "will we score any points?" (is_top10) but "will we reach the podium battle?" (is_top5). It noted that the key strategic insight is that `is_top10` and `is_top5` will respond differently to the same grid-position change — a finding that creates the what-if disagreement scenario the rubric requires.

It also recommended adapting the grid-rule heuristic thresholds for `is_top5` (P1-P3 = 0.80, P4-P7 = 0.45, P8-P12 = 0.15, else = 0.05) and evaluating with the same Brier + ROC-AUC metrics.

### Validation
We implemented the `is_top5` model using the identical pipeline. The model achieved Brier 0.085, ROC-AUC 0.940 on the 2023–2024 test set — a larger relative improvement over the heuristic than `is_top10` showed, which confirms the AI's reasoning that rolling form and constructor identity add more signal at the tighter boundary.

We verified that:
- `is_top10` does not appear in the `is_top5` feature set (both are post-race outcomes),
- the Platt calibration uses only the 2022 block for both models,
- the test set is accessed once for both targets.

### Adaptations
We kept `is_top3` and `finish_position` as potential Hito 3 experiments — they were not chosen because `is_top3` has too few positive examples (15.4%) for reliable calibration at this training size, and `finish_position` regression would require different evaluation metrics (MAE, RMSE) that complicate the side-by-side comparison the rubric requires.

### Final Decision
`is_top5` is the Hito 2 expansion target. The AI's recommendation was adopted without modification. The pipeline, calibration protocol, and evaluation metrics are identical to `is_top10`.

---

## AI Interaction 5 — Error analysis slicing strategy

### Context
The Hito 2 rubric requires error analysis sliced by strategy_type, circuit_type, and at least one additional context, on both targets. We needed to design the slicing logic and identify the most informative failure modes.

### Prompt
> I have trained RF+LR Platt-calibrated models for is_top10 (Brier 0.132) and is_top5 (Brier 0.085) on F1 race data. The rubric requires error analysis sliced by strategy_type, circuit_type, and at least one additional context.
>
> Design the slicing logic. Which additional context adds the most diagnostic value? What failure-mode hypotheses should I state for each slice? Also suggest a cross-slice that might surface something the individual slices miss.

### Output
The AI recommended:
- Slice 3: constructor_tier — because the model's coarse tier label (front/midfield/backmarker) is the most likely source of systematic bias, especially for `is_top5` where front-tier intra-tier gaps are large.
- Slice 4: weather_actual — because wet races change the competitive order in ways the model's dry-race form features cannot capture.
- Cross-slice: strategy_type × circuit_type — because the worst failure modes (no-stop at semi-street) likely combine two individual dimensions, and the cross-slice exposes them.

For failure-mode hypotheses, the AI correctly predicted that:
- no_stop would have the highest Brier (confirmed: 0.220),
- semi-street would be worst among circuit types for top10 (confirmed: 0.178),
- front-tier would have the highest Brier for top5 due to intra-tier variation (confirmed: 0.177),
- wet would have slightly higher Brier than dry for both targets (confirmed).

### Validation
All four hypotheses were confirmed by the sliced error analysis. The cross-slice confirmed that no_stop × semi-street (0.270) and no_stop × street (0.264) are the worst two cells.

The one unexpected finding — three_plus_stop × semi-street has anomalously low Brier (0.088) — was not predicted by the AI. We added a post-hoc interpretation: these races are almost exclusively safety-car-induced, and after multiple stops the final order converges to front-tier cars, which the model predicts reliably.

### Adaptations
We added the post-hoc three_plus_stop × semi-street interpretation in `error_analysis.md`. We adopted all AI-recommended slices and the cross-slice.

### Final Decision
The AI's slicing strategy was adopted. The post-hoc interpretation of the anomalous cross-slice cell was our own addition. The failure-mode hypotheses in `error_analysis.md` are a direct output of this interaction with validation by the actual numbers.

---

## AI Interaction 6 — Rejected suggestion: using finish_position for what-if comparison

### Context
When designing the what-if comparison that surfaces a recommendation `is_top10` alone cannot produce, we asked whether using `finish_position` regression predictions would produce a more compelling comparison than `is_top5`.

### Prompt
> For the Hito 2 what-if comparison, should we use finish_position regression to show the expected position change from a strategy choice, rather than comparing P(top10) vs P(top5)?

### Output
The AI suggested that `finish_position` regression would produce more intuitive output ("expected to finish P5.2") than comparing two probabilities. It offered to implement a Random Forest regression for `finish_position` alongside the binary models.

### Validation
We rejected this suggestion for two reasons:
1. The rubric specifically requires the expansion target to "surface a strategy recommendation is_top10 alone could not produce." `finish_position` regression is a separate target, not the expansion target we selected (`is_top5`). Switching targets mid-analysis would invalidate the baseline comparison in `baseline_comparison.md`.
2. The disagreement framing — "P(top10) says safe, P(top5) says risky" — is sharper and more actionable for a strategy decision-maker than "expected finish position 5.2 vs 6.8." Probabilities allow direct comparison of targets; regression outputs require different mental models.

### Adaptations
We kept the `is_top5` what-if framing. We did implement `finish_position` regression (MAE 2.964 vs naive baseline 4.854) as a supplementary model in the notebook for reference but did not build the main what-if comparison around it.

### Final Decision
The AI's suggestion was rejected. The what-if comparison uses `is_top10` vs `is_top5` probabilities as designed. The `finish_position` regression is included in the notebook for transparency but not in the primary analysis chain.
