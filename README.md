# Adaptive Traffic Signal Optimization Agent

An intelligent traffic-management system that adapts traffic signal timings to changing vehicle demand. The project combines rule-based traffic control, a Proximal Policy Optimization (PPO) reinforcement-learning agent, simulation, and a safety supervisor to explore ways of reducing unnecessary waiting at intersections.

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black" alt="React">
  <img src="https://img.shields.io/badge/TypeScript-Frontend-3178C6?logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Reinforcement_Learning-PPO-7B61A8" alt="PPO">
  <img src="https://img.shields.io/badge/Simulation-SUMO%20%2F%20JunctionSimulator-555555" alt="Simulation">
</p>

---

## Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Signal Control Strategies](#signal-control-strategies)
- [How It Works](#how-it-works)
- [Safety and Emergency Handling](#safety-and-emergency-handling)
- [Technology Stack](#technology-stack)
- [Getting Started](#getting-started)
- [Using the Application](#using-the-application)
- [Reinforcement Learning Details](#reinforcement-learning-details)
- [API Overview](#api-overview)
- [Testing](#testing)
- [Project Documentation](#project-documentation)
- [License](#license)

## Project Overview

Conventional traffic signals often use fixed schedules. A fixed schedule cannot always respond to changing traffic: one road may have a long queue while another has little or no traffic.

The **Adaptive Traffic Signal Optimization Agent** is designed to make signal timing responsive to traffic conditions. It models an intersection, observes traffic demand, selects a signal phase and green duration, and tracks measures such as queue length, waiting time, and vehicle throughput.

The application includes a dashboard for running simulations, comparing control strategies, and testing emergency-vehicle priority.

> **Project scope:** This is a simulation and decision-support project. Its results depend on the selected traffic data, simulation assumptions, and evaluation settings. It is not a certified controller for deployment on public roads.

## Features

### Adaptive signal control

- Adjusts green-light allocation according to observed traffic demand.
- Considers queue lengths, vehicle arrivals, and waiting time.
- Includes both rule-based control and a trained PPO reinforcement-learning policy.

### Strategy comparison

- Runs multiple signal-control strategies against a common traffic-demand replay.
- Compares average waiting time, queue length, throughput, and related measures.
- Provides a comparison report that can be exported in JSON or CSV format.

### Live intersection simulation

- Displays the current signal phase and remaining signal time.
- Shows traffic queues and simulation progress.
- Supports starting, pausing, and resetting a simulation.

### Emergency vehicle priority

- Supports priority requests for ambulances, fire engines, and police vehicles.
- Coordinates a transition to the requested approach.
- Uses yellow and all-red clearance stages before granting the priority phase.

### Safety supervision

- Prevents conflicting signal phases from being active at the same time.
- Enforces configured minimum and maximum green durations.
- Preserves yellow and all-red transition intervals.
- Includes safeguards intended to reduce excessive waiting on an approach.

### Traffic data and simulation

- Supports a SUMO integration and a fallback junction simulator.
- Includes traffic-data documentation and a data dictionary.
- Uses Passenger Car Unit (PCU) values to represent different vehicle classes where configured.

## Signal Control Strategies

The comparison module includes four strategies:

| Strategy | Category | Description |
| --- | --- | --- |
| **Fixed-Time** | Baseline | Uses predetermined signal timings without adapting to current demand. |
| **Vehicle Actuated** | Sensor / demand driven | Uses the presence of vehicles or queues to decide whether green time should continue. |
| **Dynamic Adaptive** | Rule-based adaptive | Allocates green time using traffic conditions such as queue length and waiting time. |
| **Deep AI (PPO)** | Reinforcement learning | Uses a PPO policy to select a signal phase and green duration from the environment observation. |

The strategies are intended to be evaluated under the same traffic-demand scenario so that their results can be compared fairly.

## How It Works

At a high level, the system follows this loop:

1. **Observe traffic:** Read the current queues, arrival rates, waiting delays, and signal state.
2. **Select a strategy:** Use the chosen controller to decide which phase should receive green time and for how long.
3. **Check safety:** Pass the requested action through the safety supervisor.
4. **Advance the simulation:** Apply the signal state and update vehicle movement.
5. **Measure performance:** Record traffic measures and make them available to the dashboard and comparison reports.

### System architecture

The project is organized into three main parts:

- **Frontend:** A React dashboard for simulation controls, strategy comparison, traffic visualization, and emergency requests.
- **Backend:** A FastAPI service that manages the simulation, controllers, safety logic, emergency requests, and API responses.
- **Traffic and learning environment:** The simulation adapter and Gymnasium environment used by the control strategies and PPO agent.

For a deeper explanation, see [ARCHITECTURE.md](ARCHITECTURE.md).

## Safety and Emergency Handling

Signal optimization must not override basic intersection safety. The project therefore separates the controller's requested action from the safety supervisor that validates signal transitions.

The documented configuration uses:

| Setting | Value |
| --- | ---: |
| Minimum green duration | 10 seconds |
| Maximum green duration | 55 seconds |
| Yellow clearance | 3 seconds |
| All-red clearance | 2 seconds |

When an emergency request is received, the system transitions through the configured clearance stages before granting priority. The supervisor also prevents conflicting approaches from receiving green simultaneously.

These timings are simulation settings, not a substitute for a road-authority-approved traffic-signal plan.

## Technology Stack

| Area | Technologies |
| --- | --- |
| Backend API | Python, FastAPI, Uvicorn, Pydantic |
| Frontend | React, TypeScript, Vite |
| Styling and charts | Tailwind CSS, Recharts, Lucide |
| Reinforcement learning | Gymnasium, Stable-Baselines3, PPO, PyTorch |
| Numerical and data tools | NumPy, Pandas, Scikit-learn |
| Traffic simulation | SUMO, TraCI, Sumolib, JunctionSimulator fallback |
| Testing | Pytest |

## Getting Started

### Prerequisites

Install the following before running the project:

- Python 3.11 or later
- Node.js 18 or later
- npm
- Git

SUMO is required for the SUMO-backed simulation path. The project also documents a fallback simulator.

### 1. Clone the repository

```bash
git clone https://github.com/Japnoor2407/Adaptive-Traffic-Signal-Optimization-Agent-.git
cd Adaptive-Traffic-Signal-Optimization-Agent-
```

### 2. Set up the Python environment

Create and activate a virtual environment from the repository root.

**Windows PowerShell**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the backend dependencies:

```bash
pip install -r requirements.txt
```

If your checkout does not contain a root-level `requirements.txt`, use the dependency file provided in the backend directory or the project's environment documentation.

### 3. Configure environment variables

Review `.env.example` and create a local `.env` file if the application requires environment-specific values.

Do not commit credentials, API keys, or private configuration values.

### 4. Start the backend

From the repository root, run:

```bash
python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000
```

Once the server has started:

- API base: http://127.0.0.1:8000
- Health endpoint: http://127.0.0.1:8000/api/health
- Interactive API documentation: http://127.0.0.1:8000/docs

### 5. Start the frontend

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite, normally:

http://localhost:5173

## Using the Application

### Compare the strategies

1. Open the dashboard.
2. Go to the strategy comparison area.
3. Select a simulation duration and traffic condition.
4. Run the comparison.
5. Review the available performance measures and export the report if needed.

### Run an intersection simulation

1. Open the live intersection view.
2. Select a control strategy.
3. Start the simulation.
4. Observe the signal phase, countdown, and traffic queues.
5. Pause or reset the simulation as required.

### Test emergency priority

1. Open the emergency vehicle priority controls.
2. Select the emergency vehicle type and its approach.
3. Submit the priority request.
4. Observe the transition and clearance stages before the priority phase is activated.

The exact labels and available controls may vary with the current frontend implementation.

## Reinforcement Learning Details

The project defines a custom Gymnasium environment named `TrafficSignalEnv` and uses Stable-Baselines3 PPO for policy training.

### Observation space

The documented observation is a normalized vector with **21 values**:

| Observation | Count |
| --- | ---: |
| Queue lengths by approach | 4 |
| Vehicle arrival rates by approach | 4 |
| Mean waiting delays by approach | 4 |
| Starvation timers by approach | 4 |
| Active phase encoding | 2 |
| Elapsed green duration | 1 |
| Intersection density | 1 |
| Episode progress | 1 |
| **Total** | **21** |

### Action space

The documented action space is `Discrete(16)`. Each action combines one of two phases (North–South or East–West) with one of eight green-duration choices:

`10, 15, 20, 25, 30, 40, 50, 55 seconds`.

### Reward function

The environment documents the following reward structure:

```text
R = 0.35 × ΔQ
  + 0.30 × T
  - 0.20 × W
  - 0.10 × S
  - 0.05 × P_switch
  - 0.05 × P_balance
```

Where:

- `ΔQ`: queue-clearance contribution
- `T`: throughput contribution
- `W`: waiting-delay penalty
- `S`: starvation penalty
- `P_switch`: penalty for excessive phase switching
- `P_balance`: penalty related to imbalance between approaches

### Train the PPO agent

The documented training command is:

```bash
python -m backend.rl.train --timesteps 100000 --episode-duration 900 --checkpoint-freq 10000
```

Check the training script's command-line help and configuration before changing parameters. The documented checkpoint location is:

```text
backend/models/rl/ppo_traffic_latest.zip
```

Training performance should be assessed with repeatable evaluation episodes and compared against the non-learning baselines. A training reward by itself does not demonstrate reduced real-world traffic congestion.

## API Overview

The backend exposes REST endpoints for health checks, controller selection, simulation, traffic state, benchmarking, emergency requests, safety events, and PPO training.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/api/health` | Check backend health. |
| GET | `/api/controller/modes` | List available controller modes. |
| POST | `/api/controller/mode` | Select a controller mode. |
| POST | `/api/controller/step` | Advance the controller by one simulation step. |
| POST | `/api/simulation/start` | Start a simulation session. |
| POST | `/api/simulation/pause` | Pause or resume the simulation. |
| POST | `/api/simulation/reset` | Reset simulation state. |
| GET | `/api/traffic/current` | Retrieve current traffic and signal state. |
| GET | `/api/traffic/history` | Retrieve traffic history. |
| POST | `/api/lab/run` | Run a strategy comparison. |
| GET | `/api/lab/history` | List comparison sessions. |
| GET | `/api/lab/report/{id}` | Retrieve a comparison report. |
| GET | `/api/lab/export/{id}` | Export a comparison report. |
| POST | `/api/emergency/request` | Submit an emergency-priority request. |
| POST | `/api/emergency/clear` | Clear an active emergency priority. |
| GET | `/api/emergency/status` | Retrieve the active emergency status. |
| GET | `/api/emergency/events` | Retrieve emergency event history. |
| GET | `/api/safety/events` | Retrieve safety-supervisor event history. |
| POST | `/api/rl/train` | Start PPO training. |
| GET | `/api/rl/training/status` | Retrieve PPO training status. |

For request schemas and interactive endpoint testing, start the backend and visit http://127.0.0.1:8000/docs.

## Testing

The repository README previously documented the following test command:

```bash
pytest backend/tests/test_system.py backend/tests/test_rl_environment.py backend/tests/test_sumo_integration.py backend/tests/test_emergency_lab.py -v
```

Run it from the repository root after installing the project dependencies. Test availability and results depend on the files and dependencies in your local checkout.

## Project Documentation

- [Architecture](ARCHITECTURE.md) — system components and how they interact.
- [Dataset profile](DATASET_PROFILE.md) — information about the traffic dataset.
- [Data dictionary](data_dictionary.csv) — descriptions of dataset fields.
- [.env.example](.env.example) — example environment configuration.

## Contributing

Contributions, bug reports, and suggestions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make and test your changes.
4. Commit with a clear message.
5. Open a pull request describing the change.

For substantial changes, open an issue first to discuss the proposed approach.

## License

This project is distributed under the MIT License. See the [LICENSE](LICENSE) file for details, if present in the repository.
