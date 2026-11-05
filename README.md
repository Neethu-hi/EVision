# EVision: Empirical Public EV Charging Demand and Hotspot Optimization Framework for India

EVision is an end-to-end data science and decision-support platform engineered to solve the public electric vehicle (EV) charging infrastructure siting problem across India. By harmonizing multi-source government records, economic indicators, climate data, and machine learning models, EVision provides a unified framework for cross-functional teams to plan, validate, and execute charging infrastructure deployments.

---

## Table of Contents

1. Executive Overview and Stakeholder Collaboration
2. The Core Problem: Why the EV-to-Charger Ratio Fails
3. End-to-End System Workflow: How All Modules Collaborate
   - Module 1: Multi-Source Ingestion and Data Harmonization
   - Module 2: The Three-Model Statistical Engine (M1, M2, M3, and CDS)
   - Module 3: Dual-Mode Bridge (FastAPI Service and Offline Snapshots)
   - Module 4: The Collaborative Decision Engine (Plan Rollout Wizard)
   - Module 5: Evidence Exploration and Deep Diagnostics
4. Detailed Methodology and Model Specifications
   - Effective Public Charging Demand Metric
   - M1: Macro Adoption Drivers (Negative Binomial Count Model)
   - M2: High-Demand Classification (Regularized Logistic Regression and LDA)
   - M3: 12-Month Growth Projections (SARIMA and Holt-Winters)
   - CDS: Composite Demand Score and Sensitivity Robustness
5. Data Ingestion, Governance, and Provenance
   - City Universe and Analysis Coverage
   - Authoritative Primary Sources
   - Data Hygiene and Integrity Principles
6. System Architecture and Repository Structure
7. Installation and Quick Start
   - Prerequisites
   - Setting Up and Running the Backend
   - Setting Up and Running the Frontend
   - Automated Parity and Scenario Verification
8. Key Empirical Insights
9. License and Acknowledgments

---

## 1. Executive Overview and Stakeholder Collaboration

Deploying public EV charging infrastructure in India requires coordination across diverse stakeholders with conflicting priorities:
- Charge Point Operators (CPOs): Seek maximum charger utilization rates, short payback cycles, and dependable revenue streams.
- Commercial Fleet Operators: Need reliable daytime fast chargers and depot access for three-wheeler delivery vans and four-wheeler ride-hailing cabs.
- Municipal Authorities and Urban Planners: Aim for equitable spatial coverage, reduced urban vehicular emissions, and minimal street congestion.
- Power Distribution Companies (Discoms): Require advance demand visibility to upgrade substation transformers and manage local grid loads.
- Infrastructure Investment Committees: Demand rigorous, empirical risk models rather than marketing projections before releasing capital.

In practice, these groups often work in silos, using incompatible spreadsheets and naive assumptions like aggregate EV counts. EVision bridges this gap by providing a single source of analytical truth. It brings econometric modeling, machine learning, and financial scenario planning into a single shared environment where technical teams, commercial operators, and public planners can collaboratively evaluate siting decisions.

The platform aggregates 3,519 city-month observations across 51 major Indian cities from January 2021 through September 2026. Every recommendation traces directly back to underlying empirical evidence, enabling clear cross-stakeholder alignment.

---

## 2. The Core Problem: Why the EV-to-Charger Ratio Fails

The conventional industry metric for charging adequacy is the raw EV-to-charger ratio: total registered electric vehicles in a city divided by the total number of public charging points.

While intuitive, this metric is fundamentally flawed and leads to capital misallocation:

### 2.1 Vehicle Category Charging Asymmetry
- Two-Wheelers (2Ws): In most Indian cities, personal electric scooters and motorcycles account for 55% to 65% of total EV registrations. However, private two-wheeler owners charge almost exclusively overnight at home or during work hours via standard low-power AC sockets. They do not queue at public commercial fast-charging plazas.
- Three-Wheelers (3Ws): Electric passenger autos and cargo e-rickshaws operate on tight commercial schedules. They have limited on-board battery capacity, run double shifts, and depend entirely on public opportunity fast charging or battery-swapping stations.
- Four-Wheelers (4Ws): Personal passenger cars split their charging between home AC chargers and occasional highway top-ups. In contrast, commercial four-wheeler fleets (taxis, shared mobility, and corporate logistics) rely on dedicated urban DC fast-charging depots.

