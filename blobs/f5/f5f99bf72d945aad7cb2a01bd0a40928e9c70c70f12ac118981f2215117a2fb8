-- ============================================================================
--  poggy_chess · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_chess starts: missing tables,
--  columns and indexes are added and anything already there is left alone.
--  There is nothing to import.
--
--  To manage it yourself instead, set PoggyCoreConfig.Sql.AutoInstall = false
--  in poggy_core/config.lua and import this file, or run
--  `poggycore sql install poggy_chess` in the server console.
-- ============================================================================


-- ----------------------------------------------------------------------------
--  poggy_chess_games: one row per game of chess or checkers.
--
--  Seats are "white" and "black" in both games (in checkers the white seat
--  plays the red pieces). result is the winning seat, or draw / none.
--  state is the position (a FEN for chess, 32 squares + turn for checkers);
--  moves is the whole game as JSON, one entry per move with the position after
--  it, which is what the game review in "My Record" replays.
--  time_control is "minutes+increment" ("5+3"), empty for a game without a
--  clock; white_ms / black_ms are the time left on each clock at the last save.
--  Character ids are the framework's own, as text.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_chess_games` (
    `id`                  INT AUTO_INCREMENT PRIMARY KEY,
    `table_id`            VARCHAR(64)  NOT NULL,
    `game_type`           VARCHAR(16)  NOT NULL DEFAULT 'chess',
    `variant`             VARCHAR(16)  DEFAULT NULL,
    `rules`               TEXT,
    `white_char_id`       VARCHAR(64)  DEFAULT NULL,
    `black_char_id`       VARCHAR(64)  DEFAULT NULL,
    `white_name`          VARCHAR(100) DEFAULT NULL,
    `black_name`          VARCHAR(100) DEFAULT NULL,
    `white_identifier`    VARCHAR(100) DEFAULT NULL,
    `black_identifier`    VARCHAR(100) DEFAULT NULL,
    `is_ai_game`          TINYINT(1)   DEFAULT 0,
    `ai_color`            VARCHAR(8)   DEFAULT 'black',
    `ai_difficulty`       VARCHAR(16)  DEFAULT NULL,
    `status`              ENUM('active','completed','abandoned') DEFAULT 'active',
    `result`              ENUM('white','black','draw','none')    DEFAULT 'none',
    `result_reason`       VARCHAR(24)  DEFAULT NULL,
    `fen`                 TEXT,
    `pgn`                 TEXT,
    `moves`               MEDIUMTEXT,
    `move_count`          INT          DEFAULT 0,
    `current_turn`        ENUM('white','black') DEFAULT 'white',
    `white_captures`      TEXT,
    `black_captures`      TEXT,
    `check_count_white`   INT          DEFAULT 0,
    `check_count_black`   INT          DEFAULT 0,
    `wager_amount`        INT          DEFAULT 0,
    `wager_currency`      VARCHAR(8)   DEFAULT NULL,
    `wager_state`         VARCHAR(12)  DEFAULT 'none',
    `white_rating`        INT          DEFAULT NULL,
    `white_rating_change` INT          DEFAULT NULL,
    `black_rating`        INT          DEFAULT NULL,
    `black_rating_change` INT          DEFAULT NULL,
    `time_control`        VARCHAR(16)  DEFAULT NULL,
    `white_ms`            INT          DEFAULT NULL,
    `black_ms`            INT          DEFAULT NULL,
    `created_at`          DATETIME     DEFAULT CURRENT_TIMESTAMP,
    `updated_at`          DATETIME     DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `last_moved_at`       DATETIME     DEFAULT NULL,
    `completed_at`        DATETIME     DEFAULT NULL,
    INDEX `idx_table_id`  (`table_id`),
    INDEX `idx_status`    (`status`),
    INDEX `idx_white`     (`white_char_id`),
    INDEX `idx_black`     (`black_char_id`),
    INDEX `idx_type`      (`game_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Added in 2.0.0 (servers that ran 1.x already have the table).
ALTER TABLE `poggy_chess_games` ADD COLUMN `game_type` VARCHAR(16) NOT NULL DEFAULT 'chess';
ALTER TABLE `poggy_chess_games` ADD COLUMN `variant` VARCHAR(16) DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `rules` TEXT;
ALTER TABLE `poggy_chess_games` ADD COLUMN `ai_color` VARCHAR(8) DEFAULT 'black';
ALTER TABLE `poggy_chess_games` ADD COLUMN `result_reason` VARCHAR(24) DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `moves` MEDIUMTEXT;
ALTER TABLE `poggy_chess_games` ADD COLUMN `wager_amount` INT DEFAULT 0;
ALTER TABLE `poggy_chess_games` ADD COLUMN `wager_currency` VARCHAR(8) DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `wager_state` VARCHAR(12) DEFAULT 'none';
ALTER TABLE `poggy_chess_games` ADD INDEX `idx_type` (`game_type`);
ALTER TABLE `poggy_chess_games` ADD COLUMN `white_rating` INT DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `white_rating_change` INT DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `black_rating` INT DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `black_rating_change` INT DEFAULT NULL;

-- Added in 2.1.0: the chess clock.
ALTER TABLE `poggy_chess_games` ADD COLUMN `time_control` VARCHAR(16) DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `white_ms` INT DEFAULT NULL;
ALTER TABLE `poggy_chess_games` ADD COLUMN `black_ms` INT DEFAULT NULL;

-- 1.x stored character ids as numbers; every framework's id fits as text.
ALTER TABLE `poggy_chess_games` MODIFY COLUMN `white_char_id` VARCHAR(64) DEFAULT NULL;
ALTER TABLE `poggy_chess_games` MODIFY COLUMN `black_char_id` VARCHAR(64) DEFAULT NULL;


-- ----------------------------------------------------------------------------
--  poggy_chess_payouts: money the script owes a character and has not been
--  able to hand over yet (a wager refunded after a server restart, while the
--  player was away). It is paid the next time that character is online, and
--  paid_at is filled in. Nothing here is ever deleted by the script.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_chess_payouts` (
    `id`          INT AUTO_INCREMENT PRIMARY KEY,
    `char_id`     VARCHAR(64)  NOT NULL,
    `amount`      INT          NOT NULL,
    `currency`    VARCHAR(8)   NOT NULL DEFAULT 'cash',
    `reason`      VARCHAR(32)  NOT NULL,
    `game_id`     INT          DEFAULT NULL,
    `created_at`  DATETIME     DEFAULT CURRENT_TIMESTAMP,
    `paid_at`     DATETIME     DEFAULT NULL,
    INDEX `idx_payout_char` (`char_id`),
    INDEX `idx_payout_paid` (`paid_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


-- ----------------------------------------------------------------------------
--  poggy_chess_ratings: each character's Elo rating, one row per game type
--  (chess, checkers). Only finished games between two players are rated.
--  name is the character's name at their last rated game, for the
--  leaderboard. The script can rebuild this table from the games at any time
--  (`/chesstable rerate`), so it holds nothing that is not in the games.
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS `poggy_chess_ratings` (
    `char_id`     VARCHAR(64)  NOT NULL,
    `game_type`   VARCHAR(16)  NOT NULL,
    `name`        VARCHAR(100) DEFAULT NULL,
    `rating`      INT          NOT NULL DEFAULT 1200,
    `peak`        INT          NOT NULL DEFAULT 1200,
    `games`       INT          NOT NULL DEFAULT 0,
    `wins`        INT          NOT NULL DEFAULT 0,
    `draws`       INT          NOT NULL DEFAULT 0,
    `losses`      INT          NOT NULL DEFAULT 0,
    `updated_at`  DATETIME     DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`char_id`, `game_type`),
    INDEX `idx_rating_board` (`game_type`, `rating`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
