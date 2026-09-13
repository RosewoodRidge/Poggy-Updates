--[[
    poggy_core — notifications, client side.

    Renderer selection, in the order set by PoggyCoreConfig.NotifyRenderers:

      1. 'native': poggy_core's own RDR2 UI-feed renderer (cl_notify_native.lua).
         The same natives poggy_util and vorp_core draw with, so every style
         looks exactly as it always has. Needs nothing else to be running.
         'poggy_util' is accepted as the old name for this renderer, so a
         config written before 0.11.0 keeps working.
      2. 'framework': the detected framework's own client notification API.
      3. 'chat': so a notification is never silently swallowed.

    The native renderer is part of this resource, so in practice it is always
    the one used; the other two exist for a native call that errors.
]]

PoggyCore = PoggyCore or {}

local N = PoggyCore.NativeNotify

local resolved = nil

local function pick()
    if resolved then return resolved end
    for _, name in ipairs(PoggyCoreConfig.NotifyRenderers or {}) do
        if (name == "native" or name == "poggy_util") and N then
            resolved = "native"; break
        elseif name == "framework" then
            local fw = PoggyCore.ClientState and PoggyCore.ClientState.framework
            if fw and fw ~= "standalone" then resolved = "framework"; break end
        elseif name == "chat" then
            resolved = "chat"; break
        end
    end
    return resolved or "chat"
end

local function chatFallback(text, kind)
    TriggerEvent("chat:addMessage", {
        color = (kind == "error" and { 180, 81, 78 })
             or (kind == "success" and { 127, 166, 106 })
             or (kind == "warning" and { 194, 161, 92 })
             or { 243, 231, 216 },
        args = { "Server", text },
    })
end

--- Plain notification, drawn as a right-hand tip.
---@param text string
---@param kind string|nil 'info' | 'success' | 'error' | 'warning'
---@param duration number|nil
---@return boolean
function PoggyCore.RenderNotify(text, kind, duration)
    if type(text) ~= "string" or text == "" then return false end
    duration = tonumber(duration) or PoggyCoreConfig.NotifyDuration

    local mode = pick()

    if mode == "native" then
        if pcall(N.Right, text, duration) then return true end
    end

    if mode ~= "chat" then
        local fw = PoggyCore.ClientState.framework
        local ok = pcall(function()
            if fw == "vorp" then
                exports.vorp_core:GetCore().NotifyRightTip(text, duration)
            elseif fw == "rsg" then
                exports["rsg-core"]:GetCoreObject().Functions.Notify(text, nil, kind, duration)
            elseif fw == "qbr" then
                -- QBR takes a numeric style id and no type string at all.
                -- 4 is ShowBasicTopNotification, the safest of the nine.
                exports["qbr-core"]:Notify(4, text, duration)
            elseif fw == "redem" then
                exports["redem_roleplay"]:ShowSimpleRightText(text, duration)
            elseif fw == "rpx" then
                lib.notify({ description = text, type = (kind == "warning") and "inform" or (kind or "inform") })
            else
                error("no framework renderer")
            end
        end)
        if ok then return true end
    end

    chatFallback(text, kind)
    return true
end

--- Richer notification: a title and a description together.
---@param opts table { title, description, kind, duration }
---@return boolean
function PoggyCore.RenderNotifyRich(opts)
    if type(opts) ~= "table" then return false end
    local title    = opts.title
    local desc     = opts.description
    local duration = tonumber(opts.duration) or PoggyCoreConfig.NotifyDuration

    if not title and not desc then return false end
    if not title or not desc then
        return PoggyCore.RenderNotify(title or desc, opts.kind, duration)
    end

    local mode = pick()

    if mode == "native" then
        if pcall(N.Top, title, desc, duration) then return true end
    end

    if mode ~= "chat" then
        local fw = PoggyCore.ClientState.framework
        local ok = pcall(function()
            if fw == "vorp" then
                exports.vorp_core:GetCore().NotifySimpleTop(title, desc, duration)
            elseif fw == "rsg" then
                exports["rsg-core"]:GetCoreObject().Functions.Notify(title, desc, opts.kind, duration)
            elseif fw == "qbr" then
                exports["qbr-core"]:Notify(7, title, duration, desc)
            elseif fw == "redem" then
                exports["redem_roleplay"]:ShowTopNotification(title, desc, duration)
            elseif fw == "rpx" then
                lib.notify({ title = title, description = desc, type = opts.kind or "inform" })
            else
                error("no framework renderer")
            end
        end)
        if ok then return true end
    end

    chatFallback(title .. " — " .. desc, opts.kind)
    return true
end

-- ---------------------------------------------------------------------------
-- Styled notifications
-- ---------------------------------------------------------------------------
-- Exact-parity renderers for the RDR2 notification styles the Poggy scripts
-- use. Each draws natively first, then through VORP's client event for the
-- same style if the native call errors, then falls back to the portable path.
--
-- The bottom-of-screen banner is 'objective' here, NotifyObjective in
-- poggy_util and vorp:TipBottom in VORP: one style under three names.
--
-- `timed` styles hold the banner on screen for `duration` and then clear it,
-- so they run on their own thread rather than blocking the caller.

