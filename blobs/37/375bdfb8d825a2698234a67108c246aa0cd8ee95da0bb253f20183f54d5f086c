--[[
    poggy_core — world prompts.

    Prompts are the one interaction area where the frameworks nearly agree: RSG,
    QBR and RPX all expose createPrompt with almost the same signature, and RPX
    copied QBR's outright. VORP and RedEM have no equivalent, so this file ships
    a small native implementation for them.

    A prompt is a key the player can hold near a point in the world. Registration
    is idempotent by name, so calling Register twice with the same name replaces
    rather than duplicating.
]]

PoggyCore = PoggyCore or {}

local prompts = {}   -- name -> entry
local groups  = {}   -- name -> prompt group handle (native path)

--- Map a friendly key name to its RedM control hash.
local KEYS = {
    E = 0x760A9C6F, G = 0x760A9C6F,   -- INPUT_CONTEXT
    F = 0xCEFD9220,                    -- INPUT_CONTEXT_SECONDARY
    ENTER = 0xC7B5340A,
    SPACE = 0xD9D0E1C0,
}

local function keyHash(key)
    if type(key) == "number" then return key end
    return KEYS[tostring(key):upper()] or KEYS.E
end

-- ---------------------------------------------------------------------------
-- Native implementation (VORP, RedEM, standalone)
-- ---------------------------------------------------------------------------

local nativeRunning = false

local function startNativeLoop()
    if nativeRunning then return end
    nativeRunning = true

    CreateThread(function()
        while nativeRunning do
            local wait = 500
            local ped  = PlayerPedId()
            local pos  = GetEntityCoords(ped)

            for _, entry in pairs(prompts) do
                if entry.coords then
                    local dist = #(pos - entry.coords)
                    if dist < (entry.distance or 2.0) then
                        wait = 0
                        entry.group = entry.group or GetRandomIntInRange(0, 0xffffff)
                        PromptSetActiveGroupThisFrame(entry.group, CreateVarString(10, "LITERAL_STRING", entry.label))
                        if entry.handle and PromptHasHoldModeCompleted(entry.handle) then
                            if PoggyCore.IsCallable(entry.onActivate) then
                                CreateThread(function() entry.onActivate() end)
                            end
                        end
                    end
                end
            end

            Wait(wait)
        end
    end)
end

local function registerNative(entry)
    local handle = PromptRegisterBegin()
    PromptSetControlAction(handle, keyHash(entry.key))
    PromptSetText(handle, CreateVarString(10, "LITERAL_STRING", entry.label))
    PromptSetEnabled(handle, true)
    PromptSetVisible(handle, true)
    PromptSetHoldMode(handle, true)
    entry.group = GetRandomIntInRange(0, 0xffffff)
    PromptSetGroup(handle, entry.group)
    PromptRegisterEnd(handle)
    entry.handle = handle
    startNativeLoop()
end

-- ---------------------------------------------------------------------------
-- Public
-- ---------------------------------------------------------------------------

---@param tag string calling resource
---@return table
function PoggyCore.BuildPromptApi(tag)
    local P = {}

    --- Register a world prompt.
    ---@param name string unique within this resource
    ---@param opts table { coords, key, label, onActivate, distance }
    ---@return boolean
    function P.Register(name, opts)
        if type(name) ~= "string" or type(opts) ~= "table" then return false end
        local id = tag .. ":" .. name

        -- Replace rather than duplicate.
        if prompts[id] then P.Remove(name) end

        local entry = {
            id         = id,
            resource   = tag,
            coords     = opts.coords,
            key        = opts.key or "E",
            label      = opts.label or "Interact",
            onActivate = opts.onActivate,
            distance   = opts.distance or 2.0,
        }

        local fw = PoggyCore.ClientState and PoggyCore.ClientState.framework

        -- Use the framework's own prompt system where there is one; they handle
        -- draw distance and grouping better than a generic loop can.
        local ok = false
        if fw == "rsg" then
            ok = pcall(function()
                exports["rsg-core"]:createPrompt(id, entry.coords, keyHash(entry.key), entry.label,
                    { func = entry.onActivate })
            end)
        elseif fw == "qbr" then
            ok = pcall(function()
                exports["qbr-core"]:createPrompt(id, entry.coords, keyHash(entry.key), entry.label,
                    { event = entry.onActivate })
            end)
        elseif fw == "rpx" then
            ok = pcall(function()
                exports["rpx-core"]:createPrompt(id, entry.coords, keyHash(entry.key), entry.label,
                    { func = entry.onActivate })
            end)
        end

        entry.delegated = ok
        prompts[id] = entry

        if not ok then
            registerNative(entry)
        end
        return true
    end

    function P.Remove(name)
        local id = tag .. ":" .. name
        local entry = prompts[id]
        if not entry then return false end

        if entry.delegated then
            local fw = PoggyCore.ClientState.framework
            pcall(function()
                if fw == "rsg" then exports["rsg-core"]:deletePrompt(id)
                elseif fw == "qbr" then exports["qbr-core"]:deletePrompt(id)
                elseif fw == "rpx" then exports["rpx-core"]:deletePrompt(id)
                end
            end)
        elseif entry.handle then
            PromptDelete(entry.handle)
        end

        prompts[id] = nil
        return true
    end

    --- Drop every prompt this resource registered.
    function P.Clear()
        for id, entry in pairs(prompts) do
            if entry.resource == tag then
                P.Remove(id:sub(#tag + 2))
            end
        end
        return true
    end

    return P
end

-- Clean up after a resource that stops, so its prompts do not linger.
AddEventHandler("onClientResourceStop", function(resource)
    for id, entry in pairs(prompts) do
        if entry.resource == resource then
            if entry.handle then pcall(PromptDelete, entry.handle) end
            prompts[id] = nil
        end
    end
end)
