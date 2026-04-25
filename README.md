# tomek-synth · Bellink plugin marketplace

Claude plugin marketplace published by **Bellink**. First customer wave: Polish real estate (Zboralscy Białystok). Future plugins for additional verticals (legal, beauty, fitness, finance) will live as siblings under `plugins/`.

This repo IS a marketplace — anyone can add it to their Claude with one command:

```bash
/plugin marketplace add barflotek/tomek-synth
```

## Plugins in this marketplace

| Plugin | Purpose | Status |
|---|---|---|
| [`zboralscy-social`](plugins/zboralscy-social) | Social-media operator for Zboralscy Białystok — listings, sprzedane, reels, Open Day | ✅ Live |

## What's a "synth"

A **synth** = a packaged Claude plugin tailored to one customer's deployment. It bundles:

- **Skills** (markdown files in `plugins/<name>/skills/<skill>/SKILL.md`) — instructions Claude loads on demand
- **MCP server config** (`plugins/<name>/.mcp.json`) — points to Bellink's hosted tools at `app.bellink.io`
- **Plugin manifest** (`plugins/<name>/.claude-plugin/plugin.json`) — metadata + version

The capabilities (render images, publish to Meta, fetch from EstiCRM, etc.) live behind Bellink's MCP server. The plugin is the thin client wrapper that teaches Claude how + when to call them.

## Architecture

```
Customer's claude.ai (Tomek)
         │
         │ /plugin install zboralscy-social@bellink
         ▼
[Plugin installed locally — markdown skills + .mcp.json]
         │
         │ Tools called when skills fire
         ▼
Bellink (app.bellink.io · MCP server)
         ├─ zboralscy_render_post → Fly render service
         ├─ meta_create_instagram_post → Meta Graph API
         ├─ esticrm_register_inquiry → EstiCRM
         └─ ... ~130 tools across Drive / Gmail / Meta / EstiCRM / Zboralscy
```

The plugin is open and copyable. The capabilities are auth-gated at Bellink. **Stop paying → MCP key revoked → plugin tools fail.** Standard SaaS pattern.

## Repo layout

```
tomek-synth/
├── README.md
├── .claude-plugin/
│   └── marketplace.json          ← marketplace registry
└── plugins/
    └── zboralscy-social/          ← first plugin
        ├── .claude-plugin/
        │   └── plugin.json        ← manifest
        ├── .mcp.json              ← MCP server pointer
        ├── README.md
        └── skills/
            ├── social-media/      ← identity + operating rules
            ├── handbook/          ← voice, hooks, cadence
            ├── fly-templates/     ← render tool reference
            ├── playbook-new-listing/
            ├── playbook-sale-closed/
            └── playbook-local-spotlight/
```

## Build status

- Format: Claude Plugin spec (Anthropic, January 2026) — see [knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) reference
- License: All rights reserved (private use by Bellink customers)
- Multi-platform: Claude today; ChatGPT and Manus adapters when those product lines stabilize their plugin formats

## Roadmap (next plugins)

- `zboralscy-leads` — lead capture from Gmail/Otodom inbox + EstiCRM registration + Polish auto-reply (Synth #2, target May 2026)
- `zboralscy-viewings` — viewing scheduling, confirmations, post-viewing feedback (Synth #3)
- `zboralscy-reports` — weekly performance digest from EstiCRM (Synth #4)

Each future plugin = new folder under `plugins/`, registered in `marketplace.json`. No Bellink backend changes per plugin (tools already exist or get added once, plugins reuse).
