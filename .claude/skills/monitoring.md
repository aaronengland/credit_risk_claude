---
name: monitoring
description: Populates the monitoring notebook in 07_monitoring/ with post-deployment model monitoring including PSI, CSI, target drift, AUC trend, calibration degradation, and KDE over time.
user_invocable: true
---

# Monitoring

Populate the `07_monitoring/notebook.ipynb` with post-deployment model monitoring for credit risk.

## Instructions

Edit the existing `07_monitoring/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. All plots saved to `output/`. Each function in its own code cell. Legend placement: `loc='lower right'`.

### Functions

One function per code cell, ordered to match execution order.

1. `calculate_psi(arr_expected, arr_actual, int_n_bins=10)` - Population Stability Index. Thresholds: <0.10 no shift, 0.10-0.25 moderate, >0.25 significant.

2. `calculate_csi(df_expected, df_actual, list_feature_cols, int_n_bins=10)` - Characteristic Stability Index per feature using percentile-based bins from expected data.

3. `plot_psi_trend(df_data, str_date_col, str_pred_col, arr_baseline, str_train_end, str_valid_end, str_freq='M', str_filename)` - PSI bar chart over time with threshold lines (orange=0.10, red=0.25). Vertical lines mark train end (black) and validation end (gray).

4. `plot_csi(df_csi, str_filename)` - Horizontal bar chart of CSI per feature, color-coded (steelblue=ok, orange=moderate, red=significant).

5. `plot_target_drift_over_time(df_data, str_date_col, str_target, str_pred_col, str_train_end, str_valid_end, str_freq='M', str_filename)` - Dual line plot: actual default rate vs mean predicted PD over time. Vertical split boundary lines.

6. `plot_auc_over_time(df_data, str_date_col, str_target, str_pred_col, str_train_end, str_valid_end, str_freq='M', str_filename)` - AUC line plot over time. Vertical split boundary lines.

7. `plot_calibration_over_time(df_data, str_date_col, str_target, str_pred_col, str_train_end, str_valid_end, str_freq='M', str_filename)` - Brier score line plot over time. Vertical split boundary lines.

8. `plot_kde_over_time(df_data, str_date_col, str_pred_col, str_freq='Q', str_filename)` - Overlaid KDE plots per time period with median in legend.

### Data Setup

- Read clean data (train, validation, test) from `03_preprocessing/` for model predictions
- Read raw splits from `02_data_split/` for the date column (not available in clean data after ColumnTransformer)
- Generate predictions for all three splits
- Build monitoring DataFrames combining date, target, and predictions
- Combine into `df_all` for full timeline visualization
- Compute `str_train_end` and `str_valid_end` (YYYY-MM format) for vertical boundary lines
- Training predictions serve as the baseline population

### Analysis Sections

- Overall PSI with threshold interpretation
- PSI over time (monthly) on `df_all` with split boundary lines. Training months serve as a control (should be near-zero PSI).
- CSI by feature (train vs test clean data) with color-coded bar chart
- Target drift on `df_all`: actual vs predicted over time with split boundary lines
- AUC over time on `df_all` with split boundary lines
- Brier score over time on `df_all` with split boundary lines
- KDE of predictions over time (quarterly) on `df_all`
- Monitoring summary CSV
