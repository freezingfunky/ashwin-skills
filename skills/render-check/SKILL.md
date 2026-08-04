---
name: render-check
version: 1.0.0
description: >
  Verify a frontend change ACTUALLY rendered before calling it done. Inspect the
  served HTML + compiled CSS on the live dev server — never trust source edits or a
  normal browser refresh. Catches the dev-server staleness trap (Fast Refresh
  silently drops edits) that makes every "fix" look identical to the last.
  Use after any UI/layout/Tailwind change, and as the gate before saying "done".
---

# Render Check

The source file is not the truth. The **served markup + compiled CSS** are the truth.

## Why this skill exists

AI agents love to say "done" after editing a React/Tailwind file. Sometimes the **source**
changed but the **browser** is still serving a stale build (Next.js Fast Refresh quietly
stopped updating). Typecheck passes. Curl returns 200. The screen doesn't change.

**Iron rule: do not tell the user a visual change is done until the served HTML contains the
new markup AND the served CSS contains the new rule.** "I edited the file and tsc passes" is
not evidence the change rendered.

## Discover project specifics first

Do not hardcode ports or paths. Before checking:

1. Find how this repo starts the dev server (package.json scripts, README, CLAUDE.md).
2. Note the **port** and the **route** under test.
3. Note the build-cache dir (often `.next/` for Next.js; `.vite/` / `dist/` for Vite).
4. Find the served CSS link from the HTML — don't invent the path.

## Protocol (run before confirming a frontend change)

### 1. Make sure the server is alive and freshly compiled

```bash
# Replace PORT with the project's dev port
lsof -ti :PORT >/dev/null 2>&1 && echo "up" || echo "down"
```

If the change touched layout/Tailwind and the result looks unchanged, **assume staleness
first** — restart clean rather than re-editing CSS:

```bash
lsof -ti :PORT | xargs kill -9 2>/dev/null
rm -rf .next   # or this project's cache dir
# restart via the project's usual dev command (background)
until curl -fsS -o /dev/null "http://localhost:PORT/<route>"; do sleep 2; done
```

### 2. Confirm the new MARKUP is in the served HTML

Grep the live page for the exact classes/attributes/text you just added:

```bash
URL="http://localhost:PORT/<route>"
curl -sS "$URL" --max-time 30 | grep -c "<the new class or text>"
```

`0` = the change did not render. Stop. Restart the server (step 1). Re-check. Do NOT keep
editing source — the source may already be correct.

**Grep gotcha — interpolation markers:** React may insert `<!-- -->` around `{interpolations}`,
so a substring that straddles a dynamic value won't match raw HTML. Grep a purely-static
substring, or strip tags first:

```bash
curl -sS "$URL" | python3 -c "import sys,re,html;print(html.unescape(re.sub(r'<[^>]+>',' ',sys.stdin.read())))" | grep -c "<text>"
```

**Grep gotcha — RSC / hydration payloads:** App Router and similar frameworks embed data as
JSON in the HTML. A string can appear in the count even when it's not in the visible DOM.
To verify what actually *renders*, inspect the tag-stripped text region around an anchor —
don't trust total counts alone.

### 3. Confirm the new CSS RULE was generated

A class in the HTML does nothing if the CSS toolchain never emitted the rule (Tailwind JIT
only generates classes it sees at build time):

```bash
# Next.js example — adapt the CSS URL pattern to this stack
CSS=$(curl -sS "$URL" --max-time 30 | grep -oE '/[^"]+\.css' | head -1)
BODY=$(curl -sS "http://localhost:PORT$CSS" --max-time 30)
echo "$BODY" | grep -c '<utility-name>'   # expect >= 1
```

`0` = CSS never picked up the class → stale build or an unscanned file. Restart clean (step 1).

### 4. (When layout is the doubt) verify the COMPUTED result

Markup + rules present but it still looks wrong = a real CSS logic bug (not staleness). Only
now reason about the CSS. If a headless browser is available, screenshot at a desktop width
and inspect computed styles. Common height-chain culprits:

- A flex/grid child needs `min-h-0` to scroll instead of growing.
- A grid filling a height needs its row set to `1fr`, not auto.
- `h-full` (percentage) is flaky across flex nesting — prefer stretch.

### 5. Tell the user to hard-refresh

Even with the server fixed, their tab caches old CSS/HTML. End with: **"hard-refresh
(Cmd+Shift+R / Ctrl+Shift+R)"** — a normal refresh often won't pull the new asset.

## Done gate

Only say a visual change is done when you can state, concretely:

- served HTML contains `<new markup>` (grep count ≥ 1), and
- served CSS contains `<new rule>` (grep count ≥ 1), and
- the user has been told to hard-refresh.

If you can't produce those two grep results, the honest status is **NOT done — unverified**,
not "done".

## Output format

Report the evidence, not the intent:

```
RENDER CHECK — /<route>
  server:    restarted clean / already fresh
  markup:    <token> ×N  ✓/✗
  css:       <rule> present  ✓/✗
  computed:  inspected / not inspected
  action:    hard-refresh your tab
  verdict:   RENDERED ✓  /  NOT RENDERED — restarted, re-check
```
