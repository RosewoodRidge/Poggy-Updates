--[[
    poggy_core — script identity (server).

    Every Poggy script declares its product id in its fxmanifest.lua:

        poggy_id 'poggy_scene'

    The id is the name the update feed, the release list and poggy_core know the
    script by. It never changes. The FOLDER is the server owner's choice and may
    be renamed, so nothing in poggy_core may assume the two are the same. A
    script without the line has the id of its folder, exactly as before 0.13.0.

    How poggy_core learns which folder holds an id:

      1. Registration (core.register). The bridge (template/poggy.lua) sends its
         poggy_id and version the first time PoggyReady() passes on the server,
         before sql.install. The folder comes from the dispatcher, the resource
         that actually made the call, never from the payload, so a script can
         only ever register itself. A restart replaces the entry; stopping the
         resource removes it.

      2. Manifest scan, only for an id nobody has registered: a stopped script, a
         script whose bridge has not reached PoggyReady() (no framework, still
         starting), or a console command naming a folder. poggy_id is read from
         the fxmanifest.lua FILE, because GetResourceMetadata keeps the old value
         until `refresh`. The scan is cached; Identity.Refresh() drops it, and
         every update run starts with that.

    Two folders claiming one id are never guessed between. A second registration
    while the first holder is started is refused with one red line naming both,
    and Locate() refuses that id until one of them stops.
]]

PoggyCore = PoggyCore or {}

local Identity = PoggyCore.Identity or {}
PoggyCore.Identity = Identity

local BRIDGE = "@poggy_core/template/poggy.lua"

local byId      = {}   -- id -> { id, folder, version, registeredAt }
local byFolder  = {}   -- folder -> id
local conflicts = {}   -- id -> { [folder] = true, ... } while two started folders claim it
local scanCache = nil  -- see Identity.Scan

local function log(msg)
    print((PoggyCore.PREFIX or "^5[Poggy Core]^7 ") .. msg)
end

local function started(folder)
    local ok, state = pcall(GetResourceState, folder)
    return ok and state == "started"
end

local function sortedKeys(set)
    local out = {}
    for k in pairs(set) do out[#out + 1] = k end
    table.sort(out)
    return out
end

--- The value of `key 'value'` (or "value", or key('value')) in manifest text.
function Identity.ManifestField(text, key)
    if type(text) ~= "string" then return nil end
    text = text:gsub("^\239\187\191", "")
    -- Alternating the Lua string delimiters keeps these free of escapes.
    for _, q in ipairs({ "'", '"' }) do
        local body = "%s*%(?%s*" .. q .. "([^" .. q .. "\n]+)" .. q
        local v = text:match("\n%s*" .. key .. body) or text:match("^%s*" .. key .. body)
        if v then
            v = v:match("^%s*(.-)%s*$")
            if v ~= "" then return v end
        end
    end
    return nil
end

--- The poggy_id a folder's fxmanifest.lua declares (file first, then the cached
--- metadata), or nil. Also returns the manifest text it read.
function Identity.DeclaredId(folder)
    local okText, text = pcall(LoadResourceFile, folder, "fxmanifest.lua")
    text = okText and text or nil
    local id = Identity.ManifestField(text, "poggy_id")
    if not id and GetResourceMetadata then
        local okMeta, meta = pcall(GetResourceMetadata, folder, "poggy_id", 0)
        if okMeta and type(meta) == "string" and meta:match("%S") then id = meta:match("^%s*(.-)%s*$") end
    end
    return id, text
end

--- A usable id: letters, digits, _ - and dots.
local function validId(id)
    return type(id) == "string" and #id <= 64 and id:match("^[%w_%-%.]+$") ~= nil
end

-- ---------------------------------------------------------------------------
-- Registration
-- ---------------------------------------------------------------------------

--- Register `folder` under `id`. Returns true, entry  or  false, reason
--- ("bad_argument", "id_mismatch" or "duplicate_id").
function Identity.Register(folder, id, version)
    if type(folder) ~= "string" or folder == "" then return false, "bad_argument" end

    -- The id a script may claim is the one its own manifest declares, or its
    -- folder name when it declares none. The bridge computes exactly that, so
    -- only a hand-made call can differ, and it is not believed.
    local declared = Identity.DeclaredId(folder)
    local expected = declared or folder
    if id == nil then id = expected end
    if not validId(id) then return false, "bad_argument" end
    if id ~= expected then
        log(("^3⚠️  %s asked to register as poggy_id '%s', but its fxmanifest.lua says '%s'. Refused.^7")
            :format(folder, id, expected))
        return false, "id_mismatch"
    end

    local current = byId[id]
    if current and current.folder ~= folder then
        if started(current.folder) then
            local set = conflicts[id] or {}
            conflicts[id] = set
            set[current.folder], set[folder] = true, true
            log(("^1❌ poggy_id '%s' is declared by two folders: %s and %s. %s was refused, and poggy_core will not"
                .. " update '%s' until one of them is removed or stopped.^7")
                :format(id, current.folder, folder, folder, id))
            return false, "duplicate_id"
        end
        -- The first holder stopped without poggy_core hearing about it.
        byFolder[current.folder] = nil
    end

    -- A folder that registered under another id before its restart.
    local previous = byFolder[folder]
    if previous and previous ~= id and byId[previous] and byId[previous].folder == folder then
        byId[previous] = nil
    end

    local entry = {
        id = id,
        folder = folder,
        version = version ~= nil and tostring(version) or nil,
        registeredAt = os.time(),
    }
    byId[id] = entry
    byFolder[folder] = id
    return true, entry
end

--- Drop everything known about a folder (it stopped).
function Identity.Forget(folder)
    local id = byFolder[folder]
    if id then
        byFolder[folder] = nil
        if byId[id] and byId[id].folder == folder then byId[id] = nil end
    end
    for cid, set in pairs(conflicts) do
        if set[folder] then conflicts[cid] = nil end
    end
