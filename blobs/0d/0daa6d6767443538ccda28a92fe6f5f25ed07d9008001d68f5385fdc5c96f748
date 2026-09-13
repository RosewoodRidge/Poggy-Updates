# Poggy Skillcheck

A standalone Dead-by-Daylight style circular skillcheck system for RedM. Rendered entirely in HTML/CSS/JS via NUI with full sound effects, shake animations, multi-repetition support, and a "great zone" mechanic.

![RedM](https://img.shields.io/badge/Platform-RedM-red) ![Lua](https://img.shields.io/badge/Language-Lua%20%2F%20JS-blue) ![Version](https://img.shields.io/badge/Version-1.0.1-green)

---

## Table of Contents

- [For Users (Server Players)](#for-users-server-players)
  - [How It Works](#how-it-works)
  - [Controls](#controls)
  - [Visual & Audio Feedback](#visual--audio-feedback)
  - [Tips](#tips)
- [For Developers](#for-developers)
  - [Installation](#installation)
  - [Resource Structure](#resource-structure)
  - [Calling the Skillcheck (Export)](#calling-the-skillcheck-export)
  - [Parameters](#parameters)
  - [Return Value](#return-value)
  - [Usage Examples](#usage-examples)
  - [Test Command](#test-command)
  - [Configuration](#configuration)
  - [Architecture & Control Flow](#architecture--control-flow)

---

## For Users (Server Players)

### How It Works

When triggered by a game action (lockpicking, crafting, fishing, etc.), a circular ring appears on your screen with a rotating needle. A highlighted zone on the ring marks the **success area**, and a smaller inner portion marks the **great area**. Your goal is to press the input key while the needle is inside the success zone.

- **Success Zone** — Hitting anywhere inside the highlighted arc counts as a pass.
- **Great Zone** — A smaller, brighter portion within the success zone. Landing here gives a bonus "great" hit.
- **Fail** — Missing the zone entirely (or not pressing in time) fails the skillcheck.

Some skillchecks require **multiple consecutive hits** (repetitions). You must pass every repetition to succeed overall. A streak counter in the UI shows your progress (e.g., `2 / 3`).

### Controls

| Action | Key |
|--------|-----|
| Hit the skillcheck | **Spacebar** |

Press **Space** when the rotating needle overlaps the success zone.

### Visual & Audio Feedback

| Event | Visual | Sound |
|-------|--------|-------|
| Skillcheck incoming | Ring fades in | Warning tone (`incoming.mp3`) |
| Successful hit | White flash | `good.mp3` |
| Great hit | Gold flash | `great.mp3` |
| Failed hit | Red tint overlay | — |
| Completion | Ring fades out | — |

### Tips

- **Watch the needle speed.** Harder skillchecks have faster needles and smaller success zones.
- **Listen for the incoming sound.** It plays ~1 second before the skillcheck appears, giving you time to prepare.
- **The ring may shake.** Some skillchecks add screen shake to increase difficulty — stay focused on the needle position, not the ring movement.
- **Needle direction can change.** The needle may rotate clockwise, counter-clockwise, or randomly between reps.

---

## For Developers

### Installation

1. Copy the `poggy_skillcheck` folder into your server's `resources/` directory.
2. Add `ensure poggy_skillcheck` to your `server.cfg` (ensure it starts before any resources that use it).
3. No dependencies required — this is a fully standalone client-side resource.

### Resource Structure

```
poggy_skillcheck/
├── config.lua                  -- Shared configuration (defaults, sounds)
├── fxmanifest.lua              -- Resource manifest
├── README.md                   -- This file
├── client/
│   └── cl_skillcheck.lua       -- Client export, input handling, NUI bridge
└── ui/
    ├── index.html              -- NUI page (canvas + overlays)
    ├── skillcheck.css           -- Styling, flash states, animations
    ├── skillcheck.js            -- Ring rendering, collision, callback
    └── sfx/skillcheck/
        ├── incoming.mp3         -- Pre-skillcheck warning tone
        ├── good.mp3             -- Normal success sound
        └── great.mp3            -- Great zone success sound
```

### Calling the Skillcheck (Export)

The skillcheck is triggered from **client-side Lua** via a single blocking export:

```lua
local result = exports.poggy_skillcheck:StartSkillCheck(options)
```

- **`options`** — A table of parameters (all optional). Omit the table entirely or pass `{}` to use all defaults.
- **Returns** immediately once the player completes or fails the skillcheck.
- **Blocking** — The calling thread yields until the skillcheck resolves. Always call from inside a `Citizen.CreateThread` or a callback, never from the main thread.

### Parameters

All parameters are optional. Any omitted parameter falls back to the default defined in `config.lua`.

| Parameter | Type | Range | Default | Description |
|-----------|------|-------|---------|-------------|
| `speed` | number | 1 – 5 | `3` | Needle rotation speed. 1 = slow, 5 = very fast. |
| `difficulty` | number | 1 – 5 | `3` | Size of the success zone. 1 = large/easy, 5 = small/hard. |
| `repetition` | number | 1 – 10 | `1` | Number of consecutive skillchecks the player must pass. |
| `randomizer` | number | 0 – 5 | `0` | Random screen position offset. 0 = always centered, 5 = up to ~50% scatter. |
| `shake` | boolean | — | `false` | Whether the skillcheck ring shakes during play. |
| `shakeSpeed` | number | 1 – 5 | `2` | How fast the ring oscillates when `shake` is enabled. |
| `shakeDist` | number | 1 – 5 | `2` | How far the ring moves from its origin when shaking (in pixels). |
| `timeBetween` | number | 100 – 500 | `250` | Milliseconds of delay between repetitions (only applies when `repetition > 1`). |
| `direction` | string | `"cw"` / `"ccw"` / `"rand"` | `"cw"` | Needle rotation direction. `"cw"` = clockwise, `"ccw"` = counter-clockwise, `"rand"` = random per rep. |
| `great` | number | 0 – 100 | `30` | Percentage of the success zone that counts as the "great" zone. Set to `0` to disable great hits. |

Values outside their valid range are clamped automatically.

### Return Value

The export returns a table with three fields:

```lua
{
    success    = bool,    -- true if the player hit inside the success zone on all reps
    great      = bool,    -- true if EVERY rep was a "great" hit
    greatCount = number,  -- total number of great hits across all reps
}
```

**Important:** If `success` is `false`, the player failed at least one repetition. `greatCount` still reflects any great hits achieved before the failure.

### Usage Examples

#### Basic — All Defaults

```lua
Citizen.CreateThread(function()
    local result = exports.poggy_skillcheck:StartSkillCheck()

    if result.success then
        print("Passed!")
    else
        print("Failed!")
    end
end)
```

#### Easy Lockpick — Slow, Large Zone

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    speed      = 1,
    difficulty = 1,
})
```

#### Hard Crafting — Fast Needle, Tiny Zone, 3 Reps

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    speed      = 5,
    difficulty = 5,
    repetition = 3,
    timeBetween = 300,
})

if result.success then
    if result.great then
        -- Player nailed every single rep in the great zone
        TriggerServerEvent('crafting:bonusItem')
    else
        TriggerServerEvent('crafting:normalItem')
    end
end
```

#### Fishing — Medium Difficulty with Shake and Random Direction

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    speed      = 3,
    difficulty = 3,
    repetition = 5,
    shake      = true,
    shakeSpeed = 3,
    shakeDist  = 3,
    direction  = "rand",
    great      = 20,
})
```

#### Dynamic Difficulty — Scale with Player Skill

```lua
-- Scale difficulty based on player level or context
local playerLevel = GetPlayerLevel() -- your own function

local result = exports.poggy_skillcheck:StartSkillCheck({
    speed      = math.min(playerLevel, 5),
    difficulty = math.min(playerLevel, 5),
    repetition = math.min(math.floor(playerLevel / 2), 10),
})
```

#### Reward Great Hits

```lua
local result = exports.poggy_skillcheck:StartSkillCheck({
    repetition = 4,
    great      = 25,
})

if result.success then
    local bonus = result.greatCount * 10  -- $10 bonus per great hit
    GivePlayerMoney(bonus)
    print("Bonus: $" .. bonus .. " (" .. result.greatCount .. " great hits)")
end
```

#### No Great Zone

```lua
-- Set great to 0 to disable the great zone entirely
local result = exports.poggy_skillcheck:StartSkillCheck({
    great = 0,
})
-- result.great will always be false, result.greatCount will always be 0
```

### Test Command

A built-in chat command is available for testing without writing any code:

```
/skillcheck [speed] [difficulty] [repetition] [randomizer] [shake] [shakeSpeed] [shakeDist] [timeBetween] [direction] [great]
```

**Examples:**

```
/skillcheck                              -- All defaults
/skillcheck 5 5                          -- Fast + hard
/skillcheck 4 4 3 0 true 3 3 300 cw 50  -- Full custom
/skillcheck on                           -- Unmute sounds
/skillcheck off                          -- Mute sounds
```

Arguments are positional. You can supply as many or as few as you want — the rest default. The result is printed to the F8 console.

### Configuration

Edit `config.lua` to change global defaults and sound settings. These values apply when parameters are omitted from the export call.

```lua
Config.SkillCheck = {
    Enabled = true,        -- Master toggle (false disables all skillchecks)

    Defaults = {
        speed       = 3,
        difficulty  = 3,
        repetition  = 1,
        randomizer  = 0,
        shake       = false,
        shakeSpeed  = 2,
        shakeDist   = 2,
        timeBetween = 250,
        direction   = "cw",
        great       = 30,
    },

    Sounds = {
        Volume        = 0.1,   -- Base volume (0.0 – 1.0)
        IncomingBoost = 1,     -- Multiplier for the incoming warning tone
        Incoming      = "incoming.mp3",
        Good          = "good.mp3",
        Great         = "great.mp3",
    },
}
```

**Adding custom sounds:** Replace the `.mp3` files in `ui/sfx/skillcheck/` and update the filenames in `Config.SkillCheck.Sounds`.

### Architecture & Control Flow

```
1. Client script calls export
       │
       ▼
2. Lua validates & clamps parameters
       │
       ▼
3. NUI message "skillcheckStart" sent to JS
       │
       ▼
4. JS renders ring on canvas, starts needle rotation
       │
       ▼
5. Lua spawns input thread polling for Spacebar
       │
       ├──── Player presses Space ────▶ NUI "skillcheckKeyPress"
       │                                      │
       │                                      ▼
       │                               JS checks needle vs. zone
       │                                      │
       │                          ┌───────────┴───────────┐
       │                          ▼                       ▼
       │                    Hit (good/great)            Miss
       │                    Flash + sound           Red tint + fail
       │                          │                       │
       │                          ▼                       │
       │                   More reps left?                │
       │                    Yes → loop                    │
       │                    No  → done                    │
       │                          │                       │
       └──────────────────────────┴───────────────────────┘
                                  │
                                  ▼
6. JS sends NUI callback "skillcheckResult"
       │
       ▼
7. Lua receives { success, great, greatCount }
       │
       ▼
8. Export returns result to calling script
```

**Key implementation details:**

- **Client-only** — No server-side scripts or events. All logic runs on the client.
- **Blocking export** — Uses `Citizen.Await` internally, so the calling thread yields until resolution.
- **NUI focus is NOT taken** — The skillcheck uses a Spacebar poll thread, not NUI focus/cursor. Players remain in full game control.
- **Thread-safe** — A `skillcheckBusy` flag prevents overlapping skillchecks. If called while one is active, it waits.
- **Sound preloading** — Audio files are preloaded on resource start to avoid first-play latency.

---

## License

Free Poggy script. Use it and change it on your own server; please do not resell or re-upload it as your own.
