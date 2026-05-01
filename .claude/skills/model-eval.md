---
name: model-eval
description: Populates the model evaluation notebook in 05_model_eval/ with metrics, curves, and SHAP analysis for credit risk model performance assessment.
user_invocable: true
---

# Model Evaluation

Populate the `05_model_eval/notebook.ipynb` with model evaluation code for credit risk modeling.

## Instructions

Edit the existing `05_model_eval/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. All plots saved to `output/`. Each function in its own code cell. Legend placement: `loc='lower right'`.

### Imports Cell

```python
import os
import warnings
import boto3
import joblib
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import shap
from sklearn.metrics import roc_auc_score, roc_curve, precision_recall_curve, average_precision_score, brier_score_loss, confusion_matrix, ConfusionMatrixDisplay
from scipy.stats import ks_2samp
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. `compute_metrics(arr_y_true, arr_y_pred)` - Returns dict with: `flt_auc`, `flt_gini` (2*AUC-1), `flt_ks` (KS statistic), `flt_pr_auc` (average precision), `flt_brier` (Brier score), `flt_median_pred` (median predicted probability), `flt_mean_pred` (mean predicted probability), `flt_target_mean` (actual default rate).

2. `plot_roc_curves(dict_splits, str_filename)` - ROC curves for all splits. AUC in legend per split. Colors: steelblue, salmon, seagreen.

3. `plot_precision_recall_curves(dict_splits, str_filename)` - PR curves for all splits. PR AUC in legend per split.

4. `plot_calibration(dict_splits, int_n_bins=10, str_filename)` - Calibration plot (predicted vs actual by decile). Brier score in legend per split.

5. `plot_kde_predictions(dict_splits, str_filename)` - KDE of predicted probabilities across splits. Median in legend per split. Suppress warnings.

6. `plot_confusion_matrix(arr_y_true, arr_y_pred, flt_threshold=0.5, str_filename)` - Confusion matrix on a single split (test set).

7. `plot_shap_pdp(model, arr_X, list_feature_cols, str_filename)` - Grid of SHAP partial dependence scatter plots (feature value vs SHAP value) for each feature. 3-column layout. Horizontal line at y=0.

8. `plot_decile_analysis(arr_y_true, arr_y_pred, str_filename)` - Bins predictions into 10 deciles using `pd.qcut`, with decile 1 = highest risk (highest predicted PD) and decile 10 = lowest risk (standard credit risk convention). For each decile computes predicted PD range, count, actual default rate, and cumulative capture rate (accumulating from the highest-risk decile downward so that "top N deciles capture X% of defaults"). Prints a summary DataFrame table. Plots a bar chart of actual default rate by decile with PD range as x-labels. Steelblue bars with black edgecolor, annotated with actual default rate. Uses `rotation=45` for x-labels.

9. `plot_threshold_analysis(arr_y_true, arr_y_pred, str_filename)` - Evaluates thresholds at 0.10, 0.15, 0.20, 0.25, 0.30, 0.50. For each threshold computes precision, recall, F1, and approval rate (proportion below threshold). Prints a summary DataFrame table. Plots a multi-line chart with precision (steelblue), recall (salmon), F1 (seagreen), and approval rate (gray dashed). Legend at lower right.

10. `generate_reason_codes(model, arr_X, list_feature_cols, int_top_n=4, str_filename)` - Generates SHAP-based adverse action reason codes per applicant (regulatory requirement under ECOA/CFPB). Uses `shap.TreeExplainer` to compute SHAP values, then for each observation selects the top N features with positive SHAP values (pushing toward default). Maps feature names to consumer-facing reason descriptions via `dict_reason_map`. Displays a DataFrame of the top 10 highest-risk applicants with their top 4 reason codes. Plots a horizontal bar chart of reason frequency (% of applicants) using steelblue bars with black edgecolor. Returns `list_reasons` (list of lists of reason strings per applicant).

### Constants Cell

```python
# bucket, step
# s3 input: s3://{str_bucket}/03_preprocessing
# target: default_12m
# model path: ../04_model/output/xgboost_model.joblib
# feature cols path: ../04_model/output/feature_cols.joblib
# output directory
```

### Analysis Section

#### Read Data and Model
- Read `df_train_clean`, `df_valid_clean`, `df_test_clean` from S3 parquet
- Combine train and valid into `df_train_valid` (the model was trained on this combined set)
- Load model and feature columns from `04_model/output/`

#### Generate Predictions
- Generate predicted probabilities for train+valid (combined) and test
- Build `dict_splits` mapping split name to (arr_y_true, arr_y_pred) tuples: `'Train+Valid'` and `'Test'`

#### Metrics Summary
Markdown: explains AUC, Gini, KS, PR AUC, Brier, median prediction. Notes that the model was trained on combined train+valid data, so Train+Valid metrics reflect in-sample performance while Test is the true out-of-time holdout.
- `compute_metrics` for each split
- Build `df_metrics` with columns: str_split, flt_auc, flt_gini, flt_ks, flt_pr_auc, flt_brier, flt_median_pred, flt_mean_pred, flt_target_mean
- Save to `output/metrics_summary.csv`

#### ROC Curves
- `plot_roc_curves(dict_splits)`

#### Precision-Recall Curves
- `plot_precision_recall_curves(dict_splits)`

#### Calibration
Markdown: explains calibration importance for credit risk pricing and capital allocation.
- `plot_calibration(dict_splits)`

#### KDE of Predictions
Markdown: explains distribution stability across splits, divergence indicates overfitting or drift.
- `plot_kde_predictions(dict_splits)`

#### Confusion Matrix (Test Set)
- `plot_confusion_matrix(arr_y_test, arr_pred_test)`

#### SHAP Partial Dependence Plots
Markdown: explains PDP for verifying monotone constraints and understanding nonlinear relationships. Uses test set only for unbiased interpretation.
- `plot_shap_pdp(model, df_test[list_feature_cols].values, list_feature_cols)`

#### Decile Analysis
Markdown: explains this is standard in credit risk for evaluating rank-ordering within score bands. A well-discriminating model shows monotonically increasing default rates from lowest to highest decile.
- `plot_decile_analysis(arr_y_test, arr_pred_test)`

#### Threshold Sensitivity Analysis
Markdown: explains that the optimal cutoff depends on the cost of false positives vs false negatives and varies by business use case. Shows precision, recall, F1, and approval rate across thresholds.
- `plot_threshold_analysis(arr_y_test, arr_pred_test)`

#### Adverse Action Reason Codes
Markdown: explains that under ECOA and CFPB guidance, lenders must provide specific, accurate reasons when taking adverse action. SHAP values identify the top features pushing each applicant toward default, mapped to consumer-facing descriptions. Top 4 reasons per applicant.
- `list_reason_codes = generate_reason_codes(model, df_test[list_feature_cols].values, list_feature_cols)`
