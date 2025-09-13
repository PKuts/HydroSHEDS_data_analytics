# HydroSHEDS Data Analytics & Catchment Analytics for Bhutan

End-to-end analytics on HydroSHEDS rasters (DEM, DIR—ESRI D8 flow direction, ACC—flow accumulation) for Bhutan + buffer: EDA, watershed delineation, pour-point generation. Calculation of DEM and ACC statistics based on 186 watersheds and 10 basins, including slope and relief calculation

---

## Overview

This project analyzes HydroSHEDS rasters for the Bhutan + buffer region. The goal is to convert GeoTIFFs as needed, run exploratory data analysis (EDA), delineate catchments (watersheds) from internal confluences, and compute key attributes for downstream machine learning.

This repository is part of an Omdena initiative focused on natural-hazard (flood/landslide) prediction for Bhutan, with an emphasis on data analysis and preprocessing for ML.

---

## Data Source

- HydroSHEDS official products (3″ ≈ 90 m): DEM, DIR (ESRI-D8), ACC.
- Original rasters were downloaded from HydroSHEDS and clipped to Bhutan + buffer.
- Website: <https://www.hydrosheds.org>

> ⚠️ Note: Large rasters and intermediate data are not stored in this repository. Use the specified paths and keep raw data out of Git.

---

## What the Pipeline Does

The pipeline performs the following steps:

- **Streams from ACC:** Threshold the flow accumulation raster (ACC) to create a binary stream mask (`streams.tif`).
- **Internal Confluence Pour Points:** For each stream cell, count upstream stream neighbors using the ESRI-D8 flow direction. Cells with an inflow of 2 or more are considered candidates.
- **Filter & Merge Pour Points:** Keep only candidates with an ACC ≥ 120,000 cells. Merge candidates within a 4-pixel Chebyshev radius and keep the representative with the maximum ACC.
- **Catchment Delineation:** Derive local catchments for each pour point and compute summary attributes (area, elevation statistics, etc.).

---

## Parameters (Defaults)

| Name                   | Default | Meaning                                                           |
|------------------------|:-------:|-------------------------------------------------------------------|
| STREAM_THRESHOLD_CELLS | 30,000  | The ACC threshold used to define streams (in cells).              |
| MIN_CONFLUENCE_ACC     | 120,000 | The minimum ACC at a confluence to be kept as a pour point.       |
| MERGE_RADIUS_PX        |   4     | The Chebyshev (L_infty) radius (in pixels) for merging pour points.|

Export to Sheets

> **Rule of Thumb:** At a 3″ (~90 m) resolution, 1 cell is approximately equal to 0.0081 km². Therefore, 30,000 cells ≈ 243 km² and 120,000 cells ≈ 972 km² of contributing area.

---

## Repository Structure

To ensure the structure is displayed correctly, I'm providing it as a code block. This is a reliable method that GitHub's Markdown always renders properly.

```text
repo-root/
├─ code/
│  ├─ Catchments.ipynb
│  ├─ HydroSHEDS.ipynb
│  ├─ HydroSHEDS_DEM.ipynb
│  ├─ Quicklook_aux_layers_bhutan.ipynb
│  └─ 186_Watershed_boundary/
│     ├─ 186_watersheds_dem_analysis.ipynb
│     └─ calc_watershed_slope_relief.ipynb
│
├─ data/
│  ├─ HydroSHEDS/
│  │  ├─ as_dir_Bhutan_and_buffer.tif
│  │  ├─ as_acc_Bhutan_and_buffer.tif
│  │  ├─ as_dem_Bhutan_and_buffer.tif
│  │  ├─ as_con_3s.tif
│  │  ├─ as_msk_3s.tif
│  │  ├─ hyd_as_lup_15s.tif
│  │  └─ bt_out/
│  │     ├─ streams.tif
│  │     ├─ pp_internal_confluences.tif
│  │     ├─ watersheds_internal.tif
│  │     ├─ watersheds_internal.{shp,dbf,shx,prj}
│  │     ├─ watersheds_internal.gpkg
│  │     ├─ watersheds_internal_summary.csv
│  │     ├─ watersheds_internal_enriched.csv
│  │     ├─ csv/
│  │     │  ├─ dem_Bhutan_and_buffer.csv
│  │     │  ├─ dir_Bhutan_and_buffer.csv
│  │     │  └─ acc_Bhutan_and_buffer.csv
│  │     └─ plots/
│  │        ├─ map_watersheds_full.png
│  │        ├─ hist_watershed_areas.png
│  │        └─ bar_top20_areas.png
│  │
│  ├─ 10_basins/
│  │  ├─ Basin boundary.cpg
│  │  ├─ Basin boundary.dbf
│  │  ├─ Basin boundary.prj
│  │  ├─ Basin boundary.sbn
│  │  ├─ Basin boundary.sbx
│  │  ├─ Basin boundary.shp
│  │  ├─ Basin boundary.shp.xml
│  │  ├─ Basin boundary.shx
│  │  └─ basin_dem_acc_stats.csv
│  │
│  └─ 186_Watershed_boundary/
│     ├─ 186 Watershed boundary.cpg
│     ├─ 186 Watershed boundary.dbf
│     ├─ 186 Watershed boundary.prj
│     ├─ 186 Watershed boundary.sbn
│     ├─ 186 Watershed boundary.sbx
│     ├─ 186 Watershed boundary.shp
│     ├─ 186 Watershed boundary.shp.xml
│     ├─ 186 Watershed boundary.shx
│     └─ watershed_dem_acc_stats.csv
│
├─ .gitignore
└─ README.md
```

