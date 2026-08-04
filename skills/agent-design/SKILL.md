---
name: agent-design
description: >
  How to build an AI agent that actually works — start with a decision-map that forces a
  singular outcome (or splits into multiple agents), design one in-session, then optionally
  fan out parked agents in parallel after confirm. Locks Task · Context · Memory · Eval.
  Use when designing, building, reviewing, or debugging an LLM agent, subagent, tool-using
  assistant, or skill. For the fill-in template see reference/agent-spec-template.md.
---

# Agent design — decision map, then four pillars

Most "I'm not good at building agents" is really "I build them by feel and can't
predict whether the next one will be good." The fix is to name the method.

A working agent is four things, and a weak agent is usually missing one:

> **Task · Context · Memory · Eval.**

But first: **one agent = one outcome.** If you can name two outcomes that succeed or
fail independently, you have two agents — each gets its own design pass.

## 0. Decision map — singular outcome or split

Do this **before** filling the pillars. Completion criterion: either (a) one singular
DONE/FAILED pair locked, or (b) an explicit split list, **this session designs one**, and
parked agents are either fan-out-spawned after confirm or left for later re-entry.

### Steps

1. **Draft the singular outcome** in one sentence (what "this agent worked" means).
2. **List aspects** that feel like separate focuses (intake, summarize, notify, …).
3. **Smell test each aspect:** same DONE/FAILED as the draft outcome?
   - Same → keep inside this agent.
   - Independent success/fail → **split** into a separate agent name + its DONE.
4. **Emit one of:**
   - **One agent** — proceed to pillars below for that outcome.
   - **Multiple agents** — print the full split table (name · DONE · FAILED). Pick
     **one** to design in this session (recommend the dependency-root). **Park** the rest.
     Then run **§0b Fan-out** before or while continuing pillars for the in-session agent.
5. **Fog vs size:**
   - One outcome, still foggy (open decisions) → stay in the decision map; resolve
     open questions one at a time until the outcome is sharp.
   - One outcome, too big for one run → still **one** agent design; use handoff/memory
     across runs — do **not** invent a second agent unless the outcome splits.

### 0b. Fan-out (parked agents → parallel sessions)

**Never spawn without an explicit user confirm.** Silent fan-out is a failure mode.

After the split list is on screen, ask **once**:

> Spawn parked agents in parallel now? (yes / no — default no)

#### If **no**

Print the park list with a one-liner each: `Re-enter /agent-design for <name> — DONE: …`
Continue pillars for the in-session agent only.

#### If **yes**

1. Keep designing the **in-session** agent here (pillars + spec).
2. For **each parked** agent, launch **one** parallel session whose *only* job is
   `/agent-design` for that agent — already locked DONE/FAILED, skip re-smell of the
   parent mega-job, go straight to pillars + spec.
3. Use whatever launcher this runtime supports (try in order; stop at first that works):
   - **Cursor:** Task / background subagent — one task per parked agent; prompt must
     include agent name, DONE, FAILED, and “run agent-design pillars only; write the
     filled spec to a path the user names (default `.scratch/agent-design/<name>.md`).”
   - **Claude Code:** `claude --bg --name "agent-design:<name>" "<prompt as above>"`
     (or the host’s equivalent background-agent API).
   - **Neither available:** do not fake parallelism — fall back to the park list and
     tell the user the runtime can’t spawn; they re-enter manually.
4. Report back: table of parked agent → session id / job name / output path (or “queued”).
5. Each parallel session must produce its own filled `reference/agent-spec-template.md`
   (or the agreed path). Do **not** merge specs into one mega-doc.

**Prompt template for each spawned session:**

```
Run /agent-design for agent "<name>" only.
DONE when: <…>
FAILED when: <…>
Parent split context (read-only): <one-line list of sibling agents>.
Skip mega-job smell test; pillars + filled spec only.
Write the completed spec to .scratch/agent-design/<name>.md
```

### When *not* to use this skill

- You already have a locked one-job agent and only need to implement it.
- You want a whole-product roadmap (use a planning/wayfinding skill instead).
- Optional: if you use [wayfinder](https://github.com/mattpocock/skills) for large
  multi-decision efforts, chart there first — then return here to design each agent.
  This skill does **not** vendor or require wayfinder.

### Failure modes

- **Mega-agent:** multiple DONEs in one spec → split, then fan-out or re-enter.
- **Premature pillars:** filling Task before the smell test → rewind to step 3.
- **Silent fan-out:** spawning without listing the split and getting **yes** → always
  surface the list and wait for confirm.
- **Fake parallelism:** claiming sessions launched when no Task/`claude --bg`/equivalent
  ran → say so and fall back to park list.

## 1. Task — one job, with a bright line for *done* and *failed*

An agent needs to know when it's finished **and** when it has failed, without you. The
#1 failure mode is unbounded scope ("fix everything") — it never converges and you can't
tell success from drift.

- Give it one job, stated as an optimization target or a deliverable, not a vibe.
- Define the done condition and the failure condition explicitly.
- **Diagnostic:** *Can the agent tell, on its own, when it's done and when it's failed?*

## 2. Context — the minimum that makes the right move obvious

Context engineering beats prompt engineering. Hand the agent exactly the map it needs —
no more (burns tokens, dilutes attention), no less (it guesses).

- Prefer a precise, structured hand-off over a document dump.
- Decide what's **preloaded** vs fetched **just-in-time**.
- **Diagnostic:** *What's the smallest context that makes the right action obvious — and
  is each piece preloaded or fetched on demand?*

## 3. Memory — what persists across runs, with a write/read/supersede discipline

Memory is not a junk drawer.

- Decide what *must* persist between runs vs what's ephemeral to one run.
- Define the **write**, **read**, and **supersession** rules. Supersession is often a
  *safety* property — a stale fact that should have been overridden is a bug.
- Audience-scope memory: who/what is allowed to see each piece.
- **Diagnostic:** *What must it remember, who may see it, and what makes a new fact
  override an old one?*

## 4. Eval — how you know it's good, and how it improves

An agent without an eval is a vibe; an agent with one is a system.

- Deterministic agents: tests / assertions.
- Generative agents: a **rubric over a gold set**. A **bright line** can be merge-blocking.
- Prefer an *independent* grader from the generator.
- **Diagnostic:** *What rubric or test tells you this agent regressed?*

## Patterns that recur in good agents

- **Propose-then-confirm for anything that mutates.**
- **Reversible + audited.**
- **Scoped tools (least privilege).**
- **Guardrails against known failure modes.**
- **Eval-as-contract for fixers** — failing test = definition of done.

## The fix for "good by feel"

1. Run the **decision map** (section 0) until you have one outcome (or a split list).
2. On a split: design one here; **confirm** before fan-out (§0b).
3. Fill the agent spec + pre-flight checklist in `reference/agent-spec-template.md`.

## Worked example (product-agnostic)

User wants: "an agent that intakes support tickets, drafts a reply, and posts to Slack."

Decision map smell test:

| Agent | DONE |
|---|---|
| ticket-intake | structured ticket object |
| reply-drafter | draft meeting tone rubric |
| slack-poster | message posted (propose→confirm) |

Independent DONEs → three agents. This session designs **ticket-intake**. Ask:
“Spawn `reply-drafter` and `slack-poster` in parallel now?” — on **yes**, launch two
background sessions with the prompt template; on **no**, print the park list.
