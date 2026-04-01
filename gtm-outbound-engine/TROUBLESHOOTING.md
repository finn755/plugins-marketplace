# GTM Outbound Engine — Troubleshooting Guide

This file covers the most common failure modes across all three skills. For each issue: what you see, why it happens, and how to fix it.

---

## Setup Failures

### S1 — "I can't find the Personas database in Notion"

**Symptom:** The skill asks which persona to use and you can't locate a Personas DB, or the skill returns an error when trying to fetch personas.

**Cause:** The database hasn't been created yet, or it was created under the wrong workspace/team space.

**Fix:**
1. In Notion, navigate to the TeamSpace where you want to run campaigns (e.g. Bunker Base → Ventures → [Venture Name])
2. Create a new full-page database called **Personas**
3. Add the 11 required fields listed in persona-builder's Pre-flight section (Name, Industry, Company Size, Seniority, Channel, Source Type, Digital Presence, Pain Points, Value Hook, Notes, Created)
4. Confirm the Source Type field is a **Select** property with exactly two options: `Research — Assumptions` and `Collected — Market Data`
5. Confirm Digital Presence is a **Multi-select** property

---

### S2 — "The skill says it can't write to Notion"

**Symptom:** After completing the persona questionnaire or lead search, the Notion write step fails with a permission or connection error.

**Cause:** The Notion MCP is not connected, or the integration doesn't have access to the database you're trying to write to.

**Fix:**
1. Check that the Notion connector is active in your Cowork session (visible in the connector panel)
2. In Notion, open the database → click `...` → `Connections` → confirm the Claude integration is listed
3. If the integration isn't listed, add it: Settings → Connections → find Claude/Anthropic → grant access
4. Retry the skill from the last confirmed step

---

### S3 — "The Lead Lists database doesn't have a Run ID field"

**Symptom:** The skill tries to write leads with a Run ID but the field doesn't exist, or the write partially fails.

**Cause:** The Lead Lists DB was created without the Run ID field, or the field was named differently.

**Fix:**
1. Open the Lead Lists database in Notion
2. Add a new property: **Text** type, named exactly `Run ID`
3. The Run ID format is `[venture-slug]-[persona-slug]-[YYYY-MM-DD]` (e.g. `aventi-venue-operators-2026-03-26`)
4. Do not use a relation field — Run ID is a plain text string, not a linked relation

---

### S4 — "I don't know which databases exist in my Notion"

**Symptom:** You're not sure if you've set up the right databases, or the skill is asking for a database that you can't identify.

**Fix:**
1. Ask Claude: "Search my Notion for databases named Personas, Lead Lists, and Campaign Outputs"
2. Claude will use the Notion MCP to list available databases
3. If a database is missing, follow the setup instructions in README.md → Notion Setup

---

## Search Failures (lead-list-builder)

### L1 — "WebSearch returned no leads / the results don't match my persona"

**Symptom:** Step 3 completes but returns 0 leads, or returns results completely unrelated to the persona.

**Cause:** Search queries were too narrow, or the persona's Digital Presence locations don't match where they actually congregate.

**Fix:**
1. Ask Claude to show you the exact search queries it ran
2. Try broadening: remove geographic constraints, simplify the role title (e.g. "AV company director" → "AV hire company UK")
3. Consider whether your Digital Presence selection is accurate — if you selected LinkedIn but this persona is mostly found via directories, switch the location
4. For UK-specific personas, add "UK" explicitly to queries rather than relying on geographic inference

---

### L2 — "All leads came back LinkedIn-only — no emails found"

**Symptom:** The quality report shows 80-100% LinkedIn-only leads with near-zero email capture.

**Cause:** This persona's contact info is not publicly available through web search. Common for corporate-facing B2B roles where email addresses aren't listed on websites.

