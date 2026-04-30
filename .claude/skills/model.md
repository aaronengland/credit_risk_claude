---
name: model
description: Populates the model notebook in 04_model/ with Bayesian hyperparameter tuning and XGBoost training for credit risk modeling.
user_invocable: true
---

# Model

Populate the `04_model/notebook.ipynb` with XGBoost model training using Bayesian hyperparameter tuning for credit risk modeling.

## Critical Rules

- Load preprocessed data from `03_preprocessing/` in S3.
- Exclude non-feature columns (IDs, leaky variables, dates) here, not in preprocessing.
- Use Optuna for Bayesian hyperparameter tuning.
- Use validation data for early stopping.
- Save the trained XGBoost model to `output/` using joblib.
- Each function must be in its own code cell.
- All variable names must use type prefixes. All markdown headers use `####`.
- Each major step gets its own markdown description explaining the what and why.

## Instructions

### Imports Cell

```python
import os
import warnings
import boto3
import joblib
import optuna
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import xgboost as xgb
from sklearn.metrics import roc_auc_score
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. `plot_optimization_history(study, str_filename='output/optimization_history.png')` - Line plot of objective value (AUC) over trials.

2. `plot_feature_importance(model, list_feature_cols, str_filename='output/feature_importance.png')` - Horizontal bar chart of XGBoost feature importance sorted by importance. Use gain-based importance.

### Constants Cell

Standard bucket, step, target, plus:
- `str_s3_input` pointing to `s3://{str_bucket}/03_preprocessing/`
- `list_exclude_cols` - columns to exclude from features (loan_id, origination_date, dob, charged_off_amount, paid_interest_amount, and target)
- `int_n_trials` - number of Optuna trials
- Output directory

### Analysis Section

#### Read Data
- Read `df_train_clean`, `df_valid_clean`, `df_test_clean` from S3 parquet

#### Define Features
Markdown: explains which columns are excluded and why (IDs, leaky post-outcome variables, dates).
- Define `list_feature_cols` by excluding `list_exclude_cols`
- Separate features and target for each split

#### Bayesian Hyperparameter Tuning
Markdown: explains why Bayesian optimization (more efficient than grid/random search, models the objective function). Explains early stopping prevents overfitting and finds optimal n_estimators automatically.
- Define Optuna objective function that:
  - Sets XGBoost hyperparameters from trial suggestions
  - Uses scale_pos_weight for class imbalance
  - Trains with early stopping on validation set
  - Returns validation AUC
- Run study with `int_n_trials` trials

#### Optimization History
- `plot_optimization_history(study)`

#### Train Final Model
Markdown: explains retraining with best hyperparameters.
- Train XGBoost with best params and early stopping on validation
- Print best validation AUC

#### Feature Importance
- `plot_feature_importance(model, list_feature_cols)`

#### Save
- Save model to `output/xgboost_model.joblib`
- Save best hyperparameters to `output/best_params.csv`
