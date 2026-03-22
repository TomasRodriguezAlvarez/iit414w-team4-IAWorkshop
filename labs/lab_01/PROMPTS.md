# AI Usage Documentation

## Entry 1 – EDA Interpretation Support

**Context:**  
During exploratory data analysis I needed help interpreting correlations, temporal splits, and identifying potential traps in the dataset.

**Prompt(s):**  
How should I interpret correlations between finish position and a Top-10 indicator?  
How can I detect spurious correlations in exploratory analysis?  
What is a correct temporal train/validation/test split for race data?

**Output:**  
The AI explained that strong correlations may appear when a variable directly defines the target (e.g., finish position and Top-10).  
It also suggested using chronological splits by season to avoid temporal leakage.

**Validation:**  
I checked the dataset structure and confirmed that `is_top10` is derived from finish position.  
This confirmed that the correlation is structural rather than predictive.

**Adaptations:**  
I incorporated this insight into the trap-awareness section and avoided using finish position as a predictive feature.

**Final Decision:**  
Partially used. The conceptual explanation helped guide the analysis, but all decisions and implementations were verified in the notebook.

---

## Entry 2 – Baseline Evaluation Metrics

**Context:**  
While building the heuristic baseline, I wanted to add evaluation metrics aligned with Burkov Chapter 2.

**Prompt(s):**  
Which metrics should be used for a binary classification baseline besides accuracy?  
Can ROC-AUC be applied to a rule-based classifier?

**Output:**  
The AI suggested using Precision, Recall, F1-score, and ROC-AUC to complement accuracy.

**Validation:**  
I reviewed Burkov Chapter 2 and confirmed these metrics are standard for binary classification evaluation.

**Adaptations:**  
I implemented these metrics in `baseline.ipynb` using the validation split (season 2023).

**Final Decision:**  
Used. These metrics were added to evaluate the heuristic baseline more comprehensively.

---

## Entry 3 – Repository Setup Assistance

**Context:**  
I needed to create a clean `.gitignore` and repository structure for a Python/Jupyter project.

**Prompt(s):**  
Create a minimal `.gitignore` suitable for a Python project using Jupyter notebooks.

**Output:**  
The AI suggested ignoring Python cache files, virtual environments, notebook checkpoints, and OS/editor artifacts.

**Validation:**  
I checked common Python `.gitignore` templates and confirmed the suggestions were standard.

**Adaptations:**  
I added entries for `.ipynb_checkpoints`, virtual environments, and cache directories.

**Final Decision:**  
Used. The `.gitignore` was implemented in the repository.