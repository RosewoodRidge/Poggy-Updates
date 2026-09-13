--[[
    poggy_core — callbacks, client side. Matched pair with server/sv_callbacks.lua.

    Every request carries its own id, so two in-flight calls to the same callback
    name cannot overwrite each other. That is the bug in both RSG's and QBR's
    callback systems, and it is the reason this is our own transport.
]]

PoggyCore = PoggyCore or {}

local pending  = {}   -- requestId -> promise
local handlers = {}   -- name -> { fn, resource }
local nextId   = 0

local function newId()
    nextId = nextId + 1
    return ("c%d_%d"):format(nextId, GetGameTimer())
end

-- ---------------------------------------------------------------------------
-- Server responses to our requests
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:cb:response", function(requestId, results)
    local p = pending[requestId]
    if not p then return end
    pending[requestId] = nil
    p:resolve(results or { n = 0 })
end)

-- ---------------------------------------------------------------------------
-- Server calling us
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:cb:clientRequest", function(requestId, name, args)
    local entry = handlers[name]
    if not entry then
        TriggerServerEvent("poggy_core:cb:clientResponse", requestId, { n = 0 })
        return
    end
    CreateThread(function()
        local packed = table.pack(pcall(entry.fn, table.unpack(args or {}, 1, (args and args.n) or 0)))
        if not packed[1] then
            print(("[poggy_core] client callback '%s' (%s) threw: %s")
                :format(name, entry.resource, tostring(packed[2])))
            TriggerServerEvent("poggy_core:cb:clientResponse", requestId, { n = 0 })
            return
        end
        TriggerServerEvent("poggy_core:cb:clientResponse", requestId,
            table.pack(table.unpack(packed, 2, packed.n)))
    end)
end)

-- ---------------------------------------------------------------------------
-- Public
-- ---------------------------------------------------------------------------

--- Call a server callback and block until it answers.
--- Returns the packed results { n = count, ... }, or nil plus 'bad_argument',
--- 'needs_thread' or 'timeout'. The callback.await verb is built on this, so a
--- timeout can be told apart from a callback that answered with nothing.
function PoggyCore.CallbackAwaitPacked(name, ...)
    if type(name) ~= "string" then return nil, PoggyCore.Err.BAD_ARG end

    -- Same coroutine requirement as the server side. In Lua 5.4
    -- coroutine.running() always returns a thread plus a "is this the main one"
    -- boolean; the boolean is what tells us whether we can yield.
    local _, isMain = coroutine.running()
    if isMain then
        print(("[poggy_core] Core.Callback.Await('%s') was called outside a thread. " ..
            "Wrap it in CreateThread(function() ... end), or use Core.Callback.Trigger."):format(name))
        return nil, PoggyCore.Err.NEEDS_THREAD
    end

    local requestId = newId()
    local p = promise.new()
    pending[requestId] = p

    TriggerServerEvent("poggy_core:cb:request", requestId, name, table.pack(...))

    SetTimeout(PoggyCoreConfig.RpcTimeout, function()
        if pending[requestId] then
            pending[requestId] = nil
            print(("[poggy_core] callback '%s' timed out after %dms."):format(name, PoggyCoreConfig.RpcTimeout))
            p:resolve(false)
        end
    end)

    local results = Citizen.Await(p)
    if results == false then return nil, PoggyCore.Err.TIMEOUT end
    return results
end

--- Call a server callback and block until it answers. Returns nothing on timeout.
function PoggyCore.CallbackAwait(name, ...)
    local results = PoggyCore.CallbackAwaitPacked(name, ...)
    if not results then return nil end
    return table.unpack(results, 1, results.n or 0)
end

--- Non-blocking form, for code that is not on a coroutine.
function PoggyCore.CallbackTrigger(name, cb, ...)
    local args = table.pack(...)
    CreateThread(function()
        local results = table.pack(PoggyCore.CallbackAwait(name, table.unpack(args, 1, args.n)))
        if PoggyCore.IsCallable(cb) then cb(table.unpack(results, 1, results.n)) end
    end)
end

--- Register a callback the server can invoke on this client.
function PoggyCore.CallbackRegister(resource, name, fn)
    if type(name) ~= "string" or not PoggyCore.IsCallable(fn) then return false end
    handlers[name] = { fn = fn, resource = resource }
    return true
end
