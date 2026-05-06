# PROMPTS.md

## AI Interaction 1

### Context
We were starting the Hito 1 notebook and needed to understand the assignment requirements, especially the locked temporal split, leakage rules, and acceptable baseline design.

### Prompt
> Explain the assignment requirements and identify:
> - the correct temporal split,
> - which variables are leakage,
> - which variables can be used as pre-race features,
> - and what kind of baseline is acceptable for Hito 1.

### Output
The AI explained that the required split was:
- train: 2019–2021
- calibration: 2022
- test: 2023–2024

It also identified:
- post-race leakage variables such as:
  - `finish_position`
  - `points`
  - `is_top10`
  - `dnf`
  - `status`
- strategy variables that should only be treated as scenario inputs:
  - `n_stops`
  - `compound_sequence`
  - `stint_lengths`
- acceptable pre-race features such as:
  - `grid_position`
  - `qualifying_position`
  - `constructor_tier`
  - `driver_prior3_avg_finish`

The AI also suggested using Logistic Regression as a simple F1-defendable baseline.

### Validation
We manually checked the notebook feature lists and verified that:
- leakage variables were excluded,
- the split matched the assignment exactly,
- and calibration used only the 2022 block.

### Adaptations
We added a dedicated leakage audit section in the notebook to explicitly separate:
- pre-race features,
- scenario-input variables,
- and post-race audit columns.

### Final Decision
We adopted the locked temporal split and used only leakage-safe pre-race variables in the predictive baseline.

---

## AI Interaction 2

### Context
After implementing the first logistic baseline, we wanted to improve performance while remaining simple and leakage-safe.

### Prompt
> Suggest simple improvements to the baseline model while keeping the notebook explainable and consistent with the assignment requirements.

### Output
The AI suggested:
- trying Random Forest in addition to Logistic Regression,
- using Platt calibration on the 2022 block,
- and combining both models into a weighted ensemble.

It also recommended evaluating:
- Brier Score,
- Log Loss,
- ROC-AUC,
- and calibration curves.

### Validation
We implemented:
- Logistic Regression,
- Random Forest,
- Platt calibration,
- and an RF/LR ensemble.

We verified that:
- only pre-race features were used,
- the test set remained untouched until final evaluation,
- and calibration used only the 2022 split.

### Adaptations
We experimented with different ensemble weights and kept the final configuration:

`ensemble_rf70_lr30_platt_calibrated`

because it produced the best Brier score on the untouched test set.

### Final Decision
We selected the calibrated RF/LR ensemble as the final Hito 1 baseline because it:
- matched the docent Brier baseline,
- remained leakage-safe,
- and produced calibrated probabilities suitable for strategy-risk estimation.

---

## AI Interaction 3

### Context
During experimentation, we considered using strategy variables directly as predictors.

### Prompt
> Would including `n_stops`, `compound_sequence`, and `stint_lengths` improve predictive performance?

### Output
The AI explained that those variables would likely improve predictive performance because they are strongly related to race outcome.

### Validation
After reviewing the assignment leakage policy, we confirmed that these variables are post-race observations in the raw dataset.

### Adaptations
Instead of using them as standard predictive features, we documented them as:
- scenario-input variables,
- intended for future what-if comparisons.

They were excluded from the Hito 1 predictive pipeline.

### Final Decision
We rejected the use of strategy variables as ordinary predictors and kept the final baseline restricted to leakage-safe pre-race information.