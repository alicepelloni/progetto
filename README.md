# SEER breast cancer survival

This project studies five-year overall survival prediction using SEER breast cancer data. It compares predictive models and investigates whether synthetic examples could support minority-class augmentation while preserving clinical relationships and subgroup representation.

The binary endpoint assigns class 1 to recorded deaths before 60 months and class 0 to follow-up of at least 60 months. Patients recorded alive with shorter follow-up are excluded because their five-year outcome is unknown. Outcome fields are excluded from the predictors.

| Notebook | Purpose |
|---|---|
| [breast_cancer.ipynb](breast_cancer.ipynb) | Shared data preparation and endpoint definition |
| [seer_mlp_baseline.ipynb](seer_mlp_baseline.ipynb) | Neural classifier baseline |
| [synthetic_data_feasibility.ipynb](synthetic_data_feasibility.ipynb) | Exploratory analysis for synthetic augmentation |
| [seer_cvae_synthetic_generation.ipynb](seer_cvae_synthetic_generation.ipynb) | Conditional generation, tuning and plausibility audits |
| [seer_cvae_previous_run.ipynb](seer_cvae_previous_run.ipynb) | Archived CVAE notebook with earlier outputs |

Install dependencies from the repository root in your active Python environment. The comments in [requirements.txt](requirements.txt) describe the Windows CPU and Colab PyTorch setups.

```bash
python -m pip install -r requirements.txt
jupyter lab
```

## MLP classifier

[`seer_mlp_baseline.ipynb`](seer_mlp_baseline.ipynb) provides a compact PyTorch MLP baseline for predicting death before 60 months. Its configuration cell controls the stratified 80% development / 20% held-out test split, model and training settings, and experiment mode. The default development experiment performs nested stratified cross-validation to compare binary cross-entropy (BCE) with focal loss and tune learning rate, dropout, weight decay and focal-loss gamma. Preprocessing is fitted within each fold; the inner folds select configurations by average precision (AP), and outer folds estimate performance. Training uses an adaptive learning-rate scheduler and early stopping. The final test evaluation is disabled during development and should only be run after choices are frozen.

### Recorded nested cross-validation results

Run `20261111T121452Z_ca6e014c` completed the five-fold outer / three-fold inner comparison on the 80% development pool: **2,616 patients** with **359 positive outcomes**. The held-out 20% test set was not evaluated. Values below are the mean and standard deviation across the five outer folds; the paired difference is Focal minus BCE.

| Metric | BCE | Focal | Paired difference (Focal − BCE) |
|---|---:|---:|---:|
| Average precision (AP) | 0.3716 ± 0.0504 | 0.3746 ± 0.0506 | +0.0030 |
| ROC AUC | 0.7424 ± 0.0195 | 0.7423 ± 0.0209 | −0.0001 |
| Brier score (lower is better) | **0.1046 ± 0.0039** | 0.1170 ± 0.0026 | +0.0124 |

Focal loss had slightly higher AP in four of five outer folds, but the average gain was small; ROC AUC was effectively unchanged. Its Brier score was worse in all five folds, indicating less accurate probability estimates under this metric. These results do not establish a meaningful overall advantage for focal loss. BCE is the more conservative choice when probability quality matters, while focal loss may merit further study if ranking the positive cases by AP is the main objective. Fold standard deviations describe observed fold-to-fold variability; they are not confidence intervals. The out-of-fold pooled scores (AP 0.3468 vs 0.3502, ROC AUC 0.7405 vs 0.7406, Brier 0.1046 vs 0.1170) give the same qualitative comparison, but aggregate AP is not the arithmetic mean of fold APs.

The saved split emitted a warning that it was a custom stratified split rather than an exactly aligned baseline split. Therefore these results support the within-notebook BCE-versus-Focal comparison, but direct score comparisons with other baseline notebooks require aligning the eligible cohort and row-level split. Run artifacts, including fold metrics, out-of-fold predictions, partitions, histories and checkpoints, are saved under `results/mlp/<run-id>/`; the recorded run is available locally at `results/mlp/20261111T121452Z_ca6e014c/`.

## Conditional variational autoencoder

The CVAE learns mixed clinical and demographic features conditional on the binary outcome. It generates class-1 examples and samples T Stage, N Stage and overall stage jointly from combinations supported by the training data. The default stratified split is approximately **64% training, 16% validation and 20% test**. Preprocessing and generation support are fitted on training patients. Validation guides model selection, and test patients are held out.

Edit **Configuration and hyperparameters** before running the notebook from a fresh kernel. A fresh tuning run uses `RESUME_FROM = None` and `AUTO_TUNE = True`. For the recorded post-training analysis, set `RESUME_FROM` to `Path("cvae_output/20261111_121445_regularized")`, `RUN_LR_COMPARISON = False`, `RUN_AGE_OFFSET = True` and `AGE_OFFSET_SOURCE` to `Path("cvae_output/20261111_130551_evaluation")`. The last path supplies the saved adaptive LR checkpoint. These paths require the corresponding local run files.

### Results

The automatic search selected `run_14` with a synthetic validation score of **0.1450**, compared with **0.1470** for the reference configuration. The saved model's overall representation screen still failed, with a maximum gap of **12.8 percentage points**. In a separate fine-tuning comparison from the same checkpoint, adaptive LR scored **0.1276** versus **0.1343** for fixed LR, averaged over three generation seeds with one training seed.

The latest evaluation (`20261111_133405_evaluation`) calibrated an offset of **+0.50** on logits for ages 61–70. Calibration targeted the class-1 training share after filtering. The paired evaluation used three separate generation seeds and 600 accepted records per seed and arm.

| Metric | Zero offset | Calibrated offset |
|---|---:|---:|
| Accepted share aged 61–70 | 31.50% | 42.89% |
| Mean absolute gap to training (40.42%) | 8.92 pp | 2.47 pp |
| Mean absolute gap to validation (29.17%) | 2.33 pp | 13.72 pp |
| Age × race joint TVD vs training | 0.144 | 0.101 |
| Age × T Stage joint TVD vs training | 0.146 | 0.114 |
| Age × overall stage joint TVD vs training | 0.197 | 0.153 |
| Mean pairwise association error vs training | 0.0789 | 0.0794 |

The offset improved agreement with the intended training reference for age and the three age-related joint distributions shown above. The validation reference gave the opposite result for the 61–70 share. Overall association error remained close, and the joint distribution of N Stage and binned positive nodes retained a TVD of about **0.33** versus training in both arms.

All **3,600 accepted diagnostic records** passed the implemented hard rules. In each arm, **309/1,800** records had at least one audited pair absent from real class-1 training data, and **199/1,800** received a tumor-size/T Stage flag under the notebook's conditional coding assumptions. These counts are review signals. Pairwise checks and empirical support provide partial evidence of plausibility. The calibrated samples have no demonstrated downstream benefit on real patients.

### Output files

Runs write to local `cvae_output/<run>/`, which is ignored by Git. The general `summary.json` describes the selected search model. The following files describe the separate age-offset experiment on the adaptive checkpoint.

| File within the run | Contents |
|---|---|
| `metrics/age_offset_calibration.csv` | Candidate offsets and calibration scores |
| `metrics/age_offset_comparison.csv` | Paired metrics for zero and calibrated offsets |
| `audit/age_offset_record_plausibility.csv` | Per-record rules, flags and training support counts |
| `metrics/age_offset_conditional_dependencies.csv` | Joint and conditional discrepancies with subgroup counts |
| `metrics/age_offset_associations.csv` | Pairwise associations against each real reference |
