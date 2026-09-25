--[[===========================================================================
    poggy_markets · config/config.lua
    ---------------------------------------------------------------------------
    Main settings.  This is the only file most servers ever need to touch.

    Store locations live in  config/stores.lua
    Item lists live in       config/catalog_default.lua
    Optional modules live in config/pricing.lua, config/exchange.lua
    and config/ghostbuyer.lua

    Everything here has a working default.  You can start the resource without
    changing a single line.
===========================================================================]]--

Config = Config or {}

-- ===========================================================================
--  1.  MODULES
--  ---------------------------------------------------------------------------
--  Everything below the core store system is opt-in.  Leave them off for a
--  plain, fast store script; switch one on when you want the extra depth.
-- ===========================================================================

Config.Modules = {
    -- Supply-and-demand pricing.  Prices fall when players flood the market
    -- with an item and recover while it is scarce.  Tracked per item, shared
    -- server-wide.  See config/pricing.lua.
    dynamicPricing = true,

    -- Commodities exchange.  A trading floor NPC where players hold long and
    -- short positions on tracked commodities.  Requires dynamicPricing.
    -- See config/exchange.lua.
    exchange = true,

    -- Ghost buyers.  Simulated NPC customers occasionally buy from player
    -- shops so owners still see income on a quiet server.
    -- See config/ghostbuyer.lua.
    ghostBuyer = true,
}

-- ===========================================================================
--  2.  GENERAL
-- ===========================================================================

-- Print verbose logs to the server console.  Leave false on a live server.
Config.debug = false

-- How close a player must stand before the shop prompt appears (metres).
Config.interactionDistance = 1.5

-- Key that opens a shop.  See config/keys.lua for the full list of hashes.
Config.openKey = "G"

-- Key that buys a storefront, shown only at unowned purchasable stores.
-- It is deliberately not the same key as the one that opens a shop, and it
-- asks for a second press before taking any money.
Config.purchaseKey = "B"

-- Clerk used when a store's configured model does not exist on this server.
-- Must be a model you know loads; this one is used across RedM servers.
Config.fallbackClerkModel = "cs_mp_travellingsaleswoman"

-- How many days of history the dashboard revenue chart covers.  The chart's
-- heading is generated from this number, so the two cannot disagree.
-- Top sellers and top earners are always all-time.
Config.analyticsChartDays = 14

-- Currency symbol used throughout the UI.
Config.currency = "$"

-- Items that can never be stocked, bought, or sold at any shop, no matter
-- what a config or a player tries.  Use this for quest items, licences,
-- admin tools -- anything that would break your server if it became tradeable.
Config.blacklistedItems = {
    -- "some_quest_item",
}

-- ===========================================================================
--  3.  PLAYER-OWNED SHOPS
-- ===========================================================================

Config.PlayerShops = {
    -- Master switch.  Off = NPC stores only, no player ownership at all.
    enabled = true,

    -- Item a player consumes to found a new shop where they stand.
    -- Set to false to disable player-founded shops entirely (players can then
    -- only own the storefronts you marked purchasable in config/stores.lua).
    creationItem = "shoptoken",

    -- How many shops one character may own.
    maxPerPlayer = 2,

    -- Admins ignore maxPerPlayer.
    adminBypassMax = true,

    -- Storage slots a brand-new shop starts with.
    startingSlots = 1000,

    -- Cost per additional storage slot when upgrading.
    slotUpgradeCost = 0.75,

    -- Minimum distance between two player shops (metres).
    minSpacing = 10.0,

    -- Cost for an owner to relocate their shop.
    relocateCost = 750,

    -- Blip shown for player-owned shops.
    blipSprite = -242384756,

    -- Let owners set their own Discord webhook for shop logs.
    allowOwnerWebhooks = true,
}

