# Predicting Subscription Fatigue and Platform Switching Behavior Among OTT Users in India

Business Analytics individual case study.

## Student Details
- **Name:** Harshini Vennela
- **Register Number:** CB.SC.U4CSE23455
- **Section:** E
- **Sector:** Entertainment / OTT streaming services

## 1. Problem Statement

OTT platforms (Netflix, Amazon Prime Video, Disney+ Hotstar, etc.) compete for the same viewers, and leaving one costs a subscriber almost nothing — a plan can be paused, downgraded or cancelled in a few taps. Many households hold several subscriptions at once and rotate them as the value they see changes ("multi-homing").

**Subscription fatigue** — the build-up of dissatisfaction that comes before a user switches or cancels — rarely has a single cause. A subscriber who finds a plan too expensive for how little they watch it needs a different response from one who is happy with the price but tired of buffering, weak recommendations or a thin catalogue. Applying a generic discount to the wrong reason wastes marketing spend without fixing the real problem.

The business problem is therefore twofold: **which subscribers are likely to switch**, and **what kind of fatigue lies behind it**, so that retention spend can be targeted rather than applied uniformly. The prediction target is whether a respondent expects to switch, downgrade or cancel their primary OTT subscription within the next six months.

## 2. Objectives

1. Identify which pricing, content, experience and viewing-behaviour factors are most closely associated with subscription fatigue and switching intention.
2. Diagnose whether each respondent's fatigue is mainly cost-, content- or experience-driven, and measure how common each type is.
3. Analyze the influence of pricing, content quality, viewing behaviour and customer satisfaction on platform switching.
4. Build and evaluate an interpretable classification model for six-month switching intention.
5. Turn the results into segment-specific, fatigue-type-specific retention recommendations.

## 3. Dataset

**Collection method.** The primary data was collected with a structured Google Forms questionnaire shared with OTT subscribers, covering six blocks: demographics, subscription details, viewing behaviour, satisfaction and experience ratings (five-point Likert scale), switching history, and a few open-text questions. No names, e-mail addresses or phone numbers were collected — every respondent is identified only by `Record_ID`.

**Real vs. synthetic records — read this before trusting any percentage in this repo.** The questionnaire produced only **13 real, usable responses** — far too few to train or evaluate a classifier. To make the workflow demonstrable, the analytical dataset was expanded to **10,000 records (13 real + 9,987 synthetic)**, where the synthetic rows were generated to follow the answer patterns and relationships observed in the 13 real responses. Every row carries a `Data_Source` column (`Real` or `Synthetic`) so the two groups can always be separated.

Because **99.87% of rows are synthetic**, every percentage, chart and model score in the notebook and report describes this *analytical* dataset — it is **not** a statistically representative sample of OTT users in India. The results should be read as a worked demonstration of the analytics pipeline and as hypotheses to be re-tested once more genuine responses are collected, not as population-level findings.

**Files in `data/`:**

| File | Rows | Description |
|---|---|---|
| `final_ott_dataset_10000.csv` | 10,000 × 32 cols | **The dataset used by `analysis.ipynb`.** 13 real survey responses + 9,987 synthetic records generated from their patterns. |
| `final_ott_dataset_10000.xlsx` | 10,000 × 32 cols | Same data as above, in Excel format. |
| `OTT_BA_Survey_120_Rows.csv` | 120 × 33 cols | An earlier export from the same Google Forms questionnaire (13 real responses + an earlier, smaller round of 107 synthetic rows), kept in the repository for provenance/reference. The notebook does **not** read this file. |

**Key variables:** the target `Switch_Intent` (Yes/No, six-month horizon); six 1–5 satisfaction ratings (pricing, content, content variety, recommendation quality, advertisement tolerance, streaming experience); subscription and spending details (platform, number of subscriptions, tenure, monthly spend, account sharing); viewing behaviour (hours watched, genre, device, time of day); switching history and competitor awareness; and four derived scores — `Cost_Fatigue_Score`, `Content_Fatigue_Score`, `Experience_Fatigue_Score`, `Overall_Fatigue_Score` — plus a `Dominant_Fatigue_Type` label built from them.

## 4. Data Collection Method

Responses were gathered through a self-administered Google Forms questionnaire distributed to OTT subscribers. The form had no identifying fields; each response was assigned a `Record_ID` on export. As noted above, the form returned 13 usable responses, which is the entire real-data foundation of this project — this is disclosed explicitly rather than presented as a larger genuine sample.

## 5. Technologies Used

- Python 3
- Pandas, NumPy
- Matplotlib, Seaborn
- Scikit-learn (`LogisticRegression`, `ColumnTransformer`, `Pipeline`, `StratifiedKFold`, metrics)
- Jupyter Notebook
- Excel / CSV for data storage

## 6. Data Preparation

