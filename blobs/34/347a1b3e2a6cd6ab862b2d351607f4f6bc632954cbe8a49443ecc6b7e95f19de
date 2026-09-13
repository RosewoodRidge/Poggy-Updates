--[[
    poggy_core — callbacks, server side.

    Deliberately our own transport rather than the framework's.

    Both RSG and QBR store pending callbacks keyed by NAME with no request id,
    and clear the entry after the first response. Two concurrent calls to the
    same callback name therefore lose one of the two responses — a real bug that
    shows up as a request that silently never returns under load. QBR
    additionally has no server-to-client callbacks at all.

    This implementation keys on a per-request id, times out rather than hanging
    forever, and works identically on every framework including standalone.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util

local handlers       = {}   -- name -> { fn = , resource = }
local clientPending  = {}   -- requestId -> { p = promise, src = number }   (server -> client)
local nextId         = 0

local function newId()
    nextId = nextId + 1
    return ("s%d_%d"):format(nextId, GetGameTimer())
end

-- ---------------------------------------------------------------------------
-- Client -> server
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:cb:request", function(requestId, name, args)
    local src = source

    if type(requestId) ~= "string" or type(name) ~= "string" then return end

    local entry = handlers[name]
    if not entry then
        -- Answer anyway. A client left waiting on a name that no longer exists
        -- is the failure mode this whole file is meant to remove.
        TriggerClientEvent("poggy_core:cb:response", src, requestId, { n = 0 })
        Util.Debug("unknown callback '%s' requested by %d", name, src)
        return
    end

    CreateThread(function()
        local packed = table.pack(pcall(entry.fn, src, table.unpack(args or {}, 1, (args and args.n) or 0)))
        local ok = packed[1]
        if not ok then
            Util.Error("callback '%s' (%s) threw: %s", name, entry.resource, tostring(packed[2]))
            TriggerClientEvent("poggy_core:cb:response", src, requestId, { n = 0 })
            return
        end
        -- Drop the pcall status, keep the rest.
        local results = table.pack(table.unpack(packed, 2, packed.n))
        TriggerClientEvent("poggy_core:cb:response", src, requestId, results)
    end)
end)

-- ---------------------------------------------------------------------------
-- Server -> client responses
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:cb:clientResponse", function(requestId, results)
    local entry = clientPending[requestId]
    -- Only the client that was asked may answer. Anyone else naming a live
    -- request id is ignored, so a player cannot answer another player's menu.
    if not entry or entry.src ~= source then return end
    clientPending[requestId] = nil
    entry.p:resolve(results or { n = 0 })
end)

--- A player who leaves takes every answer they owed with them. Resolve those
--- requests now rather than letting each one run out its timeout, and say why:
--- a menu waiting on a dropped player is 'closed', not 'timeout'.
AddEventHandler("playerDropped", function()
    local src = source
    for requestId, entry in pairs(clientPending) do
        if entry.src == src then
            clientPending[requestId] = nil
            entry.p:resolve({ n = 0, dropped = true })
        end
    end
end)

--- Ask a client and wait, with a caller-chosen timeout. Internal: AwaitClient
--- below is the public form, and sv_ui.lua uses this directly because a menu
--- waits on a person, not on code, and needs a longer limit than RpcTimeout.
--- Returns the packed results { n = count, ... }, or nil plus 'bad_argument',
--- 'needs_thread', 'timeout', or 'dropped' (the player left while we waited).
---@param tag string calling resource, for the log
---@param src any
---@param name string
---@param timeoutMs number|nil defaults to PoggyCoreConfig.RpcTimeout
function PoggyCore.AwaitClientPacked(tag, src, name, timeoutMs, ...)
    local n = tonumber(src)
    if not n or type(name) ~= "string" then return nil, PoggyCore.Err.BAD_ARG end
    if not Util.CanYield() then
        Util.Error("[%s] a client callback ('%s') was awaited outside a thread. "
            .. "Wrap the call in CreateThread(function() ... end).", tag or "?", name)
        return nil, PoggyCore.Err.NEEDS_THREAD
    end

    local requestId = newId()
    local p = promise.new()
    clientPending[requestId] = { p = p, src = n }

    TriggerClientEvent("poggy_core:cb:clientRequest", n, requestId, name, table.pack(...))

    SetTimeout(timeoutMs or PoggyCoreConfig.RpcTimeout, function()
        if clientPending[requestId] then
            clientPending[requestId] = nil
            Util.Debug("[%s] client callback '%s' to %d timed out", tag or "?", name, n)
            p:resolve(false)
        end
    end)

    local results = Citizen.Await(p)
    if results == false then return nil, PoggyCore.Err.TIMEOUT end
    if type(results) == "table" and results.dropped then return nil, "dropped" end
    return results
end

-- ---------------------------------------------------------------------------
-- Public API
-- ---------------------------------------------------------------------------

---@param tag string calling resource
---@return table
function PoggyCore.BuildCallbackApi(tag)
    local C = {}

    --- Register a callback the client can call.
    --- `fn(src, ...)` returns whatever should go back.
    function C.Register(name, fn)
        if type(name) ~= "string" or not PoggyCore.IsCallable(fn) then return false end
        if handlers[name] and handlers[name].resource ~= tag then
            Util.Warn("[%s] callback '%s' was already registered by %s. Overwriting.",
                tag, name, handlers[name].resource)
        end
        handlers[name] = { fn = fn, resource = tag }
        return true
    end

    function C.Unregister(name)
        if handlers[name] and handlers[name].resource == tag then
            handlers[name] = nil
            return true
        end
        return false
    end

    --- Call a callback registered on a client. Blocks until it answers or times
    --- out. QBR cannot do this at all natively; here it works everywhere.
    ---@return any ... whatever the client returned, or nil on timeout
    function C.AwaitClient(src, name, ...)
        local results = PoggyCore.AwaitClientPacked(tag, src, name, PoggyCoreConfig.RpcTimeout, ...)
        if not results then return nil end
        return table.unpack(results, 1, results.n or 0)
    end

    return C
end

--- Drop a stopping resource's callbacks, so a restart does not leave stale
--- handlers pointing at dead functions.
AddEventHandler("onResourceStop", function(resource)
    local dropped = 0
    for name, entry in pairs(handlers) do
        if entry.resource == resource then
            handlers[name] = nil
            dropped = dropped + 1
        end
    end
    if dropped > 0 then
        Util.Debug("dropped %d callback(s) from %s", dropped, resource)
    end
end)
