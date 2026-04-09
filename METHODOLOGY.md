# Methodology: City-Level EV Swap Charging Load Engine

This document outlines the step-by-step mathematical and simulation methodology used to estimate the grid load and energy consumption of a city-wide electric vehicle (EV) battery swapping network.

---

## 1. Overview
The engine follows a discrete-event simulation approach at a **15-minute resolution**. It models the flow of batteries from the moment a user arrives at a station to the point where the swapped battery is fully charged and the load is cleared from the grid.

**Core Pipeline:**
`Demand Generation` → `Station Allocation` → `Queueing (M/M/c/K)` → `Charging` → `Load Aggregation`

---

## 2. Step-by-Step Methodology

### Step 1: Demand Generation
The simulation begins by generating the total city-wide swap demand for a 24-hour period.
*   **Source**: A normalized 24-hour historic curve is used as a template.
*   **Stochasticity**: For each 15-minute interval $t$, the number of battery arrivals follows a **Poisson Distribution**.
*   **Formula**:
    $$N_{city}(t) \sim \text{Poisson}(\lambda_{city}(t))$$
    Where $\lambda_{city}(t)$ is the expected number of arrivals at time $t$, derived from the user-defined morning and evening peaks.

### Step 2: Station Allocation
The total city demand is distributed among the available stations ($M$).
*   **Assumption**: Demand is uniformly distributed across the network.
*   **Formula**:
    $$\lambda_i(t) = \lambda_{city}(t) \times w_i$$
    Where $w_i = \frac{1}{M}$ for all stations.

### Step 3: Queueing Engine (M/M/c/K)
Each station is modeled as an **M/M/c/K queue**, where:
*   **c**: Number of chargers per station.
*   **K**: Total system capacity (Chargers + Waiting spots).
*   **Logic**:
    *   If a charger is free, the battery starts charging immediately.
    *   If all chargers are busy but the queue is not full ($< K$), the battery enters a First-Come-First-Served (FCFS) queue.
    *   If the queue is full ($= K$), the arrival is "dropped" and does not contribute to the load.

### Step 4: Charging Model
The engine assumes a constant-current constant-voltage (CCCV) profile simplified to a linear charging model for research-grade load estimation.
*   **Energy Delivered ($E_{bat}$)**: Fixed at 80% of total capacity (e.g., swapping a 20% SOC battery for 100%).
    $$E_{bat} = 0.8 \times E_{max}$$
*   **Charge Duration ($T_{charge}$)**: Calculated based on the charger's power rating ($P_{charger}$).
    $$T_{charge} = \frac{E_{bat}}{P_{charger}}$$

### Step 5: Load Aggregation (Core Output)
The grid load is the instantaneous power drawn by all active chargers across the entire city.
*   **Active Chargers**: The sum of all chargers across all $M$ stations currently in the "Charging" state at time $t$.
*   **Formula**:
    $$Load(t) = \sum_{i=1}^{M} (\text{active\_chargers}_i(t) \times P_{charger})$$

### Step 6: Energy Consumption
The total daily energy consumed by the network is the integral of the load curve over the 24-hour period.
*   **Formula**:
    $$E_{total} = \sum_{t=1}^{96} Load(t) \times \Delta t$$
    Where $\Delta t = 0.25$ hours (15 minutes).

---

## 3. Summary of Parameters

| Parameter | Symbol | Description |
| :--- | :--- | :--- |
| **Morning Peak** | $\lambda_M$ | Target arrivals/15min at 09:00 |
| **Evening Peak** | $\lambda_E$ | Target arrivals/15min at 18:30 |
| **Number of Stations** | $M$ | Total facilities in the city |
| **Chargers per Station** | $c$ | Number of charging slots per facility |
| **Charger Power** | $P$ | Power rating per slot in kW |
| **Battery Capacity** | $E_{max}$| Total capacity of the battery pack in kWh |
| **Queue Multiplier** | $K/c$ | Buffer size relative to charger count |

---

## 4. Key Performance Indicators (KPIs)

1.  **Peak Load (kW)**: The maximum value observed on the aggregated $Load(t)$ curve.
2.  **Load Factor**: Measures the "flatness" of the load curve.
    $$\text{Load Factor} = \frac{\text{Average Load}}{\text{Peak Load}}$$
3.  **Utilization**: Percentage of the total installed capacity ($M \times c \times P$) actually used over 24 hours.
