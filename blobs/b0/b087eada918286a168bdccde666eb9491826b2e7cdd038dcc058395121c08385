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

local function themeFor(poggyId)
    poggyId = type(poggyId) == "string" and poggyId ~= "" and poggyId or "unknown"
    local t = PoggyCore.Themes.For(current, poggyId, rev)
    t.skinId = skinFor(poggyId)
    t.skin = t.skinId ~= nil
    return t
end

exports("ThemeFor", themeFor)

RegisterNetEvent("poggy_core:theme", function(snap)
    if type(snap) ~= "table" then return end
    current = snap
    rev = tonumber(snap.rev) or (rev + 1)
    -- poggy_core's own page (menu, input, /poggy) changes at once; every other
    -- script's page asked once when it loaded, and takes the new theme when
    -- that script restarts or the player reconnects (0.23.1).
    SendNUIMessage({ poggyTheme = themeFor(CORE_ID) })
    TriggerEvent("poggy_core:themeChanged", rev)
end)

RegisterNUICallback("poggy_core:theme", function(_, cb)
    cb(themeFor(CORE_ID))
end)

TriggerServerEvent("poggy_core:theme:hello")
