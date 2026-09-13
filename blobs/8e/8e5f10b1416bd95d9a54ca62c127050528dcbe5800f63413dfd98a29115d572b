Config = {}

-- ── Debug ────────────────────────────────────────────────────────────────
Config.debug          = false        -- when true, prints resolveBadge info to server console

-- ── Commands ─────────────────────────────────────────────────────────────
Config.command        = "badge"      -- /pbadge  → opens badge menu / toggle
Config.showDistance   = 2             -- how far away others can see the badge display

-- ── Skeleton Attachment Defaults ─────────────────────────────────────────
-- These are the baseline values for a freshly attached badge.
-- Players can fine-tune via the editor NUI and save presets.
--
-- Available SKEL_Spine bones (top to bottom):
--   SKEL_Spine5  – collar / upper chest (near neckline)
--   SKEL_Spine4  – upper chest
--   SKEL_Spine3  – mid chest (heart area)
--   SKEL_Spine2  – lower chest / solar plexus
--   SKEL_Spine1  – stomach / upper abdomen
--   SKEL_Spine0  – waist / belt line
--   SKEL_SpineRoot – pelvis / hip center
--
Config.defaultBone   = "SKEL_Spine5"
Config.defaultOffset = vector3(0.081, 0.169, 0.142)
Config.defaultRotation = vector3(-152.25, 83.1, -55.5)

-- ── Per-prefix rotation overrides ────────────────────────────────────────
-- If a prop model begins with one of these prefixes, its default rotation
-- is replaced.  Players can still override via presets.  The stock "s_"
-- badges below sit right with Config.defaultRotation, so nothing is needed
-- for them; add a prefix here for custom badge packs that face the wrong way.
Config.prefixRotations = {
    ["kh_"]  = vector3(121.75, 74.75, -36),  -- example: a custom badge pack
}

-- ── Badge NUI Display (bottom-right badge flash) ─────────────────────────
Config.badgeDisplayDuration = 8   -- seconds the badge image shows to nearby players

-- ── Show animation (badge held in hand) ──────────────────────────────────
Config.showAnimDuration = 5000   -- ms the badge is held up before the outro plays

-- ── Badge Definitions ────────────────────────────────────────────────────
-- Each entry defines a badge "set".  Multiple jobs can share the same set.
-- Grades map to { badge = <image name>, prop = <model name> }.
--
-- Fields per badge set:
--   jobs   = list of job names that may use this set
--   grades = [gradeNumber] → { badge, prop, useName, useServerName }
--
-- `badge` is the image filename (without .png) in ui/images/
-- `prop`  is the in-game prop model spawned on the player
--
-- Job names are matched EXACTLY (case-sensitive) against the character's
-- job, so list every spelling your server uses.  A job that matches but has
-- no entry for the player's grade gets no badge, so cover every grade the
-- job has (0-5 below covers the usual range; add more if you need them).
--
-- Stock game badge props (no custom stream needed):
--   s_badgedeputy01x     Deputy
--   s_badgepolice01x     Police
--   s_badgepinkerton01x  Pinkerton
--   s_badgesherif01x     Sheriff
--   s_badgeusmarshal01x  US Marshal
--
-- To add a department: copy a set below, change `jobs` to your job name(s),
-- and pick a prop and an image per grade.  Custom props from a streamed
-- badge pack work the same way (see Config.prefixRotations if they need a
-- different default rotation).
Config.serverName = "Your Server"

Config.badges = {
    -- ── Law: Sheriff ─────────────────────────────────────────────────────
    {
        jobs = { "sheriff" },
        grades = {
            [0] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
            [1] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
            [2] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
            [3] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
            [4] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
            [5] = { badge = "sheriff", prop = "s_badgesherif01x", useName = true, useServerName = false },
        },
    },

    -- ── Law: Deputy ──────────────────────────────────────────────────────
    {
        jobs = { "deputy" },
        grades = {
            [0] = { badge = "silverbadge", prop = "s_badgedeputy01x", useName = true, useServerName = false },
            [1] = { badge = "silverbadge", prop = "s_badgedeputy01x", useName = true, useServerName = false },
            [2] = { badge = "silverbadge", prop = "s_badgedeputy01x", useName = true, useServerName = false },
            [3] = { badge = "silverbadge", prop = "s_badgedeputy01x", useName = true, useServerName = false },
            [4] = { badge = "silverbadge", prop = "s_badgedeputy01x", useName = true, useServerName = false },
            [5] = { badge = "silverbadge", prop = "s_badgedeputy01x", useName = true, useServerName = false },
        },
    },

    -- ── Law: Police ──────────────────────────────────────────────────────
    -- vorp_police ships its law job as "Police" (capital P), so both
    -- spellings are listed.
    {
        jobs = { "police", "Police" },
        grades = {
            [0] = { badge = "police", prop = "s_badgepolice01x", useName = true, useServerName = false },
            [1] = { badge = "police", prop = "s_badgepolice01x", useName = true, useServerName = false },
            [2] = { badge = "police", prop = "s_badgepolice01x", useName = true, useServerName = false },
            [3] = { badge = "police", prop = "s_badgepolice01x", useName = true, useServerName = false },
            [4] = { badge = "police", prop = "s_badgepolice01x", useName = true, useServerName = false },
            [5] = { badge = "police", prop = "s_badgepolice01x", useName = true, useServerName = false },
        },
    },

    -- ── Law: US Marshal ──────────────────────────────────────────────────
    {
        jobs = { "marshal", "usmarshal" },
        grades = {
            [0] = { badge = "marshal", prop = "s_badgeusmarshal01x", useName = true, useServerName = false },
            [1] = { badge = "marshal", prop = "s_badgeusmarshal01x", useName = true, useServerName = false },
            [2] = { badge = "marshal", prop = "s_badgeusmarshal01x", useName = true, useServerName = false },
            [3] = { badge = "marshal", prop = "s_badgeusmarshal01x", useName = true, useServerName = false },
            [4] = { badge = "marshal", prop = "s_badgeusmarshal01x", useName = true, useServerName = false },
            [5] = { badge = "marshal", prop = "s_badgeusmarshal01x", useName = true, useServerName = false },
        },
    },

    -- ── Law: Pinkerton ───────────────────────────────────────────────────
    {
        jobs = { "pinkerton" },
        grades = {
            [0] = { badge = "pinkerton", prop = "s_badgepinkerton01x", useName = true, useServerName = false },
            [1] = { badge = "pinkerton", prop = "s_badgepinkerton01x", useName = true, useServerName = false },
            [2] = { badge = "pinkerton", prop = "s_badgepinkerton01x", useName = true, useServerName = false },
            [3] = { badge = "pinkerton", prop = "s_badgepinkerton01x", useName = true, useServerName = false },
            [4] = { badge = "pinkerton", prop = "s_badgepinkerton01x", useName = true, useServerName = false },
            [5] = { badge = "pinkerton", prop = "s_badgepinkerton01x", useName = true, useServerName = false },
        },
    },
}
