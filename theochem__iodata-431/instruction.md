# Issue

**Title:** Add helpers to parse floating point numbers written by Fortran code

## Description

.

This adds `parse_fortran_float` (one word) and `parse_fortran_floats` (many words, as an array) to `iodata.utils`, based on the code in the issue. They accept the notations that Fortran code writes: `1.5D+03`, `1.5d-03`, and no letter at all when the exponent has three digits, e.g. `1.234567-123`. A word that is already valid for Python takes the fast path.

One difference with the code in the issue: the word is stripped first. Without it a padded negative number, such as `" -1.5"`, got an `E` inserted before its sign, because the sign is not at index 0.

The helper is now used where the parsers did the `D` to `E` replacement themselves: the FCHK `_convert_word`, the Molden basis set, the WFN sections (the exponents) and the two places in the Gaussian log parser. The FCHK regular expression that splits numbers without whitespace is unchanged. I left the MWFN loader alone as #419 is open and changes that part.

Tests for both functions are added to `test_utils.py`, with the notations above, padded and signed numbers, and invalid words. The rest of the test suite passes (600, without the slow tests), and `ruff check` and `ruff format` are clean.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `1f27cada94f1`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.