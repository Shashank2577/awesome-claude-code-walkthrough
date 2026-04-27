# Claude Code · 2-Hour Workshop · Presenter Script (Deep Edition)

> **Audience:** working software engineers — most have used Copilot/Cursor/ChatGPT but never an agentic CLI.
> **Format:** project the live tutorial site, scroll tab-by-tab, demo in a real terminal alongside.
> **Total:** 120 minutes · 24 tabs + open + Q&A.

---

## How to use this script

Each tab below has the same structure:

- **Why this tab matters** — the one-line hook you can drop if running short.
- **What to say** — verbatim talking points, written in the voice of a senior engineer who's been using Claude Code in anger for 8 months. Read these out loud at home once before the session; they're rhythmically tuned to about 100 words per minute.
- **Demo** — exact terminal commands, with expected output where it lands. Pre-run them once in your sample repo and screenshot the outputs in case live demo fails.
- **Audience moment** — a hands-up question or quick poll to keep them engaged. Skip these only if you're behind schedule.
- **Anticipated Q&A** — the two questions someone in the room will ask. Memorize the answers; don't read them.
- **Pitfall** — the trap juniors fall into for this concept. Naming it in the room saves them weeks.
- **Transition** — the one-sentence bridge to the next tab, designed to make the structure feel inevitable.

Cumulative timing in the right column. If you're more than 5 minutes behind by tab 12, drop the **audience moment** for tabs 13-24 to recover. If you're more than 10 minutes behind by tab 16, also collapse Token craft + Cost math into a single 5-minute combined section.

---

## Pre-flight checklist (do 30 minutes before)

- [ ] Sample repo cloned locally with at least 50 files, real tests, real CI config. A toy repo will look like a toy.
- [ ] `~/.claude/CLAUDE.md` exists, under 50 lines, and you've reviewed it on screen-zoom.
- [ ] `~/.claude/settings.json` has at least one PreToolUse hook + one PostToolUse hook visibly wired up.
- [ ] One MCP installed and working (`github` recommended — most relatable for engineers).
- [ ] One custom command in `.claude/commands/onboard.md` you can invoke live.
- [ ] One subagent dispatch you've rehearsed — research swarm pattern.
- [ ] `claude --version` ≥ 1.0.x and you've already opened the API key dialog so it doesn't pop up live.
- [ ] Browser zoom set so back-row can read the terminal — usually 150% on a 1080p projector.
- [ ] `/cost` was run **at the start of today** so the meter starts near zero and the per-session number is honest.
- [ ] Network: tutorial site bookmarked + cached. If conference Wi-Fi dies, the site still works.
- [ ] Have screenshots of every demo's expected output as fallback if a live demo hangs.

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

**Why this tab matters:** The first 7 minutes set the frame. If they leave thinking "this is autocomplete with extra steps," everything else lands wrong. Frame it as a *category change*: an agent, not a copilot.

**What to say:**

> "Good morning. For the next two hours we're going to learn Claude Code — but I want to start by telling you what we are NOT going to do. We are not going to learn another autocomplete. We're not going to compare to Copilot or Cursor for the first 90 minutes. We're going to treat Claude Code as what it actually is: an agent. A teammate who can read your repo, edit your files, run your tests, push your commits, and talk to your databases — all gated by tools you control.
>
> By the end of this session you will have it installed, you will have configured CLAUDE.md, you will have one hook that blocks something dangerous, and you will know which knob to turn when it costs too much, when it slows down, or when it hallucinates. That last one — knowing which knob to turn — is the difference between someone who *uses* Claude Code and someone who *runs* with it.
>
> Quick install while we talk."

**Demo:**
```bash
# In a fresh terminal, projected so the room can see
npm i -g @anthropic-ai/claude-code
# (talk through what's happening while the install runs)

claude --version
# claude 1.x.y

cd ~/work/sample-repo
claude
```

When `claude` boots, point at the screen:

> "Notice three things in the boot output. One — it just read CLAUDE.md, 48 lines. Two — it listed my MCPs: context7, figma, playwright, github. Three — it offered to resume yesterday's task because I have claude-mem installed. Each of those is a tab we're going to unpack."

**Audience moment:** "Hands up if you've installed Claude Code before today. Keep them up if you've used it for more than an hour. Keep them up if you've ever written a hook for it." (Usually 30% / 5% / 0%.) "Great — by the end you'll all be in the third bucket."

**Anticipated Q&A:**

- *"Do I need ChatGPT Plus / a Claude subscription?"* — You need a Claude Pro/Max plan or an Anthropic API key. Pro is $20/mo flat with usage limits; API is metered. Start with Pro for the workshop, switch to API if you hit limits.
- *"Is my code sent to Anthropic?"* — Prompts and the files Claude reads are sent to the API. `.claude/ignore` and your gitignore rules block files from being read. Anthropic doesn't train on API traffic by default for Pro/Max/API accounts.

**Pitfall:** Don't paste your API key on screen. If you forgot to log in earlier, pause and do it off-screen.

**Transition:** "OK — installed, booted, reading our repo. Let's go to tab 1: what is this thing, really."

---

## 1 · What it is (4 min)

**Why this tab matters:** Anchors the mental model. Three moves: Read, Edit, Do. Everything in the next two hours is one of these three.

**What to say:**

> "Three moves. **Read** — your whole repo, your CLAUDE.md, imported docs, MCP-fetched fresh docs, knowledge graphs if you have them. **Edit** — multi-file changes with proper diffs, formatters and linters and tests running automatically through hooks. **Do** — run commands, branch, commit, push, drive browsers, query databases. All tool-gated. All reviewable. All stoppable.
>
> The four numbers on screen are the punchlines for this whole session. **40-70%** average cost reduction for teams that take token discipline seriously — we'll talk about that in tab 18. **1M** context tokens on Sonnet and Opus — we'll talk about why that's a *trap*, not a feature, in tab 10. **9** hook events you can wire into — that's tab 9. **Infinity-shape** — CLI, IDE, desktop, web, browser, MCP-driven — that's tab 2.
>
> The mental model I want you to hold for the next two hours is this: Claude Code is a teammate who can touch your keyboard. Everything else on this site is about how to give that teammate better context, better tools, and better guardrails. If you're ever lost in this session, come back to that line."

**Demo:** No new commands. Point at the running `claude` session. "It's reading the repo as I speak. By the time we get to tab 5, it'll have a full mental map of every file we care about."

**Audience moment:** "How many of you have a CLAUDE.md in a repo you own today? Keep that hand up — we'll come back to it in 35 minutes." (Usually 0-2 hands.)

