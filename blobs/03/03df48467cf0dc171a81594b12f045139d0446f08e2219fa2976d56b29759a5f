# poggy_core

One documented framework API for RedM. Write a script once; run it on VORP, RSG
Core, QBCore RedM, RedEM:RP or RPX.

**Version 0.13.1.** The VORP adapter is complete and every Poggy resource runs
on it through `Poggy(verb, payload)`; see [Verbs added in 0.11.0](#verbs-added-in-0110).
Scripts are known by their `poggy_id`, so a server owner may rename any
script's folder; see [Script identity](#script-identity).
poggy_core is a hard dependency of every script, and it no longer needs
poggy_util for anything: it draws notifications itself. The other four
frameworks are detected but fall back to standalone, which refuses every
framework call honestly rather than pretending to succeed, and reports
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

# lets poggy_core restart the Poggy scripts it has just updated
add_ace resource.poggy_core command.refresh allow
add_ace resource.poggy_core command.ensure allow
```

No database import. Automatic updates are on from the moment you install (see
Updates); the two `add_ace` lines let poggy_core restart what it updated, and it
prints them in the console if they are missing. Every value in `config.lua` has a
working default.

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
| `gold` | yes | yes | no | **no** | **no** |
| `rol` | yes | no | no | no | no |

`Remove` refuses and returns `false, 'no_funds'` rather than taking a player
negative, on every framework, including the ones that would allow it.

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

Which jobs count as law or medical is in `config.lua`, not in code.

### Inventory — server

```lua
Core.Inventory.CanCarry(src, item, qty)     --> boolean, reason
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

Weapons are a separate namespace, deliberately: only VORP and RSG model them as
first-class objects.

```lua
Core.Weapons.Get(src) / Core.Weapons.Add(src, name, ammo, comps) / Core.Weapons.Remove(src, id)
```

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

### Permissions — server

```lua
Core.Perms.GetGroup(src)   --> string
Core.Perms.IsAdmin(src)    --> boolean
```

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
| `inv.items` | server, thread | `search?`, `limit?`, `checkImages?` | array of `{ name, label, desc, weight, limit, type, usable, group, image }`, plus `hasImage` with `checkImages` | `SELECT * FROM items`, cached for `ItemCacheSeconds` and dropped when vorp_inventory restarts. Items only, no weapons |
| `inv.itemInfo` | server, thread | `item`, `checkImages?` | one item, as above | the same cache; `not_found` for an unknown item |
| `inv.imageBase` | both | none | a URL prefix | `nui://vorp_inventory/html/img/items/`; an icon is prefix .. name .. `.png` |
| `inv.close` | server | `src` | `true` | `closeInventory(src)`: the player's own inventory, not a container |
| `weapon.canCarry` | server, thread | `src`, `qty?`, `weapon?` | boolean; `false` comes with err `no_space` | `canCarryWeapons(src, qty, cb, weapon)` |
| `storage.weapons` | server, thread | `id` | array of `{ id, name, label, serial, desc }` | `getCustomInventoryWeapons`; `not_found` for a container that is not registered. The id is namespaced like every storage verb |
| `callback.await` | client, thread | `name`, `args?` as `{ n = count, ... }` | packed results `{ n = count, ... }` | poggy_core's own callback transport; `timeout` after `RpcTimeout` |

`char.offline`, `char.list`, `inv.items`, `inv.itemInfo` and `job.set` with
`persist` read or write VORP's tables through oxmysql's exports. Without
oxmysql they refuse with `unsupported`; poggy_core does not depend on it. On the
standalone adapter every one of these refuses with `unsupported`, and
`players.onDuty` returns an empty list.

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
`ApplyOnStart = true`, `CheckIntervalMinutes = 60`), poggy_core checks the feed
when the server starts and every hour. It does this for every Poggy script whose published
`fxmanifest.lua` version is higher than the local one. Same or older is never
touched. For each such script it:

1. installs the new files, merging config and translation files: settings the
   update adds are added, settings it removes are cleaned up, and the owner's
   values are never changed;
2. backs up every original it replaced in `poggy_core/update_backups/`;
3. runs `refresh` once and restarts each updated script (`RestartUpdated`).

At server start the restarts happen immediately. Updates installed while players
are online are written at once, but their restarts wait until the server is
empty (`RestartWhenEmptyOnly`). poggy_core never restarts itself: its own
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
```

Admin-gated in game; available unrestricted from the server console. The smoke
test modifies nothing.

---

## What is not built yet

Honest list. Phase 0 was scoped to everything except menus.

| Area | Status |
|---|---|
| `Core.Menu.Open` / `Core.Input` | **Not implemented.** Returns `false, 'not_implemented'` and logs once. Use `Core.Menu.Native()` for the framework's own menu handle in the meantime. |
| RSG, QBR, RedEM, RPX adapters | **Not written.** Detection knows about them, and `Core.HasAdapter()` returns false so a consumer can keep its own path; without an adapter the core falls back to standalone and refuses framework calls. |
| `Core.Job.SetDuty` on VORP | Unsupported. `vorp_core` has no duty concept and `vorp_police` exposes no setter. |
| `money.bank` on VORP | Unsupported. `vorp_core` genuinely has no bank. |
| Client prompt natives | Written but **not yet verified in game.** Nothing consumes `Core.Prompt` yet, so the risk is contained; test before relying on it. |
| Kind-specific notification styling | Notifications render, but `kind` does not yet change icon or colour. Icon dictionaries differ per framework and were not guessable from source alone. |
| VORP store window, menus | Not abstracted. vorp_inventory's store window (the `syn_store` events) and `vorp_menu` are used directly by the scripts that need them. |
| Weapons in `inv.items` | Not included; the registry lists items only. |

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
| `poggy_balloon` | money, character presence | the menu (`vorp_menu`, until `Core.Menu` exists) |
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
it list the framework traps that shaped the interface.
