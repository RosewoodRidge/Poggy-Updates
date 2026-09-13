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
-- Copying another character's full appearance reads skinPlayer / compPlayer /
-- compTints out of the `characters` table.  Those columns are VORP's schema —
-- RSG and QBCore store clothing in a different shape entirely — so the player
-- browser is offered only where it can actually work.  Animals and custom ped
-- models run on every framework.
-- ============================================================================
function PT.SupportsPlayerSkins()
    local ok, fw = Poggy('core.framework', {})
    return ((ok and fw and fw.id) or "unknown") == "vorp"
end
