# Issue

**Title:** Fix the energy, num2sym and nuclear names of the WFX dump

## Description

.

All four points of the issue are fixed in the WFX module:

- The loader documents `atcorenums` in its `@document_load_one` call.
- An energy of 0.0 was written as NaN because of `data.energy or np.nan`. Now only `None` gives NaN.
- The dump function does not update the `num2sym` dictionary of the periodic module anymore, it makes a local copy with `Bq` for the ghost atom. (Before, the dictionary of the whole package got a new key as soon as a WFX file was dumped.)
- The nuclear names are made from `atnums` and not from `atcorenums`. With ECPs the core charge is not the atomic number, so the names were of the wrong element, and a fractional core charge made the writer fail with a `KeyError`.

The tests are added to `test_wfx.py`: the zero energy, `num2sym` not being changed, the names with a core charge of 6.0 and 6.5 for an oxygen, and a ghost atom (`Bq`). Without the change the first four fail, with it the tests of the WFX, WFN, FCHK and Molden modules pass (241, without the slow ones). `ruff check` and `ruff format` are clean.

The same energy bug in the WFN writer is in #429.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `785cf84e7814`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.