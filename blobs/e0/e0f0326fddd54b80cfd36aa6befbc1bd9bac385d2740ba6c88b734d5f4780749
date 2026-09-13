--[[
    Poggy() — the bridge to poggy_core.

    Every Poggy resource loads this file straight from poggy_core, as its first
    shared script, and depends on poggy_core:

        shared_script '@poggy_core/template/poggy.lua'
        dependency 'poggy_core'

    So there is exactly one copy, this one. Nothing is copied into the scripts,
    and updating poggy_core updates every script's bridge (each script picks it
    up when it restarts). Change it here and nowhere else.

    ------------------------------------------------------------------------
    One function. Every script, every call.

        local ok, value, err = Poggy(verb, payload)

    Examples, all the same shape:

        Poggy('money.add',     { src = src, amount = 50, reason = 'supply drop' })
        Poggy('char.get',      { src = src })
        Poggy('perms.isAdmin', { src = src })
        Poggy('inv.add',       { src = src, item = 'bandage', qty = 2 })
        Poggy('notify',        { src = src, text = 'Crate secured.', kind = 'success' })

    Returns:
        ok     boolean  did it happen
        value  any      the result, or nil
        err    string   why not, or nil

    ------------------------------------------------------------------------
    Branch on `ok`, and nothing else.

    poggy_core is a hard dependency, so it is always running when this file is.
    What can still happen is a core that is not ready yet (the framework is
    still resolving) or has no framework under it. Then every call returns
    false, nil, 'no_core', which looks exactly like any other failed call:

        local ok, char = Poggy('char.get', { src = src })
        if not ok then return end

    So there is only ever one thing to handle. Work that must run at startup,
    such as registering storage, waits with PoggyReady() first.

    A callback may be passed as the third argument. It is called with the same
    three values, and Poggy() then returns nothing:

        Poggy('money.get', { src = src }, function(ok, balance)
            if ok then print(balance) end
        end)

    ------------------------------------------------------------------------
    Minimum poggy_core.

    A script that calls verbs added in a later poggy_core declares it in its
    fxmanifest.lua:

        poggy_core_min '0.11.0'

    The file on disk can be newer than the poggy_core that is running: the
    updater installs poggy_core last and never restarts it. Against an older
    running core this bridge prints one red line at boot, PoggyReady() returns
    false, and every Poggy() call returns false, nil, 'core_too_old' (each verb
    is named in the console the first time it is refused). Nothing half-works.
    PoggyWriteOwnFile keeps working, so the update can still be installed.

    ------------------------------------------------------------------------
    Script identity (poggy_core 0.13.0).

    Every script declares its product id in its fxmanifest.lua:

        poggy_id 'poggy_scene'

    It is the name the update feed and poggy_core know the script by, and it
    never changes. A server owner may rename the FOLDER; the script keeps
    working, because it refers to itself with GetCurrentResourceName() and this
    bridge tells poggy_core which id lives in that folder (core.register, the
    first time PoggyReady() passes on the server).
]]

local BRIDGE_VERSION = "1.2.0"

local RESOURCE = GetCurrentResourceName()
local VERSION  = GetResourceMetadata(RESOURCE, "version", 0) or "?"
local handle   = nil             -- cached export table, once poggy_core is up
local complained = false

