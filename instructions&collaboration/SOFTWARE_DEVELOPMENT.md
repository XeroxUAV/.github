# 💻 Software Development & Engineering Guidelines

> **Mandatory Team Policy:** Every software developer and robotics engineer across Xerox UAV repositories must strictly adhere and be completely loyal to the engineering standards, architecture patterns, verification hierarchy, and safety protocols defined in this document. Bypassing these architectural guidelines or skipping testing stages threatens vehicle survival and is strictly forbidden.

---

## 📖 Table of Contents
1. [Core Software Curriculum & Progression](#-core-software-curriculum--progression)
   - [Phase 2: Concurrency & Async vs. Sync](#phase-2-concurrency--async-vs-sync)
   - [Phase 3: Pythonic Design Patterns for AI & Robotics Systems](#phase-3-pythonic-design-patterns-for-ai--robotics-systems)
   - [Phase 4: Clean Architecture & Modular Project Layout](#phase-4-clean-architecture--modular-project-layout)
2. [Why Modern Architecture? Benefits Over Classic & Anti-Pattern Ways](#-why-modern-architecture-benefits-over-classic--anti-pattern-ways)
   - [1. Modern Concurrency vs. Classic Blocking Execution](#1-modern-concurrency-vs-classic-blocking-execution)
   - [2. Design Patterns vs. Spaghetti Conditionals & Hardcoded Logic](#2-design-patterns-vs-spaghetti-conditionals--hardcoded-logic)
   - [3. Clean Architecture (Hexagonal) vs. Monolithic Script Spaghetti](#3-clean-architecture-hexagonal-vs-monolithic-script-spaghetti)
3. [Technology Stack & ROS 2 Architecture](#-technology-stack--ros-2-architecture)
4. [C++ Real-Time & Systems Guidelines](#-c-real-time--systems-guidelines)
5. [Verification Hierarchy: The 6-Step Safety Ladder](#-verification-hierarchy-the-6-step-safety-ladder)
6. [Git & Collaboration Workflow](#-git--collaboration-workflow)

---

## 🎓 Core Software Curriculum & Progression

Building reliable robotics and aerial software requires moving beyond simple sequential scripting. Every member must study and master Phases 2, 3, and 4.

---

### Phase 2: Concurrency & Async vs. Sync

- **Goal:** Learn when to use asynchronous event loops versus threads versus multiprocessing. This is vital for robotics services handling concurrent telemetry streams, camera feeds, and heavy tensor workloads without locking the system.
- **Key Concepts:**
  - **CPU-bound vs. I/O-bound:** Knowing the difference between heavy computations (tensor operations, computer vision filters) and waiting for data (fetching MAVLink packets, camera frame capture, network requests).
  - **The Python GIL (Global Interpreter Lock):** When `asyncio` is the right choice vs. when `multiprocessing.Pool` or `concurrent.futures.ProcessPoolExecutor` is mandatory.
  - **Common Async Pitfalls:** Blocking the event loop with synchronous operations (e.g., `time.sleep()`, heavy NumPy matrix manipulations, or blocking file I/O).

#### 📺 Recommended Videos

| Topic | Video / Course | Channel / Instructor | Description |
| :--- | :--- | :--- | :--- |
| **Async Deep Dive** | [Async IO in Python: A Complete Walkthrough](https://www.youtube.com/watch?v=t5Bo1Je9EmE) | ArjanCodes | Comprehensive guide to event loops, coroutines, tasks, and non-blocking I/O patterns. |
| **Concurrency Models**| [Async vs Threading vs Multiprocessing in Python](https://www.youtube.com/watch?v=0kXaLh8Fz3k) | mCoding | Clear visual benchmarks explaining GIL bottlenecks, memory overhead, and when to use each model. |

---

### Phase 3: Pythonic Design Patterns for AI & Robotics Systems

- **Goal:** Prevent hardcoded logic, shotgun surgery, and fragile conditional chains when loading vision models, swapping communication backends, or configuring flight pipelines.
- **Key Concepts:**
  - **Strategy Pattern:** Swapping preprocessing, postprocessing, or control logic at runtime (e.g., switching Non-Maximum Suppression algorithms or path smoothing filters without modifying core nodes).
  - **Factory / Registry Pattern:** Dynamically creating models or backends (e.g., `TorchBackend`, `ONNXBackend`, `TensorRTBackend`, `MockBackend`) based on configuration files.
  - **Observer / Pub-Sub:** Clean event handling, telemetry dispatching, metrics collection, and progress reporting without tight coupling.
  - **Dependency Injection (DI):** Passing abstract interfaces and protocols into components rather than hardcoding class instantiations inside functions.

#### 📺 Recommended Videos

| Topic | Video / Course | Channel / Instructor | Why It Matters |
| :--- | :--- | :--- | :--- |
| **The Fast Overview (11m)** | [10 Design Patterns Explained in 10 Minutes](https://www.youtube.com/watch?v=tv-_1er1mWI) | Fireship | Fast-paced, visual, and entertaining overview of Factory, Singleton, Strategy, Observer, and Decorator. |
| **Core OOP Patterns (10m)** | [8 Design Patterns EVERY Developer Should Know](https://www.youtube.com/watch?v=tAuRQs_d9F8) | NeetCode | Whiteboard breakdowns of Factory, Adapter, Strategy, and Facade—indispensable for modular pipelines. |
| **Decoupling Creation (14m)**| [The Factory Pattern in Python // Separate Creation From Use](https://www.youtube.com/watch?v=s_4ZrtQs8Do) | ArjanCodes | Demonstrates idiomatic Python protocols and class factories separating "what to construct" from "how to execute". |
| **Full Architecture Course (1h 9m)**| [Design Patterns Tutorial \| Build Scalable Python Applications](https://www.youtube.com/watch?v=0fc6hvk4sZw) | Interview Simplified | Comprehensive, production-grade guide to building scalable, pattern-driven applications. |

---

### Phase 4: Clean Architecture & Modular Project Layout

- **Goal:** Strictly isolate pure robotics domain rules (state estimation, kinematics, flight mission logic) from volatile delivery mechanisms (ROS 2 nodes, FastAPI, CLI, MAVLink) and external infrastructure (databases, camera hardware drivers, GPU inference engines).
- **Key Concepts:**
  - **The Dependency Rule:** Dependencies point strictly inward. Core domain entities and algorithms must NEVER import external frameworks (no `rclpy`, `torch`, `cv2`, or database drivers inside domain logic).
  - **Ports & Adapters (Hexagonal Architecture):** Defining abstract interfaces (ports) for hardware/sensors and concrete implementations (adapters) that fulfill them.
  - **Layered Directory Layout:** Organizing packages cleanly into `domain/`, `application/`, `infrastructure/`, and `presentation/`.

#### 📺 Recommended Videos

| Topic | Video / Course | Channel / Instructor | Description |
| :--- | :--- | :--- | :--- |
| **Code Refactoring** | [From Spaghetti Code to Clean Python](https://www.youtube.com/watch?v=mH7e7fs9gaE) | ArjanCodes | Step-by-step refactoring of an entangled script into clean, cohesive modules. |
| **Project Structuring** | [Anatomy of a Scalable Python Project](https://www.youtube.com/watch?v=Af6Zr0tNNdE) | ArjanCodes | How to organize modern Python packages for long-term scalability and clean separation of concerns. |
| **Clean Architecture** | [Clean Architectures in Python](https://www.youtube.com/watch?v=qDTi1h3-Rvs) | Leonardo Giordani | Author of *Clean Architectures in Python* explaining the core tenets of Hexagonal architecture in Python. |

---

## 🚀 Why Modern Architecture? Benefits Over Classic & Anti-Pattern Ways

In mission-critical UAV systems, poor software design directly causes dropped telemetry, thread lockups, frozen camera pipelines, and catastrophic physical crashes. Every member must understand the concrete benefits of our modern architectural standards:

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                 XEROX UAV SOFTWARE DESIGN                               │
├─────────────────────┬────────────────────────────────────┬──────────────────────────────┤
│ Engineering Concern │ Modern Standard                    │ Classic Anti-Pattern         │
├─────────────────────┼────────────────────────────────────┼──────────────────────────────┤
│ Concurrency         │ AsyncIO for I/O + Multiprocessing  │ Blocking synchronous loops   │
│ Model/Backend Mgmt  │ Factory & Strategy Patterns        │ Giant nested if/elif ladders │
│ System Architecture │ Clean Architecture (Ports/Adapters)│ Monolithic mixed script      │
│ Component Wiring    │ Dependency Injection & Protocols   │ Hardcoded class imports      │
└─────────────────────┴────────────────────────────────────┴──────────────────────────────┘
```

---

### 1. Modern Concurrency vs. Classic Blocking Execution

#### ❌ The Classic Anti-Pattern:
- Writing synchronous, blocking loops (`while True: read_camera(); compute_yolo(); send_mavlink(); time.sleep(0.05)`).
- **The Failure:** If `compute_yolo()` experiences a latency spike (e.g. 150 ms instead of 30 ms), the entire loop stalls. MAVLink heartbeats are dropped, the flight controller assumes companion computer loss, and triggers an emergency landing or RTL.

#### ✅ Modern Standard (AsyncIO + Worker Multiprocessing):
- **Non-blocking Event Loops:** Async event loops handle telemetry streaming, serial I/O, and WebSocket ground communication without ever stalling.
- **Dedicated Worker Processes:** Heavy tensor operations and OpenCV image filtering run in isolated subprocesses (`ProcessPoolExecutor`) that bypass Python's GIL.
- **Benefit:** Real-time telemetry never drops, sensor buffers never overflow, and CPU cores on Jetson/RPi are utilized to 100% efficiency.

---

### 2. Design Patterns vs. Spaghetti Conditionals & Hardcoded Logic

#### ❌ The Classic Anti-Pattern:
- Hardcoding models and backends inside functions with giant `if/elif/else` blocks:
  ```python
  # ❌ Fragile, hard to test, violates Open/Closed Principle
  def run_detection(frame, backend_type):
      if backend_type == "torch":
          import torch
          # 20 lines of torch inference
      elif backend_type == "tensorrt":
          import tensorrt
          # 25 lines of tensorrt logic
      elif backend_type == "onnx":
          # ...
  ```

#### ✅ Modern Standard (Factory, Strategy & Registry Patterns):
- **Strategy Pattern:** Define a strict interface (`Protocol`):
  ```python
  from typing import Protocol
  import numpy as np

  class DetectorStrategy(Protocol):
      def detect(self, image: np.ndarray) -> list[Detection]: ...
  ```
- **Factory / Registry Pattern:** Concrete implementations (`TensorRTDetector`, `ONNXDetector`, `MockDetector`) register with a central factory. The main node requests an implementation via configuration.
- **Benefits:**
  1. **Zero Risk to Flight Logic:** Adding a new neural network architecture never requires modifying existing flight state machine code.
  2. **Instant Mocking:** Developers can switch to `MockDetector` in 1 line of configuration, running full integration tests on laptops without an NVIDIA Jetson or CUDA GPU.

---

### 3. Clean Architecture (Hexagonal) vs. Monolithic Script Spaghetti

#### ❌ The Classic Anti-Pattern:
- Mixing ROS 2 node publishers, OpenCV image capture, business decisions, and file logging inside a single 800-line monolithic script.
- **The Failure:** You cannot test whether your obstacle avoidance logic works without starting ROS 2 daemons, connecting a physical camera, and spawning Gazebo. Unit testing is impossible.

#### ✅ Modern Standard (Clean / Hexagonal Architecture):
We split our software into four distinct, isolated layers:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. PRESENTATION LAYER (Delivery)                            │
│    ROS 2 Nodes, CLI commands, Ground Station REST APIs      │
├─────────────────────────────────────────────────────────────┤
│ 2. APPLICATION LAYER (Use Cases & Orchestration)            │
│    Mission Supervisors, Trajectory Generators, State Machine│
├─────────────────────────────────────────────────────────────┤
│ 3. DOMAIN LAYER (Pure Mathematical Rules & Entities)        │
│    Kinematic calculations, Waypoints, Obstacle geometry     │
├─────────────────────────────────────────────────────────────┤
│ 4. INFRASTRUCTURE LAYER (Hardware & Adapters)               │
│    RealSense camera drivers, Jetson TensorRT, SQLite logs   │
└─────────────────────────────────────────────────────────────┘
```

- **The Core Rule:** Dependencies point inward. The `Domain` layer contains pure Python / C++ with ZERO dependencies on ROS 2, PyTorch, or OpenCV.
- **Benefits:**
  1. **Blazing-Fast Unit Tests:** Domain mission algorithms can be tested in **3 milliseconds** with `pytest` without ROS 2 or hardware running.
  2. **Extreme Portability:** If the team switches from ROS 2 to Zenoh or PX4 Micro-DDS in the future, 90% of our domain and mission logic remains untouched—we only swap the presentation adapter!

---

## 🛠️ Technology Stack & ROS 2 Architecture

### Core Middleware & Languages
- **ROS 2:** Standardized on **ROS 2 Humble Hawksbill** (LTS) / **Iron Irwini**.
- **C++:** Modern C++ (C++17 or C++20). Adhere to the **Google C++ Style Guide**.
- **Python:** Python 3.10+, complying with the modern toolchain in [`PYTHON_DEVELOPMENT.md`](PYTHON_DEVELOPMENT.md).
- **Build System:** `colcon` with `ament_cmake` (C++) and `ament_python` (Python).

### ROS 2 Architectural Rules
1. **Node Lifecycle:** Use `rclcpp_lifecycle` (Lifecycle Nodes) for mission-critical nodes to ensure deterministic state transitions (`configure`, `activate`, `deactivate`, `cleanup`).
2. **Quality of Service (QoS) Profiles:**
   - **High-rate sensor streams (IMU, Odometry, Camera at >30 Hz):** Use **Best Effort** reliability and **Volatile** durability to minimize latency.
   - **Critical commands & state transitions (Arm, Disarm, Mode Switch, Waypoint):** Use **Reliable** reliability and **Transient Local** durability.
3. **Execution & Spinners:** Avoid single-threaded spinners if a node handles both high-frequency sensor callbacks and heavy computation. Use `MultiThreadedExecutor` with isolated callback groups (`MutuallyExclusiveCallbackGroup`).

---

## ⚡ C++ Real-Time & Systems Guidelines

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

## 🪜 Verification Hierarchy: The 6-Step Safety Ladder

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

## 🤝 Git & Collaboration Workflow

### Branching Strategy
- `master` / `main`: Production-ready, flight-tested code only.
- `develop`: Integration branch for tested features.
- `feature/<feature-name>`: Working branch for individual features.
- `fix/<bug-name>`: Bug fixes for reported issues.

### Conventional Commit Messages
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

### Code Review & Pull Request Requirements
- Every PR requires at least **one approved review** from a sub-team lead.
- PRs must link to an open Issue tracking the requirement.
- Author must provide Gazebo/SITL test logs or video screen recordings demonstrating successful behavior before physical bench authorization.
