-- ══════════════════════════════════════════════════════════════════
--  ____    ___    ____   ____  __   __
-- |  _ \  / _ \  / ___| / ___| \ \ / /
-- | |_) || | | || |  _ | |  _   \ V /
-- |  __/ | |_| || |_| || |_| |   | |
-- |_|     \___/  \____| \____|   |_|
--  _____  ____      _    ____  _   _   ____  ___ _   _ ____
-- |_   _||  _ \    / \  / ___|| | | | | __ )|_ _| \ | / ___|
--   | |  | |_) |  / _ \ \___ \| |_| | |  _ \ | ||  \| \___ \
--   | |  |  _ <  / ___ \ ___) |  _  | | |_) || || |\  |___) |
--   |_|  |_| \_\/_/   \_\____/|_| |_| |____/|___|_| \_|____/
-- ══════════════════════════════════════════════════════════════════
-- Dependencies: poggy_core

Config = {}

-- Developer mode (do not enable on production servers)
Config.Developer = false

-- Discord logging
Config.EnableLogs = false
Config.LogsWebhook = "YOUR DISCORD WEBHOOK HERE"

-- Interaction settings
Config.BlockTrashIfPlayerIsNearRange = 3.0 -- Minimum distance from other players to interact (anti-exploit)
Config.InteractionRadius = 1.5             -- Prompt activation radius around each bin
Config.SearchKey = 0x41AC83D1              -- Key hash: search a trash bin
Config.StorageKey = 0xFF8109D8             -- Key hash: open bin storage

-- Auto-detection of existing world trash bin props
-- The client scans the object pool for any of these models and registers
-- them as interactable bins alongside the manually configured ones below.
Config.AutoDetect = true  -- Set false to disable entirely
Config.AutoDetectModels = {           -- Model names to scan for; extend as needed
    "p_streettrashcannbx01x",
    "p_trashbin01x",
    "p_trashbin01bx",
}
Config.AutoDetectRescanInterval = 15    -- Seconds between rescans (GetGamePool streams new objects as player moves)
Config.AutoDetectDefaults = {           -- Applied to every auto-detected bin
    storageitemlimit = 20,
    storagemaxweight = 4000000,
}

-- Storage wipe settings
Config.WipeStorageOnScriptStart = true -- Wipe all bin inventories on server/script restart
Config.AllowWeaponsInStorages = false  -- Allow weapons in bin storage (not recommended)

-- Staff wipe command
Config.AllowStaffGroupsToWipeWithCommand = true -- Allow listed groups to wipe all bin inventories via command
Config.WipeCommand = 'emptytrash'
Config.StaffGroups = {
	'superadmin',
	'admin',
	'moderator',
}

-- Search cooldown (seconds) — random interval between min and max
Config.RefillTimeMin = 1800 -- 30 minutes
Config.RefillTimeMax = 7200 -- 120 minutes

-- Maximum number of distinct items a player can find per trash bin search (0 = nothing found)
Config.MaxItemsPerSearch = 2

-- Map blips
Config.EnableBlips = false
Config.BlipColor = GetHashKey("BLIP_MODIFIER_DISTANCE_FADE_SHORT")
Config.SetBlipSprite = 1109348405

