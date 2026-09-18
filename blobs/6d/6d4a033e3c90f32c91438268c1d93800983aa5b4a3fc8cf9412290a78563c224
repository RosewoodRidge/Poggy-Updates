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

-- ---------------------------------------------------------------------------
-- Large payloads (0.18.0)
--
-- One net event carries everything in one reliable burst, and a big one (the
-- settings hub sends its search index and a script's whole config model, often
-- hundreds of KB) can stall or drop a connection. So anything packed larger
-- than CHUNK_BYTES is msgpack-packed into one string, cut into CHUNK_BYTES
-- pieces and sent as ordinary events a few milliseconds apart. The other side
-- joins the pieces and unpacks them into exactly the table that was packed.
--
-- Not latent events. They were tried first, and on RedM a large latent answer
-- reached the client empty, with no error on either side (18 September 2026).
-- ---------------------------------------------------------------------------

local CHUNK_BYTES  = 16 * 1024
local CHUNK_PAUSE  = 25                  -- ms between pieces, about 640 KB/s
local MAX_INCOMING = 8 * 1024 * 1024     -- the most one client request may carry
local INCOMING_TTL = 60000               -- a request whose pieces stop arriving is dropped after this

local function pack(value)
    local ok, packed = pcall(msgpack.pack, value)
    if ok and type(packed) == "string" then return packed end
    return nil, packed
end

--- Send `results` to one client as the answer to requestId. Runs on a thread:
--- a large answer waits between its pieces.
local function respond(src, requestId, results, name)
    results = wire(results)
    local packed, why = pack(results)
    if not packed then
        Util.Warn("the answer to '%s' for player %d could not be packed (%s); it was sent empty",
            tostring(name), src, tostring(why))
        TriggerClientEvent("poggy_core:cb:response", src, requestId, { n = 0 })
        return
    end
    if #packed <= CHUNK_BYTES then
        TriggerClientEvent("poggy_core:cb:response", src, requestId, results)
        return
    end
    local total = math.ceil(#packed / CHUNK_BYTES)
    Util.Debug("answer to '%s' for %d: %d bytes in %d pieces", tostring(name), src, #packed, total)
    for i = 1, total do
        if not GetPlayerName(src) then return end   -- they left mid-answer
        TriggerClientEvent("poggy_core:cb:chunk", src, requestId, i, total,
            packed:sub((i - 1) * CHUNK_BYTES + 1, i * CHUNK_BYTES))
        if i < total then Wait(CHUNK_PAUSE) end
    end
end

-- ---------------------------------------------------------------------------
-- Client -> server
-- ---------------------------------------------------------------------------

local function handleRequest(src, requestId, name, args)
    args = unwire(args) or { n = 0 }
    local entry = handlers[name]
    if not entry then
        -- Answer anyway. A client left waiting on a name that no longer exists
        -- is the failure mode this whole file is meant to remove.
        TriggerClientEvent("poggy_core:cb:response", src, requestId, { n = 0 })
        Util.Warn("player %d asked for callback '%s', which nothing on the server has registered", src, name)
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
        respond(src, requestId, results, name)
    end)
end

RegisterNetEvent("poggy_core:cb:request", function(requestId, name, args)
    local src = source
    if type(requestId) ~= "string" or type(name) ~= "string" then return end
    handleRequest(src, requestId, name, args)
end)

-- A large request arrives in pieces (see Large payloads above).
local incoming = {}   -- "<src>:<requestId>" -> { name, total, got, size, parts, started }

local function sweepIncoming()
    local now = GetGameTimer()
    for key, c in pairs(incoming) do
        if now - c.started > INCOMING_TTL then incoming[key] = nil end
    end
end

RegisterNetEvent("poggy_core:cb:requestChunk", function(requestId, name, i, total, part)
    local src = source
    if type(requestId) ~= "string" or type(name) ~= "string" or type(part) ~= "string" then return end
    i, total = math.tointeger(i), math.tointeger(total)
    if not i or not total or i < 1 or total < 1 or i > total or total * CHUNK_BYTES > MAX_INCOMING then return end

    local key = src .. ":" .. requestId
    local c = incoming[key]
    if not c then
        sweepIncoming()
        c = { name = name, total = total, got = 0, size = 0, parts = {}, started = GetGameTimer() }
        incoming[key] = c
    end
    if c.name ~= name or c.total ~= total or c.parts[i] then return end

    c.size = c.size + #part
    if c.size > MAX_INCOMING then
        incoming[key] = nil
        Util.Warn("player %d sent more than %d bytes for '%s'; dropped", src, MAX_INCOMING, name)
        return
    end
    c.parts[i] = part
    c.got = c.got + 1
    if c.got < c.total then return end

    incoming[key] = nil
    local ok, args = pcall(msgpack.unpack, table.concat(c.parts, "", 1, c.total))
    if not ok or type(args) ~= "table" then
        Util.Warn("player %d sent a request for '%s' that could not be unpacked", src, name)
        TriggerClientEvent("poggy_core:cb:response", src, requestId, { n = 0 })
        return
    end
    handleRequest(src, requestId, name, args)
end)

AddEventHandler("playerDropped", function()
    local prefix = tostring(source) .. ":"
    for key in pairs(incoming) do
        if key:sub(1, #prefix) == prefix then incoming[key] = nil end
    end
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
    entry.p:resolve(unwire(results) or { n = 0 })
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

    TriggerClientEvent("poggy_core:cb:clientRequest", n, requestId, name, wire(table.pack(...)))

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
