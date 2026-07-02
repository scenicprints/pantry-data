# pantry-data

Single source of truth for the kitchen. **`pantry.json`** is written by the
[Pantry app](https://github.com/scenicprints/pantry) (and updated by BodyComp's
Cook screen when a meal is subtracted) and read by the AI chef.

Raw URL for the chef:
`https://raw.githubusercontent.com/scenicprints/pantry-data/main/pantry.json`

## Schema

Every item is tracked either **by weight (grams)** or **by count** (e.g. eggs).

### Weight item
```json
{
  "id": "unique_id",
  "name": "ground turkey",
  "barcode": "0123456789012",
  "total_weight_g": 454,
  "remaining_weight_g": 454,
  "price": 5.99,
  "price_per_gram": 0.0132,
  "macros_per_100g": { "protein_g": 27, "calories": 170, "carbs_g": 0, "fat_g": 9 },
  "expiration_date": "2026-07-10",
  "expiring_soon": false,
  "date_added": "2026-07-02",
  "last_price": 5.99,
  "updated_at_ms": 0
}
```

### Count item  (`unit: "count"`)
Used for things that aren't weighed (eggs, cans). **Amounts are whole units,
not grams.** `macros_per_unit` is optional and often absent.
```json
{
  "id": "unique_id",
  "name": "eggs",
  "unit": "count",
  "total_count": 12,
  "remaining_count": 9,
  "price": 3.49,
  "price_per_unit": 0.29,
  "macros_per_unit": { "protein_g": 6, "calories": 72, "carbs_g": 0, "fat_g": 5 },
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
  "macros_per_100g": { "protein_g": 27, "calories": 170, "carbs_g": 0, "fat_g": 9 } }
```

## Notes for the chef
- **Check `unit`.** If it is `"count"`, the item is measured in whole units —
  use `remaining_count` (e.g. "9 eggs"), NOT grams. Otherwise it's grams.
- `price_per_gram` / `price_per_unit` and `expiring_soon` are derived by the
  app and written out so you don't recompute them.
- `updated_at_ms` is sync bookkeeping — ignore it.
- Prioritize items with `expiring_soon: true`.
