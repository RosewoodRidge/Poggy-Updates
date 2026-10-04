local time = (Config.meoverheadduration or 10) * 1000
local scene = false 
local findspot = false 
local textcoords = {}
local displaytext
local x,y,z
local scenes = {}
local players = {}
local isplay = false 
local allowscene = true 
local allowscene2 = true 

-- NUI / placement state
local myCharId   = -1       -- this player's character: from poggy_core:charLoadedLocal, and from the server (below)
local editAny    = false    -- the admin group may change anyone's scene

-- Framework calls go through Poggy(verb, payload) from poggy_core.

--- Bottom-of-screen objective banner, unchanged in appearance.
local function notify(text, duration)
    Poggy('notify.styled', { style = 'objective', text = text, duration = duration })
end
local nuiPhase   = nil      -- nil | 'text' | 'position'
-- A sign being placed (client/signs.lua): what it says, and how it is turned. nil for floating text.
local pendingSign = nil
local signRot     = { x = 0.0, y = 0.0, z = 0.0 }
local signBase    = { x = 0.0, y = 0.0, z = 0.0, h = 0.0 }   -- where it first appeared, and which way the player faced
local editingId   = nil      -- the scene being changed (4 beside a scene you may change), or nil when making a new one
local editBase    = nil      -- a sign being changed stays where it is: its own place and the way it faces
local signLive    = false    -- the sign being written is standing in the world
local signLiveKey = nil      -- what it says, to tell a change of words from a move
local nuiHovered = false    -- true when mouse is over any NUI panel
-- Per-player overhead display stack: [localPlayerIndex] = {{text, expiresAt, showBg}, ...}
local meStack    = {}

function GetPlayers()
	local players = {}
    for i = 0, 256 do
        if NetworkIsPlayerActive(i) then
            table.insert(players, GetPlayerServerId(i))
        end
    end
    return players
end

-- The server says who this player is. The load event alone is not enough: it does not come again
-- when the script is restarted with the player already in, and then no scene looked like their own.
RegisterNetEvent('poggy_scene:whoami')
AddEventHandler('poggy_scene:whoami', function(charid, admin)
    if tonumber(charid) and tonumber(charid) > 0 then myCharId = tonumber(charid) end
    editAny = admin == true
end)

RegisterNetEvent('poggy_scene:stopscene')
AddEventHandler('poggy_scene:stopscene', function(x)
    allowscene2 = x
end)

-- ── NUI HELPERS ─────────────────────────────────────────────────────────
--- Open the Scene Editor: empty to make a new scene, or on a scene to change it (press 5 beside your own).
local function openSceneNUI(scene)
    local coords = GetEntityCoords(PlayerPedId())
    x = coords.x
    y = coords.y
    z = coords.z
    nuiPhase = 'text'
    editingId = nil
    local edit = nil
    if scene then
        editingId = scene.id
        x, y, z = scene.coords.x + 0.0, scene.coords.y + 0.0, scene.coords.z + 0.0
        if scene.sign and PoggySigns.Available() then
            -- the sign stays where it is: the sliders start from its own place and the way it faces
            local rot = scene.sign.rot or {}
            editBase = { x = x, y = y, z = z, h = (tonumber(rot.z) or 0.0) + 0.0 }
            signBase = { x = editBase.x, y = editBase.y, z = editBase.z, h = editBase.h }
            signRot = { x = (tonumber(rot.x) or 0.0) + 0.0, y = (tonumber(rot.y) or 0.0) + 0.0, z = signBase.h }
            signLive, signLiveKey = true, nil
            PoggySigns.Hide(scene.id)               -- its old self comes down; the editor shows the new one
            edit = { sign = SignMeasure(scene.sign), rx = signRot.x, ry = signRot.y }
        else
            edit = { text = scene.text }
        end
    end
    -- Full keyboard capture for text entry (no game input pass-through)
    SetNuiFocus(true, true)
    SendNUIMessage({ type = 'open_scene_text', signs = PoggySigns.NuiData(), edit = edit })
end

local function closeSceneNUI()
    PoggySigns.ClearPreview()
    PoggySigns.Hide(nil)                -- a sign that was being edited goes back up (as it was, or as changed)
    editingId, editBase = nil, nil
    pendingSign, signLive, signLiveKey = nil, false, nil
    findspot    = false
    nuiPhase    = nil
    nuiHovered  = false
    SetNuiFocus(false, false)
    SetNuiFocusKeepInput(false)
    SendNUIMessage({ type = 'close_scene_ui' })
