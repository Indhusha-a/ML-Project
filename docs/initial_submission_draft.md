# IT3091 Machine Learning — Initial Submission

**Group Number:** [Your Group Number]
**Track:** Guided Data Track
**Dataset Code:** 3 — Education (Student Performance)
**Dataset Source:** [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/320/student-performance)

---

## 1. Business Scenario

> *Business scenario, as given in the assignment descriptor:* **An academic institution wants to identify patterns that can support student performance and early intervention.**

The institution's academic advisors and teaching staff need to move beyond reactive end-of-year assessments. By analysing data collected at the start of the academic year — including demographic, social, and school-related attributes — they want to identify students who are likely to struggle with their final grade before it is too late to act. The practical goal is to direct limited intervention resources (tutoring, counselling, additional academic support) toward the students who need them most, rather than spreading them uniformly across the student population.

---

## 2. Lenses, With Reasons

### 2.1 Primary Lens: Performance Prediction

The final period grade (G3) is the direct, measurable outcome that the institution cares about. A continuous regression on G3 gives advisors an actual predicted score for each student, which is more actionable than a simple pass/fail label: it reveals the *degree* of expected struggle, not just whether a student is at risk. A regression framing is therefore the most direct translation of the business question — "how well will this student do?" — into a computable target.

> **Why not pure classification?** A binary pass/fail classifier discards the difference between a student predicted to score 9 and one predicted to score 2. Both fail, but the severity of required intervention is very different. Regression preserves this information and allows advisors to rank and triage students.

### 2.2 Secondary Lens: At-Risk Student Identification

This lens directly strengthens the primary lens rather than adding disconnected work. Once a predicted grade is produced, the question becomes: **who is at risk and what defines their risk profile?** By applying a threshold (e.g., predicted G3 < 10) to the regression output, we can generate an ordered list of at-risk students. Feature importance from the regression model then explains *why* a student is predicted to struggle — whether it is past failures, low study time, high absenteeism, or a lack of educational support — giving advisors evidence-based guidance on the type of intervention needed, not just a list of names.

### 2.3 Why No Third Lens Was Added

A third lens, such as pure student segmentation via clustering, was deliberately excluded. Unsupervised grouping would not connect back to the institution's core need — predicting and acting on academic risk. Adding it would dilute focus and create disconnected work. The assignment descriptor is explicit that marks are awarded for defensible reasoning, not for attempting many analyses. The group chose to go deep on two strongly connected lenses rather than shallow on three.

---

## 3. Proposed Task and Output

### 3.1 Unit of Analysis

One individual student record (one row in the dataset), representing a single student enrolled in either Mathematics or Portuguese Language at one of two secondary schools in Portugal.

### 3.2 Dataset Overview

| Property | Math Dataset (`student-mat.csv`) | Portuguese Dataset (`student-por.csv`) |
|---|---|---|
| Records | 395 students | 649 students |
| Features | 33 attributes | 33 attributes |
| Target variable | G3 (0–20 scale) | G3 (0–20 scale) |
| Mean final grade (G3) | 10.42 | 11.91 |
| Std deviation (G3) | 4.58 | 3.23 |
| Pass rate (G3 ≥ 10) | 67.1% | — |
| Students wanting higher education | 94.9% | — |
| Students with no prior failures | 79.0% | — |

> **Note:** 382 students appear in both datasets and can be identified using the merge key provided in `student-merge.R`.

### 3.3 Exact Task

**Primary task:** Supervised regression. The target variable is G3 (final period grade), a numeric value between 0 and 20. The model learns from a student's demographic, social, and academic attributes recorded *before* the final grade is known, in order to predict G3 for students not seen during training.

**Secondary task:** At-risk identification. Using the regression output, students with a predicted G3 below a defined threshold (10 on a 20-point scale) are flagged as at-risk. Feature importance analysis explains the dominant drivers of low predicted grades, enabling targeted intervention.

### 3.4 Exact Output

1. A **predicted final grade (G3)** on a 0–20 scale for each student.
2. A **ranked feature importance table** — computed by two independent methods (tree-based importance and permutation importance) — showing which student attributes most strongly predict academic performance.
3. An **at-risk student report** listing flagged students (predicted G3 < 10) ordered by predicted grade, together with the top contributing risk factors for each.

### 3.5 Delivery Form

The output is delivered as a notebook-based decision support tool for academic advisors. It is deliberately **not** framed as an autonomous grade-prediction engine. Every predicted grade is paired with its top feature importance drivers so that a human advisor can understand and act on the prediction, rather than treating the model as an unexplainable black box that replaces professional academic judgement.

---

## 4. Proposed Workflow

The project follows a single, coherent pipeline from the business problem through to a final recommendation. Each stage is deliberately linked to the ones before and after it, so that early decisions (e.g., how G3 = 0 "dropout" rows are handled) are carried through consistently to the final evaluation.

