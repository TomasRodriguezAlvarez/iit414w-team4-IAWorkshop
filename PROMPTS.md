## Objective
Document how AI assistance and Burkov Chapter 2 concepts were used to add Precision, Recall, F1-score, and ROC-AUC to the heuristic baseline.

## What I Asked the AI
1. For a binary heuristic baseline, which metrics from Burkov Ch. 2 should be reported in addition to accuracy
3. Can ROC-AUC be used with a rule based model, and what score should be used if the model is threshold-based
4. How should I interpret these metrics for decision-making if classes are close to balanced

## What I Understood
- Accuracy alone can hide important errors.
- ROC-AUC measures ranking quality across thresholds and is useful as a threshold-independent check.
- A heuristic baseline can still be evaluated with these metrics, not only ML models.

## What I Did Not Understand at First
- Whether ROC-AUC is valid for a deterministic rule.
- Which score to pass into ROC-AUC when predictions are binary labels.

## Clarification Reached
- ROC-AUC is most informative with a continuous score.
- For this F1 task, using a pre race score derived from grid position (better grid = higher score) is acceptable and consistent with the no-leakage constraint.

## Implementation in baseline.ipynb
- Validation split: season 2023.
- Metrics computed: Precision, Recall, F1-score, ROC-AUC.
- Extra diagnostic: confusion matrix (TN, FP, FN, TP).
- Leakage-safe setup preserved by using pre-race information only.

## Objective (Gitignore)
Document the request to add a project-level gitignore for this repository.

## What I Asked the AI (Gitignore)
1. Build a clean gitignore for a Python and Jupyter lab project

## Implementation (.gitignore)
- Added ignores for Python cache 
- Added ignores for virtual environments
- Added ignores for editor/system artifacts and local env files
