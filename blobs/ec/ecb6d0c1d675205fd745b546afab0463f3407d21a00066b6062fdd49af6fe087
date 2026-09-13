-- ============================================================================
--  poggy_auction · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_auction starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import. To manage it yourself instead, set
--  PoggyCoreConfig.Sql.AutoInstall = false in poggy_core/config.lua and import
--  this file, or run `poggycore sql install poggy_auction` in the server console.
-- ============================================================================

-- ============================================================================
-- AUCTION LISTINGS, MAILBOX AND BID HISTORY
-- ============================================================================

CREATE TABLE IF NOT EXISTS `poggy_auction_listings` (
    `id`              INT AUTO_INCREMENT PRIMARY KEY,
    `seller_id`       VARCHAR(100)   NOT NULL,
    `seller_name`     VARCHAR(100)   NOT NULL,
    `item_name`       VARCHAR(100)   NOT NULL,
    `item_label`      VARCHAR(150)   NOT NULL,
    `item_metadata`   TEXT           DEFAULT NULL,
    `quantity`        INT            NOT NULL DEFAULT 1,
    `start_price`     DECIMAL(12,2)  NOT NULL,
    `buyout_price`    DECIMAL(12,2)  DEFAULT NULL,
    `current_bid`     DECIMAL(12,2)  DEFAULT NULL,
    `current_bidder`  VARCHAR(100)   DEFAULT NULL,
    `bidder_name`     VARCHAR(100)   DEFAULT NULL,
    `bid_count`       INT            NOT NULL DEFAULT 0,
    `duration`        INT            NOT NULL,
    `category`        VARCHAR(50)    DEFAULT 'misc',
    `status`          ENUM('active','sold','expired','cancelled') NOT NULL DEFAULT 'active',
    `expires_at`      DATETIME       NOT NULL,
    `created_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    `updated_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_status    (`status`),
    INDEX idx_seller    (`seller_id`),
    INDEX idx_category  (`category`, `status`),
    INDEX idx_expires   (`expires_at`, `status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_auction_mailbox` (
    `id`              INT AUTO_INCREMENT PRIMARY KEY,
    `char_id`         VARCHAR(100)   NOT NULL,
    `type`            ENUM('item','money') NOT NULL,
    `item_name`       VARCHAR(100)   DEFAULT NULL,
    `item_label`      VARCHAR(150)   DEFAULT NULL,
    `item_metadata`   TEXT           DEFAULT NULL,
    `quantity`        INT            DEFAULT 0,
    `amount`          DECIMAL(12,2)  DEFAULT 0.00,
    `reason`          VARCHAR(255)   DEFAULT NULL,
    `listing_id`      INT            DEFAULT NULL,
    `collected`       TINYINT(1)     NOT NULL DEFAULT 0,
    `created_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_char     (`char_id`, `collected`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_auction_bids` (
    `id`              INT AUTO_INCREMENT PRIMARY KEY,
    `listing_id`      INT            NOT NULL,
    `bidder_id`       VARCHAR(100)   NOT NULL,
    `bidder_name`     VARCHAR(100)   NOT NULL,
    `bid_amount`      DECIMAL(12,2)  NOT NULL,
    `created_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_listing  (`listing_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ============================================================================
-- SHIPMENT ORDERS (government supply catalog deliveries)
-- ============================================================================

CREATE TABLE IF NOT EXISTS `poggy_shipment_orders` (
    `id`              INT AUTO_INCREMENT PRIMARY KEY,
    `char_id`         VARCHAR(100)   NOT NULL,
    `char_name`       VARCHAR(100)   NOT NULL,
    `item_name`       VARCHAR(100)   NOT NULL,
    `item_label`      VARCHAR(150)   NOT NULL,
    `quantity`        INT            NOT NULL DEFAULT 1,
    `price_per_unit`  DECIMAL(12,2)  NOT NULL,
    `shipping_fee`    DECIMAL(12,2)  NOT NULL DEFAULT 0.00,
    `total_paid`      DECIMAL(12,2)  NOT NULL,
    `company_key`     VARCHAR(50)    NOT NULL DEFAULT 'rail',
    `company_name`    VARCHAR(150)   NOT NULL DEFAULT 'Valentine Rail Freight Co.',
    `status`          ENUM('processing','in_transit','out_for_delivery','delivered','cancelled') NOT NULL DEFAULT 'processing',
    `delivery_shop_id` INT           DEFAULT NULL,
    `archived`        TINYINT(1)     NOT NULL DEFAULT 0,
    `processing_ends` DATETIME       NOT NULL,
    `transit_ends`    DATETIME       NOT NULL,
    `delivery_ends`   DATETIME       NOT NULL,
    `created_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_char   (`char_id`),
    INDEX idx_status (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Added after the first release: shop delivery (poggy_markets) and archiving.
ALTER TABLE `poggy_shipment_orders` ADD COLUMN `delivery_shop_id` INT DEFAULT NULL AFTER `status`;
ALTER TABLE `poggy_shipment_orders` ADD COLUMN `archived` TINYINT(1) NOT NULL DEFAULT 0 AFTER `delivery_shop_id`;

-- ============================================================================
-- ITEM REQUESTS (player want listings)
-- ============================================================================

CREATE TABLE IF NOT EXISTS `poggy_item_requests` (
    `id`              INT AUTO_INCREMENT PRIMARY KEY,
    `requester_id`    VARCHAR(100)   NOT NULL,
    `requester_name`  VARCHAR(100)   NOT NULL,
    `item_name`       VARCHAR(100)   NOT NULL,
    `item_label`      VARCHAR(150)   NOT NULL,
    `quantity`        INT            NOT NULL DEFAULT 1,
    `qty_filled`      INT            NOT NULL DEFAULT 0,
    `price_per_unit`  DECIMAL(12,2)  NOT NULL,
    `escrow_total`    DECIMAL(12,2)  NOT NULL,
    `escrow_spent`    DECIMAL(12,2)  NOT NULL DEFAULT 0.00,
    `is_persistent`   TINYINT(1)     NOT NULL DEFAULT 0,
    `expires_at`      DATETIME       DEFAULT NULL,
    `status`          ENUM('open','partial','filled','cancelled','expired') NOT NULL DEFAULT 'open',
    `created_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    `updated_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_requester (`requester_id`),
    INDEX idx_status    (`status`),
    INDEX idx_expires   (`expires_at`, `status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_request_fills` (
    `id`              INT AUTO_INCREMENT PRIMARY KEY,
    `request_id`      INT            NOT NULL,
    `fulfiller_id`    VARCHAR(100)   NOT NULL,
    `fulfiller_name`  VARCHAR(100)   NOT NULL,
    `quantity`        INT            NOT NULL,
    `price_per_unit`  DECIMAL(12,2)  NOT NULL,
    `gross_pay`       DECIMAL(12,2)  NOT NULL,
    `tax_charged`     DECIMAL(12,2)  NOT NULL DEFAULT 0.00,
    `net_pay`         DECIMAL(12,2)  NOT NULL,
    `created_at`      TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_request   (`request_id`),
    INDEX idx_fulfiller (`fulfiller_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
