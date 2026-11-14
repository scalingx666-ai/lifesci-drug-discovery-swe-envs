# Issue

**Title:** Lbf fixup

## Description

---
name: lbf fixup
about: fixes subsetting in prebuilt tool `long-bond-finder`
title: 'Long bond finder subset fix'
labels: 'bug-fix'
assignees: ''
---
# Found a small but critical error in long-bond finder that makes its subsetting selection fail--it didn't call prune-bonds after subsetting.

## Changes proposed in this pull request
    - added call to prune bonds for scope if scope has bonds.
        - If model doesn't have all bonds, this test is skipped.


## Checklist
    - no open issue is addressed.
    - Have you applied a formatter to your code (e.g. flake8 or black for python, clang-format for C++)? Yes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `79e4ed168412`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.