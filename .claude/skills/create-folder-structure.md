---
name: create-folder-structure
description: Creates the folder structure for a credit risk modeling project with numbered step directories and template notebooks.
user_invocable: true
---

# Create Folder Structure

Create the complete folder structure for the credit risk modeling project. This scaffolds all step directories with template notebooks following the project's standard layout.

## Instructions

1. Create the following directories in the project root:
   - `01_eda/`
   - `02_data_split/`
   - `03_preprocessing/`
   - `04_model/`
   - `05_model_eval/`
   - `06_disparate_impact/`
   - `api/`

2. In each directory (01 through 06), create a `notebook.ipynb` file following the template structure from the root `notebook.ipynb`:
   - Cell 1 (code): imports - only `import os` as the starting point
   - Cell 2 (markdown): `#### Functions`
   - Cell 3 (code): empty (placeholder for function definitions)
   - Cell 4 (markdown): `#### Constants`
   - Cell 5 (code): constants - must include `str_bucket` and `str_step` derived from `os.getcwd()`
   - Cell 6 (markdown): `#### Analysis`
   - Cell 7 (code): empty (placeholder for analysis)

3. For `api/`, do NOT create a notebook. Instead create placeholder files:
   - `app.py` (empty, for FastAPI application)
   - `Dockerfile` (empty, for Docker image)
   - `requirements.txt` (empty, for dependencies)

4. The constants cell must always contain:
```python
# bucket
str_bucket = os.getcwd().split('/')[4].replace('_', '-')
print(f'Bucket: {str_bucket}')

# step
str_step = os.getcwd().split('/')[-1]
print(f'Step: {str_step}')
```

5. All variable names must use type prefixes (str_, flt_, int_, df_, list_, dict_, tpl_, etc.).

6. After creating the folder structure, update the README.md in the project root to document the structure.
