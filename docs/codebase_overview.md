# Codebase Overview

## Method

```
New Task
   │
   ├─► Retrieval
   │       Match by language instruction + initial observation similarity (DINOv2 + CLIP)
   │
   ├─► Abstraction
   │       A VLM selects object-relational distance pairs from the retrieved rollout
   │       Sub-trajectory segmentation by velocity thresholds
   │
   ├─► Alignment
   │       Match the live scene to the reference trajectory in distance space (DTW)
   │
   └─► Guidance injection
           Loss on deviation from the reference distance trends
           Gradient through a differentiable dynamics model (OSC integrator / UR5 FK)
           Golden-section line search on the step size with a do-no-harm guard
           Relevance gate: a relation activates when the end-effector approaches its objects
```

## Pipeline

```
retrieval → abstraction → alignment → guidance → policy action correction
```

| Module | Role |
|---|---|
| `core/retrieval` | Retrieve a relevant demo via DINOv2 / CLIP embeddings + task similarity |
| `core/abstraction` | Extract distance pairs and their reference trends from the retrieved demo |
| `core/alignment` | Align the live timeline to the demo via DTW |
| `core/guidance` | Compute the differentiable guidance gradients and line search |
| `perception/` | Open-vocabulary detection, segmentation, tracking, 3D reconstruction |
| `features/` | Modular distance / geometric feature extractors |
| `vlm/` | VLM-based distance-pair selection and demo-to-scene object correspondence |
| `policies/` | Policy-server clients (OpenPI, GR00T, LeRobot) |
| `envs/` | Environment client/server (LIBERO-PRO simulator, real UR5e cell) |

## Project Structure

```
src/policy_abstraction/
├── core/              # Pipeline phases: retrieval → abstraction → alignment → guidance
├── evaluation/        # Episode runner, benchmarks, live rollout orchestration
├── perception/        # Object detection, segmentation, 3D reconstruction, tracking
├── features/          # Modular distance/geometric feature extractors
├── vlm/               # VLM-based distance-pair selection and object correspondence
├── retrieval/         # Embedding-based demo retrieval (DINOv2 + CLIP)
├── analysis/          # Offline demo analysis (sub-trajectories, distance pairs)
├── policies/          # Policy client wrappers (OpenPI, GR00T, LeRobot)
├── envs/              # Environment client/server (LIBERO-PRO, real UR5e)
├── dynamics/          # Differentiable forward dynamics models
├── experiments/       # Operator-driven real-robot experiment tooling
├── demo/              # Live guidance demo GUI for the UR5e
├── calibration/       # Camera calibration for the real cell
├── visualization/     # Rerun 3D viewer, alignment plots, video annotation
├── data/              # Dataset loaders, trajectory recorders (.vla format)
└── utils/             # Point clouds, depth, gripper kinematics, transforms

packages/
├── policy_steering/   # Guidance functions, line search, differentiable kinematics
└── open_perception/   # Open-vocabulary multi-camera segmentation and tracking (vendored)
```

## Scripts

The main entry point is [`scripts/evaluate.py`](../scripts/evaluate.py). It
orchestrates the full evaluation loop: launches policy and environment
servers, retrieves demonstrations, runs guided rollouts with perturbations,
and saves videos, episodes, and a JSON summary.

| Script | Purpose |
|---|---|
| `scripts/evaluate.py` | **Main evaluation** — run tasks (LIBERO-PRO, real UR5e) with guided policy steering |
| `scripts/perturbation_sweep.py` | Run the LIBERO-PRO perturbation matrix from a sweep config (`configs/sweeps/`) |
| `scripts/experiments/collect_experiment.py` | **Operator-driven collection** — many rollouts against one loaded policy, with a grid overlay and hotkeys (see [`docs/real_experiments.md`](real_experiments.md)) |
| `scripts/experiments/run_cable_position_experiment.sh` | The positional distribution-shift sweep on the UR5e, baseline vs guided |
| `scripts/experiments/run_object_disambiguation_experiment.sh` | The object-generalization experiment on the UR5e |
| `scripts/run_ur5_demo.sh` | **Live guidance demo** — one loaded policy, guidance switched on and off from a window (see [`docs/demo.md`](demo.md)) |
| `scripts/encode_datasets.py` | Encode demos with DINOv2 + CLIP embeddings (`*_embeddings.npz`) |
| `scripts/replay_episode.py` | Replay recorded `.vla` episodes in the Rerun viewer |
| `scripts/servers/serve_libero_env.py` | WebSocket server for the LIBERO-PRO simulator |
| `scripts/servers/serve_ur5_env.py` | WebSocket server for a real UR5e cell (see [`docs/ur5.md`](ur5.md)) |
| `scripts/servers/serve_groot.py` | Inference server for GR00T N1.7 checkpoints |
| `scripts/servers/serve_lerobot.py` | Inference server for LeRobot pi0.5 checkpoints (real robot) |
