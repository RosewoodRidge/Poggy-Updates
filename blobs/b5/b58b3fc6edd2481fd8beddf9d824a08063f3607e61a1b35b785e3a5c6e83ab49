-- ============================================================================
--  poggy_crafting · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_crafting starts: missing tables,
--  columns and indexes are added and anything already there is left alone.
--  There is nothing to import.
--
--  To manage it yourself instead, set PoggyCoreConfig.Sql.AutoInstall = false
--  in poggy_core/config.lua and import this file, or run
--  `poggycore sql install poggy_crafting` in the server console.
-- ============================================================================

-- ----------------------------------------------------------------------------
--  1. Shopping list
--  ---------------------------------------------------------------------------
--  One row per ingredient a character has added from a recipe. The gathering
--  tracker reads the same rows, so a list survives a relog and a server
--  restart.
--
--  charidentifier is VARCHAR so the same schema works on RSG and QBCore, whose
--  character ids are strings. On VORP it holds a number and behaves as before.
--
--  Config.ShoppingList.enabled = false leaves this table unused, not dropped.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_crafting_list` (
    `id`             INT(11)      NOT NULL AUTO_INCREMENT,
    `charidentifier` VARCHAR(64)  NOT NULL DEFAULT '0',
    `item_name`      VARCHAR(100) NOT NULL DEFAULT '',
    `item_label`     VARCHAR(255) NOT NULL DEFAULT '',
    `quantity`       INT(11)      NOT NULL DEFAULT 1,
    `checked`        TINYINT(1)   NOT NULL DEFAULT 0,
    `recipe_name`    VARCHAR(255) NOT NULL DEFAULT '',
    `cat_color_num`  TINYINT(4)   NOT NULL DEFAULT 1,
    `created_at`     TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    INDEX `idx_poggy_crafting_list_char` (`charidentifier`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ----------------------------------------------------------------------------
--  2. The campfire item
--  ---------------------------------------------------------------------------
--  Config.CampfireItem names a usable item that places a campfire. This adds it
--  if your database does not have it already; an item you have defined
--  yourself is left exactly as it is.
--
--  If you renamed Config.CampfireItem, change the name here to match, or add
--  your own row -- the script registers whatever the config names.
--
--  The icon for it is in docs/item_images/. Copy it to
--  vorp_inventory/html/img/items/ (or your framework's equivalent) or the
--  browser shows a placeholder.
--
--  `items` is VORP's table. On a framework that keeps its items elsewhere
--  (RSG: rsg-core/shared/items.lua) there is no such table, so this is marked
--  only-if-table and poggy_core skips it -- add the item to that file by hand.
--  Last in the file so the table above is created whatever the framework.
-- ----------------------------------------------------------------------------
-- poggy: only-if-table items
INSERT IGNORE INTO `items` (`item`, `label`, `limit`, `can_remove`, `type`, `usable`)
VALUES ('campfire', 'Campfire', 5, 1, 'item_standard', 1);
