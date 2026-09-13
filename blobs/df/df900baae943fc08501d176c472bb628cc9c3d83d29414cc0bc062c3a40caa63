# Poggy Transform

Become any animal, any ped in the game, or any character on your server —
their face, their body, their clothes and their colourways.

Ninety transformables ship in the box, each with its own attack moves, brawl
style and emote set. Everything is unlocked by default; lock whatever you want
behind a job whenever you're ready.

---

## Requirements

| | |
|---|---|
| **poggy_core 0.11.0+** | Framework detection, jobs, admin groups, notifications and the character browser. Start it before this resource. |
| **oxmysql** | poggy_core reads the character browser's data through it. |
| **A framework** | VORP Core, RSG Core, QBCore or RedEM:RP — detected at runtime. |

There is no framework setting to change and no version to pick.

---

## Install

1. Drop `poggy_transform` into your resources folder.
2. `ensure poggy_transform` **after** `poggy_core`.
3. Give your admins the permission (see below).
4. Start the server and read the console — it prints the framework it found,
   how many transformables loaded, and whether the character browser is
   available.

Expected on a healthy start:

```
[poggy_transform] framework: vorp | 90 transformables (0 job-locked) | character browser: enabled
```

---

## Commands

| Command | Who | What |
|---|---|---|
| `/transform` | Everyone | Opens the menu |
| `/ae` | While transformed | Opens the emote picker |
| `/ae <name>` | While transformed | Plays an emote directly, e.g. `/ae howl` |
| `/ae stop` | While transformed | Stops the current emote |
| `/rcp` | While wearing a character | Re-applies the appearance if clothing colours drift |

Rename `/transform` with `Config.Command`.

---

## Admin access

Admins bypass every job lock, and are the only ones who get the custom ped box
and the character browser.

**ACE — works on every framework.** In `server.cfg`:

```
add_ace group.admin poggy_transform.admin allow
```

**Framework groups — VORP only.** `Config.AdminGroups` is checked after ACE, so
existing VORP `admin` / `superadmin` / `god` groups keep working with no setup.

On RSG, QBCore and RedEM:RP, use the ACE permission.

---

## Locking things behind a job

Every entry ships unlocked (`job = ""`). To restrict one, put a job name in:

```lua
{
    id       = "wolf",
    label    = "Wolf",
    model    = "A_C_Wolf",
    job      = "hunter",   -- only the hunter job can become a wolf
    ...
}
```

Job names come from your framework through poggy_core, so use whatever name
your framework uses. Admins always bypass this.

---

## Adding your own

Add an entry to `Config.Animals` in `config.lua`:

```lua
{
    id         = "my_ped",
    label      = "My Ped",
    model      = "A_C_Wolf",       -- any valid RDR3 ped model
    image      = "my_ped.png",     -- drop the file in ui/images/
    category   = "predator",       -- must match a Config.Categories id
    job        = "",               -- "" = anyone
    aggressive = false,
},
```

`config.lua`, `ui/images/` and `ui/style.css` are all left open, so you
can add transformables, add card art and restyle the menu without touching
anything encrypted.

**No image?** The card falls back to a category icon, so a missing file is
never a broken image.

---

## The character browser

Admins can browse every character on the server and wear their exact
appearance — face, body, clothing and tints — applied entirely client-side.
Nothing is written to the database, and the character being copied is not
affected in any way.

**This feature is VORP-only.** It reads `skinPlayer`, `compPlayer` and
`compTints` from the `characters` table, which is VORP's schema; RSG and
QBCore store clothing in a different shape. On those frameworks the Players
tab simply doesn't appear — animals and custom ped models work normally.

---

## Anti-abuse

Attacks are validated at both ends. The server won't relay attacks faster than
`Config.AttackMinInterval`, and the target won't accept one from further away
than the animal's reach plus `Config.AttackRangeTolerance`. Raise the tolerance
if players report attacks missing on a laggy server.

---

## Support

Poggy Scripts Discord — https://discord.com/invite/rBarFeuzFj
