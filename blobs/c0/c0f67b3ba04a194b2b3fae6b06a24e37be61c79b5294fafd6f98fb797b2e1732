# Poggy Badge

A fully configurable badge system for RedM (VORP framework). Players with qualifying jobs can attach a 3D badge prop to their character, fine-tune its placement with a real-time editor, show it to nearby players with an animation, and save/load position presets.

---

## Features

- **Job & Grade Locked** — Badges are resolved per-job and per-grade. Only characters with a matching job/grade in `config.lua` can access a badge.
- **Real-Time 3D Editor** — NUI panel with sliders for X/Y/Z position and Pitch/Roll/Yaw rotation. Changes apply instantly on the attached prop.
- **Bone Selector** — Dropdown to pick which skeleton bone the badge attaches to (collar, chest, stomach, waist, etc.). Defaults to collar/upper chest.
- **Preset System** — Save and load named presets per character, stored in MySQL. Share different setups for different outfits.
- **Show Badge Animation** — Pocket watch emote (intro → hold → outro) plays while a badge image flashes on nearby players' screens.
- **Camera Orbit** — While the editor is open, click-and-hold outside the panel to orbit/move the camera. Release to lock it again.
- **Per-Prefix Rotation Overrides** — Props with specific model name prefixes (e.g. `kh_`) can have different default rotations automatically applied.
- **Admin Prop Swap** — Admins can use `/bprop <model>` to replace the attached badge with any prop model, keeping the current position.

---

## Dependencies