### 2.2 The Allocation Blind Spot
- False Shortages: A city with 100,000 personal electric two-wheelers and only 50 public chargers shows an alarming ratio of 2,000 EVs per charger. Under naive metrics, an operator might build high-cost public fast chargers there, only to see near-zero customer footfall.
- Unseen Deficits: A logistics center or wholesale trade hub with 4,000 commercial electric three-wheelers, 1,000 electric commercial vans, and only 15 public chargers will appear to have low demand under a raw ratio. In reality, it suffers from severe charging congestion, vehicle downtime, and unmet commercial demand.

EVision resolves this contradiction by introducing an Effective Demand Index that weights vehicle classes by their empirical reliance on public charging infrastructure.

---

## 3. End-to-End System Workflow: How All Modules Collaborate

EVision functions as a closed-loop system where each component feeds directly into the next. Data flows from raw government registries through statistical inference, API distribution, and client-side decision tooling:

```
+-----------------------------------------------------------------------------------+
| 1. INGESTION & HARMONIZATION                                                      |
| MoRTH Vahan  |  BEE Stations  |  Census 2011  |  MoSPI Income  |  Open-Meteo ERA5 |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
| 2. DATA PIPELINE & FEATURE ENGINEERING (backend/src/evision/pipeline)             |
| - Grid completeness check across 51 cities x 69 months (3,519 observations)       |
| - 72 IQR outliers winsorised to boundary percentiles                              |
| - Engineered features: market share, 3-month rolling mean, 10km fixed supply      |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
| 3. FOUR-STAGE STATISTICAL SUITE (backend/src/evision/models)                      |
| - M1 Negative Binomial : Explains macro market drivers (IRR estimates)            |
| - M2 Regularized Logit/LDA: Classifies structural high vs low demand potential    |
| - M3 SARIMA / Holt-Winters : 12-month forward projections per city               |
| - CDS Engine : Merges gap, growth, classification, and supply into a 0-100 score  |
+-----------------------------------------------------------------------------------+
                                         |
                    +--------------------+--------------------+
                    |                                         |
                    v                                         v
+--------------------------------------+   +----------------------------------------+
| 4A. REST API SERVICE (FastAPI)       |   | 4B. SYNCHRONIZED OFFLINE SNAPSHOTS     |
| Real-time HTTP endpoints serving:    |   | export_frontend_snapshots.py syncs:    |
| /api/cities, /api/hotspots,          |   | frontend/src/data/snapshots/*.json     |
| /api/demand, /api/forecasts          |   | (Enables air-gapped demo parity)       |
+--------------------------------------+   +----------------------------------------+
                    \                                         /
                     +-------------------+-------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
| 5. COLLABORATIVE USER INTERFACE (frontend/src/components)                         |
|                                                                                   |
| WORKFLOW A: PLAN ROLLOUT WIZARD                                                   |
| - User inputs budget, target charger count, 2W/3W/4W priority, and risk stance    |
| - Algorithm computes optimal city allocation and capital distribution             |
| - "Why This City?" panel presents transparent quantitative justification          |
|                                                                                   |
| WORKFLOW B: EXPLORE THE EVIDENCE WORKBENCH                                        |
| - Interactive India vector map with choropleth and demand bubbles                 |
| - Hotspot priority ranking matrix with Monte Carlo sensitivity controls           |
| - SARIMA vs Holt-Winters holdout forecast comparison charts                       |
| - Effective vs raw demand divergence diagnostics                                  |
| - Individual city profile cards with historical panel traceability                |
+-----------------------------------------------------------------------------------+
```

### Module 1: Multi-Source Ingestion and Data Harmonization
The backend data engine pulls from five distinct authorities. The pipeline joins disparate spatial and temporal boundaries (RTO codes to municipal jurisdictions, monthly registration panels to static census records) into a coherent rectangular panel of 3,519 city-months.

