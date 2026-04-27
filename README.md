# awesome-claude-code-walkthrough

A 2-hour, hands-on field guide to **Claude Code** — Anthropic's agentic coding CLI.

> **Live site:** https://shashank2577.github.io/awesome-claude-code-walkthrough/

This repo hosts a single-page interactive tutorial (`index.html`) that walks through every surface of Claude Code: models, slash commands, hooks, MCP, plugins, skills, subagents, context discipline, cost math, and spec-driven development. It is designed to be projected during a live workshop while a presenter narrates and demos against a real terminal.

## What's inside

| File | Purpose |
| ---- | ------- |
| `index.html` | The full interactive tutorial (24 tabs). Open in any modern browser. |
| `docs/SESSION_SCRIPT.md` | Minute-by-minute presenter script for a 2-hour workshop. |
| `docs/HANDOUT.md` | Attendee handout — install steps, takeaways, mental models. |
| `docs/COMMANDS_CHEATSHEET.md` | One-pager of slash commands, hooks, and config paths. |
| `.nojekyll` | Tells GitHub Pages to serve the file as-is (no Jekyll build). |

## How to run the workshop

1. **Before the session** — share the live URL with attendees and ask them to install Claude Code (`npm i -g @anthropic-ai/claude-code` or visit claude.ai/code).
2. **During the session** — project `index.html` and follow `docs/SESSION_SCRIPT.md`. Each tab in the top nav corresponds to one section of the script.
3. **After the session** — distribute `docs/HANDOUT.md` and `docs/COMMANDS_CHEATSHEET.md`.

## Local preview

```bash
git clone https://github.com/Shashank2577/awesome-claude-code-walkthrough.git
cd awesome-claude-code-walkthrough
python3 -m http.server 8080
# open http://localhost:8080
```

## Credits

Made with ♥ by Engineering Core · engineering_core@taazaa.com
