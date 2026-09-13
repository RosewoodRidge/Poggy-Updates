--- poggy_core — the single entry point every Poggy script uses (client side).
---
--- The mirror of sv_dispatch.lua. Same export name, same call shape, same
--- return contract, so the one vendored Poggy() bridge works unchanged on both
--- sides and a script author never has to remember which side they are on.
---
---     local ok, value, err = exports.poggy_core:Do(verb, payload)
---
--- The client surface is deliberately much smaller than the server's. A client
--- cannot be trusted with money, inventory or jobs, so those verbs are declared
--- server-only in sh_verbs.lua and asking for one here is refused by name
--- rather than quietly ignored. `src` is meaningless on the client and is
--- ignored where a payload carries it.

local Err = PoggyCore.Err

local H = {}

local function ret(value, err)
    if value == nil then return false, nil, err or Err.NOT_FOUND end
    return true, value, nil
end

-- character ------------------------------------------------------------------

H["char.get"] = function(C, p) return ret(C.GetChar(p.force)) end
H["char.id"]  = function(C)    return ret(C.GetCharId()) end

-- jobs -----------------------------------------------------------------------

H["job.get"] = function(C)
    local name, grade, label, gradeLabel, onDuty = C.Job.Get()
    if name == nil then return false, nil, Err.NO_CHAR end
    return true, {
        name = name, grade = grade, label = label,
        gradeLabel = gradeLabel, onDuty = onDuty,
    }, nil
end

H["job.has"]       = function(C, p) return true, C.Job.Has(p.job, p.minGrade) and true or false, nil end
H["job.isLaw"]     = function(C)    return true, C.Job.IsLaw() and true or false, nil end
H["job.isMedical"] = function(C)    return true, C.Job.IsMedical() and true or false, nil end

-- money ----------------------------------------------------------------------

-- Only the capability question is answerable here; balances and mutations are
-- server business.
H["money.supports"] = function(C, p) return true, C.Has("money." .. tostring(p.currency)) and true or false, nil end

-- notifications --------------------------------------------------------------

H["notify"] = function(_, p)
    PoggyCore.RenderNotify(p.text, p.kind, p.duration)
    return true, true, nil
end

H["notify.rich"] = function(_, p)
    PoggyCore.RenderNotifyRich({
        title = p.title, description = p.description, kind = p.kind,
        duration = p.duration, dict = p.dict, icon = p.icon, color = p.color,
    })
    return true, true, nil
end

H["notify.styled"] = function(_, p)
    local drawn = PoggyCore.RenderStyled(p.style, {
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
    if not drawn then return false, nil, Err.BAD_ARG end
    return true, true, nil
end

-- 0.11.0 ---------------------------------------------------------------------

--- Re-dress the local ped from the saved character. On VORP that is
--- vorp_character's reload command, whose name lives in that resource's config.
H["char.reloadSkin"] = function(C)
    if C.GetFramework() ~= "vorp" then return false, nil, Err.UNSUPPORTED end
    ExecuteCommand(PoggyCoreConfig.VorpReloadSkinCommand or "rc")
    return true, true, nil
end

--- Same answer as the server, from the shared framework table: no round trip.
H["inv.imageBase"] = function(C)
    local def = PoggyCore.Frameworks[C.GetFramework()]
    return ret(def and def.itemImageBase, Err.UNSUPPORTED)
end

--- A server callback, awaited. The value is the packed results { n, ... }.
H["callback.await"] = function(_, p)
    local args = type(p.args) == "table" and p.args or { n = 0 }
    local results, why = PoggyCore.CallbackAwaitPacked(p.name, table.unpack(args, 1, args.n or #args))
    if not results then return false, nil, why end
    return true, results, nil
end

-- ui (0.14.0) ----------------------------------------------------------------
-- Drawn by poggy_core's own page (ui/, client/cl_menu.lua). Both wait for the
-- player, so like callback.await they need a thread; PoggyCore.Ui says so
-- itself and answers needs_thread. The calling resource rides along so the
-- page can be taken down if that script stops while it is up. `src` is
-- meaningless here and ignored, as everywhere on the client.

H["menu.open"] = function(_, p, resource)
    return PoggyCore.Ui.Menu({
        title = p.title, items = p.items, subtitle = p.subtitle,
        cursor = p.cursor, closeText = p.closeText,
    }, resource)
end

H["menu.close"] = function()
    return true, PoggyCore.Ui.Close(), nil
end

H["input.text"] = function(_, p, resource)
    return PoggyCore.Ui.Input({
        title = p.title, placeholder = p.placeholder, default = p.default,
        maxLength = p.maxLength, numeric = p.numeric, submitText = p.submitText,
        cursor = p.cursor,
    }, resource)
end

-- core -----------------------------------------------------------------------

H["core.ready"]   = function(C) return true, (C.IsReady() and C.HasAdapter()) and true or false, nil end
H["core.version"] = function()  return true, PoggyCore.VERSION, nil end
H["core.has"]     = function(C, p) return true, C.Has(p.capability) and true or false, nil end
H["core.caps"]    = function(C) return ret(C.Caps) end

H["core.framework"] = function(C)
    -- GetFramework() returns the id only. The client has no adapter object, so
    -- the label comes from the shared framework table.
    local id  = C.GetFramework()
    local def = PoggyCore.Frameworks[id]
    return true, { id = id, label = (def and def.label) or id }, nil
end

-- ---------------------------------------------------------------------------

--- Run one verb. Returns ok, value, err.
--- @param resource string the calling resource; handlers that must know who
---   asked (the ui verbs, for cleanup) read it, and a payload cannot fake it
local function dispatch(verb, payload, resource)
    if type(verb) ~= "string" then return false, nil, Err.BAD_ARG end

    local allowed, why = PoggyCore.VerbAllowed(verb, "client")
    if not allowed then
        if why == "unknown_verb" then
            print(("[poggy_core] ^3WARN^7 dispatch: '%s' is not a verb. See /poggycore verbs.")
                :format(verb))
        else
            print(("[poggy_core] ^3WARN^7 dispatch: '%s' is server-side and cannot be called from a client.")
                :format(verb))
        end
        return false, nil, why
    end

    local okArgs, missing = PoggyCore.VerbCheck(verb, payload)
    if not okArgs then
        print(("[poggy_core] ^3WARN^7 dispatch: '%s' called without '%s'."):format(verb, tostring(missing)))
        return false, nil, Err.BAD_ARG
    end

    local handler = H[verb]
    if not handler then return false, nil, Err.NOT_IMPL end

    local Core = PoggyCore.Self()
    if not Core or not Core.IsReady() then return false, nil, Err.NOT_READY end
    if not Core.HasAdapter() then return false, nil, Err.UNSUPPORTED end

    local ok, a, b, c = pcall(handler, Core, payload or {}, resource or "poggy_core")
    if not ok then
        print(("[poggy_core] ^3WARN^7 dispatch: '%s' errored: %s"):format(verb, tostring(a)))
        return false, nil, Err.FRAMEWORK_ERR
    end
    return a, b, c
end

exports("Do", function(verb, payload)
    return dispatch(verb, payload, GetInvokingResource() or "poggy_core")
end)

function PoggyCore.Do(verb, payload)
    return dispatch(verb, payload, "poggy_core")
end