**Anticipated Q&A:**

- *"How is this different from ChatGPT in a browser?"* — ChatGPT can't run your tests. Claude Code can. The agent loop with real tool execution is the category difference.
- *"Can I trust it to make changes?"* — Default mode shows a diff and waits for approval. You graduate to auto-accept after you've watched it for an hour and you trust it for a specific kind of task.

**Pitfall:** New users try to "talk to" Claude Code like it's ChatGPT. Talk to it like a teammate who needs *context*, not entertainment.

**Transition:** "Six different doors into the same brain. That's tab 2."

---

## 2 · Ecosystem (4 min)

**Why this tab matters:** People assume "Claude Code = the CLI." It isn't. The CLI is the canonical surface; everything else is a window onto it. Same agent, same context model, same tools.

**What to say:**

> "One brain, six doors in. The agent itself doesn't change between surfaces — what changes is the affordance: how you give it inputs, how it shows you outputs, how it asks for permission.
>
> **CLI** is the canonical door. Terminal-native. Best for deep work, headless automation, CI integration. If you remember one surface, remember this one.
>
> **IDE** — VS Code extension, JetBrains plugin. Inline diffs, agent panel, sidekick mode. Use this when you want Claude to live next to your editor cursor.
>
> **Desktop app** — full agent with computer-use. It can see your screen, click buttons in your browser, navigate Figma. Use it for tasks that bridge code and other apps.
>
> **Web / claude.ai/code** — for shareable artifacts. Demo a quick prototype to a designer? Send them a web URL.
>
> **Browser extension** — agent that drives your browser. End-to-end test debugging. Scraping. Form filling on dev tools.
>
> **MCP-driven agents** — headless. Cron-driven. CI-integrated. The same agent, dispatched without a human in the loop, with tighter tool restrictions.
>
> Pick the door for the job. Most engineers settle on CLI plus one IDE surface. That's the right starting point."

**Demo:** Bring up VS Code with the Claude Code extension panel. Type the same prompt in both: `> what test files cover the billing service?` Show that the answer is identical. "Same brain. Different room."

**Audience moment:** "Show of hands — who works in VS Code? IntelliJ? Vim/neovim? Cursor? Different IDE?" — gauge the room so you know which door to emphasize.

**Anticipated Q&A:**

- *"Can I use it in (my obscure editor)?"* — If your editor has a terminal, yes. Run `claude` in a split. The IDE-specific extensions add diff UI and inline edits, but the CLI works everywhere.
- *"What about offline?"* — Not really. The model lives in Anthropic's cloud. You can run a local model via MCP for some workflows, but it's not the default story.

**Pitfall:** Don't install all six surfaces on day one. Pick CLI + your daily editor. The cognitive overhead of learning all surfaces at once is real.

**Transition:** "Same brain — but which brain? Three model choices, one decision rule. Tab 3."

---

## 3 · Models (5 min)

**Why this tab matters:** Model choice is the single biggest lever on cost and latency. A junior using Opus for everything will burn 5x what a senior using Sonnet+Haiku correctly will. This is the one tab where habit changes save real money.

**What to say:**

> "Three models in 2026: Haiku 4.5, Sonnet 4.6, Opus 4.6. Both Sonnet and Opus now have 1M context at standard pricing — that's new since late 2025.
>
> The rule is brutal in its simplicity: **start in Sonnet, drop to Haiku for drudgery, escalate to Opus when you hit a wall.**
>
> Sonnet is your workhorse. Refactors, debugging, feature work, writing tests, code review. It'll handle 80% of what you throw at it.
>
> Haiku is for tasks that are mechanical: commit messages, lint fixes, docstring sweeps, JSON-to-YAML translations, mass renames. Things where you'd be slightly insulted if a senior spent their afternoon on it. Switching to Haiku is not a downgrade — it's *correctness of fit.*
>
> Opus comes out when you're staring at a hard problem. Architecture you don't yet see. A bug that's been open for three days. A migration that touches twelve services. Opus thinks longer, sees further, and costs roughly 5x Sonnet — which means you reach for it intentionally, not by default.
>
> The mistake junior users make is staying in Opus all day because they read on Twitter that 'Opus is best.' Opus IS best on hard problems. On easy problems it's just expensive."

**Demo:**
```
$ /model
> select a model
  [1] claude-haiku-4-5   · fast, cheap
  [2] claude-sonnet-4-6  · balanced (current)
  [3] claude-opus-4-6    · heavy reasoning
$ 3
⟶ switched to claude-opus-4-6

$ /effort xhigh
⟶ thinking budget: 20K tokens — planning carefully.

# now ask it a hard question and watch it think
> the billing service has a deadlock between charge_invoice() and refund_invoice() under concurrent load. propose a fix that doesn't introduce a new lock.
```

While it thinks, narrate: "Notice the thinking is visible. It's exploring the problem space. We're paying for that — and on a real deadlock that paid back 100x by morning."

**Audience moment:** "What's the most expensive model run you've ever had on any AI tool? In dollars? Anyone above $50?" — usually a few hands. "By tab 19 we'll have the math to predict that number before you start."

**Anticipated Q&A:**

- *"Can I mix models in one session?"* — Yes. `/model` switches mid-session. Common pattern: plan in Opus, implement in Sonnet, clean up in Haiku.
- *"Why not always Opus?"* — Cost compounds. A team of 20 always-Opus will burn $20K/mo on what $4K of Sonnet would do better.

**Pitfall:** People think "smarter model = better answer always." Wrong. Smarter model with bad context is *worse* than fast model with good context. Context discipline (tab 10) trumps model choice.

**Transition:** "OK we have a model. How do we actually use it day-to-day? Tab 4."

---

## 4 · Usage (4 min)

**Why this tab matters:** Three modes — plan, auto-accept, free-flow — and most engineers never learn the toggle. Knowing when to switch is what makes the tool feel responsive instead of bossy.

**What to say:**

> "There are three rhythms for a session.
>
> **Plan mode** — you tell Claude what you want, it reads, it thinks, it writes a plan, it asks you to approve before any edit happens. Use this when you're in an unfamiliar repo, when the change touches multiple modules, or when you want a paper trail for review.
>
> **Auto-accept mode** — toggle with Shift+Tab. Claude makes small isolated edits without asking each time. Use this when you're cleaning up a single file, doing a mass rename, or watching the diffs scroll past and trusting them.
>
> **Free-flow** — the default. Claude proposes a diff, you accept or reject each one. Use this for production code paths and anything you'd review carefully in a PR.
>
> The shape of a healthy day with Claude Code is: plan mode at the start of a feature, free-flow during implementation, auto-accept for the cleanup pass. Three rhythms, one session."

