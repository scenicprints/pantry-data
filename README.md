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
