-- ============================================================================
--  poggy_markets · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_markets starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_markets` in the server console.
-- ============================================================================

--  poggy_markets uses the standard `playershops` and `shop_sales_log` tables.
--  If you are coming from syn_stores your data is already there and this file
--  only adds the columns that were missing -- nothing is dropped or renamed.

-- ----------------------------------------------------------------------------
--  1. Shops
--  ---------------------------------------------------------------------------
--  Created only if you do not already have it.  An existing table is left
--  alone here and topped up in step 2.
--
--  Note on charidentifier: this is VARCHAR so the same schema works on QBCore
--  and RSG, whose character ids are strings.  On VORP it holds a number and
--  behaves exactly as before.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `playershops` (
    `id`             INT(11)       NOT NULL AUTO_INCREMENT,
    `identifier`     VARCHAR(64)   NOT NULL DEFAULT '0',
    `charidentifier` VARCHAR(64)   NOT NULL DEFAULT '0',
    `name`           VARCHAR(64)   NOT NULL DEFAULT '',
    `coords`         LONGTEXT      NOT NULL,
    `items`          LONGTEXT      NOT NULL,
    `weapons`        LONGTEXT      NOT NULL,
    `buyitems`       LONGTEXT      NOT NULL,
    `solditems`      LONGTEXT      NOT NULL,
    `employees`      LONGTEXT      NOT NULL,
    `slots`          INT(11)       NOT NULL DEFAULT 1000,
    `ledger`         DOUBLE        NOT NULL DEFAULT 0,
    `taxledger`      DOUBLE        NOT NULL DEFAULT 0,
    `price`          DOUBLE        NOT NULL DEFAULT 0,
    `level`          INT(11)       NOT NULL DEFAULT 0,
    `blip`           TINYINT(1)    NOT NULL DEFAULT 1,
    `blipsprite`     VARCHAR(64)   NOT NULL DEFAULT '',
    `webhook`        LONGTEXT      NOT NULL,
    `repo`           TINYINT(1)    NOT NULL DEFAULT 0,
    PRIMARY KEY (`id`),
    KEY `idx_owner` (`charidentifier`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  2. Columns poggy_markets needs
--  ---------------------------------------------------------------------------
--  These were added to syn_stores over time by separate migration files, so an
--  older database may be missing some.  Each one is added only when the column
--  is not already there.
-- ----------------------------------------------------------------------------
ALTER TABLE `playershops` ADD COLUMN `weapons`    LONGTEXT    NOT NULL;
ALTER TABLE `playershops` ADD COLUMN `buyitems`   LONGTEXT    NOT NULL;
ALTER TABLE `playershops` ADD COLUMN `solditems`  LONGTEXT    NOT NULL;
ALTER TABLE `playershops` ADD COLUMN `employees`  LONGTEXT    NOT NULL;
ALTER TABLE `playershops` ADD COLUMN `webhook`    LONGTEXT    NOT NULL;
ALTER TABLE `playershops` ADD COLUMN `blipsprite` VARCHAR(64) NOT NULL DEFAULT '';
ALTER TABLE `playershops` ADD COLUMN `taxledger`  DOUBLE      NOT NULL DEFAULT 0;
ALTER TABLE `playershops` ADD COLUMN `price`      DOUBLE      NOT NULL DEFAULT 0;
ALTER TABLE `playershops` ADD COLUMN `repo`       TINYINT(1)  NOT NULL DEFAULT 0;

-- Empty JSON columns break decoding, so give any NULL or blank ones a value.
UPDATE `playershops` SET `items`     = '[]' WHERE `items`     IS NULL OR `items`     = '';
UPDATE `playershops` SET `weapons`   = '[]' WHERE `weapons`   IS NULL OR `weapons`   = '';
UPDATE `playershops` SET `buyitems`  = '[]' WHERE `buyitems`  IS NULL OR `buyitems`  = '';
UPDATE `playershops` SET `solditems` = '[]' WHERE `solditems` IS NULL OR `solditems` = '';
UPDATE `playershops` SET `employees` = '[]' WHERE `employees` IS NULL OR `employees` = '';
UPDATE `playershops` SET `coords`    = '{}' WHERE `coords`    IS NULL OR `coords`    = '';
UPDATE `playershops` SET `webhook`   = ''   WHERE `webhook`   IS NULL;

-- ----------------------------------------------------------------------------
--  3. Sales log
--  ---------------------------------------------------------------------------
--  Drives the dashboard and analytics tabs.  Existing syn_stores history is
--  picked up automatically because this is the same table.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `shop_sales_log` (
    `id`               INT(11)      NOT NULL AUTO_INCREMENT,
    `shop_id`          INT(11)      NOT NULL,
    `shop_type`        TINYINT      NOT NULL DEFAULT 1,
    `char_id`          VARCHAR(64)  NOT NULL DEFAULT '0',
    `char_name`        VARCHAR(100)          DEFAULT '',
    `item_name`        VARCHAR(100) NOT NULL DEFAULT '',
    `item_label`       VARCHAR(100)          DEFAULT '',
    `quantity`         INT(11)      NOT NULL DEFAULT 1,
    `unit_price`       DOUBLE       NOT NULL DEFAULT 0,
    `total_price`      DOUBLE       NOT NULL DEFAULT 0,
    `transaction_type` ENUM('sale','purchase','withdraw','deposit','tax','ghost')
                                    NOT NULL DEFAULT 'sale',
    `created_at`       TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `idx_shop_date` (`shop_id`, `created_at`),
    KEY `idx_item`      (`item_name`),
    KEY `idx_type`      (`transaction_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- poggy_markets logs two transaction kinds syn_stores did not have.  Without
