# PROMPTS.md

## AI Interaction 1 — Leakage framing and temporal validation

### Context
We were building the Hito 1 baseline notebook for the F1 Race Strategy Advisor capstone. The assignment imposed strict temporal splits (train 2019–2021, calibration 2022, test 2023–2024) and emphasized avoiding leakage from post-race strategy variables.

### Prompt
> Review the assignment instructions and explain which variables should be treated as:
> - pre-race features,
> - scenario inputs,
> - target leakage / post-race observations.
>
> Also explain how the temporal split and calibration block should be implemented correctly.

### Output
The AI identified:
- acceptable pre-race features such as:
  - `qualifying_position`
  - `grid_position`
  - `constructor_tier`
  - `driver_prior3_avg_finish`
  - `constructor_prior3_avg_finish`
- strategy variables that must be framed as scenario inputs:
  - `n_stops`
  - `compound_sequence`
  - `stint_lengths`
- outcome/leakage variables that should never be used as predictors:
  - `finish_position`
  - `points`
  - `is_top10`
  - `dnf`
  - `status`

The AI also recommended:
- train = 2019–2021
- calibration = 2022 only
- test = 2023–2024 untouched

and explained that calibration should be learned only from the 2022 block.

### Validation
We manually checked the notebook feature lists to confirm:
- no target leakage columns were included,
- the split matched the assignment exactly,
- calibration used only the 2022 data.

### Adaptations
We expanded the leakage audit section into a dedicated notebook cell showing:
- target/outcome columns,
- scenario-input columns,
- audit/post-race columns.

We also clarified in `framing.md` that strategy variables are treated as user-controlled scenario inputs rather than magically known pre-race information.

### Final Decision
We adopted the leakage-safe framing and preserved the locked temporal split exactly as specified in the assignment.

---

## AI Interaction 2 — Baseline modeling and calibration strategy

### Context
After implementing a simple grid-rule heuristic baseline, we needed a stronger but still defendable Hito 1 baseline using only pre-race features.

### Prompt
> Suggest a simple but F1-defendable baseline model for predicting `is_top10`.
>
> The model must:
> - avoid leakage,
> - respect the locked temporal split,
> - support probability calibration,
> - and remain interpretable enough for Hito 1.

### Output
The AI suggested:
- Logistic Regression as a transparent baseline,
- optional Random Forest experimentation,
- Platt calibration using only the 2022 block,
- evaluation using:
  - Brier score,
  - Log loss,
  - ROC-AUC,
  - calibration curves.

It also suggested combining models into a calibrated ensemble if the results remained leakage-safe and reproducible.

### Validation
We executed:
- the grid-rule baseline,
- logistic regression,
- Platt-calibrated logistic regression,
- calibrated Random Forest,
- calibrated RF + LR ensemble.

We verified that:
- all models used only approved pre-race features,
- calibration used only 2022,
- evaluation was performed only once on 2023–2024.

The resulting ensemble achieved approximately:
- Brier ≈ 0.132
- ROC-AUC ≈ 0.892

which matched the docent reference baseline.

### Adaptations
We adjusted:
- feature engineering,
- preprocessing,
- ensemble weighting,
- and calibration handling.

We also retained the simpler logistic baseline in the notebook for transparency and comparison against the stronger ensemble approach.

### Final Decision
We selected the calibrated RF + LR ensemble as the main Hito 1 baseline because it:
- matched the docent reference performance,
- remained leakage-safe,
- used only pre-race information,
- and produced calibrated probability outputs appropriate for strategy-risk decisions.

---

## AI Interaction 3 — Rejected / modified suggestion

### Context
During experimentation, AI suggested including strategy variables directly in the predictive model.

### Prompt
> Would including `n_stops`, `compound_sequence`, and `stint_lengths` improve predictive performance?

### Output
The AI noted that these variables could improve predictive power because they are strongly associated with race outcomes.

### Validation
After reviewing the assignment leakage rules, we determined that directly treating these variables as ordinary predictors would create an invalid framing unless explicitly handled as scenario inputs.

### Adaptations
Instead of using them as standard pre-race predictors, we:
- excluded them from the baseline predictive pipeline,
- documented them as scenario-input variables,
- and moved their use into the what-if strategy framing.

### Final Decision
We rejected the original recommendation to use strategy variables as ordinary predictive features and instead followed the assignment’s leakage-safe scenario framing.