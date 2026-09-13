--[[
    poggy_core — server dispatcher.

    Resolves the framework, picks an adapter, and builds the Core object that
    every consumer gets from exports.poggy_core:Get().

    Get() returns a Core bound to the *calling* resource. That gives us three
    things for free: storage ids namespaced per resource, usable-item
    registrations attributed to the right owner so they are dropped when it
    stops and re-registered after an inventory restart (sv_usables.lua), and
    warnings that name the resource at fault.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

local State = {
    ready     = false,
    framework = "standalone",
    adapter   = nil,
    core      = nil,
    --- True when we detected a framework but have no adapter written for it, so
    --- the standalone adapter is standing in and every framework call refuses.
    --- Consumers check this through Core.HasAdapter() before deciding whether
    --- they can rely on us; without it, a half-supported framework would look
    --- exactly like a working one.
    noAdapter = false,
    waiting   = {},   -- Core.Ready callbacks queued before resolution
}

PoggyCore.State = State

-- ---------------------------------------------------------------------------
-- Resolution
-- ---------------------------------------------------------------------------

local function selectAdapter(id)
    return PoggyCore.Adapters[id] or PoggyCore.Adapters.standalone
end

--- Set by the last resolve so `/poggycore detect` can report why it landed
--- where it did instead of leaving you to infer it from one warning line.
PoggyCore.LastResolve = { attempts = 0, adapterError = nil, probed = {} }

local function resolve()
    local started = PoggyCore.StartedFrameworks()
    if #started > 1 then
        Util.Warn("More than one framework core is running (%s). Detection order in config decides; set ForceFramework to be explicit.",
            table.concat(started, ", "))
    end

    local deadline = GetGameTimer() + (PoggyCoreConfig.FrameworkTimeout or 30000)
    local id, core

    if PoggyCoreConfig.ForceFramework == "standalone" then
        id = "standalone"
    else
        -- DetectFramework returns 'standalone' both while a framework core is
        -- still starting and when there is genuinely nothing there. Those are
        -- not the same answer, so 'standalone' cannot end the loop — treating
        -- it as final is what made this give up on the first pass and report no
        -- framework on a server that has one. Keep asking until the deadline.
        local attempts = 0
        while true do
            attempts = attempts + 1
            id, core = PoggyCore.DetectFramework()
            if core then break end
            if GetGameTimer() >= deadline then
                id = "standalone"
                break
            end
            Wait(500)
        end
        PoggyCore.LastResolve.attempts = attempts
    end

    id = id or "standalone"

    -- Detected a framework we have no adapter for. Say so loudly: silently
    -- standing the standalone adapter in would report framework='rsg' while
    -- refusing every call, which looks identical to a working install.
    State.noAdapter = (id ~= "standalone") and (PoggyCore.Adapters[id] == nil)
    if State.noAdapter then
        Util.Error("Detected %s, but no adapter for it has been written yet. " ..
            "Every framework call will refuse. Consumers should check Core.HasAdapter().", id)
    end

    local adapter = selectAdapter(id)
    local ok, err = adapter:init(core)
    PoggyCore.LastResolve.adapterError = (not ok) and tostring(err) or nil
    if not ok then
        Util.Error("The %s adapter failed to start: %s. Falling back to standalone.",
            id, tostring(err))
        adapter = PoggyCore.Adapters.standalone
        adapter:init(nil)
        id = "standalone"
    end

    State.framework = id
    State.adapter   = adapter
    State.core      = core
    State.ready     = true

    if id == "standalone" then
        Util.Warn("No framework detected. Every framework call will refuse with '%s'.", Err.UNSUPPORTED)
        -- Say what was actually looked at. "No framework detected" on a server
        -- that plainly has one is not a useful thing to read at 2am.
        for _, fid in ipairs(PoggyCoreConfig.DetectionOrder) do
            local def = PoggyCore.Frameworks[fid]
            local state = GetResourceState(def.resource)
            if state == "started" then
                Util.Warn("  %s is started but its handshake never succeeded. " ..
                    "poggy_core needs exports.%s to answer with a usable core object.",
                    def.resource, def.resource)
            elseif state ~= "missing" then
                Util.Warn("  %s is '%s', not 'started'.", def.resource, state)
            end
        end
    end

    -- The banner once per start, on the first resolve; a later `/poggycore resolve`
    -- gets one plain line instead.
    local fwText = id == "standalone" and "^3no framework (standalone)^7"
        or ("^2%s^7 ^9(%s)^7"):format(adapter.label, id)
    if not PoggyCore.BannerShown then
        PoggyCore.BannerShown = true
        local lines = {
            ("^2✅ ready^7  ·  v%s  ·  framework: %s  ·  console: ^5poggycore^7 [status|update|selftest|detect]")
                :format(PoggyCore.VERSION, fwText),
        }
        if PoggyCore.Updates and PoggyCore.Updates.Describe then
            lines[2] = PoggyCore.Updates.Describe()
        end
        Util.Banner(lines)
    else
        Util.Log("^2✅ ready^7  ·  v%s  ·  framework: %s", PoggyCore.VERSION, fwText)
    end

    for _, fn in ipairs(State.waiting) do
        CreateThread(function() fn() end)
    end
    State.waiting = {}

    TriggerEvent("poggy_core:ready", id)
    TriggerClientEvent("poggy_core:ready", -1, id, PoggyCore.VERSION,
        PoggyCore.CapsSnapshot(), not State.noAdapter)
end

--- The capability map as a plain table, for shipping to clients.
function PoggyCore.CapsSnapshot()
    if not State.adapter then return {} end
    local out = {}
    for k, v in pairs(State.adapter.caps or {}) do
        if v == true then out[k] = true end
    end
    return out
end

--- A client that started after us, or reconnected, asking where things stand.
RegisterNetEvent("poggy_core:clientHello", function()
    if not State.ready then return end
    TriggerClientEvent("poggy_core:ready", source, State.framework,
        PoggyCore.VERSION, PoggyCore.CapsSnapshot(), not State.noAdapter)
end)

CreateThread(function()
    -- One tick, so every framework resource that starts alongside us has begun,
    -- and so every file in this resource has finished loading (sv_callbacks.lua
    -- loads after this one and defines BuildCallbackApi).
    Wait(0)

    -- Internal callbacks our own client files rely on. Registered here rather
    -- than at file scope because BuildCallbackApi is defined in a later file.
    local Self = PoggyCore.Self()

    --- The client's own character, for the slow path in cl_core.lua when the
    --- framework has not replicated a usable state bag.
    Self.Callback.Register("poggy_core:getChar", function(src)
        return Self.GetChar(src)
    end)

    resolve()
end)

--- Re-run detection on demand. Used by /poggycore resolve, so a framework that
--- came up late can be picked up without restarting the resource.
function PoggyCore.Resolve()
    resolve()
end

-- A late-started framework (someone ran `ensure vorp_core` by hand) should still
-- be picked up rather than leaving us stuck on standalone for the session.
AddEventHandler("onResourceStart", function(resource)
    if State.framework ~= "standalone" then return end
    for id, def in pairs(PoggyCore.Frameworks) do
        if def.resource == resource then
            Util.Log("%s started after us. Re-resolving.", resource)
            CreateThread(function() Wait(1000); resolve() end)
            return
        end
    end
end)

-- ---------------------------------------------------------------------------
-- Guards
-- ---------------------------------------------------------------------------

--- Common preamble for any call that needs a framework and a player.
---@return table|nil adapter
---@return number|nil src
---@return string|nil err
local function guardSrc(src)
    if not State.ready then return nil, nil, Err.NOT_READY end
    local n, err = Util.Source(src)
    if not n then return nil, nil, err end
    return State.adapter, n
end

local function guard()
    if not State.ready then return nil, Err.NOT_READY end
    return State.adapter
end

-- ---------------------------------------------------------------------------
-- Core construction
-- ---------------------------------------------------------------------------

-- ---------------------------------------------------------------------------
-- Item registry cache (inv.items / inv.itemInfo), shared by every consumer
-- ---------------------------------------------------------------------------

local itemCache  = { at = nil, list = nil, byName = nil }
local imageCache = {}   -- item name -> boolean, whether its icon file exists

--- The registry, from cache when it is younger than ItemCacheSeconds.
local function itemRegistry(a)
    local ttl = (tonumber(PoggyCoreConfig.ItemCacheSeconds) or 300) * 1000
    if itemCache.list and itemCache.at and (GetGameTimer() - itemCache.at) < ttl then
        return itemCache.list, itemCache.byName
    end
    local list, err = a:itemRegistry()
    if type(list) ~= "table" then return nil, nil, err or Err.FRAMEWORK_ERR end
    local byName = {}
    for _, it in ipairs(list) do byName[it.name] = it end
    itemCache.list, itemCache.byName, itemCache.at = list, byName, GetGameTimer()
    return list, byName
end

--- A copy, so a caller in this Lua state cannot edit the cache.
local function copyItem(a, it, checkImages)
    local c = {}
    for k, v in pairs(it) do c[k] = v end
    if checkImages then
        if imageCache[it.name] == nil then
            imageCache[it.name] = a:itemImageExists(it.name) and true or false
        end
        c.hasImage = imageCache[it.name]
    end
    return c
end

-- An inventory restart can mean new items; read the registry again next time.
AddEventHandler("onResourceStart", function(resource)
    if resource == "vorp_inventory" or resource == "rsg-inventory" then
        itemCache.list, itemCache.byName, itemCache.at = nil, nil, nil
        imageCache = {}
    end
end)

local cores = {}   -- resource name -> bound Core

local function buildCore(resource)
    local tag = resource or "unknown"

    --- Storage ids are namespaced per resource so two scripts cannot collide,
    --- and so we stay clear of the prefixes RSG reserves.
    ---
    --- A resource that already has containers in the wild must keep its existing
    --- ids or every stored item is orphaned: items are keyed by container id in
    --- the inventory tables, so renaming the container hides its contents. Such
    --- a resource calls Core.Storage.UseRawIds(true) once at startup and
    --- everything below passes ids through untouched.
    ---
    --- 0.13.0: the prefix carries the caller's poggy_id, not its folder name, so a
    --- server owner renaming the folder does not hide what is stored. A script
    --- with no poggy_id line, or whose id equals its folder, keeps exactly the
    --- ids it had. Raw ids are untouched. Poggy() calls reach storage through
    --- poggy_core's own Core (PoggyCore.Self()), whose id is poggy_core.
    local storageRaw = false
    local storageTag = nil
    local function storageId(id, opts)
        if storageRaw or (opts and opts.raw) then return id end
        if not storageTag then
            local I = PoggyCore.Identity
            storageTag = (tostring(I and I.Id(tag) or tag):gsub("[^%w]", "_"))
        end
        local prefix = ("%s_%s_"):format(PoggyCoreConfig.StoragePrefix, storageTag)
        if id:sub(1, #prefix) == prefix then return id end
        return prefix .. id
    end
    local function setStorageRaw(v) storageRaw = v and true or false end

    local Core = {}

    Core.VERSION   = PoggyCore.VERSION
    Core.Err       = PoggyCore.Err
    Core.Currency  = PoggyCore.Currency
    Core.Caps      = PoggyCore.Caps

    -- --- lifecycle ----------------------------------------------------------

    function Core.IsReady() return State.ready end

    --- Framework id, or 'standalone'. Prefer Core.Has() for behaviour decisions.
    function Core.GetFramework() return State.framework end

    --- True when a real adapter is driving the detected framework.
    ---
    --- False means one of two things: no framework is present, or a framework
    --- was detected that poggy_core cannot drive yet. Either way every framework
    --- call will refuse, so a consumer that has its own legacy path should keep
    --- using it rather than switching over.
    function Core.HasAdapter()
        return State.ready and State.framework ~= "standalone" and not State.noAdapter
    end

    --- Run fn once the framework has resolved. Runs immediately if it already has.
    function Core.Ready(fn)
        if not PoggyCore.IsCallable(fn) then return end
        if State.ready then
            CreateThread(function() fn() end)
        else
            State.waiting[#State.waiting + 1] = fn
        end
    end

    --- Block until ready, or until timeout. Returns true when ready.
    function Core.WaitReady(timeoutMs)
        local deadline = GetGameTimer() + (timeoutMs or 30000)
        while not State.ready and GetGameTimer() < deadline do Wait(50) end
        return State.ready
    end

    --- Does the detected framework support this capability? See PoggyCore.Caps.
    function Core.Has(capability)
        if not State.ready then return false end
        return State.adapter.caps[capability] == true
    end

    --- Hard version gate for third-party consumers. Errors loudly at boot rather
    --- than failing mysteriously later.
    function Core.RequireVersion(min)
        local function parts(v)
            local a, b, c = v:match("^(%d+)%.(%d+)%.?(%d*)$")
            return tonumber(a) or 0, tonumber(b) or 0, tonumber(c) or 0
        end
        local ha, hb, hc = parts(PoggyCore.VERSION)
        local na, nb, nc = parts(tostring(min))
        local have = ha * 10000 + hb * 100 + hc
        local need = na * 10000 + nb * 100 + nc
        if have < need then
            error(("[poggy_core] %s needs poggy_core >= %s but this is %s.")
                :format(tag, min, PoggyCore.VERSION), 2)
        end
        return true
    end

    -- --- identity -----------------------------------------------------------

    ---@return PoggyChar|nil
    function Core.GetChar(src)
        local a, n = guardSrc(src)
        if not a then return nil end
        return a:getChar(n)
    end

    ---@return PoggyChar|nil
    function Core.GetCharByCharId(charId)
        local a = guard()
        if not a or charId == nil then return nil end
        return a:getCharByCharId(charId)
    end

    --- Canonical character id for a source, as a string.
    function Core.GetCharId(src)
        local char = Core.GetChar(src)
        return char and char.charId or nil
    end

    function Core.GetPlayers()
        local a = guard()
        if not a then return {} end
        return a:getPlayers()
    end

    --- A character by id, online or not. Needs a thread.
    ---@return PoggyChar|nil
    function Core.GetCharOffline(charId, withAppearance)
        local a, err = guard()
        if not a then return nil, err end
        if charId == nil then return nil, Err.BAD_ARG end
        return a:charOffline(charId, withAppearance and true or false)
    end

    --- Every character, online or not. opts: search, limit, offset. Needs a thread.
    function Core.ListChars(opts)
        local a, err = guard()
        if not a then return nil, err end
        return a:charList(type(opts) == "table" and opts or {})
    end

    --- Server ids of players on duty, optionally with a job (name or array) and
    --- at least minGrade. Unknown duty counts as off duty.
    function Core.GetPlayersOnDuty(job, minGrade)
        local a, err = guard()
        if not a then return nil, err end
        local list = job ~= nil and (type(job) == "table" and job or { job }) or nil
        minGrade = tonumber(minGrade)
        local out = {}
        for _, src in ipairs(a:getPlayers()) do
            local char = a:getChar(src)
            if char and char.onDuty == true then
                local match = true
                if list then
                    match = PoggyCore.InList(char.job, list)
                        and (not minGrade or (tonumber(char.jobGrade) or 0) >= minGrade)
                end
                if match then out[#out + 1] = src end
            end
        end
        return out
    end

    -- --- money --------------------------------------------------------------

    Core.Money = {}

    function Core.Money.Supports(currency)
        return Core.Has("money." .. tostring(currency))
    end

    function Core.Money.Get(src, currency)
        local a, n, err = guardSrc(src)
        if not a then return nil, err end
        currency = currency or PoggyCore.Currency.CASH
        if not Core.Money.Supports(currency) then
            Util.WarnOnce(tag, "money." .. currency,
                "asked for '%s' but %s has no such currency.", currency, State.framework)
            return nil, Err.UNSUPPORTED
        end
        return a:moneyGet(n, currency)
    end

    local function moneyMutate(op, src, currency, amount, reason)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        currency = currency or PoggyCore.Currency.CASH
        local amt, aerr = Util.Amount(amount)
        if not amt then return false, aerr end
        if not Core.Money.Supports(currency) then
            Util.WarnOnce(tag, "money." .. currency,
                "tried to %s '%s' but %s has no such currency.", op, currency, State.framework)
            return false, Err.UNSUPPORTED
        end
        Util.Debug("[%s] money.%s %s %s (%s)", tag, op, currency, amt, reason or "-")
        if op == "add"    then return a:moneyAdd(n, currency, amt, reason) end
        if op == "remove" then return a:moneyRemove(n, currency, amt, reason) end
        return a:moneySet(n, currency, amt, reason)
    end

    function Core.Money.Add(src, currency, amount, reason)
        return moneyMutate("add", src, currency, amount, reason)
    end

    function Core.Money.Remove(src, currency, amount, reason)
        return moneyMutate("remove", src, currency, amount, reason)
    end

    function Core.Money.Set(src, currency, amount, reason)
        return moneyMutate("set", src, currency, amount, reason)
    end

    -- --- jobs ---------------------------------------------------------------

    Core.Job = {}

    --- @return string name, number grade, string label, string|nil gradeLabel, boolean|nil onDuty
    function Core.Job.Get(src)
        local char = Core.GetChar(src)
        if not char then return nil end
        return char.job, char.jobGrade, char.jobLabel, char.jobGradeLabel, char.onDuty
    end

    --- persist: also write the job to the framework's database. From a thread
    --- the write is awaited, and a failure returns false with the in-memory job
    --- already set. Off a thread it runs on its own and a failure is logged.
    function Core.Job.Set(src, name, grade, label, persist)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        if type(name) ~= "string" or name == "" then return false, Err.BAD_ARG end
        local ok, jerr = a:jobSet(n, name, grade, label)
        if not ok or not persist then return ok, jerr end

        if Util.CanYield() then
            local pok, perr = a:jobPersist(n, name, grade, label)
            if not pok then
                Util.WarnOnce(tag, "job.persist." .. tostring(perr),
                    "job.set persist failed (%s); the job is set in memory only.", tostring(perr))
                return false, perr or Err.FRAMEWORK_ERR
            end
            return true
        end

        CreateThread(function()
            local pok, perr = a:jobPersist(n, name, grade, label)
            if not pok then
                Util.Warn("[%s] job.set persist for player %s failed (%s); the job is set in memory only.",
                    tag, tostring(n), tostring(perr))
            end
        end)
        return true
    end

    function Core.Job.SetDuty(src, onDuty)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        if not Core.Has("job.duty") then
            Util.WarnOnce(tag, "job.duty", "%s cannot set duty state.", State.framework)
            return false, Err.UNSUPPORTED
        end
        return a:jobSetDuty(n, onDuty and true or false)
    end

    --- Case-insensitive. `want` is a job name or an array of them.
    function Core.Job.Has(src, want, minGrade)
        local char = Core.GetChar(src)
        if not char then return false end
        local list = type(want) == "table" and want or { want }
        if not PoggyCore.InList(char.job, list) then return false end
        if minGrade and char.jobGrade < minGrade then return false end
        return true
    end

    function Core.Job.IsLaw(src)
        local char = Core.GetChar(src)
        return char and PoggyCore.InList(char.job, PoggyCoreConfig.LawJobs) or false
    end

    function Core.Job.IsMedical(src)
        local char = Core.GetChar(src)
        return char and PoggyCore.InList(char.job, PoggyCoreConfig.MedicalJobs) or false
    end

    --- Normalised job-change subscription. Every framework signals this
    --- differently and two of them barely signal it at all; sv_events wires
    --- whatever exists into this one event.
    function Core.Job.OnChange(fn)
        if not PoggyCore.IsCallable(fn) then return end
        AddEventHandler("poggy_core:jobChanged", fn)
    end

    -- --- inventory ----------------------------------------------------------

    Core.Inventory = {}

    function Core.Inventory.CanCarry(src, item, qty)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        local name, amt = Util.Item(item, qty)
        if not name then return false, amt end
        return a:invCanCarry(n, name, amt)
    end

    --- Always capacity-checked first, on every framework without exception.
    --- RSG's own AddItem drops the item on the ground when it does not fit and
    --- returns false; QBR has no capacity check at all. Checking here is what
    --- makes `ok` mean the same thing everywhere.
    function Core.Inventory.Add(src, item, qty, meta)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        local name, amt = Util.Item(item, qty)
        if not name then return false, amt end

        local can, cerr = a:invCanCarry(n, name, amt)
        if not can then return false, cerr or Err.NO_SPACE end

        Util.Debug("[%s] inv.add %s x%d", tag, name, amt)
        return a:invAdd(n, name, amt, meta)
    end

    function Core.Inventory.Remove(src, item, qty, meta)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        local name, amt = Util.Item(item, qty)
        if not name then return false, amt end
        Util.Debug("[%s] inv.remove %s x%d", tag, name, amt)
        return a:invRemove(n, name, amt, meta)
    end

    function Core.Inventory.Count(src, item, meta)
        local a, n = guardSrc(src)
        if not a then return 0 end
        if type(item) ~= "string" then return 0 end
        return a:invCount(n, item, meta)
    end

    function Core.Inventory.Has(src, item, qty)
        return Core.Inventory.Count(src, item) >= (tonumber(qty) or 1)
    end

    ---@return PoggyItem[]
    function Core.Inventory.Get(src)
        local a, n = guardSrc(src)
        if not a then return {} end
        return a:invGet(n)
    end

    function Core.Inventory.SetMeta(src, itemId, meta, amount)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        return a:invSetMeta(n, itemId, meta, amount)
    end

    --- Run fn when a player uses `item`. Held in poggy_core's own registry
    --- (sv_usables.lua): re-registered after the inventory resource restarts,
    --- dropped when this resource stops. Do not watch for the inventory
    --- restarting yourself.
    function Core.Inventory.RegisterUsable(item, fn)
        -- `tag` is the calling resource; the registry keys ownership on it.
        return PoggyCore.Usables.Register(tag, item, fn)
    end

    --- The item registry. opts: search (name or label), limit, checkImages.
    ---@return table|nil items
    function Core.Inventory.Items(opts)
        local a, err = guard()
        if not a then return nil, err end
        opts = type(opts) == "table" and opts or {}
        local list, _, lerr = itemRegistry(a)
        if not list then return nil, lerr end

        local needle = (type(opts.search) == "string" and opts.search ~= "") and opts.search:lower() or nil
        local limit  = tonumber(opts.limit)
        local out = {}
        for _, it in ipairs(list) do
            if not needle
                or tostring(it.name):lower():find(needle, 1, true)
                or tostring(it.label or ""):lower():find(needle, 1, true) then
                out[#out + 1] = copyItem(a, it, opts.checkImages)
                if limit and #out >= limit then break end
            end
        end
        return out
    end

    --- One item from the registry, or nil, 'not_found'.
    function Core.Inventory.ItemInfo(item, checkImages)
        local a, err = guard()
        if not a then return nil, err end
        if type(item) ~= "string" or item == "" then return nil, Err.BAD_ARG end
        local _, byName, lerr = itemRegistry(a)
        if not byName then return nil, lerr end
        local it = byName[item]
        if not it then return nil, Err.NOT_FOUND end
        return copyItem(a, it, checkImages)
    end

    --- The URL prefix item icons live under.
    function Core.Inventory.ImageBase()
        local a, err = guard()
        if not a then return nil, err end
        local base = a:imageBase()
        if not base then return nil, Err.UNSUPPORTED end
        return base
    end

    --- Close the player's own inventory screen.
    function Core.Inventory.Close(src)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        return a:invClose(n)
    end

    -- --- weapons ------------------------------------------------------------
    -- Deliberately separate from Inventory: only VORP and RSG model weapons as
    -- first-class objects. RedEM and RPX treat them as ordinary items, so a
    -- merged API would be a lie on two of five frameworks.

    Core.Weapons = {}

    function Core.Weapons.Get(src)
        local a, n = guardSrc(src)
        if not a then return {} end
        return a:weaponsGet(n)
    end

    function Core.Weapons.Add(src, weapon, ammo, components)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        return a:weaponAdd(n, weapon, ammo, components)
    end

    function Core.Weapons.Remove(src, weaponId)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        return a:weaponRemove(n, weaponId)
    end

    --- Room for `qty` more weapons (optionally a named one)? false comes with 'no_space'.
    function Core.Weapons.CanCarry(src, qty, weapon)
        local a, n, err = guardSrc(src)
        if not a then return false, err end
        local amt = tonumber(qty or 1)
        if not amt or amt <= 0 or amt % 1 ~= 0 then return false, Err.BAD_ARG end
        return a:weaponCanCarry(n, amt, weapon)
    end

    -- --- storage ------------------------------------------------------------
    -- Implemented in sv_storage.lua, which owns the registry and the re-register
    -- bookkeeping. This is just the binding.

    Core.Storage = PoggyCore.BuildStorageApi(tag, storageId, guard, setStorageRaw)

    -- --- notifications ------------------------------------------------------

    local notify = setmetatable({}, {
        __call = function(_, src, text, kind, duration)
            return PoggyCore.Notify(src, text, kind, duration)
        end,
    })

    function notify.Rich(src, opts)
        return PoggyCore.NotifyRich(src, opts)
    end

    -- Style-preserving variants. Use these when migrating code that already
    -- picked a specific on-screen style; use the plain Core.Notify for new code,
    -- because it is the one that stays portable.
    function notify.Tip(src, text, duration)
        return PoggyCore.NotifyStyled(src, "tip", { text = text, duration = duration })
    end
    function notify.Right(src, text, duration)
        return PoggyCore.NotifyStyled(src, "right", { text = text, duration = duration })
    end
    function notify.Objective(src, text, duration)
        return PoggyCore.NotifyStyled(src, "objective", { text = text, duration = duration })
    end
    function notify.Top(src, title, subtitle, duration)
        return PoggyCore.NotifyStyled(src, "top",
            { title = title, subtitle = subtitle, duration = duration })
    end
    function notify.Advanced(src, text, dict, icon, color, duration)
        return PoggyCore.NotifyStyled(src, "advanced",
            { text = text, dict = dict, icon = icon, color = color, duration = duration })
    end

    Core.Notify = notify

    -- --- callbacks ----------------------------------------------------------
    -- Our own transport, not the framework's. See sv_callbacks.lua for why.

    Core.Callback = PoggyCore.BuildCallbackApi(tag)

    -- --- permissions --------------------------------------------------------

    Core.Perms = {}

    function Core.Perms.GetGroup(src)
        local a, n = guardSrc(src)
        if not a then return "user" end
        return a:permGroup(n)
    end

    function Core.Perms.IsAdmin(src)
        local group = tostring(Core.Perms.GetGroup(src)):lower()
        return group == "admin" or group == "superadmin" or group == "god"
            or group == "owner" or group == "headadmin" or group == "developer"
    end

    -- --- menus --------------------------------------------------------------
    -- Menus and inputs are client-side concepts and are not built yet; this is
    -- the deliberate Phase 0 scope line. The server side only exists so a call
    -- from the wrong side fails with a useful message rather than a nil index.

    Core.Menu = {
        Open = function()
            Util.WarnOnce(tag, "menu.server",
                "Core.Menu is client-side only, and is not implemented yet in any case.")
            return false, Err.NOT_IMPL
        end,
        Close = function() return false, Err.NOT_IMPL end,
    }

    Core.Input = function()
        Util.WarnOnce(tag, "input.server", "Core.Input is client-side only.")
        return nil, Err.NOT_IMPL
    end

    -- --- escape hatch -------------------------------------------------------

    --- The framework's own core object. Using this makes your script
    --- framework-specific; it exists so nobody is ever blocked.
    function Core.Native()
        local a = guard()
        if not a then return nil end
        return a:nativeCore()
    end

    return Core
end

--- Public entry point. Returns a Core bound to the calling resource.
exports("Get", function()
    local resource = GetInvokingResource() or "poggy_core"
    if not cores[resource] then
        cores[resource] = buildCore(resource)
        Util.Debug("built Core for %s", resource)
    end
    return cores[resource]
end)

--- Internal handle for our own files.
function PoggyCore.Self()
    if not cores["poggy_core"] then cores["poggy_core"] = buildCore("poggy_core") end
    return cores["poggy_core"]
end

-- ---------------------------------------------------------------------------
-- Normalised job-change signal
-- ---------------------------------------------------------------------------
-- VORP fires vorp:playerJobChange and vorp:playerJobGradeChange as local server
-- events. Other frameworks differ, and two of them barely signal at all; each
-- adapter's wiring goes here so consumers only ever see poggy_core:jobChanged.

AddEventHandler("vorp:playerJobChange", function(src, newJob, oldJob)
    if State.framework ~= "vorp" then return end
    local char = State.adapter:getChar(src)
    TriggerEvent("poggy_core:jobChanged", src, newJob, char and char.jobGrade or 0, oldJob, nil)
    -- 0.11.0: the client hears it too (cl_core re-broadcasts poggy_core:jobChangedLocal).
    TriggerClientEvent("poggy_core:jobChanged", src, newJob, char and char.jobGrade or 0, oldJob, nil)
end)

AddEventHandler("vorp:playerJobGradeChange", function(src, newGrade, oldGrade)
    if State.framework ~= "vorp" then return end
    local char = State.adapter:getChar(src)
    TriggerEvent("poggy_core:jobChanged", src, char and char.job or nil, newGrade, char and char.job or nil, oldGrade)
    TriggerClientEvent("poggy_core:jobChanged", src, char and char.job or nil, newGrade, char and char.job or nil, oldGrade)
end)

-- Character selection, normalised. VORP fires vorp:SelectedCharacter locally
-- with (source, character).
AddEventHandler("vorp:SelectedCharacter", function(src, character)
    if State.framework ~= "vorp" then return end
    -- Carry the character id with the event. Scripts that key anything on the
    -- character need it the moment they hear about the load, and making them
    -- ask for it straight afterwards is a guaranteed round trip.
    local charId = character and character.charIdentifier and tostring(character.charIdentifier) or nil
    TriggerEvent("poggy_core:charLoaded", src, charId)
    TriggerClientEvent("poggy_core:charLoaded", src, charId)
end)
