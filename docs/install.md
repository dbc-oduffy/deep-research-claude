# Install Playbook (agent-readable)

> This file is the install paper trail. The expected entry point is a human asking their Claude Code agent to install this plugin — the agent reads this file and follows it. Humans who want to install by hand can read it too; the steps are the same, just done manually.

## What this plugin is

A Claude Code plugin (`.claude-plugin/plugin.json` at the repo root). It ships:

- `commands/` — slash commands (`/deep-research:research --mode=web|repo|structured`, `/deep-research setup`)
- `agents/` — scout / specialist / synthesizer definitions
- `skills/`, `pipelines/` — supporting prompts and protocols
- `notebooklm/` — optional sub-plugin for Pipeline D (media research)

The plugin requires the **Agent Teams** experimental flag in Claude Code. Without it, every pipeline fails to launch.

## Agent: install procedure

Follow these steps in order. Do not skip the verification at the end.

### 1. Pick the install location

The convention is `~/.claude/plugins/deep-research-claude/`. If `~/.claude/plugins/` does not yet exist, create it. If a directory already exists at the target, treat that as an existing install and run **Update** (below) instead of **Install**.

### 2. Clone the repo

```bash
git clone https://github.com/dbc-oduffy/deep-research-claude.git ~/.claude/plugins/deep-research-claude
```

On Windows + Git Bash the same path expands correctly. If the user is on Windows + PowerShell only, use `$env:USERPROFILE\.claude\plugins\deep-research-claude` as the target.

### 3. Enable Agent Teams

Read `~/.claude/settings.json`. Ensure the `env` object contains:

```json
"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
```

If `env` is missing, add it. If the key is already set to `"1"`, leave it alone. Do not touch unrelated keys. The change takes effect on the next Claude Code restart — flag this to the user at the end.

### 4. (Optional) Pipeline D — NotebookLM media research

Skip unless the user explicitly wants Pipeline D. It costs context and requires extra setup.

1. The `notebooklm/` sub-plugin is already in the cloned repo; it is **disabled by default** to keep context light.
2. Pipeline D needs the [`notebooklm-mcp-cli`](https://github.com/jacob-bd/notebooklm-mcp-cli) MCP server. Check availability with `command -v notebooklm-mcp || npx --yes notebooklm-mcp --version`. If missing, suggest `npm install -g notebooklm-mcp-cli`.
3. NotebookLM auth happens out-of-band: the user runs `nlm login` in their terminal. The agent cannot do this — surface the instruction.
4. The user enables the sub-plugin in `~/.claude/settings.json` only when they want to run Pipeline D, and disables it after.

### 5. Verify

After clone + env var, the user must restart Claude Code for the env var to take effect. After restart, the agent (or the user) runs:

```
/deep-research setup
```

This is the canonical health check. It reports per-pipeline status. If `Agent Teams env var` shows anything other than `1`, installation is not complete — re-check `~/.claude/settings.json`.

A green `/deep-research setup` is the install success signal. Do not declare success without it.

### 6. Smoke test (optional but recommended)

```
/deep-research web "test topic — multi-agent orchestration"
```

Pipeline A is the cheapest verification path. Repo and structured pipelines need additional inputs.

## Agent: update procedure

If `~/.claude/plugins/deep-research-claude/` already exists:

```bash
cd ~/.claude/plugins/deep-research-claude
git fetch origin
git status   # confirm clean — abort if user has local edits
git pull --ff-only origin main
```

If the working tree is dirty, **stop and ask the user** before doing anything destructive. Local edits to a plugin directory usually mean the user is hacking on it.

After update, re-run `/deep-research setup` to confirm health.

## Agent: uninstall procedure

```bash
rm -rf ~/.claude/plugins/deep-research-claude
```

Optionally remove `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` from `~/.claude/settings.json` only if the user has no other plugin relying on Agent Teams. When in doubt, leave the env var alone.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `/deep-research` not found after install | Claude Code not restarted, or plugin path wrong | Restart; verify `~/.claude/plugins/deep-research-claude/.claude-plugin/plugin.json` exists |
| Pipeline launches then immediately fails | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` not `1` | Set in `~/.claude/settings.json`, restart |
| `/notebooklm-research` not found | NotebookLM sub-plugin disabled | Enable it in `~/.claude/settings.json` |
| Pipeline D auth errors | NotebookLM session expired | User runs `nlm login` in their terminal |
| `coordinator-safe-commit: command not found` mid-pipeline | Running standalone without coordinator plugin | See README "Optional dependency: `coordinator-safe-commit`" — substitute `git add <paths> && git commit -m ...` |

## Manual install (humans)

Same steps as above without the agent in the loop:

1. `git clone https://github.com/dbc-oduffy/deep-research-claude.git ~/.claude/plugins/deep-research-claude`
2. Add `"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"` to the `env` block of `~/.claude/settings.json`.
3. Restart Claude Code.
4. Run `/deep-research setup` to verify.
5. (Optional) Set up Pipeline D per step 4 above.
