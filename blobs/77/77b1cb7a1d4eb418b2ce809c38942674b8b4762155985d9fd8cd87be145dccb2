--[[
    poggy_core — the Poggy theme, client side (0.23.0)

    Holds the theme the server sent (sv_theme.lua) and answers, for any
    Poggy script, what its page should look like:

        exports.poggy_core:ThemeFor(poggyId)
            -> { id, enabled, preset, rev, palette, skin, skinId }

    Each script's bridge (@poggy_core/template/poggy.lua) calls it when its
    page's theme.js asks (the NUI callback "poggy_core:theme"); poggy_core's
    own page asks the same callback here. Spec:
    docs/reference/poggy-theme-spec.md.

    Each player's own screen settings (0.24.0), from /poggyui (the name:
    PoggyCoreConfig.Theme.PlayerCommand, optional, "poggyui" without it):
      - size: every answer carries `scale = { player, default, fit }`, and
        theme.js zooms the page to it;
      - look: "Light" (the Ledger preset, for reading) for this player only,
        or, when the owner allows it (Theme.PlayerThemes), any ready-made
        preset. Never Custom; the owner's OwnLook list always wins;
      - less motion: `motion = "less"`, and theme.js stills every animation.
    All three are kept on the player's PC (resource KVP), per server.
    /poggyui opens the panel on poggy_core's own page (ui/script.js), which
    previews each change there as it is made.
]]

local SELF = GetCurrentResourceName()
local CORE_ID = GetResourceMetadata(SELF, "poggy_id", 0) or SELF

-- Until the server answers: the config this client downloaded with the resource.
local current = PoggyCoreConfig.Theme or {}
local rev = 0
local skins = {}                  -- poggyId -> does ui/hub/theme-<id>.css exist

local function hasSkin(id)
    if skins[id] == nil then
        local safe = tostring(id):gsub("[^%w_%-]", "")
        skins[id] = LoadResourceFile(SELF, "ui/hub/theme-" .. safe .. ".css") ~= nil
    end
    return skins[id]
end

--- The skin a script's page wears: theme-<poggyId>.css, or for the FiveM
--- line of a script (poggy_tickets_fivem) the skin of the RedM one, since
--- the page is the same page.
local function skinFor(poggyId)
    if hasSkin(poggyId) then return poggyId end
    local base = poggyId:gsub("_fivem$", "")
    if base ~= poggyId and hasSkin(base) then return base end
    return nil
end

-- ── the player's own screen settings (0.24.0) ─────────────────────────
local KVP_SCALE, KVP_LOOK, KVP_MOTION = "poggy_core:ui_scale", "poggy_core:ui_look", "poggy_core:ui_motion"
local SCALE_MIN, SCALE_MAX = 50, 200

local function clampPercent(n)
    n = tonumber(n)
    if not n then return nil end
    n = math.floor(n / 5 + 0.5) * 5
    if n < SCALE_MIN then n = SCALE_MIN elseif n > SCALE_MAX then n = SCALE_MAX end
    return n
end

