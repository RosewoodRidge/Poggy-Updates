--[[
    poggy_core — dependents: the resources that stop when poggy_core stops (0.15.0).

    Every Poggy script declares `dependency 'poggy_core'`, so `restart poggy_core`
    stops all of them, and FXServer never starts them again on its own. This
    file does, once poggy_core is back.

    A dependent is any resource whose fxmanifest.lua declares a `poggy_id`, or
    lists poggy_core under `dependency` / `dependencies`. The manifest metadata
    is read first (GetNumResourceMetadata / GetResourceMetadata; `dependencies`
    lands under the key "dependency", both keys are read), the manifest text
    when the natives are not there.

    Restart versus fresh boot. On a fresh server boot every Poggy script is
    "stopped" as well: server.cfg simply has not reached them yet and will start
    them in its own order, so nothing may be started then. The rule:

      1. While poggy_core runs it keeps a record of every dependent it has seen
         started: each onResourceStart, plus whatever was already started when
         poggy_core itself came up. A dependent the owner stops on its own leaves
         the record again; one that stops in the fifteen seconds before
         poggy_core stops is kept, because it stopped with poggy_core.
      2. The record is written to the convar `poggy_core_last_stop` (SetConvar)
         on every change and once more when poggy_core stops. A convar set at
         run time lives in the server process only: a server restart clears it.
         So on a fresh boot there is no record, and nothing is started.
      3. When poggy_core starts and finds a record, the server was already
         running and poggy_core was restarted. Every recorded resource that is
         still a dependent and is "stopped" is started with `ensure`
         (ExecuteCommand; the same `add_ace resource.poggy_core command.ensure
         allow` line the updater uses; StartResource when that ACE is missing).
         A resource the server never started is not in the record and is left
         alone. The record is then consumed, and this run starts its own.
      4. Belt and braces: the record is ignored during the first ten seconds of
         server uptime (GetGameTimer), when no dependent can have stopped yet.

    A poggy_core older than 0.15.0 wrote no record, so the first restart after
    an upgrade finds none: when the server has been up for more than five
    minutes and dependents are stopped, one grey line lists them with the
    `ensure` commands to run. Every restart after that has a record.

    PoggyCoreConfig.RestartDependents = false turns the restart off; the record
    is still kept, so the switch can be turned on without a server restart. This
    is not a file write, so it also runs on a development server.

    poggycore dependents lists every dependent with its state and the record.

    Waiting for the players' games (0.20.4). Starting the dependents two seconds
    after poggy_core was enough for the SERVER, and wrong for the CLIENTS. Every
    Poggy script loads `@poggy_core/template/poggy.lua` (and most the prompt
    library) out of poggy_core on the player's machine. After `restart
    poggy_core` each connected game stops poggy_core, fetches whichever of its
    files changed (the hub alone is 1.7 MB) and starts it again; a dependent is
    unchanged, so it is cached and starts at once. When poggy_core had changed,
    the dependents started on the client first, the `@poggy_core/...` files were
    not there to load, and the scripts ran without their bridge:

        attempt to call a nil value (global 'Poggy')
        attempt to call a nil value (global 'PoggyReady')
        attempt to index a nil value (field 'PoggyPromptGroup')

    A second `restart poggy_core` always worked, because by then nothing had
    changed and the client had poggy_core up within milliseconds. That is why it
    looked random: it happened exactly when poggy_core had been updated.

    So poggy_core's client says when it is up (`poggy_core:clientUp`, the last
    thing cl_core.lua does), the server keeps the ids that said so in the convar
    `poggy_core_clients_up`, and after a restart the dependents are started once
    every one of THOSE players who is still connected has said so again, or
    after CLIENT_WAIT_MS. Players who were still loading in at the restart never
    said so before, so they are not waited for: a joining game loads every
    resource in order anyway.
]]

PoggyCore = PoggyCore or {}

local Dependents = PoggyCore.Dependents or {}
PoggyCore.Dependents = Dependents

local PREFIX = PoggyCore.PREFIX or "^5[Poggy Core]^7 "

Dependents.CONVAR = "poggy_core_last_stop"
local MIN_UPTIME_MS      = 10 * 1000        -- a record found earlier than this is not a restart
local STOPPED_WITH_US_MS = 15 * 1000        -- a dependent that stopped this close before us stopped because of us
local UPGRADE_HINT_MS    = 5 * 60 * 1000    -- no record, but the server has been up this long: say what is stopped
local START_DELAY_MS     = 2000             -- let poggy_core finish starting before dependents are started
local SETTLE_MS          = 1500             -- how long a started resource gets before its state is read
local CLIENT_WAIT_MS     = 20 * 1000        -- the longest the players' games get to load poggy_core again
local CLIENT_POLL_MS     = 250

Dependents.CLIENTS_CONVAR = "poggy_core_clients_up"

local function log(msg)
    print(PREFIX .. msg)
end

local function me()
    local ok, name = pcall(GetCurrentResourceName)
    return (ok and type(name) == "string") and name or "poggy_core"
end

local function now()
    local ok, t = pcall(GetGameTimer)
    return (ok and tonumber(t)) or 0
end

local function state(folder)
    local ok, s = pcall(GetResourceState, folder)
    return ok and tostring(s) or "unknown"
end

local function trim(s)
    return (tostring(s):match("^%s*(.-)%s*$"))
end

--- Is RestartDependents on? Unset counts as on.
local function enabled()
    return PoggyCoreConfig == nil or PoggyCoreConfig.RestartDependents ~= false
end

-- ---------------------------------------------------------------------------
-- What is a dependent
-- ---------------------------------------------------------------------------

--- Does `folder` list poggy_core under dependency / dependencies?
local function dependsOnCore(folder, own, text)
    if GetNumResourceMetadata and GetResourceMetadata then
        for _, key in ipairs({ "dependency", "dependencies" }) do
            local okN, n = pcall(GetNumResourceMetadata, folder, key)
            for i = 0, ((okN and tonumber(n)) or 0) - 1 do
                local okV, v = pcall(GetResourceMetadata, folder, key, i)
                if okV and type(v) == "string" and trim(v) == own then return true end
            end
        end
    end
    -- The manifest text: a dependency line naming poggy_core exactly. The bridge
    -- path ('@poggy_core/template/poggy.lua') does not match this.
    if type(text) == "string" and text:find("dependenc", 1, true) then
        for _, q in ipairs({ "'", '"' }) do
            if text:find(q .. own .. q, 1, true) then return true end
        end
    end
    return false
end

--- "poggy_id", id  |  "dependency", folder  |  nil when it is not a dependent.
local function describe(folder, own)
    local declared, text
    local I = PoggyCore.Identity
    if I and I.DeclaredId then
        declared, text = I.DeclaredId(folder)
    else
        local ok, t = pcall(LoadResourceFile, folder, "fxmanifest.lua")
        text = ok and t or nil
    end
    if declared then return "poggy_id", declared end
    if dependsOnCore(folder, own, text) then return "dependency", folder end
    return nil
end

--- Is this folder a dependent of poggy_core?
function Dependents.Is(folder)
    if type(folder) ~= "string" or folder == "" or folder == me() then return false end
    return describe(folder, me()) ~= nil
end

--- Every dependent on this server, sorted by folder:
--- { folder, id, state, why = "poggy_id" | "dependency" }.
function Dependents.List()
    local out, own = {}, me()
    if not (GetNumResources and GetResourceByFindIndex) then return out end
    local okN, n = pcall(GetNumResources)
    for i = 0, ((okN and tonumber(n)) or 0) - 1 do
        local okR, folder = pcall(GetResourceByFindIndex, i)
        if okR and type(folder) == "string" and folder ~= "" and folder ~= own then
            local why, id = describe(folder, own)
            if why then
                out[#out + 1] = { folder = folder, id = id, state = state(folder), why = why }
            end
        end
    end
    table.sort(out, function(a, b) return a.folder < b.folder end)
    return out
end

-- ---------------------------------------------------------------------------
-- The record
-- ---------------------------------------------------------------------------

local seen      = {}   -- folder -> true: dependents seen started while this poggy_core runs
local stoppedAt = {}   -- folder -> GetGameTimer() when it stopped on its own

--- The names the record holds: everything seen started. When poggy_core is
--- stopping (withRecent), also what stopped within STOPPED_WITH_US_MS: those
--- stopped because it is.
local function recordNames(withRecent)
    local names, t = {}, now()
    for folder in pairs(seen) do names[#names + 1] = folder end
    if withRecent then
        for folder, at in pairs(stoppedAt) do
            if not seen[folder] and t - at <= STOPPED_WITH_US_MS then names[#names + 1] = folder end
        end
    end
    table.sort(names)
    return names
end

local function writeRecord(withRecent)
    if not SetConvar then return end
    pcall(SetConvar, Dependents.CONVAR, ("%d|%s"):format(os.time(), table.concat(recordNames(withRecent), ",")))
end

local function clearRecord()
    if not SetConvar then return end
    pcall(SetConvar, Dependents.CONVAR, "")
end

--- The record the previous poggy_core left: { at = os.time(), names = {...} }, or nil.
function Dependents.ReadRecord()
    local ok, raw = pcall(GetConvar, Dependents.CONVAR, "")
    if not ok or type(raw) ~= "string" or raw == "" then return nil end
    local at, list = raw:match("^(%d+)|(.*)$")
    if not at then return nil end
    local names = {}
    for name in list:gmatch("[^,]+") do names[#names + 1] = name end
    return { at = tonumber(at), names = names }
end

--- Note a dependent that started (onResourceStart, or found started at our start).
local function noteStarted(folder)
    seen[folder] = true
    stoppedAt[folder] = nil
    writeRecord()
end

--- Note a dependent that stopped. Whether it stopped with us is only known
--- when we stop: recordNames() keeps anything that stopped just before.
local function noteStopped(folder)
    if not seen[folder] and not stoppedAt[folder] then return end
    seen[folder] = nil
    stoppedAt[folder] = now()
    writeRecord()
end

-- ---------------------------------------------------------------------------
-- Restore
-- ---------------------------------------------------------------------------

-- ---------------------------------------------------------------------------
-- Which players' games have poggy_core running
-- ---------------------------------------------------------------------------

local clientsUp = {}   -- [src] = true, this run

local function readClients()
    local ok, raw = pcall(GetConvar, Dependents.CLIENTS_CONVAR, "")
    local ids = {}
    if ok and type(raw) == "string" then
        for id in raw:gmatch("%d+") do ids[#ids + 1] = tonumber(id) end
    end
    return ids
end

-- Read while this file loads, before any event of this run can be handled:
-- the first clientUp of this run overwrites the convar.
local clientsBefore = readClients()

local function writeClients()
    if not SetConvar then return end
    local ids = {}
    for id in pairs(clientsUp) do ids[#ids + 1] = id end
    table.sort(ids)
    pcall(SetConvar, Dependents.CLIENTS_CONVAR, table.concat(ids, ","))
end

function Dependents.ClientUp(src)
    src = tonumber(src)
    if not src or src <= 0 or clientsUp[src] then return end
    clientsUp[src] = true
    writeClients()
end

function Dependents.ClientGone(src)
    src = tonumber(src)
    if src and clientsUp[src] then clientsUp[src] = nil; writeClients() end
end

local function connected(id)
    local ok, name = pcall(GetPlayerName, id)
    return ok and name ~= nil
end

--- Wait until every player whose game had poggy_core before the restart has it
--- again. Returns the milliseconds waited and the ids that never answered.
--- Counted in polls, not by the clock, so it ends even if the clock does not move.
function Dependents.WaitForClients(ids)
    local waited, missing = 0, {}
    for _ = 0, math.floor(CLIENT_WAIT_MS / CLIENT_POLL_MS) do
        missing = {}
        for _, id in ipairs(ids or {}) do
            if not clientsUp[id] and connected(id) then missing[#missing + 1] = id end
        end
        if #missing == 0 or waited >= CLIENT_WAIT_MS then break end
        Wait(CLIENT_POLL_MS)
        waited = waited + CLIENT_POLL_MS
    end
    return waited, missing
end

local function aceAllowed(object)
    if IsPrincipalAceAllowed == nil then return false end
    local ok, allowed = pcall(IsPrincipalAceAllowed, "resource." .. me(), object)
    return ok and allowed == true
end

--- Run when poggy_core starts. Yields (Wait), so call it from a thread.
--- Returns the list of folders it started (possibly empty) and a reason word.
function Dependents.Restore()
    local own = me()

    -- A fresh Lua state has nothing seen yet; whatever is started already
    -- belongs in this run's record.
    seen, stoppedAt = {}, {}
    for _, d in ipairs(Dependents.List()) do
        if d.state == "started" then seen[d.folder] = true end
    end

    local record = Dependents.ReadRecord()
    local uptime = now()
    if not record then
        writeRecord()
        -- An older poggy_core kept no record. Long after boot, stopped
        -- dependents are worth one grey line, but they are never started blind.
        if uptime >= UPGRADE_HINT_MS then
            local stopped = {}
            for _, d in ipairs(Dependents.List()) do
                if d.state == "stopped" then stopped[#stopped + 1] = d.folder end
            end
            if #stopped > 0 then
                log(("^9%d Poggy script(s) are stopped and there is no record of what stopped with %s"
                    .. " (a poggy_core older than 0.15.0 keeps none): %s. To start them: ensure %s^7")
                    :format(#stopped, own, table.concat(stopped, ", "), table.concat(stopped, ", ensure ")))
            end
        end
        return {}, "fresh boot"
    end

    -- Consumed: from here on this run keeps its own record.
    clearRecord()
    writeRecord()
    if uptime < MIN_UPTIME_MS then return {}, "too early" end
    if #record.names == 0 then return {}, "nothing recorded" end

    Wait(START_DELAY_MS)

    -- The players' games must have poggy_core back before a script that loads
    -- its bridge out of it is started (see the top of this file).
    local waited, missing = Dependents.WaitForClients(clientsBefore)
    if #missing > 0 then
        log(("^3⚠️  %d player(s) had not loaded %s again after %d s (id %s). Starting the scripts anyway;"
            .. " if theirs show \"attempt to call a nil value (global 'Poggy')\", run: restart %s^7")
            :format(#missing, own, math.floor(waited / 1000), table.concat(missing, ", "), own))
    elseif waited > 0 then
        log(("^9waited %.1f s for %d player(s) to load %s again before starting its scripts^7")
            :format(waited / 1000, #clientsBefore, own))
    end

    -- Still a dependent, and stopped now: a resource that came back some other
    -- way, or was removed from the server, is left alone.
    local wanted = {}
    for _, folder in ipairs(record.names) do
        if Dependents.Is(folder) and state(folder) == "stopped" then wanted[#wanted + 1] = folder end
    end
    if #wanted == 0 then return {}, "nothing stopped" end

    if not enabled() then
        log(("^9RestartDependents is off. %d Poggy script(s) stopped with %s and are still stopped: %s."
            .. " To start them: ensure %s^7"):format(#wanted, own, table.concat(wanted, ", "),
                table.concat(wanted, ", ensure ")))
        return {}, "disabled"
    end

    local useEnsure = aceAllowed("command.ensure")
    for _, folder in ipairs(wanted) do
        if useEnsure then
            ExecuteCommand("ensure " .. folder)
        else
            pcall(StartResource, folder)
        end
    end
    Wait(SETTLE_MS)

    local restarted, failed = {}, {}
    for _, folder in ipairs(wanted) do
        local s = state(folder)
        if s == "started" or s == "starting" then
            restarted[#restarted + 1] = folder
            seen[folder] = true
        else
            failed[#failed + 1] = { folder = folder, state = s }
        end
    end
    writeRecord()

    if #restarted > 0 then
        log(("^2restarted %d Poggy script(s)^7 that stopped with it: %s"):format(#restarted, table.concat(restarted, ", ")))
    end
    for _, f in ipairs(failed) do
        log(("^3⚠️  %s stopped with %s and did not come back (state: %s). Run: ensure %s^7")
            :format(f.folder, own, f.state, f.folder))
    end
    if #failed > 0 and not useEnsure then
        log(("   ^9add_ace resource.%s command.ensure allow^7 ^9in server.cfg lets poggy_core use ensure^7"):format(own))
    end
    return restarted, "restarted"
end

-- ---------------------------------------------------------------------------
-- poggycore dependents
-- ---------------------------------------------------------------------------

function Dependents.Command(say)
    local list = Dependents.List()
    say(("%d dependent resource(s): a poggy_id in the manifest, or dependency '%s'"):format(#list, me()))
    for _, d in ipairs(list) do
        local colour = d.state == "started" and "^2" or (d.state == "stopped" and "^3" or "^9")
        say(("  %-24s %s%-10s^7 %s%s"):format(d.folder, colour, d.state, d.why,
            (d.why == "poggy_id" and d.id ~= d.folder) and (" " .. d.id) or ""))
    end
    local record = Dependents.ReadRecord()
    if record then
        say(("record (%s): %d resource(s) seen started, written %s — %s")
            :format(Dependents.CONVAR, #record.names, os.date("%Y-%m-%d %H:%M:%S", record.at),
                #record.names > 0 and table.concat(record.names, ", ") or "none"))
    else
        say(("record (%s): none"):format(Dependents.CONVAR))
    end
    local up = {}
    for id in pairs(clientsUp) do up[#up + 1] = id end
    table.sort(up)
    say(("players whose game has %s running: %d%s ^9(after a restart the scripts wait for these, %d s at most)^7")
        :format(me(), #up, #up > 0 and (" (id " .. table.concat(up, ", ") .. ")") or "", CLIENT_WAIT_MS / 1000))
    say(("RestartDependents: %s — after `restart %s`, the recorded resources that are stopped are started again")
        :format(enabled() and "^2on^7" or "^9off^7", me()))
end

-- ---------------------------------------------------------------------------
-- Events
-- ---------------------------------------------------------------------------

if RegisterNetEvent and AddEventHandler then
    -- A player's game has poggy_core running (again). Anyone may send it; all it
    -- can do is end a wait early for the sender's own id.
    RegisterNetEvent("poggy_core:clientUp")
    AddEventHandler("poggy_core:clientUp", function() Dependents.ClientUp(source) end)
    AddEventHandler("playerDropped", function() Dependents.ClientGone(source) end)
end

if AddEventHandler then
    AddEventHandler("onResourceStart", function(resource)
        if resource == me() then
            CreateThread(function()
                local ok, err = pcall(Dependents.Restore)
                if not ok then log("^1❌ restarting the dependents stopped with an error: " .. tostring(err) .. "^7") end
            end)
        elseif Dependents.Is(resource) then
            noteStarted(resource)
        end
    end)

    AddEventHandler("onResourceStop", function(resource)
        if resource == me() then
            writeRecord(true)
        else
            noteStopped(resource)
        end
    end)
end
