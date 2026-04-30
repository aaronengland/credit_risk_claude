---
name: preprocessing
description: Populates the preprocessing notebook in 03_preprocessing/ with sklearn pipeline-based preprocessing for credit risk modeling with XGBoost.
user_invocable: true
---

# Preprocessing

Populate the `03_preprocessing/notebook.ipynb` with sklearn pipeline-based preprocessing code for credit risk modeling with XGBoost.

## Critical Rules

- NEVER fit preprocessors on anything but the training data. Fit on train, transform train/valid/test.
- NEVER drop any columns in preprocessing. Column exclusion happens in 04_model.
- Save the fitted pipeline to `output/` using joblib immediately after fitting.
- Save preprocessed DataFrames to S3 as .parquet files.
- Each function must be in its own code cell.
- All variable names must use type prefixes. All markdown headers use `####`.
- Each pipeline step gets its own markdown description explaining what it does and why.

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
   - Engineers `int_age` from `dob` (years between dob and current date). Does NOT drop `dob`.
   - Engineers `flt_payment_to_income` = `loan_amount / stated_income`
   - Does NOT drop any columns. Column exclusion is handled in 04_model.
   - Returns a DataFrame

2. `plot_missing_before_after(df_before, df_after, str_filename='output/missing_before_after.png')` - Grouped horizontal bar chart comparing proportion missing before and after preprocessing. Only shows columns that had missing values before.

### Constants Cell

```python
# bucket
str_bucket = os.getcwd().split('/')[4].replace('_', '-')
print(f'Bucket: {str_bucket}')

# step
str_step = os.getcwd().split('/')[-1]
print(f'Step: {str_step}')

# s3 input path
str_s3_input = f's3://{str_bucket}/02_data_split'
print(f'S3 Input: {str_s3_input}')

# target
str_target = 'default_12m'
print(f'Target: {str_target}')

# output directory
os.makedirs('output', exist_ok=True)
```

### Analysis Section

Each step gets its own `####` markdown header with a description explaining the what and why.

#### Read Data
- Read `df_train`, `df_valid`, `df_test` from S3 parquet files in `02_data_split/`
- Print shapes

#### Feature Engineering
Markdown: explains that FeatureEngineer creates new features (age, payment-to-income) without dropping any columns. Column exclusion deferred to 04_model to keep preprocessing fast.
- Apply `FeatureEngineer` to `df_train` to identify column types
- Separate numeric and categorical feature lists (excluding target)

#### Numeric Preprocessing
Markdown: explains median imputation (robust to skewed distributions common in credit risk data). No scaling because XGBoost is tree-based and invariant to monotonic transformations.
- Define `numeric_transformer` with `SimpleImputer(strategy='median')`

#### Categorical Preprocessing
Markdown: explains constant imputation ("missing" as its own category, since missingness can be informative in credit risk). Ordinal encoding preferred over one-hot for XGBoost (avoids sparse features, XGBoost learns splits on ordinal values). Unknown categories encoded as -1.
- Define `categorical_transformer` with `SimpleImputer(strategy='constant', fill_value='missing')` then `OrdinalEncoder(handle_unknown='use_encoded_value', unknown_value=-1)`

#### Assemble Pipeline
Markdown: explains the pipeline chains FeatureEngineer with ColumnTransformer, encapsulated in a single serializable object for API reuse.
- Build `ColumnTransformer` and wrap in `Pipeline`

#### Fit
Markdown: explains fitting on training data only to prevent data leakage.
- `pipeline.fit(df_train)`

#### Save (pipeline)
- `joblib.dump(pipeline, 'output/preprocessing_pipeline.joblib')` immediately after fitting

#### Transform
- Transform all three splits using fitted pipeline
- Reconstruct DataFrames with proper column names
- Add target column back
- Print shapes, display head

#### Missing Values Before and After
- `plot_missing_before_after(df_train_fe, df_train_clean)`

#### Save (data)
- Save `df_train_clean`, `df_valid_clean`, `df_test_clean` to S3 as parquet:
  - `s3://{str_bucket}/{str_step}/df_train_clean.parquet`
  - `s3://{str_bucket}/{str_step}/df_valid_clean.parquet`
  - `s3://{str_bucket}/{str_step}/df_test_clean.parquet`
