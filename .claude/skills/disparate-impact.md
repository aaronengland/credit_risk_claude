---
name: disparate-impact
description: Populates the disparate impact notebook in 06_disparate_impact/ with fair lending analysis comparing model outcomes across protected age groups.
user_invocable: true
---

# Disparate Impact

Populate the `06_disparate_impact/notebook.ipynb` with fair lending disparate impact analysis on the test set.

## Instructions

Edit the existing `06_disparate_impact/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. All plots saved to `output/`. Each function in its own code cell. Legend placement: `loc='lower right'`.

### Analysis Overview

Compare model performance and prediction distributions between age < 60 and age >= 60 on the test set only. Age is a protected class under ECOA.

### Functions

1. `compute_metrics_by_group(df_data, str_group_col, str_pred_col, str_target)` - Returns DataFrame with per-group: int_n, flt_target_mean, flt_pred_mean, flt_pred_median, flt_auc.

2. `plot_kde_by_group(df_data, str_group_col, str_pred_col, str_filename)` - KDE of predictions by group with median in legend. Colors: steelblue, salmon.

3. `plot_metric_comparison(df_metrics, str_metric, str_title, str_filename)` - Bar chart comparing a metric across groups with values annotated. Y-axis padded.

### Analysis Sections

- Read test data and model from 04_model/output/
- Create age group column (< 60 vs >= 60)
- Metrics summary table by group, saved to CSV
- Bar charts: mean predicted probability, actual default rate, AUC by group
- KDE of predictions by age group
- KS test between age groups (statistic and p-value)
