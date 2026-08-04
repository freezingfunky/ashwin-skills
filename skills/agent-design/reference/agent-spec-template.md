# Agent spec template + pre-flight checklist

Fill this in *before* building any agent. Start with **0. DECISIONS**. Copy the block,
answer every field; "N/A" is a valid answer but must be a *deliberate* one.

## The spec

```
AGENT: <name — one job in the name if you can>

0. DECISIONS (decision map)
   - Singular outcome (one sentence):
   - DONE when:
   - FAILED when:
   - Aspects considered (list):
   - Smell test: any aspect with an independent DONE/FAILED?
       - No → one agent (continue)
       - Yes → SPLIT list (name + DONE each); this session designs only:
   - Fan-out parked agents in parallel? Y/N (default N):
       - If Y: launcher used + session ids / output paths:
       - If N: park one-liners for re-entry:
   - Open questions still foggy (or "none"):
   - Ready to spec pillars? Y/N

1. TASK
   - Job (one sentence):
   - Optimization target / deliverable:
   - DONE when:
   - FAILED when:
   - Explicitly NOT this agent's job:

2. CONTEXT
   - Preloaded every run (keep minimal):
   - Fetched just-in-time (and how it fetches):
   - The hand-off shape (structured map > document dump):
   - What it must NOT see:

3. MEMORY
   - Persists across runs:
   - Ephemeral to one run:
   - WRITE rule (what's worth saving):
   - READ rule (what's relevant to recall):
   - SUPERSEDE rule (what overrides an older fact):
   - Audience / visibility scope:

4. EVAL
   - How "good" is measured (assertion | rubric-over-gold-set):
   - The bright line that must never be crossed:
   - Grader independence (different model / held-out cases):
   - Where the eval runs (local | CI-blocking | scheduled):

5. SAFETY / SHAPE
   - Does it mutate anything? If yes → propose-then-confirm gate:
   - Reversibility (batch undo?) + audit trail:
   - Tools it's allowed (least privilege — list them):
   - Known failure modes, encoded as guardrails:

6. OUTPUT CONTRACT
   - Exact shape it returns (schema / format):
   - Who/what consumes it:
```

## Pre-flight checklist (run before shipping)

0. Did the decision map lock a **singular** outcome (or an explicit split + this agent only)? ▢
1. Can the agent tell, unaided, when it's **done**? ▢
2. Can it tell when it's **failed** (not just "kept going")? ▢
3. Is the context the **minimum** that makes the right move obvious — nothing dumped? ▢
4. Is expensive/volatile context fetched **just-in-time**, not preloaded? ▢
5. If it has memory: is there a **supersession** rule (no stale facts win)? ▢
6. Is there an **eval** — an assertion or a rubric/gold-set — that catches regression? ▢
7. If generative: is the grader **independent** of the generator? ▢
8. If it mutates anything: is the write **gated** (propose → confirm) and **reversible**? ▢
9. Does it have **only** the tools its one job needs? ▢
10. Are this agent's **known failure modes** written down as explicit guardrails? ▢

Any unchecked box is a known hole — ship it knowingly or fix it, but don't discover it
later.

## Three shapes, for calibration

- **A tuner / optimizer** (improves an artifact against a metric): the eval *is* the
  task. Loop = inspect worst cases → form a falsifiable hypothesis → make the *smallest*
  change → re-run the eval → keep / revert / expand. Guardrail: never tune to a single
  case; never edit the grader to mask a regression.
- **A propose-confirm mutator** (changes real state): reads and previews autonomously,
  *proposes* a plan, writes only after an explicit confirm; every change shares a batch
  id so the whole batch reverts.
- **A streaming / real-time agent** (e.g. voice): compose it from small, testable
  processors in the stream rather than one monolith; each processor is unit-tested.

The common thread: a bounded task, the minimum context, an honest memory rule, and an
eval that fails loudly. Everything else is detail.
