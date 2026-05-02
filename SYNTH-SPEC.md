# Synth Spec

What a Bellink synth IS. Descriptive, not prescriptive — extracted from synth #1 (`zboralscy-social`). Treat this as a starting structure, not a cage. When a synth genuinely needs to deviate, document why in `decisions.md` for that synth.

---

## Definition

A **synth** = a vertical-specialised Claude plugin packaged for one customer's deployment. It bundles:

- **Skills** — markdown files with frontmatter that Claude loads on demand, teaching it when + how to operate
- **MCP server config** — points to Bellink's hosted tools at `app.bellink.io`
- **Plugin manifest** — metadata for the Claude plugin marketplace

The capabilities (render, publish, fetch CRM data, send DMs) live behind Bellink's MCP server. The synth is the thin client wrapper that gives Claude domain knowledge and routing rules.

## Required directory layout

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json              ← manifest: name, version, description, author, homepage
├── .mcp.json                    ← MCP server pointer to app.bellink.io
├── README.md                    ← human-readable overview (what it does, who for, how to install)
└── skills/
    ├── <main>/SKILL.md          ← REQUIRED — identity + when-to-fire + operating rules
    ├── handbook/SKILL.md        ← REQUIRED — domain knowledge (voice, terminology, regulations)
    ├── <tool-ref>/SKILL.md      ← REQUIRED — tool surface reference (when to call which tool)
    └── playbook-<task>/SKILL.md ← REQUIRED (one or more) — concrete task recipes
```

The plugin must also be registered in the marketplace's `.claude-plugin/marketplace.json` at the repo root.

## Required skill types

Every synth has at minimum these four skills. Each `SKILL.md` opens with frontmatter:

```yaml
---
name: <skill-name>
description: <one-line trigger-rich description — Claude uses this to decide when to load>
---
```

The `description` field matters. Claude only loads skills whose description matches the user's request. Bad descriptions = skill never fires. Include trigger phrases (in the customer's language).

### 1. Identity skill (`skills/<main>/SKILL.md`)
- **Purpose:** the operator persona — who Claude is acting as, what brand, what guardrails
- **Must contain:** identity rules (language, brand, tone), how-to-operate rules (listen first, no guessing, ask under ambiguity), routing rules (when to load other skills)
- **Anti-pattern:** putting domain knowledge here — that goes in handbook

### 2. Handbook skill (`skills/handbook/SKILL.md`)
- **Purpose:** deep domain knowledge for the vertical
- **Must contain:** voice rules, terminology, sector-specific conventions, regulations, formulas, hook libraries, anything that would take a human operator years to learn
- **Anti-pattern:** task recipes — those go in playbooks

### 3. Tool-reference skill (`skills/<tool-ref>/SKILL.md`)
- **Purpose:** authoritative reference for the MCP tool surface this synth uses
- **Must contain:** every tool name, what it accepts, what it returns, when to use it, gotchas
- **Anti-pattern:** duplicating MCP tool docs — link to authoritative source where possible

### 4. Playbook skills (`skills/playbook-<task>/SKILL.md`, one per common workflow)
- **Purpose:** concrete recipes for common user requests
- **Must contain:** trigger phrases (when to fire), step-by-step procedure, tool call sequence, error handling
- **Naming:** `playbook-<verb>-<noun>` (e.g., `playbook-new-listing`, `playbook-sale-closed`)
- **Count:** start with the 3 highest-frequency tasks, add more as user asks reveal gaps

## Naming conventions

| Thing | Convention | Example |
|---|---|---|
| Plugin name | `<customer>-<domain>` lowercase, kebab-case | `zboralscy-social`, `zboralscy-leads` |
| Identity skill | matches the domain part of plugin name | `social-media`, `leads`, `viewings` |
| Playbook skill | `playbook-<task>` | `playbook-new-listing` |
| Customer abbreviation | first word, lowercase | `zboralscy`, `tmfc` |

## Versioning

- `plugin.json` `version` follows semver
- Bump patch (`1.0.X`) for handbook/playbook content edits
- Bump minor (`1.X.0`) for new playbooks or new tool surface
- Bump major (`X.0.0`) for breaking changes that would confuse Tomek (renamed triggers, removed skills)

## Marketplace registration

Every synth must be registered in `.claude-plugin/marketplace.json` at repo root with: `name`, `description`, `author`, `source` (relative path), `category`, `homepage`.

## Tenant gating (CRITICAL)

A synth's tools are typically branded/scoped to one customer. **Single-tenant gating must be enforced** at two places in `mcp-multitenant-hub/app/api/mcp/server/route.ts`:

1. **Tool listing handler** — filter out tools belonging to other customers' synths so non-allowlisted users don't even see them
2. **Tool dispatch handler** — reject calls with a 403-style error when caller's email isn't on the synth's allowlist

See `~/ops-vault/businesses/tomek/decisions.md` 2026-04-22 entry. Currently de-facto only — must be made real before synth #2 ships, otherwise tools leak.

## Reuse before building

Before writing any new skill, check `~/ops-vault/reference/skills-marketplace.md`. Two skills already in our stack reusable across synths:

- `social-content` (Corey Haines, MIT) — strategy/copy/voice for any social-media synth
- `social-media-publishing` (in-house) — per-platform dimensions, safe zones, Meta gotchas

Don't rebuild these per synth. Reference them from the identity skill's "When to load other skills" section.

## What's NOT in this spec yet

These will be defined after synth #2 ships:

- `_shared/` skill folder for cross-synth concerns (capture-first, error handling, "I don't have access to X", escalation)
- Scaffolding script: `bin/new-synth <plugin-name>` that copies the skeleton
- Validation checklist: "is this synth ready to ship"
- Test/golden-path harness: does the synth fire on expected triggers, route to expected tools

Don't pre-design these. Let synth #2 (`zboralscy-leads`) reveal what's truly shared.
