# Claude Code · Attendee Handout (Deep Edition)

> Self-paced reference. Read once after the workshop. Bookmark for the first month.

---

## 1. What Claude Code is — and isn't

**Is:** an *agentic* coding CLI. A teammate that reads your repo, edits files, runs commands, pushes commits, and talks to external systems through MCP.

**Is not:** autocomplete (that's Copilot). Not chat-in-IDE (that's Cursor). Not a chatbot (that's claude.ai).

**The category difference is the agent loop.** Claude can call tools. Tools run on your machine. Side effects happen. You stay in control via approvals and hooks.

**Three moves to remember:** Read → Edit → Do.

**One mental model:** a teammate who can touch your keyboard. Everything else in this handout is about giving that teammate better context, better tools, and better guardrails.

---

## 2. Install and first run

```bash
# Install
npm i -g @anthropic-ai/claude-code

# Verify
claude --version

# First run — auths via browser or API key
cd ~/your-repo
claude
```

You'll be prompted for either Pro/Max account login (browser flow) or an Anthropic API key. For the workshop start with Pro; switch to API once you're spending more than $40/mo.

**Required:** Node 20+. macOS, Linux, or Windows (WSL recommended).

**Test it:**
```
> what does this repo do? read CLAUDE.md if it exists, otherwise summarize from the top-level files.
```

If you got a coherent answer, you're set up.

---

## 3. The 6 commands you'll actually use

Out of ~30 slash commands, six earn their slot in your muscle memory.

| Command | When to reach for it | Example |
| ------- | -------------------- | ------- |
| `/model` | Task changes character | `/model opus` before debugging a hard issue |
| `/effort` | Stuck on a wall | `/effort xhigh` to give Opus 20K thinking tokens |
| `/compact` | Context past 50% | `/compact preserve the failing test names` |
| `/clear` | Starting a new task | `/clear` between unrelated tasks in same session |
| `/cost` | End of every session | Track for one week — calibrates intuition forever |
| `/hooks` | Mid-session policy change | Edit hooks without leaving Claude |

Beyond the six: `/resume` (rewind), `/plan` (force plan mode), `/team` (named subagents), `/mcp` (manage servers), `/skill` (invoke skill), `/plugin` (install/list).

---

## 4. Three models, one decision rule

> **Start in Sonnet. Drop to Haiku for drudgery. Escalate to Opus when you hit a wall.**

| Model | Use it for | Cost (per M tok in/out) | Approx vs Sonnet |
| ----- | ---------- | ----------------------- | ---------------- |
| Haiku 4.5 | commit messages, lint fixes, docstrings, mass renames | ~$0.80/$4 | 1/5x |
| Sonnet 4.6 | most coding, refactors, debugging, feature work | $3/$15 | 1x |
| Opus 4.6 | architecture, hard bugs, planning, complex migrations | ~$15/$75 | 5x |

Both Sonnet and Opus support 1M context. Don't take that as license to ignore context discipline (section 7).

**Why not always Opus?** Cost compounds. A team of 20 always-Opus burns $20K/mo on what $4K of Sonnet does better. Plus, smarter model with bad context is *worse* than fast model with good context.

---

## 5. The agent loop — the API of Claude Code

```
01 user message
02 model turn               ← /effort changes thinking budget here
03 PreToolUse hook fires    ← block / shape destructive things here
04 tool executes            ← Edit, Write, Bash, Read, Grep, etc.
05 PostToolUse hook fires   ← format / lint / test here
06 tool result returns      ← feeds into next turn's context
07 stop + (optional) notify ← desktop alert / Slack ping here
```

**Every behavior you'll ever change in Claude Code is one of these seven steps.** Memorize it. The next 5 sections all map back to it.

---

## 6. Setup ritual for a new repo (do this once, ~30 minutes)

### Step 1 — `CLAUDE.md` at repo root

Keep it under 50 lines. Cover:
- What this repo is, who uses it, why it exists.
- How to run / test / deploy. Concrete commands, not prose.
- 3-5 conventions worth enforcing (logging, error handling, naming).
- Paths to deeper docs (don't paste them inline).

Bad CLAUDE.md (200 lines of architecture diagrams in markdown). Good CLAUDE.md (40 lines of "this is the dev loop, here's the test command, here are the gotchas").

### Step 2 — `.claude/settings.json` with hooks

Two hooks, in order, in your first repo:

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash",
        "hooks": [{ "type": "command",
                    "command": "~/.claude/scripts/block-destructive.sh" }] }
    ],
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command",
                    "command": "prettier --write \"$TOOL_INPUT_FILE\" 2>/dev/null || true" }] }
    ]
  }
}
```

The destructive-shell blocker script lives in `~/.claude/scripts/block-destructive.sh` (template in CHEATSHEET.md).

### Step 3 — One custom command in `.claude/commands/`

Pick a task you do at least twice a week. Write a 20-30 line markdown file. Examples that pay off:
- `/onboard` — explain this repo to a new engineer
- `/review-pr` — apply your team's PR review rubric
- `/runbook` — generate an incident runbook for a service

### Step 4 — Install one MCP that matches your work

Pick one of `github`, `context7`, `playwright`, `figma`, or write your own for an internal API. Don't install all four. Live with one for a week.

### Step 5 — Commit `.claude/` to the repo

```bash
git add .claude/ CLAUDE.md
git commit -m "claude: shared agent setup for the team"
```

New engineers `git pull` and immediately have your team's hooks, commands, and MCP config.

---

## 7. Context discipline (the rules that save you from yourself)

Context rot is real. Models get measurably dumber past ~50% utilization. The fix is not bigger context — it's smaller, fresher context.

| Utilization | Zone | What to do |
| ----------- | ---- | ---------- |
| < 50% | Crisp | Keep going. Don't compact yet. |
| 50–70% | Compact soon | `/compact` *with guidance* |
| > 70% | Dumb zone | `/compact` or `/clear` *now* |

**The /compact guidance pattern:**

```
/compact preserve the failing test names in tests/billing/, the new columns added to invoices.sql, and the typescript signature of charge_invoice. drop everything else.
```

The guidance argument is your knob to keep what matters verbatim. Without it, /compact does its best, but it's a guess.

**Three habits that compound:**

1. **Compact at task boundaries**, not when the banner screams. Finish a sub-task → compact → start the next.
2. **Markdown in, not HTML/PDF/DOCX.** Convert before pasting. Pandoc is your friend. A 10K-word PDF balloons to 60K tokens; the same as markdown is closer to 12K.
3. **Use rewind (`/resume`).** A bad branch is not a forced restart — it's a checkout. Use it as a normal tool.

---

## 8. Token discipline (the only rules that matter)

1. **Cache everything stable.** System prompts, CLAUDE.md, large reference docs. Cache hits are ~10% the cost of normal reads. Put unstable parts at the end of your prompt to keep the cache valid.
2. **Compact at 50%, not 90%.** Past 70% the model is measurably dumber.
3. **One task per session.** Long sessions are not productivity; they're decay. Five 30-min sessions beat one 2.5hr ramble on every axis.
4. **Watch `/cost`** at session end for a week. Vibes lie. The meter doesn't.
5. **Match model to task.** Always-Opus is the #1 reason teams overspend.
6. **Subagents for heavy reads.** Anything > 30 files; dispatch.
7. **Tune extended thinking.** /effort low / medium / high / xhigh. Don't pay for thinking you don't need.
8. **Audit MCPs and plugins quarterly.** Sprawl is real.

**Cost math (memorize):**

```
Sonnet 4.6:  $3 / $15 per million tokens (in / out)
Opus 4.6:    ~5x Sonnet
Haiku 4.5:   ~1/5 Sonnet

