# Repository Guidelines

## Project Structure

This repository contains analysis notebooks and their source data. See `README.md` for notebook roles, run instructions, and recorded MLP and CVAE results.

- `breast_cancer.ipynb` — shared data preparation, five-year outcome definition, train/test split, and preprocessing.
- `seer_mlp_baseline.ipynb` - neural classifier baseline.
- `synthetic_data_feasibility.ipynb` - exploratory analysis for synthetic augmentation.
- `seer_cvae_synthetic_generation.ipynb` - active conditional generator, tuning and plausibility audits.
- `seer_cvae_previous_run.ipynb` - archived CVAE notebook.
- `Copia di SEER Breast Cancer Dataset .csv` — SEER breast cancer dataset used by the notebook.
- `.gitignore` — excludes local environments, notebook checkpoints, serialized model artifacts, and secrets.

There is currently no separate source-code package or automated test directory. Keep project analysis in the notebook unless a reusable module is clearly needed.

## Project Goal

Develop and assess a machine-learning classifier that predicts five-year overall survival from clinical information available at diagnosis. The notebook labels death before 60 months as class 1 and follow-up of at least 60 months as class 0; patients recorded alive with less than 60 months of follow-up are censored and excluded from this binary task. Evaluate predictive performance alongside relevant FUTURE-AI criteria such as fairness and explainability; do not treat a single score such as F1 as the whole project objective.

## Development and Run Commands

Install the notebook’s core dependencies and launch Jupyter:

```bash
python -m pip install -r requirements.txt
jupyter lab breast_cancer.ipynb
```

Run cells from a fresh kernel in order. Use the CVAE configuration cell to select a fresh run or restore compatible saved checkpoints. Preserve saved notebook outputs when editing layout or code unless the user requests clearing them.

The MLP configuration cell controls its stratified train/test split, training settings, and nested BCE-versus-Focal experiment. The default nested workflow tunes on inner folds and reports outer-fold metrics on the development pool; keep the test split untouched until model and analysis choices are frozen. Save run-specific outputs under `results/mlp/<run-id>/` and describe validation results as development estimates, not final test performance. Check split alignment before comparing MLP metrics with other notebook baselines.

## Notebook and Data Practices

Use clear section headings and short explanatory Markdown before substantial analysis. Keep Python indentation at four spaces; use descriptive `UPPER_SNAKE_CASE` for shared constants and `snake_case` for variables and functions. Prefer pandas and scikit-learn patterns already used in the notebook.

Prevent leakage: exclude `Survival Months`, `Status`, and `label` from model features; fit preprocessing and any resampling inside an sklearn pipeline and cross-validation folds. Use stratification for train/test and CV splits. Select models and hyperparameters on training data only; reserve the test set for the final evaluation. Report metrics suited to the imbalanced outcome, not accuracy alone. Document assumptions and justify feature encoding choices, especially for ordinal clinical stages.

For CVAE audits, report training and validation references separately, including conditional distributions and per-record rule flags. Treat rare combinations as review signals. Keep checkpoint binaries and generated patient-level audit files out of Git.

## Validation

No automated tests or formatter are configured. Before proposing notebook changes, verify the `.ipynb` remains valid JSON and run affected cells from a clean kernel when the dataset environment is available. Review tables, plots, and metrics for consistency with the stated five-year label and censoring rules.

## Commits and Pull Requests

Git history contains short, focused messages in both Italian and English. Use a concise imperative subject, for example `Add five-year outcome EDA`. Pull requests should summarize the analytical change, note any dataset or label assumptions, and include screenshots or a brief description for material output changes. Do not commit credentials, local environment files, or generated model binaries.
