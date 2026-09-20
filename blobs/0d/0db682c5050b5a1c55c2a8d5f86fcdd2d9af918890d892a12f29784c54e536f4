-- ============================================================================
--  poggy_tickets · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_tickets starts: missing tables,
--  columns and indexes are added and anything already there is left alone.
--  There is nothing to import.
--
--  To manage it yourself instead, set PoggyCoreConfig.Sql.AutoInstall = false
--  in poggy_core/config.lua and import this file, or run
--  `poggycore sql install poggy_tickets` in the server console.
-- ============================================================================

-- ----------------------------------------------------------------------------
--  poggy_tickets: one row per ticket. Times are unix seconds.
--  `conversation` is the whole back-and-forth as one JSON list; only the server
--  writes it. The identifier columns are what lets staff act on a player who
--  has already logged off. `unseen` = the player has news they have not read.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_tickets` (
  `id`                   INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `status`               VARCHAR(16)  NOT NULL DEFAULT 'open',
  `category`             VARCHAR(32)  NOT NULL,
  `priority`             VARCHAR(16)  NOT NULL DEFAULT 'medium',
  `description`          TEXT         NOT NULL,
  `clip_url`             VARCHAR(300) NULL,
  `want_presence`        TINYINT      NOT NULL DEFAULT 1,
  `reporter_account`     VARCHAR(128) NOT NULL,
  `reporter_char`        VARCHAR(64)  NULL,
  `reporter_name`        VARCHAR(128) NULL,
  `reporter_identifiers` LONGTEXT     NULL,
  `reported_account`     VARCHAR(128) NULL,
  `reported_char`        VARCHAR(64)  NULL,
  `reported_name`        VARCHAR(128) NULL,
  `reported_identifiers` LONGTEXT     NULL,
  `pos_x`                FLOAT        NOT NULL DEFAULT 0,
  `pos_y`                FLOAT        NOT NULL DEFAULT 0,
  `pos_z`                FLOAT        NOT NULL DEFAULT 0,
  `heading`              FLOAT        NOT NULL DEFAULT 0,
  `town`                 VARCHAR(64)  NULL,
  `claimed_by`           VARCHAR(128) NULL,
  `claimed_by_name`      VARCHAR(128) NULL,
  `close_reason`         VARCHAR(32)  NULL,
  `close_note`           VARCHAR(500) NULL,
  `closed_by_name`       VARCHAR(128) NULL,
  `discord_message_id`   VARCHAR(32)  NULL,
  `conversation`         LONGTEXT     NULL,
  `unseen`               TINYINT      NOT NULL DEFAULT 0,
  `created_at`           BIGINT       NOT NULL,
  `claimed_at`           BIGINT       NULL,
  `closed_at`            BIGINT       NULL,
  PRIMARY KEY (`id`),
  INDEX `idx_status` (`status`),
  INDEX `idx_reporter` (`reporter_account`),
  INDEX `idx_reported` (`reported_account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  poggy_ticket_staff: who is staff. One row per ACCOUNT, so it covers every
--  character that person plays. `roles` is a JSON list: admin, mod, helper,
--  developer. Anyone your framework counts as an admin is an Admin here too,
--  without a row.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_staff` (
  `account`    VARCHAR(128) NOT NULL,
  `name`       VARCHAR(128) NULL,
  `roles`      VARCHAR(255) NOT NULL,
  `hired_by`   VARCHAR(128) NULL,
  `hired_at`   BIGINT       NOT NULL,
  PRIMARY KEY (`account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  poggy_ticket_warnings: warnings given to players. A warning with no
--  acknowledged_at is shown again when the player next logs in.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_warnings` (
  `id`              INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `account`         VARCHAR(128) NOT NULL,
  `name`            VARCHAR(128) NULL,
  `reason`          VARCHAR(500) NOT NULL,
  `warned_by_name`  VARCHAR(128) NULL,
  `ticket_id`       INT UNSIGNED NULL,
  `created_at`      BIGINT       NOT NULL,
  `acknowledged_at` BIGINT       NULL,
  PRIMARY KEY (`id`),
  INDEX `idx_account` (`account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  poggy_ticket_audit: everything staff did, one row per action. Only ever
--  added to. Admins read it in the staff panel.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_audit` (
  `id`         BIGINT       NOT NULL AUTO_INCREMENT,
  `account`    VARCHAR(128) NOT NULL,
  `name`       VARCHAR(128) NULL,
  `action`     VARCHAR(32)  NOT NULL,
  `ticket_id`  INT UNSIGNED NULL,
  `target`     VARCHAR(128) NULL,
  `detail`     VARCHAR(500) NULL,
  `created_at` BIGINT       NOT NULL,
  PRIMARY KEY (`id`),
  INDEX `idx_time` (`created_at`),
  INDEX `idx_who` (`account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  poggy_ticket_help: a record of each "I need help right now" press: who,
--  where, when, and who went. These are not tickets.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_help` (
  `id`             INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `account`        VARCHAR(128) NOT NULL,
  `name`           VARCHAR(128) NULL,
  `pos_x`          FLOAT        NOT NULL DEFAULT 0,
  `pos_y`          FLOAT        NOT NULL DEFAULT 0,
  `pos_z`          FLOAT        NOT NULL DEFAULT 0,
  `town`           VARCHAR(64)  NULL,
  `responder_name` VARCHAR(128) NULL,
  `created_at`     BIGINT       NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