local STYLE = {
    tip = {
        native = function(a) N.Tip(a.text, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:Tip", a.text, a.duration) end,
    },
    right = {
        native = function(a) N.Right(a.text, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:TipRight", a.text, a.duration) end,
    },
    objective = {
        native = function(a) N.Objective(a.text, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:TipBottom", a.text, a.duration) end,
    },
    top = {
        native = function(a) N.Top(a.title, a.subtitle, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:ShowTopNotification", a.title, a.subtitle, a.duration) end,
    },
    advanced = {
        native = function(a) N.Advanced(a.text, a.dict, a.icon, a.color, a.duration, a.quality, a.showQuality) end,
        vorp   = function(a)
            TriggerEvent("vorp:ShowAdvancedRightNotification", a.text, a.dict, a.icon, a.color, a.duration, a.quality)
        end,
    },
    -- 0.11.0: the rest of the styles poggy_util offered.
    location = {
        native = function(a) N.Location(a.text, a.location, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:NotifyTop", a.text, a.location, a.duration) end,
    },
    left = {
        native = function(a) N.Left(a.title, a.subtitle, a.dict, a.icon, a.duration, a.color) end,
        vorp   = function(a) TriggerEvent("vorp:NotifyLeft", a.title, a.subtitle, a.dict, a.icon, a.duration, a.color) end,
    },
    leftRank = {
        native = function(a) N.LeftRank(a.title, a.subtitle, a.dict, a.icon, a.duration, a.color) end,
        vorp   = function(a) TriggerEvent("vorp:LeftRank", a.title, a.subtitle, a.dict, a.icon, a.duration, a.color) end,
    },
    basicTop = {
        native = function(a) N.BasicTop(a.text, a.duration) end,
    },
    center = {
        native = function(a) N.Center(a.text, a.duration, a.color) end,
        vorp   = function(a) TriggerEvent("vorp:ShowSimpleCenterText", a.text, a.duration) end,
    },
    bottomRight = {
        native = function(a) N.BottomRight(a.text, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:ShowBottomRight", a.text, a.duration) end,
    },
    fail = {
        timed  = true,
        native = function(a) N.Fail(a.title, a.subtitle, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:failmissioNotifY", a.title, a.subtitle, a.duration) end,
    },
    dead = {
        timed  = true,
        native = function(a) N.Dead(a.title, a.audioRef, a.audioName, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:deadplayerNotifY", a.title, a.audioRef, a.audioName, a.duration) end,
    },
    update = {
        timed  = true,
        native = function(a) N.Update(a.title, a.subtitle, a.duration) end,
        vorp   = function(a) TriggerEvent("vorp:updatemissioNotify", a.title, a.subtitle, a.duration) end,
    },
    warning = {
        timed  = true,
        native = function(a) N.Warning(a.title, a.subtitle, a.audioRef, a.audioName, a.duration) end,
        vorp   = function(a)
            TriggerEvent("vorp:warningNotify", a.title, a.subtitle, a.audioRef, a.audioName, a.duration)
        end,
    },
}

--- The style names, for validation and /poggycore output.
PoggyCore.NotifyStyles = {}
for name in pairs(STYLE) do PoggyCore.NotifyStyles[name] = true end

local function renderStyle(def, a)
    local mode = pick()

    if mode == "native" and def.native then
        if pcall(def.native, a) then return end
    end

    if mode ~= "chat" and def.vorp and PoggyCore.ClientState.framework == "vorp" then
        if pcall(def.vorp, a) then return end
    end

    -- No equivalent on this renderer: fall back to the portable path rather
    -- than showing nothing.
    local text = a.text or ((a.title and a.subtitle) and (a.title .. " — " .. a.subtitle))
        or a.title or a.subtitle or ""
    PoggyCore.RenderNotify(text, "info", a.duration)
end

--- Render one style. Returns false for a style name that does not exist.
---@param style string see STYLE above
---@param a table style arguments
---@return boolean
function PoggyCore.RenderStyled(style, a)
    local def = STYLE[style]
    if not def then return false end
    a = a or {}
    a.duration = tonumber(a.duration) or PoggyCoreConfig.NotifyDuration

    if def.timed then
        CreateThread(function() renderStyle(def, a) end)
    else
        renderStyle(def, a)
    end
    return true
end

RegisterNetEvent("poggy_core:notifyStyled", function(style, a)
    PoggyCore.RenderStyled(style, a)
end)

-- Server-driven notifications land here.
RegisterNetEvent("poggy_core:notify", function(text, kind, duration)
    PoggyCore.RenderNotify(text, kind, duration)
end)

RegisterNetEvent("poggy_core:notifyRich", function(opts)
    PoggyCore.RenderNotifyRich(opts)
end)