-- ---------------------------------------------------------------------------
--  Taxes and repossession
--  A shop that cannot pay its tax on collection day is flagged as repossessed:
--  it disappears from the map and the owner loses access until an admin
--  restores it.  Nothing is deleted.
-- ---------------------------------------------------------------------------
Config.Tax = {
    -- Set false to switch taxes and repossession off completely.
    enabled = true,

    -- Amount taken from each shop ledger on collection day.
    amount = 75,

    -- "monthly" or "weekly".
    schedule = "monthly",

    -- Monthly: collect on this day of the month, at this time.
    monthly = { day = 1, hour = 12, minute = 0 },

    -- Weekly: collect on these days of the month, at this time.
    weekly  = { days = { 3, 10, 17, 24 }, hour = 6, minute = 10 },
}

-- ===========================================================================
--  4.  STAFF AND PERMISSIONS
--  ---------------------------------------------------------------------------
--  A shop owner can hire other characters directly from the shop UI.  Each
--  hire gets a role, and the role decides what they can do.
--
--  This is self-contained: poggy_markets does not need a jobs script, and it
--  will not change anyone's job unless you turn on Config.ShopJobs below.
--  With Config.ShopJobs on, a shop's job grade per role is set there too.
-- ===========================================================================

Config.Roles = {
    owner = {
        viewDashboard = true, viewInventory = true, depositItems   = true,
        setPrices     = true, removeStock   = true, manageBuyList  = true,
        viewLedger    = true, depositLedger = true, withdrawLedger = true,
        viewAnalytics = true, viewSoldItems = true, manageStaff    = true,
        changeSettings = true,   -- rename, blip, upgrade, relocate, webhook
    },
    manager = {
        viewDashboard = true, viewInventory = true, depositItems   = true,
        setPrices     = true, removeStock   = true, manageBuyList  = true,
        viewLedger    = true, depositLedger = true, withdrawLedger = true,
        viewAnalytics = true, viewSoldItems = true, manageStaff    = false,
        changeSettings = false,
    },
    employee = {
        viewDashboard = true, viewInventory = true, depositItems   = true,
        setPrices     = false, removeStock  = false, manageBuyList = false,
        viewLedger    = true, depositLedger = true, withdrawLedger = false,
        viewAnalytics = true, viewSoldItems = true, manageStaff    = false,
        changeSettings = false,
    },
}

-- ---------------------------------------------------------------------------
--  Optional: shop jobs
--  ---------------------------------------------------------------------------
--  OFF by default, and deliberately so.  When enabled, a shop that has a job
--  gives that job to its owner and to everyone the owner hires, at the grade
--  for their role (below), and takes it away again when they are let go or
--  the shop is sold, repossessed or deleted.  Job-locked content elsewhere
--  (crafting recipes, doors, boss menus) then works for shop staff.
--
--  A shop's job is set by server staff only: in /poggy -> Poggy Markets (or
--  Poggy Multijob) -> Shop jobs, with the admin command
--  /pmshopjob <shopId> <job|none|reset>, or else the store's `job` field in
--  config/stores.lua.  No player (owner or staff) can change it; the staff
--  tab shows it locked.
--
--  With poggy_multijob running, the job is ADDED to the character's job list
--  and their current job is left alone; they switch to it with /multijob.
--  Without it, the job is set directly (it replaces their current job).
--
--  Every start, and whenever a character loads in, the shop jobs are checked
--  against the staff lists and put right.  poggy_markets only ever takes away
--  a job it gave.
--
--  With this off, poggy_markets never touches a player's job, and shop access
--  is decided purely by ownership and the staff list above.
-- ---------------------------------------------------------------------------
Config.ShopJobs = {
    enabled = false,

    -- Job grade for each role at a shop that has a job: the owner gets
    -- `owner`, a hired manager `manager`, a hired employee `employee`.
    -- The same numbers decide access by job (grantAccessByJob below).
    grades = { employee = 1, manager = 2, owner = 3 },

    -- Older setting, still honoured: a number here is the grade given to the
    -- owner instead of grades.owner.  false = use grades.owner.
    ownerGrade = false,

    -- Without poggy_multijob: the job a character is moved to when a shop
    -- job is taken away from them (let go, or the shop sold, repossessed or
    -- deleted) and they are wearing it.  false = leave their job alone.
    -- With poggy_multijob, its own Config.DefaultJob is used instead.
    fallbackJob = "unemployed",

    -- Additionally grant job-based access: a character whose job matches a
    -- shop's job gets in without being on the staff list, by grade: at least
    -- grades.owner = owner, grades.manager = manager, grades.employee =
    -- employee, lower = no access.  Give two shops the same job and its
    -- holders get into both.
    grantAccessByJob = true,
}

