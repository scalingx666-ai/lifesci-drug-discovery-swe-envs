# Issue

**Title:** Load GPAW PAW projections for defect eigenvalue analysis

## Description

GPAW restart files can now supply the orbital weights needed by `DefectEntry.get_eigenvalue_analysis()`. The parser reads stored collinear PAW projections and converts them to orbital weights using GPAW’s bound-projector convention.

Projections are loaded on demand or when `parse_projected_eigen=True`. If a restart lacks projections, the error explains how to save them.

All 50 GPAW tests passed locally, including comparisons with GPAW’s native PDOS for MgO and spin-polarised O₂, end-to-end analysis of the Mg vacancy, and handling of restarts without projections. Pre-commit checks also passed.

Related to #156.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `cfe128b0a9f8`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.