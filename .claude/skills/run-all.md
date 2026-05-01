---
name: run-all
description: Executes all modeling skills sequentially to build the full credit risk model pipeline from EDA through deployment and documentation.
user_invocable: true
---

# Run All

Execute all modeling skills sequentially to build the complete credit risk model pipeline.

## Instructions

Run each of the following skills in order. Each skill populates the notebook for its respective step. After all notebooks are populated, the user should run them on their compute environment (e.g., SageMaker).

### Execution Order

1. `/create-folder-structure` - scaffold directories and template notebooks
2. `/eda` - populate EDA notebook
3. `/data-split` - populate data split notebook
4. `/preprocessing` - populate preprocessing notebook
5. `/model` - populate model training notebook
6. `/model-eval` - populate model evaluation notebook
7. `/disparate-impact` - populate disparate impact notebook
8. `/deployment` - populate API notebook
9. `/documentation` - generate comprehensive README

### Notes

- Each skill is independent and can be re-run individually if adjustments are needed
- The user must run each notebook on their compute environment between steps if downstream notebooks depend on upstream outputs (e.g., model training depends on preprocessed data being saved to S3)
- Constants like `str_target`, `str_date_col`, and `list_exclude_cols` may need to be adjusted per project
