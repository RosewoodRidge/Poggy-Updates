--[[
    poggy_core — usable items (0.13.1).

    Core.Inventory.RegisterUsable hands the framework a function to run when a
    player uses an item. On VORP that lands in vorp_inventory's USABLE_ITEMS
    table, which lives in memory inside vorp_inventory: the moment that resource
    restarts every handler is gone, and every item registered through poggy_core
    (fishing rods, the journal, the market deed, the armor kit...) stops working
    until the script that owns it restarts too. Each script used to watch
    onResourceStart for vorp_inventory on its own. This file does it once.

    What vorp_inventory 4.x actually does with a registration, read from
    server/services/inventoryApiService.lua (REGISTER_ITEM):

      * USABLE_ITEMS[name] = cb — one handler per item, the last registration
        wins, silently.
      * The resource name it is handed goes into REGISTERED_ITEMS for a debug
        print fifteen seconds after its start, and that table is then wiped. It
        is not used for anything else.
      * Nothing in vorp_inventory's server listens for onResourceStop, so a
        stopped script's handler stays in USABLE_ITEMS as a dead funcref until
        something replaces it. unRegisterUsableItem exists but nobody calls it.

    So poggy_core keeps its own registry, item -> { fn, resource, registeredAt }:

      * a script stopping drops its entries here AND unregisters them in the
        inventory, which is the clean-up VORP never does;
      * the inventory resource starting again (the adapter names it through
        `inventoryResource`) replays every entry once its exports answer, with
        a bounded wait, and prints one grey summary line;
      * a re-resolve (`poggycore resolve`) replays too, because the adapter
        takes a fresh inventory handle on that path.

    Register() is what Core.Inventory.RegisterUsable and the inv.registerUsable
    verb both call. The owner is always the calling resource — `tag` in
    sv_core.lua, the dispatcher's third argument in sv_dispatch.lua — never
    anything a payload could name.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

--- item -> { fn, resource, registeredAt }
local registry = {}
PoggyCore.UsableRegistry = registry

local Usables = {}
PoggyCore.Usables = Usables

--- After the inventory resource starts, how often to ask whether its exports
--- answer, and how many times before giving up (40 x 500ms = 20s). vorp_inventory
--- is usable well inside that; the bound exists so a resource that starts and
--- dies does not leave a thread polling forever.
Usables.READY_POLL_MS  = 500
Usables.READY_ATTEMPTS = 40

--- The ready adapter, or nil plus why not. PoggyCore.State is set by sv_core.lua,
--- which loads after this file, so it is read at call time.
local function adapter()
    local S = PoggyCore.State
    if not S or not S.ready or not S.adapter then return nil, Err.NOT_READY end
    return S.adapter
end

--- Hand one entry to the framework. Errors are caught: with the inventory
--- resource stopped the export proxy throws, and that must not take the caller
--- down — the entry stays in the registry for when the inventory is back.
local function push(a, item, entry)
    local ok, res, err = pcall(a.invRegisterUsable, a, item, entry.fn, entry.resource)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res == false then return false, err or Err.FRAMEWORK_ERR end
    return true
end

local function sortedItems()
    local out = {}
    for item in pairs(registry) do out[#out + 1] = item end
    table.sort(out)
    return out
end

--- Register `item` for `resource`. Returns ok, err from the framework call; the
--- entry is kept either way once the arguments are valid, so an item registered
--- while the inventory is down is registered for real when it comes up.
---
--- The same item again from the same resource replaces the entry quietly (a
--- script re-running its setup). From a different resource it replaces it too —
--- the inventory keeps one handler per item, so pretending otherwise would be a
--- lie — but says so once, in yellow, naming both scripts.
function Usables.Register(resource, item, fn)
    local a, err = adapter()
    if not a then return false, err end
    if type(item) ~= "string" or item == "" or not PoggyCore.IsCallable(fn) then
        return false, Err.BAD_ARG
    end
    if type(resource) ~= "string" or resource == "" then resource = "poggy_core" end

    local old = registry[item]
    if old and old.resource ~= resource then
        Util.Warn("usable item '%s' was registered by %s; %s has taken it over. " ..
            "The inventory keeps one handler per item, so the last registration wins.",
            item, old.resource, resource)
    end

    local entry = { fn = fn, resource = resource, registeredAt = os.time() }
    registry[item] = entry
    Util.Debug("[%s] inv.registerUsable %s", resource, item)
    return push(a, item, entry)
end

--- Drop every entry a resource owns (it stopped), and unregister each in the
--- inventory so no dead funcref is left behind. Returns how many were dropped.
function Usables.Forget(resource)
    local dropped = {}
    for item, entry in pairs(registry) do
        if entry.resource == resource then dropped[#dropped + 1] = item end
    end
    if #dropped == 0 then return 0 end
    table.sort(dropped)

    local a = adapter()
    for _, item in ipairs(dropped) do
        registry[item] = nil
        if a and a.invUnregisterUsable then
            -- Best effort: the inventory may be down at the same time.
            pcall(a.invUnregisterUsable, a, item)
        end
    end
    Util.Debug("dropped %d usable item(s) from %s: %s", #dropped, resource, table.concat(dropped, ", "))
    return #dropped
end

--- Re-register every entry. Returns how many the framework accepted, and how
--- many there were. One summary line, grey when everything went through.
function Usables.Replay(reason)
    local a = adapter()
    if not a then return 0, 0 end
    local items = sortedItems()
    if #items == 0 then return 0, 0 end

    local n = 0
    for _, item in ipairs(items) do
        if push(a, item, registry[item]) then n = n + 1 end
    end
    if n == #items then
        Util.Log("^9re-registered %d usable item(s) after %s^7", n, reason)
    else
        Util.Warn("re-registered %d of %d usable item(s) after %s; the framework refused the rest.",
            n, #items, reason)
    end
    return n, #items
end

--- Does the inventory answer its exports yet? An adapter without a probe is
--- taken at its word.
function Usables.InventoryReady(a)
    if not a.invReady then return true end
    local ok, ready = pcall(a.invReady, a)
    return ok and ready == true
end

--- The inventory resource started (again). Wait until its exports answer, then
--- replay. Returns true when a replay was scheduled, so the harness can tell a
--- restart of some other resource apart from one that matters.
function Usables.OnInventoryStart(resource)
    local a = adapter()
    if not a or not a.inventoryResource or a.inventoryResource ~= resource then return false end
    if next(registry) == nil then return false end

    CreateThread(function()
        local ready = false
        for _ = 1, Usables.READY_ATTEMPTS do
            if Usables.InventoryReady(a) then ready = true; break end
            Wait(Usables.READY_POLL_MS)
        end
        if not ready then
            local count = #sortedItems()
            Util.Warn("%s started but its exports did not answer within %ds, so %d usable item(s) " ..
                "were not re-registered. Once it is up, run `poggycore resolve`.",
                resource, math.floor(Usables.READY_ATTEMPTS * Usables.READY_POLL_MS / 1000), count)
            return
        end
        Usables.Replay(resource .. " restarted")
    end)
    return true
end

--- Every entry, sorted by owning resource then item, for /poggycore usables.
function Usables.List()
    local out = {}
    for item, entry in pairs(registry) do
        out[#out + 1] = { item = item, resource = entry.resource, registeredAt = entry.registeredAt }
    end
    table.sort(out, function(x, y)
        if x.resource ~= y.resource then return x.resource < y.resource end
        return x.item < y.item
    end)
    return out
end

--- How many, and how many per resource, for /poggycore status.
function Usables.Count()
    local n, byResource = 0, {}
    for _, entry in pairs(registry) do
        n = n + 1
        byResource[entry.resource] = (byResource[entry.resource] or 0) + 1
    end
    return n, byResource
end

AddEventHandler("onResourceStart", function(resource)
    Usables.OnInventoryStart(resource)
end)

AddEventHandler("onResourceStop", function(resource)
    Usables.Forget(resource)
end)

-- A re-resolve builds the adapter again, inventory handle included. The first
-- resolve finds an empty registry (nothing can register before ready), so this
-- only ever replays after `poggycore resolve` or a late-started framework.
AddEventHandler("poggy_core:ready", function()
    if next(registry) ~= nil then Usables.Replay("framework re-resolve") end
end)
