# zboralscy-social

Social-media operator for **Zboralscy Białystok** — Polish real estate agency. Drafts, renders, and publishes brand-consistent posts (listings, sprzedane, market updates), carousels (room-by-room property tours), reels, and event announcements (Dzień Otwarty) to Instagram and Facebook.

## What it does

| Workflow | Skill | Output |
|---|---|---|
| Post a new listing as a 5-6 slide carousel | `playbook-new-listing` | T1 cover + T8 room slides + T4C closer → IG carousel + FB photo |
| Announce a closed deal | `playbook-sale-closed` | T2 Sprzedane → IG + FB |
| Publish a neighborhood / luxury reel | `playbook-local-spotlight` | T3C intro + your video + T6 outro composed into one MP4 |
| Brand voice + caption craft | `handbook` | Polish copy in Tomek's voice, hashtag + cadence rules |
| Tool reference | `fly-templates` | Every render template with its fields, tier system, gotchas |

## Tier system (for property posts)

Templates **T1, T7, T8** support `tier`:
- **`standard`** — functional layout: photo dominant, feat-strip with icons (rooms, parking, bathroom, m²), agent footer with portrait + phone. Use for listings ≲ 1M zł.
- **`luxury`** — editorial: italic display serif, decorative oversized sygnet on T7, no feat-strip / no agent footer. Use for premium properties.

## Tools (via Bellink MCP at `app.bellink.io`)

- `zboralscy_list_templates`, `zboralscy_render_post`, `zboralscy_compose_reel`
- `meta_create_instagram_post`, `meta_create_page_photo`, `meta_create_instagram_reel`, `meta_create_instagram_story`
- `gmail_*`, `drive_*`, `esticrm_*` (when needed)

## Install

Add the marketplace, then install:

```bash
/plugin marketplace add barflotek/tomek-synth
/plugin install zboralscy-social@bellink
```

After install, Claude prompts to authenticate with Bellink. Use the credentials provided to you separately.

## Brand reference

- Navy (Atramentowy): `#1C364A`
- Gold (Piaskowy): `#D6B36A`
- Typography: Cormorant Garamond + Inter (free fallbacks for the brand's licensed Alliance No.1 + HV Clio — substitution sanctioned by the official identity book, page 13)

## Tenant gating

Plugin works against any Bellink account, but the `zboralscy_*` tools are allowlisted server-side. Only authorized agents (currently `t.b.ukosna@gmail.com`) can render under the Zboralscy brand. To onboard another agent: contact bartek@fencingprocorp.com.