-- 0 / "" / false when the player never chose (the owner's settings then).
local stored = GetResourceKvpInt(KVP_SCALE)
local playerScale = (stored and stored > 0) and (clampPercent(stored) or 0) or 0
local playerLook = GetResourceKvpString(KVP_LOOK) or ""
local playerCalm = GetResourceKvpInt(KVP_MOTION) == 1

--- May this player wear `look`? "" (the server's theme) and "light" always;
--- a ready-made preset only while the owner allows it; Custom never.
local function lookAllowed(look)
    if look == "" or look == "light" then return true end
    if look == "custom" or current.PlayerThemes ~= true then return false end
    return PoggyCore.Themes.Presets[look] ~= nil
end

--- The theme settings this player sees: the owner's, with the player's
--- preset in place of the owner's when they chose one (and may). OwnLook,
--- the rest, stay the owner's.
local function playerConfig()
    local look = playerLook
    if look == "" or not lookAllowed(look) then return current end
    local cfg = {}
    for k, v in pairs(current) do cfg[k] = v end
    cfg.Preset = look == "light" and "ledger" or look
    return cfg
end

--- What a page is told about its size; theme.js works the zoom out of it
--- (the screen's own size is only known there). The owner's default is two
--- optional lines, Theme.Scale (percent) and Theme.FitScreen; without them 100.
local function scaleAnswer()
    return {
        player  = playerScale > 0 and playerScale or nil,
        default = clampPercent(current.Scale) or 100,
        fit     = current.FitScreen == true,
    }
end

local function themeFor(poggyId, cfg)
    poggyId = type(poggyId) == "string" and poggyId ~= "" and poggyId or "unknown"
    local t = PoggyCore.Themes.For(cfg or playerConfig(), poggyId, rev)
    t.skinId = skinFor(poggyId)
    t.skin = t.skinId ~= nil
    t.scale = scaleAnswer()
    t.motion = playerCalm and "less" or nil
    return t
end

exports("ThemeFor", themeFor)

RegisterNetEvent("poggy_core:theme", function(snap)
    if type(snap) ~= "table" then return end
    current = snap
    rev = tonumber(snap.rev) or (rev + 1)
    -- poggy_core's own page (menu, input, /poggy) changes at once; every other
    -- script's page asks again the next time its script sends it a message
    -- (opening a screen), so it takes the new theme then (0.24.0).
    SendNUIMessage({ poggyTheme = themeFor(CORE_ID) })
    TriggerEvent("poggy_core:themeChanged", rev)
end)

RegisterNUICallback("poggy_core:theme", function(_, cb)
    cb(themeFor(CORE_ID))
end)

-- ── /poggyui: the player's panel ──────────────────────────────────────
-- On poggy_core's own page (ui/script.js). Size, look and motion are shown on
-- that page as they change; only "Done" keeps them. Every other Poggy screen
-- takes them the next time it opens (theme.js asks again then).
local panelOpen = false

local function lookChoices()
    local list = {
        { id = "", label = "Server theme" },
        { id = "light", label = "Light (easier to read)" },
    }
    if current.PlayerThemes == true then
        for _, id in ipairs(PoggyCore.Themes.Order) do
            local p = PoggyCore.Themes.Presets[id]
            if id ~= "custom" and p then
                list[#list + 1] = { id = id, label = p.label or id }
            end
        end
    end
    -- Each with the theme it gives poggy_core's page, for the live preview.
    for _, c in ipairs(list) do
        local cfg = current
        if c.id ~= "" then
            cfg = {}
            for k, v in pairs(current) do cfg[k] = v end
            cfg.Preset = c.id == "light" and "ledger" or c.id
        end
        c.theme = themeFor(CORE_ID, cfg)
    end
    return list
end

local function openPanel()
    if panelOpen then return end
    panelOpen = true
    SendNUIMessage({
        action = "uiprefs.open",
        scale = scaleAnswer(), min = SCALE_MIN, max = SCALE_MAX,
        look = lookAllowed(playerLook) and playerLook or "",
        looks = lookChoices(),
        motion = playerCalm,
    })
    SetNuiFocus(true, true)
end

local function closePanel()
    if not panelOpen then return end
    panelOpen = false
    SetNuiFocus(false, false)
end

exports("OpenScreenSettings", openPanel)

RegisterNUICallback("poggy_core:uiprefs:save", function(data, cb)
    data = type(data) == "table" and data or {}
    -- size: a percent, or nothing (back to the owner's default)
    local pick = clampPercent(data.scale)
    playerScale = pick or 0
    if pick then SetResourceKvpInt(KVP_SCALE, pick) else DeleteResourceKvp(KVP_SCALE) end
    -- look
    local look = type(data.look) == "string" and data.look or ""
    if not lookAllowed(look) then look = "" end
    playerLook = look
    if look ~= "" then SetResourceKvp(KVP_LOOK, look) else DeleteResourceKvp(KVP_LOOK) end
    -- motion
    playerCalm = data.motion == true
    if playerCalm then SetResourceKvpInt(KVP_MOTION, 1) else DeleteResourceKvp(KVP_MOTION) end

    SendNUIMessage({ poggyTheme = themeFor(CORE_ID) })
    closePanel()
    TriggerEvent("poggy_core:themeChanged", rev)
    cb({ ok = true })
end)

RegisterNUICallback("poggy_core:uiprefs:close", function(_, cb)
    closePanel()
    cb({ ok = true })
end)

AddEventHandler("onResourceStop", function(name)
    if name == SELF and panelOpen then SetNuiFocus(false, false) end
end)

do
    local cfg = type(PoggyCoreConfig.Theme) == "table" and PoggyCoreConfig.Theme or {}
    local cmd = type(cfg.PlayerCommand) == "string" and cfg.PlayerCommand:gsub("^/", ""):gsub("%s", "") or ""
    if cmd == "" then cmd = "poggyui" end
    RegisterCommand(cmd, function() openPanel() end, false)
    TriggerEvent("chat:addSuggestion", "/" .. cmd, "Your Poggy screens: size, light or dark, less motion")
end

TriggerServerEvent("poggy_core:theme:hello")
