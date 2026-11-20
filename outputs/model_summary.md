# Stroke Dataset: Missingness and Baseline Model Findings

## Project scope

This project uses a stroke-prediction dataset to practise SQL, Python, machine learning, and potentially Power BI. The model predicts the dataset’s **recorded stroke label**; it does not establish a person’s future stroke risk or provide a clinical assessment.

The data contains **6,053 records and 27 columns**. The target is `stroke`, where `0` means no recorded stroke and `1` means recorded stroke:

- No stroke: **5,773** (95.4%)
- Stroke: **280** (4.6%)

## Missing data

The original data has no missing values for `SEQN`, `gender`, `age`, `race`, `diabetes`, or `stroke`.

| Feature | Missing | Percent |
|---|---:|---:|
| alcohol_frequency | 1,649 | 27.24% |
| sleep_bedtime | 97 | 1.60% |
| bmi | 94 | 1.55% |
| bmi_category | 94 | 1.55% |
| weight_kg | 81 | 1.34% |
| height_cm | 67 | 1.11% |
| sleep_duration_hours | 63 | 1.04% |
| high_cholesterol | 35 | 0.58% |
| coronary_heart_disease | 22 | 0.36% |
| angina | 15 | 0.25% |
| kidney_disease | 11 | 0.18% |
| thyroid_problem | 11 | 0.18% |
| liver_condition | 10 | 0.17% |
| congestive_heart_failure | 8 | 0.13% |
| heart_attack | 8 | 0.13% |
| cancer | 6 | 0.10% |
| hypertension | 4 | 0.07% |
| smoking_status | 4 | 0.07% |
| education | 4 | 0.07% |
| marital_status | 4 | 0.07% |
| employment_status | 1 | 0.02% |

### Alcohol frequency

The documentation says some survey answers, such as “Never” or “I don’t know,” may have been converted to null. The affected records cannot be identified or recovered, so the nulls are treated as **unclear**, not assumed to mean “Never.” Alcohol frequency was excluded from the initial machine-learning models.

### Height, weight, and BMI

Every missing BMI value coincides with missing height or weight:

- All three missing: **54** records
- Weight and BMI missing; height present: **27**
- Height and BMI missing; weight present: **13**

This pattern is consistent with BMI depending on height and weight; its missingness was not treated as an independent issue.

BMI missingness was **1.4%** in the non-stroke group and **3.9%** in the stroke group. Weight and height were also proportionally more often missing in the stroke group. These are descriptive differences and do not establish why values are missing.

Missingness varied only slightly by age:

| Age group | Weight missing | Height missing |
|---|---:|---:|
| 20–39 | 1.22% | 1.22% |
| 40–59 | 1.27% | 0.98% |
| 60–80 | 1.45% | 1.12% |

By gender, weight was missing in **1.47%** of female records and **1.18%** of male records; height was missing in **1.11%** of each group. The source documentation does not clarify how height and weight were collected, so no cause is inferred.

Sleep duration and bedtime also showed co-missingness. Irregular sleep was considered as a possible explanation, but the pattern does not establish that.

Overall, the exploration found patterns in observed data but did not determine whether the remaining missingness is MCAR, MAR, or MNAR.

## Modeling data preparation

For the initial model, `stroke` was the target; `SEQN` was excluded as an identifier, and alcohol frequency was excluded because its missing values are ambiguous. BMI was used instead of `weight_kg`, `height_cm`, and `bmi_category`. `diabetes`—including its valid `0.5` prediabetes value—and the 0/1 health-condition fields were treated as categorical.

The numeric predictors were age, BMI, and sleep duration. Their missing values were median-imputed, with missingness indicators. Categorical predictors were one-hot encoded; missing values were represented as a `Missing` category. Preprocessing was fitted on training data, not on the test data.

A complete-case version using the selected predictors retained **5,744 of 6,053** records. It retained **251 of 280** stroke cases, meaning 29 stroke records were excluded. The imputed version retained more training records.

Data was split into training and test sets using stratification:

- Training: **4,842** records; stroke rate **4.63%**
- Test: **1,211** records; stroke rate **4.62%**

## Baseline and model comparison

A dummy classifier that always predicted “no stroke” achieved about **95% accuracy**, but detected **zero** stroke cases. This demonstrates why accuracy alone is misleading for this imbalanced target.

At the default 0.5 cutoff, the random forest detected **1 of 56** test stroke cases. Its accuracy was also about 95%, but its stroke recall was only **1.8%**.

### Complete-case versus imputed model

Both models were compared on the same **1,152 complete test records**:

| Model | Stroke cases detected | Precision | Recall | Average precision |
|---|---:|---:|---:|---:|
| Imputed random forest | 1 of 49 | 0.143 | 0.020 | 0.129 |
| Complete-case random forest | 1 of 49 | 0.125 | 0.020 | 0.116 |

The results were similar. Imputation retained more training records and did not perform worse in this comparison.

### Random forest versus logistic regression

Five-fold cross-validation on the training data gave:

| Model | Average precision (mean ± fold standard deviation) |
|---|---:|
| Random forest | 0.143 ± 0.020 |
| Logistic regression | 0.159 ± 0.028 |

The logistic regression mean was slightly higher, but the difference was small relative to fold-to-fold variation. Average precision is a ranking metric, not accuracy; a rough no-skill reference is the stroke prevalence, about **0.046**.

Both models then used a training cross-validation cutoff selected to flag no more than 15% of records. The cutoffs differed because each model produces scores on a different scale:

| Model | Cutoff | Training CV flagged | Training CV precision | Training CV recall |
|---|---:|---:|---:|---:|
| Random forest | 0.2200 | 14.8% | 0.155 | 0.496 |
| Logistic regression | 0.6873 | 14.8% | 0.155 | 0.496 |

On the held-out test set:

| Model | Flagged | Test precision | Test recall | Test average precision |
|---|---:|---:|---:|---:|
| Random forest | 195 / 1,211 (16.1%) | 0.133 | 0.464 | 0.136 |
| Logistic regression | 197 / 1,211 (16.3%) | 0.112 | 0.393 | 0.125 |

The random forest detected **26 of 56** stroke cases; logistic regression detected **22 of 56**. Both flagged slightly more than the 15% training cap on the test set. The test-set difference is small and based on only 56 stroke cases; there is no confident winner.

## Limitations and next steps

- Treat these models as **exploratory practice**, not clinical tools or predictors of future stroke.
- The 15% flag limit is a project constraint, not a medically validated operating point.
- The test set has already been used to inspect results, so it is no longer a fully untouched final evaluation set.
- Keep the original data unchanged. The model-prepared data is not a general-purpose cleaned dataset.
- Next, document this baseline and continue with descriptive SQL analysis and a Power BI dashboard.