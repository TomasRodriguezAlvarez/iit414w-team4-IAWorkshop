# F1 Race Strategy Advisor — Hito 1

This repository contains the Hito 1 deliverables for the F1 Race Strategy Advisor capstone project.

The goal of this milestone is to:
- frame the decision problem,
- implement a leakage-safe baseline,
- apply the locked temporal split,
- calibrate probability outputs,
- and evaluate the model on the untouched 2023–2024 test set.

---

# Repository Structure

```text
.
├── framing.md
├── PROMPTS.md
├── README.md
├── hito1_baseline.ipynb
└── f1_strategy_race_level.csv

```
# Files
framing.md

Contains:
- decision context,
- target and metric justification,
- baseline rationale,
- what-if scenarios,
- dataset limitations,
- Hito 2 experiment ideas.

# hito1_baseline.ipynb

Executable notebook implementing:
- data loading,
- temporal split,
- leakage audit,
- heuristic baseline,
- Logistic Regression baseline,
- Random Forest baseline,
- RF/LR ensemble,
- Platt calibration,
- evaluation metrics,
- calibration curve.

# PROMPTS.md

Documents AI-assisted interactions used during development:
- prompts,
- outputs,
- validation,
- adaptations,
- and final decisions.

# Dataset

The notebook expects the dataset:
```python
f1_strategy_race_level.csv
```
to be located in the same folder as the notebook.

# Locked Design Decisions
The notebook follows the assignment constraints exactly:

|Split|Seasons|
|---|---|
|Train|2019–2021|
|Calibration|2022|
|Test|2023–2024|

Target:
```python
is_top10
```

# Baseline Summary

The notebook evaluates:
- a simple grid-rule heuristic baseline,
- Logistic Regression,
- Random Forest,
- and a calibrated RF/LR ensemble.

The final selected model is:
```python
ensemble_rf70_lr30_platt_calibrated
```
Probability calibration is performed using only the 2022 calibration block.

Main evaluation metrics:
- Brier Score,
- Log Loss,
- ROC-AUC,
- calibration curve.

# Leakage Policy

The project explicitly separates:

## Pre-race features

Used as predictive inputs:
- grid position,
- qualifying position,
- constructor tier,
- circuit type,
- recent driver/constructor performance.

## Scenario-input variables

Reserved for future what-if strategy comparisons:
- n_stops
- compound_sequence
- stint_lengths

## Forbidden leakage variables

Never used as predictors:
- finish_position
- points
- is_top10
- dnf
- status

# Installation

Recommended environment:
- Python 3.10+
- Jupyter Notebook

Install dependencies:
```python
pip install pandas numpy matplotlib scikit-learn
```

# Running the Notebook

Launch Jupyter:
```python
jupyter notebook
```

Open:
```python
hito1_baseline.ipynb
```

Then run all cells from top to bottom.
The notebook is designed to run end-to-end from a clean clone.

# Expected Outputs

The notebook produces:
- split summaries,
- leakage audit table,
- baseline comparison metrics,
- calibrated evaluation metrics,
- calibration curve plot,
- short performance summary.