# EVision: Empirical Public EV Charging Demand and Hotspot Optimization Framework for India

EVision is an end-to-end data science and decision-support platform designed to solve the electric vehicle (EV) charging infrastructure siting problem across India. By harmonizing multi-source government records, economic covariates, and machine learning models, EVision identifies where public EV charging stations are most urgently required.

---

## Table of Contents

1. Executive Overview
2. The Core Problem: Why the EV-to-Charger Ratio Fails
3. The EVision Solution and Methodology
   - Effective Demand Metric
   - M1: Macro Adoption Drivers (Negative Binomial Count Model)
   - M2: High-Demand Classification (Regularized Logistic Regression and LDA)
   - M3: 12-Month Growth Forecasts (SARIMA and Holt-Winters)
   - CDS: Composite Demand Score and Hotspot Ranking
4. Data Ingestion and Provenance
   - Evaluated Universe and Coverage
   - Primary Data Sources
   - Data Hygiene and Integrity Principles
5. System Architecture
   - Backend Architecture
   - Frontend Architecture
   - Offline Snapshot Redundancy
6. Project Directory Structure
7. Installation and Quick Start
   - Prerequisites
   - Setting Up and Running the Backend
   - Setting Up and Running the Frontend
8. Key Empirical Findings
9. About the Author
10. License and Acknowledgments

---

## 1. Executive Overview

India is witnessing exponential growth in electric mobility, yet the public charging infrastructure remains fragmented, thin, and unevenly distributed across states. For Charge Point Operators (CPOs), municipal planners, state nodal agencies, and infrastructure funds, deciding where to deploy capital presents a significant strategic risk.

Deploying fast chargers in low-utilization areas results in stranded capital, while under-allocating to high-density commercial corridors causes severe congestion and slows EV adoption. EVision replaces intuition and crude averages with an empirical, multi-tier statistical framework. It aggregates 3,519 city-month observations across 51 major Indian cities from January 2021 through September 2026, pairing registrations with demographics, power supply, fuel economics, and climate features.

The platform provides both an analytical API and an interactive web dashboard with two primary workflows:
- Plan Rollout Mode: An interactive capital allocation wizard that converts investor budget constraints, target charger volume, vehicle category focus, and risk appetites into a ranked, defensible deployment schedule with city-by-city rationale.
- Explore the Evidence Mode: A research-grade workbench featuring interactive choropleth maps, hotspot rankings, demand gap diagnostics, time-series forecasts, and city-level drilldown panels.

---

## 2. The Core Problem: Why the EV-to-Charger Ratio Fails

The most common metric used in public discussions is the raw EV-to-charger ratio: total registered electric vehicles in a city divided by the total number of public charging points.

This metric is fundamentally misleading for two primary reasons:

1. Heterogeneous Charging Behavior by Vehicle Category:
   - Two-wheelers (2Ws): Personal electric two-wheelers represent over 55-60% of all EV registrations in many Indian cities. However, private two-wheeler owners charge their vehicles almost exclusively at home or at the workplace using standard low-power AC sockets. They rarely queue at public fast-charging stations.
   - Three-wheelers (3Ws): Commercial electric three-wheelers (e-rickshaws and e-autos) operate intensive daily shifts and depend heavily on rapid turnaround, battery swapping, or public opportunity charging.
   - Four-wheelers (4Ws): Personal passenger cars use a hybrid mix of residential charging and occasional highway top-ups, whereas commercial taxi fleets and logistics delivery vans rely strictly on dependable urban public DC fast chargers.

2. Misleading Hotspot Identification:
   - A city with 100,000 electric two-wheelers and 50 chargers appears to have an acute shortage under a raw ratio (2,000 EVs per charger). Yet the actual demand pressure on public street charging may be negligible.
   - Conversely, a logistics hub with 5,000 commercial electric three-wheelers and four-wheelers with only 10 public chargers has an immediate charging deficit, despite having a much lower raw EV count.

EVision corrects this structural blind spot by formulating an Effective Demand index that computes the true public charging burden.

---

## 3. The EVision Solution and Methodology

EVision processes raw city data through a four-stage analytical pipeline:

### 3.1 Effective Demand Metric

Instead of raw counts, EVision weights the vehicle fleet based on empirical reliance on public charging infrastructure:

- Effective Public EV Demand = (0.05 * 2W Stock) + (0.85 * 3W Stock) + (0.50 * 4W Stock)

This weighted total is then evaluated against the verified public charger inventory within fixed 10-kilometer and 15-kilometer urban radius buffers, establishing the true Demand-to-Supply Gap.

### 3.2 M1: Macro Adoption Drivers (Negative Binomial Count Model)

To explain what structural factors drive monthly EV registrations, Model 1 employs a Negative Binomial generalized linear model with log-link. This model explicitly accounts for overdispersion in registration event counts across cities.

Key Findings:
- Total Non-EV Automotive Market Size: The strongest positive determinant of monthly EV registrations (Incidence Rate Ratio [IRR] = 2.13 to 2.72, p < 0.001). Cities with high baseline vehicle turnover adopt EVs significantly faster.
- Per-Capita Income Proxy: Exhibits a negative IRR (0.56 to 0.62), indicating that early EV volume in India has been driven by cost-conscious commercial adoption (3W commercial fleets and utilitarian 2Ws) in high-density, tier-2/tier-3 hubs, rather than luxury personal four-wheelers.
- State Policy Incentives: State-level EV policy scores yield a positive relationship (IRR = 1.28 to 1.35, p = 0.001), underscoring the influence of road-tax exemptions and local operational subsidies.

### 3.3 M2: High-Demand Classification (Regularized Logistic Regression and LDA)

Model 2 classifies cities into High-Demand versus Low-Demand tranches using exogenous structural variables (excluding EV stock, chargers, and growth figures to guarantee zero label leakage).

Methodological Highlights:
- Algorithms: Elastic-Net Regularized Logistic Regression and Linear Discriminant Analysis (LDA).
- Validation: Repeated 5-fold cross-validation and Leave-One-Out Cross-Validation (LOOCV).
- Performance: Cross-validated Area Under the Curve (AUC) reaches 0.861 for LDA and 0.844 for Logistic Regression (increasing to 0.914 to 0.977 depending on covariate subsetting). LOOCV accuracy ranges between 78% and 80%, maintaining an exceptional generalizability margin with an in-sample to out-of-sample gap of under 0.07.

### 3.4 M3: 12-Month Growth Forecasts (SARIMA and Holt-Winters)

Model 3 generates forward-looking 12-month projections of EV fleet expansion for every analysed city.

Methodological Highlights:
- Model Selection: For each city, EVision runs an automated contest between Seasonal Auto-Regressive Integrated Moving Average (SARIMA) with exogenous regressors and Holt-Winters Exponential Smoothing.
- Model Championship: Models are selected based on Bayesian Information Criterion (BIC), Akaike Information Criterion (AIC), and rolling out-of-sample holdout Mean Absolute Error (MAE).
- Stability: Out of 51 cities, 28 cities select SARIMA and 23 select Holt-Winters. All 51 city series pass strict numerical convergence gates, eliminating exploded variance or negative forecast projections.

### 3.5 CDS: Composite Demand Score and Hotspot Ranking

The flagship output of EVision is the Composite Demand Score (CDS), a normalized 0 to 100 ranking that synthesizes:
1. Current Effective Demand Gap (40% weight): Unmet public charging need based on weighted vehicle category mix.
2. 12-Month Projected Growth Rate (30% weight): Forecasted adoption pressure from Model 3.
3. Macroeconomic and Adoption Readiness (20% weight): Classification probability from Model 2.
4. Urban Infrastructure Baseline (10% weight): Population density, power grid readiness, and corridor connectivity.

Tiers:
- Critical Priority (CDS 80 - 100): Immediate infrastructure deficit; high utilization guaranteed.
- High Priority (CDS 65 - 79): Fast-growing commercial adoption; urgent need for fast-charging hubs.
- Moderate Priority (CDS 50 - 64): Maturing markets suitable for steady phased rollouts.
- Low Priority (CDS below 50): Developing or residential-dominated ecosystems.

Robustness and Sensitivity:
To ensure the ranking is not an artifact of arbitrary weights, EVision conducts Monte Carlo weight perturbations. Across 1,000 simulation passes, 14 to 15 of the top-15 ranked cities remain stably within the priority set.

