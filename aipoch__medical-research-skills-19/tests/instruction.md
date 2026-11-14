# Issue

**Title:** fix(skills): correct deadline zones and generated output escaping

## Description

Fix three implementation defects found while reviewing unpublished Skills for the Open-Science Marketplace:

- Resolve resubmission deadlines with their stored IANA timezone, expose the documented `--timezone` option, and validate dates, unknown zones, DST gaps and ambiguous overlaps before saving. Interactive entry now asks for a timezone. Replace obsolete standard-library package requirements with `tzdata` for portable timezone data.
- Escape `<` in citation-network JSON embedded inside an HTML script element, preventing labels from terminating the script while preserving decoded graph data.
- Correct double-escaped regexes in the SVG generator so documented outline headings produce slides, title prefixes are stripped and spaces become filename separators. Emit actual line breaks between SVG bullet elements.

Validation: `python3 tests/publication_regressions.py` passes all 6 focused regressions locally (Python 3.14). Coverage includes cross-zone equivalence, DST transitions and invalid input, stored-data stability, CLI/interactive entry, hostile HTML labels, outline/prompt parsing and generated SVG XML. The read-only PR/manual workflow passed on Ubuntu, macOS and Windows with Python 3.10: https://github.com/aipoch/medical-research-skills/actions/runs/34758068451. It runs only these reviewed modules using temporary files, without network calls or unrelated Skill execution. `git -c core.whitespace=cr-at-eol diff --check` passes; the SVG source retains its existing CRLF line endings.

Visual validation: generated a two-slide synthetic outline, verified the SVG text in the browser and inspected a 1200 x 675 local raster rendering. Browser screenshot capture timed out; no browser screenshot is claimed.

Compatibility: no dual-format aliases or migration. Valid existing deadline JSON keeps its schema and bytes when read; remaining-time calculations now honor the existing timezone field. Invalid or ambiguous stored wall times require correction instead of silently using the machine timezone. The deadline tracker now documents Python 3.9+ and installation of its timezone-data dependency. Corrected SVG filenames can differ from the previously broken output; existing generated files are not renamed or deleted.

Enums/states: none added. The urgency table now describes the six levels already implemented.

Persistence: new deadlines continue to use the existing JSON timezone field. No new fields, automatic data rewrites, App database changes or Marketplace installation states.

Scope: these are upstream fixes only. This PR does not clear remaining content/provenance review findings, enroll Skills in the Marketplace, replace its pinned source commit, or modify its 383 published entries. Merging this upstream PR matches the repository's existing automatic release workflow for changed Skill paths.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f5ef65b9bea7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.