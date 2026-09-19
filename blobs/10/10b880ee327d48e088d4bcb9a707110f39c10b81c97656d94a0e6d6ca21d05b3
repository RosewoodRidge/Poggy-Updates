# Poggy Transform

Become any animal, any ped in the game, or any character on your server:
their face, their body, their clothes and their colours.

Ninety transformations ship in the box. The animals have their own attack
moves, and wolves and dogs have emotes too. Everything is unlocked by default.
Lock whatever you want behind a job whenever you are ready.

---

## Features

- A card menu with ninety animals and characters, sorted into tabs.
- Animals that can attack, pounce, jump and taunt.
- Emotes for wolves and dogs: howl, sit, sleep, roll and more.
- Legendary animals at up to one and a half times normal size.
- Job locks per transformation.
- For admins: any ped model by name, and a browser to wear any character's
  exact appearance (VORP).
- Attacks checked at both ends against abuse.
- Every setting can be changed in game with **`/poggy`**.

---

## Requirements

| Needs | Why |
|---|---|
| **poggy_core** 0.13.0 or newer | Framework detection, jobs, admin groups, notifications and the character browser. Start it first. |
| **A framework poggy_core supports** | VORP, RSG or QBR, detected at start. |
| **oxmysql** | Only for the character browser (poggy_core reads saved appearances through it). |

There is no framework setting to change.

---

## Install

1. Put `poggy_transform` in your resources folder.
2. Add `ensure poggy_transform` to `server.cfg`, **after** `poggy_core`.
3. Give your admins the permission (see [Admins](#admins)).
4. Start the server and read the console.

A healthy start prints the framework it found, how many transformations
loaded, and whether the character browser is on:

```
[poggy_transform] framework: vorp | 90 transformables (0 job-locked) | character browser: enabled
```

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/transform` | Everyone | Opens the menu. Pick a card to transform, or turn back. |
| `/ae` | While an animal with emotes | Opens the emote picker. |
| `/ae <name>` | While an animal with emotes | Plays an emote, e.g. `/ae howl`. |
| `/ae stop` | While an animal with emotes | Stops a looping emote. Backspace does too. |
| `/rcp` | While wearing a character | Re-applies the look if clothing colours drift. |

Rename `/transform` with `Config.Command`. `/ae` and `/rcp` are fixed.

---

## Controls in animal form

| Key | Does |
|---|---|
| Left click | Basic attack. |
| `INPUT_ENTER` control | Pounce, for animals that have one. |
| `INPUT_RELOAD` control | Taunt: growl, howl or roar. |
| Space | Jump attack, for animals that have one. |
| R1 / RB | Crouch. |

Change the extra keys in `Config.AttackInputs`.

---

## Admins

Admins see every transformation, whatever the job lock. Only admins get the
custom ped box and the character browser.

**ACE.** Works on every framework. Add to `server.cfg`:

```
add_ace group.admin poggy_transform.admin allow
```

**Character groups.** `Config.AdminGroups` is checked after the ACE, against
the group poggy_core reports. The default `admin`, `superadmin` and `god`
groups work with no setup.

---

## Configuration

Everything is in `config.lua`, and **every setting can be changed in game with
`/poggy`**.

| Setting | What it does |
|---|---|
| `Config.Command` | The menu command. |
| `Config.AttackCooldown` | Wait between attacks, in ms. |
| `Config.AttackInputs` | Which keys trigger the extra attack moves. |
| `Config.AttackMinInterval` | Anti-abuse: fastest the server passes on attacks, in ms. |
| `Config.AttackRangeTolerance` | Anti-abuse: extra metres a target accepts on top of the animal's reach. |
| `Config.Animals` | Every transformation: model, card, category, job lock, attacks, emotes. |
| `Config.AdminAce`, `Config.AdminGroups` | Who counts as an admin. |

The in-game **Help** tab has guides for adding transformations and tuning
attacks.

### Locking a transformation behind a job

Every entry ships unlocked (`job = ""`). Put a job name in to restrict it:

```lua
{
    id       = "wolf",
    label    = "Wolf",
    model    = "A_C_Wolf",
    job      = "hunter",   -- only the hunter job can become a wolf
    ...
}
```

Use the job name your framework uses. It is not case-sensitive. Admins always
bypass it.

### Adding your own

Add an entry to `Config.Animals`:

```lua
{
    id         = "my_ped",
    label      = "My Ped",
    model      = "A_C_Wolf",       -- any valid RDR3 ped model
    image      = "my_ped.png",     -- put the file in ui/images/
    category   = "predator",       -- predator, prey, bird, aquatic, legendary or people
    job        = "",               -- "" = anyone
    aggressive = false,
},
```

`config.lua`, `ui/images/` and `ui/style.css` are left open, so you can add
transformations, add card art and restyle the menu without touching anything
encrypted.

**No image?** The card shows a category icon instead, so a missing file is
never a broken image.

---

## The character browser

Admins can browse every character on the server and wear their exact look:
face, body, clothing and tints. It is applied on the admin's screen only.
Nothing is written to the database, and the character being copied is not
touched.

It needs saved appearances from poggy_core (`char.offline` with appearance,
then `char.reloadSkin` to put your own look back). Today only the VORP adapter
returns appearances, so the browser is VORP-only. On other frameworks the
Players tab does not appear, and the console says
`character browser: unavailable`. Everything else works on every framework
poggy_core supports.

---

## Anti-abuse

Attacks on players are checked at both ends.

- The server will not pass on attacks faster than `Config.AttackMinInterval`.
- The target will not accept an attack from further away than the animal's
  reach plus `Config.AttackRangeTolerance`.
- The target must have PvP on.

Raise the tolerance if players report attacks missing on a laggy server.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| "You don't have access to any transformations." | Every entry is job-locked and the player has none of those jobs. |
| No Players tab | You are not an admin, or the server is not on VORP, or oxmysql is missing. Check the console banner. |
| An animal will not attack | It needs `aggressive = true` and an `attackAnim`. |
| Attacks miss other players | The target needs PvP on. On a laggy server, raise `Config.AttackRangeTolerance`. |
| A new category shows as its id with a ❓ | The menu only knows the six shipped categories by name. Use one of those. |
| A transformation does nothing | Check the model name. Turn on `Config.Debug` and read the F8 console. |

---

## Changelog

- **1.2.2**: Settings metadata for the `/poggy` hub, and a clearer README.
- **1.2.1**: The character browser's gate now also requires poggy_core's
  `char.offline` capability (so a VORP server without oxmysql is told honestly
  instead of failing), and the "not available" message names the framework.

---

## Support

Poggy Scripts Discord: https://discord.com/invite/rBarFeuzFj