**Fix (choose one or combine):**
- **Switch channel:** If Lemlist is your sequencing tool, LinkedIn-only leads are still usable — they'll receive LinkedIn connection/message steps only, skipping email steps automatically
- **Enrich with Apollo:** Apollo is the best tool for finding B2B email addresses from names + company domains. Select "Apollo" at the enrichment gate
- **Enrich with Lemlist Finder:** If you have a Lemlist account, Lemlist's email finder works well from LinkedIn URLs
- **Supadata about/team pages:** For company-level email capture (not personal), scraping the company's `/about` or `/team` page finds director names and sometimes personal emails

---

### L3 — "Supadata scrape returned an error or empty result"

**Symptom:** During enrichment, Supadata returns an error, a 404, or the scraped page has no useful content.

**Cause:** The page URL is incorrect, the company's website blocks scraping, or the about/team page uses a JavaScript-rendered layout that Supadata can't read.

**Fix:**
1. Verify the URL manually — open it in a browser and confirm the page exists and has content
2. Try the `/contact` page instead of `/about` or `/team`
3. If the page is JavaScript-rendered (content only appears after a moment of loading), Supadata's scraper can't read it — mark this lead as "manual enrichment needed" in Notes and move on
4. Do not retry the same URL more than once — it costs a credit each time

---

### L4 — "The skill keeps searching the same source and won't move to the next location"

**Symptom:** The skill appears stuck on one Digital Presence location and isn't progressing through the others.

**Fix:** This is a model reasoning quirk. Simply tell Claude: "Move on to the next Digital Presence location" or "Skip LinkedIn and search directories instead." The skill will continue from the next source.

---

## Export Failures (lead-list-builder / copy-generator)

### E1 — "The CSV file looks broken when I open it — columns are misaligned"

**Symptom:** Opening the CSV in Excel or Google Sheets shows data in wrong columns, or the file appears as a single column.

**Cause:** The CSV contains commas inside email body text that weren't properly quoted.

**Fix:**
1. Ask Claude to regenerate the CSV with all text fields wrapped in double quotes
2. When importing into Google Sheets: File → Import → select the file → confirm "Comma" as separator → check "Convert text to numbers" is OFF
3. If the copy columns (subjectA, bodyA) contain line breaks, these need to be replaced with spaces before export — ask Claude: "Remove all line breaks from the copy columns in the CSV"

---

### E2 — "Lemlist says my column names don't match / import fails"

**Symptom:** Importing the Lemlist CSV shows mapping errors or the wrong columns are matched.

**Cause:** Lemlist requires camelCase column names exactly: `email`, `firstName`, `lastName`, `companyName`, `linkedinUrl`. Any variation (capitalisation, underscores, spaces) will fail.

**Fix:**
1. Ask Claude: "Show me the column headers from the CSV"
2. Confirm they match exactly: `email`, `firstName`, `lastName`, `companyName`, `linkedinUrl`, `personaName`
3. If headers are wrong, ask Claude: "Rename the CSV headers to Lemlist camelCase format and regenerate the file"

---

### E3 — "Instantly says my column names don't match / import fails"

**Symptom:** Importing the Instantly CSV shows column mapping errors.

**Cause:** Instantly requires PascalCase: `Email`, `FirstName`, `LastName`, `CompanyName`, `LinkedinUrl`.

**Fix:**
1. Ask Claude: "Show me the column headers from the CSV"
2. Confirm PascalCase format
3. Ask Claude: "Rename headers to Instantly PascalCase format and regenerate"

---

### E4 — "The copy in the CSV has em-dashes (—) and I need to fix them"

**Symptom:** You notice em-dashes in the subject lines or email bodies — Lemlist/Instantly may render these incorrectly in some email clients.

**Fix:** Ask Claude: "Replace all em-dashes in the copy columns with hyphens and regenerate the CSV." This is a known formatting issue and is flagged in the quality checklist inside copy-generator.

---

### E5 — "Campaign Outputs status is still showing 'Draft' after export"

**Symptom:** You've exported the CSV, but the Notion Campaign Outputs record still shows Status = Draft.

