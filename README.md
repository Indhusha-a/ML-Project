# Student Performance — At-Risk Prediction

IT3091 Machine Learning | Guided Data Track | Education (Dataset Code 3)

**Demo Video:** https://youtu.be/sAZqNCwBggo

## Business Problem

An academic institution wants to identify patterns that support student
performance and early intervention. This project predicts a student's final
grade (G3) and flags students likely to be at risk, so that limited
tutoring/counselling resources can be directed to the students who need them
most.

## Dataset

- **Source:** [UCI Machine Learning Repository — Student Performance](https://archive.ics.uci.edu/dataset/320/student-performance)
- **File used:** `data/student-mat.csv` (Math course, 395 students, 33 attributes)
- **MD5 fingerprint:** `4dc304be95c60de6ee13fb8769469dd7`
- See `docs/data_dictionary.md` for full column definitions.

## Approach

- **Primary lens:** performance prediction (regression on G3).
- **Secondary lens:** at-risk student identification (predicted G3 < 10 threshold).
- **Models compared:** Linear Regression (baseline), Random Forest, Ridge
  Regression, Gradient Boosting.
- **Evaluation:** 80/20 train/test split, 5-fold cross-validation for tuning,
  RMSE/MAE/R² and residual analysis on the held-out test set.
- Full reasoning behind every major decision (dataset scope, handling of
  G3 = 0 rows, G1/G2 inclusion, model selection, etc.) is recorded in
  `docs/decision_log.md`.

## Project Structure

```
ML-Project/
├── data/                     Raw dataset (UCI Student Performance, Math)
├── notebooks/
│   └── student_performance.ipynb   All analysis: EDA → preprocessing →
│                                    4 models → evaluation → recommendation
├── docs/
│   ├── data_dictionary.md          Column-by-column dataset reference
│   ├── decision_log.md             Every major decision, options considered, why
│   ├── eda_insight_log.md          Condensed EDA findings
│   └── initial_submission_draft.md Initial Submission: lenses, task/output,
│                                    workflow, and per-member responsibilities
├── requirements.txt          Exact package versions used
└── README.md
```

## Running the Notebook

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m ipykernel install --user --name=ml-project --display-name="ML-Project (venv)"
jupyter notebook
```

Open `notebooks/student_performance.ipynb`, select the `ML-Project (venv)`
kernel, and run all cells top to bottom. Random seed is fixed at 42
throughout for reproducibility.

## Key Results

| Model | Test RMSE | Test R² |
|---|---|---|
| **Gradient Boosting (recommended)** | 1.945 | 0.816 |
| Random Forest | 1.994 | 0.806 |
| Ridge Regression | 2.261 | 0.751 |
| Linear Regression | 2.314 | 0.739 |

Top predictive features (agreed by two independent importance methods): G2,
absences, G1. Full results, the Random Forest overfitting finding, at-risk
flagging performance, and limitations are in `docs/decision_log.md`.

## AI Use Disclosure

Portions of this project's structure and documentation
were developed with AI assistance (Claude). All modelling decisions,
parameter choices, and interpretations were reviewed and executed by the
author.
