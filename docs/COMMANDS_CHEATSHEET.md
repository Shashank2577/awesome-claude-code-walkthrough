# Claude Code · Commands & Configuration Cheatsheet (Deep Edition)

> Every slash command, hook event, MCP install, config path, and live-config example you'll need.
> Print double-sided. Pin to wall. Don't memorize — bookmark.

---

## 1. Slash commands · session control

| Command | Effect | Example |
| ------- | ------ | ------- |
| `/model` | Pick model interactively | `/model` then `2` for Sonnet |
| `/model <name>` | Switch directly | `/model opus` |
| `/effort low\|medium\|high\|xhigh` | Thinking budget per turn | `/effort xhigh` for hard problems (~20K tokens) |
| `/compact` | Summarize + drop history | `/compact` (no guidance, best-effort) |
| `/compact <guidance>` | Compact with pinning | `/compact preserve failing test names` |
| `/clear` | Wipe history, fresh context | Between unrelated tasks |
| `/resume` | Rewind to a previous turn | `/resume` lists turns; `/resume 3` rewinds 3 |
| `/cost` | Tokens + $ spent this session | End of every session |
| `/exit` or `Ctrl+D` | End session | |

## 2. Slash commands · workflow

| Command | Effect |
| ------- | ------ |
| `/plan` | Force plan mode for the next task |
| `/team` | Spawn named subagents |
| `/hooks` | Edit hooks live |
| `/mcp` | List/manage MCP servers |
| `/plugin` | Install/list plugins |
| `/skill <name>` | Invoke a skill explicitly |
| `/help` | Built-in help |
| `/feedback` | Report a bug to Anthropic |
| `/<your-custom>` | Any markdown file in `.claude/commands/` |

## 3. Keyboard

| Key | Action |
| --- | ------ |
| `Shift+Tab` | Toggle auto-accept mode |
| `Esc` | Cancel current tool call (mid-execution) |
| `Ctrl+C` (twice) | Quit session |
| `Ctrl+D` | Exit session |
| `↑` / `↓` | Recall previous prompts |
| `Ctrl+R` | Reverse-search prompts |
| `Tab` | Autocomplete file paths in prompts |

## 4. CLI flags

```bash
claude                         # interactive session
claude --version               # version check
claude --model <name>          # boot with specific model
claude --resume <session-id>   # continue a previous session
claude -p "<prompt>"           # one-shot, prints and exits
claude --print                 # render last response and exit
claude --mcp-config <path>     # custom MCP config path
claude --no-mcp                # disable all MCPs for this session
claude --strict                # require approval on every tool call
claude --allow <path>          # let Claude read paths normally blocked
claude --output-format json    # machine-readable output
```

`claude -p` is great for shell scripts:
```bash
SUMMARY=$(claude -p "summarize the last commit in 1 sentence")
echo "$SUMMARY" >> CHANGELOG.md
```

---

## 5. Hook events (the 9)

| Event | Fires | Stdin gets | Common use |
| ----- | ----- | ---------- | ---------- |
| `SessionStart` | Session boot | session metadata | Print MOTD, load env, log start |
| `UserPromptSubmit` | Every prompt before model sees it | the prompt text | Inject ticket ID, redact secrets |
| `PreToolUse` | Before every tool call | tool name + inputs | **Block destructive commands** |
| `PostToolUse` | After every tool call | tool result + inputs | **Format / lint / test on edit** |
| `Notification` | Long-running tool finishes | tool + duration | Desktop alert, Slack ping |
| `Stop` | Model finishes a turn | turn summary | Audit log, telemetry |
| `SubagentStop` | Subagent finishes | subagent result | Aggregate logs |
| `PreCompact` | Before /compact runs | what's about to drop | Snapshot history for replay |
| `SessionEnd` | Session quit | full transcript metadata | Export cost, archive transcript |

### Hook contract (memorize)

