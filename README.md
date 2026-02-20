# CS2 Skin Mapping Generator

![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue?logo=python&logoColor=white)
![License: MIT](https://img.shields.io/badge/license-MIT-green)
![Data: SteamDatabase](https://img.shields.io/badge/data-SteamDatabase-orange)
![Zero config](https://img.shields.io/badge/setup-zero_config-brightgreen)

> The definitive, always-fresh CS2 skin → market hash name mapping.
> No API keys. No manual updates. No stale data. Just run it.

Generates a **complete CS2 weapon and knife skin database** directly from Valve's game files — sourced live from [SteamDatabase/GameTracking-CS2](https://github.com/SteamDatabase/GameTracking-CS2). One command gives you JSON and Excel outputs ready to drop into any CS2 trading bot, price tracker, or research project.

---

## Quick Start

```bash
python generate_skin_mappings.py
```

No setup required. Game data is auto-downloaded and cached on first run.

---

## Features

- **All weapon skins** — every paintable weapon from AK-47 to R8 Revolver
- **All knife skins** — every knife defindex × every knife paint kit, fully enumerated
- **Steam Market hash names** — plug directly into Steam API / Steam Market calls
- **Wear range data** — `wear_min` / `wear_max` float values per skin
- **Smart caching** — fetches from SteamDatabase once, reuses locally until you `--force`
- **Auto-encoding detection** — handles Valve's UTF-16 LE and UTF-8 VDF files transparently
- **Auto-installs `openpyxl`** — zero manual dependency setup
- **Built-in verification** — spot-checks AWP | Dragon Lore, ★ Karambit | Marble Fade, and more

---

## Output Files

| File | Format | Contents |
|------|--------|----------|
| `data/skin-market-mapping.json` | JSON array | `defindex`, `paint_index`, `market_hash_name`, weapon/skin names, wear range |
| `data/skin-market-mapping.xlsx` | Excel (.xlsx) | Same data in a styled, column-widened spreadsheet |

---

## Sample Output

```json
[
  {
    "defindex": 7,
    "paint_index": 801,
    "market_hash_name": "AK-47 | Asiimov",
    "weapon": "AK-47",
    "skin": "Asiimov",
    "paint_kit_name": "cu_ak47_asiimov",
    "wear_min": 0.18,
    "wear_max": 1.0
  },
  {
    "defindex": 9,
    "paint_index": 344,
    "market_hash_name": "AWP | Dragon Lore",
    "weapon": "AWP",
    "skin": "Dragon Lore",
    "paint_kit_name": "cu_awp_dragon_lore",
    "wear_min": 0.01,
    "wear_max": 0.7
  },
  {
    "defindex": 507,
    "paint_index": 413,
    "market_hash_name": "★ Karambit | Marble Fade",
    "weapon": "Karambit",
    "skin": "Marble Fade",
    "paint_kit_name": "aa_fade",
    "wear_min": 0.0,
    "wear_max": 0.08
  }
]
```

---

## Usage in Your Project

**Python**

```python
import json

with open("data/skin-market-mapping.json") as f:
    skins = json.load(f)

# Build a fast lookup by (defindex, paint_index)
index = {(s["defindex"], s["paint_index"]): s for s in skins}

skin = index[(7, 801)]
print(skin["market_hash_name"])  # "AK-47 | Asiimov"
print(skin["wear_min"], skin["wear_max"])  # 0.18  1.0
```

**Node.js / TypeScript**

```ts
import skins from './data/skin-market-mapping.json'

const index = new Map(skins.map(s => [`${s.defindex}:${s.paint_index}`, s]))

const skin = index.get('7:801')
console.log(skin?.market_hash_name)  // "AK-47 | Asiimov"
```

---

## CLI Flags

| Flag | Effect |
|------|--------|
| *(none)* | Use cached game files, regenerate all output |
| `--force` | Re-download fresh game files from SteamDatabase |

Run `--force` after a CS2 update to pick up new skins.

---

## How It Works

```
SteamDatabase GitHub
       │
       ▼
items_game.txt + csgo_english.txt   (cached in .cache/)
       │
       ▼
 VDF Parser (single-pass)
 ├─ paint_kits     → paint_index, name, wear_min, wear_max
 ├─ items          → defindex, weapon class
 └─ loot_lists     → weapon_class ↔ paint_kit pairs
       │
       ▼
 Name resolver (csgo_english.txt locale)
       │
       ▼
 Skin builder
 ├─ Weapon skins   → loot pairs × defindexes
 └─ Knife skins    → all knife defindexes × all knife paint kits
       │
       ▼
 Output: JSON + Excel   (data/)
       │
       ▼
 Verification checks
```

1. **Fetch** — Downloads `items_game.txt` and `csgo_english.txt` from SteamDatabase (or uses cache)
2. **Parse** — Single-pass VDF parser extracts paint kits, item definitions, and weapon↔paint loot pairs
3. **Resolve** — Localises display names via `csgo_english.txt` locale keys
4. **Build** — Constructs all weapon skin combinations, then crosses every knife defindex against every knife paint kit
5. **Write** — Outputs styled JSON and Excel
6. **Verify** — Spot-checks known skins to catch regressions after CS2 updates

---

## Requirements

- Python **3.9+**
- Internet connection for the initial fetch (or `--force`)
- `openpyxl` — **auto-installed** if not present

---

## Data Source

Game data is sourced from [SteamDatabase/GameTracking-CS2](https://github.com/SteamDatabase/GameTracking-CS2), which tracks CS2 game files in real time. This means the generator is always compatible with the current live game — no waiting for a maintainer to update hardcoded tables.

---

## Use Cases

- **CS2 trade bots** — resolve `(defindex, paint_index)` pairs from Steam inventory to market hash names
- **Price trackers** — build a skin price DB keyed on stable `defindex + paint_index` identifiers
- **Inventory managers** — display correct weapon/skin display names in any language
- **Data analysis** — full skin universe in Excel for market research or float range studies
- **Game tools / overlays** — TypeScript constants ready to ship in web apps

---

## Contributing

Pull requests are welcome. If a skin or knife is missing:

1. Run with `--force` to rule out a stale cache
2. Open an issue with the `defindex`, `paint_index`, and expected `market_hash_name`

---

## License

[MIT](LICENSE)
