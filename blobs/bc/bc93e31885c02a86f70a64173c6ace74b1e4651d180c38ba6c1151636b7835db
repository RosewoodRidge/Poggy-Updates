# poggy_scene

A RedM roleplay utility resource that lets players place floating scene text in the world, set persistent status labels on their character, send in-character `/me` and `/do` actions, and view nearby `/me` messages in a tidy on-screen box.

---

## Table of Contents

1. [Features at a Glance](#features-at-a-glance)
2. [Commands](#commands)
3. [Scene Editor — Step by Step](#scene-editor--step-by-step)
4. [Status Editor — Step by Step](#status-editor--step-by-step)
5. [ME Display Box](#me-display-box)
6. [Removing a Scene](#removing-a-scene)
7. [Configuration Reference](#configuration-reference)
8. [UI Themes](#ui-themes)

---

## Features at a Glance

| Feature | What it does |
|---|---|
| **Scene text** | Place coloured, size-formatted floating text anchored to a spot in the world. Visible to all nearby players. Persists across reconnects and server restarts. |
| **Status label** | Attach a persistent coloured text tag above your character that all nearby players see. Save commonly-used statuses as presets. |
| **/me / /do** | Send in-character action and description messages. Text floats above your character with a dark background and also appears in the ME Box. |
| **/id / /cash** | Show your server ID or current cash amount as a floating label above your character. |
| **ME Display Box** | A semi-transparent chat box near the top of the screen that collects nearby `/me` messages so you never miss them even if the overhead text scrolls away. |

---

## Commands

| Command | Usage | Description |
|---|---|---|
| `/scene` | `/scene` | Opens the Scene Editor to place world text. |
| `/me` | `/me texts a note` | Sends an in-character action. Floats above your character and appears in the ME Box. |
| `/do` | `/do The wanted poster is torn` | Sends an out-of-character description. Floats above your character. |
| `/status` | `/status Injured, left arm` | Sets a persistent status label above your character. Using it alone (`/status`) opens the full Status Editor with colour options and saved presets. |
| `/cstatus` | `/cstatus` | Clears your active status label. |
| `/id` | `/id` | Displays your server ID as a floating label above your character (visible to nearby players). |
| `/cash` | `/cash` | Displays your current cash as a floating label above your character. |
| `/mebox` | `/mebox <option>` | Controls the ME Display Box. See [ME Display Box](#me-display-box) for all options. |

> All command names are configurable by a server admin in `config.lua`.

### Database

The scene and status tables are created automatically when the script starts (`sql/install.sql`). To import the file yourself instead, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

---

## Scene Editor — Step by Step

Scenes are floating lines of text placed at a fixed point in the world. Everyone within range can read them without interaction.

### Placing a Scene

1. Type `/scene` in chat.
2. The **Scene Editor** panel opens in the bottom-right corner.

**Step 1 — Write your text**

- Click inside the text box and type your scene description.
- All text is automatically converted to **uppercase** when placed in the world (this is an RDR2 engine limitation).

**Formatting your text:**

- **Font size** — Use the dropdown on the left of the toolbar to change how large the text is before you type or after selecting existing text.
- **Colour** — Click any of the coloured squares in the toolbar to set the colour. The active colour has a gold border. Click a swatch first, then type, and that section will be that colour. You can mix colours on the same line by changing the swatch mid-sentence.

> **Tip:** If you colour text and then change the font size, the colour is preserved. You can also select existing typed text and then click a colour or size to reformat it.

3. Click **Confirm →** when you are happy with the text.

**Step 2 — Position the text**

A position editor replaces the text panel. Your text is now floating in the world so you can see it live.

- Use the **◄ ►** arrow buttons on each axis (X, Y, Z) to nudge the text in that direction.
- Scroll the **mouse wheel** over an axis value for fine adjustments.
- Type a number directly into any axis field and press **Enter**.
- **X** = left/right, **Y** = forward/back, **Z** = up/down.

4. Click **✓ Place** to save the scene. It is now visible to all nearby players and persists on the server.
5. Click **✕ Cancel** at any point to go back to the text editor without losing your work.

### Visibility

Scenes are visible to any player within the distance set by `Config.scenevisabilitydistance` (default: **5 units**).

---

## Status Editor — Step by Step

A status is a short piece of text that floats above your character at all times, visible to other players nearby. It is intended for things like injury states, moods, or RP tags.

### Setting a Status

**Quick method:** `/status Your text here` — applies instantly with no colour formatting.

**Full editor method:** `/status` with no arguments opens the Status Editor panel.

1. Click inside the **Status Text** box and type your status.
2. Use the **colour swatches** in the toolbar to choose a colour before or after typing.
3. *(Optional)* Enter a **Preset name** in the name field and click 💾 to save it for later.
4. Click **▶ Apply** to set the status.

### Presets

Saved presets appear as a list at the bottom of the Status Editor. Clicking a preset name loads it into the text box for editing. Click **▶** to apply it instantly, or **✕** to delete it.

### Clearing a Status

- Click **■ Clear** in the Status Editor, or
- Type `/cstatus` in chat.

---

## ME Display Box

The ME Box is a semi-transparent overlay near the top-centre of your screen that collects `/me` messages from players close to you. This lets you read emote messages even when the overhead text above a player's head has already faded.

### Controlling the Box

Use `/mebox <option>`:

| Option | Effect |
|---|---|
| `auto` | The box appears when a `/me` is received and automatically fades after the configured timeout (default 30 seconds). **This is the default mode.** |
| `persist` | The box stays visible at all times as long as there are messages in it. It never fades automatically. |
| `off` | Hides the box entirely. You will not see any incoming `/me` messages in it. |
| `small` | Makes the box narrower. |
| `normal` | Resets the box to its default width. |
| `large` | Makes the box wider. |
| `move` | Enters **drag mode** — you can click and drag the box anywhere on screen. Press **ESC** to save the new position and exit drag mode. The position is remembered permanently. |

### Message Behaviour

- Messages **older than 5 minutes** are automatically and permanently removed from the box. This is separate from the box fading — faded messages still exist and come back if the box is re-shown (e.g. when you press **T** to open chat).
- When the box re-appears after being hidden, it scrolls to the **most recent** messages automatically. Older messages are still accessible by **scrolling up** inside the box.
- Your own `/me` messages always appear in your ME Box regardless of distance.

### Scrolling

If the box has more messages than fit on screen, you can scroll up/down with the **mouse wheel** while hovering over it. Scrolling back down to the bottom shows the latest messages.

---

## Removing a Scene

Walk up close to a scene you placed (or any scene if your job has admin permission). When you are within range, a hint appears on screen:

> **Press 4 To Remove**

Press **4** to delete the scene. It is removed for all players immediately.

- You can always remove **your own** scenes.
- Players with certain staff jobs (configured in `Config.removejobs`) can remove **any** scene.

---

## Configuration Reference

All settings live in `config.lua`. A server admin edits this file — players do not need to touch it. The table below explains every option in plain language.

### General

| Option | Default | What it does |
|---|---|---|
| `Config.scenevisabilitydistance` | `5` | How close (in world units) a player must be to see floating scene text. |
| `Config.denysceneinhideout` | `false` | If `true`, players cannot place scenes while inside the gang hideout zone. |
| `Config.joblock` | `false` | If `true`, only jobs listed in `Config.allowjobs` can use `/scene`. |
| `Config.allowjobs` | `{}` | List of job names allowed to use `/scene` when `joblock` is enabled. |
| `Config.removejobs` | `{...}` | Job names that can remove **any** scene, not just their own. |
| `Config.webhook` | `""` | Discord webhook URL — fill this in to log scene placements to a Discord channel. |

### Commands

| Option | Default | What it does |
|---|---|---|
| `Config.scenecommand` | `"scene"` | The chat command to open the Scene Editor (`/scene`). |
| `Config.mecommand` | `"me"` | The command for in-character actions (`/me`). |
| `Config.ooccommand` | `"do"` | The command for OOC descriptions (`/do`). |
| `Config.statuscommand` | `"status"` | The command for setting a status label. |
| `Config.stopdisplay` | `"cstatus"` | The command to clear your status label. |
| `Config.id` | `"id"` | The command to show your server ID above your character. |
| `Config.cash` | `"cash"` | The command to show your cash above your character. |

### Overhead /me Text (floating above peds)

| Option | Default | What it does |
|---|---|---|
| `Config.meoverheadduration` | `10` | How many **seconds** the floating `/me` or `/do` text stays visible above a player's head before disappearing. |
| `Config.meoverheadradius` | `20` | How far away (world units) you can be and still see floating `/me` text above other players. |

### ME Display Box

| Option | Default | What it does |
|---|---|---|
| `Config.meboxcommand` | `"mebox"` | The chat command to control the ME Box. |
| `Config.meboxdistance` | `6` | How close (world units) another player must be for their `/me` to appear in **your** ME Box. |
| `Config.meboxtimeout` | `30` | In **auto** mode, how many seconds after the last message before the box fades. |
| `Config.meboxmaxlines` | `7` | How many lines are visible at once before the box starts scrolling. |
| `Config.meboxhistory` | `50` | Maximum number of messages kept in the scroll-back buffer. Oldest are trimmed once this limit is reached. |
| `Config.meboxdissipateafter` | `300` | How many **seconds** before an individual message is permanently deleted from the box (default: 5 minutes). |
| `Config.meboxDefaultNameColor` | `"#eeebe4"` | The default colour of the sender's name in the ME Box when their job has no specific colour. |
| `Config.meboxJobColors` | `{...}` | A table mapping job names to hex colour codes. The sender's name appears in that colour in the ME Box. Add or remove jobs as needed. |

### UI Theme

| Option | Default | What it does |
|---|---|---|
| `Config.uitheme` | `"amber-frontier"` | The colour scheme used for the Scene and Status editor panels. See [UI Themes](#ui-themes) below. |

---

## UI Themes

Change `Config.uitheme` in `config.lua` to switch the look of the editor panels. The change takes effect the next time a player loads their character.

| Theme name | Colours |
|---|---|
| `"amber-frontier"` | **Old West gold & amber** — the default dark sepia look. |
| `"steel-blue"` | **Modern dark navy** with cool steel-blue accents. |
| `"ivory-noir"` | **Black & white** — no colour tint at all, pure monochrome. |
| `"lavender-dusk"` | **Soft pastel lavender** and purple tones. |
| `"crimson-dusk"` | **Deep dramatic red** — dark background with blood-red accents. |
| `"emerald-ridge"` | **Earthy forest green** — dark background with green accents. |

> The ME Display Box is intentionally not affected by the UI theme — it always matches the poodlechat overlay style.
