# Lab 3 — Model Comparison

## Overview

This project compares multiple machine learning models to predict the number of championship points a Formula 1 driver will score in a race.

The task is framed as a regression problem using historical race data from 2019 to 2024.

---

## Files

* `lab3_model_comparison.ipynb`: Main notebook with data processing, models, and results
* `framing_decision.md`: Justification of the prediction problem
* `comparison_table.md`: Summary of model performance
* `memo.md`: Non-technical summary of results

---

## Requirements

* Python 3.10+
* pandas
* numpy
* scikit-learn
* xgboost

Install dependencies:

```bash
pip install pandas numpy scikit-learn xgboost
```

---

## How to Run

1. Place the dataset `jolpica_2019_2024_results.csv` in the same directory as the notebook.
2. Open `lab3_model_comparison.ipynb`.
3. Run all cells from top to bottom.

---

## Reproducibility

* All models use `RANDOM_SEED = 414` for reproducibility.
* Temporal validation is used:

  * Train: 2019–2023
  * Test: 2024

---

## Notes

Results may vary slightly depending on the execution environment, but overall trends should remain consistent.
