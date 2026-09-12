# GLOFGuard

Research prototype for **relative GLOF (Glacial Lake Outburst Flood) risk assessment** of glacial lakes in Bhutan.

GLOFGuard is an educational research prototype. Its predictions are not official forecasts and should not replace Bhutanese government, scientific, hydrological, meteorological, or emergency-management systems.

This repository currently implements **Phase 1–3** only: project structure, database, Bhutan map, dashboard, lake list, and lake detail. AI models are **not trained** and the UI does not invent segmentation accuracy or flood probability.

## Problem

Glacial lakes in the Bhutan Himalaya can grow as glaciers retreat. Rapid expansion, combined with climate and hydrological conditions, can contribute to GLOF hazard. Monitoring needs multi-temporal satellite imagery plus environmental context.

## Solution (target system)

Use multi-temporal satellite imagery, climate information, hydrological information, and machine learning to detect changes in Bhutan's glacial lakes and provide a research prototype for relative GLOF risk assessment and early warning.

Current milestone: a working monitoring dashboard on **demo/simulated** lake records with a live map.

## Architecture

```text
Satellite → U-Net → Lake Detection → Change Detection
                                      ↓
Climate + Hydrology → Feature Extraction
                                      ↓
                              Random Forest
                                      ↓
                                Risk Level
                                      ↓
                            Map + Dashboard
                                      ↓
                              SMS / Email
```

Phase 1–3 wiring:

```text
SQLite (PostGIS-ready schema) → FastAPI → Next.js dashboard + Leaflet Bhutan map
```

## AI models

| Model | Role | Status |
| --- | --- | --- |
| U-Net + MobileNetV2 encoder | Glacial lake segmentation | **NOT TRAINED** — code stubs only |
| Random Forest | Relative GLOF risk (LOW–CRITICAL) | **NOT TRAINED** — code stubs only |

Do not treat dashboard risk scores as calibrated probabilities. Display form is **Risk Score: n/100**, never “n% chance of GLOF”.

## Dataset format

```text
data/satellite/images/     # paired scenes (later)
data/satellite/masks/      # labelled masks (later)
data/demo/lakes.json       # DEMO DATA seed for the UI
data/demo/bhutan.geojson   # national boundary for the map
```

## Installation

Requires Python 3.10+ (3.11+ preferred). On Windows, `py -3.10 -m venv .venv` works if `python` still points at an older install.

```bash
# from the repository root
py -3.10 -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
copy .env.example .env   # or: cp .env.example .env

cd frontend
npm install
```

Optional PostgreSQL/PostGIS:

```bash
docker compose --profile postgres up db
```

Then set `DATABASE_URL` and `USE_POSTGIS=true` in `.env`.

## Running the backend

From the repository root (so SQLite lands in `data/glofguard.db`):

```bash
.\.venv\Scripts\activate
$env:PYTHONPATH = (Get-Location)
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

API docs: http://localhost:8000/docs  
Health: http://localhost:8000/health

## Running the frontend

```bash
cd frontend
npm run dev
```

Open http://localhost:3000

`NEXT_PUBLIC_API_URL` defaults to `http://localhost:8000`.

## Training models

```bash
python scripts/train_segmentation.py
python scripts/evaluate_segmentation.py
python scripts/train_risk_model.py
python scripts/evaluate_risk_model.py
```

These commands currently print **Model not trained — demo mode**. They do not fabricate metrics.

## Demo mode

`DEMO_MODE=true` in `.env`.

- Lake names use publicly described Bhutan Himalayan lakes.
- Area, growth, perimeter, and relative risk scores are **SIMULATED DATA**.
- The UI shows **DEMO MODE** and **DEMO DATA** badges.
- No live satellite, climate, SMS, or email APIs are required.

## API documentation

Implemented now:

- `GET /health`
- `GET /api/lakes`
- `GET /api/lakes/{id}`
- `GET /api/lakes/{id}/history`
- `GET /api/lakes/{id}/satellite`
- `GET /api/dashboard/summary`
- `GET /api/map/layers`
- `GET /api/map/bhutan`
- `GET /api/alerts`
- `GET /api/climate/{lake_id}`
- `GET /api/hydrology/{lake_id}`

Scaffolded for later phases (not functional products yet): satellite analyze/segment, risk predict, email/SMS.

## Environmental data

Adapters live in `backend/services/providers.py`:

- `SatelliteDataProvider`
- `ClimateDataProvider`
- `HydrologyDataProvider`

Swap implementations later for Sentinel-2, Landsat, Copernicus, NASA/USGS, or Bhutanese monitoring feeds. Do not hard-code fake live APIs.

Risk band defaults are in `risk_config.yaml`. They are **not** official NCHM / DHMS thresholds.

## Limitations

- No trained segmentation or risk model.
- No real satellite processing pipeline in this phase.
- Demo hydrology/climate endpoints return empty series.
- SQLite stores lat/lon; PostGIS can be enabled later.
- Seed risk values are for UI layout only.

## Scientific disclaimer

GLOFGuard must not be used as:

- a claim that a GLOF will happen on a given date
- a guarantee that a lake will burst
- a replacement for official emergency warning

Use the language **relative GLOF risk**, **risk assessment**, **satellite-derived change**, and **model prediction**.

## Tests

```bash
pytest
```
