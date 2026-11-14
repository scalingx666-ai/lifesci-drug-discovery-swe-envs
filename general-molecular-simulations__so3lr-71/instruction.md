# Issue

**Title:** flash: periodic images for small boxes (under twice a cutoff)

## Description

## Summary

SO3LR-Flash now runs periodic orthorhombic boxes down to a few Å. Until now, flash and the neighbour-list engine both refused a box under twice the long-range cutoff (24 Å at the default 12 Å). The only way to run one was the ASE `So3lrCalculator` or a supercell.

- **Long range (box under 2 × `lr_cutoff`):** the long-range kernel walks the box's periodic images directly. It does not replicate atoms.
- **Short range (box under 2 × 4.5 Å):** each neighbour-list entry packs its image shift into the existing index (`j·(2KS+1)³ + shift code`). The Pallas kernels and the CUDA kernels decode it the same way, including an atom's pairs with its own images.
- **Boxes at or above twice each cutoff** run exactly the code they ran before.
- **Refusals:** `check.box_problem` refuses only boxes whose image walk would exceed 16 cells, for example `--lr-cutoff 1000` in a 12.4 Å box.

Both the Pallas-Triton and the CUDA kernels support small boxes. The memory layout and the per-step cost of large boxes are unchanged.

## Speed (A100-40GB, `so3lr-tauri-m-2606`, NVT, real-space long range at 12 Å)

Time per MD step for the same physical system:

| System | flash, small box (this PR) | flash, smallest accepted supercell | neighbour-list engine, supercell | ASE `So3lrCalculator`, small box |
|---|---|---|---|---|
| 64 waters, 12.4 Å (192 atoms) | **1.47 ms** (CUDA), 1.99 ms (Pallas) | 2.87 ms (2×2×2, 1,536 atoms) | 15.0 ms | 622 ms |
| Diamond 3.57 × 3.57 × 7.13 Å (16 atoms) | **1.42 ms** (CUDA), 2.29 ms (Pallas) | 5.81 ms (7×7×4, 3,136 atoms) | 52.4 ms | 107 ms |

Speedups of flash with CUDA on the small box:

| | over the supercell on flash | over the supercell on the neighbour-list engine | over the ASE calculator |
|---|---|---|---|
| Water | 2.0× | 10× | 420× |
| Diamond | 4.1× | 37× | 75× |

How these were measured:
- **MD runs:** `so3lr nvt`, 6 × 200 steps; the first cycle, which includes compilation, is excluded.
- **ASE calculator:** 100 velocity-Verlet steps after 5 warm-up steps.
- **Neighbour-list engine:** it refuses the small boxes themselves ("cell heights ... must be at least twice the neighbour-list cutoff"), so it is timed on the supercell.

Large boxes are unchanged. A 12,288-atom water box needs no images, and it runs at the same speed on main and on this branch:

| Kernels | main | this branch |
|---|---|---|
| CUDA | 11.6 ms/step | 11.6 ms/step |
| Pallas | 30.4 ms/step | 30.3 ms/step |

## Validation

All on an A100:

- **Small box against its supercell, in float64:**
  - A periodic box and its supercell are the same system, so the energy per atom, forces and stress must agree. They agree to 1e-10 for energy and 1e-9 for forces and stress.
  - The supercells (4×4×4 water, 9×9×5 diamond) are large enough to need no images, so the reference runs none of the new code.
  - 64 waters with PME exercises the long-range image walk.
  - A rattled diamond cell narrower than the 4.5 Å short-range cutoff also exercises short-range images, including self-image pairs.
- **CUDA float32 against float64** on the same two boxes: 1e-6 for energy, 1e-4 for forces and stress.
- **MD:** `so3lr npt` runs in the 12.4 Å water box.
- **Re-run after review:** all of the above were run again on the final commit, after the simplify and code-review fixes.
- **Existing tests:**
  - `test_cuda_kernels_follow_float64` and `test_flash_matches_neighbour_lists` pass unchanged.
  - The CPU suite passes except `test_flash_runs_chargeseparation_charge_sites_as_perform_md`, which fails the same way on main.

## Tests

The new tests are kept minimal:

- **GPU:** `test_small_box_follows_its_supercell` and `test_cuda_small_box`, each run on the water and diamond boxes, plus `test_md_in_a_small_box_on_flash`.
- **CPU:** `test_cell_grid_takes_every_box_from_the_smallest` covers the layout for boxes from 1 Å up to the old minimum.
- **Changed tests:**
  - `test_md_logs_its_engine` passes `--lr-cutoff 1000` to keep its fallback case, because a 12.4 Å box now runs on flash.
  - `test_restart_box_is_checked` uses a 0.5 Å restart box.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `48cac8eb4f48`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.