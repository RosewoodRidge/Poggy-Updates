--[[===========================================================================
    poggy_markets · config/exchange.lua
    ---------------------------------------------------------------------------
    The commodities exchange.  Only read when Config.Modules.exchange is true,
    and it also needs Config.Modules.dynamicPricing, because there is nothing
    to trade against without moving prices.

    THE IDEA
    --------
    A trading floor NPC where players speculate on the markets defined in
    config/pricing.lua.  They hold positions rather than items: go long
    if you think pelts are cheap, short if you think they are about to crash.
    Positions settle in cash against the live market price.

    This is a period-appropriate commodities desk, not a stock market -- there
    are no companies, only goods your players already trade.

    A caution worth taking seriously: an exchange lets players make money
    without leaving town.  The position cap and the trade fee below are what
    keep that in check.  Do not raise them without watching what happens.
===========================================================================]]--

Config = Config or {}

Config.Exchange = {

    -- ── WHERE ──────────────────────────────────────────────────────────────
    -- Each desk is a clerk you can walk up to.  Stand where you want one and
    -- run /pmhere to get the coordinates, then paste them here.
    desks = {
        { name   = "Blackwater Exchange",
          coords = vec3(-852.84, -1233.46, 43.46), heading = 357.0,
          npc    = "mp_u_m_m_trader_01", blip = true },

        { name   = "Saint Denis Exchange",
          coords = vec3(2647.05, -1294.80, 51.25), heading = 297.0,
          npc    = "mp_u_m_m_trader_01", blip = true },

        { name   = "Valentine Exchange",
          coords = vec3(-303.22, 772.56, 117.70),  heading = 106.0,
          npc    = "mp_u_m_m_trader_01", blip = true },
    },

    -- ── WHICH MARKETS ARE TRADEABLE ────────────────────────────────────────
    -- Market keys from Config.Pricing.markets.  Leave a market out to let it
    -- move with the economy without letting anyone speculate on it.
    tradableMarkets = { "pelts", "fish" },

    -- ── LIMITS ─────────────────────────────────────────────────────────────
    -- Largest position, in units, a character may hold in one commodity.
    -- This is the main brake on someone cornering a market.
    maxPositionSize = 100,

    -- Largest number of separate open positions per character.
    maxOpenPositions = 10,

    -- Fee charged on both opening and closing, on top of the pricing module's
    -- own spread.  This is what stops rapid in-and-out trading being free.
    tradeFee = 0.015,

    -- ── SHORT SELLING ──────────────────────────────────────────────────────
    -- Shorting lets players profit from a falling market, which makes dumping
    -- goods strategic rather than just annoying.  It also lets a player lose
    -- more than they staked, so it is off by default.
    allowShorts = false,

    -- Collateral held against a short, as a fraction of position value.
    -- Only meaningful when allowShorts is true.
    shortMargin = 0.50,

    -- ── ACCOUNTS ───────────────────────────────────────────────────────────
    -- Players fund a separate exchange balance rather than trading straight
    -- from their pocket, so a bad position cannot leave them unable to eat.
    useSeparateAccount = true,
    minimumDeposit     = 10.0,
}