end

-- Status NUI helpers
local function openStatusNUI()
    SetNuiFocus(true, true)
    TriggerServerEvent('poggy_scene:loadStatusPresets')
    SendNUIMessage({ type = 'open_status_ui' })
end

local function closeStatusNUI()
    SetNuiFocus(false, false)
    SendNUIMessage({ type = 'close_status_ui' })
end

--- A sign's words as one plain line, for lists and logs.
local function signPlainText(sign)
    local lines = {}
    for i, runs in ipairs(sign.lines) do
        local parts = {}
        for j, run in ipairs(runs) do parts[j] = run.t end
        lines[i] = table.concat(parts)
    end
    return table.concat(lines, ' / ')
end

-- ── SIGN BOARDS: written and placed in one step ────────────────────────────
-- While the editor is on Sign board, the sign stands in the world as it is being written: every
-- change to the words, the board or a slider is shown at once (client/signs.lua builds it).
-- The sliders work from where the player stood when the sign first appeared (right, away, up),
-- not along the map's own axes.

local function applySignSliders(data)
    if type(data) ~= 'table' then return end
    local dx, dy, dz = tonumber(data.dx) or 0.0, tonumber(data.dy) or 0.0, tonumber(data.dz) or 0.0
    local h = math.rad(signBase.h)
    x = signBase.x + math.cos(h) * dx - math.sin(h) * dy
    y = signBase.y + math.sin(h) * dx + math.cos(h) * dy
    z = signBase.z + dz
    signRot.x = tonumber(data.rx) or 0.0
    signRot.y = tonumber(data.ry) or 0.0
    signRot.z = signBase.h + (tonumber(data.rz) or 0.0)
end

local function stopSignLive()
    PoggySigns.ClearPreview()
    pendingSign, signLive, signLiveKey = nil, false, nil
end

-- NUI: the sign changed (words, board, lettering) and/or a slider moved.
--   data.sign   the sign as the editor has it, when it fits; absent when only a slider moved
--   data.slide  the six slider values
RegisterNUICallback('sign_live', function(data, cb)
    cb('ok')
    if nuiPhase ~= 'text' or not PoggySigns.Available() or type(data) ~= 'table' then return end
    if not signLive then
        if editBase then
            -- a sign being changed: where it already is. (The editor page tidies itself up as it opens
            -- and that used to forget this, so the sign jumped to stand in front of the player.)
            signBase = { x = editBase.x, y = editBase.y, z = editBase.z, h = editBase.h }
        else
            -- a new sign: in front of the player, facing them
            local ped = PlayerPedId()
            local front = GetOffsetFromEntityInWorldCoords(ped, 0.0, 1.8, 0.45)
            signBase = { x = front.x, y = front.y, z = front.z, h = GetEntityHeading(ped) }
        end
        signLive = true
    end
    applySignSliders(data.slide)
    if type(data.sign) == 'table' then
        local measured = SignMeasure(data.sign, tonumber((Config.signs or {}).maxLetters) or 60)
        if measured then
            local key = json.encode(measured)
            if key ~= signLiveKey then
                pendingSign = measured
                -- remembered only once it is standing: a build that was overtaken is asked for again
                if PoggySigns.Preview(pendingSign, { x = x, y = y, z = z }, signRot) then
                    signLiveKey = key
                    PoggySigns.MovePreview({ x = x, y = y, z = z }, signRot)      -- the sliders may have moved meanwhile
                end
                return
            end
        end
    end
    PoggySigns.MovePreview({ x = x, y = y, z = z }, signRot)
end)

-- NUI: back to floating text: take the sign down.
RegisterNUICallback('sign_live_stop', function(data, cb)
    stopSignLive()
    cb('ok')
end)

