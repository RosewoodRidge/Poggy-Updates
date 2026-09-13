--[[
    poggy_core — the menu and the text input, server side (0.14.0).

    The screens are drawn on the client (ui/, client/cl_menu.lua). A server
    script names a player, and the request rides poggy_core's own callback
    transport to that client, waits there for the answer, and comes back as
    the same ok, value, err triple the client-side verb returns:

        local ok, pick = Poggy('menu.open', { src = src, title = 'Stable', items = { ... } })

    Two things differ from an ordinary callback. A menu waits on a person, so
    the limit is PoggyCoreConfig.Ui.Timeout (five minutes by default), not
    RpcTimeout; on a timeout the client is told to take the page down and the
    caller gets false, 'timeout'. And a player who disconnects mid-menu answers
    'closed' at once rather than running out the clock (sv_callbacks.lua).

    Items travel to the client and back, so a value must be plain data: a
    string, number, boolean or a table of those. A function cannot cross.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Ui = {}

local Ui   = PoggyCore.Ui
local Util = PoggyCore.Util
local Err  = PoggyCore.Err

--- How long to wait for the player's answer (ms). Ui.Timeout when set and
--- positive, otherwise RpcTimeout, so an older config without the Ui block
--- still gets a limit rather than none.
local function waitLimit()
    local ui = PoggyCoreConfig.Ui
    local t = type(ui) == "table" and tonumber(ui.Timeout) or nil
    if not t or t <= 0 then t = PoggyCoreConfig.RpcTimeout end
    return t
end

--- A connected player, or nil plus why.
local function player(src)
    local n, why = Util.Source(src)
    if not n then return nil, why end
    if GetPlayerName(n) == nil then return nil, Err.NOT_FOUND end
    return n
end

--- Send one request to the client's ui and wait for its answer.
local function roundTrip(src, name, opts)
    local n, why = player(src)
    if not n then return false, nil, why end
    if not Util.CanYield() then return false, nil, Err.NEEDS_THREAD end

    local results, err = PoggyCore.AwaitClientPacked("poggy_core", n, name, waitLimit(), opts)
    if not results then
        if err == Err.TIMEOUT then
            -- Nobody is waiting for the answer any more: take the page down so
            -- the player is not left in a menu that leads nowhere.
            TriggerClientEvent("poggy_core:ui:close", n)
            return false, nil, Err.TIMEOUT
        end
        if err == "dropped" then return false, nil, "closed" end
        return false, nil, err
    end
    if (results.n or 0) == 0 then
        -- The client had no handler (a poggy_core older than 0.14.0 on that
        -- client) or the handler threw. Either way, nothing was shown.
        return false, nil, Err.FRAMEWORK_ERR
    end
    return results[1] and true or false, results[2], results[3]
end

-- ---------------------------------------------------------------------------
-- Public
-- ---------------------------------------------------------------------------

--- Open a list menu on a player's screen and wait for the answer.
---@param src any
---@param opts table { title, items = { { label, value?, desc?, right?, disabled? } }, subtitle?, cursor?, closeText? }
---@return boolean ok
---@return table|nil value { value, index, item } when ok
---@return string|nil err 'closed' | 'timeout' | 'bad_argument' | 'not_found' | 'needs_thread'
function Ui.Menu(src, opts)
    if type(opts) ~= "table" or type(opts.title) ~= "string" then return false, nil, Err.BAD_ARG end
    if type(opts.items) ~= "table" or #opts.items == 0 then return false, nil, Err.BAD_ARG end
    return roundTrip(src, "poggy_core:ui:menu", {
        title     = opts.title,
        items     = opts.items,
        subtitle  = opts.subtitle,
        cursor    = opts.cursor,
        closeText = opts.closeText,
    })
end

--- Open a text box on a player's screen and wait for the answer.
---@param src any
---@param opts table { title, placeholder?, default?, maxLength?, numeric?, submitText?, cursor? }
---@return boolean ok
---@return string|number|nil value
---@return string|nil err
function Ui.Input(src, opts)
    if type(opts) ~= "table" or type(opts.title) ~= "string" then return false, nil, Err.BAD_ARG end
    return roundTrip(src, "poggy_core:ui:input", {
        title       = opts.title,
        placeholder = opts.placeholder,
        default     = opts.default,
        maxLength   = opts.maxLength,
        numeric     = opts.numeric,
        submitText  = opts.submitText,
        cursor      = opts.cursor,
    })
end

--- Close whatever poggy_core has open on a player's screen. Fire-and-forget:
--- the waiting caller, wherever it is, gets false, 'closed'.
---@return boolean ok
---@return string|nil err
function Ui.Close(src)
    local n, why = player(src)
    if not n then return false, why end
    TriggerClientEvent("poggy_core:ui:close", n)
    return true, nil
end
