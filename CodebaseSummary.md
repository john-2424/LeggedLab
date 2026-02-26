# LeggedLab Codebase — Detailed, Relevance-Categorized Inventory

This document provides a **repo-wide inventory** and explanation of **every folder and file** in this repository, grouped by practical relevance to development and operation.

## Relevance model

- **R1 — Runtime Critical**: Directly controls simulator behavior, environment stepping, observations, rewards, training, or inference.
- **R2 — Configuration & Embodiment Critical**: Robot embodiment/configuration assets and terrain definitions that strongly affect dynamics and learning results.
- **R3 — Packaging/Project Operations**: Installation, docs, licensing, and templates.
- **R4 — VCS/Internal Metadata**: Git internals needed for source control history but not runtime behavior.

---

## 1) Folder inventory (every folder)

### R1 / R2 / R3 folders (project content)
- `./` — repository root containing package, docs, and packaging metadata.
- `./.github` — project meta files; currently contains shared license header template.
- `./legged_lab` — top-level Python package.
- `./legged_lab/assets` — robot asset namespace and path constants.
- `./legged_lab/assets/fftai` — FFTAI robot module namespace.
- `./legged_lab/assets/fftai/gr2` — GR2 robot asset package.
- `./legged_lab/assets/fftai/gr2/configuration` — layered USD composition files for GR2.
- `./legged_lab/assets/unitree` — Unitree robot module namespace.
- `./legged_lab/assets/unitree/g1` — G1 robot asset package.
- `./legged_lab/assets/unitree/g1/configuration` — layered USD composition files for G1.
- `./legged_lab/assets/unitree/h1` — H1 robot asset package.
- `./legged_lab/envs` — task registration and robot-specific env configuration modules.
- `./legged_lab/envs/base` — base environment implementation and generic config primitives.
- `./legged_lab/envs/g1` — G1 task/reward specializations.
- `./legged_lab/envs/gr2` — GR2 task/reward specializations.
- `./legged_lab/envs/h1` — H1 task/reward specializations.
- `./legged_lab/mdp` — reward terms and MDP helpers.
- `./legged_lab/scripts` — executable training/inference scripts.
- `./legged_lab/terrains` — terrain generator presets and custom ray-caster integration.
- `./legged_lab/utils` — task registry, CLI args, keyboard device helper.
- `./legged_lab/utils/env_utils` — scene-construction helper modules.

### R4 folders (git internals)
- `./.git`
- `./.git/branches`
- `./.git/hooks`
- `./.git/info`
- `./.git/logs`
- `./.git/logs/refs`
- `./.git/logs/refs/heads`
- `./.git/logs/refs/remotes`
- `./.git/objects`
- `./.git/objects/info`
- `./.git/objects/pack`
- `./.git/refs`
- `./.git/refs/heads`
- `./.git/refs/remotes`
- `./.git/refs/tags`

---

## 2) File inventory (every file) with relevance and purpose

## R1 — Runtime Critical

### Environment registration and orchestration
- `legged_lab/envs/__init__.py`
  - Imports env/agent configs and registers tasks into the global `task_registry`.
  - Central map from task name (`h1_flat`, `g1_rough`, etc.) to `(env class, env cfg, agent cfg)`.

### Base environment runtime
- `legged_lab/envs/base/base_env.py`
  - Core `VecEnv` implementation.
  - Initializes simulation (`SimulationContext`), scene, sensors, command generator, reward manager, event manager.
  - Defines action processing pipeline (including optional delay buffer), step loop with decimation, reset logic, observation stacking, clipping/noise, terrain curriculum updates.
  - Exposes `get_observations()`, `step()`, `reset()`, and reset conditions (timeout/contact).

### Robot-specific environment/reward composition
- `legged_lab/envs/g1/g1_config.py`
  - Reward term definitions and weighting for G1.
  - `flat` and `rough` env/agent variants (rough enables height scanning + recurrent policy).
- `legged_lab/envs/h1/h1_config.py`
  - Same role as above for H1, with H1-specific thresholds/joint grouping/reward weights.
- `legged_lab/envs/gr2/gr2_config.py`
  - Same role as above for GR2 with GR2-specific body/joint naming and weights.

### MDP reward functions
- `legged_lab/mdp/rewards.py`
  - Implements reward/penalty kernels used by manager terms:
    - tracking (`track_lin_vel_xy_yaw_frame_exp`, `track_ang_vel_z_world_exp`)
    - stability/orientation (`flat_orientation_l2`, `body_orientation_l2`)
    - efficiency (`energy`, `joint_acc_l2`, `action_rate_l2`)
    - contact-quality (`undesired_contacts`, `feet_slide`, `feet_stumble`, `body_force`, `feet_air_time_positive_biped`)
    - safety/kinematics (`joint_deviation_l1`, `feet_too_near_humanoid`, `is_terminated`).
- `legged_lab/mdp/__init__.py`
  - Re-exports IsaacLab MDP utilities + local reward functions to a shared namespace.

### Scene and runtime utility wiring
- `legged_lab/utils/env_utils/scene.py`
  - Builds scene graph/config: terrain importer, articulation placement, contact sensor, optional height scanner, lights.
- `legged_lab/utils/task_registry.py`
  - Global registry implementation for task class + env/agent configs.
- `legged_lab/utils/cli_args.py`
  - CLI extensions for RSL-RL options and config override mapping.
- `legged_lab/utils/keyboard.py`
  - Interactive keyboard device helper in play mode (`R` reset behavior).
- `legged_lab/utils/__init__.py`
  - Re-export convenience for `task_registry`.
- `legged_lab/utils/env_utils/__init__.py`
  - Package marker for env utility namespace.

