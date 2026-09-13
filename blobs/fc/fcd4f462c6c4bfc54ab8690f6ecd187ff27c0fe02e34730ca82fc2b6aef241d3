--[[
    poggy_core — storage (containers / stashes).

    The most divergent area across frameworks, so this file does real work rather
    than just forwarding:

    * VORP container definitions are memory-only. They vanish on restart, so
      every consumer must re-register at boot. We keep our own registry of what
      each resource asked for, and replay it automatically when the framework
      resolves or when vorp_inventory restarts.
    * RSG persists definitions but locks a stash to one viewer at a time.
    * QBR has no registration at all; a stash exists the moment it is opened.

    Registering on every boot is the only behaviour that is correct on all of
    them, so Register() is idempotent by contract and cheap to call repeatedly.

    Container ids (0.13.0). A non-raw id is prefixed with the calling script's
    poggy_id, falling back to its folder name (storageId in sv_core.lua), where
    it used to be the folder name. A server owner may rename a script's folder;
    items are keyed by container id in the inventory tables, so a prefix that
    followed the folder would hide everything stored. Raw ids (UseRawIds, or
    opts.raw) are passed through exactly as before. No live container id
    changed: poggy_character_storage and poggy_trashbins, the only scripts that
    register storage, both use raw ids, and Poggy() reaches storage through
    poggy_core's own Core, whose id is poggy_core.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

--- Everything registered this session: id -> { resource, opts }
local registry = {}
PoggyCore.StorageRegistry = registry

--- Replay every registration. Called when the framework resolves, and again if
--- the inventory resource restarts underneath us.
local function replay(reason)
    local a = PoggyCore.State and PoggyCore.State.adapter
    if not a or not PoggyCore.State.ready then return end
    local count = 0
    for id, entry in pairs(registry) do
        local ok = pcall(function() a:stRegister(id, entry.opts) end)
        if ok then count = count + 1 end
    end
    if count > 0 then
        Util.Log("Re-registered %d container(s) after %s.", count, reason)
    end
end

AddEventHandler("poggy_core:ready", function()
    if next(registry) then replay("framework resolution") end
end)

AddEventHandler("onResourceStart", function(resource)
    -- VORP keeps definitions in memory inside vorp_inventory, so a restart of
    -- that resource silently empties every container definition on the server.
    if resource == "vorp_inventory" or resource == "rsg-inventory" then
        CreateThread(function()
            Wait(2000)
            replay(resource .. " restart")
        end)
    end
end)

--- Build the Core.Storage table for one consuming resource.
---@param tag string the calling resource name
---@param storageId fun(id: string, opts: table|nil): string namespacing function
---@param guard fun(): table|nil, string|nil
---@param setRaw fun(enabled: boolean) turn off id namespacing for this resource
---@return table
function PoggyCore.BuildStorageApi(tag, storageId, guard, setRaw)
    local S = {}

    --- Stop namespacing this resource's container ids.
    ---
    --- Only for a resource that already has containers in the wild. Items are
    --- keyed by container id inside the inventory tables, so a resource that
    --- renames its containers hides everything already stored in them. Call this
    --- once, before registering anything.
    ---
    --- The trade-off is that you are back in the server-wide flat id namespace,
    --- so pick ids nobody else would: keep your resource name in them, and stay
    --- clear of the prefixes RSG reserves (police-, marshal-, gang-, admin-,
    --- evidence-).
    function S.UseRawIds(enabled)
        if setRaw then setRaw(enabled ~= false) end
        return true
    end

    --- Register a container. Idempotent: safe and cheap to call on every boot,
    --- and you must, because VORP forgets definitions when it restarts.
    ---@param id string
    ---@param opts PoggyStorageOpts|nil
    ---@return boolean ok
    ---@return string|nil err
    function S.Register(id, opts)
        local a, err = guard()
        if not a then return false, err end
        if type(id) ~= "string" or id == "" then return false, Err.BAD_ARG end
        if not a.caps["storage"] then
            Util.WarnOnce(tag, "storage", "%s has no container support.", PoggyCore.State.framework)
            return false, Err.UNSUPPORTED
        end

        local full   = storageId(id, opts)
        local merged = PoggyCore.WithDefaults(opts, PoggyCoreConfig.StorageDefaults)
        merged.label = merged.label or id

        local ok, aerr = a:stRegister(full, merged)
        if ok then
            registry[full] = { resource = tag, opts = merged }
            Util.Debug("[%s] storage.register %s", tag, full)
        end
        return ok, aerr
    end

    ---@return boolean
    function S.IsRegistered(id)
        local a = guard()
        if not a then return false end
        return a:stIsRegistered(storageId(id)) and true or false
    end

    --- Open a container for a player.
    function S.Open(src, id)
        local a, err = guard()
        if not a then return false, err end
        local n, serr = Util.Source(src)
        if not n then return false, serr end

        local full = storageId(id)
        -- Opening something nobody registered is the most common storage bug and
        -- produces an empty container with no explanation. Say so instead.
        if not registry[full] and not a:stIsRegistered(full) then
            Util.WarnOnce(tag, "storage.open." .. tostring(id),
                "opened container '%s' before registering it.", tostring(id))
        end
        return a:stOpen(n, full)
    end

    function S.Close(src, id)
        local a, err = guard()
        if not a then return false, err end
        local n, serr = Util.Source(src)
        if not n then return false, serr end
        return a:stClose(n, storageId(id))
    end

    --- Put an item into a container directly, without a player moving it.
    ---@param charId string|number|nil owner to attribute the item to, where the
    ---       framework tracks that (VORP does)
    function S.AddItem(id, item, qty, meta, charId)
        local a, err = guard()
        if not a then return false, err end
        local name, amt = Util.Item(item, qty)
        if not name then return false, amt end
        return a:stAddItem(storageId(id), name, amt, meta, charId)
    end

    function S.RemoveItem(id, item, qty, meta)
        local a, err = guard()
        if not a then return false, err end
        local name, amt = Util.Item(item, qty)
        if not name then return false, amt end
        return a:stRemoveItem(storageId(id), name, amt, meta)
    end

    ---@return PoggyItem[]
    function S.GetItems(id)
        local a = guard()
        if not a then return {} end
        return a:stGetItems(storageId(id))
    end

    function S.SetCapacity(id, slots, maxWeight)
        local a, err = guard()
        if not a then return false, err end
        local full = storageId(id)
        local ok, aerr = a:stSetCapacity(full, slots, maxWeight)
        if ok and registry[full] then
            if slots then registry[full].opts.slots = slots end
            if maxWeight then registry[full].opts.maxWeight = maxWeight end
        end
        return ok, aerr
    end

    --- Forget the container without touching its contents.
    ---
    --- Use this, not Delete, when you are tearing a container down at runtime
    --- and expect the items to still be there next time it is registered. On
    --- VORP the definition is memory-only anyway, so this is the reversible one.
    function S.Unregister(id)
        local a, err = guard()
        if not a then return false, err end
        local full = storageId(id)
        local ok, aerr = a:stUnregister(full)
        if ok then registry[full] = nil end
        return ok, aerr
    end

    --- Destroy the container and everything inside it. Not reversible.
    function S.Delete(id)
        local a, err = guard()
        if not a then return false, err end
        local full = storageId(id)
        local ok, aerr = a:stDelete(full)
        if ok then registry[full] = nil end
        return ok, aerr
    end

    --- Weapons inside a container (0.11.0). nil, 'not_found' when it is not registered.
    function S.GetWeapons(id)
        local a, err = guard()
        if not a then return nil, err end
        if type(id) ~= "string" or id == "" then return nil, Err.BAD_ARG end
        return a:stGetWeapons(storageId(id))
    end

    --- The namespaced id this resource's `id` resolves to. Useful for logging
    --- and for the rare case that needs to talk to the framework directly.
    function S.ResolveId(id)
        return storageId(id)
    end

    return S
end
