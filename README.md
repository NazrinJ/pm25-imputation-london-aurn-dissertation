# PM2.5 Imputation in the London AURN Monitoring Network

This repository contains the code and notebooks for my MSc dissertation project:

**A Comparative Evaluation of Statistical, Machine Learning and Deep Learning Methods for Spatio-Temporal PM2.5 Imputation in the London AURN Monitoring Network**

The project evaluates different approaches for imputing missing hourly PM2.5 observations using data from selected London AURN monitoring stations between 2021 and 2025.

## Project status

The repository currently includes the completed data preparation, exploratory analysis, artificial masking setup, and statistical baseline modelling stages. Machine learning and deep learning models will be added in later stages of the project.

Current completed stages:

- Station and period selection
- Final panel construction and validation
- Missingness and data quality analysis
- PM2.5 distribution and temporal exploratory data analysis
- Spatial predictor and train-validation-test split analysis
- Artificial masking scenario generation
- Statistical baseline imputation models

Planned next stages:

- Random Forest imputation model
- XGBoost imputation model
- LSTM (unidirectional) deep learning model
- Overall model comparison across masking scenarios

## Repository structure

```text
Codes/
├── data/          # Processed or intermediate data files used by notebooks
├── models/        # Local model outputs, excluded from GitHub where large
├── notebooks/     # Jupyter notebooks for each project stage
├── results/       # Figures, tables and model evaluation outputs
├── .gitignore
└── README.md
```

## Notebooks

The current workflow is organised into sequential notebooks:

| Notebook | Purpose |
| --- | --- |
| `01_station_and_period_selection.ipynb` | Selects the study period and final AURN monitoring stations based on PM2.5 capture criteria. |
| `02_missingness_and_data_quality_eda.ipynb` | Analyses missingness, data quality, missing gap lengths and PM2.5/temperature availability. |
| `03_pm25_distribution_and_temporal_eda.ipynb` | Explores PM2.5 distribution, skewness, temporal patterns and station-level variation. |
| `04_spatial_predictor_and_split_eda.ipynb` | Examines spatial relationships and prepares train-validation-test split checks. |
| `05_modelling_dataset_and_masking_setup.ipynb` | Builds the modelling dataset and creates artificial masking scenarios. |
| `06_statistical_baseline_imputation.ipynb` | Evaluates statistical baseline imputation methods under random and block masking scenarios. |

## Data summary

The final analytical panel contains hourly PM2.5 and temperature data for five London AURN stations:

- London Bexley
- London Harlington
- London Honor Oak Park
- London Marylebone Road
- London N. Kensington

Final panel structure:

| Item | Value |
| --- | --- |
| Study period | 2021–2025 |
| Number of stations | 5 |
| Hourly records per station | 43,824 |
| Total station-hour rows | 219,120 |
| PM2.5 capture | 97.67% |
| Temperature capture | 98.08% |

## Artificial masking scenarios

Five artificial masking scenarios are used to evaluate imputation performance on originally observed PM2.5 values:

| Scenario | Description |
| --- | --- |
| `random10_point` | Random point masking at 10% intensity |
| `random30_point` | Random point masking at 30% intensity |
| `random50_point` | Random point masking at 50% intensity |
| `block24_10percent` | 24-hour block masking at approximately 10% intensity |
| `block72_10percent` | 72-hour block masking at approximately 10% intensity |

The training split is not artificially masked. Artificial masking is applied only to the validation and test splits so that model predictions can be compared against known PM2.5 ground-truth values. All masking scenarios use a fixed random seed (42), and the three random point scenarios are nested (10% ⊂ 30% ⊂ 50%).

## Statistical baseline models

The current modelling stage evaluates three PM2.5-only statistical baselines:

| Model | Description |
| --- | --- |
| Station train mean | Static station-specific mean calculated from the training split. |
| Linear interpolation | Local temporal interpolation within each station and split. |
| Kalman-ARIMA | Station-specific univariate ARIMA implemented using the SARIMAX state-space framework, without seasonal or exogenous components. |

Initial results show that Kalman-ARIMA and linear interpolation perform strongly under random point masking, while errors increase substantially under longer block-gap scenarios. The station train mean baseline acts as a lower benchmark and performs weakest across all scenarios.

## Notes on data and large files

Large raw datasets and trained model files are not included in this repository. In particular, large `.pkl` model outputs and raw data files are excluded using `.gitignore`.

This keeps the repository focused on reproducible code, notebooks and selected results rather than large local files.

## Requirements

The notebooks were developed in Python using Jupyter Notebook. Main Python libraries used include:

- pandas
- numpy
- matplotlib
- seaborn
- scikit-learn
- statsmodels

A formal `requirements.txt` file may be added later as the modelling workflow expands (XGBoost and deep learning libraries will be added with the corresponding notebooks).

## Reproducibility notes

The project is designed as a sequential notebook workflow. The notebooks should generally be run in order from Notebook 01 to Notebook 06, because later notebooks depend on processed outputs created in earlier stages.

The main leakage rule used throughout the modelling setup is:

- `PM25_original` is the ground-truth PM2.5 value used only for evaluation.
- `PM25_model_input` is the model-visible PM2.5 value used for imputation and feature construction.
