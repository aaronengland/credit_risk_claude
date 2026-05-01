# Advisory Panel Review: Credit Risk Model Project

**Date:** 2026-05-01
**Type:** Full Project Review
**Attendees:** Michael Francis, John Candido, Sean Seamonds, Aaron Hunsaker, Cameron Marsden, Colby Wight

---

## What the Panel Liked

### Methodology and Validation
- **Out-of-time split** (70/15/15 by origination date) is the correct methodology for credit risk. Target mean stability across splits (17.7%-19.0%) confirms minimal temporal drift. *(Cameron, Michael)*
- **Monotone constraints with documented rationale** for every constrained feature. SHAP PDP plots visually confirm constraints are respected, closing the loop for validators. *(Michael, Cameron, Colby)*
- **Final model trained on combined train+valid with fixed `n_estimators`** extracted from early stopping. Maximizes data utilization while preserving the regularization signal. *(Cameron, Colby)*
- **Well-calibrated probabilities without post-hoc adjustment.** Mean predicted PD closely tracks actual default rate (18.7% vs 18.3% on test). Critical for pricing and capital allocation. *(Michael, Cameron, John, Colby)*

### Leakage Prevention and Feature Governance
- **Thorough leakage prevention.** Exclusion of `charged_off_amount`, `paid_interest_amount`, `apr`, and `flt_payment_to_income` with clear rationale. Keeping them through preprocessing and excluding only at the model step is the right separation of concerns. *(Michael, Cameron, Colby)*
- **Pipeline fit-on-train-only discipline** is codified in both the skill definitions and CLAUDE.md, not just the notebook code. *(Cameron, Colby, Sean)*

### Transparency and Auditability
- **Learned parameters surfaced in the notebook.** Median imputation values and ordinal encoding mappings are displayed directly, so a validator can verify what was computed without deserializing the pipeline. *(Sean, Michael)*
- **Evaluation metrics table explains each metric and why it matters for credit risk.** Documentation written for stakeholders, not just data scientists. *(John)*
- **TextStandardizer prevents silent category proliferation** by collapsing casing variants before encoding. *(Cameron, Sean, Colby)*

### Monitoring and Deployment
- **Comprehensive monitoring suite.** PSI, CSI by feature, target drift, AUC over time, Brier over time, and KDE distributions cover the standard post-deployment concerns. *(Michael, Aaron, Cameron)*
- **Clean artifact chain from training to serving.** The Docker image is self-contained with the full preprocessing pipeline applied at inference time, eliminating training-serving skew. *(Aaron)*
- **Structured API logging** with loan_id, predicted PD, and elapsed time per request. *(Aaron)*

### Automation and Reproducibility
- **Skills-based automation** encodes the entire model lifecycle into repeatable, invocable commands. Any analyst can reproduce the framework without tribal knowledge. *(Michael, John, Colby)*
- **Proxy analysis in disparate impact** (training a separate model to predict age group membership, then comparing SHAP importance) is a rigorous approach to identifying indirect discrimination. *(Michael, John)*

---

## Opportunities for Improvement

### High Priority

1. **`FeatureEngineer` uses `pd.Timestamp.now()` for age calculation.** The `int_age` feature will produce different values every time the pipeline runs. A model trained in 2024 and served in 2026 will compute different ages for the same DOB. The reference date should be fixed at training time and stored as a fitted attribute. *(Sean, Aaron, Cameron, Colby)*

2. **Age proxy model AUC of 0.98 needs deeper investigation.** The model's features can predict age group membership with near-perfect accuracy, meaning the feature space is nearly a perfect proxy for the protected class. The small 60+ sample (n=107) limits statistical power for the KS test (p=0.34). Bootstrap confidence intervals on the AUC gap are needed to determine whether "no disparate impact" is a confident conclusion or just inconclusive. *(Michael, John, Cameron)*

3. **The `scale_pos_weight` markdown in the model notebook contradicts the code.** The Bayesian tuning markdown says `scale_pos_weight` is set to the class ratio, but the code intentionally does not use it. This documentation-code mismatch would be caught immediately in model validation and erode trust. *(John, Cameron, Colby)*

4. **Champion-challenger uses an untuned SVM, which is a strawman.** A credible challenger should be a model that could plausibly replace the champion: a tuned logistic regression (the regulatory baseline), a LightGBM, or XGBoost with a different feature set. The current comparison validates the framework but does not stress-test the champion. *(Michael, John, Cameron)*

