# GTM Outbound Engine

**Version:** 0.2.0
**Author:** Bunker Collective
**Built:** 2026-04-01
**Updated:** Added launch onboarding flow and configuration guides

A four-skill pipeline that takes an ICP brief and produces a lead list + A/B cold email copy ready to upload to Instantly, Lemlist, HubSpot, or Notion.

---

## Getting Started — Launch the Plugin

**First time using this plugin?** Say any of these:

- **"launch outbound engine"**
- **"set up outbound engine"**
- **"start outbound engine"**
- **"configure outbound engine"**

The **launch-outbound-engine** skill will:

1. Welcome you and explain what the plugin does
2. Walk you through the setup process (API keys, connectors)
3. Route you to the right skill based on where you are
4. Answer any questions about storage (Notion vs. your computer)

---

## What This Plugin Does

Run the four skills in order:

```
1. launch-outbound-engine  →  Orientation + Setup
2. persona-builder         →  Notion Personas DB
3. lead-list-builder       →  Notion Lead Lists DB  (linked to persona)
4. copy-generator          →  Notion Campaign Outputs DB  +  CSV export
```

Each skill reads the output of the previous one from Notion. The final output is a campaign-ready CSV in the format required by your chosen platform.

---

## Skills

| Skill | Trigger phrases | What it does |
|---|---|---|
| **launch-outbound-engine** | "launch outbound engine", "set up outbound engine", "start outbound engine" | Orientation, setup configuration, and routing to the right skill |
| **persona-builder** | "build a persona", "define my ICP", "create a persona" | Takes a natural language ICP brief → writes a structured persona record to Notion |
| **lead-list-builder** | "build a lead list", "find leads for this persona", "generate prospects" | Reads persona → searches for matching leads → offers enrichment → writes leads to Notion |
| **copy-generator** | "generate email copy", "write cold emails", "copy generator" | Reads persona + leads → generates A/B cold email copy → exports to your chosen platform |

---

## Quick Setup Summary

### Step 1: Configure Your Search Agent

Choose one of three options:

| Agent | Cost | Setup Time | When to use |
|---|---|---|---|
| **Tavly** | Free | 2 minutes | General B2B prospect search |
| **Exa** | Free | 2 minutes | Semantic search, niche ICPs |
| **Supadata** | Paid (credits allocated) | 30 seconds | Website scraping, highest accuracy |

Get a free API key from **tavly.com** or **exa.ai**, then tell Claude:
```
"I want to use [Tavly/Exa/Supadata]. Here's my API key: [paste-key]"
```

Or if using Supadata:
```
"Use Supadata — I know you have credits configured"
```

**Full guide:** See `CONFIGURATION-GUIDE.md`

### Step 2: Verify Notion Setup

Create three databases in your Notion workspace (takes ~10 minutes):

- **Personas DB** — Store your ICPs and target audiences
- **Lead Lists DB** — Store prospects matching your personas
- **Campaign Outputs DB** — Store generated email copy and campaign metrics

**Exact schema and property names:** See `CONFIGURATION-GUIDE.md`

### Step 3: Confirm Notion Connector is Active

In Cowork → Connectors, verify **Notion** appears in the active connectors list. If not, click **Connect** and authenticate.

### Step 4: Launch the Pipeline

Say: **"launch outbound engine"**

Claude will confirm your setup and route you to the first skill.

---

## Running the Pipeline

### Run 1: Build Your Persona

Say: **"build a persona"** or **"persona-builder"**

The skill will ask:
- What venture is this for?
- Describe your ICP in natural language (who they are, industry, size, pain)
- Is this based on research/assumptions or data you've collected?
- Where do they live online? (LinkedIn, directories, Reddit, etc.)

It writes the persona to your Notion Personas DB and returns the page URL.

---

### Run 2: Build Your Lead List

Say: **"build a lead list"** or **"find leads for this persona"**

The skill will ask:
- Which persona?
- How many contacts do you want?
- Which outreach channel? (Email / LinkedIn / Instagram / Cold Call)
- Where are you sending? (Instantly / Lemlist / HubSpot / Notion / CSV)

It searches for leads, returns a quality report (% with email, phone, LinkedIn), then offers enrichment. You decide whether to enrich and which tool to use. Leads are written to your Notion Lead Lists DB.

> **Credit tip:** If using Supadata for enrichment, deduplicate leads first. The skill does this automatically — it follows the supadata-credit-efficiency rules built in.

