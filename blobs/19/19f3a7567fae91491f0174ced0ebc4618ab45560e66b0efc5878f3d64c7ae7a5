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
-- Inventory
-- ---------------------------------------------------------------------------

--- How long inv.items / inv.itemInfo keep the item registry before reading it
--- again (seconds). Items only change when rows are added to the database, and
--- a vorp_inventory restart clears the cache anyway.
PoggyCoreConfig.ItemCacheSeconds = 300

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
