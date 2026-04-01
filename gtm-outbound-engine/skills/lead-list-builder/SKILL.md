---
name: lead-list-builder
description: >
  >
---

# Lead List Builder Skill

This skill transforms a Notion persona into a searchable list of leads. It determines the right search strategy based on the persona's source type, finds leads across the persona's Digital Presence locations, optionally enriches them, and writes results to the Lead Lists database with full provenance.

---

## Purpose

Given a persona record, this skill:
1. Routes search strategy based on Source Type (exploratory vs. defined)
2. Executes searches across Digital Presence locations
3. Returns a quality report (leads found, deliverability metrics)
4. Offers platform-specific enrichment
5. Writes leads to Notion with persona relationship linked
6. Returns count, database view link, and quality summary

---

## Pre-flight

**Load before starting:**
- supadata-credit-efficiency.md if enrichment may be needed

**Required:**
- Notion Personas database (source of persona record)
- Notion Lead Lists database (destination for leads)

**Lead Lists database schema (minimum fields):**
- Name (title)
- Run ID (text) — format `[venture-slug]-[persona-slug]-[YYYY-MM-DD]`; canonical link back to persona run. Used by copy-generator to query leads by campaign.
- Email
- First Name
- Last Name
- Company Name
- LinkedIn URL
- Digital Channel (select: Email, LinkedIn, Instagram, Phone)
- Lead Source (select: WebSearch, Directory Scrape, Apollo, Lemlist Finder, Manual)
- Quality Score (number 1–10)
- Notes
- Created (date)

---

## Config Questions

When the operator runs this skill, ask in sequence:

### Q1: Which persona?
> "Which persona are we finding leads for? Say the name (e.g., 'VP Sales @ Mid-Market B2B SaaS') or I can show you the available personas."

If they say "show me", fetch and list the available personas from the Personas database. Then ask them to pick one.

### Q2: How many contacts?
> "How many leads are you targeting for this campaign? (e.g., 50, 100, 200)"

Record the target. This determines when to stop searching.

### Q3: Outreach channel?
> "What's the primary outreach channel?
> - Email (highest priority — focus on email capture)
> - LinkedIn (LinkedIn URLs as primary contact method)
> - Instagram (Instagram handles/DMs)
> - Cold Call (phone number priority)
> - Multi-channel (balanced across all)"

Record the channel. This shapes enrichment priorities.

### Q4: Export destination?
> "Where should the final lead list go?
> - Notion (write to Lead Lists DB — recommended)
> - Instantly CSV (export Instantly-ready CSV)
> - Lemlist CSV (export Lemlist-ready CSV)
> - HubSpot (if you have HubSpot integration)
> - Email me a CSV (send via email)"

Record destination. Phase 4 supports Notion + CSV exports. API pushes to Instantly/Lemlist are Phase 5.

### Q5: Batch size (optional)
> "If we need to enrich leads, how many personas should we batch in one enrichment pass? (e.g., 1 persona at a time, or 3–5 together?)"

Default: 1 persona at a time.

---

## Orientation — Read Before Starting

**Before asking any config questions, run this self-check:**

1. **Is Notion connected?**
   Verify the Notion MCP is active. If not, stop and tell the operator: "The Notion connector doesn't appear to be active. Please check Cowork settings → Connectors before we continue."

2. **Does a persona exist to search from?**
   Use `notion-search` to confirm the Personas DB exists and has at least one record. If the operator hasn't run persona-builder yet, tell them: "I need a persona record in Notion before I can search for leads. Run persona-builder first — just say 'build a persona'."

3. **Does the Lead Lists database exist with a Run ID field?**
   Check that the Lead Lists DB exists and has a `Run ID` property (text type). If the field is missing, do not proceed — leads will have no provenance link. See TROUBLESHOOTING.md → S3.

4. **Are you clear on the Run ID you'll use?**
   The Run ID format is `[venture-slug]-[persona-slug]-[YYYY-MM-DD]`. If this is a same-day re-run of the same persona, append `-v2` to avoid collision. Confirm the Run ID with the operator before writing any records.

5. **Is Supadata enrichment likely?**
   If the persona targets organisations (clubs, companies, institutions) rather than named individuals, Supadata scraping will likely be needed. Load the supadata-credit-efficiency rules now — do not wait until the enrichment gate.

If all checks pass, proceed with the config questions below.

---

## Step-by-Step Execution

### Step 1: Fetch Persona Record

Retrieve the full persona record from Notion. Read:
- Name
- Industry
- Company Size
- Seniority
- Digital Presence (array of locations)
- Source Type ("Research — Assumptions" or "Collected — Market Data")
- Pain Points
- Value Hook

### Step 2: Route Search Strategy

**If Source Type = "Research — Assumptions":**
> **Exploratory strategy:** Test 3–5 diverse search queries across Digital Presence locations. Return source map showing which queries worked. Goal: validate assumptions about where this persona congregates.

