# Poggy Badge

Give your lawmen, doctors and officials a real 3D badge.

Players with a listed job and grade can pin a badge prop to their chest. They place it with a live editor and save the spot as a preset. **Show Badge** plays a hand-held animation and flashes the badge image on the screens of players close by.

---

## Features

- **Job and grade locked.** Each job and grade gets its own badge image and prop. Nobody else can open the editor.
- **Live 3D editor.** Sliders for X / Y / Z position and pitch / roll / yaw. The badge moves as you drag.
- **Bone picker.** Pin the badge to the collar, chest, stomach, waist or hip.
- **Presets.** Save named positions per character, one for each outfit. Stored in the database.
- **Show Badge.** A pocket-watch style animation with the badge in hand. Nearby players see the badge image, the character's name and your server name.
- **Camera orbit.** Click and hold outside the panel to look around; let go to lock the camera again.
- **Streamed badge packs.** Use any prop model. Fix packs that face the wrong way with a rotation override.
- **Admin prop swap.** `/bprop <model>` tries any prop in the current position.

---

## Requirements

| Resource | Why |
|---|---|
| **poggy_core** 0.13.0 or newer | Character data (job, grade, name) and notifications, on VORP, RSG or QBR |
| [oxmysql](https://github.com/overextended/oxmysql) | Stores badge presets |

---

## Installation

1. Put the `poggy_badge` folder in your `resources` folder.
2. Add it to `server.cfg`, after poggy_core and oxmysql:
   ```
   ensure poggy_core
   ensure poggy_badge
   ```
3. Restart the server.

The preset table is created for you when the script starts. There is nothing to import.

If you manage the database yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua` and import `sql/install.sql`.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/badge` | Jobs listed in the badge sets | Opens the badge editor. Type it again to close it. |
| `/bprop <model>` | ACE `command` | Swaps the badge for any prop model, keeps the position and opens the editor. |

The `/badge` name can be changed (`Config.command`). `/bprop` is fixed.

---

## Configuration

Every setting can be changed in game with **/poggy** (Poggy Hub). You can also edit `config.lua` by hand. Restart the script after a change.

### General

| Setting | Default | What it does |
|---|---|---|
| `Config.command` | `"badge"` | The command that opens the editor. |
| `Config.showDistance` | `2` | How close (metres) others must be to see the badge flash. |
| `Config.serverName` | `"Your Server"` | Shown under the name on the badge flash. Leave empty to hide it. |
| `Config.showAnimDuration` | `5000` | How long (milliseconds) the badge is held up. |
| `Config.badgeDisplayDuration` | `8` | Reserved. The flash currently always shows for 5 seconds. |
| `Config.debug` | `false` | Prints badge lookups to the server console. |

### Starting position

| Setting | What it does |
|---|---|
| `Config.defaultBone` | The bone a badge starts on. |
| `Config.defaultOffset` | Starting position from the bone (metres). |
| `Config.defaultRotation` | Starting rotation (degrees). |
| `Config.prefixRotations` | A different starting rotation for props whose name starts with a prefix, for example `kh_`. |

Players move the badge from there with the editor.

| Bone | Where it sits |
|---|---|
| `SKEL_Spine5` | Collar / upper chest (default) |
| `SKEL_Spine4` | Upper chest |
| `SKEL_Spine3` | Mid chest (heart) |
| `SKEL_Spine2` | Lower chest |
| `SKEL_Spine1` | Stomach |
| `SKEL_Spine0` | Waist / belt |
| `SKEL_SpineRoot` | Pelvis / hip |

### Badge sets

`Config.badges` is a list of sets. Each set gives one or more jobs a badge per grade.

```lua
{
    jobs = { "sheriff" },
    grades = {
        [0] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
        [1] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
    },
},
```

| Field | What it is |
|---|---|
| `jobs` | Job names that use this set. Matched exactly, case-sensitive. |
| `grades` | One entry per grade number. A grade with no entry gets no badge. |
| `badge` | Image for the flash: a file in `ui/images/`, without `.png`. |
| `prop` | The 3D prop model on the chest. |
| `useName`, `useServerName` | Reserved. The flash currently always shows both. |

The shipped sets cover `sheriff`, `deputy`, `police` / `Police` (vorp_police's name), `marshal` / `usmarshal` and `pinkerton`, grades 0 to 5.

**Stock badge props** (no streaming needed):

| Prop | Badge |
|---|---|
| `s_badgesherif01x` | Sheriff |
| `s_badgedeputy01x` | Deputy |
| `s_badgepolice01x` | Police |
| `s_badgeusmarshal01x` | US Marshal |
| `s_badgepinkerton01x` | Pinkerton |

See the help page **Adding a badge for a job** in `/poggy` for a step-by-step guide.

---

## Badge images

The flash images live in `ui/images/`. To add one, drop a `.png` in that folder and use its name (without `.png`) as `badge`.

| Image | Description |
|---|---|
| `sheriff`, `bronzebadge`, `silverbadge`, `goldbadge` | Sheriff and deputy stars |
| `marshal`, `bronzemarshal`, `silvermarshal`, `goldmarshal`, `federal` | Marshal badges |
| `police`, `bronzepolice`, `silverpolice`, `goldpolice` | Police badges |
| `bronzedetective`, `silverdetective`, `golddetective` | Detective badges |
| `pinkerton`, `specialagent`, `presidentbadge` | Agency and senior badges |
| `doctorbadge`, `cavalrybadge` | Medical and cavalry |

---

## Editor controls

| Control | Action |
|---|---|
| X / Y / Z sliders | Move left-right, forward-back, up-down |
| Pitch / Roll / Yaw sliders | Rotate |
| Mouse wheel on a slider | Fine-tune |
| Number box | Type an exact value |
| Bone list | Pick the bone the badge is pinned to |
| Show Badge | Close the editor and show the badge to nearby players |
| Attach / Detach | Put the badge on or take it off |
| Save | Save the current position as a named preset |
| Click a preset | Load it |
| ✕ on a preset | Delete it |
| Click and hold outside the panel | Orbit the camera |
| Escape | Close the editor |

---

## Permissions

`/bprop` needs the `command` ACE. `group.admin` has it on most servers. To grant it:

```
add_ace group.admin command allow
```

`/badge` needs no ACE. It is gated by job and grade only.

---

## Troubleshooting

**"You don't have a badge for your current job/rank."**
The character's job is not in any set, or their grade has no entry. Job names are case-sensitive. Turn on `Config.debug` to see the job and grade the server read.

**The badge faces the wrong way.**
Custom props often need a different rotation. Add their name prefix to `Config.prefixRotations`. See **Fixing badge placement and rotation** in `/poggy`.

**The badge prop does not appear.**
Check the `prop` name. A streamed prop needs its pack running. The client console prints `Failed to load model` when a model is wrong.

**Presets do not save.**
Presets are saved per character. Make sure a character is loaded, and that oxmysql is running.

**Upgrading from a version without the bone picker.**
Nothing to do. The `bone` column is added when the script starts. Old presets use `SKEL_Spine5`.

---

## Technical notes

- **Whole-number rotation.** The game resets a rotation axis set to a whole number, so the script nudges such values by 0.1. The sliders still show the clean number.
- **Animation.** `script_mp@emotes@check_pocket_watch@female@unarmed@upper`, clips `intro` and `outro`. It is loaded early so the first show has no delay.
- **Attachment.** `AttachEntityToEntity` with `isPed = true` (needed for rotation on ped bones) and rigid pinning.
- **Checks.** The server re-checks job and grade before every show, and filters by distance itself.

---

## Files

```
poggy_badge/
├── fxmanifest.lua
├── config.lua          settings
├── README.md
├── client/main.lua     attachment, editor, animation
├── server/main.lua     job checks, presets, badge flash
├── sql/install.sql     preset table (applied automatically)
├── docs/               Poggy Hub card and help pages
└── ui/                 editor and badge flash (images in ui/images)
```