## Setup & How to Run

Create a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install libraries (Python 3.8–3.11)

```bash
pip install -U numpy pandas rasterio shapely matplotlib geopandas pyogrio whitebox scipy
```

Tips

Prefer fast I/O in GeoPandas:

```bash
export GEOPANDAS_IO_ENGINE=pyogrio
```

whitebox will auto-download the whitebox_tools binary on first use.
scipy is optional but speeds up pour-point merging.

Create data folders

```bash
mkdir -p data/HydroSHEDS/bt_out/{plots,csv}
```

Download HydroSHEDS rasters

Download the Asia 3″ products and save them to:

```text
data/HydroSHEDS/
├── as_dir_Bhutan_and_buffer.tif   # ESRI-D8 flow directions — https://www.hydrosheds.org/
├── as_acc_Bhutan_and_buffer.tif   # Flow accumulation (cells) — https://www.hydrosheds.org/
├── as_dem_Bhutan_and_buffer.tif   # DEM (elevation) — https://www.hydrosheds.org/
├── as_con_3s.tif                  # Flow connectivity mask (3 arc-seconds) — https://www.hydrosheds.org/
├── as_msk_3s.tif                  # Land/sea mask (3 arc-seconds) — https://www.hydrosheds.org/
└── hyd_as_lup_15s.tif             # HydroSHEDS LUP layer (15 arc-seconds) — https://www.hydrosheds.org/
```

Download the "186 Watershed boundary" files from [NSDI Bhutan](https://nsdi.systems.gov.bt/data/Boundaries)

```text
Download the "186 Watershed boundary" and "10 Basin boundary" files from [NSDI Bhutan](https://nsdi.systems.gov.bt/data/Boundaries)

data/186_Watershed_boundary/
├── 186 Watershed boundary.shp      # main shapefile geometry (polygons of 186 watersheds)
├── 186 Watershed boundary.dbf      # attribute table (IDs and other fields linked to the shapefile)
├── 186 Watershed boundary.prj      # projection/coordinate reference system information
├── 186 Watershed boundary.shx      # index file for shapefile geometry
├── 186 Watershed boundary.cpg      # character encoding information for the DBF
├── 186 Watershed boundary.sbn      # spatial index file
├── 186 Watershed boundary.sbx      # spatial index file (auxiliary)
├── 186 Watershed boundary.shp.xml  # metadata description (XML format, NSDI Bhutan)
└── watershed_dem_acc_stats.csv     # zonal statistics (DEM & ACC per watershed), generated by `186_watersheds_dem_analysis.ipynb` and calc_watershed_slope_relief.ipynb

data/10_basins/
├── Basin boundary.shp              # main shapefile geometry (polygons of 10 major basins)
├── Basin boundary.dbf              # attribute table (IDs and basin attributes linked to the shapefile)
├── Basin boundary.prj              # projection/coordinate reference system information
├── Basin boundary.shx              # index file for shapefile geometry
├── Basin boundary.cpg              # character encoding information for the DBF
├── Basin boundary.sbn              # spatial index file
├── Basin boundary.sbx              # spatial index file (auxiliary)
├── Basin boundary.shp.xml          # metadata description (XML format, NSDI Bhutan)
└── basin_dem_acc_stats.csv         # zonal statistics (DEM & ACC per basin), generated by `186_watersheds_dem_analysis.ipynb` and calc_watershed_slope_relief.ipynb
```


Optional: If you have a Bhutan boundary GeoPackage (bhutan_boundary.gpkg, layer bhutan), the pipeline will compute inside/edge stats and an inside-only map. Otherwise, these steps are skipped automatically.

Run the notebooks
code/HydroSHEDS/Catchments.ipynb — main pipeline (streams → confluences → pour points → watersheds → vectors/CSV/plots).

code/HydroSHEDS/HydroSHEDS.ipynb — ACC conversion & EDA.

code/HydroSHEDS/HydroSHEDS_DEM.ipynb — DEM conversion & EDA.

code/HydroSHEDS/Quicklook_aux_layers_bhutan.ipynb — quicklook analysis of auxiliary HydroSHEDS layers (`as_con_3s.tif`, `as_msk_3s.tif`, `hyd_as_lup_15s.tif`).

code/186_Watershed_boundary/186_watersheds_dem_analysis.ipynb - aligned ACC and DEM to the shapefile resolution and calculated zonal statistics for each region

code/186_Watershed_boundary/calc_watershed_slope_relief.ipynb — calculation of slope and relief statistics (DEM-based) for both watershed (186) and basin (10) boundaries.

Key outputs will be saved to data/HydroSHEDS/bt_out/ (see Repository structure above).

Troubleshooting
If GeoPandas tries to use Fiona (and errors), force Pyogrio:

export GEOPANDAS_IO_ENGINE=pyogrio
On Python 3.8, the notebooks already shim importlib_resources so whitebox works.

Known Limits
The internal-confluence strategy emphasizes major junctions; very small headwater basins may be filtered out by the ACC thresholds.

HydroSHEDS 3″ (~90 m) resolution implies pixel areas vary with latitude; use equal-area reprojection for accurate polygon areas.

Reproducibility
Tunable parameters are listed in the notebooks (see the parameters table).

Changing thresholds will change the number of pour points and basins.

## License
Released under the MIT License — feel free to use, copy, modify, and distribute.

## Questions & Contributions
Got a fix or a cool idea? Pull requests are very welcome.
Have questions? Open an issue or start a discussion.