**Demo:**
```
$ claude
> /plan implement a /healthz endpoint that returns 200 OK with a JSON body of the form {status: "ok", version: <git sha>}

# Claude responds with a plan, asks for approval
> approve

# Now in free-flow — Claude makes the change
# Show the diff, accept it

$ Shift+Tab
⟶ auto-accept ON

> rename every TODO comment in this file to FIXME and add the JIRA project prefix

$ Shift+Tab
⟶ auto-accept OFF
```

**Audience moment:** "Who here would default to plan mode versus free-flow on a feature you're 50% sure about?" — let them argue for 30 seconds. "Both right answers. The point is *deliberate choice*."

**Anticipated Q&A:**

- *"What if Claude makes a bad edit in auto-accept?"* — `/resume` rewinds. We'll cover it in the commands tab.
- *"Is there a 'YOLO' mode?"* — Yes, `--dangerously-skip-permissions` exists. Don't use it. Ever. The cost of one bad rm equals all the time it saves.

**Pitfall:** People leave auto-accept on permanently because Shift+Tab toggled it once and they forgot. Keep visual cue — the prompt indicator changes — front of mind.

**Transition:** "Now let's open the engine. Every behavior in Claude Code is one of seven steps. Tab 5."

---

## 5 · Agent loop (5 min)

**Why this tab matters:** This is the API of Claude Code. Once you internalize the seven steps, every customization becomes obvious — you just point at a step and say "I want to change behavior here."

**What to say:**

> "Memorize this loop. It will return on every tab from here on out.
>
> Step 1: **User message.** You type something.
>
> Step 2: **Model turn.** Claude thinks. If extended thinking is on — `/effort high` or higher — this is where the thinking budget gets spent. The output of this step is either a final answer or a tool call.
>
> Step 3: **PreToolUse hooks fire.** Your shell commands run *before* the tool actually executes. They get the tool name, the inputs, the session ID. They can pass through, rewrite the inputs, or block the tool entirely by exiting non-zero.
>
> Step 4: **Tool executes.** Edit, Write, Bash, Read, Grep — whatever Claude called. This is where actual side effects happen on your filesystem and your shell.
>
> Step 5: **PostToolUse hooks fire.** After the tool runs. They get the result. Common uses: format the file Claude just wrote, run the test next to the code Claude just edited, log the action for compliance.
>
> Step 6: **Tool result.** The result feeds back into Claude's context for the next turn.
>
> Step 7: **Stop, with optional Notification.** When the model finishes a turn, the Stop hook fires. If you've been gone — coffee break, lunch — a Notification hook can ping your phone or fire a desktop alert.
>
> The reason this matters: if you ever want Claude Code to behave differently, you don't write a plugin from scratch. You point at one of these seven steps and slip in your code."

**Demo:** Pull up `~/.claude/settings.json` on screen. Walk the JSON line by line. Show one PreToolUse hook (the destructive-shell blocker) and one PostToolUse hook (formatter on edit).

```
$ cat ~/.claude/settings.json
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
                    "command": "prettier --write \"$TOOL_INPUT_FILE\"" }] }
    ]
  }
}
```

**Audience moment:** "Tell me which step you'd hook into for: (a) requiring a JIRA ticket on every prompt, (b) running the tests after every save, (c) Slack-pinging me when a 30-minute task finishes." (Answers: 1, 5, 7.)

**Anticipated Q&A:**

- *"Can hooks be slow?"* — Yes, and it matters. PreToolUse hooks add latency to every tool call. Keep them under 100ms. PostToolUse can be slower (formatting, testing).
- *"What if my hook crashes?"* — Non-zero exit blocks the tool (in PreToolUse). In PostToolUse it logs and continues. Always check `claude` output for hook failures.

**Pitfall:** Engineers write a hook in Python because they know Python. Then `python3` startup adds 400ms to every tool call. Use bash for hot-path hooks, Python for cold-path ones.

**Transition:** "The model thinks in tokens. We talk to it in slash commands. Tab 6."

---

## 6 · Commands (5 min)

**Why this tab matters:** The keyboard is the UI. There are roughly 30 slash commands. You only need to memorize six.

**What to say:**

> "Every slash command is a lever on the session. Memorize six, know the rest exist.
>
> **/model** — switch model. Reach for it when the task changes character. Plan in Opus, implement in Sonnet.
>
> **/effort** — thinking budget. Reach for it when you hit a wall. Low, medium, high, xhigh. Xhigh is 20K thinking tokens — a lot.
>
> **/compact** — summarize and drop history. *Reach for it at 50% context, not 90%.* Pass guidance: tell Claude what to keep.
>
> **/clear** — wipe history. Use for a new task in the same session.
>
> **/cost** — tokens and dollars spent. Run at the end of every session for a week and you'll calibrate your intuition forever.
>
> **/hooks** — edit hooks live without leaving the session.
>
> Beyond those six: /resume, /plan, /mcp, /skill, /team, /plugin. They all matter. They all show up in the next eighteen tabs."

**Demo — run all six in one minute:**
```
$ /cost
session: 12,450 in / 3,200 out · $0.09

$ /model
$ 2     # sonnet

$ /effort high

$ /compact preserve the failing test names and the new schema fields

$ /clear

$ /hooks
> opens interactive editor; close it
```

