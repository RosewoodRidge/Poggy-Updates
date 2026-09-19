# Using it in your own scripts

Call the skillcheck from **client-side** Lua. The call waits until the player finishes, so run it inside a thread.

```lua
CreateThread(function()
    local result = exports.poggy_skillcheck:StartSkillCheck({
        speed      = 3,
        difficulty = 3,
        repetition = 2,
    })

    if result.success then
        -- passed
    end
end)
```

Every option can be left out. Anything you leave out uses the defaults from the **Default difficulty** tab.

## Options

| Option | Values | Meaning |
|---|---|---|
| `speed` | 1 – 5 | Needle speed. 1 = slow. |
| `difficulty` | 1 – 5 | Zone size. 1 = large and easy. |
| `repetition` | 1 – 10 | Hits in a row needed to pass. |
| `randomizer` | 0 – 5 | How far from the screen centre the ring may appear. |
| `shake` | true / false | Shake the ring. |
| `shakeSpeed` | 1 – 5 | How fast it shakes. |
| `shakeDist` | 1 – 5 | How far it shakes. |
| `timeBetween` | 100 – 500 | Milliseconds between repetitions. |
| `direction` | `"cw"`, `"ccw"`, `"rand"` | Needle direction. |
| `great` | 0 – 100 | Percent of the zone that is a great hit. |
| `startOffset` | 0 – 50 | Random start position, percent of the ring. |

Values outside the range are clamped.

## What you get back

| Field | Meaning |
|---|---|
| `success` | `true` when every repetition was hit. |
| `great` | `true` when every repetition was a great hit. |
| `greatCount` | How many great hits there were. |
| `cancelled` | Usually `true` when the check was cancelled. Treat any `success = false` as a fail. |

## Things to know

- **One at a time.** If a skillcheck is already running, a second call returns `success = false` straight away. Check first with `exports.poggy_skillcheck:IsSkillCheckBusy()`.
- **Cancel.** `exports.poggy_skillcheck:CancelSkillCheck()` stops the running check and returns `true` (or `false` when nothing was running). The waiting `StartSkillCheck` call then returns `success = false`.
- **Switched off.** When **Skillchecks enabled** is off, every call returns `success = true` at once.
- **Warning tone.** The call plays a warning tone and waits one second before the ring appears.
