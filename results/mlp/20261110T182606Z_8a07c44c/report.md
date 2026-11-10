# SEER MLP benchmark — computed run report

Run: 20261110T182606Z_8a07c44c

Dataset SHA256: 1d64363e87b306bdb241cb7e105271514f74564842c02e01dbce2f4884266d09

Eligible records: 3270; excluded early-censored: 753. Split: reproduced breast_cancer.ipynb source recipe; verify baseline imports.

## Predictive performance

Raw probabilities; validation-balanced-accuracy thresholds; mean ± sample SD across fixed-split seeds.

MLP-A: roc_auc=0.703 ± 0.003; average_precision=0.325 ± 0.010; sensitivity=0.693 ± 0.036; specificity=0.574 ± 0.064; precision=0.207 ± 0.016; f1=0.318 ± 0.016; balanced_accuracy=0.634 ± 0.014; brier=0.108 ± 0.001; ece=0.025 ± 0.005

MLP-B: roc_auc=0.705 ± 0.005; average_precision=0.336 ± 0.015; sensitivity=0.589 ± 0.126; specificity=0.687 ± 0.133; precision=0.239 ± 0.035; f1=0.334 ± 0.020; balanced_accuracy=0.638 ± 0.012; brier=0.223 ± 0.011; ece=0.316 ± 0.019

Weighted-minus-standard sensitivity difference at validation-selected thresholds: -0.104. This is descriptive, not a significance test.

## Calibration

Brier and ECE by prespecified probability variant (lower is better):

MLP-A/platt: Brier=0.108; ECE=0.027.

MLP-A/raw: Brier=0.108; ECE=0.025.

MLP-B/platt: Brier=0.107; ECE=0.032.

MLP-B/raw: Brier=0.223; ECE=0.316.

Platt parameters use disjoint training-derived validation records; the small calibration sample limits reliability. Test plots must not be used to choose a calibration variant.

## Fairness

Raw, validation-threshold disparities, averaged across seeds; uncertainty and sample sizes are in fairness/subgroup_metrics_intervals.csv.

MLP-A/Race: mean TPR gap=0.422; FPR gap=0.371.

MLP-A/age_group: mean TPR gap=0.205; FPR gap=0.211.

MLP-A/intersection: mean TPR gap=1.000; FPR gap=0.580.

MLP-B/Race: mean TPR gap=0.467; FPR gap=0.346.

MLP-B/age_group: mean TPR gap=0.201; FPR gap=0.249.

MLP-B/intersection: mean TPR gap=1.000; FPR gap=0.612.

Underpowered subgroup rows: 168/360 (repeated across seed/policy/calibration). Pointwise intervals and sparse denominators preclude a blanket fairness conclusion.

Race: weighted-minus-standard mean equalized-odds maximum-gap difference=+0.029; compare with sensitivity difference -0.104 above.

age_group: weighted-minus-standard mean equalized-odds maximum-gap difference=+0.034; compare with sensitivity difference -0.104 above.

intersection: weighted-minus-standard mean equalized-odds maximum-gap difference=+0.000; compare with sensitivity difference -0.104 above.

## Explainability

MLP-A leading original features: Reginol Node Positive, Age, Tumor Size, Progesterone Status, N Stage.

MLP-B leading original features: Age, Reginol Node Positive, Progesterone Status, Tumor Size, N Stage.

Pairwise attribution rank Spearman range: 0.852 to 0.984. Includes seed/weight comparisons; see stability.csv for pairs.

Attributions explain raw logits, not causation. Correlated staging/nodal features and interpolation limit interpretation; inspect saved additivity residuals.

## Robustness

Across bounded ±1 mm stage-consistent size scenarios: largest mean |Δp|=0.0024; largest changed-record decision-flip fraction=0.009.

The perturbations cover a narrow measurement scenario. Seed SD concerns optimization at one split and does not quantify cohort or external-site uncertainty.

## Baseline comparison

No baseline predictions imported; comparative performance/fairness is unavailable.

## Suitability for synthetic-data experiments

The code exposes a fixed-preprocessor original-feature augmentation interface. Readiness remains provisional: inspect AP/sensitivity/Brier seed SD, sparse-group intervals and attribution stability before prespecifying an augmentation experiment. There is no universal stability cutoff that this run can certify.

Synthetic generation is absent. Validation/test records remain real and untouched by training. Generator provenance and clinical validity require independent verification.

## Limitations

Single-source, single-split complete-case classification; possible informative censoring; unverified timing of predictors; no verified patient identity beyond exact-record deduplication; small calibration and subgroup samples; no external validation. No clinical deployment claim.