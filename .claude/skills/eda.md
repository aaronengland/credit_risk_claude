---
name: eda
description: Populates the EDA notebook in 01_eda/ with exploratory data analysis code for credit risk modeling. Follows a top-down approach from high-level overview to detailed feature analysis.
user_invocable: true
---

# Exploratory Data Analysis

Populate the `01_eda/notebook.ipynb` with exploratory data analysis code for credit risk modeling. The EDA follows a top-down approach, starting at the highest level and drilling down progressively.

## Instructions

Edit the existing `01_eda/notebook.ipynb`, preserving the template structure (imports, functions, constants, analysis). All variable names must use type prefixes (str_, flt_, int_, df_, list_, dict_, tpl_, etc.). All markdown headers use `####`. All plots must be saved to the `output/` folder.

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

1. `plot_data_types(df_data, str_filename='output/data_types.png')` - Takes a DataFrame. Creates a bar chart of the count of each data type (e.g., int64, float64, object). Saves to `str_filename`.

2. `plot_proportion_missing(df_data, str_filename='output/proportion_missing.png')` - Takes a DataFrame. Creates a horizontal bar chart showing the proportion of missing values per column, sorted descending. Only include columns with missing values. Saves to `str_filename`.

3. `plot_target(df_data, str_target, str_filename='output/target_distribution.png')` - Takes a DataFrame and the target column name. Creates a bar chart of target value counts with proportions annotated on the bars. Saves to `str_filename`.

4. `descriptive_statistics(df_data)` - Takes a DataFrame, selects numeric columns, returns `df_data.describe().T` with the index reset and the index column renamed to `str_column`.

5. `plot_violin(df_data, str_target, str_filename='output/violin_plots.png')` - Takes a DataFrame and target column name. Creates a grid of violin plots for every numeric feature, split/hued by the target variable. Saves to `str_filename`.

6. `plot_correlation_with_target(df_data, str_target, str_filename='output/correlation_with_target.png')` - Takes a DataFrame and target column name. Computes the correlation of every numeric feature with the target. Creates a horizontal bar chart sorted by absolute correlation. Saves to `str_filename`.

### Constants Cell

Keep the existing `str_bucket` and `str_step` constants. Add:

```python
# s3 path
str_s3_path = f's3://{str_bucket}/00_data_collection/data.csv'
print(f'S3 Path: {str_s3_path}')

# target
str_target = 'charged_off'
print(f'Target: {str_target}')

# output directory
os.makedirs('output', exist_ok=True)
```

### Analysis Section

Add the following analysis steps. Each step gets its own `####` markdown header cell followed by one or more code cells.

#### Read Data
- Read the CSV from S3 into `df_raw` using `pd.read_csv(str_s3_path)`
- Print the number of rows and columns
- If a date column exists, print the date range (min and max)
- Print the mean of the target variable
- Display `df_raw.head()`

#### Data Types
- Call `plot_data_types(df_raw)` to visualize the count of each data type
- Display the plot

#### Missing Values
- Call `plot_proportion_missing(df_raw)` to visualize missing value proportions
- Display the plot

#### Target Variable
- Call `plot_target(df_raw, str_target)` to visualize the target distribution
- Display the plot

#### Descriptive Statistics
- Call `descriptive_statistics(df_raw)` and store in `df_desc`
- Display `df_desc`
- Save `df_desc` to `output/descriptive_statistics.csv`

#### Violin Plots
- Call `plot_violin(df_raw, str_target)` to visualize distributions of all numeric features split by target
- Display the plot

#### Correlation with Target
- Call `plot_correlation_with_target(df_raw, str_target)` to visualize each feature's correlation with the target
- Display the plot

#### Save
- Save `df_raw` to `s3://{str_bucket}/01_eda/data.csv`
- Print confirmation message

## Notes

- The approach is always top-down: start with the big picture (shape, date range, target mean), then drill into types, missingness, target, distributions, and finally correlations.
- All plots use `plt.tight_layout()` before saving.
- All plots are saved to the `output/` folder for documentation purposes.
- Use `plt.figure(figsize=(...))` with appropriate sizes for readability.
- Use `plt.close()` after saving each plot to prevent display clutter.
