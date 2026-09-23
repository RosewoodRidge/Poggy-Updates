# The Nearby tab

The first tab in the emote menu is not a list of emotes. It is a list of what
the player can **do with what is around them**: sit on that chair, stand at
the bar, lean on that railing, pump that water, knock on that door.

Pick one and the character walks over, turns, and does it. Nothing is
teleported. `Backspace`, `/ec` or the **Stop emote** button ends it, whether
the character is still walking or already sitting.

---

## Where the rows come from

Most of them come from the game itself. Red Dead's props and places carry
**scenario points**, the spots the game's own NPCs use: the seats of every
chair and bench, the standing spots at every bar, the pumps, stoves,
campfires, pianos and railings. The script asks the game which points are near
the player and lets the character use them the way an NPC would. There are no
lists of chair models and no seat offsets to tune: if the game put a seat on
it, the player can sit on it.

Two config lists add actions for things the game has no point for:

| List | What it adds |
|---|---|
| `Config.Nearby.Keywords` | A scenario for any prop whose model name contains a word: lean on any barrel, rummage in any crate, knock on any door. |
| `Config.Nearby.Wall` | A wall found beside the player: lean back on it, lean a shoulder on it, brace against it. |

The rows are grouped by the thing they belong to, closest thing first, so a
chair's three ways of sitting stay together under **Chair · 1.2 m**.

---

## Two-part actions

Some of the game's points are the first of a **chain**: pick the bale up,
carry it, set it down at a linked point. A row like that is marked **carry**
and, with `Config.Nearby.Chains` on, runs the whole chain the way an NPC
would. Whatever the character sets down at the end is left where the game
put it; only something still in the hand when the action stops is removed.

---

## Naming things

The game's scenario types have names like `PROP_HUMAN_SEAT_CHAIR_DRINKING`.
`Config.Nearby.Labels` turns the common ones into words:

```lua
{ scenario = "PROP_HUMAN_SEAT_CHAIR_DRINKING", label = "Sit and drink" },
```

Anything not in the list is named from the scenario itself, so
`WORLD_HUMAN_CLEAN_TABLE` shows as **Clean Table**. To find the name of a
scenario you want to label, turn on `ShowUnknown` for a moment and stand
next to the thing.

---

## Walking about with the tab open

With the mouse **off** the menu the player keeps their controls: they can
walk and look about while the menu is open, and on the Nearby tab the list
follows them, looking again whenever they move. With the mouse **over** the
menu the game gets nothing, so a click on a row is only a click on a row.
A marker (`Config.Nearby.Marker`, a soft pink circle by default) sits on
whatever the highlighted row belongs to, so "Sit down" points at its chair.
The preview character is not shown on the Nearby tab. `Tab` jumps to the
next thing in the list, `Enter` does the highlighted action, `Esc` closes.

Typing only searches with the mouse over the menu and on a tab other than
Nearby, because off the menu `W` is for walking.

---

## Blocking a broken one in play

Some of the game's scenarios are badly broken in custom-built areas: a seat
that puts the character through the floor, a lean that faces the wrong way.
A **dev** sees a small `×` on every Nearby row. Click it and the row asks
**Here only** or **Everywhere**. Here only blocks that action at that one
spot (within `Config.Nearby.BlockRadius` metres); Everywhere blocks the
scenario type on every prop. Either way it is off the tab **for everyone,
at once**, and it is written into `Config.Nearby.Blocked` in `config.lua`:

```lua
Blocked = {
    { scenario = "PROP_HUMAN_SEAT_CHAIR", x = 2412.35, y = -1210.80, z = 44.21 }, -- Poggy, 2026-09-23
    { scenario = "WORLD_HUMAN_LEAN_WALL_LEFT" },
},
```

So the list is kept with the config, an update never touches it, and it
ports with the file. Rows can be added or removed by hand too. The row stays
visible to devs, crossed out, with `↺` to allow it again.

Who is a dev is set under **Blocking in game**: anyone poggy_core counts as an
admin, and anyone with the ACE `poggy_emotes.nearby.block`:

```
add_ace group.dev poggy_emotes.nearby.block allow
```

Use Here only for a scenario that is fine elsewhere and broken in one
custom-built room. Use Everywhere when the scenario itself is wrong on every
copy of the prop.

---

## Hiding things

`Config.Nearby.Prefixes` is the allow-list: only scenario types whose name
starts with one of those prefixes are offered. The defaults keep the human
and player ones and drop the animal, ambient and cutscene types.

`Config.Nearby.Hidden` is a list of patterns that are never shown, whatever
the prefix. Add a scenario there to take it away.

To take the whole tab away, set `Config.Nearby.Enabled = false`, or remove
`"nearby"` from `Config.Menu.CategoryOrder` on the Menu tab.

---

## Adding a keyword action

```lua
{ keywords = { "anvil" }, label = "Hammer", scenario = "WORLD_HUMAN_HAMMER_GROUND", distance = 0.3, face = "toward" },
```

`keywords` is plain text that must appear in the prop's model name.
`distance` is metres from the **edge** of the prop (its bounding box), not its
centre. `face = "away"` puts the character's back to the prop, for leaning and
sitting on edges.

A keyword action is skipped on a prop that already has a native point of the
same scenario, so nothing shows twice.

---

## When it says "You cannot get there"

If the character goes `Config.Nearby.WalkTimeout` seconds without getting
any closer to the spot, the action is cancelled and the message shows. The
clock counts from the last step of progress, not from the click, so a long
way round a table is fine. Something in the way, or a spot a person cannot
stand on, is what trips it. Move closer, or pick the thing from the other
side. `Config.Nearby.Debug` prints what the walk sees in the F8 console.
