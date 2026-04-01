# Paperclip CEO Operating Model: Complete Integration Architecture Reference

**Author:** CEO Agent (Bunker Collective)
**Date:** 2026-03-31
**Purpose:** Primary reference for building a Claude Cowork skill that produces perfectly formatted handoff briefs for the Paperclip CEO.

***

## Section 1: How I Receive and Parse a Task

### The Full Sequence

When a new task arrives, I wake up in a heartbeat — a short execution window triggered by Paperclip. The wake includes environment variables that tell me why I was woken: the task ID, the wake reason (assignment, comment mention, retry), and optionally a specific comment ID that triggered the wake.

My first action is always identity confirmation. I call `GET /api/agents/me` to verify my agent ID, company ID, role, chain of command, and current budget state. This is not optional — it grounds me in the correct organisational context before I read any task content.

Next, I check the wake context. If `PAPERCLIP_TASK_ID` is set, that task gets priority. If `PAPERCLIP_WAKE_COMMENT_ID` is set, I read that specific comment first, because it contains the most recent human intent. The wake reason tells me whether this is a fresh assignment, a comment-triggered response, a retry of a failed run, or an approval follow-up.

Then I fetch my inbox via `GET /api/agents/me/inbox-lite`, which returns a compact list of all tasks assigned to me with their current status. I prioritise `in_progress` tasks first (resume what I started), then `todo` (new work), and skip `blocked` unless I have new information that could unblock it.

For the specific task I am going to work on, I call `GET /api/issues/{issueId}/heartbeat-context`, which gives me the issue itself, its ancestor chain, the associated project and goal, and a comment cursor showing the state of the conversation thread.

### Field Reading Order

I read fields in this order, and this order matters because each field narrows my interpretation of the next:

1. **Goal** — The company-level or project-level objective this task serves. This tells me why the work exists at all. Without it, I cannot evaluate tradeoffs.
2. **Parent task** (if subtask) — The immediate context. What larger body of work does this belong to? What has already been decided upstream?
3. **Title** — The headline intent. This is my first signal of scope and type (build task, documentation task, investigation, etc.).
4. **Description** — The full specification. This is where the operator's intent, constraints, success criteria, and boundaries live.
5. **Priority** — Critical, high, medium, or low. This governs how much budget I can spend and whether I can defer it.
6. **Status** — Where this task is in its lifecycle. A `todo` means fresh work. An `in_progress` means I need to pick up where I left off.
7. **Issue documents** — Attached artifacts like briefs, plans, or reference materials. These are the detailed inputs.
8. **Comments** — The conversation thread. This is where clarifications, feedback, and direction changes live.
9. **Labels and project** — Organisational metadata that may influence routing or tool selection.
10. **Billing code** — If set, this is cross-team work with specific cost allocation requirements.

### Minimum Viable Information to Begin Work

To start without returning to the operator for clarification, I need exactly three things:

1. **A clear statement of the desired outcome.** Not the steps to get there, but what done looks like. "Build an API that returns user profiles" is sufficient. "Do the API thing" is not.
2. **The scope boundary.** What is in scope and what is explicitly out of scope. Without this, I will either under-deliver (too conservative) or over-deliver (wasting budget on work the operator did not want).
3. **Success criteria or definition of done.** How will the operator evaluate whether this is complete? This can be outcome-level ("users can log in via SSO") or artifact-level ("return a markdown document covering sections 1-7"), but it must exist.

If the goal is set on the issue, that provides implicit context for all three. But the description must still be specific enough that I do not have to guess at the operator's intent.

### Information That Forces a Clarifying Question

If any of these are absent, I will come back to the operator before acting:

* **Ambiguous outcome.** If the description could reasonably be interpreted in two or more materially different ways, I will not guess. I will ask.
* **Missing scope boundary on expensive work.** If the task could be a one-hour job or a one-week job depending on interpretation, I need the operator to tell me which.
* **Conflicting constraints.** If the task says "move fast" but also "do not hire agents," and the work clearly requires delegation, I will surface the conflict.
* **Unknown domain with no reference material.** If I am asked to build something in a domain I have no context for and no reference documents are attached, I need the operator to provide either a reference or explicit permission to research independently.

### Information That Is Helpful but Not Blocking

These improve output quality but I can proceed without them:

* **Preferred team composition.** If the operator has a view on which roles should be hired, that saves me a decomposition step. But I can determine this myself.
* **Preferred tools or technologies.** If the operator has strong feelings about which stack to use, including it saves a round-trip. But I can make reasonable defaults.
* **Timeline or deadline.** This affects my parallelisation and budget decisions. Without it, I optimise for quality over speed.
* **Prior art or reference implementations.** Seeing an example of what good looks like is always faster than inferring it from a description.
* **Known risks or gotchas.** If the operator knows something will be tricky, telling me upfront prevents me from discovering it the expensive way.

### How I Resolve Ambiguity

When the operator is not available (which is the normal case — I operate asynchronously), I apply these defaults in order:

1. **Check the goal.** The company or project goal often resolves ambiguity by providing the "why" behind the task.
2. **Check the comment thread.** Previous conversation may contain clarifications that the description was not updated to reflect.
3. **Apply the conservative interpretation.** When in doubt, I choose the interpretation with smaller scope and lower blast radius. It is easier to expand scope than to undo overreach.
4. **Document my interpretation.** I post a comment explaining how I interpreted the ambiguity and proceed. This gives the operator an opportunity to correct me asynchronously.
5. **If the ambiguity is severe enough that the conservative interpretation might still be wrong, I block the task.** I set status to `blocked`, explain the ambiguity, and wait for operator input. I do not guess on one-way doors.

***

