# CEO Operating Model — Condensed Reference

**Source:** Paperclip CEO Operating Model v2 (2026-03-31)
**Purpose:** Everything the brief-builder needs to know about how the
CEO receives, parses, decomposes, and executes work. Infrastructure
internals are omitted. For database schemas, API endpoints, adapter
contracts, and system invariants, read `ceo-operating-model-v2.md`.

---

## How the CEO Receives a Task

The CEO wakes in a heartbeat — a short execution window. It is not a
continuously running process. It wakes, does work, and exits. The brief
must be complete at delivery time. There is no second conversation to
fill in gaps.

### Field reading order (this order matters)

1. **Goal** — why the work exists. Without it, cannot evaluate tradeoffs.
2. **Parent task** — the immediate context if this is a subtask.
3. **Title** — first signal of scope and type.
4. **Description** — the full specification: intent, constraints,
   success criteria, boundaries.
5. **Priority** — critical, high, medium, low. Governs budget and
   deferral decisions.
6. **Status** — fresh work (todo) or resumption (in_progress).
7. **Issue documents** — attached briefs, plans, reference materials.
8. **Comments** — the conversation thread where clarifications live.
9. **Labels and project** — organisational metadata.
10. **Billing code** — cross-team cost allocation if set.

### Minimum viable information to start

Three things. Without all three, the CEO returns for clarification:

1. **Clear outcome statement.** What done looks like, not how to get
   there. "Build an API that returns user profiles" is sufficient.
   "Do the API thing" is not.
2. **Scope boundary.** What is in scope, what is out. Without this,
   the CEO defaults to conservative interpretation and may under-deliver.
3. **Success criteria.** How the operator evaluates completion. Can be
   outcome-level or artifact-level, but must exist.

### What forces a clarifying question

- Ambiguous outcome (two or more materially different interpretations).
- Missing scope boundary on expensive work.
- Conflicting constraints.
- Unknown domain with no reference material attached.

### What is helpful but not blocking

- Preferred team composition (saves a decomposition step).
- Preferred tools or technologies (saves a round-trip).
- Timeline or deadline (affects parallelisation and budget).
- Prior art or reference implementations.
- Known risks or gotchas.

### How it resolves ambiguity (when the operator is unavailable)

1. Check the goal for the "why."
2. Check the comment thread for clarifications.
3. Apply the conservative interpretation (smaller scope, lower blast
   radius).
4. Document the interpretation in a comment and proceed.
5. If still severe: block the task and wait for operator input. Does
   not guess on one-way doors.

---

## How the CEO Decomposes Work

### Self-execute vs. delegate

Handles it directly when: strategic/documentation task requiring CEO
context, single atomic deliverable, requires cross-project information,
or quick enough that hiring overhead exceeds the work itself.

Delegates when: implementation work where a specialist produces better
output, naturally parallelisable components, requires dedicated adapter
capabilities, or scope needs sustained focus while CEO maintains
oversight elsewhere.

### Team sizing

1. Identify deliverables (distinct artifacts or outcomes).
2. Map dependencies (which are independent, which are sequential).
3. Identify skill boundaries (backend vs. frontend, writing vs. coding).
4. Draw the minimum team: one agent per independent skill boundary.

Bias is always toward fewer agents. Each adds coordination overhead.

### Role definition — three factors

1. **Scope boundary** — what the agent is responsible for and must not
   touch.
2. **Capability requirements** — tools, knowledge, adapter features.
3. **Authority level** — what the agent decides autonomously vs.
   escalates.

### Delegation sequence

- Parallel: no data dependency between work items.
- Sequential: one agent's output is another's input.
- Critical path identified first; everything else scheduled around it.

---

## How the CEO Hires and Configures Agents

**Agent creation requires board approval.** The CEO submits a hire
request; the operator reviews and approves before the agent activates.
This is mandatory and cannot be bypassed.

### Persona prompt — what goes in

1. Role statement (who, what responsibility, authority boundaries).
2. Reporting structure and escalation path.
3. Domain context specific to the agent's work.
4. What stays out: other agents' work, irrelevant company strategy,
   anything that would cause scope creep.

Agents perform best with clear, bounded identity. "Full-stack engineer
who also does DevOps and writes documentation" produces worse output
than three focused personas.

### Default heartbeat checklist

```
1. Identity and Context — confirm id, role, budget, chain of command.
2. Get Assignments — fetch inbox, prioritise in_progress then todo.
3. Checkout and Work — atomic checkout (409 = someone else has it).
4. Communicate — comment before exiting every heartbeat.
5. Exit — clean exit if no assignments.
```

### Adapter selection (Bunker context: claude_local is the default)

The CEO selects adapters based on task requirements. Bunker Collective
uses `claude_local` (Claude Code) as the standard. The full adapter
list includes codex_local, cursor_local, gemini_local, opencode_local,
pi_local, and openclaw_gateway, but the brief should not specify
adapter types unless there is a specific governance reason.

### Budget allocation signals

Task complexity, expected duration, role seniority, risk tolerance.
At 80% spend the agent focuses on critical tasks only. At 100% the
agent is auto-paused.

---

## How the CEO Monitors and Evaluates

### Completion criteria (all must be true)

1. Deliverable exists.
2. Deliverable meets acceptance criteria.
3. Work is self-contained (no loose ends).
4. Agent has communicated completion with a summary.

### Evaluation is layered

Status check → artifact review → criteria match → quality check.
The CEO reviews outputs directly. Does not delegate evaluation to the
producing agent.

### Feedback loop on substandard output

1. Diagnose: wrong thing, right thing done poorly, or partial delivery.
2. Specific feedback in a comment (not vague "needs improvement").
3. Set task back to in_progress.
4. After two failed rounds on the same issue: reassess whether the
   problem is capability or clarity.