-- this, tax collection and ghost sales fail to write a log row.  It only runs
-- when the column's type is not already this enum.
ALTER TABLE `shop_sales_log`
    MODIFY COLUMN `transaction_type`
    ENUM('sale','purchase','withdraw','deposit','tax','ghost')
    NOT NULL DEFAULT 'sale';

ALTER TABLE `shop_sales_log` ADD COLUMN `shop_type` TINYINT NOT NULL DEFAULT 1;

-- ----------------------------------------------------------------------------
--  4. Shop deed
--  ---------------------------------------------------------------------------
--  Only needed when Config.PlayerShops.creationItem is set.  Change the table
--  name if your framework keeps its item list somewhere other than `items`.
-- ----------------------------------------------------------------------------
INSERT IGNORE INTO `items` (`item`, `label`, `limit`, `can_remove`, `type`, `usable`)
VALUES ('shoptoken', 'Shop Deed', 5, 1, 'item_standard', 1);

-- ----------------------------------------------------------------------------
--  5. Dynamic pricing
--  ---------------------------------------------------------------------------
--  Used by modules/pricing when Config.Modules.dynamicPricing is true.  One row
--  per tracked item, plus the snapshots behind the price charts.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_markets_prices` (
    `item_name`   VARCHAR(100)  NOT NULL,
    `market`      VARCHAR(50)   NOT NULL DEFAULT '',
    `item_label`  VARCHAR(200)  NOT NULL DEFAULT '',
    `base_price`  DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    `multiplier`  DECIMAL(6,4)  NOT NULL DEFAULT 1.0000,
    `total_sold`  INT UNSIGNED  NOT NULL DEFAULT 0,
    `last_trade`  TIMESTAMP     NULL DEFAULT NULL,
    `trend`       ENUM('up','down','stable') NOT NULL DEFAULT 'stable',
    `updated_at`  TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP
                                ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`item_name`),
    KEY `idx_market` (`market`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_markets_price_history` (
    `id`          BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `item_name`   VARCHAR(100)    NOT NULL,
    `price`       DECIMAL(10,2)   NOT NULL DEFAULT 0.00,
    `multiplier`  DECIMAL(6,4)    NOT NULL DEFAULT 1.0000,
    `recorded_at` TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `idx_item_time` (`item_name`, `recorded_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  6. Commodities exchange
--  ---------------------------------------------------------------------------
--  Used by modules/exchange when Config.Modules.exchange and dynamicPricing are
--  both true: each character's exchange balance, open positions and trades.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_markets_accounts` (
    `char_id`    VARCHAR(64)   NOT NULL,
    `balance`    DECIMAL(12,2) NOT NULL DEFAULT 0.00,
    `updated_at` TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP
                               ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`char_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_markets_positions` (
    `id`        BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `char_id`   VARCHAR(64)     NOT NULL,
    `item_name` VARCHAR(100)    NOT NULL,
    `side`      ENUM('long','short') NOT NULL DEFAULT 'long',
    `quantity`  INT             NOT NULL DEFAULT 0,
    `avg_price` DECIMAL(10,4)   NOT NULL DEFAULT 0.0000,
    `opened_at` TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uq_position` (`char_id`, `item_name`, `side`),
    KEY `idx_char` (`char_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_markets_trades` (
    `id`         BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `char_id`    VARCHAR(64)     NOT NULL,
    `item_name`  VARCHAR(100)    NOT NULL,
    `action`     VARCHAR(20)     NOT NULL,
    `quantity`   INT             NOT NULL DEFAULT 0,
    `price`      DECIMAL(10,4)   NOT NULL DEFAULT 0.0000,
    `total`      DECIMAL(12,2)   NOT NULL DEFAULT 0.00,
    `fee`        DECIMAL(10,2)   NOT NULL DEFAULT 0.00,
    `pnl`        DECIMAL(12,2)   NOT NULL DEFAULT 0.00,
    `created_at` TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `idx_char_time` (`char_id`, `created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
