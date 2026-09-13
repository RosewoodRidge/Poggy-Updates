--[[===========================================================================
    poggy_markets · config/catalog_default.lua
    ---------------------------------------------------------------------------
    What each type of store trades.

        Config.Catalog.<ListName> = {
            sell = { ... }   -- the store sells these TO players
            buy  = { ... }   -- the store buys these FROM players
        }

    A store type points at a list name (see config/stores.lua), so editing a
    list here changes every store of that type at once.

    Entry format:
        { name = "item_name", label = "Display Name", price = 1.50 }

    Optional per-entry fields:
        type   = "item_weapon"   for weapons (default "item_standard")
        stock  = 50              starting stock (default: unlimited for NPC stores)
        hidden = true            stocked but not shown to customers

    IMPORTANT
    ---------
    `name` must match an item in your server's items table.  This default
    catalog uses common RedM item names, but every server names things
    differently -- check these against your own database before going live.
    Any item that does not exist is skipped with a console warning at startup
    rather than breaking the store.
===========================================================================]]--

Config = Config or {}
Config.Catalog = {}

-- ===========================================================================
--  GENERAL STORE
-- ===========================================================================
Config.Catalog.GeneralStore = {
    sell = {
        { name = "consumable_bread",       label = "Bread",              price = 0.75 },
        { name = "consumable_cornmeal",    label = "Cornmeal",           price = 0.60 },
        { name = "consumable_crackers",    label = "Crackers",           price = 0.50 },
        { name = "consumable_canned_beans",label = "Canned Beans",       price = 1.20 },
        { name = "consumable_coffee",      label = "Coffee",             price = 0.90 },
        { name = "consumable_sugar",       label = "Sugar",              price = 0.55 },
        { name = "consumable_salt",        label = "Salt",               price = 0.40 },
        { name = "water",                  label = "Canteen of Water",   price = 0.50 },
        { name = "matches",                label = "Box of Matches",     price = 0.35 },
        { name = "lantern",                label = "Lantern",            price = 4.00 },
        { name = "rope",                   label = "Rope",               price = 2.50 },
        { name = "bandage",                label = "Bandage",            price = 1.50 },
        { name = "soap",                   label = "Bar of Soap",        price = 0.45 },
        { name = "candle",                 label = "Candle",             price = 0.30 },
        { name = "papers",                 label = "Rolling Papers",     price = 0.25 },
        { name = "tobacco",                label = "Tobacco",            price = 1.00 },
    },
    buy = {
        { name = "consumable_bread",       label = "Bread",              price = 0.25 },
        { name = "consumable_cornmeal",    label = "Cornmeal",           price = 0.20 },
        { name = "tobacco",                label = "Tobacco",            price = 0.35 },
        { name = "rope",                   label = "Rope",               price = 0.80 },
    },
}

-- ===========================================================================
--  GUNSMITH
-- ===========================================================================
Config.Catalog.Gunsmith = {
    sell = {
        -- Weapons
        { name = "WEAPON_MELEE_KNIFE",              label = "Knife",                  price =   8.00, type = "item_weapon" },
        { name = "WEAPON_MELEE_HATCHET",            label = "Hatchet",                price =  12.00, type = "item_weapon" },
        { name = "WEAPON_LASSO",                    label = "Lasso",                  price =   8.00, type = "item_weapon" },
        { name = "WEAPON_KIT_BINOCULARS",           label = "Binoculars",             price =  12.00, type = "item_weapon" },
        { name = "WEAPON_BOW",                      label = "Bow",                    price =  16.00, type = "item_weapon" },
        { name = "WEAPON_REVOLVER_DOUBLEACTION",    label = "Double-Action Revolver", price =  22.00, type = "item_weapon" },
        { name = "WEAPON_REVOLVER_CATTLEMAN",       label = "Cattleman Revolver",     price =  46.00, type = "item_weapon" },
        { name = "WEAPON_PISTOL_VOLCANIC",          label = "Volcanic Pistol",        price =  90.00, type = "item_weapon" },
        { name = "WEAPON_REPEATER_WINCHESTER",      label = "Winchester Repeater",    price =  92.00, type = "item_weapon" },
        { name = "WEAPON_RIFLE_VARMINT",            label = "Varmint Rifle",          price =  72.00, type = "item_weapon" },
        { name = "WEAPON_RIFLE_SPRINGFIELD",        label = "Springfield Rifle",      price = 156.00, type = "item_weapon" },
        { name = "WEAPON_SHOTGUN_DOUBLEBARREL",     label = "Double-Barrel Shotgun",  price = 106.00, type = "item_weapon" },

        -- Ammunition
        { name = "ammorevolvernormal",   label = "Revolver Rounds",        price = 0.25 },
        { name = "ammorevolverexpress",  label = "Revolver Rounds (Express)", price = 0.50 },
        { name = "ammopistolnormal",     label = "Pistol Rounds",          price = 0.25 },
        { name = "ammorepeaternormal",   label = "Repeater Rounds",        price = 0.25 },
        { name = "ammoriflenormal",      label = "Rifle Rounds",           price = 0.25 },
        { name = "ammoshotgunnormal",    label = "Shotgun Shells",         price = 0.25 },
        { name = "ammoarrownormal",      label = "Arrows",                 price = 0.10 },
        { name = "ammovarmint",          label = "Varmint Rounds",         price = 0.15 },

        -- Upkeep
        { name = "gunoil",               label = "Gun Oil",                price = 0.40 },
    },
    buy = {
        { name = "gunoil",               label = "Gun Oil",                price = 0.10 },
        { name = "ammorevolvernormal",   label = "Revolver Rounds",        price = 0.08 },
        { name = "ammoriflenormal",      label = "Rifle Rounds",           price = 0.08 },
    },
}

