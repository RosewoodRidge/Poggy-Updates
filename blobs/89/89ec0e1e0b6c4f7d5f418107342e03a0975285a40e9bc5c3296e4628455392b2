--[[===========================================================================
    poggy_markets · config/stores.lua
    ---------------------------------------------------------------------------
    Where the shops are.

    HOW THIS WORKS
    --------------
    A store type (below) carries the defaults: which item lists it sells and
    buys, what its map blip looks like, and which clerk stands behind the
    counter.  A store entry then only has to say WHERE it is and WHAT TYPE it
    is.  Everything else is optional.

    So a complete, working store is three lines:

        { type = "generalstore", name = "Valentine General Store",
          coords = vector3(-325.42, 797.53, 117.88), heading = 88.0,
      npc = false },

    If that type has a clerk, the clerk comes with it.  There is no second NPC
    file to keep in sync -- delete the store and its clerk goes too.

    Most town shops have no clerk of their own, because the game already puts a
    shopkeeper behind those counters.  Only the ones that trade from a stall or
    a yard get one.  See the note above Config.StoreTypes.

    ADDING A STORE WITHOUT COUNTING PIXELS
    --------------------------------------
    Stand where you want the counter, face the way the clerk should face, and
    run:

        /pmhere generalstore Valentine General Store

    It prints a finished block to your console, ready to paste below.
===========================================================================]]--

Config = Config or {}

-- ===========================================================================
--  BLIP SPRITES
--  ---------------------------------------------------------------------------
--  RedM blip sprite hashes are build-dependent, so poggy_markets ships only
--  the two that are known-good, and every store type points at one of them.
--  Swap in your own hashes here and every store of that type follows.
-- ===========================================================================

Config.BlipSprites = {
    shop      = 1475879922,   -- generic storefront
    outfitter = 1202244626,   -- camp / trapper
}

-- ===========================================================================
--  STORE TYPES
--  ---------------------------------------------------------------------------
--  sell / buy refer to list names in your catalog file (see config/catalog_*).
--  Set either to false when the store does not trade that direction.
--
--  blipSprite is a name from Config.BlipSprites above ("shop", "outfitter"),
--  or a sprite hash written as a number.
--
--  npcModel is the clerk.  false means the store has no clerk of its own, which
--  is right for anywhere the game already places a shopkeeper behind the
--  counter -- general stores, gunsmiths, blacksmiths, saloons, stables and
--  doctors all have one already, and adding a second gives you two people
--  standing in the same spot.  Types that trade from a stall or a yard, where
--  the game leaves nobody, get a clerk.
-- ===========================================================================

Config.StoreTypes = {
    generalstore = {
        label      = "General Store",
        sell       = "GeneralStore",
        buy        = "GeneralStore",
        blipSprite = "shop",
        npcModel   = false,
    },
    gunsmith = {
        label      = "Gunsmith",
        sell       = "Gunsmith",
        buy        = "Gunsmith",
        blipSprite = "shop",
        npcModel   = false,
    },
    blacksmith = {
        label      = "Blacksmith",
        sell       = "Blacksmith",
        buy        = "Blacksmith",
        blipSprite = "shop",
        npcModel   = false,
    },
    horsesupply = {
        label      = "Horse Supply",
        sell       = "HorseSupply",
        buy        = "HorseSupply",
        blipSprite = "shop",
        npcModel   = false,
    },
    saloon = {
        label      = "Saloon",
        sell       = "Saloon",
        buy        = "Saloon",
        blipSprite = "shop",
        npcModel   = false,
    },
    butcher = {
        label      = "Butcher",
        sell       = false,          -- buys only; nothing on the shelves
        buy        = "Butcher",
        blipSprite = "shop",
        npcModel   = "S_M_M_UNIBUTCHERS_01",
    },
    doctor = {
        label      = "Doctor",
        sell       = "Doctor",
        buy        = false,
        blipSprite = "shop",
        npcModel   = false,
    },
    fishmarket = {
        label      = "Fish Market",
        sell       = "Fish",
        buy        = "Fish",
        blipSprite = "shop",
        npcModel   = "mp_dr_u_f_m_MISSINGFISHERMAN_01",
    },
    lumber = {
        label      = "Lumber Yard",
        sell       = "Lumber",
        buy        = "Lumber",
        blipSprite = "shop",
        npcModel   = "s_m_m_strlumberjack_01",
    },
    outfitter = {
        label      = "Camp Outfitter",
        sell       = "Camping",
        buy        = false,
        blipSprite = "outfitter",
        npcModel   = "re_beartrap_males_01",
    },
}

