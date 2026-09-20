Config = {}

-- ============================================================================
--  poggy_tickets · settings
--  Every setting here can also be changed in game with /poggy.
-- ============================================================================

-- Language of every message and of the windows. Options: "en"
Config.Language = "en"

-- ---------------------------------------------------------------------------
-- Opening the windows
-- ---------------------------------------------------------------------------

-- The chat command players type to open the ticket form (without the slash).
Config.Command = "ticket"

-- The key that opens the ticket form. The default is Page Up. 0 turns the key off;
-- the command always works.
Config.OpenKey = 0x446258B6

-- The chat command staff type to open the staff panel (without the slash).
Config.StaffCommand = "tickets"

-- The key that opens the staff panel, for staff only. The default is Page Down.
-- 0 turns the key off; the command always works.
Config.StaffKey = 0x3C3DD371

-- The chat command that opens the Players menu: everyone online, with Warn,
-- Kick and Ban, and no ticket needed. "/mod 12" opens it on player 12.
-- For staff who may warn, kick or ban (mods and admins).
Config.ModCommand = "mod"

-- The chat command staff type to turn their invisibility on or off.
Config.InvisCommand = "invis"

-- The chat command staff type to go off duty, or back on. Off duty means no
-- sounds, pop-ups or badge. Everyone is back on duty when they log in again.
Config.DutyCommand = "staffduty"

-- ---------------------------------------------------------------------------
-- The ticket form
-- ---------------------------------------------------------------------------

-- Who a player can pick as the reported player.
--   "nearby"  players within NearbyRadius, nearest first, as "id · name"
--   "all"     everyone online, as "id · name"
--   "ids"     everyone online, server ids only (no names)
Config.PlayerList = "nearby"

-- How close, in metres, a player must be to show in the "nearby" list.
Config.NearbyRadius = 100

-- Minutes a player must wait between tickets. 0 means no wait.
Config.TicketCooldownMinutes = 15

-- How many open tickets one player may have at once.
Config.MaxOpenTickets = 2

-- Tell the player when no staff who handle their kind of ticket are on duty.
-- The ticket is always saved either way.
Config.TellWhenNoStaff = true

-- The kinds of ticket. Each row:
--   id             short name, never change it once tickets exist
--   label          what players and staff read
--   priority       the priority picked for the player to start with:
--                  "urgent", "high", "medium" or "low"
--   reportsPlayer  true: the form asks which player this is about
--   wantsClip      true: the form asks for a video link (never required)
Config.Categories = {
    { id = "bug",     label = "Bug",                 priority = "medium", reportsPlayer = false, wantsClip = true },
    { id = "cheater", label = "Cheater",             priority = "high",   reportsPlayer = true,  wantsClip = true },
    { id = "report",  label = "Player report",       priority = "medium", reportsPlayer = true,  wantsClip = true },
    { id = "help",    label = "Help",                priority = "medium", reportsPlayer = false, wantsClip = false },
    { id = "stuck",   label = "Stuck or lost items", priority = "high",   reportsPlayer = false, wantsClip = false },
    { id = "other",   label = "Other",               priority = "low",    reportsPlayer = false, wantsClip = false },
}

-- Which staff roles see which kind of ticket. Admins always see everything.
-- Roles: "mod", "helper", "developer". The key is a category id from above.
Config.Routing = {
    ["bug"]     = { "developer" },
    ["cheater"] = { "mod" },
    ["report"]  = { "mod" },
    ["help"]    = { "mod", "helper" },
    ["stuck"]   = { "mod", "helper" },
    ["other"]   = { "mod", "developer" },
}

-- ---------------------------------------------------------------------------
-- Help requests (the "I need help now" button: no form, no ticket)
-- ---------------------------------------------------------------------------

-- Minutes a player must wait between help requests.
Config.HelpCooldownMinutes = 5

-- Minutes before a help request nobody closed disappears.
Config.HelpExpireMinutes = 10

-- Which staff roles are told about help requests. Admins always are.
Config.HelpRoles = { "mod", "helper" }

-- ---------------------------------------------------------------------------
-- Staff
-- ---------------------------------------------------------------------------

-- Minutes a claimed ticket waits for a staff member who logged off before it
-- is free for someone else to claim.
Config.StaleClaimMinutes = 15

-- Show on-duty staff a small count of open tickets on screen.
Config.ShowBadge = true

-- Teleporting to a player or a ticket.
Config.Teleport = {
    Invisible = true,   -- arrive invisible and unable to be hurt
    Behind    = 3.0,    -- metres behind the player to arrive at
}

-- How loud the notification sound is: 0.0 (silent) to 1.0 (full).
Config.SoundVolume = 0.4

-- Days a closed ticket is kept. 0 keeps them forever.
Config.PurgeClosedAfterDays = 0

-- ---------------------------------------------------------------------------
-- Discord
-- ---------------------------------------------------------------------------

-- Each ticket is one post in your channel. The same post is edited as the
-- ticket is claimed and solved; it is never deleted.
Config.Discord = {
    -- The channel's webhook address. Leave as it is for no Discord posts.
    Webhook = "YOUR DISCORD WEBHOOK HERE",
    -- The name the posts are made under.
    Name = "Tickets",
    -- A different webhook for one kind of ticket. The key is a category id,
    -- for example ["bug"] = "https://discord.com/api/webhooks/...".
    CategoryWebhooks = {},
}
