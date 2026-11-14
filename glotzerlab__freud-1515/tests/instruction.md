# Issue

**Title:** Update dependency prefix-dev/pixi to v0.84.1

## Description

This PR contains the following updates:

| Package | Type | Update | Change |
|---|---|---|---|
| [prefix-dev/pixi](https://redirect.github.com/prefix-dev/pixi) | uses-with | minor | `v0.83.0` → `v0.84.1` |

---

### Release Notes

<details>
<summary>prefix-dev/pixi</summary>

### [`v0.84.1`](https://redirect.github.com/prefix-dev/pixi/releases/tag/v0.84.1)

[Compare Source](https://redirect.github.com/prefix-dev/pixi/compare/v0.84.0...v0.84.1)

##### \[0.84.1] - 2026-11-04

##### ✨ Highlights

This patch release fixes a memory-usage regression introduced in v0.82.1,
alongside some smaller fixes and work towards future features.

##### Features

- Integrate conda-pypi wheels using released rattler by [@&#8203;nichmor](https://redirect.github.com/nichmor) in
- pixi-build-ros: Rtklib by [@&#8203;Timple](https://redirect.github.com/Timple) in

##### Fixes

- cli: Fix multiple reporters problems by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- export: Honour `--ignore-source-errors` in conda explicit export by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- run: Detect only the installed environment's host by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- reporters: Hide the repodata bar on clear with downloads in flight by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in

##### Maintenance

- Adapt to new `exclude-newer` design by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Publish dev release builds of main via octo-sts by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- pixi_api: type the run errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in

### [`v0.84.0`](https://redirect.github.com/prefix-dev/pixi/releases/tag/v0.84.0)

[Compare Source](https://redirect.github.com/prefix-dev/pixi/compare/v0.83.0...v0.84.0)

##### \[0.84.0] - 2026-11-02

##### ✨ Highlights

`exclude-newer` has been redesigned to support more precise and configurable exclusions.
The top-level `[exclude-newer]` and `[pypi-exclude-newer]` tables have been deprecated
in favour of corresponding `[workspace]` sub-tables. These tables can now consist of
fine-grained `exemptions` to a given `cutoff`, which can be used to, for example, apply
a blanket exclusion period while exempting vetted security releases.
New `pixi workspace exclude-newer` and `pixi global exclude-newer` subcommands
allow configuration via the command line.

TLS options are now available for `pixi self-update`, and TLS configuration is
inspectable via `pixi info`.

This release includes a `rattler` bugfix which avoids hash collisions for nonequal
package versions, eliminating a class of false cache-hits.
Lock files written by older Pixi versions may need to have hashes
updated by `pixi lock` before they are considered locked (even if no dependencies change).
If this affects your workspace, it is recommended to have all collaborators on the workspace
use Pixi>=0.84.0, to achieve lock file stability.

##### Breaking changes

Errors are now emitted instead of warnings when a configuration file fails to parse.

- config: Make a config file that fails to parse a hard error by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in

##### Features

- self-update: Expose TLS options by [@&#8203;baszalmstra](https://redirect.github.com/baszalmstra) in
- lock: Honor exclude-newer exemptions in satisfiability by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- manifest: Deprecate the top-level `exclude-newer` tables by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- global: Use match spec exemptions in `exclude-newer` by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- cli: Add exclude-newer commands for workspaces and pixi global by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Add cutoffs to exclude-newer exemptions by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- pixi_api: Expose run and shell by [@&#8203;haecker-felix](https://redirect.github.com/haecker-felix) in
- Per-index exclude-newer for PyPI extra indexes by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Add nanobind-dev and python3-typing-extensions as in rosdistro by [@&#8203;EzraBrooks](https://redirect.github.com/EzraBrooks) in
- info: Show effective TLS root certificates by [@&#8203;baszalmstra](https://redirect.github.com/baszalmstra) in
- pixi-build-backend: Let backends generate multi-output recipes by [@&#8203;EzraBrooks](https://redirect.github.com/EzraBrooks) in
- Add sha256 to workspace explicit export by [@&#8203;bollwyvl](https://redirect.github.com/bollwyvl) in
- publish: No `publish = true` required if explicit `--path` option is given by [@&#8203;Timple](https://redirect.github.com/Timple) in
- pixi_api: Add GlobalContext with pixi global list by [@&#8203;haecker-felix](https://redirect.github.com/haecker-felix) in
- config: Add `exclude-newer` by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- publish: Build with the exclude-newer of the workspace by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in

##### Fixes

- Bump webpki-root-certs to 1.0.11 to drop removed eMudhra roots by [@&#8203;baszalmstra](https://redirect.github.com/baszalmstra) in
- publish: Keep Artifactory context path when uploading by [@&#8203;wolfv](https://redirect.github.com/wolfv) in
- Keep PyPI releases without an upload time for exempted packages by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Reuse the build backend environment with a relative exclude-newer by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- cli: Honour colour settings in help and argument errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- cache: Use xxh3 for all on-disk cache names by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Skip system CA store for pypi mapping middleware client by [@&#8203;r3wretrhy](https://redirect.github.com/r3wretrhy) in
- global: Remove packages with custom exposed names by [@&#8203;Hofer-Julian](https://redirect.github.com/Hofer-Julian) in
- pixi-build-ros: Support cross-compiling `ament_python` packages by [@&#8203;Timple](https://redirect.github.com/Timple) in
- tasks: Fix various bugs by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- exclude-newer: Resolve relative cutoffs once per workspace load by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in

##### Documentation

- Document exclude-newer exemptions by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Add `release` scope to automated release commit messages by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Adapt vouch request by [@&#8203;Hofer-Julian](https://redirect.github.com/Hofer-Julian) in
- Clarify how to ping us by [@&#8203;Hofer-Julian](https://redirect.github.com/Hofer-Julian) in
- setup-pixi: Mention setup-pixi being pre-v1 by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- setup-pixi: Document configuration input and bump to v0.11.3 by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Fix PowerShell custom shell example for GitHub Actions by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- setup-pixi: Fix instructions for enabling debug logging by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in

##### Maintenance

- release: Provide Cargo through a shared Rust feature by [@&#8203;Hofer-Julian](https://redirect.github.com/Hofer-Julian) in
- core: Add `Cause` wrapper, type the lock-file load, write and prefix errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- global: Type the trampoline, install and completions errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- global: Type the global directory and conda-meta errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Update rattler and rattler-build crates by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Add Cargo.lock from pixi-build example by [@&#8203;pavelzw](https://redirect.github.com/pavelzw) in
- Cover cache-name collision for zero-component versions by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Revamp contributor docs and PR template by [@&#8203;baszalmstra](https://redirect.github.com/baszalmstra) in
- global: Type the global manifest errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Enforce vouching for pull requests by [@&#8203;Hofer-Julian](https://redirect.github.com/Hofer-Julian) in
- global: Type the project discovery and list errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- global: Type the install and sync errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- pixi_api: Type the shell errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- api: Type the init errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- pixi-build-ros: Add tqdm by [@&#8203;Timple](https://redirect.github.com/Timple) in
- Fix missing `InitError::Config` by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Remove duplicate `InitError::Config` by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- api: Type the workspace setter errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- api: Type the add errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Require approval before publishing releases by [@&#8203;wolfv](https://redirect.github.com/wolfv) in
- api: Type the task errors by [@&#8203;lucascolley](https://redirect.github.com/lucascolley) in
- Add AGENTS.md by [@&#8203;Hofer-Julian](https://redirect.github.com/Hofer-Julian) in
- lock: Never install the conda prefix in the PyPI satisfiability check by [@&#8203;ytausch](https://redirect.github.com/ytausch) in

##### New Contributors

- [@&#8203;r3wretrhy](https://redirect.github.com/r3wretrhy) made their first contribution in
- [@&#8203;EzraBrooks](https://redirect.github.com/EzraBrooks) made their first contribution in

</details>

---

### Configuration

📅 **Schedule**: (in timezone America/Detroit)

- Branch creation
  - Between 05:00 PM and 11:59 PM, only on Wednesday (`* 17-23 * * 3`)
- Automerge
  - At any time (no schedule defined)

🚦 **Automerge**: Enabled.

♻ **Rebasing**: Whenever PR is behind base branch, or you tick the rebase/retry checkbox.

🔕 **Ignore**: Close this PR and you won't be reminded about this update again.

---

 - [ ] <!-- rebase-check -->If you want to rebase/retry this PR, check this box

---

This PR was generated by [Mend Renovate](https://mend.io/renovate/). View the [repository job log](https://developer.mend.io/github/glotzerlab/freud).
<!--renovate-debug:eyJjcmVhdGVkSW5WZXIiOiI0NC4yMDQuMCIsInVwZGF0ZWRJblZlciI6IjQ0LjIwNC4wIiwidGFyZ2V0QnJhbmNoIjoibWFpbiIsImxhYmVscyI6WyJkZXBlbmRlbmNpZXMiXX0=-->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `25dddec0113f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.