| Resource | Purpose |
|----------|---------|
| poggy_core | Character data (job, grade, name) and notifications, on whichever framework it drives |
| [oxmysql](https://github.com/overextended/oxmysql) | Database queries for preset storage |

---

## Installation

1. **Copy** the `poggy_badge` folder into your server's `resources/` directory.

2. **Add to server.cfg:**
   ```
   ensure poggy_badge
   ```

   The preset table is created automatically when the script starts (`sql/install.sql`). To import the file yourself instead, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

3. **(Optional) Grant admin prop-swap access.** The `/bprop` command requires the `command` ACE:
   ```
   add_ace group.admin command allow
   ```
   Any player in `group.admin` will be able to use `/bprop`.

---

## Commands

| Command | Access | Description |
|---------|--------|-------------|
| `/badge` | Job-locked | Opens the badge editor. Toggles the editor if already open. |
| `/bprop <model_name>` | Admin only | Swaps the current badge prop with the specified model, keeping position/rotation. Opens the editor. |

---

## Configuration

All configuration lives in `config.lua`.

### General

| Key | Type | Description |
|-----|------|-------------|
| `Config.command` | string | The command players type to open the badge editor (default: `"badge"`) |
| `Config.showDistance` | number | Max distance (game units) at which nearby players see the badge flash |
| `Config.serverName` | string | Server name shown on the badge display flash |
| `Config.badgeDisplayDuration` | number | Seconds the badge image displays to nearby players |
| `Config.showAnimDuration` | number | Milliseconds the show animation holds before playing the outro |

### Attachment Defaults

| Key | Type | Description |
|-----|------|-------------|
| `Config.defaultBone` | string | Skeleton bone the badge attaches to by default |
| `Config.defaultOffset` | vector3 | Default XYZ position offset |
| `Config.defaultRotation` | vector3 | Default pitch/roll/yaw rotation |

#### Available Bones

| Bone Name | Location |
|-----------|----------|
| `SKEL_Spine5` | Collar / upper chest (near neckline) |
| `SKEL_Spine4` | Upper chest |
| `SKEL_Spine3` | Mid chest (heart area) |
| `SKEL_Spine2` | Lower chest / solar plexus |
| `SKEL_Spine1` | Stomach / upper abdomen |
| `SKEL_Spine0` | Waist / belt line |
| `SKEL_SpineRoot` | Pelvis / hip center |

### Prefix Rotation Overrides

```lua
Config.prefixRotations = {
    ["kh_"] = vector3(121.75, 74.75, -36),
}
```

If a badge prop model name starts with a listed prefix, its default rotation is replaced with the specified values. Players can still override via presets.

### Badge Definitions

Badges are defined as sets in `Config.badges`. Each set has:

```lua
{
    jobs = { "JobName1", "JobName2" },  -- VORP job names
    grades = {
        [0] = {
            badge         = "silverbadge",     -- filename in ui/images/ (without .png)
            prop          = "s_badgedeputy01x", -- in-game prop model
            useName       = true,               -- show character name on flash
            useServerName = false,              -- show server name on flash
        },
        -- more grades...
    },
}
```

Multiple jobs can share the same badge set, or each job can have its own entry.

Job names are matched exactly (case-sensitive) against the character's job. A job that matches but has no entry for the player's grade gets no badge, so list every grade the job has.

The shipped config uses only the stock game badge props, mapped to the generic law jobs `sheriff`, `deputy`, `police` / `Police` (vorp_police's own job name), `marshal` / `usmarshal` and `pinkerton`:

| Prop | Badge |
|------|-------|
| `s_badgedeputy01x` | Deputy |
| `s_badgepolice01x` | Police |
| `s_badgepinkerton01x` | Pinkerton |
| `s_badgesherif01x` | Sheriff |
| `s_badgeusmarshal01x` | US Marshal |

Streamed badge packs work the same way: use their model name as `prop`, and add a `Config.prefixRotations` entry if they need a different default rotation.

---

## Badge Images

Place `.png` images in `ui/images/`. The `badge` field in each grade definition references the filename without the extension.

### Included Images

| Filename | Description |
|----------|-------------|
| `silverbadge` | Silver deputy/sheriff badge |
| `bronzebadge` | Bronze sheriff badge |
| `goldbadge` | Gold sheriff badge |
| `federal` | Federal marshal badge |
| `presidentbadge` | Presidential/senior badge |
| `specialagent` | Special agent badge |
| `doctorbadge` | Medical doctor badge |
| `cavalrybadge` | Cavalry badge |
| `pinkerton` | Pinkerton badge |
| `marshal` | Marshal badge |
| `sheriff` | Generic sheriff badge |
| `police` | Police badge |
| `silverdetective` | Silver detective badge |
| `bronzedetective` | Bronze detective badge |
| `golddetective` | Gold detective badge |
| `silvermarshal` | Silver marshal badge |
| `bronzemarshal` | Bronze marshal badge |
| `goldmarshal` | Gold marshal badge |
| `silverpolice` | Silver police badge |
| `bronzepolice` | Bronze police badge |
| `goldpolice` | Gold police badge |

To add a custom badge image, drop the `.png` into `ui/images/` and reference its name (without `.png`) in the config.

---

## Editor Controls

| Control | Action |
|---------|--------|
| **X / Y / Z sliders** | Adjust position offset (L/R, Forward/Back, Up/Down) |
| **Pitch / Roll / Yaw sliders** | Adjust rotation |
| **Mouse wheel on any slider** | Fine-tune the value |
| **Number input box** | Type an exact value |
| **Bone dropdown** | Change which skeleton bone the badge attaches to |
| **Show Badge** | Closes editor, plays the badge-flash animation to nearby players |
| **Attach / Detach** | Toggle whether the 3D prop is visible on your character |
| **Save** | Save current settings as a named preset |
| **Preset name click** | Load that preset's values |
| **✕ on preset** | Delete that preset |
| **Click-hold outside panel** | Orbit/move the camera |
| **Release click** | Lock camera back in place |
| **Escape** | Close the editor |

---

## File Structure

```
poggy_badge/
├── fxmanifest.lua      Resource manifest
├── config.lua          Configuration (commands, bones, badges, etc.)
├── README.md           This file
├── client/
│   └── main.lua        Client-side logic (attachment, NUI, animation)
├── server/
│   └── main.lua        Server-side logic (job resolution, presets, broadcasting)
├── sql/
│   └── install.sql     Database schema (applied automatically on start)
└── ui/
    ├── index.html      Editor & display NUI markup
    ├── script.js       NUI logic & event handlers
    ├── style.css       Dark western theme styling
    └── images/         Badge PNG images
        ├── silverbadge.png
        ├── goldbadge.png
        └── ... (21 images)
```

---

## Upgrading

Upgrading from a version without the bone selector needs nothing: the `bone` column is added automatically when the script starts. Existing presets will default to `SKEL_Spine5` (collar/upper chest).

---

## Technical Notes

- **Rotation Whole Numbers** — The RDR3 engine has a bug where rotation axes snap/reset at exact integer values. The script automatically nudges rotation values by +0.1 when they land on a whole number. Slider displays show the clean number; the actual value sent to the engine is offset.
- **Animation** — Uses the `script_mp@emotes@check_pocket_watch@female@unarmed@upper` animation dictionary with `intro` and `outro` clips. The dict is pre-warmed when badge data arrives to avoid first-use delays.
- **Attachment** — Uses `AttachEntityToEntity` with `isPed = true` (required for rotation to work on ped bones) and `useSoftPinning = false` (rigid attachment).
- **Camera Control** — NUI focus is toggled between `SetNuiFocus(true, true)` (cursor visible, camera locked) and `SetNuiFocus(true, false)` + `SetNuiFocusKeepInput(true)` (cursor hidden, camera orbits) via JS mousedown/mouseup events.
