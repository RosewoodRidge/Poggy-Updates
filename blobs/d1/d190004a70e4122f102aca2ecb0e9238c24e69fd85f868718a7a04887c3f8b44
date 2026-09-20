--[[
    poggy_core — configuration

    Everything here is deliberately data rather than code, so a server owner can
    change behaviour without touching the adapters.
]]

PoggyCoreConfig = {}

-- ---------------------------------------------------------------------------
-- Framework
-- ---------------------------------------------------------------------------

--- Force a framework instead of detecting one.
--- One of: 'vorp', 'rsg', 'qbr', 'redem', 'rpx', 'standalone', or false to detect.
--- Useful on a server mid-migration that has two framework cores present.
PoggyCoreConfig.ForceFramework = false

--- Detection order. The first framework whose core resource is *started* and
--- whose handshake succeeds wins. VORP is first here deliberately: poggy_util
--- checked RSG first, which meant a server running both resolved to RSG.
PoggyCoreConfig.DetectionOrder = { "vorp", "rsg", "qbr", "redem", "rpx" }

--- How long to wait for the framework core object before giving up (ms).
PoggyCoreConfig.FrameworkTimeout = 30000

-- ---------------------------------------------------------------------------
-- Restarting poggy_core (0.15.0)
-- ---------------------------------------------------------------------------

--- Every Poggy script depends on poggy_core, so `restart poggy_core` stops all
--- of them and FXServer does not start them again. With this on, poggy_core
--- starts them itself once it is back (`ensure <script>`, through the same
--- `add_ace resource.poggy_core command.ensure allow` line the updater uses),
--- and prints one line naming them.
---
--- It only ever starts scripts that were running when poggy_core stopped: it
--- keeps a note of them in the convar `poggy_core_last_stop`, which a server
--- restart clears, so on a fresh boot (when every script is simply not started
--- yet) nothing is touched and server.cfg starts them in its own order. A script
--- you stopped yourself is not started. `poggycore dependents` lists them.
PoggyCoreConfig.RestartDependents = true

-- ---------------------------------------------------------------------------
-- Storage
-- ---------------------------------------------------------------------------

--- Every container id registered through Core.Storage gets this prefix plus the
--- calling script's poggy_id (its folder name when it declares none), e.g.
--- 'pg_poggy_markets_valentine'. The id, not the folder: an owner may rename
--- the folder, and the stored items must still be found.
---
--- This is not cosmetic. RSG's client-facing stash route rejects any id starting
--- 'police-', 'marshal-', 'gang-', 'admin-' or 'evidence-', and VORP/QBR share a
--- single flat id namespace with every other resource on the server. Prefixing
--- keeps us clear of both.
PoggyCoreConfig.StoragePrefix = "pg"

--- Default container options, merged under whatever the caller passes.
PoggyCoreConfig.StorageDefaults = {
    slots        = 50,
    maxWeight    = 200000,
    shared       = true,
    allowWeapons = false,
}

-- ---------------------------------------------------------------------------
-- Jobs
-- ---------------------------------------------------------------------------

--- Jobs that Core.Job.IsLaw() treats as law enforcement.
--- This lived hard-coded in poggy_util/client/cl_framework.lua; it is config now
--- because every server names these differently.
PoggyCoreConfig.LawJobs = {
    "police", "sheriff", "marshal", "lawman", "deputy",
    "ranger", "constable", "fib", "agent", "detective", "trooper",
}

--- Jobs that Core.Job.IsMedical() treats as medical.
PoggyCoreConfig.MedicalJobs = {
    "doctor", "medic", "nurse", "ems", "ambulance", "surgeon",
}

-- ---------------------------------------------------------------------------
-- Characters
-- ---------------------------------------------------------------------------

--- The command char.reloadSkin runs on VORP: vorp_character's
--- Config.ReloadCharCommand. Change it here if you changed it there.
PoggyCoreConfig.VorpReloadSkinCommand = "rc"

-- ---------------------------------------------------------------------------
-- Notifications
-- ---------------------------------------------------------------------------

--- Renderer preference, tried in order until one is available.
---   'native'    — poggy_core's own RDR2 notification renderer (no other resource needed)
---   'framework' — the detected framework's own notification API
---   'chat'      — last-resort chat message, so a notify is never silently lost
--- 'poggy_util' is still accepted and means 'native'; poggy_util is not needed.
PoggyCoreConfig.NotifyRenderers = { "native", "framework", "chat" }

--- Default duration in ms when a caller does not give one.
PoggyCoreConfig.NotifyDuration = 4000

-- ---------------------------------------------------------------------------
-- Database
-- ---------------------------------------------------------------------------

