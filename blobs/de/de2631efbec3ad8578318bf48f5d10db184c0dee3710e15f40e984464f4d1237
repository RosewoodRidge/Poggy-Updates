-- ============================================================================
--  poggy_multijob · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_multijob starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_multijob` in the server console.
-- ============================================================================

-- `cid` is the framework's character id as text: a number on VORP, a
-- citizenid such as 'ABC12345' on RSG.
CREATE TABLE IF NOT EXISTS `poggy_multijob` (
    `cid`        VARCHAR(64)  NOT NULL,
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

-- 1.7.1: `cid` used to be INT(11), which cannot hold an RSG citizenid. Sent
-- only while the column's type differs; the numbers already stored on VORP
-- are kept as text.
ALTER TABLE `poggy_multijob` MODIFY COLUMN `cid` VARCHAR(64) NOT NULL;
