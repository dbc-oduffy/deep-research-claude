# Changelog

All notable changes to the deep-research plugin are documented here.

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
