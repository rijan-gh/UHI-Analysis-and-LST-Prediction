# Urban Heat Island Analysis and Land Surface Temperature Prediction
### Using Machine Learning and Remote Sensing Data
#### A Comparative Study of Kathmandu Valley and Hetauda

---

## Project Overview

This repository contains the complete research code, data, and results for a Computer Engineering college minor project studying **Urban Heat Islands (UHI)** and **Land Surface Temperature (LST)** dynamics across two contrasting urban environments in Nepal:

| Study Area | Character |
|---|---|
| **Kathmandu Valley** | Dense, established urban core |
| **Hetauda** | Rapidly urbanising mid-hill city |

**Study Period:** 2015–2025 (annual pre-monsoon observations, March–May)
**Spatial Resolution:** 500 m × 500 m grid

---

## Repository Structure

```
uhi-lst-nepal/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   |
│   ├── kathmandu/
│   │   ├── Kathmandu_Dataset.csv
│   │   ├── csv/                ← LULC_Area_Stats + LULC_LST_Relationship CSVs
│   │   ├── lst_tifs/           ← LST GeoTIFFs (gitignored — large)
│   │   ├── lulc_tifs/          ← LULC GeoTIFFs (gitignored)
│   │   └── uhi_tifs/           ← UHI hotspot GeoTIFFs (gitignored)
│   └── hetauda/
│       └── (same structure)
│
│
├── 01_lst_prediction/
│   ├── LST_prediction_kathmandu.ipynb
│   ├── LST_prediction_hetauda.ipynb
│   └── results/
│       ├── figures/
│       ├── tables/
│       └── models/                 ← trained .pkl models (gitignored)
│
├── 02_lulc_uhi_analysis/
│   ├── lulc_uhi_analysis.ipynb
│   └── results/
│       ├── figures/
│       └── tables/
│
├── 03_timeseries_forecasting/
│   ├── time_series_analysis.ipynb
│   └── results/
│       ├── figures/
│       └── tables/
│
├── 04_comparative_analysis/
│   ├── comparative_analysis.ipynb
│   └── results/
│       ├── figures/
│       └── tables/
│
├── gee/
│   └── scripts/                    ← Google Earth Engine scripts
│       ├── 01_lst_export.js
│       ├── 02_uhi_classification.js
│       └── 03_lulc_extraction.js
│
└── docs/
    ├── report/                     ← Final project report
    |-- presentation/           ← Presentation slides
    └── validation_notes.md         ← Data validation and methodology notes

```

---

## Analysis Modules

### Module 1 — LST Prediction (Machine Learning)
**Notebooks:** `01_lst_prediction/code/`

Predicts Land Surface Temperature using supervised machine learning:
- **Models:** Random Forest, XGBoost (baseline + Optuna-tuned)
- **Evaluation:** RMSE, MAE, R² with 5-fold GroupKFold cross-validation (grouped by `Grid_ID`)
- **Features:** NDVI, NDBI, MNDWI, BSI, Albedo, Elevation, Slope, Aspect, Population Density, land-cover fractions, focal neighborhood indices, Latitude, Longitude, Year

### Module 2 — LULC Change & UHI Analysis
**Notebook:** `02_lulc_uhi_analysis/code/`

Analyses land use / land cover dynamics and their relationship with UHI:
- LULC area trend analysis (2015–2025)
- LULC transition matrices
- LST-by-LULC-class analysis
- LST difference maps (2015 → 2025)
- **Surface Urban Heat Island Intensity (SUHII):**
  `SUHII = Mean(Urban LST) − Mean(Vegetation + Agriculture LST)`
- UHI hotspot zone transition matrices

### Module 3 — Time-Series Trend Analysis & Forecasting
**Notebook:** `03_timeseries_forecasting/code/`

