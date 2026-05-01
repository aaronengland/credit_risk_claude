---
name: disparate-impact
description: Populates the disparate impact notebook in 06_disparate_impact/ with fair lending analysis comparing model outcomes across protected age groups.
user_invocable: true
---

# Disparate Impact

Populate the `06_disparate_impact/notebook.ipynb` with fair lending disparate impact analysis on the test set only.

## Instructions

Edit the existing `06_disparate_impact/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. All plots saved to `output/`. Each function in its own code cell. Legend placement: `loc='lower right'`.

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
import xgboost as xgb
from sklearn.metrics import roc_auc_score
from scipy.stats import ks_2samp
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. `compute_metrics_by_group(df_data, str_group_col, str_pred_col, str_target)` - Returns DataFrame with per-group: str_group, int_n, flt_target_mean, flt_pred_mean, flt_pred_median, flt_auc.

2. `plot_kde_by_group(df_data, str_group_col, str_pred_col, str_filename)` - KDE of predictions by group with median in legend. Colors: steelblue, salmon.

3. `plot_metric_comparison(df_metrics, str_metric, str_title, str_filename)` - Bar chart comparing a single metric across groups with values annotated. Y-axis padded.

4. `plot_pred_vs_actual_by_group(df_metrics, str_filename='output/pred_vs_actual_by_age.png')` - Grouped bar chart showing median predicted PD (steelblue), mean predicted PD (seagreen), and actual default rate (salmon) side by side per age group with values annotated. Y-axis padded.

5. `plot_age_vs_credit_importance(model_age, model_credit, arr_X_age, arr_X_credit, list_cols, str_filename)` - Grouped horizontal bar chart comparing SHAP importance from the age proxy model (salmon) vs the credit risk model (steelblue). Both normalized to 0-1, sorted by age SHAP. Highlights features that are important for both predicting age and default.

6. `plot_age_shap_pdp(model_age, arr_X, list_cols, str_filename)` - Grid of SHAP partial dependence scatter plots from the age proxy model. 3-column layout, salmon color. Shows each feature's relationship with age group prediction.

7. `bootstrap_auc_ci(arr_y_true, arr_y_pred, int_n_bootstrap=1000, flt_ci=0.95)` - Resample with replacement int_n_bootstrap times, compute AUC for each resample, return (flt_mean, flt_lower, flt_upper) for the confidence interval. Skips resamples where only one class is present.

### Constants Cell

```python
# bucket, step
# s3 input: s3://{str_bucket}/03_preprocessing
# target: default_12m
# model path: ../04_model/output/xgboost_model.joblib
# feature cols path: ../04_model/output/feature_cols.joblib
# age column: int_age
# age threshold: 60
# output directory
```

### Analysis Section

#### Read Data and Model
- Read test data only from S3 parquet
- Load credit risk model and feature columns from 04_model/output/

#### Define Age Groups
Markdown: ECOA context, int_age excluded from model features but must verify no disparate outcomes.
- Generate credit risk predictions on test set
- Create `str_age_group` column (< 60 vs >= 60)

#### Metrics by Age Group
Markdown: explains comparison of metrics between groups.
- `compute_metrics_by_group`, save to `output/disparate_impact_metrics.csv`

#### Predicted PD vs Actual Default Rate by Age Group
- `plot_pred_vs_actual_by_group(df_di_metrics)` - single grouped bar chart comparing median predicted PD, mean predicted PD, and actual default rate side by side

#### AUC by Age Group
- `plot_metric_comparison` for flt_auc

#### KDE of Predictions by Age Group
Markdown: similar distributions suggest comparable treatment.
- `plot_kde_by_group`

#### KS Test
Markdown: formal statistical test for distributional differences.
- KS test between age group predictions, print statistic and p-value

#### Bootstrap Confidence Intervals on AUC
Markdown: with small sample sizes, point estimates can be misleading and bootstrap CIs quantify uncertainty around AUC differences.
- Compute bootstrap 95% CIs on AUC for each age group using `bootstrap_auc_ci`
- Compute the bootstrap distribution of the AUC difference (group1 - group2) by resampling each group independently
- Print AUC with CI for each group
- Print AUC difference with CI
- State whether the CI for the difference includes zero (not statistically significant if yes)

#### Age Proxy Model
Markdown: train XGBoost to predict age group using credit model features to identify proxy variables. Note that this proxy model is trained and scored on the same data, so the AUC is an upper-bound estimate. In production, cross-validation should be used for a more conservative estimate.
- Quick XGBoost (100 trees, depth 4, random_state=42) to predict age group
- Print proxy model AUC

#### Age Proxy vs Credit Risk Feature Importance
Markdown: features important for both age and default prediction are proxy concerns.
- `plot_age_vs_credit_importance` comparing SHAP from both models

#### Age Proxy SHAP Partial Dependence Plots
Markdown: reveals direction/shape of each feature's relationship with age.
- `plot_age_shap_pdp` on test set features
