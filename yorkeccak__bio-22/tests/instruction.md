# Issue

**Title:** Refine the enterprise card copy and make it dismissible

## Description

## Summary
- Enterprise card now reads "Search and research for your AI" with a line about bringing the search, data and deep research behind the app to your own agents and workflows.
- Adds a dismiss control. The choice is remembered in localStorage so the card stays closed on later visits.
- Hidden below the medium breakpoint, where it collided with the mobile top bar.

## Test plan
- [x] tsc and eslint pass
- [ ] Preview deploy green

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `08eec3581add`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.