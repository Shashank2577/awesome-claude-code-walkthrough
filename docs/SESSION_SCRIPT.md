# Claude Code · 2-Hour Workshop · Presenter Script

> Audience: working software engineers — most have used Copilot/Cursor/ChatGPT but never an agentic CLI.
> Format: project the live site, scroll tab-by-tab, demo in a real terminal alongside.
> Total: 120 minutes.
>
> **Setup before you start:** open two windows side-by-side — the tutorial site on the left, a terminal in a sample repo on the right. In the terminal, pre-warm `claude` so the first run isn't a 15-second cold start. Have `~/.claude/CLAUDE.md`, `~/.claude/settings.json`, and at least one `.claude/commands/*.md` file ready to show.

## Timing map

| # | Tab | Min | Cumulative |
| - | --- | --- | ---------- |
| 0 | Welcome + live install demo | 7 | 0:07 |
| 1 | What it is | 4 | 0:11 |
| 2 | Ecosystem | 4 | 0:15 |
| 3 | Models | 5 | 0:20 |
| 4 | Usage | 4 | 0:24 |
| 5 | Agent loop | 5 | 0:29 |
| 6 | Commands | 5 | 0:34 |
| 7 | Commands ref | 3 | 0:37 |
| 8 | Custom cmds | 4 | 0:41 |
| 9 | Hooks | 7 | 0:48 |
| 10 | Context | 5 | 0:53 |
| 11 | Context pro | 3 | 0:56 |
| 12 | Session limits | 3 | 0:59 |
| 13 | Subagents | 6 | 1:05 |
| 14 | Plugins | 4 | 1:09 |
| 15 | Plugins deep | 3 | 1:12 |
| 16 | MCPs | 7 | 1:19 |
| 17 | Skills | 6 | 1:25 |
| 18 | Token craft | 4 | 1:29 |
| 19 | Cost math | 4 | 1:33 |
| 20 | Spec-driven | 5 | 1:38 |
| 21 | Case study | 3 | 1:41 |
| 22 | By role | 4 | 1:45 |
| 23 | What's new | 3 | 1:48 |
| 24 | Resources + close | 2 | 1:50 |
| Q | Q&A buffer | 10 | 2:00 |

---

## 0 · Welcome + live install (7 min)

**Say:** "For the next two hours we're going to learn Claude Code — not as a chat tool, but as a teammate who can touch your keyboard. By the end you'll have it installed, configured, hooked into your repo, and you'll know which knobs to turn when it costs too much, slows down, or hallucinates."

**Demo:**
```bash
# in a fresh terminal
npm i -g @anthropic-ai/claude-code   # one line install
claude --version                      # confirm
cd ~/work/some-real-repo
claude                                # boot the agent
```

While it boots, point at the screen: "It just read CLAUDE.md, listed my MCPs, and offered to resume yesterday's task. We'll unpack each of those."

**Transition:** "Top nav — 24 tabs. Each is a self-contained idea. Let's start at the beginning."

---

## 1 · What it is (4 min)

**Headline on screen:** *"It's your terminal's senior engineer, on tap."*

**Say:** "Claude Code does three things — Read, Edit, Do. It reads your repo and the markdown you point it at, edits multiple files with proper diffs, and executes commands — git, tests, browsers, databases — all gated by tools you can review or block. The four numbers on screen — 40-70% cost reduction, 1M context, 9 hook events, ∞ surfaces — are what the rest of this session is about."

**Key line to repeat:** "Think of it as a teammate who can touch your keyboard. Everything else is about giving that teammate better context, better tools, better guardrails."

**Audience prompt:** "How many of you have CLAUDE.md in a repo you own? Keep that hand up — we'll come back to it in 40 minutes."

---

## 2 · Ecosystem (4 min)

**Click through the six surfaces:** CLI, IDE, Desktop, Web, Browser, MCP-driven agents.

**Say:** "One brain, six doors in. Same agent, same context model, same tools — just different surfaces. The CLI is the canonical surface; everything else is a window onto it. Pick the door for the job: terminal for deep work, IDE for inline edits, desktop for screen-aware tasks, web for shareable artifacts, browser for end-to-end testing, MCP-driven agents for unattended runs."