-- NUI: place the sign as it stands.
RegisterNUICallback('sign_place', function(data, cb)
    cb('ok')
    local measured = type(data) == 'table' and PoggySigns.Available()
        and SignMeasure(data.sign, tonumber((Config.signs or {}).maxLetters) or 60)
    if not measured then
        notify(Config.Language.signnofit, 4000)
        return
    end
    if not signLive then
        if editBase then
            signBase = { x = editBase.x, y = editBase.y, z = editBase.z, h = editBase.h }
        else
            local ped = PlayerPedId()
            local front = GetOffsetFromEntityInWorldCoords(ped, 0.0, 1.8, 0.45)
            signBase = { x = front.x, y = front.y, z = front.z, h = GetEntityHeading(ped) }
        end
    end
    applySignSliders(data.slide)
    measured.rot = { x = signRot.x, y = signRot.y, z = signRot.z }
    if editingId then
        TriggerServerEvent("poggy_scene:editscene", editingId, { x = x, y = y, z = z }, signPlainText(measured), measured)
    else
        TriggerServerEvent("poggy_scene:savescene", { x = x, y = y, z = z }, signPlainText(measured), GetPlayers(), measured)
    end
    closeSceneNUI()
end)

-- NUI: player confirmed scene text — switch to position-editor phase
RegisterNUICallback('scene_text_confirm', function(data, cb)
    local text = (data and data.text) or ''
    if text ~= '' then
        displaytext = text
        nuiPhase = 'position'
        -- Camera locked by default; click-hold outside panel to orbit
        SetNuiFocus(true, true)
        SetNuiFocusKeepInput(false)
        SendNUIMessage({ type = 'open_scene_position', x = x, y = y, z = z, text = displaytext })
        findspot = true
    end
    cb('ok')
end)

-- NUI: Remove, in the editor opened on a scene that is already placed.
RegisterNUICallback('scene_remove', function(data, cb)
    if editingId then
        local scene = scenes[editingId]
        TriggerServerEvent("poggy_scene:removescene", editingId, scene and scene.charid, GetPlayers())
    end
    closeSceneNUI()
    cb('ok')
end)

-- NUI: cancelled at text-entry phase
RegisterNUICallback('scene_text_cancel', function(data, cb)
    closeSceneNUI()
    cb('ok')
end)

-- NUI: player adjusted position with scroll/arrows — update local preview coords
RegisterNUICallback('scene_position_update', function(data, cb)
    if data then
        x = tonumber(data.x) or x
        y = tonumber(data.y) or y
        z = tonumber(data.z) or z
    end
    cb('ok')
end)

-- NUI: player confirmed placement — save scene server-side
RegisterNUICallback('scene_place_confirm', function(data, cb)
    if data then
        x = tonumber(data.x) or x
        y = tonumber(data.y) or y
        z = tonumber(data.z) or z
    end
    textcoords = { x = x, y = y, z = z }
    if editingId then
        TriggerServerEvent("poggy_scene:editscene", editingId, textcoords, displaytext, nil)
    else
        TriggerServerEvent("poggy_scene:savescene", textcoords, displaytext, GetPlayers())
    end
    closeSceneNUI()
    cb('ok')
end)

-- NUI: cancelled at position phase
RegisterNUICallback('scene_place_cancel', function(data, cb)
    closeSceneNUI()
    cb('ok')
end)

-- NUI: cancelled at position phase — revert to text editor (keep NUI open)
RegisterNUICallback('scene_place_revert', function(data, cb)
    nuiPhase = 'text'
    findspot = false
    -- NUI focus stays on — the JS side switches back to the text panel
    cb('ok')
end)

-- NUI: mouse entered / left panel — track hover state (used in control loop).
RegisterNUICallback('scene_hover', function(data, cb)
    nuiHovered = (data ~= nil and data.hovered == true)
    cb('ok')
end)

-- Camera orbit: click-hold outside panel to orbit camera (position phase, or while a sign is being written)
RegisterNUICallback('scene_camera_hold', function(data, cb)
    if nuiPhase == 'position' or (nuiPhase == 'text' and signLive) then
        if data and data.holding then
            SetNuiFocus(true, false)        -- keep NUI active but hide cursor
            SetNuiFocusKeepInput(true)      -- game gets input → camera orbits
        else
            SetNuiFocus(true, true)         -- restore cursor
            SetNuiFocusKeepInput(false)     -- NUI captures input → camera locked
        end
    end
    cb('ok')
end)

-- ── STATUS PRESET NUI CALLBACKS ────────────────────────────────────────────
RegisterNUICallback('status_cancel', function(data, cb)
    closeStatusNUI()
    cb('ok')
end)

