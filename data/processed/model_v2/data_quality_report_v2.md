# Data Quality Report — HCMC Traffic LOS Forecasting

## 1. Dataset overview

| Attribute | Value |
|---|---|
| Dataset version | V2 |
| Preparation objective | LOS at t+30 minutes |
| Train samples | 4,879 |
| Holdout samples | 2,998 |
| Total final samples | 7,877 |
| Features | 19 |
| Numerical features | 15 |
| Categorical features | 4 |
| Target | target_los_30m |
| Target classes | A, B, C, D, E, F |

## 2. Deliverables

- `train_dataset_v2.parquet`
- `holdout_dataset_v2.parquet`
- `train_metadata_v2.parquet`
- `holdout_metadata_v2.parquet`
- `feature_schema_v2.json`
- `data_quality_report_v2.md`

## 3. Target coverage

- Original traffic observations: 33,441
- Valid future targets before road-length filtering: 7,898
- Target coverage before road-length filtering: 23.62%
- Samples excluded by the source-length filter: 21
- Final samples: 7,877

## 4. Target distribution

| LOS | Train | Train (%) | Holdout | Holdout (%) |
|---|---:|---:|---:|---:|
| A | 956 | 19.59% | 1635 | 54.54% |
| B | 153 | 3.14% | 1004 | 33.49% |
| C | 803 | 16.46% | 32 | 1.07% |
| D | 1701 | 34.86% | 35 | 1.17% |
| E | 1071 | 21.95% | 24 | 0.80% |
| F | 195 | 4.00% | 268 | 8.94% |


## 5. Final data-quality validation

- Total checks: 25
- Passed: 25
- Failed: 0

| Check | Result |
|---|---|
| All 5 core deliverables exist | PASS |
| Train release shape | PASS |
| Holdout release shape | PASS |
| Train metadata shape | PASS |
| Holdout metadata shape | PASS |
| Train and Holdout columns match | PASS |
| Feature columns match schema | PASS |
| Numerical and categorical features are complete and disjoint | PASS |
| Target exists | PASS |
| No missing values in Train | PASS |
| No missing values in Holdout | PASS |
| No missing metadata | PASS |
| No infinite numerical values | PASS |
| All retained road lengths are positive | PASS |
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


## 6. Data-preparation operations

- Integrated traffic observations with road-segment attributes and node coordinates.
- Integrated historical weather using a backward as-of join with a 30-minute tolerance.
- Generated the future LOS label by exact same-segment matching at t+30 minutes.
- Audited target coverage and temporal distribution.
- Created a chronological Train/Holdout partition and checked target-time separation.
- Refined temporal, road, and weather feature columns.
- Audited categorical availability and numerical feature drift.
- Audited source road-length precision using endpoint coordinates.
- Excluded observations mapped to source segments with a recorded length of 0 m.
- Preserved separate metadata for sample identification and temporal traceability.
- Exported and reloaded the V2 Parquet deliverables.

## 7. Source-length filtering policy

The source road table contains 154 segments whose recorded integer length is 0 m. The endpoint-coordinate audit found that source lengths match the floored Haversine distance for the supplied road table. Thus, these records are sub-meter according to the endpoint coordinates.

The V2 pipeline excludes observations mapped to these segments to avoid zero-valued length features. This is a documented dataset policy; it does not establish that the corresponding road geometries are malformed.

## 8. Known limitations

- Exact future LOS labels are available for 23.62% of the original traffic observations before road-length filtering.
- LOS class proportions differ substantially between Train and chronological Holdout.
- Formal source LOS classification thresholds have not been independently verified.
- Weather variables come from the historical weather source location and are not road-level station measurements.
- The observed roads and historical period should not be assumed to represent all roads or all traffic conditions in Ho Chi Minh City.

## 9. Handoff notes

- The feature table contains the selected predictors and `target_los_30m`.
- Segment identifiers and timestamps are stored separately in the metadata files.
- `feature_schema_v2.json` defines the feature names and their numerical/categorical groups.
- Categorical encoding, numerical scaling, and class balancing have not been applied in this data-preparation pipeline.
- The chronological Holdout partition has been preserved unchanged.

## 10. Conclusion

The V2 release passed all 25 implemented final data-quality checks.

The exported dataset is structurally validated for handoff to the downstream modeling stage. These checks do not establish label correctness, statistical representativeness, or future model performance.

Report version: 2.0
