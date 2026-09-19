# Poggy AnimTool

A timeline editor for RedM animation scenes, inside the game.

- Chain animations back to back on an animation track.
- Attach props to your character, with keyframed position and rotation, and in / out times.
- Preview it all live on your own ped.
- Export a Lua scene table that any resource can play with the bundled runtime.

---

## Requirements

| Resource | Why |
|---|---|
| `poggy_core` | Required by every Poggy script. Must start first. |

No database tables. No config file.

---

## Installation

1. Put the `poggy_animtool` folder in your server's `resources` folder.
2. Add this line to `server.cfg`, **after** `ensure poggy_core`:
   ```
   ensure poggy_animtool
   ```
3. Give your staff the ACE permission (see **Permissions**).
4. Restart the server.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/animtool` | ACE `command.animtool` | Opens the editor, or closes it when it is open. |

Players without the permission get no reply.

---

## Permissions

`/animtool` needs the ACE `command.animtool`. Add it in `server.cfg`:

```
add_ace group.admin command.animtool allow
```

If your admins already have the ACE `command` (every command), they already have access.

---

## Configuration

AnimTool has no settings. Its card in **`/poggy`** (the Poggy settings hub) shows this README, the command and the help pages.

---

## Editor basics

- **Library** (left): search or browse animation dictionaries. Click a clip to preview it on your ped. Double-click, press `+`, or drag it onto the timeline to add it. `★` saves favourites.
- **Timeline** (bottom): clips play back to back. Drag a clip to reorder it. Drag its edges to trim. Drag the right edge past the clip's own length to *hold* the last frame. Each prop gets a lane: drag the bar ends for in / out points; diamonds are keyframes.
- **Inspector** (right): add props, choose the bone, parent prop and scale, and pose with sliders. With keyframes present, the sliders edit the keyframe under the playhead. With **Auto-key** on, a new keyframe is made wherever you pose. `K` keys the current pose.
- **Projects**: Save and Open (per player, stored on the player's own PC). **Import** and **Export → Project JSON** to share. Unsaved work is kept and offered for restore next time.
- Press `?` in the tool for every shortcut.

### Shortcuts

| Key | Action |
|---|---|
| Space | Play / pause |
| Home / End | Go to start / end |
| ← → | Step one frame (Shift: one second) |
| ↑ ↓ | Previous / next clip edge or keyframe |
| K | Add or update a keyframe at the playhead |
| Delete | Delete the selected keyframe, clip or prop |
| Ctrl+Z / Ctrl+Y | Undo / redo |
| Ctrl+C / Ctrl+V | Copy pose / paste as keyframe |
| Ctrl+D | Duplicate the selected clip or prop |
| Ctrl+S | Save project |
| Ctrl+F | Search the library |
| L · A · S | Loop · auto-key · snap on or off |
| F · + · − | Fit timeline · zoom in · zoom out |
| Esc | Close a dialog, stop a preview, or close the tool |

While the editor is open, your game time is held at 10:00 so the lighting stays the same. Only you see this.

---

## Using an exported scene

**Export → Scene (Lua)** gives you a `scene` table. Ship `client/scene_player.lua` (the **Runtime** tab shows it) with your resource, loaded before your script:

```lua
-- fxmanifest.lua
client_scripts { "scene_player.lua", "my_script.lua" }
```

```lua
-- my_script.lua
local scene = { ... }                     -- pasted from AnimTool

-- one-shot
ScenePlayer.playOnce(PlayerPedId(), scene)

-- or with control
local player = ScenePlayer.new(PlayerPedId())
player:load(scene)
player:play()                 -- :pause()  :seek(t)  :stop()  :setSpeed(s)  :setLoop(b)
player.onFinish = function() player:destroy() end
```

`destroy()` stops the animation and deletes the props.

`client/scene_player.lua` stays readable so you can ship it inside your own resources.

---

## How it fits together

| Piece | Role |
|---|---|
| `ui/` | The editor. It owns the whole project (clips, props, keyframes, undo history). Every edit is synced to the game as one scene. |
| `client/scene_player.lua` | `ScenePlayer`, the playback runtime: master clock, clip switching, props and keyframe interpolation. The tool uses it for preview **and exports the same file**. |
| `client/client.lua` | Editor glue: NUI callbacks, library preview, clip length probing, project save / load (client storage). |
| `server/server.lua` | The permission check for `/animtool`. |
| `ui/data/` | Animation dictionary and object model lists, split into `anim_chunk_0..13.json` and `obj_chunk_0..4.json` (the counts are `ANIM_CHUNKS` / `OBJ_CHUNKS` in `ui/js/app.js`). |

---

## Troubleshooting

**`/animtool` does nothing.**
You do not have the ACE `command.animtool`. Add it (see **Permissions**) and restart, or run the `add_ace` line in the server console.

**A clip shows as failed or skipped.**
Its animation dictionary did not load. Pick another clip or check the name. Skipped clips are left out of the export.

**My projects are gone on another PC.**
Projects are stored on the PC you saved them on. Use **Export → Project JSON** and **Import** to move them.

**A prop does not appear.**
The model name may be wrong. The editor shows an error for props that fail to spawn.

---

## Developing the UI outside the game

`ui/js/nui.js` contains a `MockGame` that stands in for `client.lua` when the page is opened in a normal browser, so the full editor can be tried without RedM. Serve the resource folder (for example `python -m http.server 8765`) and open `http://localhost:8765/ui/index.html`.
