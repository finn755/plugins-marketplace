---
name: copy-generator
description: >
  >
---

# Copy Generator Skill

This skill generates A/B cold email copy for a given persona and lead list. It produces email body variants (pain-led and question-led), subject line pairs, and exports both Notion records and Instantly/Lemlist-ready CSVs with platform-specific formatting.

---

## Purpose

Given a persona and associated leads, this skill:
1. Reads persona attributes (pain points, value hook, industry, stage signals)
2. Reads lead data (name, company, email)
3. Generates 2 subject line options (direct and curiosity-led)
4. Generates 2 email body options (pain-led and question-led variants)
5. Applies cold-email quality bar throughout (25–75 words, no em-dashes, binary CTA, prospect-first)
6. Writes campaign record to Notion Campaign Outputs DB
7. Exports platform-specific CSV (Instantly or Lemlist format)
8. Returns Notion page URL + confirms CSV attached

---

## Pre-flight

**Read before starting:**
- cold-email.md (reference all core principles, stage-specific angles, quality checklist)

**Required:**
- Notion Personas database (to read persona record)
- Notion Lead Lists database (to fetch leads linked to persona)
- Notion Campaign Outputs database (to write output copy)

**Campaign Outputs schema (minimum fields):**
- Name (title) — format: `[Persona Name] - Campaign [Date] - [Angle A/B]`
- Persona (relation to Personas DB)
- Campaign Goal (select: Demo Request, Call Booking, Pilot Sign-up, Information Request, Other)
- Lead Count (number)
- Subject Line A (text)
- Subject Line B (text)
- Email Body A (rich text)
- Email Body B (rich text)
- A/B Test Plan (rich text)
- CSV Attached (URL/file reference)
- Instantly Format? (checkbox)
- Lemlist Format? (checkbox)
- Created (date)
- Status (select: Draft, Ready, Sent, In Review)

---

## Orientation — Read Before Starting

**Before asking any config questions, run this self-check:**

1. **Is Notion connected?**
   Verify the Notion MCP is active. If not, stop: "The Notion connector doesn't appear to be active. Please check Cowork settings → Connectors before continuing."

2. **Does the Campaign Outputs database exist?**
   Use `notion-search` to confirm Campaign Outputs DB exists. If it doesn't, tell the operator: "I need a Campaign Outputs database in Notion to save the copy output. See README.md → Setup → Step 2 for the required field schema. Once created, run this skill again."

3. **Do you have a Run ID from a lead-list-builder run?**
   This is the most common failure point. copy-generator queries leads by Run ID. If the operator doesn't know their Run ID, say: "Do you have a Run ID from when you ran lead-list-builder? It looks like this: `aventi-venue-operators-2026-03-26`. If not, I can search for leads matching your persona name — just tell me the venture and approximate date."

4. **Are there leads in the Lead Lists DB?**
   If the operator says they haven't run lead-list-builder yet, stop: "I need leads in the Lead Lists database before I can generate copy. Run lead-list-builder first — just say 'build a lead list'."

5. **Is this a same-day re-run or a new campaign?**
   If the operator ran this exact persona + date combination before, there may already be a Campaign Outputs record. Ask: "Have you already generated copy for this persona today? If so, I'll create a new record rather than overwriting the existing one."

If all checks pass, proceed with the config questions below.

---

## Config Questions

### Q1: Which persona?
> "Which persona are we creating copy for? (e.g., 'VP Sales @ Mid-Market B2B SaaS')"

Fetch and list available personas if needed. Operator selects one.

### Q2: Campaign goal?
> "What's the primary goal of this campaign?
> - Demo Request (book a product demo)
> - Call Booking (intro/discovery call)
> - Pilot Sign-up (join a pilot program)
> - Information Request (learn more)
> - Other (specify)"

Record the goal. This informs CTA phrasing.

