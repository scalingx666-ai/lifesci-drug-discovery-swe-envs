# Issue

**Title:** fix: hide compiled build artifacts from project file tree

## Description

PR #154 intentionally made user-imported reference files (.docx, .zip, media, …) visible in the file tree, but the removed ignore list also covered compiled build artifacts — so `.pyc`, `.so`, etc. started showing up, contradicting both the `__pycache__` directory skip that #154 kept and the PR #129 regression test (which has been failing on main since).

- Restore ignore rules for `.pyc .pyo .pyd .o .obj .so .dylib .dll` (reference-file visibility from #154 is unchanged)
- Update the #154 tests that asserted `.pyc`/`.pyd` visibility; the #129 test passes again
- Full suite: 200/200 green, biome + tsc clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `67dde775ac08`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.