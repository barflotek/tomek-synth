---
name: fly-templates
description: Authoritative reference for the Zboralscy render service and zboralscy_* MCP tools — every template (T1, T2, T3A, T3B, T3C, T4, T4C, T5, T6, T7 Open Day, T8 Carousel Interior), their fields, the standard/luxury tier system, the Bellink-proxy rule, Meta gotchas (9004 host reputation, page_photo vs page_post). Load BEFORE any render call to know the right fields.
---

# Fly Templates — Tool Mastery Guide

Everything about the Zboralscy render service (`zboralscy-templates.fly.dev`) and the `zboralscy_*` MCP tools. Read this once; from then on, the synth should be able to render any template without guessing fields.

---

## 1. The stack (1-minute overview)

```
claude.ai (Tomek)
     │  MCP
     ▼
Bellink (~/mcp-multitenant-hub, deployed to Vercel as app.bellink.io)
     │  ├─ zboralscy_list_templates  → reads template schema
     │  ├─ zboralscy_render_post     → POSTs to Fly, rewrites URL
     │  └─ /zboralscy/img/[filename] → proxy route for Meta publishing
     │
     ▼  HTTPS
Fly.io (zboralscy-templates.fly.dev)
     │  ├─ Puppeteer + Chromium
     │  ├─ templates.schema.json + public/render.html
     │  └─ stored PNGs/JPEGs at /a/<hash>.jpg
     │
     ▼
Returned URL:  https://app.bellink.io/zboralscy/img/<hash>.jpg
     │
     ▼
meta_create_instagram_post  /  meta_create_page_photo
     │
     ▼
IG / FB Page
```

Single-tenant today. Gated by `ZBORALSCY_ALLOWLIST = ['t.b.ukosna@gmail.com']` in `app/api/mcp/server/route.ts`. Non-allowlisted users don't see the tools at all.

---

## 2. MCP tools

### 2.1 `zboralscy_list_templates`

**Call first** in every new conversation. Gives you the authoritative schema — fields, defaults, image bindings. Never hardcode field names from this doc; the service is the source of truth.

```
Parameters: (none)
Returns: Record<string, TemplateInfo>
```

Each `TemplateInfo`:

```ts
{
  id: string;                // "T1", "T2", "T3A", etc.
  purpose: string;           // human-readable description
  dimensions: [number, number];  // [1080, 1080] or [1080, 1920]
  fields: {
    [fieldName]: {
      type: "string" | "url";
      default?: string;
      note?: string;
      bind?: "bg";           // image fields render as background
    }
  }
}
```

### 2.2 `zboralscy_render_post`

```
Parameters:
  templateId: string   (required)   "T1" | "T2" | "T3A" | "T3B" | "T3C" | "T4" | "T4C" | "T5" | "T6"
  data:       object   (required)   template-specific fields (see § 3)
Returns: RenderedAsset
  {
    id:     string,   // template ID
    url:    string,   // ALREADY proxied: https://app.bellink.io/zboralscy/img/<hash>.jpg
    width:  number,
    height: number,
    bytes:  number
  }
```

**Key behavior:** The URL it returns is already rewritten from `fly.dev/a/*.jpg` → `bellink.io/zboralscy/img/*.jpg`. Never reconstruct the URL yourself. Never substitute the Fly origin. See § 4 for why.

### 2.3 `zboralscy_compose_reel`

Composes a finished MP4 reel server-side: prepends intro frame (default T3C drone story) → pins user-supplied video clip in middle → appends T6 agent outro. Returns a **Bellink-proxied MP4 URL** safe to pass to Meta publishing tools. ffmpeg pipeline runs on Fly.

```
Parameters:
  source:    object   (required)
    url:       string   (required)   public http(s) URL to source video
    start:     number   (optional)   seconds to skip at start (default 0)
    duration:  number   (optional)   seconds to use (default 10, capped at 30)
  data:      object   (optional)   template fields piped into intro+outro frames
                                    T3C uses: location, title, area, price, cta
                                    T6 uses: agentName, agentRole, phone, email, www, portrait
  intro:     string   (optional)   intro template ID (must be 9:16). Default "T3C"
  outro:     string   (optional)   outro template ID (must be 9:16). Default "T6"
  introSec:  number   (optional)   intro display seconds. Default 1.5
  outroSec:  number   (optional)   outro display seconds. Default 2.0
  jobLabel:  string   (optional)   filename prefix (alphanumeric). Default "REEL"

Returns: ComposedReel
  {
    id:           string,   // jobLabel uppercased
    url:          string,   // ALREADY proxied: https://app.bellink.io/zboralscy/img/<hash>.mp4
    width:        1080,
    height:       1920,
    durationSec:  number,   // total composed length (intro + source + outro)
    bytes:        number,
    intro:        string,   // intro template ID used
    outro:        string    // outro template ID used
  }
```

