# Multi-Hazard Prediction System

> A digital-twin platform for predicting and visualizing flood and landslide
> risk across selected regions of India.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![React](https://img.shields.io/badge/React-Vite-61DAFB)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-009688)
![AI/ML](https://img.shields.io/badge/AI%2FML-XGBoost-orange)

## 1. Overview

The Multi-Hazard Prediction System creates a digital twin of Uttarakhand,
Assam, and the Western Ghats. It combines terrain, hydrology, rainfall, and
machine-learning data to estimate flood, landslide, and composite risk through
an interactive React dashboard.

## 2. Problem Statement

Floods and landslides are influenced by interacting factors such as rainfall,
terrain, soil saturation, elevation, slope, and upstream water flow. Traditional
single-factor analysis may not represent these relationships accurately or
provide an accessible way to explore potential scenarios.

## 3. Objectives

- Model terrain and downhill water flow for supported regions.
- Estimate flood and landslide risk using physics-based indicators.
- Improve predictions using historical data and XGBoost.
- Provide historical, live, and what-if rainfall simulations.
- Visualize results through interactive maps and charts.

## 4. Key Features

- Terrain-aware flood, landslide, and composite-risk calculations.
- Historical, live rainfall, and dynamic rainfall scenario modes.
- Interactive Leaflet map with hazard layers.
- Plotly charts and 3D terrain visualizations.
- Adaptive grid refinement for higher-risk areas.
- Fast dynamic rainfall updates.
- GeoJSON output for web visualization.

## 5. Screenshots & Demo

The dashboard provides an interactive map, hazard overlays, dynamic rainfall
scenarios, 3D terrain visualization, and simulation plots.

### 3D Hazard Simulation

![3D hazard simulation](docs/screenshots/dashboard-3d-simulation.png)

### Hazard Map

![Regional hazard map](docs/screenshots/hazard-map.png)

### Simulation Plots

![Historical simulation plots](docs/screenshots/simulation-plots.png)

The dashboard runs locally at the URL provided by Vite, usually
`http://localhost:5173`.

## 6. System Architecture

```text
React/Vite dashboard
        |
        v
FastAPI REST API
        |
        +--> Simulation engine
        |      +--> Terrain and hydrology services
        |      +--> Rainfall services
        |      +--> ML risk enrichment
        |
        +--> GeoJSON, plots, and progress files
```

## 7. Technology Stack

- **Frontend:** React, Vite, React Leaflet, Plotly, Lucide React
- **Backend:** Python, FastAPI, Uvicorn
- **Scientific computing:** NumPy, GeoPandas, Rasterio, Matplotlib
- **Machine learning:** XGBoost, scikit-learn, pandas
- **Geospatial visualization:** Leaflet and GeoJSON

## 8. Project Structure

```text
.
├── core/                 Terrain, grid, graph, and hydrology calculations
├── data/                 DEM and rainfall source data
├── frontend/             React/Vite dashboard
├── ml_models/            Trained model artifacts
├── outputs/              Generated GeoJSON, plots, and progress files
├── services/             Rainfall, hazard, ML, and state services
├── main.py               Simulation engine entry point
├── server.py             FastAPI application
└── train_ml_model.py     ML training workflow
```

## 9. Installation & Setup

### Prerequisites

- Python 3.10 or newer
- Node.js 18 or newer
- npm
- Required project data and model assets

### Backend

From the repository root:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install fastapi uvicorn numpy matplotlib geopandas rasterio tqdm xgboost scikit-learn pandas shapely
```

### Frontend

```powershell
cd frontend
npm install
```

Large DEM and NumPy files are excluded from Git. Place required assets under
`data/` and `ml_models/`.

## 10. Environment Configuration

The backend uses port `8345` by default. The frontend currently uses the
backend URL `http://localhost:8345`.

To select a specific Python interpreter for simulations, set
`DIGITALTWINS_PYTHON`:

```powershell
$env:DIGITALTWINS_PYTHON = "C:\path\to\python.exe"
```

## 11. Running the Application

Start the backend from the repository root:

```powershell
.\venv\Scripts\python.exe server.py
```

In a second terminal, start the frontend:

```powershell
cd frontend
npm run dev
```

Frontend commands:

```powershell
npm run build
npm run preview
npm run lint
```

## 12. API Documentation

The FastAPI service provides interactive documentation at
`http://localhost:8345/docs`.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/simulate` | Run a simulation. |
| `POST` | `/api/generate-plots` | Run a simulation and generate plots. |
| `GET` | `/api/geojson` | Retrieve processed GeoJSON. |
| `GET` | `/api/progress` | Read simulation progress. |
| `GET` | `/outputs/<file>` | Serve generated output files. |

Example request:

```powershell
curl.exe -X POST http://localhost:8345/api/simulate `
  -H "Content-Type: application/json" `
  -d '{\"region\":\"assam\",\"mode\":\"dynamic\",\"rainfall\":250}'
```

Supported regions are `uttarakhand`, `assam`, and `western_ghats`.
Supported modes are `historical`, `live`, and `dynamic`.

## 13. Dataset & Data Sources

The system uses:

- Digital Elevation Models for terrain and elevation processing.
- Rainfall and CHIRPS-derived data for historical and live scenarios.
- Historical flood and landslide event records for model training.
- Generated GeoJSON and cached terrain data for dashboard rendering.

Data files may be subject to their original source licenses and are not all
stored in this repository.

## 14. Methodology & Implementation

1. DEM data is reprojected and divided into analysis cells.
2. Elevation and slope are calculated for each cell.
3. A downhill graph and upstream map estimate water movement.
4. Soil saturation, rainfall, flood index, and landslide stress are computed.
5. Physics-based indicators are combined into a composite risk score.
6. XGBoost uses rainfall and hazard features to provide ML risk enrichment.
7. Results are clipped to the selected region and returned as compact GeoJSON.

Dynamic mode applies rainfall adjustments to cached results for fast
what-if analysis.

## 15. Results & Evaluation

The trained model workflow reports ROC-AUC, PR-AUC, and F1-score metrics.
Current documented training results report approximately:

- **ROC-AUC:** 0.80–0.90
- **PR-AUC:** 0.70–0.85
- **F1-score:** 0.70–0.85

See [`ML_TRAINING_RESULTS.md`](./ML_TRAINING_RESULTS.md) for the detailed
training summary and [`ML_TRAINING_CONDITIONS.md`](./ML_TRAINING_CONDITIONS.md)
for the training workflow.

## 16. Limitations

- Historical training data is limited and may cause overfitting.
- Western Ghats event labels are currently sparse.
- Detailed rainfall and terrain coverage depends on available source data.
- Results should support analysis and planning, not replace official warnings.
- Initial historical simulations may require significant processing time.

## 17. Future Scope

- Add elevation, slope, and aspect features from higher-resolution DEMs.
- Expand and regularly update the historical event dataset.
- Improve uncertainty quantification and confidence intervals.
- Add region-specific models and broader geographic coverage.
- Integrate additional real-time rainfall and alerting services.
- Deploy the dashboard and API as a scalable cloud service.

## 18. Contributing

1. Fork the repository.
2. Create a feature branch.
3. Make focused changes and add or update tests where applicable.
4. Run `npm run lint` and relevant validation commands.
5. Open a pull request describing the change.

Please avoid committing secrets, large generated outputs, or restricted data.

## 19. License

No license file is currently included in this repository. Add a `LICENSE`
file before distributing or reusing the project commercially.

## 20. Authors & Acknowledgements

Developed by the Multi-Hazard Prediction System project team.

Thanks to the maintainers of Python, FastAPI, React, Vite, Leaflet, Plotly,
GeoPandas, Rasterio, NumPy, scikit-learn, XGBoost, and the providers of the
terrain, rainfall, and historical hazard datasets used by the project.
