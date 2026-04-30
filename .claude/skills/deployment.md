---
name: deployment
description: Populates the API notebook in api/ with FastAPI application, Dockerfile, and Docker build using %%writefile for all artifacts.
user_invocable: true
---

# Deployment

Populate the `api/notebook.ipynb` with code to create all API artifacts via `%%writefile`, build a Docker image, and clean up.

## Instructions

Edit the existing `api/notebook.ipynb`. All variable names must use type prefixes. All markdown headers use `####`. Each function in its own code cell.

### Flow

1. Copy model artifacts (preprocessing pipeline, model, feature cols) into api/ for Docker build context
2. `%%writefile requirements.txt` - Python dependencies
3. `%%writefile app.py` - FastAPI application
4. `%%writefile Dockerfile` - Docker image definition
5. Build Docker image
6. Clean up copied artifacts with `os.remove()`

### app.py Structure

- Load preprocessing pipeline, XGBoost model, and feature column list at startup
- Define Pydantic request schema (`LoanApplication`) matching the raw input columns
- Define response schema (`PredictionResponse`) with `flt_probability_of_default`
- `/health` GET endpoint for health checks
- `/predict` POST endpoint that:
  - Converts input to DataFrame
  - Runs through preprocessing pipeline
  - Selects model features
  - Returns predicted probability of default

### Dockerfile

- Base: `python:3.11-slim`
- Install requirements
- Copy artifacts and app
- Expose port 8080
- Run uvicorn

### Clean Up

After Docker build, remove the copied `.joblib` artifacts from the api/ directory using `os.remove()`. They are baked into the Docker image and no longer needed in the build context.
