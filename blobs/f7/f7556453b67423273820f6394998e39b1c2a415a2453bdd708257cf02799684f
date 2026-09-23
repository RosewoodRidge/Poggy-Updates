# Poggy Emotes

An emote menu for RedM with over 330 animations, a search box that actually
finds things, favourites saved per character, and a stand-in character who acts
each emote out before you commit to it.

---

## Contents

1. [Features](#features)
2. [Requirements](#requirements)
3. [Installation](#installation)
4. [Commands and keys](#commands-and-keys)
5. [Using the menu](#using-the-menu)
6. [Categories](#categories)
7. [The Nearby tab](#the-nearby-tab)
8. [Adding your own emotes](#adding-your-own-emotes)
9. [Hiding emotes](#hiding-emotes)
10. [Configuration](#configuration)
11. [Database](#database)
12. [Translations](#translations)
13. [Upgrading from another emote script](#upgrading-from-another-emote-script)
14. [Troubleshooting](#troubleshooting)

---

## Features

| Feature | What it does |
|---|---|
| **Nearby** | The first tab is not emotes: it is what you can do with what is around you. Sit on that chair, stand at the bar, lean on that railing, pump that water. Pick one and your character walks over and does it. |
| **330+ emotes** | Scenarios, animations and the game's own built-in emotes, sorted into eight categories. |
| **Search** | Type any part of a name. The search looks across every category at once, not just the one you have open. |
| **Favourites** | Star the ones you use. They are saved to your character and are there when you come back. |
| **Recent** | The emotes you played last, newest first, so the one you want is usually one click away. |
| **Watch it first** | A ▶ button in front of every emote plays it on a stand-in character beside you, props and all, without closing the menu. Only you see it. |
| **Props that let go** | Props are put away on every path out: cancelling, dying, mounting, swimming, playing something else, or the script restarting. That includes the props the game itself hands out, like the guitar. |
| **Ragdoll key** | `Z` drops your character limp on the ground, `Z` again lets them get up. With a cooldown so it cannot be spammed to dodge a lasso. |
| **Props filter** | One button narrows any category, or a search, to the emotes that put something in your hand. |
| **Keep menu open** | A switch at the bottom of the menu. On, the menu stays up while you play emotes; each player's choice is remembered. |
| **Keyboard driven** | Arrows to move, Enter to play, Esc to close, and typing anywhere jumps to the search box. |
| **Owner editable** | Add your own emotes, re-label or re-time the shipped ones, and hide the ones you do not want, all from `config.lua` or `/poggy`. |

---

## Requirements

| Resource | Why |
|---|---|
| `poggy_core` | Required by every Poggy script. Must start first. |
| `oxmysql` | Stores each character's favourites and recent emotes. |

---

## Installation

1. Put the `poggy_emotes` folder in your server's `resources` folder.
2. Add this line to `server.cfg`, **after** `ensure poggy_core`:
   ```
   ensure poggy_emotes
   ```
3. Restart the server.

There is no SQL to import. poggy_core creates the table the first time the
script starts.

---

## Commands and keys

| Command | Who | What it does |
|---|---|---|
| `/emotes` | Everyone | Opens the emote menu. Typing it again closes it. |
| `/e <name>` | Everyone | Plays one emote straight away, e.g. `/e wave`. On its own it opens the menu. |
| `/ec` | Everyone | Stops the emote and puts away any prop. |
| `/ragdoll` | Everyone | Drops your character limp. Again to get up. |

| Key | What it does |
|---|---|
| `End` | Opens and closes the menu. |
| `Backspace` | Stops the emote and puts away any prop. Works at any time, including while riding. |
| `Z` | Ragdoll. Press once to drop, press again to get up (after 2 seconds, by default). Not a key to hold. Does nothing while the chat box is open. |

RedM has no in-game rebinding screen, so both keys are set in `config.lua` and
are the same for every player. Change them under `Config.Keys`, or in `/poggy`
on the General tab.

The open key is read straight from the keyboard, which is how it can be a key
the game has no control for, like `End`. `Config.Keys.OpenMenuControl` adds a
game control as well, for controller players.

All three command names are configurable too, so `/e` can be `/emote` or
anything else that does not clash with another script.

---

## Using the menu

| Action | How |
|---|---|
| Find an emote | Start typing. The search looks at both the shown name and the `/e` name. |
| Play it | Click the name, or move to it with the arrow keys and press Enter. |
| Watch it first | Click the **▶** in front of the name. The stand-in character acts it out and the menu stays open. Hovering a row, or moving to it with the arrow keys, does the same. |
| Star it | Click the star on the right of the row. |
| See only prop emotes | **Props only**, beside the search box. It narrows whatever is showing. |
| Play several in a row | Flip **Keep menu open** at the bottom. The menu stays up after each emote, and the switch is remembered. |
| Change category | Click one on the left, or use the left and right arrow keys when the search box is empty. |
| Stop an emote | The **Stop emote** button, `Backspace`, or `/ec`. |
| Close | `Esc`, the `×`, or `End` again. |

The menu opens on the first category that has anything in it, so a player who
has never starred an emote does not land on an empty Favourites tab.

Emotes marked **PROP** put something in your character's hand.

---

## Categories

Eight ship with the script, plus three the script builds for each player:

| Category | What is in it |
|---|---|
| Nearby | What the player can do with the things around them right now. See [The Nearby tab](#the-nearby-tab). |
| Favourites | The emotes that player starred. |
| Recent | The emotes that player played last. |
| Gestures & Reactions | Waves, claps, points, insults, greetings. |
| Conversation | Talking, listening, nodding, arguing. |
| Idle & Leaning | Standing about, leaning on walls, rails and posts. |
| Sitting & Laying | Sitting on the ground, laying down, reading, playing guitar. |
| Working | Sweeping, chopping, repairing, feeding animals. |
| Eat, Drink & Smoke | Bottles, coffee, stew, cigarettes and cigars. |
| Dance | Dancing, including the fire and sword dances. |
| Injury | Limping, clutching wounds, collapsing, passing out. |

Which categories appear, and their order, is `Config.Menu.CategoryOrder`.
Remove one to hide it and every emote in it.

---

## The Nearby tab

Chairs, benches, bars, pianos, water pumps, stoves, campfires and railings
all carry **scenario points**: the spots the game's own NPCs sit, stand and
work at. The Nearby tab asks the game which points are around the player and
lists them as actions, grouped by the thing they belong to, closest first:

```
CHAIR - 1.2 m
  Sit down
  Sit and drink
  Sit at the table
SPOT - 2.4 m
  Stand at the bar
WALL - 0.6 m
  Lean back on the wall
  Lean on the wall (left side)
```

Pick one and the character walks over, turns, and plays the enter animation
the way an NPC would. Nothing is teleported. `Backspace`, `/ec` or **Stop
emote** ends it, whether the character is still walking or already sitting.
Any prop the game handed out on the way (a broom, a pitchfork, the hay on it)
is put away with it, and one the exit animation dropped on the ground is
removed too.

With the mouse off the menu the player keeps their controls and can walk
about with the menu open; the list follows them. Over the menu, the game gets
no input. There is no preview character on this tab. A dev (an admin, or
anyone with the ACE `poggy_emotes.nearby.block`) sees an `×` on every row:
click, choose **Here only** or **Everywhere**, and that action is off the tab
for everyone. The block is written into `Config.Nearby.Blocked` in
`config.lua`, so it survives updates and ports with the file. That is how the
scenarios that are broken in custom-built areas get found and removed in play.

Two config lists add actions for things the game has no point for:
`Config.Nearby.Keywords` (a scenario for any prop whose model name contains a
word: lean on any barrel, rummage in any crate, knock on any door) and
`Config.Nearby.Wall` (a wall found beside the player). Everything is under
`Config.Nearby`, and in `/poggy` on the Nearby tab. The help page **The
Nearby tab** covers naming, hiding and adding actions.

---

## Adding your own emotes

Put them in `Config.CustomEmotes` in `config.lua`, or add them from the Emotes
tab in `/poggy`. An update never touches that list.

```lua
Config.CustomEmotes = {
    { name = "salute", label = "Salute", category = "gestures", gender = "any",
      kind = "kit", hash = 901097731 },
}
```

Giving a custom emote the same `name` as one that ships replaces it, which is
how you re-label, re-categorise or re-time a shipped emote without editing code.

Full instructions, including prop emotes and where to find animation names, are
in `docs/help/adding-emotes.md`.

---

## Hiding emotes

```lua
Config.HiddenEmotes = { "pee", "vomitkneel" }
```

A hidden emote is gone from the menu **and** from `/e`, so a player who knows
the name still cannot play it. Use the name in brackets in the menu, not the
label.

---

## Configuration

Everything is in `config.lua`, and every setting is in `/poggy` with a tooltip.
The ones most worth knowing:

| Setting | What it does |
|---|---|
| `Config.Behaviour.CancelOnMove` | Ends the emote when the player walks off. On by default. |
| `Config.Behaviour.AllowWhileMounted` | Lets emotes play on a horse or wagon. Off by default: most animations look broken mounted. |
| `Config.Behaviour.HolsterWeapons` | Puts weapons away before an emote starts. |
| `Config.Ragdoll.Enabled` / `Key` | The ragdoll key: on or off, and which key. |
| `Config.Ragdoll.MinSeconds` / `Cooldown` | The least time down before the key lets you up (2 s), and the wait before you can drop again (3 s). |
| `Config.Menu.StayOpenAfterPlay` | Where the **Keep menu open** switch starts for a new player. After that it is the player's own choice. |
| `Config.Menu.FilterByGender` | Hides emotes built for the other gender. |
| `Config.Menu.RecentCount` | How many recent emotes to remember. 0 turns Recent off. |
| `Config.Preview.Enabled` | The stand-in preview character. |
| `Config.Preview.OffsetRight` / `OffsetForward` | Where the preview stands, in metres. |

---

## Database

One table, `poggy_emotes_player`: one row per character holding two lists, the
emotes they starred and the ones they played last.

poggy_core creates it when the script starts, so there is nothing to import. To
manage it yourself instead, set `PoggyCoreConfig.Sql.AutoInstall = false` in
`poggy_core/config.lua` and import `sql/install.sql`, or run
`poggycore sql install poggy_emotes` in the server console.

---

## Translations

Every word a player sees is in `translations.lua`, including the menu itself and
the category names. Copy the `["en"]` block, rename it, translate it, and put
its name in `Config.Language`.

---

## Upgrading from another emote script

If you ran **js_emotes**, your players' favourites come across on the first
start: the old `emote_favorites` table is read once and folded into the new one.
The old table is left exactly as it is, so nothing is lost.

`/e <name>` keeps working for every emote name that script used, so players do
not have to relearn anything and existing macros keep working.

More in `docs/help/upgrading.md`.

---

## Troubleshooting

| Problem | Cause |
|---|---|
| **The menu does not open.** | Check the server console: it says if `Config.Keys.OpenMenuKey` is not a key it knows, or if this RedM build cannot read the keyboard directly. `/emotes` always works. |
| **An emote does nothing.** | The animation dictionary is not in your game build, or a custom emote has a typo in `dict` or `anim`. The server console says which. |
| **A prop is stuck in someone's hand.** | Press `Backspace` or type `/ec`. If it happens repeatedly, please report it: every path out of an emote is supposed to clean the prop up. |
| **The preview character is missing.** | `Config.Preview.Enabled` is off, or `Config.Preview.Model` is not a valid ped model. The server console says if the model failed to load. |
| **The preview stands in a wall.** | Move it with `Config.Preview.OffsetRight` and `OffsetForward`. |
| **An emote is in the wrong category.** | Add a row to `Config.CustomEmotes` with the same `name` and the category you want. |
| **Emotes stop the moment a player moves.** | That is `Config.Behaviour.CancelOnMove`. Turn it off to make emotes hold. |
