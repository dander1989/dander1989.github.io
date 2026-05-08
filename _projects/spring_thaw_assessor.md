---
title: "Spring Thaw: Snowmelt & Flood Vulnerability Assessor"
excerpt: "A spatial analysis project that identifies residential properties at highest risk of flooding during spring snowmelt by integrating FEMA flood zones, historical (April 12, 2024) snow water equivalent (SWE) data, and building footprints."
header:
  teaser: /assets/images/01_risk_by_building_1.png
layout: single
sidebar:
  nav: "main"
priority: 3
---

# Spring Thaw: Snowmelt & Flood Vulnerability Assessor
## Steamboat Springs, Colorado

A spatial analysis project that identifies residential properties at highest risk of flooding during spring snowmelt by integrating FEMA flood zones, historical (April 12, 2024) snow water equivalent (SWE) data, and building footprints.

**Live Interactive Map:**
View the web map here: 
https://dander1989.github.io/Spring-Thaw-Flood-Assessor/risk_map.html

---

## Problem Statement

Spring flooding isn't just about proximity to a river; rapid snowmelt is the primary catalyst. By enriching standard flood zone and building footprint data with historical snowpack levels, emergency managers can transition from generic "flood warnings" to pinpointing exactly which neighborhoods face the highest immediate risk from localized snowmelt.

This project demonstrates how open geospatial data and spatial analysis can support actionable emergency preparedness decisions.

---

## Project Outcome

**525 buildings** in Steamboat Springs' flood zones have been ranked by snowmelt flood vulnerability, enabling:
- Emergency managers to prioritize evacuation and preparation efforts
- City planners to identify high-risk neighborhoods for infrastructure investment
- Residents to understand their exposure to spring flood risk

**Risk Distribution:**
- Very High Risk: 217 buildings (41%)
- High Risk: 222 buildings (42%)
- Medium Risk: 57 buildings (11%)
- Low Risk: 29 buildings (6%)

---

## Data Sources

| Source        | Type                 | Resolution    | Coverage       | Notes                           |
| ------------- | -------------------- | ------------- | -------------- | ------------------------------- |
| NOAA SNODAS   | Raster (SWE)         | 1 km          | Continental US | April 12, 2024 snapshot         |
| OpenStreetMap | Vector (buildings)   | Feature-level | Global         | 5,794 buildings in study area   |
| FEMA NFHL     | Vector (flood zones) | Polygon       | United States  | Routt County, CO (2005 vintage) |

---

## Methodology

### 1. Study Area Definition
Steamboat Springs city boundary fetched from OpenStreetMap using `osmnx.geocode_to_gdf()`. A 0.35 km buffer was applied to capture complete raster grid cells at the boundary edge (prevents clipping artifacts).

### 2. Snowpack Data Processing
- Downloaded SNODAS binary data (April 12, 2024) from NOAA NSIDC
- Extracted tar archive and identified SWE file (code 1034)
- Converted binary to GeoTIFF using GDAL with proper geotransform
- Clipped to buffered boundary using Rasterio
- Applied scale factor (÷ 1000) to convert stored integers to meters
- Replaced negative values with 0 (model artifacts in high-elevation areas)

### 3. Vector Data Preparation
- Fetched OSM building footprints (all types, 5,794 total)
- Downloaded FEMA flood hazard zones for Routt County
- Filtered to high-risk zones (A, AE, AO designations per FEMA standards)
- Reprojected all datasets to EPSG:26913 (UTM Zone 13N, meters) for accurate distance calculations
- Reprojected back to EPSG:4326 for export and web visualization

### 4. Raster-to-Vector Grid Conversion
Converted clipped SNODAS raster to regular vector grid (168 cells, 1 km × 1 km each) with SWE values attached. Used nested loops with geotransform calculations to generate grid cell polygons:

```python
min_x = minx + (col * pixel_width)
max_x = min_x + pixel_width
max_y = maxy + (row * pixel_height)
min_y = max_y + pixel_height
polygon = Polygon([(min_x, max_y), (max_x, max_y), (max_x, min_y), (min_x, min_y)])
```

### 5. Spatial Analysis (DuckDB)
All spatial operations performed using DuckDB with the spatial extension:

**Building-Flood Zone Intersection:**
```sql
SELECT DISTINCT buildings.id, buildings.geom
FROM buildings
JOIN flood_zones ON ST_Intersects(buildings.geom, flood_zones.geom)
```
Result: 525 buildings (9% of total)

**SWE Value Assignment:**
```sql
SELECT buildings.id, AVG(grid.swe) as avg_swe
FROM buildings
JOIN grid ON ST_Intersects(buildings.geom, grid.geom)
GROUP BY buildings.id
```
Averaged SWE values for buildings intersecting multiple grid cells.

