# Poggy Admin Blips

See every player on the map, live. Each blip shows the player's server ID and character name.

Only staff see the blips. Players never know they are there.

---

## Features

- **Live player tracking.** Every player on the map, with server ID and character name.
- **Admin groups.** Access comes from your framework's admin groups, through poggy_core.
- **Optional name list.** Limit blips to named admins only.
- **Your own look.** Change the icon, colour and label of the blips.
- **Show or hide.** Each admin can turn the blips on and off with `/ahb`.
- **Hide your own blip.** You do not see a blip on top of yourself.
- **Works on VORP, RSG and QBR** through poggy_core.

---

## Requirements

| Resource | Why |
|---|---|
| `poggy_core` | Framework bridge: admin groups and character names. Must start first. |

No database tables. Nothing to import.

---

## Installation

1. Put the `poggy_admin_blips` folder in your server's `resources` folder.
2. Add this line to `server.cfg`, **after** `ensure poggy_core`:
   ```
   ensure poggy_admin_blips
   ```
3. Restart the server.
4. Check the admin groups (see **Configuration**). The defaults are `admin` and `superadmin`.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/ahb` | Admins | Shows or hides the player blips on your own map. Blips start visible when you join. |
| `/abtest` | Admins, server console | Shows what each blip really is, and can switch the method live. See **Blip methods**. Name set by `Config.TEST_COMMAND`; `false` removes it. |

"Admins" means a player in one of the **Admin groups** (`Config.ADMIN_GROUPS`).

---

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy settings hub). You can also edit `config.lua` by hand. Restart the script after a change.

| Setting | Default | What it does |
|---|---|---|
| `Config.ADMIN_GROUPS` | `{ "admin", "superadmin" }` | Groups whose members see the blips and may use `/ahb`. Case does not matter. |
| `Config.ALLOWED_NAMES` | `{}` | Optional. When it has names, only these players get blips, and they must also be in an admin group. Exact Steam / RedM display name, case-sensitive. |
| `Config.BLIP_SPRITE` | `"blip_ambient_companion"` | The map icon for each player. |
| `Config.BLIP_MODIFIER` | `"BLIP_MODIFIER_DEBUG_GREEN"` | The blip colour. |
| `Config.BLIP_STYLE` | `"BLIP_STYLE_ENEMY"` | The style the blip is created with. Rarely changed. |
| `Config.BLIP_NAME_FORMAT` | `"{id} \| {name}"` | The blip label. `{id}` = server ID, `{name}` = character name. |
| `Config.HIDE_OWN_BLIP` | `true` | Hide your own blip from yourself. |
| `Config.METHOD` | `"hybrid"` | How the blips work. See **Blip methods** below. |
| `Config.UPDATE_INTERVAL_MS` | `2000` | How often positions are sent to admins, in milliseconds. With `hybrid` they are only used for players out of range. |
| `Config.GLIDE` | `true` | An out-of-range blip glides from one position to the next instead of jumping. |
| `Config.TEST_COMMAND` | `"abtest"` | The staff command that shows what each blip really is and can switch method live. `false` removes it. |
| `Config.INIT_WAIT_TIME` | `5000` | How long to wait after a player joins before the admin check, in milliseconds. |
| `Config.PENDING_RETRY_INTERVAL` | `5000` | How often players still loading are checked again, in milliseconds. |
| `Config.DEBUG` | `false` | Print debug lines to the server console. |

### Blip colours

| Value | Colour |
|---|---|
| `BLIP_MODIFIER_DEBUG_GREEN` | Green |
| `BLIP_MODIFIER_DEBUG_RED` | Red |
| `BLIP_MODIFIER_DEBUG_BLUE` | Blue |
| `BLIP_MODIFIER_DEBUG_YELLOW` | Yellow |

### Examples

Allow a moderator group too:

```lua
Config.ADMIN_GROUPS = {
    "admin",
    "superadmin",
    "moderator",
}
```

Only two named admins get blips (they must still be in an admin group):

```lua
Config.ALLOWED_NAMES = {
    "PlayerName1",
    "PlayerName2",
}
```

Red blips labelled `[1] John Smith`:

```lua
Config.BLIP_MODIFIER = "BLIP_MODIFIER_DEBUG_RED"
Config.BLIP_NAME_FORMAT = "[{id}] {name}"
```

---

## Permissions

| Who | How |
|---|---|
| See the blips | Be in a group listed in `Config.ADMIN_GROUPS`. If `Config.ALLOWED_NAMES` has names, also be on that list. |
| Use `/ahb` | Be in a group listed in `Config.ADMIN_GROUPS`. |

There are no ACE permissions to set.

---

## Troubleshooting

**Blips do not appear.**

1. Set `Config.DEBUG = true` and restart the script.
2. Rejoin and read the server console. It shows the groups found for you and whether you were registered as an admin.
3. Check your group is in `Config.ADMIN_GROUPS`.
4. If you use `Config.ALLOWED_NAMES`, check your name matches exactly, including capitals.
5. If no groups are found, your character may load slowly. Raise `Config.INIT_WAIT_TIME`.

**Blips move in jumps.**
Only an out-of-range blip can: a player in range is attached to their ped and moves every frame. Check `Config.GLIDE` is on, or lower `Config.UPDATE_INTERVAL_MS` (more network traffic). With the `coords` method every blip moves at that rate.

**`/ahb` says I have no permission.**
Your group is not in `Config.ADMIN_GROUPS`.

**A player has no blip.**
Players only get a blip once their character has loaded and they have spawned. Until then they are skipped.

---

## Blip methods

A blip can only be attached to a player's ped while that ped exists on the
admin's machine, and the server only streams the peds near you (about 424 units).
Blips are not shared between players either: each admin's game draws its own. So
the default method is a hybrid:

| Method | What it does |
|---|---|
| `hybrid` | In range: the blip is attached to the ped, the game moves it every frame, and it costs no network traffic. Out of range: a coordinate blip from the server, gliding between updates. **Recommended.** |
| `coords` | Every blip is a coordinate blip moved by the server. How the script worked before 1.3.0. |

A twice-a-second check swaps a blip from one kind to the other as a player comes
into range, leaves it, or respawns.

### Checking it: `/abtest`

Staff only (and the server console).

```
/abtest status          what each blip really is, and how far away
/abtest coords          switch method: hybrid or coords
/abtest rate 500        how often positions are sent, in milliseconds
/abtest glide off       jump instead of glide
```

`status` counts blips **attached to a ped** against blips **fed by
coordinates**, gives the distance of the farthest attached one, and how many
updates the server sent in the last minute. The full list prints to your F8
console.

Changes made with `/abtest` last until the script restarts, and apply to every
admin. To keep one, set it in the config.

---

## Changelog

- **1.3.0**: Blips attach to the player's ped when they are in range, so they move with the game instead of being moved by the server (`Config.METHOD = "hybrid"`, the new default). Players out of range still get a coordinate blip, now gliding between updates (`Config.GLIDE`), and the shipped update interval is 2000 ms because only those players need it. `/abtest` (`Config.TEST_COMMAND`) shows what each blip really is and switches method live. `Config.METHOD = "coords"` with `Config.UPDATE_INTERVAL_MS = 500` is the old behaviour exactly.
- **1.2.2**: Every setting in `config.lua` is now read by the script (before, only the admin groups and allowed names were; the rest used fixed values). The shipped update interval is 500 ms, the value the script actually used. Hub page and in-game settings support (`/poggy`).
- **1.2.1**: Net events renamed from `vorp_admin_blips:*` to `poggy_admin_blips:*`; nothing framework-specific is left in the resource. Admin groups and character names come from poggy_core (`perms.groups`, `char.get`), so it runs on any framework poggy_core supports.

---

## Credits and licence

- **Author:** Poggy
- **Version:** 1.2.2

Free Poggy script, provided as-is for use on RedM servers.
