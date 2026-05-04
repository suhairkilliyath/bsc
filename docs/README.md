# Bangalore Surf Club — content guide

Everything you'd want to edit lives in **`config.js`**. Both `index.html` and `program.html` read from the same `CONFIG` object, so a change in `config.js` updates both pages.

```
personal/
├── config.js        ← all editable content (this is the file you'll touch most)
├── index.html       ← homepage (hero, calendar, swell map, logbook, ethos, FAQ, join)
├── program.html     ← annual-program builder (picker + trips)
├── favicon.svg
├── og-cover.svg
└── images/          ← photos referenced from the logbook
```

## How to edit content

Open `config.js`. It's one big object with named sections:

| You want to change… | Edit this section in `config.js` |
| --- | --- |
| Club name, tagline, contact email, Instagram handle | `club` |
| The 8 surf destinations (lat/lng, season, swell rating, difficulty, blurb) | `destinations` |
| The per-month picks shown on the homepage Calendar **and** used by the Program builder | `calendar` |
| Past + upcoming trips (the "Where we've been / Where we're going" grid) | `logbook.entries` |
| The four "what we are / aren't" cards | `ethos` |
| Headlines on the swell map section | `swellMap` |
| The bottom join bar text + CTA | `joinBar` |
| The "Talk to us" form (labels, role dropdown, success message) | `joinForm` |

After saving, just refresh the page in the browser — no build step.

---

## Section-by-section guide

### `club` — top-of-site basics

```js
club: {
  name: "Bangalore Surf Club",
  tagline: "Monthly surf trips from Bangalore.",
  taglineAccent: "from Bangalore.",   // the italic green part of the tagline
  established: "Est. 2026",
  contactEmail: "hello@blsurf.club",
  instagram: "https://instagram.com/bangaloresurfclub",
  formspreeId: "mkokaawo",            // form submissions go here
}
```

`taglineAccent` must be a substring of `tagline` — the page splits on it to render the italic accent.

### `destinations` — the 8 surf spots

Each entry powers the swell map pin, the in-season detector on the hero, and is referenced **by `name`** from `calendar`.

```js
{
  month: "Oct–Mar",                  // season window — uses an en-dash (–), NOT a hyphen
  name: "Varkala",                   // ← must match calendar picks exactly
  region: "Kerala",
  lat: 8.74, lng: 76.71,             // map pin
  swell: 5,                          // 1–5 (5 = strongest seasonal swell)
  peak: true,                        // true = bold green pin in season; false = lighter
  coords: "8.74°N / 76.71°E",        // display string in the swell-map info card
  temp: "26–28°C",                   // water temp display string
  difficulty: "Beginner to Intermediate",
  quote: "A long sand point under red laterite cliffs…",
  sourceUrl: "https://www.surf-forecast.com/breaks/Varkala"
}
```

**Rules:**
- `month` uses an **en-dash** `–` (not `-`). Wrap-arounds like `"Nov–Apr"` are supported.
- `swell` is 1–5. 5 = peak. Used both as the swell-map bar count and the homepage chip dot intensity.
- `difficulty` is a free-text string but the swell-map filter looks for the words **"beginner"**, **"intermediate"**, **"advanced"**, or **"all"** inside it. So write things like `"Beginner to Intermediate"`, `"Intermediate to Advanced"`, or `"All levels"` — anything else won't filter correctly.
- `name` is the join key with `calendar`. If you rename a destination here, find/replace every `dest: "<old name>"` in `calendar` too.

#### Adding a new destination

1. Add a new object to `destinations` with all required fields.
2. Add it to as many months in `calendar` as it works in (under the right skill levels). If you skip this step, the spot only appears on the swell map, not the calendar or program.

### `calendar` — per-month picks per skill level

Each of the 12 months has three lists of picks (one per level). The **first pick is the primary** shown on the homepage and used as the program-builder default; the rest are alternates and appear as "Also working:" chips.

```js
{ month: "Jan", picks: {
  beginner:     [
    { dest: "Varkala",  note: "First-timers welcome…" },     // primary
    { dest: "Weligama", note: "Wide bay, sandy bottom…" },   // alternate 1
    { dest: "Goa",      note: "Small fun waves…" }           // alternate 2
  ],
  intermediate: [ /* same shape */ ],
  advanced:     [ /* same shape */ ]
}}
```

**Rules:**
- `dest` must match exactly one `destinations[].name`. Typos = silently broken card.
- Months **must** stay in order Jan → Dec. The program builder treats them as a circular calendar starting from any month.
- A level can have **0–N picks**. With 0 picks, the calendar shows "No fit this month" for that level and the program builder skips that month.

**Seasonality cheat-sheet (rule of thumb when adding picks):**

