# Witnesses - RedM Crime Detection System

A sophisticated NPC witness system for RedM that creates realistic consequences for criminal actions. When players commit crimes, nearby NPCs will witness the event, flee to report it, and alert law enforcement.

## Framework Support

Witnesses runs on **poggy_core**, which detects your framework and talks to it. There is nothing to set in the config: install poggy_core and Witnesses works on any framework poggy_core supports.

## Features

- **Framework-agnostic**: Everything framework-shaped goes through poggy_core
- **Dynamic Witness System**: NPCs who witness crimes will try to flee and report to authorities
- **Multiple Crime Types**: Detects shooting, threatening with weapons, melee combat, lassoing, trampling, and vehicle hijacking
- **Witness Escape Logic**: Witnesses must successfully escape the player to report the crime
- **Law Enforcement Alerts**: Integration with law enforcement jobs to receive crime notifications
- **Blip System**: Visual tracking of witness locations on the map
- **Persistent Blips**: Alert blips that remain on the map until the responding officer arrives at the location
- **Town-Based Detection**: Configure which towns have active witness systems
- **Town Blacklisting**: Disable witness system in specific towns like Van Horn or Wapiti
- **Weapon Detection**: Smart weapon detection to determine if actions are threatening
- **NPC Law Response**: Spawn NPC law enforcement when player LEO isn't on duty
- **Engaged Crimes System**: Configure which crime types trigger NPC law enforcement response
- **Player LEO Priority**: Automatically disable NPC law when player LEO is on duty
- **Extensive Configuration**: Highly customizable via config/config.lua and config/npc.lua

## Dependencies

- **poggy_core** 0.11.0 or newer (required)
- **PolyZone** (required): https://github.com/mkafrin/PolyZone
- A duty resource poggy_core can read, for duty checks (on VORP: **vorp_police** / **vorp_medic**). Optional, see below.
- A jail script with a server export, if you want NPC law arrests to add jail time (`Config.LawResponse.JailExportCall` in `config/npc.lua`). Optional, see below.

## Installation

1. Install and start **poggy_core** and **PolyZone**
2. Extract the `poggy_witnesses` folder into your server's `resources` directory
3. Add `ensure poggy_witnesses` to your server.cfg, after poggy_core and PolyZone
4. Restart your server

### Upgrading from the `Witnesses` folder

From 1.2.0 the resource folder is called `poggy_witnesses` (it used to be `Witnesses`).

1. Copy your settings out of the old `config.lua` and `config_npc.lua` (and `translations.lua` if you changed any text).
2. Delete the old `Witnesses` folder and put `poggy_witnesses` in its place.
3. In server.cfg, change `ensure Witnesses` to `ensure poggy_witnesses`.
4. Put your settings back: `config.lua` is now `config/config.lua`, `config_npc.lua` is now `config/npc.lua`, and `translations.lua` stays next to `fxmanifest.lua`.

If another resource of yours uses the Witnesses API, update it too:

- `exports.Witnesses:CreateWitness(...)`, `exports.Witnesses:CreateWitnessLocal(...)` and `exports.Witnesses:TriggerAlertForPlayer(...)` become `exports.poggy_witnesses:...`
- the server events `witnesses:onWitnessCreated`, `witnesses:onWitnessStopped`, `witnesses:onWitnessReported` and `witnesses:onAlertSent` become `poggy_witnesses:onWitnessCreated` and so on

The alert commands and every setting are unchanged.

## Configuration

The script is highly configurable through the `config/config.lua` file:

- **Duty Check**: `Config.DutyCheckEnabled` decides whether law jobs only get alerts while on duty
- **Debug Settings**: Toggle debugging information
- **Witness Parameters**: Control cooldowns, maximum witnesses, search radius, etc.
- **Crime Detection**: Enable/disable specific crime types and their parameters
- **Town Zones**: Set which towns have active witness detection
- **Town Blacklisting**: Prevent witness detection in specific towns
- **Non-threatening Items**: Configure which items don't trigger threatening alerts
- **NPC Blacklist**: Prevent specific NPC models from becoming witnesses
- **Alert Configuration**: Customize alert messages, blips, and job requirements
- **Notification Options**: Toggle top notifications for witness alerts

