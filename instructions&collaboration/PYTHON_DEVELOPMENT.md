# 🐍 Python Development & Collaboration Guidelines

> **Mandatory Team Policy:** All team members working on Python codebases across Xerox UAV repositories must strictly follow and comply with the guidelines, modern toolchain standards, and verification requirements documented in this file.

---

## 📖 Table of Contents
1. [Core Learning Curriculum](#-core-learning-curriculum)
   - [Phase 1: Clean Foundations, Typing & Modern Tooling](#phase-1-clean-foundations-typing--modern-tooling)
2. [Modern Toolchain: Astral Suite (uv, ruff, ty)](#-modern-toolchain-astral-suite-uv-ruff-ty)
3. [Domain Modeling: Dataclasses & Pydantic](#-domain-modeling-dataclasses--pydantic)
4. [UAV & Robotics Real-Time Best Practices](#-uav--robotics-real-time-best-practices)
5. [Testing & Quality Assurance](#-testing--quality-assurance)
6. [Pull Request (PR) Checklist](#-pull-request-pr-checklist)

---

## 🎓 Core Learning Curriculum

Every new and active member contributing Python code to Xerox UAV must complete this phased training curriculum.

### Phase 1: Clean Foundations, Typing & Modern Tooling

- **Goal:** Stop bad Python habits, master strict typing, use dataclasses / Pydantic, and automate code standards with Astral's modern toolchain (`uv`, `ruff`, `ty`).
- **Key Concepts:**
  - Static type hints (`typing.Protocol`, `TypedDict`, generics, `Optional`, `Union`) and strict static analysis.
  - Modern domain modeling with frozen `dataclasses` (or `Pydantic` where runtime schema validation and serialization matter).
  - Project bootstrapping and dependency management with `uv` (`uv init`, `uv add`, deterministic lockfiles), replacing slow `pip`/`venv`.
  - Blazing-fast formatting and linting via `ruff` + type checking via `ty` (or `mypy`/`pyright`).

#### 📺 Recommended Videos & Courses

| Topic | Video / Course | Channel / Instructor |
| :--- | :--- | :--- |
| **Package Management** | [Python Tutorial: UV - A Faster, All-in-One Package Manager](https://www.youtube.com/watch?v=AMdG7IjgSPM) | Corey Schafer |
| **Linting & Formatting** | [Python Tutorial: Ruff - Fast Linter & Formatter](https://www.youtube.com/watch?v=828S-DMQog8) | Corey Schafer |
| **Type Checking** | [ty - Python type-checker from Astral](https://www.youtube.com/watch?v=aDViJRLQr30) | BugBytes |
| **Domain Modeling** | [Python dataclasses will save you HOURS](https://www.youtube.com/watch?v=vBH6GRJ1REM) | mCoding |
| **Data Validation** | [Pydantic V2 Crash Course & Tutorial](https://www.youtube.com/watch?v=7aBRk_JP-qY) | BugBytes |
| **Clean Code & Habits**| [25 nooby Python habits you need to ditch](https://www.youtube.com/watch?v=qUeud6DvOWI) | mCoding |

---

### 🛠️ Hands-on Task: Task 1

> **Objective:** Refactor an untyped, legacy UAV telemetry & inference script into a modern, production-grade module using `uv`, `ruff`, `ty`, and `dataclasses` / `pydantic`.

#### Step-by-Step Instructions:
1. **Initialize Project:**
   ```bash
   uv init uav-inference-task
   cd uav-inference-task
   uv add pydantic
   uv add --dev ruff ty pytest
   ```

2. **Configure `pyproject.toml` with strict Ruff rules and Ty:**
   ```toml
   [project]
   name = "uav-inference-task"
   version = "0.1.0"
   description = "Xerox UAV Telemetry and Inference Pipeline"
   readme = "README.md"
   requires-python = ">=3.10"
   dependencies = [
       "pydantic>=2.7.0",
   ]

   [tool.ruff]
   line-length = 88
   target-version = "py310"

   [tool.ruff.lint]
   select = [
       "E",   # pycodestyle errors
       "F",   # pyflakes
       "I",   # isort (import sorting)
       "UP",  # pyupgrade (modern python syntax)
       "B",   # flake8-bugbear (common bug patterns)
       "SIM", # flake8-simplify
   ]
   ```

3. **The Challenge Script:**
   Below is the untyped, unformatted practice script with bad habits (mutable defaults, raw dictionaries, bare `except`, no types, missing f-strings, `range(len(...))` anti-patterns). Save this file as `legacy_pipeline.py`:

```python
import os, sys, json, math, time
from typing import *

def calculate_distance(p1, p2):
    # calculate Euclidean distance
    d = math.sqrt((p1["x"]-p2["x"])**2 + (p1["y"]-p2["y"])**2 + (p1["z"]-p2["z"])**2)
    return d

def load_flight_log(filepath):
    try:
        f = open(filepath, "r")
        raw = f.read()
        f.close()
        data = json.loads(raw)
        return data
    except:
        print("Failed to load file")
        return None

def filter_detections(detections, min_confidence=0.5, valid_classes=[]):
    valid_classes.append("obstacle")
    results = []
    for i in range(len(detections)):
        item = detections[i]
        if item["conf"] >= min_confidence:
            if item["label"] in valid_classes:
                results.append(item)
    return results

def compute_bounding_box_center(bbox):
    xmin = bbox[0]
    ymin = bbox[1]
    xmax = bbox[2]
    ymax = bbox[3]
    return {"cx": (xmin + xmax) / 2, "cy": (ymin + ymax) / 2}

def run_mock_inference(frame_batch, model_weights="weights.bin"):
    output = []
    for i in range(0, len(frame_batch)):
        frame = frame_batch[i]
        simulated_det = {
            "track_id": i + 100,
            "label": "gate",
            "conf": 0.88,
            "bbox": [12.0, 14.5, 120.0, 180.0]
        }
        output.append(simulated_det)
    return output

def evaluate_uav_state(telemetry, targets=[]):
    status = {}
    if telemetry.has_key("battery"):
        if telemetry["battery"] < 14.8:
            status["warning"] = "LOW_BATTERY"
    if telemetry.get("armed") == True:
        status["state"] = "ARMED"
    else:
        status["state"] = "DISARMED"
    return status

def main():
    mock_log = '{"telemetry": {"battery": 15.2, "armed": true, "pos": {"x": 10.0, "y": 20.0, "z": 5.0}}, "detections": [{"label": "gate", "conf": 0.92, "bbox": [0,0,50,50]}, {"label": "tree", "conf": 0.3, "bbox": [10,10,20,20]}]}'
    with open("sample_log.json", "w") as fp:
        fp.write(mock_log)
    
    data = load_flight_log("sample_log.json")
    if data != None:
        filtered = filter_detections(data["detections"], 0.6, ["gate"])
        print("Filtered detections: " + str(filtered))
        state = evaluate_uav_state(data["telemetry"])
        print("State: " + str(state))

if __name__ == "__main__":
    main()
```

4. **Task Requirements:**
   - Convert all raw dictionaries (`p1`, `p2`, `telemetry`, `detection`, `bbox`) into frozen **`@dataclass(frozen=True)`** or **Pydantic V2 `BaseModel`**.
   - Eliminate all mutable default arguments (`valid_classes=[]`, `targets=[]`).
   - Eliminate `range(len(...))` and replace with direct iteration or `enumerate()`.
   - Remove bare `except:` and manual file closes (use `with open(...)`).
   - Add complete type annotations across all function arguments and returns.
   - Run and ensure **zero warnings** with:
     ```bash
     uv run ruff check . --fix
     uv run ruff format .
     uv run ty .
     ```

---

## ⚡ Modern Toolchain: Astral Suite (`uv`, `ruff`, `ty`)

We have standardized our Python environment on Astral's high-performance Rust-based toolchain:

| Tool | Role | Why We Use It in Xerox UAV |
| :--- | :--- | :--- |
| **`uv`** | Package & Environment Manager | 10-100x faster than `pip` and `poetry`. Deterministic cross-platform lockfiles (`uv.lock`). |
| **`ruff`** | Linter & Formatter | Replaces `flake8`, `black`, `isort`, and `pylint` in a single unified, ultra-fast tool. |
| **`ty`** / **`mypy`** | Static Type Checker | Catches null-pointer bugs, mismatched return types, and schema drift before flights. |

### Commands Every Member Must Use:
```bash
# Add a new dependency and update lockfile
uv add numpy opencv-python

# Add a development tool
uv add --dev ruff ty pytest

# Run linter checks
uv run ruff check .

# Auto-format codebase
uv run ruff format .

# Run static type checker
uv run ty .
```

---

## 📦 Domain Modeling: Dataclasses & Pydantic

Raw Python dictionaries (`{"x": 1, "y": 2}`) are strictly forbidden in core UAV modules. They lack schema enforcement, autocomplete, and type safety.

### When to Use `@dataclass(frozen=True)`
Use frozen dataclasses for internal, high-frequency mathematical domain structures where zero serialization overhead is desired:

```python
from dataclasses import dataclass

@dataclass(frozen=True, slots=True)
class Position3D:
    x_m: float
    y_m: float
    z_m: float

@dataclass(frozen=True, slots=True)
class FlightAttitude:
    roll_rad: float
    pitch_rad: float
    yaw_rad: float
```

### When to Use Pydantic V2 (`BaseModel`)
Use Pydantic when parsing external data: JSON telemetry packets, MAVLink configuration YAMLs, ground station REST payloads, or AI inference outputs that require runtime validation:

```python
from pydantic import BaseModel, Field

class BoundingBox(BaseModel):
    xmin: float = Field(ge=0.0)
    ymin: float = Field(ge=0.0)
    xmax: float
    ymax: float

class DetectionResult(BaseModel):
    track_id: int
    label: str
    confidence: float = Field(ge=0.0, le=1.0)
    bbox: BoundingBox
```

---

## ⏱️ UAV & Robotics Real-Time Best Practices

1. **Never Block Real-Time Loops:**
   - Avoid `time.sleep()` in any high-rate telemetry or control callback.
   - Use non-blocking event loops (`asyncio`) or ROS 2 rate timers (`node.create_timer(interval, callback)`).
2. **Memory & Garbage Collection:**
   - Avoid high-frequency allocations within loops operating at $> 20\text{ Hz}$ (e.g. avoid creating new large lists or dictionaries inside IMU callbacks).
   - Reuse pre-allocated NumPy buffers for camera frames and sensor batches.
3. **Threading vs. Multiprocessing:**
   - CPU-bound tasks (image filtering, neural net inference) must run in a separate process (`multiprocessing`), a C++ ROS node, or offloaded via C extensions (OpenCV, TensorRT).
   - Use threading only for I/O-bound operations (serial telemetry logging, socket communication).
4. **Defensive Error Handling:**
   - Always catch specific exceptions (never bare `except:`).
   - If a sensor read or MAVLink message fails, log the anomaly and trigger safe default behavior (e.g., zero velocities or hold position).

---

## 🧪 Testing & Quality Assurance

- **Framework:** Use `pytest` for all unit and integration testing.
- **Test Structure:** Place tests in a dedicated `tests/` directory mirroring the source structure.
- **Mocking:** Always mock physical hardware, serial ports, and live network streams during unit tests:
  ```python
  from unittest.mock import MagicMock
  import pytest
  
  def test_arm_command_timeout():
      mock_drone = MagicMock()
      mock_drone.is_armed.return_value = False
      assert not mock_drone.is_armed()
  ```
- **Coverage Goal:** At least 80% branch coverage on core logic, mission state machines, and calculations.

---

## ✅ Pull Request (PR) Checklist

Before submitting a PR for any Python repository in Xerox UAV:
- [ ] Environment configured and managed with `uv`.
- [ ] Code formatted with `uv run ruff format .`.
- [ ] All linting checks pass with zero warnings (`uv run ruff check .`).
- [ ] Static type checker passes cleanly (`uv run ty .` or `uv run mypy .`).
- [ ] Dataclasses / Pydantic models used instead of raw unstructured dictionaries.
- [ ] All unit tests pass (`uv run pytest tests/`).
- [ ] Google-style docstrings provided for all new functions and public APIs.
