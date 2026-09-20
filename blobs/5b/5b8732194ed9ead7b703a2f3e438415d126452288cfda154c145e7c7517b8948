--[[
    poggy_core — the settings hub, client side (0.18.0).

    /poggy opens a full-screen page (ui/hub/) where the owner edits every Poggy
    script's config files. This file is only the bridge: it holds no state of
    its own beyond "is the page open", and trusts nothing the page says. Every
    real decision (who may edit, locks, saving) is made on the server by
    server/sv_hub.lua, which re-checks each call.

        page  -> POST https://poggy_core/hub { call, args }
              -> server callback poggy_core:hub:<call>(src, ...args)
              <- { ok = true, value } | { ok = false, err, ...extra }

        server -> net event poggy_core:hub:event (type, data)
              -> page message { action = 'hub:event', type, data }

    Local page callbacks: hubClose (focus back to the game, locks released on
    the server) and hubPosition (the player's ped, for "Use my position").
    Contract: docs/reference/poggy-hub-spec.md §6.
]]

PoggyCore = PoggyCore or {}

local RESOURCE = GetCurrentResourceName()

local isOpen  = false   -- the page is showing and holds focus
local opening = false   -- an 'open' call is in flight; a second /poggy waits for it

local function hubCfg(key, default)
    local hub = PoggyCoreConfig and PoggyCoreConfig.Hub
    if type(hub) ~= "table" or hub[key] == nil then return default end
    return hub[key]
end

local function notify(text, kind)
    if PoggyCore.RenderNotify then
        PoggyCore.RenderNotify(text, kind)
    else
        TriggerEvent("chat:addMessage", { args = { "Poggy Hub", text } })
    end
end

-- ---------------------------------------------------------------------------
-- Server calls
-- ---------------------------------------------------------------------------

--- Shape whatever the server callback returned into the one answer the page
--- understands. Two forms are accepted, so this file does not have to change
--- if sv_hub.lua settles on the other one:
---   a single table     { ok = bool, value = ..., err = ..., files = ... }
---   poggy_core's triple ok, value, err (+ an optional 4th table of extra
---                      fields, or err itself a table { err = 'stale', files = ... })
--- Extra fields on a refusal (files for 'stale', path and reason for
--- 'invalid', holder for 'locked') are passed through untouched.
local function shapeAnswer(results)
    if not results or (results.n or 0) == 0 then
        return { ok = false, err = "no_answer" }
    end

    local first = results[1]

    if type(first) == "table" and type(first.ok) == "boolean" then
        return first
    end

    if type(first) ~= "boolean" then
        -- A bare value with no status: the callback answered, so count it as ok.
        return { ok = true, value = first }
    end

    if first then
        return { ok = true, value = results[2] }
    end

    local answer = { ok = false }
    local err, extra = results[3], results[4]
    if err == nil and type(results[2]) == "string" then err = results[2] end
    if type(err) == "table" then
        extra = err
        err = err.err or err.code or err.reason
    end
    if type(extra) ~= "table" and type(results[2]) == "table" then extra = results[2] end
    if type(extra) == "table" then
        for k, v in pairs(extra) do
            if k ~= "ok" then answer[k] = v end
        end
    end
    answer.err = err ~= nil and tostring(err) or answer.err or "failed"
    return answer
end

--- Call poggy_core:hub:<call> and wait for it. Must run on a thread.
---@param call string
---@param args table|nil array of arguments
---@return table answer { ok, value?, err?, ... }
--- Calls that can restart scripts get longer than RpcTimeout: the server waits
--- up to ten seconds for each restart before it answers.
local SLOW_CALLS = { restart = true, start = true, save = true, undo = true, saveRoles = true, itemList = true }
local SLOW_TIMEOUT = 30000

local function callServer(call, args)
    args = type(args) == "table" and args or {}
    local n = args.n or #args
    local limit = SLOW_CALLS[call] and math.max(SLOW_TIMEOUT, PoggyCoreConfig.RpcTimeout or 0) or PoggyCoreConfig.RpcTimeout
    local results, why = PoggyCore.CallbackAwaitPackedFor("poggy_core:hub:" .. call, limit, table.unpack(args, 1, n))
    if not results then
        return { ok = false, err = why or "timeout" }
    end
    return shapeAnswer(results)
end

-- ---------------------------------------------------------------------------
-- Opening and closing
-- ---------------------------------------------------------------------------

local commandName = tostring(hubCfg("Command", "poggy")):gsub("^/", "")

