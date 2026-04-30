---
name: deployment
description: Populates the API notebook in api/ with FastAPI application, Dockerfile, and Docker build using %%writefile for all artifacts.
user_invocable: true
---

# Deployment

Populate the `api/notebook.ipynb` with code to create all API artifacts via `%%writefile`, build a Docker image, and clean up.

## Instructions

Edit the existing `api/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. No Functions section needed for this notebook.

### Imports Cell

```python
import os
import shutil
```

### Constants Cell

```python
# bucket, step
# artifact source paths:
#   ../03_preprocessing/output/preprocessing_pipeline.joblib
#   ../04_model/output/xgboost_model.joblib
#   ../04_model/output/feature_cols.joblib
# docker image name: str_bucket
# output directory
```

### Analysis Sections

#### Copy Model Artifacts
Markdown: explains copying artifacts into build context.
- `shutil.copy2` for preprocessing_pipeline.joblib, xgboost_model.joblib, feature_cols.joblib

#### requirements.txt
- `%%writefile requirements.txt` with fastapi, uvicorn, pandas, numpy, scikit-learn, xgboost, joblib

#### app.py
Markdown: explains the FastAPI application structure.
- `%%writefile app.py` with:
  - Structured logging (timestamp | level | message format)
  - Logs artifact loading at startup with feature count
  - Logs each prediction request with loan_id, PD, and elapsed time
  - Logs health check requests
  - Load preprocessing pipeline, XGBoost model, feature column list at startup
  - Pydantic `LoanApplication` request schema matching raw input columns (Optional for nullable fields)
  - Pydantic `PredictionResponse` with `flt_probability_of_default`
  - `/health` GET endpoint
  - `/predict` POST endpoint: converts to DataFrame, runs pipeline.transform, selects model features, returns predicted PD

#### Dockerfile
- `%%writefile Dockerfile`
- Base: `python:3.11-slim`
- WORKDIR /app
- Install requirements
- Copy artifacts (pipeline, model, feature_cols) and app.py
- Expose port 8080
- CMD: uvicorn app:app --host 0.0.0.0 --port 8080

#### Build Docker Image
- `!docker build -t {str_image_name} .`

#### Clean Up
Markdown: explains artifacts are baked into Docker image and no longer needed.
- `os.remove()` all artifacts: .joblib files, app.py, Dockerfile, requirements.txt
- Use try/except FileNotFoundError for safety