**Cause:** The status update is a separate step that requires the operator to confirm export completion.

**Fix:** Tell Claude: "Mark the [Run ID] campaign output as Exported in Notion." Claude will update the Status field to `Exported` and set the Export Date to today.

---

## Operational Confusion

### O1 — "I can't find the Run ID from a previous session"

**Symptom:** You want to re-run copy-generator for a campaign that was run earlier, but you don't know the Run ID.

**Fix:**
1. Ask Claude: "Search Notion Lead Lists for records with a Run ID containing [venture name]"
2. Claude will return matching records and their Run IDs
3. Run IDs follow the format `[venture-slug]-[persona-slug]-[YYYY-MM-DD]` — if you know the venture and approximate date, you can reconstruct it (e.g. `aventi-venue-operators-2026-03-19`)
4. Alternatively, check the Campaign Outputs database — each campaign output record stores the Run ID

---

### O2 — "I ran the same persona twice on the same day — now I have duplicate Run IDs"

**Symptom:** Two sets of leads in the Lead Lists DB share the same Run ID, making it impossible to distinguish which run is which.

**Fix:**
1. For the second run, append a suffix to the Run ID: `aventi-venue-operators-2026-03-26-v2`
2. Notify Claude at the start of the run: "Use Run ID aventi-venue-operators-2026-03-26-v2 for this batch"
3. In Notion, manually update the Run ID field on the duplicate records to include the suffix

---

### O3 — "I ran copy-generator before lead-list-builder — now the leads have no copy"

**Symptom:** You ran copy-generator but it returned an error or found no leads to write copy for.

**Cause:** copy-generator requires leads to already exist in the Lead Lists DB with a valid Run ID. It reads from that database, not from a search.

**Fix:** Run lead-list-builder first for the persona. Once leads are written to the Lead Lists DB with a Run ID, run copy-generator and provide that Run ID.

---

### O4 — "The skill is asking questions I already answered in a previous session"

**Symptom:** You're re-running a skill and Claude is asking config questions (which persona, how many leads, etc.) that you answered before.

**Cause:** Each Cowork session starts fresh — Claude doesn't retain memory of previous session responses.

**Fix:** This is expected behaviour. Keep a note of your standard campaign config:
- Venture name and Run ID prefix
- Target lead count
- Outreach channel (Email / LinkedIn / Instagram)
- Export destination (Lemlist CSV / Notion / Instantly CSV)

You can paste this at the start of a session: "Use these defaults for this run: [venture], [count], [channel], [export]."

---

## Model Reasoning Issues

### M1 — "The email copy is too long / more than 75 words"

**Symptom:** Generated emails are 100+ words. This reduces reply rates.

**Fix:** Ask Claude: "Rewrite [variant A/B] to be under 60 words. Keep the same angle and CTA." If it keeps generating long copy, add: "Count the words explicitly and stop at 60."

---

### M2 — "The Notion page URL Claude gave me doesn't open"

**Symptom:** Claude returns a Notion URL after writing a record, but clicking it shows a 404 or access denied.

**Cause:** Claude sometimes constructs Notion URLs from the page ID rather than reading the actual shareable URL. The URL format may be slightly incorrect.

**Fix:** Ask Claude: "Fetch the actual URL for the [persona name] page in the Personas database." Claude will use the Notion MCP to retrieve the correct URL directly from the API response.

---

### M3 — "Claude is confusing two different personas or ventures mid-session"

**Symptom:** Copy output references the wrong venture, or lead searches are using the wrong persona attributes.

**Cause:** Long sessions with multiple personas can cause context bleed.

**Fix:** At the start of any step where accuracy matters, state explicitly: "We are now working on [venture name] / [persona name] / Run ID [X]. Confirm before proceeding." Claude will confirm before running the next step.

---

*GTM Outbound Engine — Phase 4 Plugin*
*Refer to README.md for setup and operating guide*
