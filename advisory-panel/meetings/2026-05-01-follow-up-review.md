# Advisory Panel Follow-Up Review: Credit Risk Model Project

**Date:** 2026-05-01
**Type:** Follow-Up Review (Post-Feedback Implementation)
**Attendees:** Michael Francis, John Candido, Sean Seamonds, Aaron Hunsaker, Cameron Marsden, Colby Wight

---

## Feedback Addressed (from First Review)

The team addressed 8 of the original 20 findings. The panel acknowledges the following improvements:

| Item | Status | Summary |
|------|--------|---------|
| scale_pos_weight doc mismatch | FIXED | Notebook markdown now correctly states no weights are used *(all panelists)* |
| SVM strawman challenger | FIXED | Replaced with tuned logistic regression via GridSearchCV; AUC delta narrowed to 0.008 *(all panelists)* |
| Decile/score-band analysis | FIXED | Monotonically increasing default rates from 1.6% to 57.4% across deciles *(Cameron, Colby, John)* |
| Threshold sensitivity analysis | FIXED | Precision/recall/F1/approval rate at six thresholds *(Cameron, Colby, John)* |
| Bootstrap CIs on disparate impact | FIXED | 95% CI for AUC difference includes zero (-0.06, 0.22); properly inconclusive *(Michael, John, Cameron)* |
| open_trades constraint rationale | FIXED | Documented: "could go either way; credit experience vs over-extension" *(all panelists)* |
| state exclusion rationale | PARTIALLY FIXED | Documented in notebook but not propagated to README feature table *(Michael, John, Colby)* |
| Data contract validation | FIXED | Schema checks added to preprocessing and model notebooks *(Sean, Colby)* |

---

## Remaining Concerns

### Still Outstanding from First Review

1. **FeatureEngineer reference date (High Priority).** The `fit()` method stores `pd.Timestamp.now()`, which is captured at training time and serialized with the pipeline. This prevents train-serve skew for the deployed model, but re-fitting on a different day produces different age values for the same data. The fix should use a deterministic date tied to the data (e.g., max origination_date from training). The docstring claims the issue is resolved, but the underlying implementation does not deliver full reproducibility. *(Michael, John, Cameron, Colby; Sean and Aaron consider it partially fixed for the deployment path)*

2. **Age proxy model AUC of 0.98 still not deeply investigated.** Bootstrap CIs addressed the AUC gap between age groups, but the proxy model itself is trained and scored on the same data (in-sample), inflating the 0.98 figure. No narrative interpretation of which features drive the proxy AUC exists. *(Michael, John, Cameron)*

3. **Dirty state values ("00", "??", "xx") not investigated.** Still encoded as valid categories. No analysis of prevalence or correlation with target. *(Sean, Colby)*

4. **loan_id still median-imputed.** Median of 12,445 applied to an identifier column. Conceptually wrong. *(John, Colby)*

5. **No adverse action reason codes.** Regulatory gap for credit decisioning models. *(Michael)*

6. **No API input validation.** No domain-reasonable range checks on the `/predict` endpoint. *(Aaron)*

7. **No model versioning or artifact registry.** Artifacts silently overwritten on retraining. *(Aaron)*

### New Findings

8. **Champion-challenger data asymmetry.** The champion was trained on combined train+valid (21,511 rows) but the challenger logistic regression was trained on train only (17,715 rows). This gives the champion a 22% data advantage, undermining the comparison's credibility. *(Michael, John, Cameron)*

9. **Decile cumulative capture rate computed in wrong direction.** Decile 1 (lowest risk) shows 100% capture, which is trivially true. Standard convention is to accumulate from highest-risk decile downward. The README narrative is correct but the table is inverted. *(Cameron, Colby)*

10. **README says "50 trials" but notebook runs 100.** Documentation-code mismatch of the same category as the fixed scale_pos_weight issue. *(John, Colby)*

11. **No random seeds for Optuna or XGBoost.** Re-running the notebook produces different hyperparameters and models. *(Colby)*

12. **FeatureEngineer reference date not surfaced in learned parameters.** Validators cannot see what date was used for age calculation without deserializing the pipeline. *(Aaron)*

13. **API build artifacts deleted after Docker build.** `app.py`, `Dockerfile`, and `requirements.txt` are removed in the cleanup cell, making rebuilds impossible without re-running the notebook. *(Aaron)*

14. **Dependency versions not pinned in requirements.txt.** scikit-learn version drift can break pipeline deserialization. *(Aaron)*

15. **Monitoring is still notebook-based.** No automated scheduling, alerting, or observability integration. *(Aaron)*

---

## Recommended Next Steps

### Immediate
1. Fix FeatureEngineer to use a deterministic reference date (e.g., max origination_date from training data)
2. Update README: change "50 trials" to "100 trials", add state exclusion rationale to feature table
3. Fix decile capture rate to accumulate from highest-risk decile downward
4. Train challenger logistic regression on combined train+valid for fair comparison

### Short-Term
5. Surface FeatureEngineer reference date in learned parameters section
6. Add narrative interpretation of age proxy model (which features drive 0.98 AUC, what it means)
7. Investigate dirty state values and document findings
8. Add random seeds to Optuna sampler and XGBoost for reproducibility
9. Strengthen model notebook data contract to validate feature columns exist

### Medium-Term
10. Build SHAP-based adverse action reason codes
11. Add API input validation with domain-reasonable range checks
12. Implement model versioning (hash artifacts, include version in API responses)
13. Pin exact dependency versions in requirements.txt
14. Persist API build artifacts (app.py, Dockerfile) in repository

### Long-Term
15. Stand up operational monitoring with automated scheduling and alerting
16. Add CI/CD pipeline with integration tests for the API
