# GTM Outbound Engine — Quick Reference Card

**Print this. Pin it. Reference it.**

---

## Launch the Plugin

Say any of these:
- "launch outbound engine"
- "set up outbound engine"
- "start outbound engine"
- "configure outbound engine"

---

## The 3-Skill Pipeline

```
ICP Description
    ↓
[1] PERSONA BUILDER
    ↓
Persona in Notion
    ↓
[2] LEAD LIST BUILDER
    ↓
Leads in Notion
    ↓
[3] COPY GENERATOR
    ↓
CSV + Campaign in Notion
```

---

## Pre-Flight: Two Things to Set Up

### 1. Pick a Search Agent

| Agent | Cost | Setup | When |
|---|---|---|---|
| Tavly | Free | 2 min | General B2B search |
| Exa | Free | 2 min | Niche/semantic search |
| Supadata | Paid (credits allocated) | 30 sec | High accuracy |

**Tell Claude:**
- If Tavly/Exa: "I want [Agent]. Here's my API key: [paste]"
- If Supadata: "Use Supadata — I have credits"

### 2. Verify Notion Setup

| Check | How |
|---|---|
| Connector active? | Cowork → Connectors → Notion |
| Personas DB exists? | Look for it in your Notion workspace |
| Lead Lists DB exists? | Look for it |
| Campaign Outputs DB exists? | Look for it |
| Fields correct? | See CONFIGURATION-GUIDE.md for exact schema |

**If any DB is missing:** Create it in Notion (takes ~10 min). See CONFIGURATION-GUIDE.md for exact field names and types.

---

## Running the Pipeline

### Step 1: Build Persona

Say: **"Build a persona"** or **"Define my ICP"**

Claude will ask:
- What venture?
- Who's your ideal customer? (natural language description)
- Is this research/assumptions or real data?
- Where do they live online? (LinkedIn, Reddit, etc.)

Claude writes a structured persona to your Personas DB in Notion.

**Time:** 5–10 minutes

---

### Step 2: Find Leads

Say: **"Build a lead list"** or **"Find leads for [persona name]"**

Claude will ask:
- Which persona?
- How many leads?
- Outreach channel? (Email / LinkedIn / Cold Call)
- Export to? (Notion / CSV / HubSpot)

Claude searches using your configured agent, offers enrichment, writes leads to Lead Lists DB.

**Time:** 5–15 minutes (depending on search source)

**Pro tip:** If you chose Supadata, enable enrichment. Higher accuracy for email/phone.

---

### Step 3: Generate Copy

Say: **"Generate email copy"** or **"Create campaign copy"**

Claude will ask:
- Which persona?
- Campaign goal? (Demo request / Call / Pilot / Other)
- Export format? (Instantly / Lemlist / HubSpot / Notion only)

Claude generates A/B subject lines + email bodies, writes to Campaign Outputs DB, exports CSV.

**Time:** 3–5 minutes

**Output formats:**
- Instantly: PascalCase columns (Email, FirstName, LastName, etc.)
- Lemlist: camelCase columns (email, firstName, lastName, etc.)
- HubSpot: snake_case columns (first_name, last_name, etc.)

---

## Notion Database Schema (Cheat Sheet)

### Personas DB Properties

```
Name (title)
Industry (text)
Company Size (text)
Seniority (text)
Channel (text)
Source Type (select: Research — Assumptions / Collected — Market Data)
Digital Presence (multi-select: LinkedIn, Reddit, Instagram, Directories, Podcasts, Forums, Website, Facebook Groups, Twitter/X)
Pain Points (rich text)
Value Hook (rich text)
Notes (rich text)
Created (date)
```

### Lead Lists DB Properties

```
Name (title)
Company (text)
Role (text)
LinkedIn URL (URL)
Email (email)
Phone (phone)
Persona (relation → Personas DB)
Source (select: Web Search / Supadata / Apollo / Manual)
Status (select: New / Enriched / Loaded)
Quality Score (number)
Notes (rich text)
```

### Campaign Outputs DB Properties

```
Name (title)
Persona (relation → Personas DB)
Subject Line A (rich text)
Subject Line B (rich text)
Email Body A (rich text)
Email Body B (rich text)
Campaign Goal (select: Demo / Call / Pilot / Other)
Lead Count (number)
Angle Label (rich text)
A/B Test Plan (rich text)
CSV Attached (URL)
Status (select: Draft / Ready / Sent)
```

---

## Troubleshooting (One-Liners)

| Problem | Solution |
|---|---|
| "API key error" | Confirm your API key is copied correctly (no extra spaces) |
| "Can't find Notion databases" | Confirm Notion connector is active in Cowork |
| "Persona has no leads" | Run Lead List Builder first; check the Run ID matches |
| "Leads missing emails" | Re-run Lead List Builder with Supadata enrichment |
| "CSV upload fails" | Check column names match your platform (PascalCase vs camelCase) |
| "I want to use my computer instead" | Tell Claude: "Save to [folder path] instead of Notion" |

