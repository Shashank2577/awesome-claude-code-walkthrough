# Claude Code · Attendee Handout

> One-page takeaway from the 2-hour workshop. Print double-sided, fold once.

## What is Claude Code?

Claude Code is an **agentic coding CLI** — a teammate that reads your repo, edits files, runs commands, and talks to external systems through MCP. It is *not* autocomplete (Copilot) and *not* chat-in-IDE (Cursor). It is a separate category: an agent.

**Three moves:** Read → Edit → Do.

## Install (5 minutes, do this today)

```bash
npm i -g @anthropic-ai/claude-code
claude --version
cd ~/your-repo
claude
```

First run will prompt for an API key or browser auth.

## The 6 commands you'll actually use

| Command | When to reach for it |
| ------- | -------------------- |
| `/model` | Task changes character (start of session, before hard problems) |
| `/effort` | Stuck on a wall — raise thinking budget |
| `/compact` | Context meter past 50% |
| `/clear` | Starting a new task |
| `/cost` | End of every session |
| `/hooks` | Mid-session policy change |

## Three models, one rule

> **Start in Sonnet. Flip to Haiku for drudgery. Flip to Opus when you hit a wall.**

| Model | Use it for | Cost |
| ----- | ---------- | ---- |
| Haiku 4.5 | commit messages, lint fixes, docstrings | cheapest |
| Sonnet 4.6 | most coding, refactors, debugging | balanced |
| Opus 4.6 | architecture, hard bugs, planning | priciest |

## The agent loop (memorise this)

```
01 user message
02 model turn
03 PreToolUse hook   ← block / shape destructive things here
04 tool executes
05 PostToolUse hook  ← format / lint / test here
06 tool result
07 stop + notify
```

Every behavior you'll ever change in Claude Code is one of these seven steps.

## Setup ritual for a new repo (do this once)

1. **Add `CLAUDE.md`** at repo root, **under 50 lines.** Cover:
   - what this repo is
   - how to run / test / deploy
   - 3-5 conventions worth enforcing
   - paths to deeper docs (don't paste them inline)
2. **Add `.claude/settings.json`** with hooks:
   - `PreToolUse` to block `rm -rf`, force-push to main, `DROP TABLE`
   - `PostToolUse` to run `<your-formatter>` on edited files
3. **Add one `.claude/commands/onboard.md`** — your team's onboarding ritual.
4. **Install one MCP** that matches your work — `github`, `context7`, `playwright`, or `figma`.
5. Commit `.claude/` so the team gets the same setup.

## Token discipline (the only rules that matter)

- **Cache everything stable.** System prompts, CLAUDE.md, big docs.
- **Compact at 50%, not 90%.** Past 70% the model is measurably dumber.
- **One task per session.** Long sessions are not productivity; they are decay.
- **Watch `/cost`** at session end. Vibes lie.
- **Match model to task.** Always-Opus is the #1 reason teams overspend.

## Context zones

| Utilization | Zone | What to do |
| ----------- | ---- | ---------- |
| < 50% | Crisp | Keep going |
| 50-70% | Compact soon | `/compact` *with guidance* |
| > 70% | Dumb zone | `/compact` or `/clear` now |

## Subagents — when to dispatch

Reach for a subagent when:
- the task requires reading > 30 files
- you need parallel research (3 directions, 1 synthesis)
- the work is independent (won't need feedback mid-flight)

Subagents have their own context window. Your main stays clean. Their result returns as a single message.

## MCPs to know

| MCP | Why |
| --- | --- |
| `context7` | fresh library docs (no 2023-vintage hallucinations) |
| `github` | issues, PRs, reviews from the CLI |
| `playwright` | browser automation for E2E debugging |
| `figma` | design specs into the prompt |

Install a new MCP:
```bash
claude mcp add <name> -- <command>
```

## Skills vs Commands vs Plugins

- **Command** (`.claude/commands/foo.md`) — a recipe you invoke with `/foo`.
- **Skill** (`.claude/skills/bar.md`) — a habit Claude auto-invokes when the situation matches.
- **Plugin** — a bundle of commands + skills + hooks + MCPs you install in one line.

## Spec-driven workflow (senior pattern)

```
Requirements  →  Design  →  Task breakdown  →  Implement  →  Verify
   (what/why)   (architect)   (testable units)   (one at a time)  (tests/types/lint)
```

Junior: "build me a feature." Senior: "here is the spec, propose a design, break it down, implement task 1."

## Cost napkin math

```
Sonnet 4.6:  $3 / $15 per million tokens (in / out)
Opus 4.6:    ~5x Sonnet
Haiku 4.5:   ~1/5 Sonnet
```

A 1-hour deep session ≈ ~$1.50 on Sonnet with extended thinking. Multiply by 5 for Opus. Track it.

## What to do this week

1. Install Claude Code on your work laptop.
2. Drop a 30-line CLAUDE.md into one repo you own.
3. Add one PreToolUse hook that blocks one destructive command.
4. Write one custom slash command for a task you do twice a week.
5. Run `/cost` at the end of every session for 5 days. Notice the pattern.

That's it. Five small steps, in order, and Claude Code becomes part of your default loop.

## Resources

- **Workshop tutorial (this site):** https://shashank2577.github.io/awesome-claude-code-walkthrough/
- **Claude Code docs:** https://docs.claude.com/en/docs/claude-code/overview
- **MCP registry:** https://github.com/modelcontextprotocol/servers
- **Internal:** engineering_core@taazaa.com

---

*Engineering Core · Taazaa · 2026*