### Duty Checks

With `Config.DutyCheckEnabled = true`, a player with a law job only receives alerts while on duty, unless the alert sets `overrideDutyCheck = true`. Duty comes from poggy_core; on VORP an officer is on duty after going on duty in vorp_police (or vorp_medic). When no duty resource is running, nobody counts as on duty.

If your duty script is one poggy_core does not read, set an override that returns `true` when the player is on duty:

```lua
Config.DutyExportCall = "return exports['my_duty_script']:IsOnDuty(playerid)"
```

Leave it `""` to use poggy_core.

## How It Works

1. When a player commits a crime (shooting, fighting, etc.), nearby NPCs become witnesses
2. Witnesses will flee from the player to report the crime
3. If a witness successfully escapes (reaches a safe distance for a set duration), they report the crime
4. Law enforcement players receive an alert with location information
5. Players can eliminate witnesses before they report to prevent law enforcement notification

## Player Victim Configuration

The system can be configured to recognize players as victims of crimes through the `Config.AllowPlayersAsVictims` option. This allows crime detection when actions are performed against other players, though players cannot become witnesses themselves.

## Persistent Blips

Persistent blips allow alert blips to remain on the map until the responding officer physically arrives at the crime scene location. This is useful for ensuring officers don't lose track of alerts.

### Configuration (config/config.lua)

```lua
Config.PersistentBlipEnabled = true           -- Master toggle for persistent blip feature
Config.PersistentBlipClearDistance = 25.0     -- Distance in meters at which persistent blips are cleared when player arrives
Config.PersistentBlipCheckInterval = 1000     -- How often (in ms) to check player distance from persistent blips
```

### Per-Alert Configuration

Add `persistentBlip = true` to any alert in `Config.Alerts` to enable persistent blips for that alert type:

```lua
{
    name = 'NEED DOCTOR',
    command = 'calldoctor',
    -- ... other settings ...
    persistentBlip = true  -- Blip stays until doctor arrives
}
```

## Engaged Crimes & Player LEO Priority

The system can be configured to only spawn NPC law enforcement for specific "engaged" crime types, and to automatically disable NPC law response when player law enforcement is on duty.

### Engaged Crimes (config/npc.lua)

Only crimes listed in `Config.EngagedCrimes` will trigger NPC law enforcement response. Other crimes will still alert player LEO but won't spawn NPC officers:

```lua
Config.EngagedCrimes = {
    "Shooting",    -- Shooting triggers NPC law response
    "Hijacking",   -- Hijacking triggers NPC law response
    -- "Threatening", -- Uncomment to enable NPC response for threatening
    -- "Melee",       -- Uncomment to enable NPC response for melee
    -- "Lassoing",    -- Uncomment to enable NPC response for lassoing
    -- "Trampling"    -- Uncomment to enable NPC response for trampling
}
```

### Player LEO Priority (config/npc.lua)

With `PlayerLEOPriority = true`, NPC law does not spawn while at least one player with a job from `Config.PoliceJobList` is on duty (duty as described above).

### NPC Arrests and Jail (config/npc.lua)

When a player surrenders to NPC law they are arrested and the law response ends. Jail time is off by default: stock VORP has no jail API (vorp_police only jails through its `/jail` command and exports nothing), so `Config.LawResponse.JailExportCall` ships empty and the `JailTime` table is not used.

If you run a jail script that has a server export, put the call in `JailExportCall`. `src`, `charId` and `timeInSeconds` are replaced with the real values before it runs:

```lua
Config.LawResponse.JailExportCall = "exports.another_jail:JailPlayer(src, timeInSeconds, 'Arrested by law')"
```

Leave it `""` to arrest without jail. The old default (`exports.vorp_police:StartJailTimerForPlayer(...)`) is ignored if it is still in your config.

## License & Legal

© 2025 All Rights Reserved

This resource is protected by copyright law. The config file is available for configuration, but the client and server files are escrowed. You may not redistribute, modify, or repackage this resource without explicit permission from the author.

This product is for commercial use only within your server. Resale, transfer, or redistribution is strictly prohibited.