-- ===========================================================================
--  BLACKSMITH
-- ===========================================================================
Config.Catalog.Blacksmith = {
    sell = {
        { name = "iron_ingot",   label = "Iron Ingot",     price = 4.00 },
        { name = "steel_ingot",  label = "Steel Ingot",    price = 7.00 },
        { name = "copper_ingot", label = "Copper Ingot",   price = 5.00 },
        { name = "nails",        label = "Nails",          price = 0.40 },
        { name = "horseshoe",    label = "Horseshoe",      price = 2.00 },
        { name = "hammer",       label = "Hammer",         price = 3.50 },
        { name = "shovel",       label = "Shovel",         price = 4.50 },
        { name = "pickaxe",      label = "Pickaxe",        price = 5.50 },
    },
    buy = {
        { name = "iron_ore",     label = "Iron Ore",       price = 1.20 },
        { name = "copper_ore",   label = "Copper Ore",     price = 1.40 },
        { name = "coal",         label = "Coal",           price = 0.80 },
        { name = "silver_ore",   label = "Silver Ore",     price = 3.00 },
        { name = "gold_ore",     label = "Gold Ore",       price = 6.00 },
        { name = "scrap_metal",  label = "Scrap Metal",    price = 0.50 },
    },
}

-- ===========================================================================
--  HORSE SUPPLY
-- ===========================================================================
Config.Catalog.HorseSupply = {
    sell = {
        { name = "hay",            label = "Bale of Hay",       price = 1.00 },
        { name = "oats",           label = "Oats",              price = 0.80 },
        { name = "carrot",         label = "Carrot",            price = 0.30 },
        { name = "apple",          label = "Apple",             price = 0.30 },
        { name = "sugarcube",      label = "Sugar Cube",        price = 0.20 },
        { name = "horsebrush",     label = "Horse Brush",       price = 2.50 },
        { name = "horse_medicine", label = "Horse Medicine",    price = 3.00 },
        { name = "horse_stimulant",label = "Horse Stimulant",   price = 2.00 },
        { name = "saddlebag",      label = "Saddlebag",         price = 12.00 },
    },
    buy = {
        { name = "hay",            label = "Bale of Hay",       price = 0.30 },
        { name = "oats",           label = "Oats",              price = 0.25 },
        { name = "carrot",         label = "Carrot",            price = 0.08 },
        { name = "apple",          label = "Apple",             price = 0.08 },
    },
}

-- ===========================================================================
--  SALOON
-- ===========================================================================
Config.Catalog.Saloon = {
    sell = {
        { name = "beer",       label = "Beer",           price = 1.20 },
        { name = "whisky",     label = "Whisky",         price = 3.20 },
        { name = "vodka",      label = "Vodka",          price = 1.60 },
        { name = "tequila",    label = "Tequila",        price = 2.40 },
        { name = "absinthe",   label = "Absinthe",       price = 2.80 },
        { name = "moonshine",  label = "Moonshine",      price = 2.40 },
        { name = "cigar",      label = "Cigar",          price = 0.25 },
        { name = "cigarette",  label = "Cigarette",      price = 0.15 },
        { name = "stew",       label = "Bowl of Stew",   price = 1.50 },
    },
    buy = {
        { name = "moonshine",  label = "Moonshine",      price = 0.75 },
        { name = "whisky",     label = "Whisky",         price = 1.00 },
        { name = "beer",       label = "Beer",           price = 0.40 },
    },
}

