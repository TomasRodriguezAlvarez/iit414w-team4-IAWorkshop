# error_analysis.md — Hito 2

All metrics computed on the untouched 2023–2024 test set (889 rows).
Brier score is reported per slice; lower = better calibration and discrimination.
Both `is_top10` and `is_top5` are analyzed side-by-side in each slice.

---

## Slice 1: Strategy type

| strategy_type | n | Brier (top10) | Brier (top5) | Actual top10 rate | Actual top5 rate |
|---|---|---|---|---|---|
| no_stop | 15 | **0.2198** | 0.0827 | 0.000 | 0.000 |
| one_stop | 353 | 0.1395 | 0.0880 | 0.561 | 0.283 |
| three_plus_stop | 153 | 0.1422 | **0.1007** | 0.405 | 0.196 |
| two_stop | 368 | **0.1174** | 0.0745 | 0.544 | 0.272 |

**Failure mode — no_stop (top10):** Brier 0.220, nearly 7× worse than two_stop. All 15 no-stop races ended outside the top 10 (actual rate = 0.000), but the model assigns non-trivial P(top10) based on grid position. No-stop strategies are virtually always forced by mechanical failures or accident damage early in the race — contexts where the pre-race features are uninformative about the outcome. The model has no signal that a no-stop occurred during the race.

**Failure mode — three_plus_stop (top5):** The model overestimates P(top5) for three-plus-stop races. These races often involve safety cars or unexpected tire failures that reshuffled the order; the model cannot anticipate those events pre-race. The actual top5 rate (19.6%) is below what the model predicts for the average driver in this slice.

**Why two_stop performs best:** Two-stop strategies are the modal planned strategy for mid-to-front-tier drivers under normal conditions. The model's training distribution is well-matched to this slice.

---

## Slice 2: Circuit type

| circuit_type | n | Brier (top10) | Brier (top5) | Actual top10 rate | Actual top5 rate |
|---|---|---|---|---|---|
| permanent | 579 | 0.1255 | 0.0866 | 0.518 | 0.259 |
| semi-street | 118 | **0.1782** | 0.0698 | 0.509 | 0.254 |
| street | 192 | 0.1240 | 0.0874 | 0.521 | 0.260 |

**Failure mode — semi-street (top10):** Brier 0.178, 42% higher than street circuits. Semi-street circuits (Singapore, Monaco's outer sectors, Miami, Baku) combine tight walls with high safety-car probability. The model's pre-race features — grid position, constructor tier, rolling form — are correct inputs, but their predictive relationship to finish position degrades sharply under safety-car conditions. Grid position at Baku, for example, is systematically less predictive than at Silverstone because the safety car frequently neutralizes multi-second gaps built over many laps.

**Observation:** `is_top5` at semi-street circuits has the *lowest* Brier (0.070) of any circuit type, which appears counterintuitive. The explanation is that top-5 finishes at semi-street circuits are heavily concentrated among front-tier constructors starting near the front; the model correctly assigns low P(top5) to the majority of the grid, making calibrated errors small even when the circuit is chaotic.

---

## Slice 3: Constructor tier

| constructor_tier | n | Brier (top10) | Brier (top5) | Actual top10 rate | Actual top5 rate |
|---|---|---|---|---|---|
| backmarker | 312 | 0.1223 | **0.0085** | 0.189 | 0.006 |
| front | 170 | 0.1063 | **0.1765** | 0.877 | 0.677 |
| midfield | 407 | **0.1506** | 0.1044 | 0.619 | 0.278 |

**Failure mode — front tier (top5):** Brier 0.177, more than 20× higher than backmarker for the same target. The model underestimates P(top5) for front-tier drivers, particularly those starting from positions 3–6 after grid penalties. This is the sharpest systematic bias in the model. The root cause is that the `constructor_tier` feature lumps all front-tier constructors together; in 2023–2024, the gap between Red Bull (dominant) and Ferrari/Mercedes (strong but inconsistent) was substantial and is not captured by a single tier label.

**Failure mode — midfield (top10):** Brier 0.151, the highest among tiers for top10. Midfield results in 2023–2024 were highly volatile — Aston Martin went from front-tier pace early in 2023 to genuine midfield by the end of the season, and McLaren moved in the opposite direction. Rolling 3-race averages lag these momentum shifts, producing calibration errors in the direction of the lagged tier.

**Backmarker (top5):** Near-zero Brier because the model correctly assigns near-zero P(top5) to backmarker drivers, who almost never crack the top 5 even under safety-car conditions.

---

## Slice 4: Weather

| weather_actual | n | Brier (top10) | Brier (top5) | Actual top10 rate | Actual top5 rate |
|---|---|---|---|---|---|
| dry | 768 | 0.1303 | 0.0799 | 0.521 | 0.260 |
| wet | 121 | **0.1442** | **0.1138** | 0.496 | 0.248 |

**Failure mode — wet (both targets):** Both targets show higher Brier in wet conditions. The effect is larger for `is_top5` (+0.034) than for `is_top10` (+0.014). Wet races produce more position-order volatility — drivers with strong wet-weather pace (e.g., Alonso, Verstappen) outperform their rolling averages systematically, while others underperform. The model has no pre-race wet-pace feature and cannot distinguish these drivers at the individual skill level (only at the constructor level). This is a known consequence of the dataset limitation: `weather_actual` is a post-race label and cannot be used as a pre-race predictor; a weather-forecast feature is required instead.

---

## Cross-slice: Strategy type × Circuit type (Brier, is_top10)

| | permanent | semi-street | street |
|---|---|---|---|
| no_stop | 0.1877 | 0.2698 | 0.2643 |
| one_stop | 0.1331 | 0.1897 | 0.1263 |
| three_plus_stop | 0.1469 | 0.0878 | 0.1149 |
| two_stop | 0.1078 | 0.1725 | 0.1142 |

**Key finding:** The two worst cells are no_stop × semi-street (0.270) and no_stop × street (0.264). These represent the combination of maximum model ignorance (no-stop signals race incident) with maximum circuit unpredictability (semi-street and street). A strategy advisor should explicitly flag these combinations as outside the model's reliable operating range.

**Three_plus_stop × semi-street (0.088)** is anomalously good — smaller than two_stop × semi-street. This likely reflects that three-plus-stop races at semi-street circuits are almost exclusively safety-car-induced, and after multiple stops the final order converges to track position of front-tier cars, which the model's constructor-tier features predict reliably.

---

## Summary of failure-mode hypotheses

| Failure mode | Target(s) affected | Root cause | Potential mitigation |
|---|---|---|---|
| No-stop races | top10 most | Race incidents not visible pre-race | Treat no-stop as OOD; flag and abstain |
| Semi-street circuits | top10 primarily | Safety-car volatility degrades grid-position signal | Safety-car-probability pre-race feature |
| Front-tier, positions 3–6 | top5 severely | Tier label too coarse; misses intra-tier gaps | Driver-specific or constructor-specific features |
| Midfield volatility | top10 | Mid-season car development not captured by 3-race rolling avg | Longer rolling window; momentum slope feature |
| Wet weather | both | No wet-pace feature; uses dry-race form | Pre-race weather forecast feature (scenario input) |
