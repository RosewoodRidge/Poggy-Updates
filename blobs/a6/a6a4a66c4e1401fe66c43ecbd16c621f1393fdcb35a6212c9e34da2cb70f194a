--[[
    poggy_core — held jobs, job holders, character profiles, duty and law (0.25.0).

    Until 0.25.0 poggy_core knew one job per character: the one the framework
    has them wearing. Three more job stores exist in the wild and nothing read
    them together: VORP 3.3's own multijob (characters.multijobs), rsg-multijob
    (player_jobs) and poggy_multijob's table. A law script's records could not
    say who the deputies are. This file answers that in one place:

      jobs.of        every job a character holds, worn or not
      jobs.holders   every character holding a job, offline included
      char.profile   what the framework knows about a character beyond the name
      char.list      withJob = true adds the worn job and online to each row

    Who answers:

      * A `jobs` provider (sv_providers.lua) when one is registered: a
        multijob script whose table is the truth. raw = true bypasses it; the
        provider itself uses that to read the framework's worn job.
      * Otherwise the adapter: jobsOf(charId, src), jobsHolders(names, online),
        charProfile(charId). VORP adds its own multijob, RSG adds rsg-multijob
        when it runs. Online characters are always read live, never from a
        table the framework has not saved yet.

    Duty: with a `duty` provider (a law script with its own duty), every duty
    question on every framework goes to it, and the answer is mirrored into the
    replicated state bag `poggyDuty` so the client's job.get agrees. Without
    one nothing changes: the adapter answers as before.

    Law: job.isLaw is the LawJobs list, plus (LeoJobsAreLaw) every job the
    framework itself types as law ("leo" on RSG and QBCore), published to
    clients through GlobalState.poggyLeoJobs.

    This file also holds the SQL the three frameworks keyed by citizenid share
    (RSG, QBR on RedM; QBCore on FiveM): players.charinfo and players.job are
    JSON there, so the queries are the same and live here once.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

local Jobs = {}
PoggyCore.Jobs = Jobs

local function providers() return PoggyCore.Providers end

--- A provider that asks poggy_core the same question from inside its answer
--- (a duty provider's get calling char.get, a jobs provider's of calling
--- jobs.of without raw) would loop forever. While a provider is answering for
--- a key, the same question for that key goes to the framework instead. The
--- provider runs in its own resource, so the key is the question, not the
--- coroutine.
local busy = {}

local function guarded(key, fn, ...)
    busy[key] = (busy[key] or 0) + 1
    local a, b, c = fn(...)
    busy[key] = busy[key] - 1
    if busy[key] <= 0 then busy[key] = nil end
    return a, b, c
end

local function adapter()
    local S = PoggyCore.State
    if not S or not S.ready then return nil end
    return S.adapter
end

--- "" and nil are both "nothing".
local function nz(v)
    if v == nil or v == "" then return nil end
    return v
end

--- The most rows a holders query reads from a framework table. A job held by
--- more characters than this on one server is not a real case; the cap only
--- keeps a typo ("%") from reading the whole table into memory.
local HOLDERS_CAP = 5000

--- JSON columns come back as strings from oxmysql; decode defensively.
function Jobs.DecodeJson(v)
    if type(v) == "table" then return v end
    if type(v) ~= "string" or v == "" then return {} end
    local ok, t = pcall(json.decode, v)
    return (ok and type(t) == "table") and t or {}
end

--- A time from the database as unix seconds. oxmysql hands DATETIME and
--- TIMESTAMP back as milliseconds; a string is parsed; anything else is nil.
function Jobs.ToUnix(v)
    if type(v) == "number" then
        if v > 100000000000 then return math.floor(v / 1000) end
        return math.floor(v)
    end
    if type(v) == "string" then
        local y, mo, d, h, mi, s = v:match("^(%d%d%d%d)%-(%d%d)%-(%d%d)[ T]?(%d*):?(%d*):?(%d*)")
        if y then
            return os.time({ year = tonumber(y), month = tonumber(mo), day = tonumber(d),
                hour = tonumber(h) or 0, min = tonumber(mi) or 0, sec = tonumber(s) or 0 })
        end
        return tonumber(v)
    end
    return nil
end

--- charId -> { src, char } for every loaded character.
function Jobs.Online(a)
    local out = {}
    for _, src in ipairs(a:getPlayers() or {}) do
        local ch = a:getChar(src)
        if ch and ch.charId and ch.charId ~= "" then
            out[tostring(ch.charId)] = { src = tonumber(src), char = ch }
        end
    end
    return out
end

-- ---------------------------------------------------------------------------
-- The job registry, for labels
-- ---------------------------------------------------------------------------

local registry = { at = -1e9, byName = nil }

--- lower-cased job name -> { name, label, type, grades = { [n] = { grade, label, boss } } }.
--- Only where the framework has a real registry (job.registry): VORP's
--- "registry" is a DISTINCT over characters and would only echo names back.
--- Kept a minute; RSG and QBCore can add jobs while running.
function Jobs.Registry(a)
    if not a or not a.caps or a.caps["job.registry"] ~= true or not a.jobsList then return {} end
    if registry.byName and (os.time() - registry.at) < 60 then return registry.byName end
    local byName = {}
    local ok, list = pcall(a.jobsList, a)
    if ok and type(list) == "table" then
        for _, j in ipairs(list) do
            if type(j) == "table" and j.name then
                local grades = {}
                for _, g in ipairs(j.grades or {}) do
                    if type(g) == "table" and tonumber(g.grade) then grades[tonumber(g.grade)] = g end
                end
                byName[tostring(j.name):lower()] = { name = j.name, label = j.label, type = j.type, grades = grades }
            end
        end
    end
    registry.byName, registry.at = byName, os.time()
    return byName
end

function Jobs.DropRegistry() registry.byName = nil end

--- One job entry in the published shape.
local function entry(e, reg, defaultSource)
    if type(e) ~= "table" or e.name == nil or e.name == "" then return nil end
    local name  = tostring(e.name)
    local grade = tonumber(e.grade) or 0
    local def   = reg and reg[name:lower()]
    local g     = def and def.grades[grade]
    return {
        name       = name,
        label      = nz(e.label) and tostring(e.label) or (def and def.label) or name,
        grade      = grade,
        gradeLabel = nz(e.gradeLabel) or (g and g.label) or nil,
        active     = e.active,   -- normalised by the caller: nil means "not said"
        source     = nz(e.source) or defaultSource or "framework",
    }
end

--- A job name, or an array of them, as a clean array. nil when unusable.
local function jobNames(job)
    local list = type(job) == "table" and job or { job }
    local out, seen = {}, {}
    for _, n in ipairs(list) do
        if type(n) == "string" and n ~= "" and #n <= 255 and not seen[n:lower()] then
            seen[n:lower()] = true
            out[#out + 1] = n
        end
    end
    if #out == 0 or #out > 50 then return nil end
    return out
end
Jobs.Names = jobNames

-- ---------------------------------------------------------------------------
-- jobs.of
-- ---------------------------------------------------------------------------

--- Resolve { charId | src } to charId (string) and the online src, or nil, err.
local function whoOf(a, p)
    local src = tonumber(p.src)
    if src then
        local ch = a:getChar(src)
        if not ch then return nil, nil, Err.NO_CHAR end
        return tostring(ch.charId), src
    end
    if p.charId == nil or p.charId == "" then return nil, nil, Err.BAD_ARG end
    local charId = tostring(p.charId)
    local live = a:getCharByCharId(charId)
    return charId, live and tonumber(live.source) or nil
end
Jobs.WhoOf = whoOf

--- The job a character is wearing, by name, for a provider that did not say.
local function wornJob(a, charId, src)
    if src then
        local ch = a:getChar(src)
        if ch then return ch.job end
    end
    if a.charOffline then
        local ch = a:charOffline(charId, false)
        if type(ch) == "table" then return ch.job end
    end
    return nil
end

--- payload { charId | src, raw? } -> ok, array of { name, label, grade, gradeLabel, active, source }
function Jobs.Of(p)
    local a = adapter()
    if not a then return false, nil, Err.NOT_READY end
    local charId, src, werr = whoOf(a, p)
    if not charId then return false, nil, werr end

    local P = providers()
    local list, defaultSource
    if not p.raw and P.Has("jobs") and not busy["of:" .. charId] then
        local ok, v, perr = guarded("of:" .. charId, P.Call, "jobs", "of", charId, src)
        if not ok then return false, nil, perr end
        if type(v) ~= "table" then return false, nil, Err.FRAMEWORK_ERR end
        list, defaultSource = v, P.Owner("jobs")
    else
        if not a.jobsOf then return false, nil, Err.UNSUPPORTED end
        local v, err = a:jobsOf(charId, src)
        if not v then return false, nil, err or Err.NOT_FOUND end
        list = v
    end

    local reg = Jobs.Registry(a)
    local out, byName, unsaid = {}, {}, false
    for _, raw in ipairs(list) do
        local e = entry(raw, reg, defaultSource)
        if e then
            local key = e.name:lower()
            local have = byName[key]
            if not have then
                byName[key] = e
                out[#out + 1] = e
            elseif e.active == true and have.active ~= true then
                -- The same job twice (worn, and on a list too): the worn one wins.
                for k, v in pairs(e) do have[k] = v end
            end
            if e.active == nil then unsaid = true end
        end
    end

    -- A provider that did not mark the worn job: mark it from the framework.
    if unsaid then
        local worn = wornJob(a, charId, src)
        for _, e in ipairs(out) do
            if e.active == nil then e.active = worn ~= nil and e.name:lower() == tostring(worn):lower() end
        end
    end
    for _, e in ipairs(out) do e.active = e.active == true end

    table.sort(out, function(x, y)
        if x.active ~= y.active then return x.active end
        return x.label:lower() < y.label:lower()
    end)
    return true, out, nil
end

-- ---------------------------------------------------------------------------
-- jobs.holders
-- ---------------------------------------------------------------------------

--- payload { job, minGrade?, activeOnly?, limit?, offset?, raw? }
--- -> ok, array of { charId, fullName, firstName, lastName, job, label, grade,
---    gradeLabel, active, online, src?, source }
function Jobs.Holders(p)
    local a = adapter()
    if not a then return false, nil, Err.NOT_READY end
    local names = jobNames(p.job)
    if not names then return false, nil, Err.BAD_ARG end
    local minGrade   = tonumber(p.minGrade)
    local activeOnly = p.activeOnly == true
    local limit      = tonumber(p.limit)
    local offset     = math.max(0, math.floor(tonumber(p.offset) or 0))
    if limit then limit = math.max(0, math.floor(limit)) end

    local want = {}
    for _, n in ipairs(names) do want[n:lower()] = true end

    local online = Jobs.Online(a)
    local P = providers()
    local rows, defaultSource
    local hkey = "holders:" .. table.concat(names, "\0"):lower()
    if not p.raw and P.Has("jobs") and not busy[hkey] then
        -- The provider returns every match; filtering, order and paging are
        -- done here, the same for every source.
        local ok, v, perr = guarded(hkey, P.Call, "jobs", "holders", { jobs = names, minGrade = minGrade, activeOnly = activeOnly })
        if not ok then return false, nil, perr end
        if type(v) ~= "table" then return false, nil, Err.FRAMEWORK_ERR end
        rows, defaultSource = v, P.Owner("jobs")
    else
        if not a.jobsHolders then return false, nil, Err.UNSUPPORTED end
        local v, err = a:jobsHolders(names, online)
        if not v then return false, nil, err or Err.FRAMEWORK_ERR end
        rows = v
    end

    local reg = Jobs.Registry(a)
    local out, byKey = {}, {}
    for _, r in ipairs(rows) do
        local e = type(r) == "table" and entry({
            name = r.job or r.name, label = r.label or r.jobLabel, grade = r.grade,
            gradeLabel = r.gradeLabel, source = r.source,
        }, reg, defaultSource) or nil
        local charId = type(r) == "table" and r.charId ~= nil and tostring(r.charId) or nil
        if e and charId and charId ~= "" and want[e.name:lower()] then
            local o = online[charId]
            local first, last = r.firstName, r.lastName
            if o then first, last = o.char.firstName, o.char.lastName end
            local row = {
                charId     = charId,
                firstName  = first or "",
                lastName   = last or "",
                fullName   = nz(r.fullName) and not o and r.fullName
                             or ((first or "") .. " " .. (last or "")):gsub("^%s+", ""),
                job        = e.name,
                label      = e.label,
                grade      = e.grade,
                gradeLabel = e.gradeLabel,
                active     = r.active == true,
                online     = o ~= nil,
                src        = o and o.src or nil,
                source     = e.source,
            }
            -- A provider that did not say which job is worn: an online
            -- character's is known for free.
            if r.active == nil and o then row.active = tostring(o.char.job or ""):lower() == e.name:lower() end

            local key = charId .. "\0" .. e.name:lower()
            local have = byKey[key]
            if not have then
                byKey[key] = row
                out[#out + 1] = row
            elseif row.active and not have.active then
                for k, v in pairs(row) do have[k] = v end
            end
        end
    end

    local kept = {}
    for _, row in ipairs(out) do
        if (not activeOnly or row.active) and (not minGrade or row.grade >= minGrade) then
            kept[#kept + 1] = row
        end
    end
    table.sort(kept, function(x, y)
        local xn, yn = x.fullName:lower(), y.fullName:lower()
        if xn ~= yn then return xn < yn end
        if x.charId ~= y.charId then return x.charId < y.charId end
        return x.job < y.job
    end)

    if offset == 0 and not limit then return true, kept, nil end
    local page = {}
    for i = offset + 1, #kept do
        if limit and #page >= limit then break end
        page[#page + 1] = kept[i]
    end
    return true, page, nil
end

-- ---------------------------------------------------------------------------
-- jobs.add / jobs.remove / jobs.setGrade (provider only)
-- ---------------------------------------------------------------------------

--- op: "add" | "remove" | "setGrade". Without a jobs provider there is no
--- list to change: 'unsupported', and the caller falls back to job.set.
function Jobs.Mutate(op, p)
    local a = adapter()
    if not a then return false, nil, Err.NOT_READY end
    local P = providers()
    if not P.Has("jobs") then return false, nil, Err.UNSUPPORTED end
    if not P.HasFn("jobs", op) then return false, nil, Err.NOT_IMPL end
    if type(p.job) ~= "string" or p.job == "" or #p.job > 255 then return false, nil, Err.BAD_ARG end
    local charId, _, werr = whoOf(a, p)
    if not charId then return false, nil, werr end

    local grade = p.grade
    if op == "setGrade" or grade ~= nil then
        grade = tonumber(grade)
        if not grade or grade < 0 or grade ~= math.floor(grade) then return false, nil, Err.BAD_ARG end
    end
    local label = p.label ~= nil and tostring(p.label) or nil

    local ok, v, perr
    if op == "add" then
        ok, v, perr = P.Call("jobs", "add", charId, p.job, grade or 0, label)
    elseif op == "remove" then
        ok, v, perr = P.Call("jobs", "remove", charId, p.job)
    else
        ok, v, perr = P.Call("jobs", "setGrade", charId, p.job, grade)
    end
    if not ok then return false, nil, perr or Err.FRAMEWORK_ERR end
    return true, v == nil and true or v, nil
end

-- ---------------------------------------------------------------------------
-- char.profile and char.list withJob
-- ---------------------------------------------------------------------------

--- payload { charId } -> ok, { charId, fullName, online, dob?, age?, gender,
--- nationality?, nickname?, description?, lastSeen? }. A key the framework
--- does not keep is absent, never guessed.
function Jobs.Profile(p)
    local a = adapter()
    if not a then return false, nil, Err.NOT_READY end
    if p.charId == nil or p.charId == "" then return false, nil, Err.BAD_ARG end
    if not a.charProfile then return false, nil, Err.UNSUPPORTED end
    local prof, err = a:charProfile(tostring(p.charId))
    if type(prof) ~= "table" then return false, nil, err or Err.NOT_FOUND end
    prof.charId   = tostring(prof.charId or p.charId)
    prof.gender   = PoggyCore.NormaliseGender(prof.gender)
    prof.lastSeen = Jobs.ToUnix(prof.lastSeen)
    prof.age      = tonumber(prof.age)
    for _, k in ipairs({ "dob", "nationality", "nickname", "description" }) do
        prof[k] = nz(prof[k]) ~= nil and tostring(prof[k]) or nil
    end
    prof.online = prof.online == true
    return true, prof, nil
end

--- char.list rows gain job, jobLabel, jobGrade, online (and src) when asked.
--- An online character's job is the live one, not the table's.
function Jobs.DecorateList(a, rows)
    if type(rows) ~= "table" then return rows end
    local online = Jobs.Online(a)
    local reg = Jobs.Registry(a)
    for _, r in ipairs(rows) do
        local o = online[tostring(r.charId)]
        r.online = o ~= nil
        r.src = o and o.src or nil
        if o then
            r.job, r.jobLabel, r.jobGrade = o.char.job, o.char.jobLabel, o.char.jobGrade
        end
        r.job = nz(r.job) or "unemployed"
        r.jobGrade = tonumber(r.jobGrade) or 0
        local def = reg[tostring(r.job):lower()]
        r.jobLabel = nz(r.jobLabel) or (def and def.label) or r.job
    end
    return rows
end

-- ---------------------------------------------------------------------------
-- Duty
-- ---------------------------------------------------------------------------

local BAG = "poggyDuty"

local function setBag(src, value)
    local ok, st = pcall(function() return Player(src).state end)
    if not ok or not st then return end
    local okGet, now = pcall(function() return st[BAG] end)
    if okGet and now == value then return end
    pcall(function() st:set(BAG, value, true) end)
end

function Jobs.DutyProvided()
    return providers().Has("duty")
end

--- True while the duty provider is answering for `src`: the caller then gets
--- the framework's own duty (see `busy` above).
function Jobs.DutyBusy(src)
    return busy["duty:" .. tostring(src)] ~= nil
end

--- The same for set: a provider whose set calls job.duty without raw sets
--- the framework's duty instead of calling itself again.
function Jobs.DutySetBusy(src)
    return busy["dutyset:" .. tostring(src)] ~= nil
end

--- The duty provider's answer for one player: true, false or nil (cannot
--- tell). The state bag follows it.
function Jobs.DutyOf(src)
    local ok, v, err = guarded("duty:" .. tostring(src), providers().Call, "duty", "get", src)
    local on
    if ok then
        if v == true then on = true elseif v == false then on = false end
    elseif err == nil then
        on = false   -- a plain `return false` from get
    end
    setBag(src, on)
    return on
end

--- Set duty through the provider. Returns ok, err.
function Jobs.SetDuty(src, onDuty)
    local ok, _, err = guarded("dutyset:" .. tostring(src), providers().Call, "duty", "set", src, onDuty and true or false)
    if not ok then return false, err or Err.FRAMEWORK_ERR end
    setBag(src, onDuty and true or false)
    return true
end

--- Server ids on duty through the provider, filtered by the worn job.
function Jobs.OnDutyList(a, list, minGrade)
    local P = providers()
    if P.HasFn("duty", "list") then
        local ok, v, err = P.Call("duty", "list", list, minGrade)
        if not ok then return nil, err end
        local out = {}
        for _, src in ipairs(type(v) == "table" and v or {}) do
            if tonumber(src) then out[#out + 1] = tonumber(src) end
        end
        return out
    end
    local out = {}
    for _, src in ipairs(a:getPlayers()) do
        if Jobs.DutyOf(src) == true then
            local match = true
            if list then
                local char = a:getChar(src)
                match = char ~= nil and PoggyCore.InList(char.job, list)
                    and (not minGrade or (tonumber(char.jobGrade) or 0) >= minGrade)
            end
            if match then out[#out + 1] = src end
        end
    end
    return out
end

--- A duty provider arrived: every online player's bag from its answer. It
--- left: every bag cleared, so the client falls back to the framework.
local function syncBags(_, resource)
    local a = adapter()
    if not a then return end
    for _, src in ipairs(a:getPlayers()) do
        if resource then Jobs.DutyOf(src) else setBag(src, nil) end
    end
end

if PoggyCore.Providers and PoggyCore.Providers.Watch then
    PoggyCore.Providers.Watch("duty", function(kind, resource)
        -- Off the caller's stack: the provider may still be finishing its start.
        CreateThread(function() syncBags(kind, resource) end)
    end)
end

-- A character loaded while a provider runs gets its bag at once.
AddEventHandler("poggy_core:charLoaded", function(src)
    if not Jobs.DutyProvided() then return end
    src = tonumber(src)
    if src then CreateThread(function() Jobs.DutyOf(src) end) end
end)

-- ---------------------------------------------------------------------------
-- Law
-- ---------------------------------------------------------------------------

local leo = { at = -1e9, set = {}, key = "" }

--- lower-cased name -> true for every job the framework types "leo", when
--- LeoJobsAreLaw is on. From the registry, so a minute old at most.
function Jobs.LeoJobs(force)
    if PoggyCoreConfig.LeoJobsAreLaw ~= true then return {} end
    local a = adapter()
    if not a then return {} end
    if not force and (os.time() - leo.at) < 60 then return leo.set end
    leo.at = os.time()
    local set, names = {}, {}
    if a.caps and a.caps["job.registry"] == true and a.jobsList then
        local ok, list = pcall(a.jobsList, a)
        if ok and type(list) == "table" then
            for _, j in ipairs(list) do
                if type(j) == "table" and j.name and tostring(j.type or ""):lower() == "leo" then
                    set[tostring(j.name):lower()] = true
                    names[#names + 1] = tostring(j.name)
                end
            end
        end
    end
    table.sort(names)
    leo.set = set
    local key = table.concat(names, ",")
    if key ~= leo.key then
        leo.key = key
        -- The client's job.isLaw reads this; replicated to every client.
        pcall(function() GlobalState.poggyLeoJobs = names end)
    end
    return set
end

--- Is `name` a law job? LawJobs, plus the framework's "leo" jobs when on.
function Jobs.IsLawJob(name)
    if type(name) ~= "string" or name == "" then return false end
    if PoggyCore.InList(name, PoggyCoreConfig.LawJobs or {}) then return true end
    return Jobs.LeoJobs()[name:lower()] == true
end

local leoLoop = false
AddEventHandler("poggy_core:ready", function()
    Jobs.DropRegistry()
    CreateThread(function() Jobs.LeoJobs(true) end)
    if leoLoop then return end   -- a re-resolve (poggycore resolve) fires ready again
    leoLoop = true
    CreateThread(function()
        -- RSG and QBCore can add jobs while running; look again now and then.
        while true do
            Wait(300000)
            Jobs.LeoJobs(true)
        end
    end)
end)

-- ---------------------------------------------------------------------------
-- law.report
-- ---------------------------------------------------------------------------

--- Forward a crime report to the law provider. The calling resource rides
--- along as `source`; nothing in the payload can name another one.
function Jobs.LawReport(p, resource)
    if type(p.event) ~= "string" or p.event == "" or #p.event > 128 then return false, nil, Err.BAD_ARG end
    local report = {
        event   = p.event,
        suspect = p.suspect,
        coords  = p.coords,
        extra   = p.extra,
        src     = tonumber(p.src),
        source  = resource,
        at      = os.time(),
    }
    return providers().Call("law", "report", report)
end

-- ---------------------------------------------------------------------------
-- Shared SQL for the citizenid frameworks (RSG, QBR, QBCore)
-- ---------------------------------------------------------------------------
-- players.charinfo and players.job are JSON there; the names are extracted in
-- SQL (MariaDB 10.2+ / MySQL 5.7+), as char.list already does.

local PJ = {}
PoggyCore.PlayersJson = PJ

PJ.FIRST     = "JSON_UNQUOTE(JSON_EXTRACT(charinfo, '$.firstname'))"
PJ.LAST      = "JSON_UNQUOTE(JSON_EXTRACT(charinfo, '$.lastname'))"
PJ.JOB_NAME  = "JSON_UNQUOTE(JSON_EXTRACT(job, '$.name'))"
PJ.JOB_LABEL = "JSON_UNQUOTE(JSON_EXTRACT(job, '$.label'))"
PJ.JOB_GRADE = "JSON_EXTRACT(job, '$.grade.level')"

--- The job table PlayerData keeps (or the column holds), as a worn entry.
function PJ.Entry(job)
    job = type(job) == "table" and job or {}
    local grade = type(job.grade) == "table" and job.grade or {}
    return {
        name = nz(job.name) or "unemployed", label = job.label,
        grade = tonumber(grade.level) or 0, gradeLabel = grade.name,
        active = true, source = "framework",
    }
end

local function marks(n)
    local m = {}
    for i = 1, n do m[i] = "?" end
    return table.concat(m, ", ")
end
PJ.Marks = marks

--- The worn job of a character, live when online.
function PJ.JobsOf(a, charId, src)
    if src then
        local _, pd = a:raw(src)
        if pd then return { PJ.Entry(pd.job) } end
    end
    local rows, err = Util.DbQuery("SELECT job FROM players WHERE citizenid = ? LIMIT 1", { tostring(charId) })
    if not rows then return nil, err end
    if not rows[1] then return nil, Err.NOT_FOUND end
    return { PJ.Entry(Jobs.DecodeJson(rows[1].job)) }
end

--- Everyone wearing one of `names`: offline from the table, online live.
function PJ.Holders(a, names, online)
    local want = {}
    for _, n in ipairs(names) do want[n:lower()] = true end
    local sql = ("SELECT citizenid, %s AS firstname, %s AS lastname, job FROM players WHERE %s IN (%s) LIMIT %d")
        :format(PJ.FIRST, PJ.LAST, PJ.JOB_NAME, marks(#names), HOLDERS_CAP)
    local rows, err = Util.DbQuery(sql, names)
    if not rows then return nil, err end

    local out = {}
    for _, r in ipairs(rows) do
        local cid = tostring(r.citizenid)
        if not online[cid] then
            local e = PJ.Entry(Jobs.DecodeJson(r.job))
            if want[e.name:lower()] then
                e.charId, e.firstName, e.lastName, e.job = cid, r.firstname, r.lastname, e.name
                out[#out + 1] = e
            end
        end
    end
    for cid, o in pairs(online) do
        local _, pd = a:raw(o.src)
        if pd then
            local e = PJ.Entry(pd.job)
            if want[e.name:lower()] then
                e.charId, e.job = cid, e.name
                out[#out + 1] = e
            end
        end
    end
    return out
end

--- charinfo, and players.last_updated when the table has it.
function PJ.Profile(a, charId)
    local id = tostring(charId)
    local info, lastSeen, online
    local live = a:getCharByCharId(id)
    if live and live.source then
        local _, pd = a:raw(live.source)
        info = pd and type(pd.charinfo) == "table" and pd.charinfo or {}
        lastSeen, online = os.time(), true
    else
        local hasSeen = Util.DbColumnExists("players", "last_updated")
        local rows, err = Util.DbQuery("SELECT charinfo" .. (hasSeen and ", last_updated" or "")
            .. " FROM players WHERE citizenid = ? LIMIT 1", { id })
        if not rows then return nil, err end
        if not rows[1] then return nil, Err.NOT_FOUND end
        info = Jobs.DecodeJson(rows[1].charinfo)
        lastSeen = hasSeen and rows[1].last_updated or nil
    end
    return {
        charId      = id,
        firstName   = info.firstname or "",
        lastName    = info.lastname or "",
        fullName    = ((info.firstname or "") .. " " .. (info.lastname or "")):gsub("^%s+", ""),
        dob         = nz(info.birthdate),
        gender      = info.gender,
        nationality = nz(info.nationality),
        nickname    = nz(info.nickname),
        description = nz(info.description),
        lastSeen    = lastSeen,
        online      = online == true,
    }
end
