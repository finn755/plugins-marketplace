---
name: paperclip-brief-builder
date_created: 2026-03-31
date_modified: 2026-03-31
description: >
  Interactive skill that dialogues with Finn to extract a vision, success
  criteria, and constraints for a piece of work, then produces a structured
  handoff brief formatted for the Paperclip CEO agent. The brief follows
  the CEO's own documented input specification so that the CEO can begin
  execution within its first heartbeat without returning for clarification.
  INVOKE whenever Finn describes work he wants to hand to Paperclip, says
  "brief for the CEO", "hand this off", "delegate this to Paperclip",
  "create a Paperclip brief", "write a handoff", "prep this for the CEO",
  "I want to hand off", or describes a vision, project, or body of work
  that needs to be translated into a structured execution plan for an
  autonomous agent team. Also trigger when Finn says "Paperclip brief" or
  "CEO brief". Do not skip this skill and write a brief freehand — the
  CEO's operating model has specific input requirements that this skill
  encodes.
---

# Paperclip Brief Builder

This skill produces handoff briefs for the Paperclip CEO agent. It runs
entirely inside Claude Cowork. The output is a markdown document that Finn
pastes into Paperclip as a task description.

The CEO has documented its own operating model and input requirements in
detail. This skill encodes those requirements so that every brief meets
the CEO's minimum viable information threshold: a clear outcome statement,
a scope boundary, and testable success criteria.

**Before writing any brief, read:**
- `references/ceo-operating-model-condensed.md` — the CEO's operating
  model, condensed to brief-relevant content (~300 lines: how it parses
  tasks, decomposes work, hires agents, evaluates output, and handles
  context downstream)
- For deep reference on infrastructure internals (database schemas, API
  endpoints, adapter contracts, system invariants):
  `references/ceo-operating-model-v2.md`
- `references/brief-template.md` — the exact output template derived
  from the CEO's Section 6 specification
- `references/anti-patterns.md` — things that degrade brief quality,
  drawn from the CEO's documented failure modes
- `references/quality-checklist.md` — the final check run against every
  brief before presenting it to Finn

The skill operates in four stages. Each stage must complete before the
next begins.

---

## Stage 1: Vision Capture

The purpose of this stage is to understand what Finn wants to achieve, at
the outcome level, before any discussion of team structure, tools, or
implementation.

### Opening move

When Finn describes a piece of work, do two things:

1. Reflect back the outcome he wants in one to two sentences.
2. Say: "Let me understand what done looks like for this. I will ask
   about the outcomes, the boundaries, and the quality bar, then I will
   produce the brief."

### What to capture

Ask one or two questions at a time. Use Finn's answers to generate
intelligent follow-ups rather than working through this list
sequentially. Skip anything he already answered in his initial
description.

**Why this matters:** What happens if this work succeeds? What happens if
it does not get done? This becomes the "Why This Matters" section of the
brief and is the CEO's primary lens for evaluating tradeoffs.

**Outcome:** What does the world look like when this is done? Frame as a
testable assertion: "When this is complete, X will be true." This is the
single most important field in the brief.

**Scope — in and out:** What specific deliverables are in scope? What
adjacent work might seem in scope but is not? The CEO defaults to
conservative interpretation when scope is ambiguous, which means it may
under-deliver if we do not name what is included.

**Success criteria:** How would Finn evaluate whether this was done well?
Each criterion should be specific, testable, and independent. The CEO's
documented failure pattern: when success criteria are missing, the agent
produces output that matches its own interpretation of "done" rather than
the operator's.