- Hooks are **shell commands** (anything you can run in your shell).
- Stdin is **JSON** with at least: `tool_name`, `tool_input`, `session_id`, `cwd`.
- **Non-zero exit** from a `PreToolUse` hook **blocks** the tool call.
- **Stdout** from a `PreToolUse` hook is treated as **rewritten input** (shape, don't block).
- Other hooks ignore exit code unless documented otherwise.
- Timeout: ~5 seconds per hook by default. Configurable.

### Hook environment variables

```
$TOOL_NAME           name of the tool being called
$TOOL_INPUT_FILE     for Edit/Write — path to the file
$SESSION_ID          unique per session, useful for log correlation
$CLAUDE_CWD          current working directory
$CLAUDE_USER         OS username
```

---

## 6. Settings file structure

`~/.claude/settings.json` (global) or `<repo>/.claude/settings.json` (repo-level):

```json
{
  "model": "sonnet",
  "effort": "medium",
  "auto_accept": false,
  "permissions": {
    "allow_paths": ["~/work/**", "/tmp/claude/**"],
    "deny_paths": [".env*", "secrets/**"]
  },
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          { "type": "command", "command": "echo 'session $SESSION_ID started' >> ~/.claude/log" }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.claude/scripts/block-destructive.sh" }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "~/.claude/scripts/redact-secrets.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "prettier --write \"$TOOL_INPUT_FILE\" 2>/dev/null || true" },
          { "type": "command", "command": "eslint --fix \"$TOOL_INPUT_FILE\" 2>/dev/null || true" }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          { "type": "command", "command": "~/.claude/scripts/require-ticket.sh" }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          { "type": "command", "command": "osascript -e 'display notification \"Claude finished\" with title \"Claude Code\"'" }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          { "type": "command", "command": "claude -p '/cost' >> ~/.claude/cost-log.jsonl" }
        ]
      }
    ]
  },
  "mcp": {
    "servers": {
      "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
      "context7": { "command": "npx", "args": ["-y", "@upstash/context7-mcp"] }
    }
  }
}
```

---

## 7. Config paths

| Path | Scope |
| ---- | ----- |
| `~/.claude/CLAUDE.md` | global instructions for every session |
| `<repo>/CLAUDE.md` | repo-level instructions |
| `~/.claude/settings.json` | global hooks, model defaults, MCP registry |
| `<repo>/.claude/settings.json` | repo-level overrides |
| `~/.claude/commands/*.md` | global slash commands |
| `<repo>/.claude/commands/*.md` | repo-level slash commands |
| `~/.claude/skills/*.md` | global skills |
| `<repo>/.claude/skills/*.md` | repo skills |
| `~/.claude/ignore` | files Claude must never read |
| `<repo>/.claude/ignore` | repo-level read-blocks |
| `~/.claude/scripts/` | conventional location for hook scripts |
| `~/.claude/log/` | conventional location for hook logs |

**Resolution order:** repo-level overrides global. The `<repo>/.claude/` is preferred for team-shared config; `~/.claude/` for personal.

---

## 8. MCP — install matrix

```bash
# Add an MCP server
claude mcp add <name> -- <command-to-launch-server>

# List
claude mcp list

# Remove
claude mcp remove <name>

# Inspect
claude mcp info <name>
```

### Servers worth knowing (April 2026)

```bash
# Fresh library docs (kills hallucinated APIs)
claude mcp add context7   -- npx -y @upstash/context7-mcp

# GitHub: issues, PRs, reviews
claude mcp add github     -- npx -y @modelcontextprotocol/server-github

# Browser automation
claude mcp add playwright -- npx -y @playwright/mcp

# Filesystem outside the repo
claude mcp add fs         -- npx -y @modelcontextprotocol/server-filesystem ~/work

# Linear (issue tracking)
claude mcp add linear     -- npx -y @linear/mcp-server

# Slack (messaging)
claude mcp add slack      -- npx -y @modelcontextprotocol/server-slack

# Sentry (error tracking)
claude mcp add sentry     -- npx -y @sentry/mcp-server

# PostgreSQL (read-only by default)
claude mcp add pg         -- npx -y @modelcontextprotocol/server-postgres "$DATABASE_URL"

# Memory (across sessions)
claude mcp add memory     -- npx -y @modelcontextprotocol/server-memory
```

### Writing your own MCP server

The MCP spec is small — JSON-RPC over stdio. Minimum viable server (Node):

```javascript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server({ name: 'my-internal-api', version: '1.0.0' });

server.setRequestHandler('tools/list', async () => ({
  tools: [{
    name: 'fetch_user',
    description: 'Fetch a user from our internal API by ID',
    inputSchema: { type: 'object', properties: { id: { type: 'string' } }, required: ['id'] }
  }]
}));

server.setRequestHandler('tools/call', async (req) => {
  if (req.params.name === 'fetch_user') {
    const user = await fetch(`https://api.internal/users/${req.params.arguments.id}`).then(r => r.json());
    return { content: [{ type: 'text', text: JSON.stringify(user) }] };
  }
});

