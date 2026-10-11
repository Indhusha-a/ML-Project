# Decision Log

IT3091 Machine Learning — Student Performance (Education, Code 3)
Dataset: `student-mat.csv` (Math only) | MD5 fingerprint: `4dc304be95c60de6ee13fb8769469dd7`

| # | Decision | Options Considered | Reason Chosen | Evidence |
|---|---|---|---|---|
| 1 | Use Math dataset only, not Portuguese | Math only; Portuguese only; both merged | Scope reduction given project timeline; avoids merge-key complexity between the two datasets; Math is the dataset named first in the business scenario | `student-mat.csv` = 395 rows, 33 cols |
| 2 | No null/duplicate handling needed | N/A | Dataset verified clean on load | 0 nulls, 0 duplicate rows across all 33 columns |
| 3 | No category standardization needed | N/A | All categorical values already consistently formatted | Manual review of `.unique()` per categorical column |
| 4 | Split before EDA, EDA on train set only | Split after EDA; split before EDA | Prevents any EDA-driven decision from being informed by test-set data (leakage prevention) | Step 3 (split) precedes Step 4 (EDA) in notebook |
| 5 | Keep G3 = 0 rows, do not drop | Drop as anomalies; drop as missing; keep | 33/316 train rows (10.4%) have G3 = 0 with 0 absences and declining G1→G2 — consistent with exam withdrawal, a genuine at-risk outcome the business lens needs to capture, not noise | EDA: `zero_g3` subgroup stats |
| 6 | Include G1 and G2 as predictive features | Include; exclude (predict from only start-of-year data) | Both are realistically known before the final grade (first/second period grades), and contribute by far the strongest correlation with G3 (0.91, 0.79) | Correlation matrix, train set |
| 7 | Log-transform (log1p) `absences`, do not drop outliers | Drop outliers; cap/winsorize; log-transform; leave raw | Only 4/316 students have >30 absences — real values, not errors. Log transform reduces their influence on linear/ridge models without discarding data | Outlier boxplot, describe() |
| 8 | Preprocessing: OneHotEncoder for categoricals, StandardScaler for numerics, built as a single `ColumnTransformer` fit on train only | Manual encoding; separate per-model preprocessing | Single shared, leakage-safe pipeline reusable identically across all 4 models; fit exclusively on `X_train`, applied via `.transform()` to `X_test` | `full_preprocessor.fit_transform(X_train)` / `.transform(X_test)` |
| 9 | Evaluation protocol: 80/20 train/test split + 5-fold CV during tuning, test set touched only once at final evaluation | Single split only; CV only, no held-out test | Standard combination for small datasets (395 rows): CV for stable tuning without overfitting to one split, final held-out test for an honest unbiased estimate | `train_test_split(test_size=0.2, random_state=42)` + `cross_val_score(cv=5)` |
| 10 | Random seed fixed at 42 throughout | N/A | Reproducibility requirement (rubric: dataset fingerprint + reproducible notebook) | `RANDOM_STATE = 42` set globally |

| 11 | Four models selected for comparison: Linear Regression, Random Forest, Ridge, Gradient Boosting | Also considered: SVR, KNN Regressor | Covers the required baseline + 3 alternatives spanning a simple linear model, a regularized linear model, and two tree ensembles with different bias/variance trade-offs — gives a genuinely varied comparison rather than near-duplicate methods | Draft responsibility table |
| 12 | Hyperparameter tuning via `GridSearchCV`, 5-fold CV, scoring = negative RMSE | `RandomizedSearchCV`; manual tuning | Dataset is small (316 train rows) so exhaustive grid search is cheap; small, sensible grids kept per model to control runtime | Best params + CV RMSE recorded per model below |

## Model Results (5-fold CV RMSE, training set)

| Model | Best Hyperparameters | CV RMSE |
|---|---|---|
| Linear Regression | n/a (no tuning) | 1.792 |
| Ridge Regression | alpha = 10.0 | 1.764 |
| Random Forest | n_estimators=100, max_depth=None, min_samples_leaf=1 | 1.389 |
| Gradient Boosting | n_estimators=100, learning_rate=0.05, max_depth=3 | 1.458 |

