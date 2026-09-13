--- poggy_core — the single entry point every Poggy script uses (server side).
---
--- One export. One call shape. Everything a script needs goes through it:
---
---     local ok, value, err = exports.poggy_core:Do(verb, payload)
---
--- Scripts do not call this directly — they call Poggy(), the one-line bridge
--- vendored into each resource, which forwards to here. The point of the
--- indirection is that a script's copy of that bridge never has to change:
--- adding a capability means adding a verb here, and nothing gets re-uploaded
--- to Cfx.
---
--- Contract, the same for every verb:
---     ok     boolean  did the thing happen
---     value  any      the result, or nil
---     err    string   why not, from PoggyCore.Err, or nil
---
--- An unknown verb, a verb called on the wrong side, or a payload missing a
--- required key all fail loudly rather than quietly doing nothing. A typo is a
--- bug, and a bug that says nothing is the expensive kind.

local Err = PoggyCore.Err
local tag = "dispatch"

-- ---------------------------------------------------------------------------
-- Handlers
--
-- Each takes (Core, p) and returns ok, value, err. They exist to flatten the
-- Core API's various return shapes into the one contract above — some Core
-- methods return `value, err`, some `ok, err`, and Job.Get returns five values.
-- ---------------------------------------------------------------------------

local H = {}

local function ret(value, err)          -- for reads: nil means it failed
    if value == nil then return false, nil, err or Err.NOT_FOUND end
    return true, value, nil
end

local function did(ok, err)             -- for writes
    if ok then return true, true, nil end
    return false, nil, err or Err.FRAMEWORK_ERR
end

-- database (0.12.0) ----------------------------------------------------------
-- Always the calling resource's own sql/ folder: the resource comes from the
-- dispatcher, never from the payload, so one script cannot run another's files.

H["sql.install"] = function(C, p, resource) return PoggyCore.Sql.Verb(resource, false) end
H["sql.check"]   = function(C, p, resource) return PoggyCore.Sql.Verb(resource, true) end

-- identity (0.13.0) ----------------------------------------------------------
-- The bridge registers its script. The folder is the calling resource from the
-- dispatcher; a payload cannot name another one (server/sv_identity.lua).

H["core.register"] = function(C, p, resource) return PoggyCore.Identity.Verb(resource, p) end

-- character ------------------------------------------------------------------

H["char.get"]     = function(C, p) return ret(C.GetChar(p.src)) end
H["char.byId"]    = function(C, p) return ret(C.GetCharByCharId(p.charId)) end
H["char.id"]      = function(C, p) return ret(C.GetCharId(p.src)) end
H["players.list"] = function(C)     return ret(C.GetPlayers()) end

-- 0.11.0
H["char.offline"]   = function(C, p) return ret(C.GetCharOffline(p.charId, p.appearance)) end
H["char.list"]      = function(C, p) return ret(C.ListChars({ search = p.search, limit = p.limit, offset = p.offset })) end
H["players.onDuty"] = function(C, p) return ret(C.GetPlayersOnDuty(p.job, p.minGrade)) end

-- money ----------------------------------------------------------------------

H["money.get"]      = function(C, p) return ret(C.Money.Get(p.src, p.currency)) end
H["money.add"]      = function(C, p) return did(C.Money.Add(p.src, p.currency, p.amount, p.reason)) end
H["money.remove"]   = function(C, p) return did(C.Money.Remove(p.src, p.currency, p.amount, p.reason)) end
H["money.set"]      = function(C, p) return did(C.Money.Set(p.src, p.currency, p.amount, p.reason)) end
H["money.supports"] = function(C, p) return true, C.Money.Supports(p.currency) and true or false, nil end

-- jobs -----------------------------------------------------------------------

H["job.get"] = function(C, p)
    local name, grade, label, gradeLabel, onDuty = C.Job.Get(p.src)
    if name == nil then return false, nil, Err.NO_CHAR end
    return true, {
        name = name, grade = grade, label = label,
        gradeLabel = gradeLabel, onDuty = onDuty,
    }, nil
end

H["job.set"]       = function(C, p) return did(C.Job.Set(p.src, p.job, p.grade, p.label, p.persist)) end
H["job.duty"]      = function(C, p) return did(C.Job.SetDuty(p.src, p.onDuty)) end
H["job.has"]       = function(C, p) return true, C.Job.Has(p.src, p.job, p.minGrade) and true or false, nil end
H["job.isLaw"]     = function(C, p) return true, C.Job.IsLaw(p.src) and true or false, nil end
H["job.isMedical"] = function(C, p) return true, C.Job.IsMedical(p.src) and true or false, nil end

