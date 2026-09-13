--[[
    poggy_core — the public contract.

    This file is the specification. It defines the canonical vocabulary (currency
    names, capability strings, the Char shape) and the annotations that make the
    API autocomplete in an editor. It contains no framework knowledge and no
    behaviour; adapters implement against it, and consumers read it.
]]

PoggyCore = PoggyCore or {}

--- Read from fxmanifest.lua rather than written here, because the two had
--- already drifted: the manifest said one version and this said another, and
--- every consumer's version check reads this one. The manifest is what a
--- customer and the release pipeline see, so it is the authority.
local function readVersion()
    local me = GetCurrentResourceName()

    -- GetResourceMetadata serves a cached parse of the manifest, and a plain
    -- `restart` does not refresh it: this reported 0.5.0 through five releases
    -- while the manifest on disk said 0.6.3. Read the file instead and keep the
    -- metadata only as a fallback. A version string that lies is worse than no
    -- version at all, and the whole version-registry idea rests on this value.
    local manifest = LoadResourceFile(me, "fxmanifest.lua")
    if manifest then
        -- Two quote styles, and the line may or may not be the first one in the
        -- file. Alternating the Lua string delimiters keeps every pattern free
        -- of backslash escapes, which is how the previous attempt ended up with
        -- a literal newline inside a string and failed to parse at all.
        local v = manifest:match("\nversion%s+'([^']+)'")
               or manifest:match('\nversion%s+"([^"]+)"')
               or manifest:match("^version%s+'([^']+)'")
               or manifest:match('^version%s+"([^"]+)"')
        if v then return v end
    end

    return GetResourceMetadata(me, "version", 0) or "0.0.0"
end

PoggyCore.VERSION = readVersion()

--- Can this value be called?
---
--- CFX hands a function that crossed a resource boundary back as a *funcref*,
--- which is a callable table, not a raw function. `type()` on it returns
--- "table". So every `type(x) == "function"` check against something that came
--- from another resource is wrong, and silently so.
---
--- This is exactly what stopped VORP being detected: `getUser` is a function
--- inside vorp_core and a table by the time poggy_core sees it, so the
--- handshake refused a core that was working perfectly.
---
--- Applies in both directions — a callback handed to us by a consumer resource
--- arrives the same way.
function PoggyCore.IsCallable(v)
    local t = type(v)
    if t == "function" then return true end
    if t ~= "table" and t ~= "userdata" then return false end
    local ok, mt = pcall(getmetatable, v)
    return ok and type(mt) == "table" and mt.__call ~= nil
end

-- ---------------------------------------------------------------------------
-- Canonical currencies
-- ---------------------------------------------------------------------------
-- Frameworks disagree about money more than about anything else. VORP has no
-- bank at all and indexes currencies by number; RSG has eight named types and
-- re-derives balances from inventory items; RedEM and RPX have no gold. These
-- four names are the vocabulary every adapter translates into, and anything
-- outside the list is passed through untranslated when Core.Money.Supports()
-- says the framework knows it.

PoggyCore.Currency = {
    CASH = "cash",
    BANK = "bank",
    GOLD = "gold",
    ROL  = "rol",
}

-- ---------------------------------------------------------------------------
-- Capabilities
-- ---------------------------------------------------------------------------
-- Core.Has(capability) answers these. A consumer that wants to stay portable
-- checks a capability instead of checking Core.Framework, because the framework
-- name is not a promise about behaviour and the capability is.

PoggyCore.Caps = {
    -- money
    MONEY_CASH        = "money.cash",
    MONEY_BANK        = "money.bank",
    MONEY_GOLD        = "money.gold",
    MONEY_ROL         = "money.rol",
    -- character / jobs
    CHAR_ONDUTY       = "char.onduty",        -- can the framework answer "is this player on duty"
    JOB_REGISTRY      = "job.registry",       -- is there a shared jobs table with labels and grades
    JOB_DUTY          = "job.duty",           -- can duty be set
    JOB_EVENT         = "job.event",          -- does the framework emit a job-change signal
    -- inventory
    INV_ITEMS         = "inventory.items",
    INV_WEAPONS       = "inventory.weapons",  -- weapons as first-class objects, not items
    INV_METADATA      = "inventory.metadata",
    INV_CARRYCHECK    = "inventory.carrycheck", -- native capacity check, vs one we compute
    -- storage
    STORAGE           = "storage",
    STORAGE_PERSIST   = "storage.persist",    -- do container *definitions* survive a restart
    STORAGE_PERMS     = "storage.permissions",
    STORAGE_WEAPONS   = "storage.weapons",
    -- misc
    MENU_NATIVE       = "menu.native",        -- a framework menu object is reachable
    PERMISSIONS       = "permissions",
    -- 0.11.0
    CHAR_OFFLINE      = "char.offline",       -- characters readable while offline (char.offline, char.list)
    JOB_PERSIST       = "job.persist",        -- job.set can write the job to the database
    INV_REGISTRY      = "inventory.registry", -- the item registry is readable (inv.items, inv.itemInfo)
}

