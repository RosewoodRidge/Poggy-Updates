# poggy_core

One documented framework API. Write a script once; run it on **RedM** (VORP, RSG
Core, QBCore RedM) or **FiveM** (ESX, QBCore).

The two games ship as separate products built from one source: the folder is
`poggy_core` on both, the feed id is `poggy_core` on RedM and
`poggy_core_fivem` on FiveM, so a server is never sent the other game's files.
See [FiveM](#fivem-esx-and-qbcore-0210).

**Version 0.17.1.** Admins on the framework's user record now pass `perms.isAdmin`; see [Fixed in 0.17.1](#fixed-in-0171-admins-on-the-user-record). The VORP adapter is complete and every Poggy resource runs
on it through `Poggy(verb, payload)`; see [Verbs added in 0.11.0](#verbs-added-in-0110).
Scripts are known by their `poggy_id`, so a server owner may rename any
script's folder; see [Script identity](#script-identity). `restart poggy_core`
brings the scripts that stopped with it back; see
[Restarting poggy_core](#restarting-poggy_core-0150). After the start-up update
check it lists, once, the Poggy scripts a server does not have; see
[The catalogue](#the-catalogue-0150).
poggy_core is a hard dependency of every script, and it no longer needs
poggy_util, vorp_menu or vorp_inputs for anything: it draws notifications,
menus and text boxes itself; see [Menu and input](#menu-and-input-0140). The RSG
adapter (rsg-core 2.3.13 / rsg-inventory 2.8.5) is proven in game; see
[RSG](#rsg). The QBR adapter is written against qbr-core 1.0.3 / qbr-inventory
1.0.1 and **proven in game on 14 September 2026** (`poggycore selftest full` passes); see [QBR](#qbr). RedEM:RP and RPX are
detected but not supported, and no adapter is planned for them: on those
poggy_core falls back to standalone, which refuses every framework call
honestly rather than pretending to succeed, and reports
`Core.HasAdapter() == false`. See [What is not built yet](#what-is-not-built-yet).

```lua
local Core = exports.poggy_core:Get()

Core.Ready(function()
    Core.Storage.Register('town_safe', { label = 'Town Safe', slots = 60 })
end)

RegisterNetEvent('myscript:buy', function(item, price)
    local src = source
    if not Core.Inventory.CanCarry(src, item, 1) then
        return Core.Notify(src, 'Your satchel is full.', 'error')
    end
    if not Core.Money.Remove(src, 'cash', price, 'myscript:buy') then
        return Core.Notify(src, 'You cannot afford that.', 'error')
    end
    Core.Inventory.Add(src, item, 1)
    Core.Notify(src, 'Purchased.', 'success')
end)
```

---

## Install

Drop `poggy_core` in your resources folder and start it **before** anything that
uses it:

```cfg
ensure vorp_core
ensure vorp_inventory
ensure poggy_core

# lets poggy_core restart the Poggy scripts it has just updated, and bring
# them back after `restart poggy_core`
add_ace resource.poggy_core command.refresh allow
add_ace resource.poggy_core command.ensure allow
```

No database import. Automatic updates are on from the moment you install (see
Updates); the two `add_ace` lines let poggy_core restart what it updated and
start the scripts that stop with it (see
[Restarting poggy_core](#restarting-poggy_core-0150)), and it prints them in the
console if they are missing. Every value in `config.lua` has a working default.

`lua54 'yes'` is set. Any resource that consumes poggy_core should set it too.

---

## Script identity

Every Poggy script declares its product id in its `fxmanifest.lua`:

```lua
poggy_id 'poggy_scene'
```

- **What it is.** The name the update feed, the release list and poggy_core know
  the script by. It names the script, not its folder.
- **Never change it.** A different id is a different product: the script would
  stop receiving updates, and migrations and container ids recorded under the
  old id would no longer be found.
- **The folder may be renamed.** A server owner can call the folder anything.
  Scripts refer to themselves with `GetCurrentResourceName()`, and when a script
  starts its bridge tells poggy_core which id lives in which folder
  (`core.register`, sent the first time `PoggyReady()` passes, before
  `sql/install.sql` runs). poggy_core takes the folder from the call itself,
  never from the payload, and refuses an id the script's manifest does not
  declare. Stopping the script removes the registration.
- **One folder per id.** A second folder registering an id that a started folder
  already holds is refused with one red line naming both, and the updater
  leaves that id alone until one of them is stopped or removed.
- **No `poggy_id` line** means the id is the folder name, exactly as before 0.13.0.

What uses the id rather than the folder: the updater (see [Updates](#updates)),
the prefix of namespaced container ids (`pg_<poggy_id>_<id>`), and the
`poggy_migrations` table. `poggycore scripts` lists every registered script
with its folder and version.

---

## Restarting poggy_core (0.15.0)

Every Poggy script declares `dependency 'poggy_core'`, so `restart poggy_core`
stops all of them, and FXServer does not start them again. poggy_core does: a
couple of seconds after it is back it runs `ensure <script>` for each one that
stopped with it and prints one line:

```
[Poggy Core] restarted 3 Poggy script(s) that stopped with it: poggy_fishing, poggy_markets, poggy_scene
```

Nothing is printed when nothing stopped with it. A script that does not come
back is named in yellow with the `ensure` to run. The `ensure` goes through
the `add_ace resource.poggy_core command.ensure allow` line from
[Install](#install); without it poggy_core falls back to `StartResource`.

**Which scripts.** A dependent is any resource whose `fxmanifest.lua` declares
a `poggy_id`, or lists poggy_core under `dependency` / `dependencies`.
`poggycore dependents` lists them with their state.

**Restart or fresh boot.** On a fresh server boot every Poggy script is
"stopped" too — `server.cfg` has just not reached them yet, and it will start
them in its own order — so poggy_core must not touch them then. The rule:

1. While it runs, poggy_core keeps a record of every dependent it has seen
   started (each `onResourceStart`, plus whatever was already running when it
   came up). A script you stop yourself leaves the record; one that stops in the
   fifteen seconds before poggy_core stops is kept, because it stopped with it.
2. The record lives in the convar `poggy_core_last_stop`, written on every
   change and once more when poggy_core stops. A convar set at run time exists
   in the server process only, so a server restart clears it: on a fresh boot
   there is no record and nothing is started.
3. When poggy_core starts and finds a record, it was restarted on a running
   server. Every recorded script that is still a dependent and is `stopped` is
   ensured; a script the server never started is not in the record and is left
   alone. The record is then consumed and this run starts its own. As a second
   check, a record found in the first ten seconds of server uptime is ignored.

A poggy_core older than 0.15.0 kept no record, so the first
`refresh` + `ensure poggy_core` after upgrading finds none: when the server has
been up for more than five minutes and dependents are stopped, one grey line
names them with the `ensure` commands to run. Every restart after that has a
record.

`PoggyCoreConfig.RestartDependents = false` turns the restart off; the record is
still kept, so the switch works without a server restart. This is not a file
write, so it also runs on a development server.

---

## Settings hub: /poggy (0.18.0)

`/poggy` in game opens a full-screen hub with a card for every Poggy script on
the server. Opening a card shows that script's settings, its lists (shops,
recipes, locations...), its commands, its README and help pages, and the
history of every change made from the hub, and lets you restart it.

**The config file is the truth.** The hub changes the script's own
`config.lua` (and `config/*.lua`, `translations.lua`) in place: every comment
and every line it does not change stays exactly as it was. There is no second
copy of your settings anywhere; the script reads its `Config` as it always has.
After a save the hub offers a restart, because most scripts read their config
once when they start. The database tables below are a mirror and a history,
rebuilt from the files and never written back into them. If they disagree, the
file wins.

**Opening is quick (0.22.0).** poggy_core reads every script's config in the
background when it starts, and again whenever a script restarts, is updated or
is saved from the hub, so `/poggy` shows the cards without reading any files.
A config file edited by hand while the server runs is read when you open that
script, and its card is up to date from then on.

### Who can use it

| ACE | What it allows |
|---|---|
| `poggy.settings` | open the hub and edit (anyone the framework counts as an admin may too) |
| `poggy.settings.takeover` | take over a script someone else is editing |

```cfg
add_ace group.admin poggy.settings allow
add_ace group.admin poggy.settings.takeover allow

# lets the hub restart a script after you save (the updater uses the same line)
add_ace resource.poggy_core command.ensure allow
```

Without the `command.ensure` line, restarts fall back to stopping and starting
the resource, which some servers refuse. Every call the page makes is checked
again on the server; nothing the game client sends is trusted.

### One editor per script

Opening a script takes its lock. Anyone else who opens it sees it read-only,
with who is editing it and since when. The lock is released when you leave the
script, close the hub, disconnect, or do nothing for `Hub.IdleMinutes` (a
warning comes at `Hub.IdleWarnMinutes`). Someone with
`poggy.settings.takeover` can take it over: you are told who took it, and your
unsaved changes are dropped.

While a script is locked the console says so:

```
[poggy] Jane is editing Supply Drops in /poggy. Do not edit its config files by hand until they finish.
```

and the updater leaves it alone: an automatic update waits for the next check,
and `poggycore update <id> apply` refuses and names the editor.

If a file changes on disk while you have it open (someone edited it by hand,
or an update merged it), your save is refused rather than overwriting it.
Reload the script and make the change again.

### What a save does

1. Every value is checked on the server: it must have the setting's type and
   meet the limits in the script's `docs/hub.json` (min, max, allowed values,
   length, webhook and colour formats). Settings `docs/hub.json` marks
   `readonly` or `hidden`, and rows the hub cannot rewrite safely, are refused
   whatever the page sends. One bad value refuses the whole save.
2. Each file is copied to
   `poggy_core/update_backups/<folder>__<yyyymmdd-hhmmss>__settings__<file>`
   (the folder the updater uses; `/` in the file name becomes `_`).
3. The file is written by the script itself (`PoggyWriteOwnFile` in the
   bridge), so the script must be running. A stopped script can be viewed but
   not saved; the hub offers Start.
4. One history row per change, with who made it.

poggy_core's own card works the same way, except that poggy_core never
restarts itself: restart it by hand (it restarts every Poggy script).

### Roles

`PoggyCoreConfig.Roles` in `config.lua` holds master lists of jobs and admin
groups (`lawmen`, `medics`, `staff` to start with). In the hub, any list of
job or group names in any script can be linked to a role; the link is a
comment on that line, `-- poggy:role lawmen`, and the script never reads it.
Saving a role on the hub's Roles page rewrites every linked list in every
script and restarts the scripts that changed. Scripts someone else is editing,
and stopped scripts, are skipped and named.

### Changed from the shipped config

The hub marks settings you changed from the version as it shipped, and can
reset them. The shipped files come from the update feed (the same copy the
updater merges against) and are cached in the database. Offline, or on a
development build that was never published, the badges and Reset are simply
not shown.

### Console

```
poggycore settings                              every script, its state, who is editing it, file fingerprints
poggycore settings show <id>                    every setting, its value, file:line
poggycore settings set <id> <path> <json value> the same save the hub makes, logged as "console" (console only)
poggycore settings unlock <id>                  release a script's lock; the editor is told (console only)
```

`show` and the list also work from chat for admins; `set` and `unlock` run from
the server console only. For example:

```
poggycore settings set poggy_markets Config.Debug true
poggycore settings set poggy_markets Config.Webhook "https://discord.com/api/webhooks/..."
```

### Database

poggy_core creates both tables at start when they are missing (no import):

| Table | What it holds |
|---|---|
| `poggy_settings` | one row per script: the mirror of its settings (`toggles`, `inputs`, `lists`, as JSON), the file fingerprint, and the shipped config of the running version (`defaults`) |
| `poggy_settings_log` | one row per change: script, file, path, operation, old and new value, who, when. Rows older than `Hub.HistoryDays` are deleted at start. |

Without oxmysql the hub still edits files; it only keeps no history.

### For script authors: docs/hub.json

Everything in the hub works without it (labels come from the setting names,
tooltips from the config comments). `docs/hub.json` adds the card (label,
tagline, category, description), commands, help pages, tabs, and per-setting
labels, tooltips, limits, pickers, `live` (no restart needed), `advanced`,
`hidden` and `readonly`, plus list columns, row fields and templates. Add
`'docs/icon.png'` to the manifest's `files` for the card's icon. The format is
in `docs/reference/poggy-hub-spec.md` §5 (in the development repository).

Anything a script runs as code (a Lua string passed to `load`, text spliced
into SQL) must be marked `"readonly": true`: the server then refuses to change
it from the hub.

---

## Theme (0.23.0)

Every Poggy script's own screens, and poggy_core's menus and `/poggy`, share one
look: the Poggy inventory's dark panels, thin light edges, square corners and
one accent colour. Black and gold (**Rosewood**) by default. Pick another in
`/poggy` → Poggy Core → **Theme**, or in `config.lua`:

```lua
PoggyCoreConfig.Theme = {
    Preset  = "rosewood",   -- rosewood blackwater lemoyne saint_denis ambarino tumbleweed outlaw silver ledger custom
    Custom  = { Background = "#141414", Text = "#f6efe3", Accent = "#d6ad68", ... },
    OwnLook = {},           -- scripts that keep their own colours, e.g. { "poggy_supplydrops" }
}
```

The theme is live: nothing restarts, and each screen takes the new look the next
time it opens. Screens drawn with pictures (fishing's brass plate, the market's
leather ledger) keep their art; set that script's own skin to `"default"` to
give it the theme instead.

**For script authors.** A page follows the theme with one line at the end of
its `<head>`:

```html
<script src="https://cfx-nui-poggy_core/ui/hub/theme.js"></script>
```

and a skin in poggy_core, `ui/hub/theme-<poggy_id>.css`, that restyles it with
the theme's `--pg-*` tokens. The bridge answers the page's request for the
theme; nothing is ever sent into the page. Tokens, house style and rules:
`docs/reference/poggy-theme-spec.md` on the development machine.
`exports.poggy_core:ThemeFor(poggyId)` (client) returns what a page gets.

## Design rules

These are promises, not preferences. Code against them.

**Nothing silently no-ops.** Every mutating call returns `ok, err`. Every read
returns a value or `nil` plus a reason. If the framework cannot do the thing, you
find out at the call site.

**Capabilities, not framework names.** `Core.Has('money.gold')` tells you what
will actually work. `Core.GetFramework()` exists for logging, not for branching.

**One shape everywhere.** VORP's inventory is callback-based, RSG's is
synchronous, RedEM's returns a handle object. All of it is normalised to
synchronous-looking calls, so `Core.Inventory.Add` reads the same on all five.

**Capacity is checked before every add.** RSG's own `AddItem` drops the item on
the ground when it does not fit and returns false; QBR has no capacity check at
all. `Core.Inventory.Add` checks first on every framework, so `ok` means the same
thing everywhere.

**Anything that does not need the framework is not delegated.** Notifications,
prompts and callbacks are ours. That removes four incompatible notify signatures
and a real concurrency bug in two frameworks' callback systems.

---

## API

Get a handle. It is bound to your resource, which is how container ids get
namespaced and how warnings know who to blame.

```lua
local Core = exports.poggy_core:Get()   -- server or client
```

### Lifecycle

| Call | Returns | Notes |
|---|---|---|
| `Core.Ready(fn)` | — | Runs `fn` once the framework resolves. Runs immediately if it already has. |
| `Core.WaitReady(ms)` | `boolean` | Blocks. Needs a thread. |
| `Core.IsReady()` | `boolean` | |
| `Core.GetFramework()` | `string` | `vorp`, `rsg`, `qbr`, `redem`, `rpx`, `standalone` |
| `Core.Has(cap)` | `boolean` | See [Capabilities](#capabilities) |
| `Core.HasAdapter()` | `boolean` | False when no framework, **or** one we cannot drive yet. Check this before relying on us. |
| `Core.RequireVersion('0.1')` | `true` | Errors at boot if poggy_core is older |
| `Core.Native()` | framework core | Escape hatch. Makes your script framework-specific. |

### Character — server

```lua
local char = Core.GetChar(src)          --> PoggyChar | nil
local char = Core.GetCharByCharId(id)   --> PoggyChar | nil   (online only)
local id   = Core.GetCharId(src)        --> string | nil
local list = Core.GetPlayers()          --> { source, ... }
```

`PoggyChar`:

| Field | Type | Notes |
|---|---|---|
| `charId` | string | **Always a string**, even on VORP where it is an integer internally |
| `ownerId` | string | Account level: steam on VORP and RedEM, license on the rest |
| `firstName` `lastName` `fullName` | string | |
| `job` `jobLabel` | string | |
| `jobGrade` | number | Normalised from RSG/QBR's grade *table* |
| `jobGradeLabel` | string or nil | nil on VORP, which has no per-grade label |
| `onDuty` | boolean or **nil** | **nil means the framework cannot tell you.** Handle it. |
| `group` | string | |
| `gender` | string | `'m'`, `'f'` or `'unknown'` |
| `money` | table | Only supported currencies are present |
| `native` | table | The framework's own character object |

On the client, `Core.GetChar()` takes no source and returns the local character,
read from the framework's state bag where possible and from the server otherwise.
It is cached for five seconds and invalidated immediately on character load or
job change.

### Money — server

```lua
Core.Money.Supports(currency)                     --> boolean
Core.Money.Get(src, currency)                     --> number | nil, err
Core.Money.Add(src, currency, amount, reason)     --> ok, err
Core.Money.Remove(src, currency, amount, reason)  --> ok, err
Core.Money.Set(src, currency, amount, reason)     --> ok, err
```

Canonical currencies: `cash`, `bank`, `gold`, `rol`. Check `Supports` first,
because the spread is wide:

| | VORP | RSG | QBR | RedEM | RPX |
|---|---|---|---|---|---|
| `cash` | yes | yes | yes | yes | yes |
| `bank` | **no** | yes | yes | yes | yes |
| `gold` | yes | yes, integers only | only if the server adds it to `MoneyTypes` | **no** | **no** |
| `rol` | yes | **no** | no | no | no |

`Remove` refuses and returns `false, 'no_funds'` rather than taking a player
negative, on every framework, including the ones that would allow it (RSG lets
the bank reach -5000; poggy_core does not).

**Money items on RSG.** With rsg-core's `EnableMoneyItems` (on by default) cash
is re-derived from `dollar` and `cent` inventory items, and `AddMoney` ends in
an inventory add that rsg-inventory drops on the ground when it does not fit.
`Core.Money.Add` therefore capacity-checks the money items first and returns
`false, 'no_space'` instead. Gold is the same when `EnableGoldItems` is on. RSG's
other five account types (`valbank`, `rhobank`, `blkbank`, `armbank`,
`bloodmoney`) are not exposed; use `Core.Native()` for them.

**On QBR** the accounts are whatever `QBConfig.Money.MoneyTypes` lists: `cash`
and `bank` in the stock config, and `money.gold` is declared only when a
server has added `gold` there. There are no money items. qbr-core itself only
refuses a negative balance for the types in `DontAllowMinus` (cash alone by
default) and would let the bank go under; poggy_core refuses every overdraw
with `no_funds`.

### Jobs

```lua
-- server
local name, grade, label, gradeLabel, onDuty = Core.Job.Get(src)
Core.Job.Set(src, 'sheriff', 2)        --> ok, err    (one call on every framework)
Core.Job.SetDuty(src, true)            --> ok, err    (unsupported on VORP)
Core.Job.Has(src, 'sheriff', 1)        --> boolean    (name or array, case-insensitive)
Core.Job.IsLaw(src) / Core.Job.IsMedical(src)
Core.Job.OnChange(function(src, job, grade, oldJob, oldGrade) end)

-- client: same, without the source argument
```

`OnChange` is normalised. VORP fires two separate events, RSG one, QBR only a
client event, RedEM nothing except from an admin command, and RPX signals only
through a state bag. You get one event.

Since 0.11.0 the client hears job changes too: poggy_core relays them and
re-broadcasts `poggy_core:jobChangedLocal` (job, grade, oldJob, oldGrade) on the
client, the same way `poggy_core:charLoadedLocal` works.

**`onDuty` on VORP** comes from vorp_police's `isOnDuty` export and from the
`isPoliceDuty` / `isMedicDuty` state bags that vorp_police and vorp_medic set.
Before 0.11.0 the adapter called an export name that does not exist, so
`onDuty` was `nil` for everyone. It is still `nil` when neither resource runs.

**On RSG** duty is part of the job (`PlayerData.job.onduty`), so `onDuty` is
always a boolean and `Core.Job.SetDuty` works. `jobGradeLabel` is the grade's
name from `RSGShared.Jobs`. `Core.Job.Set` takes the label from that table, not
from the caller, and returns `false, 'not_found'` for a job the table does not
list. A grade left `nil` keeps the player's current grade when the job name is
unchanged (RSG itself would reset it to 0). `persist = true` writes the `job`
JSON column of `players` immediately; RSG also saves the row itself every few
minutes and on disconnect.

**On QBR** duty is also part of the job (`PlayerData.job.onduty`), labels and
grade names come from `QBShared.Jobs`, `Core.Job.Set` refuses an unknown job
with `not_found`, and a `nil` grade keeps the current grade when the job name
is unchanged (qbr-core itself would land on grade 0, "No Grades"). Setting a
job resets duty to that job's `defaultDuty`, because qbr-core does. qbr-core
fires **no server event** on a job change, so `OnChange` works like this:
poggy_core remembers each player's last job, re-checks it after its own
`Job.Set`, and re-checks it when the player's client hears qbr-core's
`QBCore:Client:OnJobUpdate` and pokes the server (`poggy_core:qbr:jobPoke`);
the job itself is always read back from qbr-core, nothing from the client is
trusted. A duty toggle is not relayed. `persist = true` writes the `job` JSON
column of `players`.

Which jobs count as law or medical is in `config.lua`, not in code. RSG's stock
job names (`vallaw`, `rholaw`, `blklaw`, `strlaw`, `stdenlaw`) are not in the
default `LawJobs` list; add them there on an RSG server. QBR's stock `police`
and `ambulance` are in the default lists.

### Inventory — server

```lua
Core.Inventory.CanCarry(src, item, qty)     --> boolean, reason
Core.Inventory.MaxCarry(src, item, cap)     --> number, err   (0.16.0)
Core.Inventory.Add(src, item, qty, meta)    --> ok, err
Core.Inventory.Remove(src, item, qty, meta) --> ok, err
Core.Inventory.Count(src, item, meta)       --> number
Core.Inventory.Has(src, item, qty)          --> boolean
Core.Inventory.Get(src)                     --> PoggyItem[]
Core.Inventory.SetMeta(src, itemId, meta, amount)
Core.Inventory.RegisterUsable(item, fn)     -- fn(src, item)
```

`Add` always capacity-checks first and returns `false, 'no_space'` rather than
letting the framework improvise.

`Remove` verifies the player has the items before removing, so `false,
'not_found'` is meaningful.

**`RegisterUsable` survives an inventory restart (0.13.1).** `vorp_inventory`
keeps usable-item handlers in memory, forgets all of them when it restarts, and
never drops one whose script has stopped. poggy_core keeps its own registry of
every item registered through it: when `vorp_inventory` is answering again it
re-registers each one once and prints one grey line, `re-registered N usable
item(s) after vorp_inventory restarted`; when a script stops, its items are
dropped and unregistered. **Do not watch `onResourceStart` for `vorp_inventory`
in your script** — that is done here, once, for everyone. One handler per item:
registering an item another script already handles replaces it, with one yellow
line naming both scripts. `/poggycore usables` lists item → script.

Weapons are a separate namespace, deliberately: VORP models them as first-class
objects with their own loadout table.

```lua
Core.Weapons.Get(src) / Core.Weapons.Add(src, name, ammo, comps) / Core.Weapons.Remove(src, id)
```

**On RSG a weapon is an inventory item** (`type = 'weapon'`, a serial in
`info.serie`). `Core.Weapons` is a view over those items: `Get` lists them with
`id` = the serial (the slot when there is none), `Remove` takes that id, `Add`
gives the item after a capacity check. `ammo` and `components` are accepted and
**not applied** on RSG: ammunition is a separate item the player loads and
components belong to rsg-weaponcomp; neither has a server export. The same
weapons also appear in `Core.Inventory.Get` and `inv.items` there.

**Usable-item handlers on RSG** are held by rsg-core (`CreateUseableItem`),
which calls them as `(source, itemData)`. poggy_core wraps the call so the
handler always receives one table with `source`, `name`, `amount`, `slot`,
`metadata` and `item`, the shape the Poggy scripts already read on VORP.

**On QBR** the inventory lives in qbr-core (`Player.Functions.AddItem` and
friends), not in qbr-inventory, and there is no capacity check to call:
`CanCarry` computes the weight against `QBConfig.Player.MaxWeight` and a stack
or free slot against `MaxInvSlots` (`inventory.carrycheck` is false). Item
names are lower-case and the VORP weapon spelling is mapped the same way as on
RSG. `Remove` walks stacks itself, because qbr-core's own `RemoveItem` only
takes from a single stack that holds the whole amount. Weapons are items with
`type = 'weapon'` and a serial in `info.serie` that qbr-core stamps on
creation; `Core.Weapons` is the same view as on RSG, and `ammo` / `components`
are not applied. Usable handlers are held by qbr-core (`CreateUseableItem`,
called as `(source, item)`) and wrapped into the same table shape; they are
replayed if qbr-core restarts. `SetMeta` takes the slot as `itemId`. `Close`
runs qbr-inventory's `closeinv` client command for the player.

### Storage — server

```lua
Core.Storage.Register(id, opts)        --> ok, err     -- idempotent, call every boot
Core.Storage.IsRegistered(id)          --> boolean
Core.Storage.Open(src, id)             --> ok, err
Core.Storage.Close(src, id)            --> ok, err
Core.Storage.AddItem(id, item, qty, meta, charId)
Core.Storage.RemoveItem(id, item, qty, meta)
Core.Storage.GetItems(id)              --> PoggyItem[]
Core.Storage.SetCapacity(id, slots, maxWeight)
Core.Storage.Unregister(id)            --> ok, err     -- forget it, keep the items
Core.Storage.Delete(id)                --> ok, err     -- destroy it and the items
Core.Storage.UseRawIds(true)           -- opt out of id namespacing
Core.Storage.ResolveId(id)             --> the namespaced id actually used
```

**`Unregister` and `Delete` are not the same thing.** Unregister forgets the
container while its contents stay in the database, so registering it again brings
them back. Delete destroys them. On VORP the definition is memory-only anyway, so
Unregister is the one you almost always want at runtime.

**`UseRawIds(true)` is for a resource that already has containers in the wild.**
Items are keyed by container id inside the inventory tables, so a resource that
starts namespacing its ids hides everything already stored. Call it once before
registering anything, and pick ids nobody else would.

`opts`: `label`, `slots`, `maxWeight`, `shared`, `allowWeapons`,
`whitelistItems`, `jobAccess` (`{ [job] = minGrade }`), `charAccess`
(`{ charId, ... }`), `raw` (skip namespacing).

**`maxWeight` is ignored on VORP.** A stock `vorp_inventory` container can only be
weight-limited through a field that collides with one of its own method names,
which breaks the container. VORP containers are limited by `slots` instead. Pass
both and the script works on every framework.

**Register on every boot.** VORP keeps container definitions in memory only, so
they vanish on restart. Register is idempotent and cheap, and poggy_core replays
every registration automatically if `vorp_inventory` restarts underneath you.

**On RSG** a container is an rsg-inventory stash (`CreateInventory`). Its
contents persist in the `inventories` table; its label, slots and maxweight do
not, so Register on every boot is just as necessary there, and poggy_core
replays registrations when `rsg-inventory` restarts. `storage.persist` is false.
`AddItem` and `RemoveItem` write the row at once (rsg-inventory itself only
writes when a player closes the stash). A stash is single-occupancy: `Open`
returns `false, 'framework_error'` while another player has it open, or while
the opener's own inventory is busy. `jobAccess`, `charAccess`, `shared`,
`allowWeapons` and `whitelistItems` have no counterpart and are ignored
(`storage.permissions` is false). `Delete` empties the stash, drops it and
removes its row; `Unregister` saves and drops it, leaving the row.

**On QBR** a container is a qbr-inventory stash, which is one row of the
`stashitems` table (`stash`, `items` JSON) that qbr-inventory loads into
memory when a player opens it and writes back when they close it. qbr-inventory
has no server export for any of this, so poggy_core keeps the definition
(label, slots, maxWeight) itself, hands it to qbr-inventory at open time, and
reads and writes the row directly for `AddItem`, `RemoveItem`, `GetItems`,
`GetWeapons` and `Delete` (each needs a thread). Two consequences: a write made
while a player has that stash open is overwritten when they close it, and the
title on screen is always `Stash-<id>`. `Open` asks the player's own client to
send the open, the way every QBR script does, and refuses while the player's
inventory is busy. `Register` on every boot is still required (the definition
is in memory). Nothing on a QBR server creates `stashitems`: poggy_core checks
for it at start and, when it is missing, prints the `CREATE TABLE` to run and
refuses storage with `unsupported` until it exists.

**Ids are namespaced per script** as `pg_<poggy_id>_<id>`, so renaming a folder
does not hide what is stored (a script without a `poggy_id` uses its folder
name, which is what every script used before 0.13.0). Two scripts cannot
collide, and we stay clear of the prefixes RSG reserves (`police-`, `marshal-`,
`gang-`, `admin-`, `evidence-`). Pass `raw = true` to opt out.

### Notifications

```lua
Core.Notify(src, text, kind, duration)   -- server
Core.Notify(text, kind, duration)        -- client
Core.Notify.Rich(src, { title = , description = , kind = , duration = })
```

`kind` is `info`, `success`, `error` or `warning`.

**Style-preserving variants.** The four kinds above are the portable API and the
right choice for new code. Migrating existing code is different: a script that
has always drawn a bottom-of-screen objective banner should keep drawing one,
because where a message appears is a visible part of the game. These five map
onto the RDR2 notification styles directly.

```lua
Core.Notify.Tip(src, text, duration)        -- bottom tip
Core.Notify.Right(src, text, duration)      -- right-hand tip
Core.Notify.Objective(src, text, duration)  -- bottom objective banner
Core.Notify.Top(src, title, subtitle, duration)
Core.Notify.Advanced(src, text, dict, icon, color, duration)
-- client: the same, without the src
```

poggy_util calls the objective banner `NotifyObjective` and VORP calls the same
thing `vorp:TipBottom`; they are one style under two names.

**More styles (0.11.0).** The rest of the styles poggy_util offered, so a script
can stop calling poggy_util without changing what players see. Through the verb:

| style | payload keys | replaces poggy_util's |
|---|---|---|
| `location` | `text`, `location` | `NotifyTop` |
| `left` | `title`, `subtitle`, `dict`, `icon`, `color` | `NotifyLeft` |
| `leftRank` | `title`, `subtitle`, `dict`, `icon`, `color` | `NotifyLeftRank` |
| `basicTop` | `text` | `NotifyBasicTop` |
| `center` | `text`, `color` | `NotifyCenter` |
| `bottomRight` | `text` | `NotifyBottomRight` |
| `fail` | `title`, `subtitle` | `NotifyFail` |
| `dead` | `title`, `audioRef`, `audioName` | `NotifyDead` |
| `update` | `title`, `subtitle` | `NotifyUpdate` |
| `warning` | `title`, `subtitle`, `audioRef`, `audioName` | `NotifyWarning` |

```lua
Poggy('notify.styled', { src = src, style = 'left', title = 'Sheriff',
    subtitle = 'Shots fired', dict = 'generic_textures', icon = 'tick', duration = 5000 })
```

Every style takes `duration`; `advanced` also takes `quality` and
`showQuality`. `fail`, `dead`, `update` and `warning` hold the banner for
`duration` and clear it on their own thread, so they never block the caller.
An unknown style fails with `bad_argument`; before 0.11.0 it returned true and
drew nothing. A script using these styles needs `poggy_core_min '0.11.0'`.

Renderers are tried in the order set in `config.lua`: poggy_core's own native
renderer (`client/cl_notify_native.lua`, the same RDR2 natives poggy_util and
vorp_core draw with, so nothing looks different), then the framework's own,
then chat. `'poggy_util'` in an older config is read as `'native'`; poggy_util
does not have to be running. A notification is never silently lost.

### Callbacks

Our own transport, not the framework's, because both RSG and QBR key pending
callbacks by name with no request id and lose a response when two calls to the
same name are in flight at once.

```lua
-- server
Core.Callback.Register('myscript:getStock', function(src, shopId)
    return getStock(shopId)
end)
local answer = Core.Callback.AwaitClient(src, 'myscript:askClient', arg)

-- client
local stock = Core.Callback.Await('myscript:getStock', shopId)
Core.Callback.Trigger('myscript:getStock', function(stock) end, shopId)
Core.Callback.Register('myscript:askClient', function(arg) return something end)
```

`Await` needs a thread. Calling it outside one prints a message that says so
rather than the engine's unhelpful coroutine error. Requests time out after
`RpcTimeout` and resolve to nothing rather than hanging.

Server-to-client callbacks work on every framework, including QBR, which has no
such thing natively.

### Prompts — client

```lua
Core.Prompt.Register('shop_door', {
    coords = vector3(-321.0, 803.0, 117.0),
    key = 'E', label = 'Open shop', distance = 2.0,
    onActivate = function() openShop() end,
})
Core.Prompt.Remove('shop_door')
Core.Prompt.Clear()
```

Delegates to the framework's own prompt system on RSG, QBR and RPX; uses a native
implementation on VORP and RedEM. Registration is idempotent by name, and every
prompt a resource registered is cleaned up when it stops.

### Menu and input (0.14.0)

poggy_core draws its own list menu and text box (`ui/` in this resource), so
no Poggy script needs `vorp_menu` or `vorp_inputs`, and every framework gets
the same screens: a titled list with an optional subtitle, each row a label
with an optional right-hand text and a description line for the highlighted
row, disabled rows greyed, a close button, keyboard (arrows or W/S, Enter,
Escape or Backspace to close) and mouse (hover highlights, click picks,
right-click closes), and a scrolling list when it is long. The look matches
the other Poggy UIs: dark panel, parchment text, gold accents, serif headings.
Nothing in it loads from the internet.

Three verbs. The two that wait for the player need a thread (a `CreateThread`,
an event handler, or a callback), the same rule as `callback.await`.

| Verb | Side | Payload | Value |
|---|---|---|---|
| `menu.open` | both, thread | `title`, `items`; `src?` (server), `subtitle?`, `cursor?`, `closeText?` | `{ value, index, item }` — or `false, 'closed'` when the player backs out |
| `menu.close` | both | `src?` (server) | `true` (also when nothing was open) |
| `input.text` | both, thread | `title`; `src?` (server), `placeholder?`, `default?`, `maxLength?`, `numeric?`, `submitText?` | the text, or a number when `numeric = true` — or `false, 'closed'` |

`items` is an array of `{ label, value?, desc?, right?, disabled? }`. The
answer's `item` is your own table for the chosen row, `index` its 1-based
position and `value` its `value` (the index when it has none). One menu or
input at a time: opening another replaces it, and the earlier caller gets
`false, 'closed'`. `cursor` shows the mouse (default `PoggyCoreConfig.Ui.Cursor`,
`true`); keyboard navigation works either way. Escape always closes.

**Where it sits.** `PoggyCoreConfig.Ui.Position` places the menu and the text
box for every Poggy script at once: `'center'` (default), `'left'`, `'right'`
(centred vertically, against that side), `'top-left'`, `'top-right'`,
`'bottom-left'` or `'bottom-right'`. `PoggyCoreConfig.Ui.Margin` is the gap in
pixels from the screen edge (`40`). The panel is pinned by its edge rather than
centred by a transform, and the description area under the list is reserved
at three to five lines (longer text is cut with an ellipsis), so moving the
highlight never shifts the rows: centre and top positions grow downward, and
the bottom ones hold the description at its full five lines so the panel's
height never changes. The config is read on every open, so an edit shows on the
next menu without a restart. Colours are not configurable yet; the palette is
one block of CSS variables at the top of `ui/style.css` for when a shared theme
layer arrives.

**Client**, from any script:

```lua
CreateThread(function()
    local ok, pick = Poggy('menu.open', {
        title    = 'Stable',
        subtitle = 'Pick a horse',
        items    = {
            { label = 'Arabian',  value = 'horse_arabian',  right = '$120', desc = 'Fast and nervous.' },
            { label = 'Shire',    value = 'horse_shire',    right = '$80',  desc = 'Strong and steady.' },
            { label = 'Mustang',  value = 'horse_mustang',  right = 'sold', disabled = true },
        },
    })
    if not ok then return end                 -- 'closed': the player backed out
    print(pick.value, pick.index, pick.item.label)

    local okName, name = Poggy('input.text', { title = 'Name your horse', placeholder = 'Buttercup', maxLength = 24 })
    if okName then print('named ' .. name) end

    local okQty, qty = Poggy('input.text', { title = 'How many?', default = 1, numeric = true })
    if okQty then print(qty + 1) end          -- a number, not a string
end)

Poggy('menu.close', {})                       -- take down whatever is open
```

**Server**, naming the player. The request rides poggy_core's own callback
transport to that client and returns the client's answer. It waits up to
`PoggyCoreConfig.Ui.Timeout` (five minutes by default; `RpcTimeout` when the
config has no `Ui` block), then closes the page and returns `false, 'timeout'`.
A player who disconnects mid-menu answers `false, 'closed'` at once.

```lua
RegisterNetEvent('mystable:browse', function()
    local src = source                        -- an event handler is a thread already
    local ok, pick = Poggy('menu.open', { src = src, title = 'Stable', items = horsesFor(src) })
    if not ok then return end
    if not Poggy('money.remove', { src = src, amount = pick.item.price }) then
        return Poggy('notify', { src = src, text = 'You cannot afford that.', kind = 'error' })
    end
    Poggy('notify', { src = src, text = 'Bought ' .. pick.item.label, kind = 'success' })
end)

Poggy('menu.close', { src = src })
```

Because the items travel to the client and back, a `value` on the server side
must be plain data: a string, number, boolean, or a table of those. A function
cannot cross.

The same thing on the Core object: `Core.Menu.Open(opts)`, `Core.Menu.Close()`,
`Core.Input.Text(opts)` on the client; `Core.Menu.Open(src, opts)`,
`Core.Menu.Close(src)`, `Core.Input.Text(src, opts)` on the server. `Open` and
`Text` return the value, or `false, err`. `Core.Menu.Native()` still hands over
the framework's own menu object for scripts that have not moved yet.

A script using these declares `poggy_core_min '0.14.0'`.

### Permissions — server

```lua
Core.Perms.GetGroup(src)   --> string
Core.Perms.IsAdmin(src)    --> boolean
```

On VORP the group is the character's `group` column. On RSG it is ACE: the
highest of `RSGCore.Config.Server.Permissions` (`god`, `developer`, `headadmin`,
`admin`, `mod`, `helper`) that `IsPlayerAceAllowed` grants, the same test
rsg-core's `HasPermission` makes; `perms.groups` lists every level held. A
player with none is `user`. The levels are granted in `server.cfg`
(`add_ace rsgcore.<level> <level> allow` plus `add_principal` lines); a server
without those lines has no admins as far as rsg-core or poggy_core can tell.
QBR is the same test over `QBConfig.Permissions` (`god`, `admin`, `mod`),
granted with `add_ace qbcore.<level> <level> allow`; the stock QBR
`server.cfg` template has the `qbcore.<level>` principals but a server must
add those `add_ace` lines itself, or everyone is `user`.

---

## RSG

The adapter (`server/adapters/rsg.lua`) is written against rsg-core 2.3.13 and
rsg-inventory 2.8.5, read from source, and proven in game on 14 September 2026
(`poggycore selftest full` 64/64). What it does differently from VORP is noted
section by section above; in one place:

| Area | On RSG |
|---|---|
| Character | `charId` is the `citizenid`; `ownerId` the Rockstar licence; `group` the ACE level |
| Money | `cash`, `bank`, `gold` (integer); money items are capacity-checked first; no `rol` |
| Jobs | duty and grade labels from `RSGShared.Jobs`; `Job.Set` refuses unknown jobs; `SetDuty` works; one job-change event, relayed only when name or grade changes |
| Inventory | `CanAddItem` / `AddItem` / `RemoveItem` / `GetItemCount`; `meta` matched by poggy_core on remove and count; `SetMeta` takes the slot as `itemId` |
| Weapons | a view over `type = 'weapon'` items; ids are serials; ammo and components not applied |
| Storage | rsg-inventory stashes; contents persist, definitions do not; single-occupancy; no permissions |
| Offline | `players` table (JSON `charinfo` and `job`); appearance from `playerskins` as `{ skin, clothes }` |
| Item registry | `RSGShared.Items`, no database; weapons included; images under `nui://rsg-inventory/html/images/` by the item's `image` field |
| Notifications | poggy_core's native renderer as everywhere; the `framework` renderer sends `ox_lib:notify` |
| Menus | `Core.Menu.Native()` returns ox_lib's `lib` when it is started |

Detection: `exports['rsg-core']:GetCoreObject()` must answer with a table that
has `Functions`. The core object is a copy (every export result is), so the
adapter re-fetches the player on every call and never writes to `PlayerData`.

---

## QBR

The adapter (`server/adapters/qbr.lua`) is written against qbr-core 1.0.3 and
qbr-inventory 1.0.1, read from source on the QBR test server, and **has not
yet run on a QBR server**. Until it has, treat every QBR line in this file as
a claim to be checked with `poggycore test` and `poggycore selftest full`. In
one place:

| Area | On QBR |
|---|---|
| Character | `charId` is the `citizenid`; `ownerId` the Rockstar licence; `group` the ACE level; gender from `charinfo.gender` (0/1) |
| Money | `cash`, `bank`; `gold` only when the server's `MoneyTypes` has it; no money items; no `rol`; every overdraw is `no_funds` |
| Jobs | duty and grade labels from `QBShared.Jobs`; `Job.Set` refuses unknown jobs and resets duty to the job's default; `SetDuty` works; no server event, so poggy_core re-checks after its own set and on a client poke; relayed only when name or grade changes |
| Inventory | qbr-core's `Player.Functions`; capacity computed here (no `CanAddItem`); `meta` matched by poggy_core on remove and count; `Remove` spans stacks; `SetMeta` takes the slot as `itemId`; `Close` runs the `closeinv` command |
| Weapons | a view over `type = 'weapon'` items; ids are the serial qbr-core stamps; ammo and components not applied |
| Storage | qbr-inventory stashes as `stashitems` rows, read and written directly; definitions kept by poggy_core and passed at open; open goes through the player's client; no permissions; `stashitems` must exist (poggy_core prints the CREATE if not) |
| Offline | `players` table (JSON `charinfo` and `job`); appearance from `playerskins` as `{ model, skin, clothes }` |
| Item registry | `QBShared.Items` through `GetItems()`, no database; weapons included; images under `nui://qbr-inventory/html/images/` by the item's `image` field, which is often not `name.png` |
| Notifications | poggy_core's native renderer as everywhere; the `framework` renderer sends `QBCore:Notify` (style 4, or 7 with a subtitle) |
| Menus | `Core.Menu.Native()` returns the `qbr-menu` export proxy when it is started |
| Escape hatch | `Core.Native()` is the `qbr-core` export proxy (there is no core object): `Core.Native():GetPlayer(src)` |

Detection: `exports['qbr-core']:GetPlayers()` and `GetItems()` must both
answer with a table. Every QBR API is a flat export on `qbr-core`; every
Player it returns is a copy, so the adapter re-fetches the player on every
call and mutates only through `Player.Functions.*`. Usable-item handlers live
in qbr-core, so that is the resource whose restart replays them.

Caps declared: `money.cash`, `money.bank` (each only if in `MoneyTypes`),
`money.gold` (only if in `MoneyTypes`), `char.onduty`, `job.registry`,
`job.duty`, `job.event`, `inventory.items`, `inventory.weapons`,
`inventory.metadata`, `inventory.registry`, `storage` (needs oxmysql,
qbr-inventory and the `stashitems` table), `storage.weapons`, `permissions`,
`char.offline` and `job.persist` (need oxmysql), `menu.native` (needs
qbr-menu). Not declared: `money.rol`, `inventory.carrycheck`,
`storage.persist`, `storage.permissions`.

---

## FiveM: ESX and QBCore (0.21.0)

Both adapters were written from the installed source on the FiveM test servers
and **neither has run on a live server yet**. Until they have, treat every line
below as a claim to check with `poggycore test` and `poggycore selftest full`.

The verb contract is identical on both games. What differs is what a framework
can actually do, and that is what `Core.Has(capability)` is for — a script that
asks before it acts works on all five frameworks without a branch.

### ESX

`server/adapters/esx.lua`, against es_extended 1.15.2 with esx_identity,
esx_addoninventory and esx_notify.

| Area | On ESX |
|---|---|
| Character | `charId` is `xPlayer.identifier`; with esx_multicharacter that is `char1:<licence>`, so the slot lives inside the string and there is no numeric character id. `ownerId` is the bare licence. Names come from esx_identity's variables, falling back to splitting the player name; gender from `sex` |
| Money | `cash` is the `money` account, `bank` is `bank`. No `gold`, no `rol`. `black_money` is deliberately not exposed as a currency. Every overdraw is `no_funds`: ESX itself does not check the balance and would go negative |
| Jobs | one job per character — **no multi-job of any kind**. `Job.Set` reads the job back, because ESX's `setJob` prints and returns for a job it does not know. Duty is `setJob`'s third argument and persists in `metadata.jobDuty`. The job list is cached at start, because `GetJobs` blocks |
| Inventory | weight only: no slots, and **no per-item metadata**, so `Inv.SetMeta` refuses. An item must already be a row in the `items` table or the add fails silently, so an unknown name answers `not_found`. `canCarryItem` is the carry check |
| Weapons | ESX's separate loadout, not items, so they never appear in `Inv.Get` |
| Storage | `esx_addoninventory` shared inventories. Reading and writing work; **`Storage.Open` refuses**, because stock ESX ships no stash UI at all. Counts are checked from the items table first, since `getItem` creates a zero row as a side effect and `removeItem` has no floor |
| Offline | the `users` table by identifier; first and last names exist only when esx_identity has added the columns |
| Item registry | `ESX.GetItems()`. **No item images**: stock ESX draws FontAwesome class names, so `Inv.ImageBase()` is nil |
| Notifications | poggy_core's native renderer as everywhere; the `framework` renderer goes through `xPlayer.showNotification` and never `esx_notify` directly, because ESX's own wrapper un-swaps the message and type arguments |
| Menus | `esx_menu_default` or `esx_context` when started |
| Escape hatch | `Core.Native()` is the ESX shared object: `Core.Native().GetPlayerFromId(src)` |

Detection: `exports['es_extended']:getSharedObject()` must answer a table with
`GetPlayerFromId`. Note ESX 1.15.2 also needs `esx_lib` started; without it the
core is half-working and poggy_core says so at start.

Caps declared: `money.cash`, `money.bank`, `char.onduty`, `job.registry`,
`job.duty`, `job.event`, `inventory.items`, `inventory.weapons`,
`inventory.carrycheck`, `inventory.registry`, `storage`, `storage.persist`,
`permissions`, `char.offline` and `job.persist` (need oxmysql), `menu.native`
(needs esx_menu_default or esx_context). Not declared: `money.gold`,
`money.rol`, `inventory.metadata`, `storage.permissions`, `storage.weapons`.

### QBCore

`server/adapters/qbcore.lua`, against qb-core 1.3.0 and qb-inventory 2.2.3.
It is the sibling of the QBR adapter, but not a copy: the FiveM stack has real
exports where QBR needed workarounds.

| Area | On QBCore |
|---|---|
| Character | `charId` is the `citizenid`; `ownerId` the licence; `group` the highest ACE level in the configured `Permissions` order; gender from `charinfo.gender` (0/1) |
| Money | `cash`, `bank`. No `gold` — the third type here is `crypto`, which is not a poggy currency. Every overdraw is `no_funds`, which is stricter than qb-core's own -5000 `MinusLimit` and makes `ok` mean the same thing on every framework |
| Jobs | grades are keyed by the stringified number and labels come from the shared table; `Job.Set` answers `not_found` for a job it does not know. `SetDuty` works. `QBCore:Server:OnJobUpdate` fires, so unlike QBR there is no client poke |
| Inventory | qb-inventory's exports. `CanAddItem` is a real carry check. `Inv.Remove` spans stacks because `RemoveItem` is single-slot; a failure part way has already taken what it took. `SetMeta` writes one key per call |
| Weapons | a view over `type = 'weapon'` items; the serial is `info.serie`, which qb-inventory stamps itself |
| Storage | qb-inventory stashes through `CreateInventory` / `OpenInventory` / `CloseInventory`. Capacity is **not** persisted — the `inventories` row holds items only — so poggy_core keeps the definitions and replays them before every operation, or qb-inventory refuses the write |
| Offline | `GetOfflinePlayerByCitizenId`; appearance is `{ model, skin }`, because FiveM's `playerskins` has no `clothes` column |
| Item registry | `GetShared('Items')`, read fresh each time because a cached core object goes stale after `AddItem`. Images under `nui://qb-inventory/html/images/` by the item's `image` field, which is often not `name.png` |
| Notifications | the `framework` renderer sends `QBCore:Notify` with a **variant name** (`success`, `error`, `warning`, `primary`), not QBR's numeric style; a title with a description becomes `{ text, caption }` |
| Menus | `qb-menu` when started |
| Escape hatch | `Core.Native()` is the qb-core core object: `Core.Native().Functions.GetPlayer(src)` |

Detection: `exports['qb-core']:GetCoreObject()` must answer a table with
`Functions`. What comes back is a copy, so the adapter re-fetches the player on
every call and mutates only through `Player.Functions.*`.

Caps declared: everything ESX declares, plus `inventory.metadata` and
`storage.weapons`. Not declared: `money.gold`, `money.rol`,
`storage.permissions`.

### What is not there yet

`Inv.UnregisterUsable` refuses on **both**: ESX has no deregistration at all,
and qb-core's `CreateUseableItem(key, nil)` is a no-op that leaves the old
handler running. A script that stops leaves its handler behind until something
overwrites it, which is why poggy_core re-registers unconditionally at start.

---

## Verbs added in 0.11.0

Scripts call these through `Poggy(verb, payload)` like every other verb. A
script that uses any of them, or a notification style added in 0.11.0, must
declare it in its `fxmanifest.lua`; see [Minimum poggy_core](#minimum-poggy_core).
"thread" means the verb waits on the framework or the database, so call it from
a thread or an event handler.

| Verb | Side | Payload | Value | On VORP |
|---|---|---|---|---|
| `players.onDuty` | server | `job?` (name or array), `minGrade?` | array of server ids | players whose `onDuty` is `true` (vorp_police `isOnDuty`, or the `isPoliceDuty` / `isMedicDuty` state bag); duty nobody can report counts as off |
| `char.offline` | server, thread | `charId`, `appearance?` | the character plus `online`; with `appearance = true` also `appearance = { skin, comps, tints }` | the live character when online, otherwise the `characters` row; `not_found` when there is none |
| `char.list` | server, thread | `search?`, `limit?`, `offset?` | array of `{ charId, ownerId, firstName, lastName, fullName }` | every `characters` row, by name; `search` matches the full name |
| `char.reloadSkin` | client | none | `true` | runs `PoggyCoreConfig.VorpReloadSkinCommand` (vorp_character's reload command, `rc` by default) |
| `job.set` with `persist = true` | server | `persist` | `true` | after the in-memory set, `UPDATE characters SET job, jobgrade, joblabel` (grade and label only when given). On a thread a failed write returns `false` with the job already set in memory; off a thread the write runs on its own and a failure is logged |
| `inv.items` | server, thread | `search?`, `limit?`, `checkImages?` | array of `{ name, label, desc, weight, limit, type, usable, group, image }`, plus `hasImage` with `checkImages` | `SELECT * FROM items`, read once when poggy_core starts and kept (items cannot change while the server runs); read again only after an `install.sql` seeds item rows. Callers asking during the read share it. Items only, no weapons |
| `inv.itemInfo` | server, thread | `item`, `checkImages?` | one item, as above | the same cache; `not_found` for an unknown item |
| `inv.imageBase` | both | none | a URL prefix | `nui://vorp_inventory/html/img/items/`; an icon is prefix .. name .. `.png` (RSG: `nui://rsg-inventory/html/images/`, QBR: `nui://qbr-inventory/html/images/`, both by the item's `image` field, which `inv.items` already resolves into `image`) |
| `inv.close` | server | `src` | `true` | `closeInventory(src)`: the player's own inventory, not a container |
| `weapon.canCarry` | server, thread | `src`, `qty?`, `weapon?` | boolean; `false` comes with err `no_space` | `canCarryWeapons(src, qty, cb, weapon)` |
| `storage.weapons` | server, thread | `id` | array of `{ id, name, label, serial, desc }` | `getCustomInventoryWeapons`; `not_found` for a container that is not registered. The id is namespaced like every storage verb |
| `callback.await` | client, thread | `name`, `args?` as `{ n = count, ... }` | packed results `{ n = count, ... }` | poggy_core's own callback transport; `timeout` after `RpcTimeout` |

`char.offline`, `char.list`, `inv.items`, `inv.itemInfo` and `job.set` with
`persist` read or write VORP's tables through oxmysql's exports. Without
oxmysql they refuse with `unsupported`; poggy_core does not depend on it. On
RSG the same is true of `char.offline`, `char.list` and `job.set` with
`persist` (the `players` table); `inv.items` and `inv.itemInfo` read
`RSGShared.Items` and need no database. On the standalone adapter every one of
these refuses with `unsupported`, and `players.onDuty` returns an empty list.

## Verbs added in 0.14.0

Drawn by poggy_core itself, so they behave the same on every framework; see
[Menu and input](#menu-and-input-0140) for the item shape and examples.
"thread" means the verb waits for the player, so call it from a thread or an
event handler.

| Verb | Side | Payload | Value | Notes |
|---|---|---|---|---|
| `menu.open` | both, thread | `title`, `items`; `src?` on the server, `subtitle?`, `cursor?`, `closeText?` | `{ value, index, item }` | `false, 'closed'` when the player backs out; on the server also `timeout` (after `Ui.Timeout`) and `not_found` (no such player) |
| `menu.close` | both | `src?` on the server | `true` | the waiting caller gets `false, 'closed'` |
| `input.text` | both, thread | `title`; `src?` on the server, `placeholder?`, `default?`, `maxLength?`, `numeric?`, `submitText?` | string, or number when `numeric` | `false, 'closed'` when cancelled |

## Verbs added in 0.16.0

| Verb | Side | Payload | Value | Notes |
|---|---|---|---|---|
| `inv.maxCarry` | server, thread | `src`, `item`, `cap?` (default 1000) | how many more of the item the player can hold, `0` to `cap` | Found from the adapter's own carry check by halving (about ten checks for a cap of 1000), so it follows the framework's rule: VORP's per-item `limit` together with the inventory, RSG's and QBR's weight and slots. `not_found` for an unknown item |
| `weapon.maxCarry` | server, thread | `src`, `weapon?`, `cap?` (default 1000) | how many more weapons (of that weapon, when given) fit | the same, from `weapon.canCarry` |

Use these rather than reading a framework's limit yourself. A script that reads
VORP's `limit` column is right on VORP and silently wrong on RSG and QBR, which
have no such column and cap by weight instead. A script calling them declares
`poggy_core_min '0.16.0'`.

## Verbs added in 0.18.0

| Verb | Side | Payload | Value | Notes |
|---|---|---|---|---|
| `jobs.list` | server, thread | | array of `{ name, label, grades = { { grade, label } } }`, sorted by name | RSG and QBR: the core's shared jobs table. VORP keeps a job as free text on the character, so it is the distinct jobs and grades in the `characters` table, with the name as the label (best effort; empty without oxmysql). Standalone: `{}`. The settings hub's job picker uses it. |

## Bans (0.19.0)

A ban is on the player's **account**, not the character: every identifier the
server gives for them (`license`, `license2`, `fivem`, `steam`, `discord`; never
`ip`) is hashed, and a connecting player is refused when any one of theirs
matches an active ban. Lifting a ban keeps the row, marked as lifted.

**It fails open.** The connect check reads memory only and never waits on the
database. No database, a failed load, an error of any kind: nobody is refused.
A fault in poggy_core never locks players out of a server.

Ban from the console or chat (`poggycore ban / unban / baninfo`), from `/poggy`
(poggy_core, **Bans**), or from a script through the verbs. Tables `poggy_bans`
and `poggy_ban_hashes` are created on start. Config: `PoggyCoreConfig.Bans`.

| Verb | Side | Payload | Value | Notes |
|---|---|---|---|---|
| `ban.add` | server, thread | `reason`, and `src` or `identifiers`; optional `category` (`"local"`, `"cheat"`), `duration` (seconds; absent = permanent), `evidence` (http/https), `name`, `by` (staff server id, or text), `byName`, `source`, `sourceRef` | ban id | Drops everyone online on that account. |
| `ban.remove` | server, thread | `id`, `reason`; optional `by`, `byName` | `true` | The row stays. |
| `ban.check` | server | `src` or `identifiers` | `{ banned, ban?, network = { count, blocked } }` | Memory only. `network` is always zero until the shared ban network ships. |
| `ban.list` | server, thread | optional `search`, `active`, `limit`, `offset` | array of bans, newest first | No raw identifiers, only hashes. |
| `player.kick` | server | `src`; optional `reason` | `true` | |
| `player.identifiers` | server | `src` | `{ name, identifiers, hashes }` | What a report stores so a player can be banned after they leave. |

**These verbs do not check who is asking.** The script that calls them decides
who may ban. The console command and the hub panel check it themselves.

Events (server): `poggy_core:ban:added`, `poggy_core:ban:removed`, each with the ban.

Scripts that use these need `poggy_core_min '0.19.0'`.

### The ban network (0.20.0)

On by default, opt-out (`PoggyCoreConfig.Bans.Network.Enforce` / `.Submit`). A
**cheating** ban is one vote; a player with votes from 3 counting servers is
refused wherever the network is enforced. Local bans never leave the server.
Only hashes and a player count are sent. The list is a public file of
`hash → count`, read every ten minutes and held in memory, so a connect never
waits on the internet; a failed read keeps the last list, and with no list
nobody is refused. `poggycore bannet` shows the status; `poggycore banallow
<identifier> <reason>` lets one blocked player in and counts as a voice for
them. `ban.check` now fills `network = { count, blocked }`. Server event
`poggy_core:ban:networkFlag` (`{ src, name, count, blockAt }`) fires when
someone with 1+ votes, not blocked, joins.

## Fixed in 0.17.1: admins on the user record

`perms.isAdmin` used to read only the character's group. VORP keeps admin on
the **user** record, so a server owner who was plainly an admin was refused
by their own admin commands (Balloon's spawn command, Supply Drops' admin
commands, and any script that asks `perms.isAdmin`). It now checks every
group poggy_core can see: the character's, the framework's user object, and
the adapter's permission levels (RSG's ACE levels). The recognised admin
groups are unchanged: admin, superadmin, god, owner, headadmin, developer.
`perms.groups` and `Core.Perms.Groups(src)` return that same list.

## Verbs added in 0.17.0: providers

A **provider** is a Poggy script that supplies something the framework lacks
and registers once at start; poggy_core then routes the matching verbs to it
(`server/sv_providers.lua`). Without one nothing changes.

| Verb | Side | Payload | Value | Notes |
|---|---|---|---|---|
| `bank.register` | server | `fns` = `{ get(src), add(src, amount, reason), remove(...), set(...) }` | `true` | From then on `money.get/add/remove/set` with `currency = 'bank'` go to the provider on every framework, and `money.supports 'bank'` is true even on VORP. Poggy Banking is the provider; it keeps accounts per branch and pays a wage into the character's home branch. |
| `treasury.register` | server | `fns` = `{ collect, index, rates, state, disburse, balance, report }` | `true` | One provider per kind; a second registration while the first runs is refused by name. |
| `treasury.collect` | server | `kind`, `amount`, `src?`, `charId?`, `source?`, `meta?` | `true` | A levy or fee lands in the treasury. `unsupported` when none is installed, so a script deletes the fee as it always did. |
| `treasury.index` | server | | the price index, `1.0` = normal | Markets and the Auction House multiply their own prices by it, defaulting to `1.0` when refused. |
| `treasury.rates` | server | | `{ withdrawalLevy, transferFee, depositInterest, loanRate, salesTaxAdjust }` | |
| `treasury.state` | server | | `{ verdict, event, index, supply, reserveWeeks, warmup }` | |
| `treasury.disburse` | server | `account`, `amount`, `src`, `reason?` | `true` | A department account pays a player. |
| `treasury.balance` | server | `account` | number | |
| `treasury.report` | server | `metric`, `value`, `source?`, `meta?` | `true` | Activity, fire-and-forget: `shop_sale`, `auction_sale`, `order`… The treasury builds a per-server baseline from it. |

Every provider function answers the verb contract (`ok, value, err`) and runs
under `pcall`; one that throws is named once and the caller sees
`framework_error`. A script calling these declares `poggy_core_min '0.17.0'`.

---

## Minimum poggy_core

A script that calls something added in a later poggy_core says so:

```lua
-- fxmanifest.lua
poggy_core_min '0.11.0'
```

The version that counts is the one **running**. The updater installs
poggy_core last and never restarts it, so the files on disk can be newer than
the core in memory. Two things enforce the requirement:

- **The bridge** (`template/poggy.lua`) asks the running core for its version.
  If it is older, the resource prints one red line at boot, `PoggyReady()`
  returns `false`, and every `Poggy()` call returns `false, nil, 'core_too_old'`
  (each verb is named in the console the first time it is refused). Nothing
  half-works. `PoggyWriteOwnFile` keeps working, so the update can still land.
- **The updater** installs such a resource but does not restart it, and says
  `installed, not restarted — needs poggy_core X and vY is running`, with how to
  make it take effect: restart the server, or run `refresh`, `ensure poggy_core`,
  `ensure <resource>`.

Versions compare as numbers, so `0.10.0` is newer than `0.9.0`.

## Updates

### Automatic updates (on by default)

Out of the box (`PoggyCoreConfig.Updates` in `config.lua`: `AutoUpdate = true`,
`ApplyOnStart = true`), poggy_core checks the feed once, when the server
starts (0.22.0: the hourly check while running is gone; `poggycore update all
apply` installs updates without a restart). It does this for every Poggy script whose published
`fxmanifest.lua` version is higher than the local one. Same or older is never
touched. For each such script it:

1. installs the new files, merging config and translation files: settings the
   update adds are added, settings it removes are cleaned up, and the owner's
   values are never changed;
2. backs up every original it replaced in `poggy_core/update_backups/`;
3. runs `refresh` once and restarts each updated script (`RestartUpdated`).

The restarts happen immediately, at server start. poggy_core never restarts itself: its own
update takes effect on the next server restart (or `refresh` then
`ensure poggy_core` by hand). Escrowed scripts update the same way; nobody
re-downloads from the portal for an update.

Automatic restarts need two lines in `server.cfg`. poggy_core prints them if
they are missing:

```cfg
add_ace resource.poggy_core command.refresh allow
add_ace resource.poggy_core command.ensure allow
```

To review updates before they go in, turn both switches off:

```lua
AutoUpdate   = false,
ApplyOnStart = false,
```

poggy_core then only lists what is available in the console. Install with
`poggycore update all apply`, or one script with `poggycore update <name> apply`.

### Script ids and folders

The update feed knows scripts by `poggy_id` (see [Script identity](#script-identity)).
For each id the updater finds the folder on this server: the folder that
registered the id, or, for a script that has not registered (stopped, or still
waiting for its framework), the folder whose `fxmanifest.lua` declares it, read
from the file rather than the cached metadata. Everything local uses that
folder: its state and version, the files and the `PoggyWriteOwnFile` bridge that
writes them, the backups in `update_backups/`, and `refresh` / `ensure`. Console
lines show the id, plus `(folder: my_scene)` when the folder is different.

`poggycore update <name>` takes the id or a folder name. An id no folder
declares is "not on this server". An id two folders declare is updated in
neither, with a red line naming both, until one of them is gone.

A script is published under its release folder name, so that name, its `Name`
in the release list and its `poggy_id` must all be the same.

### Development servers

The server the scripts are written on is the source of every update, so an
update applied there would overwrite source with escrowed builds. Put

```
set poggy_dev_server 1
```

in its `server.cfg`. Every writing path — `apply`, `stage`, `writetest`, and
the automatic runs — then refuses before touching anything, with one yellow
line per run: `this server is marked as a development server ... updates are
reported but never written`. `check` and `nettest` still work, the start-up
check still lists what is newer, and the banner reads `dev server: updates
read-only`. It is a convar rather than a `config.lua` setting on purpose:
`config.lua` ships to customers and the merge keeps an owner's values, so a
flag in the development copy would be carried into every release.

### .fxap files

`.fxap` files are per-customer escrow licence tokens. The updater never
downloads, compares, writes, backs up or deletes one, in any resource. A feed
entry for one is skipped with a single grey `ignored .fxap in feed` line, and
`PoggyWriteOwnFile` refuses any path ending in `.fxap`.

### The catalogue (0.15.0)

The feed's `index.json` lists every published Poggy script. After the start-up
check, poggy_core compares that list with the scripts on this server (an id is
installed when a folder registered it or declares it in its manifest, the same
lookup the updater uses) and prints the rest once, as one grey and blue block:

```
[Poggy Core] Other Poggy scripts (not installed here):
[Poggy Core]    Poggy Markets                https://rosewoodridge.xyz/store/poggy-markets
[Poggy Core]    Poggy Fishing                https://rosewoodridge.xyz/store/poggy-fishing
[Poggy Core]    Poggy Admin Blips (free)     https://rosewoodridge.xyz/store/poggy-admin-blips
[Poggy Core]    … see https://rosewoodridge.xyz/store
```

It is a notice, not an advert: never red or yellow, at most ten rows and then
`and N more`, nothing at all when every published script is installed, once per
boot, on every server including a development server.
`PoggyCoreConfig.Updates.ShowCatalog = false` turns it off; `poggycore catalog`
prints it on demand, reading the feed afresh.

**Feed fields.** Each entry in `index.json` (`resources[<poggy_id>]`) carries
`version` and `files`; from 0.15.0 the updater also reads three optional fields
a publish may add, each with a fallback:

| Field | Meaning | When missing |
|---|---|---|
| `label` | the display name, e.g. `"Poggy Markets"` | the id is shown |
| `store` | the absolute product page, e.g. `https://rosewoodridge.xyz/store/poggy-markets` (must start with `http`) | the store front, `https://rosewoodridge.xyz/store` |
| `free` | `true` for a free script: `(free)` is shown after the name | not free |

Rows are sorted by label.

## Database (0.12.0)

Every Poggy script with tables keeps them in exactly one file, `sql/install.sql`.
The bridge runs it through poggy_core the first time `PoggyReady()` is called on
the server (the boot report always does), and `PoggyReady()` waits until it is
done, so a script's tables exist before its first query, even on a first-ever
start. Server owners import nothing.

The file is written to run on every start, on MariaDB and MySQL 8:

| Statement | What poggy_core does |
|---|---|
| `CREATE TABLE IF NOT EXISTS` | creates the table when it is missing |
| `ALTER TABLE t ADD COLUMN c …` | adds it only when missing (write it **without** `IF NOT EXISTS`) |
| `ALTER TABLE t ADD INDEX name (…)`, `CREATE INDEX name ON t (…)` | adds it only when missing; indexes need a name |
| `ALTER TABLE t MODIFY COLUMN c TYPE …` | runs only when the column's type differs |
| `INSERT IGNORE …` | runs every start; existing rows are never changed |
| `UPDATE … WHERE …` | runs every start; the WHERE must make it repeatable |
| `SELECT …` | ignored |

It checks `information_schema` before each ADD, so a normal start sends no DDL and
raises no database error (oxmysql reports every failed query to error loggers).
Anything that cannot repeat safely (DROP, RENAME, DELETE, a plain INSERT,
`ON DUPLICATE KEY UPDATE`, other ALTER clauses) is refused with a yellow line and
belongs in a migration: `sql/migrations/001.sql`, `002.sql`, … Each runs once,
in order, after `install.sql`, recorded in the `poggy_migrations` table; a failed
one is not recorded and runs again next start. Since 0.13.0 a migration is
recorded under the script's `poggy_id`, so renaming the folder does not run it
again; rows recorded under the current folder name still count. A migration may begin with
`-- poggy: only-if-table <name>` to count as done, without running, when that
table does not exist.

The same line works on a single statement, in `install.sql` as well as in a
migration: written on the comment line before a statement, it runs that
statement only when the table exists and otherwise skips it, quietly. Use it
for rows a script seeds into a table the *framework* owns, such as VORP's
`items` or `characters`, which RSG does not have (its items live in
`rsg-core/shared/items.lua`); put those statements last so the script's own
tables come first. Skips are counted in grey on the summary line, e.g.
`created 2 tables, skipped 3 statements (no items table)`; a start with nothing
else to do stays silent.

A normal start with nothing to do prints nothing. Changes print one green line,
e.g. `poggy_scene  database: created 2 tables`; problems print in red or yellow.

```
PoggyCoreConfig.Sql.AutoInstall = false   -- run nothing automatically
poggycore sql check <resource|all>         -- console: what would change
poggycore sql install <resource|all>       -- console: run it now
```

Scripts using it declare `poggy_core_min '0.12.0'`. Against an older running core
the bridge skips the install rather than calling a verb that core does not have.

---

## Capabilities

```lua
if Core.Has('money.gold') then ... end
```

| Capability | Meaning |
|---|---|
| `money.cash` `money.bank` `money.gold` `money.rol` | Currency is supported |
| `char.onduty` | The framework can answer "is this player on duty" |
| `job.registry` | A shared jobs table with labels and grades exists |
| `job.duty` | Duty can be set |
| `job.event` | The framework emits a job-change signal of its own |
| `inventory.items` `inventory.weapons` `inventory.metadata` | |
| `inventory.carrycheck` | Native capacity check, versus one we compute |
| `storage` | Containers are supported |
| `storage.persist` | Container **definitions** survive a restart (false on VORP) |
| `storage.permissions` `storage.weapons` | |
| `menu.native` | A framework menu object is reachable |
| `permissions` | |
| `char.offline` | Characters can be read while offline (`char.offline`, `char.list`) |
| `job.persist` | `job.set` can write the job to the database |
| `inventory.registry` | The item registry is readable (`inv.items`, `inv.itemInfo`) |

Error codes returned as the second value: `not_ready`, `no_char`, `unsupported`,
`not_implemented`, `bad_argument`, `no_space`, `no_funds`, `not_found`,
`timeout`, `framework_error`, `needs_thread`. They are on `Core.Err`. The
bridge adds `no_core` and `core_too_old` (see [Minimum poggy_core](#minimum-poggy_core)).

---

## Diagnostics

```
/poggycore          framework, version, capabilities, registered containers
/poggycore test     read-only smoke test against your own character
/poggycore caps     the full capability map
/poggycore scripts  registered Poggy scripts: poggy_id, folder (when different), version
/poggycore usables  usable items registered through poggy_core: item → script
/poggycore dependents  resources that stop with poggy_core, their state, and the restart record
/poggycore catalog  published Poggy scripts this server does not have, with store links
/poggycore settings [show <id>]  the settings hub from the console: scripts, lock holders, every setting (see Settings hub)
```

Admin-gated in game; available unrestricted from the server console. The smoke
test modifies nothing.

---

## What is not built yet

Honest list. Phase 0 was scoped to everything except menus.

| Area | Status |
|---|---|
| `Core.Menu.Open` / `Core.Input` | **Built in 0.14.0** (`menu.open`, `menu.close`, `input.text`); a list menu and a text box. Sliders, grids, tick boxes and item images from `vorp_menu` are not abstracted; `Core.Menu.Native()` still hands over the framework's own menu for those. |
| RSG adapter | **Proven in game (14 September 2026).** `poggycore selftest full` passes 64/64 on the RSG test server (rsg-core 2.3.13 / rsg-inventory 2.8.5); every Poggy script starts and its menus, shops, storage and auctions work there. |
| QBR adapter | **Proven in game (14 September 2026).** 0.14.1 against qbr-core 1.0.3 / qbr-inventory 1.0.1; `poggycore selftest full` passes on the QBR test server and every Poggy script runs there. |
| RedEM:RP, RPX | **Not supported, not planned.** Detection still names them so `Core.HasAdapter()` is false and every framework verb refuses honestly. |
| `Core.Job.SetDuty` on VORP | Unsupported. `vorp_core` has no duty concept and `vorp_police` exposes no setter. |
| `money.bank` on VORP | Unsupported by `vorp_core` itself. **Since 0.17.0** a bank provider (Poggy Banking, `bank.register`) answers for `'bank'` on every framework, VORP included. |
| Client prompt natives | Written but **not yet verified in game.** Nothing consumes `Core.Prompt` yet, so the risk is contained; test before relying on it. |
| Kind-specific notification styling | Notifications render, but `kind` does not yet change icon or colour. Icon dictionaries differ per framework and were not guessable from source alone. |
| VORP store window | Not abstracted. vorp_inventory's store window (the `syn_store` events) is used directly by the scripts that need it. |
| Weapons in `inv.items` | Not included on VORP; the registry lists items only. Included on RSG, where weapons are items (`type = 'weapon'`). |

Nothing outside this folder has been modified. No existing script calls
poggy_core yet; that is Phase 1.

---

## Who uses it

Phase 1 retargeted the three resources that had already written their own
bridges. Each keeps its public bridge API exactly as it was, so nothing else in
those resources changed; the internals now prefer poggy_core and fall back to
their original adapters for any framework poggy_core cannot drive.

| Resource | Routed through poggy_core | Still on its own path |
|---|---|---|
| `poggy_markets` | character, money, jobs, items, weapons, usable items | notifications (keeps its objective-banner style) |
| `poggy_character_storage` | character, group, money, containers, items, weapons | item icons, the VORP store window |
| `poggy_auction` | character, money, items, callbacks | notifications |
| `poggy_witnesses` | character, jobs, duty, players, weapons, identifiers | its own notification overrides |
| `poggy_supplydrops` | character, money, items, weapons | notifications |
| `poggy_fishing_pack` | character id, items, capacity checks, notifications | — |
| `poggy_multijob` | admin groups, notifications | job persistence (writes VORP's `characters` columns; see below) |
| `poggy_trashbins` | containers, items, groups, names, notifications | — |
| `poggy_balloon` | money, character presence | the menu (`vorp_menu`; `menu.open` exists since 0.14.0 and the script has not moved yet) |
| `poggy_badge` | character | — |
| `poggy_transform` | framework, jobs, admin groups | — |
| `poggy_scene` | character id, notifications | — |
| `poggy_admin_blips` | groups, names, notifications | — |

poggy_core is now a hard dependency of every one of them
(`dependency 'poggy_core'`), so the fallback code kept from the migration can no
longer run and is being removed. Each prints one boot line saying whether
poggy_core is ready.

Two things were deliberately **not** migrated. `poggy_multijob` writes VORP's
`characters` table directly when switching jobs, because VORP's own setters
only change in-memory state and other resources read the database; poggy_core
has no concept of persisting a job yet, so abstracting half of it would leave a
hybrid that is harder to follow than either version. And every resource keeps
its own notification *style* — see below.

---

## Adding a framework

One file. Copy `server/adapters/standalone.lua`, which is the interface
specification, fill in every method it lists, declare a `caps` table, and add the probe
to `shared/sh_detect.lua`. The dispatcher calls those method names and nothing
else, so an adapter cannot silently miss one; `/poggycore caps` will show what
you declared.

`server/adapters/vorp.lua` is the worked example, and the comments at the top of
it list the framework traps that shaped the interface. `server/adapters/rsg.lua`
is the second one, written the same way; its header lists the RSG traps, with
the rsg-core and rsg-inventory line numbers each was read from.
`server/adapters/qbr.lua` is the third, and shows what to do when a framework
has no core object, no capacity check and no storage exports: compute what is
missing, go to the table the framework itself uses, and declare only the caps
that are true.

An adapter may also define `permGroups(src)` (an array of every group the
player holds, highest first); `perms.groups` adds them when it is present.