--- Every Poggy script keeps its tables in sql/install.sql. When a script starts,
--- poggy_core runs that file for it: missing tables, columns and indexes are
--- added, anything already there is skipped, and nothing is dropped or
--- overwritten. Pending sql/migrations/001.sql, 002.sql ... run once each and are
--- recorded in the `poggy_migrations` table.
---
--- false: nothing runs on its own. Import the files yourself, or run
---     poggycore sql install <resource|all>
--- from the server console.
PoggyCoreConfig.Sql = {
    AutoInstall = true,
}

-- ---------------------------------------------------------------------------
-- Updates
-- ---------------------------------------------------------------------------

--- Where `poggycore update <resource>` looks (server console only).
---
--- The update feed is public: no account, key or token is needed.
--- (Only a private copy of the feed needs a read-only GitHub token, set in
--- server.cfg with `set`, never sets/setr: set poggy_github_token "github_pat_...")
---
--- On a development server (the one the scripts are written on) put
---     set poggy_dev_server 1
--- in server.cfg. Updates are then reported but never written or restarted, so
--- an update cannot overwrite the source with an escrowed build. That is a
--- convar and not a setting here on purpose: this file ships to customers, and
--- the merge keeps an owner's values, so a flag in the development copy would
--- be carried into every release.
PoggyCoreConfig.Updates = {
    -- "github":  the published update feed in the GitHub repository Repo.
    -- "website": the same feed served from Url (not usable on a .xyz domain:
    --            FXServer refuses every .xyz hostname).
    Source = "github",
    Repo   = "RosewoodRidge/Poggy-Updates",
    Branch = "main",
    Url    = "https://rosewoodridge.xyz/api/updates/scripts",

    -- Development only: `poggycore update <resource> repo=Owner/Name` reads a
    -- source repository directly instead of the published feed. SourceRepo is
    -- the default for that flag; leave it unset unless you develop the scripts.
    -- SourceRepo = "Owner/Name",
    Repos      = {},

    -- A resource is only ever updated when the published fxmanifest.lua version
    -- is higher than the one on this server. Same or older: nothing is touched.
    CheckOnStart      = true,    -- on start, log every Poggy resource with a newer version
    ApplyOnStart      = true,   -- install them on start only (same as AutoUpdate, but only at start)
    StartDelaySeconds = 20,

    -- Automatic updates. On by default: your Poggy scripts keep themselves
    -- current and you never install an update by hand.
    --   true:  every available update is checked AND installed, at start and on
    --          every periodic check. Config files are merged (your values kept),
    --          originals are backed up in poggy_core/update_backups/.
    --   false: poggy_core only tells you in the console what is available.
    AutoUpdate           = true,
    CheckIntervalMinutes = 60,     -- check again while running; 0 = only at start
    -- After installing, run `refresh` once and restart each updated resource.
    -- This needs two lines in server.cfg (poggy_core prints them if missing):
    --     add_ace resource.poggy_core command.refresh allow
    --     add_ace resource.poggy_core command.ensure allow
    -- poggy_core never restarts itself: its own update takes effect on the next
    -- server restart (or `refresh` then `ensure poggy_core` by hand).
    RestartUpdated       = true,
    -- Updates installed while players are online: files are written at once, but
    -- the restarts wait until the server is empty. At start they happen at once.
    RestartWhenEmptyOnly = true,

    -- After the start-up check, list the published Poggy scripts this server
    -- does not have, once per boot: name and store link, grey, at most ten
    -- rows. Nothing is printed when every published script is installed.
    -- `poggycore catalog` prints it on demand whatever this says.
    ShowCatalog          = true,
}

-- ---------------------------------------------------------------------------
-- Timeouts and diagnostics
-- ---------------------------------------------------------------------------

--- How long to wait on a framework callback before treating it as failed (ms).
--- VORP's inventory API is callback-based; without this a dropped callback would
--- hang the calling coroutine forever.
PoggyCoreConfig.CallbackTimeout = 5000

--- How long a Core.Callback request may take before it resolves to nil (ms).
PoggyCoreConfig.RpcTimeout = 10000

--- Print every dispatched call. Very noisy; for adapter development only.
PoggyCoreConfig.Debug = false

--- Print a one-line warning the first time a resource calls something the
--- detected framework cannot do. Leave this on.
PoggyCoreConfig.WarnUnsupported = true

