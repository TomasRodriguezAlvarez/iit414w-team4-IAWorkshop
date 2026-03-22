# AI Usage Documentation

## Entry 1 – Feature Engineering Ideas

**Context:**  
At the start of Lab 2 I needed candidate features that could improve Top-10 prediction while respecting pre race availability and temporal validation constraints.

**Prompt(s):**  
Give me new features that could be used in this case to predict F1 Top-10 finishes using only pre race information.

**Output:**  
The AI suggested features such as previous race finish (lag), rolling average finish over the last 3 races, driver-circuit interaction, and constructor tier based on prior-season performance.

**Validation:**  
I checked each feature against the leakage guard logic and confirmed that lag/rolling features used shift(1), and that constructor tier used previous season information only.

**Adaptations:**  
I implemented three features in the notebook: prev_finish_position, avg_finish_last_3, and constructor_tier. I did not implement the driver-circuit interaction in the final model.

**Final Decision:**  
Used. The suggestions directly informed the selected features for the Lab 2 model.


## Entry 2 – Prior-Period Baseline Implementation

**Context:**  
I needed to replace placeholder zeros in the comparison table with a real prior period baseline result.

**Prompt(s):**  
Implement a prior period baseline using the average finish position of the last 3 races with shift(1), and include it in the markdown comparison export.

**Output:**  
The AI provided a baseline function using grouped rolling mean with shift(1), generated validation metrics, and updated the export table row to use real values.

**Validation:**  
I executed the notebook cell and confirmed the comparison table printed non zero metrics for the prior period baseline.

**Adaptations:**  
I kept the baseline label as optional in the table and maintained the same metric set used by other baselines.

**Final Decision:**  
Used. The prior period row now reports real numbers.