### Medium Priority

5. **No data quality validation at any stage.** Zero assertions on incoming data: no schema checks, no expected-column validation, no duplicate detection. An upstream schema change will cause silent failures. *(Sean)*

6. **`open_trades` has no monotone constraint (set to 0) without documented rationale.** More open trades generally indicates higher credit exposure. If directionality is genuinely ambiguous, that rationale should be explicitly stated. *(Michael, John, Cameron, Colby)*

7. **No decile/score-band analysis in model evaluation.** Credit risk validators universally expect a table showing predicted PD ranges, counts, actual default rates, and capture rates by decile. *(Cameron, Colby)*

8. **Confusion matrix uses a 0.5 threshold, which is misleading for a ~19% base rate.** A threshold analysis showing precision, recall, and approval rate at multiple cutoffs would better inform business decisions. *(John, Cameron)*

9. **No adverse action reason code analysis.** Regulators require specific, accurate reasons for credit denials. SHAP-based reason codes mapped to consumer-facing language are a standard requirement for models used in credit decisioning. *(Michael)*

10. **Dirty state values ("00", "??", "xx") are never investigated.** These likely represent data quality issues in the raw data and should be flagged or quarantined. *(Sean, Colby)*

11. **`loan_id` is median-imputed as a numeric feature.** While excluded from model features downstream, imputing an identifier is conceptually wrong and could mask data quality issues. *(John, Colby)*

### Lower Priority

12. **No input validation on the `/predict` API endpoint.** Malformed requests produce unhandled 500 errors with Python tracebacks. Domain-reasonable range checks and structured error responses are needed. *(Aaron)*

13. **No model versioning or artifact registry.** Artifacts are silently overwritten on retraining. In a regulated environment, every prediction must be traceable to the exact model version. *(Aaron)*

14. **The bucket name derivation (`os.getcwd().split('/')[4]`) is brittle.** It assumes a specific directory depth and will break in different environments. *(Sean)*

15. **`TextStandardizer` uses nested Python loops.** Vectorized approaches would be significantly faster on larger datasets. *(Sean, Colby)*

16. **No CI/CD pipeline or automated testing for the API.** Docker images are built manually from notebook cells. *(Aaron)*

17. **The `/health` endpoint does not validate that artifacts are loaded.** A readiness probe should run a dummy prediction through the full pipeline. *(Aaron)*

18. **State is excluded from the model without documented rationale.** Geographic features can be important risk predictors; the exclusion reason should be stated explicitly. *(Michael, Colby)*

19. **Monitoring is retrospective (notebook-based), not operational.** Production monitoring needs automated scheduling, alerting thresholds, and integration with an observability platform. *(Aaron)*

20. **No sensitivity analysis or stress testing.** No analysis of how the model behaves under stressed economic conditions. *(Michael)*

---

## Recommended Next Steps

### Immediate Fixes
1. Fix the `FeatureEngineer` age calculation to use a fixed reference date stored during `fit()`
2. Correct the stale `scale_pos_weight` markdown in the model notebook
3. Document rationale for `open_trades` having no monotone constraint and for `state` exclusion

### Short-Term Enhancements
4. Add bootstrap confidence intervals to the disparate impact analysis for the 60+ group
5. Replace the SVM challenger with a tuned logistic regression
6. Add a decile/score-band analysis to model evaluation
7. Add a threshold sensitivity analysis to the confusion matrix section
8. Add data contract validation (schema checks) at the entry point of each notebook
9. Investigate dirty state values ("00", "??", "xx") in EDA

### Medium-Term Initiatives
10. Build an adverse action reason code module using SHAP
11. Expand disparate impact analysis to include race/ethnicity (via BISG proxy) and sex
12. Add a model risk tiering section documenting intended use, risk tier, and governance proportionality
13. Implement model versioning and artifact registry
14. Add input validation and readiness health check to the API
15. Add conceptual soundness documentation (theoretical basis, functional form justification, known limitations)

### Long-Term Infrastructure
16. Stand up operational monitoring with automated PSI/CSI calculations and alerting
17. Implement CI/CD pipeline with smoke tests for the Docker image
18. Add stress testing and sensitivity analysis for capital allocation scenarios
19. Pin exact dependency versions in Docker to prevent serialization compatibility issues