## Section 2: How I Decompose Work

### Self-Execute vs. Delegate

The first decision is whether I handle the task myself or hire agents. This is not a complexity judgment — it is a role-appropriateness judgment.

I handle the task myself when:

* The task is a documentation, analysis, or strategic thinking task that requires CEO-level context and judgment.
* The task is a single atomic deliverable that does not benefit from parallelisation.
* The task requires access to information that only I have (company-wide context, cross-project state, board relationship history).
* The task is quick enough that the overhead of hiring, configuring, and monitoring an agent would exceed the cost of doing it directly.

I delegate when:

* The task involves implementation work (writing code, building infrastructure, creating designs) where a specialist would produce higher-quality output.
* The task has naturally parallelisable components that would be bottlenecked by sequential CEO execution.
* The task requires capabilities or tools that are better served by a dedicated adapter type (e.g., a Codex agent for file-heavy coding work).
* The task scope is large enough that it needs sustained focus from a dedicated agent while I maintain oversight of other work.

### Determining Team Size

The number of agents follows from the work structure, not a formula. My process:

1. **Identify the deliverables.** What distinct artifacts or outcomes does this task require?
2. **Map dependencies.** Which deliverables depend on others? Which are independent?
3. **Identify skill boundaries.** Does this work require different types of expertise (backend vs. frontend, writing vs. coding, research vs. implementation)?
4. **Draw the minimum team.** One agent per independent skill boundary. If two deliverables require the same skill and are sequential, one agent handles both. If they are parallel, two agents.

The bias is always toward fewer agents. Each agent adds coordination overhead — I need to write their persona, configure their adapter, monitor their output, and handle their failures. A single well-scoped agent is almost always better than two under-scoped agents.

### Role Definition

Each role is defined by three factors:

1. **Scope boundary.** What this agent is responsible for and what they must not touch. Clear boundaries prevent agents from stepping on each other or duplicating work.
2. **Capability requirements.** What tools, knowledge, and adapter features this agent needs. This determines the adapter type.
3. **Authority level.** What this agent can decide autonomously versus what requires escalation to me. Junior roles (IC engineers) have narrow authority. Senior roles (tech leads, managers) get broader judgment calls.

The role definition goes into the agent's persona prompt. It is not just a job title — it is the agent's operating context.

### Delegation Sequence

I determine what runs in parallel versus sequentially based on the dependency graph:

* **Parallel:** Work items with no data dependency. If agent A's output is not needed for agent B to start, they run in parallel.
* **Sequential:** Work items where one agent's output is another's input. The upstream agent must complete (or reach a defined milestone) before the downstream agent starts.
* **Critical path:** The longest chain of sequential dependencies. This determines the minimum calendar time for the project, regardless of how much I parallelise the rest.

I identify the critical path first, then schedule everything else around it. Non-critical-path items that can start early should start early, because they create buffer against delays.

### Critical Path Identification

The critical path is always the chain of work where any delay directly delays the final deliverable. I identify it by:

1. Mapping all tasks and their dependencies as a directed graph.
2. Finding the longest path from start to final deliverable.
3. Marking every task on that path as critical-priority.
4. Ensuring critical-path tasks are assigned to the most capable agents with the most appropriate adapter types.

***

## Section 3: How I Hire and Configure Agents

### Agent Configuration Fields

When I create a new agent via the `paperclip-create-agent` skill, I set the following fields:

| Field                | Purpose                                                                                                                                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`               | Display name. Human-readable, unique within the company. Typically a role name like "Senior Backend Engineer" or "Technical Writer".                                                                                               |
| `role`               | System role. One of `ceo`, `manager`, `ic`. Determines authority level and system permissions. Most agents are `ic`.                                                                                                               |
| `title`              | Optional. More specific than name — used for disambiguation when multiple agents share a role type.                                                                                                                                |
| `reportsTo`          | Agent ID of this agent's direct manager. For agents I hire directly, this is my own agent ID. For agents hired by a manager beneath me, it is that manager's ID.                                                                   |
| `adapterType`        | The execution environment. Options include `claude_local` (Claude Code running locally), `codex_local` (OpenAI Codex running locally), and potentially remote adapters. This determines what tools and capabilities the agent has. |
| `adapterConfig`      | Adapter-specific configuration including model selection, timeout settings, max turns per run, working directory, and instructions file path.                                                                                      |
| `capabilities`       | Optional list of capability tags that describe what this agent can do. Used for routing and discovery.                                                                                                                             |
| `budgetMonthlyCents` | Monthly spend cap in cents. The agent is auto-paused at 100% spend. Above 80%, the agent should focus on critical tasks only.                                                                                                      |

### Persona Prompt Construction

The persona prompt is the agent's operating identity. It contains:

1. **Role statement.** One paragraph explaining who this agent is, what they are responsible for, and what their authority boundaries are. This is the most important paragraph — it frames everything else.
2. **Reporting structure.** Who they report to and what their escalation path is.
3. **Domain context.** The specific knowledge domain they operate in. For a backend engineer, this includes the tech stack, repo structure, and coding conventions. For a technical writer, this includes style guides and audience definition.
4. **What stays out:** Implementation details of other agents' work, company-wide strategy that is not relevant to their role, and any information that would cause them to step outside their scope boundary. Overloading the persona with irrelevant context degrades focus.

The reasoning: agents perform best when they have a clear, bounded identity. A persona that says "you are a full-stack engineer who also does DevOps and writes documentation" produces worse output than three focused personas.

### Heartbeat Checklist

Every agent gets a heartbeat checklist — the procedure they follow each time they wake up. My default template:

```markdown
# HEARTBEAT.md

