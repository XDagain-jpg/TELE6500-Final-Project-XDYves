# Predictive Maintenance: Remaining Useful Life (RUL) Estimation  
**Data Analysis & Modeling of Aircraft Engine Degradation**

---

## Context — What Is the Problem?

In aviation and industrial systems, unexpected engine failures are costly and risky.  
Modern engines generate large volumes of sensor data during operation, but raw data alone
does not directly indicate *how close a system is to failure*.

The goal of this project is to **estimate Remaining Useful Life (RUL)** — the number of
operational cycles an engine can continue before failure — using multivariate time-series
sensor data.

This project uses the NASA C-MAPSS simulated turbofan engine datasets (FD001–FD004),
which represent fleets of identical engines operating under varying conditions and
fault modes :contentReference[oaicite:0]{index=0}.

From a business perspective, accurate RUL prediction enables:
- Proactive maintenance scheduling
- Reduced downtime and maintenance cost
- Improved safety and asset utilization

---

## Dataset Overview

The datasets increase in complexity by introducing multiple operating conditions and
multiple fault modes, allowing evaluation of model robustness.

| Dataset | Train Engines | Test Engines | Operating Conditions | Fault Modes |
|------|------|------|------|------|
| FD001 | 100 | 100 | 1 | HPC Degradation |
| FD002 | 260 | 259 | 6 | HPC Degradation |
| FD003 | 100 | 100 | 1 | HPC + Fan Degradation |
| FD004 | 248 | 249 | 6 | HPC + Fan Degradation |

Each engine starts with unknown initial wear and manufacturing variation, operates
normally at first, then gradually degrades until failure.  
Training sequences run until failure, while test sequences stop *before* failure,
requiring extrapolation of remaining life.

---

## Approach — How Would You Solve It?

The solution is structured as a **data analysis and modeling pipeline**, not just
a single predictive model.

### 1. Understanding the Data Generating Process
- Engines belong to a fleet, but differ in initial wear
- Sensor readings are noisy and influenced by operating conditions
- Fault onset is unobserved and gradual
- Test data ends prior to failure, mirroring real operational constraints

This framing informed all downstream modeling choices.

---

### 2. Exploratory Data Analysis (EDA)
EDA focused on identifying:
- Sensors with monotonic or degradation-related trends
- Variability across engines and early-life cycles
- Distribution shifts introduced by different operating conditions
- Noise vs. signal in individual sensors

Key insight:  
> Raw sensor values are not directly comparable across conditions or engines.

---

### 3. Feature Engineering
Feature engineering prioritized **robust degradation signals** over raw measurements:
- Normalization to reduce operating-condition effects
- Temporal aggregation (rolling statistics, trends)
- Emphasis on change over time rather than absolute magnitude

This allowed models to focus on *how behavior evolves* instead of static sensor values.

---

### 4. Modeling Strategy
RUL prediction was treated as a **supervised regression problem** over time-series data.

Modeling followed a layered approach:
- Baseline models to establish reference performance
- More expressive models to capture nonlinear degradation patterns
- Separate evaluation across FD001–FD004 to assess generalization

Performance was evaluated using RMSE, reflecting the cost of inaccurate life predictions.

---

## Tradeoffs — What’s Good and What’s Not?

### Strengths
- Pipeline explicitly accounts for real-world constraints (truncated test data)
- Feature engineering improves robustness across operating conditions
- Evaluation across multiple datasets highlights generalization behavior
- Clear separation between data understanding, modeling, and evaluation

### Limitations
- Models predict point estimates without uncertainty bounds
- Deep or complex models reduce interpretability
- Assumes historical degradation patterns remain stable over time
- Sensor importance is inferred indirectly, not explicitly modeled

These tradeoffs reflect realistic tensions between accuracy, interpretability,
and deployability.

---

## Outcome — What Happens in Production?

In a production setting, this pipeline would be used to:
- Continuously ingest sensor data from operating engines
- Generate updated RUL estimates per engine at each cycle
- Trigger maintenance alerts when RUL crosses predefined thresholds
- Support fleet-level planning and risk assessment

Key production considerations include:
- Monitoring data drift across operating conditions
- Periodic model retraining as new failure data becomes available
- Adding uncertainty estimates to support risk-aware decisions
- Integrating predictions into maintenance and operations workflows

This project demonstrates how a data analyst can move from raw sensor data
to actionable maintenance insights while maintaining awareness of real-world
constraints and tradeoffs.

---
## Pipeline Summary — How the System Works

This project implements a **two-layer, cluster-based regression pipeline** designed for
multivariate time-series sensor data, with the goal of improving Remaining Useful Life (RUL)
prediction under varying operating conditions.

### Layer 1: Behavior Segmentation via Time-Series Similarity
Engine trajectories are first analyzed based on their **temporal behavior patterns** rather
than raw sensor values.

- Feature-engineered sensor sequences are compared using **Dynamic Time Warping (DTW)**,
  allowing alignment of degradation trends that evolve at different speeds
- Engines with similar degradation dynamics are grouped into clusters
- This step separates heterogeneous operating behaviors that a single global model
  would struggle to capture

**Purpose:**  
Reduce variability caused by differing degradation rates and operating regimes.

---

### Layer 2: Cluster-Specific Weighted Regression
Within each cluster, a dedicated regression model is trained to predict RUL.

- Models are fit using **cluster-specific data**, improving local accuracy
- Observations closer to end-of-life are given higher importance via **weighted regression**
- This emphasizes late-stage degradation, which is most critical for maintenance decisions

**Purpose:**  
Improve prediction accuracy by learning localized degradation-to-failure relationships.

---

### Why This Works
By combining:
- Feature engineering to extract degradation signals
- Time-series alignment to compare engines fairly
- Clustering to handle heterogeneity
- Weighted regression to prioritize critical lifecycle stages

the pipeline balances **global structure** with **local specialization**, resulting in more
robust RUL predictions across engines with diverse operating profiles.