**Source URL types:**
- **Direct media** (`.mp4`, `.mov`, `.m4v`, `.webm`, `.mkv`): fetched via `curl`. Fastest path. Tomek can drop a Drive/Dropbox/WeTransfer "direct download" link.
- **Streaming hosts** (YouTube, Vimeo, Drive `view` links, etc.): fetched via `yt-dlp` with `--download-sections` (lazy fetch — only the requested `start..start+duration` window).

**Same proxy rule as render_post:** the `url` returned is already `bellink.io/zboralscy/img/<hash>.mp4`. Trust it. Don't reconstruct from Fly. The proxy route at `app/zboralscy/img/[filename]/route.ts` handles `.mp4` the same way it handles `.jpg`.

**Latency:** 30-90s typical (yt-dlp fetch + ffmpeg normalise + render frames + concat). Acceptable as a sync MCP call. Set Tomek's expectation: "renderowanie reela trwa około minuty."

**Failure modes:**
- Source URL unreachable → 500 from upstream with `curl`/`yt-dlp` stderr in error message
- Source > 30s requested → automatically clamped to 30s
- intro/outro is not 9:16 → 400 "intro X is not 1080x1920; use T3A/T3B/T3C/T6"
- ffmpeg crash → 500 with last 800 chars of stderr

---

## 3. Template catalog

Nine templates. All 1080px wide. Feed posts are **4:5 portrait (1080×1350)** so they survive Instagram's profile-grid crop intact; reel frames are 9:16 (1080×1920). The central design area is still 1080×1080 — bands of 135px top + bottom carry the navy brand frame plus a "Zboralscy · Białystok" wordmark in the bottom band (acts as the grid-thumb identifier).