**If Source Type = "Collected — Market Data":**
> **Defined strategy:** Use high-confidence search patterns against known locations. Focus on scale. Goal: maximize lead volume.

### Step 3: Execute Search

For each Digital Presence location selected in the persona:

#### LinkedIn
- Pattern: `[Role/Title] at [Company Type] | [Industry]`
- Search via WebSearch for LinkedIn profiles
- Extract: name, headline, company, email (if visible in bio or profile snippets)

#### Directories (Associations, Clubs, Governing Bodies)
- Example patterns: sports governing body websites, industry directories, club networks
- Use WebSearch to find directory URLs
- If enrichment needed: Use supadata_scrape for individual directory pages
- Extract: name, company name, contact info from directory entry

#### Instagram
- Pattern: `[Role keyword] @ [Region] instagram.com`
- Use WebSearch for Instagram profiles matching persona
- Extract: handle, bio, website link (if present)

#### Reddit
- Pattern: subreddit matching industry/role, find active posters
- Use WebSearch to find relevant communities
- Extract: username, if email present in profile or comments

#### Forums / Podcasts / Facebook Groups
- Use WebSearch to find relevant communities
- Extract: names of active participants, any contact info in profiles

#### Websites
- If persona runs or represents a business, search for "best [industry] [company type] [region]"
- Use WebSearch + supadata_scrape to extract contact pages
- Extract: name, email, company

### Step 4: Return Quality Report

Before surfacing the quality report, classify each lead:

**Generic inbox detection:** If an email matches the pattern `info@`, `hello@`, `contact@`, `team@`, or similar catch-all prefixes, flag it as a generic inbox. Generic inboxes score max 5 regardless of other fields. Add a `Notes` callout: "Generic inbox — director name not confirmed." These should be prioritised for Supadata enrichment (about/team page scrape) before the campaign runs.

**LinkedIn-only detection:** If a lead has a LinkedIn URL but no email, flag it as LinkedIn-step only. These leads are valid for multi-channel sequences (Lemlist LinkedIn steps) but cannot receive email steps. Note this clearly in the quality report.

Synthesize findings into:

```
LEAD SEARCH QUALITY REPORT
═══════════════════════════════════

Persona: [Name]
Target: [N] leads
Outreach Channel: [Channel]

RESULTS BY SOURCE
─────────────────
LinkedIn:        [M] leads found | [%] with email | [%] with LinkedIn URL
Directories:     [M] leads found | [%] with company name | [%] with email
Instagram:       [M] leads found | [%] with handle | [%] with website
Forums/Reddit:   [M] leads found | [%] with contact info | [%] with email
Other:           [M] leads found

TOTALS
──────
Raw Leads Found:       [Total]
Named + email:         [Count] ([%])  ← strongest leads
Generic inbox:         [Count] ([%])  ← flag for enrichment
LinkedIn-only:         [Count] ([%])  ← LinkedIn step only
No contact info:       [Count] ([%])  ← exclude or enrich

RECOMMENDATION
──────────────
[Assessment of data quality]
- If >60% named email capture: Ready for email campaign
- If generic inboxes >30% of list: Run Supadata about/team page scrape first
- If 30–60% email: Recommend enrichment pass
- If <30% email: Recommend multi-channel enrichment before campaign

Next Step: Enrich? (Y/N) [Recommended tool: Apollo for B2B / Supadata about+team pages for generic inboxes / Lemlist Finder for emails]
```

### Step 5: HITL Gate — Offer Enrichment

**This gate runs before writing leads to Notion.** Only write leads to the Lead Lists DB once enrichment is complete (or operator confirms proceeding as-is). This avoids creating records that immediately need updating.

Present to operator:

> "Quality report above. Ready to export as-is, or enrich first?
>
> **Enrichment options:**
> - **None** — proceed now, write leads to Notion as-is
> - **Supadata Website Scrape** — scrape about/team pages to find director names behind generic inboxes (1 credit per company). Recommended if >30% of list is generic inboxes.
> - **Apollo** (B2B lead lookup — email, phone, job history)
> - **Lemlist Finder** (email finding from names + company domain — good for LinkedIn-only leads)
> - **LinkedIn Verification** (confirm profile URLs are live)
>
> Which would help most?"

### Step 6: Execute Enrichment (if selected)

If operator chooses enrichment:

**For Supadata enrichment (generic inbox resolution):**
- Load supadata-credit-efficiency before starting
- Priority targets: leads with generic inboxes. Scrape the company's `/about`, `/team`, or `/contact` page (1 credit each — use Map first if contact page URL is unknown)
- Extract: director/owner name from team page, personal email if listed
- Update the raw lead data before writing to Notion
- Track credit usage in cost report

