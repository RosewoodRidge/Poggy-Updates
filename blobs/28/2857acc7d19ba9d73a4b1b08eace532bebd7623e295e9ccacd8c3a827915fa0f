# Poggy Balloon

Hot air balloons for RedM: NPC taxi rides to any waypoint, self-flown rentals, and passengers who sit properly in the basket.

No more passengers ragdolling in the basket, and no more captains standing still.

---

## Features

### Balloon taxi

- **Operators across the map.** Strawberry, Rhodes, Valentine, Annesburg, Saint Denis, Blackwater, Armadillo and Tumbleweed. Each has a map blip.
- **Fly to your waypoint.** Set a waypoint, pay the fare, board, and take off when you are ready.
- **Smart pilot.** Climbs over hills and mountains, slows on approach and lands gently.
- **In-flight options:** stop here, change destination, "Step On It!" (faster), change seat, disembark.

### Balloon rental

- **Fly it yourself.** Rent a balloon for a set time (1 hour by default).
- **Timer.** The player is warned before the rental runs out, then the balloon is taken back.
- **Per station.** Turn rentals on or off for each station.

### Flying and riding

- **Captain controls.** Move, climb (Shift), descend (Ctrl), boost, brake and altitude lock.
- **Camera-relative steering.** Forward means where you look. Switch to north/south/east/west at any time.
- **Up to 4 passengers** with sitting animations and an invisible floor, so nobody ragdolls.
- **Captain animations.** The captain pulls the burner rope while flying.
- **Synced.** Everyone sees the same seats and animations.

### Languages

English, French, Spanish and German included. Add more in `translations.lua`.

---

## Requirements

| Resource | Why |
|---|---|
| **poggy_core** 0.14.0 or newer | Money, character checks, prompts, notifications and the station menu, on VORP, RSG or QBR |

No menu resource (such as vorp_menu) is needed.

---

## Installation

1. Put the `poggy_balloon` folder in your `resources` folder.
2. Add it to `server.cfg`, after poggy_core:
   ```
   ensure poggy_core
   ensure poggy_balloon
   ```
3. Restart the server.

### Upgrading from `poggy-balloon`

The folder was renamed from `poggy-balloon` to `poggy_balloon`.

1. Delete the old `poggy-balloon` folder and add `poggy_balloon`.
2. In `server.cfg`, change `ensure poggy-balloon` to `ensure poggy_balloon`.
3. If you changed `config.lua` or `translations.lua`, copy your values across. The settings are the same.

---

## How players use it

### Taking a taxi ride

1. Set a waypoint on the map.
2. Walk up to a balloon operator and press **Talk** (G by default).
3. Pick **Take a Ride**. The fare is taken from your cash.
4. Board the balloon within 60 seconds.
5. Press **Take Off** (B by default) when ready.
6. During the flight, use the prompts on screen: **Stop Here**, **Change Destination**, **Step On It!**, **Change Seat**, **Disembark**.
7. After landing, get out. The pilot waits (30 seconds by default), then the balloon flies away.

### Renting a balloon

1. Talk to an operator where rentals are on.
2. Pick **Rent a Balloon**. The price is taken from your cash.
3. Get in as the pilot and fly anywhere until the rental ends.

### Flying as captain

| Control | Action |
|---|---|
| W / S | Forward / back |
| A / D | Left / right |
| Shift (hold) | Burn and climb |
| Ctrl (hold) | Descend |
| Boost prompt (hold) | Move faster |
| Brake prompt (hold) | Slow down |
| Lock Altitude prompt | Hold your current height |
| Toggle Control Mode prompt | Switch between camera-relative and north/south/east/west steering |

The key for each prompt is shown on screen.

### Riding as a passenger