---

## 4. Data Ingestion and Provenance

### 4.1 Evaluated Universe and Coverage

- Configured Cities: 68 Indian cities defined in the universe schema (`config/cities.yaml`).
- Fully Analyzed Cities: 51 cities with complete multi-dimensional covariates.
- Covariate-Poor Excluded Cities: 17 cities have Vahan registration data but lack standardized city-level charger maps, Census 2011 boundaries, or verified fuel prices (tracked transparently in `data/processed/manifest.json`).
- Temporal Range: January 2021 through September 2026 (69 monthly periods per city).
- Panel Size: 3,519 city-month observations.
- Observed Data Share: 96.9% observed across the panel.

### 4.2 Primary Data Sources

1. MoRTH Vahan Dashboard and Exports:
   - RTO-level monthly vehicle registrations segmented by fuel type (Electric BOV, Pure EV, Petrol, Diesel, Hybrid, CNG).
   - Registrations segmented by vehicle category (2W, 3W, 4W) and vehicle class (Transport vs Non-Transport).
2. Bureau of Energy Efficiency (BEE):
   - Public Charging Station (PCS) locations, charger capacities, and geographic coordinates.
3. Office of the Registrar General and Census Commissioner of India:
   - Census 2011 city and urban agglomeration population, geographical area, literacy rate, and density figures.
4. Ministry of Statistics and Programme Implementation (MoSPI) and Reserve Bank of India (RBI):
   - Net State Domestic Product (NSDP) per capita used as purchasing-power and income proxies.
5. Petroleum Planning and Analysis Cell (PPAC):
   - Monthly retail motor spirit (petrol) prices across state capitals and regional hubs.
6. Open-Meteo ERA5 Reanalysis:
   - Historical temperature profiles, cooling degree days (CDD), and extreme summer heat days influencing EV battery thermal efficiency and HVAC power consumption.

### 4.3 Data Hygiene and Integrity Principles

- Zero Fabricated Data: Data gaps are explicitly recognized and logged. Missing values are never silently fabricated or back-filled.
- Source Lineage: Every row in the dataset carries a `panel_source` tag (`vahan_export`, `dashboard_cache`, or `imputed`).
- Outlier Management: 72 extreme outliers detected via Interquartile Range (IQR) filters were winsorised to boundary percentiles rather than dropped, preserving continuous time continuity.
- Panel-First Approach: Cross-sectional snapshots with 51 rows fail standard statistical power tests; framing the problem as a 3,519-row panel enables robust time-series econometric modeling.

---

## 5. System Architecture

The EVision platform consists of a Python backend and a React/TypeScript frontend.

```
+-----------------------------------------------------------------------+
|                            DATA PIPELINE                              |
|  MoRTH Vahan  |  BEE Stations  |  Census 2011  |  MoSPI  |  Open-Meteo|
+-----------------------------------------------------------------------+
                                   |
                                   v
+-----------------------------------------------------------------------+
|                       BACKEND ENGINE (Python)                         |
|  Data Cleaning  -->  Feature Engineering  -->  Master Panel (3,519x116)|
|                                                                       |
|  STATISTICAL MODELS:                                                  |
|  - M1: Negative Binomial Count Regression (Market Drivers)            |
|  - M2: Regularized Logistic Regression & LDA (Demand Classifier)      |
|  - M3: SARIMA & Holt-Winters Forecasting (12-Month Projections)       |
|  - CDS: Multi-Criteria Composite Demand Scoring Framework             |
|                                                                       |
|  FASTAPI APPLICATION:                                                 |
|  - Endpoints: /api/cities, /api/hotspots, /api/demand, /api/forecasts |
|  - Static Snapshot Generator (export_frontend_snapshots.py)          |
+-----------------------------------------------------------------------+
                                   |
                                   v
+-----------------------------------------------------------------------+
|                       FRONTEND CLIENT (React)                         |
|  React 18  |  TypeScript  |  Vite  |  Tailwind CSS  |  Recharts       |
|                                                                       |
|  MODE 1: PLAN ROLLOUT (Scenario Planning Engine)                      |
|  - Budget & Charger Allocation Constraints                            |
|  - Vehicle Class Targeting (2W / 3W / 4W)                             |
|  - Strategic Risk Appetite & Infrastructure Thresholds                |
|  - City "Why This City?" Deep Rationale & Deployment Action Plans     |
|                                                                       |
|  MODE 2: EXPLORE THE EVIDENCE (Analytical Research Workbench)         |
|  - Interactive India Choropleth & Sizing Bubbles                      |
|  - CDS Priority Ranking Matrix & Sensitivity Table                    |
|  - Time-Series Forecast Comparison with Holdout Verification          |
|  - Effective vs Raw Demand Gap Diagnostic Charts                      |
|  - Granular City Historical Panel & Provenance Inspector              |
+-----------------------------------------------------------------------+
```

