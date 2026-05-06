# Hito 1 — Problem Framing + Baseline

## 1. Decision Context

We are supporting the race strategy team of a mid-to-top tier Formula 1 team. The decision is made before the race, mainly on Sunday morning, with possible adjustments during the first part of the race.

The tool supports strategy planning by estimating the probability that a driver finishes in the Top 10. The main use case is comparing how different strategic assumptions may affect the chance of scoring points.

This is not a fully automated race strategy system. It is a decision-support baseline that helps the strategist reason about risk, probability, and scenario comparison.

---

## 2. Target & Primary Metric

The locked target is:

`is_top10`

This target indicates whether a driver finished inside the Top 10.

The primary metric is **Brier Score**, because the project depends on calibrated probabilities, not only correct classifications. In F1 strategy, it is important to know whether a Top 10 result is estimated as 55%, 75%, or 90%, because different probabilities imply different levels of risk.

We also report **Log Loss** and **ROC-AUC** to monitor probability quality and ranking performance.

---

## 3. Baseline Plan

The notebook implements the required locked temporal split:

- Train: 2019–2021
- Calibration: 2022
- Test: 2023–2024

The baseline pipeline starts with a simple grid-rule heuristic and then evaluates stronger leakage-safe models using only pre-race information.

The final best model in the notebook is:

`ensemble_rf70_lr30_platt_calibrated`

This model combines:

- 70% Random Forest probability
- 30% Logistic Regression probability
- Platt calibration using only the 2022 calibration block

The pre-race features include grid/qualifying position, constructor tier, circuit type, constructor name, and recent driver/constructor performance indicators. These are defensible because they are known before the race and reflect F1 logic: starting position, team strength, and recent form strongly influence the probability of a Top 10 finish.

On the untouched 2023–2024 test set, the notebook reports:

| Model | Brier | Log Loss | ROC-AUC |
|---|---:|---:|---:|
| Grid-rule baseline | 0.160 | 0.494 | 0.839 |
| Logistic baseline | 0.134 | 0.427 | 0.887 |
| Logistic + Platt | 0.135 | 0.429 | 0.887 |
| Random Forest + Platt | 0.132 | 0.424 | 0.887 |
| RF70/LR30 Ensemble + Platt | 0.132 | 0.422 | 0.889 |

The best model matches the calibrated docent reference Brier score of 0.132 and comes close to the docent ROC-AUC of 0.892.

---

## 4. What-if Comparison Plan

For Hito 1, the baseline model is trained only with pre-race features. For later strategy comparison, the tool can be extended to simulate scenario inputs such as pit stop count and tire sequence.

Two concrete scenarios are:

**Scenario A — Stop count strategy**

Lewis Hamilton at the British Grand Prix, starting P7.

Compare:

- 1-stop strategy: Medium → Hard
- 2-stop strategy: Medium → Medium → Hard

Goal: estimate whether the extra stop improves or reduces the probability of finishing Top 10.

**Scenario B — Compound choice strategy**

Charles Leclerc at the Monaco Grand Prix, starting P4.

Compare:

- aggressive soft-tire opening stint
- conservative hard-tire opening stint

Goal: estimate whether early pace or track-position protection is more valuable for Top 10 probability.

---

## 5. Dataset Limitations

We acknowledge the following limitations:

1. The dataset starts in 2019, so the model does not learn from older F1 seasons or previous regulation eras.

2. Some qualifying information is incomplete. For example, `qualifying_time_s` is not usable, so the model relies more heavily on grid and qualifying position.

3. Strategy features such as `n_stops`, `compound_sequence`, and `stint_lengths` are post-race observations in the raw data. We do not use them as ordinary pre-race predictors in the baseline. They are treated only as future scenario inputs for what-if analysis.

These limitations mean the model should be interpreted as a baseline decision-support tool, not as a deployment-ready strategy engine.

---

## 6. Experiments Planned for Hito 2

1. **Gradient boosting model**

Hypothesis: a boosted tree model may capture non-linear interactions between grid position, constructor strength, and circuit type better than the current baseline.

2. **Additional momentum features**

Hypothesis: rolling driver and constructor performance features may improve calibration when teams improve or decline during a season.

3. **Weather and race-condition audit**

Hypothesis: the importance of grid position may change under wet or disrupted race conditions, so model performance should be checked separately for those cases.

---

## 7. Team Workflow

For this milestone, the work was divided into:

- implementing the notebook and temporal split,
- building the leakage audit,
- testing the baseline models,
- writing the framing document,
- documenting AI assistance in `PROMPTS.md`,
- and checking that the repository runs end-to-end before submission.