### Module 2: The Three-Model Statistical Engine
The models do not operate in isolation; each validates and enriches the others:
- M1 isolates the fundamental macro drivers of adoption, verifying that market turnover and policy incentives explain growth.
- M2 uses strictly exogenous demographic, spatial, and economic predictors to determine whether a city possesses the fundamental conditions to support high demand (with zero label leakage).
- M3 tracks dynamic momentum, running time-series tournaments per city to forecast short-term fleet expansion.
- CDS (Composite Demand Score) acts as the synthesizer, taking the current physical gap, the projected 12-month momentum from M3, and the structural probability from M2 to produce an actionable 0 to 100 ranking.

### Module 3: Dual-Mode Bridge (FastAPI Service and Offline Snapshots)
Data science teams generate outputs using Python scripts. The results are served over a structured FastAPI REST service. Simultaneously, the automated script `export_frontend_snapshots.py` dumps exact JSON representations into the frontend source tree. This architecture guarantees that the frontend can run in live client mode connected to the API, or in an entirely standalone offline mode for boardroom presentations and field evaluations with zero setup friction.

### Module 4: The Collaborative Decision Engine (Plan Rollout Wizard)
Instead of forcing non-technical decision-makers to interpret raw regression outputs, the Plan Rollout Wizard translates corporate requirements into an optimized deployment schedule:
- Financial Inputs: Total investment capital, maximum budget allocation per city, and per-charger installation cost assumptions.
- Fleet Focus: Selectable prioritization between two-wheeler commuters, three-wheeler fleets, and four-wheeler commercial taxis.
- Risk Tolerance: Conservative (allocating only to proven top-tier metros), Balanced (spreading across established and emerging corridors), or Aggressive (capturing early-mover advantages in high-growth tier-2 markets).
- "Why This City?" Panel: For every allocated city, the platform generates a concrete rationale based on fleet composition, forecast slope, current charger deficit, and policy support. This gives project teams the exact documentation required for executive investment memos.

### Module 5: Evidence Exploration and Deep Diagnostics
Analysts can toggle into "Explore the Evidence" to interrogate the data behind every recommendation. They can inspect raw vs effective demand divergence, verify SARIMA holdout errors, evaluate Monte Carlo sensitivity bounds, and review data provenance for individual RTOs.

---

## 4. Detailed Methodology and Model Specifications

### 4.1 Effective Public Charging Demand Metric

The baseline fleet weighting reflects real-world urban charging patterns across Indian operating conditions:

- Effective Public EV Fleet = (w_2w * 2W_Stock) + (w_3w * 3W_Stock) + (w_4w * 4W_Stock)
  - w_2w = 0.05 (low public infrastructure reliance; predominant home AC overnight charging)
  - w_3w = 0.85 (high public infrastructure reliance; intensive commercial duty cycle)
  - w_4w = 0.50 (moderate public infrastructure reliance; blend of depot and home charging)

Demand-to-Supply Ratio:
- Effective Demand Ratio = Effective Public EV Fleet / Public Chargers within 10km Radius

### 4.2 M1: Macro Adoption Drivers (Negative Binomial Count Model)

Model 1 investigates what city-level attributes correlate with monthly EV sales volume. Because monthly registration counts exhibit overdispersion (variance significantly exceeding the mean), a Negative Binomial regression with a log-link function is utilized:

- Dominant Factor: Total non-EV vehicle market size is the strongest positive predictor (Incidence Rate Ratio [IRR] = 2.13 to 2.72, p < 0.001). EV adoption thrives where overall automotive replacement velocity is highest.
- Income Counter-Intuition: Per-capita income exhibits an IRR of 0.56 to 0.62. This demonstrates that early Indian EV adoption has been heavily spearheaded by cost-sensitive commercial operations (e-rickshaws and delivery fleets) in high-density tier-2/tier-3 hubs, rather than premium consumer four-wheelers in high-income metros.
- State Policy Impact: State-level EV policy scores show an IRR of 1.28 to 1.35 (p = 0.001), validating that upfront registration fee waivers and road-tax subsidies measurably accelerate adoption.

