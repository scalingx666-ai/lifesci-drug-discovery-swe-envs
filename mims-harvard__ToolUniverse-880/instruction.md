# Issue

**Title:** Release tooluniverse 1.5.10

## Description

Bumps the package to 1.5.10. `publish-pypi` compares pyproject's version against PyPI's latest, so merging this is the release.

## What 1.5.9 -> 1.5.10 ships

Fourteen commits. The user-facing ones:

- **#879** three silent failures reported this month: nine documentation sites read the result envelope as a list or by `len()` (including a `safe_search` helper whose `isinstance(result, list)` branch returned `"Unexpected result format"` for *every successful call*); SMCP started with no `instructions`, so a client got a tool list and no statement of what the server is for; and `ClinicalTrials_search_studies` stamped `total_results_is_exact: true` on a zero that the AND-combining of `query.term` had produced, which is the wrong zero to call exact.
- **#878** two READMEs disagreed about how many skills exist and both were wrong — 229 skill directories, listed as 68 and 69.
- **#872 / #871 / #867** the field axis: 2,704 tools measured against what their upstreams actually return, ~60 wrappers widened to stop dropping fields they never asked for.
- **#865 / #866 / #863 / #861 / #860 / #859** the bundle's API-key count, the two shipped skill trees, a guide that told readers to catch an exception tool calls never raise, 235 wrapper annotations promising a list or string where the framework can answer with a dict, and the documentation standard that was itself the source of the stale "1000+ tools".

## Version strings

Eight, in eight files. #858 bumped four; the three plugin manifests already read 1.5.9 because the hand-bump happened to sit one ahead of the package. That coincidence is spent — `test_the_plugin_version_is_the_version_that_was_released` (added in #859) now holds them equal, so bumping pyproject alone fails it. `server.json` is included for consistency; `publish-mcp-registry` rewrites it from pyproject at publish time regardless.

## What was measured before cutting this

- 3,743 source files compile; all 417 modules changed since v1.5.9 import.
- Zero new `raise` statements in the hand-written tool modules across the whole window — the "tool calls return errors, never raise" contract is intact.
- Wheel built and installed into a clean 3.12 venv: `import tooluniverse`, `tu --version`, `tu list`, `tu info` all work from the installed package.
- Full local unit suite green; the three generators produce zero drift at the merge commit.
- **Full live sweep, all 8 shards, 687 patterns.** 35 categories fail and **none of their output contains a Python exception** — every failure is an upstream 429/403/5xx/timeout, a missing optional dependency, or an exhausted free allowance, each reported as a value. 21 were already failing at the 11-09 baseline; the other 14 are rate limits, anti-abuse 403s and upstream outages in families whose code this window did not touch.

## Known, not fixed here

The sweep found **40 tool families where a field declared since v1.5.9 has the wrong type in its `return_schema`** — `chembl.max_phase` declared integer but returned `'2.0'`, `gwas.or_value` number but `'1.33'`, `genebe.strand` integer but `False`, `umls.partialSearch` boolean but the request URL, and ~40 more. These are a side effect of the field-axis work: the fields reach the caller correctly, but their declared types were inferred rather than measured.

Nothing validates `return_schema` at runtime — `execute_function.py` discards it and `base_tool.py`'s jsonschema call checks *arguments* — so this cannot error or crash a user's call. It is visible in `tu test` and to an agent that reads a schema and trusts the type. Worth its own PR with a measured type per field; it is not a reason to hold a release that carries four real fixes. 16 further families have the same defect from before this window.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `7e682705f488`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.