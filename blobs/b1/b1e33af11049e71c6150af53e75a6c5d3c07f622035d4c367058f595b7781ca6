-- ============================================================================
--  poggy_scene · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_scene starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_scene` in the server console.
-- ============================================================================

-- Scene text placed in the world. Loaded into memory when the script starts.
CREATE TABLE IF NOT EXISTS `playerscenes` (
    `id`     INT(11)  NOT NULL AUTO_INCREMENT,
    `charid` INT(11)  DEFAULT NULL,
    `coords` LONGTEXT DEFAULT ('{}'),
    `text`   LONGTEXT DEFAULT (''),
    `placed_at` TIMESTAMP NULL DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- When each scene was placed (1.3.2+). Scenes placed before have none (NULL).
ALTER TABLE `playerscenes` ADD COLUMN `placed_at` TIMESTAMP NULL DEFAULT NULL;

-- Saved status presets, per character.
CREATE TABLE IF NOT EXISTS `playerstatuses` (
    `id`     INT(11)     NOT NULL AUTO_INCREMENT,
    `charid` INT(11)     DEFAULT NULL,
    `name`   VARCHAR(64) DEFAULT '',
    `text`   LONGTEXT    DEFAULT (''),
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
