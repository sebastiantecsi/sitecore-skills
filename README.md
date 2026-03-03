# Sitecore Marketplace SDK — Claude Code Skills

Claude Code skills for building [Sitecore Marketplace](https://developers.sitecore.com/marketplace) apps. Covers the full lifecycle: scaffolding, building UI, integrating APIs, and deploying.

## Install

Add this repo as a Claude Code plugin:

```bash
claude plugins add /path/to/sitecore-skills
```

Or clone into your project's `.claude/skills/` directory.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **Scaffold App** | `/scaffold-app` | Scaffold a new Marketplace app with the correct SDK packages |
| **SDK Reference** | `/sdk-reference` | Look up SDK queries, mutations, subscriptions, and types |
| **Build Component** | `/build-component` | Build UI with the Blok design system (Sitecore's shadcn theme) |
| **Deploy to Vercel** | `/deploy-vercel` | Deploy with correct CSP headers and env vars |
| **Add Extension Point** | `/add-extension-point` | Add a route for a specific extension type (custom field, widget, etc.) |
| **Add XM Cloud API** | `/add-xmc-api` | Integrate Sites, Pages, Authoring, Search, or Agent APIs |
| **Add AI Skill** | `/add-ai-skill` | Add Brand Review API for AI-powered content analysis |

## Getting Started

1. Run `/scaffold-app` to create a new Marketplace app
2. Run `/add-extension-point` to add your first extension point
3. Run `/build-component` to build the UI
4. Run `/deploy-vercel` when ready to deploy

## SDK Packages

| Package | Purpose |
|---------|---------|
| `client` (required) | Core SDK client — queries, mutations, subscriptions |
| `xmc` (optional) | XM Cloud APIs — Sites, Pages, Authoring, Content Transfer, Search, Agent |
| `ai` (optional) | AI Skills — Brand Review API |

## Links

- [Sitecore Marketplace SDK Docs](https://developers.sitecore.com/marketplace/sdk)
- [Sitecore Developer Portal](https://developers.sitecore.com)
- [Claude Code Skills Docs](https://docs.anthropic.com/en/docs/claude-code/skills)
