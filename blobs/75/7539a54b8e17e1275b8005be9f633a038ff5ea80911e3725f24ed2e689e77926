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
--  Optional: tie shop ownership to a job
--  ---------------------------------------------------------------------------
--  OFF by default, and deliberately so.  When enabled, buying a storefront
--  that declares a `job` in config/stores.lua also sets the buyer's job.
--
--  Leave this OFF if another resource already manages jobs -- otherwise both
--  will fight over the same character and the last writer wins.  With it off,
--  poggy_markets never touches a player's job, and shop access is decided
--  purely by ownership and the staff list above.
-- ---------------------------------------------------------------------------
Config.ShopJobs = {
    enabled = false,

    -- Grade given to the owner when their job is set.
    ownerGrade = 4,

    -- Job a character is moved to when they sell or lose their shop.
    -- Set to false to leave their job alone on sale.
    fallbackJob = "unemployed",

    -- Additionally grant job-based access: a character whose job matches a
    -- store's `job` field gets in without being on the staff list.
    -- Grade 3+ = owner, 2 = manager, 1 = employee, 0 = no access.
    grantAccessByJob = true,
}

-- ===========================================================================
--  5.  ADMIN
-- ===========================================================================

Config.Admin = {
    -- Framework groups treated as admin.
    groups = { "admin", "superadmin", "god", "owner" },

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
    },

    -- Discord webhook for admin-level shop events. Paste your webhook URL here;
    -- an empty value ("") or this placeholder disables it.
    webhook = "YOUR DISCORD WEBHOOK HERE",
    webhookAvatar = "",
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