-- inventory ------------------------------------------------------------------

H["inv.add"]     = function(C, p) return did(C.Inventory.Add(p.src, p.item, p.qty or 1, p.meta)) end
H["inv.remove"]  = function(C, p) return did(C.Inventory.Remove(p.src, p.item, p.qty or 1, p.meta)) end
H["inv.count"]   = function(C, p) return true, C.Inventory.Count(p.src, p.item, p.meta) or 0, nil end
H["inv.has"]     = function(C, p) return true, C.Inventory.Has(p.src, p.item, p.qty or 1) and true or false, nil end
H["inv.get"]     = function(C, p) return ret(C.Inventory.Get(p.src)) end
H["inv.setMeta"] = function(C, p) return did(C.Inventory.SetMeta(p.src, p.itemId, p.meta, p.amount)) end

-- 0.11.0
H["inv.items"]     = function(C, p) return ret(C.Inventory.Items({ search = p.search, limit = p.limit, checkImages = p.checkImages })) end
H["inv.itemInfo"]  = function(C, p) return ret(C.Inventory.ItemInfo(p.item, p.checkImages)) end
H["inv.imageBase"] = function(C)    return ret(C.Inventory.ImageBase()) end
H["inv.close"]     = function(C, p) return did(C.Inventory.Close(p.src)) end

H["inv.canCarry"] = function(C, p)
    -- Returns boolean plus a reason when it is false; the reason rides in err
    -- so the caller can show it without a second call.
    local can, why = C.Inventory.CanCarry(p.src, p.item, p.qty or 1)
    return true, can and true or false, (not can) and why or nil
end

-- weapons --------------------------------------------------------------------

H["weapon.add"]    = function(C, p) return did(C.Weapons.Add(p.src, p.weapon, p.ammo, p.components)) end
H["weapon.remove"] = function(C, p) return did(C.Weapons.Remove(p.src, p.weaponId)) end
H["weapon.get"]    = function(C, p) return ret(C.Weapons.Get(p.src)) end

H["weapon.canCarry"] = function(C, p)
    -- Same shape as inv.canCarry: the reason for a false rides in err.
    local can, why = C.Weapons.CanCarry(p.src, p.qty or 1, p.weapon)
    return true, can and true or false, (not can) and why or nil
end

-- storage --------------------------------------------------------------------

H["storage.register"]   = function(C, p) return did(C.Storage.Register(p.id, p.opts)) end
H["storage.registered"] = function(C, p) return true, C.Storage.IsRegistered(p.id) and true or false, nil end
H["storage.open"]       = function(C, p) return did(C.Storage.Open(p.src, p.id)) end
H["storage.close"]      = function(C, p) return did(C.Storage.Close(p.src, p.id)) end
H["storage.addItem"]    = function(C, p) return did(C.Storage.AddItem(p.id, p.item, p.qty or 1, p.meta, p.charId)) end
H["storage.removeItem"] = function(C, p) return did(C.Storage.RemoveItem(p.id, p.item, p.qty or 1, p.meta)) end
H["storage.items"]      = function(C, p) return ret(C.Storage.GetItems(p.id)) end
H["storage.capacity"]   = function(C, p) return did(C.Storage.SetCapacity(p.id, p.slots, p.maxWeight)) end
H["storage.unregister"] = function(C, p) return did(C.Storage.Unregister(p.id)) end
H["storage.delete"]     = function(C, p) return did(C.Storage.Delete(p.id)) end
H["storage.rawIds"]     = function(C, p) return did(C.Storage.UseRawIds(p.enabled)) end
H["storage.weapons"]    = function(C, p) return ret(C.Storage.GetWeapons(p.id)) end

-- notifications --------------------------------------------------------------

H["notify"] = function(_, p)
    PoggyCore.Notify(p.src, p.text, p.kind, p.duration)
    return true, true, nil
end

H["notify.rich"] = function(_, p)
    PoggyCore.NotifyRich(p.src, {
        title = p.title, description = p.description, kind = p.kind,
        duration = p.duration, dict = p.dict, icon = p.icon, color = p.color,
    })
    return true, true, nil
end

