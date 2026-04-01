# Bunker Collective — Plugin Marketplace

Private plugin marketplace for the Bunker Collective team. Contains packaged skills and agents for Claude Code / Cowork.

## Available Plugins

| Plugin | Version | Description |
|--------|---------|-------------|
| [GTM Outbound Engine](./gtm-outbound-engine/) | v0.2.0 | ICP → Lead List → Cold Copy pipeline |

## Adding This Marketplace to Claude

Run this in any Claude Code / Cowork session:

```
/plugin → Marketplaces → Add Marketplace → bunker-collective/plugins-marketplace
```

Or via the CLI:

```bash
claude plugin marketplace add bunker-collective/plugins-marketplace
```

Then browse and install:

```
/plugin → Browse Plugins
```

## Installing a Plugin

1. Open `/plugin` in Claude
2. Go to **Browse Plugins**
3. Find the plugin you want
4. Click **Install for me** (local scope — just this project)

## Adding a New Plugin

1. Create a folder in this repo: `your-plugin-name/`
2. Add your skills inside: `your-plugin-name/skills/skill-name/SKILL.md`
3. Add plugin docs: `README.md`, `CONFIGURATION-GUIDE.md` etc.
4. Add an entry to `marketplace.json`
5. Commit and push — team picks it up automatically within ~30 minutes

## Updating a Plugin

Edit the files, bump the version in `marketplace.json`, commit, and push. Claude will detect the update next sync.

---

*Access restricted to Bunker Collective team members with GitHub org access.*
