---
name: data-split
description: Populates the data split notebook in 02_data_split/ with out-of-time train/validation/test splitting for credit risk modeling.
user_invocable: true
---

# Data Split

Populate the `02_data_split/notebook.ipynb` with out-of-time data splitting code for credit risk modeling.

## Instructions

Edit the existing `02_data_split/notebook.ipynb`, preserving the template structure (imports, functions, constants, analysis). All variable names must use type prefixes. All markdown headers use `####`. All plots must be saved to the `output/` folder. The main DataFrame is always named `df`. Each function must be in its own code cell.

### Imports Cell

```python
import os
import boto3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. `plot_observations(df_train, df_valid, df_test, str_filename='output/observations_by_split.png')` - Bar chart showing the number of observations in each split with count and percent annotated on bars.

2. `plot_target_drift(df_train, df_valid, df_test, str_target, str_filename='output/target_drift.png')` - Bar chart showing the mean of the target in each split with mean value annotated on bars.

### Plot Rules (apply to ALL plotting functions)

- Always pad the y-axis so bar labels never overlap with the plot boundary. Use `ax.set_ylim(top=ax.get_ylim()[1] * 1.15)`.
- Do NOT rotate x-labels by default. Use `rot=0`.
- Always use `plt.tight_layout()` before saving.
- Always use `plt.savefig(str_filename, dpi=300)` then `plt.show()`.

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
- Use sorted date order to compute split indices
- `df_train` = first 70%, `df_valid` = next 15%, `df_test` = last 15%
- Build `df_split_summary` DataFrame with columns: `str_split`, `int_n_rows`, `int_n_cols`, `str_date_min`, `str_date_max`, `flt_target_mean`
- Save to `output/split_summary.csv` and display

#### Observations by Split
- `plot_observations(df_train, df_valid, df_test)`

#### Target Drift
- `plot_target_drift(df_train, df_valid, df_test, str_target)`

#### Save
- Save `df_train`, `df_valid`, `df_test` to S3 as .parquet files:
  - `s3://{str_bucket}/{str_step}/df_train.parquet`
  - `s3://{str_bucket}/{str_step}/df_valid.parquet`
  - `s3://{str_bucket}/{str_step}/df_test.parquet`
