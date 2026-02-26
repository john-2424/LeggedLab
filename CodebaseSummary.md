# LeggedLab Codebase Detailed Summary

## Relevance Tiers
- **Tier A (Core Runtime):** Files directly used to define environments, rewards, terrain generation, task registration, and train/play execution.
- **Tier B (Configuration & Assets):** Robot asset definitions and conversion metadata used by Tier A at runtime.
- **Tier C (Project/Meta):** Packaging, licensing, and auxiliary metadata.

## Folder-by-folder map

### Tier A folders
- `legged_lab/`: package root.
- `legged_lab/envs/`: task registration and environment definitions.
- `legged_lab/envs/base/`: generic vectorized RL environment and base config classes.
- `legged_lab/envs/g1/`, `legged_lab/envs/h1/`, `legged_lab/envs/gr2/`: robot-specific reward and env/agent configs.
- `legged_lab/mdp/`: reward terms and MDP-level logic imported by configs.
- `legged_lab/terrains/`: terrain presets and custom ray-caster behavior for height scanning.
- `legged_lab/utils/`: CLI args, task registry, keyboard helper.
- `legged_lab/utils/env_utils/`: scene assembly helper used by base env.
- `legged_lab/scripts/`: executable train/play entry points.

### Tier B folders
- `legged_lab/assets/`: robot asset namespace and asset root constants.
- `legged_lab/assets/unitree/`: Unitree robot configs and USD files (H1, G1).
- `legged_lab/assets/fftai/`: FFTAI GR2 robot configs and USD files.
- `legged_lab/assets/*/*/configuration/`: USD composition layers (`*_base`, `*_physics`, `*_sensor`).

### Tier C folders
- `.github/`: shared license header template.
- `.git/`: git metadata.

## File-by-file inventory

### Tier A (Core Runtime)
- `legged_lab/__init__.py`: Defines root path constants used by the package.
- `legged_lab/envs/__init__.py`: Imports all task configs and registers six tasks in the global `task_registry` (`h1/g1/gr2` × `flat/rough`).
- `legged_lab/envs/base/base_env.py`: Core `VecEnv` implementation (simulation loop, command generation, reward manager, reset logic, observation assembly, optional height scan, action delay, terrain curriculum).
- `legged_lab/envs/base/base_config.py`: Atomic configclasses for scene, robot, normalization, commands, noise, domain randomization, and sim step settings.
- `legged_lab/envs/base/base_env_config.py`: Composes base env defaults and default PPO runner (`RslRlOnPolicyRunnerCfg`) plus domain randomization event terms.
- `legged_lab/envs/g1/g1_config.py`: G1 reward terms and `flat/rough` env+agent specializations (rough uses height scan + recurrent policy).
- `legged_lab/envs/h1/h1_config.py`: H1 reward terms and `flat/rough` specializations similar to G1.
- `legged_lab/envs/gr2/gr2_config.py`: GR2 reward terms and `flat/rough` specializations.
- `legged_lab/mdp/__init__.py`: Re-exports IsaacLab MDP functions + local reward functions.
- `legged_lab/mdp/rewards.py`: Custom reward function implementations (tracking, penalties, contacts, stumble/slide/force, joint deviations, foot spacing, termination handling).
- `legged_lab/terrains/__init__.py`: Re-exports terrain presets.
- `legged_lab/terrains/terrain_generator_cfg.py`: Defines two terrain suites: `GRAVEL_TERRAINS_CFG` and curriculum-enabled `ROUGH_TERRAINS_CFG`.
- `legged_lab/terrains/ray_caster_cfg.py`: Extends base ray-caster config to bind custom `RayCaster` class.
- `legged_lab/terrains/ray_caster.py`: Overrides sensor reset to resample XYZ drift per env.
- `legged_lab/utils/__init__.py`: Re-exports global task registry.
- `legged_lab/utils/task_registry.py`: Global task registry class storing task class + env cfg + train cfg.
- `legged_lab/utils/cli_args.py`: CLI argument injection and overrides from CLI into agent config.
- `legged_lab/utils/keyboard.py`: Keyboard helper for interactive play mode (`R` key forces reset by setting long episode lengths).
- `legged_lab/utils/env_utils/__init__.py`: Package marker.
- `legged_lab/utils/env_utils/scene.py`: Builds `InteractiveSceneCfg` with terrain, robot, contact sensor, lighting, and optional height scanner.
- `legged_lab/scripts/train.py`: Training entrypoint; launches app, resolves task cfgs, handles distributed settings, resumes checkpoints, writes config snapshots, runs PPO learning.
- `legged_lab/scripts/play.py`: Inference/play entrypoint; loads checkpoint, exports JIT/ONNX policy, runs rollout loop, optional keyboard control.

