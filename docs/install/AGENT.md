---
manifest: agent-install-manifest.json
contract: docs/wiki/agent-install-contract.md
---

# Agent install guide — deep-research-claude

> **Chain position:** deep-research-claude sits at **step 4 of 5** in the install DAG. It is an
> OPTIONAL dependency of holodeck only — the broader coordinator ecosystem does not require it.
> Its single direct dep is `coordinator-claude` (soft severity).

> **You are an agent reading this file** because the setup skill dispatched you to walk the
> install chain for deep-research-claude. Follow the install contract at
> `docs/wiki/agent-install-contract.md`. Do not improvise. If a step fails, stop and report.

---

## Chain posture

deep-research-claude occupies **chain step 4 of 5** in the five-position install DAG:

```
1. coordinator-claude   (chain root; always installed first)
2. project-rag          (hard dep of holodeck; soft dep of ue-addon)
3. project-rag-ue-addon (hard dep of holodeck; optional otherwise)
4. deep-research-claude  ← YOU ARE HERE (OPTIONAL dep of holodeck only)
5. claude-unreal-holodeck (chain leaf)
```

This repo declares **one direct dep**: `coordinator-claude` at `soft` severity.

- **Why soft?** coordinator-claude is strongly recommended (it provides the session-management
  spine, persistent handoffs, and the review pipeline that make deep-research valuable), but a
  standalone install is possible via the direct scripts. The chain-walker warns on absence and
  offers to install; it does not block.
- **Holodeck OPTIONAL posture.** Holodeck lists deep-research-claude as `optional` in its own
  manifest. The install chain passes through this repo only if the operator opted in at the
  pre-restart question. Nothing downstream of deep-research (in the holodeck chain) treats it
  as a hard gate.

## Orientation-less posture

deep-research is orientation-less — no orientation baton is seeded and no prior handoff is
superseded (situational use; invoke `/deep-research` when needed).

This posture is documented in the coordinator's install-deep-research template at
`plugins/coordinator/templates/handoffs/install-deep-research.md` § Note.
Coordinator seeds the install-leg spinoff for deep-research (from that template) when the
operator opts in; deep-research itself does NOT seed a spinoff or supersede anything. After
install, the only ceremony needed is `/reload-plugins` + `/reload-skills` — deep-research is
then live. Invoke `/deep-research` situationally; there is no persistent orientation arc to
maintain.

## Install via the chain-walker

The `/deep-research:setup` skill reads `agent-install-manifest.json` at this path and walks
the dep chain dynamically. For each dep it:

1. Checks sibling presence at `../<sibling_dir_name>` (implicit sibling check).
2. Runs the declared `functional_probe` (here: `file_exists` at
   `plugins/coordinator/CLAUDE.md` within the coordinator sibling).
3. On `soft` dep absent: warns loudly, offers to clone-and-walk via `gh`, proceeds if declined.
4. Composes the v2 standalone-script invocation (per contract § Walker composition):
   `<consumer_install_args...> --i-am-agent <upstream skip_dep_check> <upstream accept_hallucination_risk>`

Full walker contract: `docs/wiki/agent-install-contract.md`.

## Override flags

Both flags must be passed together; either alone exits with code 93.

| Flag (key in manifest) | CLI string (value in manifest) | Purpose |
|---|---|---|
| `skip_dep_check` | `--skip-dep-check` | First flag of the override pair |
| `accept_hallucination_risk` | `--accept-missing-deps-risk` | Second flag; signals explicit accept of missing-dep risk |

> **Authority boundary note.** These flag values are upstream-authored — deep-research-claude
> declares them here. A consumer chain-walker reads them from this manifest at dispatch time
> rather than hard-coding them, so a rename in this manifest propagates automatically. The
> key names (`skip_dep_check`, `accept_hallucination_risk`) are schema-canonical per the
> v2 contract.

## What `/deep-research` provides

Multi-agent deep-research pipelines for Claude Code:

- **Pipeline A (Internet Research)** — 1 Haiku scout + 3–5 Sonnet specialists + 1 Opus synthesizer
- **Pipeline B (Repo Research)** — 2 Haiku scouts → 4 Sonnet specialists → 1 Opus synthesizer
- **Pipeline C (Structured Research)** — schema-conforming batch research with adversarial peer challenges
- **Pipeline D (NotebookLM Research)** — media research via NotebookLM MCP (requires NotebookLM server)

Invoke via `/deep-research:research --mode=<web|repo|structured>` or `/notebooklm-research`.
All pipelines require `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in `settings.json`.

---

<!-- spec-backlink: docs/plans/2026-06-15-deep-research-install-chain-application-phase-b.md §7 C1 -->
<!-- spec-backlink: plugins/coordinator/docs/wiki/agent-install-contract.md -->
