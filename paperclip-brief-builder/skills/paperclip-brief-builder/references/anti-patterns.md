# Anti-Patterns in Paperclip Briefs

These are patterns the CEO has documented as degrading brief quality.
Check every brief against this list before presenting to Finn.

---

## Brief-Level Anti-Patterns

**Prescribing implementation steps.** "First build the database schema,
then the API, then the frontend" constrains the CEO's sequencing
judgment. The CEO determines delegation sequence, parallelisation,
and critical path from the outcome and scope. Describe what needs to
exist when the work is done, not the order of construction.

**Vague qualifiers.** "Make it good," "use best practices," "ensure
quality." These add no information. The CEO cannot act on them and
will either ignore them or ask for clarification, which wastes a
heartbeat. Replace with testable criteria: "handles 1000 events per
minute" is actionable; "make it performant" is not.

**Scope by implication.** Assuming the CEO will infer adjacent
requirements. If you want it done, name it in the In Scope section.
If you do not want it done, name it in the Out of Scope section. The
CEO defaults to conservative interpretation when scope is ambiguous,
which means it may under-deliver rather than over-deliver.

**Over-specification of non-critical details.** Specifying button
colours when the brief is about backend infrastructure. Noise
degrades signal. The CEO documented that sub-agent task descriptions
are most effective at 1000-2000 words — excessive detail in the
brief cascades into bloated sub-tasks.

**Excessive context.** A 50-page reference document is less useful
than three paragraphs covering what actually matters for this task.
If the full document is needed, attach it separately and point to
the specific sections that matter. The CEO summarises context for
sub-agents — give it the relevant parts, not the complete history.

**"Best practices" as a constraint.** This defers judgment to the
agent without specifying whose best practices or for what context.
The CEO flagged this as a specific anti-pattern. If there are
conventions to follow, name them. If not, let the CEO apply its own
judgment.

---

## Success Criteria Anti-Patterns

**Non-testable criteria.** "The system should be reliable" cannot be
evaluated. "The system recovers from failure within 30 seconds" can.
Every criterion must be verifiable by a concrete action — run a test,
check a dashboard, call an endpoint, read a document.

**Compound criteria.** "The API handles authentication and returns
user profiles and logs all requests." This is three criteria
compressed into one. If one passes and two fail, the compound
criterion is ambiguous. Split into independent, testable items.

**Criteria that duplicate the outcome.** If the outcome says "users
can log in via SSO" and a success criterion also says "users can log
in via SSO," the criterion adds nothing. Criteria should decompose
the outcome into its verifiable components, not restate it.

---

## Scope Anti-Patterns

**Missing Out of Scope section.** Without explicit exclusions, the
CEO may either over-deliver (spending budget on adjacent work the
operator did not want) or under-deliver (excluding something the
operator considered obviously in scope). Always name at least one
exclusion.

**Scope described as activities rather than deliverables.** "Research
the competitive landscape" is an activity. "A competitive landscape
document covering the top 5 competitors with pricing, features, and
positioning analysis" is a deliverable. The CEO can verify delivery
of an artifact; it cannot verify the thoroughness of an activity.

---

## Constraint Anti-Patterns

**Missing budget constraint.** Without a budget ceiling, the CEO
optimises for quality, which may result in higher spend than the
operator expected. Even "no hard ceiling — optimise for quality" is
better than silence, because it makes the tradeoff explicit.

**Missing scope flexibility statement.** The CEO identified this as
the single statement that most reduces decision paralysis. Without
it, the CEO may block and escalate when it could have resolved a
scope issue autonomously. Always include one of: "deliver core
outcome first, flag extensions as follow-up" or "stop and come back
to me if scope is wrong."

---

## Tool and Technology Anti-Patterns

**Specifying tools in the brief.** The CEO documented that agents
discover and select tools autonomously based on their adapter type.
Specifying tools constrains the agent, assumes knowledge of the
agent's environment, and creates maintenance burden. The exception:
compliance or governance requirements ("all deployments must use the
CI/CD pipeline") are constraints, not tool specifications.

**Specifying adapter types.** Bunker Collective uses claude_local
(Claude Code) as the standard adapter. The CEO selects adapters
based on task requirements. Do not specify adapter types in the brief
unless there is a specific reason a task requires a particular
execution environment.
