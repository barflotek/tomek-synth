# tomek-synth

Synth bundle for **Tomek / Zboralscy Białystok** — first customer deployment of the synth primitive.

A **synth** is a packaged agent persona: a small, focused skill bundle that turns a generic chat client (today: claude.ai Pro; tomorrow: Slack, Teams, custom) into a domain operator. Built on the three-layer architecture from `~/ops-vault/businesses/tomek/decisions.md` (2026-04-19): **context + apps + playbooks**.

This repo is the source of truth. The synth is activated by loading these files into Tomek's claude.ai Project as Skills.

## What's in here today

```
tomek-synth/
├── README.md                    ← you are here
├── synths/
│   └── social-media/            ← first synth: Tomek's social-media operator
│       ├── SKILL.md             ← entry point (loaded first)
│       ├── handbook.md          ← posting knowledge, voice, hooks
│       ├── fly-templates.md     ← how to use the zboralscy render service + MCP tools
│       └── playbooks/
│           ├── new-listing.md
│           ├── sale-closed.md
│           └── local-spotlight.md
```

## Who this is for

- **Bartek** — maintainer, source of truth, versions the synth as Tomek's workflow evolves
- **Tomek (Tomasz Brynkiewicz)** — end user, interacts via chat, never edits these files
- **Future synths** — this repo is the template for `synth #2`, `synth #3` etc. when we scale to other agencies

## What this is NOT

- Not runtime code. No Node/Python/build step. Just markdown that gets uploaded to claude.ai Skills.
- Not a framework. We evaluated Fabric (see `~/ops-vault/CAPTURE.md` 2026-04-23) — wrong primitive. We'll revisit if we decouple from claude.ai.
- Not tool code. The MCP tools (`zboralscy_*`, `meta_*`, `esticrm_*`) live in `~/mcp-multitenant-hub` (Bellink). This repo teaches the synth how to use them.

## Activation / onboarding

**TBD** — Bartek + Ben are designing the activation flow after the handbook is written. Until then this repo is read-only reference material.

## Related

- **Bellink MCP** (tools): `~/mcp-multitenant-hub`
- **Render service** (Fly): `~/zboralscy-templates` · https://zboralscy-templates.fly.dev
- **Decisions log**: `~/ops-vault/businesses/tomek/decisions.md`
- **Backlog**: `~/ops-vault/businesses/tomek/backlog.md`
- **Strategic vision**: `~/ops-vault/SYSTEM.md`
