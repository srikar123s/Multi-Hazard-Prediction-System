# Multi-Hazard Prediction System

A digital-twin application for exploring flood and landslide risk across
Uttarakhand, Assam, and the Western Ghats. The system combines terrain and
hydrology calculations with machine-learning risk scoring and presents the
results through an interactive React dashboard.

## Features

- Terrain-aware flood, landslide, and composite-risk calculations
- Historical, live rainfall, and dynamic rainfall scenario modes
- Interactive Leaflet map with hazard layers
- Plotly charts and 3D terrain visualizations
- XGBoost-based machine-learning risk enrichment
- GeoJSON output for map rendering

## Architecture

```text
React/Vite frontend  ->  FastAPI backend  ->  simulation engine
                              |
                              +-> GeoJSON and plot files in outputs/
                              +-> terrain, rainfall, and ML services
```

The backend runs on port `8345` by default. The frontend expects the backend
at `http://localhost:8345`.

## Requirements

- Python 3.10 or newer
- Node.js 18 or newer
- npm
- The project data and model assets used by the simulation

Large DEM and NumPy data files are intentionally excluded from Git. Make sure
the required files are available under `data/` and `ml_models/` before running
the full simulation.

## Installation

### Backend

From the repository root, create and activate a virtual environment:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

Install the backend dependencies:

```powershell
pip install fastapi uvicorn numpy matplotlib geopandas rasterio tqdm xgboost scikit-learn pandas shapely
```

If PowerShell prevents script activation, run the commands using
`venv\Scripts\python.exe` directly instead.

### Frontend

```powershell
cd frontend
npm install
```

## Running the application

Start the backend from the repository root:

```powershell
.\venv\Scripts\python.exe server.py
```

In a second terminal, start the frontend:

```powershell
cd frontend
npm run dev
```

Open the URL printed by Vite, usually `http://localhost:5173`.

## Simulation modes

| Mode | Description |
| --- | --- |
| `historical` | Uses generated historical simulation data and cached results when available. |
| `live` | Uses the live rainfall integration and current model inputs. |
| `dynamic` | Applies a user-provided rainfall amount for fast what-if scenarios. |

Supported regions:

- `uttarakhand`
- `assam`
- `western_ghats`

## Backend API

The FastAPI service exposes:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/simulate` | Run a simulation for a region, mode, and rainfall value. |
| `POST` | `/api/generate-plots` | Run a simulation and generate plot images. |
| `GET` | `/api/geojson` | Retrieve processed GeoJSON for a region and mode. |
| `GET` | `/api/progress` | Read the current simulation progress. |
| `GET` | `/outputs/<file>` | Serve generated GeoJSON and plot files. |

Example request:

```powershell
curl.exe -X POST http://localhost:8345/api/simulate `
  -H "Content-Type: application/json" `
  -d '{\"region\":\"assam\",\"mode\":\"dynamic\",\"rainfall\":250}'
```

## Training the ML model

The optional training workflow is documented in
[`ML_TRAINING_CONDITIONS.md`](./ML_TRAINING_CONDITIONS.md). To run it:

```powershell
.\venv\Scripts\Activate.ps1
python train_ml_model.py
```

Generated model artifacts are written to `ml_models/`.

## Project structure

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

## Frontend commands

From `frontend/`:

```powershell
npm run dev       # Start the development server
npm run build     # Create a production build
npm run preview   # Preview the production build
npm run lint      # Run ESLint
```

## Notes

- Historical simulations can take longer on the first run because terrain and
  GeoJSON data may need to be generated.
- Generated files in `outputs/` and large source assets are ignored by Git.
- The frontend currently uses a fixed backend origin of
  `http://localhost:8345`; update `frontend/src/App.jsx` if the backend runs
  elsewhere.
