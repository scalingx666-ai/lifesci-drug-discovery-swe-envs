# Issue

**Title:** Add GPAW competing-phase output parsing for chemical potentials

## Description

This PR adds GPAW competing-phase output parsing for chemical-potential analysis directly from restart files:

phase_analyzer = CompetingPhasesAnalyzer(
    "MgO",
    entries="CompetingPhases_GPAW",
    calculator="gpaw",
)

The MgO example includes the host and elemental references in this directory and uses the analyzer in its analysis script.

The parser reads structures, stored energies and metadata without loading defect electrostatic potentials or orbital data. Charged calculations and restarts without stored energy are rejected, preventing unintended SCF calculations. The stored-energy guard also applies to the general GPAW output reader.

File discovery follows GPAW filename priorities and excludes backups. Entries record actual k-point meshes, including density-based sampling. Compatibility checks warn about inconsistent energy-affecting settings while allowing different k-point and real-space meshes.

Validation with GPAW 26.7:

- 13 phase/restart tests passed using real MgO, Mg and triplet O₂ calculations and shipped restart files, without mocked calculators.
- Tests cover directory discovery, Mg-rich/O-rich limits, serialization, file priority, charge and missing-energy rejection, k-point meshes, and XC/PAW/mode mismatches.
- One existing GPAW output-reader test and two existing VASP discovery tests passed.
- Applicable pre-commit hooks and example lint/format checks passed.

Ionic convergence cannot be established from a restart alone. Band gaps and general GPAW competing-phase input generation remain outside this PR. The full test suite was not run.

Related to #156; this does not close the full tracking issue.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `7752f9f33d54`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.