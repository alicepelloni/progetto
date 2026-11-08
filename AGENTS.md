# Repository Guidelines

## Project Structure

This repository contains two analysis notebooks and their source data:

- `breast_cancer.ipynb` — shared data preparation, five-year outcome definition, train/test split, and preprocessing.
- `baseline_models.ipynb` — standalone setup and baseline pipelines for logistic regression and a decision tree.
- `Copia di SEER Breast Cancer Dataset .csv` — SEER breast cancer dataset used by the notebook.
- `.gitignore` — excludes local environments, notebook checkpoints, serialized model artifacts, and secrets.

There is currently no separate source-code package or automated test directory. Keep project analysis in the notebook unless a reusable module is clearly needed.

## Project Goal

Develop and assess a machine-learning classifier that predicts five-year overall survival from clinical information available at diagnosis. The notebook labels death before 60 months as class 1 and follow-up of at least 60 months as class 0; patients recorded alive with less than 60 months of follow-up are censored and excluded from this binary task. Evaluate predictive performance alongside relevant FUTURE-AI criteria such as fairness and explainability; do not treat a single score such as F1 as the whole project objective.

## Development and Run Commands

Install the notebook’s core dependencies and launch Jupyter:

```bash
python -m pip install jupyter pandas numpy scikit-learn
jupyter lab breast_cancer.ipynb
```

In `baseline_models.ipynb`, dataset loading is automatic: it searches the project folder (`C:\Progetti\neuroengineering_course\progetto`) and the Jupyter working directory first, then mounts Google Drive in Colab if needed. Run cells from a fresh kernel in order.

## Notebook and Data Practices

Use clear section headings and short explanatory Markdown before substantial analysis. Keep Python indentation at four spaces; use descriptive `UPPER_SNAKE_CASE` for shared constants and `snake_case` for variables and functions. Prefer pandas and scikit-learn patterns already used in the notebook.

Prevent leakage: exclude `Survival Months`, `Status`, and `label` from model features; fit preprocessing and any resampling inside an sklearn pipeline and cross-validation folds. Use stratification for train/test and CV splits. Select models and hyperparameters on training data only; reserve the test set for the final evaluation. Report metrics suited to the imbalanced outcome, not accuracy alone. Document assumptions and justify feature encoding choices, especially for ordinal clinical stages.

## Validation

No automated tests or formatter are configured. Before proposing notebook changes, verify the `.ipynb` remains valid JSON and run affected cells from a clean kernel when the dataset environment is available. Review tables, plots, and metrics for consistency with the stated five-year label and censoring rules.

## Commits and Pull Requests

Git history contains short, focused messages in both Italian and English. Use a concise imperative subject, for example `Add five-year outcome EDA`. Pull requests should summarize the analytical change, note any dataset or label assumptions, and include screenshots or a brief description for material output changes. Do not commit credentials, local environment files, or generated model binaries.