Walk up to a balloon and hold **F** (Enter as Passenger). Up to four passengers fit in the basket.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/spawnballoon` | Admins (see Permissions) | Spawns a balloon next to you. |
| `/spawnballoon_local` | Everyone, only when Admin only is off | Spawns a balloon next to you. |
| `/balloon_server_status` | ACE `command.balloon_server_status` | Prints every balloon and its seats to the server console. |
| `/balloon_taxi_debug toggle \| level <0-4> \| status` | Everyone (own F8 console) | Taxi debugging. |
| `/balloon_debug_main level <0-4> \| toggle \| status` | Everyone (own F8 console) | Seat and boarding debugging. |
| `/balloon_debug_anim level <0-4> \| toggle` | Everyone (own F8 console) | Animation debugging. |
| `/balloon_debug_ctrl level <0-4> \| toggle` | Everyone (own F8 console) | Captain controls debugging. |
| `balloon_taxi_server_debug toggle \| status \| clear <id>` | Server console | Taxi logging, active rides, clear a stuck ride. |
| `balloon_rental_debug status \| end <id>` | Server console | Active rentals, end one early. |

The debug commands only print to the console of the player who types them. Debug levels: 0 off, 1 errors, 2 warnings, 3 info, 4 everything.

---

## Configuration

Every setting can be changed in game with **/poggy** (Poggy Hub). You can also edit `config.lua` and `translations.lua` by hand. Restart the script after a change.

### Main settings

| Setting | Default | What it does |
|---|---|---|
| `Config.BalloonTaxi.Enabled` | `true` | Taxi rides on or off. |
| `Config.BalloonTaxi.Price` | `3` | Fare for one ride ($). |
| `Config.BalloonTaxi.CruisingAltitude` | `80.0` | Flying height above ground (metres). |
| `Config.BalloonTaxi.TravelSpeed` | `22.0` | Normal speed (m/s). |
| `Config.BalloonTaxi.TravelSpeedFast` | `30.0` | "Step On It!" speed (m/s). |
| `Config.BalloonTaxi.ExitWaitTime` | `30000` | How long the pilot waits after landing (ms). |
| `Config.BalloonRental.Enabled` | `true` | Rentals on or off. |
| `Config.BalloonRental.Price` | `50` | Price of one rental ($). |
| `Config.BalloonRental.RentalDurationMinutes` | `60` | Rental length (minutes). |
| `Config.BalloonRental.WarningTimeMinutes` | `5` | Warning before the rental ends (minutes). |
| `Config.Flight` | — | Captain climb and descend speeds. |
| `Config.Controls` | — | Keys for the taxi prompts (control hashes). |
| `Config.BalloonTaxi.Locations` | 8 stations | Operator position, balloon spawn point, blip, rentals on/off. |
| `Config.Commands.SpawnBalloon` | — | Who may use `/spawnballoon`. |
| `Config.Language` | `"en"` | `en`, `fr`, `es` or `de` (set in `translations.lua`). |

Help pages in `/poggy` walk through **adding a station** and **tuning flights and rentals**.

### Stations

| Station | Operator position | Rentals |
|---|---|---|
| Strawberry | -1812.9, -677.43, 149.6 | Yes |
| Rhodes | 1425.01, -1273.0, 77.27 | Yes |
| Valentine | -145.7, 611.94, 114.39 | Yes |
| Annesburg | 2865.34, 1427.64, 67.41 | Yes |
| Saint Denis | 2717.29, -1511.52, 43.19 | Yes |
| Blackwater | -697.71, -1285.4, 42.22 | Yes |
| Armadillo | -3738.5, -2589.45, -14.3 | Yes |
| Tumbleweed | -5468.53, -2962.53, -1.4 | Yes |

---

## Permissions

`/spawnballoon` checks, in order:

1. `Config.Commands.SpawnBalloon.Enabled = false` → nobody can use it.
2. `AdminOnly = false` → everyone can use it.
3. Otherwise the player needs the ACE in `AcePermission` (if set), **or** must be an admin in the framework.

Example ACE setup:

```
# with AcePermission = "command.spawnballoon" in config.lua
add_ace group.admin command.spawnballoon allow
```

`/balloon_server_status` needs its own ACE:

```
add_ace group.admin command.balloon_server_status allow
```

---

## Troubleshooting

**"Please set a waypoint on your map first."**
Taxi rides fly to your waypoint. Set one before talking to the operator.

**"You are already on a balloon ride." but I am not.**
A ride did not close properly. In the server console run `balloon_taxi_server_debug clear <playerId>`.

**No operators or blips.**
Check that the taxi or rentals are turned on. An operator only appears when a player is within 100 metres.

**The captain cannot climb.**
Keep **Scripted climb** on (`Config.Flight.ScriptedAscent = true`). The game's own burner key does not register on current game builds.

**Rental balloon has no controls.**
The captain controls only work on the `hotairballoon01` model. Keep `Config.BalloonRental.BalloonModel` set to it.

**The ready message says "1 hour" but my rentals are shorter.**
That line is plain text in `translations.lua` (`rental_ready`). Edit it to match. The menu already shows the real length.

---

## Technical notes

- **Taxi flight.** Uses the game's balloon transport natives, with look-ahead terrain avoidance and smoothed speed changes. States: waiting for entry → waiting for take-off → ascending → cruising → approaching → landing → waiting for exit.
- **Passenger detection.** `_IS_PED_ON_TRANSPORT_ENTITY` (0x159EF5B6EDCE00E8), with attachment and distance checks as fallbacks.
- **Animations.** Captains get burner controls with a rope; passengers sit; AI pilots idle on the burner.

---

## Files

```
poggy_balloon/
├── fxmanifest.lua
├── config.lua              settings
├── translations.lua        language and all player text
├── README.md
├── CHANGELOG.md
├── client/
│   ├── balloon_controls.lua   captain controls
│   ├── balloon.lua            seats and boarding
│   ├── balloonanimations.lua
│   ├── balloon_taxi.lua       taxi service
│   ├── balloon_spawn.lua
│   └── balloon_rental.lua     rentals
├── server/
│   ├── balloon_server.lua        seat tracking
│   ├── balloon_taxi_server.lua   fares, /spawnballoon
│   └── balloon_rental_server.lua rentals and expiry
└── docs/                       Poggy Hub card and help pages
```

---

## Credits

This resource combines and extends code from:

- [kibook's side saddle](https://github.com/kibook/redm-sidesaddle)
- [kibook's balloon controls](https://github.com/kibook/redm-ballooncontrols)
- [PersePixel's balloon crap](https://github.com/PersePixels/balloon-crap)

Taxi service, rentals and translations by Poggy.

## Licence

Built on open-source projects. Use and modify it for your server, and please credit the original authors.

## Video

https://medal.tv/games/red-dead-2/clips/kizULvk7B3R3miNy1
