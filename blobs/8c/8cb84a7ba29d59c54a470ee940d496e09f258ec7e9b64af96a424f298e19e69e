--[[
    poggy_core — bans (server, 0.19.0).

    Specification: docs\reference\poggy-tickets-spec.md §2 (development machine).

    A ban is against the ACCOUNT, never the character: every identifier the
    server gives for a player is hashed (SHA-256 of "poggy-bans-v1:" .. id) and
    a connecting player is refused when any of their hashes belongs to an
    active ban. Unbanning keeps the row (revoked_at), so bans are reversible
    and leave a history.

    THE RULE ABOVE ALL OTHERS: FAIL OPEN. Every Poggy script depends on
    poggy_core, so a fault here would lock players out of every Poggy server.
      - The connect check reads MEMORY only. Active bans are loaded at start
        and kept in step by Add / Remove (and re-read every few minutes, for
        the rare pair of servers sharing one database). A connect never waits
        on the database or the internet.
      - The check never defers a connection; it only ever refuses one, and
        only on a positive match.
      - The whole check is pcall'd. Any error lets the player in and says so
        once in the console. No database, no table, a failed load: nobody is
        banned, everybody gets in.

    Only hashes are meant to leave a server (the shared ban network, a later
    version). Raw identifiers stay in this server's own table.

    Verbs (sv_dispatch.lua): ban.add, ban.remove, ban.check, ban.list,
    player.kick, player.identifiers. The CALLING SCRIPT checks who asked; the
    console command and the hub panel check it themselves.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Bans = {}
PoggyCore.Bans = Bans

local SALT        = "poggy-bans-v1:"
local RELOAD_MS   = 10 * 60 * 1000
local CATEGORIES  = { cheat = true, ["local"] = true }
local SKIP_PREFIX = { ip = true }        -- an address is not an account

local function cfg() return (PoggyCoreConfig and PoggyCoreConfig.Bans) or {} end
local function enabled() return cfg().Enabled ~= false end
local function now() return os.time() end

local function cut(s, n)
    if s == nil then return nil end
    s = tostring(s)
    return #s > n and s:sub(1, n) or s
end

-- ---------------------------------------------------------------------------
-- SHA-256 (pure Lua 5.4; CFX has no hashing native that returns SHA-256)
-- ---------------------------------------------------------------------------

local K = {
    0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5, 0x3956c25b, 0x59f111f1, 0x923f82a4, 0xab1c5ed5,
    0xd807aa98, 0x12835b01, 0x243185be, 0x550c7dc3, 0x72be5d74, 0x80deb1fe, 0x9bdc06a7, 0xc19bf174,
    0xe49b69c1, 0xefbe4786, 0x0fc19dc6, 0x240ca1cc, 0x2de92c6f, 0x4a7484aa, 0x5cb0a9dc, 0x76f988da,
    0x983e5152, 0xa831c66d, 0xb00327c8, 0xbf597fc7, 0xc6e00bf3, 0xd5a79147, 0x06ca6351, 0x14292967,
    0x27b70a85, 0x2e1b2138, 0x4d2c6dfc, 0x53380d13, 0x650a7354, 0x766a0abb, 0x81c2c92e, 0x92722c85,
    0xa2bfe8a1, 0xa81a664b, 0xc24b8b70, 0xc76c51a3, 0xd192e819, 0xd6990624, 0xf40e3585, 0x106aa070,
    0x19a4c116, 0x1e376c08, 0x2748774c, 0x34b0bcb5, 0x391c0cb3, 0x4ed8aa4a, 0x5b9cca4f, 0x682e6ff3,
    0x748f82ee, 0x78a5636f, 0x84c87814, 0x8cc70208, 0x90befffa, 0xa4506ceb, 0xbef9a3f7, 0xc67178f2,
}

local function rrot(x, n) return ((x >> n) | (x << (32 - n))) & 0xffffffff end

--- SHA-256 of a string, as 64 lower-case hex characters.
function Bans.Sha256(msg)
    msg = tostring(msg)
    local len = #msg
    msg = msg .. "\128" .. string.rep("\0", (55 - len) % 64) .. string.pack(">I8", len * 8)

    local h0, h1, h2, h3 = 0x6a09e667, 0xbb67ae85, 0x3c6ef372, 0xa54ff53a
    local h4, h5, h6, h7 = 0x510e527f, 0x9b05688c, 0x1f83d9ab, 0x5be0cd19
    local w = {}

    for chunk = 1, #msg, 64 do
        for i = 0, 15 do w[i] = string.unpack(">I4", msg, chunk + i * 4) end
        for i = 16, 63 do
            local a, b = w[i - 15], w[i - 2]
            local s0 = rrot(a, 7) ~ rrot(a, 18) ~ (a >> 3)
            local s1 = rrot(b, 17) ~ rrot(b, 19) ~ (b >> 10)
            w[i] = (w[i - 16] + s0 + w[i - 7] + s1) & 0xffffffff
        end
        local a, b, c, d, e, f, g, h = h0, h1, h2, h3, h4, h5, h6, h7
        for i = 0, 63 do
            local S1 = rrot(e, 6) ~ rrot(e, 11) ~ rrot(e, 25)
            local ch = (e & f) ~ ((~e & 0xffffffff) & g)
            local t1 = (h + S1 + ch + K[i + 1] + w[i]) & 0xffffffff
            local S0 = rrot(a, 2) ~ rrot(a, 13) ~ rrot(a, 22)
            local maj = (a & b) ~ (a & c) ~ (b & c)
            local t2 = (S0 + maj) & 0xffffffff
            h, g, f, e, d, c, b, a = g, f, e, (d + t1) & 0xffffffff, c, b, a, (t1 + t2) & 0xffffffff
        end
        h0 = (h0 + a) & 0xffffffff  h1 = (h1 + b) & 0xffffffff
        h2 = (h2 + c) & 0xffffffff  h3 = (h3 + d) & 0xffffffff
        h4 = (h4 + e) & 0xffffffff  h5 = (h5 + f) & 0xffffffff
        h6 = (h6 + g) & 0xffffffff  h7 = (h7 + h) & 0xffffffff
    end
    return ("%08x%08x%08x%08x%08x%08x%08x%08x"):format(h0, h1, h2, h3, h4, h5, h6, h7)
end

-- ---------------------------------------------------------------------------
-- Identifiers
-- ---------------------------------------------------------------------------

--- "License:ABC " -> "license:abc"; nil for anything that is not "<type>:<value>"
--- or that is not an account (ip:).
local function cleanIdentifier(id)
    if type(id) ~= "string" then return nil end
    id = id:gsub("^%s+", ""):gsub("%s+$", ""):lower()
    local kind = id:match("^([%w_]+):.+$")
    if not kind or SKIP_PREFIX[kind] or #id > 128 then return nil end
    return id
end

--- The hash of one identifier.
function Bans.Hash(identifier)
    local id = cleanIdentifier(identifier)
    return id and Bans.Sha256(SALT .. id) or nil
end

--- A list of identifiers, cleaned and deduped, and their hashes (same order).
function Bans.Clean(list)
    local ids, hashes, seen = {}, {}, {}
    if type(list) == "string" then list = { list } end
    if type(list) ~= "table" then return ids, hashes end
    for _, raw in ipairs(list) do
        local id = cleanIdentifier(raw)
        if id and not seen[id] then
            seen[id] = true
            ids[#ids + 1] = id
            hashes[#hashes + 1] = Bans.Sha256(SALT .. id)
        end
    end
    return ids, hashes
end

--- Every account identifier of a connected (or connecting) player.
function Bans.IdentifiersOf(src)
    local raw = {}
    src = tonumber(src)
    if src and src > 0 and GetNumPlayerIdentifiers then
        for i = 0, (GetNumPlayerIdentifiers(src) or 0) - 1 do
            raw[#raw + 1] = GetPlayerIdentifier(src, i)
        end
    end
    return Bans.Clean(raw)
end

-- ---------------------------------------------------------------------------
-- Database
--
-- Created at start the way the hub's tables are (sv_hub_save.lua): asked of
-- information_schema first, CREATE sent only when the table is missing.
-- Times are unix seconds, so nothing depends on the database's time zone.
-- ---------------------------------------------------------------------------

local TABLES = {
    poggy_bans = [[
CREATE TABLE IF NOT EXISTS `poggy_bans` (
  `id`             INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `hashes`         LONGTEXT     NOT NULL,
  `identifiers`    LONGTEXT     NOT NULL,
  `name`           VARCHAR(128) NULL,
  `category`       VARCHAR(16)  NOT NULL DEFAULT 'local',
  `reason`         VARCHAR(500) NOT NULL,
  `evidence_url`   VARCHAR(300) NULL,
  `banned_by`      VARCHAR(128) NULL,
  `banned_by_name` VARCHAR(128) NULL,
  `created_at`     BIGINT       NOT NULL,
  `expires_at`     BIGINT       NULL,
  `revoked_at`     BIGINT       NULL,
  `revoked_by`     VARCHAR(128) NULL,
  `revoke_reason`  VARCHAR(500) NULL,
  `source`         VARCHAR(64)  NULL,
  `source_ref`     VARCHAR(64)  NULL,
  `players_online` INT          NOT NULL DEFAULT 0,
  `net_state`      VARCHAR(16)  NOT NULL DEFAULT 'none',
  PRIMARY KEY (`id`),
  INDEX `idx_active` (`revoked_at`, `expires_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
    poggy_ban_hashes = [[
CREATE TABLE IF NOT EXISTS `poggy_ban_hashes` (
  `ban_id` INT UNSIGNED NOT NULL,
  `hash`   CHAR(64)     NOT NULL,
  PRIMARY KEY (`ban_id`, `hash`),
  INDEX `idx_hash` (`hash`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
}
local TABLE_ORDER = { "poggy_bans", "poggy_ban_hashes" }

local DB = { ready = false }
Bans.DB = DB

--- Replaceable for tests: DB.query(sql, params) -> rows | nil, err;
--- DB.exec(sql, params) -> affected | nil, err. Both wait: call from a thread.
DB.query = function(sql, params)
    local Sql = PoggyCore.Sql
    if not Sql or not Sql.Db then return nil, "no database" end
    return Sql.Db.query(sql, params)
end
DB.exec = function(sql, params)
    local Sql = PoggyCore.Sql
    if not Sql or not Sql.Db then return nil, "no database" end
    return Sql.Db.exec(sql, params)
end
-- ---------------------------------------------------------------------------
-- The memory the connect check reads
-- ---------------------------------------------------------------------------

local active = {}      -- ban id -> ban (active ones only)
local byHash = {}      -- hash -> { ban id, ... }
local warnedCheck = false

local function decode(text)
    if type(text) ~= "string" or text == "" then return {} end
    local ok, t = pcall(json.decode, text)
    return ok and type(t) == "table" and t or {}
end

local function isActive(ban, at)
    if ban.revoked_at then return false end
    if ban.expires_at and ban.expires_at <= (at or now()) then return false end
    return true
end

local function rowToBan(r)
    return {
        id = tonumber(r.id),
        hashes = decode(r.hashes),
        identifiers = decode(r.identifiers),
        name = r.name,
        category = r.category or "local",
        reason = r.reason or "",
        evidence_url = r.evidence_url,
        banned_by = r.banned_by,
        banned_by_name = r.banned_by_name,
        created_at = tonumber(r.created_at) or 0,
        expires_at = tonumber(r.expires_at),
        revoked_at = tonumber(r.revoked_at),
        revoked_by = r.revoked_by,
        revoke_reason = r.revoke_reason,
        source = r.source,
        source_ref = r.source_ref,
        players_online = tonumber(r.players_online) or 0,
        net_state = r.net_state or "none",
    }
end

local function remember(ban)
    active[ban.id] = ban
    for _, h in ipairs(ban.hashes) do
        local list = byHash[h]
        if not list then list = {} byHash[h] = list end
        list[#list + 1] = ban.id
    end
end

local function forget(id)
    local ban = active[id]
    if not ban then return end
    active[id] = nil
    for _, h in ipairs(ban.hashes) do
        local list = byHash[h]
        if list then
            for i = #list, 1, -1 do if list[i] == id then table.remove(list, i) end end
            if #list == 0 then byHash[h] = nil end
        end
    end
end

--- Read every active ban into memory. A failure keeps what is already there.
function Bans.Load()
    local rows, err = DB.query(
        "SELECT * FROM `poggy_bans` WHERE `revoked_at` IS NULL AND (`expires_at` IS NULL OR `expires_at` > ?)",
        { now() })
    if not rows then return false, err end
    active, byHash = {}, {}
    for _, r in ipairs(rows) do
        local ban = rowToBan(r)
        if ban.id then remember(ban) end
    end
    return true
end

--- Create the tables when missing, then load. Returns true when bans can be used.
function Bans.Init()
    for _, name in ipairs(TABLE_ORDER) do
        local rows, err = DB.query(
            "SELECT 1 AS found FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ? LIMIT 1",
            { name })
        if not rows then
            Util.Warn("bans: could not read the database (%s). Bans are off; nobody is refused.", tostring(err))
            return false
        end
        if #rows == 0 then
            local ok, e = DB.exec(TABLES[name])
            if not ok then
                Util.Error("bans: could not create %s: %s. Bans are off; nobody is refused.", name, tostring(e))
                return false
            end
            Util.Log("^2✅ bans: created table %s^7", name)
        end
    end
    local ok, err = Bans.Load()
    if not ok then
        Util.Warn("bans: could not load the ban list (%s). Nobody is refused until it loads.", tostring(err))
        return false
    end
    DB.ready = true
    return true
end

-- ---------------------------------------------------------------------------
-- Reading
-- ---------------------------------------------------------------------------

--- The active ban matching any of these hashes, or nil. Memory only.
function Bans.Match(hashes)
    local at = now()
    for _, h in ipairs(hashes or {}) do
        local list = byHash[h]
        if list then
            for _, id in ipairs(list) do
                local ban = active[id]
                if ban and isActive(ban, at) then return ban end
            end
        end
    end
    return nil
end

--- "3 days", "5 hours", "permanent": how long is left of a ban.
function Bans.Remaining(ban, at)
    if not ban.expires_at then return "permanent" end
    local left = ban.expires_at - (at or now())
    if left <= 0 then return "expired" end
    if left >= 172800 then return ("%d days"):format(left // 86400) end
    if left >= 7200 then return ("%d hours"):format(left // 3600) end
    return ("%d minutes"):format(math.max(1, left // 60))
end

--- "2h", "7d", "30m", "1w", "perm" -> seconds (nil = permanent) | false, reason.
function Bans.ParseTime(text)
    text = tostring(text or ""):lower()
    if text == "perm" or text == "permanent" or text == "forever" or text == "0" then return nil end
    local n, unit = text:match("^(%d+)([mhdw])$")
    n = tonumber(n)
    if not n or n <= 0 then return false, "time looks like 30m, 2h, 7d, 1w or perm" end
    local mult = ({ m = 60, h = 3600, d = 86400, w = 604800 })[unit]
    return n * mult
end

--- What a refused player reads.
function Bans.Message(ban)
    local lines = {
        "You are banned from this server.",
        "Reason: " .. (ban.reason ~= "" and ban.reason or "none given"),
        ban.expires_at and ("Time left: " .. Bans.Remaining(ban)) or "This ban is permanent.",
        ("Ban #%d"):format(ban.id),
    }
    local appeal = cfg().AppealText
    if type(appeal) == "string" and appeal ~= "" then lines[#lines + 1] = appeal end
    return table.concat(lines, "\n")
end

--- The public view of a ban (verbs and the panel): no raw identifiers.
local function view(ban)
    return {
        id = ban.id, name = ban.name, category = ban.category, reason = ban.reason,
        evidence = ban.evidence_url, by = ban.banned_by_name, createdAt = ban.created_at,
        expiresAt = ban.expires_at, revokedAt = ban.revoked_at, revokedBy = ban.revoked_by,
        revokeReason = ban.revoke_reason, source = ban.source, sourceRef = ban.source_ref,
        active = isActive(ban), remaining = Bans.Remaining(ban), hashes = ban.hashes,
        playersOnline = ban.players_online,
    }
end
Bans.View = view

--- ban.check: { banned, ban?, network = { count, blocked } }.
function Bans.Check(p)
    local hashes
    if p.src then _, hashes = Bans.IdentifiersOf(p.src) else _, hashes = Bans.Clean(p.identifiers) end
    local ban = Bans.Match(hashes)
    local Net = PoggyCore.BanNet
    local count = (Net and Net.State.ready and Net.Live()) and Net.Count(hashes) or 0
    return {
        banned = ban ~= nil,
        ban = ban and view(ban) or nil,
        network = { count = count, blocked = count > 0 and count >= Net.BlockAt() },
    }
end

--- ban.list: newest first. opts: search, active (boolean), limit, offset.
function Bans.List(opts)
    opts = opts or {}
    local where, params = {}, {}
    if opts.active == true then
        where[#where + 1] = "`revoked_at` IS NULL AND (`expires_at` IS NULL OR `expires_at` > ?)"
        params[#params + 1] = now()
    elseif opts.active == false then
        where[#where + 1] = "(`revoked_at` IS NOT NULL OR (`expires_at` IS NOT NULL AND `expires_at` <= ?))"
        params[#params + 1] = now()
    end
    if type(opts.search) == "string" and opts.search ~= "" then
        local like = "%" .. cut(opts.search, 64):gsub("[%%_]", "\\%0") .. "%"
        where[#where + 1] = "(`name` LIKE ? OR `reason` LIKE ? OR `identifiers` LIKE ? OR `banned_by_name` LIKE ?)"
        for _ = 1, 4 do params[#params + 1] = like end
    end
    local limit = math.max(1, math.min(500, math.floor(tonumber(opts.limit) or 100)))
    local offset = math.max(0, math.floor(tonumber(opts.offset) or 0))
    local sql = "SELECT * FROM `poggy_bans`"
        .. (#where > 0 and (" WHERE " .. table.concat(where, " AND ")) or "")
        .. (" ORDER BY `id` DESC LIMIT %d OFFSET %d"):format(limit, offset)
    local rows, err = DB.query(sql, params)
    if not rows then return nil, err end
    local out = {}
    for _, r in ipairs(rows) do out[#out + 1] = rowToBan(r) end
    return out
end

--- One ban by id, active or not.
function Bans.Get(id)
    id = tonumber(id)
    if not id then return nil end
    if active[id] then return active[id] end
    local rows = DB.query("SELECT * FROM `poggy_bans` WHERE `id` = ? LIMIT 1", { id })
    return rows and rows[1] and rowToBan(rows[1]) or nil
end

-- ---------------------------------------------------------------------------
-- Writing
-- ---------------------------------------------------------------------------

--- { identifier, name } of whoever did it: a server id, 0 / nil = console.
local function actor(by, byName)
    local src = tonumber(by)
    if src and src > 0 then
        local ids = Bans.IdentifiersOf(src)
        local id = ids[1]
        for _, x in ipairs(ids) do if x:find("^license:") then id = x break end end
        return id or ("player " .. src), byName or GetPlayerName(src) or ("player " .. src)
    end
    if type(by) == "string" and by ~= "" and not src then return cut(by, 128), cut(byName or by, 128) end
    return "console", byName and cut(byName, 128) or "console"
end

local function playerCount()
    local n = 0
    if GetNumPlayerIndices then n = GetNumPlayerIndices() or 0 end
    return n
end

--- Ban an account. Returns the ban id | nil, reason.
--- p: src OR identifiers; reason; category ('local' | 'cheat'); duration (seconds,
--- nil = permanent); evidence; name; by (server id or text), byName; source, sourceRef.
function Bans.Add(p)
    if not enabled() then return nil, "bans are turned off in poggy_core's config" end
    if not DB.ready then return nil, "the ban list is not ready (no database?)" end

    local reason = type(p.reason) == "string" and p.reason:gsub("^%s+", ""):gsub("%s+$", "") or ""
    if reason == "" then return nil, "a ban needs a reason" end

    local src = tonumber(p.src)
    local ids, hashes, name
    if src and src > 0 and GetPlayerName(src) then
        ids, hashes = Bans.IdentifiersOf(src)
        name = p.name or GetPlayerName(src)
    else
        src = nil
        ids, hashes = Bans.Clean(p.identifiers)
        name = p.name
    end
    if #ids == 0 then return nil, "no account identifier to ban (is that player online?)" end

    local category = CATEGORIES[p.category] and p.category or "local"
    local duration = tonumber(p.duration)
    local at = now()
    local expires = (duration and duration > 0) and (at + math.floor(duration)) or nil
    local by, byName = actor(p.by, p.byName)
    local evidence = type(p.evidence) == "string" and p.evidence:find("^https?://") and cut(p.evidence, 300) or nil

    -- The marker finds our own row again: LAST_INSERT_ID cannot be trusted
    -- across oxmysql's connection pool.
    local marker = Bans.Sha256(("%s|%d|%d"):format(hashes[1], at, math.random(1, 2 ^ 31)))
    -- Only the columns that have a value are sent: a nil in the middle of a
    -- parameter list does not survive the trip to oxmysql.
    local cols, params = {}, {}
    local function col(name, value)
        if value == nil then return end
        cols[#cols + 1] = "`" .. name .. "`"
        params[#params + 1] = value
    end
    col("hashes", json.encode(hashes))
    col("identifiers", json.encode(ids))
    col("name", cut(name, 128))
    col("category", category)
    col("reason", cut(reason, 500))
    col("evidence_url", evidence)
    col("banned_by", by)
    col("banned_by_name", byName)
    col("created_at", at)
    col("expires_at", expires)
    col("source", cut(p.source, 64))
    col("source_ref", marker)
    col("players_online", playerCount())
    local ok, err = DB.exec(("INSERT INTO `poggy_bans` (%s) VALUES (%s)"):format(
        table.concat(cols, ", "), ("?, "):rep(#cols):sub(1, -3)), params)
    if not ok then return nil, "the database refused the ban: " .. tostring(err) end

    local rows = DB.query("SELECT `id` FROM `poggy_bans` WHERE `source_ref` = ? AND `created_at` = ? LIMIT 1",
        { marker, at })
    local id = rows and rows[1] and tonumber(rows[1].id)
    if not id then return nil, "the ban was written but could not be read back" end
    local ref = cut(p.sourceRef, 64)
    if ref then
        DB.exec("UPDATE `poggy_bans` SET `source_ref` = ? WHERE `id` = ?", { ref, id })
    else
        DB.exec("UPDATE `poggy_bans` SET `source_ref` = NULL WHERE `id` = ?", { id })
    end

    for _, h in ipairs(hashes) do
        DB.exec("INSERT IGNORE INTO `poggy_ban_hashes` (`ban_id`, `hash`) VALUES (?, ?)", { id, h })
    end

    local ban = {
        id = id, hashes = hashes, identifiers = ids, name = name, category = category, reason = cut(reason, 500),
        evidence_url = evidence, banned_by = by, banned_by_name = byName, created_at = at, expires_at = expires,
        source = p.source, source_ref = p.sourceRef, players_online = playerCount(), net_state = "none",
    }
    remember(ban)
    Util.Log("bans: #%d %s banned by %s (%s, %s): %s", id, tostring(name or ids[1]), byName, category,
        Bans.Remaining(ban), ban.reason)

    -- Anyone online on that account goes now, whichever character they are on.
    for _, player in ipairs(GetPlayers()) do
        local _, theirs = Bans.IdentifiersOf(player)
        local hit = false
        for _, h in ipairs(theirs) do
            for _, mine in ipairs(hashes) do if h == mine then hit = true break end end
            if hit then break end
        end
        if hit then DropPlayer(player, Bans.Message(ban)) end
    end

    TriggerEvent("poggy_core:ban:added", view(ban))
    return id
end

--- Unban. The row stays, marked revoked. Returns true | nil, reason.
function Bans.Remove(p)
    if not DB.ready then return nil, "the ban list is not ready (no database?)" end
    local id = tonumber(p.id)
    local ban = id and Bans.Get(id)
    if not ban then return nil, "there is no ban with that number" end
    if ban.revoked_at then return nil, "that ban was already lifted" end

    local reason = type(p.reason) == "string" and p.reason:gsub("^%s+", ""):gsub("%s+$", "") or ""
    if reason == "" then return nil, "lifting a ban needs a reason" end

    local _, byName = actor(p.by, p.byName)
    local at = now()
    local ok, err = DB.exec(
        "UPDATE `poggy_bans` SET `revoked_at` = ?, `revoked_by` = ?, `revoke_reason` = ? WHERE `id` = ? AND `revoked_at` IS NULL",
        { at, byName, cut(reason, 500), id })
    if not ok then return nil, "the database refused: " .. tostring(err) end

    forget(id)
    ban.revoked_at, ban.revoked_by, ban.revoke_reason = at, byName, cut(reason, 500)
    Util.Log("bans: #%d (%s) lifted by %s: %s", id, tostring(ban.name or "?"), byName, ban.revoke_reason)
    TriggerEvent("poggy_core:ban:removed", view(ban))
    return true
end

--- player.kick.
function Bans.Kick(src, reason)
    src = tonumber(src)
    if not src or src <= 0 or not GetPlayerName(src) then return nil, "that player is not online" end
    DropPlayer(src, (type(reason) == "string" and reason ~= "") and cut(reason, 500) or "You were kicked.")
    return true
end

-- ---------------------------------------------------------------------------
-- The connect check. Memory only, pcall'd, never defers. See the header.
-- ---------------------------------------------------------------------------

--- The message to refuse this connecting player with, or nil to let them in.
function Bans.OnConnecting(src)
    if not enabled() or not DB.ready then return nil end
    local _, hashes = Bans.IdentifiersOf(src)
    local ban = Bans.Match(hashes)
    if ban then return Bans.Message(ban) end
    -- The shared network (sv_bannet.lua): memory only, like the check above.
    local Net = PoggyCore.BanNet
    if Net and Net.Blocked then return (Net.Blocked(hashes)) end
    return nil
end

AddEventHandler("playerConnecting", function(name, setKickReason)
    local src = source
    local ok, msg = pcall(Bans.OnConnecting, src)
    if not ok then
        if not warnedCheck then
            warnedCheck = true
            Util.Warn("bans: the connect check failed (%s). The player was let in. This is said once.", tostring(msg))
        end
        return
    end
    if msg then
        if setKickReason then pcall(setKickReason, msg) end
        CancelEvent()
        -- The identifier is printed so an owner can let a network-blocked player in:
        --     poggycore banallow <identifier> <reason>
        local ids = Bans.IdentifiersOf(src)
        Util.Log("bans: refused %s (%s).", tostring(name), tostring(ids[1] or "no identifier"))
    end
end)

-- ---------------------------------------------------------------------------
-- Console: poggycore ban | unban | baninfo
-- ---------------------------------------------------------------------------

local function when(t) return t and os.date("%d %b %Y %H:%M", t) or "-" end

local function describe(ban, say)
    say(("^3#%d^7  %s  [%s]  %s"):format(ban.id, tostring(ban.name or "?"), ban.category,
        isActive(ban) and ("^1active^7, " .. Bans.Remaining(ban)) or (ban.revoked_at and "^2lifted^7" or "^2expired^7")))
    say(("     reason: %s"):format(ban.reason))
    say(("     by %s on %s%s"):format(tostring(ban.banned_by_name), when(ban.created_at),
        ban.source and ("  (from " .. ban.source .. (ban.source_ref and (" " .. ban.source_ref) or "") .. ")") or ""))
    if ban.revoked_at then
        say(("     lifted by %s on %s: %s"):format(tostring(ban.revoked_by), when(ban.revoked_at), tostring(ban.revoke_reason)))
    end
end

--- args: the words after "poggycore"; args[1] is ban | unban | baninfo.
function Bans.Command(src, args, say)
    local sub = (args[1] or ""):lower()
    if not enabled() then say("bans are turned off (PoggyCoreConfig.Bans.Enabled).") return end
    if not DB.ready then say("^1the ban list is not ready^7: no database, or the tables could not be made. See the start of the console.") return end

    if sub == "ban" then
        local target, timeText = args[2], args[3]
        if not target or not timeText then
            say("usage: poggycore ban <server id | identifier> <30m|2h|7d|1w|perm> [cheat] <reason>")
            return
        end
        local duration, why = Bans.ParseTime(timeText)
        if duration == false then say("^1" .. why .. "^7") return end
        local from, category = 4, "local"
        if (args[4] or ""):lower() == "cheat" then category, from = "cheat", 5 end
        local reason = table.concat(args, " ", from)
        local p = { reason = reason, category = category, duration = duration, by = src, source = "console" }
        if tonumber(target) then p.src = tonumber(target) else p.identifiers = { target } end
        local id, err = Bans.Add(p)
        say(id and ("^2banned^7: ban #%d"):format(id) or ("^1not banned^7: " .. tostring(err)))

    elseif sub == "unban" then
        local id = tonumber(args[2])
        if not id then say("usage: poggycore unban <ban number> <reason>") return end
        local ok, err = Bans.Remove({ id = id, reason = table.concat(args, " ", 3), by = src })
        say(ok and ("^2ban #%d lifted^7"):format(id) or ("^1not lifted^7: " .. tostring(err)))

    elseif sub == "baninfo" then
        local target = args[2]
        if not target then
            local list = Bans.List({ active = true, limit = 20 }) or {}
            say(("%d active ban(s)%s"):format(#list, #list == 20 and " (showing the newest 20)" or ""))
            for _, ban in ipairs(list) do describe(ban, say) end
            return
        end
        local found = {}
        if target:find(":", 1, true) then
            local _, hashes = Bans.Clean({ target })
            local rows = hashes[1] and DB.query(
                "SELECT b.* FROM `poggy_bans` b JOIN `poggy_ban_hashes` h ON h.`ban_id` = b.`id` WHERE h.`hash` = ? ORDER BY b.`id` DESC",
                { hashes[1] }) or {}
            for _, r in ipairs(rows) do found[#found + 1] = rowToBan(r) end
        else
            local n = tonumber(target)
            if n and GetPlayerName(n) then
                -- A server id: every ban on any of that player's identifiers.
                local _, hashes = Bans.IdentifiersOf(n)
                local seen = {}
                for _, h in ipairs(hashes) do
                    local rows = DB.query(
                        "SELECT b.* FROM `poggy_bans` b JOIN `poggy_ban_hashes` h ON h.`ban_id` = b.`id` WHERE h.`hash` = ?", { h }) or {}
                    for _, r in ipairs(rows) do
                        local ban = rowToBan(r)
                        if ban.id and not seen[ban.id] then seen[ban.id] = true found[#found + 1] = ban end
                    end
                end
                say(("%s (server id %d): %d ban(s) on record"):format(GetPlayerName(n), n, #found))
            elseif n then
                local ban = Bans.Get(n)
                if ban then found[1] = ban end
            end
        end
        if #found == 0 then say("nothing found for " .. target) end
        for _, ban in ipairs(found) do describe(ban, say) end
    end
end

-- ---------------------------------------------------------------------------
-- /poggy → poggy_core → Bans (hub spec §10). The hub checks the edit
-- permission before it calls this.
-- ---------------------------------------------------------------------------

local TIME_OPTIONS = {
    { value = "perm", label = "Permanent" }, { value = "1h", label = "1 hour" }, { value = "12h", label = "12 hours" },
    { value = "1d", label = "1 day" }, { value = "3d", label = "3 days" }, { value = "7d", label = "7 days" },
    { value = "30d", label = "30 days" }, { value = "custom", label = "Other (type it below)" },
}

local function panelRead()
    if not DB.ready then
        return { columns = {}, rows = {}, note = "The ban list is not ready: no database, or its tables could not be made." }
    end
    local list = Bans.List({ limit = 500 }) or {}
    local rows = {}
    for _, ban in ipairs(list) do
        local live = isActive(ban)
        rows[#rows + 1] = {
            key = tostring(ban.id),
            cells = {
                id = ban.id,
                name = ban.name or "?",
                category = ban.category == "cheat" and "Cheating" or "Local",
                reason = ban.reason,
                by = ban.banned_by_name or "?",
                when = when(ban.created_at),
                state = live and ("Active, " .. Bans.Remaining(ban))
                    or (ban.revoked_at and ("Lifted by " .. tostring(ban.revoked_by)) or "Expired"),
            },
            note = ban.revoked_at and ("Lifted: " .. tostring(ban.revoke_reason)) or nil,
            level = live and "error" or nil,
            actions = live and { "unban" } or {},
        }
    end

    local players = { { value = "", label = "Nobody online (use an identifier)" } }
    for _, player in ipairs(GetPlayers()) do
        players[#players + 1] = { value = tostring(player), label = ("%s · %s"):format(player, GetPlayerName(player) or "?") }
    end
    table.sort(players, function(a, b) return (tonumber(a.value) or 0) < (tonumber(b.value) or 0) end)

    return {
        note = "A ban is on the account, so it follows the player across characters. Lifting a ban keeps it in this list.",
        columns = {
            { field = "id", label = "#", width = 60 }, { field = "name", label = "Player" },
            { field = "category", label = "Kind", width = 100 }, { field = "reason", label = "Reason" },
            { field = "by", label = "By" }, { field = "when", label = "When" }, { field = "state", label = "State" },
        },
        rows = rows,
        actions = {
            { id = "ban", label = "Ban a player", icon = "shield", scope = "panel", danger = true, confirm = "simple",
              input = {
                  { field = "player", label = "Online player", options = players },
                  { field = "identifier", label = "Or an identifier", placeholder = "license:…  (for someone who has left)" },
                  { field = "time", label = "For how long", options = TIME_OPTIONS, default = "perm", required = true },
                  { field = "custom", label = "Other length", placeholder = "30m, 2h, 7d, 1w" },
                  { field = "category", label = "Kind", default = "local", required = true, options = {
                      { value = "local", label = "Local (rule break)" }, { value = "cheat", label = "Cheating" } } },
                  { field = "reason", label = "Reason (the player reads this)", picker = "multiline", required = true },
              } },
            { id = "unban", label = "Lift ban", icon = "unlock", scope = "row", confirm = "simple",
              input = { { field = "reason", label = "Why is it lifted?", required = true } } },
        },
    }
end

local function panelAction(payload, who)
    local input = type(payload.input) == "table" and payload.input or {}
    local by, byName = who and who.src or 0, who and who.name or nil
    if payload.action == "ban" then
        local timeText = input.time == "custom" and input.custom or input.time
        local duration, why = Bans.ParseTime(timeText)
        if duration == false then return false, why end
        local p = { reason = input.reason, category = input.category, duration = duration,
            by = by, byName = byName, source = "hub" }
        if tonumber(input.player) then p.src = tonumber(input.player)
        elseif type(input.identifier) == "string" and input.identifier ~= "" then p.identifiers = { input.identifier }
        else return false, "Pick an online player or type an identifier." end
        local id, err = Bans.Add(p)
        if not id then return false, err end
        return true, ("Banned. Ban #%d."):format(id)
    elseif payload.action == "unban" then
        local ok, err = Bans.Remove({ id = payload.key, reason = input.reason, by = by, byName = byName })
        if not ok then return false, err end
        return true, "Ban lifted."
    end
    return false, "unknown action"
end

exports("HubPanel", function(panelId, action, payload, who)
    if panelId ~= "bans" then return false, "unknown panel" end
    if action == "read" then return panelRead() end
    if action == "action" then return panelAction(type(payload) == "table" and payload or {}, who) end
    return false, "This list is changed with its buttons."
end)

-- ---------------------------------------------------------------------------
-- Start
-- ---------------------------------------------------------------------------

CreateThread(function()
    if not enabled() then return end
    local waited = 0
    while GetResourceState("oxmysql") ~= "started" and waited < 60000 do
        Wait(500)
        waited = waited + 500
    end
    Bans.Init()
    while true do
        Wait(RELOAD_MS)
        if DB.ready then pcall(Bans.Load) end
    end
end)