### 6. Vulnerability Scoring

**Normalization:**
- SWE normalized to 0-100 scale: `(swe - 0) / 32.512 * 100`
- Distance to river normalized to 0-100 scale: `(max_distance - actual_distance) / (max_distance - min_distance) * 100`

**Final Score:**
- Distance weight: 50%
- SWE weight: 50%
- Formula: `(normalized_distance * 0.5) + (normalized_swe * 0.5)`

**Risk Categories** (based on final score):
- 0-25: Low Risk
- 25-50: Medium Risk
- 50-75: High Risk
- 75-100: Very High Risk

---

## Key Findings

1. **Geographic Concentration:** 477 of 525 buildings (91%) are within 1 km of the Yampa River, indicating the primary flood driver is proximity to the main channel.

2. **Snowpack Variation:** SWE ranges from 0-32.5 meters across the study area. High-elevation areas show extreme values (up to 32.767m capped value), consistent with SNODAS model documentation.

3. **Risk Stratification:** The 50/50 weighting of distance and SWE produces meaningful differentiation: buildings far from the river score lower even with moderate snowpack, while buildings adjacent to the river score highest.

4. **Data Quality:** Approximately 25% of clipped raster cells had negative or zero SWE values due to model limitations in high-elevation zones. These were set to 0 (conservative estimate).

---

## Deliverables

### Maps & Data (in `data/output/`)
- **01_risk_by_building(1-4).png** - Buildings colored by risk category with flood zones overlay throughout Steamboat Springs, CO.
- **02_SNODAS_Grid.png** - SNODAS grid cells showing snowpack distribution and building risks
- **03_snodas_clipped.png** - Snowpack data alone (context for SWE variation)
- **04_steamboat_all_buildings.png** - Full building inventory in study area
- **05_flood_zones.png** - FEMA flood hazard areas
- **bldgs_vulnerability.csv** - Ranked buildings (sorted by vulnerability score descending)
- Images that aren't mentioned are supplemental and showing my overall process.

### Data (in `/data/processed/`) 
Main data here, but kept other processed data 
- `bldgs_vulnerability_final.geojson` - 525 buildings with vulnerability scores and risk categories
- `snodas_grid.geojson` - 168 grid cells with SWE values
- `a_fld_zones_area.geojson` - High-risk flood zone polygons

