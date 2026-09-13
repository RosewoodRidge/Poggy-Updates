-- ============================================================================
--  poggy_multijob · migration 001 · rows from `marshal_multi_jobs`
--  ---------------------------------------------------------------------------
--  multijob 1.3.0 and earlier kept jobs in `marshal_multi_jobs`. poggy_core
--  runs this once, after install.sql, to copy those rows into `poggy_multijob`.
--  A server that never had the old table records it as done without running.
--
--  `marshal_multi_jobs` is never dropped or edited apart from the `joblabel`
--  column added below, so it stays your rollback. Delete it yourself once you
--  are satisfied.
-- ============================================================================
-- poggy: only-if-table marshal_multi_jobs

-- Older installs predate `joblabel`. Add it to the OLD table so the copy below
-- has a column to read.
ALTER TABLE `marshal_multi_jobs`
  ADD COLUMN `joblabel` VARCHAR(255) DEFAULT 'Unknown';

-- Copy every row. INSERT IGNORE never overwrites a row already in the new table.
INSERT IGNORE INTO `poggy_multijob`
    (`cid`, `job`, `joblabel`, `jobgrade`, `firstname`, `lastname`, `lastonline`)
SELECT
    `cid`,
    `job`,
    COALESCE(`joblabel`, 'Unknown'),
    `jobgrade`,
    `firstname`,
    `lastname`,
    `lastonline`
FROM `marshal_multi_jobs`;
