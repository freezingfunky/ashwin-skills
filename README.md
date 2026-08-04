# Ashwin Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

Agent skills I use every day to ship real product — not vibe coding. Small, composable, born from failure modes in Cursor + Claude Code.

These skills are designed to be small, easy to adapt, and composable. They work with any model that loads agent skills. Hack around with them. Make them your own.

## Installation (30-second setup)

```bash
npx skills@latest add freezingfunky/ashwin-skills
```

Pick the skills you want, and which coding agents to install them on.

Or copy the folders under `skills/` into your agent's skills directory
(e.g. `~/.claude/skills/`, `~/.cursor/skills/`).

## Why these skills exist

### #1: The agent said the UI was fixed. The screen didn't change.

Source edited, typecheck green, Fast Refresh quietly stale — source ≠ served HTML/CSS.

**Fix:** [`/render-check`](./skills/render-check/SKILL.md) — prove the *live* markup and compiled CSS changed before you call it done.

### #2: You switched from Cursor to Claude Code. The other chat has no idea what you were doing.

Chats don't sync across tools. Pasting a wall of context is lossy and slow.

**Fix:** [`/dual-tool-handoff`](./skills/dual-tool-handoff/SKILL.md) — named thread-summary handoff in the repo (opt-in); for **switching** tools, not keeping two sessions in sync. Other tool resumes by name or via `LATEST`.

### #3: The agent can do everything — so it does nothing reliably.

Flaky agents are usually missing Task · Context · Memory · Eval, often wrapped around *multiple* outcomes.

**Fix:** [`/agent-design`](./skills/agent-design/SKILL.md) — decision-map forces a **singular outcome** (or splits and optionally fans out in parallel after confirm); then lock the four pillars on purpose.

## Skills

| Skill | Job |
|---|---|
| **[render-check](./skills/render-check/SKILL.md)** | Prove served HTML/CSS changed before calling a UI fix done |
| **[dual-tool-handoff](./skills/dual-tool-handoff/SKILL.md)** | Named, switch-only Cursor ↔ Claude Code handoff |
| **[agent-design](./skills/agent-design/SKILL.md)** | Decision-map → singular outcome → pillars (optional parallel fan-out) |

## License

MIT — see [LICENSE](./LICENSE).
