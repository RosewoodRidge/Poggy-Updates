--[[
    poggy_core — internal server utilities.

    Not part of the public API. The important piece here is Await(), which is how
    a callback-shaped framework API becomes a synchronous-looking call.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Util = {}

local Util = PoggyCore.Util

-- ---------------------------------------------------------------------------
-- Logging
-- ---------------------------------------------------------------------------

-- Every poggy_core console line starts with the same prefix. Colour codes are
-- ^0-^9 (1 red, 2 green, 3 yellow, 5 light blue, 7 default, 9 grey); every line
-- still reads correctly when a log viewer strips them.
PoggyCore.PREFIX = "^5[Poggy Core]^7 "

function Util.Log(fmt, ...)
    print((PoggyCore.PREFIX .. fmt):format(...))
end

function Util.Warn(fmt, ...)
    print((PoggyCore.PREFIX .. "^3⚠️  " .. fmt .. "^7"):format(...))
end

function Util.Error(fmt, ...)
    print((PoggyCore.PREFIX .. "^1❌ " .. fmt .. "^7"):format(...))
end

function Util.Debug(fmt, ...)
    if not PoggyCoreConfig.Debug then return end
    print((PoggyCore.PREFIX .. "^9debug " .. fmt .. "^7"):format(...))
end

-- ---------------------------------------------------------------------------
-- Start banner
-- ---------------------------------------------------------------------------

local BANNER_POGGY = {
    "██████╗  ██████╗  ██████╗  ██████╗ ██╗   ██╗",
    "██╔══██╗██╔═══██╗██╔════╝ ██╔════╝ ╚██╗ ██╔╝",
    "██████╔╝██║   ██║██║  ███╗██║  ███╗ ╚████╔╝ ",
    "██╔═══╝ ██║   ██║██║   ██║██║   ██║  ╚██╔╝  ",
    "██║     ╚██████╔╝╚██████╔╝╚██████╔╝   ██║   ",
    "╚═╝      ╚═════╝  ╚═════╝  ╚═════╝    ╚═╝   ",
}
local BANNER_CORE = {
    " ██████╗ ██████╗ ██████╗ ███████╗",
    "██╔════╝██╔═══██╗██╔══██╗██╔════╝",
    "██║     ██║   ██║██████╔╝█████╗  ",
    "██║     ██║   ██║██╔══██╗██╔══╝  ",
    "╚██████╗╚██████╔╝██║  ██║███████╗",
    " ╚═════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝",
}

--- Printed once, when the framework has resolved. lines: one or two info lines.
function Util.Banner(lines)
    for i = 1, #BANNER_POGGY do
        print("^3" .. BANNER_POGGY[i] .. "  ^5" .. BANNER_CORE[i] .. "^7")
    end
    for _, line in ipairs(lines or {}) do
        print(PoggyCore.PREFIX .. line)
    end
end

--- Warn once per (resource, key) pair, so an unsupported call in a loop does not
--- fill the console.
local warned = {}
function Util.WarnOnce(resource, key, fmt, ...)
    if not PoggyCoreConfig.WarnUnsupported then return end
    local id = (resource or "?") .. "|" .. key
    if warned[id] then return end
    warned[id] = true
    Util.Warn("[%s] " .. fmt, resource or "?", ...)
end

-- ---------------------------------------------------------------------------
-- Await
-- ---------------------------------------------------------------------------

--- Run a callback-style function and block until it calls back, or time out.
---
--- VORP's inventory API is entirely callback-shaped:
---     exports.vorp_inventory:getItemCount(src, cb, itemName)
--- while RSG and RPX return synchronously. Wrapping the callback form here is
--- what lets Core.Inventory.Count() read the same on every framework.
---
--- Two things this guards against:
---   * a callback that is never invoked (a dropped VORP call would otherwise
---     hang the calling coroutine forever)
---   * a callback invoked twice, which would resolve an already-resolved promise
---     and throw
---
---@generic T
---@param invoke fun(resolve: fun(...)) call the framework here, pass it `resolve`
---@param timeoutMs number|nil defaults to PoggyCoreConfig.CallbackTimeout
---@return boolean ok false on timeout
---@return any ... whatever the framework passed to the callback
function Util.Await(invoke, timeoutMs)
    -- Citizen.Await needs a coroutine to yield from. Being called off one is a
    -- programming error in the consumer, and the native message for it
    -- ("attempt to yield from outside a coroutine") says nothing useful about
    -- where it came from, so catch it here with something actionable.
    if not Util.CanYield() then
        Util.Error("A framework call that has to wait was made outside a thread. " ..
            "Wrap the calling code in CreateThread(function() ... end), or use an event handler.")
        return false
    end

    local p = promise.new()
    local settled = false

    local function settle(timedOut, ...)
        if settled then return end
        settled = true
        p:resolve({ timedOut = timedOut, args = table.pack(...) })
    end

    local ok, err = pcall(invoke, function(...) settle(false, ...) end)
    if not ok then
        Util.Error("Await: the framework call threw: %s", tostring(err))
        settle(true)
    end

    SetTimeout(timeoutMs or PoggyCoreConfig.CallbackTimeout, function()
        settle(true)
    end)

    local result = Citizen.Await(p)
    if result.timedOut then
        return false
    end
    return true, table.unpack(result.args, 1, result.args.n)
end

--- True when we are on a coroutine that can yield. Await() needs one.
---
--- Note for anyone copying this: in Lua 5.4 `coroutine.running()` always returns
--- a thread, plus a boolean saying whether it is the MAIN one. Testing the first
--- return for nil (the 5.1 idiom) is always false here and the guard never
--- fires. The second return is the one that matters.
---@return boolean
function Util.CanYield()
    local _, isMain = coroutine.running()
    return not isMain
end

-- ---------------------------------------------------------------------------
-- Argument validation
-- ---------------------------------------------------------------------------

--- Validate a player source. Returns the numeric source, or nil plus an error.
---@param src any
---@return number|nil
---@return string|nil
function Util.Source(src)
    local n = tonumber(src)
    if not n or n <= 0 then return nil, PoggyCore.Err.BAD_ARG end
    return n
end

--- Validate an item name plus a positive quantity.
---@param item any
---@param qty any
---@return string|nil
---@return number|string
function Util.Item(item, qty)
    if type(item) ~= "string" or item == "" then return nil, PoggyCore.Err.BAD_ARG end
    local n = tonumber(qty or 1)
    if not n or n <= 0 or n % 1 ~= 0 then return nil, PoggyCore.Err.BAD_ARG end
    return item, n
end

--- Validate a money amount. Rejects negatives and non-numbers; several
--- frameworks accept a negative amount and do something surprising with it.
---@param amount any
---@return number|nil
---@return string|nil
function Util.Amount(amount)
    local n = tonumber(amount)
    if not n or n < 0 then return nil, PoggyCore.Err.BAD_ARG end
    return n
end

-- ---------------------------------------------------------------------------
-- Database
-- ---------------------------------------------------------------------------
-- A few verbs read or write a framework's own tables: offline characters, the
-- item registry, a persisted job. They go through oxmysql's exports, not its
-- lib file, so poggy_core gains no hard dependency: without oxmysql those
-- verbs refuse with 'unsupported' and nothing else changes. Both wait for the
-- answer, so like every other framework wait they need a thread.

function Util.DbAvailable()
    return GetResourceState("oxmysql") == "started"
end

--- SELECT. Returns an array of rows, or nil plus an error code.
function Util.DbQuery(sql, params)
    if not Util.DbAvailable() then return nil, PoggyCore.Err.UNSUPPORTED end
    if not Util.CanYield() then return nil, PoggyCore.Err.NEEDS_THREAD end
    local ok, rows = Util.Await(function(resolve)
        exports.oxmysql:query(sql, params or {}, resolve)
    end)
    if not ok then return nil, PoggyCore.Err.TIMEOUT end
    if type(rows) ~= "table" then return nil, PoggyCore.Err.FRAMEWORK_ERR end
    return rows
end

--- UPDATE / INSERT / DELETE. Returns the affected row count, or nil plus an
--- error code. Zero is a success: MySQL counts only rows whose value changed.
function Util.DbExecute(sql, params)
    if not Util.DbAvailable() then return nil, PoggyCore.Err.UNSUPPORTED end
    if not Util.CanYield() then return nil, PoggyCore.Err.NEEDS_THREAD end
    local ok, affected = Util.Await(function(resolve)
        exports.oxmysql:update(sql, params or {}, resolve)
    end)
    if not ok then return nil, PoggyCore.Err.TIMEOUT end
    affected = tonumber(affected)
    if not affected then return nil, PoggyCore.Err.FRAMEWORK_ERR end
    return affected
end
