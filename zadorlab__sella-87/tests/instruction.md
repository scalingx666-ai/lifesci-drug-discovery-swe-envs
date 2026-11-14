# Issue

**Title:** Fix constrained IRC convergence checks

## Description

## Summary

- Project gradients onto the unconstrained subspace during IRC inner-loop convergence checks.
- Log the same projected forces, including when constraints were supplied directly to Sella rather than attached to the ASE atoms.
- Add a regression potential with a nonzero force on a frozen atom and test both ASE `FixAtoms` and Sella `Constraints`.

Without this projection, frozen slab forces can cause a false `IRCInnerLoopConvergenceFailure` even though the free-coordinate corrector has converged.

## Testing

- `345 passed`

.
Related to #20.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `6bacb9bd45be`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.