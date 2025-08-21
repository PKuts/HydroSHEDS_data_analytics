HydroSHEDS DATA ANALYTICS and Catchment Analytics for Bhutan
Overview

This project analyzes HydroSHEDS rasters (DEM, DIR – D8 flow direction, ACC – flow accumulation) for the Bhutan + buffer region.
Convert GeoTIFFs to tabular form where needed, run EDA, delineate catchments (watersheds) from internal confluences, and compute key attributes for downstream ML.

This repository is part of an Omdena initiative on natural-hazard (flood/landslide) prediction for Bhutan—data analysis and preprocessing for machine learning.

Data source

HydroSHEDS official products (3″ ≈ 90 m): DEM, DIR (ESRI-D8), ACC.
Original rasters were downloaded from HydroSHEDS and clipped to Bhutan + buffer.

What the pipeline does

Streams from ACC – threshold ACC to create a binary stream mask (streams.tif).

Internal confluence pour points – for each stream cell, count upstream stream neighbors by ESRI-D8; cells with inflow ≥ 2 are candidates.

Filter & merge pour points – keep only candidates with ACC ≥ 120,000 cells, merge within 4 px (Chebyshev), keep max-ACC as representative.

Watershed delineation – WhiteboxTools Watershed assigns each pixel to the nearest pour point (watersheds_internal.tif).

Clip & polygonize – clip to the working grid; polygonize to SHP/GPKG.

Attributes & EDA – compute area (equal-area CRS), optional fraction inside Bhutan, export summaries, quick visualizations.

Key outputs

Rasters (in data/HydroSHEDS/bt_out/):

streams.tif

pp_internal_confluences.tif

watersheds_internal.tif

Vectors:

watersheds_internal.shp (+ sidecars)

watersheds_internal.gpkg (layer: watersheds)

Tables:

watersheds_internal_summary.csv (basic)

watersheds_internal_enriched.csv (ws_id, area_km2, centroid lon/lat, optional inside stats)

Plots (in bt_out/plots/):

map_watersheds_full.png – basin map

hist_watershed_areas.png – area distribution

bar_top20_areas.png – top-20 basin areas

Repository layout (suggested)
code/
  Catchments .ipynb            # main notebook: pipeline + EDA  
  HydroSHEDS.ipynb             # ACC-focused analysis
  HydroSHEDS_DEM.ipynb         # DEM-focused analysis 
data/
  HydroSHEDS/
    as_dem_3s.tif
    as_acc_3s.tif
    as_dif_3s.tif
    bt_out/
      streams.tif
      pp_internal_confluences.tif
      watersheds_internal.*          # tif/shp/prj/dbf/gpkg
      watersheds_internal_summary.csv
      watersheds_internal_enriched.csv
      plots/
        map_watersheds_full.png
        hist_watershed_areas.png
        bar_top20_areas.png


