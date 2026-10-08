# 🧠 AI Development & Perception Guidelines

> **Mandatory Team Policy:** All team members contributing machine learning models, computer vision pipelines, or autonomy algorithms must strictly abide by the prerequisite engineering standards, model export workflows, and fail-safe safety policies outlined below.

---

## ⚠️ Prerequisites: Read These First!

Before writing or integrating any AI / Machine Learning code in Xerox UAV, **you must first review and comply with our foundational engineering standards**:

1. [**`PYTHON_DEVELOPMENT.md`**](PYTHON_DEVELOPMENT.md):
   - You must manage projects, dependencies, and environments using **`uv`** (not classic pip/conda).
   - Use **`ruff`** for linting/formatting and **`ty`** (or `mypy`) for strict type annotations.
   - Use **frozen dataclasses** or **Pydantic V2** for structuring bounding boxes, detections, and telemetry. Raw dictionaries are strictly forbidden.
2. [**`SOFTWARE_DEVELOPMENT.md`**](SOFTWARE_DEVELOPMENT.md):
   - **Phase 2 (Concurrency):** Use `multiprocessing` / worker processes for CPU/GPU-heavy tensor operations to prevent locking I/O event loops.
   - **Phase 3 (Design Patterns):** Wrap models using the **Strategy** and **Factory** patterns so inference backends can be swapped seamlessly (e.g., `TorchBackend` vs. `TensorRTBackend` vs. `MockBackend`).
   - **Phase 4 (Clean Architecture):** Keep pure perception algorithms isolated from hardware drivers and external transport layers.
3. [**`GIT_GITHUB_INSTRUCTION.md`**](GIT_GITHUB_INSTRUCTION.md):
   - Adhere to Conventional Commits and branch naming conventions (`feat/ai-...`). Never commit large model weights (`.pt`, `.engine`) to Git; use **DVC**.

---

