-- ============================================================================
--  poggy_character_storage · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_character_storage starts: missing
--  tables, columns and indexes are added and everything already there is left
--  alone. There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_character_storage` in the
--  server console. (The table keeps its original name, character_storage.)
-- ============================================================================

CREATE TABLE IF NOT EXISTS `character_storage` (
  `id`               INT(11)       NOT NULL AUTO_INCREMENT,
  `owner_charid`     INT(11)       NOT NULL,
  `storage_name`     VARCHAR(50)   NOT NULL,
  `pos_x`            FLOAT(10,6)   NOT NULL,
  `pos_y`            FLOAT(10,6)   NOT NULL,
  `pos_z`            FLOAT(10,6)   NOT NULL,
  `authorized_users` LONGTEXT      NULL,
  `authorized_jobs`  TEXT          NOT NULL DEFAULT ('{}'),
  `capacity`         INT(11)       NOT NULL DEFAULT 50,
  `created_at`       TIMESTAMP     DEFAULT CURRENT_TIMESTAMP,
  `last_accessed`    TIMESTAMP     DEFAULT CURRENT_TIMESTAMP,
  `is_preset`        TINYINT(1)    NOT NULL DEFAULT 0,
  `money_balance`    DECIMAL(15,2) NOT NULL DEFAULT 0.00,
  `ledger_history`   LONGTEXT      NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `owner_charid` (`owner_charid`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;

-- Columns added after 1.0.5. Tables from older versions get whichever are missing.
ALTER TABLE `character_storage`
  ADD COLUMN `authorized_jobs` TEXT NOT NULL DEFAULT ('{}'),
  ADD COLUMN `last_accessed` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ADD COLUMN `is_preset` TINYINT(1) NOT NULL DEFAULT 0,
  ADD COLUMN `money_balance` DECIMAL(15,2) NOT NULL DEFAULT 0.00,
  ADD COLUMN `ledger_history` LONGTEXT NULL DEFAULT NULL;

-- `authorized_users` (1.1.0+) holds [{"id":123,"level":"basic"}, ...] instead of
-- the old [123, 456]. No schema change: rows in the old format are converted by
-- ParseAuthorizedUsers() the first time they are read. Access levels are in the README.
