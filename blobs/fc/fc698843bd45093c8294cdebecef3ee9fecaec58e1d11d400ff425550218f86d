--[[
    poggy_core — client dispatcher.

    The client deliberately does not run its own framework handshake. Five
    different client-side handshakes all racing resource start is exactly how
    poggy_util ends up with a two-second blocking round-trip and a five-second
    cache; instead the server tells us what it resolved, once, and we cache the
    character locally with proper invalidation.
]]

PoggyCore = PoggyCore or {}

local State = {
    ready      = false,
    hasAdapter = false,
    framework = "standalone",
    version   = PoggyCore.VERSION,
    caps      = {},
    waiting   = {},
    char      = nil,
    charAt    = 0,
}

PoggyCore.ClientState = State

local CACHE_MS = 5000

-- ---------------------------------------------------------------------------
-- Server handshake
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:ready", function(framework, version, caps, hasAdapter)
    State.framework  = framework or "standalone"
    State.version    = version or PoggyCore.VERSION
    State.caps       = caps or {}
    State.hasAdapter = hasAdapter and true or false
    State.ready      = true
    State.char, State.charAt = nil, 0

    for _, fn in ipairs(State.waiting) do
        CreateThread(function() fn() end)
    end
    State.waiting = {}
end)

--- Ask the server where things stand, in case we started after it did.
CreateThread(function()
    Wait(500)
    while not State.ready do
        TriggerServerEvent("poggy_core:clientHello")
        Wait(2000)
    end
end)

-- Anything that can change the character invalidates the cache immediately,
-- rather than waiting for the cache window to expire.
RegisterNetEvent("poggy_core:charLoaded", function(charId)
    State.char, State.charAt = nil, 0
    -- Re-broadcast locally so scripts can subscribe without a net event of
    -- their own, and hand them the id they almost always want next.
    TriggerEvent("poggy_core:charLoadedLocal", charId)
end)

RegisterNetEvent("poggy_core:charChanged", function()
    State.char, State.charAt = nil, 0
end)

-- 0.11.0: a job change on the server. Invalidate the cached character, then
-- re-broadcast locally, the same way charLoaded is, so a script can subscribe
-- with AddEventHandler('poggy_core:jobChangedLocal', fn(job, grade, oldJob, oldGrade)).
RegisterNetEvent("poggy_core:jobChanged", function(job, grade, oldJob, oldGrade)
    State.char, State.charAt = nil, 0
    TriggerEvent("poggy_core:jobChangedLocal", job, grade, oldJob, oldGrade)
end)

-- ---------------------------------------------------------------------------
-- Character
-- ---------------------------------------------------------------------------

local function fetchChar()
    -- Fast path. VORP writes the character into the player state bag on select
    -- and updates each field on mutation, so when it is replicated to us there
    -- is no need for a round-trip at all.
    local bag = LocalPlayer.state and LocalPlayer.state.Character
    if type(bag) == "table" and bag.CharId then
        return {
            charId    = tostring(bag.CharId),
            source    = GetPlayerServerId(PlayerId()),
            firstName = bag.FirstName or "",
            lastName  = bag.LastName or "",
            fullName  = ((bag.FirstName or "") .. " " .. (bag.LastName or "")):gsub("^%s+", ""),
            job       = bag.Job or "unemployed",
            jobLabel  = bag.JobLabel or bag.Job or "",
            jobGrade  = tonumber(bag.Grade) or 0,
            group     = bag.Group or "user",
            gender    = PoggyCore.NormaliseGender(bag.Gender),
            money     = { cash = bag.Money, gold = bag.Gold, rol = bag.Rol },
            onDuty    = nil,
            native    = bag,
        }
    end

    -- Slow path, through our own callback transport.
    return PoggyCore.CallbackAwait("poggy_core:getChar")
end

--- Local character, cached briefly and invalidated on any change event.
---@return PoggyChar|nil
function PoggyCore.GetLocalChar(force)
    local now = GetGameTimer()
    if not force and State.char and (now - State.charAt) < CACHE_MS then
        return State.char
    end
    local char = fetchChar()
    if char then
        State.char, State.charAt = char, now
    end
    return char
