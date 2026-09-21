-- poggy: only-if-table emote_favorites
-- ============================================================================
--  poggy_emotes · migration 001
--  ---------------------------------------------------------------------------
--  Brings favourites across from js_emotes, the script poggy_emotes replaces.
--
--  js_emotes kept one row per starred emote in `emote_favorites`. poggy_emotes
--  keeps one row per character with the names in a JSON array, so each
--  character's rows are gathered into a single list here.
--
--  Runs once, and only on a server that has the old table. Everyone else
--  skips it. The old table is left exactly as it is, so nothing is lost if
--  you want to go back.
-- ============================================================================

INSERT IGNORE INTO `poggy_emotes_player` (`charidentifier`, `favourites`, `recents`)
SELECT
    CAST(`charidentifier` AS CHAR),
    CONCAT('["', GROUP_CONCAT(`emote_name` SEPARATOR '","'), '"]'),
    '[]'
FROM `emote_favorites`
WHERE `emote_name` IS NOT NULL AND `emote_name` <> ''
GROUP BY `charidentifier`;
