# MLP network development (no test-based selection)

Common selected candidate: compact_regularized.

Selection: mean stopping-validation AP over both weighting configurations and all fixed seeds. Scores are development estimates, not independent validation.

Candidate comparison: network_candidate_summary.csv; per-weighting/per-seed results: network_development.csv.

compact_regularized: mean AP=0.3034, SD over seed/weighting combinations=0.0121, parameters=705.

original: mean AP=0.2981, SD over seed/weighting combinations=0.0126, parameters=1665.

regularized: mean AP=0.2938, SD over seed/weighting combinations=0.0098, parameters=1665.

Per-weighting plots use seed-only SD. Threshold/calibration subsets remain reserved until final evaluation is enabled. Inspect validation loss, AP, Brier and initialization variability together; selection by AP does not certify fairness or calibration.