### Train and inference entrypoints
- `legged_lab/scripts/train.py`
  - Launches Isaac app, fetches cfgs via task registry, applies CLI/distributed overrides, creates `OnPolicyRunner`, writes params snapshots, executes training loop.
- `legged_lab/scripts/play.py`
  - Loads trained checkpoint, creates environment in eval-style settings, exports policy to JIT/ONNX, runs simulation loop with inference policy.

### Package root
- `legged_lab/__init__.py`
  - Defines package root/env directory constants.

## R2 — Configuration & Embodiment Critical

### Shared asset namespace
- `legged_lab/assets/__init__.py`
  - Exposes `ISAAC_ASSET_DIR` for resolving robot USD paths.

### Unitree robots
- `legged_lab/assets/unitree/__init__.py`
  - Re-exports Unitree articulation cfg symbols.
- `legged_lab/assets/unitree/unitree.py`
  - Defines `H1_CFG` and `G1_CFG` articulation configs:
    - USD spawn paths and rigid/articulation properties
    - initial pose/joint posture
    - grouped actuator parameters (limits, stiffness/damping, armature).
- `legged_lab/assets/unitree/g1/config.yaml`
  - URDF→USD converter settings used to generate the G1 USD.
- `legged_lab/assets/unitree/g1/g1.usd`
  - Primary G1 USD asset.
- `legged_lab/assets/unitree/g1/configuration/g1_base.usd`
  - Base geometry/structure layer for G1.
- `legged_lab/assets/unitree/g1/configuration/g1_physics.usd`
  - Physics tuning layer for G1.
- `legged_lab/assets/unitree/g1/configuration/g1_sensor.usd`
  - Sensor wiring layer for G1.
- `legged_lab/assets/unitree/h1/h1.usd`
  - Primary H1 USD asset.

### FFTAI GR2 robot
- `legged_lab/assets/fftai/__init__.py`
  - Re-exports FFTAI articulation cfg symbols.
- `legged_lab/assets/fftai/fftai.py`
  - Defines `GR2_CFG` articulation config (spawn, init, actuator groups/params).
- `legged_lab/assets/fftai/gr2/config.yaml`
  - URDF→USD converter settings used to generate GR2 USD.
- `legged_lab/assets/fftai/gr2/gr-2.usd`
  - Primary GR2 USD asset.
- `legged_lab/assets/fftai/gr2/configuration/gr-2_base.usd`
  - Base geometry/structure layer for GR2.
- `legged_lab/assets/fftai/gr2/configuration/gr-2_physics.usd`
  - Physics tuning layer for GR2.
- `legged_lab/assets/fftai/gr2/configuration/gr-2_sensor.usd`
  - Sensor wiring layer for GR2.

### Terrain and sensor extension configs
- `legged_lab/terrains/terrain_generator_cfg.py`
  - Defines terrain suites:
    - `GRAVEL_TERRAINS_CFG` (non-curriculum random roughness)
    - `ROUGH_TERRAINS_CFG` (curriculum with stairs, pits, boxes, waves, random rough).
- `legged_lab/terrains/ray_caster_cfg.py`
  - Binds custom `RayCaster` class into sensor cfg type.
- `legged_lab/terrains/ray_caster.py`
  - Extends reset logic to resample scanner drift values.
- `legged_lab/terrains/__init__.py`
  - Re-exports terrain cfgs.

### Base config classes (bridge R1↔R2)
- `legged_lab/envs/base/base_config.py`
  - Atomic `configclass` definitions for scene/robot/noise/commands/domain randomization/sim.
- `legged_lab/envs/base/base_env_config.py`
  - Composed default env cfg and PPO runner cfg with domain randomization event defaults.

## R3 — Packaging / Project Operations
- `README.md`
  - Overview, install, run command, references, citation.
- `setup.py`
  - Python package metadata and runtime dependencies (`IsaacLab`, `rsl-rl-lib`).
- `LICENSE.txt`
  - BSD-3-Clause license text.
- `.github/LICENSE_HEADER.txt`
  - Copyright header template for source files.
- `CODEBASE_DETAILED_SUMMARY.md`
  - This detailed inventory and relevance guide.

## R4 — Git metadata files
- Files under `.git/...`
  - Repository history, refs, object storage, logs, hooks, and config internals for source control operations.

---

## 3) System wiring summary (end-to-end runtime)

1. `train.py` / `play.py` parse CLI and launch Isaac app.
2. Importing `legged_lab.envs` triggers task registration in `task_registry`.
3. Selected task returns robot-specific `BaseEnvCfg` + `BaseAgentCfg` variants.
4. `BaseEnv` creates simulator + scene (`SceneCfg`), command generator, event manager, reward manager.
5. Reward manager executes robot-configured terms backed by `mdp/rewards.py` functions.
6. Terrain/scanner behavior comes from `terrain_generator_cfg.py` and custom `RayCaster` drift logic.
7. RSL-RL `OnPolicyRunner` trains or runs inference policy.

---

## 4) Priority map for maintainers

- **Highest impact when debugging behavior/performance**:
  - `legged_lab/envs/base/base_env.py`
  - `legged_lab/mdp/rewards.py`
  - `legged_lab/envs/*/*_config.py`
  - `legged_lab/utils/env_utils/scene.py`
  - `legged_lab/scripts/train.py`
- **Highest impact when changing embodiment/sim fidelity**:
  - `legged_lab/assets/unitree/unitree.py`
  - `legged_lab/assets/fftai/fftai.py`
  - robot USD files under `legged_lab/assets/**`
  - `legged_lab/terrains/terrain_generator_cfg.py`
- **Operational/setup impact**:
  - `README.md`, `setup.py`, task registration in `legged_lab/envs/__init__.py`.
