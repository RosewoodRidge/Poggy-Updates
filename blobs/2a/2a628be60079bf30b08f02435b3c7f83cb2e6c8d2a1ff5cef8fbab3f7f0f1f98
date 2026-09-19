-- poggy: only-if-table crafting_shopping_list
-- ============================================================================
--  poggy_crafting · migration 001
--  ---------------------------------------------------------------------------
--  Carries shopping lists over from the older `crafting_shopping_list` table
--  used before this script became poggy_crafting. It runs once, only on a
--  database that actually has that table, and only after install.sql has
--  created `poggy_crafting_list`.
--
--  The old table is left in place. Drop it yourself once you are satisfied the
--  lists came across:
--      DROP TABLE `crafting_shopping_list`;
-- ============================================================================

INSERT INTO `poggy_crafting_list`
    (`charidentifier`, `item_name`, `item_label`, `quantity`, `checked`, `recipe_name`, `cat_color_num`, `created_at`)
SELECT
    CAST(`charid` AS CHAR), `item_name`, `item_label`, `quantity`, `checked`, `recipe_name`, `cat_color_num`, `created_at`
FROM `crafting_shopping_list`;
