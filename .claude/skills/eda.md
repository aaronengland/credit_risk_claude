---
name: eda
description: Populates the EDA notebook in 01_eda/ with exploratory data analysis code for credit risk modeling. Follows a top-down approach from high-level overview to detailed feature analysis.
user_invocable: true
---

# Exploratory Data Analysis

Populate the `01_eda/notebook.ipynb` with exploratory data analysis code for credit risk modeling. The EDA follows a top-down approach, starting at the highest level and drilling down progressively.

## Instructions

Edit the existing `01_eda/notebook.ipynb`, preserving the template structure (imports, functions, constants, analysis). All variable names must use type prefixes (str_, flt_, int_, df_, list_, dict_, tpl_, etc.). All markdown headers use `####`. All plots must be saved to the `output/` folder. The main DataFrame is always named `df`. Each function must be in its own code cell.

### Imports Cell

```python
import os
import boto3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

### Functions Cell

One function per code cell, ordered to match execution order.

1. `descriptive_statistics(df_data)` - Runs `df_data.select_dtypes(include=[np.number]).describe().T` with index reset and renamed to `str_column`. Adds:
   - `str_dtype` - data type of each column
   - `int_n_unique` - unique count per column
   - `flt_proportion_missing` - proportion missing per column
   - `flt_min_zscore` - z-score of the column minimum (guards against std == 0)
   - `flt_max_zscore` - z-score of the column maximum (guards against std == 0)
   - `int_outliers` - 1 if abs of min or max z-score >= 3, else 0

2. `plot_data_types(df_desc, str_filename='output/data_types.png')` - Takes `df_desc` (output of `descriptive_statistics`). Bar chart of `str_dtype` value counts with count and percent annotated on bars.

3. `plot_proportion_missing(df_desc, str_filename='output/proportion_missing.png')` - Takes `df_desc`. Horizontal bar chart of `flt_proportion_missing` for columns where it is > 0, sorted descending.

4. `plot_target(df_data, str_target, str_filename='output/target_distribution.png')` - Bar chart of target value counts with count and proportion annotated on bars.

5. `plot_violin(df_data, str_target, str_filename='output/violin_plots.png')` - Grid of violin plots for every numeric feature (excluding target), using `hue=str_target`. Suppress warnings with `warnings.filterwarnings('ignore')` and restore after.

6. `plot_correlation_with_target(df_data, str_target, str_filename='output/correlation_with_target.png')` - Horizontal bar chart of each numeric feature's correlation with target, sorted by absolute value. Positive = steelblue, negative = salmon.

### Plot Rules (apply to ALL plotting functions)

- Always pad the y-axis so bar labels never overlap with the plot boundary. Use `ax.set_ylim(top=ax.get_ylim()[1] * 1.15)` for vertical bar charts.
- Do NOT rotate x-labels by default. Use `rot=0`. Only rotate if labels are long strings that would overlap.
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

# output directory
os.makedirs('output', exist_ok=True)
```

### Analysis Section

Each step gets its own `####` markdown header cell followed by one or more code cells. The main DataFrame is always named `df`.

#### Read Data
- `df = pd.read_csv(str_s3_path)`
- Print rows, columns, date range (auto-detect date columns), target mean
- `df.head()`

#### Descriptive Statistics
- `df_desc = descriptive_statistics(df)`, save to `output/descriptive_statistics.csv`, display

#### Data Types
- `plot_data_types(df_desc)`

#### Missing Values
- `plot_proportion_missing(df_desc)`

#### Target Variable
- `plot_target(df, str_target)`

#### Violin Plots
- `plot_violin(df, str_target)`

#### Correlation with Target
- `plot_correlation_with_target(df, str_target)`

## Notes

- EDA does not save data to S3. It is purely for analysis and documentation (plots saved to `output/`).
- The approach is always top-down: start with the big picture (shape, date range, target mean), then descriptive stats, then drill into types, missingness, target, distributions, and correlations.
- `plot_data_types` and `plot_proportion_missing` take `df_desc` as input, not `df`.