### Tier B (Configuration & Assets)
- `legged_lab/assets/__init__.py`: Defines `ISAAC_ASSET_DIR` root.
- `legged_lab/assets/unitree/__init__.py`: Re-exports Unitree articulation configs.
- `legged_lab/assets/unitree/unitree.py`: Defines `H1_CFG` and `G1_CFG` articulation setup (USD path, init pose, actuator groups and gains/limits).
- `legged_lab/assets/unitree/g1/config.yaml`: URDF→USD conversion recipe metadata for G1.
- `legged_lab/assets/unitree/g1/g1.usd`: Primary G1 robot USD asset.
- `legged_lab/assets/unitree/g1/configuration/g1_base.usd`: G1 base composition layer.
- `legged_lab/assets/unitree/g1/configuration/g1_physics.usd`: G1 physics layer.
- `legged_lab/assets/unitree/g1/configuration/g1_sensor.usd`: G1 sensor layer.
- `legged_lab/assets/unitree/h1/h1.usd`: Primary H1 robot USD asset.
- `legged_lab/assets/fftai/__init__.py`: Re-exports FFTAI articulation configs.
- `legged_lab/assets/fftai/fftai.py`: Defines `GR2_CFG` articulation setup (USD path, init pose, grouped actuator tuning).
- `legged_lab/assets/fftai/gr2/config.yaml`: URDF→USD conversion recipe metadata for GR2.
- `legged_lab/assets/fftai/gr2/gr-2.usd`: Primary GR2 robot USD asset.
- `legged_lab/assets/fftai/gr2/configuration/gr-2_base.usd`: GR2 base composition layer.
- `legged_lab/assets/fftai/gr2/configuration/gr-2_physics.usd`: GR2 physics layer.
- `legged_lab/assets/fftai/gr2/configuration/gr-2_sensor.usd`: GR2 sensor layer.

### Tier C (Project/Meta)
- `README.md`: High-level project purpose, dependencies (IsaacSim/IsaacLab/RSL-RL), install and run instructions, references.
- `setup.py`: Editable-install package metadata and runtime deps.
- `LICENSE.txt`: BSD-3-Clause license text.
- `.github/LICENSE_HEADER.txt`: Shared copyright header template.

## Runtime architecture (how pieces connect)
1. `legged_lab/scripts/train.py` / `play.py` parse args, launch Isaac app, import `legged_lab.envs` so task registration runs.
2. `legged_lab/envs/__init__.py` registers each task to `task_registry`.
3. `task_registry` returns `(env_cfg, agent_cfg, env_class)` for selected task.
4. `BaseEnv` constructs simulation context + scene via `SceneCfg` (`utils/env_utils/scene.py`), command generator, event manager, reward manager.
5. Reward manager executes terms declared in robot-specific config classes (`g1/h1/gr2_config.py`) and implemented in `mdp/rewards.py`.
6. Terrain behavior comes from `terrains/terrain_generator_cfg.py`; rough tasks also use custom height scan through `RayCaster`.
7. RSL-RL `OnPolicyRunner` runs PPO training/inference.

