# Playbook · Local / Luxury Spotlight (Reel)

Create a 9:16 reel spotlight for a neighborhood or luxury listing. Uses T3C (cover / opening frame) + T6 (agent outro).

---

## Scope — what this playbook does TODAY vs tomorrow

**Today (v1 — weekend launch):**
Render the two key STILL frames for the reel:
- **T3C** — opening / hook frame (9:16, 1080×1920)
- **T6** — agent outro (9:16, 1080×1920, always the closing frame)

Tomek (or his editor) composes the final MP4 externally — prepend T3C as intro (~1.5s), use his own apartment/neighborhood video in the middle, append T6 as outro (~2s). Music bed + subtitles added in his video tool.

**Tomorrow (v2 — post-weekend):**
Full server-side composition via `zboralscy_compose_reel` — ffmpeg on Fly stitches intro + video + outro automatically. Backlog item, ~2-3h build. See [`../fly-templates.md` § 2.3](../fly-templates.md) and `~/ops-vault/businesses/tomek/backlog.md`.

**This playbook covers v1.** Update it when `zboralscy_compose_reel` ships.

---

## When to run

**Trigger phrases (Polish):**
- "reel dla tej oferty"
- "story luxury" / "premium listing"
- "reel z drona"
- "lokalne" / "sąsiedztwo reel"
- "spotlight [dzielnica]"

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

### 4. Render T3C (opening frame)

```
zboralscy_render_post({
  templateId: "T3C",
  data: {
    location: "<Miasto lub Dzielnica · kontekst>",
    title:    "<hook z kroku 2, z <br>>",
    area:     "<m²>" | (omit),
    price:    "<cena z 'zł'>" | (omit),
    cta:      "Zobacz wirtualny spacer →" | "<custom>",
    photo:    "<URL zdjęcia>"
  }
})
```

Returns URL. Show preview to Tomek.

### 5. Render T6 (outro frame)

```
zboralscy_render_post({
  templateId: "T6",
  data: {
    agentName: "Tomasz<br>Brynkiewicz",
    agentRole: "Pośrednik · Doradca",
    phone:     "+48 669 996 948",
    email:     "bialystok@zboralscy-group.pl",
    www:       "zboralscy-group.pl",
    portrait:  "<cached portrait URL>"
  }
})
```

Returns URL. Show preview.

### 6. Deliver package to Tomek

**DO NOT publish automatically** — this is still a manual-edit workflow in v1.

```
✅ Frames wyrenderowane

INTRO (T3C, 9:16): <URL>
OUTRO (T6, 9:16): <URL>

Następne kroki (manualnie):
1. Ściągnij oba frame'y (zapisz jako .jpg)
2. W edytorze wideo (CapCut / Premiere / DaVinci):
   • wklej INTRO jako pierwszy klip (1.5-2s)
   • dodaj Twoje wideo apartamentu / dzielnicy w środku (25-27s)
   • wklej OUTRO jako ostatni klip (2s)
3. Dodaj muzykę w tle (royalty-free — Epidemic, Artlist, Pixabay)
4. Burned-in napisy po polsku (synced z voiceover lub samoopisowe)
5. Eksport: 9:16, 1080x1920, 30fps, MP4
6. Kiedy reel gotowy — wyślij do mnie i opublikuję na IG/FB

Draft caption do posta:
[generate caption — short, hook-first, hashtags]
```

### 7. When Tomek returns with the final MP4

Publish manually:

```
meta_create_instagram_reel({   // if tool available — otherwise meta_create_instagram_post with video_url
  video_url: <Tomek's uploaded MP4>,
  caption:   <caption z kroku 6, poprawiony>
})
```

Follow same error-handling pattern as other playbooks.

---

## Output contract

On frame delivery (step 6):

```
✅ Frames gotowe — czekam na Twój finalny reel

INTRO: <URL T3C>
OUTRO: <URL T6>
Hook: "<title>"
Location: "<location>"

Instrukcja montażu powyżej. Kiedy masz MP4 — daj znać, opublikuję.
```

On reel publish (step 7):

```
✅ Reel opublikowany

IG: <link>
FB: <link>
Caption: <tekst>
```

---

## Failure recovery

| Problem | Action |
|---|---|
| Brak dobrego photo | Zaproponuj Tomkowi, żeby zrobił zdjęcie drone'm / pojechał sfotografować miejsce. Nie renderuj z placeholder. |
| Hook za długi (3+ linie) | Tnij. Max 2 linie, ~15-20 znaków na linię. |
| Tomek chce composite server-side | Powiedz mu że to jest `zboralscy_compose_reel`, jeszcze nie wdrożone. Oszacowanie: ~2-3h buildu, backlog. |
| Tomek wysłał MP4 ale Meta odrzuca | Sprawdź format: MP4, H.264, 1080x1920, max 90s dla reela. Rozmiar: max 100MB IG, max 250MB FB. |
| Błąd 9004 na video | Ta sama host-reputation reguła — video też przez Bellink-proxy? Sprawdź. Jeśli Tomek uploaded direct — przekieruj przez Bellink. |

---

## Edge cases

- **Wewnętrzne ujęcia mieszkania (nie drone, nie neighborhood)**: T3C nadal pasuje, ale rozważ T3A (hook-list style) dla ciekawszej dynamiki.
- **Reel bez ceny / "wkrótce w ofercie"**: T3C bez `area` i `price`, framing: "Premiera już w maju — zobacz pierwszy podgląd".
- **Neighborhood reel z kilkoma scenami**: T3C tylko jako intro, ciało reela to Tomka seria klipów — template nie kontroluje środka.
- **Współpraca z videografikiem**: jeśli Tomek wysyła nam gotowy reel bez intro/outro — nadal możemy dogrywać nasze frames w post-production (konsystencja brandu).

---

**Back:** [`../SKILL.md`](../SKILL.md) · [`../handbook.md`](../handbook.md) · [`../fly-templates.md`](../fly-templates.md)
