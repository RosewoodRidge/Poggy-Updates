# Poggy Storage

**What's yours stays yours.**

Player-owned storage for RedM. Players buy a storage where they stand, keep
their gear in it, and decide who else gets a key. It survives restarts. It can
be shared with named friends or a whole job. It costs whatever you say.

**Free.** Every framework call goes through `poggy_core`, so there is no
framework setting to change.

---

## Features

- Player-owned storages that survive server restarts.
- Paid capacity upgrades that get dearer each time.
- Access for named players at three levels: Manager, Member and Basic.
- Access for a whole job, by grade.
- A money balance and a full ledger in every storage.
- Map blips for the storages a player can open.
- Job storages set in the config (police evidence, doctor cabinets and so on).
- Job-locked armories with endless stock.
- Optional Discord logging for job storages and armories.
- A **Storages** panel in `/poggy`: every storage in one table, to rename,
  resize, move, give to another character or delete, manage who has a key, and
  see, add or remove what is inside. Admin commands too.
- English, Spanish and German.
- Every setting can be changed in game with **`/poggy`**.

---

## Requirements

| Needs | Why |
|---|---|
| **poggy_core** 0.14.0 or newer | Characters, money, containers, notifications, menus and text prompts. Free. |
| **[oxmysql](https://github.com/overextended/oxmysql)** | The database. |
| **A framework poggy_core supports** | VORP, RSG or QBR. The stashes live in its inventory. |

`vorp_menu`, `vorp_inputs` and `vorp_inventory` are **not** needed. Every menu,
prompt, container and shop is drawn by poggy_core.

---

## Install

1. Put `poggy_character_storage` in your resources folder.
2. Add `ensure poggy_character_storage` to `server.cfg`, **after** `poggy_core`
   and `oxmysql`.
3. Give your admins the permission (see [Admins](#admins)).
4. Start the server and read the console.

There is no SQL to import. poggy_core creates and updates the
`character_storage` table every time the script starts. To manage the database
yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in
`poggy_core/config.lua` and import `sql/install.sql`.

A healthy start prints:

```
✅ [Poggy] poggy_character_storage  v1.5.1  ready · poggy_core v0.14.0 · <framework>
```

### Upgrading from `character_storage`

The folder was renamed from `character_storage` to `poggy_character_storage`.

1. Remove the old `character_storage` folder and put in the new one.
2. Change `ensure character_storage` to `ensure poggy_character_storage` in
   `server.cfg`.

Your data carries over. The table is still `character_storage`, the containers
keep their `character_storage_<id>` ids, and the admin ACE is still
`character_storage.admin`. If another resource called this one by name
(`exports.character_storage` or `character_storage:` events), point it at
`poggy_character_storage`.

---

## Managing storages: the /poggy panel

The main admin tool is the **Storages** tab in `/poggy` (poggy_core's settings
hub). It lists every storage, player-bought and job storages alike, with its
owner, place, size, upgrades and who has a key. Changes apply at once.

- **Rename** or **resize** a player storage by editing the cell.
- **Move to my position**, **Change owner** and **Delete storage** on each row.
  Moving and deleting do exactly what `/movestorage` and `/deletestorage` do.
- Open a row to see **who has access**: give a character a key at a level,
  change the level, or take it away.
- **Contents** on any row, job storages too, shows what is inside and lets you
  add, remove or empty it.

Job storages are read-only here apart from Contents: they are set in
`config.lua`. Deleting a storage unregisters its container but does not empty
it (see [Commands](#commands)). Only people who may edit settings in `/poggy`
can use the panel, and every change is printed to the console with who made it.
The in-game Help page **Managing storages in /poggy** walks through it.

---

## Commands

Command names can be changed in the config. The table shows the defaults.

| Command | Who | What it does |
|---|---|---|
| `/createstorage` | Everyone | Buy a storage where you stand. |
| `/movestorage <id> <x> <y> <z>` | Admins | Move a player storage. |
| `/deletestorage <id>` | Admins | Delete a player storage. Its container is unregistered, not emptied: the items stay in the inventory database, out of reach. |
| `/storageadmin show` | Admins | Show every storage blip on the map. |
| `/storageadmin hide` | Admins | Back to only the ones you can open. |
| `/adminshop` | Admins | A free shop with every item on the server. |
| `/adminshop cache` | Admins | Rebuild that shop's item list. |

To open a storage or an armory, walk up to it and press **G**.

---

## Using a storage

The owner can rename it, upgrade it, manage who has access, deposit and
withdraw money, and read the ledger.

### Access levels

| Level | Can do |
|---|---|
| **Owner** | Everything, including rename and managing access. |
| **Manager** | Open, deposit, withdraw, ledger, upgrade. |
| **Member** | Open, deposit, ledger. |
| **Basic** | Open the storage only. |

The owner gives named players a level. For a whole job, the player's grade
decides the level: grades in `Config.ManagerJobGrades` get Manager, grades in
`Config.MemberJobGrades` get Member, and every other grade gets Basic.

### How storage is stored

The `character_storage` table holds each storage's details: owner, position,
who may open it, capacity, balance and ledger.

The items live in your framework's inventory, in a container called
`character_storage_<id>` (a `vorp_inventory` custom inventory on VORP). So item
handling, weights and the inventory window are the ones your players know.

---

## Configuration

Everything is in `config.lua`, and **every setting can be changed in game with
`/poggy`**. The main areas:

| Area | Settings |
|---|---|
| **Prices** | Creation price, first upgrade price, price increase, slots per upgrade. |
| **Player storages** | Starting capacity, storages per character, open distance, expiry, blips, job-grade access levels. |
| **Job storages** | `Config.DefaultStorages`: locations, capacity, jobs, blip, Discord. |
| **Clerks** | `Config.NPCs`: decorative NPCs at storage desks. |
| **Armories** | `Config.ArmoryShops` and the item list `Config.WeaponArmoryItems`. |
| **Admins** | `Config.AdminAce`, `Config.AdminGroups`, command names. |
| **Language** | `Config.DefaultLanguage` and `Config.Translations`. |

Discord logging stays off for any storage or armory until its `webhook` is a
real URL.

The in-game **Help** tab has step-by-step guides for job storages, armories
and prices.

---

## Armories and the admin shop

Armories and `/adminshop` are poggy_core menus. Each item is a row with its
price on the right and its description underneath.

1. Pick an item and say how many. Weapons are always one.
2. The job is checked again on every take.
3. The script checks the player can carry it.
4. The money is taken, then the item is given.
5. If the inventory refuses the item, the money is refunded.

An armory's `sellitems` is the name of an item list in the config, such as
`"WeaponArmoryItems"`. Older configs that say `sellitems = Config.WeaponArmoryItems`
keep working.

---

## Admins

**ACE.** Add to `server.cfg`:

```
add_ace group.admin character_storage.admin allow
```

**Character groups.** `Config.AdminGroups` is checked after the ACE, against
the group poggy_core reports for the character. The default `admin`,
`superadmin` and `god` groups work with no setup.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| A storage looks empty after a restart | Make sure `poggy_core` starts before this script. |
| No prompt at a storage | Stand closer, or raise **Open distance** (`Config.AccessRadius`). |
| No blips at all | Check `Config.UseBlips` is on. |
| An admin command says no permission | Grant the ACE above, or add their group to `Config.AdminGroups`. |
| A job storage will not open for a job | Job names are case-sensitive. Match the name your framework uses exactly. |
| Discord posts nothing | Paste a real webhook URL and turn `enabled` on, then restart. |
| Discord posts a new message every restart | Paste the message ids the console printed into the config. |
| Text shows a key like `storage_created` | That line is missing from your language block. Copy it from `english`. |

---

## Version history

- **1.5.1**: The **Storages** panel in `/poggy` (see above). `/movestorage`
  and `/deletestorage` now share their code with it, and say so when the
  database refuses a move.
- **1.5.0**: Settings metadata for the `/poggy` hub. The admin command usage
  messages now use the live command names, and armories can name their item
  list (`sellitems = "WeaponArmoryItems"`).
- **1.4.0**: Framework-agnostic: menus, prompts and the shops are drawn by
  poggy_core; no `vorp_menu`, `vorp_inputs` or `vorp_inventory` dependency.
  Needs poggy_core 0.14.0.

See [CHANGELOG.md](CHANGELOG.md) for the full history.

---

## License

[MIT](LICENSE): free to use, change and share.

Support: Poggy Scripts Discord, https://discord.com/invite/rBarFeuzFj
