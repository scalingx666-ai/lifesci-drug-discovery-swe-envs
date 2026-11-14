# Issue

**Title:** Three silent failures reported this month: a key check that always passed, a server that said nothing, and a zero that meant the opposite

## Description

Three issues filed in the last week, each reproduced live before being fixed,
each its own commit, every guard proven to fail on its parent.

### #869 -- a documented API-key check that printed success whether or not the key worked

A tool call returns `{status, data, metadata}`. `docs/guide/api_keys.rst`, the
page a reader follows to confirm their keys work, decided with
`"success" if len(result) > 0 else "failed"`. `len()` of the envelope is 3
whatever happened.

The issue named one bad guard in the literature tutorial; there are six, four
of them the same block copied. The worst was not in the report: `safe_search`
returned `"Unexpected result format"` for **every call that succeeded**,
because an envelope is a dict without an `error` key and fell through to the
else.

The guard does not ban a spelling. It reads each code block, finds the names
bound from `tu.run(...)`, and fails if such a name is then used as a sequence.

### #875 -- the MCP host was told this server's name and nothing else

FastMCP's `instructions` is the one piece of text a host puts verbatim into
the model's context at session start. SMCP passed only `name`. Under deferred
tools the model therefore saw `find_tools`, `grep_tools`, `execute_tool` and
nothing saying those names reach genes, drugs, trials or variants.

The text is derived from what the server actually loaded, so it stays true as
the catalogue grows and as a deployment's filters narrow it. A caller-supplied
value is left alone.

Two things only running it revealed: `find_tools` is registered straight onto
FastMCP and never enters `_exposed_tools`, so reading that set alone
advertised four entry points on a five-entry-point server; and `compact_mode`
and `tool_finder` are category keys but are this server's own plumbing --
listing them as subject areas would send the model looking for a
"compact_mode" database.

### #870 -- zero trials because every word had to match

`query.term` ANDs its words against the indexed record text:

| query_term | studies |
|---|---|
| `KEYNOTE-045` | 1 -- NCT02256436, the trial |
| `KEYNOTE-045 pembrolizumab urothelial carcinoma second-line platinum` | **0** |

Every word of the second is true of that trial, and 51,291 studies match at
least one of them. `total_results_is_exact` was declared on the zero, which
made it harder to doubt rather than easier.

Same answer this module already gives for rewritten queries: disclose, never
substitute. An empty many-worded term query now carries `term_match_check`
with the OR'd wording and how many studies match any one word.

It is built so it can come back negative: `query_term="zzqq wwxx yyvv"`
reports that no study matches even one word, so that empty result is **not**
an artefact of the AND. A check that says the same thing whatever the counts
carries no information.

---

Full unit suite green (exit 0, zero FAILED/ERROR). Generated files are the
generators' own output; only the two touched tools' hashes moved.

. . .

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `996970f3b6df`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.