RegisterNUICallback('status_apply', function(data, cb)
    local text = (data and data.text) or ''
    if text ~= '' then
        if isplay then TriggerServerEvent('poggy_scene:stopdisplay', GetPlayers()) end
        isplay = true
        TriggerServerEvent('poggy_scene:shareDisplay2', Config.Language.stat .. text, GetPlayers())
    end
    closeStatusNUI()
    cb('ok')
end)

RegisterNUICallback('status_clear', function(data, cb)
    if isplay then
        TriggerServerEvent('poggy_scene:stopdisplay', GetPlayers())
        isplay = false
    end
    closeStatusNUI()
    cb('ok')
end)

RegisterNUICallback('status_save', function(data, cb)
    local name = (data and data.name) or ''
    local text = (data and data.text) or ''
    if text ~= '' then TriggerServerEvent('poggy_scene:saveStatusPreset', name, text) end
    cb('ok')
end)

RegisterNUICallback('status_delete', function(data, cb)
    local id = data and data.id
    if id then TriggerServerEvent('poggy_scene:deleteStatusPreset', id) end
    cb('ok')
end)

RegisterNetEvent('poggy_scene:receiveStatusPresets')
AddEventHandler('poggy_scene:receiveStatusPresets', function(presets)
    SendNUIMessage({ type = 'status_presets', presets = presets })
end)

RegisterCommand(Config.scenecommand, function(source, args)
    if allowscene and allowscene2 then
        openSceneNUI()
    else
        notify(Config.Language.notinhideout, 4000)
    end
end)

RegisterNetEvent('poggy_scene:getscenes')
AddEventHandler('poggy_scene:getscenes', function(scene)
    scenes = scene
    PoggySigns.SetScenes(scenes)
end)

RegisterNetEvent('poggy_scene:getscenes2')
AddEventHandler('poggy_scene:getscenes2', function()
    local state = 1
    TriggerServerEvent("poggy_scene:pullscenes",state,0)
end)

-- Character selected: poggy_core carries the character id on its own load event.
AddEventHandler("poggy_core:charLoadedLocal", function(charid)
    if charid then myCharId = tonumber(charid) or myCharId end
    -- Own thread: this handler runs inside poggy_core's net handler, so no Wait here.
    CreateThread(function()
        Wait(1000)
        local state = 1
        TriggerServerEvent("poggy_scene:pullscenes",state,0)
        TriggerServerEvent("poggy_scene:getstatus")
    end)
end)
AddEventHandler(
    "onResourceStart",
    function(resourceName)
        if resourceName == GetCurrentResourceName() then
            Wait(1000)
            local state = 1
            TriggerServerEvent("poggy_scene:pullscenes",state,0)
            TriggerServerEvent("poggy_scene:getstatus")
        end
    end
)

Citizen.CreateThread(function()
    while true do
        Wait(1)
        sleep = true 
        if Config.denysceneinhideout then  
            local player = PlayerPedId()
            local coords = GetEntityCoords(player)    
            if GetDistanceBetweenCoords(coords,1785.01,-821.53,191.01 , true) < 100 then
                sleep = false 
                allowscene = false 
            else 
                allowscene = true 
            end
        end
        if sleep then 
            Wait(500)
        end
    end
end)