-- ===========================================================================
--  5.  ADMIN
-- ===========================================================================

Config.Admin = {
    -- Framework groups treated as admin.
    groups = { "admin", "superadmin", "god", "owner" },

    -- An ACE that also opens the shop admin panel (/pmadmin), for staff whose
    -- framework group is not in the list above:
    --     add_ace group.moderator poggy_markets.admin allow
    -- false = groups only.
    ace = "poggy_markets.admin",

    -- Command names.  Set any to false to unregister that command.
    commands = {
        giveShop   = "pmgiveshop",   -- /pmgiveshop <serverId>       create a shop at their feet
        deleteShop = "pmdelshop",    -- /pmdelshop <shopId>          delete permanently
        moveShop   = "pmmoveshop",   -- /pmmoveshop <shopId> <x> <y> <z>
        repoShop   = "pmreposhop",   -- /pmreposhop <shopId>         hide from players
        unrepoShop = "pmunreposhop", -- /pmunreposhop <shopId>       restore
        storeHere  = "pmhere",       -- /pmhere <type> <name...>     print a ready-to-paste
                                     --   store block for wherever you are standing.
                                     --   This is how you add stores; you never have to
                                     --   hand-write coordinates.
        shopJob    = "pmshopjob",    -- /pmshopjob <shopId> [job|none|reset]  show or set a
                                     --   shop's job (Config.ShopJobs); /poggy's Shop jobs
                                     --   panel does the same.  none = no job,
                                     --   reset = back to config/stores.lua.
        shopAdmin  = "pmadmin",      -- /pmadmin  the shop admin panel: every shop in a
                                     --   searchable table; route to it, open its manager,
                                     --   transfer, repossess / restore, adjust the ledger,
                                     --   remove staff, set its job.
    },

    -- Discord webhook for admin-level shop events. Paste your webhook URL here;
    -- an empty value ("") or this placeholder disables it.
    webhook = "YOUR DISCORD WEBHOOK HERE",
    webhookAvatar = "",
}

-- ---------------------------------------------------------------------------
--  Owners handing a shop over
--  ---------------------------------------------------------------------------
--  An owner can give their shop to a player standing near them, from the
--  manager's Settings tab: they type the shop's name to confirm, and the other
--  player has to accept.  Stock, ledger and staff go with the shop.  A
--  storefront reserved for a job (purchaseJobs in config/stores.lua) can only
--  go to someone holding that job, and nobody can end up with more than
--  Config.PlayerShops.maxPerPlayer shops.  Admins can transfer any shop to
--  anyone with /pmadmin.
-- ---------------------------------------------------------------------------
Config.Transfers = {
    -- false = only admins can move a shop to a new owner.
    enabled = true,

    -- How close (metres) the other player must be.
    distance = 10.0,

    -- Seconds the other player has to accept.
    offerSeconds = 60,
}

-- ===========================================================================
--  6.  SELLING TO SHOPS
-- ===========================================================================

Config.Selling = {
    -- Honour item durability/decay when players sell to a shop.
    -- Items below `minCondition` are refused.  Requires an inventory that
    -- exposes a condition value in item metadata; ignored otherwise.
    useCondition = false,
    minCondition = 70,

    -- Preserve item metadata when items move between player and shop.
    keepMetadata = true,
}

-- ===========================================================================
--  7.  INTERFACE
-- ===========================================================================

Config.UI = {
    -- The look of the shop, store manager and exchange panels.
    --   "default"  the built-in dark theme
    --   "leather"  stitched leather and parchment, in the style of the
    --              game's own satchel (ui/css/skin-leather.css)
    -- Any other name loads ui/css/skin-<name>.css, so a skin of your own is
    -- a copy of skin-leather.css under a new name, plus its images.
    skin = "leather",
}
