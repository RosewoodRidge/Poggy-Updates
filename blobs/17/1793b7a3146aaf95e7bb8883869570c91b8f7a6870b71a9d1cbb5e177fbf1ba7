# Poggy Trash Bins

A fully-featured trash bin interaction system for RedM. Players can search bins for randomised loot and use them as shared world storage — all managed through a single, easy-to-edit config file. Works on every framework poggy_core supports.

## Features

- **Search & Loot** — Players search trash bins with a timed animation and progress bar. Loot is rolled from a shared, configurable loot table with per-item drop chances.
- **Persistent Storage** — Each bin has its own shared container (a custom inventory on VORP, a stash on RSG) that any player can access, enabling emergent gameplay like dead-drops and stash spots.
- **Auto-Detection** — Optionally scans the world for existing trash bin props and registers them as interactable bins automatically — no manual coordinate entry needed.
- **33 Pre-configured Locations** — Ships with bins placed across Valentine, Annesburg, Strawberry, Blackwater, Saint Denis, Tumbleweed, Armadillo, and more.
- **Anti-Exploit** — Proximity check prevents interaction while another player is too close. Server-side distance and cooldown validation.
- **Cooldown System** — Each bin has a randomised cooldown between searches to prevent farming.
- **Staff Wipe Command** — Authorised staff groups can wipe all bin inventories live with a single command.
- **Discord Logging** — Optional webhook logging of player search results.
- **Built-in Progress Bar** — Styled NUI progress bar with no external dependency required, or bring your own.
- **Map Blips** — Optional blip markers for all bin locations.
- **Fully Configurable** — Loot tables, cooldowns, interaction keys, search duration, staff groups, and more — all in one config file.

## Dependencies

| Dependency | Required |
|---|---|
| **poggy_core** | Yes |
| A framework poggy_core supports ([vorp_core](https://github.com/VORPCORE/vorp-core-lua) + [vorp_inventory](https://github.com/VORPCORE/vorp_inventory-lua) verified; rsg-core + rsg-inventory supported) | Yes |

The script has no database tables of its own and never queries the framework's inventory tables; every container operation goes through poggy_core.

## Installation

1. Place the `poggy_trashbins` folder into your server's `resources` directory.
2. Add `ensure poggy_core` and then `ensure poggy_trashbins` to your `server.cfg`.
3. Edit `config.lua` to customise loot tables, bin locations, keys, cooldowns, and other settings.
4. Restart your server.

## Escrow

| File | Status |
|---|---|
| `config.lua` | **Open** — fully editable |
| `translations.lua` | **Open** — every player-facing string |
| `client/main.lua` | Escrowed |
| `server/main.lua` | Escrowed |
| `ui/progressbar.html` | Escrowed |

## Configuration Overview

All settings are in `config.lua`:

- **Loot Table** (`Config.CommonLoot`) — Add, remove, or adjust items and drop chances in one place.
- **Bin Locations** (`Config.TrashBins`) — Add new bins by appending an entry with coordinates and a unique `storageid`.
- **Auto-Detection** (`Config.AutoDetect`) — Automatically finds world trash bin props and registers them.
- **Cooldowns** (`Config.RefillTimeMin` / `Config.RefillTimeMax`) — Control how long before a bin can be searched again.
- **Search Duration** (`Config.SearchTimeMin` / `Config.SearchTimeMax`) — How long the search animation plays.
- **Staff Command** (`Config.WipeCommand`) — In-game command for authorised staff to wipe all bin inventories.
- **Translations** (`translations.lua`) — All player-facing text is configurable for localisation.

## Changelog

- **1.5.1** — The startup wipe and the staff wipe command no longer query the framework's inventory table directly (`character_inventories` does not exist on RSG, so the script errored at start). A wipe now registers the bin, destroys its contents with poggy_core's `storage.delete`, registers it again and confirms it is empty through `storage.items`, sweeping any leftovers with `storage.removeItem`. `oxmysql` is no longer a direct dependency (poggy_core still needs it). Behaviour on VORP is unchanged.
- **1.5.0** — Layout to the owner's standard; every framework call through poggy_core.

## Support

- Discord: [https://discord.com/invite/rBarFeuzFj](https://discord.com/invite/rBarFeuzFj)
- Store: [https://rosewoodridge.tebex.io/](https://rosewoodridge.tebex.io/)
