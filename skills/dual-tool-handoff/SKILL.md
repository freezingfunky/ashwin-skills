---
name: dual-tool-handoff
version: 1.3.0
description: >
  Opt-in only. Write or read a named Cursor ↔ Claude Code handoff under
  `.scratch/handoffs/` when the user explicitly asks. Always ask which thread.
  After write, fill Diff check alignment (ok|warn) against git paths — warn never
  blocks. Switch-only, not live sync. Do not run proactively.
---

# Dual-tool handoff — opt-in, named, switch-only

Chats do not sync across Cursor, Claude Code, Codex, etc. The **repo file** does.
**Off by default** — only act when the user asks for a handoff.

**Diff ≠ why.** Git supplies paths; the agent curates `shipped` / `left`. Diff check
**warns** when they look misaligned — it never rewrites the story and never blocks the write.

## Contract

- **Named:** every handoff gets a short **thread-summary name** (e.g. `auth-form-ui`,
  `oss-skills-launch`) — same string both tools use to resume.
- **Always ask which thread** before write or resume (unless the user already named
  one in the same message).
- **Switch-only:** use this when leaving one tool for the other. It does **not** keep
  two live sessions in sync. Last write to `LATEST.md` is “what I’m switching to now.”
- **Not parallel sync:** if two chats write at once, they will clobber `LATEST`; prefer
  one active switch at a time. Older named files remain for explicit resume.
- **Diff check:** script fills Tree + Paths; agent fills Alignment + Note. **warn ≠ block.**

## When *not* to use

- The user did not ask to hand off / resume.
- Same-tool session compact (use that product’s native handoff).
- Trying to mirror two tools running **at the same time** — out of scope; pick one
  writer, then switch.

## Pick the thread (do this first)

Before writing or reading, **stop and ask** which thread — unless the user’s message
already includes a clear name (“hand off **auth form UI**” / “resume oss-skills-launch”).

1. List existing threads: non-`LATEST`, non-dated-snapshot files in
   `.scratch/handoffs/` whose names look like `<slug>.md` (skip
   `YYYY-MM-DD-HHMM-*.md` snapshots if you can tell them apart; when unsure, list
   all `.md` except `LATEST.md` and let the user pick).
2. Present choices like:
   - existing: `auth-form-ui`, `oss-skills-launch`, …
   - **New thread** — ask for a 2–5 word summary to use as the name
3. Wait for the answer. Do **not** default to `LATEST` when more than one named
   file exists.
4. If the folder is empty and this is a write: ask for a new thread name (you may
   suggest one from shipped/left, but the user confirms).
5. If the folder is empty and this is a resume: say there is nothing to resume.

## Write (when asked)

1. Run **Pick the thread**. Reuse an existing name to continue that thread; choose
   **New** only for a new line of work. Say the chosen name aloud so the other tool
   can quote it.
2. Draft **shipped** / **left** from the session (agent narrative — not from the diff).
3. Run the script with `--name` (fills Meta, Working tree, Diff check Tree+Paths):

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

4. **Fill Diff check** (required before done):
   - Read **Tree** and **Paths**.
   - Set **Alignment** using the bar below.
   - Set **Note** (required if warn).
   - Edit the named file + `LATEST.md` (same body) so placeholders are gone.
5. Tell the user the session **name** to resume on the other tool.

Writes:

| File | Role |
|---|---|
| `.scratch/handoffs/<slug>.md` | **Canonical** named handoff (resume by name) |
| `.scratch/handoffs/LATEST.md` | Copy of the same body — **current switch target** |
| `.scratch/handoffs/<stamp>-<slug>.md` | Immutable snapshot |

Use `--tool cursor` or `--tool claude-code` (or other) for the tool you’re **leaving**.

If the script is unavailable, copy `templates/LATEST.template.md` →
`.scratch/handoffs/<slug>.md`, fill it (include **Name:** and Diff check), then copy
to `LATEST.md`. Collect paths yourself via `git status` / `git diff --name-only` /
`git diff --cached --name-only`.

### Alignment bar (shipped only)

| Case | Alignment |
|---|---|
| `shipped` themes overlap touched paths | **ok** |
| Decision-only / narrative; clean or unrelated paths; Note explains | **ok** (Note required) |
| Clean tree + `shipped` claims code landed | **warn** |
| Overclaim (e.g. “auth done”, diff only README) | **warn** |

Unsure → **warn** + Note. Never rewrite `shipped`/`left` from the diff. Do not validate `left`.

## Read (when asked to resume)

1. Run **Pick the thread** (unless the user already named one).
2. Read `.scratch/handoffs/<slug>.md` for the chosen thread.
3. Only if there is **exactly one** named handoff and the user said a bare
   “resume from handoff” with no name: you may use that file or `LATEST.md`
   without asking — still tell them which name you opened.
4. If they named a thread that doesn’t exist: list what does and ask again.
5. Use branch, ticket/spec, shipped/left, Diff check (if warn, treat as advisory),
   resume one-liner. Do not reopen closed decisions.

## Failure modes

- **Skipped the ask:** wrote or resumed without confirming the thread when multiple
  exist (or when the user didn’t name one) → rewind and ask.
- **Missing `--name`:** never write anonymous; ask.
- **Skipped Diff check / left placeholders:** incomplete — fill Alignment (+ Note if warn).
- **Marked ok without reading Paths:** rewind; apply the alignment bar.
- **Rewrote shipped from the diff:** undo — Diff ≠ why.
- **PHI / secrets:** redact — IDs and paths only.
- **Parallel writers:** warn that this skill is switch-only; `LATEST` reflects the
  last write, not a merge of two sessions.

## Done when

- Thread was confirmed (or clearly named in the user’s message)
- Named file + `LATEST.md` exist and match (on write)
- **Diff check** present: Alignment filled; Note present if warn
- User has been told the **name** to quote on the other tool
- Resume one-liner is printable
- No secrets / transcript bodies

## Example

1. User: “hand off to Claude Code”
2. Agent lists: `auth-form-ui` · `oss-skills-launch` · **New thread**
3. User: “auth form UI”
4. Agent drafts shipped/left → runs script → fills Alignment (e.g. ok) + Note if needed
5. On the other tool: “resume” → agent lists threads → user picks **auth form UI**