-- ===========================================================================
--  STORES
--  ---------------------------------------------------------------------------
--  Required : type, name, coords
--  Optional : heading      which way the clerk faces (degrees, default 0)
--             blip         false to hide from the map (default true)
--             blipSprite   override the type's blip: a Config.BlipSprites
--                          name or a sprite hash
--             sell / buy   override the type's item lists
--             npc          false for no clerk, true for the type's clerk (or
--                          the fallback clerk) on the prompt, or a table:
--                            { model    = "...",
--                              coords   = vector3(x, y, z),   -- where the CLERK stands
--                              heading  = 0.0,
--                              grounded = false }
--                          `coords` is absolute, not an offset.  A clerk is
--                          usually not standing where the customer is -- see
--                          the note below.
--             jobLock      { "sheriff" } -- only these jobs may enter
--             purchasable  offer this storefront for sale.  On its own this
--                          does nothing: the store must ALSO have a shopId
--                          pointing at a row in `playershops`, and that row
--                          must be unowned or repossessed.  A shop someone
--                          already owns never shows a FOR SALE prompt.
--             price        cost to buy it.  Falls back to the `price` column
--                          on the playershops row when left out.
--             purchaseJobs { "valstables" } -- only characters holding one
--                          of these jobs may buy it (the job they wear, or
--                          any job on their poggy_multijob list).  Left out,
--                          anyone with the money may.  Admins always may.
--             job          the shop's job: given to its owner and staff
--                          (only applies when Config.ShopJobs.enabled;
--                          an admin can override it with /pmshopjob)
--
--             shopId       the row in `playershops` this storefront owns.
--                          Set it if you already have rows for your shops --
--                          ownership, stock and ledger then carry straight
--                          over, and the FOR SALE prompt appears at that
--                          row's coordinates rather than at the counter.
--                          Leave it out and a row is created on first sale.
--
--             purchaseCoords  where the FOR SALE prompt appears, when it is
--                          not the counter and not the row's coordinates.
--                          Most shops put the sale at the front door.
--
--  THREE LOCATIONS, NOT ONE
--  ------------------------
--  A shop can involve three different points, and conflating them is the
--  fastest way to end up with a clerk you cannot reach:
--
--    store.coords      where the PLAYER stands to get the prompt
--    npc.coords        where the CLERK stands -- behind a counter, through a
--                      service window, on the far side of a stall.  Often a
--                      couple of metres away and a metre lower.  If you leave
--                      this out, the clerk spawns on top of the prompt, which
--                      is wrong nearly everywhere.
--    purchaseCoords    where the FOR SALE prompt appears, resolved as
--                      purchaseCoords -> the playershops row's coords -> the
--                      prompt.
--
--  `npc.offset` still works as a relative alternative to npc.coords, but
--  absolute coordinates are clearer and are what /pmhere prints.
--
--  Coordinates below are verified in-world positions.  `grounded = true` snaps
--  the clerk to the ground; leave it false when the store is under a roof,
--  which is most of them.
-- ===========================================================================

--  NOTE ON BUYING
--  --------------
--  No store below is for sale.  Which storefronts players may buy, and for how
--  much, depends entirely on your own `playershops` rows -- so shipping guesses
--  here would put your admin shops on the market at prices nobody chose.
--
--  To offer one for sale, add its row id and a price:
--
--      { type = "generalstore", name = "Valentine General Store",
--        coords = vector3(-325.42, 797.53, 117.88), heading = 88.0,
--        shopId = 16, purchasable = true, price = 3000 },
--
--  The FOR SALE prompt then appears at that row's coordinates, and disappears
--  the moment somebody buys it.

