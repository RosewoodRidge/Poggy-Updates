--[[
    poggy_core — the Poggy theme, server side (0.23.0)

    Keeps the live theme (PoggyCoreConfig.Theme) and hands it to every
    client, which answers each Poggy script's page from it (cl_theme.lua,
    ui/hub/theme.js). Spec: docs/reference/poggy-theme-spec.md.

    The theme settings are live: after the settings hub saves poggy_core's
    config.lua, sv_hub_save.lua calls PoggyCore.Theme.Reload(), which reads
    the Theme block back out of the file and sends it to everyone. Nothing
    restarts. poggy_core's own page changes at once; a script's page asks once,
    when it loads, so it follows when that script restarts or the player reconnects.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local SELF = GetCurrentResourceName()

local Theme = { rev = 1 }
PoggyCore.Theme = Theme

--- The theme as clients get it: only the plain data, never functions.
local function snapshot()
    local cfg = PoggyCoreConfig.Theme or {}
    local own = {}
    if type(cfg.OwnLook) == "table" then
        for _, v in ipairs(cfg.OwnLook) do
            if type(v) == "string" and v ~= "" then own[#own + 1] = v end
        end
    end
    local custom = {}
    if type(cfg.Custom) == "table" then
        for k, v in pairs(cfg.Custom) do
            if type(v) == "string" or type(v) == "number" then custom[k] = v end
        end
    end
    return {
        Preset  = type(cfg.Preset) == "string" and cfg.Preset or PoggyCore.Themes.Default,
        Custom  = custom,
        OwnLook = own,
        rev     = Theme.rev,
    }
end

function Theme.Snapshot() return snapshot() end

--- Send the theme to one player, or to everyone (-1).
function Theme.Send(target)
    TriggerClientEvent("poggy_core:theme", target or -1, snapshot())
end

--- Read PoggyCoreConfig.Theme again from config.lua on disk, as the hub left
--- it, and send it to everyone. The file is run in a sandbox of its own: only
--- the Theme block is taken from it, nothing else in the running config moves.
--- Returns true, or false and why.
function Theme.Reload()
    local text = LoadResourceFile(SELF, "config.lua")
    if not text then return false, "config.lua could not be read" end
    local env = setmetatable({}, { __index = _G })
    local chunk, err = load(text, "@poggy_core/config.lua", "t", env)
    if not chunk then
        Util.Warn("theme: config.lua does not parse, the theme was not reloaded: %s", tostring(err))
        return false, err
    end
    local ok, runErr = pcall(chunk)
    if not ok then
        Util.Warn("theme: config.lua failed to run, the theme was not reloaded: %s", tostring(runErr))
        return false, runErr
    end
    local fresh = type(env.PoggyCoreConfig) == "table" and env.PoggyCoreConfig.Theme or nil
    if type(fresh) ~= "table" then return false, "no PoggyCoreConfig.Theme in config.lua" end

    PoggyCoreConfig.Theme = fresh
    Theme.rev = Theme.rev + 1
    Theme.Send(-1)
    Util.Log("theme: %s, sent to every player", (PoggyCore.Themes.Palette(fresh)))
    return true
end

-- A client asks once its theme code has loaded (cl_theme.lua).
RegisterNetEvent("poggy_core:theme:hello", function()
    Theme.Send(source)
end)
