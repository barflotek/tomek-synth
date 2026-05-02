# Synth Patterns — what worked, what didn't, what to ask before building

Living document. Drawn from synth #1 (`zboralscy-social`). Append entries after every synth retro. Don't rewrite history.

---

## Synth Viability Checklist (run before building)

Source: `@theeasyvc` IG carousel, captured 2026-04-27 → `~/ops-vault/reference/easyvc-zillow-lesson.md`. Use it as a 30-minute gate before committing to a new synth.

1. Can you name 12 workflows in this vertical no one has built software for?
2. Does the operator (the customer) have 5+ years of experience in that industry?
3. Is the problem too small or too unsexy for OpenAI to prioritize?
4. Does the synth generate proprietary workflow data every day?
5. Will switching costs be structural in 18 months?
6. Can you name the 6 steps that break and why?
7. Is the moat the model, or the 40 handoffs around it?
8. Are we racing Sam Altman, or are we racing nobody?

**Score 6+ ✓ → proceed. 4–5 ✓ → push back, scope smaller. ≤3 ✓ → don't build.** Document the score in the synth's `decisions.md` or in the roadmap entry.

---

## What worked (synth #1 — zboralscy-social)

| Pattern | Why it worked |
|---|---|
| Tight vertical scope (Polish RE specifically, not "real estate generally") | Tomek is the operator; his vocabulary, conventions, and regulations are concrete, not abstract |
| Four-skill-type structure (identity / handbook / tool-ref / playbooks) | Each skill has a single purpose; Claude loads only what's needed; descriptions stay sharp |
| Trigger phrases in the customer's language (Polish) embedded in skill `description` frontmatter | Skills fire reliably on real user phrasing ("nowa oferta", "sprzedane", "reel z drona") |
| Plugin marketplace + one-line install (`/plugin marketplace add barflotek/tomek-synth`) | Zero install friction for the customer |
| Bellink hosts MCP, plugin is thin client | Capabilities update without re-shipping the plugin; revoke MCP key = synth dies cleanly |
| Per-customer credentials in MCP `connections.credentials` JSON (not new schema) | No DB migrations per customer; trivial to onboard the next one |
| Reused existing skills (`social-content` MIT + in-house `social-media-publishing`) | Didn't rebuild generic copywriting / publishing knowledge per synth |
| Documented architectural decisions in `~/ops-vault/businesses/tomek/decisions.md` append-only | Future synths inherit the reasoning, not just the result |

## What didn't / known gotchas

| Gotcha | Why it bit | Fix |
|---|---|---|
| Single-tenant gating is **de-facto, not enforced** | "Safe today because no other tenant has zboralscy credentials" — not a real gate | **Add explicit email allowlist check** in `tools/list` and tool dispatch (`mcp-multitenant-hub/app/api/mcp/server/route.ts`) BEFORE shipping synth #2 |
| Brand-specific tool refs (`fly-templates`) can't be reused for non-Zboralscy customers | Render service is branded + Polish RE-specific | Future tool refs that may be cross-customer must abstract brand into a per-tenant `brand_profile` credential |
| Meta API "lying" responses (HTTP 200 + `success:false`, `/feed` silently dropping image params) | Meta's surface returns OK on partial failures | `MetaClient.request()` now preserves `error_subcode`, `fbtrace_id`, `error_user_title`, `error_user_msg`. Don't strip these in any future connector |
| EstiCRM auth: HTTP 200 with `{status: false}` on auth errors (not `{success: false}`) | Wrong field check accepted bad credentials initially | Always parse `status` AND `success`; require BOTH `success === true` AND `Array.isArray(data)` |
| Fly cold-start (~7.5s first render) | Fly machine spins down on idle | If usage goes daily, set `min_machines_running = 1` (~$3/mo). Warm renders ~2s |
| IG profile grid effectively crops 1080×1080 → 4:5 | IG 2025+ display rule | Either render 1080×1350 native OR redesign with safe zones. Don't ship 1:1 expecting full visibility |
| `fly.dev` URLs fail Meta publishing (host reputation) | Meta blocks low-reputation hosting subdomains | Proxy through `app.bellink.io/zboralscy/img/...` — the render tool already does this; trust its output |
| No standard "I don't have access to X" handling | Each playbook re-implements graceful-failure copy | When `_shared/` lands (post synth #2), put it there |

## Reuse before building

Always check before authoring a new skill:

1. `~/ops-vault/reference/skills-marketplace.md` — registry of skills already evaluated, the ones in use, and the ones we explicitly skipped (and why)
2. `~/.claude/skills/` — what's locally installed
3. **VoltAgent / awesome-agent-skills** (1100+ MIT skills) — primary external registry
4. **Anthropic official skills** — gold-standard format reference

Skill we currently reuse across synth work:
- `social-content` (Corey Haines) — strategy/copy/voice/hooks/cadence
- `social-media-publishing` (in-house) — per-platform dimensions + safe zones + Meta gotchas

## When to deviate from `SYNTH-SPEC.md`

The four-skill template fits content/publishing-shaped synths well. It will probably bend for:
- **Read-only / monitoring synths** — may not need a handbook (just identity + tool-ref + watch playbooks)
- **Pure orchestration synths** — may need additional `_routing/` skills for multi-agent handoff
- **Compliance-heavy verticals** (finance, legal, medical) — may need a separate `regulations/` skill split out from handbook

Document the deviation + reason in that synth's `decisions.md`. Don't silently fork the spec.
