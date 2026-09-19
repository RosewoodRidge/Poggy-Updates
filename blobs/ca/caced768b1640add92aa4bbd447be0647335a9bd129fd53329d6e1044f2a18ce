# Poggy Trash Bins

Searchable trash bins for RedM.

Players search bins in town for random loot. Each bin also has a small shared storage that anyone can open, good for dead drops and stash spots. Works on every framework poggy_core supports.

## Features

- **Search and loot.** A timed search with an animation and a progress bar. Loot comes from one shared table with a chance per item.
- **Shared storage.** Every bin has its own container (a custom inventory on VORP, a stash on RSG) that any player can open.
- **Auto-detection.** Bins already standing in the world can become searchable on their own. No coordinates needed.
- **33 ready-made bins** across Valentine, Annesburg, Strawberry, Blackwater, Saint Denis, Tumbleweed, Armadillo and more.
- **Anti-exploit.** No searching while another player stands too close. The server checks distance and cooldown too.
- **Cooldowns.** Each bin has a random cooldown between searches, so nobody can farm one.
- **Staff wipe command.** Staff groups can empty every bin at once.
- **Discord logging.** Optional log of what each player found.
- **Built-in progress bar.** No other resource needed.
- **Map blips.** Optional blips for your listed bins.

## Requirements

| Resource | Needed |
|---|---|
| `poggy_core` | Yes |
| A framework poggy_core supports | Yes. VORP (vorp_core + vorp_inventory) is verified; RSG (rsg-core + rsg-inventory) is supported. |

The script has no database tables of its own. Every storage action goes through poggy_core.

## Installation

1. Put the `poggy_trashbins` folder in your resources folder.
2. Add `ensure poggy_core`, then `ensure poggy_trashbins`, to `server.cfg`.
3. Check that the items in the loot list exist on your server.
4. Restart the server.

## How it plays

1. Walk up to a bin. Two prompts appear: **Search the Trash Bin** and **Take a look inside the Trash Bin**.
2. **Search**: the character rummages for 5 to 10 seconds, then finds up to 2 items (or nothing).
3. The bin is then empty for 30 minutes to 2 hours.
4. **Look inside**: opens the bin's shared storage.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/emptytrash` | Staff groups, console | Empties every bin's storage and resets their search cooldowns. |

- The command name is set by `Config.WipeCommand`.
- Staff groups are listed in `Config.StaffGroups` (`superadmin`, `admin`, `moderator` by default).
- Turn the command off with `Config.AllowStaffGroupsToWipeWithCommand = false`.

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy Hub). You can also edit `config.lua` and `translations.lua` by hand. Restart the script after a change.

| Setting | What it controls |
|---|---|
| `Config.CommonLoot` | The shared loot list: item, chance (percent), fewest and most. |
| `Config.TrashBins` | Your listed bins: position, whether to spawn a bin prop, storage id and slots. |
| `Config.AutoDetect`, `Config.AutoDetectModels` | Finding bins that already stand in the world. |
| `Config.RefillTimeMin` / `Config.RefillTimeMax` | The cooldown after a search, in seconds. |
| `Config.SearchTimeMin` / `Config.SearchTimeMax` | How long a search takes, in milliseconds. |
| `Config.MaxItemsPerSearch` | The most different items one search can find. |
| `Config.InteractionRadius`, `Config.BlockTrashIfPlayerIsNearRange` | Prompt distance, and how close another player may be. |
| `Config.WipeStorageOnScriptStart` | Empty every bin at each restart. |
| `Config.AllowWeaponsInStorages` | Whether weapons may go in bins. |
| `Config.EnableBlips` | Map blips for listed bins. |
| `Config.EnableLogs`, `Config.LogsWebhook` | Discord logging. |
| `translations.lua` | Every player-facing message. |

### Giving one bin its own loot

Every bin uses `Config.CommonLoot`. To give one bin a different table, add a `loot = { ... }` list to that bin, in the same shape as `Config.CommonLoot`.

### Storage ids

Each bin's storage is saved under its `storageid`. Give every bin a different one. Changing an id loses whatever is stored in that bin.

## Escrow

| File | Status |
|---|---|
| `config.lua` | **Open**, fully editable |
| `translations.lua` | **Open**, every player-facing message |
| `client/main.lua` | Escrowed |
| `server/main.lua` | Escrowed |
| `ui/progressbar.html` | Escrowed |

## Troubleshooting

**Searching never finds anything.**
Check that the loot items exist in your items table, and that `Config.MaxItemsPerSearch` is above 0. A full inventory also means nothing is given.

**"Someone is standing too close to you!"**
Another player is within `Config.BlockTrashIfPlayerIsNearRange` metres of the bin.

**"It looks like someone has already messed around here."**
The bin is on cooldown. Wait, or lower `Config.RefillTimeMin` / `Config.RefillTimeMax`.

**Things left in bins disappear after a restart.**
That is `Config.WipeStorageOnScriptStart = true`. Set it to `false` to keep bin contents across restarts.

**The wipe command does nothing.**
Check your character's group is in `Config.StaffGroups`, and that `Config.AllowStaffGroupsToWipeWithCommand` is on.

## Changelog

- **1.5.2** — Poggy Hub support: settings, lists and help pages for `/poggy`. Bins no longer need `loot = Config.CommonLoot`: a bin without its own `loot` uses the shared list. Old configs that still have the line work as before.
- **1.5.1** — The startup wipe and the staff wipe command no longer query the framework's inventory table directly (`character_inventories` does not exist on RSG, so the script errored at start). A wipe now registers the bin, destroys its contents with poggy_core's `storage.delete`, registers it again and confirms it is empty through `storage.items`, sweeping any leftovers with `storage.removeItem`. `oxmysql` is no longer a direct dependency (poggy_core still needs it). Behaviour on VORP is unchanged.
- **1.5.0** — Layout to the owner's standard; every framework call through poggy_core.

## Support

- Discord: [https://discord.com/invite/rBarFeuzFj](https://discord.com/invite/rBarFeuzFj)
- Store: [https://rosewoodridge.tebex.io/](https://rosewoodridge.tebex.io/)
