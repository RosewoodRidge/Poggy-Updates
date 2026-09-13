--[[
    poggy_core — notifications, server side.

    Notifications do not need a framework. Every framework's notify function is
    ultimately a wrapper over the same RDR2 natives, and the four signatures are
    mutually incompatible (QBR takes a numeric id 1-9 and no type string at all).
    So poggy_core owns the vocabulary and picks a renderer at runtime.

    Renderer order comes from config. The default prefers poggy_core's own
    native renderer (client/cl_notify_native.lua, the same natives poggy_util
    drew with, so nothing looks different), then the framework's own, then chat
    so a notification is never silently lost. 'poggy_util' in an older config is
    read as 'native': poggy_core no longer needs poggy_util to be running.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util

local KINDS = { info = true, success = true, error = true, warning = true }

--- Resolve which renderer to use, once, and remember it.
local resolved = nil
local function renderer()
    if resolved then return resolved end
    for _, name in ipairs(PoggyCoreConfig.NotifyRenderers or {}) do
        if name == "native" or name == "poggy_util" then
            resolved = "native"; break
        elseif name == "framework" then
            local a = PoggyCore.State and PoggyCore.State.adapter
            if a and type(a.notify) == "function" and PoggyCore.State.framework ~= "standalone" then
                resolved = "framework"; break
            end
        elseif name == "chat" then
            resolved = "chat"; break
        end
    end
    resolved = resolved or "chat"
    Util.Log("Notification renderer: %s", resolved)
    return resolved
end

--- Send a notification to a player.
---@param src number
---@param text string
---@param kind string|nil 'info' | 'success' | 'error' | 'warning'
---@param duration number|nil ms
---@return boolean
function PoggyCore.Notify(src, text, kind, duration)
    local n = tonumber(src)
    if not n or type(text) ~= "string" then return false end
    kind = KINDS[kind] and kind or "info"
    duration = tonumber(duration) or PoggyCoreConfig.NotifyDuration

    local mode = renderer()

    if mode == "native" then
        -- The native renderer is client-side: one net event to our own client file.
        TriggerClientEvent("poggy_core:notify", n, text, kind, duration)
        return true
    end

    if mode == "framework" then
        local a = PoggyCore.State.adapter
        local ok = pcall(function() a:notify(n, text, kind, duration) end)
        if ok then return true end
        -- Fall through to chat rather than swallowing it.
    end

    TriggerClientEvent("chat:addMessage", n, {
        color = (kind == "error" and { 180, 81, 78 })
             or (kind == "success" and { 127, 166, 106 })
             or (kind == "warning" and { 194, 161, 92 })
             or { 243, 231, 216 },
        args  = { "Server", text },
    })
    return true
end

--- Richer notification. Falls back to the plain form on any renderer that
--- cannot do the full shape.
---@param src number
---@param opts table { title, description, kind, duration, dict, icon, color }
---@return boolean
function PoggyCore.NotifyRich(src, opts)
    local n = tonumber(src)
    if not n or type(opts) ~= "table" then return false end
    opts.kind     = KINDS[opts.kind] and opts.kind or "info"
    opts.duration = tonumber(opts.duration) or PoggyCoreConfig.NotifyDuration

    local mode = renderer()

    if mode == "native" then
        TriggerClientEvent("poggy_core:notifyRich", n, opts)
        return true
    end

    if mode == "framework" then
        local a = PoggyCore.State.adapter
        if type(a.notifyRich) == "function" then
            local ok = pcall(function() a:notifyRich(n, opts) end)
            if ok then return true end
        end
    end

    local text = opts.title and opts.description
        and (opts.title .. " — " .. opts.description)
        or (opts.title or opts.description or "")
    return PoggyCore.Notify(n, text, opts.kind, opts.duration)
end

-- ---------------------------------------------------------------------------
-- Styled notifications
-- ---------------------------------------------------------------------------
-- The four semantic kinds above are the portable API. These five are the RDR2
-- notification *styles*, and they exist so a script that has always shown a
-- bottom-of-screen objective banner keeps showing exactly that after it is
-- migrated. Changing where a message appears is a visible change to the game,
-- and a refactor should not make one.
--
-- Rendering is entirely client-side, so the server just names the style.

local STYLE_ARGS = {
    tip         = { "text", "duration" },
    right       = { "text", "duration" },
    objective   = { "text", "duration" },
    top         = { "title", "subtitle", "duration" },
    advanced    = { "text", "dict", "icon", "color", "duration", "quality", "showQuality" },
    -- 0.11.0: the remaining styles poggy_util offered, drawn by poggy_core itself.
    location    = { "text", "location", "duration" },
    left        = { "title", "subtitle", "dict", "icon", "color", "duration" },
    leftRank    = { "title", "subtitle", "dict", "icon", "color", "duration" },
    basicTop    = { "text", "duration" },
    center      = { "text", "color", "duration" },
    bottomRight = { "text", "duration" },
    fail        = { "title", "subtitle", "duration" },
    dead        = { "title", "audioRef", "audioName", "duration" },
    update      = { "title", "subtitle", "duration" },
    warning     = { "title", "subtitle", "audioRef", "audioName", "duration" },
}

--- Is this a style name notify.styled accepts?
function PoggyCore.IsNotifyStyle(style)
    return STYLE_ARGS[style] ~= nil
end

--- Send a style-specific notification.
---@param src number
---@param style string a key of STYLE_ARGS
---@param args table keyed by the names in STYLE_ARGS
---@return boolean
function PoggyCore.NotifyStyled(src, style, args)
    local n = tonumber(src)
    if not n or not STYLE_ARGS[style] then return false end
    TriggerClientEvent("poggy_core:notifyStyled", n, style, args or {})
    return true
end

--- Broadcast to everyone. Used by admin tooling more than by scripts.
function PoggyCore.NotifyAll(text, kind, duration)
    for _, src in ipairs(GetPlayers()) do
        PoggyCore.Notify(tonumber(src), text, kind, duration)
    end
end
