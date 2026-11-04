# Multi-Hazard Prediction System

A physics-informed digital twin for exploring flood, landslide, and combined
hazard risk across Uttarakhand, Assam, and the Western Ghats.

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=111827)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white)
![AI/ML](https://img.shields.io/badge/AI%2FML-XGBoost-EC6C00)

## Table of contents

1. [Overview](#1-overview)
2. [Problem Statement](#2-problem-statement)
3. [Objectives](#3-objectives)
4. [Key Features](#4-key-features)
5. [Screenshots & Demo](#5-screenshots--demo)
6. [System Architecture](#6-system-architecture)
7. [Technology Stack](#7-technology-stack)
8. [Project Structure](#8-project-structure)
9. [Installation & Setup](#9-installation--setup)
10. [Environment Configuration](#10-environment-configuration)
11. [Running the Application](#11-running-the-application)
12. [API Documentation](#12-api-documentation)
13. [Dataset & Data Sources](#13-dataset--data-sources)
14. [Methodology & Implementation](#14-methodology--implementation)
15. [Results & Evaluation](#15-results--evaluation)
16. [Limitations](#16-limitations)
17. [Future Scope](#17-future-scope)
18. [Contributing](#18-contributing)
19. [License](#19-license)
20. [Authors & Acknowledgements](#20-authors--acknowledgements)
21. [Deployment Notes](#deployment-notes)
22. [Troubleshooting](#troubleshooting)
23. [Development](#development)

## 1. Overview

The Multi-Hazard Prediction System combines terrain processing, hydrology
approximations, rainfall inputs, and a trained XGBoost model into an
interactive web dashboard. It divides each supported region into analysis
cells and estimates:

- flood intensity and probability;
- landslide stress;
- composite and hybrid risk;
- rainfall-dependent changes in risk; and
- locations that may need closer inspection through adaptive refinement.

The application presents these results as an interactive Leaflet map,
Plotly charts, and a generated 3D terrain/hazard view.

This is an experimental analysis and scenario-exploration tool. It is not an
official warning service and must not be used as the sole basis for emergency
response, evacuation, construction, or public-safety decisions.

## 2. Problem Statement

Floods and landslides are influenced by interacting variables rather than one
measurement. Rainfall intensity, accumulated water, elevation, terrain slope,
soil saturation, flow accumulation, and historical hazard patterns can all
vary across a small geographic area.

Single-factor maps may hide these interactions and usually provide limited
support for what-if analysis. This project addresses that problem by combining
terrain and hydrology calculations with rainfall scenarios and machine-learning
enrichment in one interactive digital-twin workflow.

The system helps users explore where risk indicators may increase, compare
rainfall assumptions, and inspect the physical features behind a score. It
does not replace official forecasts or emergency-management systems.

## 3. Objectives

1. Build a reusable digital representation of selected Indian regions.
2. Convert DEM and rainfall data into grid-cell-level hazard indicators.
3. Model water memory, flow accumulation, saturation, flood intensity, and
   landslide stress.
4. Combine interpretable physics-based outputs with historical-event ML
   predictions.
5. Support historical, live, and fast dynamic rainfall scenarios.
6. Deliver results through a browser dashboard, compact GeoJSON, charts, and
   3D terrain visualization.
7. Cache expensive terrain and simulation results for repeated exploration.

## 4. Key Features

- Terrain-aware flood, landslide, and composite-risk calculations.
- DEM reprojection to UTM coordinates for distance and area calculations.
- Grid-cell generation, slope/elevation extraction, downhill graphs, and flow
  accumulation.
- Soil-memory and saturation updates across simulation steps.
- XGBoost-based risk enrichment using historical event and rainfall features.
- Historical, live-rainfall, and dynamic what-if modes.
- Adaptive refinement of high-risk cells.
- Compact GeoJSON responses for browser visualization.
- Probability histograms, multi-hazard plots, rainfall-scenario plots,
  statistics, and progress reporting.
- Cached terrain and GeoJSON artifacts to avoid repeating expensive work.

Supported regions are `uttarakhand`, `assam`, and `western_ghats`. Supported
modes are `historical`, `live`, and `dynamic`.

## 5. Screenshots & Demo

### Hazard map

![Regional hazard map](docs/screenshots/hazard-map.png)

### 3D hazard simulation

![3D hazard simulation](docs/screenshots/dashboard-3d-simulation.png)

### Simulation plots

![Simulation plots](docs/screenshots/simulation-plots.png)

To run the demo locally, start the backend and frontend as described in
[Running the Application](#11-running-the-application). The dashboard normally
runs at `http://localhost:5173`, while the API runs at
`http://localhost:8345`.

## 6. System Architecture

```text
                         +-----------------------------+
                         | React + Vite dashboard      |
                         | Leaflet + Plotly + 3D view  |
                         +--------------+--------------+
                                        |
                              HTTP JSON / GeoJSON
                                        |
                         +--------------v--------------+
                         | FastAPI / Uvicorn           |
                         | server.py :8345            |
                         +------+----------------------+
                                |
              +-----------------+-----------------+
              |                                   |
  +-----------v-----------+           +-----------v-----------+
  | Simulation engine     |           | Output/static files   |
  | main.py               |           | outputs/              |
  +-----------+-----------+           +-----------------------+
              |
     +--------+---------+------------------+
     |                  |                  |
 +---v---+          +---v---+          +---v---+
 | core/ |          |services/|        |ml_models/|
 | DEM,  |          |rainfall,|        |XGBoost   |
 | grid, |          |hazards, |        |artifact  |
 | flow  |          |state    |        +---------+
 +---+---+          +----+----+
     |                   |
     +---------+---------+
               |
        data/ DEM, rainfall,
        and historical events
```

The backend validates simulation requests, launches the engine when needed,
serves generated files, clips output to regional boundaries, compacts GeoJSON,
and reports progress. The frontend consumes those resources and renders map,
chart, and 3D views.

### Request lifecycle

```text
User selects region, mode, and rainfall
                  |
                  v
        React calls POST /api/simulate
                  |
                  v
        FastAPI validates the request
                  |
          +-------+--------+
          |                |
          v                v
     Cached output     Run main.py
     is returned       with CLI args
          |                |
          +-------+--------+
                  |
                  v
     GeoJSON is clipped and compacted
                  |
                  v
       Frontend renders map and charts
```

### Component responsibilities

| Component | Responsibility |
| --- | --- |
| `main.py` | Orchestrates terrain, rainfall, hydrology, hazard, and output processing. |
| `core/grid.py` | Creates the spatial analysis grid. |
| `core/reproject.py` | Reprojects DEM data into a metric coordinate system. |
| `core/terrain.py` | Calculates elevation and slope features. |
| `core/graph.py` | Builds the downhill graph. |
| `core/hydrology.py` | Calculates flow accumulation. |
| `services/memory_service.py` | Updates rainfall memory with decay. |
| `services/hazard_service.py` | Calculates hazard scores, compacts GeoJSON, and loads ML artifacts. |
| `services/rainfall_service.py` | Loads CHIRPS data and maps rainfall to cells. |
| `services/live_rainfall_service.py` | Retrieves live rainfall inputs. |
| `services/refinement_service.py` | Refines selected high-risk cells. |
| `server.py` | Exposes the API, starts simulations, and serves output files. |
| `frontend/src/App.jsx` | Implements dashboard state, API calls, maps, and charts. |

## 7. Technology Stack

| Layer | Technologies | Responsibility |
| --- | --- | --- |
| Frontend | React, Vite, React Leaflet, Leaflet | Dashboard and interactive map |
| Charts/3D | Plotly, `react-plotly.js`, Lucide React | Charts, terrain view, icons |
| API | Python, FastAPI, Uvicorn, Pydantic | REST API and static files |
| Scientific | NumPy, pandas, SciPy, xarray | Arrays, tables, rainfall processing |
| Geospatial | GeoPandas, Shapely, Rasterio, pyproj | DEMs, geometry, reprojection |
| Hydrology | NetworkX, pysheds | Downhill graphs and flow calculations |
| ML | XGBoost, scikit-learn | Risk enrichment and evaluation |
| Data formats | GeoTIFF, NetCDF, NumPy, CSV, GeoJSON | Inputs, models, and outputs |

## 8. Project Structure

```text
.
├── core/                 Terrain, grid, graph, and hydrology calculations
├── services/             Rainfall, hazard, state, and refinement services
├── data/                 DEMs, rainfall files, and event data
├── ml_models/            Trained model and feature metadata
├── outputs/              Generated GeoJSON, plots, stats, and caches
├── frontend/             React/Vite dashboard
├── main.py               Simulation-engine CLI entry point
├── server.py             FastAPI application
├── train_ml_model.py     ML training workflow
├── ML_TRAINING_CONDITIONS.md
├── ML_TRAINING_RESULTS.md
├── PROJECT_EXPLANATION.txt
└── TECHNICAL_SPECIFICATION_DEEP_DIVE.txt
```

Important modules include `core/grid.py`, `core/terrain.py`,
`core/hydrology.py`, `services/hazard_service.py`,
`services/rainfall_service.py`, `main.py`, and `server.py`.

## 9. Installation & Setup

### Prerequisites

- Windows, macOS, or Linux.
- Python 3.10 or newer.
- Node.js 18 or newer and npm.
- The repository's DEM, rainfall, event, and model files.
- Adequate disk space for raster processing and generated GeoJSON.

The repository does not currently contain a `requirements.txt`; install the
Python dependencies used by the source modules:

```powershell
py -3.10 -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install fastapi "uvicorn[standard]" numpy matplotlib geopandas rasterio tqdm xgboost scikit-learn pandas shapely requests urllib3 xarray scipy pyproj networkx pysheds
```

If PowerShell activation is unavailable, use the interpreter directly:

```powershell
.\venv\Scripts\python.exe -m pip install fastapi "uvicorn[standard]" numpy matplotlib geopandas rasterio tqdm xgboost scikit-learn pandas shapely requests urllib3 xarray scipy pyproj networkx pysheds
```

Install the frontend dependencies:

```powershell
Set-Location frontend
npm install
```

Available frontend scripts:

```text
npm run dev       Start Vite development server
npm run build     Create production build
npm run preview   Preview production build
npm run lint      Run ESLint
```

## 10. Environment Configuration

The frontend currently uses `http://localhost:8345` as its backend origin,
defined in `frontend/src/App.jsx`. If the API host or port changes, update
that value before starting the frontend.

The backend selects the interpreter used to launch `main.py` in this order:

1. `DIGITALTWINS_PYTHON`, if it points to an existing interpreter;
2. `venv\Scripts\python.exe` in the repository; or
3. the current Python interpreter.

PowerShell example:

```powershell
$env:DIGITALTWINS_PYTHON = "C:\path\to\python.exe"
```

Region input paths are configured in `main.py` under `REGION_CONFIG`. There
is currently no committed `.env.example`; never commit credentials, tokens,
or local machine paths.

### Region configuration

Each `REGION_CONFIG` entry in `main.py` associates a public region ID with:

| Value | Purpose |
| --- | --- |
| `dem` | Source DEM used for terrain processing. |
| `utm` | Projected DEM used for metric grid calculations. |
| `flow` | Precomputed flow-accumulation NumPy array. |
| `output` | Primary simulation output path. |
| `lat`, `lon` | Initial map and simulation center. |
| `rainfall` | CHIRPS NetCDF source. |

If a data file is moved, update the corresponding mapping and verify that the
file exists before running the engine.

### Runtime and security configuration

The development server enables permissive CORS and binds to `0.0.0.0:8345`.
This is convenient for local development but should not be exposed directly
to the public internet. A production deployment should restrict CORS, use
HTTPS, add authentication and rate limiting where appropriate, and keep
datasets and model artifacts outside public upload paths.

## 11. Running the Application

Use two terminals from the repository root.

### Terminal 1: start the API

```powershell
.\venv\Scripts\python.exe server.py
```

The API listens on `http://localhost:8345`. Interactive documentation is
available at `http://localhost:8345/docs`.

### Terminal 2: start the dashboard

```powershell
Set-Location frontend
npm run dev
```

Open the URL printed by Vite, normally `http://localhost:5173`.

### First-run order

Run historical mode for a region before dynamic mode. Historical mode creates
the base GeoJSON and terrain artifacts that fast dynamic mode can reuse.
Historical processing may be slow on a cold cache.

## 12. API Documentation

All API paths are served from `http://localhost:8345`.

### `GET /api/progress`

Returns the latest progress from `outputs/progress.json`:

```json
{
  "progress": 100,
  "status": "Historical simulation complete"
}
```

### `POST /api/simulate`

Request:

```json
{
  "region": "assam",
  "mode": "dynamic",
  "rainfall": 250
}
```

`region` must be `uttarakhand`, `assam`, or `western_ghats`. `mode` must be
`historical`, `live`, or `dynamic`. Rainfall defaults to `100.0`.

```powershell
curl.exe -X POST http://localhost:8345/api/simulate `
  -H "Content-Type: application/json" `
  -d '{\"region\":\"assam\",\"mode\":\"dynamic\",\"rainfall\":250}'
```

The response contains status, message, output GeoJSON path, optional plot
paths, logs, and—when using the fast dynamic path—inline compact GeoJSON.

Typical successful response:

```json
{
  "status": "success",
  "message": "Simulation completed for assam in dynamic mode.",
  "geojson": "/outputs/assam_dynamic.geojson",
  "plots": null,
  "logs": "..."
}
```

Unsupported regions or modes return HTTP 400. Missing cached input can return
HTTP 404. Simulation and file-processing errors are reported as HTTP 500 with
a diagnostic detail message.

### `POST /api/generate-plots`

Runs a simulation with plot generation enabled:

```powershell
curl.exe -X POST http://localhost:8345/api/generate-plots `
  -H "Content-Type: application/json" `
  -d '{\"region\":\"uttarakhand\",\"mode\":\"historical\",\"rainfall\":100}'
```

Generated plots include probability histograms, multi-hazard comparisons, and
rainfall-scenario charts.

### `GET /api/geojson`

Retrieves processed map data:

```text
/api/geojson?region=assam&mode=historical&include_ml=true&compact=true
```

Query parameters are `region`, `mode`, `include_ml` (default `true`), and
`compact` (default `true`).

The returned document follows the GeoJSON `FeatureCollection` shape:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Polygon",
        "coordinates": []
      },
      "properties": {
        "flood": 0.0,
        "landslide": 0.0,
        "risk": 0.0,
        "probability": 0.0,
        "runtime_rainfall_mm": 250.0,
        "risk_level": "low"
      }
    }
  ]
}
```

Common properties include `flood`, `landslide`, `risk`, `probability`,
`physics_risk_score`, `ml_risk_score`, `hybrid_risk_score`,
`runtime_rainfall_mm`, and `risk_level`. Exact fields can vary between raw,
summary, historical, live, and dynamic outputs.

### `GET /outputs/<filename>`

Serves generated files such as:

```text
/outputs/assam_historical_summary.geojson
/outputs/assam_historical_multi_hazard.png
/outputs/progress.json
```

## 13. Dataset & Data Sources

The project uses:

- DEM GeoTIFFs for Uttarakhand, Assam, and the Western Ghats;
- projected UTM DEMs and flow-accumulation NumPy arrays;
- CHIRPS rainfall NetCDF files;
- `data/finalss_india_flood_landslide.csv` historical events;
- `ml_models/ml_risk_model.pkl` and related feature metadata; and
- generated GeoJSON, PNG, JSON, and cache files.

Expected input paths include:

```text
data/3dem/assam_small.tif
data/3dem/uttarakhand_dem_small.tif
data/3dem/western_ghats_dem.tif
data/dem/*_utm_small.tif
data/dem/*_flow.npy
data/rainfall/chirps-v2.0.<year>.days_p05.nc
data/finalss_india_flood_landslide.csv
```

Source datasets may have separate licenses and redistribution restrictions.
Review the original provider terms before publishing or sharing them.
Large DEMs, arrays, caches, generated outputs, and the `data/` directory are
ignored by `.gitignore`.

### Input data quality checks

Before a long simulation, confirm:

1. the DEM opens and has a valid coordinate reference system;
2. the projected DEM covers the same region as the source DEM;
3. the flow array dimensions match the raster used to derive it;
4. rainfall files contain the expected time, latitude, longitude, and
   rainfall variables;
5. event rows contain usable latitude, longitude, year, location, and hazard
   type values; and
6. the selected region has a matching entry in `REGION_CONFIG`.

The application can produce technically valid output even when source data is
incomplete, so these checks are important before interpreting results.

## 14. Methodology & Implementation

### Terrain and hydrology

1. Load a region DEM and projected UTM raster.
2. Generate a base analysis grid.
3. Calculate elevation and slope statistics.
4. Build a downhill graph for downstream movement.
5. Calculate flow accumulation and upstream relationships.
6. Refine high-risk cells when the adaptive refinement threshold is met.

### Physics-based simulation

Rainfall increases each cell's water memory, while a decay factor represents
drainage or drying. Memory, upstream contribution, slope, and flow produce a
flood index, saturation value, landslide stress, and composite physics risk.

### Machine-learning enrichment

`services/hazard_service.py` loads the XGBoost artifact when ML enrichment is
requested. Output properties can include `ml_risk_score`,
`physics_risk_score`, and `hybrid_risk_score`.

### Dynamic mode

Dynamic mode applies a rainfall amount to cached base GeoJSON rather than
rerunning the full simulation for every input. Rainfall is bounded to
`0`–`500` mm. Zero rainfall clears hazard values; higher rainfall can activate
more cells by lowering the activation threshold.

### Risk calculation flow

```text
rainfall + previous water memory + upstream flow + terrain
                              |
                              v
                  flood index and saturation
                              |
                              v
                     landslide stress
                              |
                              v
                    composite physics risk
                              |
                         optional ML
                              |
                              v
                       GeoJSON response
```

Many displayed values are normalized or clipped to `[0, 1]` so they can be
compared in map colors and dashboard metrics. This is a modeling convention;
the values should not automatically be interpreted as calibrated probabilities.

### Adaptive refinement

The engine identifies cells whose probability exceeds an absolute and
percentile-based threshold. Eligible cells are refined only when their grid
level is below the configured maximum depth. A cap on the number of cells
prevents one run from expanding without bound. Refinement improves local
detail but increases processing time and output size.

### Output generation

A run can write full and summary GeoJSON, compact GeoJSON, PNG charts, JSON
statistics, terrain caches, and `progress.json`. The API prefers compact or
summary artifacts for browser requests to reduce transfer and parsing time.

## 15. Results & Evaluation

The recorded ML training summary reports approximately:

| Metric | Documented range | Meaning |
| --- | --- | --- |
| ROC-AUC | 0.80–0.90 | Discrimination between positive and negative samples |
| PR-AUC | 0.70–0.85 | Precision-recall behavior for imbalanced data |
| F1-score | 0.70–0.85 | Balance between precision and recall |

These are training-workflow results, not a guarantee of operational accuracy.
The documented results identify a small positive event set, sparse Western
Ghats coverage, and rainfall dominance in some model outputs.

See [`ML_TRAINING_RESULTS.md`](./ML_TRAINING_RESULTS.md) and
[`ML_TRAINING_CONDITIONS.md`](./ML_TRAINING_CONDITIONS.md) for the complete
training data, features, hyperparameters, and evaluation notes.

### Training pipeline

```text
Historical event CSV
        |
        v
Filter regions, years, and coordinates
        |
        v
Load simulation GeoJSON and calculate cell centroids
        |
        v
Match events to nearby cells
        |
        +--> positive event samples
        +--> negative background samples
        |
        v
Engineer physics, terrain, rainfall, and interaction features
        |
        v
Train/test split and stratified validation
        |
        v
XGBoost classifier and calibration
        |
        v
ml_models/ml_risk_model.pkl
```

The training script currently records a `0.5` degree spatial matching
tolerance, uses rainfall years `2013`, `2018`, and `2022`, and removes several
identifying fields from the final feature set to reduce leakage. Treat the
executable training configuration as the source of truth when it differs from
older explanatory notes.

### Feature groups

| Group | Examples | Purpose |
| --- | --- | --- |
| Physics | Flood probability, landslide stress, composite risk | Preserve simulation signals |
| Terrain | Elevation, slope, aspect, roughness | Describe landscape susceptibility |
| Hydrology | Flow accumulation, distance to water | Describe water concentration |
| Rainfall | Annual, monsoon, recent rainfall, variability | Describe precipitation patterns |
| Interactions | Slope x rainfall, flow x slope | Capture compound conditions |

### Re-training checklist

Before replacing the model artifact:

- record dataset date range and row counts;
- verify positive and negative class counts;
- check for duplicate events and spatial leakage;
- evaluate per-region performance, not only aggregate metrics;
- inspect calibration and threshold behavior;
- compare feature importance with the previous model; and
- run a historical simulation to confirm artifact compatibility.

## 16. Limitations

- The model is an experimental hybrid of simplified physical indicators and
  historical-data-driven ML.
- Historical event coverage is limited and the positive sample count is small.
- Western Ghats labels are sparse or unavailable in the documented workflow.
- DEM resolution, rainfall coverage, event geocoding, and matching tolerance
  affect results.
- Live mode depends on network access and rainfall-service availability.
- Dynamic mode is a fast adjustment of cached data, not a complete simulation
  rerun for every slider value.
- A high score identifies a cell for investigation; it does not prove that a
  flood or landslide will occur.

Validate results against local observations, authoritative forecasts, and
qualified disaster-management experts before making real-world decisions.

## Deployment Notes

The repository is currently organized for local development. A production
deployment should package the frontend and backend separately or serve the
built frontend through a reverse proxy:

```text
Browser
  |
  v
HTTPS reverse proxy
  |----------------------> frontend build
  |
  +----------------------> FastAPI/Uvicorn
                                  |
                                  +--> data/
                                  +--> ml_models/
                                  +--> outputs/
```

For a production-like local API process:

```powershell
.\venv\Scripts\python.exe -m uvicorn server:app --host 0.0.0.0 --port 8345
```

Large DEMs, rainfall files, and generated GeoJSON should use mounted storage
or object storage rather than source control. Keep model, source-data, and
configuration versions together so results can be reproduced.

## 17. Future Scope

- Expand and validate historical event labels.
- Add higher-resolution elevation, slope, aspect, soil, and drainage features.
- Add rainfall anomalies and longer climatological baselines.
- Train region-specific and hazard-specific models.
- Quantify uncertainty and display confidence intervals.
- Validate predictions against independent held-out events.
- Integrate official real-time rainfall and alert sources.
- Add automated API and simulation-invariant tests.
- Package the backend and frontend for reproducible deployment.

## 18. Contributing

1. Create a focused feature branch.
2. Preserve existing region and mode behavior unless the change is intended.
3. Update related documentation and tests.
4. Run `npm run lint`, `npm run build`, and backend syntax checks.
5. Describe data, model, API, or UX changes in the pull request.

For training changes, include dataset scope, matching assumptions, metrics,
and known limitations. Do not commit credentials, virtual environments, large
datasets, or generated outputs.

## 19. License

This repository does not currently include a `LICENSE` file. Until an explicit
license is added, all rights remain with the copyright holder and reuse or
redistribution should be treated as restricted.

## 20. Authors & Acknowledgements

### Authors

The repository does not currently contain a formal author list. Add verified
contributors and ownership details here when they are confirmed.

### Acknowledgements

This project uses open-source technologies including React, Vite, FastAPI,
Uvicorn, GeoPandas, Rasterio, Shapely, NumPy, pandas, xarray, XGBoost,
scikit-learn, Leaflet, Plotly, NetworkX, and pysheds. Data providers and
original dataset authors should be credited according to their respective
licenses and citation requirements.

## Troubleshooting

### `ModuleNotFoundError`

Confirm that the project interpreter is being used:

```powershell
.\venv\Scripts\python.exe -c "import fastapi, geopandas, rasterio, xarray; print('imports ok')"
```

Install missing packages into that same environment.

### Dynamic mode has no base GeoJSON

Run historical mode for the same region first:

```powershell
.\venv\Scripts\python.exe main.py --region assam --mode historical --skip_plots 1
```

### Dashboard has no map data

Confirm that `server.py` is running on port `8345`, `/docs` opens, the
selected output exists under `outputs/`, and the frontend backend origin
matches the API URL.

### API returns `404` for GeoJSON

The API requires the raw region/mode GeoJSON to exist before it can return a
processed response. Run the corresponding simulation first and check the
lowercase region ID:

```powershell
Get-ChildItem outputs\assam_*.geojson
```

### API returns `500` while launching a simulation

Inspect the backend terminal for the child-process error. Common causes are
missing raster/rainfall files, incompatible geospatial packages, a missing
model artifact, insufficient memory, or an interpreter selected by
`DIGITALTWINS_PYTHON` that lacks dependencies.

Run the command directly for a complete traceback:

```powershell
.\venv\Scripts\python.exe main.py --region assam --mode historical --skip_plots 1
```

### Live mode cannot retrieve rainfall

Check network access, the live rainfall service implementation, and upstream
API availability. Use historical or dynamic mode to test the rest of the
application without a live-data dependency.

### Windows file-lock problems

Stop an old backend process before restarting it, and avoid editing generated
files while a simulation is writing them. If a generated file is corrupted,
remove only that specific file and rerun the relevant region/mode.

## Development

```powershell
Set-Location frontend
npm run lint
npm run build

Set-Location ..
.\venv\Scripts\python.exe -m compileall core services main.py server.py train_ml_model.py
```

When changing simulation behavior, validate at least one historical region and
one dynamic rainfall scenario. When changing API responses, verify
`/api/progress`, `/api/simulate`, `/api/geojson`, and `/docs`.

### Recommended validation matrix

| Change | Minimum validation |
| --- | --- |
| Frontend component or styling | `npm run lint`, `npm run build`, browser smoke test |
| API validation or response shape | Test valid and invalid requests and inspect `/docs` |
| Terrain or hydrology code | Compile and run one historical region |
| Dynamic rainfall logic | Test `0`, `250`, and `500` mm |
| ML feature or artifact code | Load/train the model and run enriched output |
| GeoJSON clipping/compaction | Compare compact and non-compact responses |

### Coding conventions

- Keep region IDs lowercase and consistent across Python and React.
- Reuse existing core and service modules instead of duplicating calculations.
- Keep generated files under `outputs/`.
- Preserve explicit errors for invalid inputs and failed child processes.
- Update this README when commands, endpoints, input paths, or output fields
  change.