await server.connect(new StdioServerTransport());
```

Half a day for a working prototype. Worth it for any internal system queried 10+ times a week.

---

## 9. Plugins

```bash
/plugin list                       # what's installed
/plugin search <keyword>           # browse marketplace
/plugin install <name>             # install
/plugin remove <name>              # uninstall
/plugin info <name>                # what it brings (commands, hooks, MCPs)
/plugin update <name>              # update to latest
```

**Worth trying:** `superpowers`, `claude-mem`, `graphify`. Pick one. Don't install three at once.

**Anatomy of a plugin** — a folder with:
```
my-plugin/
├── plugin.json            ← manifest
├── commands/              ← slash commands shipped
├── skills/                ← skills shipped
├── hooks/                 ← settings.json fragments
├── mcps/                  ← MCP servers to register
└── README.md
```

---

## 10. Custom command templates

### `/onboard` — explain this repo to a new engineer

`.claude/commands/onboard.md`:
```markdown
---
description: Onboard a new engineer to this repo
---
You are onboarding a new engineer.

1. Read CLAUDE.md and confirm the dev loop.
2. List every external service this repo talks to (databases, APIs, queues).
3. Identify the three files a new engineer should read first, in order.
4. Summarize the testing strategy in under 50 words.
5. Flag any onboarding pain points you spot in the code.

Keep total output under 400 words.
```

### `/review-pr <pr-number>` — apply our PR review rubric

`.claude/commands/review-pr.md`:
```markdown
---
description: Review the given PR with our team's rubric
---
You are reviewing PR #$ARGUMENTS using the team rubric below.

Rubric (in priority order):
1. Correctness — does the change match the spec? Any obvious bugs?
2. Tests — coverage of the change, including edge cases.
3. Security — secrets, input validation, authz.
4. Performance — obvious N+1, unnecessary loops, memory growth.
5. Style — only after 1-4 are clean.

Output format:
- A blocking comment if any of 1-3 fail.
- A recommended comment for 4.
- A nit comment for 5.

Use the github MCP to read the diff and post comments.
```

### `/runbook <service-name>` — generate a runbook

```markdown
---
description: Generate an incident runbook for a service
---
Generate an incident runbook for the service: $ARGUMENTS.

Sections:
- What this service does (1 paragraph)
- SLO + SLI (current + targets)
- Top 3 alert types and what they mean
- Diagnostic commands (logs, metrics, traces — be specific)
- Common failures and resolutions (last 90 days)
- Escalation contacts

Output as markdown for `runbooks/$ARGUMENTS.md`.
```

---

## 11. Skill templates

### `debugging.md` — root-cause-first

```markdown
---
name: debugging
description: Use when investigating a bug — root cause first, fix second
---
1. Reproduce the bug deterministically. If you can't reproduce, ask for a minimal repro before changing code.
2. Form a hypothesis BEFORE changing code. Articulate explicitly.
3. Add a failing test that captures the bug. Run it; confirm red.
4. Fix until the test passes.
5. Add at least one regression guard.
6. Summarize the root cause in 2 sentences for the commit message.
```

### `tdd.md` — test-first feature work

```markdown
---
name: tdd
description: Use when starting a new feature — test-first discipline
---
For every new function or behavior:
1. Write the failing test first. Watch it fail for the right reason.
2. Implement the minimum to make it pass. No extra features.
3. Refactor with the test as a safety net.
4. Add edge-case tests; iterate.

Resist:
- Writing the implementation first and the test after.
- Skipping the red-green dance.
- Implementing more than the test asks for.
```

### `brainstorm.md` — open-ended → structured

```markdown
---
name: brainstorm
description: Use when the question is open-ended ("how should we...", "what could we...")
---
Structure every answer as:
1. Three concrete options with one-line summaries.
2. For each option: implementation effort, risk, key trade-off.
3. Your recommended option, with reasoning.
4. One question for the user that would change your recommendation.
```

---

## 12. Demo recipes

### Recipe A — block destructive shell

`~/.claude/scripts/block-destructive.sh`:
```bash
#!/usr/bin/env bash
INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

PATTERNS=(
  'rm -rf /'
  'rm -rf \*'
  'git push.*--force.*(main|master)'
  'DROP TABLE'
  'DROP DATABASE'
  ':(){:|:&};:'
  'mkfs\.'
  'dd if=.*of=/dev/'
  'chmod -R 777'
)

for p in "${PATTERNS[@]}"; do
  if echo "$CMD" | grep -Eq "$p"; then
    echo "BLOCKED: matched destructive pattern: $p" >&2
    exit 1
  fi
done

exit 0
```

```bash
chmod +x ~/.claude/scripts/block-destructive.sh
```

Wire it under `PreToolUse` matcher `Bash` (see section 6).

### Recipe B — auto-test on edit

`PostToolUse` matcher `Edit|Write`:
```bash
# bash one-liner
npm test -- --findRelatedTests "$TOOL_INPUT_FILE" 2>/dev/null || true
```

Or for Python:
```bash
pytest --testmon "$TOOL_INPUT_FILE" 2>/dev/null || true
```

### Recipe C — require ticket ID

`~/.claude/scripts/require-ticket.sh`:
```bash
#!/usr/bin/env bash
INPUT=$(cat)
PROMPT=$(echo "$INPUT" | jq -r '.prompt // empty')

