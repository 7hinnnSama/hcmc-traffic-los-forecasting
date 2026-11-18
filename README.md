# HCMC Traffic LOS Forecasting

**A Data Mining final-term project at Saigon University (SGU), 2026.**

A collaborative project to prepare data and develop machine-learning models for forecasting the Traffic Level of Service (LOS) of road segments in Ho Chi Minh City, Vietnam, **30 minutes ahead**.

## 1. Project Overview

This project studies how historical traffic observations, road-network characteristics, temporal patterns, and weather conditions can be used to forecast the future traffic condition of a road segment.

The prediction task is not to classify traffic at the current observation time. It is to predict the LOS of the same road segment 30 minutes later, using only information available at the current observation time.

## 2. Research Objective

For a road segment `s` observed at time `t`, the task is:

\[
X_{s,t} \rightarrow LOS_{s,t+30}
\]

Where:

- `s` is a road segment.
- `t` is the current observation time.
- `X_{s,t}` contains features available at time `t`.
- `LOS_{s,t+30}` is the LOS label for the same segment exactly 30 minutes later.

The target uses six ordered LOS classes:

| Class | Description |
|---|---|
| A | Free flow |
| B | Reasonably free flow |
| C | Stable flow |
| D | Approaching unstable flow |
| E | Unstable flow |
| F | Forced or breakdown flow |

The target column in the prepared datasets is `target_los_30m`.

## 3. Data Sources

### HCMC traffic data

