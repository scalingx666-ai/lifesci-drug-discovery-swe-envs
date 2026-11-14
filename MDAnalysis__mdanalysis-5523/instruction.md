# Issue

**Title:** MAINT: security update for cibuildwheel (eMudhra incident)

## Description

* Related to the security incident described here: https://discuss.python.org/t/information-for-python-users-regarding-emudhra-emsign-ca-certificate-security-incident/109576

* We should be using the latest `cibuildwheel` release as part of the mitigation. We should also use a hash rather than a tag for version pinning as SPEC8 explains:
https://scientific-python.org/specs/spec-0008/#pin-github-actions-release-workflows-to-their-full-release-commit-shas

## LLM / AI generated code disclosure
LLMs or other AI-powered tools (beyond simple IDE use cases) were used in this contribution: no

## PR Checklist
<!-- Please use the following checklist to ensure the PR is ready to be reviewed/merged. -->
 - [ ] Issue raised/referenced?
 - [ ] Tests updated/added?
 - [ ] Documentation updated/added?
 - [ ] `package/CHANGELOG` file updated?
 - [ ] Is your name in `package/AUTHORS`? (If it is not, add it!)
 - [ ] I have read and understand the current [AI Policy](https://github.com/MDAnalysis/mdanalysis/blob/develop/AI_POLICY.md)
 - [ ] LLM/AI disclosure was updated.

## Developers Certificate of Origin
<!--
In order for this PR to be merged you **must certify that you are able to submit your code to be included in MDAnalysis**.
-->

I certify that I can submit this code contribution as described in the [**Developer Certificate of Origin**](https://developercertificate.org/), under the MDAnalysis [LICENSE](https://github.com/MDAnalysis/mdanalysis/blob/develop/LICENSE).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `ae9ff4b73860`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.