**Demo:** show `claude` running in terminal AND the VS Code extension's inline panel doing the same thing.

---

## 3 · Models (5 min)

**Click each model card** — Haiku 4.5, Sonnet 4.6, Opus 4.6.

**Say verbatim:** "Three models, one choice per task. Start in Sonnet. Flip to Haiku when the task is mechanical — commit messages, lint fixes, docstrings. Flip to Opus when you hit an architecture or debugging wall."

**Demo:**
```
$ /model           # show interactive picker
$ /model sonnet
$ /effort high     # raise thinking budget
$ /effort xhigh    # 20K thinking tokens — for the gnarly stuff
```

**Why this matters:** "Model choice is the biggest single lever on cost and latency. A junior using Opus for everything will burn 5x what a senior using Sonnet+Haiku correctly will."

---

## 4 · Usage (4 min)

**Say:** "There are three ways to think about a session:
1. **Plan mode** — Claude reads, thinks, writes a plan, asks you to approve before any edit. Use for unfamiliar repos.
2. **Auto-accept** — small, isolated edits where you trust the agent. Toggle with `Shift+Tab`.
3. **Free-flow** — you in the loop on each diff. Default for production code."

**Demo:**
```
$ claude
> implement a /healthz endpoint with a 200 OK
# show how it shows the diff and waits for approval
> Shift+Tab          # flip to auto-accept
```

---

## 5 · Agent loop (5 min)

**Walk through the 7 numbered steps** on the diagram:
01 User message → 02 Model turn → 03 Hooks fire (pre) → 04 Tool executes → 05 Hooks fire (post) → 06 Tool result → 07 Stop + (optional) Notification.

**Say:** "Memorise this loop. Every behavior you'll ever change in Claude Code is one of these seven steps. Want to block destructive commands? PreToolUse hook (step 3). Want to format on save? PostToolUse hook (step 5). Want to be paged when it's done? Notification (step 7). Want it to think harder? Step 2 — bigger thinking budget via `/effort`."

**Anchor:** "The loop is the API."

---

## 6 · Commands (5 min)

**Click each group**, hover the most common commands.

**The six to memorise:**
| Command | What | When |
| ------- | ---- | ---- |
| `/model` | switch model | task changes character |
| `/effort` | thinking budget | hit a wall |
| `/compact` | summarise + drop history | ~50% context |
| `/clear` | nuke history | new task |
| `/cost` | show $ + tokens spent | end of every session |
| `/hooks` | edit hooks live | mid-session policy change |

**Demo all six in one flow.**

---

## 7 · Commands ref (3 min)

**Say:** "The full reference. Don't memorise — bookmark. Two you'll discover later: `/resume` rewinds the session, `/team` orchestrates named subagents."

**Audience:** "Open the live page on your laptop and ⌘F for any command you wonder about during the rest of the session."

---

## 8 · Custom commands (4 min)

**Say:** "Any markdown file in `.claude/commands/` becomes a slash command. Project-local in the repo, global in `~/.claude/commands/`."

**Demo — write a `/onboard` command live:**
```bash
mkdir -p .claude/commands
cat > .claude/commands/onboard.md <<'EOF'
You are onboarding a new engineer.
Read CLAUDE.md, list every external service this repo talks to,
then summarise the dev loop in under 200 words.
EOF
```
Then in claude:
```
> /onboard
```

**Why this matters:** "This is how teams scale Claude Code. A 30-line command captures tribal knowledge that used to take a senior an hour to explain."

---

## 9 · Hooks (7 min — go deep)

**Say:** "Nine lifecycle events. Hooks are shell commands Claude runs at named moments. They can log, modify, or block."

**Walk through the events** (click each pill on the page): SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Notification, Stop, SubagentStop, PreCompact, SessionEnd.

**The two killer use-cases:**
1. **Block destructive shell:** PreToolUse hook that rejects `rm -rf /`, `git push --force` to main, `DROP TABLE`.
2. **Auto-run tests on edit:** PostToolUse hook that runs `npm test -- --findRelatedTests` on every file Claude edits.

