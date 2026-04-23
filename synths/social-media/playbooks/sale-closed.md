# Playbook · Sale Closed (Sprzedane)

Announce a closed deal on Instagram and Facebook — proof-of-work content.

---

## When to run

**Trigger phrases (Polish):**
- "sprzedane"
- "zamknęliśmy"
- "umowa przedwstępna podpisana"
- "oferta zamknięta"
- "post sprzedane"

**Timing rule:** Post within 48h of closing. Świeżość = wiarygodność. If more than a week passed, the story is cold — ask Tomek if he still wants it.

---

## Privacy rule (non-negotiable)

**NEVER publish the exact transaction price.** NDA / trust issues.

What we publish:
- ✅ Lokalizacja (dzielnica poziom, NIE adres / numer)
- ✅ Typ (pokoje, m²)
- ✅ Czas sprzedaży (dni od listingu do umowy)
- ✅ Liczba zainteresowanych (leads)
- ✅ Discount od ceny wyjściowej — **tylko jeśli pozytywny sygnał** (np. "sprzedane w cenie wyjściowej" lub "–4 000 zł" przy dobrej wycenie)
- ❌ Konkretna cena transakcji
- ❌ Dokładny adres, numer mieszkania
- ❌ Imię/nazwisko kupującego lub sprzedającego

If Tomek insists on a field that breaks the privacy rule, push back: "tego nie publikujemy — zaszkodzi kolejnym klientom." Don't go along.

---

## Required fields

| Field        | Source                         | Required? |
|--------------|--------------------------------|-----------|
| `location`   | Tomek / EstiCRM                | ✅         |
| `kicker`     | Big tagline, crafted from days | ✅ ("Sprzedane w X dni") |
| `days`       | Dni od listingu do umowy       | ✅         |
| `discount`   | +/– od ceny wyjściowej         | ⚠️ optional, tylko pozytywny |
| `leads`      | Ile osób zapytało              | ✅         |
| `photo`      | Zdjęcie obiektu (to samo co T1) | ✅         |

---

## Steps

### 1. Pull closing data

- **EstiCRM path**: `esticrm_get_offer({id})` → sprawdź status "sprzedane", pobierz `created_at` + `closed_at` → oblicz `days`. Liczba leadów z EstiCRM albo od Tomka.
- **Free-form path**: Tomek podaje dzielnicę / m² / pokoje / dni / leads. Confirm back.

**Pause here.** Pokaż Tomkowi parsowane dane + draft `kicker`:
- 1-7 dni → "Sprzedane w rekordowym czasie"
- 8-30 dni → "Sprzedane w <N> dni"
- 31-60 dni → "Sprzedane. <N> dni, <leads> osób zainteresowanych."
- 60+ dni → rozważ czy w ogóle postujemy jako "szybką sprzedaż" — raczej focus na jakości dopasowania

### 2. Render the T2 image

```
zboralscy_render_post({
  templateId: "T2",
  data: {
    location: "<Dzielnica · Miasto>",      // np. "Pałacowa · Białystok"
    kicker:   "Sprzedane w <N> dni",
    days:     "<N> dni",
    discount: "–<kwota> zł" | "w cenie wyjściowej" | (omit),
    leads:    "<N> osób",
    photo:    "<URL zdjęcia>"
  }
})
```

Show URL to Tomek. Accept edits.

### 3. Draft the caption

Formula from [`../handbook.md` § 4.2](../handbook.md):

```
Sprzedane.

[Dzielnica] · [pokoje] · [m²]
[N] dni od listingu do umowy. [leads] osób zapytało.

[1 zdanie: dlaczego się sprzedało szybko — konkret. Wycena. Zdjęcia. Home staging. Czas roku.]

Masz podobne mieszkanie? Umów bezpłatną wycenę.
📞 +48 669 996 948

#Sprzedane #Białystok #[Dzielnica]
```

Dla długich sprzedaży (31-60 dni): skup się na **jakości dopasowania kupującego** zamiast szybkości:

```
Sprzedane.

[Dzielnica] · [pokoje] · [m²]
[N] dni. [leads] osób zapytało. Jedno się dopasowało.

Dobra wycena + cierpliwość = umowa, która działa po obu stronach.

Masz podobne mieszkanie? Umów bezpłatną wycenę.
📞 +48 669 996 948
```

Show caption. Edits. Confirm.

### 4. Publish to Instagram

```
meta_create_instagram_post({
  image_url: <url z kroku 2>,
  caption:   <finalny tekst>
})
```

### 5. Publish to Facebook Page

```
meta_create_page_photo({
  image_url: <same url>,
  message:   <same caption>
})
```

**Do NOT use `meta_create_page_post`.**

### 6. Confirm + next-step nudge

```
✅ Opublikowano

IG: <link>
FB: <link>

Czy chcesz też teraz zgarnąć pipeline?
→ "Post o free wycenie" (T1 badge = "Bezpłatna wycena")?
→ "Ogłoszenie o nowym zleceniu" (jeśli już masz kolejne)?
```

Proof-of-work posts conwertują najlepiej jeśli są PARZONE z CTA na nowy listing lub wycenę. Don't skip step 6.

---

## Output contract

Success:

```
✅ Opublikowano SPRZEDANE

IG: <link>
FB: <link>
Zdjęcie: <bellink.io URL>

[Dzielnica] · [N] dni · [leads] zapytań
```

Failure:

```
⚠️ Nie opublikowano

Krok: <gdzie padło>
Błąd Meta: code / subcode / fbtrace_id / user_msg
```

---

## Failure recovery

| Problem | Action |
|---|---|
| Tomek chce dać cenę transakcji w poście | Odmów — privacy rule. Zaproponuj "discount" jeśli pozytywny, albo pomiń. |
| Sprzedane po obniżce dużej (-20%+) | NIE eksponuj obniżki. Zmień framing na "dopasowanie kupującego". |
| Photo URL z oferty już nie działa (wylistowana) | Tomek musi wrzucić zdjęcie ręcznie lub wziąć z własnego archiwum. |
| Tomek chce pokazać imię kupującego | Odmów — privacy rule. Jeśli kupujący sam publikuje post-sprzedażowy, to jego sprawa. |
| Liczba leadów nieznana | Estymuj ostrożnie lub pomiń — nie zmyślaj konkretnej liczby. |

---

## Edge cases

- **Sprzedaż przez innego agenta / pokazanie**: jeśli Tomek nie był jedynym agentem, framing ostrożnie — nie kradnij cudzego kredytu.
- **Zamknięcie z komplikacją (wycofany kupujący, druga runda)**: lepiej pominąć post — nie jest to czysty proof-of-work.
- **Klient prosił o dyskrecję**: nie postujemy wcale. Tomek powinien spytać przed publikacją — jeśli nie pytał, spytaj go.

---

**Back:** [`../SKILL.md`](../SKILL.md) · [`../handbook.md`](../handbook.md) · [`../fly-templates.md`](../fly-templates.md)
