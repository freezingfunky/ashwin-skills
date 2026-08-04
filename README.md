# Ashwin Skills — Claude Code & Cursor agent skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skills-black)](https://code.claude.com)
[![Cursor](https://img.shields.io/badge/Cursor-skills-blue)](https://cursor.com)

**Agent skills for Claude Code, Cursor, and other coding agents** — workflows I use every day to ship real product, not vibe coding.

Cross-platform session handoff · UI render verification · agent design with a singular outcome.

Born from failure modes switching between **Claude Code** and **Cursor** when plan limits hit mid-task.

## Install (Claude Code, Cursor, Codex, …)

```bash
npx skills add freezingfunky/ashwin-skills
```

Or copy folders from [`skills/`](./skills/) into your agent skills directory:

| Agent | Skills path |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Cursor | `~/.cursor/skills/` |
| Codex / others | see your agent’s docs |

## Skills

| Skill | What it solves | Keywords |
|---|---|---|
| **[dual-tool-handoff](./skills/dual-tool-handoff/SKILL.md)** | Claude Code ↔ Cursor session handoff when chats don’t sync cross-platform | `claude code cursor handoff`, `switch tools`, `named session` |
| **[render-check](./skills/render-check/SKILL.md)** | Prove a UI fix actually rendered (served HTML/CSS, not just source) | `fast refresh`, `next.js`, `tailwind`, `stale build` |
| **[agent-design](./skills/agent-design/SKILL.md)** | Design LLM agents with a decision map + Task · Context · Memory · Eval | `agent design`, `eval`, `singular outcome` |

## Why these skills exist

### Sessions don’t sync between Claude Code and Cursor

Hit a Claude Max / plan limit mid-refactor, switch to Cursor — blank chat, context gone. Pasting is lossy.

**Fix:** [`/dual-tool-handoff`](./skills/dual-tool-handoff/SKILL.md) — named handoff file in the repo; resume the same session name on the other tool. Switch-only (not live sync).

### The agent said the UI was fixed. The screen didn’t change.

Source edited, typecheck green, Fast Refresh stale — source ≠ served HTML/CSS.

**Fix:** [`/render-check`](./skills/render-check/SKILL.md) — prove the live markup and compiled CSS changed before calling it done.

### The agent can do everything — so it does nothing reliably

Flaky agents often wrap multiple outcomes and skip eval.

**Fix:** [`/agent-design`](./skills/agent-design/SKILL.md) — decision-map forces a singular outcome (or splits + optional parallel fan-out); then lock the four pillars.

## Quick start — dual-tool handoff

```text
1. In Claude Code: “hand off to Cursor”
2. Pick a session name (or continue an existing one)
3. Skill writes .scratch/handoffs/<slug>.md
4. In Cursor: “resume <that name>”
```

The session isn’t the source of truth. The repo file is.

## Repo layout

```text
skills/
  dual-tool-handoff/   # Claude Code ↔ Cursor handoff
  render-check/        # verify UI actually rendered
  agent-design/        # design agents on purpose
```

## Related searches

Claude Code skills · Cursor skills · agent skills · AI coding agent workflows · cross-platform handoff · Claude Max limit · Fast Refresh staleness · LLM agent design

## License

MIT — see [LICENSE](./LICENSE).