## 📖 Table of Contents
1. [Scope & Objectives](#-scope--objectives)
2. [Dual-Stage Model Workflow: Prototyping vs. Real-Time Edge](#-dual-stage-model-workflow-prototyping-vs-real-time-edge)
3. [Target Edge Hardware & Acceleration Stack](#-target-edge-hardware--acceleration-stack)
4. [Real-Time Performance & Latency Budgets](#-real-time-performance--latency-budgets)
5. [Dataset Management & Versioning (DVC)](#-dataset-management--versioning-dvc)
6. [Fail-Safe & The "Never Fly Blind" Policy](#-fail-safe--the-never-fly-blind-policy)
7. [Repository Layout & Contribution Guidelines](#-repository-layout--contribution-guidelines)

---

## 🎯 Scope & Objectives

The AI & Perception sub-team designs and deploys autonomous aerial vision systems for Xerox UAV quadcopters:
- Real-time target detection and visual tracking (landing pads, gates, search-and-rescue targets)
- Visual Odometry (VO) and SLAM for GPS-denied state estimation
- Monocular and stereo obstacle detection and depth mapping
- Embedded edge neural network inference on companion computers

---

## 🔄 Dual-Stage Model Workflow: Prototyping vs. Real-Time Edge

To maintain high research agility without sacrificing flight safety, we divide model workflows into two distinct tiers:

```
[ Tier 1: Local Development & Prototyping ]
  - PyTorch / TorchVision / Ultralytics
  - Native Checkpoints (.pt, .pth, safetensors)
  - Workstations & Laptops (CUDA or CPU)
                         ↓
           (Export & Validate via ONNX)
                         ↓
[ Tier 2: Real-Time & Edge Testing / Flight ]
  - ONNX Runtime & NVIDIA TensorRT (.engine)
  - FP16 & INT8 Quantization
  - Companion Computers (Jetson Orin, Edge NPUs)
  - Sub-33ms Latency Budgets
```

### 1. Local Prototyping & Development
- **Permitted Formats:** Engineers are free to use native PyTorch checkpoints (`.pt`, `.pth`), HuggingFace models, TorchScript, or Safetensors.
- **Workflow:** Rapid training, architecture experimentation, hyperparameter tuning, loss function validation, and offline metrics evaluation (mAP, F1-score) on developer workstations.
- **Flexibility:** Default framework inference APIs (e.g., `model(image)`) are fully permitted in local research scripts and offline notebooks.

### 2. Real-Time & Edge Deployment / Flight Testing
- **Forbidden in Flight:** Raw unoptimized PyTorch (`.pt` / `.pth`) weights must **NEVER** run in real-time closed-loop flight tasks due to unpredictable memory spikes and latency jitter.
- **Mandatory Optimized Formats:**
  - **ONNX (`.onnx`):** Intermediate portable representation for cross-platform bench verification.
  - **TensorRT (`.engine`):** Hardware-compiled inference engines optimized specifically for the target companion GPU (e.g. NVIDIA Jetson Orin).
- **Optimization Requirements:**
  - Export models with static or bounded dynamic batch sizes.
  - Quantize to **FP16** by default; apply calibrated **INT8** when higher frame rates are necessary.
  - Ensure zero memory leaks over extended 60-minute stress tests.

---

## ⚡ Target Edge Hardware & Acceleration Stack

### 1. Supported Platforms
- **Primary:** NVIDIA Jetson Orin Nano / Orin NX (JetPack 5.x / 6.x)
- **Secondary / Lightweight:** Raspberry Pi 5 with edge AI accelerator (Hailo-8 / Google Coral)

### 2. Approved Software Stack
- **Training & Prototyping:** PyTorch, TorchVision
- **Edge Inference Acceleration:** **TensorRT** (NVIDIA Jetson) & **ONNX Runtime**
- **Computer Vision:** OpenCV (C++ / Python bindings with CUDA acceleration where available)
- **Data & Serialization:** NumPy, Pydantic V2 (for bounding box and telemetry contracts)

---

## ⏱️ Real-Time Performance & Latency Budgets

AI running on a flying vehicle operates within a closed flight control loop. Latency spikes directly degrade flight stability:

| Pipeline Task | Target Frame Rate | Maximum End-to-End Latency |
| :--- | :--- | :--- |
| Obstacle Avoidance / Depth | **≥ 30 FPS** | **≤ 33 ms** |
| Target Tracking / Visual Servoing | **≥ 30 FPS** | **≤ 35 ms** |
| General Object Detection / Classification | **≥ 15 FPS** | **≤ 66 ms** |
| Visual SLAM / Odometry Pose Update | **≥ 20 Hz** | **≤ 50 ms** |

### Resource Budgets
- **VRAM / RAM Limit:** Total process memory must not exceed **60%** of available companion computer RAM to prevent Out-Of-Memory (OOM) operating system panics mid-flight.
- **Thermal Throttling:** Models must be profiled under sustained 30-minute loads to verify the companion computer does not throttle clock speeds due to excessive thermal output.

---

## 📦 Dataset Management & Versioning (DVC)

1. **No Datasets in Git:** Datasets and heavy weight files must NEVER be committed to Git repositories.
2. **Data Version Control (DVC):** Track all training, validation, and calibration datasets using **DVC** with remote storage backing (MinIO, S3, or team network drives).
3. **Aerial Domain Realism:**
   - Datasets must include real flight conditions:
     - Motion blur during high-speed roll/pitch maneuvers
     - Wide-angle and fisheye lens distortion
     - Aerial and high-angle perspectives
     - Propeller shadows, sun glare, and high-contrast shadows
4. **Independent Benchmark Set:** Maintain a locked, independent test set collected across distinct flight sessions to benchmark both `.pt` prototype and `.engine` production models.

---

## 🛡️ Fail-Safe & The "Never Fly Blind" Policy

Any neural network can fail, hallucinate, or lose visual tracking. Safety in aerial robotics demands deterministic handling of model degradation:

1. **Confidence Filtering:** Every detection must include a normalized confidence score $[0.0, 1.0]$. Detections falling below the safety threshold (default: $0.60$) must be discarded.
2. **Loss of Tracking Protocol:**
   - If a target or obstacle tracking algorithm drops detection for $> 3$ consecutive frames, emit an explicit `TRACKING_LOST` status event to the autonomy supervisor.
   - Never freeze, extrapolate wildly, or republish stale bounding box data.
3. **Heartbeat Signal to Flight Supervisor:**
   - The AI perception service must publish a continuous heartbeat signal at **10 Hz**.
   - If the companion computer freezes, crashes, or drops the heartbeat for $> 500\text{ ms}$, the flight controller must automatically trigger a safe fallback (e.g., Hold Position / Loiter, or Return-to-Launch).
4. **Decoupled Architecture:**
   - The camera ingestion pipeline, model inference worker, and command output publisher must run in decoupled worker processes/threads to ensure camera frame delays never stall the flight command pipeline.

---

## 📁 Repository Layout & Contribution Guidelines

AI repositories in Xerox UAV must follow this standardized layout:

```text
├── configs/          # YAML configs for hyperparameters, classes, and thresholds
├── data/             # DVC tracking files (.dvc)
├── models/           # Export scripts (PyTorch -> ONNX -> TensorRT engine)
├── src/
│   ├── core/         # Domain models, bounding box dataclasses, schemas
│   ├── inference/    # Factory & Strategy loaders (TorchBackend, TensorRTBackend)
│   ├── perception/   # Preprocessing, filtering, and tracking algorithms
│   └── pipeline/     # Execution runner, camera loop & telemetry bridge
├── tests/            # Unit tests for preprocessing & synthetic inference
└── Dockerfile        # Container for Jetson / CUDA environment reproduction
```

### Pre-Merge Checklist:
- [ ] Conforms to [`PYTHON_DEVELOPMENT.md`](PYTHON_DEVELOPMENT.md) (`uv`, `ruff`, `ty`, dataclasses/Pydantic).
- [ ] Conforms to [`SOFTWARE_DEVELOPMENT.md`](SOFTWARE_DEVELOPMENT.md) (Async/multiprocessing, Strategy/Factory patterns).
- [ ] Benchmark report included in PR description (showing FPS, latency in ms, and GPU memory usage on actual target hardware with TensorRT / ONNX).
- [ ] Synthetic test suite passes without errors.
- [ ] Model failure edge cases documented with graceful degradation handlers.
- [ ] Git commits follow [`GIT_GITHUB_INSTRUCTION.md`](GIT_GITHUB_INSTRUCTION.md).
