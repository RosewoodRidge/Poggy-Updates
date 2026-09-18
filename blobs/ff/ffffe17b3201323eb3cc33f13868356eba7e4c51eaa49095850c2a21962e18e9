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
-- Argument and result lists on the wire (0.18.0)
--
-- Lists travel as table.pack's { n = count, ... }. On RedM the msgpack encoder
-- can pack such a table as a plain array and drop `n`, and a missing count
-- reads as "nothing": answers arrived empty and arguments were lost (found 18
-- September 2026 through the settings hub). So a list is wrapped in a table
-- with only named keys, which always travels as a map, and the count is put
-- back on arrival. A bare list from an older build is still understood.
-- ---------------------------------------------------------------------------

local function wire(list)
    if type(list) ~= "table" then list = { n = 0 } end
    return { __pc = list.n or #list, __pv = list }
end

local function unwire(w)
    if type(w) ~= "table" then return nil end
    if w.__pc ~= nil then
        local list = type(w.__pv) == "table" and w.__pv or {}
        list.n = tonumber(w.__pc) or #list
        return list
    end
    if w.n == nil then w.n = #w end
    return w
end

-- Large payloads (0.18.0): anything packed larger than CHUNK_BYTES (a settings
-- hub save with many changes, the hub's search index, a script's config model)
-- travels as msgpack cut into pieces sent as ordinary events, and is joined
-- and unpacked on the other side. See the note in server/sv_callbacks.lua for
-- why these are not latent events.
local CHUNK_BYTES = 16 * 1024
local CHUNK_PAUSE = 25          -- ms between pieces

local lastPiece = {}            -- requestId -> GetGameTimer() of the last piece of its answer

local function sendRequest(requestId, name, args)
    args = wire(args)
    local ok, packed = pcall(msgpack.pack, args)
    if not ok or type(packed) ~= "string" or #packed <= CHUNK_BYTES then
        TriggerServerEvent("poggy_core:cb:request", requestId, name, args)
        return
    end
    local total = math.ceil(#packed / CHUNK_BYTES)
    for i = 1, total do
        TriggerServerEvent("poggy_core:cb:requestChunk", requestId, name, i, total,
            packed:sub((i - 1) * CHUNK_BYTES + 1, i * CHUNK_BYTES))
        if i < total then Wait(CHUNK_PAUSE) end
    end
end

-- ---------------------------------------------------------------------------
-- Server responses to our requests
-- ---------------------------------------------------------------------------

--- How the last answer arrived, for diagnostics (the hub reports it to the
--- server console when it cannot open, because F8 is not always readable).
PoggyCore.LastDelivery = nil

local function deliver(requestId, results, via, extra)
    local p = pending[requestId]
    if not p then return end
    results = unwire(results)
    pending[requestId] = nil
    lastPiece[requestId] = nil
    local info = { requestId = requestId, via = via or "response", kind = type(results) }
    if type(results) == "table" then
        info.n = results.n
        local keys = 0
        for _ in pairs(results) do keys = keys + 1 end
        info.keys = keys
        info.firstKind = type(results[1])
    end
    if type(extra) == "table" then for k, v in pairs(extra) do info[k] = v end end
    PoggyCore.LastDelivery = info
    if type(results) ~= "table" then
        print(("[poggy_core] ^3the answer to request %s arrived empty (%s).^7")
            :format(tostring(requestId), type(results)))
        results = nil
    end
    p:resolve(results or { n = 0 })
end

RegisterNetEvent("poggy_core:cb:response", function(requestId, results)
    deliver(requestId, results)
end)

-- A large answer arrives in pieces.
local incoming = {}   -- requestId -> { total, got, parts }

RegisterNetEvent("poggy_core:cb:chunk", function(requestId, i, total, part)
    if not pending[requestId] then incoming[requestId] = nil return end
    if type(part) ~= "string" or type(i) ~= "number" or type(total) ~= "number" then return end
    local c = incoming[requestId]
    if not c then
        c = { total = total, got = 0, parts = {} }
        incoming[requestId] = c
    end
    lastPiece[requestId] = GetGameTimer()
    if c.parts[i] then return end
    c.parts[i] = part
    c.got = c.got + 1
    if c.got < c.total then return end

    incoming[requestId] = nil
    local joined = table.concat(c.parts, "", 1, c.total)
    local unpackErr
    local ok, results = pcall(function() return msgpack.unpack(joined) end)
    if not ok then
        unpackErr = tostring(results)
        print(("[poggy_core] ^3a large answer could not be unpacked: %s^7"):format(unpackErr))
        results = nil
    end
    deliver(requestId, results, "pieces", { pieces = c.total, bytes = #joined, unpackErr = unpackErr })
end)

-- ---------------------------------------------------------------------------
-- Server calling us
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:cb:clientRequest", function(requestId, name, args)
    args = unwire(args) or { n = 0 }
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
            wire(table.pack(table.unpack(packed, 2, packed.n))))
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
    return PoggyCore.CallbackAwaitPackedFor(name, PoggyCoreConfig.RpcTimeout, ...)
end

--- The same, with its own time limit (ms) instead of RpcTimeout, for a call
--- known to take longer: the settings hub restarting a script waits up to ten
--- seconds on the server, which is RpcTimeout's whole budget.
function PoggyCore.CallbackAwaitPackedFor(name, timeoutMs, ...)
    if type(name) ~= "string" then return nil, PoggyCore.Err.BAD_ARG end
    timeoutMs = tonumber(timeoutMs) or PoggyCoreConfig.RpcTimeout

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

    sendRequest(requestId, name, table.pack(...))

    -- A large answer still arriving in pieces is not a timeout: keep waiting
    -- while pieces keep coming.
    local function expire()
        if not pending[requestId] then return end
        local last = lastPiece[requestId]
        if last and GetGameTimer() - last < 5000 then
            SetTimeout(1000, expire)
            return
        end
        pending[requestId] = nil
        lastPiece[requestId] = nil
        incoming[requestId] = nil
        print(("[poggy_core] callback '%s' timed out after %dms."):format(name, timeoutMs))
        p:resolve(false)
    end
    SetTimeout(timeoutMs, expire)

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
