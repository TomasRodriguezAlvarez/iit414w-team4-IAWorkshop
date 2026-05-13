# whatif_comparison.md — Hito 2

This document demonstrates the core strategic insight that `is_top5` adds over `is_top10` alone:
**a strategy that preserves points-scoring probability may sacrifice podium-challenge probability,
and that trade-off is invisible when only modeling the top-10 boundary.**

---

## The disagreement scenario: Hamilton, British GP 2024 (Silverstone)

**Context:** Lewis Hamilton, Mercedes, 2024 British Grand Prix, round 12.
Feature values drawn from the actual 2024 British GP dataset row.

| Feature                         | Value                               |
| ------------------------------- | ----------------------------------- |
| `grid_position`                 | 2 (baseline) / 7 (dropped scenario) |
| `driver_prior3_avg_finish`      | 3.67                                |
| `constructor_prior3_avg_finish` | 3.17                                |
| `driver_circuit_prior_avg`      | 1.83                                |
| `constructor_tier`              | front                               |
| `circuit_type`                  | permanent                           |
| `round`                         | 12                                  |

### What the two targets say

| Scenario     | Grid | Strategy label   | P(top10)  | P(top5)   |
| ------------ | ---- | ---------------- | --------- | --------- |
| Front row    | P2   | n_stops=1, M-H   | **0.922** | **0.884** |
| Grid dropped | P7   | n_stops=1, M-H   | **0.854** | **0.678** |
| Grid dropped | P7   | n_stops=2, M-M-H | **0.854** | **0.678** |

_(Note: n_stops is a scenario input label — not a model feature. The pre-race probabilities do not change when only the strategy label changes; they change when grid position or other pre-race features change.)_

### The disagreement — quantified

| Metric   | P2    | P7    | Change | Relative drop |
| -------- | ----- | ----- | ------ | ------------- |
| P(top10) | 0.922 | 0.854 | −0.068 | −7.4%         |
| P(top5)  | 0.884 | 0.678 | −0.206 | −23.3%        |

The grid drop from P2 to P7 costs **7pp on P(top10)** but **21pp on P(top5)** — three times the relative damage at the tighter boundary.

### Why this matters for strategy

**If the team only models `is_top10`:**

> "Hamilton at P7 has an 85% chance of scoring points. That's high. 1-stop M-H is safe — no reason to take the extra pit-stop risk."

**After adding `is_top5`:**

> "Hamilton at P7 has only a 68% chance of reaching the top 5 — 22pp below the P2 baseline. The grid drop hasn't just shifted him slightly down the order; it has effectively removed him from the podium battle. **A 2-stop M-M-H is justified not to protect points-scoring (still 85%) but to recover track position for a top-5 challenge.** The marginal risk of an extra stop is worth taking given how much P(top5) was lost from starting P7."

This recommendation is structurally invisible from `is_top10` alone. The 85% P(top10) would be read as "safe." Only the P(top5) drop reveals that the driver is now off-pace for the tier of finish the team is targeting.

---

## Secondary scenario: Leclerc, Monaco GP — tier sensitivity

**Context:** Charles Leclerc, Ferrari, Monaco GP. Two seasons with different constructor tiers.

| Scenario                   | Season | Grid | Tier     | P(top10) | P(top5) |
| -------------------------- | ------ | ---- | -------- | -------- | ------- |
| 2024 — P1, front-tier form | 2024   | 1    | front    | 0.931    | 0.904   |
| 2024 — P4, front-tier form | 2024   | 4    | front    | 0.898    | 0.842   |
| 2023 — P6, midfield form   | 2023   | 6    | midfield | 0.797    | 0.552   |

**Observation:** Moving from P1 to P4 (front-tier, 2024) costs −0.033 on P(top10) and −0.062 on P(top5) — moderate degradation at both boundaries. The midfield-tier scenario from 2023 costs an additional −0.245 on P(top5). `is_top10` still shows a high 0.797, suggesting a "likely points" outcome; `is_top5` shows only 0.552, revealing the scenario as a coin flip for a top-5 result.

**Strategy implication:** A team running Leclerc from P6 in midfield form should not plan a conservative 1-stop assuming a top-5 is within reach. The P(top5) = 0.552 number tells the strategist the car is not positioned for a podium — overfit strategy should be focused on securing P(top10) rather than gambling on an aggressive soft-start.

---

## Summary: what `is_top5` adds

| Decision question                                    | `is_top10` alone              | `is_top10` + `is_top5`                                          |
| ---------------------------------------------------- | ----------------------------- | --------------------------------------------------------------- |
| Should Hamilton attempt a 2-stop from P7?            | "No — points are safe at 85%" | "Yes — P(top5) dropped 21pp; 2-stop recovers the podium battle" |
| Is Leclerc competitive from P6 in midfield form?     | "Probably — 80% P(top10)"     | "Not for top 5 — 55%; optimize for points, not podium"          |
| How does grid-position sensitivity differ by target? | Appears moderate              | Is 3× more severe at the top-5 boundary                         |
