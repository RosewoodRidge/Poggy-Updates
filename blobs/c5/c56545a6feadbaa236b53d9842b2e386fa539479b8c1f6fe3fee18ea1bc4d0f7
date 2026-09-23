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

--- The framework table itself is per-game and comes from the game layer's
--- shared/sh_frameworks.lua, which the manifest loads FIRST. RedM has five
--- entries, FiveM has two, and the logic below is the same either way. An empty
--- table is a broken build rather than a standalone server, so say so once.
PoggyCore.Frameworks = PoggyCore.Frameworks or {}
if next(PoggyCore.Frameworks) == nil then
    print("[poggy_core] No framework table was loaded. shared/sh_frameworks.lua is missing from this build; every server will look standalone.")
end

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
