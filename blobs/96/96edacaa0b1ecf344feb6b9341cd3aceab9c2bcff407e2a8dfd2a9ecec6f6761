-- ============================================================================
--  poggy_emotes · database
--  ---------------------------------------------------------------------------
--  poggy_core runs this file every time poggy_emotes starts: missing tables,
--  columns and indexes are added and everything already there is left alone.
--  There is nothing to import.
--
--  To manage it yourself instead, set PoggyCoreConfig.Sql.AutoInstall = false
--  in poggy_core/config.lua and import this file, or run
--  `poggycore sql install poggy_emotes` in the server console.
-- ============================================================================

-- One row per character: the emotes they starred, and the ones they played
-- last. Both are stored as a JSON array of emote names, e.g. ["wave","bow"].
--
-- charidentifier is a VARCHAR because RSG and QBR character ids are text,
-- not numbers. VORP's numeric ids store here perfectly well.
CREATE TABLE IF NOT EXISTS `poggy_emotes_player` (
    `charidentifier` VARCHAR(64) NOT NULL,
    `favourites`     TEXT        DEFAULT ('[]'),
    `recents`        TEXT        DEFAULT ('[]'),
    `updated_at`     TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`charidentifier`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
