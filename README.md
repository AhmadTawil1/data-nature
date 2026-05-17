# Data Nature 🌿

**Satellite-driven ecological monitoring and heat anomaly detection for Northern Israel**

A multi-module Streamlit application that combines NDVI and Land Surface Temperature (LST) satellite data with machine learning, genetic algorithms, and AI-powered research retrieval — built as part of the Ecological Models Lab course, Spring 2026.

> **Team:** Alaa Barazi · Ahmad Tawil

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Data](#data)
- [Roadmap](#roadmap)

---

## Overview

Data Nature monitors vegetation health and surface temperature anomalies across **8 ecological sites** in Northern Israel using Landsat and MODIS satellite imagery sourced via Google Earth Engine. The platform spans historical data from **2000–2026** and provides tools for analysis, simulation, forecasting, and AI-assisted research synthesis.

---

## Features

### Modules

| Page | Description |
|------|-------------|
| **🗺️ Heatmap** | Interactive NDVI & LST satellite layers rendered over Northern Israel via Folium + Earth Engine |
| **🚨 Anomaly Detection** | Z-score analysis against a 30-day rolling baseline — flags heat anomalies across a 180-day window at three severity levels (Warning / Severe / Critical) |
| **🔬 What-If Simulator** | Cellular-automaton vegetation simulator — model how land cover changes propagate over time |
| **🧬 Planting Optimization** | Genetic algorithm to identify the optimal planting locations for maximum heat reduction |
| **📈 Forecast** | 7-day LST forecast using three ML models trained on Landsat time-series data |
| **📋 Reports** | Export site summaries as formatted PDF reports for any date range |
| **🤖 Research Assistant (RAG)** | Semantic search over 5 indexed academic papers with Gemini 2.5 Flash AI synthesis |

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Streamlit Frontend                   │
│  Home · Heatmap · Anomalies · Simulator · Forecast   │
│         Optimization · Reports · RAG                  │
└────────────────────┬─────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │    src/data_nature      │   Python library
        │                         │
        │  ingest/   → Earth Engine satellite ingest
        │  stats/    → Z-score, baseline, anomaly logic
        │  models/   → ML forecast (scikit-learn / torch)
        │  viz/      → Plotly / Folium chart helpers
        │  rag/      → ChromaDB retriever + PDF loader
        │  reports/  → ReportLab PDF generation
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │         Data Layer      │
        │                         │
        │  data/mock/   → CSV mock data for offline dev
        │  data/raw/    → Raw satellite exports
        │  data/processed/chroma/ → ChromaDB vector store
        │  papers/      → Indexed academic PDFs (RAG)
        └─────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │    External Services    │
        │                         │
        │  Google Earth Engine    → satellite imagery
        │  Google Gemini 2.5 Flash → RAG synthesis
        │  ChromaDB               → vector embeddings
        └─────────────────────────┘
```

### Key Design Decisions

- **Mock-first development** — all pages work offline using CSV fixtures in `data/mock/`, with the live Earth Engine path gated behind authentication. This lets collaborators run and develop without API credentials.
- **`src` layout** — the `data_nature` library lives under `src/` and is installed as an editable package, keeping app logic separate from page scripts.
- **Stateless pages** — each Streamlit page is self-contained and path-resolves back to the repo root, making them portable across environments.

---

## Project Structure

```
data-nature/
├── app/
│   ├── Home.py              # Dashboard entry point
│   ├── ui.py                # Shared UI components (page_hero, section_label)
│   └── pages/
│       ├── 1_Heatmap.py
│       ├── 2_Anomalies.py
│       ├── 3_Simulator.py
│       ├── 4_Optimization.py
│       ├── 5_Forecast.py
│       ├── 6_Reports.py
│       └── 7_RAG.py
├── src/
│   └── data_nature/
│       ├── ingest/          # Earth Engine satellite data pipeline
│       ├── stats/           # Anomaly detection logic
│       ├── models/          # ML forecast models
│       ├── viz/             # Chart and map helpers
│       ├── rag/             # ChromaDB retriever + PDF loader
│       └── reports/         # PDF report generation
├── data/
│   ├── mock/                # Offline CSV fixtures
│   ├── raw/                 # Raw satellite data exports
│   └── processed/           # Processed outputs (ChromaDB, etc.)
├── papers/                  # Academic PDFs indexed by the RAG module
├── tests/                   # Pytest test suite
├── pyproject.toml           # Project metadata and dependencies
├── requirements.txt         # Pinned dependencies for deployment
└── .env                     # API keys (not committed — see Configuration)
```

---

## Getting Started

### Prerequisites

- Python **3.11 or 3.12** (3.13 is not supported)
- Git

### 1. Clone the repository

```bash
git clone https://github.com/AhmadTawil1/data-nature.git
cd data-nature
```

### 2. Install dependencies

**Option A — pip (recommended for most users)**

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

**Option B — uv (faster)**

```bash
pip install uv
uv sync
```

### 3. Configure environment variables

Copy the template and fill in your keys:

```bash
cp .env.example .env
```

See [Configuration](#configuration) for details on which keys are required.

### 4. Run the app

```bash
streamlit run app/Home.py
```

The app opens at `http://localhost:8501`.

> **No API keys?** All pages fall back to mock CSV data automatically. You can explore every feature without Earth Engine or Gemini credentials.

---

## Configuration

Create a `.env` file in the project root (it is gitignored):

```env
# Required for the Research Assistant (RAG) page
GEMINI_API_KEY=your_google_gemini_api_key

# Required for live satellite imagery on the Heatmap page
# Authenticate once with: earthengine authenticate
# No key needed in .env — Earth Engine uses OAuth credentials stored locally
```

| Variable | Required | Used by |
|---|---|---|
| `GEMINI_API_KEY` | For RAG page | Gemini 2.5 Flash synthesis |
| Earth Engine auth | For live Heatmap | `earthengine authenticate` CLI |

Pages that require missing credentials display a graceful fallback and still render using mock data.

---

## Data

All mock data is generated programmatically via scripts in `data/mock/`:

| File | Description |
|------|-------------|
| `site_locations.csv` | Coordinates and land cover type for 8 monitored sites |
| `site_monthly.csv` | Monthly NDVI and LST values per site (2000–2026) |
| `anomalies.csv` | Detected heat anomalies with severity and status |
| `lst_timeseries.csv` | Daily LST time-series per site |
| `lst_history.csv` | Historical LST baseline data |
| `lst_forecast.csv` | 7-day forecast output per site and model |
| `model_metrics.csv` | ML model evaluation metrics (MAE, RMSE, R²) |

To regenerate mock data, run the generator scripts directly:

```bash
python data/mock/generate_anomalies.py
python data/mock/generate_forecast.py
python data/mock/generate_heatmap.py
```

---

## Roadmap

### In Progress

- [ ] Live Earth Engine ingest pipeline — replace mock CSVs with real Landsat/MODIS pulls on authentication
- [ ] Full test coverage for `stats/` and `models/` modules

### Planned

- [ ] **Real-time alert system** — push notifications or email alerts when a new Critical anomaly is detected
- [ ] **Multi-region support** — extend monitoring beyond Northern Israel to arbitrary AOI polygons drawn by the user
- [ ] **Deep learning forecast** — replace the current scikit-learn models with an LSTM / Transformer trained on the full 25-year time-series
- [ ] **Sentinel-2 integration** — add 10m-resolution Sentinel-2 imagery for higher-fidelity NDVI maps
- [ ] **User authentication** — multi-user support with per-user site assignments and report history
- [ ] **Scheduled ingestion** — cron-based pipeline to automatically pull new satellite passes and update the anomaly detector
- [ ] **Mobile-responsive layout** — refactor the Streamlit UI to render well on tablets and mobile browsers
- [ ] **REST API layer** — expose core analysis endpoints (anomaly detection, forecast) as a FastAPI service for third-party integration
- [ ] **Docker deployment** — containerize the app and Earth Engine credentials for one-command deployment

---

## Academic Papers (RAG Index)

The Research Assistant module is pre-indexed over the following papers:

1. *Google Earth Engine: Planetary-scale geospatial analysis for everyone* — Gorelick et al.
2. *Examination of the Relationship Between Surface Temperature and Spectral Land Cover*
3. *Machine Learning Prediction of Future LST*
4. *Errors in Time-Series Remote Sensing and an Open Access Application for Detecting and Visualizing Spatial Data Outliers Using Google Earth Engine*
5. *Spatial-RAG: Spatial Retrieval Augmented Generation*

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Streamlit, Plotly, Folium |
| Satellite data | Google Earth Engine, `earthengine-api`, `geemap` |
| Geospatial | GeoPandas, Shapely, rasterio, rioxarray |
| ML / Stats | scikit-learn, PyTorch, statsmodels, SciPy |
| RAG | ChromaDB, Google Gemini 2.5 Flash, `pypdf` |
| Reports | ReportLab |
| Package management | uv / pip |
| Python | 3.11 – 3.12 |

---

*Data Nature v0.1.0 · Ecological Models Lab — Spring 2026*
