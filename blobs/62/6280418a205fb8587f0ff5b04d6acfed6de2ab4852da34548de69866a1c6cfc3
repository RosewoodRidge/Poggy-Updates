# poggy_util

Optional server utilities for RedM. Each utility has its own switch in
`config.lua`, so a server turns on only what it wants.

poggy_util has no framework code of its own. Characters, jobs, duty, money,
inventory, permissions and notifications all come from **poggy_core**, so the
utilities work on whatever framework poggy_core drives.

## Requirements

- `poggy_core` 0.12.0 or newer
- `oxmysql`
- `weathersync`, only for the clock and weather widget

## Install

1. Put `poggy_util` in your resources folder.
2. Add `ensure poggy_util` to `server.cfg` after `poggy_core` and `oxmysql`.
3. For the armor repair kit, copy `docs/armor_kit.png` into your inventory's
   item image folder.
4. Set `Config.Help.ServerName` in `config.lua` and replace the example guide
   in `config/help.lua` with your own.

There is no SQL to import. poggy_core runs `sql/install.sql` when poggy_util
starts (see [Database](#database)).

## Utilities

| Utility | Switch | Default |
| :--- | :--- | :--- |
| Area of Play | `Config.AOP.Enabled` | on |
| Armor protection | `Config.ArmorProtection.Enabled` | off |
| Clock and weather | `Config.ClockWeather.Enabled` | on |
| Duty count | `Config.DutyCount.Enabled` | on |
| Music zones | `Config.MusicZones.Enabled` | on |
| Object removal | `Config.ObjectRemoval.Enabled` | on |
| Dead entity cleanup | `Config.ObjectRemoval.DeadEntityCleanup.Enabled` | off |
| Unstuck | `Config.Unstuck.Enabled` | on |
| Weapon jam | `Config.WeaponJam.Enabled` | on |
| Help menu | `Config.Help.Enabled` | on |
| Government stipend | `Config.Stipend.Enabled` | off |

`Config.Debug = true` prints debug lines from every utility.

### Area of Play
Shows the zone where most players are and the player count. Zones are
`Config.AOP.Zones`. The clock and the duty count sit inside the same box, so they
only show while the AOP does.

### Armor protection
While a player wears something in the native armor clothing slot, torso shots
are blocked. The armor absorbs `MaxShots` hits, then breaks until the player uses
the repair kit (`RepairKitItem`). A HUD icon shows what is left; `/movearmor`
lets each character drag it somewhere else.

### Clock and weather
The game time and weather from `weathersync`, under the AOP.

### Duty count
On-duty law and doctors, inside the AOP box. Law is a job listed in
`Config.DutyCount.LawJobs`; doctors are the jobs poggy_core counts as medical
(`PoggyCoreConfig.MedicalJobs` in `poggy_core/config.lua`). Duty comes from
poggy_core.

### Music zones
Plays a YouTube track near a zone's locations, quieter with distance and when
looking away. Each zone needs `youtubeId`, `radius`, `volume` and `locations`.

### Object removal
Deletes the models in `Config.ObjectRemoval.Objects` on every client every
`IntervalMinutes`. With dead entity cleanup on, it also clears dead NPCs, horses,
animals and wagons.

### Unstuck
Teleports the player to the nearest road after `DelaySeconds`. Moving or dying
cancels it.

### Weapon jam
Dirty guns can jam when fired, likelier the dirtier they are. A jammed gun
clicks instead of firing and works again once cleaned.

### Help menu
A searchable guide with an "I'm stuck" button. It ships with example content:
`config/help.lua` holds the categories, sections and items, all marked
`-- replace with your own`. The header shows `Config.Help.ServerName`; an
optional logo goes in `Config.Help.Logo` (a URL, or a file in `ui/` that you also
list under `files {}` in `fxmanifest.lua`). With no logo set, nothing is shown.

### Government stipend
Pays unemployed characters (`UnemployedJobName`) cash every `IntervalMinutes`,
once they have been on the server `MinimumTenureDays`, rising with tenure. It
reads `characters.created_at`, which `sql/install.sql` adds. That table is
VORP's, so **the stipend is VORP-only**: on any other framework it prints one
yellow console line at start and stays off, and the stipend statements in
`sql/install.sql` fail harmlessly (see [Database](#database)). Every other
utility goes through poggy_core and works on any framework it supports.

## Commands

| Command | Who | What |
| :--- | :--- | :--- |
| `/aop` | player | Hide or show the AOP box |
| `/aop l`, `/aop r` | player | Move it left or right (remembered) |
| `/movearmor` | player | Drag the armor icon; click anywhere to save |
| `/togglemusic` | player | Music zones on or off (remembered) |
| `/unstuck` | player | Teleport to the nearest road (`Config.Unstuck.Command`) |
| `/help` | player | Open the help menu |
| `/removeobjects [model ...]` | admin | Remove the given models, or all configured ones |
| `/cleandead` | admin | Run dead entity cleanup now |

Admin commands need one of the player's groups to be in
`Config.ObjectRemoval.AdminGroups`, or the server console.

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

`sql/install.sql` creates `poggy_util_armor_hud` (armor icon position per
character), adds the `armor_kit` item to `items` if it is missing, and adds
`characters.created_at` for the stipend (even while the stipend is off). That
column is added in two steps that each run once: the first adds it so that
existing characters get `2025-11-01 00:00:00` (`Config.Stipend.LegacyDate`),
the second changes its default so new characters get their creation time. To
use another legacy date, change it in `sql/install.sql` before poggy_util first
starts. A column that is already there is left alone.

`items` and `characters` belong to VORP, so those statements are marked
`-- poggy: only-if-table` and poggy_core skips them on a framework without
the table (the summary line says so in grey). On a framework without an
`items` table (RSG), the `armor_kit` item must be added to the framework's
item list by hand (RSG: `rsg-core/shared/items.lua`); poggy_util prints one
yellow line at start while it is missing. The stipend is VORP-only anyway (it
is the one utility that reads a framework table directly) and disables itself
with a console line elsewhere.

## Changelog

- **2.0.4** — Runs on frameworks without `items` / `characters` tables (RSG):
  those statements in `sql/install.sql` are skipped there instead of erroring,
  and the armor kit prints one yellow line at start while the item is missing
  from your framework's item list.
- **2.0.3** — The stipend checks `core.framework` at start and disables itself
  with one console line on anything but VORP instead of erroring on the tenure
  query; README says which utility is VORP-only and why.

To manage the database yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in
`poggy_core/config.lua`, or run `poggycore sql install poggy_util` in the console.

## Upgrading from 1.x

2.0.0 removes everything poggy_core now provides. If another resource called
these poggy_util exports, move it to poggy_core's `Poggy()` verbs:

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

Also removed: the debug commands `/testnotifyleft`, `/armordebug`,
`/armorstatus`, `/mountdebug`, `/forcemount` and `/spawnmount`, and the unused
location finder and sell finder screens. `Config.PoggyDebug` is now
`Config.Debug`.

The armor icon position moved from a column on the character table to
`poggy_util_armor_hud`; a player who had moved the icon sets it again with
`/movearmor`.

## Credits

Poggy
