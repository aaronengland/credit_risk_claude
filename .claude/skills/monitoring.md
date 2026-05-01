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

3. `plot_psi_trend(df_data, str_date_col, str_pred_col, arr_baseline, str_freq='M', str_filename)` - PSI bar chart over time with threshold lines (orange=0.10, red=0.25).

4. `plot_csi(df_csi, str_filename)` - Horizontal bar chart of CSI per feature, color-coded (steelblue=ok, orange=moderate, red=significant).

5. `plot_target_drift_over_time(df_data, str_date_col, str_target, str_pred_col, str_freq='M', str_filename)` - Dual line plot: actual default rate vs mean predicted PD over time.

6. `plot_auc_over_time(df_data, str_date_col, str_target, str_pred_col, str_freq='M', str_filename)` - AUC line plot over time.

7. `plot_calibration_over_time(df_data, str_date_col, str_target, str_pred_col, str_freq='M', str_filename)` - Brier score line plot over time.

8. `plot_kde_over_time(df_data, str_date_col, str_pred_col, str_freq='Q', str_filename)` - Overlaid KDE plots per time period with median in legend.

### Analysis Sections

- Read train (baseline) and test (production proxy) data, load model
- Generate predictions, use training predictions as baseline
- Overall PSI with threshold interpretation
- PSI over time (monthly)
- CSI by feature with color-coded bar chart
- Target drift: actual vs predicted over time
- AUC over time (discrimination degradation)
- Brier score over time (calibration degradation)
- KDE of predictions over time (quarterly)
- Monitoring summary CSV