--- True when version a is older than version b. Compares the numbers, not the
--- text: as strings "0.10.0" < "0.3.0", which would call a newer core outdated.
--- Used by the poggy_core_min check below.
local function olderThan(a, b)
    local pa, pb = {}, {}
    for n in tostring(a):gmatch("%d+") do pa[#pa + 1] = tonumber(n) end
    for n in tostring(b):gmatch("%d+") do pb[#pb + 1] = tonumber(n) end
    for i = 1, math.max(#pa, #pb) do
        local x, y = pa[i] or 0, pb[i] or 0
        if x ~= y then return x < y end
    end
    return false
end

--- The `poggy_core_min` this resource declares in its fxmanifest.lua, or nil.
--- The file is read first: GetResourceMetadata serves a cached parse that a
--- plain restart does not refresh. A client may have no copy of the manifest
--- to read, so the metadata is the fallback there.
local function declaredCoreMin()
    local manifest = LoadResourceFile(RESOURCE, "fxmanifest.lua")
    if manifest then
        -- Alternating the Lua string delimiters keeps these free of escapes.
        local v = manifest:match("\n%s*poggy_core_min%s+'([^']+)'")
               or manifest:match('\n%s*poggy_core_min%s+"([^"]+)"')
               or manifest:match("^%s*poggy_core_min%s+'([^']+)'")
               or manifest:match('^%s*poggy_core_min%s+"([^"]+)"')
        if v then return v end
    end
    local meta = GetResourceMetadata(RESOURCE, "poggy_core_min", 0)
    if meta and meta ~= "" then return meta end
    return nil
end

local CORE_MIN = declaredCoreMin()
local tooOld   = nil             -- the running poggy_core's version, when older than CORE_MIN
local refused  = {}              -- verbs already named as refused, so a loop cannot flood the console

--- Is poggy_core up, does it have a framework under it, and is it new enough?
local function ready()
    if handle then return true end
    if tooOld then return false end
    if GetResourceState("poggy_core") ~= "started" then return false end

    local ok, api = pcall(function() return exports.poggy_core end)
    if not ok or not api then return false end

    -- Ask the core whether it is actually driving something. A core that has
    -- started but found no framework is not usable, and pretending otherwise
    -- is how silent no-ops happen.
    local good, alive = pcall(function()
        local o, v = api:Do("core.ready", {})
        return o and v
    end)
    if not good or not alive then return false end

    -- The version that matters is the RUNNING core's, which only the core can
    -- report: its fxmanifest.lua on disk may already be the newer one.
    if CORE_MIN then
        local okV, running = pcall(function()
            local o, v = api:Do("core.version", {})
            return o and v or nil
        end)
        if okV and running and olderThan(running, CORE_MIN) then
            tooOld = tostring(running)
            print(("^1❌ [Poggy]^7 %-24s ^9v%-8s^7 ^1needs poggy_core %s or newer, but v%s is running^7"
                .. " ^9· framework features are off until poggy_core is restarted"
                .. " (restart the server, or run: refresh, ensure poggy_core, ensure %s)^7")
                :format(RESOURCE, VERSION, CORE_MIN, tooOld, RESOURCE))
            return false
        end
    end

    handle = api
    return true
end

--- The single entry point. See the header.
function Poggy(verb, payload, cb)
    local ok, value, err

    if not ready() then
        if tooOld then
            ok, value, err = false, nil, "core_too_old"
            local key = tostring(verb)
            if not refused[key] then
                refused[key] = true
                print(("^1❌ [Poggy]^7 %s: Poggy('%s') refused ^9— needs poggy_core %s, v%s is running^7")
                    :format(RESOURCE, key, CORE_MIN, tooOld))
            end
        else
            ok, value, err = false, nil, "no_core"
        end
    else
        ok, value, err = handle:Do(verb, payload or {})
    end

    if cb then return cb(ok, value, err) end
    return ok, value, err
end

-- ---------------------------------------------------------------------------
-- Identity and database (server only)
--
-- The first time PoggyReady() is called on the server (the boot report below
-- always does), in this order, and PoggyReady() returns once both are done:
--
--   1. core.register: tells poggy_core that this folder is the script with this
--      poggy_id, so the updater, storage ids and migrations find it under any
--      folder name. poggy_core takes the folder from the call itself, never
--      from the payload. Skipped against a poggy_core older than 0.13.0.
--   2. sql.install: runs sql/install.sql, so a script's tables exist before its
--      first query, even on a first-ever start. Nothing is run for a script
--      without the file, and nothing against a poggy_core older than 0.12.0.
-- ---------------------------------------------------------------------------

local IDENTITY_CORE_MIN = "0.13.0"
local SQL_CORE_MIN = "0.12.0"
local dbState = nil              -- nil: not started, "running", "done"

--- A `key 'value'` line from manifest text, trimmed, or nil.
local function manifestValue(manifest, key)
    if not manifest then return nil end
    local v = manifest:match("\n%s*" .. key .. "%s+'([^'\n]+)'")
           or manifest:match('\n%s*' .. key .. '%s+"([^"\n]+)"')
           or manifest:match("^%s*" .. key .. "%s+'([^'\n]+)'")
           or manifest:match('^%s*' .. key .. '%s+"([^"\n]+)"')
    v = v and v:match("^%s*(.-)%s*$")
    if v == "" then return nil end
    return v
end

--- This script's product id: poggy_id in its fxmanifest.lua, read from the file
--- (the metadata keeps an old value until `refresh`), then the metadata, then
--- the folder name. The folder may be renamed; the id never is.
local function poggyId()
    local v = manifestValue(LoadResourceFile(RESOURCE, "fxmanifest.lua"), "poggy_id")
    if v then return v end
    local meta = GetResourceMetadata(RESOURCE, "poggy_id", 0)
    if type(meta) == "string" and meta:match("%S") then return (meta:match("^%s*(.-)%s*$")) end
    return RESOURCE
end

--- Runs in the thread of the first PoggyReady() caller (which can already wait);
--- later callers wait for it to finish.
local function onFirstReady()
    dbState = "running"
    local ok, err = pcall(function()
        local okV, cver = Poggy("core.version", {})
        if not okV or not cver then return end

        if not olderThan(cver, IDENTITY_CORE_MIN) then
            -- A refusal (another started folder already holds this id) is
            -- printed by poggy_core, naming both folders. The script still starts.
            local manifest = LoadResourceFile(RESOURCE, "fxmanifest.lua")
            Poggy("core.register", { id = poggyId(), version = manifestValue(manifest, "version") or VERSION })
        end

        if LoadResourceFile(RESOURCE, "sql/install.sql") and not olderThan(cver, SQL_CORE_MIN) then
            -- poggy_core prints what it changed, and any error. The result is not
            -- needed here: a failed install still lets the script start, so the
            -- owner sees the error in the console instead of a silent hang.
            Poggy("sql.install", {})
        end
    end)
    if not ok then
        print(("^1❌ [Poggy]^7 %s: start-up registration stopped with an error: %s^7"):format(RESOURCE, tostring(err)))
    end
    dbState = "done"
end

--- Blocks until poggy_core is usable, or the timeout passes. Returns a boolean.
--- Only needed for work that must run at startup, such as registering storage.
--- On the server it also waits for this script to register its poggy_id and for
--- its sql/install.sql to be applied.
--- Returns false at once when the running poggy_core is older than poggy_core_min.
function PoggyReady(timeoutMs)
    local deadline = GetGameTimer() + (timeoutMs or 30000)
    while not ready() do
        if tooOld then return false end
        if GetGameTimer() > deadline then return false end
        Wait(250)
    end
    if IsDuplicityVersion() then
        if not dbState then onFirstReady() end
        while dbState ~= "done" do
            if GetGameTimer() > deadline then return false end
            Wait(100)
        end
    end
    return true
end

-- ---------------------------------------------------------------------------
-- Boot report
--
-- One line, the same format in every Poggy resource, so a customer's console
-- reads as one product rather than eighteen unrelated scripts. poggy_core is a
-- dependency, so it is running; what this reports is whether it is usable.
-- ---------------------------------------------------------------------------

CreateThread(function()
    if not PoggyReady(30000) then
        complained = true
        -- ready() has already printed the one red line for a core that is too old.
        if tooOld then return end
        print(("^1❌ [Poggy]^7 %-24s ^9v%-8s^7 ^1poggy_core is not ready after 30 s^7 ^9— framework features are off; run poggycore detect^7")
            :format(RESOURCE, VERSION))
        return
    end

    local okFw, fw   = Poggy("core.framework", {})
    local _,    cver = Poggy("core.version", {})

    -- Say plainly when the core is up but driving nothing. A green "ready" on a
    -- server with no framework would be a lie, and the customer would only find
    -- out when something quietly did not happen.
    if not okFw or not fw or fw.id == nil or fw.id == "standalone" then
        print(("^3⚠️  [Poggy]^7 %-24s ^9v%-8s^7 ^3no framework^7 ^9· poggy_core v%s · framework features are off^7")
            :format(RESOURCE, VERSION, cver or "?"))
        return
    end

    print(("^2✅ [Poggy]^7 %-24s ^9v%-8s^7 ^2ready^7 ^9· poggy_core v%s · %s^7")
        :format(RESOURCE, VERSION, cver or "?", fw.label or fw.id))
end)

-- ---------------------------------------------------------------------------
-- Update receiver (server only)
--
-- A server script may write files only inside its own resource folder. The
-- write test on 13 September 2026 proved it: poggy_core's SaveResourceFile into
-- another resource returns false, and io.open refuses every write. So when
-- poggy_core updates this resource, it downloads the files and
-- hands each one here, and this resource saves it into its own folder.
--
-- Only poggy_core may call it, and only with a path inside this resource.
-- Anything else is refused, so no other resource can use it to rewrite this one.
-- It does not go through ready(), so it works even when poggy_core_min is not
-- met: that is exactly when the update needs to be installed.
-- ---------------------------------------------------------------------------

--- `set poggy_dev_server 1` (or "true") in server.cfg marks a development server,
--- whose folders ARE the scripts' source: it never accepts update files. This is
--- a second lock beside poggy_core's own, so no bug there can overwrite source.
local function isDevServer()
    local okInt, n = pcall(GetConvarInt, "poggy_dev_server", 0)
    if okInt and tonumber(n) == 1 then return true end
    local okStr, s = pcall(GetConvar, "poggy_dev_server", "")
    if okStr and type(s) == "string" then
        s = s:lower():match("^%s*(.-)%s*$")
        return s == "1" or s == "true"
    end
    return false
end

if IsDuplicityVersion() then
    exports("PoggyWriteOwnFile", function(rel, data)
        if GetInvokingResource() ~= "poggy_core" then
            return { ok = false, err = "refused: only poggy_core may write files here" }
        end
        if isDevServer() then
            return { ok = false, err = "refused: development server (poggy_dev_server); update files are never written here" }
        end
        if type(rel) ~= "string" or type(data) ~= "string" then
            return { ok = false, err = "refused: bad arguments" }
        end
        -- .fxap files are per-customer licence tokens. Nothing may replace one.
        if rel:lower():sub(-5) == ".fxap" then
            return { ok = false, err = "refused: .fxap files are licence tokens and are never written" }
        end
        if rel == "" or rel:find("..", 1, true) or rel:find("^[/\\]") or rel:find(":", 1, true) then
            return { ok = false, err = "refused: path outside the resource: " .. rel }
        end
        if SaveResourceFile(RESOURCE, rel, data, #data) then
            return { ok = true }
        end
        return { ok = false, err = "SaveResourceFile returned false (does the folder exist?)" }
    end)
end
