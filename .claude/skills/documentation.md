---
name: documentation
description: Updates the README.md with comprehensive documentation of every step in the credit risk modeling process, including all plots, tables, and decisions.
user_invocable: true
---

# Documentation

Update the `README.md` to comprehensively document every step of the credit risk modeling process.

## Instructions

The README must be consumable by stakeholders who want to understand every decision made. It should include:

### Structure

1. **Title and Project Overview** - what the model does, dataset summary
2. **Table of Contents** - bulleted list with anchor links to each section
3. **One section per step** (EDA, Data Split, Preprocessing, Model, Model Eval, Disparate Impact, API)
4. **Tech Stack** and **Folder Structure** at the end

### Per-Section Requirements

Each section should include:
- Description of what was done and why
- All plots from the step's `output/` folder, embedded as `![Description](path/to/image.png)`
- All CSV data from the step's `output/` folder, rendered as markdown tables
- Key observations and interpretations of results
- Explanations of decisions (e.g., why median imputation, why monotone constraints)

### Tables

Read CSV files from each step's `output/` folder and render them as markdown tables with rounded values for readability. Include:
- `01_eda/output/descriptive_statistics.csv`
- `02_data_split/output/split_summary.csv`
- `04_model/output/best_params.csv`
- `05_model_eval/output/metrics_summary.csv`
- `06_disparate_impact/output/disparate_impact_metrics.csv`

### Tone

- Written for stakeholders (model validators, risk officers, regulators)
- Explain the "why" behind every decision
- Interpret results, do not just show them
- Use horizontal rules (`---`) between major sections
