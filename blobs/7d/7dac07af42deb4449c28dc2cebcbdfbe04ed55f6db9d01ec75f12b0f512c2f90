-- ============================================================================
--  poggy_util · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_util starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_util` in the server console.
-- ============================================================================

-- ----------------------------------------------------------------------------
--  1. Armor HUD position
--  ---------------------------------------------------------------------------
--  Where each character dragged the armor icon with /movearmor, as "top,left"
--  in CSS pixels.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_util_armor_hud` (
    `charid`   VARCHAR(64) NOT NULL,
    `position` VARCHAR(32) NOT NULL,
    PRIMARY KEY (`charid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  2. Government stipend tenure (Config.Stipend; off by default)
--  ---------------------------------------------------------------------------
--  The stipend reads `characters`.`created_at` to work out how many days a
--  character has been on the server. VORP's `characters` table has no such
--  column, so it is added here, in two steps that together give existing
--  characters a legacy date and new characters their real creation time:
--
--  Step 1 (ADD COLUMN) is sent only when the column is missing, so it runs
--  once. Every row that already exists gets the legacy date, which makes those
--  characters count as old (Config.Stipend.LegacyDate documents this date;
--  change it HERE, before poggy_util first starts, if your server launched on
--  another day). The column is deliberately declared TIMESTAMP so that step 2
--  has a type difference to react to.
--
--  Step 2 (MODIFY COLUMN) is sent only when the column's type differs from the
--  one written here. Right after step 1 the type is TIMESTAMP, so step 2 runs
--  once, turns the column into DATETIME (the values are kept) and sets the
--  default for new rows to the time of creation. From then on the types match
--  and neither step is sent again. A server whose column was already DATETIME
--  (added by an earlier poggy_util) is left alone by both steps.
--
--  `characters` is VORP's table. Each step is marked `-- poggy: only-if-table
--  characters`, so on a framework without it (RSG) poggy_core skips the step
--  instead of failing. The stipend is VORP-only anyway (it reads this table
--  directly); server/stipend.lua disables itself on other frameworks.
-- ----------------------------------------------------------------------------
-- poggy: only-if-table characters
ALTER TABLE `characters` ADD COLUMN `created_at` TIMESTAMP NULL DEFAULT '2025-11-01 00:00:00';

-- poggy: only-if-table characters
ALTER TABLE `characters` MODIFY COLUMN `created_at` DATETIME NULL DEFAULT CURRENT_TIMESTAMP;

-- ----------------------------------------------------------------------------
--  3. Armor repair kit item
--  ---------------------------------------------------------------------------
--  Config.ArmorProtection.RepairKitItem. The icon is docs/armor_kit.png; copy
--  it into your inventory's item image folder.
--
--  `items` is VORP's table. On a framework that keeps its items elsewhere
--  (RSG: rsg-core/shared/items.lua) poggy_core skips this statement and
--  poggy_util prints the item name to add by hand.
-- ----------------------------------------------------------------------------
-- poggy: only-if-table items
INSERT IGNORE INTO `items` (`item`, `label`, `limit`, `can_remove`, `type`, `usable`, `desc`, `weight`)
VALUES ('armor_kit', 'Armor Repair Kit', 5, 1, 'item_standard', 1, 'A toolkit used to repair damaged body armor.', 1.00);
