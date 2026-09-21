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
  `escalated_to`         VARCHAR(500) NULL,
  `resolution`           TEXT         NULL,
  `resolution_by`        VARCHAR(128) NULL,
  `is_public`            TINYINT      NOT NULL DEFAULT 0,
  `archived_at`          BIGINT       NULL,
  `archived_by_name`     VARCHAR(128) NULL,
  `last_player_at`       BIGINT       NULL,
  `last_staff_at`        BIGINT       NULL,
  `nudged_at`            BIGINT       NULL,
  PRIMARY KEY (`id`),
  INDEX `idx_status` (`status`),
  INDEX `idx_reporter` (`reporter_account`),
  INDEX `idx_reported` (`reported_account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Added in 1.1.0. poggy_core adds only the ones that are missing.
--   escalated_to   JSON list of role ids brought onto the ticket (they see it too)
--   resolution     the answer staff marked, pinned in the chat and shown to players
--   is_public      1: any player may read it, names hidden (never a player report)
--   archived_at    hidden from every list but the Archived filter
--   last_*_at      who spoke last, for the "still need help?" nudge and auto-close
ALTER TABLE `poggy_tickets` ADD COLUMN `escalated_to` VARCHAR(500) NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `resolution` TEXT NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `resolution_by` VARCHAR(128) NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `is_public` TINYINT NOT NULL DEFAULT 0;
ALTER TABLE `poggy_tickets` ADD COLUMN `archived_at` BIGINT NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `archived_by_name` VARCHAR(128) NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `last_player_at` BIGINT NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `last_staff_at` BIGINT NULL;
ALTER TABLE `poggy_tickets` ADD COLUMN `nudged_at` BIGINT NULL;
ALTER TABLE `poggy_tickets` ADD INDEX `idx_public` (`is_public`);

-- ----------------------------------------------------------------------------
--  poggy_ticket_staff: who is staff. One row per ACCOUNT, so it covers every
--  character that person plays. `roles` is a JSON list of role ids from
--  poggy_ticket_roles. Anyone your framework counts as an admin is an Admin
--  here too, without a row.
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

-- ----------------------------------------------------------------------------
--  poggy_ticket_roles (1.1.0): the staff roles, yours to add to and rename.
--  Edited in the staff panel (Staff -> Roles) or in /poggy, never here.
--    rank      higher outranks lower; it orders lists and nothing else
--    powers    JSON list of what the role may do; ["*"] is everything
--    kinds     JSON list of ticket kinds it sees; ["*"] is every kind.
--              NULL: not set yet, so the first start copies Config.Routing
--    chats     JSON list of OTHER roles whose staff chat it may read; ["*"] all
--    helps     1: told about "I need help right now" presses.
--              NULL: not set yet, so the first start copies Config.HelpRoles
--    webhook   a Discord webhook the role's staff chat is copied to
--    locked    1: cannot be deleted or lose powers (Admin, so nobody is locked out)
--    deleted   1: removed by an owner. The row stays so it is not seeded again.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_roles` (
  `id`         VARCHAR(32)  NOT NULL,
  `label`      VARCHAR(64)  NOT NULL,
  `color`      VARCHAR(16)  NULL,
  `rank`       INT          NOT NULL DEFAULT 0,
  `powers`     TEXT         NULL,
  `kinds`      TEXT         NULL,
  `chats`      TEXT         NULL,
  `helps`      TINYINT      NULL,
  `webhook`    VARCHAR(300) NULL,
  `locked`     TINYINT      NOT NULL DEFAULT 0,
  `deleted`    TINYINT      NOT NULL DEFAULT 0,
  `created_at` BIGINT       NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT IGNORE INTO `poggy_ticket_roles` (`id`, `label`, `color`, `rank`, `powers`, `kinds`, `chats`, `helps`, `locked`) VALUES
  ('admin', 'Admin', '#f08a8a', 100, '["*"]', '["*"]', '["*"]', 1, 1);
INSERT IGNORE INTO `poggy_ticket_roles` (`id`, `label`, `color`, `rank`, `powers`, `chats`) VALUES
  ('mod', 'Mod', '#f2b880', 50, '["claim","reply","close","teleport","category","priority","assign","escalate","resolve","warn","kick","notes"]', '["helper"]');
INSERT IGNORE INTO `poggy_ticket_roles` (`id`, `label`, `color`, `rank`, `powers`, `chats`) VALUES
  ('helper', 'Helper', '#8fd3a8', 20, '["claim","reply","close","teleport","category","priority","escalate"]', '[]');
INSERT IGNORE INTO `poggy_ticket_roles` (`id`, `label`, `color`, `rank`, `powers`, `chats`) VALUES
  ('developer', 'Developer', '#b9a8f0', 30, '["claim","reply","close","teleport","category","priority","escalate","resolve"]', '[]');

-- ----------------------------------------------------------------------------
--  poggy_ticket_chats (1.1.0): staff chat rooms an admin made. Every role also
--  has a room of its own, which needs no row. Only the ROOM is kept: messages
--  live in memory and are gone at the next restart, on purpose.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_chats` (
  `id`         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `label`      VARCHAR(64)  NOT NULL,
  `roles`      TEXT         NULL,
  `webhook`    VARCHAR(300) NULL,
  `created_by` VARCHAR(128) NULL,
  `created_at` BIGINT       NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  poggy_ticket_canned (1.1.0): ready-made replies. `keywords` is a comma list;
--  a reply whose words appear in what the player wrote is offered first.
--  {player}, {staff} and {id} are filled in when it is used.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_canned` (
  `id`         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `label`      VARCHAR(64)  NOT NULL,
  `body`       VARCHAR(500) NOT NULL,
  `keywords`   VARCHAR(300) NULL,
  `deleted`    TINYINT      NOT NULL DEFAULT 0,
  `created_at` BIGINT       NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (1, 'On my way', 'Hello {player}, this is {staff}. I am on my way to you now.', 'help,stuck,where,come');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (2, 'Need a clip', 'Thanks for the report. Do you have a video clip? Paste the link here and we can act much faster.', 'cheat,cheater,hack,hacker,aimbot,rdm,shot,killed,exploit');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (3, 'Checking now', 'Thanks {player}. We are checking this now and will write here as soon as we know more.', 'report,bug,broken');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (4, 'Steps to repeat it', 'Thanks for the bug report. What were you doing just before it happened, and does it happen every time?', 'bug,glitch,error,broken,crash');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (5, 'Lost items', 'Sorry about that. Which items did you lose, how many, and roughly when? We will check the logs.', 'lost,items,inventory,missing,gone,disappeared,horse,wagon');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (6, 'Try relogging', 'Please log out fully and back in, then tell us here whether it is still happening.', 'invisible,frozen,loading,black screen,desync');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (7, 'Dealt with', 'This has been dealt with. Thank you for reporting it; it helps keep the server fair.', '');
INSERT IGNORE INTO `poggy_ticket_canned` (`id`, `label`, `body`, `keywords`) VALUES
  (8, 'Anything else?', 'Is there anything else we can help with? If not, we will close this ticket.', '');

-- ----------------------------------------------------------------------------
--  poggy_ticket_notes (1.1.0): staff notes about a player. Not a warning: the
--  player never sees it and it counts toward nothing.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_notes` (
  `id`         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `account`    VARCHAR(128) NOT NULL,
  `name`       VARCHAR(128) NULL,
  `note`       VARCHAR(500) NOT NULL,
  `by_name`    VARCHAR(128) NULL,
  `created_at` BIGINT       NOT NULL,
  PRIMARY KEY (`id`),
  INDEX `idx_account` (`account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  poggy_ticket_watch (1.1.0): players staff want to hear about when they log in.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_watch` (
  `account`    VARCHAR(128) NOT NULL,
  `name`       VARCHAR(128) NULL,
  `reason`     VARCHAR(300) NULL,
  `by_name`    VARCHAR(128) NULL,
  `created_at` BIGINT       NOT NULL,
  PRIMARY KEY (`account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------------------------------------------------------
--  The web panel (1.1.0, off unless Config.Web.Enabled). Two small tables.
--  poggy_ticket_web: this server's community ID and its key for the relay.
--  poggy_ticket_weblinks: which Cfx.re account ("fivem:<id>") belongs to which
--  game account. Every player, staff or not: written at login when their game
--  client names a Cfx.re account, or when they confirm a link code on the website.
--  poggy_ticket_webchars: the characters an account has played since the web panel
--  was on, so the website can show tickets under each one.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_ticket_web` (
  `k` VARCHAR(32) NOT NULL,
  `v` TEXT        NULL,
  PRIMARY KEY (`k`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_ticket_weblinks` (
  `account` VARCHAR(128) NOT NULL,
  `cfx_id`  VARCHAR(16)  NOT NULL,
  `name`    VARCHAR(128) NULL,
  `admin`   TINYINT      NOT NULL DEFAULT 0,
  `seen_at` BIGINT       NOT NULL,
  PRIMARY KEY (`account`),
  INDEX `idx_cfx` (`cfx_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `poggy_ticket_webchars` (
  `account` VARCHAR(128) NOT NULL,
  `char_id` VARCHAR(64)  NOT NULL,
  `name`    VARCHAR(128) NULL,
  `seen_at` BIGINT       NOT NULL,
  PRIMARY KEY (`account`, `char_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
