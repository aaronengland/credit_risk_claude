---
name: champion-challenger
description: Populates the champion-challenger notebook in 08_champion_challenger/ with side-by-side model comparison for credit risk model replacement decisions.
user_invocable: true
---

# Champion Challenger

Populate the `08_champion_challenger/notebook.ipynb` with champion vs challenger model comparison for credit risk.

## Instructions

Edit the existing `08_champion_challenger/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. All plots saved to `output/`. Each function in its own code cell. Legend placement: `loc='lower right'`.

### Functions

One function per code cell, ordered to match execution order.

1. `compute_model_metrics(arr_y_true, arr_y_pred, str_model_name)` - Returns dict with str_model, flt_auc, flt_gini, flt_ks, flt_brier, flt_median_pred.

2. `plot_roc_comparison(arr_y_true, dict_models, str_filename)` - Overlaid ROC curves with AUC in legend. Champion=steelblue, Challenger=salmon.

3. `plot_kde_comparison(dict_models, str_filename)` - Overlaid KDE plots with median in legend.

4. `plot_calibration_comparison(arr_y_true, dict_models, int_n_bins=10, str_filename)` - Overlaid calibration plots with Brier in legend.

5. `plot_metric_comparison(df_metrics, str_metric, str_title, str_filename)` - Bar chart comparing a metric between models.

### Constants

- Champion model path and feature columns (current production model)
- Challenger model path and feature columns (new candidate model)
- Both evaluated on the same test set

### Analysis Sections

- Read test data, load both models
- Generate predictions from both on same test set
- Metrics comparison table (AUC, Gini, KS, Brier, median pred), saved to CSV
- AUC, Gini, KS bar chart comparisons
- ROC curve comparison
- Calibration comparison with Brier in legend
- KDE comparison with median in legend
- Automated recommendation: logic based on AUC and Brier improvement/degradation with clear recommendation text
