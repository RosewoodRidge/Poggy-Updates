--[[===========================================================================
    poggy_markets · config/ghostbuyer.lua
    ---------------------------------------------------------------------------
    Ghost buyers.  Only read when Config.Modules.ghostBuyer is true.

    THE IDEA
    --------
    On a quiet server a player shop can sit untouched for days, which makes
    owning one feel pointless.  Ghost buyers simulate the trickle of custom a
    real store would get: every so often an unseen customer buys a few items,
    stock goes down, and the ledger goes up.

    Defaults here are deliberately stingy.  A shop earns a few dollars a day,
    not a living.  Turn the numbers up only if your server is genuinely empty.
===========================================================================]]--

Config = Config or {}

Config.GhostBuyer = {

    -- ── TIMING ─────────────────────────────────────────────────────────────
    -- Each shop runs its own cycle.  When one block ends the next is
    -- scheduled with a fresh random length, so shops do not all fire at once.
    blockMinHours = 7,
    blockMaxHours = 12,

    -- How often to look for shops that have opened since the last scan.
    newShopScanMinutes = 30,

    -- ── ELIGIBILITY ────────────────────────────────────────────────────────
    -- A shop needs at least this many units in stock before anyone shops there.
    -- An almost-empty store getting customers reads as fake.
    minTotalStock = 10,

    -- Only items the owner has made visible are bought.  Hidden items are
    -- usually back-stock the owner does not want touched.
    visibleItemsOnly = true,

    -- ── HOW MANY SALES PER BLOCK ───────────────────────────────────────────
    -- Tiers are matched by total stock; the last tier whose `min` is at or
    -- below the shop's stock wins.  Weights are relative, so they do not have
    -- to add up to anything in particular.
    tiers = {
        { min = 10, weights = { [0] = 70, [1] = 30 } },
        { min = 30, weights = { [0] = 50, [1] = 35, [2] = 15 } },
        { min = 60, weights = { [0] = 30, [1] = 35, [2] = 20, [3] = 10, [4] = 5 } },
    },

    -- ── HOW MANY UNITS PER SALE ────────────────────────────────────────────
    quantityWeights = { [1] = 60, [2] = 30, [4] = 10 },

    -- ── ANTI-ABUSE ─────────────────────────────────────────────────────────
    -- Without these, a player stocks one item at $999,999 and prints money.
    --
    --   maxItemPrice      items dearer than this are never bought (0 = no cap)
    --   minItemStock      an item needs at least this many units to qualify
    --   maxRevenuePerSale hard ceiling on one sale, whatever the maths says
    maxItemPrice      = 25.0,
    minItemStock      = 3,
    maxRevenuePerSale = 50.0,

    -- ── PAYMENT ────────────────────────────────────────────────────────────
    -- "ledger" keeps ghost income on the same path as real income, which is
    -- what you want: the owner still has to visit and withdraw.
    -- "cash" pays the owner directly when they are online.
    payTo = "ledger",

    -- ── FEEDBACK ───────────────────────────────────────────────────────────
    -- Telling owners about every ghost sale makes the illusion obvious.
    -- Leave this off unless you want players to know the system exists.
    notifyOwner   = false,
    notifyDuration = 8000,
}
