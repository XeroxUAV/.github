# 💻 Software Development & Engineering Guidelines

> **Mandatory Team Policy:** Every software developer and robotics engineer across Xerox UAV repositories must strictly adhere to the engineering standards, verification hierarchy, and safety protocols defined in this document.

---

## 1. Scope & Objectives

The Software Engineering sub-team builds the core robotics backbone of Xerox UAV:
- Robot Operating System (ROS 2) application layer and node management
- Autonomy mission logic, state machines, and path planners
- Communication bridges (MAVLink, Micro-XRCE-DDS, MAVROS, Zenoh)
- Ground Control Station (GCS) integration and real-time telemetry streaming
- Simulation environments (Gazebo Sim, PX4 SITL, ArduPilot SITL)

Software stability directly impacts drone survival. Code crashes in mid-air can lead to uncontrolled flyaways or high-velocity impacts.

---

## 2. Technology Stack & Standards

### 2.1. Core Middleware & Languages
- **ROS 2:** Standardized on **ROS 2 Humble Hawksbill** (LTS) / **Iron Irwini**.
- **C++:** Modern C++ (C++17 or C++20). Adhere to the **Google C++ Style Guide**.
- **Python:** Python 3.10+, complying with the team's [`PYTHON_DEVELOPMENT.md`](PYTHON_DEVELOPMENT.md).
- **Build System:** `colcon` with `ament_cmake` (C++) and `ament_python` (Python).

### 2.2. ROS 2 Architectural Rules
1. **Node Lifecycle:** Use `rclcpp_lifecycle` (Lifecycle Nodes) for mission-critical nodes to ensure deterministic state transitions (`configure`, `activate`, `deactivate`, `cleanup`).
2. **Quality of Service (QoS) Profiles:**
   - **High-rate sensor streams (IMU, Odometry, Camera at >30 Hz):** Use **Best Effort** reliability and **Volatile** durability to minimize network latency and prevent packet queues from lagging.
   - **Critical commands & state transitions (Arm, Disarm, Mode Switch, Waypoint):** Use **Reliable** reliability and **Transient Local** durability.
3. **Execution & Spinners:** Avoid single-threaded spinners if a node handles both high-frequency sensor callbacks and heavy computation. Use `MultiThreadedExecutor` with isolated callback groups (`MutuallyExclusiveCallbackGroup`).

---

## 3. C++ Real-Time & Systems Guidelines

1. **Memory Management & RAII:**
   - Raw pointers (`*`) and manual `delete` are strictly forbidden for resource ownership.
   - Use smart pointers: `std::unique_ptr` by default, `std::shared_ptr` only when ownership is genuinely shared.
2. **Zero Dynamic Allocation in Control Loops:**
   - Avoid `new`, `malloc`, or resizing `std::vector` inside high-frequency control callbacks (>50 Hz). Pre-allocate all buffers in the node configuration step.
3. **Static Analysis & Sanitizers:**
   - Every C++ package must compile with `-Wall -Wextra -Wpedantic -Werror`.
   - Run `clang-tidy` and `cppcheck` before submitting PRs.
   - Test under AddressSanitizer (`-fsanitize=address`) and UndefinedBehaviorSanitizer (`-fsanitize=undefined`) during SITL simulation.

---

## 4. Verification Hierarchy (The 6-Step Safety Ladder)

Every code change must climb the verification ladder step-by-step. Skipping steps is a direct violation of team policy:

```
[ Step 1: Unit Tests & Static Analysis ]
                   ↓
[ Step 2: Continuous Integration (CI) Pass ]
                   ↓
[ Step 3: SITL Simulation (Gazebo + PX4/ArduPilot) ]
                   ↓
[ Step 4: Hardware-In-The-Loop (HITL) Simulation ]
                   ↓
[ Step 5: Physical Bench Testing (PROPELLERS REMOVED!) ]
                   ↓
[ Step 6: Supervised Flight Field Test ]
```

### Safety Rules for Physical Testing:
- **Rule 1 (The Golden Propeller Rule):** NEVER plug a battery into a quadcopter on the lab bench with propellers mounted while debugging code. Propellers are mounted ONLY at the outdoor flight field immediately prior to flight.
- **Rule 2 (Failsafe Verification):** Verify that software watchdog timers and telemetry loss triggers properly command the drone into **RTL (Return To Launch)** or **Emergency Land**.

---

## 5. Git & Collaboration Workflow

### 5.1. Branching Strategy
- `master` / `main`: Production-ready, flight-tested code only.
- `develop`: Integration branch for tested features.
- `feature/<feature-name>`: Working branch for individual features.
- `fix/<bug-name>`: Bug fixes for reported issues.

### 5.2. Conventional Commit Messages
Format all commit messages strictly:
```text
<type>(<scope>): <short description>

[optional body explaining why this change was made]
```
- `feat`: A new feature or capability
- `fix`: A bug fix
- `refactor`: Code reorganization with no behavior change
- `test`: Adding or updating tests
- `docs`: Documentation updates
- `ci`: CI/CD pipeline modifications

### 5.3. Code Review & Pull Request Requirements
- Every PR requires at least **one approved review** from a sub-team lead.
- PRs must link to an open Issue tracking the requirement.
- Author must provide Gazebo/SITL test logs or video screen recordings demonstrating successful behavior.
