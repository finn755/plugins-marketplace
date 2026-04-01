---
name: launch-outbound-engine
description: >
  Launch the GTM Outbound Engine — the entry point for running your outbound sales campaign. Triggers on "launch outbound engine", "start outbound engine", "outbound engine", "/launch-outbound-engine", "set up outbound engine", "configure outbound engine", or when the user opens the plugin for the first time. Shows the orientation brief, walks through setup configuration, and routes the user to the right skill (persona-builder, lead-list-builder, or copy-generator) based on their stage.
---

# GTM Outbound Engine — Launch & Setup

Welcome. This is the entry point for the GTM Outbound Engine — the three-skill pipeline that takes an ICP definition and produces a cold email campaign ready to upload to Instantly, Lemlist, HubSpot, or Notion.

---

## What This Plugin Does (High Level)

The GTM Outbound Engine automates the entire outbound GTM motion in three steps:

1. **Persona Builder** — Translate your ICP definition into a structured buyer profile (stored in Notion)
2. **Lead List Builder** — Search for, enrich, and validate leads that match your persona
3. **Copy Generator** — Generate A/B subject lines and email bodies, formatted for your platform of choice

Each skill reads the output of the previous one from Notion. The final output is a CSV you upload to your email platform or a Notion record you can review + edit.

---

## How This Works (Step-by-Step)

**You describe your ICP** (natural language, informal) → **Persona Builder** extracts structure and writes to Notion

**You pick a persona** → **Lead List Builder** searches for matching prospects, offers enrichment, writes leads to Notion

**You pick leads + campaign goal** → **Copy Generator** writes A/B copy, exports CSV in your platform's format

Each step produces verifiable output in Notion. You're always in control — Claude doesn't execute searches or send anything without your confirmation.

---

## About Personas: Together, Not Alone

This plugin is **designed to take an already-defined persona**, but you're at full liberty to **go back and forth with Claude to create a defined persona together**.

If you're not sure exactly who your ICP is, that's fine. The Persona Builder skill will:
- Ask you to describe your ICP in plain English ("we're targeting VP Sales at mid-market B2B SaaS")
- Extract key attributes and infer missing dimensions
- Present back for your confirmation and refinement
- Let you iterate until it matches your intent

You don't need to have a perfect, detailed persona definition upfront. **Vague intent is OK** — the skill will sharpen it with you in conversation.

---

## About Storage: Notion, or Your Computer

This plugin is **designed to use Notion as the staging area for file inputs and outputs**. All three skills read from and write to your Notion workspace (Personas database → Lead Lists database → Campaign Outputs database).

**However, if you ask differently, you can use folders on your computer instead.** Just explicitly give Claude this information, and the skills will adapt:
- Instead of writing to Notion, save CSVs and JSON to a folder you specify
- Instead of reading personas from Notion, upload a CSV or JSON file
- The workflow stays the same, just the storage layer changes

**Example:** "I don't want to use Notion — can we save everything to my Desktop/outbound-campaign folder instead?"

The plugin will handle it. Just be explicit upfront.

---

## Before You Start: Setup Configuration

Before launching the pipeline, you need to configure two things:

### Configuration 1: Choose Your Search Agents & API Keys

The **Lead List Builder** uses search agents to find prospects. You have three options:

| Agent | Cost | Setup | When to use |
|---|---|---|---|
| **Tavly** | Free tier available | Free API key from tavly.com | General B2B prospect search, fast |
| **Exa** | Free tier available | Free API key from exa.ai | Semantic search, good for niche ICPs |
| **Supadata** | Paid, but we have credits configured | Already set up in the plugin | Website scraping, org contact pages, high accuracy for enrichment |

**How to Configure Your Search Agent:**

1. **Decide which agent to use:**
   - **Free + Good:** Tavly or Exa (both have 100+ free searches/month)
   - **Best Accuracy:** Supadata (we have extra credits allocated; no setup needed)

2. **Get your API key (if using Tavly or Exa):**
   - Tavly: Visit tavly.com → Sign up → Copy API key
   - Exa: Visit exa.ai → Sign up → Copy API key
   - Supadata: No action needed — we've already configured credits

3. **Tell Claude your choice:**
   ```
   "I want to use [Tavly / Exa / Supadata] for lead search."

   If Tavly/Exa: "Here's my API key: [paste-key-here]"
   If Supadata: "Use Supadata — I know you have credits configured"
   ```

4. **Claude will:**
   - Save your API key securely to the session
   - Load it automatically when Lead List Builder runs
   - Use it to search for prospects matching your persona

**Where to find secure key storage:**
If you want to store API keys persistently:
- **Cowork Sessions:** Tell Claude "Store this key in my session vault" — it saves to your local Cowork settings
- **Environment:** Keys can be stored in `.env` files in your project folder (Claude will guide you)
- **Secure Note:** Write keys down in a secure note app (1Password, Bitwarden, etc.) and reference them when asked

