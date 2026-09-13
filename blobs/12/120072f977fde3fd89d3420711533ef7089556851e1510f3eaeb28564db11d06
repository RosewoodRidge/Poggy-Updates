# Poggy AnimTool (v2)

Timeline editor for RedM animation scenes. Chain animations back-to-back on an
animation track, attach props to the player with keyframed position/rotation and
in/out visibility, preview it live on your ped, then export a Lua scene table
that any resource can play back with the bundled runtime.

Open in game with `/animtool` (requires the `command.animtool` ace).

## How it fits together

| Piece | Role |
| --- | --- |
| `ui/` | The editor. It owns the whole project (clips, props, keyframes, undo history). Every edit is synced to the game as one scene. |
| `client/scene_player.lua` | `ScenePlayer` — the playback runtime. Master clock, clip switching, prop reconciliation and keyframe interpolation. The tool uses it for preview **and exports the same file**. |
| `client/client.lua` | Editor glue: NUI callbacks, browser preview, clip duration probing, project save/load (client KVP). |
| `ui/data/` | Animation dictionary / object model lists, split into `anim_chunk_0..13.json` and `obj_chunk_0..4.json` (the counts are `ANIM_CHUNKS` / `OBJ_CHUNKS` in `ui/js/app.js`). |

## Editor basics

* **Library** (left): search or browse dictionaries. Click a clip to preview it on your ped,
  double-click / `+` / drag it onto the timeline to add it. `★` saves favourites.
* **Timeline** (bottom): clips play back-to-back. Drag a clip to reorder, drag its edges to
  trim, drag the right edge past the native length to *hold* the last frame.
  Each prop gets a lane: drag the bar ends for in/out points, diamonds are keyframes.
* **Inspector** (right): add props, choose bone / parent prop / scale, pose with sliders.
  With keyframes present the sliders edit the keyframe under the playhead, and with
  **Auto-key** on a new keyframe is created wherever you pose. `K` keys the current pose.
* **Projects**: Save / Open (per player, stored client-side), Import/Export JSON to share.
  Unsaved work is kept in the browser and offered for restore next time.
* Press `?` in the tool for the full shortcut list.

## Using an exported scene

Export → **Scene (Lua)** gives you a `scene` table. Ship `client/scene_player.lua`
(Export → **Runtime** tab shows it) with your resource, loaded before your script:

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

## Developing the UI outside the game

`ui/js/nui.js` contains a `MockGame` that stands in for `client.lua` when the page
is opened in a normal browser, so the full editor can be exercised without RedM.
Serve the resource folder (for example `python -m http.server 8765`) and open
`http://localhost:8765/ui/index.html`.
