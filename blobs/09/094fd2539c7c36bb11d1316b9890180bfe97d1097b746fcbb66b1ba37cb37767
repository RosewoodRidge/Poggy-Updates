# Witnesses

Every crime has a witness. Stop them before they reach the law.

When a player commits a crime in town, a nearby NPC sees it, runs, and tries
to report it. If the witness gets away, the law gets an alert with a blip and
a waypoint. Catch the witness first and nobody hears about it.

---

## What it does

- **Witnesses.** An NPC near the crime becomes a witness. They flinch, point
  at you and shout (or put their hands up if you are armed), then run. They
  report once they get far enough away and stay away long enough. NPCs that
  belong to other scripts (shopkeepers, job NPCs) are left alone.
- **Crimes it notices.** Shooting and shots near people, threatening someone
  with a weapon, fist fights and melee, lassoing, trampling, stealing a horse
  or wagon, riding with a hogtied hostage, looting a body and poaching. Each
  can be switched on or off, with its own chance.
- **Law alerts.** On-duty law gets an alert with what happened, where ("In
  Valentine") and when, a blip, an area circle and an optional waypoint. The
  same crime by the same player alerts once a minute at most.
- **Duty from your law script.** Presets for outsider_policeman, vorp_police,
  rsg-lawman, qbr-policejob and bcc-law, picked automatically, plus your
  framework through poggy_core and a custom option.
- **Player alert commands.** `/callpolice`, `/calldoctor`, `/undertaker`,
  `/calltrain` and any you add. Each has its own jobs, grades, blip and
  cooldown.
- **Alerts for other scripts.** Robbery, break-in, bounty, drug sale and more,
  fired by your other scripts.
- **Blips that wait.** An alert can keep its blip until the responder arrives.
- **Where it works.** Only inside the towns you list, or everywhere; with
  blacklisted areas (Sisika, Van Horn, Wapiti by default) where nothing is
  noticed.
- **NPC law response (optional).** When no lawman is on duty, a posse of NPC
  lawmen rides in from out of sight for the crimes you choose, sized by how
  serious the crime is. They call out and hold you at gunpoint first, fight
  only you, chase on horseback or on foot, and leave and vanish out of sight
  when it is over. Players can fight, flee or surrender.
- **Law immunity.** Witnesses can ignore crimes by lawmen on duty.
- **Three languages.** English, Spanish and French.

## Requirements

| Resource | Why |
|---|---|
| **poggy_core** 0.14.0 or newer | Required. Talks to your framework (VORP, RSG or QBR). |
| **PolyZone** | Required. The town zones. https://github.com/mkafrin/PolyZone |
| A duty script | Optional. For duty checks: outsider_policeman, vorp_police, rsg-lawman, qbr-policejob, bcc-law, or any other through the custom option. |
| A jail script with a server export | Optional. NPC law arrests can use it, or Witnesses' built-in Sisika jail (off by default). |

Witnesses never talks to your framework directly. Everything goes through
poggy_core.

## Installation

1. Install and start **poggy_core** and **PolyZone**.
2. Put the `poggy_witnesses` folder in your `resources` folder.
3. Add `ensure poggy_witnesses` to `server.cfg`, after poggy_core and PolyZone.
4. Restart the server.

### Upgrading from the old `Witnesses` folder

From 1.2.0 the folder is called `poggy_witnesses`.

1. Copy your settings out of the old `config.lua` and `config_npc.lua` (and
   `translations.lua` if you changed any text).
2. Delete the old `Witnesses` folder and put `poggy_witnesses` in its place.
3. In `server.cfg`, change `ensure Witnesses` to `ensure poggy_witnesses`.
4. Put your settings back. `config.lua` is now `config/config.lua`,
   `config_npc.lua` is now `config/npc.lua`, and `translations.lua` stays next
   to `fxmanifest.lua`.

If another script uses the Witnesses API, update it too:

- `exports.Witnesses:...` becomes `exports.poggy_witnesses:...`
- the server events `witnesses:onWitnessCreated` (and the others) become
  `poggy_witnesses:onWitnessCreated` and so on.

The alert commands and every setting are unchanged.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/callpolice` | Everyone | Calls the law to your position. 120 s cooldown. |
| `/calldoctor` | Everyone | Calls a doctor. The blip stays until they arrive. 120 s cooldown. |
| `/undertaker`, `/alertundertaker` | Everyone | Calls the undertaker. |
| `/calltrain` | Everyone | Asks the conductor for a pickup. |
| `/clearalerts` | Everyone | Removes all your alert blips and your waypoint. |
| `/cleanalertblips` | Everyone | Removes all your alert blips. Keeps your waypoint. |
| `/clearwaypoint`, `/clearmarker` | Everyone | Clears your waypoint and GPS route. |
| `/getmyweaponhash_cl` | Everyone | Prints the weapon you hold to the F8 console (diagnostic). |
| `/witnesscheck` | Everyone | Prints to F8 every check that decides whether a crime where you stand would be witnessed (diagnostic). |
| `/witnessduty` | ACE `command.witnessduty`, console | Shows which duty script is read and every lawman's duty state. |
| `/witnesstestalert [crime]` | ACE `command.witnesstestalert`, console (`witnesstestalert <id> [crime]`) | Sends one real crime alert (Shooting by default) to your own screen only, as an on-duty lawman gets it: popup, blip, area, route. No posse, nobody else alerted. |
| `/jail <id\|me> <minutes> [reason]` | ACE `command.jail`, console | Sends a player to the built-in Sisika jail (needs it on; at most `MaxMinutes`). `me` jails yourself. |
| `/unjail <id\|me>` | ACE `command.unjail`, console | Releases a player from the built-in jail at once. |
| `/dumpalertgroups` | ACE `command.dumpalertgroups`, console | Lists every player's job, whether it is law and whether they are on duty. |
| `/debugalertsgroups` | ACE `command.debugalertsgroups`, console | The same list, printed to the server console. |

The alert commands are set in the **Job alerts** list, so the names above are
the ones it ships with. The other alerts in that list (robbery, break-in,
bounty and so on) have odd names on purpose. They are meant to be fired by
your other scripts. Anyone who knows a name can type it, so keep them odd.

The crime alerts (`witness_shooting_alert` and the others) are fired by the
script itself when a witness reports.

---

## Configuration

**Every setting can be changed in game with `/poggy`**, with a description of
each one. Changes need a restart of the script, which the hub offers.

The files, if you prefer to edit them by hand:

| File | What it holds |
|---|---|
| `config/config.lua` | Witnesses, crimes, towns, law jobs, duty, alert blips, job alerts, ignored items and NPCs |
| `config/npc.lua` | The optional NPC law response |
| `translations.lua` | The language, and every message in English, Spanish and French |

### Highlights

| Setting | What it does |
|---|---|
| `Config.Checks` | Which crimes are noticed, and the chance for each. |
| `Config.MaxActiveWitnesses` | How many witnesses can flee at once. |
| `Config.WitnessEscape` | How far a witness must run, and how long they need to report. |
| `Config.EnforceTownLimits` | `true`: only inside the towns in `Config.Towns`. `false`: everywhere. |
| `Config.BlacklistedTowns` | Areas where nothing is ever noticed. |
| `Config.PoliceJobList` | Your law jobs. Exact names, case-sensitive. |
| `Config.PoliceJobRanks` | The grades of each law job that get alerts. |
| `Config.DutyCheckEnabled` | Only alert law jobs who are on duty. |
| `Config.Duty.Preset` | Which duty script says who is on duty. `"auto"` finds it. |
| `Config.AlertCooldown` | Seconds before the same crime by the same player alerts again (60). |
| `Config.Alerts` | Every alert and alert command. |
| `Config.LawResponse.Enabled` | Turns the NPC law response on. Off by default. |

### Duty checks

With `Config.DutyCheckEnabled = true`, a player with a law job only gets law
alerts while on duty, and the NPC law response (with `PlayerLEOPriority`) only
rides out while nobody is. An alert with `overrideDutyCheck = true` ignores
duty. Other jobs, such as doctors or the undertaker, are always alerted.

`Config.Duty.Preset` says where the answer comes from:

| Preset | Duty script | How it knows |
|---|---|---|
| `"auto"` | the first one found running, in the order below | |
| `"outsider_policeman"` | outsider_policeman | its `IsOnPoliceDuty` export (state bag `IsOnDuty` as a back-up) |
| `"vorp_police"` | vorp_police | the `isPoliceDuty` state bag |
| `"rsg_lawman"` | rsg-lawman | rsg-core's job duty, through poggy_core |
| `"qbr_policejob"` | qbr-policejob | qbr-core's job duty, through poggy_core |
| `"bcc_law"` | bcc-law, bcc-society | off duty is a job renamed `off<job>`, so a law job is on duty |
| `"framework"` | none | poggy_core's answer (VORP: vorp_police / vorp_medic; RSG and QBR: job duty) |
| `"job_only"` | none | everyone with a law job counts as on duty |
| `"custom"` | yours | `Config.Duty.Custom`: a server export `(source, job) -> true/false` and/or a state bag |

When the duty script cannot say (none running), `Config.Duty.Fallback` decides:
`"job_only"` (the default) counts every law job as on duty, `"off_duty"` counts
nobody. `/witnessduty` shows the preset in use and each lawman's state.

Any other duty script can tell Witnesses directly, with no preset:

```lua
exports.poggy_witnesses:SetOnDuty(source, true)   -- or false; nil hands it back to the preset
```

The older `Config.DutyExportCall` (a Lua string that returns `true` when on
duty; `playerid`, `job`, `charId`, `identifier` and `jobGrade` are replaced)
still works while the preset is `"auto"` or `"custom"`. It runs as code, so it
can only be changed in the file.

**Grades.** A lawman only gets an alert if their grade is in that alert's
grade list (`Config.PoliceJobRanks` for the shipped law alerts). If your law
job's grades start at 1, or go above 5, add them there.

### Alert cooldown and details

`Config.AlertCooldown` (seconds, 60 by default) stops the same crime by the
same player alerting the law again straight away: one shoot-out is one alert
a minute. A different crime has its own cooldown. Player calls such as
`/callpolice` keep their own `cooldown` from the alert's row.

Alerts say where it happened (the town, or the nearest one) and the time. Add
`Config.AlertDetails = false` to `config/config.lua` to leave that out.

### Witness behaviour (optional settings)

Add any of these to `Config.WitnessEscape` in `config/config.lua`; without
them the defaults shown apply.

```lua
Reactions = true,         -- flinch, point, shout or put their hands up before running
Shout = true,             -- witnesses shout as they react and run
IgnoreScriptPeds = true,  -- NPCs of other scripts (frozen, invincible, mission peds) never witness
```

### Blips that wait for the responder

Add `persistentBlip = true` to an alert and its blip stays until the responder
gets within `Config.PersistentBlipClearDistance` metres (25 by default).
`Config.PersistentBlipEnabled = false` turns this off everywhere.

### NPC law response (`config/npc.lua`)

Off by default. Turn it on with `Config.LawResponse.Enabled = true`.

- Only crimes in `Config.LawResponse.EngagedCrimes` bring a posse (Shooting and
  Hijacking by default). Other crimes still alert player lawmen.
- `Config.LawResponse.PlayerLEOPriority = true`: no posse while any lawman is
  on duty (as your duty script says). This is the "NPC police when no police
  are on duty" switch.
- `CrimeSeverity` makes each crime LOW, MEDIUM or HIGH. `ResponseTypes` sets
  the posse size, how many ride in on horseback (`MountedPercent`),
  aggressiveness and chance for each severity.
- The player is warned at once; the posse sets out after `ResponseTime` and
  hunts for at most `Duration`.
- Officers appear at least 70 m away, out of the player's sight, on a road
  where there is one. Riders arrive in the saddle.
- Within about 25 m they call out and hold the player at gunpoint for a few
  seconds: a chance to surrender. Shooting, drawing a gun on them, running or
  waiting it out starts the fight. They fight only the wanted player, never
  bystanders or other lawmen, and get off their horses to chase on foot.
- Get further than `DespawnDistance` from every officer for about 10 seconds
  and you have escaped. When it is over they ride or walk off and vanish once
  out of sight.
- The officers' models depend on the town (`Config.TownLawModels`); outside
  towns, bounty hunters come (`Config.DefaultLawModels`).

Fine-tuning, optional: add any of these to `Config.LawResponse` in
`config/npc.lua`. Without them the defaults shown apply.

```lua
WarnBeforeFiring = true,  -- call out and hold at gunpoint before firing
WarnDistance = 25.0,      -- metres at which they call out
WarnTime = 6,             -- seconds to surrender before they open fire
DismountDistance = 20.0,  -- a rider gets off when the player is on foot this close
Accuracy = nil,           -- 0-100; nil = by severity (about 30 low to 55 high)
RepositionStuck = true,   -- a stuck officer out of sight is moved nearer
EscapeTime = 10,          -- seconds beyond DespawnDistance to escape
DespawnTimeout = 30,      -- seconds to leave out of sight before being removed anyway
Speech = true,            -- officers call out and shout orders
```

NPC law needs the server to allow client-made entities: with
`sv_entityLockdown` set to `strict` or `relaxed` it cannot spawn, and the
server console says so.

**Surrender and jail.** When the posse is close, a player on foot can hold the
surrender prompt, and puts their hands up. The officers hold their fire and
walk in with their guns up, stopping a few metres off. One of them calls out,
walks round behind the player with his sidearm, takes hold of them, and only
then are the cuffs put on. Shooting starts the fight; moving more than
`SurrenderMovementTolerance` metres or drawing a weapon drops the surrender
and the officers go back to their warning (surrender again, or they fire).
If no officer can reach the player, they are let go rather than cuffed from
a distance.
**Jail.** After the arrest the player is jailed for `JailTime` minutes (by
the crime's severity) when something on the server can jail them:

1. `JailExportCall` in `config/npc.lua`, when set. `src`, `charId` and
   `timeInSeconds` are replaced with the real values:

   ```lua
   JailExportCall = "exports.another_jail:JailPlayer(src, timeInSeconds, 'Arrested by law')",
   ```

   It runs as code, so it can only be changed in the file.
2. Otherwise your own jail export, added by hand to `Config.Duty.Custom` in
   `config/config.lua` (`Resource` must be set):

   ```lua
   JailExport = "JailPlayer",   -- called as exports[Resource]:JailPlayer(source, minutes, reason)
   ```

3. Otherwise the built-in Sisika jail, when it is switched on (below).

None of the duty presets can jail: outsider_policeman documents no jail
export, vorp_police jails only through its `/jail` command, and rsg-lawman,
qbr-policejob and bcc-law jail only through events that check the sender is an
officer. With nothing set, the officer takes the cuffs off again after a
moment, the player is told the deputies let them go with a warning, and the
posse leaves.

**The built-in Sisika jail.** Witnesses' own jail, **off by default** so it
never fights a jail script you already run. Switch it on in `config/npc.lua`
(or `/poggy` → Witnesses → NPC law response → *Use the built-in Sisika jail*):

```lua
BuiltinJail = {
    Enabled = true,
},
```

An arrested player is faded to Sisika Penitentiary, put in the prison uniform
(weapons put away), kept on the prison grounds and shown the time left at the
bottom of the screen. Walking off the grounds brings them back; so does
respawning somewhere else after dying inside. The time counts only while they
are online: it is kept under the character id in this resource's server KVP
(no database table), saved every 30 seconds and on disconnect, so a relog or a
server restart carries on where it stopped. When it is over their own clothes
come back (poggy_core's `char.reloadSkin`: on VORP the `rc` command) and they
are faded to the far bank of the river, south-east of Saint Denis. Staff can
use `/jail` and `/unjail` (ACE, or the console); `/jail me 1`
is the quickest way to see it work.

Every other key is optional and can be added under `BuiltinJail` by hand;
these are the defaults:

| Key | Default | What it does |
|---|---|---|
| `Center` | `vector3(3339.23, -670.12, 45.83)` | Middle of the prison grounds. |
| `Radius` | `70.0` | Grounds size in metres; past it the prisoner is brought back. |
| `Spawn`, `SpawnHeading` | `vector3(3339.07, -669.41, 45.83)`, `194.42` | Where a prisoner is put. |
| `Release`, `ReleaseHeading` | `vector3(2948.23, -1234.47, 42.48)`, `95.39` | Where a freed prisoner is put. |
| `Outfit` | `true` | The prison uniform (multiplayer peds only). |
| `MaxMinutes` | `240` | The longest sentence, for `/jail` and the arrest alike. |
| `AllowEscape` | `false` | `true`: getting `EscapeDistance` metres past the grounds is an escape: the sentence ends and the player keeps running in the uniform. |
| `EscapeDistance` | `30.0` | Metres past `Radius` that count as an escape. |
| `EscapeAlert` | `true` | With `AllowEscape`, on-duty law gets a *PRISON BREAK!* alert. |
| `RestoreCommand` | `""` | RSG and QBR: the command that reloads a player's clothes (poggy_core can reload them only on VORP so far). |
| `JailCommand` | `"jail"` | The staff jail command's name, without the slash. With vorp_police running (it has its own /jail) the default is `"witnessjail"`. |
| `UnjailCommand` | `"unjail"` | The staff release command's name (`"witnessunjail"` with vorp_police running). |

The time only counts while online so that logging off is not a way to serve
it. If you rename the resource folder, sentences saved under the old name are
not seen (KVP belongs to the resource).

**Already in jail.** A player the duty script says is jailed is not reported
for a crime and no posse is sent after them (a prison fight or a jail break).
outsider_policeman answers through its `IsPlayerJailed` export (or the
`IsJailed` state bag), rsg-lawman through rsg-core's jail time (kept by
rsg-prison). For your own jail script add `IsJailedExport = "IsJailed"`
(an export that takes the source) or `JailedStateBag = "jailTime"` to
`Config.Duty.Custom`. Calls for help (`/calldoctor`) still work from jail.

---

## For developers

### Police alerts from other scripts

Raise a police alert from any server script (a robbery, a break-in, a
dispatch system). It goes to on-duty law only, with the cooldown, and brings
the NPC posse when no lawman is on duty and the crime is in `EngagedCrimes`.

```lua
local ok, info = exports.poggy_witnesses:PoliceAlert({
    src      = source,                 -- the suspect (their position is used; they are told)
    coords   = vector3(x, y, z),       -- or a position, when there is no suspect
    title    = "BANK ROBBERY",         -- the alert's heading
    message  = "The Valentine bank is being robbed!",
    crime    = "Robbery",              -- the cooldown key and the posse's crime (EngagedCrimes / CrimeSeverity)
    -- optional:
    alert    = "robbery198sd99720",    -- start from a Config.Alerts row (its icon, blip, text)
    jobs     = { "sheriff" },          -- default: Config.PoliceJobList
    blip     = "blip_cash_bag", blipTime = 120000, waypoint = true,
    npcLaw   = false,                  -- true: posse even if the crime is not engaged; false: never
    notifySuspect = false,             -- do not tell the suspect
    suspectMessage = "Someone saw you!",
})
-- ok = false, info = "cooldown", "no_position" or "jailed" (the suspect is in jail) when nothing was sent;
-- otherwise info = { sent = n, lawSent = n, npcLaw = true/false }
```

The same from a server event: `TriggerEvent('poggy_witnesses:policeAlert', data)`
(server side only; clients cannot raise it).

Send an existing alert row as it is:

```lua
exports.poggy_witnesses:TriggerAlertForPlayer(src, "graverobbing8x2kp19")
exports.poggy_witnesses:TriggerAlertForPlayer(src, "robbery198sd99720", { crime = "Robbery" }) -- may bring the posse
```

Duty and the posse:

```lua
exports.poggy_witnesses:IsOnDuty(src)            -- true when that lawman is on duty
exports.poggy_witnesses:GetOnDutyLaw()           -- { src, ... } every on-duty lawman
exports.poggy_witnesses:SetOnDuty(src, true)     -- your duty script tells Witnesses (nil = back to the preset)
exports.poggy_witnesses:GetDutyPreset()          -- "outsider_policeman", "vorp_police", ...
exports.poggy_witnesses:IsJailed(src)            -- true when the built-in jail or the duty script says the player is in jail
exports.poggy_witnesses:StartLawResponse(src, "Robbery", true)  -- send the posse now (true: skip EngagedCrimes)
exports.poggy_witnesses:EndLawResponse(src)
exports.poggy_witnesses:IsWantedByNpcLaw(src)
```

Server events: `poggy_witnesses:onPoliceAlert (src, alert, crime, info)`,
`poggy_witnesses:onLawResponseStarted (src, id, crime, severity, officers)`,
`poggy_witnesses:onLawResponseEnded (src, id, reason)`,
`poggy_witnesses:onJailed (src, charId, minutes, reason)` and
`poggy_witnesses:onJailEnded (src, charId, why)` (the built-in jail; `why` is
`served`, `staff`, `escape` or `quiet` for a character switch).

### The full witness flow for your own crimes

A witness must see it and get away before the alert is sent.

```lua
-- Server
local requestId = exports.poggy_witnesses:CreateWitness(source, "GraveRobbing", {
    alertCommand       = "graverobbing8x2kp19", -- an alert from Config.Alerts
    triggerLawResponse = true,                  -- send the NPC posse if reported
    lawActionType      = "Shooting",            -- the crime to send it for
})

-- Client
exports.poggy_witnesses:CreateWitnessLocal("GraveRobbing", { alertCommand = "graverobbing8x2kp19" })
```

Server events: `poggy_witnesses:onWitnessCreated`, `onWitnessStopped`,
`onWitnessReported` and `onAlertSent`. The full option list is at the top of
`shared/api.lua`.

---

## Troubleshooting

**Law players get no alerts.**
Check the job name is in `Config.PoliceJobList` and the grade is in
`Config.PoliceJobRanks` (grades that start at 1 need adding). If
`Config.DutyCheckEnabled` is on, they must be on duty: `/witnessduty` shows
which duty script is read and what it says for each lawman.

**Nothing is ever witnessed.**
With `Config.EnforceTownLimits = true`, crimes only count inside the towns in
`Config.Towns`. Check you are not in a blacklisted area. Check the crime is
enabled in `Config.Checks` and its chance is above 0. Lawmen on duty are immune
when `Config.LawImmunityEnabled` is on. Stand where it should happen and type
`/witnesscheck`: it prints every one of these checks to F8, with the server's
view of your job and duty.
To see the town and blacklist zones on screen, add `Config.ShowZones = true` to
config/config.lua (optional; off when absent).

**Aiming a lasso or camera starts a witness.**
Add the weapon name to `Config.NonThreateningAimItems`.
`/getmyweaponhash_cl` shows the name the script sees.

**A job NPC from another script runs off to report.**
Add its model name to `Config.BlacklistNPCs`.

**The NPC posse never comes.**
`Config.LawResponse.Enabled` must be `true`, the crime must be in
`EngagedCrimes`, and with `PlayerLEOPriority` on, no player lawmen may be on
duty (`/witnessduty`). It sets out `ResponseTime` after the report. With
`sv_entityLockdown` on, the server does not let it spawn.

**Two alerts for one crime.**
Another script (a dispatch system) also detects shots or fights. Turn its
detection off, or have it call `PoliceAlert` instead.

**A setting shows as read-only in `/poggy`.**
It is built from another setting (a job alert that uses the law jobs list or a
translation) or it runs as code. Change the setting it comes from, or edit
the file.

---

## Changelog

- **1.4.0** — Police alerts reach on-duty law only, through duty presets for
  outsider_policeman, vorp_police, rsg-lawman, qbr-policejob and bcc-law; a
  per-crime alert cooldown; where and when in every alert; the `PoliceAlert`
  export; looting and poaching; witnesses that react before they run; an NPC
  posse that rides in out of sight, warns, fights only the suspect and leaves
  cleanly; a built-in Sisika jail for NPC arrests (off by default).
- **1.3.1** — Poggy Hub support: settings, commands and help in `/poggy`.
- **1.3.0** — Framework-agnostic: no framework events or exports are used
  anywhere. Client job changes come from `poggy_core:jobChangedLocal`, duty
  from poggy_core's `job.get` / `players.onDuty`. Requires poggy_core 0.14.0.
- **1.2.1** — Duty and jail defaults fixed: no calls to exports that do not
  exist.
- **1.2.0** — Folder renamed to `poggy_witnesses`; runs on poggy_core.

## Licence

© 2025 All rights reserved.

The config and translation files are open for you to edit. The client and
server files are escrowed. You may not redistribute, modify or repackage this
resource without permission from the author. For use on your own server only.
Resale, transfer or redistribution is not allowed.