## Relevance categorization by operational importance
- **Most critical to understanding behavior:**
  - `base_env.py`, `base_env_config.py`, `g1/h1/gr2_config.py`, `mdp/rewards.py`, `scene.py`, `terrain_generator_cfg.py`, `train.py`, `play.py`.
- **Critical for robot embodiment fidelity:**
  - `assets/unitree/unitree.py`, `assets/fftai/fftai.py`, plus corresponding USD files.
- **Auxiliary but important for operability:**
  - `cli_args.py`, `task_registry.py`, `keyboard.py`, `setup.py`, `README.md`.
- **Low operational relevance (meta):**
  - license/header files and git metadata.

## Exhaustive directory listing (all folders)
- `.`
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
- `./.github`
- `./legged_lab`
- `./legged_lab/assets`
- `./legged_lab/assets/fftai`
- `./legged_lab/assets/fftai/gr2`
- `./legged_lab/assets/fftai/gr2/configuration`
- `./legged_lab/assets/unitree`
- `./legged_lab/assets/unitree/g1`
- `./legged_lab/assets/unitree/g1/configuration`
- `./legged_lab/assets/unitree/h1`
- `./legged_lab/envs`
- `./legged_lab/envs/base`
- `./legged_lab/envs/g1`
- `./legged_lab/envs/gr2`
- `./legged_lab/envs/h1`
- `./legged_lab/mdp`
- `./legged_lab/scripts`
- `./legged_lab/terrains`
- `./legged_lab/utils`
- `./legged_lab/utils/env_utils`

## Exhaustive file listing (all tracked content-level files)
- `CODEBASE_DETAILED_SUMMARY.md`
- `LICENSE.txt`
- `README.md`
- `legged_lab/__init__.py`
- `legged_lab/assets/__init__.py`
- `legged_lab/assets/fftai/__init__.py`
- `legged_lab/assets/fftai/fftai.py`
- `legged_lab/assets/fftai/gr2/config.yaml`
- `legged_lab/assets/fftai/gr2/configuration/gr-2_base.usd`
- `legged_lab/assets/fftai/gr2/configuration/gr-2_physics.usd`
- `legged_lab/assets/fftai/gr2/configuration/gr-2_sensor.usd`
- `legged_lab/assets/fftai/gr2/gr-2.usd`
- `legged_lab/assets/unitree/__init__.py`
- `legged_lab/assets/unitree/g1/config.yaml`
- `legged_lab/assets/unitree/g1/configuration/g1_base.usd`
- `legged_lab/assets/unitree/g1/configuration/g1_physics.usd`
- `legged_lab/assets/unitree/g1/configuration/g1_sensor.usd`
- `legged_lab/assets/unitree/g1/g1.usd`
- `legged_lab/assets/unitree/h1/h1.usd`
- `legged_lab/assets/unitree/unitree.py`
- `legged_lab/envs/__init__.py`
- `legged_lab/envs/base/base_config.py`
- `legged_lab/envs/base/base_env.py`
- `legged_lab/envs/base/base_env_config.py`
- `legged_lab/envs/g1/g1_config.py`
- `legged_lab/envs/gr2/gr2_config.py`
- `legged_lab/envs/h1/h1_config.py`
- `legged_lab/mdp/__init__.py`
- `legged_lab/mdp/rewards.py`
- `legged_lab/scripts/play.py`
- `legged_lab/scripts/train.py`
- `legged_lab/terrains/__init__.py`
- `legged_lab/terrains/ray_caster.py`
- `legged_lab/terrains/ray_caster_cfg.py`
- `legged_lab/terrains/terrain_generator_cfg.py`
- `legged_lab/utils/__init__.py`
- `legged_lab/utils/cli_args.py`
- `legged_lab/utils/env_utils/__init__.py`
- `legged_lab/utils/env_utils/scene.py`
- `legged_lab/utils/keyboard.py`
- `legged_lab/utils/task_registry.py`
- `setup.py`
