# 🐍 Python Development & Collaboration Guidelines

> **Mandatory Team Policy:** All team members working on Python codebases across Xerox UAV repositories must strictly follow and comply with the guidelines, modern toolchain standards, and verification requirements documented in this file.

---

## 📖 Table of Contents
1. [Core Learning Curriculum](#-core-learning-curriculum)
   - [Phase 1: Clean Foundations, Typing & Modern Tooling](#phase-1-clean-foundations-typing--modern-tooling)
2. [Modern Toolchain vs. Classic Methods](#-modern-toolchain-vs-classic-methods)
   - [1. uv vs. Classic pip / venv / poetry](#1-uv-vs-classic-pip--venv--poetry)
   - [2. ruff vs. Classic black / flake8 / isort / pylint](#2-ruff-vs-classic-black--flake8--isort--pylint)
   - [3. ty & Strict Typing vs. Untyped Python](#3-ty--strict-typing-vs-untyped-python)
   - [4. Dataclasses & Pydantic vs. Raw Dictionaries](#4-dataclasses--pydantic-vs-raw-dictionaries)
3. [Toolchain Usage & Commands](#-toolchain-usage--commands)
4. [UAV & Robotics Real-Time Best Practices](#-uav--robotics-real-time-best-practices)
5. [Testing & Quality Assurance](#-testing--quality-assurance)

---

## 🎓 Core Learning Curriculum

Every new and active member contributing Python code to Xerox UAV must study and master the concepts in this curriculum.

### Phase 1: Clean Foundations, Typing & Modern Tooling

- **Goal:** Stop bad Python habits, master strict typing, adopt structured domain modeling, and automate code quality using Astral's modern toolchain (`uv`, `ruff`, `ty`).
- **What to Learn:**
  - **Static Type Hints:** `typing.Protocol`, `TypedDict`, generics, `Optional`, and union operators (`|`) for strict static analysis.
  - **Structured Domain Modeling:** Clean modeling using frozen `dataclasses` (for zero-overhead internal state) and `Pydantic V2` (where runtime schema validation, data parsing, and serialization matter).
  - **Modern Project & Dependency Bootstrapping:** Managing Python versions, virtual environments, dependencies, and lockfiles via `uv`.
  - **Instant Linting, Formatting & Type Checking:** High-speed code quality enforcement using `ruff` and `ty` (or `mypy`/`pyright`).

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

## 🚀 Modern Toolchain vs. Classic Methods

In aerial robotics, software stability is directly tied to hardware survival. We have intentionally transitioned from the fragmented, classic Python ecosystem to a unified, modern toolchain. Below is why every team member must use these modern tools and their benefits over legacy workflows:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             XEROX UAV PYTHON STACK                          │
├──────────────────────┬─────────────────────────┬────────────────────────────┤
│ Capability           │ Modern Standard         │ Legacy Classic Approach    │
├──────────────────────┼─────────────────────────┼────────────────────────────┤
│ Package & Env Mgmt   │ uv                      │ pip + virtualenv + poetry  │
│ Linting & Formatting │ ruff                    │ flake8 + black + isort     │
│ Type Checking        │ ty / mypy               │ Dynamic / untyped Python   │
│ Data Modeling        │ dataclasses / Pydantic  │ Raw unstructured dicts     │
└──────────────────────┴─────────────────────────┴────────────────────────────┘
```

---

### 1. `uv` vs. Classic `pip` / `venv` / `poetry`

| Dimension | Classic Approach (`pip`, `virtualenv`, `poetry`) | Modern Standard (`uv`) |
| :--- | :--- | :--- |
| **Speed** | Slow resolution and installation (minutes for complex dependencies). | **10x–100x faster** written in Rust; resolutions complete in milliseconds. |
| **Tool Fragmentation** | Requires juggling `pip`, `venv`, `pip-tools`, `pyenv`, and `poetry`. | **Single unified binary** replacing all environment and package tools. |
| **Python Version Management** | Requires external tools like `pyenv` or OS package managers. | `uv` can automatically download and switch Python runtimes (`uv python install`). |
| **Lockfile Determinism** | `requirements.txt` often omits sub-dependencies or platform hashes. | Cross-platform, deterministic `uv.lock` ensures identical setups on dev laptops and companion computers. |
| **Disk Space Efficiency** | Duplicates wheels across every virtual environment on your drive. | Global content-addressable cache shares packages across all projects without copying. |

---

### 2. `ruff` vs. Classic `black` / `flake8` / `isort` / `pylint`

| Dimension | Classic Approach (`black` + `flake8` + `isort`) | Modern Standard (`ruff`) |
| :--- | :--- | :--- |
| **Tool Count** | 3 to 4 distinct tools with conflicting configurations and separate CI steps. | **One tool** for both linting and formatting. |
| **Execution Time** | Several seconds or minutes on large codebases; slows down git commit hooks. | **Sub-second execution** (often 10–50 ms), offering instant editor feedback. |
| **Auto-Fixing** | Limited fixes; requires manual intervention for many lint errors. | Automatically fixes hundreds of rule violations (`ruff check --fix`). |
| **Modern Syntax Upgrades**| Legacy manual refactoring of deprecated syntax. | Automatic codebase modernization via `UP` rules (e.g., converts `Dict[str, int]` to `dict[str, int]`). |

---

### 3. `ty` & Strict Typing vs. Untyped Python

In a desktop web app, a `TypeError` shows an error page. On an autonomous quadcopter, an unexpected `None` or missing attribute causes a crash from 30 meters in the sky.

- **Catch Bugs at Compile/Lint Time:** Static type checking catches typos, missing arguments, and `NoneType` attribute errors before the code is ever flashed onto the companion computer.
- **Self-Documenting Codebase:** Type signatures (`def arm(timeout: float) -> bool:`) immediately communicate function contracts without having to trace through lines of implementation code.
- **IDE Autocomplete & Refactoring:** Fully typed code gives developers rich autocomplete and safe renaming across the entire codebase.

---

### 4. Dataclasses & Pydantic vs. Raw Dictionaries

Classic Python robotics code often passes unstructured dictionaries:
```python
# ❌ Classic / Anti-Pattern: Fragile, error-prone, no autocomplete
telemetry = {"bat": 14.8, "armed": True, "pos": [10.0, 20.0]}
if telemetry["batery"] < 14.0:  # Typo fails silently until runtime crash!
    ...
```

Modern standard using structured models:
```python
# ✅ Modern Standard: Type-safe, autocompleted, validated
from dataclasses import dataclass
from pydantic import BaseModel, Field

# For internal high-rate loops: Zero-overhead frozen dataclass
@dataclass(frozen=True, slots=True)
class TelemetryState:
    battery_voltage: float
    is_armed: bool
    altitude_m: float

# For external telemetry/JSON/MAVLink: Validated Pydantic model
class MissionCommand(BaseModel):
    target_altitude: float = Field(gt=0.0, le=120.0)
    auto_rtl_on_low_battery: bool = True
```

- **Zero Silent Typos:** Field access (`telemetry.battery_voltage`) guarantees typos are caught immediately by `ty`/`mypy`.
- **Performance & Immutability:** `@dataclass(frozen=True, slots=True)` provides memory-efficient, immutable records that cannot be accidentally mutated in concurrent threads.
- **Runtime Validation:** Pydantic automatically validates ranges, datatypes, and missing fields when parsing incoming telemetry or ground station commands.

---

## 🛠️ Toolchain Usage & Commands

All Python projects in Xerox UAV must be configured with `pyproject.toml` and managed through `uv`:

```bash
# Initialize a new project with uv
uv init <project-name>

# Add project dependencies
uv add numpy opencv-python pydantic

# Add development tools
uv add --dev ruff ty pytest

# Format the codebase
uv run ruff format .

# Check and auto-fix linting issues
uv run ruff check . --fix

# Run static type checking
uv run ty .

# Run unit tests
uv run pytest tests/
```

### Recommended `pyproject.toml` Configuration:
```toml
[project]
name = "xerox-uav-module"
version = "0.1.0"
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
    "I",   # isort import sorting
    "UP",  # pyupgrade modern syntax
    "B",   # flake8-bugbear bug prevention
    "SIM", # flake8-simplify
]
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
- **Git & GitHub Guidelines:** For commit standards, PR requirements, and team collaboration workflows, refer directly to [`GIT_GITHUB_INSTRUCTION.md`](GIT_GITHUB_INSTRUCTION.md).
