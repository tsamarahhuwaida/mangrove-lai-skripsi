# Assessing the Effect of Radiometric Normalization on Multitemporal Mangrove Leaf Area Index Estimation using PlanetScope Imagery

**Undergraduate Thesis (Skripsi) — Cartography and Remote Sensing**
Universitas Gadjah Mada · Faculty of Geography · 2025

---

## Overview

This repository contains the full computational pipeline supporting a study on the effect of radiometric normalization on multitemporal mangrove Leaf Area Index (LAI) estimation. The study implements **Relative Radiometric Normalization (RRN)** using the **Pseudo-Invariant Feature (PIF)** approach (Schott method) to correct inter-temporal radiometric inconsistency in PlanetScope imagery, followed by vegetation index extraction, **Random Forest Regression (RFR)** modeling with Bayesian hyperparameter optimization, and comparative analysis between normalized and non-normalized LAI estimation results.

**Study Area:** Coastal Management Center (CMC), East Java, Indonesia
**Temporal Coverage:** 2016–2025

---

## Research Objectives

The primary objective of this study is to assess the effect of PIF-based radiometric normalization on monitoring LAI change dynamics of the mangrove ecosystem at CMC conservation area from 2016 to 2025. To address this primary objective, the following sub-objectives are proposed:

1. Implement the RRN method with a PIF approach to correct radiometric inconsistency in multitemporal PlanetScope imagery in support of mangrove LAI modeling at CMC
2. Develop a machine learning-based LAI estimation model using the Random Forest Regression (RFR) algorithm to obtain representative LAI predictions for the mangrove ecosystem at CMC conservation area
3. Compare the effectiveness of multitemporal mangrove LAI change estimation (2016–2025) at CMC between PIF-normalized imagery and non-normalized imagery

---

## Methodology

```
Raw Imagery (DN)
      │
      ▼
[1] DN → Float32 Conversion (÷10000)
      │
      ▼
[2] Mangrove Masking (Shapefile)
      │
      ├──── Uncorrected Path ─────────────────────────────────────────────┐
      │                                                                   │
      ▼                                                                   │
[3a] Vegetation Index Calculation                                         │
     (NDVI, EVI, SAVI, ARVI)                                             │
      │                                                                   │
      ▼                                                                   │
[3b] PIF Radiometric Correction (Schott Method)  ◄── Corrected Path ────┘
     - PIF Candidate Selection (CV threshold)
     - Band-wise Linear Regression (subject → reference)
      │
      ▼
[4]  Layer Stacking (4 spectral bands + 4 VIs → 8-band composite)
      │
      ▼
[5]  NDWI-based Water Masking
      │
      ▼
[6]  Random Forest Regression — LAI Prediction
     (Bayesian Hyperparameter Optimization)
      │
      ▼
[7]  LAI Classification (6 classes)
      │
      ▼
[8]  Hotspot Analysis (Getis-Ord Gi*)
```

---

## Repository Structure

```
mangrove-lai-skripsi/
│
├── README.md
├── .gitignore
│
└── notebooks/
    │
    ├── 01_uncorrected_imagery/
    │   └── batch_processing_uncorrected.ipynb
    │       Pipeline batch untuk citra tanpa koreksi radiometrik.
    │       Tahap: Float → Masking Mangrove → VI → Layerstack →
    │              NDWI Masking → RF LAI Prediction
    │
    ├── 02_rfr_model/
    │   └── rfr_bayesian_hyperparameter_tuning.ipynb
    │       Pembangunan model Random Forest Regression.
    │       Tahap: Data Loading → Feature Selection → Bayesian
    │              Optimization → Training/Validation/Test Evaluation →
    │              LAI Classification → Model Export (.pkl)
    │
    └── 03_corrected_pif/
        └── pif_schott_pipeline.ipynb
            Pipeline batch end-to-end dengan koreksi PIF Schott.
            Tahap: PIF Selection → Radiometric Correction → VI →
                   Masking → RF LAI → Classification → Hotspot Analysis
```

---

## Notebooks Description

### `01_uncorrected_imagery` — Batch Processing Uncorrected Imagery

Processes raw PlanetScope imagery (2016–2025) without radiometric correction. Serves as the **baseline** for comparison against PIF-corrected outputs.

**Input:** Raw `.tif` imagery (integer DN), mangrove shapefile, NDWI mask, RF model `.pkl`
**Output:** Float32 rasters, masked imagery, vegetation index rasters, layerstacks, LAI prediction maps

**Vegetation Indices Computed:**
| Index | Formula |
|---|---|
| NDVI | (NIR − Red) / (NIR + Red) |
| EVI | 2.5 × (NIR − Red) / (NIR + 6×Red − 7.5×Blue + 1) |
| SAVI | 1.5 × (NIR − Red) / (NIR + Red + 0.5) |
| ARVI | (NIR − (2×Red − Blue)) / (NIR + (2×Red − Blue)) |

---

### `02_rfr_model` — Random Forest Regression with Bayesian Optimization

Builds and evaluates the LAI prediction model from field-measured LAI and spectral sample data.

**Input:** CSV sample dataset (`LAI 4 band 4 VI_Lengkap_Repaired_fix.csv`)
**Output:** Trained model (`rf_lai_final_model.pkl`), selected features (`selected_features.pkl`), evaluation plots

**Hyperparameter Search Space (Bayesian Optimization):**
| Parameter | Range |
|---|---|
| `n_estimators` | 100 – 2000 |
| `max_depth` | 2 – 15 |
| `min_samples_split` | 2 – 30 |
| `min_samples_leaf` | 1 – 15 |
| `max_features` | 0.3 – 0.8 |

**Split:** Training / Validation / Test (label-based stratification)

---

### `03_corrected_pif` — PIF Schott Correction Pipeline (End-to-End)

Full processing pipeline incorporating PIF-based radiometric correction before LAI prediction. Uses the same RF model as the uncorrected pipeline, enabling direct comparison.

**Input:** Float32 imagery, PIF polygon shapefile (4 candidates), Z-profile shapefile, mangrove mask, NDWI mask, RF model `.pkl`
**Output:** Corrected rasters, Z-profile statistics, scatter plots, LAI maps, classified LAI maps, hotspot analysis outputs

**PIF Correction (Schott Method):**
Subject image band values are linearly regressed onto reference image band values at PIF locations. The resulting regression coefficients are applied pixel-wise to normalize inter-date radiometric differences.

---

## Dependencies

```
Python >= 3.9
numpy
pandas
geopandas
rasterio
scikit-learn
scipy
matplotlib
seaborn
joblib
```

All notebooks are designed to run on **Google Colab** with data stored on **Google Drive**.

---

## Notes

- All input imagery must be converted to `float32` (DN ÷ 10000) prior to processing
- NDWI water masks are applied from an external pre-computed folder
- The RF model (`.pkl`) is shared across both the uncorrected and corrected pipelines to ensure comparability
- Output directories are created automatically; no manual folder creation required

---

## Author

**Tsamarah Huwaida**
Cartography and Remote Sensing — Universitas Gadjah Mada
Faculty of Geography · Class of 2022
