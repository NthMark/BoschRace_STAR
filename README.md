# 🏎️ Bosch Race STAR — Top 5 Finish (CR24)

**STAR** is our team's autonomous driving stack for the **Bosch CR24 CodeRace** competition. Built on the **CARLA simulator**, this pipeline combines trajectory planning, lateral/longitudinal PID control, and an intelligent AEB (Automatic Emergency Braking) system to navigate complex driving scenarios safely and efficiently.

> 🏆 **Achievement: Top 5 Finalist — Bosch CodeRace 2024**

---

## 📸 Demo

[![Demo Video](https://img.shields.io/badge/Watch-Demo-red?logo=youtube)](https://drive.google.com/file/d/1Bu6l_fXiwiftrHQz41YbXn4cV0W5cobm/view?usp=sharing)

---

## 🧠 System Overview

The autonomous driving pipeline consists of three core modules working together in every simulation step:

```
          ┌─────────────┐
          │    Router   │
          │ (Reference  │
          │   Path)     │
          └──────┬──────┘
                 │
                 ▼
          ┌─────────────┐
          │   Planner   │
          │  (Cubic     │
          │   Spline)   │
          └──────┬──────┘
                 │
          ┌──────┴──────┐
          │             │
          ▼             ▼
   ┌──────────┐  ┌──────────┐
   │Lateral   │  │Longitudinal│
   │PID       │  │PID + AEB  │
   └────┬─────┘  └─────┬────┘
        │              │
        └──────┬───────┘
               ▼
        ┌──────────────┐
        │  VehicleControl │
        │ (steer, throttle, brake) │
        └──────────────┘
```

---

## 📂 Project Structure

```
CR24_STAR_R2/
├── __init__.py
├── baseline_driver.py          # Main driver — orchestrates planning & control
├── requirements.txt            # Dependencies (numpy)
├── controllers/
│   ├── __init__.py
│   ├── aeb_controller.py       # Automatic Emergency Braking logic
│   ├── pid_lat_controller.py   # Lateral control (steering) PID
│   └── pid_lon_controller.py   # Longitudinal control (throttle) PID
└── planners/
    ├── __init__.py
    └── cubic_spline_planner.py # Smooth trajectory generation via cubic splines
```

---

## 🧩 Components

### 1. `BaselineDriver` (`baseline_driver.py`)
The central orchestrator. At each simulation step it:
1. Retrieves the **reference path** from the router.
2. Generates a **smooth seed trajectory** using cubic spline interpolation.
3. Computes **steering** via the lateral PID controller.
4. Detects **lead vehicles** ahead and decides between throttle (cruise) and braking (AEB).

### 2. Lateral PID Controller (`controllers/pid_lat_controller.py`)
- **Purpose:** Steers the vehicle toward target waypoints.
- Uses a PID loop on the **cross-track angular error** between the vehicle heading and the target waypoint.
- Outputs a steering value in the range `[-1, 1]`.

### 3. Longitudinal PID Controller (`controllers/pid_lon_controller.py`)
- **Purpose:** Maintains a target cruising speed (10 m/s).
- Uses a PID loop on **speed error**.
- Outputs a throttle value in the range `[0, 1]`.

### 4. AEB Controller (`controllers/aeb_controller.py`)
- **Purpose:** Collision mitigation by detecting leading vehicles.
- Implements helper functions for **Time-to-Collision (TTC)**, stopping time calculations, and state-based braking logic:
  - **State 0:** No action
  - **State 1 (FCW):** Forward Collision Warning
  - **State 2 (PB1):** Partial braking @ 3.8 m/s²
  - **State 3 (PB2):** Partial braking @ 5.8 m/s²
  - **State 4 (FB):** Full braking @ 9.8 m/s²

### 5. Cubic Spline Planner (`planners/cubic_spline_planner.py`)
- **Purpose:** Generates a smooth, continuous trajectory from discrete reference waypoints.
- Implements 2D cubic spline interpolation with first and second derivative computation (for yaw and curvature).
- Based on the well-known implementation by Atsushi Sakai.

---

## 🚀 Getting Started

### Prerequisites
- [CARLA Simulator](https://carla.org/) (tested with CARLA 0.9.x)
- Python 3.7+

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/BoschRace_STAR.git
cd BoschRace_STAR/CR24_STAR_R2_1/CR24_STAR_R2

# Install dependencies
pip install -r requirements.txt
```

### Usage

The driver is designed to be used within the **Bosch CodeRace** competition framework. In your race simulation script:

```python
from CR24_STAR_R2.baseline_driver import BaselineDriver, get_driver

# The framework calls get_driver() to instantiate your driver
driver = BaselineDriver(router)

# Each simulation step, the framework calls:
control = driver.run(ego_pose, ego_dimension, ego_dynamics, npcs, timestamp)
# control is a carla.VehicleControl with steer, throttle, and brake values
```

---

## ⚙️ Key Parameters

| Parameter | Description | Default Value |
|-----------|-------------|---------------|
| `K_P` (lat) | Lateral proportional gain | 1.0 |
| `K_D` (lat) | Lateral derivative gain | 0 |
| `K_I` (lat) | Lateral integral gain | 0 |
| `K_P` (lon) | Longitudinal proportional gain | 1.0 |
| `K_D` (lon) | Longitudinal derivative gain | 1 |
| `K_I` (lon) | Longitudinal integral gain | 1 |
| Target speed | Cruising speed | 10 m/s |

---

## 🛠️ Future Improvements

- [ ] Integrate a more sophisticated **path planner** (e.g., lattice planner, RRT*).
- [ ] Replace hard-coded braking thresholds with dynamic TTC-based AEB.
- [ ] Add **velocity profiling** for smoother cornering.
- [ ] Implement **obstacle avoidance** with re-planning.
- [ ] Tune PID gains with systematic optimization (e.g., Ziegler–Nichols).

---

## 🙏 Acknowledgments

- **Bosch** for organizing the incredible CodeRace competition.
- **CARLA Team** for the open-source simulator.
- **Atsushi Sakai** for the original cubic spline planner implementation.
- All team members of **STAR** for their hard work and dedication.

---

## 📄 License

This project is for educational and competition purposes. All rights reserved.
