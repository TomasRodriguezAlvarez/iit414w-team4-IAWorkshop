# Lab 2 — Feature Engineering + Improved Baseline

## Team 4
Tomás Rodríguez
Joaquín Rodríguez

## Overview
This project extends the work from **Lab 1**, where baseline models were defined for predicting whether a driver finishes in the **Top 10** of a Formula 1 race.

In this lab, we:
- Engineer new features based on domain knowledge
- Train a simple model (Logistic Regression)
- Compare performance against Lab 1 baselines using the same metric and temporal validation

## Prediction Task
- **Target:** `top10` (1 if driver finishes in Top 10, else 0)
- **Unit:** driver-race entry
- **Moment:** pre-race (only information available before the race is used)

## Dataset
Data is obtained using the **FastF1 API**, covering seasons:

- 2022 (train)
- 2023 (validation)
- 2024 (test — not used for model selection)

Key variables:
- `driver_id`
- `constructor_id`
- `grid`
- `position`
- `top10`
- `season`, `round`

## Feature Engineering
We created at least **3 new features** using domain knowledge:

1. **prev_position (Lag)**
   - Previous race finishing position per driver
   - Captures immediate past performance

2. **avg_position_last_3 (Rolling)**
   - Mean finishing position over last 3 races
   - Captures short-term consistency

3. **top10_rate_last_3 (Rolling)**
   - Proportion of Top-10 finishes in last 3 races
   - Directly models probability of success

All features:
- Use `.shift(1)` to ensure **no leakage**
- Are computed per driver (`groupby(driver_id)`)

## Model
We train a **Logistic Regression** model using the engineered features.

- Library: `sklearn`
- `random_state = 414`
- No complex models used (as required)

## Validation Strategy
We use a strict **temporal split**:

- Train: 2022
- Validation: 2023
- Test: 2024 (held out)

No random splitting is used to avoid leakage.

## Evaluation Metric
**Primary metric: F1 Score**

Chosen because:
- Balances precision and recall
- Both false positives and false negatives matter in this task

Other reported metrics:
- Accuracy
- Precision
- Recall
- ROC-AUC

## Baselines (Lab 1)
- Majority class baseline
- Domain heuristic: `grid <= 10`

These are used as comparison targets.

## Results
The Logistic Regression model improves over both baselines, especially the domain heuristic, demonstrating that engineered features provide additional predictive signal beyond starting grid position.

See:
- `comparison_table.md`

## Error Analysis
We identify three main failure modes:
1. Strong grid position but poor race outcome
2. Drivers gaining many positions during race
3. High variability in mid-field teams

Each case suggests directions for future feature engineering.

## Reproducibility

### Requirements
Install dependencies:

```bash
pip install -r requirements.txt