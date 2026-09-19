# Witnesses

Every crime has a witness. Stop them before they reach the law.

When a player commits a crime in town, a nearby NPC sees it, runs, and tries
to report it. If the witness gets away, the law gets an alert with a blip and
a waypoint. Catch the witness first and nobody hears about it.

---

## What it does

- **Witnesses.** An NPC near the crime becomes a witness, turns to look, then
  flees. They report once they get far enough away and stay away long enough.
- **Crimes it notices.** Shooting and aiming, threatening with a weapon, fist
  fights and melee, lassoing, trampling, stealing a horse or wagon, and riding
  with a hogtied hostage. Each can be switched on or off, with its own chance.
- **Law alerts.** Law jobs get an alert with an icon, a blip, an area circle
  and an optional waypoint. Alerts can go to law jobs only while they are on
  duty.
- **Player alert commands.** `/callpolice`, `/calldoctor`, `/undertaker`,
  `/calltrain` and any you add. Each has its own jobs, grades, blip and
  cooldown.
- **Alerts for other scripts.** Robbery, break-in, bounty, drug sale and more,
  fired by your other scripts.
- **Blips that wait.** An alert can keep its blip until the responder arrives.
- **Where it works.** Only inside the towns you list, or everywhere; with
  blacklisted areas (Sisika, Van Horn, Wapiti by default) where nothing is
  noticed.
- **NPC law response (optional).** A posse of NPC lawmen rides out for the
  crimes you choose, sized by how serious the crime is. Players can fight,
  flee or surrender. It stands down when player lawmen are on duty.
- **Law immunity.** Witnesses can ignore crimes by players with a law job.
- **Three languages.** English, Spanish and French.

## Requirements

| Resource | Why |
|---|---|
| **poggy_core** 0.14.0 or newer | Required. Talks to your framework (VORP, RSG or QBR). |
| **PolyZone** | Required. The town zones. https://github.com/mkafrin/PolyZone |
| A duty script poggy_core can read | Optional. For duty checks. On VORP: vorp_police / vorp_medic. |
| A jail script with a server export | Optional. Only if NPC law arrests should add jail time. |

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
| `/dumpalertgroups` | ACE `command.dumpalertgroups`, console | Lists who is registered for which alerts, then refreshes them. |
| `/debugalertsgroups` | ACE `command.debugalertsgroups`, console | Prints the alert registrations to the server console. |

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
| `Config.DutyCheckEnabled` | Only alert law jobs who are on duty. |
| `Config.Alerts` | Every alert and alert command. |
| `Config.LawResponse.Enabled` | Turns the NPC law response on. Off by default. |

### Duty checks

With `Config.DutyCheckEnabled = true`, a player with a law job only gets alerts
while on duty. An alert with `overrideDutyCheck = true` ignores this. Duty
comes from poggy_core. On VORP, an officer is on duty after going on duty in
vorp_police (or vorp_medic). With no duty script running, nobody counts as on
duty, so law jobs get no alerts.

Other jobs, such as doctors or the undertaker, are always alerted.

If your duty script is one poggy_core does not read, set an override in
`config/config.lua` that returns `true` when the player is on duty:

```lua
Config.DutyExportCall = "return exports['my_duty_script']:IsOnDuty(playerid)"
```

`playerid`, `job`, `charId`, `identifier` and `jobGrade` are replaced with the
player's values. Leave it `""` to use poggy_core. This setting runs as code, so
it can only be changed in the file, not in `/poggy`.

### Blips that wait for the responder

Add `persistentBlip = true` to an alert and its blip stays until the responder
gets within `Config.PersistentBlipClearDistance` metres (25 by default).
`Config.PersistentBlipEnabled = false` turns this off everywhere.

### NPC law response (`config/npc.lua`)

Off by default. Turn it on with `Config.LawResponse.Enabled = true`.

- Only crimes in `Config.LawResponse.EngagedCrimes` bring a posse (Shooting and
  Hijacking by default). Other crimes still alert player lawmen.
- `Config.LawResponse.PlayerLEOPriority = true`: no posse while any player with
  a law job is on duty.
- `CrimeSeverity` makes each crime LOW, MEDIUM or HIGH. `ResponseTypes` sets
  the posse size, aggressiveness and chance for each severity.
- Officers appear at least 70 m away. Get further than `DespawnDistance` away
  and you have escaped.
- The officers' models depend on the town (`Config.TownLawModels`); outside
  towns, bounty hunters come (`Config.DefaultLawModels`).

**Surrender and jail.** A player can surrender to the posse and be arrested.
Jail time is off by default: stock VORP has no jail API. If you run a jail
script with a server export, set the call in `config/npc.lua`. `src`, `charId`
and `timeInSeconds` are replaced with the real values:

```lua
JailExportCall = "exports.another_jail:JailPlayer(src, timeInSeconds, 'Arrested by law')",
```

Leave it `""` to arrest without jail. Like the duty override, it runs as code
and can only be changed in the file.

---

## For developers

Other scripts can use the full witness flow for their own crimes.

```lua
-- Server: a witness must see it and escape before the alert is sent.
local requestId = exports.poggy_witnesses:CreateWitness(source, "GraveRobbing", {
    alertCommand       = "graverobbing8x2kp19", -- an alert from Config.Alerts
    triggerLawResponse = true,                  -- send the NPC posse if reported
    lawActionType      = "Shooting",            -- severity to use for the posse
})

-- Client
exports.poggy_witnesses:CreateWitnessLocal("GraveRobbing", { alertCommand = "graverobbing8x2kp19" })

-- Server: send an alert straight away, no witness.
exports.poggy_witnesses:TriggerAlertForPlayer(src, "graverobbing8x2kp19")
```

Server events: `poggy_witnesses:onWitnessCreated`, `onWitnessStopped`,
`onWitnessReported` and `onAlertSent`. The full option list is at the top of
`shared/api.lua`.

---

## Troubleshooting

**Law players get no alerts.**
Check the job name is in `Config.PoliceJobList` and the grade is in
`Config.PoliceJobRanks`. If `Config.DutyCheckEnabled` is on, they must be on
duty in a duty script poggy_core reads. `/dumpalertgroups` shows who is
registered.

**Nothing is ever witnessed.**
With `Config.EnforceTownLimits = true`, crimes only count inside the towns in
`Config.Towns`. Check you are not in a blacklisted area. Check the crime is
enabled in `Config.Checks` and its chance is above 0. Law jobs are immune when
`Config.LawImmunityEnabled` is on.

**Aiming a lasso or camera starts a witness.**
Add the weapon name to `Config.NonThreateningAimItems`.
`/getmyweaponhash_cl` shows the name the script sees.

**A job NPC from another script runs off to report.**
Add its model name to `Config.BlacklistNPCs`.

**The NPC posse never comes.**
`Config.LawResponse.Enabled` must be `true`, the crime must be in
`EngagedCrimes`, and with `PlayerLEOPriority` on, no player lawmen may be on
duty.

**A setting shows as read-only in `/poggy`.**
It is built from another setting (a job alert that uses the law jobs list or a
translation) or it runs as code. Change the setting it comes from, or edit
the file.

---

## Changelog

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
