---
name: playbook-local-spotlight
description: Recipe for a 9:16 reel — neighborhood spotlight, drone listing, or luxury property tour. Uses zboralscy_compose_reel to chain T3C intro + Tomek-supplied video + T6 outro into one MP4. Triggers on "reel", "drone", "spacer", "wirtualny spacer", "luxury".
---

# Playbook · Local / Luxury Spotlight (Reel)

Create a 9:16 reel spotlight for a neighborhood or luxury listing. Uses **`zboralscy_compose_reel`** — server-side ffmpeg composes T3C intro + Tomek's source video + T6 outro into one MP4. Output goes straight to Meta publishing.

---

## Two paths

| Path | When | How |
|---|---|---|
| **A. Server-composed (default)** | Tomek has a video clip he wants in the middle. URL must be public (Drive share / Dropbox share / WeTransfer / direct .mp4 / YouTube). | One `zboralscy_compose_reel` call returns the finished MP4. |
| **B. Frames-only** | Tomek wants to edit in CapCut/Premiere himself (custom transitions, music bed, subtitles). | Two `zboralscy_render_post` calls (T3C intro frame + T6 outro frame), Tomek composes externally. |

Path A is the new default for weekend launch. Path B documented at the bottom for editor-led workflows.

---

## When to run (Path A — composed reel)

**Trigger phrases (Polish):**
- "reel dla tej oferty"
- "story luxury" / "premium listing"
- "reel z drona"
- "lokalne" / "sąsiedztwo reel"
- "spotlight [dzielnica]"
- "złóż reel z mojego wideo"

