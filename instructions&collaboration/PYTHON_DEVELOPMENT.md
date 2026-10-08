# 🐍 Python Development & Collaboration Guidelines

> **Mandatory Team Policy:** All team members working on Python codebases across Xerox UAV repositories must strictly follow and comply with the guidelines, formatting rules, and verification standards documented in this file.

---

## 1. Scope & Purpose

Python is heavily utilized in the Xerox UAV ecosystem for:
- ROS 2 Python nodes (telemetry, state observers, high-level mission supervisors)
- Computer vision pipelines and AI inference wrappers
- MAVLink / MAVSDK telemetry streaming and ground script utilities
- Data parsing, flight log analysis, and simulation test harnesses

Writing clean, deterministic, typed, and well-tested Python code is critical to mission reliability and flight safety.

---

## 2. Environment & Dependency Standards

1. **Python Version:** All projects must target **Python 3.10+** (standardized across our ROS 2 Humble/Iron environments).
2. **Virtual Environments:** Never install packages directly into your global system Python. Always use isolated environments:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
3. **Dependency Specification:**
   - For standalone libraries/tools: Maintain a `pyproject.toml` or pinned `requirements.txt` / `requirements-dev.txt`.
   - For ROS 2 packages: Specify dependencies cleanly in `package.xml` and `setup.py`.
   - Never commit ad-hoc or unpinned package installations.

---

## 3. Code Style & Formatting Rules

To maintain codebase consistency, all Python repositories must pass automated linting and formatting.

| Tool | Purpose | Standard Configuration |
| :--- | :--- | :--- |
| **Black** / **Ruff** | Auto-formatting | Line length: `88` characters |
| **Flake8** / **Ruff** | Linting & style checks | PEP 8 compliance, zero undefined variables |
| **isort** | Import sorting | Alphabetical, separated into Standard, Third-party, and Local |
| **mypy** | Static type checking | Strict typing on all public functions and interfaces |

### 3.1. Static Typing Policy
All function signatures must include explicit type hints for both arguments and return values:

```python
from typing import Optional, Tuple
from dataclasses import dataclass

@dataclass(frozen=True)
class Waypoint:
    latitude: float
    longitude: float
    altitude_m: float

def compute_target_distance(
    current_pos: Waypoint, 
    target_pos: Waypoint
) -> float:
    """Compute the Euclidean distance between two 3D waypoints in meters."""
    dx = target_pos.latitude - current_pos.latitude
    dy = target_pos.longitude - current_pos.longitude
    dz = target_pos.altitude_m - current_pos.altitude_m
    return (dx**2 + dy**2 + dz**2) ** 0.5
```

### 3.2. Docstring Standard
All modules, classes, and non-trivial functions must feature **Google Style** docstrings:
```python
def arm_vehicle(timeout_sec: float = 5.0) -> bool:
    """Sends an arming command to the flight controller and verifies arm state.

    Args:
        timeout_sec: Maximum wait time in seconds before aborting arming attempt.

    Returns:
        True if vehicle successfully entered ARMED state, False otherwise.

    Raises:
        ConnectionError: If MAVLink telemetry heartbeat is lost.
    """
```

---

## 4. UAV & Robotics Real-Time Best Practices

1. **Never Block Real-Time Loops:**
   - Avoid `time.sleep()` in any high-rate or control callback.
   - Use non-blocking event loops (`asyncio`) or ROS 2 rate timers (`node.create_timer(interval, callback)`).
2. **Memory & Garbage Collection:**
   - Minimize high-frequency allocations within loops operating at >20 Hz (e.g. avoid creating new large lists or dictionaries inside IMU callbacks).
   - Reuse pre-allocated NumPy buffers for camera frames and sensor batches.
3. **Threading vs. Multiprocessing:**
   - Due to Python's Global Interpreter Lock (GIL), heavy CPU tasks (such as image filtering or neural net inference) must run in a separate process (`multiprocessing`), a C++ ROS node, or offloaded via C extensions (e.g., OpenCV, TensorRT).
   - Use threading only for I/O-bound operations (e.g., serial telemetry logging, socket communication).
4. **Defensive Error Handling:**
   - Always catch specific exceptions (never bare `except:`).
   - If a sensor read or MAVLink message fails, log the anomaly and trigger safe default behavior (e.g., zero velocities or hold position).

---

## 5. Testing & Quality Assurance

- **Framework:** Use `pytest` for all unit and integration testing.
- **Test Structure:** Place tests in a dedicated `tests/` directory mirroring the source structure.
- **Mocking:** Always mock physical hardware, serial ports, and live network streams during unit tests:
  ```python
  from unittest.mock import MagicMock
  
  def test_arm_command_timeout():
      mock_drone = MagicMock()
      mock_drone.is_armed.return_value = False
      # Test timeout handling without requiring physical flight controller
  ```
- **Coverage Goal:** At least 80% branch coverage on core logic, mission state machines, and calculations.

---

## 6. Pull Request (PR) Checklist

Before submitting a PR for any Python repository in Xerox UAV:
- [ ] Code formatted with `black` / `ruff`.
- [ ] No linting warnings (`flake8` / `ruff check .` returns 0).
- [ ] Type checks pass cleanly (`mypy .`).
- [ ] All unit tests pass (`pytest tests/`).
- [ ] Any new feature or bug fix includes corresponding unit tests.
- [ ] Docstrings provided for all new functions and public APIs.
