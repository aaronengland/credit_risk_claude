---
name: eda
description: Populates the EDA notebook in 01_eda/ with exploratory data analysis code for credit risk modeling. Follows a top-down approach from high-level overview to detailed feature analysis.
user_invocable: true
---

# Exploratory Data Analysis

Populate the `01_eda/notebook.ipynb` with exploratory data analysis code for credit risk modeling. The EDA follows a top-down approach, starting at the highest level and drilling down progressively.

## Instructions

Edit the existing `01_eda/notebook.ipynb`, preserving the template structure (imports, functions, constants, analysis). All variable names must use type prefixes (str_, flt_, int_, df_, list_, dict_, tpl_, etc.). All markdown headers use `####`. All plots must be saved to the `output/` folder. The main data frame is always named `df` (not `df_raw`).

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

Add the following helper functions. All variable names inside functions must also use type prefixes.

1. `plot_data_types(df_data, str_filename='output/data_types.png')` - Bar chart of dtype counts. Annotate each bar with count and percent. Use `rot=0` for x-labels. Pad y-axis top with `ax.set_ylim(top=ax.get_ylim()[1] * 1.15)` so labels never clip.

2. `plot_proportion_missing(df_data, str_filename='output/proportion_missing.png')` - Horizontal bar chart of proportion missing per column (only columns with missing values), sorted descending.

3. `plot_target(df_data, str_target, str_filename='output/target_distribution.png')` - Bar chart of target value counts with count and proportion annotated. Use `rot=0`. Pad y-axis top so labels never clip.

4. `descriptive_statistics(df_data)` - Returns `df_data.describe().T` with index reset and renamed to `str_column`. Add `int_n_unique` (unique count per column) and `flt_proportion_missing` (proportion missing per column).

5. `plot_violin(df_data, str_target, str_filename='output/violin_plots.png')` - Grid of violin plots for every numeric feature (excluding target), using `hue=str_target`. Suppress warnings with `warnings.filterwarnings('ignore')` and restore after.

6. `plot_correlation_with_target(df_data, str_target, str_filename='output/correlation_with_target.png')` - Horizontal bar chart of each numeric feature's correlation with target, sorted by absolute value. Positive = steelblue, negative = salmon.

### Plot Rules (apply to ALL plotting functions)

- Always pad the y-axis (or x-axis for horizontal bars) so bar labels never overlap with the plot boundary. Use `ax.set_ylim(top=ax.get_ylim()[1] * 1.15)` for vertical bar charts.
- Do NOT rotate x-labels by default. Use `rot=0`. Only rotate if labels are long strings that would overlap.
- Always use `plt.tight_layout()` before saving.
- Always use `plt.savefig(str_filename, dpi=300)` then `plt.show()`.

### Constants Cell

Keep the existing `str_bucket` and `str_step` constants. Add:

```python
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

#### Data Types
- `plot_data_types(df)`

#### Missing Values
- `plot_proportion_missing(df)`

#### Target Variable
- `plot_target(df, str_target)`

#### Descriptive Statistics
- `df_desc = descriptive_statistics(df)`, save to CSV, display

#### Violin Plots
- `plot_violin(df, str_target)`

#### Correlation with Target
- `plot_correlation_with_target(df, str_target)`

#### Save
- Save `df` to `s3://{str_bucket}/{str_step}/data.csv`
