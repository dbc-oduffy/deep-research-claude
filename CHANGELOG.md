# Changelog

All notable changes to the deep-research plugin are documented here.

## [1.5.0] - 2026-06-24

Minor release. Brings the deep-research install flow to parity with the coordinator agent-install chain, and migrates the NotebookLM (Pipeline D) MCP launch to a working transport.

### Added

- **Agent-install chain parity with coordinator.** `/deep-research:setup` is now the install-chain walker (the prior scaffolding verb was renamed `/deep-research:install`), matching the ecosystem-wide `/<plugin>:setup` walker convention. `scripts/setup.{sh,ps1}` gained `--phase` dispatch (deep-research's first), a chain-preinstall consent gate (`COORDINATOR_CHAIN_PREINSTALL_CONSENT`), and Available/Informational phase-list headers. The agent-install manifest, schema, and `AGENT.md` were expanded to the v3 contract.
- **DR capability probe** (`scripts/lib/dr_capability_probe.sh`) — a deep-research-authored preflight reporting `agent_teams` / `web_access` / `pipelines_present` / `notebooklm` / `python` readiness, composed into a 3-section `--preflight` output (capabilities first, environment prereqs filtered to what deep-research needs).
- **Vendored coordinator prereq unit** (`scripts/lib/coordinator_prereq/{prereq_probe,manifest_reader,step_zero_emit}.sh`) — byte-identical copies of the coordinator SSOT, kept in sync by `bin/verify-prereq-probe-sync.sh`. deep-research is now named as the second downstream vendor in the coordinator source headers.

### Changed

- **NotebookLM MCP launch migrated from `npx` to `uvx`.** The prior `npx`-based launch was broken (the server is a Python package); Pipeline D now launches via `uvx`. NotebookLM **v0.7.8 capabilities** were adopted into Pipeline D (including `cross_notebook_query` for single-call cross-notebook verification in the coverage auditor).
- **`python3`-first interpreter resolution** across the setup scripts, with the PowerShell `Find-Python` hardened to coordinator parity (functional probe + py-launcher, Windows-Store-stub-safe). Killed a `validate` silent-bypass.
- **Research scratch moved to `docs/research/{run-id}-{slug}-workdir`** — out of the prior kill-zone path.

### Fixed

- **Coverage auditor Workflow-dispatch persistence.** The auditor gained a `Bash` heredoc write-fallback so its `-coverage-audit.md` sidecar still lands on disk when dispatched inside a `Workflow()` (where subagent `Write` is denied), not only under Agent Teams.
- **Console-flash production findings cleared repo-wide** (PM-directed `verify-no-powershell-flash` sweep) and the `quota-tripwire` snippet propagated to 45 agents.

## [1.4.0] - 2026-05-31

Minor release. Adds an always-on post-synthesis coverage auditor across all four pipelines.

### Added

- **Coverage auditor** (`agents/coverage-auditor.md` + `pipelines/coverage-auditor-prompt-template.md`) — a fresh-eyes, non-teammate agent dispatched after every synthesis to answer "did the synthesis carry the research?" It reads specialist claim records, cross-references the synthesis, and emits a `-coverage-audit.md` sidecar; it never writes the synthesis output path. Always-on: no size floor, no opt-out.
- **Specialist fidelity relay** — idle specialists are woken to verify their own content was faithfully represented in the synthesis. Wired across all four pipelines (web / repo / structured / notebooklm) as an internal synthesizer phase before team teardown; relay triggers are pipeline-specific (Pipeline D's relay is out of scope until D gains a depth concept).

## [1.3.1] - 2026-05-09

Patch release. Scout hardening + small skill-surface tightening; no behavioral changes for end users.

### Changed

- **Repo + Internet scouts hardened against TEXT-ONLY hallucination.** Scouts now write the deliverable to disk and only emit `DONE: <path>` after verifying the file exists; inline summaries without a written file count as task failure. Aligns Pipeline A and B scouts with the coordinator-side text-only-recovery preamble snippet.
- **Skill descriptions trimmed to the 150/175-char description budget** across pipeline-facing skills; surface unchanged.

### Fixed

- **`agents/repo-scout.md`, `agents/repo-specialist.md`, `pipelines/repo-{scout,specialist}-prompt-template.md`, `skills/eval-output.md`** updated in lockstep; `commands/research.md` reflects current `--mode` flag wording.

## [1.3.0] - 2026-04-30

Agent-driven install becomes the first-class path. Also a checkpoint: rolling up state-of-the-plugin notes for anyone arriving since the last release.

### Changed

- **Install is now agent-first.** README's lead install instruction is a paste-to-agent prompt, not a shell script. The agent reads [`docs/install.md`](docs/install.md) — a structured playbook covering install, update, uninstall, optional Pipeline D / NotebookLM, verification via `/deep-research setup`, and troubleshooting — and executes the steps. Manual steps still live at the bottom of the same file for humans who want them.
- **Removed dead `bash setup/install.sh` reference** from README. The script never existed in the publish repo; the prior README pointed at a path that returned 404.

### Added

- **`docs/install.md`** — canonical install playbook. Single source of truth for the agent's install paper trail; covers Windows + POSIX paths, Pipeline D opt-in, and a per-symptom troubleshooting table.

### State of the plugin (recap)

For consumers arriving fresh, here's what `1.3.0` ships:

- **Pipeline A (Internet Research)** — Haiku scout + 3-5 Sonnet specialists with adversarial peer challenges + Opus sweep with iterative deepening for high-severity gaps.
- **Pipeline B (Repo Research)** — 2 Haiku scouts + 4 Sonnet specialists + Opus synthesizer. `--compare` produces gap-analysis vs. your project; `--deeper` adds dependency-weighted repomap; `--deepest` adds three-phase atlas generation (file index, system map, connectivity matrix, architecture summary).
- **Pipeline C (Structured Research v2.1)** — schema-conforming batch research with adversarial peer challenges, CONTESTED-field resolution, and an output-first synthesizer (skeleton → reconcile → validate → overwrite).
- **Pipeline D (NotebookLM Media Research)** — opt-in sub-plugin for YouTube / podcasts / audio Claude can't directly access. Disabled by default to keep context light; enable per-session.
- **`/deep-research setup`** — health check that reports per-pipeline status; canonical install verification command.
- **Standalone-friendly** — `bin/safe-commit` shim provides the same scope-checking discipline as the upstream coordinator workflow without requiring the full plugin.

## [1.2.2] - 2026-04-29

Maintenance release: docs trim, contribution policy, and safe-commit shim.

### Added

- **`CONTRIBUTING.md`** — explicit maintainer-approval PR policy for the publish repo. Clarifies that this mirror accepts issues and discussion but PRs are merged at the maintainer's discretion (upstream development happens elsewhere).
- **`bin/safe-commit`** — thin shim that delegates to `coordinator-safe-commit`. Gives publish-repo contributors the same scope-checking discipline as the upstream coordinator workflow without requiring the full plugin install.

### Changed

- **Trimmed `/deep-research` command docs.** `commands/repo.md` cut from 481→340 lines, `commands/web.md` from 360→233 lines. Phase-internal detail extracted to `pipelines/repo-internals.md` and `pipelines/web-internals.md` so command files stay focused on the user-facing contract.

### Internal

- Coordinator-safe-commit now optional dependency note added for standalone use.

## [1.2.1] - 2026-04-26

Maintenance release: setup hardening and publish-manifest fixes.

### Fixed

- **Publish manifest completeness.** Added `commands/setup.md`, `pipelines/repo-atlas-sketch-prompt-template.md`, and `pipelines/repo-survey-prompt-template.md` to `publish-manifest.txt`. These files were git-tracked but not listed in the manifest, risking drift on future syncs.
- **Cross-platform NotebookLM MCP config.** Replaced Windows-only `cmd /c notebooklm-mcp` invocation in `notebooklm/.mcp.json` with a direct `notebooklm-mcp` command call, restoring macOS/Linux compatibility.
- **`/setup` false-positive readiness.** Setup now also probes (a) NotebookLM sub-plugin enablement in `~/.claude/settings.json`, (b) `mcp__notebooklm__get_health` when available, and (c) `WebSearch` / `WebFetch` tool availability (Pipelines A and D depend on them). The status table reflects these new checks.
- **`/research` router missing tool.** Added `Skill` to the `allowed-tools` of `commands/research.md` — the router invokes pipeline sub-skills via the Skill tool but the previous allowlist (`Read`, `Bash`) would have blocked it.

### Changed

- **Internal artifact exclusion.** Added `.gitattributes` to mark `docs/research/pipeline-benchmark/` and `docs/research/2026-03-31-deep-research-pipeline-evidence.md` as `export-ignore`, so they're stripped from `git archive` based publishes. Files remain in-repo for historical context.

## [1.2.0] - prior release

Baseline: Pipeline A v2.2 (Internet Research), Pipeline B (Repo Research, including `--deepest` three-phase mode), Pipeline C (Structured Research), Pipeline D (NotebookLM Media Research, sub-plugin).