### Q3: Export format?
> "Which platform are you sending through?
> - Instantly (send via Instantly)
> - Lemlist (send via Lemlist)
> - Both CSV files (provide both formats for manual choice)
> - Notion only (just show me the variants, don't export)"

Record preference. Phase 4 supports both. API integration is Phase 5.

### Q4: Single or multi-persona campaign?
> "Is this a single-persona campaign, or are you combining leads from multiple personas into one export?
> - Single persona (all leads share the same ICP)
> - Multi-persona (leads from different personas in one campaign file)"

**This determines whether copy is embedded in the CSV:**
- **Single persona** — do NOT embed copy columns in the CSV. Copy is identical for every lead; set it once at the sequence level in Lemlist or Instantly. Export contact data only: `email`, `firstName`, `lastName`, `companyName`, `linkedinUrl` (Lemlist) or `Email`, `FirstName`, `LastName`, `CompanyName`, `LinkedinUrl` (Instantly). Include a note reminding the operator to paste the copy into their sequence template.
- **Multi-persona** — embed copy columns in the CSV so each lead carries its persona-specific variant. The sequence tool uses these as custom variables (`{{subjectA}}`, `{{bodyA}}`, etc.) to route different copy to different segments automatically.

---

## Step-by-Step Execution

### Step 1: Fetch Persona Record

Read the persona record from Notion:
- Industry
- Company Size
- Seniority
- Pain Points
- Value Hook
- Channel (Email, LinkedIn, Instagram, Phone)
- Source Type (Research — Assumptions vs. Collected — Market Data)

### Step 2: Fetch Associated Leads

Query Lead Lists database filtered by Run ID (format: `[venture]-[persona-slug]-[YYYY-MM-DD]`, e.g. `aventi-venue-operators-2026-03-26`). This is the canonical way to link leads to a persona run — the Lead Lists DB uses Run ID for provenance rather than a Persona relation field.

If the operator does not know the Run ID, fetch the Personas DB and ask which persona they want leads for, then query Lead Lists for the most recent Run ID matching that persona name pattern.

Read for each lead:
- First Name
- Last Name
- Email
- Company Name
- LinkedIn URL (if available)
- Quality Score
- Notes
- Lead Source

**Quality Score filter:** Before proceeding, flag any leads that meet one of these conditions:
- Quality Score ≤ 3
- Notes field contains "lower priority", "freelance", or "exclude"

Present flagged leads to the operator:
> "The following leads are flagged as lower priority: [list]. Include or exclude from this campaign?"

Wait for operator confirmation before proceeding. Do not generate copy for excluded leads.

Record count of confirmed leads. If >50 leads, note: "Generating variants for X leads; best practice is 30–50 leads per campaign to allow A/B testing."

### Step 3: Determine Stage and Angle

Based on Source Type, select angle framework:

**If Source Type = "Research — Assumptions":**
- Recommended angles:
  1. **Pattern Angle** ("We keep hearing…") — shows research without proof
  2. **Cost of Inaction** — focuses on prospect's pain, not your solution
  3. **Building With You** — position as co-creator
  4. **Honest Stage** (if pre-revenue) — transparency as differentiator

**If Source Type = "Collected — Market Data":**
- Recommended angles:
  1. **Use Case Angle** ("We're working with [type of company] on…")
  2. **Momentum** ("X similar firms are…")
  3. **Social Proof** (if growth stage — name customers or results)

Ask operator:
> "Which angle resonates with you?
> - [Angle 1]: [Brief description]
> - [Angle 2]: [Brief description]
> - [Angle 3]: [Brief description]
>
> Or I can generate one angle; which would you prefer?"

Record angle selection.

### Step 4: Generate Subject Line A (Direct)

**Pattern: Role/Pain Specific**

Formula: `[Role/Pain word] + [specificity signal]`

Examples:
- `Your sales workflow is bleeding time`
- `Revenue leaks through manual deal tracking`
- `Evaluation bottlenecks vs. scalability`

**Rules:**
- 2–5 words
- No fake "Re:" prefix
- Avoid clickbait
- Use prospect's language, not marketer speak

Generate 1 option.

### Step 5: Generate Subject Line B (Curiosity)

**Pattern: Question or Pattern-Led**

Formula: `[Pattern observation] OR [Question] OR [Cost of inaction]`

Examples:
- `The one thing every searcher avoids (for good reason)`
- `What's eating your best thinking?`
- `Series B teams say the same thing about this`

**Rules:**
- 2–5 words
- Question mark or statement
- Lead with specificity, not vagueness
- Should create low-grade FOMO without being manipulative

Generate 1 option.

### Step 6: Generate Email Body A (Pain-Led)

**Structure:**
1. **Hook** (1 sentence, prospect's world) — frame their situation
2. **Pain** (1 sentence, specific problem) — what's currently broken
3. **Bridge** (1 sentence, your product) — how you solve it
4. **CTA** (1 sentence, binary) — yes/no question or low-friction ask

**Example:**
```
Most deal teams track pipeline in spreadsheets
that go stale between reviews.

We built a live deal workspace that keeps
insights fresh across every evaluation pass.

Worth 15 minutes to see how it works?
```

**Rules:**
- Exactly 25–75 words (aim for 50)
- Prospect-first framing (about them, then solution)
- No em-dashes (use hyphens or rewrite)
- CTA is binary ("Does this resonate?", "Worth a call?", not "let's schedule time")
- No product inflating (state exactly what you do)
- One voice (conversational, not corporate)

Generate 1 variant.

### Step 7: Generate Email Body B (Question or Angle Variant)

**Structure (Question-Led Angle):**
1. **Hook** (1 sentence, their world)
2. **Question** (1 sentence, reframe the problem)
3. **Why it matters** (1 sentence, cost of inaction)
4. **CTA** (1 sentence, binary)

**Example:**
```
How do you track which evaluation insights
matter and which ones you'll forget?

Most teams realize too late that their
best thinking vanishes between passes.

Does this sound like you?
```

**Or (Social Proof / Use Case Variant — for growth stage only):**
1. **Hook** (1 sentence)
2. **What we're seeing** (1 sentence with light social proof)
3. **Why it works** (1 sentence)
4. **CTA** (1 sentence, binary)

**Example:**
```
Firms in your space are consolidating deal
evaluation into a single workspace.

Teams using it report 20% faster closes
and better visibility across partners.

Worth exploring?
```

**Rules:**
- Same as Body A (25–75 words, no em-dashes, binary CTA)
- Different angle from Body A
- Don't repeat Body A's language
- If using social proof, must be truthful (early traction = "a few firms", not "hundreds")

Generate 1 variant.

### Step 8: Apply Quality Checklist

For both subject lines and both bodies, verify:

- [ ] Under 125 words each (bodies)
- [ ] Opens with prospect's world, not sender's product
- [ ] No em-dashes
- [ ] CTA is binary or low-friction (not "let's schedule")
- [ ] At least one specificity signal (ecosystem, pain pattern, role reference)
- [ ] No claims venture can't back up
- [ ] Subject lines under 5 words
- [ ] Tone is conversational, not informative
- [ ] No product inflation (early stage: be honest about stage)

If any fail, regenerate that variant.

### Step 9: Write to Campaign Outputs DB

Create one Notion record with:

| Field | Value |
|-------|-------|
| **Name** | `[Persona] - Campaign [YYYY-MM-DD] - A/B Test` |
| **Persona** | [Link to persona record] |
| **Campaign Goal** | [from Q2] |
| **Lead Count** | [N] |
| **Subject Line A** | [Generated] |
| **Subject Line B** | [Generated] |
| **Email Body A** | [Generated] |
| **Email Body B** | [Generated] |
| **Angle Label** | [e.g., "Pain-led vs. Question-led"] |
| **A/B Test Plan** | `Group A: Email 1 = Body A + Subject A, Email 2 = Body B + Subject B. Group B: Reverse. Both groups receive both messages. Test which resonates more on first touch.` |
| **Status** | Draft |
| **Created** | [Today] |

### Step 10: Generate Export (destination-conditional and persona-count-conditional)

Fetch the full lead list one more time (ensure you have complete data: first name, last name, email, company name).

**Only generate the format(s) matching the Q3 answer. Do not produce outputs for destinations the operator did not select.**

**Whether to embed copy columns is determined by Q4:**
- Single persona → contacts only, no copy columns. Remind operator to set copy at sequence level.
- Multi-persona → embed copy columns so each lead carries persona-specific variants.

---

#### If destination = Instantly (or Both):

**Single-persona column order:**
```
Email, FirstName, LastName, CompanyName, LinkedinUrl
```

**Multi-persona column order (copy embedded):**
```
Email, FirstName, LastName, CompanyName, SubjectA, BodyA, SubjectB, BodyB, PersonaName
```

**Row example:**
```
john@example.com, John, Doe, Acme Corp, Your sales workflow is bleeding time, Most deal teams track pipeline in spreadsheets that go stale between reviews. We built a live deal workspace that keeps insights fresh across every evaluation pass. Worth 15 minutes to see how it works?, How do you track which evaluation insights matter?, Does this sound like you?, VP Sales @ Mid-Market B2B SaaS
```

**Rules:**
- Use PascalCase for standard fields (Email, FirstName, LastName, CompanyName)
- Custom variables use PascalCase: SubjectA, BodyA, SubjectB, BodyB, PersonaName
- UTF-8 encoding
- No spaces in column names
- Max 50 custom variables per import ✓ (we're using 5)

**Validate:**
- Email must be first column
- All emails are valid format
- No null values in Email column

---

#### If destination = Lemlist (or Both):

**Single-persona column order:**
```
email, firstName, lastName, companyName, linkedinUrl
```

**Multi-persona column order (copy embedded):**
```
email, firstName, lastName, companyName, subjectA, bodyA, subjectB, bodyB, personaName
```

**Row example:**
```
john@example.com, John, Doe, Acme Corp, Your sales workflow is bleeding time, Most deal teams track pipeline in spreadsheets that go stale between reviews. We built a live deal workspace that keeps insights fresh across every evaluation pass. Worth 15 minutes to see how it works?, How do you track which evaluation insights matter?, Does this sound like you?, VP Sales @ Mid-Market B2B SaaS
```

**Rules:**
- Use camelCase for all columns (email, firstName, companyName, etc.)
- UTF-8 encoding
- No spaces in column names
- Max 2,000 characters per field (our body is ~75 words = ~400 chars, safe)

**Validate:**
- Email is first column
- All emails are valid
- No null values in email column

---

#### If destination = Notion only:

Skip CSV generation entirely. Copy is already written to the Campaign Outputs DB in Step 9. No file export needed.

---

#### If destination = HubSpot or other CRM:

Export a plain CSV with standard contact fields only: `first_name`, `last_name`, `email`, `company_name`. Body copy goes into the Notion record for manual reference. Note to operator that copy fields must be mapped manually inside the CRM.

### Step 11: Attach and Update Record

If a CSV was generated, attach it to the Notion Campaign Outputs record.

Update the record only for the fields that apply:
- `CSV Attached` → [URL/reference to attached file] — only if a CSV was produced
- `Instantly Format?` → checked only if destination = Instantly or Both
- `Lemlist Format?` → checked only if destination = Lemlist or Both
- `Status` → Ready (for operator review before sending)

### Step 12: Return Output

Provide operator:

```
✓ Campaign copy generated

CAMPAIGN OUTPUTS
════════════════
Persona:        [Name]
Leads Included: [N]
Campaign Goal:  [Goal]
Angles:         Body A (pain-led) vs. Body B (question-led)

SUBJECT LINES
─────────────
A: [Subject A]
B: [Subject B]

EMAIL BODIES (word counts)
──────────────────────────
A: [Body A] — [WC] words
B: [Body B] — [WC] words

EXPORTS
───────
Notion Page: [URL to Campaign Outputs record]
[Only show lines below if the corresponding CSV was generated]
Instantly CSV: [Filename] — ready for import       ← Instantly or Both only
Lemlist CSV:   [Filename] — ready for import       ← Lemlist or Both only
[If Notion only: no CSV — copy is in Notion record above]
[If HubSpot/other: plain contact CSV — copy fields in Notion record]

NEXT STEPS
──────────
1. Review copy in Notion (any edits, send to operator first)
2. Download CSV(s)
3. Import to Instantly or Lemlist
4. Set campaign cadence: Email 1 → 2–3 days → Email 2 → 4–5 days → Email 3 (or switch channels)
5. Track reply rates and measure which variant (A vs. B) performs better
```

---

## Output

1. **Notion Campaign Outputs record** — contains all 4 email variants, A/B test plan, metadata
2. **Instantly-ready CSV** (if requested) — `[PersonaName]-Instantly-[Date].csv`
3. **Lemlist-ready CSV** (if requested) — `[PersonaName]-Lemlist-[Date].csv`
4. **Plain text summary** — subject lines and body variants for operator review

---

## Error Handling

**Persona has no leads:**
> "Persona has no leads in Lead Lists DB. Run lead-list-builder first, or check that leads are linked to this persona."

**Leads missing required fields:**
> "Some leads are missing Email or Company Name. We need at minimum Email + Last Name + Company Name for accurate personalization. Suggest:
> 1. Re-run lead-list-builder with enrichment
> 2. Manually fill in missing fields in Notion
> 3. Proceed with CSV export but flag incomplete records"

**CSV generation fails:**
> "CSV write error. Check: (1) Lead count is >0, (2) Email column has no nulls, (3) No special characters in names that break CSV encoding. Try exporting to plaintext first, then retry CSV."

**Notion Campaign Outputs write fails:**
> "Notion write error. Verify: (1) Campaign Outputs database exists, (2) All required fields exist, (3) Run ID matches the format used in Lead Lists DB. If issue persists, try saving as Draft and updating manually."

---

## Integration Notes

### Input from persona-builder
- Pain Points (inform angle selection)
- Value Hook (inform Body angle)
- Source Type (research vs. market data → angle selection)
- Channel (Email/LinkedIn/etc. → CTA phrasing)

### Input from lead-list-builder
- Lead list with emails, names, companies
- Quality scores (flag if <50% email capture before proceeding)
- Lead sources (note in campaign for reporting)

### Output to Instantly/Lemlist
- CSV is platform-ready for import
- No additional mapping needed
- Columns match platform's auto-mapping schema exactly
- Custom variables (SubjectA, BodyA, etc.) will appear as template variables: `{{SubjectA}}`, etc.

---

## Quality Checklist Summary

Before presenting any campaign to operator, verify all of:

**Subject Lines:**
- [ ] A is direct (role/pain specific)
- [ ] B is curiosity-led (question or pattern)
- [ ] Both under 5 words
- [ ] No "Re:" prefix
- [ ] No clickbait without substance

**Email Bodies:**
- [ ] A is pain-led (problem first)
- [ ] B is question-led or social-proof variant
- [ ] Both 25–75 words
- [ ] Both open with prospect's world
- [ ] No em-dashes
- [ ] No product inflation
- [ ] CTA is binary ("Does this resonate?", "Worth a call?")
- [ ] Tone is conversational
- [ ] Tested against cold-email quality checklist

**CSV Exports:**
- [ ] All emails valid format
- [ ] No nulls in required columns
- [ ] Column names match platform spec exactly
- [ ] UTF-8 encoding
- [ ] Row count matches lead count

---

*Phase 4 GTM Plugin — Skill 3 of 3*
