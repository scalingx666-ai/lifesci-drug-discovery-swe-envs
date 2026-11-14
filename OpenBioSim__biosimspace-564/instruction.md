# Issue

**Title:** Build for Python 3.12-3.14 and drop 3.10 and 3.11

## Description

This PR adds support for Python 3.13 and 3.14, dropping support for 3.10 and 3.11. The Windows CI will currently fail due to a missing Sire package because of the usual CI network and IO glitches. Will re-run once that completes.

* I confirm that I have merged the latest version of `devel` into this branch before issuing this pull request (e.g. by running `git pull origin devel`): [y]
* I confirm that I have added a test for any new functionality in this pull request: [y]
* I confirm that I have added documentation (e.g. a new tutorial page or detailed guide) for any new functionality in this pull request: [y]
* I confirm that I have permission to release this code under the GPL3 license: [y]

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `03886d6e96b8`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.