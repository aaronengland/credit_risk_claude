---
name: documentation
description: Updates the README.md with comprehensive documentation of every step in the credit risk modeling process, including all plots, tables, and decisions.
user_invocable: true
---

# Documentation

Update the `README.md` to comprehensively document every step of the credit risk modeling process, both pre- and post-deployment.

## Instructions

The README must be consumable by stakeholders who want to understand every decision made. It should include:

### Structure

1. **Claude logo** centered at the top from `img/Claude_AI_logo.svg.png`
2. **Title and Project Overview** - what the model does, dataset summary
3. **Table of Contents** - bulleted list with anchor links, split into Pre-Deployment and Post-Deployment sections
4. **Claude Code Skills table** - with Phase column (Setup, Pre-Deployment, Post-Deployment, Both)
5. **Pre-Deployment sections** (EDA, Data Split, Preprocessing, Model, Model Eval, Disparate Impact, API)
6. **Post-Deployment sections** (Monitoring, Champion-Challenger)
7. **Tech Stack** and **Folder Structure** at the end

### Per-Section Requirements

Each section should include:
- Description of what was done and why
- All plots from the step's `output/` folder, embedded as `![Description](path/to/image.png)`
- All CSV data from the step's `output/` folder, rendered as markdown tables with rounded values
- Key observations and interpretations of results
- Explanations of decisions

### Tables to Include

- `01_eda/output/descriptive_statistics.csv`
- `02_data_split/output/split_summary.csv`
- `04_model/output/best_params.csv`
- `05_model_eval/output/metrics_summary.csv`
- `06_disparate_impact/output/disparate_impact_metrics.csv`
- `07_monitoring/output/monitoring_summary.csv`
- `07_monitoring/output/csi_by_feature.csv`
- `08_champion_challenger/output/champion_challenger_metrics.csv`

### Monitoring Section

- Monitoring summary table
- PSI over time with explanation of training months as control
- CSI by feature table and plot with color-coding explanation
- Target drift, AUC over time, Brier over time with split boundary line descriptions
- KDE over time

### Champion-Challenger Section

- Metrics comparison table
- AUC, Gini, KS bar charts
- ROC, calibration, KDE comparison plots
- Clear recommendation based on results

### Tone

- Written for stakeholders (model validators, risk officers, regulators)
- Explain the "why" behind every decision
- Interpret results, do not just show them
- Tag sections as Pre-Deployment or Post-Deployment
- Use horizontal rules (`---`) between major sections
