# RoboTwin-Phys

**[Project Page](https://yefeng00.github.io/RoboTwinPhysTest/) · [Paper (arXiv:2609.26292)](https://arxiv.org/abs/2609.26292) · [Leaderboard](https://yefeng00.github.io/RoboTwinPhysTest/#leaderboard)**

RoboTwin-Phys is a physics-diverse benchmark for bimanual robot manipulation, built on the
50-task suite of [RoboTwin 2.0](https://robotwin-platform.github.io/). While existing benchmarks
randomize what the world *looks like* — appearance, layout, lighting, viewpoints — they keep the
underlying physical parameters at fixed nominal values. RoboTwin-Phys introduces
**physical-condition diversity** as an explicit evaluation dimension:
**13 physical attributes are continuously sampled at the episode level** within physically
plausible, task-aware ranges, and every condition is verified executable by an expert planner.

Evaluations of representative WAMs and VLAs reveal a **substantial robustness gap**: models that
remain effective under RoboTwin 2.0's official visual randomization degrade markedly when physical
conditions vary.

## Highlights

- **13 physical attributes, episode-level continuous sampling** — mass, center of mass, geometry
  scale, table tilt, table height, external force, camera distance/angle, friction, restitution,
  object / arm / gripper joint damping. Episode seed makes every sampled condition reproducible.
- **Task-aware physical validity** — 9 physics-sensitive tasks are automatically routed to
  dedicated, empirically calibrated configurations; all other tasks share the global configuration.
- **Expert-verified feasibility** — a sampled condition enters the benchmark only if the expert
  planner can complete the task under it, so failures measure model robustness rather than the
  absence of a solution.
- **Physics-annotated training set** — 50 tasks × 100 expert demonstrations = 5,000 episodes, each
  annotated with the 13-dimensional ground-truth physical parameters in effect during its rollout.

## Benchmark results

Aggregate success rates (%) on 50 tasks. *Clean* and *Official Random* are reported results from
public benchmark evaluations; *Physical Random* (our protocol, 100 rollout episodes per task) is
evaluated by us.

| Method | Clean | Official Random | Physical Random |
|---|---|---|---|
| Fast-WAM | 91.88 | 91.78 | **44.24** |
| Motus | 88.66 | 87.02 | **39.60** |
| FACT | 88.40 | 86.60 | **39.14** |
| π0.5 | 82.74 | 76.76 | **31.60** |
| GalaxeaVLA (G0.5) | 93.70 | 92.80 | **37.83** |

Robustness to visual and layout randomization does not transfer to physical-condition diversity —
across both WAM and VLA families. See the paper for the full 50-task breakdown and task-level
analysis.

## Dataset

The released training set (RoboTwin-Phys Training Set v1) contains 5,000 expert demonstrations in
the official RoboTwin format, plus per-episode physical ground truth:

```
<task>/
  data/episode_XXXXXXX.hdf5      # actions, states, camera streams (official RoboTwin format)
  video/episode_XXXXXXX.mp4      # head-camera rendering, 320x240 @ 30 fps
  phys_meta/episode_XXXXXXX.json # 13-dim ground-truth physical attributes + seed + camera pose
  seeds.txt                      # 100 seeds, episode_i <- seeds[i]
```

Download link: TBD. The data remain fully compatible with existing WAM/VLA pipelines; `phys_meta`
joins onto episodes by index for physical-attribute estimation, condition-aware modeling, and
physics-conditioned policy training.

## Evaluation protocol

Three environment protocols, one shared setup: **Clean** (no randomization), **Official Random**
(RoboTwin 2.0 visual/layout randomization, nominal physics), **Physical Random** (official
randomization retained + the 13 physical attributes sampled). Each task is evaluated with 100
rollout episodes; episodes the expert planner cannot complete are excluded from the pool.

## Code

The benchmark code, task environments, and configuration files will be released in this repository.
Stay tuned.

## Citation

```bibtex
@article{zhang2026robotwinphys,
  title   = {RoboTwin-Phys: Do WAMs and VLAs Understand the Physical World?},
  author  = {Zhang, Jiaqi and Ye, Feng and Yang, Mingjia and Chen, Zhihong
             and Xiang, Mingkang and Yao, Xinglin and Li, Yanbin
             and Ma, Siwei and Jia, Chuanmin},
  journal = {arXiv preprint arXiv:2609.26292},
  year    = {2026}
}
```

## Acknowledgements

Built on [RoboTwin 2.0](https://robotwin-platform.github.io/) (SAPIEN). Affiliations: Peking
University; Advanced Institute of Information Technology, Peking University; Memo; DEMX.
