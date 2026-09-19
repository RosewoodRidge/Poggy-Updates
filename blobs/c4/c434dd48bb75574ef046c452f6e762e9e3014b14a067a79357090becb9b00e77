# Playing an exported scene

AnimTool gives you a Lua **scene** table. Any client script can play it with the bundled runtime, `scene_player.lua`.

## 1. Export

1. Open the editor with `/animtool` and build your scene.
2. Click **Export**.
3. On the **Scene (Lua)** tab, click **Copy to clipboard**.
4. On the **Runtime · scene_player.lua** tab, copy `scene_player.lua` too (you only need it once per resource).

## 2. Add both to your resource

Save the runtime as `scene_player.lua` in your resource. Load it **before** your own script:

```lua
-- fxmanifest.lua
client_scripts {
    'scene_player.lua',
    'my_script.lua',
}
```

## 3. Play it

```lua
-- my_script.lua
local scene = { ... }   -- pasted from AnimTool

-- Play once. Props are cleaned up when it ends.
ScenePlayer.playOnce(PlayerPedId(), scene)
```

For more control:

```lua
local player = ScenePlayer.new(PlayerPedId())
player:load(scene)
player:play()
player.onFinish = function() player:destroy() end
```

| Call | What it does |
|---|---|
| `:play()` / `:pause()` | Start or pause. |
| `:seek(t)` | Jump to `t` seconds. |
| `:setSpeed(s)` | Playback speed, 1 = normal. |
| `:setLoop(b)` | Loop on or off. |
| `:stop()` | Stop. |
| `:destroy()` | Stop the animation and delete the props. |

## Tips

- Clips shown as *skipped* in the export failed to load or were disabled. Fix them in the editor and export again.
- `scene_player.lua` is readable source. You may ship it inside your own resources.
- Press `?` in the editor for every keyboard shortcut.