H["notify.styled"] = function(_, p)
    if not PoggyCore.IsNotifyStyle(p.style) then return false, nil, Err.BAD_ARG end
    PoggyCore.NotifyStyled(p.src, p.style, {
        -- Pass the whole styled arg set through. Dropping dict/icon/color
        -- silently stripped the icon off every 'advanced' notification.
        text        = p.text,
        title       = p.title,
        subtitle    = p.subtitle,
        duration    = p.duration,
        dict        = p.dict,
        icon        = p.icon,
        color       = p.color,
        location    = p.location,
        audioRef    = p.audioRef,
        audioName   = p.audioName,
        quality     = p.quality,
        showQuality = p.showQuality,
    })
    return true, true, nil
end

-- ui (0.14.0) ----------------------------------------------------------------
-- The screens live on the client; here `src` names whose. The two that wait
-- are declared yields, so the thread guard in dispatch() covers them; the
-- round trip, its own longer timeout and the dropped-player case are in
-- server/sv_ui.lua.

H["menu.open"] = function(_, p)
    return PoggyCore.Ui.Menu(p.src, {
        title = p.title, items = p.items, subtitle = p.subtitle,
        cursor = p.cursor, closeText = p.closeText,
    })
end

H["menu.close"] = function(_, p) return did(PoggyCore.Ui.Close(p.src)) end

H["input.text"] = function(_, p)
    return PoggyCore.Ui.Input(p.src, {
        title = p.title, placeholder = p.placeholder, default = p.default,
        maxLength = p.maxLength, numeric = p.numeric, submitText = p.submitText,
        cursor = p.cursor,
    })
end

-- permissions ----------------------------------------------------------------

H["inv.registerUsable"] = function(C, p, resource)
    if not PoggyCore.IsCallable(p.fn) then return false, nil, Err.BAD_ARG end
    -- The owner is the caller, from the dispatcher — not C, which is poggy_core's
    -- own Core. The registry drops a script's items when it stops and replays
    -- them after an inventory restart, so the attribution has to be right.
    return did(PoggyCore.Usables.Register(resource, p.item, p.fn))
end

H["callback.register"] = function(C, p)
    if not PoggyCore.IsCallable(p.fn) then return false, nil, Err.BAD_ARG end
    return did(C.Callback.Register(p.name, p.fn))
end

H["perms.group"]   = function(C, p) return ret(C.Perms.GetGroup(p.src)) end
H["perms.isAdmin"] = function(C, p) return true, C.Perms.IsAdmin(p.src) and true or false, nil end

