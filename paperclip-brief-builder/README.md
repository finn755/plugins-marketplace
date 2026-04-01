# Paperclip Brief Builder

**Version:** 1.0.0
**Author:** Finn Gane — Bunker Collective
**Built:** 2026-04-01

---

## Why This Exists

Translation layers are one of the highest ROI skill layers you can build for an agentic system. This one sits specifically between a human operator and the Paperclip CEO agent.

The insight behind it: Paperclip has a documented operating model for how the CEO parses tasks, decomposes work, hires agents, and evaluates output. If you give the CEO a well-structured brief that matches its input specification, it can begin execution within its first heartbeat without returning for clarification. If you give it a loose description, it interprets gaps on its own terms — which often produces output that matches the CEO's interpretation of "done", not yours.

This skill encodes that input specification. It dialogues with you to extract a vision, set success criteria, and establish scope — then assembles a brief the CEO can act on immediately.

---

## What It Does

A four-stage process, run entirely inside Claude:

**Stage 1 — Vision Capture**
Claude asks you about outcomes, scope, success criteria, constraints, and context. It asks one or two questions at a time and adapts based on your answers. Typically takes 3–5 exchanges. No implementation questions — those are the CEO's job.

**Stage 2 — Brief Assembly**
Claude maps everything you captured into the exact template the CEO expects, including a three-line "why chain" (company mission → project goal → this task) that gets passed verbatim to every sub-agent the CEO hires.

**Stage 3 — Quality Check**
Claude runs the brief against the CEO's documented failure modes before you ever see it. Missing fields get flagged rather than guessed.

**Stage 4 — Review and Approval**
Claude presents the brief for your review. You approve, refine, or restart. Once approved, you paste it into Paperclip.

---

## How to Use It

Trigger the skill in Claude:

```
/paperclip-brief-builder
```

Or just describe what you want to hand off. Claude will recognise the intent and launch the skill automatically.

**Example triggers:**
- "I want to hand this off to Paperclip"
- "Brief for the CEO"
- "Delegate this to Paperclip"
- "Create a Paperclip brief for [description]"

---

## A Note on Reusability

This skill was built specifically for Paperclip's CEO agent, but the underlying pattern is general: understand how a platform likes to ingest content, understand how it structures context downstream, then build a translation layer that formats your intent correctly for that platform.

If you find a different software platform you want to hand off to — and you do a deep audit of how that platform parses work and structures context — you can repurpose this skill to fit those constraints. The four-stage structure, the dialogue logic, and the quality-check approach all transfer. Only the brief template and reference files need to change.

---

## Reference Files (Inside the Skill)

The skill reads from five reference documents that encode Paperclip's operating model:

| File | What it contains |
|------|-----------------|
| `ceo-operating-model-condensed.md` | How the CEO parses tasks, decomposes work, hires agents, evaluates output — condensed to brief-relevant content |
| `ceo-operating-model-v2.md` | Full infrastructure reference — schemas, API contracts, system invariants |
| `brief-template.md` | The exact output format the CEO expects |
| `anti-patterns.md` | Things that degrade brief quality (drawn from CEO's documented failure modes) |
| `quality-checklist.md` | Final validation run before every brief is presented |

These are read automatically by the skill — you don't need to interact with them directly.

---

## Installation

```
/plugin → Browse Plugins → Paperclip Brief Builder → Install for me
```

Or via terminal:

```bash
claude plugin install paperclip-brief-builder
```

If you haven't added the Bunker Collective marketplace yet:

```bash
claude plugin marketplace add finn755/plugins-marketplace
```

---

*Bunker Collective — 2026*
