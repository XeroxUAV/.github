# 🚁 Xerox UAV Team

> **Welcome to Xerox UAV!**  
> We are an aerial robotics team building autonomous multirotor platforms (quadcopters) for research, competitions, and autonomous aerial exploration.

---

## 🌟 About Our Organization

Xerox UAV brings together engineers across artificial intelligence, systems software, flight dynamics, embedded firmware, custom electronics, and mechanical design to build high-performance, robust autonomous quadcopters.

---

## 🛠️ Sub-Teams & Core Disciplines

| Sub-Team | Core Focus | Key Technologies |
| :--- | :--- | :--- |
| 🧠 **AI & Perception** | Computer vision, target tracking, visual SLAM, edge inference | OpenCV, PyTorch, TensorRT, ONNX, Jetson Orin |
| 💻 **Software & Autonomy** | Clean architecture, state machines, telemetry, simulation | Modern C++, Python, Gazebo Sim, MAVLink |
| 🛠️ **Hardware & Mechanical** | Airframe design, structural FEA, propulsion testing, 3D printing & CNC | Fusion 360, SolidWorks, Carbon Fiber |
| ⚡ **PCB Design & Electronics** | Custom PDBs, flight controller shields, power regulators, EMI shielding | KiCad 8+, Altium, SMPS design |
| 🎯 **Control & Flight Dynamics** | 6-DoF dynamics modeling, state estimation (EKF), PID/MPC control loops | MATLAB/Simulink, Python, PX4 Tuning |
| 🎛️ **Control Boards & Firmware**| Flight controllers, low-level drivers (SPI/I2C/CAN), RTOS firmware | STM32, FreeRTOS, NuttX, PX4 Autopilot |

---

## ⚡ Engineering Standards & Toolchain

Our organization enforces modern, reliable engineering workflows across all repositories:
- **Git & Collaboration:** Standardized on Conventional Commits, GitHub Flow, and strict PR reviews.
- **Python Stack:** Astral's modern toolchain (**`uv`**, **`ruff`**, **`ty`**) with frozen dataclasses and Pydantic V2.
- **AI Acceleration:** Dual-stage workflow using PyTorch for local prototyping and hardware-compiled **TensorRT / ONNX** for sub-33ms edge inference.
- **Software Architecture:** Clean / Hexagonal Architecture separating pure flight logic from delivery protocols.
- **Hardware Rigor:** 4-layer PCB design standards, TVS / reverse-polarity protection, and zero-DRC policy.

---

## 📚 Onboarding & Team Instructions

All team members must adhere to our standardized guidelines:
- 📖 **[Root Repository README](../README.md):** Complete team overview, role roadmaps, and course recommendations.
- 📋 **Domain Guidelines in [`instructions&collaboration/`](../instructions&collaboration/):**
  - [🐙 **Git & GitHub Guidelines**](../instructions&collaboration/GIT_GITHUB_INSTRUCTION.md) – Conventional commits, branch naming, and PR workflows.
  - [🐍 **Python Development Guidelines**](../instructions&collaboration/PYTHON_DEVELOPMENT.md) – `uv`, `ruff`, `ty`, and real-time robotics best practices.
  - [⚡ **PCB Design & Hardware Guidelines**](../instructions&collaboration/PCB_DESIGN.md) – KiCad rules, power routing, and bench bring-up safety.
  - [🧠 **AI & Perception Guidelines**](../instructions&collaboration/AI_DEVELOPMENT.md) – Prototyping vs. TensorRT/ONNX edge deployment, latency budgets, and DVC datasets.
  - [💻 **Software Engineering Guidelines**](../instructions&collaboration/SOFTWARE_DEVELOPMENT.md) – Concurrency (async/sync), design patterns, and Clean Architecture.

---

### 🛡️ Safety Mantra
> **The Golden Propeller Rule:** Propellers remain strictly **OFF** during all laboratory development, code debugging, and bench testing.
