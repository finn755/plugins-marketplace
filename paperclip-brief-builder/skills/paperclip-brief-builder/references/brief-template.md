# Brief Template

This is the exact output format the Paperclip CEO expects. It is derived
from Section 6 of the CEO's operating model reference. Use this template
verbatim — do not rearrange fields or rename sections.

The CEO reads fields in this order: goal context first, then title, then
description content. The template reflects that reading order.

---

## Template

```markdown
# Brief: [Title — imperative verb phrase, e.g., "Build the user auth module"]

## Why This Matters

[The why chain, followed by 2-3 sentences connecting this work to the
company/project goal. What happens if this succeeds? What happens if it
does not get done?]

> Company mission: [one sentence]
> Project goal: [one sentence]
> This task: [one sentence]

[Expanded context on why this matters now.]

## Outcome

[One paragraph describing the desired end state. Not the steps, but what
the world looks like when this is done. Written as a testable assertion:
"When this is complete, X will be true."]

## Scope

### In Scope
- [Specific deliverable 1]
- [Specific deliverable 2]
- [Specific deliverable 3]

### Out of Scope
- [Thing that might seem in scope but is not]
- [Adjacent work that should not be touched]

## Success Criteria
- [ ] [Criterion 1 — specific, testable, independent]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Constraints
- Budget: [ceiling in dollars, or "no hard ceiling — optimise for quality"]
- Timeline: [deadline or "no hard deadline — quality over speed"]
- Governance: [what requires operator approval before the CEO acts]
- Scope flexibility: [how the CEO should handle scope misalignment —
  e.g., "deliver core outcome first, flag extensions as follow-up"
  or "stop and return to operator if scope is wrong"]
- [Any additional technical, legal, or operational constraints]

## Context
[Reference materials, prior art, related tasks, relevant history.
Include links to documents, repos, or previous task identifiers.
Point to specific sections that matter rather than attaching full
documents. If attaching documents, note which sections are relevant.]

## Team Guidance (Optional)
[If the operator has a view on team composition, include it here as
guidance, not a mandate. The CEO prefers to determine its own team
from the outcome and scope. Omit this section entirely if the
operator has no team preference.]

## Communication Preferences (Optional)
[How the operator wants to be updated during execution. Default is
async via task comments — only include this section to override
the default. Options: "post progress after each sub-task", "only
notify on completion or blocker", or a custom cadence.]
```

---

## Field Requirements

These classifications come directly from the CEO's documented input spec.

| Field                     | Status      | What happens if missing |
|---------------------------|-------------|------------------------|
| Title                     | Required    | CEO cannot begin — does not know what the work is. |
| Why This Matters          | Required    | CEO cannot evaluate tradeoffs — does not know why the work exists. |
| Outcome                   | Required    | CEO cannot determine done — no target state to work toward. |
| In Scope                  | Required    | CEO cannot bound work — defaults to conservative interpretation, may under-deliver. |
| Out of Scope              | Recommended | Without it, CEO may miss things operator wanted included or waste budget on unwanted work. |
| Success Criteria          | Required    | CEO's #1 documented failure mode: agent produces output matching its own definition of done rather than the operator's. |
| Constraints               | Recommended | Without budget/timeline, CEO optimises for quality, which may overspend or take longer than expected. |
| Scope Flexibility         | Recommended | CEO identified this as the single statement that most reduces decision paralysis during execution. |
| Context                   | Recommended | Without it, CEO researches independently — slower, may reach different conclusions. |
| Team Guidance             | Optional    | CEO can determine team from outcome and scope. Include only if operator has a strong view. |
| Communication Preferences | Optional    | CEO defaults to async event-driven comments. Include only to override. |

---

## Notes on Language

The CEO documented how it handles brief language when creating sub-tasks:

- **Outcome descriptions and success criteria:** Preserved verbatim.
  Write these carefully — they become the contract downstream.
- **Constraints:** Passed through verbatim. Do not soften.
- **Context and background:** Summarised to what is relevant for each
  sub-agent. Can be more detailed in the brief since the CEO will
  extract what each agent needs.
- **Scope boundaries:** Narrowed per sub-agent but original language
  preserved for applicable parts.

This means the outcome, success criteria, and constraints sections of
the brief carry the most weight. Invest the dialogue time there.
