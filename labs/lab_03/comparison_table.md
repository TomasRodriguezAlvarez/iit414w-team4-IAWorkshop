# Model Comparison Table

All models were evaluated using:

* Target: `points`
* Metric: Mean Absolute Error (MAE)
* Temporal split: Train (2019–2023), Test (2024)

| Model                       | Train MAE | Test MAE |    Gap |
| --------------------------- | --------: | -------: | -----: |
| Linear Regression           |     5.599 |    5.548 | -0.051 |
| Baseline 1 — Global Mean    |     5.613 |    5.552 | -0.061 |
| Baseline 2 — Grid Heuristic |     5.562 |    5.609 |  0.047 |
| XGBoost                     |     5.243 |    5.708 |  0.464 |
| Random Forest               |     4.927 |    5.860 |  0.934 |

## Model Interpretation (WHY)

* **Linear Regression**: Captures linear relationships between grid, driver, and constructor. Performs best due to stability and minimal overfitting, but improvement over baseline is small.

* **Baseline 1 — Global Mean**: Predicts the same value for all cases. Strong baseline because many drivers score zero points.

* **Baseline 2 — Grid Heuristic**: Uses starting position, which is an important predictor, but lacks additional race context.

* **XGBoost**: Models nonlinear patterns but does not generalize well due to limited feature information, showing moderate overfitting.

* **Random Forest**: Captures complex interactions but strongly overfits, memorizing training data instead of generalizing.

## Key Observations

* The best model is **Linear Regression**, but only marginally better than the baseline.
* The **Global Mean baseline is very strong**, indicating a difficult prediction problem.
* **Grid position is an important predictor**, but not sufficient alone.
* More complex models (**Random Forest, XGBoost**) show overfitting and do not improve performance.
* Missing variables (weather, incidents, strategy) limit predictive power.

## Conclusion

Increasing model complexity does not improve performance in this task.
Simple models provide more stable and reliable predictions given the available data.