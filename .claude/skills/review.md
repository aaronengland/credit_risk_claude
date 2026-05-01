---
name: review
description: Reviews all notebooks after they have been run, adjusts skills if necessary, updates the README.md with the latest outputs, and convenes the advisory panel for a documented review meeting.
user_invocable: true
---

# Review

Review all notebooks after they have been run, update skills and documentation, and convene the advisory panel.

## Steps

### 1. Read Notebook Outputs

Read all notebooks to extract the latest outputs:
- `03_preprocessing/notebook.ipynb` - learned parameters (medians, encodings, reference date)
- `04_model/notebook.ipynb` - best hyperparameters, best iteration, MLflow run ID
- `05_model_eval/notebook.ipynb` - metrics summary, decile analysis, threshold analysis, reason codes
- `06_disparate_impact/notebook.ipynb` - disparate impact metrics, bootstrap CIs, proxy model AUC
- `07_monitoring/notebook.ipynb` - PSI, CSI, monitoring summary
- `08_champion_challenger/notebook.ipynb` - champion vs challenger metrics, recommendation

Also read the CSV outputs from each step's `output/` folder for exact values.

Query the MLflow database at `04_model/mlflow.db` for the latest run's logged parameters, metrics, and run ID:
```sql
SELECT key, value FROM params WHERE run_uuid = (SELECT run_uuid FROM runs WHERE status='FINISHED' ORDER BY start_time DESC LIMIT 1) ORDER BY key;
SELECT key, value FROM latest_metrics WHERE run_uuid = (SELECT run_uuid FROM runs WHERE status='FINISHED' ORDER BY start_time DESC LIMIT 1) ORDER BY key;
SELECT run_uuid, name, status FROM runs WHERE status='FINISHED' ORDER BY start_time DESC LIMIT 1;
```

### 2. Check for Errors

Scan all notebooks for error outputs. If any cell has an error, diagnose and fix it before proceeding.

### 3. Adjust Skills If Necessary

Compare the actual notebook behavior against the skill definitions in `.claude/skills/`. If any skill does not match what the notebook actually does, update the skill. Common mismatches include:
- Function signatures that changed
- New analysis sections that were added
- Changed data flow (e.g., combined train+valid)

### 4. Update README.md

Update every section of the README.md with the latest values from the notebook outputs:
- Best hyperparameters and n_estimators
- Metrics summary table (AUC, Gini, KS, PR AUC, Brier, Median/Mean Pred, Target Mean)
- Decile analysis table
- Threshold analysis table
- Disparate impact metrics and bootstrap CIs
- Monitoring summary (PSI, baseline/production median PD)
- Champion-challenger metrics and recommendation
- MLflow run ID, logged parameters, and logged metrics
- Any new plots (embed as `![Description](path/to/image.png)`)

### 5. Convene Advisory Panel

Launch all six advisory panel members in parallel to review the current state of the project. Each member reads their profile from `advisory-panel/` and all previous meeting notes from `advisory-panel/meetings/`.

Panel members:
- Michael Francis (SVP Risk) - regulatory compliance, model governance, business impact
- John Candido (Advisor) - strategic alignment, stakeholder communication
- Sean Seamonds (Dir Data Eng) - data pipelines, data quality, platform architecture
- Aaron Hunsaker (MLOps) - deployment, monitoring, infrastructure
- Cameron Marsden (Sr DS) - validation methodology, statistical rigor
- Colby Wight (DS) - feature engineering, data quality, implementation details

Each member provides:
- What was addressed since the last review
- Remaining concerns
- New observations
- Recommendations

### 6. Document Meeting

Compile all panel feedback into a meeting document saved to `advisory-panel/meetings/` with the filename format `YYYY-MM-DD-{description}.md`. NEVER overwrite previous meeting documents.

The meeting document should include:
- Date, type, and attendees
- Feedback addressed (with status table)
- Remaining concerns (prioritized)
- New observations
- Panel assessment
- Recommended next steps

### 7. Update README with Review Summary

Add the review summary to the README.md before the Tech Stack section. Include:
- Link to the meeting document
- What was fixed (status table)
- Key remaining items
- Panel recommendation

Add the review to the Table of Contents.

### 8. Commit and Push

Commit all changes and push to the main branch.
