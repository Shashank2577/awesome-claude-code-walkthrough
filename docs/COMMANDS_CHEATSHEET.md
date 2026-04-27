# Claude Code · Commands Cheatsheet

> Every command, hook event, and config path on one page. Print it.

## Slash commands · session control

| Command | Effect |
| ------- | ------ |
| `/model` | Pick model (interactive) or `/model sonnet` |
| `/effort low\|medium\|high\|xhigh` | Thinking budget per turn |
| `/compact [guidance]` | Summarise + drop history; keep what you tell it to |
| `/clear` | Wipe history, fresh context |
| `/resume` | Rewind to a previous turn |
| `/cost` | Tokens + $ spent this session |
| `/exit` or `Ctrl+D` | End session |

## Slash commands · workflow

| Command | Effect |
| ------- | ------ |
| `/plan` | Force plan mode for the next task |
| `/team` | Spawn named subagents |
| `/hooks` | Edit hooks live |
| `/mcp` | List/manage MCP servers |
| `/plugin` | Install/list plugins |
| `/skill <name>` | Invoke a skill explicitly |

## Keyboard

| Key | Action |
| --- | ------ |
| `Shift+Tab` | Toggle auto-accept |
| `Esc` | Cancel current tool call |
| `Ctrl+C` (twice) | Quit session |
| `↑` | Recall previous prompt |

## Hook events (9)

| Event | Fires | Common use |
| ----- | ----- | ---------- |
| `SessionStart` | session boot | load context, print MOTD |
| `UserPromptSubmit` | every prompt | inject ticket ID, redact secrets |
| `PreToolUse` | before any tool | block destructive commands |
| `PostToolUse` | after any tool | format, lint, test |
| `Notification` | long-running done | desktop alert, Slack ping |
| `Stop` | model finishes a turn | log, audit |
| `SubagentStop` | subagent finishes | aggregate results |
| `PreCompact` | before /compact | snapshot what's about to be dropped |
| `SessionEnd` | session quit | export cost, archive transcript |

Hook contract:
- Hooks are shell commands.
- Stdin is JSON with `TOOL_INPUT`, `TOOL_NAME`, `SESSION_ID`, etc.
- Non-zero exit from `PreToolUse` **blocks** the tool call.
- Stdout to a `PreToolUse` hook **rewrites** `TOOL_INPUT`.

Example `~/.claude/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.claude/scripts/block-destructive.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "prettier --write \"$TOOL_INPUT_FILE\"" }
        ]
      }
    ]
  }
}
```

## Config paths

| Path | Scope |
| ---- | ----- |
| `~/.claude/CLAUDE.md` | global instructions for every session |
| `<repo>/CLAUDE.md` | repo-level instructions |
| `~/.claude/settings.json` | global hooks, model defaults |
| `<repo>/.claude/settings.json` | repo-level overrides |
| `~/.claude/commands/*.md` | global slash commands |
| `<repo>/.claude/commands/*.md` | repo-level slash commands |
| `~/.claude/skills/*.md` | global skills |
| `<repo>/.claude/skills/*.md` | repo skills |
| `~/.claude/ignore` | files Claude must never read |

## MCP

```bash
claude mcp add <name> -- <command-to-launch-server>
claude mcp list
claude mcp remove <name>
```

Common servers:
```bash
claude mcp add github     -- npx -y @modelcontextprotocol/server-github
claude mcp add context7   -- npx -y @upstash/context7-mcp
claude mcp add playwright -- npx -y @playwright/mcp
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/work
```

## Plugins

```bash
/plugin list
/plugin install <name>
/plugin remove <name>
```

Worth trying: `superpowers`, `claude-mem`, `graphify`.

## Custom command template

`.claude/commands/onboard.md`:
```markdown
---
description: Onboard a new engineer to this repo
---
You are onboarding a new engineer.
Read CLAUDE.md, list every external service this repo talks to,
then summarise the dev loop in under 200 words.
```

Invoke: `/onboard`

## Skill template

`.claude/skills/debugging.md`:
```markdown
---
name: debugging
description: Use when investigating a bug — root cause first, fix second
---
1. Reproduce the bug deterministically.
2. Form a hypothesis BEFORE changing code.
3. Add a failing test that captures the bug.
4. Fix until test passes.
5. Add regression guards.
```

Skills auto-invoke when the description matches the situation.

## CLI flags

```bash
claude --version
claude --resume <session-id>
claude --model sonnet|opus|haiku
claude -p "<prompt>"          # one-shot, no interactive session
claude --mcp-config <path>    # custom MCP config
claude --print                # render last response and exit
```

## Demo recipes

### 1. Block destructive shell

`~/.claude/scripts/block-destructive.sh`:
```bash
#!/usr/bin/env bash
INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
if echo "$CMD" | grep -Eq '(rm -rf /|git push.*--force.*main|DROP TABLE)'; then
  echo "BLOCKED: destructive pattern" >&2
  exit 1
fi
exit 0
```
Wire it up under `PreToolUse` matcher `Bash`.

### 2. Auto-test on edit

`PostToolUse` matcher `Edit|Write`:
```bash
npm test -- --findRelatedTests "$TOOL_INPUT_FILE"
```

### 3. Subagent research swarm

```
> dispatch 3 subagents in parallel:
  1. read src/billing/ and summarise the data model
  2. read tests/billing/ and list coverage gaps
  3. grep TODO/FIXME in billing/ and rank by risk
  Synthesise into a one-page assessment.
```

---

*Bookmark this page. Don't memorise it.*