Citizen.CreateThread(function()
    while true do
        Wait(1)
        local playerCoords = GetEntityCoords(PlayerPedId())

        -- Draw all nearby scene texts and track the single nearest removable candidate.
        -- Removal hint / key check are skipped while the NUI is open (nuiPhase ~= nil).
        local nearestScene = nil
        local nearestDist  = math.huge

        for k,v in pairs(scenes) do
            local d = GetDistanceBetweenCoords(playerCoords, v.coords.x, v.coords.y, v.coords.z, true)
            if d < Config.scenevisabilitydistance then
                sleep = false
                -- A sign is props (client/signs.lua), not floating text; it is wider, so its reach is too.
                if not v.sign and v.id ~= editingId then DrawText3D(v.coords.x, v.coords.y, v.coords.z, v.text) end
                -- Only consider scenes within arm's reach and when NUI is closed
                if d < (v.sign and 2.0 or 1.5) and nuiPhase == nil and d < nearestDist then
                    nearestDist  = d
                    nearestScene = v
                end
            end
        end

        -- Permission check FIRST, then only fire the remove event for the nearest scene.
        if nearestScene then
            -- Own scenes are always removable.
            -- Non-owned scenes require job permission (allowscene2 set by getstatus).
            local isOwn      = tonumber(nearestScene.charid) == tonumber(myCharId)
            local hasJobPerm = allowscene2
            local canRemove  = isOwn or hasJobPerm

            -- One key, 4. On a scene you may change (your own; any, for the admin group) it opens the
            -- Scene Editor on it, where there is also a Remove button. On someone else's it removes,
            -- for those allowed to.
            local canEdit = isOwn or editAny
            drawtext(canEdit and (Config.Language.pressedit or "Press 4 To Edit") or Config.Language.pressg2,
                0.15, 0.30, 0.1, 0.3, true, 255, 255, 255, 255, true, 10000)
            -- Suppress the quickselect so the game doesn't consume 4 as weapon-switch,
            -- then detect it via IS_DISABLED_CONTROL_JUST_PRESSED.
            Citizen.InvokeNative(0xFE99B66D079CF6BC, 0, Config.keys["4"], true) -- DISABLE_CONTROL_ACTION
            if Citizen.InvokeNative(0x91AEF906BCA88877, 0, Config.keys["4"]) then -- IS_DISABLED_CONTROL_JUST_PRESSED
                if canEdit then
                    openSceneNUI(nearestScene)
                    Wait(500)
                elseif canRemove then
                    TriggerServerEvent("poggy_scene:removescene", nearestScene.id, nearestScene.charid, GetPlayers())
                    Wait(2500) -- cooldown to prevent accidental double-activation
                else
                    -- Job-locked: not the owner and no removal permission
                    notify(Config.Language.notjob, 3000)
                    Wait(2500)
                end
            end
        end

        if sleep then
            Wait(500)
        end
    end
end)


-- Placement preview thread — draws the 3D text at the current position
-- while the NUI position editor is open.  All movement is now driven by
-- the NUI (scroll wheel / arrow buttons); no key handling here.
Citizen.CreateThread(function()
    while true do
        Wait(10)
        if findspot and nuiPhase == 'position' and not pendingSign then
            DrawText3D(x, y, z, displaytext)
        end
    end
end)

--------------------------------

RegisterCommand(Config.mecommand, function(source, args)
    local text = ''
    for i = 1,#args do
        text = text .. ' ' .. args[i]
    end
    text = text .. ' '
    TriggerServerEvent('poggy_scene:shareDisplay', text, GetPlayers(), true)
    TriggerServerEvent('poggy_scene:meMessage', text)
end)

RegisterCommand(Config.ooccommand, function(source, args)
    local text = Config.Language.ooc
    for i = 1,#args do
        text = text .. ' ' .. args[i]
    end
    text = text .. ' '
    TriggerServerEvent('poggy_scene:shareDisplay', text, GetPlayers(), true)
end)

RegisterCommand(Config.id, function(source, args)
    local text =Config.Language.id.. GetPlayerServerId(GetPlayerIndex())
    TriggerServerEvent('poggy_scene:shareDisplay', text,GetPlayers())
end)

RegisterCommand(Config.cash, function(source, args)
    local text =Config.Language.cash
    TriggerServerEvent('poggy_scene:shareDisplay3', text,GetPlayers())
end)

RegisterCommand(Config.statuscommand, function(source, args)
    -- /status alone → open preset manager NUI.
    -- /status some text → apply instantly, no UI (quick-fire still works).
    if #args > 0 then
        if isplay then TriggerServerEvent('poggy_scene:stopdisplay', GetPlayers()) end
        isplay = true
        local text = Config.Language.stat
        for i = 1, #args do text = text .. ' ' .. args[i] end
        text = text .. ' '
        TriggerServerEvent('poggy_scene:shareDisplay2', text, GetPlayers())
    else
        openStatusNUI()
    end
end)

RegisterNetEvent('poggy_scene:triggerDisplay')
AddEventHandler('poggy_scene:triggerDisplay', function(text, source, showBg)
    AddDisplay(GetPlayerFromServerId(source), text, showBg == true)
end)

RegisterNetEvent('poggy_scene:triggerDisplay2')
AddEventHandler('poggy_scene:triggerDisplay2', function(playersstatus)
    players = playersstatus
end)

