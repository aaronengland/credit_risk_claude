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

5. `plot_metric_comparison(df_metrics, str_metric, str_title, str_filename)` - Bar chart comparing a metric between models with values annotated. Y-axis padded.

### Constants

- Champion model path: `../04_model/output/xgboost_model.joblib`
- Champion feature cols: `../04_model/output/feature_cols.joblib`
- Challenger is trained inline (not loaded from file)

### Analysis Sections

#### Read Data and Models
- Read train, valid, and test clean data from S3
- Combine train and valid into `df_train_valid` (the champion was trained on combined train+valid)
- Load champion XGBoost model and feature columns

#### Train Challenger Model
Markdown: explains challenger is an untuned SVM with RBF kernel + StandardScaler pipeline, serving as a baseline comparison. Trained on the same combined train+valid data as the champion for a fair comparison.
- Train `Pipeline([StandardScaler, SVC(kernel='rbf', probability=True)])` on combined train+valid data

#### Generate Predictions
- Score both models on the same out-of-time test set
- Build `dict_models` with `'Champion (XGBoost)'` and `'Challenger (SVM)'` keys

#### Metrics Comparison
- Compute AUC, Gini, KS, Brier, median pred for both
- Save to `output/champion_challenger_metrics.csv`

#### AUC, Gini, KS Comparisons
- Bar charts for each metric

#### ROC Comparison
- Overlaid ROC curves with AUC in legend

#### Calibration Comparison
- Overlaid calibration plots with Brier in legend

#### KDE Comparison
- Overlaid KDE with median in legend

#### Recommendation
- Automated logic based on AUC and Brier deltas
- Prints delta for each metric and a clear recommendation (promote, review, or retain)
