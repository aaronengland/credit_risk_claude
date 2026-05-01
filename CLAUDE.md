# Credit Risk Model Validation Framework

This repository is a reusable framework for building credit risk models using Claude Code. It automates the full model lifecycle from EDA through deployment, following consistent standards for code, documentation, and regulatory compliance.

## How to Use

1. Clone this repository for a new model project
2. Place your raw data in S3 at `s3://{bucket}/00_data_collection/data.csv`
3. Run `/create-folder-structure` to scaffold the project
4. Run each step's skill individually, or run `/run-all` to execute the full pipeline
5. Run `/documentation` to generate the comprehensive README

## Available Skills

| Skill | Description |
|-------|-------------|
| `/create-folder-structure` | Scaffold project directories and template notebooks |
| `/eda` | Exploratory data analysis (top-down: overview, types, missing, target, distributions, correlations) |
| `/data-split` | Out-of-time train/validation/test split (70/15/15) |
| `/preprocessing` | sklearn pipeline with feature engineering, median imputation, ordinal encoding |
| `/model` | XGBoost with Bayesian tuning (Optuna), monotone constraints, early stopping |
| `/model-eval` | AUC, Gini, KS, PR AUC, Brier, calibration, KDE, confusion matrix, SHAP PDP |
| `/disparate-impact` | Fair lending analysis with proxy model and SHAP comparison |
| `/deployment` | FastAPI + Docker with logging |
| `/documentation` | Generate comprehensive README with all plots and tables |
| `/run-all` | Execute all skills sequentially |

## Notebook Conventions

All notebooks follow a standard template structure:

1. **Imports** - required libraries
2. **Functions** - one function per code cell, ordered to match execution order
3. **Constants** - bucket, step, S3 paths, target variable, output directory
4. **Analysis** - each step gets its own `####` markdown header with description

### Variable Naming

All variables must use a type prefix:

- `str_` (string), `flt_` (float), `int_` (integer), `df_` (DataFrame)
- `list_` (list), `dict_` (dictionary), `tpl_` (tuple), `arr_` (numpy array)
- `ser_` (pandas Series)

### Constants Template

Every notebook's constants cell must include:

```python
str_bucket = os.getcwd().split('/')[4].replace('_', '-')
str_step = os.getcwd().split('/')[-1]
os.makedirs('output', exist_ok=True)
```

### Data Naming

- The main DataFrame is always named `df` (not `df_raw`)
- Split DataFrames: `df_train`, `df_valid`, `df_test`
- Clean DataFrames: `df_train_clean`, `df_valid_clean`, `df_test_clean`

## Plotting Rules

- Save all plots to the `output/` folder with `dpi=300`
- Pad y-axis on bar charts so labels never clip: `ax.set_ylim(top=ax.get_ylim()[1] * 1.15)`
- Do NOT rotate x-labels by default. Use `rot=0`
- Legend placement: `loc='lower right'`
- Color palette: `steelblue` (primary), `salmon` (secondary), `seagreen` (tertiary)
- Suppress warnings in plotting functions with scoped `warnings.filterwarnings('ignore')`
- Always `plt.tight_layout()` before saving
- Always `plt.show()` after saving

## Preprocessing Rules

- NEVER drop columns in preprocessing. Column exclusion happens in the model notebook.
- NEVER fit preprocessors on anything but the training data.
- Save the preprocessing pipeline to `output/` immediately after fitting.
- Use `select_dtypes(include=[np.number])` to avoid datetime issues in describe/z-score calculations.

## S3 Data Convention

```
s3://{bucket}/00_data_collection/data.csv       # raw source data
s3://{bucket}/02_data_split/df_train.parquet     # train split
s3://{bucket}/02_data_split/df_valid.parquet     # validation split
s3://{bucket}/02_data_split/df_test.parquet      # test split
s3://{bucket}/03_preprocessing/df_train_clean.parquet
s3://{bucket}/03_preprocessing/df_valid_clean.parquet
s3://{bucket}/03_preprocessing/df_test_clean.parquet
```

## Credit Risk Domain Rules

- Use out-of-time splits (oldest = train, middle = validation, newest = test)
- Apply monotone constraints for regulatory directional consistency
- Exclude post-outcome variables (charged_off_amount, paid_interest_amount) as data leakage
- Exclude age for ECOA/fair lending compliance
- Exclude variables not known at time of application (apr, payment-to-income)
- Do NOT use scale_pos_weight or sample weights; train on natural class distribution for well-calibrated probabilities
- Always run disparate impact analysis on protected classes
- Gini and KS are standard discrimination metrics in credit risk
- Brier score validates calibration for pricing and capital allocation