**Demo — show a real settings file:**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{ "type": "command", "command": "~/.claude/scripts/block-destructive.sh" }]
    }]
  }
}
```

**Anchor:** "Hooks turn ‘smart assistant' into ‘team-policy-enforcing teammate.' Non-zero exit blocks the tool. Rewriting `TOOL_INPUT` shapes it instead. Block narrowly; shape often."

---

## 10 · Context (5 min)

**Point at the live context meter on screen.**

**Say:** "Context rot is real. Models get measurably dumber past ~50% context utilization. The fix is not bigger context — it's smaller, fresher context."

**The three zones:**
- **< 50% — Crisp.** Keep going.
- **50-70% — Compact soon.** Run `/compact` *with guidance* — tell it what to keep.
- **> 70% — Dumb zone.** `/compact` or `/clear` *now*. Don't push through it.

**Demo:**
```
$ /compact preserve the failing test names and the new schema fields, drop everything else
```

---

## 11 · Context pro (3 min)

**Say:** "Three pro moves:
1. **Compact at task boundaries**, not when the banner screams. Finish a sub-task → compact → start the next.
2. **Markdown in, not HTML/PDF/DOCX.** Convert before pasting; Claude tokenizes markdown 3-5x more efficiently.
3. **Use rewind (`/resume`).** A bad branch isn't a forced restart — it's a checkout."

---

## 12 · Session limits (3 min)

**Say:** "Every plan has weekly limits. Two principles:
1. **Watch `/cost`, not vibes.** Vibes lie; the meter doesn't.
2. **One task per session.** Long sessions are not productivity; they're context decay."

**Audience prompt:** "How many of you check `/cost` at end-of-session today? Now you all do."

---

## 13 · Subagents (6 min)

**Say:** "A subagent is an isolated Claude with its own context window, dispatched by your main session. It does the heavy lifting and reports back a small summary. Your main context stays clean."

**The killer pattern — research swarm:**
```
> dispatch 3 subagents in parallel:
  1. read every file in src/billing/ and summarise the data model
  2. read every test in tests/billing/ and list what's covered
  3. grep for TODO/FIXME in billing and rank by risk
  Then synthesise into a one-page assessment.
```

**Why it matters:** "One main session reading 200 files = context overflow. Three subagents reading ~70 files each = clean main context with three crisp summaries."

**Anchor:** "When in doubt, dispatch."

---

## 14 · Plugins (4 min)

**Say:** "A plugin is a packaged bundle of skills, commands, hooks, MCPs, and agents that you can install with one line. Like VS Code extensions, but for Claude."

**Show the plugin pill on screen:** superpowers, graphify, claude-mem.

**Demo:**
```
> /plugin install superpowers
```

---

## 15 · Plugins deep (3 min)

**Say:** "Three plugins worth knowing today:
- **superpowers** — opinionated workflow skills (debugging, brainstorming, TDD).
- **claude-mem** — persistent memory across sessions, indexed and searchable.
- **graphify** — code-review-graph for impact analysis without re-reading files."

**Anchor:** "Pick one this week. Don't install five — they compound context cost."

---

## 16 · MCPs (7 min — go deep)

**Say:** "MCP — Model Context Protocol — is how Claude talks to the world outside your filesystem. A server exposes tools (read this Linear ticket, query this database, click this button), and Claude can call them like any other tool."

**The four MCPs every team should run:**
| MCP | What it gives you |
| --- | ----------------- |
| `context7` | fresh docs for libraries (no more 2023-vintage snippets) |
| `github` | issues, PRs, reviews from the CLI |
| `playwright` | browser automation for E2E debugging |
| `figma` | design specs straight into the prompt |

**Demo — install one live:**
```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```
Then in claude:
```
> list my open PRs in this org and rank by review staleness
```

**Security note:** "MCPs run as you. Treat them like SSH keys. Audit before installing."

---

## 17 · Skills (6 min)

**Say:** "A skill is a markdown file with frontmatter that tells Claude *how* to approach a task. Unlike a custom command (which runs once), a skill is auto-invoked when the situation matches."

**Show the frontmatter pattern:**
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

**Demo:** drop that file in `~/.claude/skills/debugging.md`, then say in claude `> there's a bug in our checkout flow`. Watch it invoke the skill.

**Anchor:** "Commands are recipes you call. Skills are habits Claude picks up automatically."