A 1-hour deep session with extended thinking ≈ ~$1.05 on Sonnet
A 40-hour week of heavy use     ≈ $42 Sonnet / $210 Opus / $8 Haiku
Cache hits reduce input by ~90%
```

---

## 9. Subagents — when to dispatch

A subagent is an isolated Claude with its own context window, dispatched by your main session. Returns a summary.

**Dispatch when:**
- Task requires reading > 30 files.
- You need parallel research (3 directions, 1 synthesis).
- Work is independent (won't need feedback mid-flight).

**The pattern:**

```
> dispatch 3 subagents in parallel:
  1. read every file in src/billing/ and summarize the data model in 200 words
  2. read every test in tests/billing/ and list coverage gaps as bullet points
  3. grep for TODO/FIXME/XXX/HACK in src/billing/ and tests/billing/ and rank by risk
  Synthesize into a one-page assessment.
```

Subagents have their own context window. Your main stays clean. Their result returns as a single message. Total wall-clock time = the slowest subagent (they run in parallel).

**Don't dispatch when:**
- The task is 5 files deep — the dispatch overhead exceeds the savings.
- You need to react to interim findings — keep it in the main session.

---

## 10. MCPs — when each one matters

| MCP | Why | Install |
| --- | --- | ------- |
| `context7` | Fresh library docs (no 2023-vintage hallucinations) | `claude mcp add context7 -- npx -y @upstash/context7-mcp` |
| `github` | Issues, PRs, reviews from the CLI | `claude mcp add github -- npx -y @modelcontextprotocol/server-github` |
| `playwright` | Browser automation for E2E debugging | `claude mcp add playwright -- npx -y @playwright/mcp` |
| `figma` | Design specs into the prompt | `claude mcp add figma -- <figma-mcp-server>` |
| `filesystem` | Read/write outside the repo | `claude mcp add fs -- npx -y @modelcontextprotocol/server-filesystem ~/work` |

**Security:** MCPs run as you. Treat them like SSH keys. Audit before installing. Pin versions in production. Never install an MCP you can't read the source of.

**Writing your own:** the MCP spec is small (JSON-RPC over stdio). A working server for your internal API is a half-day prototype. Worth it for any system your team queries 10+ times a week.

---

## 11. Skills vs Commands vs Plugins (the right mental model)

| | Command | Skill | Plugin |
| -- | ------- | ----- | ------ |
| Trigger | You invoke it by name | Auto-invokes when situation matches | Bundles all of the above |
| File | `.claude/commands/foo.md` | `.claude/skills/bar.md` | npm-installable |
| Use for | Repeatable tasks you run | Habits you want Claude to pick up | Sharing community workflows |

**Command example:**
`.claude/commands/onboard.md`:
```markdown
You are onboarding a new engineer.
Read CLAUDE.md, list every external service this repo talks to,
then summarize the dev loop in under 200 words.
```
Invoke: `/onboard`

**Skill example:**
`.claude/skills/debugging.md`:
```markdown
---
name: debugging
description: Use when investigating a bug — root cause first, fix second
---
1. Reproduce deterministically.
2. Hypothesis BEFORE changing code.
3. Failing test that captures the bug.
4. Fix until test passes.
5. Regression guard.
```
Auto-invokes when you mention a bug.

**Plugin example:** `superpowers`, `claude-mem`, `graphify`. Pick one this week. Don't install five at once.

---

## 12. Spec-driven workflow (senior pattern)

```
Requirements  →  Design  →  Task breakdown  →  Implement  →  Verify
   (what/why)   (architect)   (testable units)   (one at a time)  (tests/types/lint)
