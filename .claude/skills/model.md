---
name: model
description: Populates the model notebook in 04_model/ with Bayesian hyperparameter tuning and XGBoost training for credit risk modeling.
user_invocable: true
---

# Model

Populate the `04_model/notebook.ipynb` with XGBoost model training using Bayesian hyperparameter tuning for credit risk modeling.

## Critical Rules

- Load preprocessed data from `03_preprocessing/` in S3.
- Exclude non-feature columns (IDs, leaky variables, dates, compliance-sensitive) here, not in preprocessing.
- Use Optuna for Bayesian hyperparameter tuning with `show_progress_bar=True`.
- Use validation data for early stopping.
- Apply monotone constraints for regulatory directional consistency.
- Do NOT use `scale_pos_weight` or sample weights. Train on natural class distribution.
- Save the trained XGBoost model to `output/` using joblib.
- Each function must be in its own code cell.
- All variable names must use type prefixes. All markdown headers use `####`.
- Each major step gets its own markdown description explaining the what and why.
- Legend placement: `loc='lower right'` on plots with legends.

## Instructions

### Imports Cell

```python
import os
import warnings
import boto3
import joblib
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.metrics import roc_auc_score
import shap
import optuna
import xgboost as xgb
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. `plot_optimization_history(study, str_filename='output/optimization_history.png')` - Line plot of trial AUC values with running best line. Trial AUC in steelblue with markers, best AUC line in salmon.

2. `plot_feature_importance(model, arr_X, list_feature_cols, str_filename='output/feature_importance.png')` - Grouped horizontal bar chart with both gain-based (steelblue) and SHAP (salmon) importance, normalized to 0-1 for comparison. Sorted by SHAP values. Legend at `loc='lower right'`.

3. `objective(trial)` - Optuna objective function. Returns validation AUC. Uses monotone constraints, early stopping, no scale_pos_weight.

### Constants Cell

```python
# bucket, step, s3 input, target
# columns to exclude from features:
#   loan_id (identifier), origination_date (splitting), dob (replaced by int_age),
#   charged_off_amount/paid_interest_amount (post-outcome leakage),
#   apr (not known at application), flt_payment_to_income (depends on apr),
#   int_age (fair lending compliance), state (excluded), target
# monotone constraints dict:
#   bureau_score: -1, delinq_12m: +1, utilization: +1, inquiries_6m: +1,
#   public_records: +1, employment_length_years: -1, stated_income: -1,
#   loan_amount: +1, term_months: +1, has_prior_loans_with_us: 0, channel: 0
# int_n_trials = 50
# output directory
```

### Analysis Section

#### Read Data
- Read `df_train_clean`, `df_valid_clean`, `df_test_clean` from S3 parquet

#### Define Features
Markdown: explains which columns are excluded and why. Explains monotone constraints for regulatory directional consistency.
- Define `list_feature_cols` by excluding `list_exclude_cols`
- Build `tpl_monotone_constraints` from `dict_monotone_constraints` in feature column order
- Separate features (arr_X) and target (arr_y) for each split

#### Bayesian Hyperparameter Tuning
Markdown: explains Bayesian optimization efficiency, early stopping, no weights on natural distribution.
- Run `study.optimize(objective, n_trials=int_n_trials, show_progress_bar=True)`

#### Optimization History
- `plot_optimization_history(study)`

#### Train Final Model
Markdown: explains that the final model is trained on combined train + validation data to maximize learning. The optimal number of boosting rounds (`n_estimators`) is set to `best_iteration` from the tuning phase (determined via early stopping on validation). Since the number of rounds is fixed, no early stopping or holdout is needed. The test set remains untouched.
- Reconstruct best trial model with early stopping on validation to get `best_iteration`
- Combine `arr_X_train` + `arr_X_valid` into `arr_X_combined`, same for targets
- Train final XGBoost with best params, monotone constraints, and `n_estimators=int_best_iteration` (no early stopping)

#### Feature Importance
- `plot_feature_importance(model, arr_X_combined, list_feature_cols)` - grouped gain/SHAP chart

#### Save
- Save model to `output/xgboost_model.joblib`
- Save best hyperparameters to `output/best_params.csv`
- Save feature columns to `output/feature_cols.joblib`
