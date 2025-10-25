# Data Extraction Guide — GeoBoundaries, Climate, and Soil (HWSD2)

This README explains **what to run, in which order, what folders to create, and what outputs to expect** for extracting **GeoBoundaries**, **Climate**, and **Soil** features for the Algeria–Tunisia Area of Interest (AOI).

> **Execution order:**  
> 1) `Code/Extracting Data/Boundaries.ipynb`  
> 2) `Code/Extracting Data/Climate.ipynb`  
> 3) `Code/Extracting Data/SoilNewDataset.ipynb`

---

## 1) Prerequisites

- **Python** 3.10+ (Anaconda/Miniconda recommended)
- Recommended packages:
  - `geopandas`, `shapely`, `pyproj`
  - `rasterio`, `rioxarray`, `xarray`
  - `numpy`, `pandas`
  - `matplotlib`
  - `pyodbc` (for reading `HWSD2.mdb` on Windows with Access ODBC Driver)
- **GDAL** runtime available (installed automatically with `rasterio` on most conda envs).  
- **Data available locally** (example layout below; adjust paths in notebooks if yours differ):
  ```text
  Datasets/
  ├─ GeoBoundaries/                     ← raw boundaries or country shapefiles (DZA, TUN)
  ├─ ClimateDataset/                    ← e.g., WorldClim or similar rasters (tmin/tmax/prec)
  └─ SoilDataset/
     ├─ HWSD2_RASTER/
     │  ├─ HWSD2.bil
     │  ├─ HWSD2.hdr
     │  └─ HWSD2.prj
     └─ HWSD2_DB/
        └─ HWSD2.mdb
  ```

---

## 2) Create Output Folders

Create these three folders **before** running the notebooks (Windows PowerShell example):
```powershell
mkdir ExtractedDatasets\GeoBoundaries
mkdir ExtractedDatasets\ClimateFeatures
mkdir ExtractedDatasets\SoilFeatures
```

> If they already exist, ensure they are empty or that you are okay with overwriting/adding new files.

---

## 3) Notebook 1 — `Boundaries.ipynb`

### Purpose
- Build a clean **Area of Interest (AOI)** for **Algeria + Tunisia**.
- (Optional) Generate a **regular analysis grid** (e.g., 0.1°) covering the AOI.
- Reproject to a suitable CRS when required.

### Expected Inputs
- Country boundaries (Algeria, Tunisia) from your `Datasets/GeoBoundaries` (GeoJSON, SHP, or GPKG).

### Key Parameters (typical)
- `GRID_RES_DEG = 0.1` (≈11 km) or `0.0833` (≈5 arc-min)
- Target CRS (if needed): `EPSG:4326` for lat/lon, or a projected CRS for area-based analyses.

### Outputs (written to `ExtractedDatasets/GeoBoundaries/AOI_DZA_TUN/`)
- `aoi_dza_tun.gpkg` — merged polygon of Algeria + Tunisia (primary AOI)
- `aoi_dza_tun.geojson` — same AOI in GeoJSON (handy for web/GIS tools)

> **Verification tip:** Load `aoi_dza_tun.gpkg` in QGIS and ensure the footprint matches both countries precisely (no multipolygon gaps).

---

## 4) Notebook 2 — `Climate.ipynb`

### Purpose
- Clip **climate rasters** to the AOI (and optionally to the grid).
- Extract **per-cell climate features** such as:
  - **tmin** (minimum temperature)
  - **tmax** (maximum temperature)
  - **prec** (precipitation)
- Optionally compute monthly or annual aggregates and **zonal stats** per grid cell.

### Expected Inputs
- Climate rasters in `Datasets/ClimateDataset` (e.g., WorldClim or similar).
- AOI and/or grid produced by **Notebook 1**.

### Key Parameters (typical)
- Variables: `tmin`, `tmax`, `prec`
- Temporal aggregation: monthly stacks → annual mean/sum (optional)
- Resampling: `nearest`/`bilinear` when aligning to grid resolution

### Outputs (written to `ExtractedDatasets/ClimateFeatures/`)
- `grid_0p1.gpkg` — analysis grid (0.1°) used in processing
- `tmin_2020_2024.csv` — extracted minimum temperature features
- `tmax_2020_2024.csv` — extracted maximum temperature features
- `prec_2020_2024.csv` — extracted precipitation features

> **Verification tip:** Plot a quick histogram/map to ensure ranges are realistic (e.g., `prec_annual_sum` not negative; `tmin`/`tmax` plausible for North Africa).

---

## 5) Notebook 3 — `SoilNewDataset.ipynb`

### Purpose
- Clip the **HWSD2 raster** to the AOI.
- Join **MU_GLOBAL** pixel codes to **soil attributes** stored in `HWSD2.mdb`.
- Extract **topsoil (D1: 0–20 cm)** properties for modeling.

### Expected Inputs
- `ExtractedDatasets/GeoBoundaries/AOI_DZA_TUN/aoi_dza_tun.gpkg`
- `Datasets/SoilDataset/HWSD2_RASTER/HWSD2.bil/.hdr/.prj`
- `Datasets/SoilDataset/HWSD2_DB/HWSD2.mdb`

### Key Parameters (typical)
- Layer depth: **D1 (0–20 cm)**. 
- Dominant component per mapping unit (highest percentage) when multiple components exist.

### Outputs (written to `ExtractedDatasets/SoilFeatures/`)
- `hwsd2_grid01_D1.csv` — topsoil (D1) tabular features per grid cell
- `hwsd2_grid01_D1.gpkg` — vectorized/grid-aligned features
- `hwsd2_grid01_debug.csv` — optional debug/export log of processed rows

> **Verification tip:** Display a few rasters to check units and value ranges (e.g., clay/silt/sand in %, pH around 4–9, OC mostly low in arid regions).

---

## 6) Suggested Order of Execution

1. **Boundaries.ipynb** — create AOI and grid
2. **Climate.ipynb** — extract tmin/tmax/prec for AOI/grid
3. **SoilNewDataset.ipynb** — extract D1 soil properties and (optionally) join to grid

If your modeling uses a grid, ensure **all outputs are aligned** (same CRS, extent, resolution, and transform).

---

## 7) Common Troubleshooting

- **CRS mismatch / alignment**: Reproject AOI/grid to match rasters (`EPSG:4326` for many global datasets). Use `rioxarray.rio.reproject_match()` to snap rasters to a template.
- **Access/ODBC errors (HWSD2.mdb)**: Install the **Microsoft Access Database Engine** (64‑bit) and enable the `Microsoft Access Driver (*.mdb, *.accdb)` ODBC driver.
- **NoData handling**: Verify `nodata` in `.hdr` and propagate it to outputs when masking/clipping.
- **Large rasters**: Use chunked reading or dask-backed `xarray` if memory is limited.
- **Value scaling**: Some datasets store scaled integers—confirm whether you need to apply a scale factor (check metadata).

---

## 8) Deliverables Summary

After running all three notebooks successfully, you should have:

```
ExtractedDatasets/
├─ GeoBoundaries/
│  └─ AOI_DZA_TUN/
│     ├─ aoi_dza_tun.gpkg
│     └─ aoi_dza_tun.geojson
├─ ClimateFeatures/
│  ├─ grid_0p1.gpkg
│  ├─ tmin_2020_2024.csv
│  ├─ tmax_2020_2024.csv
│  └─ prec_2020_2024.csv
└─ SoilFeatures/
  ├─ hwsd2_grid01_D1.csv
  ├─ hwsd2_grid01_D1.gpkg
  └─ hwsd2_grid01_debug.csv
```