| ID   | Purpose                                        | Ratio | Use when                                                  |
|------|------------------------------------------------|-------|-----------------------------------------------------------|
| T1   | Listing hero (standard + luxury tiers)         | 4:5   | New offer → IG feed + FB page                             |
| T2   | Sprzedane (sold)                               | 4:5   | Deal closed, announcement within 48h                      |
| T3A  | Reel cover — hook (list-style: "3 rzeczy…")    | 9:16  | Educational / top-of-funnel reel                          |
| T3B  | Reel cover — question                          | 9:16  | "Pytanie tygodnia" / Q&A reel                             |
| T3C  | Drone / luxury listing story                   | 9:16  | Premium listing, view, architecture — cinematic reel      |
| T4   | Educational carousel — tip card                | 4:5   | Carousel slides 1–N (numbered tips)                       |
| T4C  | Carousel summary / CTA card                    | 4:5   | Last slide of a carousel (drives to action)               |
| T5   | Monthly market update                          | 4:5   | Monday of first week of month — raport rynkowy            |
| T6   | Agent outro (end card for reels)               | 9:16  | ALWAYS the last frame of any reel (closes with Tomek's contact) |
| T7   | Open Day (Dzień Otwarty) — standard + luxury   | 4:5   | Announce an open house event                              |
| T8   | Carousel interior slide (photo-heavy room tour)| 4:5   | Between T1 cover + T4C closer — room-by-room photos       |

### Tier system (T1, T7, T8)

Three templates accept a `tier` field: `"standard"` (default) or `"luxury"`. The synth picks the tier based on property price + language:
- **Standard** → functional info-dense layout, photo-dominant, feat-strip with icons, agent badge, bold sans typography. Use for listings ≲ 1M zł or any regular-market property.
- **Luxury** → editorial, restrained, italic display serif headlines, solid-navy backgrounds (on T7), oversized decorative sygnet. Use for premium properties (≳ 1M zł), architecturally distinct, or when the listing description reads "luksusowy / apartament / rezydencja / willa".

When in doubt, ask Tomek: "standard czy luxury?" Don't guess on borderline cases.

### 3.1 T1 — Listing Hero (1080×1350, 4:5)

| Field         | Type           | Default                      | Notes                                                                        |
|---------------|----------------|------------------------------|------------------------------------------------------------------------------|
| `tier`        | string         | "standard"                   | `"standard"` or `"luxury"`. See Tier system § above.                         |
| `badge`       | string         | "Nowa oferta"                | Gold chip top-left. Auto-hidden on luxury tier.                              |
| `location`    | string         | "Białystok · Pałacowa"       | Format: "Miasto · Dzielnica"                                                  |
| `price`       | string         | "749 000"                    | **No currency, no "zł"** — formatted number only                              |
| `propertyName`| string         | —                            | **Luxury only** — italic property name (e.g. "Dom przy lesie") above price.   |
| `rooms`       | number\|string | 3                            | Number → auto-pluralizes ("3 POKOJE", "5 POKOI"). String → renders verbatim.   |
| `bathrooms`   | number\|string | 1                            | Same as rooms. 0 or missing → cell hidden.                                   |
| `garages`     | number\|string | 1                            | Number → "1 GARAŻ", "2 GARAŻE". String → e.g. "Parking strzeżony" verbatim.   |
| `area`        | string         | "82 m²"                      | Include unit                                                                 |
| `agentName`   | string         | "Tomasz Brynkiewicz"         | Standard tier: appears in bottom agent strip. Luxury tier: hidden.           |
| `agentPhone`  | string         | "+48 669 996 948"            | Standard tier: appears in bottom agent strip. Luxury tier: hidden.           |
| `portrait`    | url            | —                            | Tomek's portrait for the agent strip. Standard tier only.                    |
| `photo`       | url            | —                            | Hero image, `bind: bg`                                                       |

**Polish pluralization handled automatically** — pass an integer and the template picks the right form:
- `1` → singular (`pokój`, `łazienka`, `garaż`)
- `2–4` (not `12–14`) → paucal (`pokoje`, `łazienki`, `garaże`)
- `5+` or `12–14` → genitive (`pokoi`, `łazienek`, `garaży`)

**String override use-cases** (send a string instead of a number when it reads more naturally):
- Property has no formal garage but has other parking: `"garages": "Parking strzeżony"` / `"garages": "Parking na ulicy"` / `"garages": "Miejsce postojowe"`
- Rooms have a special layout: `"rooms": "Studio + alkowa"` (rarely — usually stay numeric)

**Typical standard call:**
```json
{
  "templateId": "T1",
  "data": {
    "tier": "standard",
    "badge": "Nowa oferta",
    "location": "BIAŁYSTOK · CENTRUM",
    "price": "689 000",
    "rooms": 3,
    "bathrooms": 1,
    "garages": 1,
    "area": "68 m²",
    "agentName": "Tomasz Brynkiewicz",
    "agentPhone": "+48 669 996 948",
    "portrait": "https://<tomek-portrait-url>",
    "photo": "https://<offer-photo-url>"
  }
}
```

**Typical luxury call** (editorial, no feat-strip, no agent strip):
```json
{
  "templateId": "T1",
  "data": {
    "tier": "luxury",
    "location": "SUPRAŚL · PODLASIE",
    "propertyName": "Dom przy lesie",
    "price": "1 600 000",
    "rooms": 6,
    "area": "245 m²",
    "photo": "https://<luxury-photo-url>"
  }
}
```

### 3.2 T2 — Sprzedane (1080×1350, 4:5)

| Field      | Type   | Default                      | Notes                                  |
|------------|--------|------------------------------|----------------------------------------|
| `location` | string | "Pałacowa · Białystok"       |                                        |
| `kicker`   | string | "Sprzedane w 14 dni"         | Big tagline                            |
| `days`     | string | "14 dni"                     | Time-to-close, secondary display       |
| `discount` | string | "–4 000 zł"                  | Optional — only if positive story       |
| `leads`    | string | "27 osób"                    | Liczba zapytań                         |
| `photo`    | url    | —                            | Listing photo, `bind: bg`              |

### 3.3 T3A — Reel Cover, Hook (1080×1920)

| Field     | Type   | Default                           | Notes                               |
|-----------|--------|-----------------------------------|-------------------------------------|
| `label`   | string | "REEL · 01"                        |                                     |
| `context` | string | "Białystok 2026"                  |                                     |
| `bigNum`  | string | "03"                               | Big number (200px serif)            |
| `hook`    | string | "rzeczy, które tanieją mieszkanie…" |                                     |
| `sub`     | string | "Przewiń, żeby poznać listę…"      |                                     |
| `cta`     | string | "Oglądaj"                          |                                     |
| `photo`   | url    | —                                  | Background, `bind: bg`              |

### 3.4 T3B — Reel Cover, Question (1080×1920)

| Field      | Type   | Default                             | Notes                                |
|------------|--------|-------------------------------------|--------------------------------------|
| `tag`      | string | "#TIP 04"                            |                                      |
| `kicker`   | string | "Pytanie tygodnia"                   |                                      |
| `question` | string | "Czy warto sprzedawać `<span>`zimą?`</span>`" | HTML `<span>` wraps gold-highlighted word |
| `cta`      | string | "Odpowiedź"                          |                                      |

### 3.5 T3C — Drone / Luxury Story (1080×1920)

| Field      | Type   | Default                           | Notes                                    |
|------------|--------|-----------------------------------|------------------------------------------|
| `location` | string | "Sokółka · 20 km od Białegostoku" |                                          |
| `title`    | string | "Dom z widokiem `<br>` na puszczę" | `<br>` allowed for line break           |
| `area`     | string | "158 m²"                           |                                          |
| `price`    | string | "690 000 zł"                       | T3C includes "zł" (unlike T1)           |
| `cta`      | string | "Zobacz wirtualny spacer →"        |                                          |
| `photo`    | url    | —                                  | `bind: bg`. **Video first-frame works here** — pair with user-supplied reel body. |

### 3.6 T4 — Educational Tip Card (1080×1350, 4:5)

| Field     | Type   | Default                                             | Notes                                      |
|-----------|--------|-----------------------------------------------------|--------------------------------------------|
| `num`     | string | "1"                                                 | Big numbered badge (top-left circle)       |
| `kicker`  | string | "Przed sprzedażą"                                   | Eyebrow / category — uppercase auto-applied |
| `title`   | string | "Wycena to nie przypadec."                          | Main hero — keep ≤ 8 words                 |
| `body`    | string | "3 dane, których nie znajdziesz na Otodomie…"       | Supporting paragraph — 1-2 sentences max   |
| `counter` | string | "1 / 5"                                             | **Single field** — pass exactly as displayed (e.g. `"3 / 5"`). Do NOT split into `slide` + `total` — that produces "3 / 5 / 5" garbage. |

### 3.7 T4C — Carousel Summary (1080×1350, 4:5)

| Field    | Type   | Default                      | Notes                         |
|----------|--------|------------------------------|-------------------------------|
| `kicker` | string | "Podsumowanie"               |                               |
| `title`  | string | "…"                          | Main CTA message              |
| `cta`    | string | "Umów darmową wycenę →"       | Always drives an action       |

### 3.8 T5 — Monthly Market Update (1080×1350, 4:5)

| Field          | Type   | Default                       | Notes                          |
|----------------|--------|-------------------------------|--------------------------------|
| `month`        | string | "Kwiecień 2026"               | Miesiąc + rok                  |
| `title`        | string | "Białystok — ceny mieszkań"   |                                |
| `avgPrice`     | string | "11 420 zł"                   | Cena za m² średnia              |
| `avgDelta`     | string | "+3,1% r/r"                   | r/r = rok do roku              |
| `medianPrice`  | string | "589 tys."                    | Mediana transakcji              |
| `medianDelta`  | string | "+1,8% r/r"                   |                                |
| `daysOnMkt`    | string | "62 dni"                      | Średni czas sprzedaży          |
| `daysDelta`    | string | "–8 dni m/m"                  | m/m = miesiąc do miesiąca      |
| `newListings`  | string | "341"                         | Nowe oferty w tym miesiącu     |
| `listingLabel` | string | "kwiecień 2026"               |                                |

Full navy background, no photo. Data-heavy — numbers are the hero.

### 3.9 T6 — Agent Outro (1080×1920)

**Always the last frame of every reel.** Doubles as a stand-alone "contact card" post if needed.

| Field       | Type   | Default                          | Notes                           |
|-------------|--------|----------------------------------|---------------------------------|
| `agentName` | string | "Tomasz `<br>` Brynkiewicz"      | `<br>` stacks last name below   |
| `agentRole` | string | "Pośrednik · Doradca"            |                                 |
| `phone`     | string | "+48 669 996 948"                |                                 |
| `email`     | string | "bialystok@zboralscy-group.pl"   |                                 |
| `www`       | string | "zboralscy-group.pl"             |                                 |
| `portrait`  | url    | —                                | Tomek's portrait, `bind: bg`    |

### 3.10 T7 — Open Day / Dzień Otwarty (1080×1350, 4:5)

Event announcement post for open-house viewings. **Two tier variants — visually distinct:**
- **Standard** → photo of the property as background with dark gradient + glass-blur date box.
- **Luxury** → solid navy background with oversized gold sygnet decorative half-cut off the right edge. Matches the Zboralscy visual identity reference (brand book page 1 of agent templates).

| Field      | Type   | Default                                 | Notes                                                          |
|------------|--------|-----------------------------------------|----------------------------------------------------------------|
| `tier`     | string | "standard"                              | `"standard"` (photo bg) or `"luxury"` (solid navy + sygnet)    |
| `badge`    | string | "Wydarzenie"                            | Standard only. Hidden on luxury.                               |
| `date`     | string | "10.03.2026"                            | Date of the open house, DD.MM.YYYY.                            |
| `subtitle` | string | "Apartament w Centrum Białegostoku"     | Short property descriptor. Renders sans caps with gold accent. |
| `url`      | string | "www.zboralscy-group.pl"                | Footer link.                                                   |
| `photo`    | url    | —                                       | **Standard only.** Ignored on luxury.                          |

**Typical call (standard):**
```json
{
  "templateId": "T7",
  "data": {
    "tier": "standard",
    "date": "10.05.2026",
    "subtitle": "Apartament w Centrum Białegostoku",
    "photo": "https://<property-photo-url>"
  }
}
```

**Typical call (luxury, for premium listings):**
```json
{
  "templateId": "T7",
  "data": {
    "tier": "luxury",
    "date": "10.05.2026",
    "subtitle": "Luksusowy apartament przy Parku Branickich"
  }
}
```

### 3.11 T8 — Carousel Interior Slide (1080×1350, 4:5)

Photo-heavy room tour slide — the body slides between T1 (cover) and T4C (closer) in a carousel. Use when Tomek wants to show multiple rooms of a listing.

**Pairing pattern:** T1 cover → T8 × N room slides → T4C closer. Slide counter (e.g. "2 / 5") is displayed top-right so viewers know where they are.

| Field       | Type   | Default                    | Notes                                                            |
|-------------|--------|----------------------------|------------------------------------------------------------------|
| `tier`      | string | "standard"                 | Same tier picker. Luxury → italic display name + floating text. |
| `roomLabel` | string | "Sypialnia"                | Small chip top-left. Hidden on luxury.                           |
| `slide`     | string | "2"                        | Current slide number (this one).                                 |
| `total`     | string | "5"                        | Total carousel slides (for the counter).                         |
| `roomName`  | string | "Sypialnia główna"         | Display room name — italic serif on luxury, upright on standard. |
| `roomSpec`  | string | "18 m² · Okno południowe"  | Short spec line. Sans caps for readability.                      |
| `notes`     | string | "Pełne wyposażenie, balkon"| Short descriptive sentence.                                      |
| `photo`     | url    | —                          | Hero image of the room, `bind: bg`                               |

**Rule:** Keep `tier` consistent across all slides of a single carousel. Don't mix standard T1 cover with luxury T8 slides.

**Typical call (standard, carousel slide 2 of 5):**
```json
{
  "templateId": "T8",
  "data": {
    "tier": "standard",
    "roomLabel": "Sypialnia",
    "slide": "2",
    "total": "5",
    "roomName": "Sypialnia główna",
    "roomSpec": "18 m² · Okno południowe",
    "notes": "Pełne wyposażenie, duży balkon",
    "photo": "https://<sypialnia-photo-url>"
  }
}
```

---

## 4. The Bellink-proxy rule (critical)

**Short version:** The URL returned by `zboralscy_render_post` is already `https://app.bellink.io/zboralscy/img/<hash>.jpg`. Hand it directly to `meta_create_instagram_post` or `meta_create_page_photo`. Don't modify it.

**Why it matters:** On 2026-04-22 we burned an afternoon debugging Meta error 9004 ("Only photo or video can be accepted as media type") when passing `fly.dev` URLs. Root cause: Meta runs an undocumented **host-reputation check**. `fly.dev` / `render.com` / `railway.app` etc. get rejected even with valid JPEGs + correct Content-Type. Vercel-hosted origins pass. Fix: every Fly-generated asset is proxied through `https://app.bellink.io/zboralscy/img/[filename]` before any URL escapes to Meta.

**What the synth needs to remember:**

1. Trust the `url` field from `zboralscy_render_post` — it's already safe for Meta.
2. Never reconstruct the Fly URL (e.g. don't take the `id` and build `fly.dev/a/<id>.jpg`).
3. If you ever see a `fly.dev` URL in your tool inputs, **stop**. That's a bug — report it, don't publish.
4. This rule generalises: any non-mainstream host (Render, Railway, niche CDN) must be proxied before Meta touches it.

Implementation lives at `~/mcp-multitenant-hub/app/zboralscy/img/[filename]/route.ts`. It's a transparent pass-through (streams bytes from Fly, sets `Cache-Control: public, max-age=604800, immutable`).

---

## 5. Known failure modes

### 5.1 Error 9004 — "Only photo or video can be accepted as media type"
Host rejected. If you see this with a `bellink.io` URL, something's broken upstream. Don't retry — ping Bartek.

### 5.2 `meta_create_page_post` drops images silently
Old bug, fixed by adding `meta_create_page_photo`. Rule: always use `meta_create_page_photo` for FB image posts. If you catch yourself calling `meta_create_page_post` with an `image_url` param, switch tools.

### 5.3 Render returns slowly (~5-7s)
Cold-start. `min_machines_running = 1` is set (~$3/mo) to keep one Fly machine warm, but if traffic lulls the machine might still sleep. First render after idle is slow; follow-ups are ~2s. Don't retry — wait.

### 5.4 Diacritics break in data fields
Templates render in **Cormorant Garamond** (serif, for display headlines + luxury tier) + **Inter** (sans, for labels + standard tier body). Both cover Polish diacritics (ą, ę, ś, ł, ż, ć, ó, ź, ń) natively. If a field renders as mojibake, the input was probably double-encoded (UTF-8 → Latin-1 → UTF-8 round-trip). Re-send as plain UTF-8. Font substitution note: these are close-enough stand-ins for the brand's primary fonts Alliance No.1 + HV Clio (both paid); substitution is brand-book-sanctioned on page 13.

### 5.5 Image URL unreachable / expired
`render.html` waits up to 8s per image (`bind: bg` fields). If the source URL 404s, the render continues with a broken image placeholder. Always test photo URLs with a HEAD request if unsure. For listings, prefer hotlinking from Otodom/EstiCRM directly over self-hosted uploads (stable CDNs).

### 5.6 Meta error responses — read them
`MetaClient.request()` preserves `error_subcode`, `fbtrace_id`, `error_user_title`, `error_user_msg`. If a publish fails, surface all four to Tomek before retrying. Don't swallow. Don't retry blindly.

---

## 6. Raw API reference (for deep-debugging)

If the MCP tool is misbehaving, these are the underlying Fly endpoints:

```
GET  /health                     → liveness probe
GET  /t/:id?data=<json>          → opens template in-browser, no auth (dev only)
POST /render/:id   body={...}    → returns rendered JPEG bytes (auth required in prod)
POST /store/:id    body={...}    → renders + stores, returns stable URL (auth required)
GET  /a/:filename                → serves stored image (public)
GET  /v/:filename                → serves stored video (public)
POST /compose/:id  body={...}    → composes intro+video+outro MP4 (future)
```

Auth header: `Authorization: Bearer <ZBORALSCY_API_KEY>`. Key lives in Bellink env, not here.

Config env vars on Fly:
```
PORT=4300
API_KEY=<from env>
PUBLIC_BASE_URL=https://zboralscy-templates.fly.dev
ASSET_DIR=/data/assets
VIDEO_DIR=/data/videos
TMP_DIR=/data/tmp
```

Render spec: JPEG quality 90. Puppeteer polls `window.__READY === true` before screenshot — render.html sets the flag once fonts + all `bind: bg` images finish loading (8s timeout per image).

---

**Next reading:** [`playbooks/`](playbooks/) — concrete recipes chaining these tools end-to-end.
