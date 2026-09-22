# pantry-data

Single source of truth for the kitchen. **`pantry.json`** is written by the
[Pantry app](https://github.com/scenicprints/pantry) (and updated by BodyComp's
Cook screen when a meal is subtracted) and read by the AI chef.

Raw URL for the chef:
`https://raw.githubusercontent.com/scenicprints/pantry-data/main/pantry.json`

## Schema

Each item has two independent parts:

- **Stock tracking** — either **by weight (grams)** or **by count** (e.g. eggs).
- **Nutrition** — entered **per serving** (`serving_size` + `serving_unit` +
  `macros_per_serving`). When the serving unit is grams, a derived
  `macros_per_100g` is also written for convenience.

### Weight item
```json
{
  "id": "unique_id",
  "name": "ground turkey",
  "barcode": "0123456789012",
  "total_weight_g": 454,
  "remaining_weight_g": 340,
  "price": 5.99,
  "price_per_gram": 0.0132,
  "serving_size": 112,
  "serving_unit": "g",
  "macros_per_serving": { "protein_g": 30, "calories": 190, "carbs_g": 0, "fat_g": 10 },
  "macros_per_100g":    { "protein_g": 27, "calories": 170, "carbs_g": 0, "fat_g": 9 },
  "expiration_date": "2026-07-10",
  "expiring_soon": false,
  "date_added": "2026-07-02",
  "last_price": 5.99,
  "updated_at_ms": 0
}
```

### Count item  (`unit: "count"`)
Stock is whole units (eggs, cans). Nutrition is still per serving — a serving
may be one piece or several.
```json
{
  "id": "unique_id",
  "name": "eggs",
  "unit": "count",
  "total_count": 12,
  "remaining_count": 9,
  "price": 3.49,
  "price_per_unit": 0.29,
  "serving_size": 1,
  "serving_unit": "piece",
  "macros_per_serving": { "protein_g": 6, "calories": 72, "carbs_g": 0, "fat_g": 5 },
  "expiration_date": "2026-07-20",
  "expiring_soon": false,
  "date_added": "2026-07-02",
  "last_price": 3.49,
  "updated_at_ms": 0
}
```

### Quick-add staples
```json
{ "name": "ground turkey", "barcode": "0123456789012", "last_price": 5.99,
  "serving_size": 112, "serving_unit": "g",
  "macros_per_serving": { "protein_g": 30, "calories": 190, "carbs_g": 0, "fat_g": 10 } }
```

### Untracked items — spices & "on hand"
Some items have no amount or cost, just a `category` and a flag. They ARE
available — treat them as in-stock, don't put them in new buys, and don't rely
on a specific weight.
```json
{ "id": "…", "name": "cumin", "category": "Spices", "spice": true,
  "expiring_soon": false, "date_added": "2026-07-06", "updated_at_ms": 0 }

{ "id": "…", "name": "leftover rice", "category": "Pantry",
  "quantity_unknown": true, "expiring_soon": false, "date_added": "…", "updated_at_ms": 0 }
```

## Notes for the chef
- **Macros are PER SERVING.** Use `serving_size` + `serving_unit` to scale to
  whatever amount is being cooked. For gram servings, `macros_per_100g` is also
  provided.
- **Check `unit`.** `"count"` means whole units — use `remaining_count`
  (e.g. "9 eggs"), not grams. Otherwise stock is grams.
- **`spice: true`** → own category ("Spices"), always on hand, no amount/cost.
  **`quantity_unknown: true`** → the user has it but the amount isn't tracked.
  Both are available; never list them as new buys.
- Every item has a `category` (currently `"Pantry"` or `"Spices"`).
- `serving_unit` may be a non-metric label (`cup`, `tbsp`, `cookie`, `scoop`).
- `price_per_gram` / `price_per_unit`, `expiring_soon`, and `macros_per_100g`
  are derived by the app so you don't recompute them.
- `updated_at_ms` is sync bookkeeping — ignore it.
- Prioritize items with `expiring_soon: true`.

---

# `host_brief.json` — a dinner planned in conversation

**`host_brief.json`** is the other direction: not what the kitchen holds, but
what the cook wants made. It is written by **Claude**, read by the Pantry app's
**Host Hub**, and cooked from by the app's own chef.

Planning a dinner is a conversation — this dish but with short rib instead of
mince, no cream in anything, the starter cold because the oven is busy. The
app's planner is one box and one shot, so a brief is where that back-and-forth
lands once it's settled.

**The chef still cooks.** A brief carries no recipes. The app writes the
recipes, the prep timeline and the dinner-day run sheet on the device, from
this brief plus the pantry and the prices it already knows.

```json
{
  "briefs": [
    {
      "createdAtMs": 1764201600000,
      "name": "Sarah's Birthday",
      "guests": 6,
      "eventDate": "2026-10-03",
      "guestNotes": "One guest has a tree-nut allergy.",
      "notes": "They sit down at 7. Keep the oven free after 6 — the main is in it.",
      "dishes": [
        {
          "text": "Lasagna",
          "course": "Main",
          "notes": "Short rib, not mince. No cream sauce — keep it red."
        },
        { "text": "Charred broccolini", "course": "Side" }
      ]
    }
  ]
}
```

## Fields

- **`createdAtMs`** — ms since epoch; the brief's id. Leave it out and the app
  assigns one when the brief is pasted rather than synced.
- **`name`** — what the dinner is called. Optional.
- **`guests`** — head count. Everything is scaled to it.
- **`eventDate`** — `YYYY-MM-DD`. Optional, but the prep timeline is counted
  back from it, so without one there is no schedule to build.
- **`dishes[]`** — at least one. Each has:
  - **`text`** — what to cook, in the words it was agreed in.
  - **`course`** — `Starter` | `Main` | `Side` | `Dessert` | `Drink` | `Other`.
  - **`notes`** — everything decided about *this* dish. This is the half the
    app had no way to accept: substitutions, what to leave out, how it should
    turn out. The chef is told these are decided, not suggested, and that
    ruling something out rules it out in every form — "no cream sauce" must not
    come back as a béchamel.
- **`guestNotes`** — restrictions for the people at this table, this once.
  Stacked on top of the user's permanent avoid list, never replacing it.
- **`notes`** — anything about the dinner as a whole: the occasion, what time
  people sit down, what the oven is already doing, how much should be done in
  advance.
- **`builtAtMs`** — written by the app once it has built the menu. A stamped
  brief stops showing, so the inbox doesn't fill with dinners that already
  happened. Don't set it yourself.

## Writing one

Append to `briefs` rather than replacing the file, and leave stamped briefs
where they are. The app pulls on open and on resume, so a brief written now
shows up the next time the Host Hub is opened.

An agent that can't reach this repo — Claude on a phone or in a browser — can
just hand the JSON to the user: Host Hub → **Paste a plan from Claude** takes
it, fence and all.
