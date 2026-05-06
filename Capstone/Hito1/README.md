# F1 Race Strategy Advisor — Hito 1

Decision-support baseline that estimates P(is_top10) for a driver-race combination using only pre-race information. Built for the Hito 1 capstone milestone.

---

## Repository structure

```
.
├── README.md
├── framing.md              ← decision context, metric justification, scenarios, limitations
├── PROMPTS.md              ← documented AI interactions (6-field standard)
├── hito1_baseline.ipynb    ← executable baseline notebook
└── f1_strategy_race_level.csv
```

---

## Install

Requires **Python 3.10+** and **Jupyter**.

```bash
pip install pandas numpy matplotlib scikit-learn jupyter
```

No other dependencies are needed.

---

## Run

Place `f1_strategy_race_level.csv` in the same directory as the notebook, then:

```bash
jupyter notebook hito1_baseline.ipynb
```

Select **Kernel → Restart & Run All**. The notebook runs end-to-end from a clean clone with no manual steps.

---

## Locked design decisions

| Decision | Value |
|---|---|
| Target | `is_top10` |
| Train | 2019, 2020, 2021 |
| Calibration | 2022 (Platt scaling only — never used for model selection) |
| Test | 2023, 2024 (accessed once, at final evaluation) |

---

## Notebook sections

| Section | What it does |
|---|---|
| 1. Load data | Reads CSV, prints shape |
| 2. Basic checks | Confirms required columns exist |
| 3. Locked temporal split | Creates train / calibration / test blocks |
| 4. Leakage audit | Classifies every column into: target, scenario input, audit/post-race, pre-race feature |
| 5. Grid-rule heuristic | F1-defensible baseline (positional tiers → probability) |
| 6. Feature engineering | Adds `grid_rank_inv`, `grid_x_tier`, `form_gap`, `circuit_history_missing` |
| 7. RF + LR ensemble + Platt calibration | Trains both models on 2019–2021; calibrates on 2022 |
| 8. Reference comparison | Compares against docent baseline (Brier 0.132, ROC-AUC 0.892) |
| 9. Calibration curve | Plots predicted vs. observed Top 10 rate on test set |
| 10. Result summary | Prints final metrics table |
| 11. What-if scenarios | Runs Scenario A (HAM Silverstone 2024) and Scenario B (LEC Monaco) |

---

## Leakage policy

| Category | Columns | Status |
|---|---|---|
| Pre-race features | `grid_position`, `qualifying_position`, `constructor_tier`, `circuit_type`, `constructor_name`, `driver_prior3_avg_finish`, `constructor_prior3_avg_finish`, `driver_circuit_prior_avg`, `round` | ✅ Used as predictors |
| Scenario inputs | `n_stops`, `compound_sequence`, `stint_lengths` | ⚠️ What-if inputs only — never predictors |
| Audit / post-race | `safety_car_periods`, `weather_actual`, `avg_track_temp`, pit timing columns | 🔍 Audit only |
| Target leakage | `finish_position`, `points`, `is_top10`, `dnf`, `status` | ❌ Never used as predictors |

---

## Expected outputs

Running all cells produces:

- Split shape summary (train / calibration / test row counts)
- Leakage audit table
- Grid-rule heuristic metrics
- Logistic regression, Random Forest, and ensemble metrics
- Calibration curve plot (test set)
- What-if scenario tables (Scenarios A and B)
- Final metric summary vs. docent reference
