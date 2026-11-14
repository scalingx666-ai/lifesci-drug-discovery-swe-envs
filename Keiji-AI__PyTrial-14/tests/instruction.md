# Issue

**Title:** Spot

## Description

Fixed a bug in spot.fit by putting the assignment of self.seq_data in front of calling self._fit_model.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `4c73f15d593a`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.