# F1 Top-10 Analysis

This repository contains the exploratory data analysis (EDA) and baseline modeling for predicting **Top-10 race finishes in Formula 1**.

The project is designed to be fully **reproducible**, following the reproducibility requirements of the assignment.

---

# Repository Structure

- `eda.ipynb` – Decision-oriented exploratory data analysis.
- `baseline.ipynb` – Domain heuristic baseline and optional modeling experiments.
- `DATA_QUALITY_LOG.md` – Structured log of identified data quality issues and decisions.
- `PROMPTS.md` – Documentation of AI-assisted prompts used during development.
- `requirements.txt` – Exact environment specification for reproducibility.
- `.gitignore` – Excludes cache files and notebook checkpoints.

---

# How to Reproduce the Results

## 1. Clone the repository

```bash
git clone https://github.com/TomasRodriguezAlvarez/iit414w-lab01-team4
 cd iit414w-lab01-team4
```

## 2. Install dependencies
pip install -r requirements.txt

## 3. Run the notebooks
Launch Jupyter:
jupyter notebook

Run the notebooks in the following order:
eda.ipynb
baseline.ipynb

All notebooks are designed to run top-to-bottom on a fresh environment.

# Data Source
Race data is retrieved dynamically using the FastF1 API, so no CSV files are stored in the repository.
The first execution may take slightly longer because FastF1 downloads and caches race data locally.

# Reproducibility Notes
-   RANDOM_SEED = 414 is used throughout the notebooks.
-   The test set is never used during EDA or baseline design.
-   A temporal data split is applied:
    - Train: 2022 season
    - Validation: 2023 season
    - Test: 2024 season
This preserves chronological order and prevents temporal data leakage.