Analyses historical LST trends (2015–2025) and forecasts future values:
- **Mann-Kendall** trend test
- **Sen's Slope** estimator
- **Rolling-origin cross-validation** for back-testing
- **Exponential Smoothing** (Holt's method) as a comparative baseline

### Module 4 — Comparative Analysis
**Notebook:** `04_comparative_analysis/comparative_analysis.ipynb`

Comparative analysis between Kathmandu Valley and Hetauda:
- **Status:** Fully implemented and verified. Covers SUHII comparison, LULC conversion comparison, 
  LST trend/forecast comparison, and feature importance comparison. 
- **Outputs:** Cross-city comparison plots and tables in `results/figures/` and `results/tables/`

---

## Data Sources

All data were extracted and pre-processed using **Google Earth Engine (GEE)**:

| Dataset | Source | Variables |
|---|---|---|
| Landsat 8/9 | USGS via GEE | LST, NDVI, NDBI, MNDWI, BSI, Albedo |
| SRTM DEM | NASA via GEE | Elevation, Slope, Aspect |
| WorldPop | WorldPop via GEE | Population_Density |
| ESA WorldCover / Dynamic World | GEE | LULC classes, land-cover fractions |

---

## Setup & Usage

### 1. Install dependencies
```bash
pip install -r requirements.txt
```

### 2. Obtain the datasets
The raw CSV and GeoTIFF files are large and excluded from Git. Place them in:
```
data/raw/kathmandu/Kathmandu_Dataset.csv
data/raw/kathmandu/lst_tifs/
data/raw/kathmandu/lulc_tifs/
data/raw/kathmandu/uhi_tifs/
data/raw/hetauda/  (same structure)

data/kathmandu/csv/LULC_Area_Stats_Kathmandu.csv
data/kathmandu/csv/LULC_LST_Relationship_Kathmandu.csv
data/hetauda/csv/  (same structure)
```

> Contact the project team or see `docs/` for data access instructions.

### 3. Run notebooks in order
Notebooks sit directly inside each numbered stage folder (no `code/` subfolder). Open Jupyter and run in this sequence:
1. `01_lst_prediction/LST_prediction_kathmandu.ipynb`
2. `01_lst_prediction/LST_prediction_hetauda.ipynb`
3. `02_lulc_uhi_analysis/lulc_uhi_analysis.ipynb`
4. `03_timeseries_forecasting/time_series_analysis.ipynb`
5. `04_comparative_analysis/comparative_analysis.ipynb`

---

## Key Results

| Metric | Kathmandu | Hetauda |
|---|---|---|
| Best model | XGBoost Tuned (RMSE 1.57°C, MAE 1.18°C, R² 0.85) | XGBoost Tuned (RMSE 1.48°C, MAE 1.13°C, R² 0.82) |
| SUHII 2015 → 2025 | 3.85°C → 4.95°C (peaked 5.28°C in 2020) | 1.81°C → 2.76°C (peaked 3.27°C in 2020) |
| LST trend (Sen's slope) | +0.46°C/year (p=0.005, significant) | +0.41°C/year (p=0.020, significant) |
| Forecast CV accuracy | MAE 0.90°C, RMSE 1.01°C | MAE 1.05°C, RMSE 1.17°C |
| Land converted to Builtup (2015-2025) | 207.0 km² | 31.3 km² |
| Dominant LST drivers | Distributed (NDBI/Albedo/NDVI) | Albedo/MNDWI (dominant) |
| Figures | `02_lulc_uhi_analysis/results/figures/` | `02_lulc_uhi_analysis/results/figures/` |
| SUHII results table | `02_lulc_uhi_analysis/results/tables/SUHII_results.csv` | (same file, both cities) |
| Forecast trend | `03_timeseries_forecasting/` | `03_timeseries_forecasting/` |
| Comparative analysis | `04_comparative_analysis/results/` | `04_comparative_analysis/results/` |

---

## Overall Findings

The comparative analysis reveals that **Kathmandu shows a stronger, more established, and more statistically confident UHI signal across every metric** — consistent with it being the more mature, denser urban area. **Hetauda shows a weaker but still real and significant UHI signal**, with a distinct land-cover-driven mechanism (Albedo/MNDWI) rather than Kathmandu's more distributed built-up-intensity mechanism.

This supports the project's core comparative premise: the two cities represent **different stages of urbanization with measurably different UHI dynamics**, not just a scaled-down version of the same pattern.

---

## Team

Computer Engineering Minor Research Project
Kathmandu, Nepal — 2025
