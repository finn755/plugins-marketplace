# Quality Checklist

Run this checklist against every brief before presenting it to Finn.
Each item maps to a documented requirement or failure mode from the
CEO's operating model.

---

## Required Fields (brief fails without these)

- [ ] **Title** is an imperative verb phrase (e.g., "Build the...",
      "Create a...", "Document the...")
- [ ] **Why This Matters** section exists and contains the why chain
      (company mission → project goal → this task)
- [ ] **Outcome** is written as a testable assertion ("When this is
      complete, X will be true")
- [ ] **In Scope** lists specific deliverables, not activities
- [ ] **Success Criteria** contains at least three testable,
      independent, specific criteria
- [ ] Each success criterion can be verified by a concrete action
      (run a test, check a dashboard, read a document, inspect an
      artifact)

## Recommended Fields (brief quality degrades without these)

- [ ] **Out of Scope** lists at least one explicit exclusion
- [ ] **Constraints** section includes budget (or explicit "no hard
      ceiling"), timeline (or "no hard deadline"), and governance
      requirements
- [ ] **Scope flexibility statement** is present (how the CEO should
      handle scope misalignment)
- [ ] **Context** section includes relevant reference materials with
      pointers to specific sections rather than full documents

## Language Quality

- [ ] Outcome and success criteria use outcome-level language, not
      implementation-prescriptive language
- [ ] No vague qualifiers ("make it good", "best practices", "ensure
      quality")
- [ ] No prescribed implementation steps or sequencing
- [ ] No tool specifications (unless for compliance/governance reasons)
- [ ] No adapter type specifications
- [ ] Constraints are stated as constraints, not as preferences

## Anti-Pattern Check

- [ ] No compound success criteria (each criterion tests one thing)
- [ ] No criteria that merely restate the outcome
- [ ] Scope is described as deliverables, not activities
- [ ] Context is relevant, not exhaustive
- [ ] No colloquial shorthand ("the move is...", "best practices")

## Structural Check

- [ ] Brief follows the template field order exactly (Title → Why
      This Matters → Outcome → Scope → Success Criteria →
      Constraints → Context → optional sections)
- [ ] Optional sections (Team Guidance, Communication Preferences)
      are omitted if not needed, not included empty
- [ ] Brief is self-contained — someone reading only this document
      understands what needs to happen and why

## CEO Compatibility Check

- [ ] The CEO could begin work in its first heartbeat without
      returning for clarification (the bar the CEO set for itself)
- [ ] Every field the CEO classified as "Required" is present and
      non-empty
- [ ] The why chain could be passed verbatim to a sub-agent and
      still make sense
- [ ] The scope boundary is clear enough that the CEO's documented
      conservative-interpretation default would not cause it to
      under-deliver
- [ ] If the brief includes context from another Bunker venture,
      that context is self-contained (Paperclip companies are
      fully isolated — no cross-company API references)
