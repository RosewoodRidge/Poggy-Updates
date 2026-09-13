--[[
    poggy_core — the menu and the text input (0.14.0). Client side of ui/.

    Three verbs, drawn by poggy_core itself so no script needs vorp_menu or
    vorp_inputs and every framework gets the same screens:

        menu.open   a titled list; yields until the player picks or closes
        menu.close  take it down
        input.text  one text (or number) box; yields until submitted or cancelled

    One thing open at a time. Opening a menu or input while another is up
    replaces it, and the earlier caller gets false, 'closed' — never a hang,
    never two answers. NUI focus is held only while something is open and is
    released when it closes, when this resource stops, and when the script
    that opened it stops (its thread died with it, so nothing is resolved).

    The page keeps no values: it is sent labels only and answers with an index.
    The caller's own items table is kept here and returned as-is, so a value
    can be any Lua value, not only what survives JSON.

    Server-side calls (a `src` in the payload) arrive through poggy_core's own
    callback transport as the three client callbacks registered at the bottom;
    the server's sv_ui.lua is the other half.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Ui = {}

local Ui  = PoggyCore.Ui
local Err = PoggyCore.Err

local RESOURCE = GetCurrentResourceName()

local active = nil          -- { id, kind, owner, items, promise, done }
local nextId = 0

local function newId()
    nextId = nextId + 1
    return ("u%d_%d"):format(nextId, GetGameTimer())
end

local function canYield()
    local _, isMain = coroutine.running()
    return not isMain
end

local function cfg(key, default)
    local ui = PoggyCoreConfig.Ui
    if type(ui) ~= "table" or ui[key] == nil then return default end
    return ui[key]
end

--- Where the page puts the panel, read from the config on every open so an
--- edit to config.lua shows on the next menu. The page validates both again
--- and falls back to centre / 40 px, so a typo here cannot hide the menu.
local function placement(message)
    message.position = tostring(cfg("Position", "center"))
    message.margin   = tonumber(cfg("Margin", 40)) or 40
    return message
end

--- RDR2 menu sounds, the same ones vorp_menu plays. Every call is guarded:
--- a missing sound set must never take the menu down with it.
local SOUNDS = {
    move   = { "NAV_UP",     "HUD_PLAYER_MENU" },
    select = { "SELECT",     "RDRO_Character_Creator_Sounds" },
    close  = { "MENU_CLOSE", "HUD_PLAYER_MENU" },
}

local function playSound(kind)
    if cfg("Sounds", true) == false then return end
    local s = SOUNDS[kind]
    if not s then return end
    pcall(PlaySoundFrontend, s[1], s[2], true, 0)
end

local function releaseFocus()
    SetNuiFocus(false, false)
end

--- Take the page down without answering anyone. Used when the answer can no
--- longer be delivered (the caller's resource stopped).
local function hidePage()
    SendNUIMessage({ action = "close" })
    releaseFocus()
end

--- Resolve one request. Safe to call twice; only the first counts.
local function settle(entry, ok, value, err)
    if not entry or entry.done then return end
    entry.done = true
    if active == entry then
        active = nil
        hidePage()
    end
    entry.promise:resolve({ ok = ok, value = value, err = err })
end

--- Close whatever is open as 'closed', so a new request can take the screen.
local function replaceActive()
    if active then settle(active, false, nil, "closed") end
end

--- Open something and block until it is answered.
local function run(entry, message, cursor)
    replaceActive()
    active = entry
    SendNUIMessage(message)
    -- Keyboard focus always: the page handles the arrow keys, Enter and
    -- Escape. The mouse only when asked (default from config, normally true).
    SetNuiFocus(true, cursor and true or false)

    local result = Citizen.Await(entry.promise)
    return result.ok, result.value, result.err
end

-- ---------------------------------------------------------------------------
-- Public
-- ---------------------------------------------------------------------------

--- Sanitise the caller's items into what the page needs (labels only) and
--- what is kept here (the originals). Returns page items, or nil plus an error.
local function prepareItems(items)
    if type(items) ~= "table" or #items == 0 then return nil, Err.BAD_ARG end
    local out = {}
    for i = 1, #items do
        local it = items[i]
        if type(it) ~= "table" then return nil, Err.BAD_ARG end
        local label = it.label
        if label == nil then return nil, Err.BAD_ARG end
        out[i] = {
            label    = tostring(label),
            right    = it.right ~= nil and tostring(it.right) or nil,
            desc     = it.desc ~= nil and tostring(it.desc) or nil,
            disabled = it.disabled and true or false,
        }
    end
    return out
end

--- Open a list menu and wait for the answer.
---@param opts table { title, items = { { label, value?, desc?, right?, disabled? }, ... }, subtitle?, cursor?, closeText? }
---@param owner string|nil the resource that asked, for cleanup when it stops
---@return boolean ok
---@return table|nil value { value, index, item } when ok
---@return string|nil err 'closed' | 'bad_argument' | 'needs_thread'
function Ui.Menu(opts, owner)
    if type(opts) ~= "table" or type(opts.title) ~= "string" then return false, nil, Err.BAD_ARG end
    local pageItems, why = prepareItems(opts.items)
    if not pageItems then return false, nil, why end

    if not canYield() then
        print(("[poggy_core] menu.open ('%s') was called outside a thread. It waits for the "
            .. "player, so wrap the call in CreateThread(function() ... end)."):format(opts.title))
        return false, nil, Err.NEEDS_THREAD
    end

    local cursor = opts.cursor
    if cursor == nil then cursor = cfg("Cursor", true) end

    local entry = {
        id      = newId(),
        kind    = "menu",
        owner   = owner or RESOURCE,
        items   = opts.items,
        promise = promise.new(),
        done    = false,
    }

    return run(entry, placement({
        action    = "menu.open",
        id        = entry.id,
        title     = opts.title,
        subtitle  = opts.subtitle ~= nil and tostring(opts.subtitle) or nil,
        items     = pageItems,
        closeText = opts.closeText ~= nil and tostring(opts.closeText) or nil,
    }), cursor)
end

--- Open a text box and wait for the answer.
---@param opts table { title, placeholder?, default?, maxLength?, numeric?, submitText?, cursor? }
---@param owner string|nil
---@return boolean ok
---@return string|number|nil value the text, or a number when numeric
---@return string|nil err 'closed' | 'bad_argument' | 'needs_thread'
function Ui.Input(opts, owner)
    if type(opts) ~= "table" or type(opts.title) ~= "string" then return false, nil, Err.BAD_ARG end

    if not canYield() then
        print(("[poggy_core] input.text ('%s') was called outside a thread. It waits for the "
            .. "player, so wrap the call in CreateThread(function() ... end)."):format(opts.title))
        return false, nil, Err.NEEDS_THREAD
    end

    local cursor = opts.cursor
    if cursor == nil then cursor = cfg("Cursor", true) end

    local entry = {
        id      = newId(),
        kind    = "input",
        owner   = owner or RESOURCE,
        numeric = opts.numeric and true or false,
        promise = promise.new(),
        done    = false,
    }

    return run(entry, placement({
        action      = "input.open",
        id          = entry.id,
        title       = opts.title,
        placeholder = opts.placeholder ~= nil and tostring(opts.placeholder) or nil,
        default     = opts.default ~= nil and tostring(opts.default) or nil,
        maxLength   = tonumber(opts.maxLength),
        numeric     = entry.numeric,
        submitText  = opts.submitText ~= nil and tostring(opts.submitText) or nil,
        closeText   = opts.closeText ~= nil and tostring(opts.closeText) or nil,
    }), cursor)
end

--- Close whatever is open. The waiting caller gets false, 'closed'.
--- Returns true either way: "nothing was open" is not a failure.
function Ui.Close()
    replaceActive()
    return true
end

--- Is a menu or input open right now?
function Ui.IsOpen()
    return active ~= nil
end

-- ---------------------------------------------------------------------------
-- Answers from the page
-- ---------------------------------------------------------------------------
-- Every answer names the request it is for. One for a request that has
-- already been replaced or closed is dropped, which is what stops a slow
-- click on an old menu from answering a new one.

local function current(data)
    if not active or type(data) ~= "table" or data.id ~= active.id then return nil end
    return active
end

RegisterNUICallback("ui:select", function(data, cb)
    local entry = current(data)
    if entry and entry.kind == "menu" then
        local index = tonumber(data.index)
        local item  = index and entry.items[index] or nil
        if item and type(item) == "table" and not item.disabled then
            local value = item.value
            if value == nil then value = index end
            settle(entry, true, { value = value, index = index, item = item }, nil)
        end
        -- An index the page should not have sent: leave the menu up.
    end
    cb("ok")
end)

RegisterNUICallback("ui:submit", function(data, cb)
    local entry = current(data)
    if entry and entry.kind == "input" then
        local value = data.value
        if entry.numeric then
            value = tonumber(value)
            if value == nil then
                -- The page validates before sending; this is the backstop.
                settle(entry, false, nil, Err.BAD_ARG)
                cb("ok")
                return
            end
        elseif value == nil then
            value = ""
        else
            value = tostring(value)
        end
        settle(entry, true, value, nil)
    end
    cb("ok")
end)

RegisterNUICallback("ui:close", function(data, cb)
    local entry = current(data)
    if entry then settle(entry, false, nil, "closed") end
    cb("ok")
end)

RegisterNUICallback("ui:sound", function(data, cb)
    if type(data) == "table" then playSound(data.kind) end
    cb("ok")
end)

-- ---------------------------------------------------------------------------
-- The server's side: a script on the server named a player (sv_ui.lua)
-- ---------------------------------------------------------------------------
-- These run inside the callback transport's own thread, so they may wait.
-- Each returns the same ok, value, err triple the server hands back verbatim.

PoggyCore.CallbackRegister(RESOURCE, "poggy_core:ui:menu", function(opts)
    return Ui.Menu(opts, RESOURCE)
end)

PoggyCore.CallbackRegister(RESOURCE, "poggy_core:ui:input", function(opts)
    return Ui.Input(opts, RESOURCE)
end)

RegisterNetEvent("poggy_core:ui:close", function()
    Ui.Close()
end)

-- ---------------------------------------------------------------------------
-- Cleanup
-- ---------------------------------------------------------------------------

AddEventHandler("onResourceStop", function(resource)
    if resource == RESOURCE then
        -- Our page is going away with us: make sure the game gets its input back.
        if active then active.done = true; active = nil end
        releaseFocus()
        return
    end
    -- The script that asked has stopped, and so has the thread waiting for
    -- the answer. Take the page down; there is nobody left to answer.
    if active and active.owner == resource then
        active.done = true
        active = nil
        hidePage()
    end
end)