end

-- ---------------------------------------------------------------------------
-- Core construction
-- ---------------------------------------------------------------------------

local cores = {}

local function buildCore(resource)
    local tag = resource or "unknown"
    local Core = {}

    Core.VERSION  = PoggyCore.VERSION
    Core.Err      = PoggyCore.Err
    Core.Currency = PoggyCore.Currency
    Core.Caps     = PoggyCore.Caps

    function Core.IsReady() return State.ready end
    function Core.GetFramework() return State.framework end
    function Core.Has(capability) return State.caps[capability] == true end

    --- True when a real adapter is driving the detected framework. False means
    --- no framework, or one poggy_core cannot drive yet; either way framework
    --- calls will refuse.
    function Core.HasAdapter() return State.ready and State.hasAdapter end

    function Core.Ready(fn)
        if not PoggyCore.IsCallable(fn) then return end
        if State.ready then
            CreateThread(function() fn() end)
        else
            State.waiting[#State.waiting + 1] = fn
        end
    end

    function Core.WaitReady(timeoutMs)
        local deadline = GetGameTimer() + (timeoutMs or 30000)
        while not State.ready and GetGameTimer() < deadline do Wait(50) end
        return State.ready
    end

    function Core.RequireVersion(min)
        local function parts(v)
            local a, b, c = tostring(v):match("^(%d+)%.(%d+)%.?(%d*)$")
            return tonumber(a) or 0, tonumber(b) or 0, tonumber(c) or 0
        end
        local ha, hb, hc = parts(PoggyCore.VERSION)
        local na, nb, nc = parts(min)
        if (ha * 10000 + hb * 100 + hc) < (na * 10000 + nb * 100 + nc) then
            error(("[poggy_core] %s needs poggy_core >= %s but this is %s.")
                :format(tag, min, PoggyCore.VERSION), 2)
        end
        return true
    end

    -- --- character ----------------------------------------------------------

    ---@return PoggyChar|nil
    function Core.GetChar(force) return PoggyCore.GetLocalChar(force) end

    function Core.GetCharId()
        local c = PoggyCore.GetLocalChar()
        return c and c.charId or nil
    end

    -- --- jobs ---------------------------------------------------------------

    Core.Job = {}

    function Core.Job.Get()
        local c = PoggyCore.GetLocalChar()
        if not c then return nil end
        return c.job, c.jobGrade, c.jobLabel, c.jobGradeLabel, c.onDuty
    end

    function Core.Job.Has(want, minGrade)
        local c = PoggyCore.GetLocalChar()
        if not c then return false end
        local list = type(want) == "table" and want or { want }
        if not PoggyCore.InList(c.job, list) then return false end
        if minGrade and c.jobGrade < minGrade then return false end
        return true
    end

    function Core.Job.IsLaw()
        local c = PoggyCore.GetLocalChar()
        return c and PoggyCore.InList(c.job, PoggyCoreConfig.LawJobs) or false
    end

    function Core.Job.IsMedical()
        local c = PoggyCore.GetLocalChar()
        return c and PoggyCore.InList(c.job, PoggyCoreConfig.MedicalJobs) or false
    end

    function Core.Job.OnChange(fn)
        if PoggyCore.IsCallable(fn) then AddEventHandler("poggy_core:jobChanged", fn) end
    end

    -- --- notifications ------------------------------------------------------

    local notify = setmetatable({}, {
        __call = function(_, text, kind, duration)
            return PoggyCore.RenderNotify(text, kind, duration)
        end,
    })
    function notify.Rich(opts) return PoggyCore.RenderNotifyRich(opts) end

    -- Style-preserving variants; see the server side for when to use which.
    function notify.Tip(text, duration)
        return PoggyCore.RenderStyled("tip", { text = text, duration = duration })
    end
    function notify.Right(text, duration)
        return PoggyCore.RenderStyled("right", { text = text, duration = duration })
    end
    function notify.Objective(text, duration)
        return PoggyCore.RenderStyled("objective", { text = text, duration = duration })
    end
    function notify.Top(title, subtitle, duration)
        return PoggyCore.RenderStyled("top",
            { title = title, subtitle = subtitle, duration = duration })
    end
    function notify.Advanced(text, dict, icon, color, duration)
        return PoggyCore.RenderStyled("advanced",
            { text = text, dict = dict, icon = icon, color = color, duration = duration })
    end

    Core.Notify = notify

    -- --- callbacks ----------------------------------------------------------

    Core.Callback = {
        --- Call a server callback and block until it answers. Returns nil on timeout.
        Await    = function(name, ...) return PoggyCore.CallbackAwait(name, ...) end,
        --- Non-blocking form.
        Trigger  = function(name, cb, ...) return PoggyCore.CallbackTrigger(name, cb, ...) end,
        --- Register something the server can call on us.
        Register = function(name, fn) return PoggyCore.CallbackRegister(tag, name, fn) end,
    }

    -- --- prompts ------------------------------------------------------------

    Core.Prompt = PoggyCore.BuildPromptApi(tag)

    -- --- menus and input (0.14.0) -------------------------------------------
    -- poggy_core's own page (ui/, client/cl_menu.lua): the same screens on
    -- every framework, no vorp_menu or vorp_inputs needed. Both Open and Text
    -- wait for the player, so they need a thread.

    Core.Menu = {
        --- Open a list menu and wait. Returns { value, index, item }, or
        --- false plus 'closed' (the player backed out), 'bad_argument' or
        --- 'needs_thread'. Opening one while another is up replaces it.
        ---@param opts table { title, items = { { label, value?, desc?, right?, disabled? } }, subtitle?, cursor?, closeText? }
        Open = function(opts)
            local ok, value, err = PoggyCore.Ui.Menu(opts, tag)
            if not ok then return false, err end
            return value, nil
        end,
        --- Take down whatever is open; its caller gets false, 'closed'.
        Close = function() return PoggyCore.Ui.Close(), nil end,
        --- Is a poggy_core menu or input on screen?
        IsOpen = function() return PoggyCore.Ui.IsOpen() end,

        --- The framework's own menu handle, kept for scripts that still use
        --- it. Using it makes your code framework-specific.
        Native = function()
            if State.framework == "vorp" and GetResourceState("vorp_menu") == "started" then
                return exports.vorp_menu:GetMenuData()
            end
            if (State.framework == "rsg" or State.framework == "rpx") and GetResourceState("ox_lib") == "started" then
                return lib   -- ox_lib's global
            end
            if State.framework == "qbr" and GetResourceState("qbr-menu") == "started" then
                return exports["qbr-menu"]
            end
            if State.framework == "redem" and GetResourceState("redemrp_menu_base") == "started" then
                local data
                TriggerEvent("redemrp_menu_base:getData", function(d) data = d end)
                return data
            end
            return nil, PoggyCore.Err.UNSUPPORTED
        end,
    }

    --- One text box. Returns the text (a number when numeric = true), or
    --- false plus 'closed', 'bad_argument' or 'needs_thread'. Core.Input(opts)
    --- still works and does the same as Core.Input.Text(opts).
    ---@param opts table { title, placeholder?, default?, maxLength?, numeric?, submitText?, cursor? }
    local function inputText(opts)
        local ok, value, err = PoggyCore.Ui.Input(opts, tag)
        if not ok then return false, err end
        return value, nil
    end

    Core.Input = setmetatable({ Text = inputText }, {
        __call = function(_, opts) return inputText(opts) end,
    })

    return Core
end

exports("Get", function()
    local resource = GetInvokingResource() or "poggy_core"
    if not cores[resource] then cores[resource] = buildCore(resource) end
    return cores[resource]
end)

--- Internal handle for our own client files, so cl_dispatch does not have to
--- call our own export to reach us. Mirrors PoggyCore.Self() on the server.
function PoggyCore.Self()
    if not cores["poggy_core"] then cores["poggy_core"] = buildCore("poggy_core") end
    return cores["poggy_core"]
end
