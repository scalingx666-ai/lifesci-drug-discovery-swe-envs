# Issue

**Title:** Add GHA CI

## Description

Travis will change pricing models soon, and this has been due for a long time now.

Supersedes #1232 #547

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `59fc6313b3b7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.