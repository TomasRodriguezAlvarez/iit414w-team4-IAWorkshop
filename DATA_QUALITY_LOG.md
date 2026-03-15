# Data Quality Log - Lab 1

Structured record of data quality issues identified during exploratory analysis and the decisions taken to handle them.

---

## Issue 1: grid_position - Potential missing values

- **What:** Grid position may occasionally be missing due to incomplete session data or special race conditions (e.g., pit-lane starts or session recording gaps).
- **Classification:** MAR (Missing At Random)
- **Impact:** The heuristic baseline rule (`grid_position <= 10`) cannot be applied when grid position is missing.
- **Decision:** Keep rows for exploratory analysis. For baseline prediction, treat missing grid positions as non–Top-10 predictions.
- **Justification:** Preserves dataset size while avoiding optimistic bias from excluding uncertain cases.

---

## Issue 2: finish_position - Missing values for non-finishers

- **What:** Drivers who do not finish a race (DNF) or are disqualified may not have a valid `finish_position`.
- **Classification:** MNAR (Missing Not At Random)
- **Impact:** The target variable (`is_top10`) cannot be constructed without a valid finishing position.
- **Decision:** Exclude rows with missing `finish_position` before generating the `is_top10` target.
- **Justification:** The outcome variable depends directly on final race position, and imputing this value would introduce artificial labels.

---

## Issue 3: grid_position - Domain validity / outlier check

- **What:** Grid positions should normally fall within the Formula 1 starting grid range (typically 1–20).
- **Classification:** Outlier / Domain validity issue
- **Impact:** Values outside this range could distort plots, correlations, and heuristic rules.
- **Decision:** Inspect and flag any records outside the expected range before using them in decision rules.
- **Justification:** Formula 1 grid size is constrained by regulations, so out-of-range values likely indicate data issues.

---

## Issue 4: team_name - Categorical consistency across seasons

- **What:** Team names may change across seasons due to rebranding (e.g., AlphaTauri → RB).
- **Classification:** Type / semantic consistency issue
- **Impact:** Historical aggregation by team could fragment statistics if names change between seasons.
- **Decision:** Keep raw team identifiers for Lab 1 and document the issue for potential canonical mapping in future analysis.
- **Justification:** Preserving raw values maintains traceability to the original data source.

---

## Issue 5: season / round - Data coverage completeness

- **What:** Race session data is retrieved through the FastF1 API, which may occasionally fail to load a session due to network or source availability issues.
- **Classification:** Data completeness / collection pipeline issue
- **Impact:** Missing races would affect temporal comparisons and train/validation/test splits.
- **Decision:** Keep successfully retrieved sessions and log any missing races during data collection.
- **Justification:** Ensures transparency about dataset coverage without altering the extraction pipeline.

---

## Issue 6: Post-race variables - Temporal leakage risk

- **What:** Variables such as `finish_position` and derived outcomes (`is_top10`) are only known after the race.
- **Classification:** Temporal availability / leakage risk
- **Impact:** Using post-race information as predictive features would artificially inflate model performance.
- **Decision:** Restrict baseline and predictive features to pre-race variables (e.g., `grid_position`, `season`, `round`, `team_name`).
- **Justification:** Preserves a realistic prediction scenario where decisions are made before the race begins.

---

## Issue 7: driver_name / driver_code - Potential identifier missingness

- **What:** Driver identifiers could theoretically be missing due to data logging or entry errors.
- **Classification:** MCAR (Missing Completely At Random)
- **Impact:** Missing identifiers would prevent grouping or driver-level analysis.
- **Decision:** Assume identifiers are complete; if missing values appear, drop affected rows from driver-level analysis.
- **Justification:** Identifier loss would not depend on race outcome or performance and is therefore considered random.