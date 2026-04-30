---
name: preprocessing
description: Populates the preprocessing notebook in 03_preprocessing/ with sklearn pipeline-based preprocessing for credit risk modeling with XGBoost.
user_invocable: true
---

# Preprocessing

Populate the `03_preprocessing/notebook.ipynb` with sklearn pipeline-based preprocessing code for credit risk modeling with XGBoost.

## Critical Rules

- NEVER fit preprocessors on anything but the training data. Fit on train, transform train/valid/test.
- Save the fitted pipeline to `output/` using joblib.
- Save preprocessed DataFrames to S3 as .parquet files.
- Each function must be in its own code cell.
- All variable names must use type prefixes. All markdown headers use `####`.

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
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OrdinalEncoder
from sklearn.impute import SimpleImputer
from sklearn.base import BaseEstimator, TransformerMixin
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. Custom transformer class `FeatureEngineer(BaseEstimator, TransformerMixin)`:
   - Drops leaky/ID columns: `loan_id`, `charged_off_amount`, `paid_interest_amount`
   - Drops `origination_date` (used for splitting, not a feature)
   - Engineers `int_age` from `dob` (years between dob and origination_date, or current date if origination_date already dropped)
   - Engineers `flt_payment_to_income` = `loan_amount / stated_income`
   - Drops `dob` after extracting age
   - Returns a DataFrame

2. `plot_missing_after_preprocessing(df_before, df_after, str_filename='output/missing_before_after.png')` - Grouped bar chart comparing proportion missing before and after preprocessing.

### Constants Cell

Standard bucket, step, target, plus:
- `str_s3_input` pointing to `s3://{str_bucket}/02_data_split/`
- Output directory

### Analysis Section

#### Read Data
- Read `df_train`, `df_valid`, `df_test` from S3 parquet files in `02_data_split/`

#### Define Pipeline
- Define column lists for numeric and categorical features (after feature engineering)
- Build sklearn `Pipeline` with:
  1. `FeatureEngineer` custom transformer
  2. `ColumnTransformer` with:
     - Numeric: `SimpleImputer(strategy='median')` then passthrough (no scaling for XGBoost)
     - Categorical: `SimpleImputer(strategy='constant', fill_value='missing')` then `OrdinalEncoder(handle_unknown='use_encoded_value', unknown_value=-1)`

#### Fit and Transform
- Fit pipeline on `df_train` only (NEVER on valid/test)
- Transform `df_train`, `df_valid`, `df_test`
- Reconstruct DataFrames with proper column names

#### Missing Values Before and After
- `plot_missing_after_preprocessing(...)` comparing before/after

#### Save
- Save pipeline to `output/preprocessing_pipeline.joblib`
- Save `df_train_clean`, `df_valid_clean`, `df_test_clean` to S3 as parquet
