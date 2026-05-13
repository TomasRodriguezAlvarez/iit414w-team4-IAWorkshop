# mitigations.md — Hito 2: Risks, Failure Modes, and Mitigations

Each risk is connected to a specific failure mode identified in `error_analysis.md`. Mitigations are ordered by feasibility for the Final Report.

---

## Risk 1: No-stop races (race-incident out-of-distribution)

**Failure mode identified in:** Error analysis Slice 1 — no_stop Brier = 0.220 (top10).

**Description:** No-stop races almost exclusively occur after early mechanical failures, accidents causing car damage, or retirement threats. The model has no pre-race signal that a no-stop will occur and assigns non-trivial P(top10) based on grid position. All 15 no-stop cases in the test set resulted in a finish outside the top 10 (actual rate = 0.000).

**Consequence:** The model would actively mislead a strategist if it were used to advise on a car that has already suffered damage — it would show "85% top-10 chance" while the driver is actually limping to the pits.

**Mitigation before deployment:**
1. **OOD flagging:** Any scenario where the strategy input indicates no planned pit stop should trigger a model abstention or a warning label. The tool should say "this scenario is outside the model's training distribution" rather than producing a probability.
2. **Race-phase gating:** The model should be gated on lap count — if fewer than 3 laps have occurred, the pre-race model applies. If a pit-stop window has passed without a stop, a real-time model (outside this capstone's scope) is required.

---

## Risk 2: Semi-street circuit volatility

**Failure mode identified in:** Error analysis Slice 2 — semi-street Brier = 0.178 (top10), 42% worse than street.

**Description:** Semi-street circuits (Baku, Miami, Monaco outer sector) combine high safety-car probability with tight walls. The grid-position signal degrades sharply when a safety car neutralizes multi-second gaps. The model cannot anticipate safety-car probability pre-race.

**Consequence:** The model will overstate P(top10) for back-of-grid drivers at semi-street circuits (safety car "free stops" often bring them into points contention) and understate volatility for mid-grid drivers.

**Mitigation:**
1. **Circuit-specific safety-car probability feature:** Pre-race historical safety-car rates per circuit are available from public F1 data. Adding a `circuit_safety_car_rate` feature (computed from training seasons only) would allow the model to discount grid-position signal proportionally to circuit volatility. This is a legitimate pre-race feature — it uses prior-season averages, not the race's own safety car occurrence.
2. **Wider prediction intervals at semi-street circuits:** Until a proper feature is added, the tool should display wider confidence bands for semi-street circuit predictions.

---

## Risk 3: Front-tier intra-tier gap (is_top5 bias)

**Failure mode identified in:** Error analysis Slice 3 — front-tier Brier = 0.177 for `is_top5`.

**Description:** The model lumps all front-tier constructors together. In 2023–2024 the performance gap between Red Bull and Ferrari/Mercedes was large and inconsistent across circuits. A driver labeled "front-tier" could be in the dominant car or a car 0.5s/lap off pace — the model treats them identically.

**Consequence:** The model systematically underestimates P(top5) for the dominant constructor (Red Bull in 2023) and overestimates for others. This is the most severe systematic bias in the `is_top5` target.

**Mitigation:**
1. **Constructor-specific features:** Replace the three-category `constructor_tier` with `constructor_name` rolling averages at a finer grain — e.g., a constructor-season-level average finish rather than a static tier label. This is already partially addressed by including `constructor_name` as a categorical feature, but the rolling average for the dominant constructor still lags mid-season developments.
2. **Momentum slope feature:** Compute the slope of the constructor's average finish over the last 5 races (not just the 3-race average level). A constructor improving from P5 average to P2 average over 5 races should receive a different probability than one that has been stable at P3.

---

## Risk 4: Mid-season car development (midfield volatility)

**Failure mode identified in:** Error analysis Slice 3 — midfield Brier = 0.151 (top10), highest of three tiers.

**Description:** Midfield results in 2023–2024 were highly volatile. Aston Martin declined from near-front-tier pace to genuine midfield across the season; McLaren moved in the opposite direction. The 3-race rolling average lags these shifts by weeks — during the transition period the model predicts the wrong tier of performance.

**Consequence:** The model will produce confidently wrong probabilities during a team's performance transition. A McLaren driver mid-2023 would receive a "midfield" probability when the car was approaching front-tier pace.

**Mitigation:**
1. **Slope feature (same as Risk 3 mitigation):** The direction of change in the rolling average carries information the level alone does not.
2. **Season-week interaction:** Add a `round × constructor_tier` interaction term to allow the model to learn that early-season tier labels are more reliable than late-season ones, where development has had more time to alter the competitive order.

---

## Risk 5: Wet weather — no pre-race wet-pace signal

**Failure mode identified in:** Error analysis Slice 4 — wet Brier = 0.144 (top10), 0.114 (top5).

**Description:** Wet conditions systematically change the competitive order. Drivers with strong wet-pace (Alonso, Verstappen, historically Schumacher) outperform their rolling averages; others underperform. The model uses dry-race form features and has no wet-specific signal.

**Consequence:** In wet races, the model's predictions revert to "who starts near the front with good dry-race form" — missing the wet-pace dimension entirely.

**Mitigation:**
1. **Pre-race weather forecast as a scenario input:** Rain probability from a public forecast API for the race location on race day is available pre-race. Adding `forecast_wet_probability` (0–1) as a scenario input allows the strategist to run "what if it rains?" comparisons with a model that has been trained on weather-stratified data.
2. **Wet-race driver rating:** Compute each driver's historical average position gain/loss in wet vs dry races (from training seasons only) and add it as a pre-race feature. This is a legitimate historical signal that does not use race-day information.

---

## Risk 6: Deployment context mismatch

**Description:** The model is trained on observed race data where strategy choices are endogenous (fast cars choose aggressive strategies). Deploying the model to advise on a strategy choice treats the model as if it were trained on randomized strategy assignments.

**Consequence:** Any claimed "probability improvement from switching strategy" that does not hold the car and driver fixed is confounded. The model cannot estimate "if this midfield car used a two-stop instead of one-stop, how much would P(top10) change?" — it can only estimate "given this car's pre-race characteristics, what is the baseline P(top10)?"

**Mitigation:**
1. **Explicit framing in the tool UI:** Every output probability should be labeled "baseline P(top10) given pre-race conditions" and never "expected P(top10) under this strategy."
2. **Causal disclaimer in all deliverables:** The scenario comparison tool compares pre-race conditions, not causal strategy effects. This limitation is disclosed in `leakage_audit.md`, `framing.md`, and the notebook output cells.
3. **Longer-term: causal modeling:** A proper causal approach would require either randomized strategy assignment (impossible in practice) or instrumental variable methods using natural experiments such as safety-car windows. This is outside scope for the capstone but named as the required path to a genuinely deployable tool.

---

## Summary table

| Risk | Failure mode | Severity | Mitigation feasibility |
|---|---|---|---|
| No-stop OOD | no_stop Brier = 0.220 | High | Immediate: OOD flag |
| Semi-street volatility | semi-street Brier = 0.178 | Medium | Medium-term: safety-car rate feature |
| Front-tier intra-gap | is_top5 front Brier = 0.177 | High | Medium-term: slope + constructor features |
| Midfield volatility | midfield top10 Brier = 0.151 | Medium | Medium-term: slope feature |
| Wet weather | wet Brier = 0.144 / 0.114 | Medium | Short-term: forecast scenario input |
| Causal confounding | All predictions | Structural | Long-term: causal modeling (not in scope) |

We do not recommend deploying this tool without addressing at minimum the no-stop OOD flag (Risk 1) and the causal framing disclaimer (Risk 6). Risks 2–5 degrade prediction quality in specific contexts but do not invalidate the tool's core use case.