### 5.1 Offline Snapshot Redundancy

To ensure maximum reliability during presentations, client demonstrations, or air-gapped deployments, EVision includes a snapshot synchronization mechanism (`backend/scripts/export_frontend_snapshots.py`). This script extracts fresh outputs from the FastAPI service and writes synchronized JSON snapshots directly into `frontend/src/data/snapshots/`. 

If the backend server is unreachable, the frontend automatically falls back to these verified snapshots without interruption or loss of functionality.

---

## 6. Project Directory Structure

```
EVision/
|-- backend/
|   |-- config/
|   |   |-- cities.yaml                 # 68-city universe definition and RTO codes
|   |   |-- city_coordinates.yaml       # Centroids for geospatial queries
|   |   `-- vahan_codes.yaml            # Fuel, vehicle, and state category mappings
|   |-- data/
|   |   |-- source/                     # Original raw immutable source data
|   |   |-- raw/                        # Extracted intermediate data
|   |   `-- processed/                  # Master panel, features, and model outputs
|   |-- notebooks/                      # Exploratory data analysis notebooks
|   |-- reports/                        # Markdown and Word research reports, slide decks
|   |-- scripts/
|   |   |-- build_user_dataset.py       # Compiles the 51-city monthly panel
|   |   |-- build_master_panel.py       # Merges covariates into master_ev_panel.csv
|   |   |-- run_models.py               # Fits M1, M2, M3, and calculates CDS
|   |   |-- run_api.py                  # Starts the FastAPI HTTP server
|   |   |-- make_map.py                 # Generates standalone choropleth HTML map
|   |   `-- export_frontend_snapshots.py# Syncs backend data to frontend snapshots
|   |-- src/
|   |   `-- evision/
|   |       |-- api/                    # FastAPI routes and schemas
|   |       |-- models/                 # M1, M2, M3, and CDS mathematical logic
|   |       |-- pipeline/               # Ingestion, cleaning, imputation, and derivation
|   |       `-- sources/                # Source adapters (Vahan, BEE, Open-Meteo, etc.)
|   `-- requirements.txt                # Python package dependencies
|-- frontend/
|   |-- src/
|   |   |-- components/
|   |   |   |-- plan/                   # Plan Rollout Wizard and Scenario components
|   |   |   |-- CityDetailView.tsx      # Comprehensive city profile drilldown
|   |   |   |-- DemandGapView.tsx       # Effective vs raw demand diagnostics
|   |   |   |-- ForecastView.tsx        # SARIMA / Holt-Winters visualizer
|   |   |   |-- HotspotsView.tsx        # Ranked CDS table and sensitivity analysis
|   |   |   |-- MapView.tsx             # Interactive vector India map
|   |   |   |-- OverviewView.tsx        # High-level KPIs and executive charts
|   |   |   `-- Header.tsx              # Navigation and global filters
|   |   |-- data/                       # GeoJSON map assets and synchronized snapshots
|   |   |-- lib/                        # Scenario math, export helpers, palette utilities
|   |   |-- api.ts                      # Client API caller with fallback handling
|   |   |-- App.tsx                     # Main application layout and view router
|   |   `-- types.ts                    # TypeScript interface declarations
|   |-- package.json                    # Node dependencies and build scripts
|   |-- tailwind.config.js              # Tailwind CSS design system rules
|   `-- vite.config.ts                  # Vite build tooling configuration
|-- .gitignore
`-- README.md                           # Master project documentation
```

---

## 7. Installation and Quick Start

### 7.1 Prerequisites

