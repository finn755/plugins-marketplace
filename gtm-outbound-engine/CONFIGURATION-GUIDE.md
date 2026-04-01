# GTM Outbound Engine — Configuration Guide

This guide walks you through the two-step setup process before launching the GTM Outbound Engine pipeline.

---

## Step 1: Configure Your Search Agent & API Keys

The Lead List Builder skill uses external search agents to find prospects. You must configure at least one agent before running the pipeline.

### Option A: Use Tavly (Free, Recommended for Beginners)

**What is Tavly?**
Tavly is a web search API designed for B2B prospecting. It returns structured company and contact data from public sources (LinkedIn, company websites, directories). Free tier: 100+ searches/month.

**Setup (2 minutes):**

1. Go to **tavly.com**
2. Click **Sign Up** → Create account
3. Navigate to **Settings** or **API** section
4. Copy your **API Key**
5. In Cowork, tell Claude:
   ```
   "I want to use Tavly for lead search. Here's my API key: [paste-your-key]"
   ```

Claude will save the key and load it when Lead List Builder runs.

**When to use Tavly:**
- You want a fast, general-purpose B2B search
- You're targeting companies/roles in standard databases
- You want to start immediately (free)

---

### Option B: Use Exa (Free, Best for Semantic Search)

**What is Exa?**
Exa is a semantic search engine that uses AI to understand intent. It's better for niche, specialized ICPs ("AI founders building SaaS tools" vs. just "SaaS founders"). Free tier: 100+ searches/month.

**Setup (2 minutes):**

1. Go to **exa.ai**
2. Click **Sign Up** → Create account
3. Navigate to **API Keys** or **Settings**
4. Copy your **API Key**
5. In Cowork, tell Claude:
   ```
   "I want to use Exa for lead search. Here's my API key: [paste-your-key]"
   ```

Claude will save the key and load it when Lead List Builder runs.

**When to use Exa:**
- Your ICP is niche or specialized
- You want semantic understanding ("founders obsessed with AI")
- You want better recall for non-standard roles/industries

---

### Option C: Use Supadata (Paid with Credits, Best for Accuracy)

**What is Supadata?**
Supadata is a web scraping + data extraction service. It's more powerful than search engines — it can pull contact pages from websites, extract emails, verify phone numbers, and enrich company data. Supadata usually costs credits, but we've allocated extra credits for this plugin.

**Setup (30 seconds):**

1. In Cowork, tell Claude:
   ```
   "Use Supadata for lead enrichment. I know you have credits configured."
   ```

That's it. No API key needed.

**When to use Supadata:**
- You want the highest-accuracy lead enrichment
- You're targeting organizations (clubs, academies, agencies) where you need contact pages
- You need verified email + phone data
- You have a smaller lead list (Supadata is per-request, not unlimited like free APIs)

---

### How Claude Will Save Your API Key

When you provide an API key, Claude will:

1. **Save it to the Cowork session** — accessible only within this session
2. **Load it automatically** when Lead List Builder runs
3. **Never expose it** in outputs or Notion

If you want to **persist the key across sessions**, tell Claude:
```
"Store this API key in my Cowork vault for future sessions."
```

Claude will save it to your local Cowork settings.

---

## Step 2: Configure Additional Connectors

The plugin integrates with your existing platforms. Most connectors are optional, but **Notion is required**.

### Required: Notion Connector

**What you need:**
- Notion account
- Three databases (Personas, Lead Lists, Campaign Outputs) — instructions below

**Setup (1 minute):**

1. In Cowork, go to **Connectors** (left sidebar)
2. Look for **Notion** in the active connectors list
3. If it appears → ✓ Connected. Done.
4. If it doesn't appear:
   - Click **Connect** or **+ Add Connector**
   - Select **Notion**
   - Click **Authorize** and sign in with your Notion account
   - Approve access

Once connected, Claude can read and write to your Notion workspace.

**Create Your Three Databases:**

Before running any skill, create these databases in Notion. (Takes ~10 minutes.)

**Database 1: Personas**

In Notion, create a new database with these properties:

| Property Name | Type | Mandatory? |
|---|---|---|
| Name | Title | ✓ |
| Industry | Text | ✓ |
| Company Size | Text | ✓ |
| Seniority | Text | ✓ |
| Channel | Text | ✓ |
| Source Type | Select | ✓ |
| Digital Presence | Multi-select | ✓ |
| Pain Points | Rich Text | ✓ |
| Value Hook | Rich Text | ✓ |
| Notes | Rich Text | ✗ |
| Created | Date | ✓ |

**For "Source Type" select, add these options:**
- Research — Assumptions
- Collected — Market Data

**For "Digital Presence" multi-select, add these options:**
- LinkedIn
- Reddit
- Instagram
- Directories
- Podcasts
- Forums
- Website
- Facebook Groups
- Twitter/X

---

**Database 2: Lead Lists**

Create a new database with these properties:

