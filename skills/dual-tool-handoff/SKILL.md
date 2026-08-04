---
name: dual-tool-handoff
version: 1.1.0
description: >
  Opt-in only. Write or read a named Cursor ↔ Claude Code handoff under
  `.scratch/handoffs/` when the user explicitly asks. Each handoff has a
  thread-summary name usable on both tools. For switching tools — not for
  keeping two parallel sessions in sync. Do not run proactively.
---

# Dual-tool handoff — opt-in, named, switch-only

Chats do not sync across Cursor, Claude Code, Codex, etc. The **repo file** does.
**Off by default** — only act when the user asks for a handoff.

## Contract

- **Named:** every handoff gets a short **thread-summary name** (e.g. `auth-form-ui`,
  `oss-skills-launch`) — same string both tools use to resume.
- **Switch-only:** use this when leaving one tool for the other. It does **not** keep
  two live sessions in sync. Last write to `LATEST.md` is “what I’m switching to now.”
- **Not parallel sync:** if two chats write at once, they will clobber `LATEST`; prefer
  one active switch at a time. Older named files remain for explicit resume.

## When *not* to use

- The user did not ask to hand off / resume.
- Same-tool session compact (use that product’s native handoff).
- Trying to mirror two tools running **at the same time** — out of scope; pick one
  writer, then switch.

## Write (when asked)

1. Agree a **thread-summary name** with the user (or derive one from what shipped /
   what’s left — 2–5 words, slug-safe). Say the name aloud so both sides can reuse it.
2. Run the script with `--name`:

```bash
path/to/ashwin-skills/skills/dual-tool-handoff/scripts/agent-handoff \
  --name "auth form UI" \
  --tool cursor \
  --ticket <path or n/a> \
  --spec <path or n/a> \
  --shipped "<what this session completed>" \
  --left "<what remains>" \
  --gates "none" \
  --questions "<none or list>"
```

Writes:

| File | Role |
|---|---|
| `.scratch/handoffs/<slug>.md` | **Canonical** named handoff (resume by name) |
| `.scratch/handoffs/LATEST.md` | Copy of the same body — **current switch target** |
| `.scratch/handoffs/<stamp>-<slug>.md` | Immutable snapshot |

Use `--tool cursor` or `--tool claude-code` (or other) for the tool you’re **leaving**.

If the script is unavailable, copy `templates/LATEST.template.md` →
`.scratch/handoffs/<slug>.md`, fill it (include **Name:**), then copy that file to
`LATEST.md`.

## Read (when asked to resume)

1. If the user names a thread (`resume auth-form-ui` / `handoff auth form UI`):
   read `.scratch/handoffs/<slug>.md` (slugify the same way as the script).
2. If they only say “resume from handoff” / “read the handoff”:
   read `LATEST.md` (the last switch).
3. If the named file is missing but dated snapshots exist: list recent
   `.scratch/handoffs/*.md` (excluding `LATEST`) and ask which name to open.
4. Use branch, ticket/spec, shipped/left, resume one-liner. Do not reopen closed decisions.

## Failure modes

- **Missing `--name`:** derive from shipped/left; if still empty, ask — don’t write
  an anonymous handoff.
- **PHI / secrets:** redact — IDs and paths only.
- **Parallel writers:** warn that this skill is switch-only; `LATEST` reflects the
  last write, not a merge of two sessions.

## Done when

- Named file + `LATEST.md` exist and match
- User has been told the **name** to quote on the other tool
- Resume one-liner is printable
- No secrets / transcript bodies

## Example

1. In Cursor: “hand off to Claude Code — call it **auth form UI**”
2. Script writes `auth-form-ui.md` + `LATEST.md`
3. In Claude Code: “resume **auth form UI**” → reads `auth-form-ui.md`  
   (or “resume from handoff” → reads `LATEST.md` after a single switch)