| | West coast (Karnataka, Goa, Kerala SW pulses) | East coast (Tamil Nadu, Pondicherry) | Sri Lanka |
| --- | --- | --- | --- |
| **Avoid** | Jun–Sep (monsoon — power, no visibility, dawn-only) | Oct–Dec (NE monsoon rain) — though chunky NE swells can be epic for advanced | — |
| **Beginner-friendly** | Sep–Feb (post-monsoon glass to mid-Kerala season) | Mar–Apr (shoulder), Jan–Feb tail of NE | Nov–Mar (Weligama, south coast); Jul–Aug (Arugam Baby Point only on small days) |
| **Advanced sweet spot** | Apr–May (pre-monsoon Gokarna reefs) | Nov–Dec (peak NE Covelong) | Jul–Aug (Arugam Main Point at peak) |

These are starting-point guidelines — chunky monsoon-swell windows can be great for advanced surfers, which is why some monsoon months still list `Mulki` for advanced. Adjust based on what the crew actually scored.

### `logbook.entries` — past + upcoming trips

This is the "Where we've been / Where we're going" archive grid. Add a new entry at the **top** of `entries` for each new trip.

```js
{
  month: "May 2026",                 // displayed as the small caption
  location: "Pondicherry, Tamil Nadu",
  status: "logged",                  // "logged" = past trip, "upcoming" = next trip
  headline: "First proper south-swell session of the year.",
  notes: "Two days, dawn patrols…",  // not currently shown — kept for future use
  instagram: "https://www.instagram.com/.../p/xxxx/",  // empty string = not a link
  photo: "images/mulki-apr26.jpg",   // path relative to index.html
  photo2x: "",                       // optional retina image
  spotlight: null
}
```

**Notes:**
- `status: "upcoming"` styles the tile differently and shows an "Up next" label.
- If `instagram` is non-empty, the whole tile becomes a link to that post and shows a small IG corner badge.
- If `photo` is empty, the tile shows a `?` placeholder — fine for upcoming trips with no photo yet.
- Drop new photos in `images/` and reference them as `images/<filename>.jpg`.

#### Updating "Next up" text on the join bar

The text shown next to the join CTA is `joinBar.text`:

```js
joinBar: {
  text: "Next trip is forming. Want in?",
  cta: "Talk to us →"
}
```

There's also `club.nextTrip` (e.g. `"Next up: Pondicherry — May 2026"`) — currently unused on the page but kept as a backup.

### `ethos` — the four manifesto cards

Numbered `01`–`04`. Edit `title` and `body` freely. Add or remove cards by adding/removing array entries.

### `swellMap` — section headline above the map

```js
swellMap: {
  overline: "The swell",
  heading: "Where the waves are, when.",
  sub: "Ratings come from surf-forecast.com…"
}
```

### `joinForm` — the modal form

`fields.roleOptions` is the dropdown shown to people filling out "Where are you at?". The first option (with empty `value`) is the placeholder.

---

## Common edits, recipes

### "I want to change the next trip"
1. Open `config.js`.
2. In `logbook.entries`, find the topmost `status: "upcoming"` entry. Update `month`, `location`, `headline`, `notes`.
3. Optionally update `joinBar.text` if the bar should mention the trip.
4. Save and refresh.

### "We just got back from a trip — log it"
1. In `logbook.entries`, find that trip's entry. Change `status` from `"upcoming"` to `"logged"`.
2. Add the Instagram link (`instagram`) and photo path (`photo`).
3. Save.

### "Add a new destination"
1. Add an entry to `destinations` (see field rules above).
2. Add it to at least one month in `calendar` under the right skill level(s) — otherwise it only shows on the map, not the calendar/program.

### "Change which spot is suggested for January beginners"
Open `calendar`, find `month: "Jan"`, edit `picks.beginner`. The first array entry is the primary; the rest are alternates.

### "Quickly try out picks without losing the originals"
Comment out the line you don't want with `//` at the start of the `{ dest: …, note: … }` line. JS ignores it. Uncomment to bring it back.

---

## Conventions

- **En-dash `–`** in date ranges (`"Oct–Mar"`), not hyphen `-`.
- **Destination names** are the join key — keep them consistent across `destinations` and `calendar`.
- **Skill levels** are exactly: `beginner`, `intermediate`, `advanced` (lowercase).
- The program builder uses `?skill=`, `?start=` (1-12), and `?cadence=` (`1`, `2`, `3`, or `best`) URL params — share links you generate from the page will round-trip cleanly.

## Sanity-checking after edits

1. Open `index.html` in a browser. Toggle the calendar level filter — every month should render a card (or "No fit this month" for any level you intentionally left empty).
2. Open `program.html`, change level + start month + cadence — trip cards should refresh, share button should copy a URL with all three params.
3. Hover the swell-map pins — each should match a `destinations` entry.
4. If anything renders blank, open the browser console (F12) — typos in `config.js` show up as syntax errors here.
