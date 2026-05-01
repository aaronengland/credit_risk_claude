# Advisory Panel Final Review: Credit Risk Model Project

**Date:** 2026-05-01
**Type:** Fourth Review (Final Comprehensive Evaluation)
**Attendees:** Michael Francis, John Candido, Sean Seamonds, Aaron Hunsaker, Cameron Marsden, Colby Wight

---

## Panel Verdict: APPROVED FOR PRODUCTION WITH CONDITIONS

The panel unanimously recommends the framework for production deployment. The core modeling methodology, validation rigor, regulatory documentation, and automation quality meet the standard expected for a credit risk model. The remaining items are infrastructure and data quality concerns, not modeling or compliance deficiencies.

---

## Overall Assessments

**Michael Francis (SVP Risk):** "This model package would hold up well under SR 11-7 examination. The team's responsiveness across four review cycles demonstrates the kind of iterative rigor that regulators and internal audit want to see. Ship it with conditions."

**John Candido (Advisor):** "The documentation tells a clear, complete story that any stakeholder, validator, or regulator could follow without needing to open a notebook. This is what separates a good model from a good model risk management culture."

**Sean Seamonds (Dir Data Eng):** "The core data pipeline is sound. The remaining items are data engineering and infrastructure concerns, not modeling concerns, and they are all tractable."

**Aaron Hunsaker (MLOps):** "The system is production-viable for an initial deployment. The core operational risks I raised (dependency pinning, input validation, health checks, model versioning) are all resolved."

**Cameron Marsden (Sr DS):** "The modeling methodology, validation framework, and regulatory compliance posture are ready for formal model validation submission."

**Colby Wight (DS):** "This is a production-ready demonstration framework. Every high-priority implementation issue I raised in earlier reviews has been resolved."

---

## Strengths (Consensus)

### Regulatory Defensibility
- Adverse action reason codes using SHAP mapped to consumer-facing language (ECOA/CFPB compliant)
- Monotone constraints with documented rationale for every feature
- ECOA-compliant age exclusion with proxy analysis and bootstrap CIs
- Feature exclusion rationale documented for every removed column
- No scale_pos_weight; well-calibrated probabilities for pricing and capital allocation

### Methodology
- Deterministic reference date from `max(origination_date)`, surfaced in learned parameters
- Full reproducibility: `TPESampler(seed=42)`, `random_state=42` on all models
- Out-of-time validation with stable target means (17.7%-19.0%)
- Final model trained on combined train+valid with fixed `n_estimators` from early stopping
- Fair champion-challenger: both models trained on identical data (21,511 rows)
- Logistic regression challenger provides credible regulatory baseline (AUC delta 0.009)

### Evaluation
- Comprehensive metrics: AUC, Gini, KS, PR AUC, Brier, Mean/Median Pred, Target Mean
- Decile analysis with correct top-down convention (decile 1 = highest risk)
- Threshold sensitivity analysis connecting model output to business decisions
- Bootstrap CIs quantifying uncertainty in small-sample disparate impact analysis

### Infrastructure
- MLflow experiment tracking with params, metrics, constraints, and artifacts
- API input validation with Pydantic domain constraints
- Health check testing full pipeline-through-model inference path (503 on failure)
- Model version hash in every API prediction response
- Pinned dependency versions in requirements.txt
- Data contract validation at pipeline entry points

### Automation and Documentation
- 12 Claude Code skills encoding the entire model lifecycle
- CLAUDE.md codifying team standards (naming, plotting, preprocessing, domain rules)
- README is a stakeholder-ready validation document with all plots, tables, and rationale
- Four advisory panel reviews documented with full audit trail

---

## Accepted Risks (Document in Model Risk Assessment)

| Item | Risk Level | Rationale |
|------|-----------|-----------|
| Age proxy model evaluated in-sample (AUC 0.99) | Low | Documented as upper-bound estimate; plan for cross-validation in next annual review |
| Dirty state values ("00", "??", "xx") | Low | State excluded from model features; data quality issue to be addressed upstream |
| loan_id median-imputed (12,445) | Minimal | Excluded from model features; cosmetic pipeline artifact |
| Decile 6-7 minor non-monotonicity (8.2% vs 9.2%) | Minimal | Statistical noise in bins of ~380 observations |

---

## Conditions for Unconditional Production Approval

### Required Before Production
1. Stand up a remote MLflow tracking server with model registry and promotion workflow
2. Implement automated monitoring with PSI/CSI alerting and documented escalation thresholds
3. Commit API build artifacts (app.py, Dockerfile, requirements.txt) to version control

### Model Enhancement Roadmap
4. Cross-validate the age proxy model AUC (currently in-sample)
5. Add stress testing and sensitivity analysis for capital allocation scenarios
6. Investigate and document dirty state values upstream
7. Vectorize TextStandardizer for production-scale datasets

---

## Review Cycle Summary

| Review | Findings | Resolved | Key Items |
|--------|----------|----------|-----------|
| First (initial) | 20 | - | Identified FeatureEngineer bug, SVM strawman, missing decile analysis, no data contracts |
| Second (follow-up) | 8 resolved, 7 new | 8 | Fixed scale_pos_weight doc, SVM to LR, bootstrap CIs, decile analysis, data contracts |
| Third | 8 resolved, 5 new | 15 total | Fixed deterministic ref date, random seeds, MLflow, challenger data parity, API validation |
| Fourth (final) | All high-priority resolved | 18+ total | Added adverse action codes, enhanced MLflow logging, panel approves for production |

The iterative advisory panel process identified and resolved 18+ findings across four cycles, transforming the project from an initial prototype into a production-ready, regulation-compliant credit risk modeling framework.
