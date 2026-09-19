# Poggy Util

Ten small, optional server utilities for RedM. Each one has its own switch, so you turn on only what you want.

poggy_util has no framework code of its own. Characters, jobs, duty, money, inventory, permissions and notifications all come from **poggy_core**. The utilities work on every framework poggy_core supports (VORP, RSG, QBR). The one exception is the stipend, which is VORP only.

## Requirements

| Resource | Why |
|---|---|
| `poggy_core` 0.13.0 or newer | Framework layer |
| `oxmysql` | Database |
| `weathersync` | Only for the clock and weather widget |

## Installation

1. Put `poggy_util` in your resources folder.
2. Add `ensure poggy_util` to `server.cfg`, after `poggy_core` and `oxmysql`.
3. For the armor repair kit, copy `docs/armor_kit.png` into your inventory's image folder.
4. Set your server name for the help menu, and replace the example guide with your own.

There is no SQL to import. poggy_core runs `sql/install.sql` when poggy_util starts (see [Database](#database)).

## The utilities

| Utility | What it does | Default |
|---|---|---|
| Area of play | A box showing the busiest zone and the player count | on |
| Clock and weather | Game time and weather under the area-of-play box | on |
| Duty count | On-duty law and doctors inside the box | on |
| Music zones | A YouTube track near set locations | on |
| Object removal | Deletes set object models on a timer | on |
| Dead entity cleanup | Clears dead NPCs, horses, animals and wrecked wagons | off |
| Unstuck | Teleports a stuck player to the nearest road | on |
| Weapon jam | Dirty guns can jam | on |
| Help menu | A searchable `/help` guide | on |
| Armor protection | Armor clothing blocks torso shots | off |
| Government stipend | Pays unemployed characters (VORP only) | off |

### Area of play
Every 5 seconds the server finds the zone with the most players. Each player counts for the nearest zone within the search radius.
The clock and duty count sit inside the same box, so they only show while it does.

### Clock and weather
The game time and weather from `weathersync`, under the area-of-play box.

### Duty count
On-duty law and doctors, inside the area-of-play box.
Law is any job in the law jobs list. Doctors are the jobs poggy_core counts as medical (`PoggyCoreConfig.MedicalJobs` in `poggy_core/config.lua`).

### Music zones
Plays a YouTube track near a zone's locations. It gets quieter with distance, and (optionally) when looking away.
Each zone needs a YouTube id, a radius, a volume and at least one location.

### Object removal
Deletes the listed object models on every player's game, every few minutes.
With dead entity cleanup on, it also clears dead NPCs, horses, animals and destroyed wagons.

### Unstuck
Teleports the player to the nearest road after a countdown. The player is held still; dying cancels it.

### Weapon jam
Dirty guns can jam when fired, likelier the dirtier they are. A jammed gun clicks instead of firing. It works again once cleaned.

### Help menu
A searchable guide with an **I'M STUCK** button. It ships with example content, all marked `-- replace with your own`.
The header shows your server name and an optional logo.

### Armor protection
While a player wears something in the native armor clothing slot, torso shots are blocked.
The armor absorbs a set number of hits, then breaks until the player uses the repair kit.
A HUD icon shows what is left; `/movearmor` lets each character move it.

### Government stipend
Pays unemployed characters cash on a timer, once they have been on the server long enough. The pay rises with time on the server.
It reads `characters.created_at`, which `sql/install.sql` adds. That table is VORP's, so **the stipend is VORP only**.
On any other framework it prints one yellow console line at start and stays off.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/aop` | Everyone | Hide or show the area-of-play box |
| `/aop l`, `/aop r` | Everyone | Move the box left or right (remembered) |
| `/help` | Everyone | Open or close the help menu |
| `/unstuck` | Everyone | Teleport to the nearest road (name set by `Config.Unstuck.Command`) |
| `/togglemusic` | Everyone | Music zones off or on for yourself (remembered) |
| `/movearmor` | Everyone | Drag the armor icon; click anywhere to save |
| `/removeobjects [model ...]` | Admins | Remove the given models, or every listed one |
| `/cleandead` | Admins | Run dead entity cleanup now |

Each command works only while its utility is on.

### Permissions

The two admin commands work for:

- a player with one of the staff groups in `Config.ObjectRemoval.AdminGroups` (default `admin`, `superadmin`, `moderator`),
- the server console.

No ACE is needed.

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy Hub), then a restart of poggy_util.
You can also edit the files by hand:

| File | What is in it |
|---|---|
| `config.lua` | Every utility's switch and settings |
| `config/help.lua` | The help menu's pages |

The hub has two guides: *Writing your /help guide* and *Setting up armor protection*.

`Config.Debug = true` prints debug lines from every utility.

## Exports

Client:

```lua
exports.poggy_util:ArmorIsWearing()            -- wearing armor
exports.poggy_util:ArmorGetShots()             -- shots left
exports.poggy_util:ArmorIsBroken()             -- no shots left
exports.poggy_util:ArmorShouldBlockBleeding()  -- wearing, not broken, last hit on the torso
exports.poggy_util:IsWeaponJammed(weaponHash)  -- that weapon is jammed
exports.poggy_util:OpenHelp()
exports.poggy_util:CloseHelp()
```

Server:

```lua
exports.poggy_util:RemoveObjectsByModel('s_coachlock02x')
exports.poggy_util:RemoveConfiguredObjects()   -- returns how many models were requested
```

Client event: `TriggerEvent('poggy_util:unstuck')` starts unstuck (for keybinds).

## Database

`sql/install.sql` does three things:

- creates `poggy_util_armor_hud` (armor icon position per character),
- adds the `armor_kit` item to `items` if it is missing,
- adds `characters.created_at` for the stipend (even while the stipend is off).

That column is added in two steps that each run once. The first gives existing characters `2025-11-01 00:00:00` (`Config.Stipend.LegacyDate`). The second makes new characters get their creation time.
To use another legacy date, change it in `sql/install.sql` before poggy_util first starts. Changing `LegacyDate` in the config alone does nothing. A column that is already there is left alone.

`items` and `characters` belong to VORP, so those statements are marked `-- poggy: only-if-table`. poggy_core skips them on a framework without the table.
On RSG, add `armor_kit` to `rsg-core/shared/items.lua` by hand; poggy_util prints one yellow line at start while it is missing.

To manage the database yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`, or run `poggycore sql install poggy_util` in the console.

## Troubleshooting

**The clock never shows.**
It needs `weathersync` running, and the area of play on.

**`/removeobjects` says "You don't have permission."**
Add your staff group to `Config.ObjectRemoval.AdminGroups`, or run it from the server console.

**`/cleandead` does nothing.**
Turn on dead entity cleanup (`Config.ObjectRemoval.DeadEntityCleanup.Enabled`).

**The stipend never pays.**
It is VORP only. The character's job must be the unemployed job, and old enough (`MinimumTenureDays`). Payment happens every `IntervalMinutes` to players online at that moment.

**The armor kit does nothing.**
Armor protection is off by default. Turn it on and restart.

## Changelog

- **2.0.6**: Poggy Hub support. Every setting has a label and help text in `/poggy`, with two guides. README rewritten.
- **2.0.4**: Runs on frameworks without `items` / `characters` tables (RSG): those statements in `sql/install.sql` are skipped there instead of erroring, and the armor kit prints one yellow line at start while the item is missing from your framework's item list.
- **2.0.3**: The stipend checks `core.framework` at start and disables itself with one console line on anything but VORP instead of erroring on the tenure query; README says which utility is VORP-only and why.

## Upgrading from 1.x

2.0.0 removed everything poggy_core now provides. If another resource called these poggy_util exports, move it to poggy_core's `Poggy()` verbs:

| Removed | Use instead |
| :--- | :--- |
| `Notify*`, `poggy:Tip*` and the other `poggy:` notification events | `Poggy('notify.styled', { style = ..., ... })` |
| `GetCharacter`, `GetCharacterInfo`, `GetLocalCharacterInfo`, `GetLocalFullName`, `FindCharacter` | `Poggy('char.get')`, `Poggy('char.offline')` |
| `GetPlayer*`, `GetLocalJob*`, `HasJob`, `IsLawEnforcement`, `IsPlayerOnDuty`, `IsLocalOnDuty` | `Poggy('job.get')`, `Poggy('job.has')`, `Poggy('job.isLaw')`, `Poggy('players.onDuty')` |
| `GetPlayers`, `GetPlayerWeapons` | `Poggy('players.list')`, `Poggy('weapon.get')` |
| `GetFrameworkType`, `IsFrameworkReady` | `Poggy('core.framework')`, `Poggy('core.ready')` |
| `ExecuteQuery`, `SearchRecords`, `InsertRecord`, `UpdateRecord`, `DeleteRecord` | oxmysql directly |
| `CreateBlip`, `CreateRadiusBlip`, `RemoveBlip`, `SetGPS`, `SetWaypoint`, `ClearWaypoint`, blip natives | the natives directly |
| `Joaat`, `DumpTable`, `GetCurrentWeaponHash`, `GetCurrentWeaponEntity`, `GetPlayerPed`, `GetPlayerCoords`, `IsPlayerDead`, `IsPlayerInVehicle`, `GetCurrentVehicle`, `GetNotificationClass` | the natives directly |

Also removed: the debug commands `/testnotifyleft`, `/armordebug`, `/armorstatus`, `/mountdebug`, `/forcemount` and `/spawnmount`, and the unused location finder and sell finder screens. `Config.PoggyDebug` is now `Config.Debug`.

The armor icon position moved from a column on the character table to `poggy_util_armor_hud`. A player who had moved the icon sets it again with `/movearmor`.

## Credits

Poggy