**Tomek's use cases:**
1. **Luxury / drone listing** → T3C (drone story) + T6. Example: dom z widokiem, apartament premium, działka z widokiem na puszczę.
2. **Neighborhood spotlight** → T3C repurposed with neighborhood shot + hook about the area. Example: "Dojlidy Górne — 5 min do centrum, 0 min do lasu".
3. **Hook-list reel (educational)** → T3A instead of T3C (that's a different playbook: `tip-reel.md` TBD).

If the case looks educational (Q&A, lista), redirect to T3A/T3B — don't force T3C.

---

## Required fields

### For T3C (opening)

| Field      | Source                          | Required? |
|------------|---------------------------------|-----------|
| `location` | Tomek / EstiCRM / context       | ✅         |
| `title`    | Hook — short, 2 lines max       | ✅         |
| `area`     | m² (tylko dla listingu)         | ⚠️ pomiń dla neighborhood spotlight |
| `price`    | cena z "zł"                     | ⚠️ pomiń dla neighborhood spotlight |
| `cta`      | Call-to-action                  | ✅ (default: "Zobacz wirtualny spacer →") |
| `photo`    | Hero shot / video first-frame   | ✅         |

### For T6 (outro)

**Always the same values** — Tomek's contact card. Synth uses defaults UNLESS Tomek says otherwise:

```
agentName: "Tomasz<br>Brynkiewicz"
agentRole: "Pośrednik · Doradca"
phone:     "+48 669 996 948"
email:     "bialystok@zboralscy-group.pl"
www:       "zboralscy-group.pl"
portrait:  <Tomek's portrait URL — stored, doesn't change>
```

The portrait URL should live in synth memory (future: config file). For now, ask Tomek once per session and cache.

---

## Steps

### 1. Confirm the use case

Ask Tomek one sentence:
> "Reel: listingowy (T3C z ceną), czy spotlight dzielnicy (T3C bez ceny)?"

Based on answer, determine which subset of fields is needed.

### 2. Craft the hook (`title`)

This is the single most important element. Hook at the top of the reel is the lead — see [`../handbook.md` § 3](../handbook.md).

**Listingowy hook patterns (Polish):**
- `Dom z widokiem<br>na puszczę`
- `Apartament 5 min<br>od Rynku`
- `Pod Białymstokiem,<br>20 km od centrum`
- `[Feature]<br>[Context]`

**Neighborhood hook patterns:**
- `Dojlidy Górne<br>5 min do lasu`
- `Centrum bez hałasu<br>— jak to możliwe?`
- `Gdzie kupują<br>młode rodziny w 2026`

Max 2 lines (use `<br>`). Keep lines ~15-20 characters each. Bigger = better readability in thumb scroll.

**Pause here.** Show draft hook(s) to Tomek. Accept edits. Don't proceed until he says "ok".

### 3. Pick the photo

- **Luxury / drone**: wide shot with space, sky, view. NOT close-up. If Tomek has a drone video, use the first frame (download externally, upload to the render — see `photo` field accepts any URL).
- **Neighborhood**: ulica, park, rynek, coś charakterystycznego. Unikaj zwykłych szerokopanoramowych zdjęć — szukaj kadru z duszą.
- **Avoid**: close-up details (kuchnia blat), dark interiors, small portions of building. Template overlays text on top half → photo needs space.

### 4. Get source video URL from Tomek

The composed reel needs a public video URL for the middle section. Ask:

> "Wyślij mi link do wideo (Drive, Dropbox, WeTransfer, YouTube — byle publiczny). Powiedz, od której sekundy mam zacząć i ile sekund użyć (domyślnie 0, 10s)."

**Accepted source types:**
- Direct .mp4 / .mov / .webm / .mkv URL → fetched via curl (fastest)
- Drive / Dropbox / OneDrive share link with public access → yt-dlp
- YouTube / Vimeo URL → yt-dlp with lazy section fetch
- WeTransfer link → only if it's a direct download URL (not a wetransfer.com share page)

**If Tomek pastes a Drive link**, make sure it's set to "Anyone with the link". If permissions are wrong, compose returns "upstream 403" — re-ask.

### 5. Compose the reel

```
zboralscy_compose_reel({
  source: {
    url:      "<Tomek's video URL>",
    start:    <seconds, default 0>,
    duration: <seconds, default 10, max 30>
  },
  data: {
    // T3C intro frame fields
    location: "<Miasto lub Dzielnica · kontekst>",
    title:    "<hook z kroku 2, z <br>>",
    area:     "<m²>" | (omit),
    price:    "<cena z 'zł'>" | (omit),
    cta:      "Zobacz wirtualny spacer →" | "<custom>",
    // T6 outro frame fields (the same data block feeds both)
    agentName: "Tomasz<br>Brynkiewicz",
    agentRole: "Pośrednik · Doradca",
    phone:     "+48 669 996 948",
    email:     "bialystok@zboralscy-group.pl",
    www:       "zboralscy-group.pl",
    portrait:  "<cached portrait URL>",
    // Photo for T3C intro frame (can be a still from the source video first frame)
    photo:     "<URL zdjęcia, optional>"
  },
  intro:    "T3C",  // default
  outro:    "T6",   // default
  introSec: 1.5,    // default
  outroSec: 2.0,    // default
  jobLabel: "<short label e.g. BOJARY>"
})
```

Returns `{ id, url, width: 1080, height: 1920, durationSec, bytes, intro, outro }`. The `url` is already `bellink.io/zboralscy/img/<hash>.mp4` — Meta-safe.

**Latency:** 30-90s. Tell Tomek "moment, składam reel — około minuty." Don't time-out internally.

Show the URL to Tomek (he can click to preview the MP4 in browser).

### 6. Draft the caption

Reel captions are short — the reel does the work. Lead with the hook, drop key context, hashtags.

```
[Hook — same line as the reel intro frame title]

[1-2 zdania o ofercie / temacie]

Pełne dane w komentarzu ↓

#Białystok #[Dzielnica] #Nieruchomości
```

In the first comment after publishing: full data + price + contact + virtual tour link. Algorithm IG rewards comment engagement.

### 7. Publish to Instagram

```
meta_create_instagram_reel({
  video_url: <url z kroku 5>,
  caption:   <tekst z kroku 6>
})
```

If `meta_create_instagram_reel` tool is unavailable in your Bellink session, fall back to `meta_create_instagram_post` with a `video_url` parameter (some Bellink versions accept video on the same endpoint).

### 8. Publish to Facebook Page

```
meta_create_page_post({
  message:   <caption>,
  video_url: <same MP4 URL>
})
```

(FB's video posting goes through `/feed` with `video_url`, unlike images which need `meta_create_page_photo`.)

### 9. Confirm + first-comment data drop

```
✅ Reel opublikowany

IG: <link>
FB: <link>
MP4: <bellink.io URL>

Czy mam dorzucić w pierwszym komentarzu pełne dane oferty (cena, m², link do wirtualnego spaceru)?
```

If Tomek says yes → call the IG comment API (if available) or remind him to drop the comment manually.

---

## Output contract — Path A

```
✅ Reel opublikowany

IG: <link>
FB: <link>
MP4: <bellink.io URL>
Trwanie: <durationSec> s
Hook: "<title>"
Source: <video URL>
```

Failure:

```
⚠️ Nie złożono / nie opublikowano

Krok: <gdzie padło — fetch / compose / publish>
Błąd: <exact error from upstream>
Sugerowana akcja: <eg. "Sprawdź uprawnienia do linku Drive — musi być Anyone with the link">
```

---

## Failure recovery

| Problem | Action |
|---|---|
| Source URL niedostępny / 403 | Drive: ustaw "Anyone with the link can view". Dropbox: użyj `?dl=1` na końcu URL. WeTransfer: musi być direct download URL, nie strona share. |
| `intro X is not 1080x1920` | Intro/outro musi być reelowy template. Użyj T3A/T3B/T3C jako intro, T6 jako outro. |
| Compose ffmpeg error | Source video uszkodzony lub format nietypowy. Spróbuj re-eksport do .mp4 H.264. Pokaz Tomkowi ostatnie 800 znaków stderr z błędu. |
| Brak dobrego photo dla intro frame | Pomiń `photo` field — T3C wyrenderuje się z domyślnym tłem. Albo użyj first-frame z video (Tomek wygeneruje screenshot z .mp4). |
| Hook za długi (3+ linie) | Tnij. Max 2 linie, ~15-20 znaków na linię. |
| Reel > 30s żądany | Compose service hard-clampuje source.duration do 30s. Plus intro 1.5s + outro 2s = ~33.5s max total. IG reels dopuszczają do 90s, ale dla naszego template tooling 30s to limit. |
| Błąd 9004 / Meta odrzuca MP4 | Sprawdź czy URL to `bellink.io/zboralscy/img/...` (NIE `fly.dev/v/...`). Jeśli URL wygląda dobrze → ping Bartek, może Meta dodało nową regułę reputation. |
| Tomek nie ma video, chce tylko ramki | Path B (frames-only) — sekcja niżej. |

---

## Edge cases

- **Wewnętrzne ujęcia mieszkania (nie drone, nie neighborhood)**: T3C nadal pasuje, ale rozważ T3A (hook-list style) jako intro dla ciekawszej dynamiki — w `compose_reel` wystarczy ustawić `intro: "T3A"`.
- **Reel bez ceny / "wkrótce w ofercie"**: T3C bez `area` i `price`, framing: "Premiera już w maju — zobacz pierwszy podgląd".
- **Neighborhood reel z kilkoma scenami**: compose obsługuje JEDNO źródło wideo. Jeśli Tomek ma kilka klipów, najpierw musi je skleić w jedno (CapCut), potem wrzucić jako jedną source URL.
- **Współpraca z videografikiem**: jeśli Tomek wysyła nam gotowy reel bez intro/outro — wciąż możemy dogrywać brand frames przez `compose_reel` (kompozytor zwraca `source.duration` całego klipu, owinie naszymi frames).

---

## Path B — Frames-only (manual edit)

Use this when Tomek's editor wants full control over transitions / music / subtitles. Render two stand-alone frames; Tomek composes externally.

### Steps

1. **T3C intro frame:**
   ```
   zboralscy_render_post({ templateId: "T3C", data: { location, title, area, price, cta, photo } })
   ```

2. **T6 outro frame:**
   ```
   zboralscy_render_post({ templateId: "T6", data: { agentName, agentRole, phone, email, www, portrait } })
   ```

3. **Deliver to Tomek:**
   ```
   ✅ Frames wyrenderowane (do montażu manualnego)

   INTRO (T3C, 1080×1920): <URL>
   OUTRO (T6, 1080×1920): <URL>

   Następne kroki (CapCut / Premiere / DaVinci):
   1. Ściągnij oba frame'y
   2. INTRO 1.5-2s + Twoje wideo + OUTRO 2s
   3. Muzyka w tle (royalty-free)
   4. Burned-in napisy PL
   5. Eksport: 9:16, 1080×1920, 30fps, MP4
   6. Wyślij MP4 — opublikuję
   ```

4. **When Tomek returns with the final MP4**, publish via `meta_create_instagram_reel` + `meta_create_page_post` (with `video_url`). Same caption flow as Path A step 6.

Path B is rare — default to Path A unless Tomek explicitly says he wants to edit himself.

---

**Back:** [`../SKILL.md`](../SKILL.md) · [`../handbook.md`](../handbook.md) · [`../fly-templates.md`](../fly-templates.md)
