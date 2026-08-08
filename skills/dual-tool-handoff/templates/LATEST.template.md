# Agent handoff

## Meta

- **Name:** (short thread summary — same string both tools quote)
- **Slug:** `(kebab-case)`
- **When:** (ISO or local datetime)
- **Tool (leaving):** cursor | claude-code | other
- **Branch:** `(branch)`
- **Worktree:** main checkout | path
- **Ticket:** path or n/a
- **Spec:** path or n/a
- **Claude session id:** uuid or n/a
- **Cursor session id:** composerId or n/a

## What shipped

- (what this session completed)

## What's left

- (remaining work)

## Diff check

- **Tree:** dirty | clean
- **Paths (script):**
  - (none)
- **Alignment (agent):** ok | warn
- **Note (agent):** (required if warn)

## Flags / migrations

- Flag: n/a
- Migration: n/a

## Ready for gates

- [ ] none
- [ ] (add your project gates)

## Open questions

- none

## Working tree (auto)

```
(paste git status --porcelain or "(clean)")
```

## Resume one-liner

> Resume handoff **(Name)** (`slug`) from `(ticket)` on branch `(branch)`. Spec is `(spec)`. Don't reopen closed decisions.
