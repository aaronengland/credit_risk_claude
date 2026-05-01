# Advisory Panel Third Review: Credit Risk Model Project

**Date:** 2026-05-01
**Type:** Third Review (Post-Implementation of Follow-Up Feedback)
**Attendees:** Michael Francis, John Candido, Sean Seamonds, Aaron Hunsaker, Cameron Marsden, Colby Wight

---

## Feedback Addressed (from Follow-Up Review)

The team resolved 7 of the follow-up review's findings, including the two highest-priority items.

| Item | Status | Summary |
|------|--------|---------|
| FeatureEngineer deterministic reference date | FIXED | Now uses `max(origination_date)` from training data (2024-02-02). Fully reproducible. *(all panelists)* |
| Reference date surfaced in learned parameters | FIXED | Printed in notebook alongside medians and encodings *(Aaron, Colby, Sean)* |
| Random seeds for Optuna and XGBoost | FIXED | `TPESampler(seed=42)` and `random_state=42` throughout *(all panelists)* |
| MLflow model versioning | FIXED | Logs hyperparameters, test metrics (AUC, Brier, Gini), model, and artifacts *(all panelists)* |
| Challenger trained on combined train+valid | FIXED | Eliminates 22% data asymmetry; fair comparison *(Michael, John, Cameron)* |
| Decile capture rate direction | FIXED | Decile 1 = highest risk, accumulates top-down per credit risk convention *(Cameron, Colby)* |
| API input validation | FIXED | Pydantic Field constraints for bureau_score (300-900), utilization (0-2.0), etc. *(Aaron)* |
| Readiness health check | FIXED | `/health` runs dummy prediction through model *(Aaron)* |

---

## Remaining Concerns

### Recurring Items (Open Since First or Second Review)

1. **Age proxy model still trained and scored in-sample.** The proxy AUC of 0.99 is inflated by overfitting. No cross-validation or train/test split. No narrative interpretation of which features drive it. *(Michael, John, Cameron)*

2. **Dirty state values ("00", "??", "xx") still uninvestigated.** Encoded as valid ordinal categories. No prevalence analysis or documentation. Third time flagged. *(Sean, Colby)*

3. **loan_id still median-imputed.** Median of 12,445 applied to an identifier column. Conceptually wrong. Third time flagged. *(John, Colby)*

4. **No adverse action reason codes.** Regulatory gap for credit decisioning models. *(Michael)*

### MLflow Enhancements

5. **MLflow is local-only.** Runs log to a local `mlruns/` directory. No remote tracking server, no model registry, no promotion workflow. *(John, Sean, Aaron)*

6. **MLflow run lacks key metadata.** Does not log `reference_date_`, monotone constraints, excluded columns, or the preprocessing pipeline artifact. *(Cameron, Colby, Sean)*

7. **No model version in API response.** The `/predict` response does not include the MLflow run ID or model version. Predictions are not traceable to artifacts. *(Aaron)*

### API/Deployment

8. **Health check bypasses preprocessing pipeline.** Feeds zeros directly to model, skipping `pipeline.transform()`. Does not validate the full inference path. *(Aaron)*

9. **Health check returns HTTP 200 on failure.** Should return 503 so orchestrators can route traffic away. *(Aaron)*

10. **API build artifacts still deleted after Docker build.** `app.py`, `Dockerfile`, `requirements.txt` removed in cleanup cell. *(Aaron)*

11. **Dependency versions still not pinned.** Open-ended `>=` version specifiers risk deserialization failures. *(Aaron)*

### Minor

12. **Decile 6-7 minor non-monotonicity** (8.2% vs 9.2%) should be acknowledged in README narrative. *(Colby)*

13. **`random_state` propagation is fragile.** Set inside objective function, inherited through `best_params` dict. Should be a top-level constant. *(Sean, Colby)*

14. **No `random_state` on age proxy model in disparate impact.** Inconsistent with seed discipline in model notebook. *(Cameron)*

---

## Panel Assessment

The project has matured significantly across three review cycles. The core modeling methodology is now sound: deterministic preprocessing, reproducible tuning, well-calibrated probabilities, proper out-of-time validation, and fair champion-challenger comparison. MLflow integration and API input validation close real governance gaps.

The remaining items fall into two categories:

**Items that should be resolved before production deployment:**
- Adverse action reason codes (regulatory requirement)
- MLflow remote tracking server and model registry
- Pinned dependency versions
- Persisted API build artifacts

**Items that are accepted risk for this demonstration but should be documented:**
- Age proxy in-sample evaluation (acknowledged limitation)
- Dirty state values (data quality issue to be addressed upstream)
- loan_id imputation (cosmetic, excluded from features)

---

## Recommended Next Steps

### Before Production
1. Build SHAP-based adverse action reason codes
2. Configure MLflow remote tracking server and model registry
3. Pin exact dependency versions in requirements.txt
4. Persist API build artifacts in repository
5. Add model version/run ID to API prediction response

### Documentation
6. Document the age proxy in-sample limitation and plan for cross-validation
7. Document dirty state values as known data quality issues
8. Acknowledge decile 6-7 non-monotonicity as statistical noise in README

### Refinements
9. Add `random_state=42` to age proxy model in disparate impact
10. Log `reference_date_`, monotone constraints, and preprocessing pipeline to MLflow
11. Make health check return 503 on failure and test full pipeline path
12. Define `random_state` as a top-level constant rather than inside objective function
