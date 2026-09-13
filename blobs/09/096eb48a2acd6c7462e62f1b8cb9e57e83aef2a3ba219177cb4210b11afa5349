--[[
    poggy_core — framework detection.

    Two deliberate differences from the detection in poggy_util:

    1. Order is configurable and VORP comes first. poggy_util checked RSG before
       VORP, so a server running both cores (which happens during a migration)
       silently resolved to the wrong one.

    2. A started resource is necessary but not sufficient. The resource has to be
       started AND its handshake has to work. poggy_util treated GetResourceState
       as proof, which is why a half-loaded core produced a framework type that
       every later call failed against.
]]

PoggyCore = PoggyCore or {}

--- Which resource name identifies each framework, and how to reach its core.
--- `probe` runs on the server only; the client trusts the server's answer, which
--- avoids five different client-side handshakes that can all race resource start.
PoggyCore.Frameworks = {
    vorp = {
        resource = "vorp_core",
        label    = "VORP Core",
        -- Where vorp_inventory keeps item icons. Shared, so inv.imageBase gives
        -- the same answer on both sides without a round trip.
        itemImageBase = "nui://vorp_inventory/html/img/items/",
        probe    = function()
            -- GetCore() only. vorp_core also exposes vorpAPI(), which is
            -- deprecated and, more importantly, returns an object with no
            -- getUser — so every later character lookup against it fails and
            -- silently falls back to a database read. poggy_util calls vorpAPI()
            -- first today; that is the bug this line exists to avoid.
            local ok, core = pcall(function()
                return exports.vorp_core:GetCore()
            end)
            if not ok or type(core) ~= "table" or core.getUser == nil then
                return nil
            end
            if PoggyCore.IsCallable(core.getUser) then return core end

            -- Funcref metatables are not always inspectable, so if the cheap
            -- shape check is inconclusive, settle it by actually calling
            -- something harmless. getUsers() is a plain read.
            if pcall(function() return core.getUsers() end) then return core end
            return nil
        end,
    },
    rsg = {
        resource = "rsg-core",
        label    = "RSG Core",
        probe    = function()
            local ok, core = pcall(function()
                return exports["rsg-core"]:GetCoreObject()
            end)
            if ok and type(core) == "table" and core.Functions then return core end
            return nil
        end,
    },
    qbr = {
        resource = "qbr-core",
        label    = "QBCore RedM",
        probe    = function()
            -- QBR has no core object at all; every API is a flat export on
            -- qbr-core. A successful GetPlayers call is the handshake.
            local ok = pcall(function()
                return exports["qbr-core"]:GetPlayers()
            end)
            if ok then return { flat = true } end
            return nil
        end,
    },
    redem = {
        resource = "redem_roleplay",
        label    = "RedEM:RP",
        probe    = function()
            local ok, core = pcall(function()
                return exports["redem_roleplay"]:RedEM()
            end)
            if ok and type(core) == "table" then return core end
            return nil
        end,
    },
    rpx = {
        resource = "rpx-core",
        label    = "RPX",
        probe    = function()
            local ok, core = pcall(function()
                return exports["rpx-core"]:GetObject()
            end)
            if ok and type(core) == "table" then return core end
            return nil
        end,
    },
}

--- Is a framework's core resource started?
---@param id string
---@return boolean
function PoggyCore.IsFrameworkStarted(id)
    local def = PoggyCore.Frameworks[id]
    if not def then return false end
    return GetResourceState(def.resource) == "started"
end

--- Detect the framework, server side. Returns the id and the core object.
--- Returns 'standalone', nil when nothing is present.
---@return string id
---@return table|nil core
function PoggyCore.DetectFramework()
    local forced = PoggyCoreConfig.ForceFramework
    if forced and forced ~= "standalone" then
        local def = PoggyCore.Frameworks[forced]
        if not def then
            print(("[poggy_core] ForceFramework is '%s', which is not a framework I know about."):format(tostring(forced)))
            return "standalone", nil
        end
        local core = def.probe()
        if not core then
            print(("[poggy_core] ForceFramework is '%s' but its handshake failed. Is %s started?")
                :format(forced, def.resource))
        end
        return forced, core
    end
    if forced == "standalone" then
        return "standalone", nil
    end

    for _, id in ipairs(PoggyCoreConfig.DetectionOrder) do
        if PoggyCore.IsFrameworkStarted(id) then
            local core = PoggyCore.Frameworks[id].probe()
            if core then
                return id, core
            end
            -- Started but not answering yet. Keep looking; the retry loop in
            -- sv_core will come back around.
        end
    end

    return "standalone", nil
end

--- Every framework whose core resource is started. More than one means the
--- server is mid-migration and the order in config decides.
---@return table
function PoggyCore.StartedFrameworks()
    local found = {}
    for id in pairs(PoggyCore.Frameworks) do
        if PoggyCore.IsFrameworkStarted(id) then found[#found + 1] = id end
    end
    table.sort(found)
    return found
end
