# credit_risk_claude

A credit risk modeling project built with Claude Code, demonstrating an end-to-end machine learning workflow from exploratory data analysis through model deployment.

## Project Overview

This project builds a credit risk model using XGBoost, with data sourced from S3 and the final model served via a FastAPI application packaged in Docker. Every step of the modeling process is organized into its own directory with a dedicated Jupyter notebook, and each step has a corresponding Claude Code skill for reproducible, automated execution.

## Folder Structure

```
credit_risk_claude/
├── 01_eda/                  # Exploratory data analysis
│   └── notebook.ipynb
├── 02_data_split/           # Train/test/validation splitting
│   └── notebook.ipynb
├── 03_preprocessing/        # Feature engineering and preprocessing
│   └── notebook.ipynb
├── 04_model/                # XGBoost model training and evaluation
│   └── notebook.ipynb
├── 05_model_eval/           # Model evaluation and performance metrics
│   └── notebook.ipynb
├── 06_disparate_impact/     # Fair lending and disparate impact analysis
│   └── notebook.ipynb
├── api/                     # Model serving
│   ├── app.py               # FastAPI application
│   ├── Dockerfile           # Docker image definition
│   └── requirements.txt     # Python dependencies
├── notebook.ipynb           # Notebook template
├── .claude/
│   └── skills/              # Claude Code skills for each step
└── README.md
```

## Modeling Steps

| Step | Directory | Description |
|------|-----------|-------------|
| 1 | `01_eda/` | Explore the dataset, examine distributions, identify missing values, and assess target variable balance |
| 2 | `02_data_split/` | Split data into train, test, and validation sets with stratification on the target |
| 3 | `03_preprocessing/` | Handle missing values, encode categorical features, and engineer new features |
| 4 | `04_model/` | Train an XGBoost classifier and tune hyperparameters |
| 5 | `05_model_eval/` | Evaluate model performance across train, validation, and test sets |
| 6 | `06_disparate_impact/` | Fair lending analysis and disparate impact testing |
| 7 | `api/` | Serve the trained model via FastAPI, packaged as a Docker image |

## Claude Code Skills

This project uses Claude Code skills to automate each step of the modeling workflow. Skills can be invoked independently or run sequentially.

| Skill | Description |
|-------|-------------|
| `/create-folder-structure` | Scaffolds the project directory structure with template notebooks |
| `/eda` | Populates the EDA notebook with analysis code |
| `/data-split` | Populates the data split notebook |
| `/preprocessing` | Populates the preprocessing notebook |
| `/model` | Populates the model training notebook |
| `/model-eval` | Populates the model evaluation notebook |
| `/disparate-impact` | Populates the disparate impact analysis notebook |
| `/deployment` | Creates the FastAPI app, Dockerfile, and requirements |
| `/run-all` | Runs all skills sequentially |

## Notebook Conventions

All notebooks follow a standard template structure:

1. **Imports** - required libraries
2. **Functions** - helper functions for the step
3. **Constants** - project-level constants (S3 bucket, step name)
4. **Analysis** - the core work for each step

### Variable Naming

All variables use a type prefix convention:

| Prefix | Type |
|--------|------|
| `str_` | string |
| `flt_` | float |
| `int_` | integer |
| `df_` | DataFrame |
| `list_` | list |
| `dict_` | dictionary |
| `tpl_` | tuple |

## S3 Data Convention

Data flows through S3 using a consistent path structure:

```
s3://{bucket}/00_data_collection/data.csv   # raw source data
s3://{bucket}/02_data_split/                # train/test/validation splits
s3://{bucket}/03_preprocessing/             # preprocessed features
s3://{bucket}/04_model/                     # trained model artifacts
```

The bucket name is derived from the repository directory name with underscores replaced by hyphens (e.g., `credit_risk_claude` becomes `credit-risk-claude`).

## Tech Stack

- **Language:** Python 3.11+
- **Model:** XGBoost
- **Data Storage:** AWS S3
- **Serving:** FastAPI
- **Containerization:** Docker
- **Automation:** Claude Code Skills
