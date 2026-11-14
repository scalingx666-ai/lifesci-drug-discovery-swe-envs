# Issue

**Title:** Refresh PyLOOS frame selections and cached alignments

## Description

Changing PyLOOS `skip()` or `stride()` after construction can leave the old selection in use. Changing an aligned virtual selection can also reuse transforms for different source frames. This PR refreshes selections and invalidates the affected alignment cache.

Direct child changes are included: after `child.skip(1)`, an already aligned parent could apply frame 0's transform to raw frame 1. Independent coordinates reproduced a 19.02 Å error. Parent frame maps now track each child's selection generation, so updates remain detectable after `len(child)` or `child.index()` clears its stale flag, or when two child-length changes cancel each other.

The final behavior:

- Refresh mappings before indexing, slicing, iteration, location/boundary queries and iterator-state queries. Accepted selection changes restart iteration on the next frame-list access.
- Reject inapplicable skip/stride setters with `ValueError` before mutation when a custom iterator controls selection. Bulk setters preflight every child. Retain one-shot virtual iterator indices across rebuilds.
- Fix iteration-boundary `frame()` reporting and out-of-range negative frame indices. Reject noncontiguous custom ordering in `frameBoundaries()` instead of returning invalid slices.
- Refit invalidated aligned selections before returning coordinates. Empty selections skip native alignment; invalidated `rmsd()`/`iters()` report the existing uncomputed sentinel without triggering an expensive refit.
- Document resets, custom-iterator errors, cached-metric semantics and full iterative refitting cost. Iterative transforms are not reused.

Validation used a fresh full native build and separately installed Python package. All 31 local regression methods pass, along with 640,000 randomized state operations and 75 independent numerical/analysis cases. Public [Barnaba UUCG](https://github.com/srnas/barnaba/tree/48ac4a9496c68fc48698f65c5e0d9563761681ec/test/data) coordinates for all 20,001 frames × 254 atoms match an independent MDTraj decoder exactly. NumPy alignment/selected-coordinate PCA comparisons pass, and the complete RNA pilot preserves 15 scientific artifacts and summary metrics. Tests and evidence remain outside the repository, as requested. CTest lists no registered tests, so this does not claim a full upstream test-suite pass.

Scope is Python trajectory selection/cache handling and documentation. Native code is unchanged; its pre-existing exact three-atom planar-alignment counterexample is outside this wrapper fix. `git diff --check` passes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f7aee6bb2d7e`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.