**For Apollo/Lemlist/LinkedIn (LinkedIn-only lead enrichment):**
- Priority targets: LinkedIn-only leads with no email
- Follow tool documentation
- Extract: email, phone, LinkedIn URL confirmation
- Update raw lead data before writing to Notion

After enrichment, re-run quality report:

```
ENRICHMENT RESULTS
──────────────────
Records enriched:           [N]
Director names found:       [N] (via team/about page scrape)
Additional emails found:    [N] ([+% improvement])
Additional phones found:    [N]
Credits used:               [N] (if Supadata)

Updated Named Email Rate:   [%]
Updated Generic Inbox Rate: [%]
LinkedIn-only remaining:    [N]
Ready for campaign?         [Y/N]
```

### Step 7: Write to Notion Lead Lists DB

Once enrichment is complete (or operator confirms as-is), write all leads to Notion:

- Create one Lead Lists record per lead
- Set Run ID using the format `[venture-slug]-[persona-slug]-[YYYY-MM-DD]`
- Populate all available fields: Name, Email, Company Name, LinkedIn URL, Lead Source, Notes
- Assign Quality Score:
  - 10 = Named contact + email + LinkedIn
  - 8 = Named contact + email
  - 6 = Named contact, email only
  - 5 = Generic inbox (max score for generic inboxes regardless of other fields)
  - 4 = Name + Company (no email)
  - 3 = LinkedIn URL only (LinkedIn-step candidate)
  - 2 = Partial info only
- For LinkedIn-only leads: add `Notes` callout "LinkedIn step only — no email confirmed"
- For generic inboxes where director name was not found: add `Notes` callout "Generic inbox — director name not confirmed"

### Step 8: Export

Ask operator which format they need:

> "Export format:
> - **Notion only** — leads already written to Lead Lists DB in Step 7, no CSV needed
> - **Instantly CSV** — PascalCase headers for direct Instantly import
> - **Lemlist CSV** — camelCase headers for Lemlist import (LinkedIn-only leads included with blank email; Lemlist routes them to LinkedIn steps automatically)
> - **Plain CSV** — standard format for manual import
>
> Which format?"

**If Instantly CSV:**
- Export with columns: `Email`, `FirstName`, `LastName`, `CompanyName`, `LinkedinUrl`, `PersonaName`
- Validate: Email must be first column, all emails valid format
- PascalCase headers only

**If Lemlist CSV:**
- Export with columns: `email`, `firstName`, `lastName`, `companyName`, `linkedinUrl`, `personaName`
- camelCase headers only
- For LinkedIn-only leads: leave `email` blank, populate `linkedinUrl` — do not exclude them

**If plain CSV:**
- Headers: `First Name`, `Last Name`, `Email`, `Company Name`, `LinkedIn URL`, `Quality Score`, `Notes`

### Step 9: Return Output

Provide operator:
- Count: "X leads found and exported"
- Link: "Notion view: [URL to filtered view of newly added leads]"
- Or: "CSV attached: [filename]"
- Quality: "Overall email capture: X%. Data quality: [High/Medium/Low]"
- Next step: "Ready to pass to copy-generator for A/B email variants"

---

## Output

1. **Lead records** — written to Notion or exported to CSV
2. **Quality report** — email/LinkedIn/phone capture rates
3. **Source map** — which searches/sources yielded which leads
4. **Cost report** — if enrichment was used (credits, time, cost per lead)
5. **Database view** (for Notion) — filtered to show only newly added leads from this run

---

## Error Handling

**No leads found:**
> "No results on these search patterns. This could mean:
> 1. The persona/Digital Presence combination doesn't exist at scale in these locations
> 2. The search terms need refinement
> 3. The Source Type assumption is wrong
>
> Suggest: Refine persona definition with founder, adjust Digital Presence locations, or switch Source Type to 'Collected — Market Data' if you have customer examples to scrape from."

**Low email capture rate (<30%):**
> "Only X% of leads have emails. Recommend:
> 1. Run enrichment pass (Apollo or Supadata)
> 2. Switch outreach channel to LinkedIn or Instagram if email is unavailable
> 3. Manually research top 10–20 leads for email"

**Notion write fails:**
> "Notion write error. Check: (1) Lead Lists database exists, (2) Run ID field exists and is populated, (3) API token is valid. If all checks pass, try again or export to CSV."

**Search timeout or rate limit:**
> "Search timed out. Suggestion: reduce the target lead count, focus on one Digital Presence location at a time, or use manual directory lookups instead."

---

## Integration Notes

### Input from persona-builder
- Uses Persona record created by persona-builder
- Reads Digital Presence, Source Type, Pain Points

### Output to copy-generator
- Lead list is consumed by copy-generator
- copy-generator reads email addresses and company names for personalization
- copy-generator may reference Pain Points in persona for angle selection

---

*Phase 4 GTM Plugin — Skill 2 of 3*
