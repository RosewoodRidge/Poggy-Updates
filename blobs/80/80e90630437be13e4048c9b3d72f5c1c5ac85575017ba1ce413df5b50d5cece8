# Poggy Fishing Journal

A fishing journal for Poggy Fishing. Players use the journal item to open an illustrated book with:

- a map of every water, with a pin for each,
- which fish live in each water, and which are in season now,
- the time of day each fish bites,
- lore for every species,
- their own records: fish discovered, catch counts and best weights.

A new discovery shows a toast. Beating a best weight shows a gold "Personal record" message.

Catches are recorded only while the player carries the journal.

## Requirements

| Resource | Why |
|---|---|
| `poggy_fishing` | Sends each catch to the journal, and owns the waters and seasons |
| `poggy_core` 0.13.0 or newer | Framework layer: character, inventory, usable item |
| `oxmysql` | Database |

## Installation

1. Put `poggy_fishing_journal` in your resources folder.
2. Add `ensure poggy_fishing_journal` to `server.cfg`, **after** `ensure poggy_fishing`.
3. Copy `docs/item_images/fishing_journal_fn.png` into your inventory's image folder.
   On VORP that is `vorp_inventory/html/img/items/`.
4. Give players the journal your own way: a shop, a starter kit or an admin give.

### Database

`sql/install.sql` runs by itself when the script starts. It adds the `fishing_journal_fn` item and the `poggy_fishing_journal` table.
An item you already have is never changed.
To import it yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

On RSG (no `items` table), add `fishing_journal_fn` to `rsg-core/shared/items.lua` by hand.
The script prints one yellow line at start while it is missing.

## Commands

None. Players open the journal by using the item.

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy Hub), then a restart of the script.
You can also edit `config.lua` by hand.

| Setting | What it does |
|---|---|
| `JournalConfig.JournalItem` | Item that opens the journal (default `fishing_journal_fn`) |
| `JournalConfig.OpenAnim` | Animation played while opening, and the delay before the book shows |
| `JournalConfig.Map` | Map picture and the world area it covers |
| `JournalConfig.Fish` | Fish shown in the journal: name, size, price shown, weight range, hours, species |
| `JournalConfig.SizeLabels` | Sidebar group names per size |
| `JournalConfig.TypeColours` | Map pin colour per water type |

Keep `JournalConfig.Fish` in step with Poggy Fishing's fish list. The hub guide *Keeping the journal in step with Poggy Fishing* explains what must match.

The water pins, water descriptions and species lore are built in (`shared/journal.lua`) and cannot be edited.

## Troubleshooting

**Catches are not recorded.**
The player must carry the journal item. The fish's id must also be in `JournalConfig.Fish`.

**The journal shows no seasons or no fish per water.**
`poggy_fishing` must be running. The journal reads both from it.

**A yellow line at start says the journal item is missing.**
Add `fishing_journal_fn` to your framework's item list (RSG: `rsg-core/shared/items.lua`).

## Changelog

- **1.2.2**: Poggy Hub support. Every setting has a label and help text in `/poggy`, with a guide for keeping the fish list in step. README rewritten.
- **1.2.1**: Runs on frameworks without an `items` table (RSG): the journal item row in `sql/install.sql` is skipped there instead of stopping the install, and the script prints one yellow line at start while the item is missing from your framework's item list.