| Property Name | Type | Notes |
|---|---|---|
| Name | Title | Format: "FirstName LastName — Company" |
| Company | Text | Company name |
| Role | Text | Job title or role |
| LinkedIn URL | URL | Link to LinkedIn profile |
| Email | Email | Contact email |
| Phone | Phone | Contact phone |
| Persona | Relation | Links to Personas DB |
| Source | Select | Options: Web Search / Supadata / Apollo / Manual |
| Status | Select | Options: New / Enriched / Loaded to Platform |
| Quality Score | Number | 1-10 rating |
| Notes | Rich Text | Enrichment notes |

---

**Database 3: Campaign Outputs**

Create a new database with these properties:

| Property Name | Type | Notes |
|---|---|---|
| Name | Title | Format: "PersonaName — Date — Version" |
| Persona | Relation | Links to Personas DB |
| Subject Line A | Rich Text | Email subject (variant A) |
| Subject Line B | Rich Text | Email subject (variant B) |
| Email Body A | Rich Text | Email body (variant A) |
| Email Body B | Rich Text | Email body (variant B) |
| Campaign Goal | Select | Options: Demo Request / Call Booking / Pilot Sign-up / Other |
| Lead Count | Number | Number of leads in this campaign |
| Angle Label | Rich Text | Email angle/positioning |
| A/B Test Plan | Rich Text | Testing approach |
| CSV Attached | URL | Link to exported CSV |
| Status | Select | Options: Draft / Ready / Sent |

---

### Optional: Instantly Connector

**What it does:**
Pushes your generated email campaigns directly to Instantly (no manual CSV upload).

**When to connect:**
- You use Instantly for cold email
- You want one-click campaign upload

**Status in v1:**
Not yet implemented. Export CSVs manually. (v2 will add direct API.)

---

### Optional: Lemlist Connector

**What it does:**
Pushes campaigns directly to Lemlist, including LinkedIn sequence setup.

**When to connect:**
- You use Lemlist for cold email + LinkedIn automation
- You want to combine email + LinkedIn sequences in one run

**Status in v1:**
Not yet implemented. Export CSVs manually. (v2 will add direct API.)

---

### Optional: HubSpot Connector

**What it does:**
Imports leads directly to HubSpot as contacts (no manual import step).

**When to connect:**
- You use HubSpot as your CRM
- You want leads auto-synced after each run

**Setup (if you choose this):**

1. In Cowork → Connectors, find **HubSpot**
2. Click **Connect**
3. Authorize with your HubSpot account
4. Tell Claude: "Format lead exports for HubSpot import"

---

## Verification Checklist

Before you say "I'm ready to start", confirm:

- [ ] **Search Agent configured** — You've chosen Tavly, Exa, or Supadata and provided your API key (or confirmed Supadata credits)
- [ ] **Notion connector is active** — It appears in the Cowork connectors list
- [ ] **Three Notion databases created:**
  - [ ] Personas database exists with all required fields
  - [ ] Lead Lists database exists with all required fields
  - [ ] Campaign Outputs database exists with all required fields
- [ ] **Optional connectors connected** (if you use them):
  - [ ] HubSpot (if applicable)
  - [ ] Instantly (if applicable)
  - [ ] Lemlist (if applicable)

Once all items are checked, tell Claude:

```
"Setup complete. I'm ready to launch the GTM Outbound Engine."
```

Claude will confirm all settings and route you to the first skill (Persona Builder).

---

## Troubleshooting Configuration

**"I don't know how to get my Tavly/Exa API key"**

1. Go to tavly.com or exa.ai
2. Sign up for a free account
3. Look for "Settings", "API", "Developer", or "Integrations"
4. You'll see a string that looks like: `sk_live_xyz123abc...`
5. Copy the entire string
6. Paste it in Cowork as: `"Here's my API key: sk_live_xyz123abc..."`

**"I created the Notion databases but Claude can't find them"**

1. Make sure the **Notion connector is active** in Cowork
2. Check that each database has the exact property names (capitalization matters)
3. If still not found, tell Claude: "Search my Notion for databases named 'Personas', 'Lead Lists', and 'Campaign Outputs'"

Claude will manually locate them.

**"I don't want to use Notion — can I use my computer instead?"**

Yes. Tell Claude upfront: "Store everything in [folder path] instead of Notion."

Claude will adapt the workflow to save CSVs and JSON files to your local folder instead.

**"How much will Supadata cost me?"**

Supadata typically costs $0.05–$0.15 per scrape. For a 100-lead list with enrichment, expect $5–$15. We've allocated extra credits, so check with us if you're concerned about cost.

**"Can I switch search agents mid-campaign?"**

Yes. Just tell Claude: "Switch to [Tavly / Exa / Supadata] for the next run."

Your existing leads stay in Notion. New searches will use the new agent.

---

## Next: Launch the Pipeline

Once configuration is complete, tell Claude:

**"I'm ready. Let's build a persona."**

Claude will load the Persona Builder skill and walk you through creating your first ICP.

---

*GTM Outbound Engine — Configuration Guide — Bunker Collective 2026*

