-- ============================================================================
--  poggy_badge · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_badge starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_badge` in the server console.
-- ============================================================================

-- Badge position presets, saved per character.
CREATE TABLE IF NOT EXISTS `poggy_badge_presets` (
    `id`       INT(11)      NOT NULL AUTO_INCREMENT,
    `charid`   VARCHAR(64)  NOT NULL,
    `name`     VARCHAR(32)  NOT NULL,
    `bone`     VARCHAR(32)  NOT NULL DEFAULT 'SKEL_Spine5',
    `offset_x` FLOAT        NOT NULL DEFAULT 0,
    `offset_y` FLOAT        NOT NULL DEFAULT 0,
    `offset_z` FLOAT        NOT NULL DEFAULT 0,
    `rot_x`    FLOAT        NOT NULL DEFAULT 0,
    `rot_y`    FLOAT        NOT NULL DEFAULT 0,
    `rot_z`    FLOAT        NOT NULL DEFAULT 0,
    PRIMARY KEY (`id`),
    INDEX `idx_charid` (`charid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Tables made before the bone selector had no bone column. Existing presets
-- get SKEL_Spine5 (collar / upper chest).
ALTER TABLE `poggy_badge_presets`
    ADD COLUMN `bone` VARCHAR(32) NOT NULL DEFAULT 'SKEL_Spine5' AFTER `name`;