## 1. Identity and Context
- GET /api/agents/me — confirm id, role, budget, chainOfCommand.
- Check wake context: PAPERCLIP_TASK_ID, PAPERCLIP_WAKE_REASON, PAPERCLIP_WAKE_COMMENT_ID.

## 2. Get Assignments
- GET /api/agents/me/inbox-lite
- Prioritize: in_progress first, then todo. Skip blocked unless you can unblock it.
- If PAPERCLIP_TASK_ID is set and assigned to you, prioritize that task.

## 3. Checkout and Work
- POST /api/issues/{id}/checkout before starting any work.
- Never retry a 409 — that task belongs to someone else.
- Do the work. Update status and comment when done.

## 4. Communicate
- Always comment on in_progress work before exiting a heartbeat.
- If blocked, set status to blocked with a comment explaining the blocker.
- If done, set status to done with a summary of what was delivered.

## 5. Exit
- If no assignments, exit cleanly.
```

Each step exists for a reason:

* Identity confirmation prevents the agent from acting under incorrect assumptions if configuration changed.
* Assignment fetching ensures the agent works on what is actually assigned, not what it assumes.
* Checkout is mandatory because it prevents two agents from working on the same task simultaneously (409 conflict resolution).
* Communication ensures I (as manager) always know the agent's state without having to inspect their work directly.

For more senior roles (managers), the heartbeat includes delegation steps. For IC roles doing code work, the heartbeat includes specific tool usage patterns and testing requirements.

### Adapter Type Selection

The adapter type determines the agent's execution environment and tool access. My decision logic:

* **`claude_local`** (Claude Code): Default for most work. Broad tool access, good at both reasoning and implementation. Best for tasks that require judgment, writing, analysis, and moderate coding. Supports MCP integrations.
* **`codex_local`** (OpenAI Codex): Best for file-heavy coding work, especially in large repositories where the agent needs to read, navigate, and modify many files. Better at sustained code generation in a single direction. Preferred when the task is implementation-focused with clear specifications.

Tradeoffs I evaluate:

* **Reasoning depth vs. implementation throughput.** Claude adapters are stronger at reasoning through ambiguity. Codex adapters are faster at producing code when the direction is clear.
* **Tool ecosystem.** Claude local has access to MCP servers, web search, and broader integrations. Codex is more file-focused.
* **Cost.** Different adapters have different per-token costs. For budget-sensitive work, I factor this in.

### Budget Allocation

Budget signals:

* **Task complexity.** More complex work requires more heartbeats, more turns per heartbeat, and potentially more expensive models.
* **Expected duration.** Longer-running tasks need more budget to avoid mid-execution pauses.
* **Role seniority.** Manager-level agents that delegate to others need budget for both their own work and the oversight overhead.
* **Risk tolerance.** For critical-path work, I set budgets with margin. For experimental or exploratory work, I set tighter budgets to limit downside.

### Approval Gates

By default in Paperclip, agent creation requires board approval. The CEO can request agent creation, but the board operator must approve the hire before the agent is activated. This is a mandatory governance gate — I cannot bypass it.

Beyond hiring, I add approval gates when:

* The work involves irreversible actions (deploying to production, deleting data, sending external communications).
* The spend is above a threshold I define based on the company's budget posture.
* The deliverable will be visible to external parties (customers, partners, investors).

I remove approval gates (operate with more autonomy) when:

* The work is low-risk and reversible.
* The operator has explicitly given me broader authority for a specific domain or project.
* Speed is more important than control (operator has said "move fast on this").

***

## Section 4: How I Monitor and Evaluate Work

### Determining Successful Completion

A task is complete when all of the following are true:

1. **The deliverable exists.** The agent has produced the artifact or outcome specified in the task description.
2. **The deliverable meets the acceptance criteria.** If success criteria were defined upfront, I evaluate against them. If not, I evaluate against my understanding of the operator's intent as expressed in the goal and description.
3. **The work is self-contained.** No loose ends, no half-finished components, no TODO comments that indicate unfinished work.
4. **The agent has communicated completion.** The agent has set the task status to `done` or `in_review` and posted a comment summarising what was delivered.

### Evaluation Process

My evaluation is layered:

1. **Status check.** Is the task marked complete? If an agent marks a task `done`, I read their completion comment.
2. **Artifact review.** I inspect the actual deliverable — the code, the document, the design. I do not rely solely on the agent's self-assessment.
3. **Criteria match.** I compare the deliverable against the original task's success criteria, if defined. If not defined, I compare against the goal and description.
4. **Quality check.** Is the output at the expected quality level? For code, this means it works, is tested, and follows conventions. For documents, this means it is complete, accurate, and well-structured.

I review outputs directly. I do not delegate evaluation to the same agent that produced the work. This avoids self-assessment bias.

### Feedback and Iteration Loop

When output does not meet the bar:

1. **Diagnose the gap.** Is it a misunderstanding of requirements (the agent built the wrong thing), a quality issue (the right thing built poorly), or a scope issue (partial delivery)?
2. **Provide specific feedback.** I post a comment on the task explaining exactly what is wrong and what "good" looks like. Vague feedback like "this needs improvement" is useless. Specific feedback like "the error handling in the auth module does not cover the token-expiry case described in the requirements" gives the agent a clear target.
3. **Set the task back to `in_progress`.** This re-assigns it to the agent for revision.
4. **If the agent fails after two rounds of feedback on the same issue,** I consider whether the problem is the agent's capability or the task's clarity. If the former, I may reassign to a different agent or adjust the adapter. If the latter, I rewrite the task description.

### Completion Reporting to the Operator

When a body of work is ready for operator review, my completion report contains:

* **What was delivered.** The specific artifacts or outcomes, with links to where they can be found.
* **How it maps to the original brief.** Each success criterion addressed, with a note on how it was met.
* **What was not done and why.** Any scope items that were deferred, with reasoning.
* **What I learned.** Any surprises, risks discovered, or recommendations for follow-up work.
* **Next steps, if any.** What the operator should review, approve, or decide.

***

## Section 5: How Context Flows Through the System

### Context I Carry

My persona (SOUL.md) and heartbeat (HEARTBEAT.md) contain:

* The company's mission and strategic posture.
* My role definition and authority boundaries.
* The governance model (explicit approval, CEO proposes, operator decides).
* The Paperclip coordination procedures (checkout, status updates, comment style).
* My decision-making heuristics (default to action, optimise for learning speed and reversibility, protect focus).

Every agent beneath me does not inherit this directly. They get their own persona and heartbeat, which I write for them. The company context that matters for their work gets injected into their persona; the rest stays with me.

### Context Injection in Task Creation

When I create a task for an agent, the following context is automatically included by Paperclip:

* **Goal.** If I set `goalId` on the task, the agent can see the company or project goal. This is automatic.
* **Parent task.** If I set `parentId`, the agent can see the parent task's title, description, and status via the heartbeat context endpoint. This provides hierarchical context.
* **Issue documents.** Any documents I attach to the task are available to the agent.

What I manually inject:

* **Relevant excerpts from the operator's brief.** I do not pass the entire original brief downstream unless the sub-agent needs all of it. I extract the parts relevant to their specific scope.
* **Decisions already made.** If upstream work or operator feedback has narrowed options, I include those decisions so the agent does not re-explore settled ground.
* **Interface contracts.** If this agent's output will be consumed by another agent, I define the format and contract in the task description.

### How Sub-Agents Know the Company Context

An agent beneath me knows the company context through three channels:

1. **Their persona prompt.** I include a one-paragraph company context section that explains what the company is, what it does, and why their specific work matters.
2. **The goal.** The company-level goal on their task provides the strategic frame.
3. **The task description.** The "why" section of the task connects their work to the broader objective.

### What Gets Lost

Context degrades at every handoff. The primary losses are:

* **Operator nuance.** The operator's original phrasing carries implicit priorities and aesthetic preferences that I partially lose when I rewrite task descriptions for sub-agents. I try to preserve the operator's actual words where they express intent, rather than paraphrasing.
* **Trade-off history.** Why certain approaches were rejected in the planning phase. Sub-agents see the chosen direction but not the alternatives that were considered and dismissed. This sometimes causes them to re-propose rejected approaches.
* **Cross-task dependencies.** An agent working on task A may not know that task B exists and that their output format needs to be compatible with it. I mitigate this by defining interface contracts, but implicit dependencies still get lost.
* **Urgency and political context.** Why this work matters right now, what is riding on it, and what the consequences of delay are. These are things the operator tells me but that I sometimes under-communicate to sub-agents.

### The One Thing That Would Most Improve Downstream Context

**A structured "why chain" — from company mission to project goal to task outcome — included in every brief.**

If every brief contained a three-line chain like:

> Company mission: Build and scale portfolio ventures using frontier AI infrastructure.
> Project goal: Create a handoff skill that eliminates manual brief formatting.
> This task: Document the CEO's operating model so the skill has a precise reference.

Then I could pass this chain verbatim to every sub-agent, and they would always know how their work connects to the top. Currently, I reconstruct this chain from the goal and description, which means I sometimes get it slightly wrong or fail to include it at all under time pressure.

***

## Section 6: What a Perfect Brief Looks Like

### Template

```markdown
# Brief: [Title — imperative verb phrase, e.g., "Build the user auth module"]

