---
name: playbook-new-listing
description: End-to-end recipe for publishing a new property listing as an Instagram carousel + Facebook page photo. Triggers on "nowa oferta", "nowe mieszkanie", "wystaw", "opublikuj listing", "carousel". Chains T1 cover + T8 room slides + T4C closer, then publishes to IG + FB.
---

# Playbook · New Listing

Publish a new offer to Instagram and Facebook from a single listing.

---

## When to run

**Trigger phrases (Polish):**
- "nowa oferta"
- "zrób post dla tej oferty"
- "listing spotlight"
- "wystawiamy na sprzedaż"
- "opublikuj mieszkanie" / "opublikuj dom"

**Typical inputs from Tomek (any subset):**
- EstiCRM offer ID (e.g. `#48231`)
- Photo URL or attached image
- Address / dzielnica + specs (pokoje, m², piętro, rok, cena)
- Wolny-format opis ("2 pokoje w Centrum, 52 m², 620 tys.")

---

## Required fields (before rendering)

All must be confirmed. If any missing, **ask** — don't guess.

| Field        | Source                           | Required? |
|--------------|----------------------------------|-----------|
| `location`   | Tomek's input / EstiCRM offer    | ✅         |
| `price`      | Tomek's input / EstiCRM offer    | ✅ (format: `749 000` — no "zł", no dot-separator) |
| `rooms`      | Tomek's input / EstiCRM offer    | ✅         |
| `area`       | Tomek's input / EstiCRM offer    | ✅         |
| `floor`      | Tomek's input / EstiCRM offer    | ⚠️ optional (skip if house / parter nieistotne) |
| `year`       | Tomek's input / EstiCRM offer    | ⚠️ optional (skip if nieznany) |
| `photo`      | Tomek's input / EstiCRM offer    | ✅         |
| `badge`      | Default "Nowa oferta"            | use default |

---

## Steps

### 1. Pull or confirm offer data

- If Tomek gave an **EstiCRM offer ID**: `esticrm_get_offer({id})` → parse fields. Confirm the photo URL back to Tomek.
- If Tomek gave a **free-form description**: parse what you can, ask for anything missing.
- If Tomek **attached an image**: you still need location + specs + price. Ask.

**Pause here.** Show Tomek the fields you parsed. Ask: "Czy wszystko się zgadza? Jeśli tak — renderuję." Wait for "tak" / "renderuj" / equivalent.

### 2. Render the T1 image

```
zboralscy_render_post({
  templateId: "T1",
  data: {
    badge:    "Nowa oferta",
    location: "<Miasto · Dzielnica>",
    price:    "<sformatowana liczba, bez zł>",
    rooms:    "<string>",
    area:     "<string z m²>",
    floor:    "<X/Y>" | (omit),
    year:     "<YYYY>" | (omit),
    photo:    "<URL zdjęcia>"
  }
})
```

Returns `{ id, url, width, height, bytes }`. **Use `url` verbatim** — it's already `bellink.io/zboralscy/img/…`, Meta-safe. See [`../fly-templates.md` § 4](../fly-templates.md) if you forget why.

Show the URL to Tomek (he can click to preview). If he says "popraw [pole]" → adjust data, re-render. If "ok" → proceed.

### 3. Draft the caption

Use the T1 formula from [`../handbook.md` § 4.1](../handbook.md):

```
[Hook — one line, from handbook §3.1 pattern library]

📍 [location]
🏠 [rooms] · [area] · [floor] · [year]
💰 [price] zł

[2-3 zdania opisu — dane, nie adjektiwy. Najmocniejsze cechy: widok, rozkład, stan, lokalizacja.]

Umów oglądanie →
📞 +48 669 996 948
🏷️ Tomasz Brynkiewicz · Pośrednik · Doradca

#Białystok #[Dzielnica] #Nieruchomości[Dzielnica] #Zboralscy
```

**Hashtag rules:** 3-5 max. Always: `#Białystok`, `#Zboralscy`, `#[Dzielnica]`. Plus 1-2 contextual (`#3pokoje`, `#NowaOferta`, `#ApartamentWBiałymstoku`).

**Show the caption to Tomek.** Accept edits. Do not publish until he says "publikuj".

### 4. Publish to Instagram

