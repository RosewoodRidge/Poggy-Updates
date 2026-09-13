JournalConfig = {}

---------------------------------------------------------------------------
--  ITEM
---------------------------------------------------------------------------
JournalConfig.JournalItem = "fishing_journal_fn"

---------------------------------------------------------------------------
--  ANIMATION  (WORLD_HUMAN_WRITE_NOTEBOOK scenario)
---------------------------------------------------------------------------
JournalConfig.OpenAnim = {
    scenario = "WORLD_HUMAN_WRITE_NOTEBOOK",
    delay    = 2000,   -- ms to wait in scenario before opening UI
}

---------------------------------------------------------------------------
--  MAP IMAGE
---------------------------------------------------------------------------
JournalConfig.Map = {
    imageUrl    = "img/map.jpg",
    imageWidth  = 3988,
    imageHeight = 2637,
    gameBounds  = {
        minX = -9856.17,
        maxX = 5500.0,
        minY = -5844.07,
        maxY = 4312.19,
    },
}

---------------------------------------------------------------------------
--  FISH DEFINITIONS  (mirrors poggy_fishing Config.Fish)
--  Keeps label, size, price, hours — everything the journal UI needs.
--  "item" is only used for display; the fishing script handles inventory.
---------------------------------------------------------------------------
JournalConfig.Fish = {
    -- SMALL / EASY
    ["bluegill_sm"]         = { label = "Bluegill (Small)",              size = "sm", price = 0.40, weightMin = 0.1,  weightMax = 4.12,  species = "bluegill",         item = "a_c_fishbluegil_01_sm" },
    ["perch_sm"]            = { label = "Perch (Small)",                 size = "sm", price = 0.35, weightMin = 0.1,  weightMax = 2.1,   species = "perch",            item = "a_c_fishperch_01_sm" },
    ["rock_bass_sm"]        = { label = "Rock Bass (Small)",             size = "sm", price = 0.45, weightMin = 0.1,  weightMax = 1.5,   species = "rock_bass",        item = "a_c_fishrockbass_01_sm" },
    ["redfin_pickerel_sm"]  = { label = "Redfin Pickerel (Small)",       size = "sm", price = 0.50, weightMin = 0.1,  weightMax = 1.0,   species = "redfin_pickerel",  item = "a_c_fishredfinpickerel_01_sm" },
    ["bullhead_sm"]         = { label = "Bullhead Catfish (Small)",      size = "sm", price = 0.55, weightMin = 0.1,  weightMax = 7.6,   species = "bullhead",         item = "a_c_fishbullheadcat_01_sm", hours = {20, 6} },
    ["chain_pickerel_sm"]   = { label = "Chain Pickerel (Small)",        size = "sm", price = 0.65, weightMin = 0.1,  weightMax = 4.0,   species = "chain_pickerel",   item = "a_c_fishchainpickerel_01_sm", hours = {5, 12} },
    ["sockeye_salmon_sm"]   = { label = "Sockeye Salmon (Small)",        size = "sm", price = 0.60, weightMin = 0.1,  weightMax = 3.0,   species = "sockeye_salmon",   item = "a_c_fishsalmonsockeye_01_sm", hours = {4, 10} },

    -- MEDIUM
    ["bluegill_md"]         = { label = "Bluegill (Medium)",             size = "md", price = 1.25, weightMin = 0.1,  weightMax = 4.12,  species = "bluegill",         item = "a_c_fishbluegil_01_ms" },
    ["perch_md"]            = { label = "Perch (Medium)",                size = "md", price = 1.00, weightMin = 2.2,  weightMax = 4.3,   species = "perch",            item = "a_c_fishperch_01_ms" },
    ["rock_bass_md"]        = { label = "Rock Bass (Medium)",            size = "md", price = 1.50, weightMin = 1.6,  weightMax = 3.0,   species = "rock_bass",        item = "a_c_fishrockbass_01_ms" },
    ["bullhead_md"]         = { label = "Bullhead Catfish (Medium)",     size = "md", price = 1.40, weightMin = 7.7,  weightMax = 11.0,  species = "bullhead",         item = "a_c_fishbullheadcat_01_ms", hours = {20, 6} },
    ["redfin_pickerel_md"]  = { label = "Redfin Pickerel (Medium)",      size = "md", price = 1.60, weightMin = 1.1,  weightMax = 2.1,   species = "redfin_pickerel",  item = "a_c_fishredfinpickerel_01_ms", hours = {15, 22} },
    ["chain_pickerel_md"]   = { label = "Chain Pickerel (Medium)",       size = "md", price = 1.80, weightMin = 4.1,  weightMax = 9.6,   species = "chain_pickerel",   item = "a_c_fishchainpickerel_01_ms", hours = {5, 12} },
    ["largemouth_md"]       = { label = "Largemouth Bass (Medium)",      size = "md", price = 2.00, weightMin = 2.1,  weightMax = 9.0,   species = "largemouth_bass",  item = "a_c_fishlargemouthbass_01_ms", hours = {4, 11} },
    ["smallmouth_md"]       = { label = "Smallmouth Bass (Medium)",      size = "md", price = 1.75, weightMin = 2.0,  weightMax = 6.0,   species = "smallmouth_bass",  item = "a_c_fishsmallmouthbass_01_ms", hours = {6, 14} },
    ["sockeye_salmon_md"]   = { label = "Sockeye Salmon (Medium)",       size = "md", price = 2.50, weightMin = 3.0,  weightMax = 6.0,   species = "sockeye_salmon",   item = "a_c_fishsalmonsockeye_01_ms", hours = {4, 10} },
    ["rainbow_trout_md"]    = { label = "Rainbow Trout (Medium)",        size = "md", price = 2.25, weightMin = 5.0,  weightMax = 24.0,  species = "rainbow_trout",    item = "a_c_fishrainbowtrout_01_ms", hours = {5, 21} },

    -- LARGE / HARD
    ["largemouth_lg"]       = { label = "Largemouth Bass (Large)",       size = "lg", price = 3.50, weightMin = 9.1,  weightMax = 22.0,  species = "largemouth_bass",  item = "a_c_fishlargemouthbass_01_lg", hours = {4, 10} },
    ["smallmouth_lg"]       = { label = "Smallmouth Bass (Large)",       size = "lg", price = 3.25, weightMin = 6.1,  weightMax = 11.15, species = "smallmouth_bass",  item = "a_c_fishsmallmouthbass_01_lg", hours = {5, 11} },
    ["sockeye_salmon_lg"]   = { label = "Sockeye Salmon (Large)",        size = "lg", price = 4.50, weightMin = 10.1, weightMax = 15.3,  species = "sockeye_salmon",   item = "a_c_fishsalmonsockeye_01_lg", hours = {4, 10} },
    ["sockeye_salmon_ml"]   = { label = "Sockeye Salmon (Med-Large)",    size = "lg", price = 3.75, weightMin = 6.1,  weightMax = 10.0,  species = "sockeye_salmon",   item = "a_c_fishsalmonsockeye_01_ml", hours = {4, 10} },
    ["steelhead_trout_lg"]  = { label = "Steelhead Trout (Large)",        size = "lg", price = 4.00, weightMin = 8.0,  weightMax = 20.0,  species = "steelhead_trout",  item = "a_c_fishsteelheadtrout", hours = {5, 11} },

    -- EXTRA LARGE / EXPERT
    ["northern_pike"]       = { label = "Northern Pike",                 size = "xl", price = 8.00,  weightMin = 15.0, weightMax = 55.1,  species = "northern_pike",    item = "a_c_fishnorthernpike_01_lg", hours = {4, 11} },
    ["longnose_gar"]        = { label = "Longnose Gar",                  size = "xl", price = 9.00,  weightMin = 10.0, weightMax = 56.2,  species = "longnose_gar",     item = "a_c_fishlongnosegar_01_lg", hours = {20, 6} },

    -- LEGENDARY
    ["legendary_rainbow_trout"]    = { label = "Legendary Rainbow Trout",     size = "legendary", price = 35.00, weightMin = 50.0,  weightMax = 96.0,   species = "rainbow_trout",    item = "a_c_fishrainbowtrout_01_lg", hours = {5, 9} },
    ["legendary_muskie"]           = { label = "Legendary Muskellunge",       size = "legendary", price = 35.00, weightMin = 70.0,  weightMax = 120.0,  species = "muskellunge",      item = "a_c_fishmuskie_01_lg", hours = {4, 7} },
    ["legendary_sturgeon"]         = { label = "Legendary Lake Sturgeon",     size = "legendary", price = 35.00, weightMin = 80.0,  weightMax = 150.0,  species = "lake_sturgeon",    item = "a_c_fishlakesturgeon_01_lg", hours = {20, 23} },
    ["legendary_channel_catfish"]  = { label = "Legendary Channel Catfish",   size = "legendary", price = 35.00, weightMin = 60.0,  weightMax = 110.0,  species = "channel_catfish",  item = "a_c_fishchannelcatfish_01_xl", hours = {22, 2} },
}

---------------------------------------------------------------------------
--  SIZE LABELS (for sidebar grouping)
---------------------------------------------------------------------------
JournalConfig.SizeLabels = {
    sm = "Small",
    md = "Medium",
    lg = "Large",
    xl = "Extra Large",
    legendary = "Legendary",
}

---------------------------------------------------------------------------
--  WATER TYPE COLOURS  (map pin dot colours in the UI)
---------------------------------------------------------------------------
JournalConfig.TypeColours = {
    lake   = "#e28080",
    river  = "#f7de9a",
    swamp  = "#7D8B4E",
    creek  = "#82f1b0",
    pond   = "#5767c5",
    ocean  = "#da52c3",
}
