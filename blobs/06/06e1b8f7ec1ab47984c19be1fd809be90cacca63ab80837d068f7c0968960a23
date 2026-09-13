-- ============================================================================
--  poggy_fishing · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_fishing starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_fishing` in the server console.
-- ============================================================================

-- Registers the fish, bait, rod and loot items in VORP's `items` table.
-- Item names match config.lua (Config.Fish, Config.Baits, Config.LootDrops,
-- Config.FishingRodItem, Config.ProRodItem).
--
-- Only missing items are added. Rows already present are never changed, so
-- your own edits to labels, limits and the rest are kept on every start.
--
-- `items` belongs to VORP. On a framework that keeps its items elsewhere (RSG:
-- rsg-core/shared/items.lua) there is no such table, so each block below is
-- marked `-- poggy: only-if-table items` and poggy_core skips it; the script
-- then prints the item names you need to add to that file by hand.

-- FISH, BAIT AND RODS
-- poggy: only-if-table items
INSERT IGNORE INTO `items` (`item`, `label`, `limit`, `can_remove`, `type`, `usable`) VALUES
    -- FISH
    ('a_c_fishbluegil_01_sm', 'Bluegill (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishperch_01_sm', 'Perch (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishrockbass_01_sm', 'Rock Bass (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishredfinpickerel_01_sm', 'Redfin Pickerel (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishbullheadcat_01_sm', 'Bullhead Catfish (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishchainpickerel_01_sm', 'Chain Pickerel (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishsalmonsockeye_01_sm', 'Sockeye Salmon (Small)', 10, 1, 'item_standard', 0),
    ('a_c_fishbluegil_01_ms', 'Bluegill (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishperch_01_ms', 'Perch (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishrockbass_01_ms', 'Rock Bass (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishbullheadcat_01_ms', 'Bullhead Catfish (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishredfinpickerel_01_ms', 'Redfin Pickerel (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishchainpickerel_01_ms', 'Chain Pickerel (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishlargemouthbass_01_ms', 'Largemouth Bass (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishsmallmouthbass_01_ms', 'Smallmouth Bass (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishsalmonsockeye_01_ms', 'Sockeye Salmon (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishrainbowtrout_01_ms', 'Rainbow Trout (Medium)', 10, 1, 'item_standard', 0),
    ('a_c_fishlargemouthbass_01_lg', 'Largemouth Bass (Large)', 10, 1, 'item_standard', 0),
    ('a_c_fishsmallmouthbass_01_lg', 'Smallmouth Bass (Large)', 10, 1, 'item_standard', 0),
    ('a_c_fishsalmonsockeye_01_lg', 'Sockeye Salmon (Large)', 10, 1, 'item_standard', 0),
    ('a_c_fishsalmonsockeye_01_ml', 'Sockeye Salmon (Medium-Large)', 10, 1, 'item_standard', 0),
    ('a_c_fishsteelheadtrout', 'Steelhead Trout (Large)', 10, 1, 'item_standard', 0),
    ('a_c_fishnorthernpike_01_lg', 'Northern Pike (Large)', 10, 1, 'item_standard', 0),
    ('a_c_fishlongnosegar_01_lg', 'Longnose Gar (Large)', 10, 1, 'item_standard', 0),
    -- LEGENDARY FISH
    ('a_c_fishrainbowtrout_01_lg', 'Legendary Rainbow Trout', 10, 1, 'item_standard', 0),
    ('a_c_fishmuskie_01_lg', 'Legendary Muskellunge', 10, 1, 'item_standard', 0),
    ('a_c_fishlakesturgeon_01_lg', 'Legendary Lake Sturgeon', 10, 1, 'item_standard', 0),
    ('a_c_fishchannelcatfish_01_xl', 'Legendary Channel Catfish', 10, 1, 'item_standard', 0),
    -- LURES AND BAIT (names match Config.Baits[...].item)
    ('p_lgoc_spinner_v4', 'Spinner Lure (V4)', 10, 1, 'item_standard', 1),
    ('bait_worm', 'Worm Bait', 10, 1, 'item_standard', 1),
    ('bait_cricket', 'Cricket Bait', 10, 1, 'item_standard', 1),
    ('bait_bread', 'Bread Bait', 10, 1, 'item_standard', 1),
    ('bait_crawdad', 'Crawdad Bait', 10, 1, 'item_standard', 1),
    ('p_finishedragonflylegendary01x', 'Legendary Dragon Fly Lure', 10, 1, 'item_standard', 1),
    -- FISHING RODS
    ('fishingrod', 'Fishing Rod', 10, 1, 'item_standard', 1),
    ('fishingrod_pro', 'Pro Fishing Rod', 10, 1, 'item_standard', 1);

-- LOOT DROPS (Config.LootDrops)
-- These are generic valuables many servers already define, so existing rows
-- are left alone and only missing ones are added.
-- poggy: only-if-table items
INSERT IGNORE INTO `items` (`item`, `label`, `limit`, `can_remove`, `type`, `usable`) VALUES
    ('whitepearl', 'White Pearl', 10, 1, 'item_standard', 0),
    ('redpearl', 'Red Pearl', 10, 1, 'item_standard', 0),
    ('bluepearl', 'Blue Pearl', 10, 1, 'item_standard', 0),
    ('goldpearl', 'Gold Pearl', 10, 1, 'item_standard', 0),
    ('blackpearl', 'Black Pearl', 10, 1, 'item_standard', 0),
    ('golden_nugget', 'Golden Nugget', 10, 1, 'item_standard', 0),
    ('diamond_uncut', 'Uncut Diamond', 10, 1, 'item_standard', 0);