-- ---------------------------------------------------------------------------
-- Error codes
-- ---------------------------------------------------------------------------
-- Every mutating call returns `ok, err`. `err` is one of these, so callers can
-- branch on it rather than matching strings.

PoggyCore.Err = {
    NOT_READY     = "not_ready",      -- framework has not resolved yet
    NO_CHAR       = "no_char",        -- source has no loaded character
    UNSUPPORTED   = "unsupported",    -- this framework cannot do this
    NOT_IMPL      = "not_implemented",-- poggy_core has not built this yet
    BAD_ARG       = "bad_argument",
    NO_SPACE      = "no_space",       -- carry check failed
    NO_FUNDS      = "no_funds",
    NOT_FOUND     = "not_found",
    TIMEOUT       = "timeout",        -- framework callback never came back
    FRAMEWORK_ERR = "framework_error",
    NEEDS_THREAD  = "needs_thread",  -- this verb waits on the framework,
                                     -- so it must be called from a thread
    DUPLICATE_ID  = "duplicate_id",  -- core.register: another started folder
                                     -- already holds this poggy_id (0.13.0)
    CORE_TOO_OLD  = "core_too_old",  -- returned by the Poggy() bridge, not the
                                     -- core: the script's poggy_core_min is
                                     -- newer than the poggy_core running
}

-- ---------------------------------------------------------------------------
-- Shapes (annotations only)
-- ---------------------------------------------------------------------------

---@class PoggyChar
---@field charId        string   canonical character id, always a string
---@field ownerId       string   account-level id (steam on VORP/RedEM, license elsewhere)
---@field source        number
---@field firstName     string
---@field lastName      string
---@field fullName      string
---@field job           string
---@field jobLabel      string
---@field jobGrade      number
---@field jobGradeLabel string|nil
---@field onDuty        boolean|nil  nil means "this framework cannot tell you"
---@field group         string
---@field gender        string   'm' | 'f' | 'unknown'
---@field money         table    { cash=, bank=, gold=, rol= } — only supported keys present
---@field native        table    the framework's own character object, escape hatch

---@class PoggyStorageOpts
---@field label          string|nil
---@field slots          number|nil
---@field maxWeight      number|nil
---@field shared         boolean|nil
---@field allowWeapons   boolean|nil
---@field whitelistItems boolean|nil
---@field jobAccess      table|nil   { [jobName] = minGrade }
---@field charAccess     table|nil   { charId, ... }
---@field raw            boolean|nil do not apply the id prefix (you are on your own)

---@class PoggyItem
---@field name   string
---@field amount number
---@field meta   table|nil
---@field label  string|nil
---@field weight number|nil

-- ---------------------------------------------------------------------------
-- Helpers shared by both sides
-- ---------------------------------------------------------------------------

--- Case-insensitive membership test against a config list.
---@param value string|nil
---@param list table
---@return boolean
function PoggyCore.InList(value, list)
    if type(value) ~= "string" then return false end
    local needle = value:lower()
    for i = 1, #list do
        if tostring(list[i]):lower() == needle then return true end
    end
    return false
end

--- Normalise anything a framework calls a gender into 'm' | 'f' | 'unknown'.
--- VORP stores a string, RSG and QBR store a number, RPX stores 1/0.
---@param value any
---@return string
function PoggyCore.NormaliseGender(value)
    if value == nil then return "unknown" end
    if type(value) == "number" then
        if value == 0 then return "m" end
        if value == 1 then return "f" end
        return "unknown"
    end
    local s = tostring(value):lower()
    if s == "m" or s == "male" or s == "0" then return "m" end
    if s == "f" or s == "female" or s == "1" then return "f" end
    return "unknown"
end

--- Shallow merge of `defaults` under `opts`, without mutating either.
---@param opts table|nil
---@param defaults table
---@return table
function PoggyCore.WithDefaults(opts, defaults)
    local out = {}
    for k, v in pairs(defaults) do out[k] = v end
    for k, v in pairs(opts or {}) do out[k] = v end
    return out
end
