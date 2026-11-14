# Issue

**Title:** index and id were swapped in translation from GEMMI API.

## Description

---
name: fix CIF parsing from GEMMI api
about: Very simple issue; the meaning of `atom_index` and `atom.serial` was swapped from the GEMMI to loos.
title: 'fix-cif'
labels: 'bug'
assignees: ''
---

Fixes #  CIF reading

## Changes proposed in this pull request

Bind `atom_index` to `pAtom->index` and `atom.serial` to `pAtom->id` in creating an AG from a [GEMMI mmcif atom](https://gemmi.readthedocs.io/en/stable/mol.html#atom).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `6266f705d052`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.