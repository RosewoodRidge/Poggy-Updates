# Poggy Scene

Roleplay text tools for RedM.

Players place coloured floating text in the world, wear a status tag above their head, and send `/me` and `/do` actions. Nearby `/me` messages also collect in a tidy on-screen box so nobody misses them.

---

## Contents

1. [Features](#features)
2. [Requirements](#requirements)
3. [Installation](#installation)
4. [Commands](#commands)
5. [Scene Editor](#scene-editor)
6. [Status Editor](#status-editor)
7. [The /me box](#the-me-box)
8. [Removing a scene](#removing-a-scene)
9. [Managing scenes in /poggy](#managing-scenes-in-poggy)
10. [Permissions](#permissions)
11. [Configuration](#configuration)
12. [UI themes](#ui-themes)
13. [Troubleshooting](#troubleshooting)

---

## Features

| Feature | What it does |
|---|---|
| **Scene text** | Coloured, sized text fixed to a spot in the world. Everyone nearby sees it. It stays through reconnects and server restarts. |
| **Status tag** | Coloured text that stays above your character. Save the ones you use often as presets. |
| **/me and /do** | In-character actions and scene descriptions, floating above your character. |
| **/id and /cash** | Show your server ID or your cash above your character for a few seconds. |
| **/me box** | A see-through box near the top of the screen that collects nearby `/me` messages. |
| **Scenes panel** | Every placed scene in one table in `/poggy`: who placed it, where and when. Fix its text, delete one, everything by one player, or everything older than a number of days. |

---

## Requirements

| Resource | Why |
|---|---|
| **poggy_core** 0.13.0 or newer | Characters, jobs, groups, cash and notifications, on VORP, RSG or QBR |
| [oxmysql](https://github.com/overextended/oxmysql) | Stores scenes and status presets |
| `/assetpacks` | Built into the server (FXServer); nothing to install |

---

## Installation

1. Put the `poggy_scene` folder in your `resources` folder.
2. Add it to `server.cfg`, after poggy_core and oxmysql:
   ```
   ensure poggy_core
   ensure poggy_scene
   ```
3. Restart the server.

The scene and status tables are created for you when the script starts. There is nothing to import.

If you manage the database yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua` and import `sql/install.sql`.

---

## Commands

| Command | Example | What it does |
|---|---|---|
| `/scene` | `/scene` | Opens the Scene Editor to place world text. |
| `/me` | `/me tips his hat` | An in-character action. Floats above you and goes into nearby `/me` boxes. |
| `/do` | `/do The wanted poster is torn` | A scene description, in red above you. |
| `/status` | `/status Injured, left arm` | Sets a status tag above you. Alone, `/status` opens the Status Editor. |
| `/cstatus` | `/cstatus` | Clears your status tag. |
| `/id` | `/id` | Shows your server ID above you. |
| `/cash` | `/cash` | Shows your cash above you. |
| `/mebox` | `/mebox persist` | Controls your `/me` box. See [The /me box](#the-me-box). |

Everyone can use every command. `/scene` can be limited to some jobs (see [Permissions](#permissions)).

All command names can be changed in the config.

---

## Scene Editor

Scenes are lines of text fixed to a spot in the world. Anyone close enough can read them.

### Step 1: write the text

1. Type `/scene`. The Scene Editor opens in the bottom-right corner.
2. Type your text in the box.
3. **Size:** pick a font size from the dropdown on the toolbar.
4. **Colour:** click a colour square. Type after clicking and the text is that colour. You can change colour mid-line.
5. Select text you already typed, then click a colour or size, to change it.
6. Click **Confirm →**.

Text placed in the world is shown in **capitals**. That is how the game draws it.

### Step 2: position it

The text now floats in the world so you can see it.

- Click the **◄ ►** arrows on X (left/right), Y (forward/back) and Z (up/down).
- Scroll the mouse wheel over a value to fine-tune.
- Type a number and press **Enter**.
- Click and hold outside the panel to look around.

Click **✓ Place** to save it. Click **✕ Cancel** to go back to the text without losing it.

Scene text can be read from 5 metres away by default (`Config.scenevisabilitydistance`).

---

## Status Editor

A status tag floats above your character until you clear it. Use it for injuries, moods or RP tags.

- **Quick:** `/status Your text` sets it at once, with no colours.
- **Full editor:** `/status` on its own opens the Status Editor.
  1. Type your status.
  2. Pick colours from the toolbar.
  3. Optional: type a preset name and click 💾 to save it.
  4. Click **▶ Apply**.

**Presets** are listed at the bottom. Click a name to load it, **▶** to apply it, **✕** to delete it. Presets are saved per character.

**Clear it** with **■ Clear** in the editor, or `/cstatus`.

---

## The /me box

A see-through box near the top of the screen that collects `/me` messages from players near you. Your own `/me` messages always appear.

Use `/mebox <option>`:

| Option | Effect |
|---|---|
| `auto` | Shows when a `/me` arrives, fades after 30 seconds (default). **This is the default mode.** |
| `persist` | Stays on screen while it has messages. |
| `off` | Hidden. |
| `small` / `normal` / `large` | Box width. |
| `move` | Drag the box anywhere. Press **Esc** to save the spot. |

Each player's choice is remembered on their own PC.

**How messages behave**

- Each message is deleted after **5 minutes** (`Config.meboxdissipateafter`).
- A faded box comes back with its messages when you press **T** to chat.
- Scroll with the mouse wheel over the box to read older messages.

---

## Removing a scene

Stand next to a scene. The hint **Press 4 To Remove** appears. Press **4**.

The scene is removed for everyone.

---

## Managing scenes in /poggy

The main admin tool is the **Scenes** tab in `/poggy` (poggy_core's settings
hub): **/poggy** → **Poggy Scene** → **Scenes**. It lists every scene on the
server with its text, who placed it (name and character id), where, and when.

| Do this | How |
|---|---|
| Fix a scene's text | Click the text and edit it. Colour codes (`~e~`, `~t6~` …) work as in the editor. |
| Delete one scene | The row's **Delete** button. |
| Delete everything one player placed | The row's **Delete all by this player**. You type the scene id to confirm. |
| Clean up old scenes | **Delete all older than…** at the top: a number of days. You type `CONFIRM`. |

Every change shows for every player at once, the same as removing a scene with
**4**, and is printed to the server console with who made it. Only people who
may edit settings in `/poggy` can use the panel.

Scenes do not expire on their own. Placing times are recorded from version
1.3.2; older scenes show no date, and **Delete all older than…** keeps them
unless you choose to delete them too.

---

## Permissions

| Who | Place scenes | Remove own scenes | Remove anyone's scenes |
|---|---|---|---|
| Everyone (Job lock off) | Yes | Yes | No |
| Everyone (Job lock on, Allowed jobs set) | Only jobs in `Config.allowjobs` | Yes | No |
| Jobs in `Config.removejobs` | Yes | Yes | Yes |
| Players in the `admin` group | Yes | Yes | Yes |

- Job names are exact and case-sensitive.
- With Job lock on but **Allowed jobs** empty, everyone can still place scenes.
- The `admin` group name is fixed in the script. poggy_core checks both the account group and the character group.
- Every removal is checked again on the server.

No ACE is needed.

---

## Configuration

Every setting can be changed in game with **/poggy** (Poggy Hub). You can also edit `config.lua` by hand. Restart the script after a change.

### Scenes and permissions

| Setting | Default | What it does |
|---|---|---|
| `Config.scenevisabilitydistance` | `5` | How close (metres) you must be to read scene text and status tags. |
| `Config.joblock` | `false` | Only allowed jobs can place scenes. |
| `Config.allowjobs` | `{}` | Jobs that may place scenes while Job lock is on. |
| `Config.removejobs` | `police`, `sheriff`, `marshal` | Jobs that can remove any scene. |
| `Config.denysceneinhideout` | `false` | Blocks `/scene` within 100 m of one fixed hideout spot (1785, -821, 191). |

### Commands

| Setting | Default |
|---|---|
| `Config.scenecommand` | `"scene"` |
| `Config.mecommand` | `"me"` |
| `Config.ooccommand` | `"do"` |
| `Config.statuscommand` | `"status"` |
| `Config.stopdisplay` | `"cstatus"` |
| `Config.id` | `"id"` |
| `Config.cash` | `"cash"` |
| `Config.meboxcommand` | `"mebox"` |

### Overhead text (`/me`, `/do`, `/id`, `/cash`)

| Setting | Default | What it does |
|---|---|---|
| `Config.meoverheadduration` | `10` | Seconds the text floats above a character. |
| `Config.meoverheadradius` | `20` | How far away (metres) it can be seen. |

### /me box

| Setting | Default | What it does |
|---|---|---|
| `Config.meboxdistance` | `6` | How close (metres) someone must be for their `/me` to reach your box. |
| `Config.meboxtimeout` | `30` | Seconds before the box fades in auto mode. |
| `Config.meboxmaxlines` | `7` | Lines shown before it scrolls. |
| `Config.meboxhistory` | `50` | Messages kept for scrolling back. |
| `Config.meboxdissipateafter` | `300` | Seconds before a message is deleted. |
| `Config.meboxDefaultNameColor` | `"#eeebe4"` | Name colour when the sender's job has none. |
| `Config.meboxJobColors` | law and doctor | Name colour per job. |

### Look and text

| Setting | What it does |
|---|---|
| `Config.sceneColors` | The colour swatches in the editors, in order. Each has a game colour `code`, a `hex` colour for the button and a `title`. |
| `Config.uitheme` | Editor colour scheme. See below. |
| `Config.Language` | Player messages and the prefixes for `/do`, `/status`, `/cash` and `/id`. |

`Config.webhook` is in the file but not used: nothing is sent to Discord.

---

## UI themes

Set `Config.uitheme`. Players see it after the script restarts.

| Theme | Look |
|---|---|
| `"amber-frontier"` | Old West gold and amber (default) |
| `"steel-blue"` | Dark navy with steel-blue accents |
| `"ivory-noir"` | Black and white |
| `"lavender-dusk"` | Soft lavender and purple |
| `"crimson-dusk"` | Deep red |
| `"emerald-ridge"` | Forest green |

The `/me` box is not themed. It always matches the chat overlay style.

---

## Troubleshooting

**"Cant place scene in hideout" but I am not near a hideout.**
The same message is shown when Job lock blocks you. Check `Config.joblock` and `Config.allowjobs`.

**I cannot remove someone else's scene.**
Only jobs in `Config.removejobs` and the `admin` group can. Job names are case-sensitive. Server staff can also delete any scene from the Scenes panel in `/poggy`.

**Scenes vanished after a restart.**
Scenes are loaded from the database when the script starts. Check that oxmysql is running and look for `loaded N scene(s) from DB` in the server console.

**Colours look different in the world than in the editor.**
The world uses the game's own colour codes. The editor swatch is only a close match. Adjust `hex` in `Config.sceneColors` to match better.

---

## Files

```
poggy_scene/
├── fxmanifest.lua
├── config.lua          settings and player text
├── README.md
├── client/main.lua     editors, overhead text, /me box
├── server/main.lua     scenes, statuses, permissions
├── server/hub.lua      the Scenes panel in /poggy
├── sql/install.sql     tables (applied automatically)
├── docs/               Poggy Hub card and help pages
└── ui/                 Scene and Status editors, /me box
```