Ensure you have the following installed on your machine:
- Python 3.10 or higher
- Node.js 18.0 or higher
- npm (comes bundled with Node.js)
- Git

### 7.2 Setting Up and Running the Backend

1. Open a terminal and navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Create and activate a Python virtual environment:
   - On Windows (PowerShell):
     ```powershell
     python -m venv venv
     .\venv\Scripts\Activate.ps1
     ```
   - On Linux / macOS:
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```

3. Install required Python packages:
   ```bash
   pip install -r requirements.txt
   ```

4. Build the panel and run the statistical models:
   ```bash
   python scripts/build_user_dataset.py
   python scripts/build_master_panel.py
   python scripts/run_models.py
   ```

5. (Optional) Export fresh offline snapshots to the frontend:
   ```bash
   python scripts/export_frontend_snapshots.py
   ```

6. Launch the FastAPI server:
   ```bash
   python scripts/run_api.py
   ```
   The backend API will start at `http://127.0.0.1:8000`. You can inspect the interactive Swagger API documentation at `http://127.0.0.1:8000/docs`.

### 7.3 Setting Up and Running the Frontend

1. Open a second terminal window and navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install Node dependencies:
   ```bash
   npm install
   ```

3. Run the automated scenario mathematical parity test:
   ```bash
   npm run verify:scenario
   ```

4. Launch the local development server:
   ```bash
   npm run dev
   ```
   The web application will open at `http://localhost:5173`.

---

## 8. Key Empirical Findings

1. Hotspot Concentration:
   - The top 5 high-demand cities under the Composite Demand Score are Delhi, Bengaluru, Pune, Guwahati, and Surat.
   - Delhi and Bengaluru exhibit acute demand due to immense commercial fleet operations and high utilization rates on existing fast-charging corridors.
   - Guwahati and Surat emerge as high-priority underserved frontiers where EV adoption has outpaced public charging infrastructure installations.

2. Two-Wheeler Distortion in Raw Ratios:
   - In cities such as Jaipur and Lucknow, high two-wheeler adoption artificially inflates the raw EV-per-charger ratio.
   - When adjusted using the Effective Demand metric, the immediate public DC fast-charging deficit moderates significantly, directing focus toward public 3W/4W fleet corridors.

3. Commercial Vehicles are the Linchpin of Public Charging Economics:
   - Commercial 3Ws and 4Ws represent less than 35% of total registered EVs nationwide, yet they account for over 80% of aggregate daily kilowatt-hour demand at public charging depots.
   - Profitable CPO business models depend fundamentally on targeting logistics corridors and taxi fleet bases rather than residential neighborhoods.

---

## 9. About the Author

Kirtan Kumar Sanghi is the primary developer, data architect, and researcher behind EVision.

- Role: Lead Data Scientist and Full-Stack Software Engineer
- Focus Areas: Applied Econometrics, Machine Learning, Geospatial Data Engineering, and High-Performance Web Applications.
- Core Contributions to EVision:
  - Formulated the statistical modeling architecture, including the M1 Negative Binomial count model, M2 regularized classifiers, and M3 SARIMA/Holt-Winters time-series forecasting suite.
  - Designed the data ingestion and transformation pipeline, standardizing over 3,500 city-months of heterogeneous public data into an analysis-ready panel.
  - Implemented the FastAPI service and developed the React/TypeScript frontend dashboard, featuring capital allocation scenario builders, vector maps, and diagnostic analytics.
- Contact:
  - Email: kirtankumarsanghi@gmail.com
  - GitHub: https://github.com/kirtankumarsanghi

Collaborator Note:
Frontend development and UI design contributions were supported in collaboration with Nainitha Yalamaddi (nainithayalamaddi@gmail.com).

---

## 10. License and Acknowledgments

This project is developed for educational, analytical, and infrastructure research purposes. 

Data Acknowledgments:
- Ministry of Road Transport and Highways (MoRTH), Government of India (Vahan portal)
- Bureau of Energy Efficiency (BEE), Ministry of Power, Government of India
- Office of the Registrar General & Census Commissioner, India
- Ministry of Statistics and Programme Implementation (MoSPI)
- Petroleum Planning and Analysis Cell (PPAC)
- Open-Meteo ERA5 Reanalysis Weather API
