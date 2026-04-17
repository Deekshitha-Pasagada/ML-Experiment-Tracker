# ML Experiment Tracker

Internal platform for tracking machine learning experiments,
monitoring RL training runs, exploring finetuning datasets,
and analyzing time series metrics built for research teams.

## Overview

This system solves a core research workflow problem: when you're
running dozens of RL runs or finetuning experiments, you need a
centralized place to manage datasets, track hyperparameters, log
metrics over time, and understand what happened in each experiment.

## Features

- **Experiment Management** - Create and catalog experiments with
  dataset lineage, versioning, and provenance tracking
- **Run Tracking** - Log hyperparameters, configs, and run status
  for every training run
- **Time Series Metrics** - Ingest high-volume step-level metrics
  (loss, reward, accuracy) with batch logging support
- **Data Cataloging** - Tag datasets, track lineage, and store
  metadata for reproducibility
- **Interactive Dashboard** - Visualize training curves and compare
  runs across experiments
- **REST API** - Full FastAPI backend with OpenAPI docs at `/docs`

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python, FastAPI |
| Database | PostgreSQL / SQLite |
| ORM | SQLAlchemy |
| Frontend | HTML, CSS, JavaScript, Chart.js |
| Testing | PyTest, FastAPI TestClient |

## Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Start the API server
cd backend
uvicorn main:app --reload

# Open the dashboard
open frontend/index.html

# Run tests
pytest tests/
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /experiments/ | Create new experiment |
| GET | /experiments/ | List experiments (filter by tag/dataset) |
| GET | /experiments/{id} | Get experiment details |
| POST | /runs/ | Start a new training run |
| PATCH | /runs/{id} | Update run status |
| GET | /runs/experiment/{id} | Get all runs for an experiment |
| POST | /metrics/ | Log a single metric |
| POST | /metrics/batch | Log multiple metrics at once |
| GET | /metrics/run/{id} | Get all metrics for a run |
| GET | /metrics/run/{id}/summary | Get min/max/avg summary |

## Example Usage

```python
import requests

BASE = "http://localhost:8000"

# Create an experiment with dataset lineage
exp = requests.post(f"{BASE}/experiments/", json={
    "name": "rlhf-finetuning-v3",
    "description": "RLHF finetuning on filtered dataset",
    "tags": ["rlhf", "finetuning", "safety"],
    "dataset_name": "anthropic-hh-rlhf",
    "dataset_version": "v2.1",
    "dataset_lineage": {
        "source": "internal",
        "preprocessing": "safety-filtered",
        "size": 150000
    }
}).json()

# Start a training run
run = requests.post(f"{BASE}/runs/", json={
    "experiment_id": exp["id"],
    "run_name": "run-lr-1e4-bs32",
    "hyperparameters": {
        "learning_rate": 1e-4,
        "batch_size": 32,
        "kl_coeff": 0.1
    }
}).json()

# Log training metrics in batch
requests.post(f"{BASE}/metrics/batch", json=[
    {"run_id": run["id"], "name": "reward",
     "value": 0.42, "step": 100},
    {"run_id": run["id"], "name": "loss",
     "value": 1.85, "step": 100},
    {"run_id": run["id"], "name": "kl_divergence",
     "value": 0.08, "step": 100},
])
```

## Architecture

```
┌─────────────────────────────────────────┐
│         React Dashboard (Chart.js)       │
├─────────────────────────────────────────┤
│         FastAPI REST Backend             │
│  ┌──────────┬──────────┬─────────────┐  │
│  │Experiments│  Runs   │   Metrics   │  │
│  └──────────┴──────────┴─────────────┘  │
├─────────────────────────────────────────┤
│    SQLAlchemy ORM + PostgreSQL/SQLite    │
└─────────────────────────────────────────┘
```
