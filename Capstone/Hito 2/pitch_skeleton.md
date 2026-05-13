# pitch_skeleton.md — Demo Day Prep (Block D, Wed May 13)

*This file is a workshop artifact written during the Hito 2 in-class sprint.
It seeds the Demo Day pitch for Monday May 18.*

---

## Slide 1 — Team verdict sentence

**One sentence that captures the team's recommendation:**

> "Our pre-race model shows that a grid drop from P2 to P7 costs only 7 percentage points on Top-10 probability but 21 percentage points on Top-5 probability — making the case for an aggressive 2-stop strategy invisible to any tool that models points-scoring alone."

---

## Slide 5 — What the model cannot do (honest limitations)

**Three things the model will not tell you reliably:**

1. **No-stop races are out-of-distribution.** When a driver runs zero pit stops, it almost always means a mechanical failure or accident happened during the race — not a deliberate strategy. The model assigns non-trivial Top-10 probability based on grid position and will be confidently wrong. Any deployment must flag no-stop scenarios as outside the model's reliable range.

2. **The model estimates pre-race baseline risk, not causal strategy effects.** The probabilities answer "given this driver, this car, and this starting position, what is the historical Top-10 rate?" — not "if we switch from 1-stop to 2-stop, how much will the finish position improve?" Strategy choice is endogenous: fast cars run aggressive strategies and finish well partly because of pace, not strategy. The model cannot separate these.

3. **Semi-street circuits are noisier.** At circuits like Baku, Miami, and Singapore, safety-car probability is high and grid position is systematically less predictive. The model will overstate confidence at these venues until a circuit-specific safety-car-rate feature is added.

---

## What we'd need before recommending deployment

- OOD flagging for no-stop scenarios (Risk 1 from mitigations.md)
- Causal framing disclaimer on every model output (Risk 6)
- Circuit-specific safety-car-rate feature for semi-street venues (Risk 2)

Until these three conditions are met: **we do not recommend deploying this tool as a live strategy advisor.** We recommend it as a pre-race scenario comparison tool used alongside experienced human judgment.