### 4.3 M2: High-Demand Classification (Regularized Logistic Regression and LDA)

Model 2 answers: "Based purely on exogenous demographic, geographical, and economic features, does this city have the structural traits of a high-demand market?"

- Target Variable: Binary classification (High Demand vs Low Demand), split at the median effective demand threshold.
- Leakage Prevention: All direct EV stock, charger counts, and past EV growth metrics are strictly excluded from predictors.
- Models: Elastic-Net Regularized Logistic Regression and Linear Discriminant Analysis (LDA).
- Performance:
  - Cross-validated AUC: 0.861 (LDA) and 0.844 (Logistic Regression).
  - Leave-One-Out Cross-Validation (LOOCV) Accuracy: 78% to 80%.
  - Train-to-CV Gap: Under 0.07, demonstrating high resilience against overfitting.

### 4.4 M3: 12-Month Growth Projections (SARIMA and Holt-Winters)

Model 3 forecasts monthly registration volumes 12 months ahead for all 51 cities to quantify future charging pressure:

- Tournament Architecture: For every city, SARIMA (Seasonal Auto-Regressive Integrated Moving Average) competes against Holt-Winters Exponential Smoothing.
- Evaluation Gate: Out-of-sample holdout accuracy is evaluated using Mean Absolute Error (MAE), Root Mean Squared Error (RMSE), and Mean Absolute Percentage Error (MAPE).
- Result: 28 cities select SARIMA; 23 cities select Holt-Winters. All 51 city series pass strict numerical stability gates with non-negative constraints and finite bounds.

### 4.5 CDS: Composite Demand Score and Sensitivity Robustness

The Composite Demand Score (CDS) integrates the findings of all models into a single 0 to 100 ranking:

- Formula Components:
  1. Current Effective Demand Gap (40% weight): Immediate shortage of public chargers relative to commercial fleet demand.
  2. 12-Month Projected Growth Rate (30% weight): Adoption velocity from Model 3.
  3. Structural Adoption Readiness (20% weight): M2 classification probability.
  4. Infrastructure & Density Baseline (10% weight): Population density and urban corridor integration.
- Classification Tiers:
  - Critical Priority (CDS 80 - 100): High current shortage, rapid fleet growth; top capital allocation priority.
  - High Priority (CDS 65 - 79): Fast-growing commercial hub; urgent requirement for fast-charging depots.
  - Moderate Priority (CDS 50 - 64): Steady, maturing market; suitable for phased installations.
  - Low Priority (CDS below 50): Developing market; residential charging currently satisfies demand.
- Robustness Testing:
  - Monte Carlo perturbation tests randomly vary component weights across 1,000 runs.
  - 14 to 15 of the top-15 ranked cities consistently retain their top-tier standing, proving the ranking is mathematically robust.

---

## 5. Data Ingestion, Governance, and Provenance

### 5.1 City Universe and Analysis Coverage

- Universe: 68 configured cities mapped in `config/cities.yaml`.
- Analyzed Set: 51 cities with complete multi-source covariates.
- Excluded Baseline: 17 cities have Vahan registration data but lack granular public charger coordinates, Census 2011 urban boundaries, or fuel price series. These are tracked in `data/processed/manifest.json` under `cities_without_covariates`.
- Temporal Scope: January 2021 through September 2026 (69 monthly periods per city).
- Panel Size: 3,519 city-months (96.9% observed data rate).

### 5.2 Authoritative Primary Sources

1. MoRTH Vahan Portal:
   - Monthly vehicle registrations across all RTOs.
   - Categorized by fuel type (Electric BOV, Pure EV, Petrol, Diesel, Hybrid, CNG).
   - Categorized by vehicle category (2W, 3W, 4W) and class (Transport vs Non-Transport).
2. Bureau of Energy Efficiency (BEE):
   - Verified inventory of Public Charging Stations (PCS) with geolocation coordinates and power levels.
3. Census of India (2011):
   - City and urban agglomeration population, municipal area, and density records.
