--[[
    poggy_core — diagnostics.

    /poggycore         status: framework, version, capabilities, registrations
    /poggycore test    read-only smoke test against the calling player
    /poggycore caps    the full capability map
    /poggycore verbs   the verb contract every script calls through
    /poggycore do      run one verb for real, e.g. do money.get src=me
    /poggycore detect  why the framework did or did not resolve
    /poggycore resolve re-run detection without restarting
    /poggycore selftest [full]  exercise every verb and report pass/fail
    /poggycore scripts registered Poggy scripts: poggy_id, folder (when different), version
    /poggycore usables usable items registered through poggy_core: item -> script
    /poggycore dependents  resources that stop with poggy_core, with their state (sv_dependents.lua)
    /poggycore catalog     published Poggy scripts this server does not have (sv_updates.lua)
    /poggycore update <resource|all> [check|stage|apply|writetest] [force]
                       GitHub updates, gated on the fxmanifest version (sv_updates.lua)
    /poggycore settings [show <id> | set <id> <path> <json> | unlock <id>]
                       the settings hub (/poggy) from the console (sv_hub.lua);
                       set and unlock run from the server console only

    Admin-gated through the adapter's own permission model. Available from the
    server console too, where source is 0 and the gate is skipped.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util

local function reply(src, msg)
    if src == 0 then
        print((PoggyCore.PREFIX or "^5[Poggy Core]^7 ") .. msg)
    else
        TriggerClientEvent("chat:addMessage", src, {
            color = { 243, 231, 216 }, args = { "poggy_core", msg },
        })
    end
end

--- Who may run diagnostics.
---
--- Order matters here. The adapter's own permission model is the right answer
--- when it works, but it is also the thing most likely to be broken when
--- somebody reaches for diagnostics — and a diagnostic command gated behind the
--- subsystem it diagnoses is useless exactly when it is needed. So ACE comes
--- first: it is the server's own permission system, it does not depend on a
--- framework being up, and anyone in group.admin already has it.
local function allowed(src)
    if src == 0 then return true end                       -- server console
    if IsPlayerAceAllowed(src, "command") then return true end
    if IsPlayerAceAllowed(src, "poggycore") then return true end

    local ok, isAdmin = pcall(function()
        return PoggyCore.Self().Perms.IsAdmin(src)
    end)
    return ok and isAdmin or false
end

local function status(src)
    local S = PoggyCore.State
    reply(src, ("v%s — framework: %s (%s)%s"):format(
        PoggyCore.VERSION,
        S.framework,
        S.adapter and S.adapter.label or "none",
        S.noAdapter and "  ^1NO ADAPTER — every framework call refuses^7" or ""))

    local started = PoggyCore.StartedFrameworks()
    reply(src, ("framework cores running: %s"):format(
        #started > 0 and table.concat(started, ", ") or "none"))

    local caps = {}
    for k, v in pairs(S.adapter and S.adapter.caps or {}) do
        if v == true then caps[#caps + 1] = k end
    end
    table.sort(caps)
    reply(src, ("capabilities: %d — %s"):format(#caps,
        #caps > 0 and table.concat(caps, ", ") or "none"))

    local containers, byResource = 0, {}
    for _, entry in pairs(PoggyCore.StorageRegistry or {}) do
        containers = containers + 1
        byResource[entry.resource] = (byResource[entry.resource] or 0) + 1
    end
    local parts = {}
    for res, n in pairs(byResource) do parts[#parts + 1] = ("%s(%d)"):format(res, n) end
    reply(src, ("containers registered: %d%s"):format(containers,
        #parts > 0 and (" — " .. table.concat(parts, ", ")) or ""))

    if PoggyCore.Usables then
        local n, byRes = PoggyCore.Usables.Count()
        local uparts = {}
        for res, c in pairs(byRes) do uparts[#uparts + 1] = ("%s(%d)"):format(res, c) end
        table.sort(uparts)
        reply(src, ("usable items registered: %d%s — poggycore usables"):format(n,
            #uparts > 0 and (" — " .. table.concat(uparts, ", ")) or ""))
    end

    if PoggyCore.Identity then
        local list, dup = PoggyCore.Identity.List()
        reply(src, ("Poggy scripts registered: %d%s — poggycore scripts"):format(#list,
            #dup > 0 and ("  ^1%d poggy_id(s) declared by two folders^7"):format(#dup) or ""))
    end
end

--- Every Poggy script that has registered (core.register from its bridge).
local function scripts(src)
    local I = PoggyCore.Identity
    if not I then
        reply(src, "script identity is not loaded.")
        return
    end
    local list, dup = I.List()
    reply(src, ("%d Poggy script(s) registered"):format(#list))
    for _, e in ipairs(list) do
        reply(src, ("  %-24s v%-9s %s"):format(e.id, e.version or "?",
            e.folder ~= e.id and ("^9folder: " .. e.folder .. "^7") or ""))
    end
    for _, d in ipairs(dup) do
        reply(src, ("  ^1❌ poggy_id '%s' is declared by %s: the second was refused, and updates skip it^7")
            :format(d.id, table.concat(d.folders, " and ")))
    end
    reply(src, "^9a script registers when its bridge is ready; a stopped script is not listed^7")
end

--- Every usable item registered through poggy_core, and which script owns it.
local function usables(src)
    local U = PoggyCore.Usables
    if not U then
        reply(src, "the usable-item registry is not loaded.")
        return
    end
    local list = U.List()
    reply(src, ("%d usable item(s) registered through poggy_core"):format(#list))
    for _, e in ipairs(list) do
        reply(src, ("  %-28s %s"):format(e.item, e.resource))
    end
    local S = PoggyCore.State
    local invRes = S.adapter and S.adapter.inventoryResource
    if invRes then
        reply(src, ("^9re-registered on their own when %s restarts; a stopped script's items are dropped^7"):format(invRes))
    else
        reply(src, "^9this framework has no inventory resource to watch^7")
    end
end

local function smokeTest(src)
    if src == 0 then
        reply(src, "the smoke test needs a player; run it in game.")
        return
    end

    local Self = PoggyCore.Self()
    reply(src, "running read-only smoke test...")

    local char = Self.GetChar(src)
    if not char then
        reply(src, "FAIL GetChar returned nil. Is your character loaded?")
        return
    end
    reply(src, ("PASS GetChar — charId=%s owner=%s name=%s"):format(
        char.charId, char.ownerId ~= "" and "set" or "EMPTY", char.fullName))
    reply(src, ("     job=%s grade=%d label=%s onDuty=%s"):format(
        char.job, char.jobGrade, char.jobLabel, tostring(char.onDuty)))

    for _, cur in ipairs({ "cash", "bank", "gold", "rol" }) do
        if Self.Money.Supports(cur) then
            local v = Self.Money.Get(src, cur)
            reply(src, ("PASS money.%s = %s"):format(cur, tostring(v)))
        else
            reply(src, ("SKIP money.%s unsupported on %s"):format(cur, PoggyCore.State.framework))
        end
    end

    local items = Self.Inventory.Get(src)
    reply(src, ("PASS Inventory.Get — %d stack(s)"):format(#items))

    if #items > 0 then
        local first = items[1]
        local n = Self.Inventory.Count(src, first.name)
        reply(src, ("%s Inventory.Count('%s') = %d (list said %d)"):format(
            n == first.amount and "PASS" or "WARN", first.name, n, first.amount))
    end

    local can, why = Self.Inventory.CanCarry(src, items[1] and items[1].name or "water", 1)
    reply(src, ("PASS CanCarry = %s%s"):format(tostring(can), why and (" (" .. why .. ")") or ""))

    reply(src, ("PASS Perms.GetGroup = %s, IsAdmin = %s"):format(
        Self.Perms.GetGroup(src), tostring(Self.Perms.IsAdmin(src))))

    reply(src, "smoke test complete. Nothing was modified.")
end

-- ---------------------------------------------------------------------------
-- Detection diagnosis
--
-- "No framework detected" is not an answer, it is a symptom. This runs each
-- probe again, right now, and reports what came back — so the difference
-- between "the resource is not started", "the export is missing" and "the
-- export returned the wrong shape" is visible instead of inferred.
-- ---------------------------------------------------------------------------

local function describeProbe(id)
    local def = PoggyCore.Frameworks[id]
    local state = GetResourceState(def.resource)

    if state ~= "started" then
        return ("%-10s %-16s resource state: %s"):format(id, def.resource, state)
    end

    -- Call the probe exactly as detection does, but keep the error.
    local ok, result = pcall(def.probe)
    if not ok then
        return ("%-10s %-16s started, probe ERRORED: %s"):format(id, def.resource, tostring(result))
    end
    if result == nil then
        -- Started, export reachable or not, but the handshake said no. For VORP
        -- the usual cause is the export answering with something that has no
        -- getUser, so check that directly and say which it was.
        local ok2, raw = pcall(function() return exports[def.resource]:GetCore() end)
        if not ok2 then
            return ("%-10s %-16s started, exports.%s:GetCore() failed: %s")
                :format(id, def.resource, def.resource, tostring(raw))
        end
        return ("%-10s %-16s started, GetCore returned %s, getUser is %s")
            :format(id, def.resource, type(raw),
                type(raw) == "table" and type(raw.getUser) or "n/a")
    end
    return ("%-10s %-16s started, probe OK (%s)"):format(id, def.resource, type(result))
end

local function detect(src)
    local S = PoggyCore.State
    local L = PoggyCore.LastResolve or {}

    reply(src, ("poggy_core v%s"):format(PoggyCore.VERSION))
    reply(src, ("resolved to: %s   ready=%s  noAdapter=%s  adapter=%s"):format(
        S.framework, tostring(S.ready), tostring(S.noAdapter),
        S.adapter and S.adapter.label or "none"))
    reply(src, ("detection attempts on last resolve: %d%s"):format(
        L.attempts or 0,
        L.adapterError and ("   adapter init error: " .. L.adapterError) or ""))
    reply(src, ("ForceFramework=%s  timeout=%sms  order=%s"):format(
        tostring(PoggyCoreConfig.ForceFramework),
        tostring(PoggyCoreConfig.FrameworkTimeout),
        table.concat(PoggyCoreConfig.DetectionOrder, ", ")))
    reply(src, "probing each framework now:")

    for _, id in ipairs(PoggyCoreConfig.DetectionOrder) do
        reply(src, "  " .. describeProbe(id))
    end

    reply(src, "run '/poggycore resolve' to re-run detection without restarting.")
end

-- ---------------------------------------------------------------------------
-- Verb tooling
--
-- `verbs` lists the contract. `do` runs one for real, which is how the bridge
-- gets tested without writing a throwaway script for every check.
-- ---------------------------------------------------------------------------

local function listVerbs(src, filter)
    local names, shown = PoggyCore.VerbNames(), 0
    for _, name in ipairs(names) do
        if not filter or name:find(filter, 1, true) then
            local spec = PoggyCore.Verbs[name]
            local need = #spec.args > 0 and table.concat(spec.args, ", ") or "-"
            reply(src, ("%-20s %-7s %-28s -> %s%s"):format(name, spec.side, need,
                spec.returns or "?", spec.yields and "   ^3[needs a thread]^7" or ""))
            shown = shown + 1
        end
    end
    reply(src, ("%d verb(s)%s of %d declared."):format(
        shown, filter and (" matching '" .. filter .. "'") or "", #names))
end

--- Render a returned value so a table is actually readable in chat.
local function show(v, depth)
    depth = depth or 0
    if type(v) ~= "table" then return tostring(v) end
    if depth > 1 then return "{...}" end
    local parts = {}
    for k, val in pairs(v) do
        parts[#parts + 1] = ("%s=%s"):format(tostring(k), show(val, depth + 1))
        if #parts >= 12 then parts[#parts + 1] = "..."; break end
    end
    table.sort(parts)
    return "{ " .. table.concat(parts, ", ") .. " }"
end

--- key=value arguments, coerced. `src=me` means the caller.
local function parsePayload(src, args, from)
    local p = {}
    for i = from, #args do
        local k, v = args[i]:match("^([%w_]+)=(.*)$")
        if not k then
            reply(src, ("ignoring '%s' — arguments look like key=value."):format(args[i]))
        else
            if v == "me" and k == "src" then v = src
            elseif v == "true"  then v = true
            elseif v == "false" then v = false
            elseif tonumber(v)  then v = tonumber(v)
            end
            p[k] = v
        end
    end
    return p
end

local function runVerb(src, args)
    local verb = args[2]
    if not verb then
        reply(src, "usage: /poggycore do <verb> [key=value ...]   e.g. do money.get src=me")
        return
    end

    local payload = parsePayload(src, args, 3)
    reply(src, ("-> %s %s"):format(verb, show(payload)))

    -- Straight through the dispatcher, so validation and side checks are part
    -- of what gets tested rather than being bypassed by the diagnostic.
    local ok, value, err = PoggyCore.Do(verb, payload)

    if ok then
        reply(src, ("^2ok^7  value = %s"):format(show(value)))
    else
        reply(src, ("^1failed^7  err = %s"):format(tostring(err)))
    end
end

RegisterCommand("poggycore", function(src, args)
    if not allowed(src) then
        reply(src, "you need admin for that. Add yourself with add_principal in server.cfg, "
                .. "or run it from the server console.")
        return
    end

    local sub = (args[1] or "status"):lower()

    if sub == "selftest" then
        CreateThread(function()
            PoggyCore.SelfTest(src, (args[2] or ""):lower() == "full",
                function(msg) reply(src, msg) end)
        end)
    elseif sub == "update" then
        -- Writes files, so console only: a chat command should not be able to
        -- replace server code, whatever ACE the player holds.
        if src ~= 0 then
            reply(src, "update runs from the server console only.")
            return
        end
        if (args[2] or ""):lower() == "nettest" then
            local netOpts = {}
            for i = 3, #args do
                local k, v = args[i]:match("^([%w_]+)=(.+)$")
                if k then netOpts[k] = v end
            end
            CreateThread(function()
                PoggyCore.Updates.NetTest(function(msg) reply(src, msg) end, netOpts)
            end)
            return
        end
        local mode, opts = "check", {}
        for i = 3, #args do
            local k, v = args[i]:match("^([%w_]+)=(.+)$")
            local word = args[i]:lower()
            if k then
                opts[k] = v
            elseif word == "force" then
                opts.force = true
            else
                mode = word
            end
        end
        if mode == "writetest" then
            CreateThread(function()
                PoggyCore.Updates.WriteTest(args[2], function(msg) reply(src, msg) end)
            end)
            return
        end
        if mode ~= "check" and mode ~= "stage" and mode ~= "apply" then
            reply(src, "mode must be check, stage, apply or writetest.")
            return
        end
        CreateThread(function()
            PoggyCore.Updates.Run(args[2], mode, opts, function(msg) reply(src, msg) end)
        end)
    elseif sub == "sql" then
        -- Changes the database, so console only, like update.
        if src ~= 0 then
            reply(src, "sql runs from the server console only.")
            return
        end
        local mode = (args[2] or ""):lower()
        if mode ~= "check" and mode ~= "install" then
            reply(src, "usage: poggycore sql check <resource|all>   (shows what would change)")
            reply(src, "       poggycore sql install <resource|all> (runs it now)")
            return
        end
        CreateThread(function()
            PoggyCore.Sql.Command(mode, args[3], function(msg) reply(src, msg) end)
        end)
    elseif sub == "settings" then
        if not PoggyCore.Hub or not PoggyCore.Hub.Command then
            reply(src, "the settings hub is not loaded.")
            return
        end
        local rest = {}
        for i = 2, #args do rest[#rest + 1] = args[i] end
        CreateThread(function()
            PoggyCore.Hub.Command(src, rest, function(msg) reply(src, msg) end)
        end)
    elseif sub == "scripts" then
        CreateThread(function() scripts(src) end)
    elseif sub == "dependents" then
        if not PoggyCore.Dependents then
            reply(src, "the dependents list is not loaded.")
            return
        end
        CreateThread(function() PoggyCore.Dependents.Command(function(msg) reply(src, msg) end) end)
    elseif sub == "catalog" or sub == "catalogue" then
        -- Reads the feed when no run has loaded it yet, so it needs a thread.
        CreateThread(function()
            PoggyCore.Updates.Catalog(function(msg) reply(src, msg) end, { force = true })
        end)
    elseif sub == "usables" then
        CreateThread(function() usables(src) end)
    elseif sub == "detect" then
        CreateThread(function() detect(src) end)
    elseif sub == "resolve" then
        reply(src, "re-running framework detection...")
        CreateThread(function()
            PoggyCore.Resolve()
            detect(src)
        end)
    elseif sub == "verbs" then
        CreateThread(function() listVerbs(src, args[2]) end)
    elseif sub == "do" then
        CreateThread(function() runVerb(src, args) end)
    elseif sub == "test" then
        CreateThread(function() smokeTest(src) end)
    elseif sub == "caps" then
        local S = PoggyCore.State
        local keys = {}
        for k in pairs(PoggyCore.Caps) do keys[#keys + 1] = PoggyCore.Caps[k] end
        table.sort(keys)
        for _, cap in ipairs(keys) do
            reply(src, ("%-24s %s"):format(cap,
                (S.adapter and S.adapter.caps[cap]) and "yes" or "no"))
        end
    else
        CreateThread(function() status(src) end)
    end
end, false)

-- The command list used to be printed here on its own line; it is now part of
-- the start banner (sv_core.lua), so the console shows one block, not two.
