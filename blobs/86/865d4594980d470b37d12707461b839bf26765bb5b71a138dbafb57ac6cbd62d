# Poggy Skillcheck

A Dead by Daylight style circular skillcheck for RedM.

A needle sweeps round a ring. The player presses **Space** while it is inside the zone. Other scripts call it with one export and get the result back: pass, fail, and "great" hits.

Drawn in the game's own browser layer (NUI), with sounds, shake, repetitions and a gold "great" zone.

---

## Contents

- [For players](#for-players)
- [Requirements](#requirements)
- [Installation](#installation)
- [Commands](#commands)
- [Configuration](#configuration)
- [For developers: the export](#for-developers-the-export)
- [Troubleshooting](#troubleshooting)

---

## For players

A ring appears on your screen with a turning needle.

| Part of the ring | What happens |
|---|---|
| White **success zone** | Press Space here to pass. |
| Gold **great zone** (inside the success zone) | Press Space here for a bonus "great" hit. |
| Anywhere else | The skillcheck fails. |

- Some skillchecks need **several hits in a row**. A counter shows your progress, for example `2 / 3`. One miss fails the whole check.
- A **warning tone** plays about one second before the ring appears.
- The ring may **shake** or appear **away from the centre** of the screen.
- The needle may turn **either way**, and may change direction between hits.

| Event | You see | You hear |
|---|---|---|
| Skillcheck coming | Ring fades in | Warning tone |
| Hit | White flash | Hit sound |
| Great hit | Gold flash | Great sound |
| Miss | Red tint | — |

The game keeps running while the ring is up. You are not locked into a menu.

---

## Requirements

| Resource | Why |
|---|---|
| `poggy_core` | Required by every Poggy script. Must start first. |

No database tables. No server-side setup.

---

## Installation

1. Put the `poggy_skillcheck` folder in your server's `resources` folder.
2. Add this line to `server.cfg`, **after** `ensure poggy_core` and **before** any script that uses the skillcheck:
   ```
   ensure poggy_skillcheck
   ```
3. Restart the server.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/skillcheck [options]` | Everyone | Test command. Starts a skillcheck with the values you type. Only exists when `TestCommand = true`. |
| `/skillcheck on` / `off` | Everyone | Turns your own skillcheck sounds on or off until you reconnect. Only when `TestCommand = true`. |

The test command has **no permission check**. Leave `TestCommand = false` on a live server.

The options are positional. Give as many as you like; the rest use the defaults:

```
/skillcheck [speed] [difficulty] [repetition] [randomizer] [shake] [shakeSpeed] [shakeDist] [timeBetween] [direction] [great]
```

| Example | Meaning |
|---|---|
| `/skillcheck` | All defaults |
| `/skillcheck 5 5` | Fast needle, small zone |
| `/skillcheck 4 4 3 0 true 3 3 300 cw 50` | Everything set |

---

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy settings hub). You can also edit `config.lua` by hand. Restart the script after a change.

### General

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | `true` | Master switch. When `false`, no ring is shown and **every call passes at once**. |
| `TestCommand` | `false` | Adds the `/skillcheck` test command for everyone. |

### Default difficulty (`Config.SkillCheck.Defaults`)

Used for any option a calling script leaves out.

| Setting | Default | Range | What it does |
|---|---|---|---|
| `speed` | `3` | 1 – 5 | Needle speed. 1 is about 3.5 seconds a lap; 5 is under one second. |
| `difficulty` | `3` | 1 – 5 | Zone size. 1 is about a quarter of the ring; 5 is about a fourteenth. |
| `repetition` | `1` | 1 – 10 | Hits in a row needed to pass. |
| `randomizer` | `0` | 0 – 5 | Screen position scatter. 0 = centre; each step up to 10% of the screen further. |
| `shake` | `false` | — | Shake the ring. |
| `shakeSpeed` | `2` | 1 – 5 | How fast it shakes. |
| `shakeDist` | `2` | 1 – 5 | How far it shakes (about 1 to 8 pixels). |
| `timeBetween` | `250` | 100 – 500 | Milliseconds between repetitions. |
| `direction` | `"cw"` | `cw`, `ccw`, `rand` | Clockwise, anticlockwise, or random each repetition. |
| `great` | `30` | 0 – 100 | Percent of the zone that is "great". 0 turns great hits off. |
| `startOffset` | `20` | 0 – 50 | Random needle start position, in percent of the ring. 0 = always the same place. |

`speed`, `difficulty`, `shakeSpeed` and `shakeDist` must be whole numbers. Values outside a range are clamped.

### Sounds (`Config.SkillCheck.Sounds`)

| Setting | Default | What it does |
|---|---|---|
| `Volume` | `0.1` | Volume of the sounds, 0 (silent) to 1 (full). |
| `IncomingBoost` | `1` | Multiplies the warning tone's volume. Never goes above full volume. |
| `Incoming`, `Good`, `Great` | file names | Not used by this version. |

**Your own sounds:** replace the files in `ui/sfx/skillcheck/` and keep the same names: `incoming.mp3`, `good.mp3`, `great.mp3`.

---

## For developers: the export

Call it from **client-side** Lua. The call waits until the player finishes, so always run it inside a thread or callback.

```lua
CreateThread(function()
    local result = exports.poggy_skillcheck:StartSkillCheck(options)
end)
```

`options` is optional. Leave it out, or pass `{}`, to use the defaults. Every option in the **Default difficulty** table can be passed, with the same name and range.

### Return value

```lua
{
    success    = bool,    -- true if every repetition was hit
    great      = bool,    -- true if EVERY repetition was a great hit
    greatCount = number,  -- great hits across all repetitions
}
```

If `success` is `false`, the player missed at least one repetition. `greatCount` still counts the great hits before the miss.

### Other exports

| Export | Returns | What it does |
|---|---|---|
| `IsSkillCheckBusy()` | bool | `true` while a skillcheck is running. |
| `CancelSkillCheck()` | bool | Stops the running skillcheck. `false` if nothing was running. The waiting call returns `success = false`. |

### Good to know

- **One at a time.** A call made while another skillcheck is running returns `success = false` at once. It does not wait.
- **Switched off.** With `Enabled = false`, every call returns `success = true` at once.
- **Timing.** A call plays the warning tone, waits one second, then shows the ring.
- **Client only.** No server events. Reward the player from your own server code.

### Examples

Easy lockpick:

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    speed      = 1,
    difficulty = 1,
})
```

Hard crafting, three hits, bonus for all great:

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    speed       = 5,
    difficulty  = 5,
    repetition  = 3,
    timeBetween = 300,
})

if result.success then
    if result.great then
        TriggerServerEvent('crafting:bonusItem')
    else
        TriggerServerEvent('crafting:normalItem')
    end
end
```

Fishing, with shake and random direction:

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    repetition = 5,
    shake      = true,
    shakeSpeed = 3,
    shakeDist  = 3,
    direction  = "rand",
    great      = 20,
})
```

Reward each great hit:

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({ repetition = 4, great = 25 })
if result.success then
    local bonus = result.greatCount * 10
    -- pay the bonus from your server code
end
```

No great zone:

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({ great = 0 })
-- result.great is always false, result.greatCount is always 0
```

### How it works

1. Your script calls the export.
2. The warning tone plays. One second later the ring appears.
3. The script watches for Space and passes each press to the ring.
4. The ring checks the needle against the zone: hit, great hit, or miss.
5. After the last repetition (or the first miss) the result comes back to your script.

### Files

```
poggy_skillcheck/
├── fxmanifest.lua
├── config.lua                 settings
├── README.md
├── client/cl_skillcheck.lua   exports, Space key, test command
├── ui/                        ring, styles, sounds (ui/sfx/skillcheck/*.mp3)
└── docs/                      settings hub page and help
```

---

## Troubleshooting

**Every skillcheck passes straight away.**
`Enabled` is `false`. Set it to `true` and restart.

**A skillcheck fails at once without showing.**
Another skillcheck was still running. Check `IsSkillCheckBusy()` before calling.

**No sound.**
Check `Volume` is above 0. A player who typed `/skillcheck off` has muted their own sounds until they reconnect or type `/skillcheck on`.

**`/skillcheck` does nothing.**
`TestCommand` is `false`, or `Enabled` is `false`.

---

## Licence

Free Poggy script. Use it and change it on your own server; please do not resell or re-upload it as your own.