-- ===========================================================================
--  BUTCHER  (buys game from players; sells nothing)
-- ===========================================================================
Config.Catalog.Butcher = {
    sell = {},
    buy = {
        -- Pelts
        { name = "deer_pelt",     label = "Deer Pelt",       price = 3.00 },
        { name = "buck_pelt",     label = "Buck Pelt",       price = 3.50 },
        { name = "elk_pelt",      label = "Elk Pelt",        price = 4.50 },
        { name = "bear_pelt",     label = "Bear Pelt",       price = 9.00 },
        { name = "wolf_pelt",     label = "Wolf Pelt",       price = 5.00 },
        { name = "cougar_pelt",   label = "Cougar Pelt",     price = 7.00 },
        { name = "bison_pelt",    label = "Bison Pelt",      price = 8.00 },
        { name = "boar_pelt",     label = "Boar Pelt",       price = 2.50 },
        { name = "fox_pelt",      label = "Fox Pelt",        price = 2.00 },
        { name = "beaver_pelt",   label = "Beaver Pelt",     price = 2.20 },
        { name = "rabbit_pelt",   label = "Rabbit Pelt",     price = 0.60 },
        -- Meat
        { name = "big_game_meat", label = "Big Game Meat",   price = 1.20 },
        { name = "venison",       label = "Venison",         price = 0.90 },
        { name = "mutton",        label = "Mutton",          price = 0.80 },
        { name = "poultry",       label = "Poultry",         price = 0.60 },
    },
}

-- ===========================================================================
--  DOCTOR
-- ===========================================================================
Config.Catalog.Doctor = {
    sell = {
        { name = "bandage",       label = "Bandage",           price = 1.50 },
        { name = "medicine",      label = "Medicine",          price = 3.00 },
        { name = "tonic",         label = "Health Tonic",      price = 2.50 },
        { name = "snakeoil",      label = "Snake Oil",         price = 1.75 },
        { name = "morphine",      label = "Morphine",          price = 6.00 },
        { name = "splint",        label = "Splint",            price = 4.00 },
        { name = "suture_kit",    label = "Suture Kit",        price = 5.00 },
    },
    buy = {},
}

-- ===========================================================================
--  FISH MARKET
-- ===========================================================================
Config.Catalog.Fish = {
    sell = {
        { name = "fishing_rod",   label = "Fishing Rod",       price = 8.00 },
        { name = "bait_worm",     label = "Worms",             price = 0.25 },
        { name = "bait_bread",    label = "Bread Bait",        price = 0.20 },
        { name = "bait_cheese",   label = "Cheese Bait",       price = 0.30 },
        { name = "lure_river",    label = "River Lure",        price = 1.50 },
    },
    buy = {
        { name = "bluegill",      label = "Bluegill",          price = 0.60 },
        { name = "bullhead",      label = "Bullhead Catfish",  price = 0.80 },
        { name = "chain_pickerel",label = "Chain Pickerel",    price = 1.00 },
        { name = "largemouth_bass", label = "Largemouth Bass", price = 1.40 },
        { name = "smallmouth_bass", label = "Smallmouth Bass", price = 1.20 },
        { name = "rock_bass",     label = "Rock Bass",         price = 0.90 },
        { name = "perch",         label = "Perch",             price = 0.70 },
        { name = "salmon",        label = "Salmon",            price = 2.50 },
        { name = "steelhead_trout", label = "Steelhead Trout", price = 2.20 },
        { name = "muskie",        label = "Muskie",            price = 3.00 },
        { name = "sturgeon",      label = "Sturgeon",          price = 4.00 },
    },
}

-- ===========================================================================
--  LUMBER YARD
-- ===========================================================================
Config.Catalog.Lumber = {
    sell = {
        { name = "plank",         label = "Wooden Plank",      price = 1.20 },
        { name = "beam",          label = "Support Beam",      price = 2.50 },
        { name = "firewood",      label = "Firewood",          price = 0.40 },
        { name = "axe",           label = "Felling Axe",       price = 5.00 },
        { name = "saw",           label = "Hand Saw",          price = 3.50 },
    },
    buy = {
        { name = "log",           label = "Log",               price = 0.90 },
        { name = "oak_log",       label = "Oak Log",           price = 1.30 },
        { name = "pine_log",      label = "Pine Log",          price = 1.00 },
        { name = "firewood",      label = "Firewood",          price = 0.12 },
    },
}

-- ===========================================================================
--  CAMP OUTFITTER
-- ===========================================================================
Config.Catalog.Camping = {
    sell = {
        { name = "campfire_kit",  label = "Campfire Kit",      price = 3.00 },
        { name = "bedroll",       label = "Bedroll",           price = 6.00 },
        { name = "tent",          label = "Tent",              price = 15.00 },
        { name = "cooking_pot",   label = "Cooking Pot",       price = 4.00 },
        { name = "lantern",       label = "Lantern",           price = 4.00 },
        { name = "matches",       label = "Box of Matches",    price = 0.35 },
        { name = "rope",          label = "Rope",              price = 2.50 },
        { name = "canteen",       label = "Canteen",           price = 2.00 },
        { name = "trap",          label = "Small Game Trap",   price = 5.00 },
    },
    buy = {},
}