- Cleaned column names and category labels so each variable is coded consistently.
- Converted range answers (age, income, tenure, monthly spend, weekly viewing hours) into representative numeric `_Numeric` columns.
- Kept the six Likert ratings as 1–5 integers; mapped the Yes/No target to 0/1; one-hot encoded the remaining Yes/No fields inside the modelling pipeline.
- Checked for duplicates (none found). One record had no target label and was excluded from modelling, leaving 9,999 usable rows.
- Kept open-text answers (reasons for switching/staying, suggestions) as background context — they are not model inputs.
- Derived the cost/content/experience/overall fatigue scores from the related ratings and assigned each respondent a `Dominant_Fatigue_Type`.
- Exploratory analysis: distribution plots for platform, occupation, subscriptions, switching intention and fatigue type; cross-tabulations of switching intention against pricing/content/experience ratings; and a correlation heatmap across numeric, rating and fatigue variables.

## 7. Analytics Method

**Logistic regression** (class-balanced) was used as the classification method — the target is binary, its coefficients are directly interpretable for a retention team, and it is light enough to retrain regularly on largely synthetic data. Implementation steps performed in `analysis.ipynb`:

1. Kept 9,999 labelled records with 25 predictors (16 numeric, 9 categorical).
2. Stratified 80:20 train/test split (`random_state=42`) → 7,999 train / 2,000 test records.
3. Preprocessing inside a single scikit-learn pipeline: median imputation + standardisation for numeric features, most-frequent imputation + one-hot encoding for categorical features, fitted on the training data only.
4. Logistic regression with `class_weight="balanced"`.
5. Evaluation on the held-out test set: accuracy, precision, recall, F1, confusion matrix, ROC-AUC/PR-AUC, 5-fold stratified cross-validation, a decision-threshold sweep, and two robustness checks (dropping the fatigue-score features; training on synthetic rows only and checking against the 13 real rows).

## 8. Key Results

- Analytical dataset: **10,000 records, 32 columns, 0 duplicates** (13 real, 9,987 synthetic).
- **29.88%** of respondents intend to switch within six months.
- Dominant fatigue type: **Cost 44.25%**, **Experience 34.51%**, **Content 21.24%**.
- Switching intention falls as pricing, content and streaming-experience ratings rise, most steeply for pricing.
- Test-set performance (n = 2,000, 598 real switchers): **Accuracy 62.05%, Precision 40.92%, Recall 60.70%, F1 48.89%** (confusion matrix: 363 true positives, 524 false positives, 235 false negatives, 878 true negatives). ROC-AUC ≈ 0.668.

All figures above are reproduced by running `analysis.ipynb` end-to-end and are cross-checked against `Case_Study_Report.pdf`.

## 9. Business Insights

- Fatigue has more than one face — cost is the largest single group, but 55.75% of respondents are mainly bothered by experience or content problems that a price cut alone will not fix.
- Pricing satisfaction is the clearest warning sign: users rating pricing 1–2 intend to switch at roughly one in three, versus about one in four among those rating it 5.
- Prior switching (49.88%) and competitor awareness (48.92%) are too common on their own to separate at-risk users; they work better as tie-breakers alongside the model's risk score.
- The model is best used as a screening tool (about 4 in 10 flagged users are real switchers), not as a trigger for expensive, automatic offers.

## 10. Recommendations

| Fatigue type | Recommended action |
|---|---|
| Cost-driven | Flexible/family/student plans and personalised pricing offers for price-sensitive users |
| Content-driven | Better genre-level personalisation and content acquisition aligned with preferences |
| Experience-driven | Reduce buffering, improve recommendation quality, and improve overall app/streaming experience |
| High predicted risk (any type) | Route through the classification model for a targeted, low-cost retention nudge before offering a discount |

**Caveat:** because 99.87% of the analytical dataset is synthetic, these recommendations should be treated as hypotheses for a pilot, not as validated policy, until re-tested on a larger set of genuine survey responses.

## 11. Project Structure

```
OTT-Business-Analytics-Case-Study/
├── README.md
├── analysis.ipynb
├── Case_Study_Report.pdf
└── data/
    ├── final_ott_dataset_10000.csv
    ├── final_ott_dataset_10000.xlsx
    └── OTT_BA_Survey_120_Rows.csv
```

## 12. How to Run

1. Clone or download this repository.
2. Install dependencies: `pip install pandas numpy matplotlib seaborn scikit-learn jupyter openpyxl`
3. Open `analysis.ipynb` in Jupyter (`jupyter notebook analysis.ipynb`) or VS Code / Google Colab.
4. Run all cells from top to bottom — the notebook loads `data/final_ott_dataset_10000.csv` using a relative path (`data/...`), so it will work as long as the notebook stays in the repository root next to the `data/` folder.

## Full Report

See **`Case_Study_Report.pdf`** for the complete write-up: problem statement, dataset and collection method, data preparation and EDA, analytics methodology and implementation, results, a comparison with three published studies, business insights, fatigue-type-specific recommendations, limitations, conclusion and references.
