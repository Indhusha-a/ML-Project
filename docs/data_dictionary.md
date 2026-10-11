# Data Dictionary

Dataset: UCI Student Performance — Math (`data/student-mat.csv`)
395 records, 33 attributes. Source: https://archive.ics.uci.edu/dataset/320/student-performance
MD5 fingerprint: `4dc304be95c60de6ee13fb8769469dd7`

| # | Column | Type | Description | Values / Range |
|---|---|---|---|---|
| 1 | `school` | Binary | Student's school | `GP` (Gabriel Pereira), `MS` (Mousinho da Silveira) |
| 2 | `sex` | Binary | Student's sex | `F`, `M` |
| 3 | `age` | Numeric | Student's age | 15–22 |
| 4 | `address` | Binary | Home address type | `U` (urban), `R` (rural) |
| 5 | `famsize` | Binary | Family size | `LE3` (≤3), `GT3` (>3) |
| 6 | `Pstatus` | Binary | Parents' cohabitation status | `T` (together), `A` (apart) |
| 7 | `Medu` | Ordinal | Mother's education | 0 (none) – 4 (higher education) |
| 8 | `Fedu` | Ordinal | Father's education | 0 (none) – 4 (higher education) |
| 9 | `Mjob` | Nominal | Mother's job | `teacher`, `health`, `services`, `at_home`, `other` |
| 10 | `Fjob` | Nominal | Father's job | `teacher`, `health`, `services`, `at_home`, `other` |
| 11 | `reason` | Nominal | Reason for choosing this school | `home`, `reputation`, `course`, `other` |
| 12 | `guardian` | Nominal | Student's guardian | `mother`, `father`, `other` |
| 13 | `traveltime` | Ordinal | Home-to-school travel time | 1 (<15min) – 4 (>1hr) |
| 14 | `studytime` | Ordinal | Weekly study time | 1 (<2hrs) – 4 (>10hrs) |
| 15 | `failures` | Numeric | Number of past class failures | 0–3 (4 = "3 or more") |
| 16 | `schoolsup` | Binary | Extra educational support | `yes`, `no` |
| 17 | `famsup` | Binary | Family educational support | `yes`, `no` |
| 18 | `paid` | Binary | Extra paid classes (in-subject) | `yes`, `no` |
| 19 | `activities` | Binary | Extra-curricular activities | `yes`, `no` |
| 20 | `nursery` | Binary | Attended nursery school | `yes`, `no` |
| 21 | `higher` | Binary | Wants to pursue higher education | `yes`, `no` |
| 22 | `internet` | Binary | Internet access at home | `yes`, `no` |
| 23 | `romantic` | Binary | In a romantic relationship | `yes`, `no` |
| 24 | `famrel` | Ordinal | Quality of family relationships | 1 (very bad) – 5 (excellent) |
| 25 | `freetime` | Ordinal | Free time after school | 1 (very low) – 5 (very high) |
| 26 | `goout` | Ordinal | Going out with friends | 1 (very low) – 5 (very high) |
| 27 | `Dalc` | Ordinal | Workday alcohol consumption | 1 (very low) – 5 (very high) |
| 28 | `Walc` | Ordinal | Weekend alcohol consumption | 1 (very low) – 5 (very high) |
| 29 | `health` | Ordinal | Current health status | 1 (very bad) – 5 (very good) |
| 30 | `absences` | Numeric | Number of school absences | 0–93 (observed max in data: 75) |
| 31 | `G1` | Numeric | First period grade | 0–20 |
| 32 | `G2` | Numeric | Second period grade | 0–20 |
| 33 | `G3` | Numeric | **Final grade (target variable)** | 0–20 |

## Row meaning (unit of analysis)

One row = one student enrolled in the Math course at one of two secondary
schools in Portugal (Gabriel Pereira or Mousinho da Silveira).

## Data quality notes

- No null values across all 33 columns.
- No duplicate rows.
- No inconsistent category formatting (all categorical values pre-clean).
- 33 of 395 students have `G3 = 0`. Investigated in EDA: these students have
  `absences = 0` but already-declining G1→G2 grades, consistent with exam
  non-participation/withdrawal rather than a data error — kept in the dataset
  as a genuine at-risk outcome (see `decision_log.md`, decision #5).
- `absences` is heavily right-skewed (max 75 vs. median 4); only 4 students
  exceed 30 — addressed with a log1p transform during preprocessing rather
  than removal (decision #7).

## Note on scope

This project uses the Math dataset (`student-mat.csv`) only. The companion
Portuguese-language dataset (`student-por.csv`, 649 records) and the 382
students shared between both datasets (identifiable via `student-merge.R`)
were out of scope given the project timeline — documented as a limitation.
