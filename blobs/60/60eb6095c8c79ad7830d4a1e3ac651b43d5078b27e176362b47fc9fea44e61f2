# Adding your own emotes

Your emotes go in `Config.CustomEmotes`, either in `config.lua` or on the
**Emotes** tab in `/poggy`. That list is yours: an update never changes it.

The emotes that ship with the script live in `shared/emotes.lua`. That file is
code, so an update replaces it. Do not edit it — anything you put there is lost
on the next update. To change one of those emotes, add a row to
`Config.CustomEmotes` with the same `name` and it will replace the shipped one.

---

## The fields every emote has

| Field | What |
|---|---|
| `name` | What the player types after `/e`. Lower case, no spaces. |
| `label` | The name shown in the menu. |
| `category` | Which category it sits under. It must also be in `Config.Menu.CategoryOrder`, or it will not be shown. |
| `gender` | `"any"`, `"male"` or `"female"`. Only matters while **Match emotes to the character** is on. |
| `kind` | `"scenario"`, `"kit"` or `"anim"`. The rest of the fields depend on this. |

---

## The three kinds

### Scenario

The game plays a world scenario. Easiest to add, and they always loop until
cancelled.

```lua
{ name = "mylean", label = "Lean on a Post", category = "idle", gender = "any",
  kind = "scenario", scenario = "WORLD_HUMAN_LEAN_POST_LEFT" },
```

### Built-in emote

One of the game's own emotes, the same ones the emote wheel plays. They are
identified by a number.

```lua
{ name = "mysalute", label = "Salute", category = "gestures", gender = "any",
  kind = "kit", hash = 901097731 },
```

### Animation

An animation dictionary played on the character. The most control, and the only
kind that can hold a prop.

```lua
{ name = "mystretch", label = "Stretch", category = "gestures", gender = "any",
  kind = "anim",
  dict = "amb_rest@world_human_smoking@male_a@idle_a",
  anim = "idle_a",
  speed = 2.0,       -- how quickly it fades in
  speedX = 2.0,      -- how quickly it fades out
  duration = -1,     -- milliseconds, or -1 to hold until stopped
  flags = 1 },       -- 1 loops, 2 holds the last frame, 49 is upper body only
```

**Flags** are added together. The three worth knowing:

| Flag | What |
|---|---|
| `1` | Loops. Use it with `duration = -1` for anything that should hold. |
| `2` | Holds the last frame when it ends. |
| `49` | Plays on the upper body only, so the player can still walk. |

---

## Prop emotes

Add a `prop` block to an animation emote.

```lua
{ name = "mybook", label = "Read a Book", category = "sitting", gender = "any",
  kind = "anim",
  dict = "amb_rest_sit@world_human_sit_ground@read_book@male_a@base",
  anim = "base", speed = 2.0, speedX = 2.0, duration = -1, flags = 1,
  prop = {
      model = "p_journal_open01x",
      bone  = 22798,          -- 22798 is the right hand, 34606 is the left
      x = 0.1,  y = 0.07,  z = -0.08,     -- position, in metres from the bone
      rx = 0.0, ry = 180.0, rz = 70.0,    -- rotation, in degrees
  } },
```

Getting the offsets right is fiddly. Start by copying a shipped prop emote with
a similar prop and nudge the numbers until it looks right. `poggy_animtool`
does this visually, with a live preview and keyframes, and exports the numbers.

If the prop model cannot be loaded, the emote still plays without it and the
server console says which model failed.

---

## Where to find animations and scenarios

| What | Where |
|---|---|
| Animation dictionaries and names | <https://alexguirre.github.io/animations-list/> |
| Scenarios, props, ped models and more | <https://github.com/femga/rdr3_discoveries> |
| Built in the game, with a live preview | `poggy_animtool` |

---

## Checking your work

1. Restart the script.
2. Watch the server console. An entry missing a `name`, a `kind`, or the fields
   that kind needs is skipped, and the console says how many were skipped.
3. Open the menu. If your emote is not there, check three things:
   - its `category` is in `Config.Menu.CategoryOrder`
   - its `name` is not in `Config.HiddenEmotes`
   - its `gender` matches your character, or is `"any"`
4. Play it with `/e <name>`.
