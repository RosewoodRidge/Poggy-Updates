Config = {}

-- Language: 'en' ships.  To add one, copy the Translations.en table in translations.lua.
Config.Language = 'en'

-- ============================================================================
-- DEBUG
-- ============================================================================
Config.PoggyDebug = {
    Enabled = false,
    LogToConsole = true,

    Categories = {
        CORE     = true,
        NPC      = false,
        NUI      = true,
        AUCTION  = true,
        MAILBOX  = true,
        DATABASE = false,
        BRIDGE   = false,
    },

    Level = {
        TRACE   = true,
        INFO    = true,
        WARNING = true,
        ERROR   = true,
    }
}

-- ============================================================================
-- AUCTION HOUSE LOCATIONS
-- ============================================================================
Config.Locations = {
    {
        name   = "Valentine Auction House",
        coords = vector4(-306.03, 773.49, 117.7, 334.65),
        npc = {
            model    = "cs_valauctionboss_01",
            scenario = "WORLD_HUMAN_SHOPKEEPER_MALE_A",
        },
        blip = {
            enabled = true,
            sprite  = "blip_ambient_crate",
            color   = "BLIP_MODIFIER_MP_COLOR_32",
            scale   = 0.2,
            name    = "Auction House",
        },
    },
    {
        name   = "Blackwater Auction House",
        coords = vector4(-833.21, -1349.7, 43.13, 91.54),
        npc = {
            model    = "cs_valauctionboss_01",
            scenario = "WORLD_HUMAN_SHOPKEEPER_MALE_A",
        },
        blip = {
            enabled = true,
            sprite  = "blip_ambient_crate",
            color   = "BLIP_MODIFIER_MP_COLOR_32",
            scale   = 0.2,
            name    = "Auction House",
        },
    },
    {
        name   = "Saint Denis Auction House",
        coords = vector4(2751.95, -1396.11, 45.2, 305.27),
        npc = {
            model    = "cs_valauctionboss_01",
            scenario = "WORLD_HUMAN_SHOPKEEPER_MALE_A",
        },
        blip = {
            enabled = true,
            sprite  = "blip_ambient_crate",
            color   = "BLIP_MODIFIER_MP_COLOR_32",
            scale   = 0.2,
            name    = "Auction House",
        },
    },
    {
        name   = "Armadillo Auction House",
        coords = vector4(-3668.31, -2628.68, -14.59, 358.58),
        npc = {
            model    = "cs_valauctionboss_01",
            scenario = "WORLD_HUMAN_SHOPKEEPER_MALE_A",
        },
        blip = {
            enabled = true,
            sprite  = "blip_ambient_crate",
            color   = "BLIP_MODIFIER_MP_COLOR_32",
            scale   = 0.2,
            name    = "Auction House",
        },
    },       
}

-- ============================================================================
-- INTERACTION
-- ============================================================================
Config.Interaction = {
    Distance       = 2.5,           -- Prompt activation distance
    RenderDistance  = 50.0,          -- NPC spawn / despawn distance
    Key            = 0x760A9C6F,    -- G key
    PromptText     = "Auction House",
}

-- ============================================================================
-- AUCTION RULES
-- ============================================================================
Config.Auction = {
    -- Duration options (hours) — player picks when listing
    Durations = {
        { label = "Short (2h)",      hours = 2,  depositPercent = 5  },
        { label = "Medium (8h)",     hours = 8,  depositPercent = 10 },
        { label = "Long (24h)",      hours = 24, depositPercent = 15 },
        { label = "Very Long (48h)", hours = 48, depositPercent = 20 },
    },

    -- Deposits & Taxes
    DepositEnabled    = true,           -- Charge a non-refundable deposit on listing
    SalesTaxPercent   = 5,              -- Percentage cut taken from successful sale proceeds
    MinStartPrice     = 0.50,           -- Minimum starting bid ($)
    MaxStartPrice     = 100000.00,      -- Maximum starting bid ($)
    MaxBuyoutPrice    = 500000.00,      -- Maximum buyout price ($)

    -- Bidding
    MinBidIncrement   = 0.50,           -- Minimum amount above current bid
    BidIncrementPercent = 5,            -- OR percentage of current bid (whichever is higher)

    -- Limits
    MaxActiveListings = 20,             -- Max concurrent listings per player
    MaxMailboxItems   = 50,             -- Max uncollected mailbox entries

    -- Expiration processing
    ExpirationCheckInterval = 60,       -- Seconds between server-side expiry sweeps

    -- Cancellation
    AllowCancelWithBids = false,        -- Allow sellers to cancel auctions that have bids
}