---

## 18 · Token craft (4 min)

**The eight rules:**
1. **Cache everything stable** — system prompts, CLAUDE.md, large docs.
2. **Keep CLAUDE.md under 50 lines.** Anything longer, link to it.
3. **Compact at 50%, not 90%.**
4. **Match model to task** (Haiku/Sonnet/Opus).
5. **Subagents for heavy research.**
6. **One task per session.**
7. **Tune extended thinking** with `/effort` — don't pay for thought you don't need.
8. **Watch `/cost`, not vibes.**

**Anchor:** "Token discipline is the difference between a $200/mo bill and a $2,000/mo bill for the same output."

---

## 19 · Cost math (4 min)

**Say:** "Quick napkin math. Sonnet 4.6 is $3 in / $15 out per million tokens. A typical 1-hour session reads ~150K, writes ~30K — so roughly $0.45 in, $0.45 out, plus thinking. Call it ~$1.50/hour with extended thinking on. Opus is ~5x. Haiku is ~1/5."

**The honest formula:**
```
session $ ≈ (input_tokens × $in/M) + (output_tokens × $out/M) + (thinking_tokens × $out/M)
```

**Why teams overspend:** "Always Opus, no caching, no compaction, ten MCPs loaded but two used."

---

## 20 · Spec-driven (5 min)

**Walk the 5 numbered steps on screen:**
1. **Requirements** — write the *what* and the *why*.
2. **Design** — Claude proposes architecture; you approve.
3. **Task breakdown** — split into testable units.
4. **Implement** — one task at a time.
5. **Verify** — tests, types, lint, all green.

**Say:** "This is how seniors use Claude Code. Junior pattern: ‘build me a feature.' Senior pattern: ‘here is the spec, propose a design, break it down, implement task 1.' The senior pattern produces 3x cleaner diffs and 1/4 the rework."

---

## 21 · Case study (3 min)

**Pick the case from the page** (governance / cost / rollout) and walk it. Highlight the three guardrails: block destructive shell, require ticket ID, ship a shared org CLAUDE.md.

---

## 22 · By role (4 min)

**Say:** "Same tool, three jobs.
- **Engineer** — flow, quality, context. Parallel git worktrees, subagent research, custom `/onboard`.
- **QA** — coverage, flake, accessibility. User story → Playwright suite, a11y gate in CI, flake triage.
- **Lead/Manager** — governance, cost, rollout. Daily org spend dashboard, weekly productivity digest."

---

## 23 · What's new (3 min)

**Scroll the timeline.** Highlight the last three entries: Live Artifacts, Claude Design, Managed Agents. Tell them which to try this week.

---

## 24 · Resources + close (2 min)

**Final line:** "You now have a teammate who can touch your keyboard. The work between today and being productive with this is — ten lines in CLAUDE.md, two hooks, one custom command, one MCP. Do that this week."

**Show:** the resources tab — docs, GitHub, Discord, this repo.

---

## Q&A (10 min)

**Anticipated questions + crisp answers:**

- **"Is my code sent to Anthropic?"** Yes, prompts are sent to the API. Claude Code respects `.claude/ignore` and never sends gitignored files unless you explicitly attach them.
- **"How does it compare to Copilot/Cursor?"** Copilot is autocomplete; Cursor is autocomplete + chat in IDE; Claude Code is an agent that runs commands. Different category.
- **"What about offline?"** No. Local models via MCP are an option but not the default.
- **"Does it edit my git history?"** It can commit, branch, push — only when you let it. Default is propose, you approve.
- **"How do I roll this out across a team?"** Org CLAUDE.md, shared `.claude/` in repo, hooks for guardrails, dashboard for cost. See section 21.

---

## Presenter checklist

- [ ] Sample repo cloned locally with at least 50 files
- [ ] `~/.claude/CLAUDE.md` exists and is < 50 lines
- [ ] One MCP installed and demo-able (github or context7)
- [ ] One custom command in `.claude/commands/`
- [ ] One PreToolUse hook that visibly blocks something
- [ ] Browser zoom set so back-row can read the terminal
- [ ] `/cost` run before session so the meter starts at zero
- [ ] Network: tutorial site bookmarked + cached