4. MoSPI and Reserve Bank of India (RBI):
   - Per-capita Net State Domestic Product (NSDP) utilized as regional income and commercial activity proxies.
5. Petroleum Planning and Analysis Cell (PPAC):
   - Monthly retail motor spirit (petrol) price indices.
6. Open-Meteo ERA5 Climate Reanalysis:
   - Daily temperature observations, cooling degree days, and extreme heat days impacting battery thermal performance.

### 5.3 Data Hygiene and Integrity Principles

- Transparent Gaps: Missing months are never fabricated. A row without data remains explicitly tagged as unobserved.
- Traceable Provenance: Every observation is tagged with its source origin (`vahan_export`, `dashboard_cache`, or `imputed`).
- Outlier Winsorisation: 72 severe outliers detected via Interquartile Range (IQR) filters were winsorised to boundary values rather than removed, preserving time-series continuity.
- Non-Negative Guarantees: Registration counts and derived fleet stocks have floor constraints preventing negative values.

---

## 6. System Architecture and Repository Structure

```
EVision/
|-- backend/
|   |-- config/
|   |   |-- cities.yaml                 # 68-city definitions and RTO mappings
|   |   |-- city_coordinates.yaml       # Centroids for geospatial weather queries
|   |   `-- vahan_codes.yaml            # Vehicle, fuel, and state category definitions
|   |-- data/
|   |   |-- source/                     # Immutable raw source documents
|   |   |-- raw/                        # Extracted intermediate data tables
|   |   `-- processed/                  # Master panel, engineered features, model outputs
|   |-- notebooks/                      # Exploratory data analysis and model verification
|   |-- reports/                        # Executive summaries, slide decks, and whitepapers
|   |-- scripts/
|   |   |-- build_user_dataset.py       # Builds 51-city monthly panel (~30s)
|   |   |-- build_master_panel.py       # Merges all covariates into master_ev_panel.csv
|   |   |-- run_models.py               # Fits M1, M2, M3, and generates CDS rankings
|   |   |-- run_api.py                  # Launches FastAPI server on port 8000
|   |   |-- make_map.py                 # Generates standalone interactive HTML choropleth
|   |   `-- export_frontend_snapshots.py# Syncs fresh API outputs to frontend JSON snapshots
|   |-- src/
|   |   `-- evision/
|   |       |-- api/                    # FastAPI routes, schemas, and endpoint controllers
|   |       |-- models/                 # Mathematical implementations for M1, M2, M3, and CDS
|   |       |-- pipeline/               # Ingestion, cleaning, outlier handling, and derivation
|   |       `-- sources/                # Source adapters (Vahan, BEE, Census, Open-Meteo)
|   `-- requirements.txt                # Python backend dependencies
|-- frontend/
|   |-- src/
|   |   |-- components/
|   |   |   |-- plan/                   # Plan Rollout Wizard and Scenario Planner
|   |   |   |   |-- PlanWizard.tsx      # Multi-step capital allocation wizard
|   |   |   |   |-- ResultsView.tsx     # Budget deployment schedule and summary cards
|   |   |   |   |-- CityWhyPanel.tsx    # "Why This City?" rationale drawer
|   |   |   |   |-- ScenarioPresets.tsx # CPO Aggressive, Municipal, Logistics presets
|   |   |   |   |-- StepCompany.tsx     # Capital and charger volume inputs
|   |   |   |   |-- StepMarket.tsx      # Vehicle fleet segment focus
|   |   |   |   `-- StepRisk.tsx        # Risk tolerance and geographic filters
|   |   |   |-- CityDetailView.tsx      # Comprehensive city profile drilldown
|   |   |   |-- DemandGapView.tsx       # Effective vs raw demand diagnostic comparisons
|   |   |   |-- ForecastView.tsx        # SARIMA / Holt-Winters 12-month projections
|   |   |   |-- HotspotsView.tsx        # Ranked CDS table with sensitivity analysis
|   |   |   |-- MapView.tsx             # Interactive vector India choropleth map
|   |   |   |-- OverviewView.tsx        # National adoption trendlines and executive KPIs
|   |   |   `-- Header.tsx              # Navigation bar and dataset status indicators
|   |   |-- data/
|   |   |   |-- snapshots/              # Synchronized offline JSON data snapshots
|   |   |   `-- india_map_data.json     # Vector polygon and centroid definitions
|   |   |-- lib/
|   |   |   |-- scenario.ts             # Client-side capital allocation math engine
|   |   |   |-- assumptions.ts          # Default financial and operational parameters
|   |   |   |-- confidence.ts           # Statistical confidence tier calculations
|   |   |   `-- export.ts               # CSV and JSON deployment schedule exporter
|   |   |-- api.ts                      # API service with automatic offline fallback
|   |   |-- App.tsx                     # Main layout and view state router
|   |   `-- types.ts                    # TypeScript domain interfaces
|   |-- package.json                    # Node dependencies and scripts
|   |-- tailwind.config.js              # Tailwind CSS styling configuration
|   `-- vite.config.ts                  # Vite bundler configuration
|-- .gitignore
`-- README.md                           # Master documentation
```

---

## 7. Installation and Quick Start

### 7.1 Prerequisites

Ensure your development environment meets the following requirements:
- Python 3.10 or higher
- Node.js 18.0 or higher
- npm (bundled with Node.js)
- Git

### 7.2 Setting Up and Running the Backend

1. Navigate to the backend directory:
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

3. Install the required Python packages:
   ```bash
   pip install -r requirements.txt
   ```

4. Compile the dataset and execute the statistical models:
   ```bash
   python scripts/build_user_dataset.py
   python scripts/build_master_panel.py
   python scripts/run_models.py
   ```

5. (Optional) Synchronize the backend model outputs to the frontend offline snapshot directory:
   ```bash
   python scripts/export_frontend_snapshots.py
   ```

6. Start the FastAPI REST service:
   ```bash
   python scripts/run_api.py
   ```
   The backend service will be live at `http://127.0.0.1:8000`. You can test endpoints and review interactive schemas using Swagger at `http://127.0.0.1:8000/docs`.