Citizen.CreateThread(function()
    while true do
        Wait(10)
        local coords = GetEntityCoords(PlayerPedId(), false)
        for k,v in pairs(players) do 
            local coordsMe = GetEntityCoords(GetPlayerPed(GetPlayerFromServerId(v.id)), false)
            if GetDistanceBetweenCoords(coords, coordsMe, true) < Config.scenevisabilitydistance then
                DrawText3D(coordsMe['x'], coordsMe['y'], coordsMe['z'], v.texts)
            end
        end
    end
end)

RegisterCommand(Config.stopdisplay, function(source, args)
    if isplay then 
        TriggerServerEvent('poggy_scene:stopdisplay',GetPlayers())
        Wait(1000)
    end
end)


-- Adds a new entry to the per-player overhead display stack.
-- showBg = true draws a semitransparent rounded background (used for /me and /do only).
function AddDisplay(mePlayer, text, showBg)
    if not meStack[mePlayer] then meStack[mePlayer] = {} end
    table.insert(meStack[mePlayer], {
        text      = text,
        expiresAt = GetGameTimer() + time,
        showBg    = showBg == true,
    })
end

-- Compatibility shim kept in case anything external still calls Display().
function Display(mePlayer, text) AddDisplay(mePlayer, text, false) end

-- Preload the sprite dict for rounded backgrounds, then run the centralized render loop.
Citizen.CreateThread(function()
    RequestStreamedTextureDict('feeds', false)
    while not HasStreamedTextureDictLoaded('feeds') do Wait(200) end

    while true do
        Wait(0)
        local now      = GetGameTimer()
        local myCoords = GetEntityCoords(PlayerPedId(), false)

        for playerIdx, entries in pairs(meStack) do
            -- Prune expired entries (iterate backwards to preserve indices).
            for i = #entries, 1, -1 do
                if now >= entries[i].expiresAt then table.remove(entries, i) end
            end

            if #entries == 0 then
                meStack[playerIdx] = nil
            else
                local ped = GetPlayerPed(playerIdx)
                if DoesEntityExist(ped) then
                    local pc = GetEntityCoords(ped, false)
                    -- Vdist2 returns squared distance; use Config.meoverheadradius.
                    local meRad = Config.meoverheadradius or 20
                    if Vdist2(myCoords, pc) < (meRad * meRad) then
                        for i, entry in ipairs(entries) do
                            -- i=1 is oldest (lowest), newest entries stack upward.
                            DrawMeText3D(pc.x, pc.y, pc.z, entry.text, entry.showBg, (i - 1) * 0.038)
                        end
                    end
                end
            end
        end
    end
end)
function drawtext(str, x, y, w, h, enableShadow, col1, col2, col3, a, centre)
    local str = CreateVarString(10, "LITERAL_STRING", str, Citizen.ResultAsLong())
    SetTextScale(w, h)
    SetTextColor(math.floor(col1), math.floor(col2), math.floor(col3), math.floor(a))
    SetTextCentre(centre)
    if enableShadow then 
        SetTextDropshadow(1, 0, 0, 0, 255)
    end
    Citizen.InvokeNative(0xADA9255D, 10);
    DisplayText(str, x, y)
end
function DrawText3D(x, y, z, text)
    local onScreen,_x,_y=GetScreenCoordFromWorldCoord(x, y, z)
    local px,py,pz=table.unpack(GetGameplayCamCoord())  
    local dist = GetDistanceBetweenCoords(px,py,pz, x,y,z, 1)
    local str = CreateVarString(10, "LITERAL_STRING", text, Citizen.ResultAsLong())
    if onScreen then
    	SetTextScale(0.30, 0.30)
  		SetTextFontForCurrentCommand(1)
    	SetTextColor(255, 255, 255, 215)
    	SetTextCentre(1)
    	DisplayText(str,_x,_y)
    	local factor = (string.len(text)) / 225
    	--DrawSprite("feeds", "hud_menu_4a", _x, _y+0.0125,0.015+ factor, 0.03, 0.1, 35, 35, 35, 190, 0)
    end
end