The traffic dataset is obtained from [Traffic Flow Data in Ho Chi Minh City, Vietnam (Kaggle)](https://www.kaggle.com/datasets/thanhnguyen2612/traffic-flow-data-in-ho-chi-minh-city-viet-nam). The source tables used in the preparation workflow are:

| File | Purpose |
|---|---|
| `train.csv` | Traffic observations and current LOS labels |
| `segment_status.csv` | Segment-level traffic status and velocity observations |
| `segments.csv` | Road-segment attributes and endpoint references |
| `streets.csv` | Street attributes |
| `nodes.csv` | Road-network node coordinates |

### Historical weather data

Hourly historical weather data was retrieved from the [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) for Ho Chi Minh City, using the period **2020-07-03 to 2021-04-22** and the `Asia/Ho_Chi_Minh` timezone.

The source variables include temperature, relative humidity, precipitation, rain, weather code, wind speed, and cloud cover. The final model feature set uses the selected weather variables listed in the feature schema.

The current workflow uses one city-level weather time series for the road-segment observations; it does not represent weather measurements collected separately at every road segment.

## 4. Data Preparation Methodology

The data-preparation workflow is implemented in the Jupyter notebooks under `notebooks/`:

1. **Environment check** – verify the Python environment and required libraries.
2. **Data inventory** – inspect source tables, data types, missing values, keys, relationships, and temporal coverage.
3. **Data preprocessing** – normalize timestamps, derive temporal features, validate source tables, and export preprocessed data.
4. **Data integration** – integrate traffic observations with road and weather attributes, construct the 30-minute future target, create chronological partitions, refine features, and validate the final datasets.

Important preparation principles:

- Timestamps are normalized and audited in UTC; HCMC-local temporal features use the `Asia/Ho_Chi_Minh` timezone.
- Road and weather information is aligned to traffic observations.
- A future target is retained only when the same segment has a valid LOS observation exactly 30 minutes after the current observation.
- Input features represent information available at the observation time; future LOS is used only as the target.
- Train and holdout partitions are separated chronologically to avoid training on observations later than the holdout period.
- Invalid source road segments with zero recorded length are excluded from the final modeling datasets rather than assigning them an invented road length.

## 5. Final Prepared Dataset (model_v2)

The original traffic table contains **33,441 observations**. An exact future LOS label at `t+30 minutes` was available for **7,898 observations (23.62%)**. During final road-data validation, observations associated with invalid zero-length source segments were removed: 12 from the training partition and 9 from the holdout partition.

The resulting `model_v2` dataset contains **7,877 samples**:

| Partition | Samples | Features + target |
|---|---:|---:|
| Train | 4,879 | 20 columns |
| Holdout | 2,998 | 20 columns |
| **Total** | **7,877** | **20 columns** |

Each dataset contains **19 input features and one target column**. The feature schema groups the inputs as follows:

**Numerical features (15)**

- `hour_sin`, `hour_cos`
- `weekday_sin`, `weekday_cos`
- `month_sin`, `month_cos`
- `is_weekend`
- `length`
- `center_longitude`, `center_latitude`
- `temperature_2m`, `relative_humidity_2m`, `precipitation`
- `wind_speed_10m`, `cloud_cover`

**Categorical features (4)**

- `LOS`
- `street_level`
- `street_type`
- `weather_code`

**Target:** `target_los_30m`

### Target class distribution

| Target class | Train | Holdout |
|---|---:|---:|
| A | 956 | 1,635 |
| B | 153 | 1,004 |
| C | 803 | 32 |
| D | 1,701 | 35 |
| E | 1,071 | 24 |
| F | 195 | 268 |
| **Total** | **4,879** | **2,998** |

### Temporal separation

The final partitions use a chronological holdout strategy:

- Training data ends with a latest target time of **2020-12-06 15:00:00 UTC**.
- The earliest holdout observation is **2020-12-07 09:00:00 UTC**.
- Every retained sample has a target interval of exactly 30 minutes.

### Exported artifacts

The prepared artifacts are stored in `data/processed/model_v2/`:

| File | Description |
|---|---|
| `train_dataset_v2.parquet` | Training features and target |
| `holdout_dataset_v2.parquet` | Holdout features and target |
| `train_metadata_v2.parquet` | Training sample identifiers and timestamps |
| `holdout_metadata_v2.parquet` | Holdout sample identifiers and timestamps |
| `feature_schema_v2.json` | Feature definitions, types, and dataset configuration |
| `data_quality_report_v2.md` | Final data-quality audit report |

## 6. Data Quality Assurance

The final dataset passed **24 out of 24** automated quality checks. The checks cover:

- Required deliverables and expected dataset shapes.
- Agreement between train, holdout, and feature-schema columns.
- Missing and infinite values.
- Duplicate samples and train–holdout sample overlap.
- Valid sample identifiers and metadata.
- Presence of all six LOS classes in both partitions.
- An exact 30-minute target interval.
- Chronological separation between training targets and holdout observations.

The final feature datasets contain no missing feature or target values. The detailed checks and results are documented in `data_quality_report_v2.md`.

## 7. Data Limitations

The prepared data has limitations that should be considered during subsequent model development and evaluation:

- **Limited target coverage:** only 7,898 of the 33,441 original traffic observations (23.62%) had a matching future LOS observation exactly 30 minutes later.
- **Different class distributions across time:** the training and holdout partitions have substantially different LOS distributions. In particular, the holdout set contains relatively few examples of classes C, D, and E. Metrics should therefore be interpreted with per-class results and the temporal distribution shift in mind.
- **City-level weather input:** the weather time series is associated with one HCMC location and is shared across road-segment observations; it is not segment-specific weather data.
- **Historical coverage:** the source observations cover 2020-07-03 to 2021-04-22 and may not represent current traffic patterns.
- **Invalid road records:** 154 source road segments had zero recorded length. Samples associated with these invalid segments were excluded from the final train and holdout datasets.

These limitations are documented for transparency and should be considered before drawing conclusions from model results.

## 8. Subsequent Machine-Learning Work

The prepared `model_v2` datasets are intended as inputs for the project's separate model-development workstream. The planned model families in the project include:

- Persistence baseline
- Random Forest
- HistGradientBoosting
- XGBoost

Potential evaluation measures include Accuracy, Macro F1-score, Weighted F1-score, Balanced Accuracy, per-class Recall, Confusion Matrix, Quadratic Weighted Kappa, and Ordinal Mean Absolute Error.

This README documents the data-preparation deliverables; it does not claim that these models have already been trained or evaluated.

## 9. Project Structure

```text
hcmc-traffic-los-forecasting/
│
├── data/
│   ├── raw/                         # Original CSV files (kept out of Git)
│   ├── processed/
│   │   └── model_v2/                # Selected prepared datasets and reports
│   └── external/
│
├── notebooks/
│   ├── 00_environment_check.ipynb
│   ├── 01_data_inventory.ipynb
│   ├── 02_data_preprocessing.ipynb
│   └── 03_data_integration.ipynb
│
├── src/
├── models/
├── results/
├── README.md
├── requirements.txt
├── requirements-lock.txt
└── .gitignore
```

## 10. Reproducibility

### Environment setup

Create and activate a virtual environment, then install the project dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

### Source data setup

The original traffic CSV files and `weather_hcm.csv` should be obtained from their respective sources and placed in `data/raw/` using the filenames expected by the notebooks. Raw source files are excluded from this repository; users must obtain them separately and comply with the original source terms.

### Notebook execution order

Run the notebooks in this order:

1. `00_environment_check.ipynb`
2. `01_data_inventory.ipynb`
3. `02_data_preprocessing.ipynb`
4. `03_data_integration.ipynb`

The final prepared artifacts are written to `data/processed/model_v2/`.

## 11. Project Status

**Data preparation status: Completed (`model_v2`).**

The cleaned train and holdout datasets, metadata, feature schema, and data-quality report have been exported and validated. The subsequent model-design, training, and evaluation work is a separate project stage.

## 12. Team and Contributions

**Course:** Data Mining – Final Term Project  
**Institution:** Saigon University (SGU)  
**Academic year:** 2026

| Team member | Contribution |
|---|---|
| **Nguyễn Mai Trâm** | Machine-learning model design and training workstream. |
| **7Hin** | Data inventory and exploratory data audit; data cleaning and validation; timestamp normalization and temporal feature engineering; road-network and weather integration; 30-minute future target construction; chronological dataset preparation; final data-quality audits; dataset export and documentation. |

## 13. References and Acknowledgements

- [Traffic Flow Data in Ho Chi Minh City, Vietnam – Kaggle](https://www.kaggle.com/datasets/thanhnguyen2612/traffic-flow-data-in-ho-chi-minh-city-viet-nam)
- [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api)
- [Open-Meteo API request used for this project](https://archive-api.open-meteo.com/v1/archive?latitude=10.823&longitude=106.6296&start_date=2020-07-03&end_date=2021-04-22&hourly=temperature_2m,relative_humidity_2m,precipitation,rain,weather_code,wind_speed_10m,cloud_cover&timezone=Asia%2FHo_Chi_Minh)

The original datasets and services remain subject to their respective terms and licenses. Making this repository public does not by itself grant permission to redistribute third-party source data.
