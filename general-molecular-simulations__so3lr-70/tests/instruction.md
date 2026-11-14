# Issue

**Title:** So3lrEngine: dispersion in k-space

## Description

## So3lrEngine: dispersion in k-space

`So3lrEngine` took k-space only for the electrostatics; `so3lr md` and `so3lr eval` also take it for the dispersion. This adds `kspace_dispersion` (`pme` or `ewald`) with `kspace_dispersion_smearing` and `kspace_dispersion_spacing` (so3lr's defaults when omitted), so programs built on the engine (OpenMM through openmm-ml) can run with the same long range as `so3lr md`.

- **Flash:** the settings go to flash as `so3lr md` passes them (flash takes `pme`; `ewald` falls back to the neighbour lists under `engine="auto"`).
- **Neighbour lists:** `So3lrPotential(dispersion_kspace_method=...)` with the dispersion grid (`k_grid_disp`, `k_smearing_disp`) set up per box like the electrostatics one.
- **Exact images** (cells thinner than twice the cutoff, triclinic cells): refused with an error, as the ASE calculator has no dispersion in k-space.

### Tests
- `test_dispersion_kspace_matches_md`: the engine against the potential `so3lr md` integrates (`test_pme._cli_md_energy_and_forces`), float64, 64 waters, PME electrostatics + PME dispersion.
- `test_dispersion_kspace_refuses_the_exact_images`, and two more bad-argument cases.
- `test_flash_matches_neighbour_lists_with_both_in_kspace` (GPU): flash against the neighbour lists with both in k-space.

`tests/test_engine.py` on an A100 (MeluXina): 24 passed, 1 skipped (`engine="flash"` refusal, which needs a machine without flash).

### Used in
1,509 waters with SO3LR-Tauri-M in OpenMM (openmm-ml) and in `so3lr md`, PME electrostatics and dispersion, 10 Å real-space cutoff: both on flash, NPT densities within each other's noise (~0.99 g/cm³ after 30 ps).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5bcc2319b765`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.