---

### Run 3: Generate Copy

Say: **"generate email copy"** or **"copy-generator"**

The skill will ask:
- Which persona?
- What's the campaign goal? (demo request, intro call, etc.)
- Where are you sending? (Instantly / Lemlist / Notion / HubSpot / Both)

It reads your persona and leads from Notion, generates A/B subject lines and email bodies (25–75 words, prospect-first, binary CTA, no em-dashes), writes the Campaign Outputs record to Notion, and exports only the CSV format(s) you selected.

---

## Before You Run Your First Skill — Validation Checklist

Work through this list top to bottom before triggering any skill. Each item takes less than 2 minutes to check. Skipping steps is the most common cause of failures.

### Notion setup
- [ ] **Personas database exists** in your venture's Notion workspace, with the following property types confirmed:
  - `Name` (title), `Industry` (text), `Company Size` (text), `Seniority` (text), `Channel` (text)
  - `Source Type` (select) with exactly two options: `Research — Assumptions` and `Collected — Market Data`
  - `Digital Presence` (multi-select) with options: LinkedIn, Reddit, Instagram, Directories, Podcasts, Forums, Website, Facebook Groups, Twitter/X
  - `Pain Points` (rich text), `Value Hook` (rich text), `Notes` (rich text), `Created` (date)
- [ ] **Lead Lists database exists** with a `Run ID` property (text type, not a relation)
- [ ] **Campaign Outputs database exists** with `Run ID`, `Status` (select), `Angle Label`, and `Export Date` fields
- [ ] **Claude integration is connected to all three databases:** Open each DB → `...` → Connections → confirm the integration appears

### Cowork session
- [ ] **Notion MCP connector is active** — visible in the connector panel on the left side of your Cowork session
- [ ] **You know your venture slug** (lowercase, hyphenated) — e.g. `aventi`, `coachology`, `dealdecoder`. This is used in Run IDs.

### Before lead-list-builder
- [ ] A persona record already exists in the Personas DB (run persona-builder first)
- [ ] You know your target lead count, outreach channel, and export destination

### Before copy-generator
- [ ] Leads have been written to the Lead Lists DB (run lead-list-builder first)
- [ ] You know the Run ID from the lead-list-builder run (format: `[venture]-[persona]-[YYYY-MM-DD]`)
- [ ] You have confirmed your campaign goal (demo request / intro call / pilot sign-up)

**If anything above is missing, fix it before running a skill.** See TROUBLESHOOTING.md for step-by-step fixes.

---

## Setup — Do This Before Running Any Skill

### Step 1 — Credentials You Need

| Credential | Where to get it | Required for |
|---|---|---|
| **Notion API token** | notion.so/my-integrations → New integration → copy Internal Integration Token | All skills |
| **Notion database IDs** | See Step 2 below | All skills |
| **Search Agent API key** | tavly.com or exa.ai → Sign up → copy API key | Lead List Builder |
| **Supadata API key** | supadata.ai → account → API key | Optional — lead enrichment (higher quality leads) |
| **Apollo.io account** | apollo.io | Optional — B2B email/phone enrichment |
| **Instantly account** | instantly.ai | Optional — email campaign upload |
| **Lemlist account** | lemlist.com | Optional — email + LinkedIn sequence upload |

No API keys needed for Instantly or Lemlist in v1 — the skills export a CSV you upload manually. API push is v2.

---

### Step 2 — Set Up Your Notion Venture Workspace

For each venture you want to run outbound for:

1. Open Notion → go to **Projects → Ventures**
2. Duplicate the **Persona Template** database (already built — full schema, ready to use)
3. Create a **Lead Lists** database with these properties:
   - `Name` (title) — format: `[First] [Last] — [Company]`
   - `Company` (rich text), `Role` (rich text), `LinkedIn URL` (URL), `Email` (email), `Phone` (phone)
   - `Persona` (relation → your Personas DB)
   - `Source` (select): Web Search / Supadata / Apollo / Manual
   - `Status` (select): New / Enriched / Loaded to Platform
   - `Quality Score` (number), `Notes` (rich text)

