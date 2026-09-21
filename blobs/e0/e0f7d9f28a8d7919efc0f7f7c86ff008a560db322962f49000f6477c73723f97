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

-- The key that opens the staff panel, for staff only. The default is Home.
-- (It was Page Down until 1.1.0: vorp_admin opens its own menu with that key.)
-- 0 turns the key off; the command always works.
Config.StaffKey = 0x064D1698

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

-- Show players how long their kind of ticket usually waits for staff, worked
-- out from your own tickets. Nothing shows until a kind has a few to go on.
Config.ShowExpectedWait = true

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

-- OLD, read once. Since 1.1.0 each ROLE says which kinds of ticket it sees:
-- staff panel -> Staff -> Roles, or /poggy -> Tickets -> Roles. This list is
-- only copied into a role the first time 1.1.0 starts, so your routing carries
-- over. Changing it after that does nothing.
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

-- OLD, read once, like Config.Routing above. Since 1.1.0 "told about help
-- requests" is a tick on each role.
Config.HelpRoles = { "mod", "helper" }

-- ---------------------------------------------------------------------------
-- Solved questions
-- ---------------------------------------------------------------------------

-- Staff can mark the answer in a ticket's chat and, once the ticket is closed,
-- make it public. A public ticket can be read by any player with every name
-- hidden. Nothing is public until staff say so. false turns all of it off.
Config.PublicAnswers = true

-- While a player types a ticket, show up to three public answers that look
-- like their problem, so a question already answered never becomes a ticket.
Config.SuggestAnswers = true

-- The kinds of ticket that may be made public. A kind that names a player
-- (reportsPlayer = true) can never be made public, whatever is listed here.
Config.PublicKinds = { "bug", "help", "stuck", "other" }

-- ---------------------------------------------------------------------------
-- Quiet tickets
-- ---------------------------------------------------------------------------

-- Hours after staff last wrote, with no answer from the player, before the
-- player is asked "still need help?". 0 turns it off. A player who is waiting
-- on STAFF is never nudged.
Config.NudgeAfterHours = 24

-- Hours after that question, still with no answer, before the ticket closes
-- itself as "No response". 0 never closes a ticket for you.
Config.AutoCloseAfterHours = 48

-- ---------------------------------------------------------------------------
-- Staff
-- ---------------------------------------------------------------------------

-- Minutes a claimed ticket waits for a staff member who logged off before it
-- is free for someone else to claim.
Config.StaleClaimMinutes = 15

-- Show on-duty staff a small count of open tickets on screen.
Config.ShowBadge = true

-- Open the staff panel full screen. Each staff member can switch it with the
-- button in the panel; this is only how it starts.
Config.StaffFullscreen = false

-- Show on-duty staff a small pop-up for a new staff chat message.
Config.ChatPopups = true

-- Days before a warning stops counting toward Config.Ladder below. It stays on
-- the player's record for good. 0 means warnings always count.
Config.WarningDecayDays = 90

-- What to suggest for a player's 1st, 2nd, 3rd... offence, counting warnings
-- that still count and every ban. It is only shown on the player's record as a
-- suggestion: staff always decide. The last row is used for anything beyond it.
--   "warn", "kick", or "ban:" and a length: "ban:1d" "ban:3d" "ban:7d" "ban:30d" "ban:perm"
-- An empty list turns the suggestion off.
Config.Ladder = { "warn", "warn", "kick", "ban:1d", "ban:7d", "ban:perm" }

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
-- The web panel (off by default)
-- ---------------------------------------------------------------------------

-- Tickets in a browser, at rosewoodridge.xyz/tickets, signed in with Cfx.re:
--   - every PLAYER reads and answers their own tickets, under each character, and
--     writes new ones when they are not in game
--   - anyone whose account holds a ticket role gets the staff desk as well
--   - banned players can appeal to you (Appeals, below)
-- A player is linked to their Cfx.re account by itself at login when their game
-- names one, or with a link code from /ticket, Website.
--
-- READ THIS BEFORE TURNING IT ON. Your server cannot talk to a website directly,
-- so copies of your OPEN tickets (names, what players wrote, the conversations,
-- internal notes) are sent to a relay run by Rosewood Ridge and kept there while
-- a ticket is open and for 7 days after it closes. Turn it off again and the
-- relay is told to forget your community at once. If that is not acceptable for
-- your community, leave this off: nothing else in the script needs it.
--
-- From the web, staff can read, reply, write internal notes, claim, release and
-- close. Warn, kick and ban stay in game. Every web action is checked against
-- the person's ticket role on THIS server, exactly like an in-game press, and is
-- in the audit trail as "Name (web)".
Config.Web = {
    -- true turns the website on. Your community's ONE ID is made for you and shown
    -- to everyone: at the top of /tickets and /ticket, and on their Website tab.
    Enabled = false,

    -- Where the relay is. Do not change it.
    Api = "https://poggy-tickets.poggy-bans.workers.dev",

    -- true lets banned players appeal to you from rosewoodridge.xyz/appeal. Each
    -- appeal arrives as a ticket of the kind "Ban appeal", seen by every role
    -- that may lift bans. It also lists your community BY NAME on that page.
    Appeals = false,

    -- The name players see when they pick who to appeal to. Empty uses your
    -- server's name.
    PublicName = "",

    -- One line shown above your appeal box, for example your rules on appeals.
    AppealNote = "",

    -- true lets staff WARN, KICK and BAN from the website, on the player a ticket
    -- reports. Off, those three stay in game. Each one still needs the role power,
    -- is checked again by this server, and is audited as "Name (web)". A ban from
    -- the website is always a local ban: the shared ban network is fed from in game.
    Moderation = false,

    -- true: a Cfx.re sign-in is not enough for the STAFF desk. Each browser must be
    -- confirmed once with a link code from /tickets, Website, so a stolen Cfx.re
    -- login is useless without also being in your game. Players' own tickets are
    -- not affected. Leave this on if Moderation is on.
    VerifyDevices = true,

    -- STAFF only: someone with a ticket role keeps the staff desk on the website
    -- for this many days after their last login here. A player's own tickets do
    -- not run out.
    LinkDays = 30,
}

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