**Single image** (one slide only):
```
meta_create_instagram_post({
  instagramAccountId: <id>,
  imageUrl:           <url z kroku 2>,
  caption:            <finalny tekst z kroku 3>
})
```

**Carousel** (2–10 slides — typical for new listing: T1 cover + T8 room slides + T4C closer):
```
meta_create_instagram_carousel({
  instagramAccountId: <id>,
  imageUrls:          [<url-cover>, <url-slide-2>, ..., <url-closer>],
  caption:            <finalny tekst z kroku 3>
})
```

**Always use the carousel tool when posting more than one image** — it produces ONE swipeable post on IG. Calling `meta_create_instagram_post` 6× produces 6 separate posts, NOT a carousel. Confirmed in 2026-04-27 test session.

Check response. If 9004 or other error: read `error_subcode`, `fbtrace_id`, `error_user_msg` — report verbatim to Tomek, do NOT retry.

### 5. Publish to Facebook Page

**Single image:**
```
meta_create_page_photo({
  pageId:   <id>,
  imageUrl: <same url>,
  caption:  <same caption, lekko dostosowany — FB pozwala na dłuższy opis>
})
```

**Carousel** (2–10 photos as ONE swipeable feed post):
```
meta_create_facebook_carousel({
  pageId:    <id>,
  imageUrls: [<url-cover>, <url-slide-2>, ..., <url-closer>],
  message:   <same caption>
})
```

**Same rule as IG:** if you have more than one image, use the carousel tool. Calling `meta_create_page_photo` N times produces N separate album/photo posts on the page, NOT a carousel.

**Do NOT use `meta_create_page_post`** — it silently drops image params (lying API). If in doubt, re-read [`../fly-templates.md` § 5.2](../fly-templates.md).

### 6. Confirm + log

Report back to Tomek:
- ✅ IG post ID (link, if Meta returns one)
- ✅ FB post ID (link, if Meta returns one)
- 🔗 Rendered image URL (`bellink.io/zboralscy/img/…`)

Ask if he wants a story version with link sticker (future — currently manual in IG app).

---

## Output contract

On success, the synth responds with one block:

```
✅ Opublikowano

IG: <link do posta>
FB: <link do posta>
Zdjęcie: <bellink.io URL>

Oferta: [miasto · dzielnica] · [pokoje] · [m²] · [cena] zł
Hasztagi: [lista]

Chcesz story z linkiem?
```

On failure:

```
⚠️ Nie opublikowano

Krok: <gdzie padło>
Błąd Meta:
  code:        <error.code>
  subcode:     <error_subcode>
  fbtrace_id:  <fbtrace_id>
  user_msg:    <error_user_msg>

Nie ponawiam automatycznie. Co robimy?
```

---

## Failure recovery

| Problem | Action |
|---|---|
| Tomek zmienił zdanie po rendere | Nie usuwaj rendera (trwały URL). Re-render z nowymi danymi. |
| Błąd 9004 na IG | Sprawdź czy URL to `bellink.io` (nie `fly.dev`). Jeśli tak → ping Bartek. |
| Tomek dał zły photo URL (404) | Render zadziała z broken image. Usuń, poproś o prawidłowy link, re-render. |
| EstiCRM offer ID nieznaleziony | Ask Tomek for address + specs manually. |
| Cena ma grosze / dziwny format | Renderuj jako całkowita: `749 000` (nie `749 000,00`). |
| Tomek chce CENĘ z "zł" na renderze | T1 nie wspiera "zł" w polu price (projekt templata). Cena sama, bez waluty. Jeśli nalega → T3C (story) zamiast T1. |

---

## Edge cases (don't skip these)

- **Dom vs mieszkanie**: T1 robi oba. Dla domu `floor` pomiń, `year` = rok budowy.
- **Bezczynszowe / komercyjne**: T1 nadal się nadaje, ale rozważ prostszy opis.
- **Reduction of price (obniżka)**: NIE nowa oferta — osobny playbook (TBD: `price-drop.md`).
- **Kilka zdjęć**: T1 używa JEDNEGO. Wybierz najmocniejsze (szeroki kąt, światło naturalne). Carousel → osobny playbook (TBD: `listing-carousel.md`).

---

**Back:** [`../SKILL.md`](../SKILL.md) · [`../handbook.md`](../handbook.md) · [`../fly-templates.md`](../fly-templates.md)
