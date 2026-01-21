# River Pollution Emergency Response – Dataset Documentation

This dataset accompanies the mock MCM/ICM-style problem **“The River Intake Shield”**.
It is designed for building a **continuous-time / continuous-space transport model (ODE/PDE)** and for
**emergency decision-making** (intake shutdown timing) and **sensor placement optimization**.

All files in this package are self-contained. No external data sources are required.

---

## 1. Package Contents

### 1.1 Problem Statement (same content, different formats)
- `MCM2026Training-MockA-The River Intake Shield.pdf` — print-ready (recommended for teams)
- `MCM2026Training-MockA-The River Intake Shield.docx` — editable (for organizers)

### 1.2 Data Files (`data/`)
1. `flow_monthly_usgs_01491000_raw.csv` — monthly flow statistics (USGS station 01491000), units **cfs**
2. `flow_hourly_baseline_2025_06.csv` — baseline **hourly** discharge for **June 2025**, units **m³/s**
3. `scenario_parameters.yaml` — all fixed scenario parameters (reach, intake, spill, costs, parameter ranges, etc.)
4. `spill_profile_1min.csv` — spill source term time series at **1-minute** resolution
5. `river_geometry_segments.csv` — optional piecewise-constant channel geometry by river-km segments
6. `quality_thresholds.csv` — alert/shutdown thresholds at the intake
7. `candidate_sensor_locations_km.csv` — candidate sensor locations (river-km from spill point)
8. `water_demand_profile_relative.csv` — optional diurnal demand profile (relative, unitless)

---

## 2. Conventions (coordinate, time, units)

### 2.1 Coordinate system
- The river is modeled as 1-D along the main flow direction.
- **x = 0 km**: spill location (upstream)
- **x = 30 km**: water intake location (downstream)

### 2.2 Time reference
- **t = 0** corresponds to the start of the spill event.
- All times in `spill_profile_1min.csv` are minutes since spill start.
- All timestamps in `flow_hourly_baseline_2025_06.csv` are hourly timestamps in **June 2025**.

### 2.3 Units (key ones)
- Flow rate:
  - `flow_monthly_usgs_01491000_raw.csv`: **cfs** (cubic feet per second)
  - `flow_hourly_baseline_2025_06.csv`: **m³/s**
  - Conversion: 1 cfs = 0.0283168 m³/s
- Distance: km (convert to m if using SI in PDE)
- Concentration: mg/L (note 1 mg/L = 1e-3 kg/m³)

---

## 3. File Descriptions

### 3.1 `flow_monthly_usgs_01491000_raw.csv`  (USGS monthly statistics; units: cfs)
Monthly discharge statistics for USGS station **01491000**.
This file is provided for realism/background context and for justifying the June baseline magnitude.

**Columns (typical)**
- `year`, `month`: calendar year and month
- `mean`: monthly mean discharge (cfs)
- `hmax`, `hmean`, `hmin`: long-term historical statistics for that month (cfs)
- Some additional columns (e.g., `max`, `min`, `date`) may be present depending on the export.

**How to use**
- This file is **NOT** intended for direct hourly simulation in this mock contest.
- Teams may cite it to justify whether the June 2025 baseline is “low/typical/high”.

### 3.2 `flow_hourly_baseline_2025_06.csv`  (baseline hourly discharge; units: m³/s)
Baseline hourly discharge series for **June 2025**.
This is the **recommended and standardized Q(t)** input for all teams to ensure comparability.

**Columns**
- `datetime`: timestamp (hourly, June 2025)
- `flow_m3s`: discharge (m³/s)

**Notes**
- The series is constructed to be consistent with typical June monthly statistics and includes
  a smooth diurnal variation to represent realistic sub-daily variability.
- Teams may use this file directly without modification.

### 3.3 `scenario_parameters.yaml`  (scenario constants + parameter ranges)
Single source of truth for the fixed assumptions used in the statement.

**Top-level keys (typical)**
- `river_reach`: `length_km`, `width_m`, `depth_m`
- `intake`: `location_km`, `max_intake_m3s`, `emergency_storage_m3`
- `spill_event`: `mass_kg`, `duration_minutes`
- `transport_parameters`:
  - `longitudinal_dispersion_D_m2s`: baseline and typical range (m²/s)
  - `first_order_decay_k_per_day`: baseline and typical range (1/day)
- `thresholds_mgL`: alert and shutdown thresholds at the intake
- `costs`: hourly shutdown cost and other cost placeholders
- `unit_conversions`: key unit conversions (e.g., cfs → m³/s)

**Recommended practice**
- Teams should perform **sensitivity analysis** within the provided parameter ranges rather than searching external literature.

### 3.4 `spill_profile_1min.csv`  (spill source time series; 1-minute resolution)
Time series describing the release rate during the spill.

**Columns**
- `t_min_since_spill_start`: minute index since spill start (t = 0)
- `mass_release_rate_kg_per_s`: instantaneous mass release rate (kg/s)

> The integral of the release rate over time equals the total spill mass specified in `scenario_parameters.yaml`.

### 3.5 `river_geometry_segments.csv`  (optional geometry by segment)
Piecewise-constant width and depth along the reach.

**Columns**
- `segment_start_km`, `segment_end_km`: segment boundaries (km)
- `width_m`, `depth_m`: representative geometry for that segment

> If you do not use this file, you may use the constant width/depth in `scenario_parameters.yaml`.

### 3.6 `quality_thresholds.csv`  (intake thresholds)
Thresholds used for early warning and mandatory shutdown at the intake.

**Columns**
- `location`: label (e.g., `intake`)
- `x_km`: location (km)
- `alert_threshold_mg_per_L`
- `shutdown_threshold_mg_per_L`

### 3.7 `candidate_sensor_locations_km.csv`  (candidate monitoring sites)
Candidate sensor locations for the placement optimization task.

**Columns**
- `candidate_location_km`: river km from spill point
- `note`: short comment (human-readable)

### 3.8 `water_demand_profile_relative.csv`  (optional diurnal demand)
Relative hourly demand profile for the city (dimensionless).

**Columns**
- `hour_of_day`: 0–23
- `relative_demand`: relative multiplier

> Teams may ignore this file if they do not model diurnal demand explicitly.

---

## 4. Recommended Use (for teams)

1. **Flow Q(t)**: use `flow_hourly_baseline_2025_06.csv` as the standardized hourly discharge input.
2. **Transport model**: advection–dispersion (and optional decay), with uncertainty/sensitivity analysis for parameters such as D and k.
3. **Decision policy**: design an intake shutdown/restore rule that balances safety constraints and economic cost.
4. **Sensor placement**: evaluate candidate locations using forecast lead time, false alarm/miss rates, or expected cost reduction.

Good luck and model responsibly!