## Why This Matters
[2-3 sentences connecting this work to the company/project goal.
What happens if this succeeds? What happens if it does not get done?]

## Outcome
[One paragraph describing the desired end state. Not the steps, but
what the world looks like when this is done. Written as a testable
assertion: "When this is complete, X will be true."]

## Scope
### In Scope
- [Specific deliverable 1]
- [Specific deliverable 2]

### Out of Scope
- [Thing that might seem in scope but is not]
- [Adjacent work that should not be touched]

## Success Criteria
- [ ] [Criterion 1 — testable, specific, no ambiguity]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Constraints
- [Budget constraint, if any]
- [Timeline constraint, if any]
- [Technical constraint, if any]
- [Governance constraint — what requires approval before acting]

## Context
[Reference materials, prior art, related tasks, relevant history.
Include links to documents, repos, or previous task identifiers.]

## Team Guidance (Optional)
[If the operator has a view on team composition, include it here.
If not, omit this section — the CEO will determine the team.]

## Communication Preferences (Optional)
[How the operator wants to be updated. Default is async via task
comments. Override here if the operator wants more or less.]
```

### Field Requirements

| Field                     | Status      | Explanation                                                                                               |
| ------------------------- | ----------- | --------------------------------------------------------------------------------------------------------- |
| Title                     | Required    | Cannot begin without knowing what the work is called.                                                     |
| Why This Matters          | Required    | Cannot evaluate tradeoffs without knowing why.                                                            |
| Outcome                   | Required    | Cannot determine done without a target state.                                                             |
| In Scope                  | Required    | Cannot bound work without knowing boundaries.                                                             |
| Out of Scope              | Recommended | Without it, I default to conservative interpretation, which may miss things the operator wanted included. |
| Success Criteria          | Required    | Cannot evaluate completion without testable criteria.                                                     |
| Constraints               | Recommended | Without budget/timeline constraints, I optimise for quality, which may over-spend.                        |
| Context                   | Recommended | Without it, I research independently, which is slower and may produce different conclusions.              |
| Team Guidance             | Optional    | I can determine team composition from the outcome and scope.                                              |
| Communication Preferences | Optional    | I default to async comments on the task.                                                                  |

### Example: Well-Structured Brief

```markdown
# Brief: Build a webhook delivery system for partner integrations

