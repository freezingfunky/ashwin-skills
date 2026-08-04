---
name: dual-tool-handoff
version: 1.0.0
description: >
  Opt-in only. Write or read a Cursor ↔ Claude Code (or any two coding agents) handoff
  at `.scratch/handoffs/LATEST.md` when the user explicitly asks ("handoff", "write handoff",
  "hand off to Cursor/Claude Code"). Do not run proactively. Chats do not sync; the repo file does.
---

# Dual-tool handoff — opt-in

Chats do not sync across Cursor, Claude Code, Codex, etc. The **repo file** does.
**Off by default** — only act when the user asks for a handoff.

## When *not* to use

- The user did not ask to hand off / resume from a handoff.
- Same-tool session compact (use that tool's native handoff if they want chat→chat in one product).
- The next step is a brand-new task with no continuity from this session.

## Read (when asked to resume / hand off)

1. Read `.scratch/handoffs/LATEST.md` if present.
2. Use its branch, ticket/spec pointers, shipped/left, and resume one-liner.
3. Do not reopen decisions the handoff marks as closed.

## Write (when asked)

Prefer the bundled script (auto-fills branch / dirty tree). From the **consuming repo**
(after install, copy or symlink the script to `scripts/agent-handoff`, or invoke it by path):

```bash
path/to/ashwin-skills/skills/dual-tool-handoff/scripts/agent-handoff \
  --tool claude-code \
  --ticket <path or n/a> \
  --spec <path or n/a> \
  --shipped "<what this session completed>" \
  --left "<what remains>" \
  --gates "none" \
  --questions "<none or list>"
```

Use `--tool cursor` when invoking from Cursor (or whatever tool name fits).

If the script is unavailable, copy
`skills/dual-tool-handoff/templates/LATEST.template.md` →
`.scratch/handoffs/LATEST.md` in the **project** and fill every section.

### Failure modes

- **Missing script / template:** fall back to the template copy; still write `LATEST.md`.
- **PHI / secrets in the handoff:** redact — IDs and paths only, never transcript/note bodies or keys.
- **Stale LATEST:** always overwrite on write; dated copies are kept alongside for history.

## Done when

- `.scratch/handoffs/LATEST.md` exists and matches this session
- Resume one-liner is printable for the other tool
- No secrets / transcript bodies in the handoff

## Product-agnostic example

Building a Next.js app in Cursor; switching to Claude Code for a long refactor:

1. User: "hand off to Claude Code"
2. Agent runs `agent-handoff --tool cursor --shipped "Auth form UI" --left "Wire Supabase session" --ticket n/a --spec n/a`
3. In Claude Code: "resume from handoff" → agent reads `LATEST.md` and continues on the same branch
