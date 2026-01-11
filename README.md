<!-- @format -->

# Fire Detection Data Mining Project

A complete, reproducible pipeline to extract, clean, merge, and analyze multi-source geospatial and tabular data (climate, soil, elevation, land cover, fire records) to study and model fire occurrence in Algeria and Tunisia.

The repository contains data acquisition notebooks, preprocessing and integration workflows, exploratory analysis, supervised and unsupervised modeling experiments, and the LaTeX sources for the final report.

---

## Pipeline Overview

Extraction → Cleaning → Integration/Merging → Preprocessing (resampling, normalization, splits) → EDA → Modeling (supervised | unsupervised) → Reporting

```
[Data Extraction] -> [Cleaning] -> [EDA] -> [Merging] -> [Preprocessing]  -> [ML] -> [Report]
```

---

## Repository Structure

- Extracting_Data/
  - Notebooks to fetch and prepare raw datasets (e.g., Climate.ipynb, SoilExploitation.ipynb, extract_elevation.ipynb, extract_landcover\*.ipynb, Boundaries.ipynb).
- Data_Preprocessing/
  - Initial preprocessing per domain (elevation.ipynb, fire.ipynb, land_cover\*.ipynb). Includes notes/assumptions in `notes.txt`.
- Data_Cleaning/
  - Focused cleaning notebooks for land cover and grid alignment (land_cover\*.ipynb, land_cover_grid.ipynb).
- Exploratory_Analysis/
  - EDA notebooks per dataset and merged views (Climate_Analysis.ipynb, SoilAnalysis.ipynb, Fire_Visualisation.ipynb, Elevation_analysis.ipynb, Land-Cover_Analysis.ipynb, Climate-Soil_Analysis.ipynb).
- Data_Merging/
  - Notebooks to integrate datasets: landcover_ElevSoilClimate.ipynb, mergeElevationSoilClimate.ipynb, landcover_fire.ipynb, landcoverElevSoilClimate_fire.ipynb.
- Merged_Data_Preprocessing/
  - Final preprocessing on merged data: splitting/resampling/normalization.
  - split*resampling.ipynb; normalizing*\*.ipynb
  - Normalized_Data_split/
    - X_train_original.parquet, X_train_smote.parquet, X_train_tomek.parquet, X_train_smote_tomek.parquet
  - Resampled_Data_split/
    - original/, smote/, tomek/, smote_tomek/, val/, test/, and metadata `file_structure.pkl`
  - READ_ME.txt (local guide for this stage)
- Machine_Learning/
  - Supervised_Models/
    - DT/, KNN/, RF/ (notebooks/experiments, metrics, and plots)
  - UnSupervised_Models/
    - K_means/, DBSCAN/, CLARANS/ (clustering experiments and analyses)
- CleanedDatasets/, ExtractedDatasets/, PreprocessedDatasets/
  - Intermediate/final datasets at different pipeline stages.
- Rapport_TP/
  - LaTeX report sources
  - chapters/ (extraction, cleaning, description/visualization, integration/preprocessing, feature engineering, supervised, unsupervised)
  - images/ (generated figures for each analysis/model)
  - references/ (BibTeX file)
  - main.tex, cover_page.tex, Final Report.pdf
- utils/
  - Helper utilities (if referenced by notebooks)
- README_Extraction.md, README_Extraction.pdf
  - Documentation specific to the extraction step
- requirements.txt
  - Python dependencies

---

## Data Sources and Licenses

Add exact links and licensing for each source you used. Examples/placeholders below:

- Climate: (e.g., ERA5, CHIRPS) — variables: precipitation, temperature, etc. — time range: YYYY–YYYY — spatial resolution: XX km — License: TODO — Link: TODO
- Soil: (e.g., SoilGrids) — variables: texture, organic carbon, etc. — resolution: XX m — License: TODO — Link: TODO
- Elevation: (e.g., SRTM/DEM) — resolution: 30 m / 90 m — License: TODO — Link: TODO
- Land cover: (e.g., Copernicus, ESA CCI) — classes used and any reclassification mapping — License: TODO — Link: TODO
- Fire: (e.g., MODIS/VIIRS Active Fire) — variables: FRP, lat/lon, date/time — filtering criteria — License: TODO — Link: TODO
- Boundaries: (e.g., geoBoundaries/GADM) — CRS: see below — License: TODO — Link: TODO

Coordinate Reference System (CRS) and spatial extents:

- CRS used throughout (e.g., EPSG:4326 or a projected CRS): TODO
- Spatial coverage: Algeria and Tunisia; any buffer/clip applied when merging.

Data redistribution:

- If licenses prohibit redistribution, provide scripts/notebooks to download from source and instructions to reproduce derived products.

---

## Environment Setup

- Requirements: Python 3.9+ recommended.
- Create and activate a virtual environment (Windows):

