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
| `Config.UPDATE_INTERVAL_MS` | `500` | How often positions are sent to admins, in milliseconds. Lower is smoother but costs more. |
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
Lower `Config.UPDATE_INTERVAL_MS`. This sends more network traffic.

**`/ahb` says I have no permission.**
Your group is not in `Config.ADMIN_GROUPS`.

**A player has no blip.**
Players only get a blip once their character has loaded and they have spawned. Until then they are skipped.

---

## Changelog

- **1.2.2**: Every setting in `config.lua` is now read by the script (before, only the admin groups and allowed names were; the rest used fixed values). The shipped update interval is 500 ms, the value the script actually used. Hub page and in-game settings support (`/poggy`).
- **1.2.1**: Net events renamed from `vorp_admin_blips:*` to `poggy_admin_blips:*`; nothing framework-specific is left in the resource. Admin groups and character names come from poggy_core (`perms.groups`, `char.get`), so it runs on any framework poggy_core supports.

---

## Credits and licence

- **Author:** Poggy
- **Version:** 1.2.2

Free Poggy script, provided as-is for use on RedM servers.
