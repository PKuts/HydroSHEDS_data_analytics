# HydroSHEDS Data Analytics & Catchment Analytics for Bhutan

> End-to-end analytics on HydroSHEDS rasters (DEM, DIR—ESRI D8 flow direction, ACC—flow accumulation) for Bhutan + buffer: EDA, watershed delineation, pour-point generation.

---

## Overview

This project analyzes HydroSHEDS rasters for the Bhutan + buffer region.  
Сonvert GeoTIFFs when needed, run EDA, delineate catchments (watersheds) from internal confluences, and compute key attributes for downstream machine learning.

This repository is part of an Omdena initiative on natural-hazard (flood/landslide) prediction for Bhutan — data analysis and preprocessing for ML.

---

## Data Source

- **HydroSHEDS official products** (3″ ≈ 90 m): **DEM**, **DIR** (ESRI-D8), **ACC**.  
- Original rasters were downloaded from HydroSHEDS and **clipped to Bhutan + buffer**.  
- Website: <https://www.hydrosheds.org>

> ⚠️ Large rasters and intermediate data are **not stored** in the repo. Use the paths below and keep raw data out of Git.

---

## What the pipeline does

1. **Streams from ACC** — threshold ACC to create a binary stream mask (`streams.tif`).  
2. **Internal confluence pour points** — for each stream cell, count upstream stream neighbors (ESRI-D8); cells with inflow ≥ 2 are candidates.  
3. **Filter & merge pour points** — keep only candidates with **ACC ≥ 120,000** cells; merge candidates within **4 px (Chebyshev)** and keep the max-ACC representative.  
4. **Catchment delineation** — derive local catchments per pour point and compute summary attributes (area, elevation stats, etc.).  

---

## Parameters (defaults)

| Name                    | Default  | Meaning                                                                 |
|-------------------------|----------|-------------------------------------------------------------------------|
| `STREAM_THRESHOLD_CELLS`| 30,000   | ACC threshold to define streams (cells).                                |
| `MIN_CONFLUENCE_ACC`    | 120,000  | Minimum ACC at a confluence to keep it as a pour point.                 |
| `MERGE_RADIUS_PX`       | 4        | Merge pour points within this Chebyshev (L∞) radius (pixels).           |

> Rule of thumb: at 3″ (~90 m), 1 cell ≈ 0.0081 km².  
> 30k cells ≈ 243 km²; 120k cells ≈ 972 km² contributing area.

---

## Repository structure

code/
Catchments.ipynb # main pipeline: streams → pour points → watersheds → vectors/CSV/plots
HydroSHEDS.ipynb # ACC conversion & EDA
HydroSHEDS_DEM.ipynb # DEM conversion & EDA
.gitignore
README.md

data/
└─ HydroSHEDS/
├─ as_dir_Bhutan_and_buffer.tif # ESRI-D8 flow directions — download from https://www.hydrosheds.org/
├─ as_acc_Bhutan_and_buffer.tif # Flow accumulation (cells) — download from https://www.hydrosheds.org/
├─ as_dem_Bhutan_and_buffer.tif # DEM (elevation) — download from https://www.hydrosheds.org/
└─ bt_out/
├─ streams.tif # Binary stream mask from ACC threshold
├─ pp_internal_confluences.tif # Pour points (internal confluences), unique IDs
├─ watersheds_internal.tif # Watershed (basin) ID raster
├─ watersheds_internal.shp/.dbf/.shx/.prj … # Basins vectorized to Shapefile
├─ watersheds_internal.gpkg # Basins vectorized to GeoPackage (layer: watersheds)
├─ watersheds_internal_summary.csv # Basic per-basin summary
├─ watersheds_internal_enriched.csv # Enriched per-basin table (area, centroid, etc.)
├─ csv/ # Raster-to-CSV exports
│ ├─ dem_Bhutan_and_buffer.csv # row,col,x,y,elev_m
│ ├─ dir_Bhutan_and_buffer.csv # row,col,x,y,dir_code,dir_name
│ └─ acc_Bhutan_and_buffer.csv # row,col,x,y,acc_cells
└─ plots/
├─ map_watersheds_full.png # Map of all basins
├─ hist_watershed_areas.png # Histogram of basin areas
└─ bar_top20_areas.png # Top-20 basin areas

---

## Setup & How to Run

### 1) Create a virtual environment
```bash
python3 -m venv .venv
# Linux/macOS
source .venv/bin/activate
# Windows (PowerShell)
# .venv\Scripts\Activate.ps1
2) Install libraries (Python 3.8–3.11)

pip install -U numpy pandas rasterio shapely matplotlib geopandas pyogrio whitebox scipy
Tips

Prefer fast I/O in GeoPandas:

Linux/macOS: export GEOPANDAS_IO_ENGINE=pyogrio

Windows (PowerShell): $env:GEOPANDAS_IO_ENGINE="pyogrio"

whitebox will auto-download the whitebox_tools binary on first use.

scipy is optional but speeds up pour-point merging.

3) Create data folder

mkdir -p data/HydroSHEDS/

4) Download HydroSHEDS rasters
Download Asia 3″ products, save as:
data/HydroSHEDS/
├─ as_dir_Bhutan_and_buffer.tif
├─ as_acc_Bhutan_and_buffer.tif
└─ as_dem_Bhutan_and_buffer.tif
(Optional) If you have a Bhutan boundary GeoPackage (bhutan_boundary.gpkg, layer bhutan), the pipeline will compute inside/edge stats and an inside-only map; otherwise these steps are skipped automatically.

5) Run the notebooks
code/Catchments.ipynb — main pipeline (streams → confluences → pour points → watersheds → vectors/CSV/plots)

code/HydroSHEDS.ipynb — ACC conversion & EDA
code/HydroSHEDS_DEM.ipynb — DEM conversion & EDA

Key outputs land in data/HydroSHEDS/bt_out/ as listed above.

Troubleshooting
If GeoPandas tries to use Fiona and errors, force Pyogrio:
Linux/macOS: export GEOPANDAS_IO_ENGINE=pyogrio
Windows (PowerShell): $env:GEOPANDAS_IO_ENGINE="pyogrio"

On Python 3.8, the notebooks already shim importlib_resources so whitebox works.

Known limits
Internal-confluence strategy emphasizes major junctions; very small headwater basins may be filtered out by ACC thresholds.

HydroSHEDS (3″) resolution implies ~90 m pixels; pixel-based area approximations assume near-equatorial cell size ( use equal-area reprojection for polygon areas).

Reproducibility
Tunable parameters are listed in the notebooks (see the Parameters table above).
If you change thresholds, expect a different number of pour points and basins.

License
Released under the MIT License — feel free to use, copy, modify, and distribute.

Questions & Contributions
Got a fix or a cool idea? PRs are super welcome!
Open an issue if you have questions, or start a discussion.