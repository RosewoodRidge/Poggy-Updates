-- ============================================================================
-- permissions.lua — Poggy Transform: who may use what
--
-- The player's job (for job-locked entries), the admin check, and whether the
-- character browser can work on this framework.  Framework data comes straight
-- from poggy_core through Poggy(verb, payload).
--
-- This file is deliberately left outside escrow, so an unusual admin setup can
-- be adapted locally.
-- ============================================================================

PT = PT or {}

-- ============================================================================
-- The player's job, lower-cased.  Returns "" rather than nil when unknown, so
-- callers can compare without guarding every use.
-- ============================================================================
function PT.GetJob(src)
    local ok, job = Poggy('job.get', { src = src })
    local name = ok and job and job.name
    return type(name) == "string" and name:lower() or ""
end

-- ============================================================================
-- Admin check.
--
-- ACE is checked first because it's the only mechanism that works the same on
-- every framework, and the only one a standalone server has at all.  Grant it
-- in server.cfg:
--
--     add_ace group.admin poggy_transform.admin allow
--
-- Framework groups are then checked as a convenience for servers that keep
-- their admin list inside the framework instead.
-- ============================================================================
function PT.IsAdmin(src)
    local ace = Config.AdminAce or "poggy_transform.admin"
    if IsPlayerAceAllowed(src, ace) then return true end

    -- Compared against Config.AdminGroups on purpose: perms.isAdmin uses a
    -- wider group list and would change who counts as an admin here.
    local ok, group = Poggy('perms.group', { src = src })
    if not ok or type(group) ~= "string" or group == "" then return false end
    group = group:lower()

    for _, adminGroup in ipairs(Config.AdminGroups or {}) do
        if group == tostring(adminGroup):lower() then return true end
    end

    return false
end

-- ============================================================================
-- The character browser needs two things from poggy_core: `char.offline` with
-- appearance = true (a stored skin, clothing and tints, read from the
-- framework's own table) and `char.reloadSkin` on the client.  Today only the
-- VORP adapter returns appearance data, so the browser is offered only there,
-- and only while poggy_core reports the `char.offline` capability (it drops it
-- when oxmysql is missing).  Animals and custom ped models do not need any of
-- this and run on every framework poggy_core supports.
-- ============================================================================
function PT.SupportsPlayerSkins()
    local ok, fw = Poggy('core.framework', {})
    if ((ok and fw and fw.id) or "unknown") ~= "vorp" then return false end
    local okCap, has = Poggy('core.has', { capability = "char.offline" })
    return okCap and has == true
end

-- The framework's display name for player-facing messages ("VORP Core"),
-- falling back to its id.
function PT.FrameworkLabel()
    local ok, fw = Poggy('core.framework', {})
    return (ok and fw and (fw.label or fw.id)) or "this framework"
end
