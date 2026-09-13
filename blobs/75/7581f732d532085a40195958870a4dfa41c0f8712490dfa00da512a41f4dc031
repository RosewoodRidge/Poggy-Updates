-- ============================================================================
--  poggy_fishing_journal · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_fishing_journal starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_fishing_journal` in the server console.
-- ============================================================================

-- Journal data: one row per character, JSON in the text columns.
--   discoveries:  { "ZONE_HASH": { "fish_key": true, ... }, ... }
--   catch_counts: { "fish_key": count, ... }
--   weight_data:  { "fish_key": { best, total, bestDate, firstCaught, lastCaught }, ... }
CREATE TABLE IF NOT EXISTS `poggy_fishing_journal` (
    `charid`       VARCHAR(64) NOT NULL PRIMARY KEY,
    `discoveries`  LONGTEXT DEFAULT ('{}'),
    `catch_counts` LONGTEXT DEFAULT ('{}'),
    `weight_data`  LONGTEXT DEFAULT ('{}'),
    `updated_at`   TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Tables made by older versions had no weight tracking.
ALTER TABLE `poggy_fishing_journal`
    ADD COLUMN `weight_data` LONGTEXT DEFAULT ('{}');

-- Character ids are strings on RSG (citizenid); VORP's numeric ids fit too.
-- poggy_core sends this only while the column is still INT.
ALTER TABLE `poggy_fishing_journal`
    MODIFY COLUMN `charid` VARCHAR(64) NOT NULL;

-- Journal item (usable = 1 so the inventory triggers the callback).
-- Only added when missing; an existing row is never changed.
-- `items` is VORP's table. On a framework without it (RSG keeps items in
-- rsg-core/shared/items.lua) poggy_core skips this statement and the script
-- prints the item name to add by hand.
-- poggy: only-if-table items
INSERT IGNORE INTO `items` (`item`, `label`, `limit`, `can_remove`, `type`, `usable`)
VALUES ('fishing_journal_fn', 'Fishing Journal', 1, 1, 'item_standard', 1);
