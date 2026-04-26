---
description: Set up the deep-research plugin — verify Agent Teams, check pipeline availability, configure NotebookLM. Safe to re-run.
allowed-tools: ["Read", "Bash", "Glob", "AskUserQuestion"]
argument-hint: "[--check-only]"
---

# Deep Research Setup

Verify prerequisites for the deep-research plugin's multi-agent pipelines. All pipelines require Agent Teams; Pipeline D also requires a NotebookLM MCP server.

If `$ARGUMENTS` contains `--check-only`, report status without making changes.

---

## 1. Agent Teams (required)

```bash
echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-not_set}"
```

- If `1`: ready.
- If not set: **All pipelines will fail without this.** Instruct the user to add to `~/.claude/settings.json`:
  ```json
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" }
  ```
  Takes effect on next Claude Code restart.

---

## 2. Pipeline Availability

Check which pipelines are available by looking for their command files relative to this plugin:

```bash
PLUGIN_DIR="${CLAUDE_PLUGIN_ROOT}"
for cmd in web repo structured; do
  test -f "$PLUGIN_DIR/commands/$cmd.md" && echo "$cmd: available" || echo "$cmd: missing"
done
```

Also check for the NotebookLM sub-plugin:

```bash
test -d "$PLUGIN_DIR/notebooklm" && echo "notebooklm: available" || echo "notebooklm: not installed"
```

Report:
- **Pipeline A** (Internet Research) — `/deep-research web`
- **Pipeline B** (Repo Research) — `/deep-research repo`
- **Pipeline C** (Structured Research) — `/deep-research structured`
- **Pipeline D** (NotebookLM Media Research) — `/notebooklm-research` (requires notebooklm sub-plugin)

---

## 3. NotebookLM Setup (Pipeline D only)

**Skip this section if the notebooklm sub-plugin is not installed.**

### 3a. MCP server CLI

Check if `notebooklm-mcp` is available:

```bash
command -v notebooklm-mcp 2>/dev/null || npx --yes notebooklm-mcp --version 2>/dev/null || echo "not_found"
```

- If found: ready.
- If not found: the bundled `.mcp.json` invokes `npx -y notebooklm-mcp`, so a global install is not required — `npx` will fetch on first launch. For faster startup, optionally install globally:
  ```bash
  npm install -g notebooklm-mcp
  ```
  Pipeline D requires `npx` (ships with Node.js). If `npx` itself is missing, install Node.js first.

### 3b. Sub-plugin enablement

The NotebookLM sub-plugin is disabled by default. Probe whether it's currently enabled:

```bash
SETTINGS="$HOME/.claude/settings.json"
if [ -f "$SETTINGS" ]; then
  if grep -qE '"(plugin_)?deep-research:notebooklm"\s*:\s*true' "$SETTINGS" 2>/dev/null \
     || grep -qE '"notebooklm"\s*:\s*true' "$SETTINGS" 2>/dev/null; then
    echo "notebooklm sub-plugin: enabled"
  else
    echo "notebooklm sub-plugin: NOT enabled (Pipeline D will not work until enabled)"
  fi
else
  echo "notebooklm sub-plugin: settings.json not found"
fi
```

If not enabled, instruct the user: enable the `notebooklm` sub-plugin in `~/.claude/settings.json` and restart Claude Code. Disable again after Pipeline D work to keep context lean.

### 3c. MCP health probe

If the notebooklm MCP server is running in the current session, ping it:

- If `mcp__notebooklm__get_health` is available as a tool, call it and report the result. A healthy response confirms end-to-end connectivity (tool surface → MCP server → NotebookLM auth).
- If the tool is **not** available, the sub-plugin is not loaded into this session — note that and skip.
- If the call fails (auth error, server unreachable), surface the error verbatim and recommend running `nlm login` in the terminal.

### 3d. Authentication

NotebookLM requires Google account authentication. The user must run `nlm login` in their terminal (outside Claude Code) to authenticate. This handles OAuth flow and session cookie extraction.

If auth expires mid-session, the `refresh_auth` MCP tool or re-running `nlm login` will fix it.

---

## 3.5. WebSearch / WebFetch Tool Availability

Pipelines A and D depend on `WebSearch` and `WebFetch`. Some Claude Code configs disable these tools (per-project allowed-tools restrictions, sandbox modes, or older Claude Code versions).

Probe by examining the available tool surface in the current session:

- If both `WebSearch` and `WebFetch` are present: ready.
- If either is missing: **warn loudly** in the status report. Pipelines A and D will fail. Recommend the user check `~/.claude/settings.json` (and any project-local `.claude/settings.json`) for `permissions.deny` rules that block these tools, or upgrade Claude Code if the version predates them.

---

## 4. Status Report

```
## Deep Research Setup

| Check                       | Status |
|-----------------------------|--------|
| Agent Teams env var         | ... (REQUIRED) |
| WebSearch tool              | ... (REQUIRED for Pipelines A/D) |
| WebFetch tool               | ... (REQUIRED for Pipelines A/D) |
| Pipeline A (web)            | ... |
| Pipeline B (repo)           | ... |
| Pipeline C (structured)     | ... |
| Pipeline D (notebooklm)     | ... |
| NotebookLM MCP CLI          | ... (if Pipeline D) |
| NotebookLM sub-plugin       | enabled / disabled |
| NotebookLM MCP health       | ... (mcp__notebooklm__get_health) |
| NotebookLM auth             | run `nlm login` in terminal |

### Available commands

- `/deep-research web <topic>` — Internet research with iterative deepening
- `/deep-research repo <path>` — Repository assessment (add `--compare`, `--deeper`, `--deepest`)
- `/deep-research structured <spec>` — Schema-conforming batch research
- `/notebooklm-research <topic>` — Media research (YouTube, podcasts, audio)
```

If Agent Teams is not set, make this prominent — nothing will work without it.

End with: _"Run `/deep-research web 'test topic'` to verify the setup works."_