if ! echo "$PROMPT" | grep -Eq '\b[A-Z]{2,5}-[0-9]+\b'; then
  echo "REJECTED: prompt must include a ticket ID (e.g., PROJ-1234)" >&2
  exit 1
fi
exit 0
```

Wire under `UserPromptSubmit`.

### Recipe D — subagent research swarm

```
> dispatch 3 subagents in parallel:
  1. read src/billing/ and summarize the data model in 200 words
  2. read tests/billing/ and list coverage gaps as bullets
  3. grep TODO/FIXME/XXX/HACK in src/billing/ and tests/billing/, rank by risk
  Synthesize into a one-page assessment.
```

### Recipe E — desktop notification on long task

`Notification` hook:
```bash
osascript -e "display notification \"Claude finished a long task\" with title \"Claude Code\""   # macOS
notify-send "Claude Code" "Claude finished a long task"                                          # Linux
```

### Recipe F — log every prompt for compliance

`UserPromptSubmit` hook:
```bash
INPUT=$(cat)
echo "$INPUT" | jq -c '. + {ts: now, user: env.USER}' >> ~/.claude/log/prompts.jsonl
exit 0
```

(Always exit 0 — don't block the prompt.)

### Recipe G — cost dashboard end of session

`SessionEnd` hook:
```bash
INPUT=$(cat)
SESSION=$(echo "$INPUT" | jq -r '.session_id')
COST=$(echo "$INPUT" | jq -r '.total_cost_usd')
echo "$(date -Iseconds),$SESSION,$COST" >> ~/.claude/log/cost.csv
```

---

## 13. CLAUDE.md template (under 50 lines)

```markdown
# <repo-name>

This repo is the <one-line summary>.

## Dev loop

```bash
make install    # install deps
make test       # run tests
make dev        # run locally
make build      # production build
```

## Conventions

- TypeScript strict; no `any` without justification.
- Error handling: throw `AppError` (see `src/errors.ts`); don't swallow.
- Logging: `logger.info` with structured fields, never `console.log`.
- Database: every query goes through `src/db/`; no inline SQL.
- HTTP: handlers in `src/handlers/`; all responses via `respond.ok`/`respond.err`.

## Where to look

- Architecture: `docs/architecture.md`
- API spec: `docs/api.md` (OpenAPI; source of truth)
- ADRs: `docs/adr/`

## Don't

- Modify `src/generated/*` by hand (regenerate).
- Add new dependencies without a justification in the PR description.
- Touch `migrations/` outside of a database PR.
```

That's a good CLAUDE.md. Concrete commands, concrete rules, concrete pointers.

---

## 14. Daily-use one-liners

```bash
# Fast PR review
claude -p "review the diff for PR #1234 against our rubric in .claude/skills/code-review.md. post comments via the github MCP."

# Auto-summarize last 24h commits to standup
git log --since=yesterday --oneline | claude -p "summarize for daily standup, 3 bullets"

# Generate a one-pager from CLAUDE.md
claude -p "/onboard"

# Quick debug
claude -p "explain why this stack trace happens" < error.log

# Generate test scaffolding
claude -p "generate vitest tests for src/billing/charge.ts covering happy path and 3 edge cases"
```

---

## 15. Anti-cheatsheet — what NOT to do

- ❌ Always-Opus. Costs 5x for marginal gains on most tasks.
- ❌ /compact at 90%. Too late. Compact at 50%.
- ❌ Long CLAUDE.md (200+ lines). Slows every session, crowds context.
- ❌ Hooks with > 100ms latency on hot paths (PreToolUse for Bash).
- ❌ Plugin/MCP sprawl. Audit and prune quarterly.
- ❌ Vague skill descriptions. "Use for hard problems" doesn't match. "Use when investigating a bug" matches.
- ❌ Treating Claude Code like ChatGPT. Give it context, not just questions.
- ❌ Skipping the design step in spec-driven flow. The wins compound there.
- ❌ Using `--dangerously-skip-permissions`. Don't.

---

## 16. Quick links

- **This workshop tutorial:** https://shashank2577.github.io/awesome-claude-code-walkthrough/
- **Repo:** https://github.com/Shashank2577/awesome-claude-code-walkthrough
- **Official docs:** https://docs.claude.com/en/docs/claude-code/overview
- **MCP registry:** https://github.com/modelcontextprotocol/servers
- **Plugin list:** `/plugin search`
- **Anthropic Discord:** https://discord.gg/anthropic
- **Internal:** engineering_core@taazaa.com

---

*Bookmark. Don't memorize. Engineering Core · Taazaa · 2026*