-- ---------------------------------------------------------------------------
-- Menu and text input (0.14.0)
-- ---------------------------------------------------------------------------
--- poggy_core draws the menus and text boxes the Poggy scripts open
--- (menu.open, input.text), so no script needs vorp_menu or vorp_inputs.
PoggyCoreConfig.Ui = {
    Cursor   = true,     -- show the mouse cursor in a menu or input unless the call says otherwise
    Sounds   = true,     -- the game's menu sounds on move, select and close
    Timeout  = 300000,   -- a SERVER-side menu.open / input.text waits this long (ms) for the
                         -- player's answer before returning 'timeout' and closing the page.
                         -- A menu waits on a person, so this is longer than RpcTimeout.

    --- Where the menu and the text box sit on the screen. One of:
    ---   'center'  (default)   'left'  'right'          centred vertically
    ---   'top-left'  'top-right'  'bottom-left'  'bottom-right'
    --- Applies to every Poggy script's menus and inputs. The panel is pinned
    --- by its edge so the rows never move as the description under them
    --- changes: centre and top positions grow downward; the bottom ones keep
    --- a fixed height instead (the description area held at its full five
    --- lines) since growing upward would push the rows.
    --- An unknown value falls back to 'center'.
    Position = 'center',
    --- Distance in pixels from the screen edge for the side, top and bottom
    --- positions (and the least gap a centred panel keeps from the edges).
    Margin   = 40,
}

-- ---------------------------------------------------------------------------
-- Settings hub (0.18.0)
-- ---------------------------------------------------------------------------
--- /poggy opens a full-screen hub in game where an admin edits every Poggy
--- script's settings. It changes the config files themselves (every comment
--- and every line it does not touch stays as it was), keeps a backup of each
--- file in poggy_core/update_backups/, and offers to restart the script.
---
--- Who may use it: players with the ACE poggy.settings, or anyone the
--- framework counts as an admin. Taking over a script someone else is
--- editing needs poggy.settings.takeover. Restarting a script from the hub
--- needs this line in server.cfg (the updater uses the same one):
---     add_ace resource.poggy_core command.ensure allow
PoggyCoreConfig.Hub = {
    Command         = "poggy",  -- the chat command that opens the hub
    IdleMinutes     = 10,       -- a script's lock is released after this long with no activity
    IdleWarnMinutes = 8,        -- the editor is warned at this point
    HistoryDays     = 90,       -- change history older than this is deleted when poggy_core starts
}

--- Master job and group lists, edited from the hub's Roles page. A job or
--- group list in any Poggy script's config can be linked to one of these from
--- the hub; saving a role then rewrites every linked list and restarts the
--- scripts that use it. The link is a comment on that setting's line
--- (-- poggy:role lawmen), so the script itself never reads this.
---     label  what the hub shows
---     kind   "jobs" or "groups" (admin groups)
---     list   the names, exactly as the framework spells them
PoggyCoreConfig.Roles = {
    lawmen = { label = "Lawmen", kind = "jobs", list = { "sheriff", "deputy", "marshal" } },
    medics = { label = "Medics", kind = "jobs", list = { "doctor" } },
    staff  = { label = "Staff groups", kind = "groups", list = { "admin", "superadmin", "god" } },
}

-- ---------------------------------------------------------------------------
-- Bans (0.19.0)
-- ---------------------------------------------------------------------------
--- A ban is on the player's ACCOUNT, so it follows them across characters.
--- Ban from the console or chat (poggycore ban / unban / baninfo), from
--- /poggy (poggy_core, Bans), or from a script that uses it (poggy_tickets).
--- Lifting a ban keeps it on record. If the database cannot be read, nobody
--- is refused: a fault here never locks players out.
PoggyCoreConfig.Bans = {
    Enabled    = true,                        -- false: nobody is refused, and nothing can be banned
    AppealText = "You can appeal on our Discord.",   -- the last line a banned player reads

    --- THE POGGY BAN NETWORK (0.20.0). ON BY DEFAULT: read this.
    ---
    --- A ban you mark as CHEATING is shared, as a vote, with every server
    --- running poggy_core. A player with votes from 3 or more servers is refused
    --- on every server that enforces the network, yours included. A rule-break
    --- ("local") ban never leaves your server. Lifting a ban takes your vote back.
    ---
    --- What is shared: scrambled (hashed) identifiers and your player count.
    --- Never a name, never a reason. If the network cannot be reached, nobody is
    --- refused because of it.
    ---
    --- Let one blocked player into YOUR server:  poggycore banallow <identifier> <reason>
    Network = {
        Enforce    = true,    -- refuse players the network has voted out. false: nobody is refused by it.
        Submit     = true,    -- share your CHEATING bans as votes. false: your bans stay on your server.
        BlockAt    = 3,       -- votes needed before YOU refuse someone. You can raise it; below the network's 3 has no effect.
        FlagStaff  = true,    -- tell staff when someone joins who has 1 or more votes but is not blocked
        ServerName = "",      -- shown to a player you banned, on the appeal page. Blank: your server's project name.
        AppealLink = "",      -- where they can appeal to you, for example your Discord invite
        HideName   = false,   -- true: your vote counts, but banned players are not shown your server's name
    },
}
