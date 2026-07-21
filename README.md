# RoboBench — five robot brains, one body

Benchmarking five popular open-source **VLA (vision-language-action) robot policies** on a single
standardised embodiment — the Unitree G1 humanoid — under an identical adaptation budget, scored in
simulation.

**[→ Read the report](https://guraseeswasnotavailable.github.io/RoboBench/)**

---

## What was measured

Each policy controlled the same robot on the same task — pick up a block, place it on a target —
with the same training data (216 demonstrations), the same seeds, and the same 600-step episode
budget. 150 attempts per policy, **750 scored rollouts** in total.

| Policy | Params | Score | 95% range | Success |
|---|---|---|---|---|
| **GR00T N1.7** ⚑ | 3B | **36.2** | ±4.3 | 4.0% |
| ACT *(control, from scratch)* | 52M | 25.6 | ±3.2 | 1.3% |
| SmolVLA | 450M | 23.9 | ±2.9 | 0% |
| WALL-OSS | ~4B | 23.7 | ±3.0 | 0.7% |
| π0.5 | 4.1B | 22.9 | ±2.8 | 0% |

⚑ humanoid-native — pre-trained on robots of this shape, an inherent home advantage.

Score = how much of the block's starting distance to the target was closed, 0–100, where 100 means
placed. Continuous rather than pass/fail, because outright success is too rare to rank on.

## Findings

1. **Grasping is solved; placement is not.** All 750 attempts picked the block up. Nine placed it.
2. **Only one gap is statistically real.** GR00T beats the from-scratch control by 10.6 ± 5.4 points.
   The remaining four sit within 2.8 points of each other with ±3 margins — genuinely equivalent,
   not merely unranked.
3. **Generic pre-training bought nothing here; body-matched pre-training did.** Three of four
   pre-trained models scored at or below a 52M baseline with no pre-training at all.
4. **Training loss anti-predicted capability.** π0.5 reproduced the demonstrations ~35× more closely
   than GR00T, then finished last.
5. **Two different failure modes.** Three policies carry the block near the target and park there
   until the clock expires. GR00T makes the closest approach in the field and then carries the block
   ~13 cm back out — a stopping problem, not a perception one.

## Caveats

Single task, single embodiment, simulation only. GR00T's humanoid pre-training is a home advantage
on this robot, so its win means "best for humanoids here", not "best overall". No claim is made
about real-robot transfer, and nothing here tests generalisation to new objects or instructions —
which is what pre-training is actually for.

## Method notes

- **Success zone** is 3.7 × 4.9 × 0.5 cm — anisotropic, 7.4× tighter in height than width — so
  distance is measured per-axis in units of the zone's own half-extents rather than as a plain
  Euclidean distance.
- **Log scaling.** Every policy fails in the endgame, so the score gives equal credit per halving of
  remaining error, putting the resolution where the models actually differ.
- **Error bars everywhere.** With 150 episodes the score resolves gaps of about 4 points. Gaps
  smaller than that are reported as ties rather than ranked.

Simulation: NVIDIA Isaac Sim 5.1, task `Isaac-PickPlace-RedBlock-G129-Dex1-Joint`, Unitree G1
(29 DoF, Dex1 grippers), on a single NVIDIA L40S.

---

This repository hosts the published report only. The evaluation harness and raw per-episode data
are kept separately.