-- ── Shared loot pool ─────────────────────────────────────────────────────────
-- All trash bins draw from this single table so changes only need to be made
-- in one place. Every item listed must exist in your database's items table;
-- the active entries are common VORP items. Weapons and ammunition are
-- intentionally excluded.
Config.CommonLoot = {
    -- ── Garbage / Scrap ──────────────────────────────────────────────────────
    { item = "glassbottle",                     label = "Glass Bottle",         chance = 40, minamount = 1, maxamount = 2 },
    { item = "paper",                           label = "Old Paper",            chance = 30, minamount = 1, maxamount = 2 },
    -- ── Food & Drink ─────────────────────────────────────────────────────────
    { item = "water",                           label = "Water",                chance = 25, minamount = 1, maxamount = 1 },
    { item = "consumable_coffee",               label = "Coffee",               chance = 20, minamount = 1, maxamount = 1 },
    { item = "consumable_raspberrywater",       label = "Berry Water",          chance = 15, minamount = 1, maxamount = 1 },
    { item = "consumable_kidneybeans_can",      label = "Kidney Beans Can",     chance = 15, minamount = 1, maxamount = 1 },
    { item = "consumable_salmon_can",           label = "Canned Salmon",        chance = 10, minamount = 1, maxamount = 1 },
    { item = "consumable_donut",                label = "Donut",                chance = 8,  minamount = 1, maxamount = 1 },
    -- ── Alcohol & Tobacco ────────────────────────────────────────────────────
    { item = "cigarette",                       label = "Cigarette",            chance = 25, minamount = 1, maxamount = 2 },
    { item = "beer",                            label = "Beer",                 chance = 20, minamount = 1, maxamount = 1 },
    { item = "cigar",                           label = "Cigar",                chance = 12, minamount = 1, maxamount = 1 },
    { item = "alcohol",                         label = "Alcohol",              chance = 10, minamount = 1, maxamount = 1 },
    { item = "whisky",                          label = "Whisky",               chance = 5,  minamount = 1, maxamount = 1 },
    -- ── Personal & Household ─────────────────────────────────────────────────
    { item = "cloth",                           label = "Cloth",                chance = 15, minamount = 1, maxamount = 1 },
    { item = "washcloth",                       label = "Washcloth",            chance = 10, minamount = 1, maxamount = 1 },
    { item = "hairpomade",                      label = "Hair Pomade",          chance = 8,  minamount = 1, maxamount = 1 },
    { item = "eggs",                            label = "Eggs",                 chance = 8,  minamount = 1, maxamount = 1 },
    -- ── Medicine ─────────────────────────────────────────────────────────────
    { item = "bandage",                         label = "Bandage",              chance = 8,  minamount = 1, maxamount = 1 },
    { item = "consumable_medicine",             label = "Medicine",             chance = 4,  minamount = 1, maxamount = 1 },
    -- ── Example items: uncomment only if these exist in your items table ──────
    -- { item = "bcc_empty_bottle",                label = "Empty Bottle",         chance = 35, minamount = 1, maxamount = 2 },
    -- { item = "consumable_asian_soda_blueberry", label = "Blueberry Soda",       chance = 8,  minamount = 1, maxamount = 1 },
    -- { item = "consumable_asian_soda_orange",    label = "Orange Soda",          chance = 8,  minamount = 1, maxamount = 1 },
    -- { item = "cannabisleaves",                  label = "Cannabis Leaves",      chance = 5,  minamount = 1, maxamount = 1 },
    -- { item = "marijuana_bud2",                  label = "Marijuana Bud",        chance = 4,  minamount = 1, maxamount = 1 },
    -- { item = "marijuana_joint3_6",              label = "Marijuana Joint",      chance = 4,  minamount = 1, maxamount = 1 },
    -- { item = "honey_blunt",                     label = "Honey Blunt",          chance = 3,  minamount = 1, maxamount = 1 },
    -- { item = "drug_hash",                       label = "Hash",                 chance = 3,  minamount = 1, maxamount = 1 },
    -- { item = "drug_magicmushroom",              label = "Magic Mushrooms",      chance = 2,  minamount = 1, maxamount = 1 },
    -- { item = "drug_cocaine",                    label = "Cocaine",              chance = 2,  minamount = 1, maxamount = 1 },
    -- { item = "drug_heroin",                     label = "Heroin",               chance = 2,  minamount = 1, maxamount = 1 },
    -- { item = "widows_preserve",                 label = "Widow's Preserve",     chance = 1,  minamount = 1, maxamount = 1 },
}

