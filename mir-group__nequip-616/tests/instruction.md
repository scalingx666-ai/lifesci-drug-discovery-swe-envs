# Issue

**Title:** Document XYZFileWriter chemical_symbols argument

## Description

The XYZFileWriter, TestTimeXYZFileWriter, and ValTimeXYZFileWriter docstrings tell users to pass `chemical_species`, but their constructor accepts `chemical_symbols`. This changes only those docstrings to use the actual argument name and explains when the mapping is needed: output data without atomic numbers. The mapping order remains aligned with the model's `type_names`.

Validation: `python3 -m compileall -q nequip/train/callbacks/write_xyz.py`, a docstring consistency check for all three classes, and `git diff --check` passed. The full NequIP test suite was not run because it requires the heavier ML dependencies.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `d818c7a806a6`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.