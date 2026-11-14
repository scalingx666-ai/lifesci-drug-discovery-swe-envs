# Issue

**Title:** Two READMEs disagreed about how many skills exist, and both were wrong

## Description

Reported from outside the project as #874: `README.md` advertises "68 pre-built
research workflows" while `skills/README.md` says "69 pre-built research skills".
They disagree with each other, and both are stale.

Measured on this branch's parent, there are 229 skill directories, each with a
`SKILL.md`:

| | |
|---:|---|
| 153 | research skills |
| 11 | setup and access (connect, database logins, publish a tool, share a machine) |
| 63 | install a GPU-backed model on this machine |
| 2 | developing ToolUniverse itself |

Both numbers understated the research catalogue by more than half.

The issue's own suggested correction, "230+", would have been wrong in the
other direction. It counts the 63 remote-model installers and the 2 developer
skills as research workflows, overstating the research catalogue by half, and
229 directories exist rather than 230.

### The table was the larger problem

The table under `skills/README.md`'s own **All Skills** heading listed 73 of
the 229 directories. 94 research skills and 62 remote-tool setup skills were
missing from a list that claims to be complete. Every entry in it did exist,
so nothing had been deleted -- the list simply stopped being extended. It is
now generated from the directories, split into the four sections above, each
declaring its own count.

### Why a test and not just a correction

#859 and #860 fixed this same class of stale count in other files and did not
touch these two. A number maintained by hand goes stale between the commit
that fixes it and the next skill added.

`tests/unit/test_skill_readmes_count_what_exists.py` asserts that every skill
directory appears in exactly one table, that nothing is listed that does not
exist, that each heading's declared count matches the rows under it, and that
`README.md`'s headline equals the research count. All of it fails on the
parent commit.

.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `996970f3b6df`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.