### 7.3 Setting Up and Running the Frontend

1. In a separate terminal window, navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install Node dependencies:
   ```bash
   npm install
   ```

3. Verify mathematical scenario parity and allocation invariants:
   ```bash
   npm run verify:scenario
   ```

4. Launch the local development server:
   ```bash
   npm run dev
   ```
   The application will open at `http://localhost:5173`.

---

## 8. Key Empirical Insights

1. Hotspot Concentration:
   - The top 5 high-demand cities under the Composite Demand Score are Delhi, Bengaluru, Pune, Guwahati, and Surat.
   - Delhi and Bengaluru exhibit acute demand due to immense commercial fleet operations and high utilization rates on existing fast-charging corridors.
   - Guwahati and Surat emerge as high-priority underserved frontiers where EV adoption has outpaced public charging infrastructure installations.

2. Two-Wheeler Distortion in Raw Ratios:
   - In cities such as Jaipur and Lucknow, high two-wheeler adoption artificially inflates the raw EV-per-charger ratio.
   - When adjusted using the Effective Demand metric, the immediate public DC fast-charging deficit moderates significantly, directing focus toward public 3W/4W fleet corridors.

3. Commercial Vehicles Drive Charging Economics:
   - Commercial 3Ws and 4Ws represent less than 35% of total registered EVs nationwide, yet they account for over 80% of aggregate daily kilowatt-hour demand at public charging depots.
   - Profitable CPO business models depend fundamentally on targeting logistics corridors and taxi fleet bases rather than residential neighborhoods.

---

## 9. License and Acknowledgments

This platform is developed for educational, infrastructure planning, and transportation research purposes.

Data Acknowledgments:
- Ministry of Road Transport and Highways (MoRTH), Government of India (Vahan portal)
- Bureau of Energy Efficiency (BEE), Ministry of Power, Government of India
- Office of the Registrar General & Census Commissioner, India
- Ministry of Statistics and Programme Implementation (MoSPI)
- Petroleum Planning and Analysis Cell (PPAC)
- Open-Meteo ERA5 Reanalysis Weather API
