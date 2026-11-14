# Issue

**Title:** Security: replace pull_request_target event with plain pull_request, …

## Description

…replace commenter with step summary, and secure checkout parameters.

Also enables GitHub `actions` in CodeQL so that these security checks on workflow files are automated

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5b638c7a601d`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.