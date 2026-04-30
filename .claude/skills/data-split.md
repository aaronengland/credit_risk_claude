---
name: data-split
description: Populates the data split notebook in 02_data_split/ with out-of-time train/validation/test splitting for credit risk modeling.
user_invocable: true
---

# Data Split

Populate the `02_data_split/notebook.ipynb` with out-of-time data splitting code for credit risk modeling.

## Instructions

Edit the existing `02_data_split/notebook.ipynb`, preserving the template structure (imports, functions, constants, analysis). All variable names must use type prefixes. All markdown headers use `####`. All plots must be saved to the `output/` folder. The main DataFrame is always named `df`.

### Imports Cell

```python
import os
import boto3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

### Functions Cell

1. `plot_target_drift(df_train, df_valid, df_test, str_target, str_filename='output/target_drift.png')` - Bar chart showing the mean of the target in each dataset (train, validation, test). Annotate each bar with the mean value. Use `rot=0`. Pad y-axis so labels never clip.

### Constants Cell

```python
# bucket
str_bucket = os.getcwd().split('/')[4].replace('_', '-')
print(f'Bucket: {str_bucket}')

# step
str_step = os.getcwd().split('/')[-1]
print(f'Step: {str_step}')

# s3 path
str_s3_path = f's3://{str_bucket}/00_data_collection/data.csv'
print(f'S3 Path: {str_s3_path}')

# target
str_target = 'default_12m'
print(f'Target: {str_target}')

# date column
str_date_col = 'origination_date'
print(f'Date Column: {str_date_col}')

# split proportions
flt_train = 0.70
flt_valid = 0.15
flt_test = 0.15

# output directory
os.makedirs('output', exist_ok=True)
```

### Analysis Section

#### Read Data
- `df = pd.read_csv(str_s3_path)`
- Convert date column to datetime
- Sort by date column ascending
- Print shape

#### Split Data
- Use the sorted date order to split into `df_train` (first 70%), `df_valid` (next 15%), `df_test` (last 15%)
- Print shape and date range for each split

#### Target Drift
- `plot_target_drift(df_train, df_valid, df_test, str_target)`

#### Save
- Save `df_train`, `df_valid`, `df_test` to S3 as .parquet files:
  - `s3://{str_bucket}/{str_step}/df_train.parquet`
  - `s3://{str_bucket}/{str_step}/df_valid.parquet`
  - `s3://{str_bucket}/{str_step}/df_test.parquet`
- Print confirmation messages
