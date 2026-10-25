-- ============================================================================
--  poggy_balloon · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_balloon starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_balloon` in the server console.
-- ============================================================================

-- Owned balloons, one per character. Added in 1.7.0.
CREATE TABLE IF NOT EXISTS `poggy_balloons` (
    `charid`    VARCHAR(64) NOT NULL,
    `price`     INT(11)     NOT NULL DEFAULT 0,
    `bought_at` TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`charid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