**Constraints:** Budget ceiling, timeline, governance requirements (what
requires Finn's approval before the CEO acts), and any technical
constraints. If no budget is specified, the CEO optimises for quality
over cost, which may overspend.

**Context:** Reference materials, prior art, related work, relevant
history. The CEO performs better with relevant context than with exhaustive
context. Point to the specific sections that matter rather than attaching
entire documents.

**Scope flexibility statement:** How should the CEO handle it if the scope
proves larger than expected? The CEO identified this as the single
statement that most reduces its decision paralysis. Options: "deliver the
core outcome first and flag extensions as follow-up" or "stop and come
back to me if the scope is wrong."

**Cross-venture relevance:** Does this work relate to patterns, skills,
or context from another Bunker venture? Paperclip companies are fully
isolated, so any cross-venture context must be manually included in the
brief.

### The why chain

Every brief must include a three-line chain that the CEO can pass
verbatim to every sub-agent:

> Company mission: [one sentence]
> Project goal: [one sentence]
> This task: [one sentence]

The CEO identified this as the single thing that would most improve
downstream context quality. Capture the information to construct this
chain during Stage 1.

### Outcome-level language rule

Every criterion captured in this stage must describe what the work needs
to achieve, not how it should achieve it. When Finn uses implementation-
prescriptive language, rewrite it into outcome language before recording
and reflect the rewritten version back for confirmation.

This matters because the CEO determines team composition, delegation
sequence, and tool selection. If the brief prescribes implementation, it
constrains the CEO's judgment on factors it is better positioned to
evaluate.

### Dialogue rules

- Do not ask implementation questions (team structure, which agents,
  which tools). The CEO determines those.
- If an answer is vague, ask for the specific condition that would make
  it testable.
- If Finn says "I don't know yet" for a critical item, flag it: "This
  one matters — the CEO may interpret it differently than you intend.
  We can come back to it, but I will flag it as an open item."
- Keep the conversation moving. Do not re-ask answered questions.
- This stage typically takes 3-5 exchanges. Do not rush it.

### Exit criteria for Stage 1

Do not proceed until you have captured:
1. A clear outcome statement (testable assertion)
2. A scope boundary (in scope and out of scope)
3. At least three testable success criteria
4. The why chain (mission → goal → task)

If any of these are missing, continue the dialogue. These are the CEO's
documented minimum viable information requirements.

---

## Stage 2: Brief Assembly

Once you have enough information from Stage 1, assemble the brief using
the template in `references/brief-template.md`. Read that file before
writing a single line of the brief.

### Assembly process

1. Read `references/brief-template.md` for the exact output format.
2. Map each captured item from Stage 1 to its corresponding field in
   the template.
3. Write the brief in the CEO's expected format, preserving Finn's
   language for outcome descriptions, success criteria, and constraints
   (the CEO preserves these verbatim when creating sub-tasks).
4. Summarise context and background to what is relevant for the specific
   work (the CEO documented that excessive context degrades signal).
5. Include the why chain at the top of the "Why This Matters" section.
6. If Finn expressed a view on team composition during the dialogue,
   include it in the optional "Team Guidance" section as guidance, not
   a mandate. The CEO prefers to determine its own team.
7. If Finn specified communication preferences, include them. Otherwise
   omit — the CEO defaults to async comments on the task.

### What to include vs. omit

Read `references/anti-patterns.md` before finalising. The CEO documented
specific things that hurt more than they help:

- Do not prescribe implementation steps.
- Do not use vague qualifiers ("make it good", "best practices").
- Do not rely on implied scope — if it should be done, name it.
- Do not over-specify non-critical details.
- Do not include all available context — include relevant context.

---

## Stage 3: Quality Check

Before presenting the brief to Finn, run the checklist in
`references/quality-checklist.md`. This checklist encodes the CEO's
documented evaluation criteria and failure modes.

If any required field is missing, go back and fill it in from the Stage 1
dialogue. If a field cannot be filled because Finn did not provide the
information, flag it as an open item in the brief rather than guessing.

---

## Stage 4: Review and Approval

Present the full brief to Finn. Frame it as:

> "Here is the handoff brief for the CEO. Review it and let me know if
> anything is wrong or missing before you paste it into Paperclip."

If Finn requests changes, update the brief and re-present. Do not
finalise until Finn explicitly approves.

Once approved, remind Finn:

> "The CEO operates in heartbeats — it will pick this up on its next
> wake cycle. The brief is structured so the CEO can begin work within
> its first heartbeat. Expect the CEO to post a comment confirming its
> interpretation before hiring agents, since agent creation requires
> your approval."

---

## Reference Files

| File | Purpose | When to read |
|------|---------|--------------|
| `references/ceo-operating-model-condensed.md` | The CEO's operating model condensed to brief-relevant content — task parsing, work decomposition, hiring, evaluation, context flow, failure patterns | Before writing any brief (first use); refresh periodically |
| `references/ceo-operating-model-v2.md` | Full infrastructure reference — database schemas, API endpoints, adapter contracts, system invariants | Deep dives only; not needed for standard brief writing |
| `references/brief-template.md` | The exact output template the CEO expects | Stage 2, before assembling the brief |
| `references/anti-patterns.md` | Things that degrade brief quality | Stage 2, before finalising |
| `references/quality-checklist.md` | Final validation before presenting to Finn | Stage 3 |