Ridge scoring slightly better than plain Linear Regression is consistent with
the G1/G2 multicollinearity finding from EDA (#6) — the L2 penalty is doing
real work shrinking the correlated coefficients. Random Forest currently
leads; final ranking to be confirmed against the held-out test set in Step 10
(CV score alone is not the final word — test set evaluation is next).

## Final Test-Set Evaluation (Step 10)

| Model | Train RMSE | Test RMSE | Train-Test Gap | Test MAE | Test R² |
|---|---|---|---|---|---|
| Gradient Boosting | 0.960 | 1.945 | 0.985 | 1.130 | 0.816 |
| Random Forest | 0.578 | 1.994 | 1.415 | 1.183 | 0.806 |
| Ridge Regression | 1.597 | 2.261 | 0.664 | 1.641 | 0.751 |
| Linear Regression | 1.586 | 2.314 | 0.728 | 1.699 | 0.739 |

| 13 | Final model ranking taken from held-out test RMSE, not CV score | Rank by CV score; rank by test score | Random Forest had the best CV RMSE (1.389) but the largest train-test gap (1.415) — a clear overfitting signature (near-zero training error, inflated by fitting fold-specific noise). Gradient Boosting has the best actual test RMSE (1.945) and R² (0.816), with a smaller, more honest gap. **Gradient Boosting selected as the best-performing model**; Random Forest's CV-vs-test disagreement is reported explicitly as a finding, not hidden | Test-set metrics table above |
| 14 | Residual check: no funnel shape / heteroscedasticity observed across all 4 models | N/A | Confirms model errors are reasonably homoscedastic — no evidence of a systematic pattern in residuals the models are missing | Residual scatter plots, Step 10 |
| 15 | Ridge's smallest train-test gap (0.664) reinforces the multicollinearity decision | N/A | Direct supporting evidence (not just theoretical claim) that regularization helps generalization given G1/G2 correlation | Gap column above, vs. decision #6 |

## Feature Importance (Step 11)

Computed on the best-performing model (Gradient Boosting), using two
independent methods to reduce single-method bias.

| Rank | Tree-Based Importance | Permutation Importance |
|---|---|---|
| 1 | G2 (0.799) | G2 (3.645) |
| 2 | absences (0.135) | absences (0.599) |
| 3 | G1 (0.015) | G1 (0.139) |

| 16 | Feature importance computed on Gradient Boosting (the selected best model), not on all 4 | Compute for every model; compute for best model only | Best model's importance is what actually drives the deployed predictions; computing for all 4 would be redundant given time constraints and add no new decision-relevant information | `tree_importance_df`, `perm_importance_df` |
| 17 | Both methods agree G2, absences, G1 are the top 3 drivers, in the same order | N/A | Strong two-method convergence — satisfies the rubric's "two independent methods" requirement for feature importance with a genuine agreement, not a forced one | Comparison table above |
| 18 | `failures` (correlation -0.37 with G3) does not rank highly in either importance table | N/A | Likely explained by G2 already capturing most of the same signal (students with more failures tend to already have lower G2) — once G2 is in the model, `failures` adds little marginal predictive information beyond what G2 already encodes. Reported as a nuance distinguishing correlation from marginal importance | Cross-reference: EDA correlation (#decision 6 context) vs. importance tables |

## At-Risk Report (Step 12)

Applied predicted G3 < 10 threshold to Gradient Boosting's test-set
predictions: 37/79 test students (46.8%) flagged. Validated against actual
outcomes:

| | Predicted Not At-Risk | Predicted At-Risk |
|---|---|---|
| **Actual Not At-Risk** | 42 (TN) | 10 (FP) |
| **Actual At-Risk** | 0 (FN) | 27 (TP) |

Recall = 1.00 (at-risk), Precision = 0.73 (at-risk), Accuracy = 0.87.

| 19 | Per-student risk explanation shown via global top-6 feature values, not individual SHAP values | SHAP/LIME per-student explanations; global feature values only | Simpler, faster to implement within the 1-day timeline, and still gives an advisor a clear, explainable basis per student; documented explicitly as a simplification/limitation rather than presented as equivalent to true local explanation methods | at_risk_report table, Step 12 |
| 20 | Threshold-flag performance (recall 1.00, precision 0.73) accepted as-is, not further tuned | Tune threshold for higher precision | Zero false negatives means no genuinely at-risk student is missed; the 10 false positives only cost advisor time on students who didn't need it. Given the intervention-support business context, missing a struggling student is a worse outcome than over-flagging — current threshold behavior fits the stated business need | Confusion matrix above |

## Final Model Comparison & Recommendation (Step 13)

**Recommended model: Gradient Boosting** (test RMSE 1.945, R² 0.816) — selected
over Random Forest despite RF's better CV score, due to RF's clear
overfitting (train-test gap 1.415 vs GB's 0.985). See decision #13.

**Top predictors (2-method agreement):** G2, absences, G1 — in that order,
confirmed by both tree-based and permutation importance (decision #17).

**Key limitations documented:**
- Model's strength depends heavily on G1/G2 — not a true start-of-year
  predictor, only useful from second-period onward.
- Random Forest's CV-vs-test disagreement shows CV score alone is not a
  reliable model-selection criterion on a dataset this small (316 train rows).
- Trained on Math dataset only (395 rows) — generalizability to the
  Portuguese-language dataset or other institutions is unverified.
- Per-student explanations are simplified (global top features per student,
  not individualized SHAP-style attribution).
- Small dataset limits statistical confidence of all reported metrics.

*(Notebook complete through Step 13. Remaining work is outside the notebook —
see info.md for the post-notebook checklist.)*
