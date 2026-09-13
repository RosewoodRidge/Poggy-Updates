# Poggy Storage

**What's yours stays yours.**

Player-owned storage for RedM. Your players create a container, keep their gear
in it, and decide who else gets a key. It persists through restarts, it can be
shared with named friends or with a whole job, and it costs whatever you say it
costs.

**Free.** Framework calls go through `poggy_core`, the shared Poggy framework
layer. There is no setting to change.

---

## Features

- Player-owned storage containers that persist through server restarts
- Configurable capacity, with paid upgrades
- Access management — add named players at four permission levels
- Job and job-grade access rules, so a whole role can share a stash
- Money balance and a full ledger per storage
- Map blips, optionally only for storages the player can actually open
- Preset storages for job locations (police, doctor, and so on)
- Admin commands for moving, deleting and auditing storages
- Multi-language (English, Spanish and German)

---

## Requirements

| | |
|---|---|
| **poggy_core** 0.12.0 or newer | Character data, money, containers and notifications. Free. |
| **[oxmysql](https://github.com/overextended/oxmysql)** | Database access. |
| **VORP Core** with `vorp_inventory` | The framework poggy_core drives, and the inventory the stashes live in. |

`vorp_menu` and `vorp_inputs` are **not required**. The menu and the text
prompts are part of this resource.

---

## How storage is stored

The `character_storage` table holds each storage's **metadata** — who owns it,
where it is, who may open it, its capacity, its balance and its ledger.

The **items** live in `vorp_inventory` as a custom inventory keyed
`character_storage_<id>`. That means item handling, weights and the grid window
are whatever your players already know. poggy_core registers the containers
under those same ids, so existing stashes keep their contents.

---

## Install

1. Drop `poggy_character_storage` into your resources folder.
2. `ensure poggy_character_storage` **after** `poggy_core` and `oxmysql`.
3. Give your admins the permission (below).
4. Start the server and read the console.

There is no SQL to import: poggy_core creates and upgrades the `character_storage`
table automatically every time the script starts. To manage the database yourself
instead, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`
and import `sql/install.sql`.

A healthy start prints one line:

```
✅ [Poggy] poggy_character_storage  v1.3.0  ready · poggy_core v0.12.0 · <framework>
```

### Upgrading from `character_storage`

The resource folder was renamed from `character_storage` to `poggy_character_storage`.
Remove the old `character_storage` folder, drop in the new one, and change
`ensure character_storage` to `ensure poggy_character_storage` in `server.cfg`.

Stored data is unaffected: the database table is still `character_storage`,
the containers keep their `character_storage_<id>` ids, and the admin ACE is
still `character_storage.admin`, so every storage, item, balance and
permission carries over. If another resource of yours called this one by
name (`exports.character_storage`, or its `character_storage:` events), update
it to `poggy_character_storage`.

---

## Admin access

**ACE.** In `server.cfg`:

```
add_ace group.admin character_storage.admin allow
```

**Character groups.** `Config.AdminGroups` is checked after ACE, against the
group poggy_core reports for the character, so existing `admin` / `superadmin` /
`god` groups keep working with no setup.

---

## Usage

- `/createstorage` — create a storage where you're standing
- Walk up to a storage and press the prompt key to open it

Owners can rename, upgrade capacity, manage who has access, deposit and
withdraw money, and read the ledger.

### Access levels

| Level | Can do |
|---|---|
| `owner` | Everything, including rename, delete and access management |
| `manager` | Open, deposit, withdraw, upgrade, ledger |
| `member` | Open, deposit, ledger |
| `basic` | Open the storage only |

---

## Admin commands

- `/movestorage [id] [x] [y] [z]` — move a storage
- `/deletestorage [id]` — delete a storage
- `/storageadmin show` — show every storage blip
- `/storageadmin hide` — back to only the ones you can open

---

## The in-game shop

The optional armory shop and `/adminshop` are drawn by `vorp_inventory`'s store
window, which poggy_core does not abstract. Without VORP the shop declines with
a message rather than failing silently. Storage itself is unaffected.

---

## Configuration

`config.lua` covers creation and upgrade prices, default and maximum capacity,
blips and prompts, language, admin access, the preset job storages, the armory
shops and the optional Discord tracking.

```lua
Config.UseBlips = true                -- storage blips at all
Config.OnlyShowAccessibleBlips = true -- only ones the player can open
```

Discord tracking stays off for any storage or armory whose `webhook` is empty.

---

## Version history

See [CHANGELOG.md](CHANGELOG.md) for full change logs.

---

## License

[MIT](LICENSE) - free to use, modify and redistribute.

Support: Poggy Scripts Discord — https://discord.com/invite/rBarFeuzFj
