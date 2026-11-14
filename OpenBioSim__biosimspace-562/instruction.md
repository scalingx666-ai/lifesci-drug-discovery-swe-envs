# Issue

**Title:** Fix issue #561

## Description

.

`_score_rdkit_mappings` paired every match of molecule0 with every match of molecule1. It checked each resulting mapping for duplicates by scanning a list of dicts, so the cost grew quadratically with `max_scoring_matches`. Most matches are permutations of equivalent hydrogens, so drug-sized ligands could take minutes per pair at the default of 1000.

Changes, applied to both the main module and the Sandpit copy:

- Duplicate mappings are skipped using a set of NumPy-built keys, so only new mappings are converted, aligned and scored. Pairs are visited in the same order as before.
- Each molecule's coordinates are read once. `rmsd_align` now uses a NumPy Kabsch superposition of the mapped atoms rather than a Sire `move().align()` and per-atom property lookups. `rmsd_flex_align` still uses `flexAlign`.
- Each mapping is now aligned from the original coordinates. Previously each alignment started from the previous mapping's result, and from that start Sire's alignment sometimes ended up in a worse orientation without raising an error. For example, every other benzene → toluene mapping scored 1.64 Å instead of 0.12 Å. The best mapping was unaffected in every case tested, but the rest of the ranking depended on the order mappings were visited.

| Pair (1000 matches) | Before | After |
|---|---|---|
| Factor Xa 4d → 4c | > 150 s | 0.39 s |
| TYK2 ejm_44 → ejm_43 | 4.1 s | 0.03 s |
| BACE lig_41 → lig_67 | 3.3 s | 0.03 s |

Checked against `devel` on the test-suite and OpenFE benchmark ligands, in both directions, with and without a prematch:

- `rmsd_align` and `rmsd_flex_align` give identical ranked mappings.
- Plain `rmsd` gives the same mappings and scores, but mappings with exactly equal scores can come out in a different order, because the RMSD is summed in a different order. The best mapping is unchanged.

The new `test_score_rdkit_mappings` compares the function with a brute-force reference: every unique mapping from every pair of matches, with prematch filtering, and each aligned score checked against Horn's quaternion method. It fails 6 of its 20 cases with the previous implementation.

* I confirm that I have merged the latest version of `devel` into this branch before issuing this pull request (e.g. by running `git pull origin devel`): [y]
* I confirm that I have permission to release this code under the GPL3 license: [y]

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5e8c0417dfe6`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.