### Interactive Web Map
- `risk_map.html` - Standalone Leafmap-based web map (hosted on GitHub Pages)
- See [Live Map](#live-interactive-map) link above

---

## Data Quality & Assumptions

### SNODAS Raster
- **Scale Factor:** Values stored as 16-bit signed integers multiplied by 1000; divided by 1000 to recover meters
- **Capped Values:** Buildings in extreme snowpack areas (≥32.767m) retain capped value (not masked as nodata) to preserve high-risk signal
- **Negative Values:** Approximately 25% of clipped cells had negative values (model artifacts in high-elevation zones); replaced with 0
- **Resolution:** 1 km grid may obscure local microclimates and terrain-induced snow variation

### Building Footprints (OSM)
- **Coverage:** 5,794 buildings mapped; mix of residential, commercial, and industrial
- **Not filtered by type** due to variable data quality (some areas have "residential" tags, others only "building")
- **Mapping Currency:** OSM data current as of project date; real-world changes may not be reflected

### FEMA Flood Zones
- **Vintage:** 2005 (most recent available for Routt County); does not reflect recent climate or development changes
- **Designations:** Filtered to A, AE, AO (1% annual chance flood zones per FEMA standards)
- **Limitations:** Based on historical hydrology; spring snowmelt patterns may differ from model assumptions

### Distance to River
- **Calibration:** Maximum distance set to 1,000m based on data inspection (477 of 525 buildings within 1 km)
- **Weighting:** 50/50 split with SWE reflects judgment that proximity and snowpack are equally important
- **Validation:** Visual inspection in QGIS confirmed buildings far from river score lower

---

## Limitations

1. **Single-Date Snapshot:** Analysis represents conditions on April 12, 2024; doesn't capture seasonal variation or multi-year trends
2. **Model Artifacts:** SNODAS documentation acknowledges insufficient snow removal mechanisms in high-elevation areas, causing unbounded accumulation in some grid cells
3. **FEMA Data Age:** 2005 flood zone boundaries may not reflect current river hydraulics or development patterns
4. **River Complexity:** Yampa River represented as centerline only; actual flood extent depends on discharge, channel capacity, and local topography
5. **No Real-Time Updates:** Snapshot analysis; would require scheduled data downloads for operational use
6. **Simplified Scoring:** Vulnerability based on two factors (SWE and distance); doesn't account for building elevation, structure type, or preparedness

---

## How to Reproduce

### Prerequisites
```bash
conda create -n flood-vuln python=3.10
conda activate flood-vuln
pip install geopandas rasterio gdal osmnx shapely duckdb folium leafmap pandas numpy
```

### Run Analysis
```bash
# 1. Clone repository
git clone https://github.com/[your-username]/steamboat-springs-flood-vulnerability.git
cd steamboat-springs-flood-vulnerability

# 2. Use environment.yaml to obtain libraries

# 2. Download raw data (FEMA must be manual)
# - Visit: https://hazards.fema.gov/nhl/
# - Search: Routt County, CO
# - Download flood hazard layer (shapefile)
# - Save to: data/raw/

# 3. Run notebooks in order
jupyter spring_thaw.ipynb


# 4. View outputs
ls data/processed/
```

### View Interactive Map
Open `risk_map.html` in a browser, or visit the [Live Map](#live-interactive-map).

---

## Future Enhancements

### Phase 2: Refined Modeling
- **River Network Integration:** Include tributary streams (not just main channel) for distance calculations
- **Temperature Forecast:** Integrate Open-Meteo 7-day forecast to predict melt timing
- **Building Attributes:** Add structure age, elevation, basement presence to refine risk score

### Phase 3: Automation & Deployment
- **Scheduled Updates:** Daily SNODAS downloads with automated scoring refresh
- **Web Service:** Flask/FastAPI backend with REST API for queries
- **Alert System:** Push notifications to emergency managers when vulnerability thresholds crossed

### Phase 4: Expansion
- **Multi-Town Analysis:** Replicate for Vail, Fort Collins, other Colorado flood-prone areas
- **Historical Validation:** Compare 2024 predictions to actual 2023 flood events
- **User Dashboard:** Streamlit or Dash interface for planners to run "what-if" scenarios

### Phase 5: Integration
- **Live Stream Gauges:** Incorporate USGS real-time discharge data
- **Evacuation Routes:** Overlay with road network and emergency routes
- **Damage Assessments:** Post-flood analysis to validate model accuracy

---

## Technical Stack

| Component       | Tool                               | Version |
| --------------- | ---------------------------------- | ------- |
| Language        | Python                             | 3.10+   |
| Geospatial      | GeoPandas, Rasterio, GDAL, Shapely | Latest  |
| Database        | DuckDB (spatial extension)         | 0.10+   |
| Web Mapping     | Leafmap, Folium                    | Latest  |
| Visualization   | QGIS, Matplotlib                   | Latest  |
| Version Control | Git/GitHub                         | —       |

---

## Learning Outcomes

This project demonstrates:

- **Raster Processing:** Download, convert, clip, and sample geospatial raster data (SNODAS)
- **Vector Operations:** Spatial joins, intersections, and aggregation (GeoPandas, DuckDB)
- **Spatial SQL:** Writing efficient queries with `ST_Intersects()`, `ST_Distance()`, aggregation
- **CRS Management:** Handling coordinate systems, reprojection, and validating geometric accuracy
- **Data Quality:** Identifying and handling model artifacts, outliers, and missing data
- **Geospatial Workflow:** End-to-end analysis from raw data to publication-ready maps and exports
- **Portfolio Development:** Documenting methodology, limitations, and future work for stakeholder communication

---

## Lessons Learned

1. **Data Inspection Matters:** Visual verification in QGIS caught an inflated max-distance value that would have skewed results
2. **CRS Consistency:** Reprojection issues were debugged only after checking geometry validity and explicitly setting spatial references
3. **Iterative Refinement:** River proximity integration seemed simple initially; complexity revealed the value of starting simple and adding features incrementally
4. **Documentation First:** Decisions (buffer size, scaling factors, weighting) must be explained in code comments for reproducibility

---

## Acknowledgments

- **Data Providers:** NOAA (SNODAS), OpenStreetMap, FEMA
- **Open-Source Tools:** GeoPandas, Rasterio, DuckDB, Leafmap
- **Inspiration:** Emergency management need for actionable flood risk data

---

## License

This project is licensed under the MIT License. See `LICENSE` file for details.

---

## Contact

**Author:** David J. Anderson
**Project Date:** March 2026  
**Questions?** Open an issue on GitHub or reach out via [[LinkedIn/personal site](https://www.linkedin.com/in/dander89/)]

---

## References

- NOAA SNODAS Documentation: https://nsidc.org/data/G02158/
- FEMA NFHL: https://hazards.fema.gov/nhl/
- OpenStreetMap: https://www.openstreetmap.org/
- DuckDB Spatial: https://duckdb.org/docs/extensions/spatial
- Geospatial Analysis in Python: https://geopandas.org/
  
---

**Last Updated:** March 17, 2026

---

[View Code on GitHub](https://github.com/dander1989/Spring-Thaw-Flood-Assessor)