---

### Configuration 2: Connect Additional Platforms (Optional)

The plugin can integrate with your existing email/CRM platforms. Optional connectors:

| Connector | Purpose | When to connect |
|---|---|---|
| **Notion** (required) | Store personas, leads, campaign outputs | Always — this is the pipeline hub |
| **Instantly** | Direct API push for campaigns | If you use Instantly for cold email |
| **Lemlist** | Direct API push for campaigns | If you use Lemlist for cold email + LinkedIn sequences |
| **HubSpot** | Import contacts to your CRM | If you want leads automatically added to HubSpot |

**How to Configure Connectors:**

1. **Check which connectors are active:**
   Ask Claude: **"Which connectors are currently connected in this Cowork session?"**
   Claude will check the Cowork connector panel and tell you what's available.

2. **Ensure Notion is connected:**
   - Go to **Cowork → Connectors** (left sidebar)
   - Confirm **Notion** appears in the active connectors list
   - If not, click **Connect** and authenticate with your Notion account

3. **Connect additional platforms (optional):**
   - For **Instantly:** Not required in v1. Export CSVs manually. (v2 will have direct API)
   - For **Lemlist:** Not required in v1. Export CSVs manually. (v2 will have direct API)
   - For **HubSpot:** If you want contacts auto-imported, tell Claude: "Connect HubSpot and format exports for HubSpot import"

4. **Confirm configuration:**
   Once done, Claude will say: "Connectors configured. We're ready to launch."

---

## Ready to Launch?

Once you've completed the two setup steps above, you're ready to start the pipeline.

### Next Steps (Pick One)

**If you're just starting:**
Say: **"Let's build a persona"** or **"I'm ready — launch the persona builder"**

**If you already have a persona defined in Notion:**
Say: **"Find leads for [persona name]"** or **"Build a lead list"**

**If you have leads and want to write copy:**
Say: **"Generate email copy"** or **"Create campaign copy"**

---

## Quick Database Reference

If this is your first time, you need to set up three databases in Notion (takes ~10 minutes):

### Personas Database
```
- Name (title)
- Industry (text)
- Company Size (text)
- Seniority (text)
- Channel (text)
- Source Type (select: "Research — Assumptions" / "Collected — Market Data")
- Digital Presence (multi-select: LinkedIn, Reddit, Instagram, Directories, Podcasts, Forums, Website, Facebook Groups, Twitter/X)
- Pain Points (rich text)
- Value Hook (rich text)
- Notes (rich text)
- Created (date)
```

### Lead Lists Database
```
- Name (title)
- Company (text)
- Role (text)
- LinkedIn URL (URL)
- Email (email)
- Phone (phone)
- Persona (relation → Personas DB)
- Source (select: Web Search / Supadata / Apollo / Manual)
- Status (select: New / Enriched / Loaded)
- Quality Score (number)
- Notes (rich text)
```

### Campaign Outputs Database
```
- Name (title)
- Persona (relation → Personas DB)
- Subject Line A (rich text)
- Subject Line B (rich text)
- Email Body A (rich text)
- Email Body B (rich text)
- Campaign Goal (select: Demo / Call / Pilot / Other)
- Lead Count (number)
- Angle Label (rich text)
- A/B Test Plan (rich text)
- CSV Attached (URL)
- Status (select: Draft / Ready / Sent)
```

---

## Troubleshooting

**"I don't have Notion databases set up"**
→ See the "Quick Database Reference" above. Takes ~10 minutes. Then come back.

**"I'm not sure which search agent to use"**
→ Start with **Exa** (free, semantic search, good for niche ICPs). You can always switch.

**"I want to use my computer instead of Notion"**
→ Tell Claude: "Store everything in [folder path] instead of Notion." The workflow adapts.

**"What if I don't have API keys yet?"**
→ Get free keys from Tavly or Exa (takes 2 minutes). Or use Supadata (we have credits).

---

## Next: Choose Your Configuration Path

Pick the one that matches your setup:

**Path A: "I have Notion + API key ready"**
- Tell Claude which search agent + key
- Confirm Notion connector is active
- Say "Let's build a persona"

**Path B: "I have Notion but no API key yet"**
- Tell Claude "I'm using Tavly" or "I'm using Exa"
- Get a free key (2 min) and come back
- Then follow Path A

**Path C: "I want to use my computer, not Notion"**
- Tell Claude: "Store outputs in [folder path] instead of Notion"
- Provide your folder path
- Then follow Path A or B

**Path D: "I don't have Notion yet"**
- Create the three databases (~10 min)
- Then follow Path A

---

*GTM Outbound Engine — Launch & Setup — Bunker Collective 2026*