```
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

- Jupyter (if not already installed via requirements):

```
pip install jupyterlab ipykernel
python -m ipykernel install --user --name fire-dm
```

- Geospatial stack on Windows (GDAL/GEOS/PROJ)
  - If you encounter installation issues for geopandas/fiona/pyproj/shapely, install prebuilt wheels or follow platform-specific guides.
  - Tip: Ensure PROJ and GDAL data paths are correctly detected; a fresh install via `pip install geopandas` on recent Python often works.

---

## Reproducing the Pipeline

Recommended order and expected outputs.

1. Extracting_Data/

- Run in order of dependencies: Boundaries.ipynb → Climate.ipynb → Soil*.ipynb → extract_elevation.ipynb → extract_landcover*.ipynb.
- Output: raw/intermediate files in ExtractedDatasets/.

2. Exploratory_Analysis/

- Visual sanity checks, distributions, correlations, and spatial maps.
- Figures are exported to Rapport_TP/images/ subfolders (Climate, Soil, Elevation, Fire, LandCover, etc.).

3. Data_Merging/

- Align grid/CRS, harmonize keys/indices.
- Notebooks: landcover_ElevSoilClimate.ipynb, mergeElevationSoilClimate.ipynb, landcover_fire.ipynb, landcoverElevSoilClimate_fire.ipynb.
- Output: merged feature tables in PreprocessedDatasets/ or Merged_Data_Preprocessing/.

4. Data_Preprocessing/ and Data_Cleaning/

- Clean per domain: handle missing values, align CRS, spatial joins, quality filtering, outlier treatment.
- Output: CleanedDatasets/.

1. Merged_Data_Preprocessing/

- split_resampling.ipynb: create train/val/test; generate resampling variants (original, smote, tomek, smote_tomek).
- normalizing\_\*.ipynb: apply scaling consistently across splits.
- Outputs:
  - Resampled_Data_split/ with subfolders: original/, smote/, tomek/, smote_tomek/, val/, test/; metadata in file_structure.pkl
  - Normalized_Data_split/ with artifacts such as X_train_original.parquet, X_train_smote.parquet, X_train_tomek.parquet, X_train_smote_tomek.parquet

6. Machine_Learning/

- Supervised_Models/ (DT, KNN, RF):
  - Use preprocessed splits from Merged_Data_Preprocessing/.
  - Evaluate with accuracy, F1, ROC AUC, PR curves; consider threshold tuning.
  - Plots and comparisons are saved into Rapport_TP/images/ under DT, KNN, RF folders.
- UnSupervised_Models/ (K_means, DBSCAN, CLARANS):
  - Use normalized features; assess with silhouette/Davies–Bouldin; visualize via PCA.
  - Output plots under Rapport_TP/images/ (Kmeans, DBscan, Clarans).

7. Reporting

- Build the LaTeX report in Rapport_TP/ to regenerate Final Report.pdf:

```
cd Rapport_TP
pdflatex -interaction=nonstopmode main.tex
bibtex main
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex
```

Tip (Windows): install a LaTeX distribution such as MiKTeX if not already available.

---

## Quick Start

- Launch Jupyter Lab in the repository root and open the relevant notebooks:

```
jupyter lab
```

- Supervised model (example: KNN)

  - Open Machine_Learning/Supervised_Models/KNN/ (notebook).
  - Point inputs to Merged_Data_Preprocessing/Normalized_Data_split/ and the corresponding labels.
  - Train, validate, and inspect generated metrics and figures.

- Clustering (example: K-means)
  - Open Machine_Learning/UnSupervised_Models/K_means/ (notebook).
  - Use normalized features; adjust k; review silhouette/PCA plots.

Outputs are written to Rapport_TP/images/ and to the model-specific folders.

---

## Key Design Decisions

- CRS unification strategy and choice of working CRS: TODO
- Grid definition and alignment between rasters/vectors: TODO
- Missing data handling and imputation per domain: TODO
- Class definitions/labels for fire prediction and imbalance handling (SMOTE, Tomek): SMOTE/Tomek variants provided.
- Feature engineering (e.g., climate indices, terrain derivatives, land-cover reclassification): TODO
- Train/val/test split logic and leakage prevention: TODO
- Model selection and tuning approach (e.g., predefined vs Bayesian optimization): evidence in images and notebooks.

---

## Results Summary

Provide brief highlights and point to figures and the final report.

- Supervised models: best-performing algorithm(s) with key metrics (Accuracy, F1, ROC AUC). See plots in Rapport_TP/images/KNN, DT, RF and comparisons.
- Clustering insights: number of clusters, stability, and spatial patterns. See Rapport_TP/images/Kmeans and DBscan.
- Full details, discussion, and additional figures: Rapport_TP/Final Report.pdf.

---

## Known Issues and Troubleshooting

- Windows geospatial stack (GDAL/PROJ/GEOS) can be tricky; prefer recent Python and prebuilt wheels if needed.
- Coordinate system mismatches can cause empty joins; always confirm CRS before joins.
- Large raster/point operations may require chunking/tiling to reduce memory usage.
- Re-run normalization after resampling to avoid data leakage across splits.

---
