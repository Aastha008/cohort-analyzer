# Cohort Analyzer

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/DuckDB-0.9%2B-FFF000?style=for-the-badge&logo=duckdb&logoColor=black)](https://duckdb.org/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.30%2B-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)](https://streamlit.io/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![CI Status](https://img.shields.io/badge/CI-Passing-brightgreen?style=for-the-badge&logo=githubactions&logoColor=white)](#automated-testing)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg?style=for-the-badge)](https://github.com/psf/black)

An interactive, high-performance data analytics platform for parsing transactional and user event logs to calculate retention rates, churn dynamics, and Customer Lifetime Value (LTV). Powered by **DuckDB** for in-memory vector-processed SQL transformations and rendered via **Streamlit** with **Plotly** heatmaps and decay curves.

---

## Problem Statement & Target Audience

### Problem Statement
Product managers, growth engineers, and data analysts frequently struggle to compute cohort retention matrices from raw event logs. Traditional workflows require writing complex, repetitive window functions in SQL or executing memory-intensive Pandas operations in Jupyter notebooks. Generating interactive heatmaps, drop-off matrices, and cumulative LTV curves usually requires custom, non-reusable visualization scripts.

### Target Audience
* **Product Managers & Growth Lead**: Needing real-time cohort visibility into feature retention, user drop-offs, and revenue stickiness without waiting for data engineering pipelines.
* **Data Analysts & Scientists**: Seeking an embedded SQL workbench and fast vector engine to run custom analytics over large event streams stored in CSV, Parquet, or JSON files.
* **Analytics Engineers**: Requiring programmatic API access to cohort metrics, decay vectors, and standardized JSON analytical models.

---

## Key Features

* **Multi-Format Data Ingestion**: Direct zero-copy ingestion of raw user event and transactional datasets in CSV, Parquet, and JSON formats using DuckDB.
* **Automated Cohort Calculation**: Dynamic temporal aggregation across customizable grains (daily, weekly, monthly) linked to user signup or initial conversion triggers.
* **Interactive Visualizations**: High-density cohort retention heatmaps, step-by-step decay/churn curves, and cumulative LTV metrics powered by Plotly.
* **Embedded SQL Workbench**: Run arbitrary, high-performance SQL queries directly against DuckDB tables in memory, with one-click aggregated CSV exports.
* **REST API Layer**: FastAPI backend endpoints for asynchronous upload, metadata inspection, programmatic cohort processing, and LTV calculation.

---

## Architecture & System Design

```
+-----------------------------------------------------------------------+
|                          Raw Log Ingestion                            |
|             (CSV / Parquet / JSON Event & Txn Files)                 |
+-----------------------------------------------------------------------+
                                   |
                                   v
+-----------------------------------------------------------------------+
|                      DuckDB Analytical Engine                         |
|                          (src/core.py)                                |
|   - Dynamic SQL Construction          - Temporal Aggregation          |
|   - Cohort Matrix Engine              - Cumulative LTV Curves         |
+-----------------------------------------------------------------------+
                                   |
           +-----------------------+-----------------------+
           |                                               |
           v                                               v
+---------------------+                         +---------------------+
|  REST API Service   |                         | Streamlit Analytics |
|    (src/api.py)     |                         |    (src/main.py)    |
| - Pydantic Validation|                         | - Dynamic Heatmaps  |
| - Open API Docs     |                         | - Interactive Curves|
| - JSON Endpoints    |                         | - SQL Workbench     |
+---------------------+                         +---------------------+
           |                                               |
           +-----------------------+-----------------------+
                                   |
                                   v
+-----------------------------------------------------------------------+
|                    Interactive Plotly Renderers                       |
|            (Retention Heatmaps, Churn Decay & LTV Matrices)          |
+-----------------------------------------------------------------------+
```

### Module Responsibilities

| File Path | Description / Role |
| :--- | :--- |
| `src/models.py` | Pydantic v2 domain schemas defining inputs for temporal grains, event triggers, cohort filters, LTV options, and response structures. |
| `src/core.py` | Core DuckDB analytics engine. Manages in-memory tables, dynamic cohort SQL generation, cross-tabulation, decay vector math, and query execution. |
| `src/api.py` | FastAPI application exposing endpoints for uploading log files, computing cohort tables, generating LTV metrics, and running raw SQL. |
| `src/main.py` | Streamlit interactive frontend. Handles file uploads, parameter sidebars, Plotly heatmap/decay chart rendering, and an interactive SQL workbench. |

---

## Technology Stack

* **Language**: Python 3.10+
* **Analytical Database**: DuckDB (vectorized in-memory SQL processing)
* **Data Interchange & Arrow**: Apache Arrow (`pyarrow`), Pandas, NumPy
* **REST API Framework**: FastAPI, Uvicorn, Pydantic
* **Frontend & Visualization**: Streamlit, Plotly Express, Plotly Graph Objects
* **Testing Framework**: Pytest, FastAPI TestClient

---

## Installation & Virtual Environment Setup

### 1. Prerequisites
Ensure you have Python `3.10` or higher installed on your system.

### 2. Clone the Repository
```bash
git clone https://github.com/your-org/cohort-analyzer.git
cd cohort-analyzer
```

### 3. Create and Activate Virtual Environment
```bash
# macOS/Linux
python3 -m venv venv
source venv/bin/activate

# Windows (PowerShell)
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 4. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## Environment Variables Configuration

Copy the provided `.env.example` file to create your runtime environment config:

```bash
cp .env.example .env
```

### `.env` File Options

```env
# Application Host & Port Configuration
API_HOST=0.0.0.0
API_PORT=8000

# DuckDB Database Settings
DUCKDB_MEMORY_LIMIT=4GB
DUCKDB_THREADS=4

# Streamlit Configuration
STREAMLIT_SERVER_PORT=8501
STREAMLIT_SERVER_MAX_UPLOAD_SIZE=500

# Logging Level (DEBUG, INFO, WARNING, ERROR)
LOG_LEVEL=INFO
```

---

## Usage

### 1. Running the FastAPI Backend Service
Launch the REST API service using Uvicorn:

```bash
uvicorn src.api:app --host 0.0.0.0 --port 8000 --reload
```
Access the interactive OpenAPI documentation at: `http://localhost:8000/docs`

### 2. Running the Streamlit Analytics Interface
Launch the interactive dashboard:

```bash
streamlit run src/main.py
```
Open your browser to: `http://localhost:8501`

---

### REST API Endpoints Overview

#### Ingest Dataset (`POST /api/v1/upload`)
Uploads a CSV, Parquet, or JSON log file into DuckDB.

```bash
curl -X 'POST' \
  'http://localhost:8000/api/v1/upload' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@sample_events.csv;type=text/csv'
```

#### Compute Retention Cohort (`POST /api/v1/cohorts`)
Calculates cohort retention percentages over daily, weekly, or monthly temporal grains.

```bash
curl -X 'POST' \
  'http://localhost:8000/api/v1/cohorts' \
  -H 'Content-Type: application/json' \
  -d '{
    "user_id_col": "user_id",
    "timestamp_col": "event_time",
    "cohort_grain": "monthly",
    "cohort_event": "signup",
    "retention_event": "transaction"
  }'
```

#### Calculate Cumulative LTV (`POST /api/v1/ltv`)
Computes customer lifetime value per cohort over relative periods.

```bash
curl -X 'POST' \
  'http://localhost:8000/api/v1/ltv' \
  -H 'Content-Type: application/json' \
  -d '{
    "user_id_col": "user_id",
    "timestamp_col": "event_time",
    "value_col": "amount",
    "cohort_grain": "monthly"
  }'
```

#### SQL Workbench Execution (`POST /api/v1/query`)
Execute ad-hoc DuckDB SQL queries against loaded datasets.

```bash
curl -X 'POST' \
  'http://localhost:8000/api/v1/query' \
  -H 'Content-Type: application/json' \
  -d '{
    "sql": "SELECT cohort_group, COUNT(DISTINCT user_id) as total_users FROM cohort_table GROUP BY cohort_group ORDER BY cohort_group"
  }'
```

---

## Automated Testing Instructions

The project uses `pytest` for unit and API route testing. Ensure all tests pass prior to submitting changes.

### Run All Test Suites
```bash
pytest -v
```

### Test Engine Core Isolation (`tests/test_core.py`)
```bash
pytest tests/test_core.py -v
```

### Test API Endpoints Isolation (`tests/test_api.py`)
```bash
pytest tests/test_api.py -v
```

### Run Coverage Reports
```bash
pytest --cov=src tests/
```

---

## Project Structure

```
cohort-analyzer/
│
├── .env.example            # Environment configuration template
├── .gitignore               # Ignored files, environments, and caches
├── requirements.txt         # Core dependencies (DuckDB, Streamlit, FastAPI, Plotly, etc.)
│
├── src/                     # Source Application Code
│   ├── __init__.py          # Package initialization
│   ├── models.py            # Pydantic schemas (Cohort, Ingestion, LTV, Query models)
│   ├── core.py              # DuckDB engine wrapper, matrix & LTV calculation functions
│   ├── api.py               # FastAPI route definitions and service handlers
│   └── main.py              # Streamlit dashboard, Plotly plots, SQL workbench UI
│
└── tests/                   # Automated Test Suite
    ├── __init__.py          # Test package initialization
    ├── test_core.py         # Unit tests for DuckDB engine and transformations
    └── test_api.py          # Integration tests for FastAPI endpoints
```

---

## Roadmap / Future Enhancements

- [ ] **Predictive Churn Modeling**: Integration of statistical models (e.g., BG/NBD, Gamma-Gamma) to forecast future LTV and churn risk.
- [ ] **Remote Data Lake Connectors**: Direct ingestion support for AWS S3, Google Cloud Storage, and Azure Blob storage using DuckDB extensions.
- [ ] **Automated PDF/Slack Export**: Periodic scheduled generation of cohort heatmaps and automated alert dispatch via Slack webhooks.
- [ ] **Segment Filter Builder**: Drag-and-drop UI component to perform dynamic cohort slice-and-dice based on user attributes (e.g., region, device type, acquire channel).

---

## License

Distributed under the **MIT License**. See `LICENSE` for more information.