# deep-research

Multi-agent research pipelines for Claude Code — stop leaving your session to go research elsewhere.

## Who This Is For

You're mid-session, you need real research, and you don't want to break flow to Perplexity or ChatGPT and paste results back. Or you want to deeply understand an open-source repo before building on it. Or you need structured data on 20 competitors in a consistent schema.

These pipelines delegate research to agent teams so your top-level Claude stays free. Results come back as committed markdown in `docs/research/` — artifacts, not chat messages.

## Install

This is a Claude Code plugin. The first-class install path is to hand the job to your agent — paste the prompt below into a Claude Code session and let it follow the playbook in [`docs/install.md`](docs/install.md).

> Please install the deep-research-claude plugin from https://github.com/dbc-oduffy/deep-research-claude. The repo's `docs/install.md` is the install playbook — read it and follow the steps. Verify by running `/deep-research setup` at the end and report the result.

After the agent finishes, restart Claude Code (so the Agent Teams env var takes effect) and try:

```
/deep-research:research --mode=web "agent orchestration patterns in LLM frameworks"
```

## Pipelines

### Pipeline A — Internet Research

Haiku scout builds a source corpus via web search. 3-5 Sonnet specialists deep-read sources, verify claims, and challenge each other adversarially. Opus sweep agent checks coverage, fills gaps, and writes the final document. An optional iterative deepening pass targets high-severity gaps identified in the first sweep.

```
/deep-research:research --mode=web "topic"
```

### Pipeline B — Repository Research

2 Haiku scouts inventory every file in assigned chunks. 4 Sonnet specialists deep-read, analyze architecture, and optionally compare against a second project. Opus synthesizer writes the final assessment.

```
/deep-research:research --mode=repo /path/to/repo [--compare /path/to/mine] [--survey] [--deeper] [--deepest]
```

- `--compare` — gap-analysis artifact comparing target repo to your project
- `--deeper` — dependency-weighted repomap during scoping; specialists prioritize structurally central files
- `--deepest` — adds a Sonnet atlas agent producing architecture artifacts (file index, system map, connectivity matrix)

### Pipeline C — Structured Research

Schema-conforming batch research across N entities. Haiku scout maps findings to schema fields. 1-5 Sonnet verifiers challenge each other's values (CONFIRMED / UPDATED / REFUTED / CONTESTED). Opus synthesizer resolves contested fields and outputs validated YAML/JSON.

```
/deep-research:research --mode=structured tasks/research/spec.yaml subject-key
```

### Pipeline D — NotebookLM Research

Research YouTube videos, podcasts, and media Claude can't access directly, via NotebookLM. Haiku scout ingests sources. Sonnet workers query on focused sub-questions. Opus sweep writes the synthesis.

Requires the [notebooklm-mcp-cli](https://github.com/jacob-bd/notebooklm-mcp-cli) MCP server and a Google account with NotebookLM access.

```
/notebooklm-research "topic"
```

## Commands

| Command | Pipeline |
|---------|----------|
| `/deep-research:research --mode=web <topic>` | Internet research with iterative deepening |
| `/deep-research:research --mode=repo <path>` | Repository analysis (with optional `--compare`, `--survey`, `--deeper`, `--deepest`) |
| `/deep-research:research --mode=structured <spec> <key>` | Schema-conforming batch research |
| `/notebooklm-research <topic>` | NotebookLM media research |

All pipelines are fire-and-forget — the EM spawns the team and is freed. Results are committed to `docs/research/` automatically.

## Agents

| Agent | Model | Role |
|-------|-------|------|
| **research-scout** | Haiku | Web search, source vetting, shared corpus |
| **research-specialist** | Sonnet | Source verification, adversarial peer challenges, structured claims |
| **research-synthesizer** | Opus | Coverage check, gap-filling, final document |
| **repo-scout** | Haiku | File inventory with signatures, constants, data flow |
| **repo-specialist** | Sonnet | Architecture analysis, optional project comparison |
| **structured-synthesizer** | Opus | Schema validation, contested field resolution, final YAML/JSON |

## Integration with coordinator

This plugin works standalone. When used alongside the [coordinator plugin](https://github.com/dbc-oduffy/coordinator-claude), a `PreToolUse` hook automatically suggests research pipelines when Claude reaches for ad-hoc web search — nudging toward these structured pipelines instead of one-off `WebFetch` calls that consume the coordinator's context window.

### Optional dependency: `coordinator-safe-commit`

Pipeline commands (`/web`, `/repo`, `/structured`) invoke `~/.claude/plugins/coordinator-claude/coordinator/bin/coordinator-safe-commit` for phase-end commits. The helper provides scoped staging (per-session audit-trail integrity) for users running multiple concurrent agent sessions on the same branch.

**If you have the coordinator plugin installed,** no action needed — the helper is on the expected path.

**If running deep-research standalone,** substitute either form when you encounter the command:
- `git add <explicit-paths> && git commit -m "<subject>"` — manual scoped staging
- `git add -A && git commit -m "<subject>"` — blanket staging (acceptable for solo single-session use)

## Research Backing

Pipeline design derives from published guidance (OpenAI, Perplexity, Google, Anthropic, Stanford STORM) and is validated through [controlled experiments](docs/research/2026-03-31-deep-research-pipeline-evidence.md). Anthropic independently built a [production multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) using the same core pattern — their eval showed 90.2% improvement over single-agent. We converged on the same architecture independently; this system extends it with Haiku scouts for cost efficiency, adversarial peer dynamics between specialists, and asynchronous orchestrator dispatch.

## Source of Truth

This is the canonical home of the deep-research plugin. Originally developed as part of [coordinator-claude](https://github.com/dbc-oduffy/coordinator-claude) and extracted for independent distribution.

## Acknowledgements

Pipeline D is built on [notebooklm-mcp-cli](https://github.com/jacob-bd/notebooklm-mcp-cli) by [jacob-bd](https://github.com/jacob-bd) — an MCP server that provides programmatic access to Google NotebookLM.

---

[Dónal O'Duffy](https://github.com/dbc-oduffy) & Claude
