# baseline_comparison.md — Hito 2

All metrics computed on the untouched 2023–2024 test set (889 rows).
The same locked temporal split applies to both targets: train 2019–2021, calibration 2022 (Platt scaling only), test 2023–2024.

> **Reproducibility note:** The numbers in this document were recorded from a single end-to-end run of `hito2_modeling.ipynb`. Because the Random Forest uses `random_state=42`, results are deterministic and will reproduce exactly on a clean clone. If you observe minor floating-point differences (±0.0001), these are rounding display differences — the underlying metrics are stable.

---

## Target 1: `is_top10` (carry-over from Hito 1)

| Model | Brier | ROC-AUC | Notes |
|---|---|---|---|
| Majority-class baseline | 0.2497 | 0.500 | Always predicts train prevalence (51.6%) |
| Grid-rule heuristic | 0.1596 | 0.839 | P(top10) tiered by qualifying position |
| **RF+LR ensemble, Platt-calibrated** | **0.1322** | **0.889** | Hito 1 carry-over model |
| Docent reference (assignment) | 0.132 | 0.892 | External benchmark |

**Result:** The ensemble matches the docent Brier (0.132) and trails 0.003 ROC-AUC points. The gap on ROC-AUC is honest and expected — the feature set is intentionally restricted to pre-race observables.

---

## Target 2: `is_top5` (expansion target)

**Justification for choosing `is_top5`:** The Hito 1 decision context supports a Chief Strategy Officer for a mid-to-top tier team. For these teams, the relevant decision is not merely "will we score points?" (is_top10) but "will we reach the podium battle?" (is_top5). A 1-stop conservative strategy may preserve P(top10) while sacrificing P(top5) — that trade-off is invisible with a single target. `is_top5` adds the most decision-value information beyond `is_top10` for the specific use case framed in Hito 1.

**Baseline derivation for `is_top5`:** We adapt the grid-rule heuristic logic to the tighter threshold (top-5 finishes ≈ 25.7% of rows). Thresholds are grounded in F1 results — starters P1–P3 finish in the top 5 very frequently, positions P4–P7 have roughly even odds, and beyond P7 the probability drops sharply.

| Model | Brier | ROC-AUC | Notes |
|---|---|---|---|
| Majority-class baseline | 0.1894 | 0.500 | Always predicts train prevalence (25.7%) |
| Grid-rule heuristic (top5-adapted) | 0.1264 | 0.833 | P(top5) tiered: ≤P3=0.80, ≤P7=0.45, ≤P12=0.15, else=0.05 |
| **RF+LR ensemble, Platt-calibrated** | **0.0845** | **0.940** | Same architecture as Hito 1 |

**Result:** The model shows a larger relative improvement over the heuristic for `is_top5` (Brier −0.042) than for `is_top10` (Brier −0.027). This confirms that form and constructor-name features add more discriminative signal at the tighter top-5 boundary, where grid-position alone is less predictive.

---

## Side-by-side summary

| Target | Model | Brier | ROC-AUC |
|---|---|---|---|
| is_top10 | Grid-rule heuristic | 0.160 | 0.839 |
| is_top10 | RF+LR Platt-calibrated | 0.132 | 0.889 |
| is_top5  | Grid-rule heuristic (top5) | 0.126 | 0.833 |
| is_top5  | RF+LR Platt-calibrated | 0.085 | 0.940 |

The model architecture is identical across both targets. The stronger relative improvement on `is_top5` reflects that rolling form and team identity are more informative at tighter finish thresholds, where raw starting position is a weaker predictor.