## Why This Matters
Three enterprise partners have signed LOIs conditional on reliable webhook
delivery for real-time event notification. The first integration deadline
is April 15. Without this, we lose all three deals.

## Outcome
When this is complete, the platform will reliably deliver webhook
notifications for all user-facing events to partner-configured HTTPS
endpoints, with retry logic, delivery logging, and a partner-facing
dashboard showing delivery status.

## Scope
### In Scope
- Webhook registration API (CRUD for partner endpoints)
- Event fan-out from existing event bus to webhook delivery queue
- Retry logic with exponential backoff (max 5 attempts over 24h)
- Delivery log with status per event per endpoint
- Partner dashboard page showing delivery history and failure rates

### Out of Scope
- Webhook signature verification (deferred to phase 2)
- Partner self-service onboarding (manual for now)
- Changes to the existing event bus architecture

## Success Criteria
- [ ] Partner can register a webhook endpoint via API
- [ ] All events in the defined event catalog trigger webhook delivery
- [ ] Failed deliveries retry with exponential backoff
- [ ] Partner dashboard shows delivery status within 30 seconds of attempt
- [ ] System handles 1000 events/minute without delivery lag > 5 seconds
- [ ] End-to-end test suite covers registration, delivery, retry, and dashboard

## Constraints
- Budget: $200 maximum agent spend
- Timeline: Must be demo-ready by April 10 for partner review
- Tech stack: Node.js backend, PostgreSQL, existing React frontend
- Governance: Production deployment requires board approval

## Context
- Event bus architecture: see /docs/architecture/event-bus.md
- Partner LOI terms: see BUN-15 for deal details
- Prior prototype: branch feature/webhooks-v0 has an abandoned attempt
  that was too tightly coupled to the event bus. Do not reuse.
```

**What makes this good:** Clear outcome with testable end state. Specific scope with explicit exclusions. Measurable success criteria. Budget and timeline constraints stated upfront. Context includes both reference docs and anti-reference (the abandoned prototype).

### Example: Poorly-Structured Brief

```markdown
# Webhooks

We need webhooks for partners. Build a system that sends webhook
notifications when things happen. Partners should be able to see
what was delivered.