**Full troubleshooting:** See TROUBLESHOOTING.md and CONFIGURATION-GUIDE.md

---

## Keyboard Shortcuts / Trigger Phrases

| What You Want | Say This |
|---|---|
| Start the plugin | "launch outbound engine" |
| Build a persona | "build a persona" or "persona-builder" |
| Find leads | "build a lead list" or "find leads" |
| Write copy | "generate email copy" or "copy-generator" |
| Show me my config | "what are my settings?" or "check my configuration" |
| Use a different search agent | "switch to Exa" or "use Supadata" |
| Export to a different platform | "export for Lemlist instead" |
| Re-run a step | "rebuild the lead list" or "regenerate copy" |

---

## API Key Storage Options

| Option | Security | Access | Best For |
|---|---|---|---|
| **Cowork session** | Session-local | This session only | Testing, single-user |
| **.env file** | Local, encrypted | Your computer | Long-term, local dev |
| **Secure note app** | Encrypted at rest | Manual copy-paste | Team sharing (share the app) |
| **Vault/secrets manager** | Highly secure | API access | Production, teams |

**Default:** Tell Claude your key → Cowork session → auto-load next run

**Persistent:** Tell Claude "Store this in my Cowork vault" → Cowork remembers it

---

## Notion Run ID Format

Every campaign gets a Run ID for tracking:

```
Format: [venture-slug]-[persona-slug]-[YYYY-MM-DD]

Example: coachology-vp-sales-2026-04-01
         dealdecoder-golf-coaches-2026-04-15
         aventi-executive-coaches-2026-04-20
```

You'll see this in:
- Lead Lists DB (Run ID field)
- Campaign Outputs DB (to link back to leads)

Use it to track which leads + copy belong to which campaign run.

---

## Enrichment Options

When Claude offers enrichment in Lead List Builder:

| Tool | Cost | Data | When to use |
|---|---|---|---|
| Supadata | $0.05–$0.15/lead | Email, phone, verified | Organizations, high accuracy |
| Apollo.io | Paid | B2B contact data | LinkedIn profiles, B2B only |
| Manual | Free | Copy/paste | Small lists, hands-on |

**Recommendation:** If using Supadata agent, enable enrichment for same-source data. If using Tavly/Exa, consider Apollo enrichment for B2B.

---

## Copy Output Specs

Email bodies generated by Copy Generator:

- **Length:** 25–75 words (short, punchy)
- **Structure:** Hook → Problem → CTA
- **CTA:** Binary (2 options: yes/no or demo/pass)
- **No em-dashes:** Use hyphens (—) only in persona names
- **Prospect-first:** Focus on their problem, not your product

Example:
```
Subject A: Your rep turnover rate is your #1 revenue leak

Subject B: Why golf coaches are losing players to Coachology

Body A: [Hook on their pain] [Your solution] [Binary CTA]

Body B: [Different angle] [Different proof point] [Same CTA]
```

---

## Export Formats by Platform

### Instantly CSV

```
Email,FirstName,LastName,CompanyName,SubjectA,BodyA,SubjectB,BodyB
john@company.com,John,Doe,Acme Corp,[Subject],[Body],[Subject],[Body]
```

### Lemlist CSV

```
email,firstName,lastName,companyName,subjectA,bodyA,subjectB,bodyB
john@company.com,John,Doe,Acme Corp,[Subject],[Body],[Subject],[Body]
```

### HubSpot CSV

```
first_name,last_name,email,company_name
John,Doe,john@company.com,Acme Corp
```

**Note:** Column name capitalization matters. Copy Generator outputs the exact format for your platform.

---

## One More Thing: Personas Are Drafts

You don't need a perfect ICP before starting. The Persona Builder works like this:

1. You describe it roughly ("VP Sales at B2B SaaS")
2. Claude extracts attributes and infers gaps
3. Claude presents back for your feedback
4. You refine together until it's right
5. Claude writes to Notion

The persona is **not locked**. You can:
- Edit the Notion record directly
- Re-run Persona Builder with refinements
- Change pain points / value hooks / digital presence anytime

Leads + copy will adapt.

---

## When to Use This Card

- **First launch:** Keep this visible until you've run all 3 skills once
- **Second+ campaign:** Quick reference for trigger phrases + format specs
- **Troubleshooting:** Jump to "Troubleshooting" section for quick fixes
- **Team reference:** Print + post on your team Slack/Notion

---

*GTM Outbound Engine — Quick Reference Card — Bunker Collective 2026*