end

--- The core.register verb. resource is the caller, from the dispatcher; any
--- folder in the payload is ignored.
function Identity.Verb(resource, payload)
    local Err = PoggyCore.Err or {}
    payload = type(payload) == "table" and payload or {}
    local ok, entry = Identity.Register(resource, payload.id, payload.version)
    if not ok then
        if entry == "duplicate_id" then return false, nil, Err.DUPLICATE_ID or "duplicate_id" end
        return false, nil, Err.BAD_ARG or "bad_argument"
    end
    return true, { id = entry.id, folder = entry.folder, version = entry.version }, nil
end

if AddEventHandler then
    AddEventHandler("onResourceStop", function(resource)
        Identity.Forget(resource)
    end)
end

-- ---------------------------------------------------------------------------
-- Manifest scan (fallback)
-- ---------------------------------------------------------------------------

--- { claims = { id = { folder, ... } }, folders = { folder = id }, available }.
--- Covers every resource that loads the bridge (plus poggy_core itself), and any
--- resource that declares poggy_id without loading it.
function Identity.Scan()
    if scanCache then return scanCache end
    local s = { claims = {}, folders = {}, available = false }
    if GetNumResources and GetResourceByFindIndex then
        s.available = true
        local self = GetCurrentResourceName and GetCurrentResourceName() or "poggy_core"
        local okN, n = pcall(GetNumResources)
        for i = 0, ((okN and tonumber(n)) or 0) - 1 do
            local okR, folder = pcall(GetResourceByFindIndex, i)
            if okR and type(folder) == "string" and folder ~= "" then
                local declared, text = Identity.DeclaredId(folder)
                local poggy = folder == self or (text ~= nil and text:find(BRIDGE, 1, true) ~= nil)
                if poggy or declared then
                    local id = declared or folder
                    s.folders[folder] = id
                    local list = s.claims[id] or {}
                    list[#list + 1] = folder
                    s.claims[id] = list
                end
            end
        end
        for _, list in pairs(s.claims) do table.sort(list) end
    end
    scanCache = s
    return s
end

--- Forget the cached scan. Every update run starts with this.
function Identity.Refresh()
    scanCache = nil
end

-- ---------------------------------------------------------------------------
-- Lookups
-- ---------------------------------------------------------------------------

--- The id of a folder: its registration, else what its manifest declares, else
--- the folder name itself.
function Identity.Id(folder)
    if type(folder) ~= "string" or folder == "" then return folder end
    if byFolder[folder] then return byFolder[folder] end
    return (Identity.DeclaredId(folder)) or folder
end

--- The folder holding an id. Returns
---     folder, nil, nil, how      how: "registered" | "manifest"
---     nil, "duplicate", { folders }
---     nil, "missing"
--- Errors from GetResourceState are not caught: a caller that runs this inside
--- pcall reports them against the id.
function Identity.Locate(id)
    if type(id) ~= "string" or id == "" then return nil, "missing" end
    if conflicts[id] then return nil, "duplicate", sortedKeys(conflicts[id]) end

    local entry = byId[id]
    if entry then
        if started(entry.folder) then return entry.folder, nil, nil, "registered" end
        Identity.Forget(entry.folder)          -- stale: it stopped unnoticed
    end

    local s = Identity.Scan()
    local list = s.claims[id] or {}
    if #list == 0 and s.folders[id] == nil then
        -- A folder named like the id that declares nothing else: a resource
        -- that does not load the bridge, or a server without resource natives.
        local state = GetResourceState(id)
        if state ~= nil and state ~= "missing" and state ~= "unknown" then list = { id } end
    end

    if #list == 0 then return nil, "missing" end
    if #list == 1 then return list[1], nil, nil, "manifest" end

    -- Several folders on disk: one started claimant is not a guess, it is the
    -- only one that can be running the script.
    local running = {}
    for _, folder in ipairs(list) do
        if started(folder) then running[#running + 1] = folder end
    end
    if #running == 1 then return running[1], nil, nil, "manifest" end
    return nil, "duplicate", list
end

--- A name typed in a command, as an id: an id wins over a folder of the same name.
function Identity.Resolve(name)
    if type(name) ~= "string" or name == "" then return name end
    if byId[name] or conflicts[name] then return name end
    if byFolder[name] then return byFolder[name] end
    local s = Identity.Scan()
    if s.claims[name] then return name end
    if s.folders[name] then return s.folders[name] end
    return name
end

--- "poggy_scene" or "poggy_scene (folder: my_scene)".
function Identity.Label(id, folder)
    if folder and folder ~= id then
        return ("%s (folder: %s)"):format(tostring(id), tostring(folder))
    end
    return tostring(id)
end

--- Registered scripts sorted by id, and the ids two folders claim.
function Identity.List()
    local out = {}
    for _, entry in pairs(byId) do
        out[#out + 1] = { id = entry.id, folder = entry.folder, version = entry.version, registeredAt = entry.registeredAt }
    end
    table.sort(out, function(a, b) return a.id < b.id end)
    local dup = {}
    for id, set in pairs(conflicts) do dup[#dup + 1] = { id = id, folders = sortedKeys(set) } end
    table.sort(dup, function(a, b) return a.id < b.id end)
    return out, dup
end

-- poggy_core registers itself: it loads no bridge, and the updater needs its id.
do
    local okSelf, me = pcall(GetCurrentResourceName)
    if okSelf and type(me) == "string" then
        local okText, text = pcall(LoadResourceFile, me, "fxmanifest.lua")
        text = okText and text or nil
        pcall(Identity.Register, me, nil, Identity.ManifestField(text, "version"))
    end
end
