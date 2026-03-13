# Data Quality Log - Lab 1

## Issue 1: grid_position - Missing values
- **What:** Some rows have missing `grid_position` values (for example, pit-lane starts or incomplete session records).
- **Classification:** MAR (Missing At Random) / Data availability issue
- **Impact:** The heuristic baseline (`grid_position <= 10`) cannot be applied reliably if grid position is null.
- **Decision:** Keep rows for global analysis, but in baseline prediction treat missing grid as non-Top-10 prediction (conservative default).
- **Justification:** Preserves sample size and avoids optimistic bias from dropping difficult/uncertain rows.

## Issue 2: finish_position - Missing values (DNF/DSQ/non-classified)
- **What:** `finish_position` can be missing for drivers who did not finish or were disqualified/non-classified.
- **Classification:** MNAR (Missing Not At Random)
- **Impact:** Directly affects target creation (`is_top10`) and can bias outcomes if these cases are ignored without documenting the rule.
- **Decision:** Exclude rows with missing `finish_position` before building `is_top10`.
- **Justification:** The target requires a valid final race outcome; imputing finish position would introduce artificial labels.

## Issue 3: grid_position - Invalid domain values
- **What:** Potential values outside expected Formula 1 race grid range (1-20), including zeros or out-of-range numbers.
- **Classification:** Outlier / Domain validity issue
- **Impact:** Distorts plots, correlations, and threshold rules based on grid cutoffs.
- **Decision:** Flag invalid records for audit; keep them out of decision logic when identified.
- **Justification:** Domain rules are strict in F1; invalid values should not silently influence baseline performance.

## Issue 4: team_name - Categorical consistency across seasons
- **What:** Team identifiers can vary by season branding/naming conventions.
- **Classification:** Type/semantic consistency issue
- **Impact:** Can fragment group statistics and historical team-level rates, reducing comparability over time.
- **Decision:** Keep raw values for transparency in Lab 1; document that canonical team mapping may be required for later modeling.
- **Justification:** Prevents premature manual recoding while preserving reproducibility and auditability of source data.

## Issue 5: season-round completeness - Missing race records due API load failures
- **What:** Some race sessions may fail to load (network/source issues), creating partial season coverage.
- **Classification:** Data completeness / collection pipeline issue
- **Impact:** Temporal comparisons and split evaluation may be biased if missing rounds are not acknowledged.
- **Decision:** Keep successfully loaded races and log failures during extraction.
- **Justification:** Maintains deterministic pipeline behavior while making coverage gaps explicit.

## Issue 6: Post-race fields in source data - Leakage risk
- **What:** Variables such as final points, final status, and final position are only known after the race.
- **Classification:** Temporal leakage risk (feature availability issue)
- **Impact:** Can inflate apparent predictive quality if used for pre-race prediction.
- **Decision:** Exclude post-race variables from baseline/model input features; use only pre-race fields.
- **Justification:** Ensures honest evaluation aligned with the real decision moment (before race start).
