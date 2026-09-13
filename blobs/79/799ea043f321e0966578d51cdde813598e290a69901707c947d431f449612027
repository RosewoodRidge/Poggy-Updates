--[[
    poggy_core — standalone adapter.

    The fallback when no framework is present. Every call refuses honestly rather
    than pretending to succeed, which is the whole point: a script running on a
    server with no framework should find out at the call site, not three systems
    later when a player's money silently failed to move.

    This file is also the adapter interface specification. Anyone writing a new
    adapter can copy it and fill the functions in; the dispatcher in sv_core.lua
    calls exactly these names and nothing else.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Adapters = PoggyCore.Adapters or {}

local UNSUPPORTED = function() return false, PoggyCore.Err.UNSUPPORTED end

PoggyCore.Adapters.standalone = {
    id    = "standalone",
    label = "Standalone (no framework)",

    --- Capability map. Anything absent or false means Core.Has() returns false.
    caps = {},

    --- The resource that keeps usable-item handlers in memory, if the framework
    --- has one (VORP: "vorp_inventory"). sv_usables.lua re-registers every item
    --- when it starts. nil: nothing to watch.
    inventoryResource = nil,

    --- Called once the core object has been acquired (or not).
    ---@param core table|nil
    ---@return boolean ok
    init = function(self, core)
        self.core = core
        return true
    end,

    -- --- identity -----------------------------------------------------------
    getChar         = function() return nil end,
    getCharByCharId = function() return nil end,
    getPlayers      = function() return GetPlayers() end,

    -- --- money --------------------------------------------------------------
    moneyGet    = function() return nil, PoggyCore.Err.UNSUPPORTED end,
    moneyAdd    = UNSUPPORTED,
    moneyRemove = UNSUPPORTED,
    moneySet    = UNSUPPORTED,

    -- --- jobs ---------------------------------------------------------------
    jobSet     = UNSUPPORTED,
    jobSetDuty = UNSUPPORTED,

    -- --- inventory ----------------------------------------------------------
    invCanCarry       = function() return false, PoggyCore.Err.UNSUPPORTED end,
    invAdd            = UNSUPPORTED,
    invRemove         = UNSUPPORTED,
    invCount          = function() return 0 end,
    invGet            = function() return {} end,
    invSetMeta        = UNSUPPORTED,
    invRegisterUsable = UNSUPPORTED,                       -- (item, fn, resource)
    -- 0.13.1, both optional in an adapter: without invUnregisterUsable a
    -- stopped script's handler is only dropped from poggy_core's registry;
    -- without invReady the inventory is taken to be ready as soon as it starts.
    invUnregisterUsable = UNSUPPORTED,                     -- (item)
    invReady            = function() return false end,     -- () -> boolean

    -- --- weapons ------------------------------------------------------------
    weaponsGet = function() return {} end,
    weaponAdd  = UNSUPPORTED,
    weaponRemove = UNSUPPORTED,

    -- --- storage ------------------------------------------------------------
    stRegister     = UNSUPPORTED,
    stIsRegistered = function() return false end,
    stOpen         = UNSUPPORTED,
    stClose        = UNSUPPORTED,
    stAddItem      = UNSUPPORTED,
    stRemoveItem   = UNSUPPORTED,
    stGetItems     = function() return {}, PoggyCore.Err.UNSUPPORTED end,
    stSetCapacity  = UNSUPPORTED,
    stUnregister   = UNSUPPORTED,
    stDelete       = UNSUPPORTED,

    -- --- permissions --------------------------------------------------------
    permGroup = function() return "user" end,

    -- --- notifications ------------------------------------------------------
    --- Both return false so sv_notify falls through to the next renderer, which
    --- on a frameworkless server is the chat fallback.
    notify     = function() return false end,
    notifyRich = function() return false end,

    -- --- 0.11.0 -------------------------------------------------------------
    dutyOf          = function() return nil end,           -- true | false | nil (nobody can say)
    jobPersist      = UNSUPPORTED,                         -- (src, name, grade, label)
    charOffline     = function() return nil, PoggyCore.Err.UNSUPPORTED end,  -- (charId, withAppearance)
    charList        = function() return nil, PoggyCore.Err.UNSUPPORTED end,  -- (opts)
    itemRegistry    = function() return nil, PoggyCore.Err.UNSUPPORTED end,  -- () -> array of items
    imageBase       = function() return nil end,           -- () -> URL prefix
    itemImageExists = function() return false end,         -- (name)
    invClose        = UNSUPPORTED,                         -- (src)
    weaponCanCarry  = UNSUPPORTED,                         -- (src, qty, weapon)
    stGetWeapons    = function() return nil, PoggyCore.Err.UNSUPPORTED end,  -- (id)

    -- --- escape hatch -------------------------------------------------------
    nativeCore = function(self) return self.core end,
}
