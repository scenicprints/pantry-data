# pantry-data

Single source of truth for the kitchen. **`pantry.json`** is written by the
[Pantry app](https://github.com/scenicprints/pantry) and read by the AI chef.

Raw URL for the chef:
`https://raw.githubusercontent.com/scenicprints/pantry-data/main/pantry.json`

## Schema

```json
{
  "pantry": [
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
  ],
  "quick_add_items": [
    { "name": "ground turkey", "barcode": "0123456789012", "last_price": 5.99,
      "macros_per_100g": { "protein_g": 27, "calories": 170, "carbs_g": 0, "fat_g": 9 } }
  ]
}
```

`price_per_gram` and `expiring_soon` are derived by the app and written out so
the chef doesn't recompute them. `updated_at_ms` is sync bookkeeping — ignore it.
