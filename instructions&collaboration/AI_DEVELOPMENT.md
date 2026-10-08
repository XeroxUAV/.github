# 🧠 AI Development & Perception Guidelines

> **Mandatory Team Policy:** All team members contributing machine learning models, computer vision pipelines, or autonomy algorithms must strictly abide by the real-time performance constraints, data versioning rules, and fail-safe safety policies outlined below.

---

## 1. Scope & Objectives

The AI & Perception sub-team designs and deploys autonomous capabilities for Xerox UAV quadcopters:
- Real-time object detection and tracking (target landing, gate detection, search and rescue)
- Visual Odometry (VO) and Visual SLAM for GPS-denied navigation
- Monocular / Stereo obstacle avoidance and depth estimation
- Edge neural network deployment on companion computers

UAV onboard computers operate under extreme power, thermal, and weight constraints. Models must be computationally lean, ultra-low latency, and defensively integrated with the flight control system.

---

## 2. Target Edge Hardware & Frameworks

### 2.1. Supported Platforms
- **Primary:** NVIDIA Jetson Orin Nano / Orin NX (JetPack 5.x / 6.x)
- **Secondary / Lightweight:** Raspberry Pi 5 with edge AI accelerator (Hailo-8 / Google Coral)

### 2.2. Approved Software Stack
- **Deep Learning Framework:** PyTorch (for research, training, and model architecture)
- **Edge Deployment & Acceleration:** **TensorRT** (for NVIDIA Jetson) or **ONNX Runtime**
- **Vision Libraries:** OpenCV (C++ / Python bindings, CUDA-enabled where available)
- **Middleware:** ROS 2 (rclpy / rclcpp) for publisher/subscriber communication with flight systems

---

## 3. Real-Time Performance & Latency Budgets

AI on a flying platform operates in a closed control loop. High inference latency causes state estimation lag and instability:

| Pipeline Task | Target Frame Rate | Maximum End-to-End Latency |
| :--- | :--- | :--- |
| Obstacle Avoidance / Depth | **≥ 30 FPS** | **≤ 33 ms** |
| Target Tracking / Visual Servoing | **≥ 30 FPS** | **≤ 35 ms** |
| General Object Detection / Classification | **≥ 15 FPS** | **≤ 66 ms** |
| Visual SLAM / Odometry Pose Update | **≥ 20 Hz** | **≤ 50 ms** |

### 3.1. Optimization & Quantization Requirements
- **No Raw PyTorch in Flight:** Raw unoptimized PyTorch `.pt` files must NEVER run in real-time flight loops.
- **Mandatory TensorRT Export:** Models must be converted:
  $$\text{PyTorch} \longrightarrow \text{ONNX} \longrightarrow \text{TensorRT Engine (.engine)}$$
- **Quantization:** Standardize on **FP16** precision for Jetson Orin; utilize **INT8** calibration when FPS targets require additional throughput.
- **Memory Footprint:** Monitor VRAM usage with `jtop` / `tegrastats`. Total GPU memory consumption must not exceed 60% of total available companion computer RAM to prevent Out-Of-Memory (OOM) system panics mid-flight.

---

## 4. Dataset Management & Annotation Standards

1. **Dataset Versioning:**
   - Datasets must not be stored in standard git repositories. Use **DVC (Data Version Control)** or centralized team storage (e.g., MinIO / AWS S3 buckets).
   - Tag datasets with semantic versions (`dataset-gates-v1.2.0`).
2. **Aerial Domain Challenges:**
   - Datasets must account for real quadcopter flight dynamics:
     - Severe motion blur during high-speed yaw/pitch changes
     - Lens distortion from wide-angle / fisheye lenses
     - Top-down / oblique camera angles
     - Propeller shadows, sun glint, and harsh lighting transitions
3. **Validation Split:**
   - Reserve at least 20% of data for a strictly independent test set recorded across different flight sessions and lighting conditions.

---

## 5. Fail-Safe & The "Never Fly Blind" Policy

Any AI model can fail, hallucinate, or lose visual tracking. Safety in aerial robotics requires deterministic handling of model degradation:

1. **Confidence Filtering:** Every detection output must include a normalized confidence score $[0.0, 1.0]$. Detections below the safety threshold (default: $0.60$) must be discarded.
2. **Loss of Tracking Protocol:**
   - If a target or obstacle tracking algorithm drops detection for $> 3$ consecutive frames, emit an explicit `TRACKING_LOST` status event to the autonomy supervisor.
   - Do not freeze or republish stale bounding box data.
3. **Heartbeat Signal to Flight Controller:**
   - The AI ROS 2 node must publish a continuous heartbeat topic at 10 Hz.
   - If the companion computer freezes, crashes, or drops heartbeat for $> 500\text{ ms}$, the flight controller must automatically trigger a safe fallback (e.g., Hold Position / Loiter, or Return-to-Launch).
4. **Decoupled Architecture:**
   - The camera capture, inference engine, and flight command publisher must run in decoupled worker threads or separate ROS 2 nodes to ensure camera driver delays do not stall the command pipeline.

---

## 6. Repository Structure & Contribution Rules

AI repositories in Xerox UAV must follow this standardized layout:
```text
├── configs/          # YAML configs for hyperparameters and thresholds
├── data/             # DVC tracking files (.dvc)
├── models/           # Exported ONNX / TensorRT engine generation scripts
├── src/
│   ├── inference/    # Model wrapper & TensorRT engine loader
│   ├── perception/   # Preprocessing, filtering, and tracking logic
│   └── ros_nodes/    # ROS 2 wrapper nodes
├── tests/            # Unit tests for preprocessing & synthetic inference
└── Dockerfile        # Reproducible container for Jetson / CUDA environment
```

Before merging:
- [ ] Benchmark report included in PR description (showing FPS, latency in ms, GPU memory usage on actual target hardware).
- [ ] Synthetic test suite passes without errors.
- [ ] Model failure edge cases documented with graceful degradation handlers.
