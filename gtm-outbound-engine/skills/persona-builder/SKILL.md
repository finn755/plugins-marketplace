---
name: persona-builder
description: >
  >
---

# Persona Builder Skill

This skill translates a natural language ICP brief into a structured, actionable persona record in the Notion Personas database. It enriches the brief, confirms key attributes, and produces a record that feeds directly into the lead-list-builder skill.

---

## Purpose

When a venture defines an ICP verbally ("we're targeting VP Sales at mid-market B2B SaaS companies"), this skill:
1. Extracts key persona dimensions from the brief
2. Infers missing pain points and value hooks
3. Classifies the source of the persona (research assumptions vs. market data)
4. Maps digital presence locations where this persona congregates
5. Writes a complete Notion record that lead-list-builder can use to find leads

---

## Pre-flight

**Required information:**
- Venture name
- Natural language ICP brief (at least 2–3 sentences describing who you're targeting)

**Optional information (inferred if not provided):**
- Pain points
- Value hook
- Geographic targeting
- Company size constraints
- Seniority level

**Notion setup:**
- Personas database must exist with these 11 fields:
  - Name (title)
  - Industry
  - Company Size
  - Seniority
  - Channel
  - Source Type (select: "Research — Assumptions" or "Collected — Market Data")
  - Digital Presence (multi-select: LinkedIn, Reddit, Instagram, Directories, Podcasts, Forums, Website, Facebook Groups, Twitter/X)
  - Pain Points (rich text)
  - Value Hook (rich text)
  - Notes (rich text)
  - Created (date)

---

## Orientation — Read Before Starting

**Before asking any questions, run this self-check:**

1. **Is Notion connected?**
   Ask yourself: can I reach the Notion MCP? If not, stop and tell the operator: "The Notion connector doesn't appear to be active in this session. Please check Cowork settings → Connectors and confirm Notion is connected before we continue."

2. **Does the Personas database exist?**
   Use `notion-search` to look for a database named "Personas" in the operator's workspace. If it doesn't exist, say: "I can't find a Personas database in your Notion workspace. You'll need to create one before we can save this persona — see README.md → Setup → Step 2 for the required field schema."

3. **Is this the first skill in a new campaign, or are we adding to an existing one?**
   If the operator seems to already have a persona built and wants to jump to lead-list-builder, let them know: "You can skip this skill and go straight to lead-list-builder if a persona already exists in Notion. Just say 'build a lead list' and provide the persona name."

If all three checks pass, proceed with Step 1 below.

---

## Step-by-Step Execution

### Step 1: Gather ICP Brief

Ask the operator:
> "What's the venture name, and describe in 2–3 sentences who you're trying to reach. Who are they, what are they doing, and why should they care about your solution?"

Capture the raw brief verbatim.

### Step 2: Extract Dimensions

From the brief, identify and list:
- **Role/Title** (e.g., "VP Sales", "Engineering Manager", "Club Owner")
- **Company Type** (e.g., "Mid-market SaaS", "Sports Academy", "Agency")
- **Company Size** (estimate or bucket: 1–10, 11–50, 51–200, 200+)
- **Industry** (e.g., "Software", "Sports", "Healthcare")
- **Seniority** (e.g., "Executive", "Director", "Manager", "Individual Contributor")
- **Primary Pain Point** (infer from context if not stated)
- **Value Hook** (what problem does your solution solve for them?)

### Step 3: Enrich and Confirm

Present back to the operator for confirmation:

> **Proposed Persona Attributes**
> - **Role:** [extracted role]
> - **Company Type:** [extracted type]
> - **Company Size:** [extracted size]
> - **Industry:** [extracted industry]
> - **Seniority:** [extracted seniority]
> - **Primary Pain Point:** [inferred pain point]
> - **Value Hook:** [inferred or stated value hook]
>
> Does this match your intent? Any corrections?

Wait for confirmation. If the operator refines, update the attributes.

### Step 4: Classify Source Type

Ask the operator to classify the persona:

> "Is this persona based on:
> - **Research — Assumptions** (you did desktop research or talked to 1–2 people; you're testing these assumptions)
> - **Collected — Market Data** (you have paying customers, pilot feedback, or strong market validation)"

This determines how the lead-list-builder routes search strategy.

### Step 5: Select Digital Presence Locations

Present a checklist and ask the operator to select where this persona is likely to be found:

- [ ] LinkedIn (most B2B professionals)
- [ ] Reddit (communities, niche discussions)
- [ ] Instagram (visual industries, coaches, creators, agencies)
- [ ] Directories (industry associations, governing bodies, club networks)
- [ ] Podcasts (listening to niche podcasts = deep interest signal)
- [ ] Forums (niche communities, specific industries)
- [ ] Website (official sites, blogs, about pages)
- [ ] Facebook Groups (communities, clubs, alumni networks)
- [ ] Twitter/X (tech, finance, some verticals)

Instruction to operator: "Select any location where you'd expect to find organic signals about this persona (bios, comments, participation, websites they run)."

Record the selections for the Digital Presence multi-select field.

### Step 6: Generate Persona Name

Create a Notion-friendly persona name. Pattern:

`[Seniority Level] [Role] @ [Company Type]`

Examples:
- `VP Sales @ Mid-Market B2B SaaS`
- `Club Owner @ Youth Sports Academy`
- `Engineering Manager @ Series A Startup`

### Step 7: Write to Notion

Using notion-create-pages, write a new record to the Personas database with:

| Field | Value |
|-------|-------|
| **Name** | [generated persona name] |
| **Industry** | [extracted industry] |
| **Company Size** | [extracted size] |
| **Seniority** | [extracted seniority] |
| **Channel** | [primary outreach channel — Email/LinkedIn/Instagram/Cold Call] |
| **Source Type** | [Classification from Step 4] |
| **Digital Presence** | [Selected locations from Step 5] |
| **Pain Points** | [Formatted as bullet list] |
| **Value Hook** | [Formatted as 1–2 sentences] |
| **Notes** | [Any operator notes, e.g., "Test with 3-5 leads before full campaign"] |
| **Created** | [Today's date] |

### Step 8: Return Output

Provide the operator with:
- Notion page URL
- Persona name
- Quick summary: "Persona created: [name]. Source type: [assumption/market data]. Digital presence: [locations]. Ready to pass to lead-list-builder."

---

## Output

- **Notion page URL** — clickable link to the created persona record
- **Persona record** — all 11 fields populated and linked

---

## Error Handling

**If the brief is too vague:**
> "I need a bit more specificity. Can you describe: (1) What's their job title or role? (2) What type of company do they work at? (3) What's the main problem you're solving for them?"

**If the operator can't decide on Digital Presence locations:**
> "Let's start with the obvious ones: LinkedIn (everyone B2B), Instagram (if they're visual/content creators), Directories (if they're part of an association). We can refine after the first lead run."

**If Source Type is unclear:**
> "Have you talked to customers or prospects yet? If yes → 'Collected — Market Data'. If no → 'Research — Assumptions'. Assumptions are fine — we'll test them with the lead-list-builder."

---

## Notes for Lead-List-Builder Integration

The persona record created here feeds directly into lead-list-builder as input. Key fields used downstream:
- **Source Type** determines search strategy (exploratory for assumptions, defined queries for market data)
- **Digital Presence** locations constrain which search tools are used
- **Pain Points + Value Hook** inform copy-generator's email angles
- **Company Size + Seniority** filter lead enrichment and scoring

---

*Phase 4 GTM Plugin — Skill 1 of 3*