-- Draws a single /me or /do line at the given world position.
-- screenOffsetY shifts the line upward in screen-space for stacking (0 = base line).
-- showBg draws a semitransparent black rounded rectangle behind the text.
function DrawMeText3D(x, y, z, text, showBg, screenOffsetY)
    local onScreen, _x, _y = GetScreenCoordFromWorldCoord(x, y, z)
    if not onScreen then return end

    local ry = _y - (screenOffsetY or 0)

    -- Count leading ~n~ tokens so the background follows the actual rendered line.
    -- Each ~n~ at scale 0.30 shifts rendered text ~0.022 screen units downward.
    local lineShift = 0
    local stripped  = text
    while stripped:sub(1, 3) == '~n~' do
        lineShift = lineShift + 0.022
        stripped  = stripped:sub(4)
    end

    -- Strip all remaining RDR markup tokens (~xx~, <html tags>) to get the
    -- printable character count only — so the box width truly fits the visible text.
    local clean = stripped
        :gsub('~[^~]+~', '')   -- removes ~n~, ~t6~, ~e~, etc.
        :gsub('<[^>]+>',  '')   -- removes <font size="22"> etc.

    -- Per-character width at SetTextScale(0.30) font 1, plus fixed left+right padding.
    local bgW = math.min(math.max(#clean * 0.0058 + 0.024, 0.048), 0.80)
    local bgH = 0.023
    local bgY = ry + 0.009 + lineShift

    if showBg then
        DrawSprite('feeds', 'hud_menu_4a', _x, bgY, bgW, bgH, 0.0, 0, 0, 0, 160, 0)
    end

    local str = CreateVarString(10, 'LITERAL_STRING', text, Citizen.ResultAsLong())
    SetTextScale(0.30, 0.30)
    SetTextFontForCurrentCommand(1)
    SetTextColor(255, 255, 255, 215)
    SetTextCentre(1)
    DisplayText(str, _x, ry)
end

function whenKeyJustPressed(key)
    if Citizen.InvokeNative(0x580417101DDB492F, 0, key) then
        return true
    else
        return false
    end
end

-------------------------------

-- ── ME DISPLAY BOX ──────────────────────────────────────────────────────────
-- Shows /me messages from nearby players in a textbox at top-center of screen.

local meBoxMode = 'auto'   -- 'auto' | 'persist' | 'off'
local meBoxLastMsg = 0      -- GetGameTimer() of last received message

-- Load saved mebox mode from KVP
local savedMeboxMode = GetResourceKvpString('mebox_mode')
if savedMeboxMode and (savedMeboxMode == 'auto' or savedMeboxMode == 'persist' or savedMeboxMode == 'off') then
    meBoxMode = savedMeboxMode
end

RegisterNetEvent('poggy_scene:meBoxReceive')
AddEventHandler('poggy_scene:meBoxReceive', function(data)
    if meBoxMode == 'off' then return end

    local sourceServerId = data.sourceId
    local myServerId = GetPlayerServerId(PlayerId())
    local isOwnMessage = (sourceServerId == myServerId)

    if not isOwnMessage then
        local sourcePlayer   = GetPlayerFromServerId(sourceServerId)
        local sourcePed      = GetPlayerPed(sourcePlayer)
        if not DoesEntityExist(sourcePed) then return end

        local myCoords    = GetEntityCoords(PlayerPedId(), false)
        local theirCoords = GetEntityCoords(sourcePed, false)
        if GetDistanceBetweenCoords(myCoords, theirCoords, true) > (Config.meboxdistance or 15) then return end
    end

    local nameColor = Config.meboxDefaultNameColor or '#e8dfc8'
    if Config.meboxJobColors and Config.meboxJobColors[data.job] then
        nameColor = Config.meboxJobColors[data.job]
    end

    meBoxLastMsg = GetGameTimer()

    SendNUIMessage({
        type        = 'mebox_add',
        name        = data.name,
        nameColor   = nameColor,
        text        = data.text,
        mode        = meBoxMode,
        timeout     = (Config.meboxtimeout or 60) * 1000,
        maxLines    = Config.meboxmaxlines or 7,
        dissipateMs = (Config.meboxdissipateafter or 300) * 1000,
    })
end)

-- Auto-fade thread: when in 'auto' mode, tell NUI to fade after the configured timeout
Citizen.CreateThread(function()
    while true do
        Wait(2000)
        if meBoxMode == 'auto' and meBoxLastMsg > 0 then
            if (GetGameTimer() - meBoxLastMsg) >= (Config.meboxtimeout or 60) * 1000 then
                SendNUIMessage({ type = 'mebox_fade' })
                meBoxLastMsg = 0
            end
        end
    end
end)

-- /mebox command: auto | persist | off | move
RegisterCommand(Config.meboxcommand or 'mebox', function(source, args)
    local mode = args[1] and args[1]:lower() or ''
    if mode == 'auto' then
        meBoxMode = 'auto'
        SetResourceKvp('mebox_mode', 'auto')
        meBoxLastMsg = GetGameTimer()
        notify('ME Box: Auto (fades after ' .. (Config.meboxtimeout or 60) .. 's)', 3000)
        SendNUIMessage({ type = 'mebox_mode', mode = 'auto', timeout = (Config.meboxtimeout or 60) * 1000 })
    elseif mode == 'persist' then
        meBoxMode = 'persist'
        SetResourceKvp('mebox_mode', 'persist')
        notify('ME Box: Persistent', 3000)
        SendNUIMessage({ type = 'mebox_mode', mode = 'persist' })
    elseif mode == 'off' then
        meBoxMode = 'off'
        SetResourceKvp('mebox_mode', 'off')
        notify('ME Box: Disabled', 3000)
        SendNUIMessage({ type = 'mebox_hide' })
    elseif mode == 'move' then
        SetNuiFocus(true, true)
        SendNUIMessage({ type = 'mebox_move_mode', enabled = true })
    elseif mode == 'small' or mode == 'normal' or mode == 'large' then
        SetResourceKvp('mebox_size', mode)
        SendNUIMessage({ type = 'mebox_size', size = mode })
        notify('ME Box size: ' .. mode, 3000)
    else
        notify('Usage: /' .. (Config.meboxcommand or 'mebox') .. ' auto | persist | off | move | small | normal | large', 4000)
    end
end)

-- Register poodlechat suggestion so the user sees help when typing
Citizen.CreateThread(function()
    Wait(3000)
    TriggerEvent('chat:addSuggestion', '/' .. (Config.meboxcommand or 'mebox'),
        'Toggle /me display box',
        {
            { name = 'mode', help = 'auto | persist | off | move | small | normal | large' }
        }
    )

    -- Load saved mebox position from KVP and send to NUI
    SendNUIMessage({ type = 'load_colors', colors = Config.sceneColors, historyMax = Config.meboxhistory or 50, theme = Config.uitheme or 'amber-frontier' })

    local savedTop  = GetResourceKvpString('mebox_pos_top')
    local savedLeft = GetResourceKvpString('mebox_pos_left')
    if savedTop and savedLeft and savedTop ~= '' and savedLeft ~= '' then
        SendNUIMessage({ type = 'mebox_position_load', top = savedTop, left = savedLeft })
    end

    -- Load saved mebox size from KVP
    local savedSize = GetResourceKvpString('mebox_size')
    if savedSize and savedSize ~= '' then
        SendNUIMessage({ type = 'mebox_size', size = savedSize })
    end

    -- Send initial mode if not default
    if meBoxMode ~= 'auto' then
        SendNUIMessage({ type = 'mebox_mode', mode = meBoxMode })
    end
end)

-- NUI callbacks for mebox position save/exit
RegisterNUICallback('meboxPositionSave', function(data, cb)
    if data and data.top and data.left then
        SetResourceKvp('mebox_pos_top', tostring(data.top))
        SetResourceKvp('mebox_pos_left', tostring(data.left))
    end
    cb({ ok = true })
end)

RegisterNUICallback('meboxMoveExit', function(data, cb)
    SetNuiFocus(false, false)
    -- Remove placeholder if present
    SendNUIMessage({ type = 'mebox_move_cleanup' })
    cb({ ok = true })
end)

-- Receive scroll relay from poodlechat and forward to our NUI
AddEventHandler('poggy_scene:relayScroll', function(deltaY)
    if meBoxMode ~= 'off' then
        SendNUIMessage({ type = 'mebox_scroll', delta = deltaY })
    end
end)

-- Show mebox when T is pressed (opening poodlechat input) and reset fade timer
Citizen.CreateThread(function()
    while true do
        Wait(0)
        if IsControlJustPressed(0, `INPUT_MP_TEXT_CHAT_ALL`) then
            if meBoxMode ~= 'off' then
                meBoxLastMsg = GetGameTimer()
                SendNUIMessage({ type = 'mebox_show_if_messages', timeout = (Config.meboxtimeout or 60) * 1000 })
            end
        end
    end
end)