-- ============================================================================
-- CATEGORIES (for filtering in the UI)
-- ============================================================================
Config.Categories = {
    { key = "all",        label = "All Items",     icon = "🏷️" },
    { key = "weapons",    label = "Weapons",       icon = "🔫" },
    { key = "ammo",       label = "Ammunition",    icon = "💥" },
    { key = "consumable", label = "Consumables",   icon = "🍖" },
    { key = "material",   label = "Materials",     icon = "🪵" },
    { key = "herb",       label = "Herbs",         icon = "🌿" },
    { key = "animal",     label = "Animal Parts",  icon = "🦌" },
    { key = "clothing",   label = "Clothing",      icon = "👕" },
    { key = "misc",       label = "Miscellaneous", icon = "📦" },
}

-- Map item names → categories (items not listed here default to "misc")
Config.ItemCategories = {
    -- Populate with your server's items, e.g.:
    -- ["item_ammo_rifle"]         = "ammo",
    -- ["item_meat_venison"]       = "consumable",
    -- ["item_pelt_deer_perfect"]  = "animal",
    -- ["consumable_herb_yarrow"]  = "herb",
}

-- ============================================================================
-- BLACKLIST (items that CANNOT be auctioned)
-- ============================================================================
Config.BlacklistedItems = {
    "id_card",
    "weapon_license",
    -- Add any items that should never be listed
}

-- ============================================================================
-- NOTIFICATIONS
-- ============================================================================
Config.Notifications = {
    Duration = 5000,   -- ms
}

-- ============================================================================
-- DISCORD WEBHOOKS
-- ============================================================================
Config.Discord = {
    Enabled   = false,
    Webhook   = "YOUR DISCORD WEBHOOK HERE",
    BotName   = "Auction House",
    BotAvatar = "",
    Color     = 16766720,  -- Amber
    Events = {
        Listing   = true,
        Sale      = true,
        Bid       = true,
        Expired   = true,
        Cancelled = true,
    },
}

-- ============================================================================
-- SHIPMENT ORDERS (Government Supply Catalog)
-- ============================================================================
Config.Shipment = {
    ShippingFeePercent   = 12,   -- % of subtotal charged as shipping & handling
    MaxActiveOrders      = 10,   -- max unfulfilled orders per player
    MaxQtyPerOrder       = 50,   -- max quantity per single order line
    ProcessingMinMinutes = 5,   -- minimum processing stage time (minutes)
    ProcessingMaxMinutes = 15,   -- maximum processing stage time
    TransitMinMinutes    = 5,
    TransitMaxMinutes    = 15,
    DeliveryMinMinutes   = 5,
    DeliveryMaxMinutes   = 10,
    CheckInterval        = 60,   -- seconds between stage-advancement checks
    AllowCancelProcessing = true, -- can player cancel while still Processing?
    PriceVariancePercent = 5,    -- max ±% random price variance each refresh (e.g. 5 = ±5%)

    -- Let players ship an order straight into a Poggy Markets shop's storage
    -- (owners, and staff allowed to deposit items).  Needs poggy_markets
    -- running; without it the shop field is hidden and every order goes to
    -- the mailbox.  false = mailbox only.
    ShopDelivery         = true,
}

-- Source-category badges shown in the catalogue sidebar
Config.ShipmentCompanies = {
    lumberjacking = { name = "Lumberjacking",  icon = "🪵", color = "#b5845a" },
    mining        = { name = "Mining",         icon = "⛏️",  color = "#888888" },
    farming       = { name = "Farming",        icon = "🌾", color = "#5ab55a" },
    hunting       = { name = "Hunting",        icon = "🦌", color = "#c47a2b" },
    fishing       = { name = "Fishing",        icon = "🎣", color = "#5a8fb5" },
    crafted       = { name = "Crafted",        icon = "⚒️",  color = "#9b7fd4" },
}

