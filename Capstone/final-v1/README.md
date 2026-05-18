# F1 Race Strategy Advisor — Final Report

Decision-support model that estimates P(is_top10) and P(is_top5) for a driver-race combination using only pre-race information. This repository contains all artifacts for Hito 1, Hito 2, and the Final Report submission.

---

## Repository structure

```
.
├── README.md
├── PROMPTS.md                  ← documented AI interactions (6-field standard, interactions 1–9)
├── hito1_baseline.ipynb        ← Hito 1 baseline (is_top10 only, carry-over)
├── hito2_modeling.ipynb        ← Hito 2 notebook (both targets, error analysis, what-if)
├── baseline_comparison.md      ← model vs baselines on both targets
├── error_analysis.md           ← sliced error analysis (strategy, circuit, tier, weather)
├── whatif_comparison.md        ← strategy trade-off invisible to is_top10 alone
├── leakage_audit.md            ← leakage classification + confounding treatment
├── mitigations.md              ← risks, failure modes, deployment conditions
├── framing.md                  ← decision context, metric justification, scenarios, limitations
└── f1_strategy_race_level.csv
```

---

## Install

Requires **Python 3.10+** and **Jupyter**.

```bash
pip install pandas numpy matplotlib scikit-learn jupyter
```

No other dependencies are needed. No internet access required at runtime.

---

## Run

### Runbook (execute in order)

**Step 1 — Hito 1 baseline (carry-over)**

```bash
jupyter notebook hito1_baseline.ipynb
```

Select **Kernel → Restart & Run All**.

Expected outputs:
- `is_top10` metrics table (Brier ≈ 0.134, ROC-AUC ≈ 0.886)
- Calibration curve for `is_top10`

**Step 2 — Hito 2 (both targets + error analysis + what-if)**

```bash
jupyter notebook hito2_modeling.ipynb
```

Select **Kernel → Restart & Run All**. The notebook is self-contained and runs end-to-end from a clean clone.

Expected outputs:
- `is_top10` metrics table (Brier ≈ 0.134, ROC-AUC ≈ 0.886)
- `is_top5` metrics table (Brier ≈ 0.086, ROC-AUC ≈ 0.937)
- Side-by-side comparison table (both targets vs heuristics)
- Calibration curve plots (both targets, side-by-side)
- Error analysis tables (4 slices + cross-slice pivot)
- What-if scenario table: P2 → P(top10)=0.909, P(top5)=0.842; P7 → P(top10)=0.810, P(top5)=0.541
- Final metric summary

---

## Reproducibility

| Parameter         | Value                                          |
| ----------------- | ---------------------------------------------- |
| `RANDOM_SEED`     | `414` — set as `SEED = 414` at top of notebook |
| `random_state`    | `SEED` in every `RandomForestClassifier` and `LogisticRegression` call |
| Python version    | 3.10+                                          |
| Key dependencies  | pandas, numpy, scikit-learn, matplotlib, jupyter |
| Data path         | `f1_strategy_race_level.csv` (repo root)        |
| Test set access   | Once only — final evaluation cell              |

---

## Locked design decisions

| Decision                           | Value                                                 |
| ---------------------------------- | ----------------------------------------------------- |
| Primary target (Hito 1 carry-over) | `is_top10`                                            |
| Expansion target (Hito 2)          | `is_top5`                                             |
| Train                              | 2019, 2020, 2021                                      |
| Calibration                        | 2022 (Platt scaling only — never for model selection) |
| Test                               | 2023, 2024 (accessed once, at final evaluation only)  |
| Ensemble weights                   | 70% RF + 30% LR (chosen on calibration set)           |
| RF hyperparams                     | n_estimators=100, max_depth=8, random_state=SEED       |
| LR hyperparams                     | C=1.0, max_iter=1000, random_state=SEED               |

---

## Hito 2 notebook sections

| Section                    | What it does                                                                     |
| -------------------------- | -------------------------------------------------------------------------------- |
| 1. Load data + split       | Reads CSV, applies locked temporal split, verifies both targets                  |
| 2. Feature engineering     | Same pre-race features as Hito 1; applied to all splits                          |
| 3. Shared helpers          | `evaluate_binary()`, `fit_binary_ensemble()`, `platt_calibrate()`                |
| 4. `is_top10` (carry-over) | Re-trains Hito 1 model; reports Brier 0.134, ROC-AUC 0.886                       |
| 5. `is_top5` (expansion)   | Trains same architecture on expansion target; reports Brier 0.086, ROC-AUC 0.937 |
| 6. Side-by-side comparison | Compares both models against their respective heuristic baselines                |
| 7. Calibration curves      | Predicted vs observed rate for both targets (side-by-side figure)                |
| 8. Error analysis          | Brier sliced by strategy_type, circuit_type, constructor_tier, weather_actual    |
| 9. Cross-slice             | strategy_type × circuit_type pivot for both targets                              |
| 10. What-if comparison     | Hamilton Silverstone 2024: the P(top5) drop that is_top10 cannot see             |
| 11. Summary                | Final metric table for both targets                                              |

---

## Key finding

**A grid drop from P2 to P7 at Silverstone costs 10pp on P(top10) but 30pp on P(top5).**

`is_top10` alone signals: "Hamilton still has an 81% chance of scoring points — no urgency."
`is_top5` reveals: "The podium challenge is effectively gone (54%). A 2-stop M-M-H is justified to recover track position — not to protect points, but to re-enter the top-5 battle."

This recommendation is structurally invisible from a single target. See `whatif_comparison.md` for full analysis.

---

## Leakage policy

| Category            | Columns                                                                                                                                                                                          | Status                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------- |
| Pre-race features   | `grid_position`, `qualifying_position`, `constructor_tier`, `circuit_type`, `constructor_name`, `driver_prior3_avg_finish`, `constructor_prior3_avg_finish`, `driver_circuit_prior_avg`, `round` | Used as predictors           |
| Engineered features | `grid_rank_inv`, `grid_x_tier`, `driver_constructor_avg`, `form_gap`, `circuit_history_missing`, `tier_num`                                                                                      | Derived from pre-race inputs |
| Scenario inputs     | `n_stops`, `compound_sequence`, `stint_lengths`                                                                                                                                                  | What-if labels only          |
| Audit / post-race   | `safety_car_periods`, `weather_actual`, `avg_track_temp`, pit timing columns                                                                                                                     | Error slicing only           |
| Target leakage      | `finish_position`, `points`, `is_top10`, `is_top5`, `dnf`, `status`                                                                                                                              | Never predictors             |