```

**Junior pattern:** "build me a feature."
**Senior pattern:** "here is the spec, propose a design, break it down, implement task 1."

Same tool. 3x cleaner diffs. 1/4 the rework.

**The mini-spec template:**

```
> /effort high

Requirements:
- WHAT: <one sentence>
- WHY: <user value, not implementation>
- ACCEPTANCE: <bullet criteria>

Design:
- Propose architecture in 200 words.
- Sequence diagram for the happy path.
- Type signatures for new functions.
- Wait for my approval before any edit.

Task breakdown:
- Split into 3-5 testable units.
- Each unit: scope + acceptance + dependencies.

Implement task 1.
```

---

## 13. The first month (in order)

**Week 1 — install and write CLAUDE.md.**
Install Claude Code. Drop a 30-line CLAUDE.md into one repo. Use `claude` for at least one real task per day. Run /cost at the end of every session.

**Week 2 — first hook.**
Write a PreToolUse hook that blocks one destructive command. Watch it fire. Add a PostToolUse hook for your formatter.

**Week 3 — first custom command.**
Write one custom command for a task you do twice a week. Commit it to the repo so the team gets it.

**Week 4 — first MCP.**
Install one MCP. Use it on three real tasks. Decide if it earns its keep.

After month 1: subagents, plugins, spec-driven flow.

---

## 14. Troubleshooting

| Symptom | Likely cause | Fix |
| ------- | ------------ | --- |
| "Claude hallucinated an API that doesn't exist" | No MCP for fresh docs | Install `context7` |
| "Claude got dumb mid-session" | Context > 70% | `/compact` with guidance |
| "Cost is 5x what I expected" | Always-Opus, no caching | Switch to Sonnet, ensure CLAUDE.md is cached (stable prefix) |
| "Hook is slow" | Python startup, network calls in PreToolUse | Rewrite in bash, move expensive work to PostToolUse |
| "Claude won't read a file I want it to" | Path in `.claude/ignore` or gitignore | Check both; use `--allow` if needed |
| "Session is laggy" | Too many MCPs/plugins on boot | Audit; remove what you don't use |
| "Auto-accept stuck on" | Forgot Shift+Tab toggled it | Toggle back; watch the prompt indicator |

---

## 15. Compliance and security cheatsheet

- **Code sent to Anthropic:** Yes, prompts and read files. Gitignore + `.claude/ignore` block files.
- **Training:** Anthropic does not train on API/Pro/Max traffic by default.
- **HIPAA:** BAA available on enterprise.
- **SOC 2:** In place.
- **GDPR:** European data residency option.
- **Secret hygiene:** add `.env*` to `.claude/ignore`. PreToolUse hook can scan tool inputs. UserPromptSubmit hook can scan prompts.
- **Audit trail:** Stop hook logs every turn. SessionEnd hook archives transcript.
- **Destructive command protection:** PreToolUse hook with pattern matching.

---

## 16. Resources

- **This workshop's tutorial:** https://shashank2577.github.io/awesome-claude-code-walkthrough/
- **Workshop GitHub repo:** https://github.com/Shashank2577/awesome-claude-code-walkthrough
- **Official Claude Code docs:** https://docs.claude.com/en/docs/claude-code/overview
- **MCP registry:** https://github.com/modelcontextprotocol/servers
- **Claude Code on GitHub:** https://github.com/anthropics/claude-code
- **Anthropic Discord:** https://discord.gg/anthropic
- **Internal:** engineering_core@taazaa.com

---

## 17. Quick-reference card (cut this page out)

```
INSTALL          npm i -g @anthropic-ai/claude-code
RUN              cd repo && claude

THE 6:           /model · /effort · /compact · /clear · /cost · /hooks
ZONE LINE:       compact at 50%, not 90%
MODEL RULE:      Sonnet default → Haiku drudgery → Opus walls

LOOP:            user → model → preHook → tool → postHook → result → stop+notify

KILLER HOOKS:    PreToolUse: block destructive shell
                 PostToolUse: format / test on edit

WHEN TO DISPATCH: > 30 file reads, parallel research, independent
WHEN TO MCP:     external system used 10+ times/week
WHEN SPEC:       feature changes touch 3+ files OR cross modules

FIRST MONTH:     CLAUDE.md → hook → command → MCP, in that order
```

---

*Engineering Core · Taazaa · 2026*