-- Comprehensive catalog of orderable raw/intermediate ingredients.
-- 'source' matches the sidebar filter.  Price is the base price per unit;
-- the UI applies a ±PriceVariancePercent random offset each time the tab is opened.
Config.ShipmentCatalog = {
    -- ITEM NAMES MUST EXIST: the catalogue only shows an entry whose item name is in
    -- your items table; any other entry is skipped. Most names below are stock VORP
    -- items. These are NOT stock and come from other scripts (herbs, crafting,
    -- hunting), so add them to your items table or their entries are skipped:
    --   pulp, sap, bark, nitrite, fresh_mushroom, Black_Berry, Black_Currant,
    --   Desert_Sage, Wild_Mint, Lavender, Yarrow, Agarita, American_Ginseng,
    --   Bay_Bolete, porkfat, chicken_meat, eaglef, elkantler, turtleshell,
    --   gun_barrel, corn_flour
    -- ============================================================
    -- LUMBERJACKING
    -- ============================================================
    { name = "wood",        label = "Wood",           source = "lumberjacking", price = 0.50,  desc = "Raw cut timber. Foundation for construction and crafting." },
    { name = "hwood",       label = "Hardwood",       source = "lumberjacking", price = 1.50,  desc = "Dense hardwood planks. Superior strength for frames and handles." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "pulp",        label = "Wood Pulp",      source = "lumberjacking", price = 0.30,  desc = "Soft fibrous pulp processed from timber. Used in paper and cloth." },
    { name = "fibers",      label = "Plant Fibers",   source = "lumberjacking", price = 0.35,  desc = "Long plant fibers stripped from bark and stems. Used in rope and weaving." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "sap",         label = "Tree Sap",       source = "lumberjacking", price = 0.50,  desc = "Viscous sap collected from pine and hardwood trees. Adhesive and waterproofing agent." },
    { name = "honey",       label = "Honey",          source = "lumberjacking", price = 0.80,  desc = "Raw wildflower honey harvested from forest hives. Sweetener and preservative." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "bark",        label = "Tree Bark",      source = "lumberjacking", price = 0.25,  desc = "Stripped outer bark from felled trees. Used in tanning and medicine." },

    -- ============================================================
    -- MINING
    -- ============================================================
    { name = "rock",        label = "Rock",           source = "mining", price = 0.15,  desc = "Common quarry rock. Used in construction and as an abrasive." },
    { name = "sulfur",      label = "Sulfur",         source = "mining", price = 0.50,  desc = "Yellow mineral sulfur from volcanic deposits. Key ingredient in black powder." },
    { name = "salt",        label = "Salt",           source = "mining", price = 0.50,  desc = "Crystalline salt from underground deposits. Preservative and essential mineral." },
    { name = "coal",        label = "Coal",           source = "mining", price = 0.45,  desc = "Hard coal for fueling forges and furnaces. Standard smelting fuel." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "nitrite",     label = "Saltpeter",      source = "mining", price = 0.50,  desc = "Potassium nitrate crystals found in cave deposits. Used in explosives and preservatives." },

    -- ============================================================
    -- FARMING
    -- ============================================================
    { name = "wheat",            label = "Wheat",           source = "farming", price = 0.30, desc = "Bundled wheat sheaves, ready for milling." },
    { name = "corn",             label = "Corn",            source = "farming", price = 0.25, desc = "Dried field corn kernels." },
    { name = "potato",           label = "Potato",          source = "farming", price = 0.20, desc = "Firm waxy potatoes from the field." },
    { name = "carrots",          label = "Carrots",         source = "farming", price = 0.20, desc = "Crisp root carrots." },
    { name = "tomato",           label = "Tomato",          source = "farming", price = 0.25, desc = "Ripe red tomatoes from the garden." },
    { name = "orange",           label = "Orange",          source = "farming", price = 0.35, desc = "Juicy oranges from southern orchards." },
    { name = "apple",            label = "Apple",           source = "farming", price = 0.30, desc = "Crisp red apples, harvested fresh." },
    { name = "strawberry",       label = "Strawberry",      source = "farming", price = 0.40, desc = "Sweet wild strawberries." },
    { name = "blueberry",        label = "Blueberry",       source = "farming", price = 0.35, desc = "Fresh picked blueberries." },
    { name = "cherry",           label = "Cherry",          source = "farming", price = 0.40, desc = "Dark sweet cherries from orchard trees." },
    { name = "banana",           label = "Banana",          source = "farming", price = 0.45, desc = "Ripe bananas imported from the tropics." },
    { name = "pumpkin",          label = "Pumpkin",         source = "farming", price = 0.30, desc = "Large field pumpkin." },
    { name = "chocolate",        label = "Cacao Beans",     source = "farming", price = 1.20, desc = "Dried cacao beans, the base of chocolate and tonics." },
    { name = "hop",              label = "Hops",            source = "farming", price = 0.55, desc = "Dried hop cones used in brewing." },
    { name = "lemon",            label = "Lemon",           source = "farming", price = 0.35, desc = "Tart lemons for cooking and medicine." },
    { name = "olive",            label = "Olives",          source = "farming", price = 0.50, desc = "Cured olives, also pressed for oil." },
    { name = "coffeebeans",      label = "Coffee Beans",    source = "farming", price = 0.90, desc = "Roasted arabica coffee beans." },
    { name = "consumable_peach", label = "Peach",           source = "farming", price = 0.30, desc = "Soft summer peaches." },
    { name = "redpepper",        label = "Red Pepper",      source = "farming", price = 0.30, desc = "Dried red chili peppers." },
    { name = "greenpepper",      label = "Green Pepper",    source = "farming", price = 0.25, desc = "Fresh green bell peppers." },
    { name = "beans",            label = "Beans",           source = "farming", price = 0.20, desc = "Dried pinto beans." },
    -- Not stock items: these must exist in your items table, or the entries below are skipped.
    { name = "fresh_mushroom",   label = "Mushroom",        source = "farming", price = 0.35, desc = "Fresh foraged mushrooms." },
    { name = "Black_Berry",      label = "Blackberry",      source = "farming", price = 0.35, desc = "Wild blackberries picked from the thicket." },
    { name = "Black_Currant",    label = "Black Currant",   source = "farming", price = 0.35, desc = "Tart black currants." },
    { name = "Desert_Sage",      label = "Desert Sage",     source = "farming", price = 0.60, desc = "Aromatic sage from the arid south. Used in cooking and medicine." },
    { name = "Wild_Mint",        label = "Wild Mint",       source = "farming", price = 0.50, desc = "Pungent wild mint, great for teas and remedies." },
    { name = "Lavender",         label = "Lavender",        source = "farming", price = 0.60, desc = "Fragrant lavender sprigs. Calming and medicinal." },
    { name = "Yarrow",           label = "Yarrow",          source = "farming", price = 0.55, desc = "Medicinal yarrow herb with wound-healing properties." },
    { name = "Agarita",          label = "Agarita Berry",   source = "farming", price = 0.45, desc = "Tart red berries from the agarita shrub." },
    { name = "sugar",            label = "Sugar",           source = "farming", price = 0.30, desc = "Sugar grown from sugar cane." },
    -- Not stock items: these must exist in your items table, or the entries below are skipped.
    { name = "American_Ginseng", label = "American Ginseng",source = "farming", price = 1.50, desc = "Rare ginseng root with potent medicinal value." },
    { name = "Bay_Bolete",       label = "Bay Bolete",      source = "farming", price = 0.70, desc = "Choice wild mushroom with a rich earthy flavour." },

    -- ============================================================
    -- HUNTING
    -- ============================================================
    { name = "venison",         label = "Venison",          source = "hunting", price = 0.90, desc = "Fresh deer meat, lean and flavourful." },
    { name = "biggame",         label = "Big Game Meat",    source = "hunting", price = 1.20, desc = "Hearty cuts from large game animals." },
    { name = "beef",            label = "Beef",             source = "hunting", price = 1.00, desc = "Raw beef cuts from cattle." },
    { name = "pork",            label = "Pork",             source = "hunting", price = 0.80, desc = "Raw pork from wild boar or farmstead hogs." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "porkfat",         label = "Pork Fat",         source = "hunting", price = 0.50, desc = "Rendered pork fat. Cooking fat and crafting ingredient." },
    { name = "bird",            label = "Bird Meat",        source = "hunting", price = 0.70, desc = "Small game bird meat." },
    { name = "game",            label = "Small Game Meat",  source = "hunting", price = 0.60, desc = "Meat from small wild animals — rabbit, squirrel, etc." },
    { name = "eggs",            label = "Eggs",             source = "hunting", price = 0.30, desc = "Fresh eggs from farm or wild nests." },
    { name = "wool",            label = "Wool",             source = "hunting", price = 0.65, desc = "Raw unshorn wool fleece." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "chicken_meat",    label = "Chicken",          source = "hunting", price = 0.75, desc = "Whole raw chicken." },
    { name = "deerskin",        label = "Deer Hide",        source = "hunting", price = 1.00, desc = "Tanned deer hide. Used in leatherworking." },
    { name = "wolfpelt",        label = "Wolf Pelt",        source = "hunting", price = 2.50, desc = "Thick wolf fur. Premium quality pelt." },
    { name = "foxskin",         label = "Fox Skin",         source = "hunting", price = 1.80, desc = "Soft fox pelt, prized for trimming." },
    { name = "birdfeather",     label = "Bird Feathers",    source = "hunting", price = 0.40, desc = "Assorted bird feathers. Used in fletching and millinery." },
    -- Not stock items: these must exist in your items table, or the entries below are skipped.
    { name = "eaglef",          label = "Eagle Feather",    source = "hunting", price = 1.50, desc = "Large primary feather from an eagle. Rare and valuable." },
    { name = "elkantler",       label = "Elk Antler",       source = "hunting", price = 2.00, desc = "Shed elk antler rack. Carving material and trophy." },
    { name = "turtleshell",     label = "Turtle Shell",     source = "hunting", price = 1.40, desc = "Hard tortoiseshell. Used in decoration and crafting." },

    -- ============================================================
    -- FISHING
    -- ============================================================
    { name = "lobster",         label = "Lobster",          source = "fishing", price = 3.50, desc = "Live freshwater lobster caught in river traps." },
    { name = "redpearl",        label = "Red Pearl",        source = "fishing", price = 12.00, desc = "Rare red pearl harvested from river mussels." },
    { name = "goldpearl",       label = "Gold Pearl",       source = "fishing", price = 15.00, desc = "Exceptionally rare gold pearl. Highly prized." },

    -- ============================================================
    -- CRAFTED (multi-step intermediates)
    -- ============================================================
    { name = "sand",            label = "Sand",             source = "crafted", price = 0.20, desc = "Clean silica sand. Processed from quarry rock for glassmaking." },
    { name = "glass",           label = "Sheet Glass",      source = "crafted", price = 1.00, desc = "Flat glass smelted from silica sand. Used in lanterns and containers." },
    { name = "glassbottle",     label = "Glass Bottle",     source = "crafted", price = 0.60, desc = "Blown glass bottle for liquids and tonics." },
    { name = "cloth",           label = "Woven Cloth",      source = "crafted", price = 1.50, desc = "Plain weave cloth processed from plant fibers." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "gun_barrel",      label = "Gun Barrel",       source = "crafted", price = 6.00,  desc = "Forged steel gun barrel. Essential component in firearm crafting." },
    { name = "gunoil",          label = "Gun Oil",          source = "crafted", price = 0.40,  desc = "Refined lubricant for cleaning and maintaining firearms." },
    { name = "rope",            label = "Rope",             source = "crafted", price = 1.00,  desc = "Braided hemp rope. Used in lassos and general equipment." },
    { name = "alcohol",         label = "Alcohol",          source = "crafted", price = 0.80,  desc = "Distilled grain alcohol. Solvent, fuel, and cocktail base." },
    { name = "flour",           label = "Wheat Flour",      source = "crafted", price = 1.75, desc = "Milled wheat flour, ready for baking." },
    -- Not a stock item: must exist in your items table, or the entry below is skipped.
    { name = "corn_flour",      label = "Corn Flour",       source = "crafted", price = 1.50, desc = "Ground corn meal for tortillas and cornbread." },
}

-- ============================================================================
-- ITEM REQUESTS (Player Want Listings)
-- ============================================================================
Config.Requests = {
    SalesTaxPercent      = 8,        -- % taken from fulfiller's gross pay
    MaxActiveRequests    = 10,       -- max open requests per player
    MinPricePerUnit      = 0.01,
    MaxPricePerUnit      = 50000.00,
    MaxQuantity          = 500,
    ExpirationCheckInterval = 120,   -- seconds between expiration sweeps
    Durations = {
        { label = "Short (4h)",     hours = 4  },
        { label = "Medium (12h)",   hours = 12 },
        { label = "Long (24h)",     hours = 24 },
        { label = "Extended (72h)", hours = 72 },
        { label = "Persistent",     hours = 0  }, -- hours = 0 → no expiry
    },
}
