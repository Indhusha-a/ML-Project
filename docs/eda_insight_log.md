# EDA Insight Log

All analysis below was performed on the **training set only** (316 of 395
students), after the 80/20 train/test split, to avoid any test-set leakage
into modelling decisions. Full detail and reasoning for each decision is in
`decision_log.md`; this file is the condensed insight-only view the rubric
asks for separately.

## 1. Target variable (G3) distribution

- Skewness = -0.773 (moderately left-skewed).
- The skew is driven by a cluster of students scoring G3 = 0, not a smooth
  tail — the bulk of the distribution otherwise sits in a fairly normal
  range centered around 10–11.

## 2. G3 = 0 investigation

- 33 of 316 training students (10.4%) have G3 = 0.
- All 33 also have `absences = 0`, while their G1/G2 grades were already low
  and falling (G1 mean ≈ 7.6 → G2 mean ≈ 4.4).
- Interpretation: zero absences combined with a zero final grade points to
  **exam non-participation/withdrawal**, not an attendance-driven failure or
  a data error.
- Insight for the business lens: these are a real, meaningful at-risk
  population — exactly the group the secondary lens (at-risk identification)
  needs to catch, not noise to be removed.

## 3. Missing values & duplicates

- Zero nulls across all 33 columns.
- Zero duplicate rows.
- No formatting inconsistencies found in any categorical column.
- Insight: this dataset required no missing-value imputation or
  deduplication logic — a genuine finding worth stating plainly rather than
  a gap in the analysis.

## 4. Outlier check — absences

- Heavily right-skewed: median 4, mean ≈ 5.9, max 75.
- Only 4 of 316 students exceed 30 absences — a small, genuine extreme group,
  not noise spread evenly through the data.
- Insight: real outliers, not errors — addressed via transformation
  (log1p), not removal, so no legitimate data was discarded.

## 5. Correlation with G3

| Feature | Correlation with G3 |
|---|---|
| G2 | 0.908 |
| G1 | 0.792 |
| failures | -0.374 |
| Medu | 0.216 |
| studytime | 0.138 |
| Fedu | 0.135 |

- G1 and G2 dominate, as expected (same grading scale, strong temporal
  proximity to G3).
- `failures` is the strongest non-grade predictor.
- Insight: G1/G2's strength makes them central features but also raises the
  model's dependency on mid-year data being available (see limitations in
  `decision_log.md`).

## 6. School-level comparison

- GP and MS schools show a difference in mean G3, though GP has a larger
  student sample in this dataset.
- Insight: school alone is a secondary factor relative to grade-history and
  aspiration variables.

## 7. Subgroup comparisons

| Subgroup | Mean G3 |
|---|---|
| Female | 10.03 |
| Male | 10.67 |
| Rural address | 9.62 |
| Urban address | 10.52 |
| Does **not** want higher education | 6.33 |
| Wants higher education | 10.57 |

- The largest gap by far is `higher` (aspiration to pursue higher
  education): a 4.2-point mean difference.
- Insight: aspiration is a strong behavioural signal, larger than sex or
  address — flagged as a feature likely to carry real predictive weight.

## 8. Feature importance (post-modelling, cross-referenced here for completeness)

Computed on the best-performing model (Gradient Boosting) using two
independent methods — tree-based importance and permutation importance —
which agreed on the same top 3 features in the same order: **G2, absences,
G1**. Full detail in `decision_log.md`.

- Insight: `failures`, despite its meaningful raw correlation (-0.374), did
  not rank highly in either importance method — likely because G2 already
  captures most of the same signal, leaving `failures` with little
  additional (marginal) predictive value once G2 is included.
