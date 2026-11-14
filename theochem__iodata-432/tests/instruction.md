# Issue

**Title:** Compare MWFN electron counts with a tolerance

## Description

Addresses the second point of #428 (the exact float comparisons). The first point, about `Nindbasis`, needs an MWFN example with pruned basis functions, so it isn't part of this PR.

`load_one` in `mwfn.py` checks the electron counts with `!=`:

```python
if mo.nelec != inp["Naelec"] + inp["Nbelec"]:
if mo.occsa.sum() != inp["Naelec"]:
if mo.occsb.sum() != inp["Nbelec"]:
```

MWFN files write each occupation with 6 decimals (`Occ=  0.333333`), so fractional occupations don't add up exactly. With three alpha orbitals at 1/3 in `ch3_hf_sto3g_fchk_multiwfn3.7.mwfn`, the load fails:

```
LoadError: Number of electrons in MolecularOrbitals (8.999999) is not equal to the sum of 'Naelec' and 'Nbelec' (5.0 + 4.0).
```

### Changes
- `iodata/formats/mwfn.py`: the three checks now allow a difference of up to `0.5e-6` per orbital, which is the rounding of the 6-decimal occupations.
- `iodata/test/test_mwfn.py`: `test_load_mwfn_fractional_occupations` writes a copy of the CH3 file with those occupations to `tmp_path` and loads it. It fails on `main` with the error above and passes with the fix.

### Checks
- `pytest iodata/test/test_mwfn.py`: 5 passed
- `ruff check` / `ruff format --check`: clean

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `785cf84e7814`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.