| Stage | What Happens |
|---|---|
| **1. Business problem** | Academic institution needs a defensible, data-driven approach to identifying at-risk students before the end of term. |
| **2. Lens and task formulation** | Primary lens: performance prediction (G3 regression). Secondary lens: at-risk student identification via predicted grade threshold. |
| **3. Data collection and understanding** | Student Performance dataset from UCI: 395 Math records and 649 Portuguese records; 33 attributes each including demographic, social, school, and grade features. |
| **4. Exploratory data analysis** | Grade distribution analysis (G3 skewness, zero-grade investigation), missing value audit, outlier detection (absences range: 0–75), correlation analysis (G3 vs. studytime, failures, parental education), subgroup comparisons (by school, sex, address, higher education aspiration), and feature collinearity check (G1/G2 vs. G3). |
| **5. Preprocessing and feature engineering** | Handling of G3 = 0 rows (dropout vs. genuine zero), binary encoding of categorical features, ordinal encoding of ordered numerics (e.g., Medu, studytime), outlier treatment for absences, scaling for linear models, and a leakage-safe pipeline (G1 and G2 are first-period and second-period grades that precede G3 — their inclusion must be justified). |
| **6. Train and test split with cross-validation** | An 80/20 train-test split, with 5-fold cross-validation used to check stability and reduce reliance on a single random split. |
| **7. Model comparison** | Four models are built and compared: Linear Regression (baseline), Random Forest, Ridge Regression, and Gradient Boosting (XGBoost or similar). Each is individually tuned using cross-validation-based hyperparameter search. |
| **8. Evaluation** | Root Mean Squared Error (RMSE), Mean Absolute Error (MAE), R², residual analysis, and an overfitting check comparing training and test error across all four models. |
| **9. Feature importance** | Tree-based importance and permutation importance are compared to confirm that the same features are identified as important by two independent methods, reducing single-method bias. |
| **10. Recommendation and limitations** | A final recommendation for the academic advising team, together with an honest statement of the model's limitations (e.g., sensitivity to G1/G2 inclusion, generalisability beyond Portuguese secondary schools). |

A visual workflow diagram, together with the fully executed notebook, will be included in the Final Submission.

---

## 5. Core Responsibility of Each Member

Every member contributes to every stage of the pipeline, and every member individually builds, tunes, and evaluates one complete model from start to finish. The table below states each member's **core, checkable area of responsibility**. A full stage-by-stage decision log, and a rubric-mapped contribution matrix showing exactly which notebook code cell each member owns, will be included in the Final Submission.

| Member | Core Responsibility Area | Specific Contribution |
|---|---|---|
| **M1** Mihir P.V.I IT24101254 | Business framing, model evaluation strategy, and final recommendation | Authors the stakeholder framing, lens justification, and rationale for the education scenario. Designs and owns the **evaluation framework** used across all four models — defining the metrics (RMSE, MAE, R²), the train/test split strategy, the 5-fold cross-validation logic, and the overfitting diagnostic. Carries out the grade distribution analysis (G3 skewness and zero-grade investigation) and school-level performance comparison during EDA. Conducts the residual analysis for all four models and compiles the **final four-model comparison table** from all members' results. Builds, tunes, evaluates, and interprets the **Linear Regression** baseline model in full — including interpreting coefficient signs against domain knowledge to verify that the model makes educational sense. Leads the **recommendation and limitations write-up**, translating model findings into actionable guidance for academic advisors, and documents all key decisions in the master decision log. |
| **M2** [Name] [IT24104122] | Data quality and Random Forest | Leads the missing value audit and documents the nature of each affected column. Identifies potentially anomalous records (e.g., students with high absences and G3 = 0). Carries out the parental education level vs. grade analysis and the study time vs. grade analysis. Implements missing value handling code shared across all models. Builds, tunes, evaluates, and interprets the **Random Forest** model in full, including its feature importance chart. Compiles the data dictionary and README. |
| **M3** [Name] [IT24104098] | Feature engineering and Ridge Regression | Carries out the full numeric feature correlation audit and the multicollinearity check (particularly for G1, G2, G3). Implements the feature engineering and ordinal encoding (Medu, Fedu, studytime, traveltime) used across all four models. Builds, tunes, evaluates, and interprets the **Ridge Regression** model in full, including tying its result back to the multicollinearity finding. Independently re-runs the full notebook before submission to confirm it is error-free. |
| **M4** [Name] [IT24103766] | Pipeline architecture, Gradient Boosting, and reproducibility | Carries out the subgroup performance analysis (by sex, address, romantic relationship, internet access). Implements the leakage-safe preprocessing pipeline shared by all four models — including the justified decision on whether to include G1 and G2 as features. Builds, tunes, evaluates, and interprets the **Gradient Boosting (XGBoost)** model in full, including its permutation importance chart. Records the dataset fingerprint, fixes the random seed used throughout, and prepares the reproducibility documentation. |

---

## Note on the Use of AI Assistance

The structure of this submission and the accompanying project pipeline were developed with assistance from an AI assistant (Antigravity / Google DeepMind). All group members reviewed the plan described in this document, discussed the lens and workflow choices as a team, and take ownership of the final decisions. This note will be expanded into a full AI use transparency declaration in the Final Submission.

---

*IT3091 Machine Learning — Initial Submission | Dataset: Student Performance (Education, Code 3)*
