---
name: agent-design
description: >
  How to build an AI agent that actually works — start with a decision-map that forces a
  singular outcome (or splits into multiple agents), then lock Task · Context · Memory · Eval.
  Use when designing, building, reviewing, or debugging an LLM agent, subagent, tool-using
  assistant, or skill: scoping what one agent should do, deciding what context it gets,
  designing memory across runs, safe write authority, or why an agent is flaky. Turns
  "good at building agents by feel" into "good on purpose, every time." For the fill-in
  template see reference/agent-spec-template.md.
---

# Agent design — decision map, then four pillars

Most "I'm not good at building agents" is really "I build them by feel and can't
predict whether the next one will be good." The fix is to name the method.

A working agent is four things, and a weak agent is usually missing one:

> **Task · Context · Memory · Eval.**

But first: **one agent = one outcome.** If you can name two outcomes that succeed or
fail independently, you have two agents — run this skill once per agent.

## 0. Decision map — singular outcome or split

Do this **before** filling the pillars. Completion criterion: either (a) one singular
DONE/FAILED pair locked, or (b) an explicit split list of agents each with their own DONE,
and this session continues with only one of them.

### Steps

1. **Draft the singular outcome** in one sentence (what "this agent worked" means).
2. **List aspects** that feel like separate focuses (intake, summarize, notify, …).
3. **Smell test each aspect:** same DONE/FAILED as the draft outcome?
   - Same → keep inside this agent.
   - Independent success/fail → **split** into a separate agent name + its DONE.
4. **Emit one of:**
   - **One agent** — proceed to pillars below for that outcome.
   - **Multiple agents** — list them; **park** all but one; tell the user to re-enter
     `/agent-design` for each parked agent in its own session. Do **not** auto-spawn
     agents without confirm.
5. **Fog vs size:**
   - One outcome, still foggy (open decisions) → stay in the decision map; resolve
     open questions one at a time until the outcome is sharp.
   - One outcome, too big for one run → still **one** agent design; use handoff/memory
     across runs — do **not** invent a second agent unless the outcome splits.

### When *not* to use this skill

- You already have a locked one-job agent and only need to implement it.
- You want a whole-product roadmap (use a planning/wayfinding skill instead).
- Optional: if you use [wayfinder](https://github.com/mattpocock/skills) for large
  multi-decision efforts, chart there first — then return here to design each agent.
  This skill does **not** vendor or require wayfinder.

### Failure modes

- **Mega-agent:** multiple DONEs in one spec → split and re-enter.
- **Premature pillars:** filling Task before the smell test → rewind to step 3.
- **Silent fan-out:** spawning subagents without listing parked agents → always surface the list.

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
2. Fill the agent spec + pre-flight checklist in `reference/agent-spec-template.md`.

## Worked example (product-agnostic)

User wants: "an agent that intakes support tickets, drafts a reply, and posts to Slack."

Decision map smell test:

- Intake parse → DONE: structured ticket object
- Draft reply → DONE: draft text meeting tone rubric  
- Post to Slack → DONE: message posted  

Independent DONEs → **three agents** (or intake+draft as one if they share one DONE, Slack as propose-confirm mutator). This session designs **ticket-intake** only; park `reply-drafter` and `slack-poster` for later `/agent-design` runs.
