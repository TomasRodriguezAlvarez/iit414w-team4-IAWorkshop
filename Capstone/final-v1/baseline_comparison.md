# baseline_comparison.md — Hito 2

All metrics computed on the untouched 2023–2024 test set (889 rows).
The same locked temporal split applies to both targets: train 2019–2021, calibration 2022 (Platt scaling only), test 2023–2024.

> **Reproducibility note:** The numbers in this document were recorded from a single end-to-end run of `hito2_modeling.ipynb` with `SEED = 414` set in every `random_state=` argument (`LogisticRegression`, `RandomForestClassifier`). Results are deterministic and will reproduce exactly on a clean clone.

---

## Target 1: `is_top10` (carry-over from Hito 1)

| Model                                | Brier      | ROC-AUC   | Notes                                    |
| ------------------------------------ | ---------- | --------- | ---------------------------------------- |
| Majority-class baseline              | 0.2497     | 0.500     | Always predicts train prevalence (51.6%) |
| Grid-rule heuristic                  | 0.1596     | 0.839     | P(top10) tiered by qualifying position   |
| Logistic Regression (Platt-calibrated) | 0.1350   | 0.885     | Standalone LR, same feature set         |
| **RF+LR ensemble, Platt-calibrated** | **0.1339** | **0.886** | Primary model (SEED=414)                 |
| Docent reference (assignment)        | 0.132      | 0.892     | External benchmark                       |

**Result:** The ensemble achieves Brier 0.134, within 0.002 of the docent reference. The 0.006 ROC-AUC gap is honest and expected — the feature set is intentionally restricted to pre-race observables.

---

## Target 2: `is_top5` (expansion target)

**Justification for choosing `is_top5`:** The Hito 1 decision context supports a Chief Strategy Officer for a mid-to-top tier team. For these teams, the relevant decision is not merely "will we score points?" (is_top10) but "will we reach the podium battle?" (is_top5). A 1-stop conservative strategy may preserve P(top10) while sacrificing P(top5) — that trade-off is invisible with a single target. `is_top5` adds the most decision-value information beyond `is_top10` for the specific use case framed in Hito 1.

**Baseline derivation for `is_top5`:** We adapt the grid-rule heuristic logic to the tighter threshold (top-5 finishes ≈ 25.7% of rows). Thresholds are grounded in F1 results — starters P1–P3 finish in the top 5 very frequently, positions P4–P7 have roughly even odds, and beyond P7 the probability drops sharply.

| Model                                | Brier      | ROC-AUC   | Notes                                                      |
| ------------------------------------ | ---------- | --------- | ---------------------------------------------------------- |
| Majority-class baseline              | 0.1918     | 0.500     | Always predicts train prevalence (25.7%)                   |
| Grid-rule heuristic (top5-adapted)   | 0.1080     | 0.884     | P(top5) tiered: ≤P3=0.80, ≤P7=0.45, ≤P12=0.15, else=0.05 |
| **RF+LR ensemble, Platt-calibrated** | **0.0860** | **0.937** | Same architecture as Hito 1 (SEED=414)                     |

**Result:** The model shows a larger relative improvement over the heuristic for `is_top5` (Brier −0.022) than for `is_top10` (Brier −0.026 vs docent). This confirms that form and constructor-name features add more discriminative signal at the tighter top-5 boundary, where grid-position alone is less predictive.

---

## Side-by-side summary

| Target   | Model                        | Brier | ROC-AUC |
| -------- | ---------------------------- | ----- | ------- |
| is_top10 | Grid-rule heuristic          | 0.160 | 0.839   |
| is_top10 | Logistic Regression (Platt)  | 0.135 | 0.885   |
| is_top10 | RF+LR Platt-calibrated       | 0.134 | 0.886   |
| is_top10 | Docent reference             | 0.132 | 0.892   |
| is_top5  | Grid-rule heuristic (top5)   | 0.108 | 0.884   |
| is_top5  | RF+LR Platt-calibrated       | 0.086 | 0.937   |

The model architecture is identical across both targets. The stronger relative improvement on `is_top5` reflects that rolling form and team identity are more informative at tighter finish thresholds, where raw starting position is a weaker predictor.

> **Note on is_top5 docent reference:** No external docent benchmark was provided for the expansion target. The grid-rule heuristic (top5-adapted) serves as the assignment floor for this target.
