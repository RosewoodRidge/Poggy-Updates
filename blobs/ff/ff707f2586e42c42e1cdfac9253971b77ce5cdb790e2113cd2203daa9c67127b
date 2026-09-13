# Poggy Fishing Journal

A fishing journal item for Poggy Fishing. Players open it from their inventory to see a map of every water body, which fish live there, which fish are in season right now, and their own records: fish discovered, catch counts and best weights.

Catches are recorded only while the player carries the journal.

## Requirements

| Resource | Purpose |
|----------|---------|
| `poggy_fishing` | Sends each catch to the journal; the journal reads its water zones |
| `poggy_core` (0.13.0 or newer) | Framework layer: character, inventory, usable item |
| `oxmysql` | Database access |

## Installation

1. Place `poggy_fishing_journal` in your resources folder.
2. Add `ensure poggy_fishing_journal` to your server.cfg, **after** `ensure poggy_fishing`.
3. Copy the item icon `docs/item_images/fishing_journal_fn.png` into your inventory's item image folder (on VORP: `vorp_inventory/html/img/items/`).
4. Give players the journal however you like (a shop, a starter kit, an admin give).

## Database

The `fishing_journal_fn` item and the `poggy_fishing_journal` table are added to the database automatically when the script starts (`sql/install.sql`); an item you already have is never changed. To import the file yourself instead, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

On a framework without an `items` table (RSG), poggy_core skips that row and the `fishing_journal_fn` item must be added to the framework's item list by hand (RSG: `rsg-core/shared/items.lua`); the script prints one yellow line at start while it is missing.

## Configuration at a glance

`config.lua`

| Key | What it does |
|-----|--------------|
| `JournalConfig.JournalItem` | Item name that opens the journal (default `fishing_journal_fn`) |
| `JournalConfig.OpenAnim` | Scenario played while opening, and the delay before the UI shows |
| `JournalConfig.Map` | Map image and the game-world bounds it covers |
| `JournalConfig.Fish` | Fish shown in the journal: label, size, price, weight range, active hours. Keep it in step with poggy_fishing's `Config.Fish` |
| `JournalConfig.SizeLabels` | Sidebar group names per size |
| `JournalConfig.TypeColours` | Map pin colour per water type (lake, river, swamp, creek, pond, ocean) |

`shared/journal.lua`

| Key | What it does |
|-----|--------------|
| `JournalConfig.WaterBodies` | One map pin per water zone. `zoneHash` must match a zone in poggy_fishing |
| `JournalConfig.WaterBodyLore` | Description text per water body |
| `JournalConfig.SpeciesLore` | Description text per species |

## Changelog

- **1.2.1** — Runs on frameworks without an `items` table (RSG): the journal item row in `sql/install.sql` is skipped there instead of stopping the install, and the script prints one yellow line at start while the item is missing from your framework's item list.
