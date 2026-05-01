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
- `%%writefile requirements.txt` with pinned exact versions (==) for reproducible builds: fastapi, uvicorn, pandas, numpy, scikit-learn, xgboost, joblib

#### app.py
Markdown: explains the FastAPI application structure.
- `%%writefile app.py` with:
  - Structured logging (timestamp | level | message format)
  - Logs artifact loading at startup with feature count
  - Logs each prediction request with loan_id, PD, and elapsed time
  - Logs health check requests
  - Load preprocessing pipeline, XGBoost model, feature column list at startup
  - Pydantic `LoanApplication` request schema matching raw input columns (Optional for nullable fields)
  - Input validation using Pydantic `Field` and `field_validator`:
    - `loan_amount`: `Field(gt=0)`
    - `term_months`: `Field(ge=0)`
    - `bureau_score`: optional validator checking 300-900 range if provided
    - `utilization`: optional validator checking 0-2.0 range if provided
    - `stated_income`: optional validator checking gt=0 if provided
  - Compute model version hash at startup using MD5 of xgboost_model.joblib (first 8 hex chars), stored as `str_model_version`
  - Pydantic `PredictionResponse` with `flt_probability_of_default` and `str_model_version`
  - `/health` GET readiness endpoint that runs a dummy input through the full pipeline (pipeline.transform then model.predict_proba) to verify all artifacts work end-to-end; returns HTTP 503 with `JSONResponse(status_code=503, content={'status': 'unhealthy', 'error': ...})` on failure
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
Markdown: explains model artifacts are baked into Docker image and no longer needed; build files (app.py, Dockerfile, requirements.txt) persist in the repository.
- `os.remove()` only copied .joblib artifacts (preprocessing_pipeline.joblib, xgboost_model.joblib, feature_cols.joblib)
- Do NOT remove app.py, Dockerfile, or requirements.txt (these persist in the repo)
- Use try/except FileNotFoundError for safety