-- Trash bin locations
-- Every bin draws from Config.CommonLoot. To give one bin its own loot, add
-- loot = { { item = "...", label = "...", chance = 10, minamount = 1, maxamount = 1 } }
-- to that bin.
Config.TrashBins = {
    { -- Valentine
        coords = vector3(-300.932373046875, 783.7180786132812, 117.75621795654297),
        shouldSpawn = true,
		storageid = "1",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Valentine Station
        coords = vector3(-179.72445678710938, 616.8157348632812, 113.03475189208984),
        shouldSpawn = true,
		storageid = "2",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Annesburg
        coords = vector3(2930.205810546875, 1267.233642578125, 43.60715103149414),
        shouldSpawn = true,
		storageid = "3",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Flatneck Station
        coords = vector3(-334.8014221191406, -357.54302978515625, 87.00071716308594),
        shouldSpawn = true,
		storageid = "4",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Oil Field
        coords = vector3(459.11810302734375, 671.229248046875, 115.70621490478516),
        shouldSpawn = true,
		storageid = "5",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Strawberry
        coords = vector3(-1770.1522216796875, -383.18438720703125, 156.6904754638672),
        shouldSpawn = true,
		storageid = "6",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Strawberry Sheriff
        coords = vector3(-1805.490478515625, -348.7913818359375, 163.237548828125),
        shouldSpawn = true,
		storageid = "7",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Strawberry Shop
        coords = vector3(-1794.4964599609375, -380.8761291503906, 159.2538604736328),
        shouldSpawn = true,
		storageid = "8",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Wallace Station
        coords = vector3(-1307.9842529296875, 396.01580810546875, 94.3646240234375),
        shouldSpawn = true,
		storageid = "9",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Blackwater
        coords = vector3(-775.6368408203125, -1260.4776611328125, 42.60700988769531),
        shouldSpawn = true,
		storageid = "10",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Blackwater 2
        coords = vector3(-795.203369, -1207.761475, 42.559616),
        shouldSpawn = true,
		storageid = "11",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Blackwater 3
        coords = vector3(-808.2899780273438, -1277.337646484375, 42.66415405273437),
        shouldSpawn = true,
		storageid = "12",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Blackwater 4
        coords = vector3(-791.470703125, -1317.463623046875, 42.63023376464844),
        shouldSpawn = true,
		storageid = "13",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Blackwater 5
        coords = vector3(-795.489013671875, -1202.81103515625, 43.19617080688476),
        shouldSpawn = true,
		storageid = "14",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Tumbleweed Saloon
        coords = vector3(-5516.44775390625, -2917.619140625, -2.72836256027221),
        shouldSpawn = true,
		storageid = "15",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Tumbleweed Doctor
        coords = vector3(-5501.23291015625, -2959.15087890625, -1.67579472064971),
        shouldSpawn = true,
		storageid = "16",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Tumbleweed Church
        coords = vector3(-5434.48876953125, -2927.78076171875, -0.08384954929351),
        shouldSpawn = true,
		storageid = "17",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Armadillo Shop
        coords = vector3(-3682.45751953125, -2621.814453125, -14.42071151733398),
        shouldSpawn = true,
		storageid = "18",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Armadillo Bank
        coords = vector3(-3664.582275390625, -2620.33935546875, -14.63621139526367),
        shouldSpawn = true,
		storageid = "19",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Armadillo Station
        coords = vector3(-3725.232666015625, -2609.00244140625, -13.9411449432373),
        shouldSpawn = true,
		storageid = "20",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis
        coords = vector3(2861.927734375, -1181.26123046875, 45.10415267944336),
        shouldSpawn = true,
		storageid = "21",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 2
        coords = vector3(2852.2314453125, -1251.9310302734375, 45.35592269897461),
        shouldSpawn = true,
		storageid = "22",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 3
        coords = vector3(2822.354736328125, -1314.9700927734375, 45.73060607910156),
        shouldSpawn = true,
		storageid = "23",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 4
        coords = vector3(2808.02490234375, -1397.8818359375, 44.39702606201172),
        shouldSpawn = true,
		storageid = "24",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 5
        coords = vector3(2731.668212890625, -1397.73388671875, 45.20232391357422),
        shouldSpawn = true,
		storageid = "25",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 6
        coords = vector3(2597.98291015625, -1418.8526611328125, 45.37707901000976),
        shouldSpawn = true,
		storageid = "26",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 7
        coords = vector3(2748.36328125, -1306.4720458984375, 46.63570022583008),
        shouldSpawn = true,
		storageid = "27",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 8
        coords = vector3(2670.26318359375, -1386.7506103515625, 45.64775848388672),
        shouldSpawn = true,
		storageid = "28",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 9
        coords = vector3(2577.049560546875, -1418.88427734375, 45.32400894165039),
        shouldSpawn = true,
		storageid = "29",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 10
        coords = vector3(2517.2783203125, -1310.3797607421875, 47.92992401123047),
        shouldSpawn = true,
		storageid = "30",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 11
        coords = vector3(2537.4609375, -1272.166015625, 48.25059127807617),
        shouldSpawn = true,
		storageid = "31",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 12
        coords = vector3(2713.577880859375, -1155.2415771484375, 49.37519073486328),
        shouldSpawn = true,
		storageid = "32",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    { -- Saint Denis 13
        coords = vector3(2780.111083984375, -1179.59375, 47.37085342407226),
        shouldSpawn = true,
		storageid = "33",
		storageitemlimit = 20,
		storagemaxweight = 4000000,
    },
    -- More Add Here
}

-- Search duration (milliseconds)
Config.SearchTimeMin = 5000 -- 5 seconds
Config.SearchTimeMax = 10000 -- 10 seconds

-- Progress bar (ui/progressbar.html, drawn by the script while searching)
Config.UseScriptProgressbar = true -- Set to false to hide the search progress bar

-- Player-facing text lives in translations.lua.
