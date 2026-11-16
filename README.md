# HCMC Traffic LOS Forecasting

**A Machine Learning project for forecasting Traffic Level of Service (LOS) 30 minutes ahead in Ho Chi Minh City, Vietnam.**

## 1. Project Overview

This project aims to develop machine learning models to forecast traffic Level of Service (LOS) on road segments in Ho Chi Minh City, Vietnam.

Instead of classifying the current traffic condition, the project focuses on predicting the traffic condition 30 minutes into the future using historical traffic information, road characteristics, temporal patterns, and weather conditions.

## 2. Research Objective

The prediction task is formulated as:

\[
X_{s,t} \rightarrow LOS_{s,t+30}
\]

Where:

- \(s\): Road segment.
- \(t\): Current observation time.
- \(X_{s,t}\): Features available at time \(t\).
- \(LOS_{s,t+30}\): Traffic Level of Service 30 minutes later.

The target consists of six ordered classes:

- A: Free flow
- B: Reasonably free flow
- C: Stable flow
- D: Approaching unstable flow
- E: Unstable flow
- F: Forced or breakdown flow

The original LOS labels from the source dataset will be retained and aligned to the future observation time.

## 3. Data Sources

| Dataset | Purpose |
|---|---|
| HCMC Traffic Flow Dataset | Traffic observations, LOS labels, road segments and network information |
| Open-Meteo Historical Weather API | Historical weather conditions |
| OpenStreetMap | Optional supplementary road network features |

### Original Data Sources

- [Traffic Flow Data in Ho Chi Minh City – Kaggle](https://www.kaggle.com/datasets/thanhnguyen2612/traffic-flow-data-in-ho-chi-minh-city-viet-nam)
- [Open-Meteo Historical Weather API](https://archive-api.open-meteo.com/v1/archive?latitude=10.823&longitude=106.6296&start_date=2020-07-03&end_date=2021-04-22&hourly=temperature_2m,relative_humidity_2m,precipitation,rain,weather_code,wind_speed_10m,cloud_cover&timezone=Asia%2FHo_Chi_Minh)
- [OpenStreetMap](https://www.openstreetmap.org/) (optional)

## 4. Project Methodology

The project follows these main stages:

1. Data understanding and exploratory data analysis.
2. Data cleaning and quality assessment.
3. Temporal alignment and target construction.
4. Traffic, temporal, weather and road feature engineering.
5. Time-based training, validation and testing.
6. Machine learning model development.
7. Model evaluation and interpretation.

A key methodological requirement is to prevent data leakage. Features must only contain information available at the prediction time.

## 5. Machine Learning Models

The planned models include:

- Persistence Baseline
- Random Forest
- HistGradientBoosting
- XGBoost

Model performance will be evaluated using:

- Accuracy
- Macro F1-score
- Weighted F1-score
- Balanced Accuracy
- Per-class Recall
- Confusion Matrix
- Quadratic Weighted Kappa
- Ordinal Mean Absolute Error

## 6. Project Structure

```text
hcmc-traffic-los-forecasting/
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
│
├── notebooks/
├── src/
├── models/
├── results/
│
├── README.md
├── requirements.txt
└── .gitignore
```

## 7. Data Management

Raw datasets and generated large files are stored locally and are not committed to this repository.

Users should download the original datasets from their respective sources and place them in the appropriate data directories.

## 8. Project Status

**Status: In Development**

The project is currently in the data preparation and exploratory analysis stage.

## 9. Reproducibility

The project uses Python and Jupyter Notebook. Dependency installation and execution instructions will be maintained in this repository.

## 10. Author

Machine Learning Final Term Project

Ho Chi Minh City, Vietnam.
