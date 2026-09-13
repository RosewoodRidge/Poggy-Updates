-- ============================================================================
--  poggy_multijob · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_multijob starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_multijob` in the server console.
-- ============================================================================

CREATE TABLE IF NOT EXISTS `poggy_multijob` (
    `cid`        INT(11)      NOT NULL,
    `job`        VARCHAR(255) NOT NULL,
    `joblabel`   VARCHAR(255) DEFAULT 'Unknown',
    `jobgrade`   INT(11)      NOT NULL,
    `firstname`  VARCHAR(255) NOT NULL,
    `lastname`   VARCHAR(255) NOT NULL,
    `lastonline` VARCHAR(255) NOT NULL,
    PRIMARY KEY (`cid`, `job`)
);

-- Tables created from an early copy of this file have no `joblabel`.
ALTER TABLE `poggy_multijob`
  ADD COLUMN `joblabel` VARCHAR(255) DEFAULT 'Unknown' AFTER `job`;
