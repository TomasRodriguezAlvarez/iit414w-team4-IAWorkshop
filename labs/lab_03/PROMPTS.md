# PROMPTS.md

## Prompt 1 — Initial Notebook Structure

**Prompt:**
"Give me a structure for a machine learning notebook comparing models for a regression task using MAE and temporal validation."

**What AI suggested:**
A step-by-step notebook structure including data loading, feature engineering, baselines, model training, and comparison.

**What I did with it:**
I used the structure as a base, but adapted it to the specific dataset and lab requirements, especially ensuring temporal validation instead of random splits.

---

## Prompt 2 — Handling Column Errors

**Prompt:**
"I get a KeyError when selecting columns in pandas. How do I fix it?"

**What happened:**
The AI suggested checking column names using `df.columns.tolist()`.

**What I did:**
I discovered that the dataset used `driver` and `constructor` instead of `driverRef` and `constructorRef`. I updated the code accordingly.

---

## Prompt 3 — Model Selection

**Prompt:**
"What models should I compare for a regression task?"

**What AI suggested:**
Linear Regression, Random Forest, and XGBoost.

**What I did:**
I implemented all three, along with two baselines, to comply with the lab requirement of comparing multiple models.

---

## Prompt 4 — Interpretation of Results

**Prompt:**
"Help me interpret model results where complex models perform worse than simple ones."

**What AI suggested:**
Possible overfitting and insufficient features.

**What I did:**
I analyzed the train-test gap and confirmed that Random Forest and XGBoost were overfitting. I incorporated this reasoning into the notebook and comparison table.

---

## AI Limitations Observed

* AI initially assumed incorrect column names, which caused errors.
* Some explanations were generic and had to be adapted to match the actual results.
* AI did not account for dataset-specific issues such as the high proportion of zero-point outcomes.

---

## Conclusion

AI was useful for structuring the workflow and debugging issues, but all results and interpretations were validated and adapted based on actual model outputs.
