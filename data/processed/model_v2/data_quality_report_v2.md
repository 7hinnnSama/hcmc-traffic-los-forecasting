# Data Quality Report - HCMC Traffic LOS Forecasting

## 1. Dataset Overview

| Attribute | Value |
|---|---|
| Dataset version | V2 |
| Task | Multiclass classification |
| Prediction objective | LOS at t+30 minutes |
| Train samples | 4,879 |
| Holdout samples | 2,998 |
| Total samples | 7,877 |
| Total features | 19 |
| Numerical features | 15 |
| Categorical features | 4 |
| Target | target_los_30m |
| Target classes | A, B, C, D, E, F |

## 2. Dataset Files

- train_dataset_v2.parquet
- holdout_dataset_v2.parquet
- train_metadata_v2.parquet
- holdout_metadata_v2.parquet
- feature_schema_v2.json

## 3. Target Distribution

| LOS | Train | Train (%) | Holdout | Holdout (%) |
|---|---:|---:|---:|---:|
| A | 956 | 19.59% | 1635 | 54.54% |
| B | 153 | 3.14% | 1004 | 33.49% |
| C | 803 | 16.46% | 32 | 1.07% |
| D | 1701 | 34.86% | 35 | 1.17% |
| E | 1071 | 21.95% | 24 | 0.80% |
| F | 195 | 4.00% | 268 | 8.94% |


## 4. Data Quality Validation

Total checks: 24
Passed: 24
Failed: 0

| Check | Result |
|---|---|
| All 5 deliverables exist | PASS |
| Train shape | PASS |
| Holdout shape | PASS |
| Train metadata shape | PASS |
| Holdout metadata shape | PASS |
| Train and Holdout columns match | PASS |
| Feature columns match schema | PASS |
| Numerical + categorical features complete | PASS |
| Target exists | PASS |
| No missing values in Train | PASS |
| No missing values in Holdout | PASS |
| No missing metadata | PASS |
| No infinite values | PASS |
| No duplicate Train samples | PASS |
| No duplicate Holdout samples | PASS |
| No Train-Holdout sample overlap | PASS |
| Train contains all LOS classes | PASS |
| Holdout contains all LOS classes | PASS |
| Train metadata identifiers valid | PASS |
| Holdout metadata identifiers valid | PASS |
| Train target interval = 30 minutes | PASS |
| Holdout target interval = 30 minutes | PASS |
| Temporal separation | PASS |
| No duplicate sample IDs across partitions | PASS |


## 5. Data Cleaning Operations

- Integrated traffic observations with road attributes.
- Integrated historical weather information.
- Standardized timestamps and timezone handling.
- Generated future LOS target at exactly t+30 minutes.
- Applied chronological Train/Holdout separation.
- Removed observations belonging to invalid zero-length source road segments.
- Refined numerical and categorical feature schemas.
- Preserved metadata for sample traceability.
- Exported datasets in Parquet format.

## 6. Known Limitations

### 6.1 Limited target coverage

Original traffic observations: 33,441.
Valid future targets before road filtering: 7,898.
Original target coverage: 23.62%.
Final samples after invalid road filtering: 7,877.

### 6.2 Temporal distribution shift

The LOS distribution differs substantially between Train and Holdout.
The original chronological split is preserved.
Holdout has not been resampled or used for training.

### 6.3 LOS label definition

LOS labels are inherited from the source dataset.
Their formal classification thresholds have not been independently verified.

### 6.4 Temporal and geographical coverage

The dataset covers a limited historical period and observed road segments.
It should not automatically be considered representative of all roads
and traffic conditions across Ho Chi Minh City.

### 6.5 Weather representation

Weather variables originate from a historical weather source
at a representative HCMC coordinate, not individual road-level weather stations.

## 7. Modeling Handoff Recommendations

- Use train_dataset_v2.parquet for model development.
- Reserve holdout_dataset_v2.parquet for final evaluation.
- Use train_metadata_v2.parquet and holdout_metadata_v2.parquet
  for sample identification and temporal traceability.
- Read feature_schema_v2.json before preprocessing.
- Fit encoders and scalers on training data only.
- Do not fit preprocessing transformations on Holdout.
- Consider class imbalance and temporal distribution shift
  when interpreting model performance.
- Do not modify the original target labels without documented justification.

## 8. Final Conclusion

The V2 dataset passed all 24 implemented
technical data quality checks.

It is structurally validated and ready for handoff
to the machine learning development stage.

This validation does not establish label correctness,
statistical representativeness, or future model performance.

---

Report version: 2.0