local OPEN_ERRORS = {
    denied         = "You do not have permission to open the Poggy Hub.",
    forbidden      = "You do not have permission to open the Poggy Hub.",
    no_permission  = "You do not have permission to open the Poggy Hub.",
    not_allowed    = "You do not have permission to open the Poggy Hub.",
    permission    = "You do not have permission to open the Poggy Hub.",
    timeout        = "The server did not answer. Try /%s again in a moment.",
    no_answer      = "The Poggy Hub is not available on this server yet.",
}

local function openHub()
    if opening then return end
    if isOpen then
        -- The command typed again (from F8) closes the hub. The page decides:
        -- with unsaved changes it asks first.
        SendNUIMessage({ action = "hub:requestClose" })
        return
    end

    opening = true
    CreateThread(function()
        local answer = callServer("open", {})
        opening = false

        if not answer.ok then
            print(("[poggy_core] /%s could not open the hub: err=%s message=%s")
                :format(commandName, tostring(answer.err), tostring(answer.message)))
            -- F8 is not always readable; put the details in the server console too.
            TriggerServerEvent("poggy_core:hub:diag", {
                err = answer.err, message = answer.message,
                delivery = PoggyCore.LastDelivery,
                msgpack = type(msgpack) == "table" and type(msgpack.unpack) or type(msgpack),
            })
            -- Our own wording for the known refusals; otherwise the server's message.
            local text = OPEN_ERRORS[answer.err]
                or (type(answer.message) == "string" and answer.message ~= "" and answer.message)
                or ("The Poggy Hub could not open (%s)."):format(tostring(answer.err))
            if text:find("%s", 1, true) then text = text:format(commandName) end
            notify(text, "error")
            return
        end

        isOpen = true
        SendNUIMessage({ action = "hub:open", data = answer.value })
        SetNuiFocus(true, true)
    end)
end

--- Give the game its input back. `tellServer` releases every lock this player
--- holds; it is false only when poggy_core itself is stopping.
local function closeHub(tellServer)
    local wasOpen = isOpen
    isOpen = false
    SetNuiFocus(false, false)
    if tellServer and wasOpen then
        CreateThread(function() callServer("closed", {}) end)
    end
end

RegisterCommand(commandName, function()
    openHub()
end, false)

--- Another Poggy script takes the player to its own window (poggy_tickets'
--- Players menu, opened from a hub button). The page hides without asking and
--- the locks are released, exactly as when the player closes the hub.
exports("CloseHub", function()
    if not isOpen then return false end
    SendNUIMessage({ action = "hub:close" })
    closeHub(true)
    return true
end)

CreateThread(function()
    -- The chat resource may start after us; a short wait makes the suggestion stick.
    Wait(2000)
    TriggerEvent("chat:addSuggestion", "/" .. commandName,
        "Open the Poggy Hub: settings, lists, commands and help for every Poggy script")
end)

-- ---------------------------------------------------------------------------
-- Page callbacks
-- ---------------------------------------------------------------------------

RegisterNUICallback("hub", function(data, cb)
    if type(data) ~= "table" or type(data.call) ~= "string" or not data.call:match("^[%w_]+$") then
        cb({ ok = false, err = "bad_argument" })
        return
    end
    if not isOpen then
        -- A late click after the hub closed. Nothing may act on it.
        cb({ ok = false, err = "closed" })
        return
    end
    local call, args = data.call, data.args
    CreateThread(function()
        cb(callServer(call, args))
    end)
end)

RegisterNUICallback("hubClose", function(_, cb)
    closeHub(true)
    cb({ ok = true })
end)

local function round2(n)
    return math.floor(n * 100 + 0.5) / 100
end

RegisterNUICallback("hubPosition", function(_, cb)
    local ped = PlayerPedId()
    if not ped or ped == 0 or not DoesEntityExist(ped) then
        cb({ ok = false, err = "no_ped" })
        return
    end
    local c = GetEntityCoords(ped)
    cb({
        ok = true,
        value = { x = round2(c.x), y = round2(c.y), z = round2(c.z), heading = round2(GetEntityHeading(ped)) },
    })
end)

-- ---------------------------------------------------------------------------
-- Pushes from the server: lock, kicked, idleWarning, released, scriptState
-- ---------------------------------------------------------------------------

RegisterNetEvent("poggy_core:hub:event", function(kind, data)
    if type(kind) ~= "string" then return end
    -- The page keeps nothing while closed, so a push to a closed hub is dropped.
    if not isOpen then return end
    SendNUIMessage({ action = "hub:event", type = kind, data = data })
end)

-- ---------------------------------------------------------------------------
-- Cleanup
-- ---------------------------------------------------------------------------

AddEventHandler("onResourceStop", function(resource)
    if resource ~= RESOURCE then return end
    -- The page goes with us. Never leave the player without their input.
    if isOpen then closeHub(false) end
end)