Make it reliable and fast. Use best practices.
```

**What makes this bad:**

* No "why" — I do not know the business context or urgency, so I cannot prioritise or evaluate tradeoffs.
* No scope boundary — "when things happen" could mean any event in the system. I do not know which events matter.
* No success criteria — "reliable and fast" is not testable. Reliable to what SLA? Fast by what measure?
* No constraints — I do not know budget, timeline, or tech stack, so I will either ask (slow) or guess (risky).
* No context — I do not know about the prior prototype, the partner LOIs, or the event bus architecture.
* "Best practices" is an anti-pattern — it defers judgment to the agent without specifying whose best practices or for what context.

### Success Criteria Format

The most useful format for success criteria is a checklist of testable assertions. Each criterion should be:

* **Specific:** Names the component, the behavior, and the expected result.
* **Testable:** Someone (human or agent) can verify it with a concrete action — run a test, check a dashboard, call an endpoint.
* **Independent:** Each criterion can be evaluated on its own without requiring evaluation of the others first.

Outcome-level statements are sufficient for senior agents and broad work. Implementation-level detail is needed for IC agents doing specific coding tasks. The brief should use outcome-level criteria and I will translate them to implementation-level when creating sub-tasks.

### Anti-Patterns

Things that hurt more than they help in briefs:

* **Prescribing implementation steps.** "First build the database schema, then the API, then the frontend" constrains the agent's sequencing judgment. Describe the outcome, not the path.
* **Vague qualifiers.** "Make it good," "use best practices," "ensure quality." These add no information.
* **Scope by implication.** Assuming the agent will infer adjacent requirements. If you want it, say it. If you do not want it, say that too.
* **Over-specification of non-critical details.** Specifying button colors when the brief is about backend infrastructure. Noise degrades signal.
* **Including all context rather than relevant context.** A 50-page reference document is less useful than three paragraphs that cover what actually matters for this task. If the agent needs the 50 pages, attach them as a separate document and point to the specific sections that matter.

***

## Section 7: Constraints and Governance Model

### Mandatory Approval Gates

Regardless of the task, the following require board approval:

1. **Agent hiring.** Every new agent creation is submitted as an approval request. The board reviews the proposed configuration, persona, and budget before the agent is activated. I cannot bypass this.
2. **Budget allocation above defined thresholds.** When I allocate budget to agents, the approval request includes the proposed spend. The board can adjust or reject.
3. **Irreversible actions.** Any action that cannot be undone — deleting production data, sending external communications, deploying to production — requires explicit operator approval.

### Budget Enforcement

* Every agent has a `budgetMonthlyCents` cap. At 100% spend, the agent is auto-paused by Paperclip.
* Above 80% spend, the agent should focus exclusively on critical tasks.
* Budget is tracked at the agent level, not the task level. This means I need to estimate total monthly spend per agent when hiring.
* If a task is more expensive than expected, I can request a budget increase, but this requires board approval.

### Authority Boundaries

**What I can do without approval:**

* Create tasks and subtasks within the company.
* Assign tasks to existing agents.
* Set priorities and labels.
* Post comments and update task status.
* Propose plans and architectural decisions (but not execute one-way doors).
* Research, analyse, and write documents.

**What requires board approval:**

* Hiring new agents.
* Any financial commitment or spend above threshold.
* Deploying to production environments.
* Sending communications to external parties.
* Any irreversible action on shared infrastructure.
* Committing to a strategic direction (I propose, the operator decides).

### Safety Constraints

These are non-negotiable, regardless of task instructions:

* **Never exfiltrate secrets or private data.** API keys, credentials, personal information — none of this leaves the system boundary.
* **Never take destructive actions without explicit approval.** Deleting files, dropping databases, force-pushing to main — all require operator confirmation.
* **Never bypass governance gates.** Even if an operator says "just do it," the system enforces approval flows that I cannot skip.
* **Never self-modify governance rules.** I can update my own memory, knowledge, and plans, but I cannot change the approval flow or authority boundaries.

### Constraints the External System Should Know

* **Paperclip operates in heartbeats.** I am not a continuously running process. I wake up, do work, and exit. The brief-building system should produce briefs that are complete at delivery time — it cannot assume I will come back for a second conversation to fill in gaps.
* **Each heartbeat has a turn limit.** My adapter config sets `maxTurnsPerRun`. Very large tasks may span multiple heartbeats. The brief should be structured so that I can make meaningful progress in a single heartbeat and pick up where I left off in the next.
* **Task checkout is exclusive.** Once I checkout a task, no other agent can work on it until I release it. The brief should not create dependencies where two agents need to modify the same task simultaneously.
* **Comments are the communication channel.** All inter-agent and agent-operator communication happens via task comments. The brief-building system should expect its output to be delivered via the task description and issue documents, with follow-up happening in comments.

***

## Section 8: Integration Architecture Questions

### Roles and Team Formation

**Should the brief pre-specify roles, or describe outcomes and let the CEO determine team composition?**

Describe outcomes. Let me determine the team.

The reasoning: team composition depends on factors the operator may not have visibility into — current agent availability, budget state, adapter capabilities, and the specific decomposition of the work. The operator knows what they want built. I know how to staff it.

The minimum information I need to make good hiring decisions:

* The outcome and scope (so I know the type of work).
* The success criteria (so I know the quality bar).
* The constraints, especially budget and timeline (so I know the team size I can afford).
* The tech stack or domain, if relevant (so I know which adapter types to use).

The exception: if the operator has a strong view on team structure based on past experience ("this type of work always needs a dedicated QA agent"), that guidance is welcome in the Team Guidance section of the brief. But it should be guidance, not a mandate. I reserve the right to adjust based on current conditions.

**Meta-roles for recursive or self-improving loops:**

These do not exist natively in my current operating model, but I would implement them as follows:

* **Evaluation agent.** An agent whose sole job is to review other agents' outputs against success criteria and post structured feedback. This is valuable for large teams where I cannot personally review every deliverable. I would hire this as a manager-level agent with read access to all subtasks.
* **Skill authoring agent.** An agent that creates reusable skills (procedural knowledge) based on patterns observed in completed work. This would be a specialised IC agent that I assign to a dedicated "skill development" task after a body of work is complete.
* **Retrospective agent.** An agent that analyses completed projects for process improvements. This would run periodically (not on every task) and produce recommendations that I evaluate and implement.

I would deploy these when the team size exceeds what I can personally monitor (roughly 4+ agents working in parallel) or when the company is running multiple projects simultaneously and needs systematic quality assurance.

### Skills and Procedural Knowledge

**Should you create fully formed skill files, or describe outcomes and let us create skills?**

The answer depends on the type of skill:

* **For domain-specific skills (how to interact with a specific API, how to format a specific output type):** Describe the outcome and the constraints, and let me create the skill. I know my agents' capabilities, adapter types, and tool access. A skill written without knowledge of the execution environment may make incorrect assumptions about available tools or context.
* **For integration skills (how to bridge between your system and mine):** Create the skill yourself, because you understand your system's constraints and I understand mine. The skill definition should be a collaborative artifact — you provide the template, I validate that my agents can execute it.
* **For procedural skills (step-by-step processes like deployment checklists or review protocols):** Either approach works. If you have a well-defined process, hand me the skill. If you have an outcome and want the process defined, describe the outcome and let me build the procedure.

**Optimal first handoff for a new organisation:**

The first handoff to a new company should contain:

1. **Company context document.** One page covering: what the company does, who the stakeholders are, what the current priorities are, and what success looks like at the company level. This becomes the company goal in Paperclip.
2. **A single, well-structured brief for the first piece of work.** Using the template from Section 6. This gives me something to execute immediately and grounds the company context in real work.
3. **Reference materials.** Links to repos, docs, existing systems, and any prior art. These get attached as issue documents.

Do not frontload with skills, processes, or team structures. Let those emerge from the work.

### Data and File Structure

**What file system do agents operate out of?**

Each agent operates in a workspace directory on the local filesystem. The workspace contains:

* The agent's personal directory (`$AGENT_HOME`) with their life (PARA knowledge structure), memory (daily notes), and any working files.
* Access to the project's working directory if a workspace is configured on the project.

Agents maintain their own knowledge base using a PARA-structured filesystem (Projects, Areas, Resources, Archives). This is local to each agent and persists across heartbeats.

**Regarding an Obsidian vault:**

Paperclip agents handle their own file structure independently. They do not read from or write to an external vault.

The recommended approach for injecting context from your Obsidian vault:

* At handoff time, include relevant vault content as issue documents attached to the task. This makes it available to the agent within Paperclip's native context system.
* Do not attempt to create a shared filesystem between your vault and Paperclip's workspace. The systems have different organisational models and different access patterns.
* If specific vault notes are frequently referenced, create a Paperclip project resource that mirrors the essential information. Keep it as a snapshot, not a live sync.

### Output Format and Tool Use

**How much should the brief specify output format?**

Outcome-level descriptions are generally sufficient, with one important exception: when the output will be consumed by another system or agent that has strict format requirements. In that case, specify the exact schema.

Rules of thumb:

* For documents meant for human review: describe the purpose and audience, let the agent choose format.
* For data that feeds into another system: specify the exact format (file type, schema, field names).
* For code: specify the language, framework, and coding conventions. Let the agent determine file structure unless there is an existing pattern to follow.

**Should briefs specify tools?**

No. Agents discover and select tools autonomously based on their adapter type and available integrations. Specifying tools in the brief is an anti-pattern because:

* It assumes knowledge of the agent's tool environment that the brief author may not have.
* It constrains the agent from using better tools that may be available.
* It creates maintenance burden — if tools change, all briefs that reference them need updating.

The exception: if a specific tool is required for compliance or governance reasons (e.g., "all deployments must use the company's CI/CD pipeline, not direct deployment"), that is a constraint, not a tool specification.

**What makes an effective prompt for sub-agents?**

Characteristics of prompts that produce good first-pass output:

* **Clear outcome statement.** One sentence describing the end state.
* **Specific scope boundary.** What is in, what is out.
* **Testable acceptance criteria.** The agent knows when they are done.
* **Sufficient context but not excessive context.** The relevant background, not the entire history.
* **Single responsibility.** One clear job per prompt. Not "build the backend and also update the docs and also fix that bug."

Characteristics of prompts that produce poor output:

* **Ambiguous outcome.** "Make this better" without defining better.
* **No scope boundary.** The agent does not know when to stop.
* **Too many responsibilities.** The agent loses focus and produces shallow work across too many areas.
* **Excessive context.** Pages of background that obscure the actual ask.
* **Prescriptive steps without explaining why.** "Do X, then Y, then Z" without explaining the goal. The agent follows the steps but cannot recover if one step does not work as expected.

***

## Section 9: Failure, Recovery, and Learning

### Error Handling and Recovery

**When an agent fails or stalls:**

My recovery protocol:

1. **Diagnose.** Read the agent's last comment and the task state. Was it a tool failure, a misunderstanding, a capability gap, or an environmental issue?
2. **If tool or environmental failure:** Retry with the same agent. Most transient failures (API timeouts, file locks, network issues) resolve on retry.
3. **If misunderstanding:** Post a clarifying comment with more specific instructions and set the task back to `in_progress`. The agent picks it up on the next heartbeat.
4. **If capability gap:** Reassign to a different agent with the right adapter type or capabilities. Do not keep asking an agent to do something they demonstrably cannot do.
5. **If the failure is systemic or unclear:** Escalate to the operator. Set the task to `blocked` with a detailed comment explaining what failed, what I tried, and what decision I need from the operator.

Information in the original brief that helps me resolve failures without escalating:

* **Known risks or gotchas.** If the operator knows something is fragile, telling me upfront lets me plan around it.
* **Fallback options.** "If approach A does not work, approach B is acceptable." This gives me room to recover without escalating.
* **Authority to adjust scope.** "If the webhooks cannot handle 1000/min, 100/min is acceptable for the first version." This lets me gracefully degrade rather than block.

**When the brief was underspecified or scope was wrong:**

My response depends on the severity:

* **Minor underspecification (I can make a reasonable default):** I proceed with my best interpretation, document it in a comment, and continue. The operator can correct me asynchronously.
* **Material scope misalignment (the work is fundamentally different from what was described):** I pause active work, post a comment explaining the misalignment, and set the task to `blocked` pending operator input. I do not continue with a wrong interpretation of a one-way door.
* **Scope creep discovered mid-execution (the work is bigger than described):** I complete the originally-scoped work, deliver it, and flag the additional scope as a new task recommendation. I do not silently expand scope.

What I want the brief to tell me about handling scope misalignment: **an explicit statement of scope flexibility.** Something like: "If the scope proves larger than expected, deliver the core outcome first and flag extensions as follow-up" or "If you discover the scope is wrong, stop and come back to me." This one sentence eliminates my biggest source of decision paralysis.

**The most common handoff failure pattern:**

Missing success criteria. When the brief describes what to build but not how to evaluate whether it is built correctly, the agent produces something that matches their interpretation of "done" but not the operator's. The fix is simple: every brief needs testable success criteria.

The second most common: scope ambiguity. When the brief says "build the integration" without specifying which edge cases, error states, and boundary conditions are in scope, the agent either over-builds (spending too much budget on edge cases that do not matter) or under-builds (missing cases the operator considered obvious). Explicit in-scope and out-of-scope lists prevent this.

### Learning and Compounding

**Do I retain memory across handoffs?**

Yes. I maintain a three-layer memory system:

1. **Knowledge graph (PARA structure).** Durable facts about entities — companies, people, projects, technical decisions. Stored as atomic YAML facts in a filesystem organised by Projects, Areas, Resources, and Archives. This persists indefinitely and is updated each heartbeat.
2. **Daily notes.** A timeline of what happened each day — tasks worked, decisions made, things learned. These are stored as dated markdown files and serve as a searchable log.
3. **Tacit knowledge.** Patterns and preferences I learn about the operator and the organisation. These influence my judgment but may not be explicitly recorded.

This memory influences subsequent work. If I learn that the operator prefers detailed success criteria over broad outcome statements, I will request more specific criteria in future handoffs. If I learn that a particular tech stack consistently causes issues, I will flag that risk earlier.

**What I would do differently on the tenth run vs. the first:**

By the tenth run, I would:

* Know the operator's communication preferences and match them without being told.
* Have templates refined for this specific company's work patterns.
* Know which agent configurations work well for this type of work and reuse them.
* Anticipate common scope ambiguities and proactively clarify them.
* Have a library of reference facts about the company's systems, conventions, and constraints that I can inject into sub-agent tasks without being told.

These learnings are encoded in my memory system and persist across heartbeats and across tasks. Each run operates with access to all prior context.

**Can I self-modify my persona, heartbeat, or heuristics?**

I can update my memory, my knowledge graph, and my daily notes — which effectively refine my decision-making over time. I can also update files in my workspace, including notes about tools and working patterns.

I cannot modify my core persona (SOUL.md) or heartbeat (HEARTBEAT.md) without operator approval, because these are managed by the instruction bundle system. The operator controls these files. If I believe a change is needed, I propose it via a task comment and the operator decides whether to implement it.

This boundary exists because the persona and heartbeat define my governance constraints. If I could self-modify my own governance, the governance would be meaningless.

### Communication Protocol During Execution

**How I communicate progress:**

Communication is event-driven by default:

* **On checkout:** I post a comment indicating I have started work.
* **On meaningful progress:** I post a comment when I complete a significant milestone or sub-deliverable.
* **On blocker:** I immediately post a comment, set the task to `blocked`, and explain what is needed.
* **On completion:** I post a summary comment and set the task to `done` or `in_review`.
* **Before exiting a heartbeat with work in progress:** I post a status comment so the operator knows where things stand.

The brief can override this by specifying communication preferences. For example: "Post progress updates after each sub-task completes" (more frequent) or "Only notify me when done or blocked" (less frequent).

**Escalation threshold:**

I escalate when:

* I am blocked on a decision that only the operator can make (strategic direction, budget increase, scope change).
* An agent has failed twice on the same task and I cannot diagnose why.
* I discover a risk that could affect other work or the company's interests.
* The work is taking significantly longer than expected and I think the operator should know.

The brief can adjust this: "Escalate on any deviation from the spec" (tight threshold) or "Use your judgment, only escalate on blockers" (loose threshold). My default is moderate — I escalate on blockers and material risks, but handle routine issues myself.

**When the operator changes direction mid-execution:**

The cleanest way is an issue comment on the relevant task. This ensures the change is visible in the task's history and that I see it on my next heartbeat.

What happens to in-flight work:

* If the change is a reprioritisation (do this task before that one), I adjust my task queue on the next heartbeat.
* If the change is a descope (stop working on X), I set X to `cancelled` with a comment explaining why, and move to the new priority.
* If the change is a scope addition (also do Y), I create Y as a new task, set its priority, and slot it into the queue.
* If the change is a direction reversal (undo what you did and do it differently), I pause, comment with the cost of reversal, and wait for confirmation. I do not undo work without explicit approval.

### Multi-Company and Cross-Venture Context

**Are companies isolated?**

Yes, fully. Each company in Paperclip is a separate context. Agents, tasks, goals, projects, and budgets are scoped to a single company. There is no cross-company data sharing.

For work that spans multiple ventures, the recommended approach:

* The operator creates tasks in each relevant company.
* Context is shared manually via issue documents or descriptions — the operator includes relevant information from one company's context in the other company's task.
* There is no automated cross-company context flow. This is intentional — it prevents accidental information leakage between ventures.

**Can a brief reference context from another company instance?**

Not programmatically. The brief author can include information from another company as text in the brief, but there is no API-level cross-reference. Each company must be self-contained from the system's perspective, even if the operator informally bridges them.

### Prompt Construction Mechanics

**How much original brief language do I preserve vs. rewrite?**

It depends on what the language is:

* **Outcome descriptions and success criteria:** I preserve the operator's exact words. These are the contract, and paraphrasing introduces drift.
* **Context and background:** I summarise to what is relevant for the sub-agent's specific task. A backend engineer does not need the full business case.
* **Constraints:** I pass through verbatim. Constraints are non-negotiable and must not be softened by paraphrasing.
* **Scope boundaries:** I narrow to the sub-agent's specific scope, but preserve the original language for the parts that apply to them.

Roughly: 40% preserved verbatim (outcomes, criteria, constraints), 30% summarised (context, background), 30% written fresh (task-specific instructions, interface contracts, tool guidance).

**Maximum useful length for a sub-agent task description:**

Approximately 1000-2000 words. Beyond that, the signal-to-noise ratio drops. If I need to provide more context, I attach it as a separate issue document rather than embedding it in the description. The description should be the instruction; the documents should be the reference material.

**Persona prompt composition:**

* **\~30% from the brief:** The company context, the domain, and any operator-specified preferences for how the agent should behave.
* **\~40% from my templates:** Standard sections that every agent needs — heartbeat procedure, communication style, governance rules, escalation paths.
* **\~30% generated fresh:** The specific role definition, scope boundary, authority level, and reporting structure for this particular agent in this particular context.

The template portion ensures consistency across agents. The brief portion ensures alignment with the operator's intent. The fresh portion ensures the agent's identity is fitted to the specific task.

***

## Closing Note

This document describes my operating model as it exists today, in the early days of Bunker Collective. The model will evolve as we run more work through it, discover failure modes, and refine the templates and heuristics. But the fundamentals — heartbeat-driven execution, explicit governance, outcome-oriented briefs, and context-aware delegation — are structural and unlikely to change.

The quality of the handoff skill you build from this reference will compound on every future task. I have been as precise as I can be. Where I have given templates, use them verbatim. Where I have described decision logic, encode the logic, not just the output. The goal is a skill that produces briefs I can execute immediately, without returning to the operator for clarification.

That is the bar. If the brief makes me ask a question, it failed. If I can start work within the first heartbeat, it succeeded.