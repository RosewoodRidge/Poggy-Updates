--[[===========================================================================
    poggy_markets · config/pricing.lua
    ---------------------------------------------------------------------------
    Dynamic pricing.  Only read when Config.Modules.dynamicPricing is true.

    THE IDEA
    --------
    Every tracked item carries a multiplier that starts at 1.0.  Players
    flooding the market with an item push its multiplier down; scarcity lets it
    drift back up.  The price a store quotes is the config price times that
    multiplier, clamped between a floor and a ceiling.

    The original build hardcoded this to pelts and ores.  Here you declare what
    to track, so it works for fish, crops, lumber, moonshine, or nothing at all.

    A word of warning before switching this on: dynamic pricing changes the
    economy of your whole server, not just one shop.  Start with one category
    and watch it for a week before adding more.
===========================================================================]]--

Config = Config or {}

Config.Pricing = {

    -- ── WHAT TO TRACK ──────────────────────────────────────────────────────
    -- Each entry becomes an independent market.  Items in the same market
    -- influence each other: dumping one pelt softens prices on all of them.
    --
    --   lists    = catalog list names whose BUY entries are tracked
    --   patterns = Lua patterns matched against item names, for when an item
    --              belongs to a market but not to a single catalog list
    --   exclude  = item names to leave out entirely
    --
    -- Comment out a market to stop tracking it.
    markets = {
        pelts = {
            label    = "Pelts & Hides",
            lists    = { "Butcher" },
            patterns = { "_pelt$", "_hide$", "_skin$" },
            exclude  = {},
        },

        fish = {
            label    = "Fish",
            lists    = { "Fish" },
            patterns = {},
            exclude  = {},
        },

        -- Uncomment to add an ore and metal market:
        -- ores = {
        --     label    = "Ore & Metal",
        --     lists    = { "Blacksmith" },
        --     patterns = { "_ore$", "_ingot$" },
        --     exclude  = { "iron_ingot" },
        -- },
    },

    -- ── HOW FAR PRICES CAN MOVE ────────────────────────────────────────────
    -- Multipliers are clamped to this band.  0.20 / 1.80 means a price can
    -- fall to a fifth of its config value or rise to nearly double it.
    floor   = 0.20,
    ceiling = 1.80,

    -- ── HOW FAST PRICES MOVE ───────────────────────────────────────────────
    -- Per unit traded.  These are small on purpose: at 0.012, it takes roughly
    -- 65 pelts to halve the price, which is a busy evening rather than one
    -- player with a full satchel.
    decayPerUnit  = 0.012,   -- multiplier lost per unit players SELL to a shop
    demandPerUnit = 0.008,   -- multiplier gained per unit players BUY

    -- Recovery pulls the multiplier back toward 1.0 while an item is not
    -- being traded.  Weak, so a crashed price stays crashed for a while.
    recoveryPerHour = 0.004,

    -- After this long with no sales an item is treated as scarce and recovers
    -- faster, and may drift above 1.0 toward the ceiling.
    scarcityHours = 24,
    scarcityBonus = 0.006,

    -- How much of a trade's pressure spills onto other items in the same
    -- market.  0 keeps every item independent; 0.4 means dumping one pelt
    -- moves its siblings by 40% as much.
    spillover = 0.40,

    -- ── SPREAD ─────────────────────────────────────────────────────────────
    -- Applied on top of the multiplier so buying and selling the same item
    -- back to back is always a small loss.  Without this, players can farm the
    -- gap whenever a price ticks.
    tradeFee = 0.015,        -- 1.5% each way, so a 3% round trip

    -- ── BACKGROUND DRIFT ───────────────────────────────────────────────────
    -- Small random movement so prices are never perfectly static.  Set
    -- enabled = false for a market that only ever reacts to real trades.
    drift = {
        enabled    = true,
        intervalMs = 300000,  -- recalculate every 5 minutes
        marketMax  = 0.0020,  -- largest market-wide move per tick
        itemMax    = 0.0018,  -- largest per-item jitter per tick
        momentum   = 0.88,    -- how much of the previous move carries over
    },

    -- ── HOUSEKEEPING ───────────────────────────────────────────────────────
    -- Price history, used by the exchange module's charts.  Snapshots are
    -- cheap but they add up; anything older than historyDays is pruned.
    snapshotIntervalMs = 3600000,   -- one snapshot per hour
    historyDays        = 30,
}
