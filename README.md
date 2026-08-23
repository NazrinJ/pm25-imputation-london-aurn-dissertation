# PM2.5 Imputation in the London AURN Monitoring Network

This repository contains the code and notebooks for the MSc dissertation:

**A Comparative Evaluation of Statistical, Machine Learning and Deep Learning Methods for PM2.5 Imputation in the London AURN Monitoring Network**

The project compares statistical, machine-learning and deep-learning approaches for retrospective hourly PM2.5 imputation across five selected London AURN monitoring stations using data from 2021 to 2025.

## Repository structure

```text
Codes/
├── data/
│   ├── raw/               # Raw hourly CSV files downloaded from UK-AIR AURN
│   ├── processed/         # Cleaned panel, station metadata, neighbour weights
│   └── modelling/         # Masked scenario datasets for Train, Validation and Test
├── models/
│   └── deep_learning/     # Trained model checkpoints (.pt)
├── notebooks/             # Jupyter notebooks for each project stage
├── results/
│   ├── statistical_baselines/
│   ├── ml_evaluation/
│   ├── deep_learning/
│   │   ├── 08a_data_audit/
│   │   ├── 08b_tuning/
│   │   └── 08c_final_evaluation/
│   ├── final_comparison/
│   │   └── 09_cross_tier/
│   └── regenerated_heatmaps/
├── .gitignore
└── README.md
```

## Notebooks

The workflow is organised as a sequential pipeline. Each notebook reads the verified outputs of its predecessor. No notebook re-tunes or re-fits a model frozen in an earlier stage.

| Notebook | Purpose |
|---|---|
| `01_station_and_period_selection.ipynb` | Selects the five study stations and the 2021–2025 period based on the AURN 85% data-capture threshold. Saves the processed PM2.5–temperature panel and station metadata. |
| `02_missingness_and_data_quality_eda.ipynb` | Profiles natural PM2.5 missingness by station, year, month, season and consecutive gap length. |
| `03_pm25_distribution_and_temporal_eda.ipynb` | Explores PM2.5 distributional, seasonal, diurnal and station-level patterns. |
| `04_spatial_predictor_and_split_eda.ipynb` | Examines temperature as an auxiliary predictor, computes inverse-distance-weighted nearest-neighbour relationships and verifies the chronological train–validation–test split. |
| `05_modelling_dataset_and_masking_setup.ipynb` | Constructs the modelling dataset and applies the five artificial masking scenarios to the Validation and Test periods. |
| `06_statistical_baselines.ipynb` | Evaluates three statistical baselines (station train mean, linear interpolation, Kalman-ARIMA) across all five masking scenarios. |
| `07_ml_tuning.ipynb` | Tunes Random Forest and XGBoost under non-spatial and spatio-temporal feature sets using a family-balanced Validation objective (30 candidates per configuration). |
| `08_ml_evaluation.ipynb` | Evaluates the four frozen ML configurations on the held-out Test split and computes spatial-gain diagnostics. |
| `09a_dl_data_audit.ipynb` | Audits the deep-learning data contract, constructs the fixed top-three IDW station graph and adjacency matrix. |
| `09b_dl_tuning.ipynb` | Tunes twelve structured deep-learning candidates (compact GRIN-inspired graph-enhanced bidirectional GRU) and selects the final configuration using Validation data only. |
| `09c_dl_evaluation.ipynb` | Performs final fitting and evaluation of the graph-enhanced BiGRU and the zero-neighbour ablation control. Produces spatial-gain and 72-hour gap-position diagnostics. |
| `10_cross_tier_comparison.ipynb` | Combines frozen Test outputs from all three tiers into a unified comparison. Generates the cross-tier MAE heatmap, family-balanced rankings, station-level diagnostics, gap-position profiles and accuracy–cost Pareto analysis. |
| `regenerate_heatmaps.ipynb` | Regenerates six heatmap figures with improved colour palettes and adaptive text contrast in response to supervisor feedback. Reads saved outputs only; no data values are altered. |

## Data summary

| Item | Value |
|---|---|
| Source | UK-AIR Automatic Urban and Rural Network (AURN) |
| Variables | Hourly PM2.5 (µg/m³), UK-AIR Modelled Temperature (°C) |
| Study period | 2021–2025 |
| Stations | London Bexley, London Harlington, London Honor Oak Park, London Marylebone Road, London N. Kensington |
| Total station-hour rows | 219,120 |
| Chronological split | 2021–2023 Train, 2024 Validation, 2025 Test |
| PM2.5 data capture | 97.67% |

## Artificial masking scenarios

Five artificial masking scenarios are applied to originally observed PM2.5 values in the Validation and Test periods. The Train period is not artificially masked.

| Scenario | Description |
|---|---|
| `random10_point` | Random point masking at 10% intensity |
| `random30_point` | Random point masking at 30% intensity |
| `random50_point` | Random point masking at 50% intensity |
| `block24_10percent` | 24-hour contiguous block masking at approximately 10% intensity |
| `block72_10percent` | 72-hour contiguous block masking at approximately 10% intensity |

The three random-point scenarios are nested (10% ⊂ 30% ⊂ 50%). Block scenarios place non-adjacent blocks within clean windows only. All masking uses a fixed random seed (42).

## Model configurations

Nine frozen model configurations are compared across all five scenarios:

| Tier | Configuration | Spatial setting |
|---|---|---|
| Statistical | Station train mean | Non-spatial |
| Statistical | Linear interpolation | Non-spatial |
| Statistical | Kalman-ARIMA | Non-spatial |
| Machine Learning | Random Forest non-spatial | Non-spatial |
| Machine Learning | Random Forest spatio-temporal | Spatio-temporal |
| Machine Learning | XGBoost non-spatial | Non-spatial |
| Machine Learning | XGBoost spatio-temporal | Spatio-temporal |
| Deep Learning | BiGRU zero-neighbour control | Non-spatial |
| Deep Learning | Graph-enhanced BiGRU | Spatio-temporal |

## Software environment

| Component | Version |
|---|---|
| Python | 3.11.15 |
| PyTorch | 2.11.0 (CUDA 12.8) |
| XGBoost | 3.2.0 |
| scikit-learn | (recorded in notebook manifests) |
| statsmodels | (recorded in notebook manifests) |
| GPU | NVIDIA GTX 1660 Ti (6 GB) |
| OS | Windows 10 (build 19045) |

Package versions were recorded programmatically in per-notebook environment manifests.

## Reproducibility

The notebooks should be run in order from NB01 to NB10 because later notebooks depend on outputs created in earlier stages. The leakage rule used throughout is:

- `PM25_original` is the ground-truth value used only for evaluation.
- `PM25_model_input` is the model-visible value used for imputation and feature construction.

Large raw data files and trained model checkpoints are excluded from version control via `.gitignore`. The complete dataset, all intermediate outputs and model checkpoints are deposited in the University secure shared space.