Config.Stores = {

    -- ─── GENERAL STORES ────────────────────────────────────────────────────
    { type = "generalstore", name = "Valentine General Store",
      coords = vector3(-325.42, 797.53, 117.88),   heading = 88.0,
      npc = false },

    { type = "generalstore", name = "Rhodes General Store",
      coords = vector3(1326.83, -1288.95, 77.02),  heading = 300.0,
      npc = false },

    { type = "generalstore", name = "Saint Denis General Store",
      coords = vector3(2833.00, -1313.53, 46.76),  heading = 180.0,
      npc = false },

    { type = "generalstore", name = "Blackwater General Store",
      coords = vector3(-787.86, -1326.86, 43.88),  heading = 90.0,
      npc = false },

    { type = "generalstore", name = "Strawberry General Store",
      coords = vector3(-1795.34, -385.79, 160.33), heading = 130.0,
      npc = false },

    { type = "generalstore", name = "Armadillo General Store",
      coords = vector3(-3684.88, -2629.25, -13.43), heading = 0.0,
      npc = false },

    { type = "generalstore", name = "Annesburg General Store",
      coords = vector3(2929.11, 1369.65, 45.19),   heading = 220.0,
      npc = false },

    { type = "generalstore", name = "Tumbleweed General Store",
      coords = vector3(-5491.15, -2937.72, -0.40), heading = 255.0,
      npc = false },

    { type = "generalstore", name = "Van Horn General Store",
      coords = vector3(3023.78, 567.37, 44.71),    heading = 180.0,
      npc = false },

    -- ─── GUNSMITHS ─────────────────────────────────────────────────────────
    { type = "gunsmith", name = "Valentine Gunsmith",
      coords = vector3(-279.54, 783.37, 119.50),   heading = 180.0,
      npc = false },

    { type = "gunsmith", name = "Rhodes Gunsmith",
      coords = vector3(1320.61, -1325.83, 77.88),  heading = 0.0,
      npc = false },

    { type = "gunsmith", name = "Saint Denis Gunsmith",
      coords = vector3(2717.28, -1280.14, 49.63),  heading = 90.0,
      npc = false },

    { type = "gunsmith", name = "Blackwater Gunsmith",
      coords = vector3(-833.12, -1264.84, 43.58),  heading = 270.0,
      npc = false },

    { type = "gunsmith", name = "Armadillo Gunsmith",
      coords = vector3(-3681.69, -2627.50, -13.43), heading = 0.0,
      npc = false },

    { type = "gunsmith", name = "Annesburg Gunsmith",
      coords = vector3(2945.56, 1316.20, 44.82),   heading = 180.0,
      npc = false },

    { type = "gunsmith", name = "Tumbleweed Gunsmith",
      coords = vector3(-5508.50, -2961.94, -0.64), heading = 90.0,
      npc = false },

    -- ─── BUTCHERS  (buy pelts and meat from players) ───────────────────────
    { type = "butcher", name = "Valentine Butcher",
      coords = vector3(-341.15, 767.17, 116.79),   heading = 186.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(-339.23, 767.64, 115.64),
              heading = 92.6, grounded = false } },

    { type = "butcher", name = "Rhodes Butcher",
      coords = vector3(1296.43, -1279.37, 75.91),  heading = 147.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(1297.35, -1277.56, 74.95),
              heading = 147.4, grounded = false } },

    { type = "butcher", name = "Saint Denis Butcher",
      coords = vector3(2817.85, -1329.86, 46.59),  heading = 48.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(2819.21, -1331.18, 45.58),
              heading = 47.5, grounded = false } },

    { type = "butcher", name = "Blackwater Butcher",
      coords = vector3(-761.25, -1285.75, 43.70),  heading = 271.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(-763.12, -1284.39, 42.70),
              heading = 271.4, grounded = false } },

    { type = "butcher", name = "Strawberry Butcher",
      coords = vector3(-1752.91, -396.49, 156.15), heading = 186.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(-1753.20, -392.96, 155.31),
              heading = 185.7, grounded = false } },

    { type = "butcher", name = "Armadillo Butcher",
      coords = vector3(-3691.24, -2621.06, -13.66), heading = 3.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(-3691.32, -2622.97, -14.68),
              heading = 2.6, grounded = false } },

    { type = "butcher", name = "Annesburg Butcher",
      coords = vector3(2932.56, 1302.15, 44.55),   heading = 68.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(2934.49, 1301.58, 43.55),
              heading = 67.8, grounded = false } },

    { type = "butcher", name = "Tumbleweed Butcher",
      coords = vector3(-5508.30, -2948.27, -1.80), heading = 255.0,
      npc = { model = "S_M_M_UNIBUTCHERS_01", coords = vector3(-5509.96, -2947.11, -2.82),
              heading = 255.3, grounded = false } },

    -- ─── DOCTORS ───────────────────────────────────────────────────────────
    { type = "doctor", name = "Valentine Doctor",
      coords = vector3(-286.24, 804.94, 119.46),   heading = 180.0,
      npc = false },

    { type = "doctor", name = "Rhodes Doctor",
      coords = vector3(1369.74, -1308.37, 78.04),  heading = 120.0,
      npc = false },

    { type = "doctor", name = "Saint Denis Doctor",
      coords = vector3(2718.54, -1233.32, 50.44),  heading = 0.0,
      npc = false },

    { type = "doctor", name = "Blackwater Doctor",
      coords = vector3(-788.21, -1304.51, 43.77),  heading = 90.0,
      npc = false },

    { type = "doctor", name = "Strawberry Doctor",
      coords = vector3(-1804.37, -429.72, 158.90), heading = 76.0,
      npc = false },

    { type = "doctor", name = "Armadillo Doctor",
      coords = vector3(-3731.53, -2634.82, -12.79), heading = 267.0,
      npc = false },

    { type = "doctor", name = "Annesburg Doctor",
      coords = vector3(2924.28, 1351.98, 44.83),   heading = 220.0,
      npc = false },

    -- ─── FISH MARKETS ──────────────────────────────────────────────────────
    { type = "fishmarket", name = "Blackwater Fish Market",
      coords = vector3(-762.65, -1359.54, 43.71),  heading = 96.0,
      npc = false },

    { type = "fishmarket", name = "Saint Denis Fish Market",
      coords = vector3(2663.85, -1505.82, 45.97),  heading = 2.0,
      npc = { model = "mp_dr_u_f_m_MISSINGFISHERMAN_01", coords = vector3(2663.89, -1505.70, 44.97),
              heading = 2.1, grounded = false } },

    { type = "fishmarket", name = "Annesburg Fish Market",
      coords = vector3(2983.96, 1402.61, 44.19),   heading = 112.0,
      npc = { model = "mp_dr_u_f_m_MISSINGFISHERMAN_01", coords = vector3(2983.96, 1402.61, 43.19),
              heading = 112.5, grounded = false } },

    -- ─── LUMBER YARDS ──────────────────────────────────────────────────────
    { type = "lumber", name = "Blackwater Lumber Yard",
      coords = vector3(-856.30, -1290.81, 43.44),  heading = 8.0,
      npc = { model = "s_m_m_strlumberjack_01", coords = vector3(-856.30, -1290.81, 42.44),
              heading = 8.0, grounded = false } },

    { type = "lumber", name = "Saint Denis Lumber Yard",
      coords = vector3(2744.13, -1486.52, 45.40),  heading = 7.0,
      npc = { model = "s_m_m_strlumberjack_01", coords = vector3(2744.13, -1486.52, 44.40),
              heading = 7.4, grounded = false } },

    { type = "lumber", name = "Valentine Lumber Yard",
      coords = vector3(-297.32, 739.06, 117.62),   heading = 302.0,
      npc = { model = "s_m_m_strlumberjack_01", coords = vector3(-297.32, 739.06, 116.62),
              heading = 301.9, grounded = false } },

    { type = "lumber", name = "Strawberry Lumber Yard",
      coords = vector3(-1824.27, -426.06, 159.95), heading = 76.0,
      npc = { model = "s_m_m_strlumberjack_01", coords = vector3(-1824.27, -426.06, 158.95),
              heading = 76.4, grounded = false } },

    { type = "lumber", name = "Rhodes Lumber Yard",
      coords = vector3(1377.04, -1275.99, 77.50),  heading = 226.0,
      npc = { model = "s_m_m_strlumberjack_01", coords = vector3(1377.04, -1275.99, 76.50),
              heading = 225.8, grounded = false } },

    { type = "lumber", name = "Armadillo Lumber Yard",
      coords = vector3(-3726.09, -2621.59, -13.30), heading = 267.0,
      npc = { model = "s_m_m_strlumberjack_01", coords = vector3(-3726.09, -2621.59, -14.30),
              heading = 267.5, grounded = false } },

    -- ─── CAMP OUTFITTERS ───────────────────────────────────────────────────
    { type = "outfitter", name = "Valentine Camp Outfitter",
      coords = vector3(-362.79, 721.18, 116.36),   heading = 32.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(-362.79, 721.18, 115.36),
              heading = 32.0, grounded = false } },

    { type = "outfitter", name = "Rhodes Camp Outfitter",
      coords = vector3(1324.06, -1333.94, 77.45),  heading = 180.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(1324.06, -1333.94, 76.45),
              heading = 180.4, grounded = false } },

    { type = "outfitter", name = "Blackwater Camp Outfitter",
      coords = vector3(-946.69, -1387.32, 50.64),  heading = 294.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(-946.69, -1387.32, 49.64),
              heading = 294.5, grounded = false } },

    { type = "outfitter", name = "Strawberry Camp Outfitter",
      coords = vector3(-1767.73, -431.58, 155.27), heading = 105.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(-1767.73, -431.58, 154.27),
              heading = 105.4, grounded = false } },

    { type = "outfitter", name = "Armadillo Camp Outfitter",
      coords = vector3(-3689.85, -2553.15, -13.57), heading = 2.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(-3689.85, -2553.15, -14.57),
              heading = 2.3, grounded = false } },

    { type = "outfitter", name = "Annesburg Camp Outfitter",
      coords = vector3(2976.86, 1429.50, 44.71),   heading = 221.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(2976.86, 1429.50, 43.71),
              heading = 221.3, grounded = false } },

    { type = "outfitter", name = "Tumbleweed Camp Outfitter",
      coords = vector3(-5488.93, -2869.91, -5.01), heading = 7.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(-5488.93, -2869.91, -6.01),
              heading = 7.5, grounded = false } },

    { type = "outfitter", name = "Colter Camp Outfitter",
      coords = vector3(-1336.81, 2405.63, 307.25), heading = 343.0,
      npc = { model = "re_beartrap_males_01", coords = vector3(-1336.81, 2405.63, 306.25),
              heading = 343.3, grounded = false } },

    -- ─── ADD YOUR OWN BELOW ────────────────────────────────────────────────
    --
    --  Blacksmiths, saloons and horse supply stores are wired up as types but
    --  ship without locations, because the right spot depends on which
    --  interiors and map edits your server runs.  Stand where you want one and
    --  run  /pmhere blacksmith Valentine Blacksmith  to generate the block.
    --
    --  Example with every optional field in use:
    --
    --  { type        = "saloon",
    --    name        = "Valentine Saloon",
    --    coords      = vector3(-311.62, 803.63, 119.39),
    --    heading     = 180.0,
    --    blip        = true,
    --    blipSprite  = 1475879922,       -- override the type's blip
    --    sell        = "Saloon",         -- override the type's sell list
    --    buy         = false,            -- this one buys nothing
    --    jobLock     = { "bartender" },  -- only bartenders may open it
    --    purchasable = true,
    --    price       = 7500,
    --    job         = "saloon_valentine",
    --    npc         = { model    = "cs_mp_oldman_jones",
    --                    offset   = vector3(0.0, 0.5, 0.0),
    --                    heading  = 175.0,
    --                    grounded = false },
    --  },
}