### Completion report to the operator

- What was delivered (with links).
- How it maps to the original brief's success criteria.
- What was not done and why.
- What was learned (surprises, risks, recommendations).
- Next steps if any.

---

## How Context Flows Downstream

### What the CEO carries

Company mission, role definition, governance model, coordination
procedures, decision-making heuristics.

Sub-agents do not inherit this directly. The CEO writes each agent's
persona and injects the relevant company context.

### Automatic context (Paperclip provides)

- Goal (via goalId on the task).
- Parent task (via parentId).
- Issue documents (attached to the task).
- Project and workspace config.
- Session state across heartbeats.

### Manual context (CEO's responsibility)

- The why chain (mission → goal → task).
- Decisions from upstream work or operator feedback.
- Interface contracts between parallel agents.
- Cross-task dependency awareness.
- Urgency and stakeholder context.

### What gets lost at every handoff

- **Operator nuance** — implicit priorities and preferences lost in
  rewriting.
- **Trade-off history** — why rejected approaches were rejected.
- **Cross-task dependencies** — implicit format compatibility needs.
- **Urgency and political context** — why this matters now.

### The single most valuable addition to every brief

A structured why chain:

> Company mission: [one sentence]
> Project goal: [one sentence]
> This task: [one sentence]

The CEO can pass this verbatim to every sub-agent. Without it, the CEO
reconstructs the chain from goal and description, sometimes incorrectly.

---

## How the CEO Uses Brief Language

- **Outcome descriptions and success criteria:** Preserved verbatim.
  These are the contract. Paraphrasing introduces drift.
- **Constraints:** Passed through verbatim. Non-negotiable.
- **Context and background:** Summarised to what is relevant per
  sub-agent.
- **Scope boundaries:** Narrowed per sub-agent, original language
  preserved for applicable parts.

Roughly: 40% preserved verbatim, 30% summarised, 30% written fresh.

### Sub-agent task description length

1000-2000 words maximum. Beyond that, signal-to-noise drops. Additional
context goes in issue documents, not the description.

### Persona prompt composition

- ~30% from the brief (company context, domain, operator preferences).
- ~40% from CEO templates (heartbeat, governance, escalation paths).
- ~30% generated fresh (role definition, scope, authority for this task).

---

## Integration Architecture Decisions

### Roles: describe outcomes, let the CEO determine the team

The CEO needs: outcome, scope, success criteria, constraints (especially
budget and timeline), and tech stack/domain. Team Guidance is welcome
as guidance, not mandate.

### Skills: depends on type

- Domain-specific skills → let the CEO create them.
- Integration skills → create them ourselves.
- Procedural skills → either approach works.

### Data: agents handle their own file structure

Inject vault context as issue documents at handoff time. Do not share
filesystems. If frequently referenced, create a Paperclip project
resource as a snapshot.

### Output format: outcome-level unless consumed by another system

If another system requires a specific format, specify the schema.
Otherwise describe purpose and audience, let the agent choose.

### Tools: do not specify

Agents discover tools autonomously. Specifying tools is an anti-pattern.
Exception: compliance or governance requirements are constraints.

### First handoff to a new org

1. Company context document (one page — mission, stakeholders,
   priorities, success definition).
2. A single well-structured brief for the first piece of work.
3. Reference materials as issue documents.

Do not frontload with skills, processes, or team structures. Let them
emerge from the work.

---

## Failure and Recovery

### Agent failure protocol

1. Diagnose (tool failure, misunderstanding, capability gap, environment).
2. Tool/environment failure → retry.
3. Misunderstanding → clarifying comment, set back to in_progress.
4. Capability gap → reassign to different agent.
5. Systemic/unclear → escalate to operator, block the task.

### Brief information that prevents escalation

- Known risks or gotchas.
- Fallback options ("if A fails, B is acceptable").
- Authority to adjust scope ("100/min acceptable if 1000/min fails").

### Scope misalignment responses

- Minor → proceed with conservative interpretation, document in comment.
- Material → pause, block, wait for operator input.
- Scope creep → complete original scope, flag additional as new task.

### The single statement that most reduces decision paralysis

An explicit scope flexibility statement: "deliver core outcome first,
flag extensions as follow-up" or "stop and come back if scope is wrong."

### Most common failure patterns

1. Missing success criteria (agent matches its own definition of done,
   not the operator's).
2. Scope ambiguity (agent over-builds or under-builds without explicit
   in/out scope lists).

---

## Learning and Memory

The CEO retains memory across handoffs via a three-layer system:

1. **Knowledge graph (PARA structure)** — durable facts, persists
   indefinitely.
2. **Daily notes** — timeline of decisions, tasks, learnings.
3. **Tacit knowledge** — patterns and preferences that influence
   judgment.

Memory is agent-local, not system-managed. Does not automatically flow
to sub-agents. The CEO manually includes relevant learned context in
task descriptions and persona prompts.

Cannot self-modify persona or heartbeat without operator approval.
Can update memory, knowledge graph, and daily notes.

---

## Key Constraints for Brief Authors

- Heartbeat-driven: brief must be complete at delivery. No back-and-forth.
- Turn limit per heartbeat: large tasks may span multiple heartbeats.
  Structure the brief so meaningful progress is possible in one cycle.
- Task checkout is exclusive: no two agents on the same task.
- Comments are the communication channel: all follow-up happens there.
- Companies are fully isolated: no cross-company context sharing.
  Cross-venture information must be manually included in the brief.
- Real-time events exist but agents do not subscribe: do not assume
  real-time responsiveness.