4. Create a **Campaign Outputs** database with these properties:
   - `Name` (title) — format: `[Persona] — [Date] — [Version]`
   - `Persona` (relation → Personas DB)
   - `Subject Line A`, `Subject Line B`, `Email Body A`, `Email Body B` (rich text)
   - `Campaign Goal` (select): Demo Request / Call Booking / Pilot Sign-up / Other
   - `Lead Count` (number), `Angle Label` (rich text), `A/B Test Plan` (rich text)
   - `Instantly Format?` (checkbox), `Lemlist Format?` (checkbox)
   - `CSV Attached` (URL), `Status` (select): Draft / Ready / Sent

5. Copy the database ID from each database URL (the string between the last `/` and `?v=`). You'll have three IDs:
   ```
   Personas DB ID:          [paste here]
   Lead Lists DB ID:        [paste here]
   Campaign Outputs DB ID:  [paste here]
   ```

6. Add your Notion integration to each database:
   Open the database → Share → Add connections → select your integration

---

### Step 3 — Connect Your Notion Integration in Cowork

Make sure the Notion MCP connector is active in your Cowork session. If it isn't showing, go to Cowork settings → Connectors and connect Notion.

---

## Output Formats

| Destination | CSV format | Column case | API in v1? |
|---|---|---|---|
| Instantly | `Email, FirstName, LastName, CompanyName, SubjectA, BodyA, SubjectB, BodyB` | PascalCase | No — manual upload |
| Lemlist | `email, firstName, lastName, companyName, subjectA, bodyA, subjectB, bodyB` | camelCase | No — manual upload |
| HubSpot | `first_name, last_name, email, company_name` (contact fields only) | snake_case | No — manual import |
| Notion only | No CSV | — | — |

---

## Enrichment Tools by Persona Type

| Persona type | Recommended enrichment | Why |
|---|---|---|
| Orgs / clubs / institutions | Supadata website scrape | Contact pages, emails, phone numbers |
| B2B contacts | Apollo.io | Verified email + phone by name + company |
| Professional / startup founders | LinkedIn | Role verification, current company |
| Email campaigns (any) | Lemlist Finder | Verified email discovery |

---

## Storage Options

### Default: Notion
- Structured, queryable, shareable
- All three databases store personas, leads, copy
- Easy to iterate and refine manually

### Alternative: Your Computer
- Local files (CSVs and JSON)
- Full privacy, no external storage
- Tell Claude upfront: "Save everything to [folder path] instead of Notion"
- The entire workflow adapts — no skill changes needed

---

## Troubleshooting

For comprehensive failure scenarios with step-by-step fixes, see **TROUBLESHOOTING.md** (included in this plugin).

Quick reference:

**"Persona has no leads"** — Run lead-list-builder first. Confirm leads in your Lead Lists DB have a Run ID matching the persona run.

**"Leads missing required fields"** — Re-run lead-list-builder with enrichment, or manually fill in the missing fields in Notion before running copy-generator.

**CSV upload fails in Instantly** — Check: (1) Email is the first column, (2) no nulls in the Email column, (3) UTF-8 encoding, (4) column names are PascalCase exactly as shown in Output formats above.

**Notion write fails** — Verify your integration has been added to the specific database (Share → Connections). Check that all required properties exist. See TROUBLESHOOTING.md → S2 for full fix.

**Run ID not found** — Run IDs follow the format `[venture-slug]-[persona-slug]-[YYYY-MM-DD]`. Ask Claude: "Search my Lead Lists DB for records containing [venture name]" to retrieve it. See TROUBLESHOOTING.md → O1.

---

## Documentation

- **QUICK-REFERENCE.md** — Printable quick reference card (database schemas, trigger phrases, troubleshooting)
- **CONFIGURATION-GUIDE.md** — Detailed setup instructions (search agents, connectors, database schema)
- **TROUBLESHOOTING.md** — Comprehensive failure scenarios and fixes

---

## What's Next (v2)

- Direct API push to Instantly (1000 leads per call — currently CSV only)
- Lemlist API integration (single-lead endpoint, needs rate-limit handling)
- Scheduled runs (weekly lead refresh per persona)
- Reply rate feedback loop back to persona card

---

## Plugin Versions

| Version | Date | What Changed |
|---|---|---|
| 0.1.0 | 2026-03-26 | Initial release: 3 skills (persona-builder, lead-list-builder, copy-generator) |
| 0.2.0 | 2026-04-01 | Added launch onboarding flow, configuration guide, quick reference card |

---

*GTM Outbound Engine v0.2.0 — Bunker Collective, April 2026*