H["perms.groups"] = function(C, p)
    local seen, out = {}, {}
    local function add(g)
        g = tostring(g or ""):lower()
        if g ~= "" and not seen[g] then
            seen[g] = true
            out[#out + 1] = g
        end
    end

    add(C.Perms.GetGroup(p.src))

    local ch = C.GetChar(p.src)
    if ch then add(ch.group) end

    -- The framework's own user object can carry a different group than the
    -- character record. Scripts used to reach for Core.Native() to read it,
    -- which meant every one of them handling a raw framework object. It is
    -- cheaper and safer to do it once, here.
    local native = C.Native()
    if native and PoggyCore.IsCallable(native.getUser) then
        local ok, u = pcall(native.getUser, p.src)
        if ok and u then add(u.getGroup) end
    end

    -- An adapter that can list every group a player holds (RSG: the ACE
    -- permission levels, where an admin also holds mod and helper) adds them
    -- all; the first is the one perms.group already returned.
    local a = PoggyCore.State and PoggyCore.State.adapter
    if a and a.permGroups then
        local ok, groups = pcall(a.permGroups, a, p.src)
        if ok and type(groups) == "table" then
            for _, g in ipairs(groups) do add(g) end
        end
    end

    return true, out, nil
end

-- core -----------------------------------------------------------------------

H["core.ready"]     = function(C) return true, (C.IsReady() and C.HasAdapter()) and true or false, nil end
H["core.version"]   = function()  return true, PoggyCore.VERSION, nil end
H["core.has"]       = function(C, p) return true, C.Has(p.capability) and true or false, nil end
H["core.caps"]      = function(C) return ret(C.CapsSnapshot and C.CapsSnapshot() or C.Caps) end

H["core.framework"] = function(C)
    -- GetFramework() returns the id only. Destructuring a second value out of
    -- it left every caller with a nil label, which is why the boot line read
    -- "(no framework)" on a server that had resolved VORP perfectly well.
    local id  = C.GetFramework()
    local def = PoggyCore.Frameworks[id]
    local S   = PoggyCore.State
    return true, {
        id    = id,
        label = (S.adapter and S.adapter.label) or (def and def.label) or id,
    }, nil
end

-- ---------------------------------------------------------------------------
-- The entry point
-- ---------------------------------------------------------------------------

--- Run one verb. Returns ok, value, err.
--- @param resource string the calling resource, for log lines that name a culprit
local function dispatch(resource, verb, payload)
    if type(verb) ~= "string" then
        PoggyCore.Util.Warn("dispatch: %s called Poggy() with a %s instead of a verb name.",
            resource, type(verb))
        return false, nil, Err.BAD_ARG
    end

    local allowed, why = PoggyCore.VerbAllowed(verb, "server")
    if not allowed then
        if why == "unknown_verb" then
            PoggyCore.Util.Warn("dispatch: %s asked for '%s', which is not a verb. See /poggycore verbs.",
                resource, verb)
        else
            PoggyCore.Util.Warn("dispatch: %s called '%s' on the server, but it is client-side.",
                resource, verb)
        end
        return false, nil, why
    end

    local okArgs, missing = PoggyCore.VerbCheck(verb, payload)
    if not okArgs then
        PoggyCore.Util.Warn("dispatch: %s called '%s' without '%s'.", resource, verb, tostring(missing))
        return false, nil, Err.BAD_ARG
    end

    local handler = H[verb]
    if not handler then
        -- The verb table and this file disagree. That is our bug, not theirs.
        PoggyCore.Util.Warn("dispatch: '%s' is declared in sh_verbs.lua but has no handler here.", verb)
        return false, nil, Err.NOT_IMPL
    end

    -- Several verbs wait on a framework callback, so they can only run on a
    -- coroutine. Off one, Util.Await refuses and the verb returns a plain
    -- false, which a caller cannot tell apart from "poggy_core is not here" —
    -- so it quietly falls back forever. Say what actually happened instead.
    local spec = PoggyCore.Verbs[verb]
    if spec and spec.yields and not PoggyCore.Util.CanYield() then
        PoggyCore.Util.Warn("dispatch: %s called '%s' outside a thread. It waits on the "
            .. "framework, so wrap the call in CreateThread(function() ... end).",
            resource, verb)
        return false, nil, Err.NEEDS_THREAD
    end

    local Core = PoggyCore.Self()
    if not Core.IsReady() then return false, nil, Err.NOT_READY end
    if not Core.HasAdapter() then return false, nil, Err.UNSUPPORTED end

    -- The calling resource rides along as a third argument. Only handlers that
    -- act on the caller itself (sql.*, core.register) read it; a payload cannot fake it.
    local ok, a, b, c = pcall(handler, Core, payload or {}, resource)
    if not ok then
        PoggyCore.Util.Warn("dispatch: '%s' from %s errored: %s", verb, resource, tostring(a))
        return false, nil, Err.FRAMEWORK_ERR
    end
    return a, b, c
end

--- The one export. Everything a Poggy script needs comes through here.
exports("Do", function(verb, payload)
    return dispatch(GetInvokingResource() or "poggy_core", verb, payload)
end)

--- Same thing for poggy_core's own files and for the console command.
function PoggyCore.Do(verb, payload)
    return dispatch("poggy_core", verb, payload)
end

-- ---------------------------------------------------------------------------
-- Startup self-check
--
-- Every declared verb must have a handler, and every handler must be declared.
-- Catching that here turns a customer's silent no-op into our console error.
-- ---------------------------------------------------------------------------

CreateThread(function()
    local missingHandler, undeclared = {}, {}
    for _, name in ipairs(PoggyCore.VerbNames()) do
        local spec = PoggyCore.Verbs[name]
        if spec.side ~= "client" and not H[name] then missingHandler[#missingHandler + 1] = name end
    end
    for name in pairs(H) do
        if not PoggyCore.Verbs[name] then undeclared[#undeclared + 1] = name end
    end
    if #missingHandler > 0 then
        PoggyCore.Util.Warn("dispatch: declared with no handler: %s", table.concat(missingHandler, ", "))
    end
    if #undeclared > 0 then
        PoggyCore.Util.Warn("dispatch: handler with no declaration: %s", table.concat(undeclared, ", "))
    end
end)