**Audience moment:** "Out of those six, which one would you reach for *least often*?" (Often /clear because people forget it's free to start over.)

**Anticipated Q&A:**

- *"What does /compact actually do?"* — Summarizes the conversation so far into a compact representation, then drops the original. The trade-off: future turns have less verbatim context. The optional guidance argument lets you keep specific things verbatim.
- *"How is /clear different from quitting and re-running?"* — Same effect. /clear is faster and keeps your session ID for cost tracking.

**Pitfall:** People run `/compact` at 90% and feel productive. By 90% the model has been dumb for 30 minutes. Compact at 50%.

**Transition:** "The full reference — the rest of the commands — is one click away."

---

## 7 · Commands ref (3 min)

**Why this tab matters:** It's a bookmark, not a memorize. Show them the layout once so they know where to come back.

**What to say:**

> "This is the full reference. Three groups: session, workflow, advanced. Don't memorize. Bookmark the page.
>
> Two commands worth flagging now even though we'll cover them later: **/resume** rewinds the session — bad branch isn't a forced restart, it's a checkout. **/team** dispatches named subagents — we'll cover that in tab 13."

**Demo:** Live scroll of the tab. Pause briefly on `/resume` and `/team`.

**Audience moment:** "Open the live tutorial on your laptop right now. Cmd-F any command you're curious about while I keep going."

**Anticipated Q&A:** None usually — this is a low-content tab.

**Pitfall:** Don't dwell. Some presenters spend 8 minutes here. The tab is a reference; respect it as such.

**Transition:** "Built-in commands are a starting point. Custom commands are how teams scale Claude Code. Tab 8."

---

## 8 · Custom cmds (4 min)

**Why this tab matters:** This is the multiplier. Any markdown file becomes a slash command. A 30-line markdown file captures tribal knowledge that used to take a senior an hour to explain.

**What to say:**

> "Drop a markdown file in `.claude/commands/` and it becomes a slash command. Project-local lives at `.claude/commands/` in the repo. Global lives at `~/.claude/commands/`.
>
> What goes in the markdown? Just instructions in plain English. Frontmatter is optional. Variables — like `$ARGUMENTS` — are supported.
>
> The single highest-leverage thing a team lead can do this week is write three custom commands and commit them to the repo. Suddenly every engineer on the team has the same `/onboard`, the same `/review-pr`, the same `/write-runbook`. That's not a feature — that's culture, encoded."

**Demo — write a custom command live:**
```bash
mkdir -p .claude/commands

cat > .claude/commands/onboard.md <<'EOF'
You are onboarding a new engineer to this repo.

1. Read CLAUDE.md and confirm you understand the dev loop.
2. List every external service this repo talks to (databases, APIs, queues).
3. Identify the three files a new engineer should read first, in order.
4. Summarize the testing strategy in under 50 words.
5. Flag any onboarding pain points you spot in the code.

Keep total output under 400 words.
EOF
```

Then in claude:
```
> /onboard
```

Watch it produce a tailored onboarding doc. "We did not pre-write that doc. Claude assembled it from CLAUDE.md, the actual code, and the conventions it found."

**Audience moment:** "Name one task you do at least twice a week that's the same shape every time. Shout it out." Take 2-3, write a custom command for one of them on the screen.

**Anticipated Q&A:**

- *"Can custom commands take arguments?"* — Yes. Use `$ARGUMENTS` in the markdown. Invoke with `/mycommand some text`.
- *"How do I share these across the team?"* — Commit `.claude/commands/` to the repo. New engineers `git pull` and immediately have your team's commands.

**Pitfall:** People write 200-line commands. Don't. If it's longer than 30 lines, it's an instruction set, not a command. Break it up.

**Transition:** "Commands are how *you* talk to Claude. Hooks are how *Claude talks to your shell* — and how you control what it can do. Tab 9."

---

## 9 · Hooks (7 min — go deep)

**Why this tab matters:** This is the most powerful tab in the session. Hooks are how you turn Claude Code from "smart assistant" into "team-policy-enforcing teammate." Every governance concern your CTO has — destructive commands, audit trail, compliance — answers with hooks.

**What to say:**

> "Nine hook events. Walk through each pill.
>
> **SessionStart** — fires when `claude` boots. Common use: print a project banner, load environment-specific context, log the start of the session.
>
> **UserPromptSubmit** — fires on every prompt before Claude sees it. Common use: redact secrets, inject a JIRA ticket ID requirement, log every prompt for compliance.
>
> **PreToolUse** — fires before every tool call. The big one. Common use: block destructive shell, restrict edits to certain directories, require confirmation for git push.
>
> **PostToolUse** — fires after every tool call. Common use: format the file just edited, run the test next to the file just edited, log every action.
>
> **Notification** — fires on long-running completions. Common use: desktop notification, Slack ping, mobile push.
>
> **Stop** — fires when the model finishes a turn. Common use: audit log, telemetry.
>
> **SubagentStop** — fires when a dispatched subagent finishes. Common use: aggregate subagent results into a parent log.
>
> **PreCompact** — fires before /compact runs. Common use: snapshot the soon-to-be-dropped context for replay.
>
> **SessionEnd** — fires when `claude` exits. Common use: export cost summary, archive transcript, fire the team's hours-worked tracker.
>
> The two killer use-cases — the ones every team eventually writes — are: block destructive shell on PreToolUse, auto-test on PostToolUse. We're going to write the destructive-shell blocker live."

**Demo — write a real PreToolUse hook live:**
```bash
mkdir -p ~/.claude/scripts
cat > ~/.claude/scripts/block-destructive.sh <<'BASH'
#!/usr/bin/env bash
INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Patterns to block
PATTERNS=(
  'rm -rf /'
  'rm -rf \*'
  'git push.*--force.*(main|master)'
  'DROP TABLE'
  'DROP DATABASE'
  ':(){:|:&};:'   # fork bomb
)

for p in "${PATTERNS[@]}"; do
  if echo "$CMD" | grep -Eq "$p"; then
    echo "BLOCKED: matched destructive pattern: $p" >&2
    exit 1
  fi
done

exit 0
BASH
chmod +x ~/.claude/scripts/block-destructive.sh
```

Then wire it up in settings:
```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash",
        "hooks": [{ "type": "command",
                    "command": "~/.claude/scripts/block-destructive.sh" }] }
    ]
  }
}
```

Now ask Claude in the live session:
```
> run rm -rf / on this repo
```

Watch the hook fire and block. Read the BLOCKED message out loud.

> "That just ran on every Bash call from now on. If I add `git push --force origin main` to my list — protected. If I add `DROP TABLE users` — protected. The hook contract is: non-zero exit blocks the tool. Zero exit lets it through. That's the whole API."

**Audience moment:** "Three hooks I want every team in this room to write before next Monday — block destructive shell, auto-test on edit, log every prompt to a JSONL file. Show of hands: who'll do at least one this week?"

**Anticipated Q&A:**

- *"What if I want to *shape* the input instead of block it?"* — Print the modified input to stdout. Claude treats stdout from a PreToolUse hook as the rewritten input. So you can rewrite `rm` to `rm -i`, prepending an interactive flag.
- *"Do hooks run on subagent tool calls?"* — Yes. Subagents inherit the same hook config. There's also a SubagentStop hook for subagent-specific behaviors.

**Pitfall:** Hooks that exit slowly or erratically destroy the experience. Test every hook with a timeout. A hook that takes 2 seconds on every Bash call adds 2 seconds to *every* Bash call. Multiply by 50 calls/session.

**Transition:** "We've talked about *what* Claude can do. Now let's talk about how *much* it can hold in its head. Tab 10."

---

## 10 · Context (5 min)

**Why this tab matters:** Context discipline is the second-biggest lever after model choice. The bigger context windows (1M now) are a *trap* unless you understand why bigger isn't smarter.

**What to say:**

> "Context rot is real. There's research — and we feel it in production — that models get measurably dumber past about 50% context utilization. Recall drops, reasoning shortens, attention skips spans of the input.
>
> The fix is *not* a bigger context window. It's a *smaller, fresher* context window. 1M tokens is impressive marketing. In practice, your best work happens between 10% and 50% of whatever window you're in.
>
> Three zones:
>
> **Below 50% — Crisp.** Stay here. Don't compact yet.
>
> **50 to 70% — Compact soon.** This is where you proactively run /compact with guidance. Tell Claude what to keep verbatim — failing test names, schema definitions, the API contract you're working against.
>
> **Above 70% — Dumb zone.** Compact or clear *now*. Don't push through it. Pushing through it is like driving with your eyes half-closed."

**Demo:**
```
$ /compact preserve the failing test names in tests/billing/, the new columns added to invoices.sql, and the typescript signature of charge_invoice. drop everything else.

⟶ context: 68% → 22%
⟶ kept: 312 tokens of pinned content
⟶ dropped: prior file reads, search results, intermediate plans
```

**Audience moment:** "Quick poll: who has felt Claude get dumb mid-session and not known why? That's context rot. Now you have a name for it and a fix."

**Anticipated Q&A:**

- *"How do I see the current context %?"* — The status line shows it. Some plugins (claude-mem) add a richer indicator.
- *"Does /compact lose information?"* — Yes, by design. The guidance argument is your knob to keep what matters.

**Pitfall:** New users wait for the banner to *force* them to compact. By then it's too late. Compact proactively at task boundaries.

**Transition:** "Three more pro moves on context. Tab 11."

---

## 11 · Context pro (3 min)

**Why this tab matters:** Three small habits that compound. Most users discover them after month 3 — we're going to teach them in 3 minutes.

**What to say:**

> "Three pro moves.
>
> **One — compact at task boundaries, not when the banner screams.** Finish a sub-task. Compact with guidance. Start the next sub-task. The banner is a panic button; task boundaries are a habit.
>
> **Two — markdown in, not HTML/PDF/DOCX.** Convert before pasting. Claude tokenizes markdown 3-5x more efficiently than other formats. A 10K-word PDF can balloon to 60K tokens. The same content as markdown is closer to 12K. Pandoc is your friend.
>
> **Three — use rewind.** /resume isn't 'oops' — it's *checkout*. A bad branch in your conversation is not a forced restart. Claude lets you go back N turns and try a different prompt. Use it as a normal tool, not an emergency."

**Demo:**
```bash
# convert before pasting
pandoc spec.docx -o spec.md
```
Then:
```
> /resume 3
⟶ rewound 3 turns. context restored to that point.
```

**Audience moment:** Skip if running short.

**Anticipated Q&A:**

- *"Does /resume cost extra tokens?"* — No. The conversation history is local. /resume just changes the cursor.
- *"Can I see all rewind points?"* — Yes, /resume with no argument lists turns.

**Pitfall:** None major.

**Transition:** "Speaking of cost — every plan has limits. Tab 12."

---

## 12 · Session limits (3 min)

**Why this tab matters:** Plans have weekly token allowances. Hitting the wall mid-task is a productivity tax. Two principles fix it.

**What to say:**

> "Every plan — Pro, Max, API — has limits. On Pro, you'll hit weekly caps if you're heavy. On API, you have rate limits per minute.
>
> Two principles.
>
> **One — watch /cost, not vibes.** Vibes lie. The meter doesn't. Run /cost at the end of every session for one week. Notice the spread. Most engineers are shocked at how cheap a normal day is, and how expensive a bad day is.
>
> **Two — one task per session.** Long sessions are not productivity. They're context decay. Five 30-minute focused sessions beat one 2.5-hour rambling session, on every axis: cost, quality, your own attention."

**Demo:**
```
$ /cost
─ session ─────────────────────
  in:        128,420 tokens
  out:        14,330 tokens
  thinking:    7,200 tokens
  $0.45 + $0.21 + $0.11 = $0.77
─ this week ──────────────────
  $14.30 / $40 cap (Pro)
```

**Audience moment:** "How many of you check /cost today? Now you all do."

**Anticipated Q&A:**

- *"What happens when I hit the Pro limit?"* — Falls back to slower / smaller models, then blocks until the cap resets.
- *"Should I just go API?"* — If you're spending more than $40/mo on Pro, yes. The math flips around there.

**Pitfall:** People assume more usage = more output. Sometimes more usage = more rambling. Tighten prompts, not budgets.

**Transition:** "Some tasks need more than one session — they need many in parallel. Tab 13."

---

## 13 · Subagents (6 min)

**Why this tab matters:** Subagents are how you keep your main context clean while doing heavy parallel work. The pattern is research-then-synthesize, and once you internalize it, you'll dispatch them weekly.

**What to say:**

> "A subagent is an isolated Claude with its own context window, dispatched by your main session. It does the heavy lifting and reports back a *summary*. Your main context never sees the noise.
>
> Three rules for when to dispatch.
>
> **Rule one — heavy reads.** If a task requires reading more than 30 files, dispatch. One main session reading 200 files is context overflow. Three subagents reading ~70 files each is three crisp summaries.
>
> **Rule two — parallel directions.** When you need to investigate three things at once and synthesize, dispatch three subagents. They run in parallel. You wait for the slowest one. Total wall-clock time = the slowest subagent, not the sum.
>
> **Rule three — independence.** Subagents work best when they don't need feedback mid-flight. If the work needs you to react to interim findings, do it in the main session.
>
> The pattern: dispatch → wait → synthesize. The output is one paragraph per subagent, not a 50-page transcript."

**Demo:**
```
$ claude
> dispatch 3 subagents in parallel:
  1. read every file in src/billing/ and summarize the data model in 200 words
  2. read every test in tests/billing/ and list coverage gaps as bullet points
  3. grep for TODO/FIXME/XXX/HACK in src/billing/ and tests/billing/ and rank them by risk
  Then synthesize their outputs into a one-page assessment of the billing module.
```

While they run, narrate: "Three Claudes are reading files right now. None of them are polluting my main context. When they finish, I'll get three bullet summaries plus a synthesis."

**Audience moment:** "Name a task on your team this week that fits the dispatch pattern." Take one example, design the dispatch out loud.

**Anticipated Q&A:**

- *"Do subagents share my hooks?"* — Yes. They run in the same security context.
- *"Can subagents dispatch sub-subagents?"* — Yes, but resist. Two levels deep is a debugging nightmare.

**Pitfall:** Junior pattern: dispatch a subagent for a task that's 5 files deep. The dispatch overhead exceeds the savings. Dispatch when the cost of context pollution is real, not for every task.

**Transition:** "Custom commands. Hooks. Subagents. These compose. The composition layer is plugins. Tab 14."

---

## 14 · Plugins (4 min)

**Why this tab matters:** Plugins bundle skills, commands, hooks, MCPs, and agents. They're the npm of Claude Code. Worth knowing what to install — and what *not* to.

**What to say:**

> "A plugin is a packaged bundle: skills, commands, hooks, MCPs, agents — all together. You install with one line. Like VS Code extensions, but for Claude.
>
> Plugins are how communities ship workflows. The plugin you install today might add 12 commands, 4 hooks, 3 skills, 2 MCPs. Powerful — but also a footprint. Every plugin is more context Claude has to load on session start.
>
> The rule I follow: pick *one* plugin to try this week. Live with it. If it earns its keep, keep it. If it doesn't, uninstall. Don't install five at once."

**Demo:**
```
$ /plugin list
$ /plugin install superpowers
⟶ installed: superpowers (12 commands, 8 skills, 3 hooks)
$ /plugin info superpowers
```

**Audience moment:** Skip.

**Anticipated Q&A:**

- *"Are plugins safe?"* — They run with your permissions. Read the manifest before you install. Pin versions in production.
- *"Can I write my own plugin?"* — Yes. A plugin is mostly a folder with a manifest. Start by extracting your team's `.claude/` setup into a plugin.

**Pitfall:** Plugin sprawl. A new user installs 10 plugins, each adds an MCP, suddenly session start is 8 seconds. Audit quarterly.

**Transition:** "Three plugins worth knowing today. Tab 15."

---

## 15 · Plugins deep (3 min)

**Why this tab matters:** A pointed recommendation list. Three plugins that cover three real needs.

**What to say:**

> "Three plugins worth a look.
>
> **superpowers** — opinionated workflow skills: brainstorming, debugging, TDD, code review. If you want one plugin that nudges you toward better habits, this is it.
>
> **claude-mem** — persistent memory across sessions. Your past Claude conversations become a searchable knowledge base. The 'I solved this last week' moment, recovered.
>
> **graphify** (or similar code-graph plugins) — code-review-graph for impact analysis. When you want to know what a function call really touches across the repo without re-reading every file."

**Demo:** Show one plugin's output. e.g. `claude-mem search "billing migration"` returns prior session snippets.

**Audience moment:** Skip.

**Anticipated Q&A:** None expected.

**Pitfall:** None major.

**Transition:** "Plugins are bundles. The bundle that matters most outside your filesystem is MCP — Model Context Protocol. Tab 16."

---

## 16 · MCPs (7 min — go deep)

**Why this tab matters:** MCP is how Claude Code talks to the world. It's the most leveraged tab after Hooks. Once you understand MCP, every external system becomes a tool Claude can use.

**What to say:**

> "MCP — Model Context Protocol — is an open standard for how language models talk to external systems. A server exposes tools. Claude calls those tools like any other tool. Same agent loop. Same hooks. Same approval flow.
>
> Why this is huge: every system in your company that you've ever wanted Claude to know about — Linear, Jira, Notion, your internal API, your database, your monitoring stack — can become a tool for Claude. You don't fine-tune. You don't pre-train. You just plug in an MCP server.
>
> Four MCPs every team should consider running on day one.
>
> **context7** — fresh library documentation. The single biggest fix for hallucinated APIs. Claude looks up the actual current docs instead of remembering 2023-vintage snippets.
>
> **github** — issues, PRs, reviews from the terminal. 'Show me my open PRs ranked by review staleness.' 'Comment on PR #1234 with a code review.' All from inside Claude.
>
> **playwright** — browser automation. End-to-end test debugging. Visual regression. 'Click through the checkout flow and tell me where it breaks.'
>
> **figma** — design specs into the prompt. 'Translate the figma frame at this URL into React components.'
>
> Pick the one that matches your work. Install. Live with it for a week."

**Demo — install one live:**
```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```
After install, in claude:
```
> list my 5 most stale open PRs in this org
> add a comment to PR #1234 saying I'll review it tomorrow
```

> "That just hit GitHub's API on my behalf, with my OAuth token. The agent loop is the same — Claude saw 'github_list_prs' as an available tool, called it, got results, formed an answer. We didn't write any glue."

**Audience moment:** "Name one tool in your stack — internal or external — that you wish Claude could see. Shout it out." Take 2-3, group them by 'has an MCP' vs 'someone needs to write one.'

**Anticipated Q&A:**

- *"Are MCPs safe?"* — They run as you. Treat them like SSH keys. Audit before installing. Production setups should pin versions and review server source.
- *"Can I write an MCP server for our internal API?"* — Yes. The MCP spec is small — a few endpoints over JSON-RPC. Half a day for a prototype.

**Pitfall:** Plugin/MCP sprawl. Ten installed, two used, all loaded on every session boot. Audit and prune.

**Transition:** "MCPs give Claude *tools*. Skills give Claude *habits*. Tab 17."

---

## 17 · Skills (6 min)

**Why this tab matters:** Skills are the auto-invoke layer. The difference between a recipe Claude follows when asked vs. a habit Claude picks up automatically.

**What to say:**

> "A skill is a markdown file with frontmatter. The frontmatter has a `description` field. When Claude sees a situation that matches the description, it auto-invokes the skill — without you having to remember to call it.
>
> This is the difference between a *command* and a *skill*. A command is a recipe you call by name. A skill is a habit Claude picks up because the situation cued it.
>
> The most useful skills are workflow skills:
>
> - **debugging** — invokes when you mention a bug, encourages root-cause-first methodology.
> - **brainstorming** — invokes when you start an open-ended question, structures the response into options + tradeoffs + recommendation.
> - **TDD** — invokes when you start a new feature, enforces failing test first.
> - **code review** — invokes when you ask for a review, applies a consistent rubric.
>
> Skills shape how Claude *thinks about* a task — not just what it does."

**Demo — write a skill live:**
```bash
mkdir -p ~/.claude/skills
cat > ~/.claude/skills/debugging.md <<'EOF'
---
name: debugging
description: Use when investigating a bug — root cause first, fix second
---

When the user reports a bug, follow this protocol:

1. Reproduce the bug deterministically. If you can't reproduce, ask for a minimal repro before changing any code.
2. Form a hypothesis BEFORE changing code. Articulate it explicitly.
3. Add a failing test that captures the bug. Run it; confirm red.
4. Fix until the test passes.
5. Add at least one regression guard — a related test or assertion.
6. Summarize the root cause in 2 sentences for the commit message.

Resist:
- The urge to "try a fix and see."
- Skipping reproduction because the bug "looks obvious."
- Adding the fix without a regression test.
EOF
```
Then in claude:
```
> there's a bug in our checkout flow — orders sometimes get charged twice
```

Watch Claude pick up the debugging skill — see it ask for repro steps before suggesting code.

**Audience moment:** "Tell me one bug your team has fixed twice. The shape of the recurrence problem is exactly what a regression-guard skill catches."

**Anticipated Q&A:**

- *"How does Claude decide which skill to invoke?"* — Matches the description against the user prompt. Be specific in descriptions.
- *"Can multiple skills compose?"* — Yes. Multiple skills can invoke per turn if their descriptions match.

**Pitfall:** Vague descriptions like "use for hard problems." Be specific. "Use when investigating a bug" is matchable. "Use when needed" is not.

**Transition:** "We've covered every primitive. Now let's talk about the bills you'll pay if you ignore this. Tab 18."

---

## 18 · Token craft (4 min)

**Why this tab matters:** Eight rules. Each saves real money. None require a config change — they require a habit change.

**What to say:**

> "Eight rules. I'll go fast — they're on the slide.
>
> **One — cache everything stable.** System prompts, CLAUDE.md, large reference docs. Anthropic's prompt cache cuts those by ~90% on cache hits. But the cache is *prefix*-keyed: change a single character early in the prompt and the cache invalidates. So put the unstable part *at the end*.
>
> **Two — keep CLAUDE.md under 50 lines.** Anything longer, link to it. Long CLAUDE.mds slow every session boot and crowd out the actual task context.
>
> **Three — compact at 50%, not 90%.** We covered this. Repeat: at 50%.
>
> **Four — match model to task.** Haiku for drudgery, Sonnet by default, Opus when you hit a wall. Always-Opus is the #1 reason teams overspend.
>
> **Five — subagents for heavy research.** Already covered.
>
> **Six — one task per session.** Already covered.
>
> **Seven — tune extended thinking.** /effort low, medium, high, xhigh. Don't pay for thinking you don't need. xhigh on a one-line typo fix is silly.
>
> **Eight — watch /cost, not vibes.** Already covered, last time."

**Demo:** Show a /cost output and walk the breakdown. Point at thinking tokens. "That number — 7,200 thinking tokens — is /effort high. Drop to medium and it's 1,800. 4x cheaper for this kind of task."

**Audience moment:** Skip.

**Anticipated Q&A:**

- *"Does prompt caching cost anything?"* — Cache writes cost slightly more than non-cache reads. Cache reads are ~10% of normal. Net win after the first hit.

**Pitfall:** People discover /effort xhigh and use it always. Cost spirals.

**Transition:** "OK, the actual math. Tab 19."

---

## 19 · Cost math (4 min)

**Why this tab matters:** Real numbers. Engineers respect numbers. Approximate napkin math beats marketing.

**What to say:**

> "Sonnet 4.6: $3 in / $15 out per million tokens. Opus 4.6: roughly 5x. Haiku 4.5: roughly 1/5.
>
> A typical 1-hour deep session reads ~150K tokens, writes ~30K, with extended thinking ~10K. That's:
>
> 150K × $3/M = $0.45 input
> 30K × $15/M = $0.45 output
> 10K × $15/M = $0.15 thinking
> ≈ $1.05 / hour on Sonnet with extended thinking.
>
> Multiply by 5 for Opus. Divide by 5 for Haiku. Cache hits reduce input by ~90%.
>
> A 40-hour week of heavy use on Sonnet ≈ $42. On Opus, ~$210. On Haiku, ~$8.
>
> The honest formula:
>
> session_$ ≈ (input × $in/M) + (output × $out/M) + (thinking × $out/M)
>
> Why teams overspend: always-Opus, no caching, no compaction, ten MCPs loaded but two used."

**Demo:** Live calculation with whatever /cost shows.

**Audience moment:** "Estimate your team's monthly Claude bill if 5 engineers use it 4 hours a day on Sonnet." (Answer: ~$840/mo. Cheaper than one engineer-hour saved per day per engineer.)

**Anticipated Q&A:**

- *"Where do I see usage by user across my org?"* — Anthropic Console for API. For Pro/Max, individual dashboards. Enterprise has org-level reporting.

**Pitfall:** Estimating by total tokens instead of in/out separately. Output is 5x input cost. Output volume matters more than people think.

**Transition:** "Now you know the cost. Let's talk about a workflow that *deserves* the cost. Tab 20."

---

## 20 · Spec-driven (5 min)

**Why this tab matters:** This is the senior pattern. The five-step flow is the difference between Claude as a magic-feature button and Claude as a teammate who helps you ship clean diffs.

**What to say:**

> "Five steps. Memorize them.
>
> **Step 1 — Requirements.** Write the *what* and the *why*. What is the user's job-to-be-done? Why does this need to exist? Anchor in user value, not implementation.
>
> **Step 2 — Design.** Claude proposes architecture. You critique and approve. This is where /effort high earns its keep. The output is a written design — sequence diagrams, type signatures, data flow.
>
> **Step 3 — Task breakdown.** Split the design into testable units. Each unit is small enough to be implemented in one Claude session. Each unit has its own acceptance criteria.
>
> **Step 4 — Implement.** One task at a time. Free-flow mode. Diff review on every change. Run tests after every meaningful edit (your PostToolUse hook does this for free).
>
> **Step 5 — Verify.** Tests, types, lint, all green. Code review. Spec review — does the implementation actually match the design?
>
> Junior pattern: 'build me a feature.' Senior pattern: 'here's the spec, propose a design, break it down, implement task 1.' Same tool. 3x cleaner diffs. 1/4 the rework."

**Demo:** Run a mini-spec end-to-end:
```
> /effort high. requirements: a /healthz endpoint that returns 200 OK with {status, version, uptime}. why: ops needs to wire it into our k8s readiness probe.

# wait for design
> approve

# task breakdown
> break this into testable tasks

# implement task 1
> implement task 1
```

**Audience moment:** "Name a feature you're starting next week. We'll do step 1-2 for it in 60 seconds." Take one volunteer.

**Anticipated Q&A:**

- *"This feels slow."* — It is, on day one. By week three, the spec phase saves you a day of rework.
- *"Can I skip steps?"* — Skip step 1 if the spec exists already. Don't skip step 2. Step 2 is where the wins compound.

**Pitfall:** Treating step 2 as optional. You end up with code that works and a design no one shares. Six months later it's untouchable.

**Transition:** "Let's see this in the wild. Tab 21."

---

## 21 · Case study (3 min)

**Why this tab matters:** Concrete > abstract. One real story makes the principles stick.

**What to say:**

> "Pick the story on screen — governance, cost, or rollout. Three guardrails that come up in every team I've talked to.
>
> One — **block destructive shell.** PreToolUse hook. Take 20 minutes. Done.
>
> Two — **require a ticket ID on every prompt.** UserPromptSubmit hook. Take 30 minutes. Done.
>
> Three — **ship a shared org CLAUDE.md.** Centralized. Versioned. Onboarding ritual. Take an afternoon. Done.
>
> Three small wins, in that order, in your first month. That's the rollout playbook."

**Demo:** None — narrative tab.

**Audience moment:** "Which of those three would your team push back on?" Quick discussion.

**Anticipated Q&A:** None expected.

**Pitfall:** None.

**Transition:** "Same tool, different shapes for different jobs. Tab 22."

---

## 22 · By role (4 min)

**Why this tab matters:** Concretely shows what each role gets. Helps people see themselves in the workflow.

**What to say:**

> "Same tool, three jobs.
>
> **Engineer** — flow, quality, context. Parallel git worktrees so multiple Claudes can work in parallel without fighting over branches. Subagent research swarm for heavy reads. A custom `/onboard` for new repo entry. Auto-test on every edit.
>
> **QA** — coverage, flake, accessibility. User story → Playwright suite (one prompt, one MCP, one test file). Accessibility gate in CI. Flake triage from the last 20 runs. Visual regression with pixelmatch.
>
> **Lead/Manager** — governance, cost, rollout. Daily org spend dashboard, grouped by model and user. Weekly productivity digest. Foot-gun insurance via hooks. Budget caps via API."

**Demo:** Show one item from each role. The QA one is fun:
```
> read this user story and generate a playwright suite that covers happy path + 3 edge cases
```

**Audience moment:** "Which role are you?" Show of hands. "Watch the next tab — 'What's new' — for the recent feature most relevant to your role."

**Anticipated Q&A:**

- *"My team is engineering and QA — same person."* — Pick the patterns from each list that match the work, not the title.

**Pitfall:** Adopting all 12 patterns at once. Pick three.

**Transition:** "What's new since you last looked. Tab 23."

---

## 23 · What's new (3 min)

**Why this tab matters:** Claude Code ships fast. The feature you decided wasn't useful three months ago is probably good now.

**What to say:**

> "Scroll the timeline. The big April 2026 changes:
>
> - **Live Artifacts** — shareable mini-apps from inside Claude Code.
> - **Claude Design** — design surface for prototyping UIs without leaving the agent.
> - **Managed Agents** — cloud-hosted long-running agents.
> - **Opus 4.7 with auto-mode** — model that decides when to use thinking.
> - **1-hour + forced 5-minute prompt caching** — much bigger cache windows.
>
> Pick one this week. Try it. Don't try all five."

**Demo:** Quick scroll of the timeline.

**Audience moment:** "Which one is most relevant to your role? Skim the timeline; flag one."

**Anticipated Q&A:** None expected.

**Pitfall:** None.

**Transition:** "Resources to keep going. Tab 24."

---

## 24 · Resources + close (2 min)

**Why this tab matters:** Bookmark links. They walk out with a path forward.

**What to say:**

> "On the resources tab: official docs, this site (which you can come back to), the GitHub repo (which is where this site is hosted), the MCP registry, the Anthropic Discord.
>
> Final line — and this is the only thing I want you to remember from this session:
>
> *You now have a teammate who can touch your keyboard. The work between today and being productive is — ten lines in CLAUDE.md, two hooks, one custom command, one MCP. Do that this week.*
>
> Thank you. Questions?"

**Demo:** None.

**Audience moment:** None.

**Anticipated Q&A:** All of Q&A below.

---

## Q&A (10 min)

Pre-prepared answers for the 10 most likely questions:

**1. "Is my code sent to Anthropic?"**
Prompts and the files Claude reads are sent to the API. `.claude/ignore` and gitignore prevent files from being read. Anthropic does not train on API traffic by default for Pro/Max/API.

**2. "How does it compare to Copilot/Cursor?"**
Different category. Copilot is autocomplete. Cursor is autocomplete + chat in IDE. Claude Code is an *agent* that runs commands, owns the loop. They coexist; many teams use Claude Code in CLI and Copilot in IDE.

**3. "Can it run offline?"**
No. The model lives in Anthropic's cloud. Local models via MCP exist but aren't the default story.

**4. "Does it edit my git history?"**
Only when you let it. It can commit, branch, push — every git tool call goes through the same approval flow as any other tool. Default is propose, you approve.

**5. "How do I roll this out across a team?"**
Three steps. (1) Org CLAUDE.md, shared by reference in every repo. (2) Shared `.claude/` per repo with hooks. (3) Cost dashboard. See tab 21.

**6. "What about secrets — API keys, .env files?"**
Add `.env*` to `.claude/ignore`. PreToolUse hook can also redact secrets in tool inputs. UserPromptSubmit hook can block secrets in prompts.

**7. "Can I use it in a regulated environment (HIPAA / SOC 2 / GDPR)?"**
Anthropic offers BAA for HIPAA on enterprise. SOC 2 is in place. GDPR-compliant via the European data residency option. Confirm with your compliance team.

**8. "What's the learning curve?"**
Hour 1 — install and confused. Hour 5 — productive. Hour 20 — write your first hook. Hour 50 — you'll have a personal CLAUDE.md and 4 custom commands.

**9. "How does it handle very large repos?"**
Doesn't load everything. Reads on demand via Read/Grep tools. Use a code-graph plugin for impact analysis on monorepos.

**10. "What if it makes a destructive change?"**
PreToolUse hooks block. /resume rewinds. Git is your friend. The combination of these three has caught every "oh no" moment I've seen.

---

## Post-session

Send to attendees:

- Live tutorial URL
- HANDOUT.md
- COMMANDS_CHEATSHEET.md
- A 1-week follow-up email with: "Did you write CLAUDE.md? Did you write a hook? Did you write a custom command?"

The follow-up is what converts attendance into adoption. Without it, retention drops 70%.
