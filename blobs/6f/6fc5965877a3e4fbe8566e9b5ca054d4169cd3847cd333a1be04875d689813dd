--[[
    poggy_core — the shared ban network (server, 0.20.0).

    Specification: docs\reference\poggy-tickets-spec.md §4-5 (development machine).

    A CHEATING ban on this server is a VOTE. When three servers running
    poggy_core have voted on the same player, every server that enforces the
    network refuses them. One server's ban is never a verdict; an unban takes
    the vote back; an owner can always let a blocked player in here ("allow").

    What leaves this server: SHA-256 hashes of identifiers and a player count.
    Never a name, never a reason, never a raw identifier.

    THE SAME RULE AS sv_bans.lua: FAIL OPEN.
      - The connect check reads MEMORY only: the published list (hash -> count),
        fetched from GitHub every ten minutes and kept in the database between
        restarts. A connect never waits on the internet.
      - A failed fetch keeps the list already held. A broken list is ignored.
      - Every rule that matters (who counts, the threshold, suspensions) is the
        network's, not this file's: this file is readable, so it is not trusted.

    OPT-OUT, on by default, said up front (owner's decision, 20 September 2026):
    PoggyCoreConfig.Bans.Network.Enforce and .Submit.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Net = {}
PoggyCore.BanNet = Net

-- Where the network lives (switched on 20 September 2026). An empty API would
-- make this file do nothing at all. A development server can point somewhere
-- else, or switch it off with an address of "off":
--     set poggy_bannet_api  "https://…workers.dev"
--     set poggy_bannet_list "https://raw.githubusercontent.com/…/bans.json"
local DEFAULT_API  = "https://poggy-bans.poggy-bans.workers.dev"
local DEFAULT_LIST = "https://raw.githubusercontent.com/RosewoodRidge/Poggy-Bans/main/bans.json"
local APPEAL_PAGE  = "rosewoodridge.xyz/appeal"

local REFRESH_MS   = 10 * 60 * 1000
local BEAT_SECONDS = 6 * 3600
local MAX_HASHES   = 6
local HASH         = "^[0-9a-f]+$"

local function cfg() return ((PoggyCoreConfig and PoggyCoreConfig.Bans) or {}).Network or {} end
local function api()
    local v = GetConvar("poggy_bannet_api", "")
    if v == "off" then return "" end
    return v ~= "" and v or DEFAULT_API
end
local function listUrl() local v = GetConvar("poggy_bannet_list", "") return v ~= "" and v or DEFAULT_LIST end
local function now() return os.time() end

--- Is the network switched on for this server at all?
function Net.Live()
    local B = PoggyCore.Bans
    return api() ~= "" and B ~= nil and ((PoggyCoreConfig.Bans or {}).Enabled ~= false)
        and (cfg().Enforce ~= false or cfg().Submit ~= false)
end

local state = { counts = {}, entries = 0, blockAt = 3, fetchedAt = nil, key = nil, allow = {}, serverState = nil, counting = nil }
Net.State = state

-- ---------------------------------------------------------------------------
-- Replaceable for tests
-- ---------------------------------------------------------------------------

--- Net.Http(method, url, bodyTable | nil, headers) -> status, decoded table | nil. Waits.
Net.Http = function(method, url, body, headers)
    local p = promise.new()
    headers = headers or {}
    headers["Content-Type"] = "application/json"
    headers["User-Agent"] = "poggy_core/" .. tostring(PoggyCore.VERSION or "?")
    PerformHttpRequest(url, function(status, text)
        local ok, data = pcall(json.decode, text or "")
        p:resolve({ status = tonumber(status) or 0, data = ok and type(data) == "table" and data or nil })
    end, method, body and json.encode(body) or "", headers)
    local r = Citizen.Await(p)
    return r.status, r.data
end

local function db() return PoggyCore.Bans.DB end

-- ---------------------------------------------------------------------------
-- Database: the list between restarts, this server's key, the allow list
-- ---------------------------------------------------------------------------

local TABLES = {
    poggy_core_kv = [[
CREATE TABLE IF NOT EXISTS `poggy_core_kv` (
  `k` VARCHAR(64) NOT NULL,
  `v` LONGTEXT    NULL,
  PRIMARY KEY (`k`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
    poggy_ban_network = [[
CREATE TABLE IF NOT EXISTS `poggy_ban_network` (
  `hash`  CHAR(64) NOT NULL,
  `count` INT      NOT NULL,
  PRIMARY KEY (`hash`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
    poggy_ban_allow = [[
CREATE TABLE IF NOT EXISTS `poggy_ban_allow` (
  `hash`       CHAR(64)     NOT NULL,
  `identifier` VARCHAR(128) NULL,
  `reason`     VARCHAR(500) NULL,
  `allowed_by` VARCHAR(128) NULL,
  `created_at` BIGINT       NOT NULL,
  PRIMARY KEY (`hash`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
}
local TABLE_ORDER = { "poggy_core_kv", "poggy_ban_network", "poggy_ban_allow" }

local function kvGet(k)
    local rows = db().query("SELECT `v` FROM `poggy_core_kv` WHERE `k` = ? LIMIT 1", { k })
    return rows and rows[1] and rows[1].v or nil
end

local function kvSet(k, v)
    local n = db().exec("UPDATE `poggy_core_kv` SET `v` = ? WHERE `k` = ?", { v, k })
    if not n or n == 0 then
        local rows = db().query("SELECT 1 AS found FROM `poggy_core_kv` WHERE `k` = ? LIMIT 1", { k })
        if rows and #rows == 0 then db().exec("INSERT INTO `poggy_core_kv` (`k`, `v`) VALUES (?, ?)", { k, v }) end
    end
end

--- Tables, the cached list, the key and the allow list. True when usable.
function Net.Init()
    for _, name in ipairs(TABLE_ORDER) do
        local rows, err = db().query(
            "SELECT 1 AS found FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ? LIMIT 1", { name })
        if not rows then
            Util.Warn("ban network: could not read the database (%s). The network is off; nobody is refused by it.", tostring(err))
            return false
        end
        if #rows == 0 then
            local ok, e = db().exec(TABLES[name])
            if not ok then
                Util.Error("ban network: could not create %s: %s. The network is off.", name, tostring(e))
                return false
            end
        end
    end
    local counts, n = {}, 0
    for _, r in ipairs(db().query("SELECT `hash`, `count` FROM `poggy_ban_network`") or {}) do
        local c = tonumber(r.count)
        if type(r.hash) == "string" and c and c > 0 then counts[r.hash] = c n = n + 1 end
    end
    state.counts, state.entries = counts, n
    state.blockAt = math.max(2, tonumber(kvGet("bannet_block_at")) or 3)
    state.key = kvGet("bannet_key")
    state.allow = {}
    for _, r in ipairs(db().query("SELECT `hash` FROM `poggy_ban_allow`") or {}) do state.allow[r.hash] = true end
    state.ready = true
    return true
end

-- ---------------------------------------------------------------------------
-- The list
-- ---------------------------------------------------------------------------

--- Fetch the published list. A failure of any kind keeps the one we hold.
function Net.Refresh()
    local status, data = Net.Http("GET", listUrl(), nil, {})
    if status ~= 200 or type(data) ~= "table" or type(data.bans) ~= "table" then
        return false, ("the list answered %s"):format(tostring(status))
    end
    local counts, n = {}, 0
    for hash, count in pairs(data.bans) do
        count = tonumber(count)
        if type(hash) == "string" and #hash == 64 and hash:find(HASH) and count and count >= 1 and count < 100000 then
            counts[hash] = math.floor(count)
            n = n + 1
        end
    end
    state.counts, state.entries, state.fetchedAt = counts, n, now()
    state.blockAt = math.max(2, math.floor(tonumber(data.blockAt) or 3))

    -- Kept for the next start, so a restart during a GitHub outage still enforces.
    db().exec("DELETE FROM `poggy_ban_network`")
    local marks, params = {}, {}
    local function flush()
        if #marks == 0 then return end
        db().exec("INSERT IGNORE INTO `poggy_ban_network` (`hash`, `count`) VALUES " .. table.concat(marks, ", "), params)
        marks, params = {}, {}
    end
    for hash, count in pairs(counts) do
        marks[#marks + 1] = "(?, ?)"
        params[#params + 1] = hash
        params[#params + 1] = count
        if #marks >= 200 then flush() end
    end
    flush()
    kvSet("bannet_block_at", tostring(state.blockAt))
    return true
end

--- The highest count the network holds for any of these hashes; 0 when this
--- server has allowed them. Memory only.
function Net.Count(hashes)
    local best = 0
    for _, h in ipairs(hashes or {}) do
        if state.allow[h] then return 0 end
        local c = state.counts[h]
        if c and c > best then best = c end
    end
    return best
end

--- The threshold in force: the network's, or the owner's if theirs is HIGHER.
function Net.BlockAt()
    return math.max(state.blockAt or 3, math.floor(tonumber(cfg().BlockAt) or 3), 2)
end

--- The message to refuse a connecting player with, or nil. Memory only.
function Net.Blocked(hashes)
    if not state.ready or not Net.Live() or cfg().Enforce == false then return nil end
    local n = Net.Count(hashes)
    if n < Net.BlockAt() then return nil end
    return table.concat({
        "You are blocked from this server.",
        ("You have been banned for cheating on %d servers in the Poggy network."):format(n),
        "Log in with your Cfx account to see where, and how to appeal:",
        APPEAL_PAGE,
    }, "\n"), n
end

-- ---------------------------------------------------------------------------
-- Talking to the network
-- ---------------------------------------------------------------------------

local function licenceHash()
    local key = GetConvar("sv_licenseKey", "")
    if key == "" then
        -- Not readable here: fall back to an id made once and kept.
        key = kvGet("bannet_install")
        if not key then
            key = PoggyCore.Bans.Sha256(("%d|%d|%s"):format(now(), math.random(1, 2 ^ 31), tostring(GetConvar("sv_hostname", ""))))
            kvSet("bannet_install", key)
        end
    end
    return PoggyCore.Bans.Sha256("poggy-net-v1:" .. key)
end

local function serverName()
    local n = cfg().ServerName
    if type(n) == "string" and n ~= "" then return n:sub(1, 80) end
    return tostring(GetConvar("sv_projectName", GetConvar("sv_hostname", ""))):gsub("%^%d", ""):sub(1, 80)
end

function Net.Register()
    local status, data = Net.Http("POST", api() .. "/register", { licence = licenceHash(), name = serverName(), version = PoggyCore.VERSION }, {})
    if status == 200 and data and type(data.key) == "string" and #data.key == 64 then
        state.key = data.key
        kvSet("bannet_key", data.key)
        return true
    end
    return false, (data and data.error) or ("the network answered " .. tostring(status))
end

--- A call with this server's key. Registers first when there is no key, and
--- once more when the key is refused (a database that was restored, say).
local function call(path, body)
    if not state.key then
        local ok, err = Net.Register()
        if not ok then return 0, { error = err } end
    end
    local status, data = Net.Http("POST", api() .. path, body, { ["Authorization"] = "Bearer " .. state.key })
    if status == 401 then
        state.key = nil
        local ok = Net.Register()
        if ok then status, data = Net.Http("POST", api() .. path, body, { ["Authorization"] = "Bearer " .. state.key }) end
    end
    return status, data
end

function Net.Heartbeat()
    local status, data = call("/heartbeat", { players = GetNumPlayerIndices and GetNumPlayerIndices() or 0, version = PoggyCore.VERSION,
        name = serverName(), appeal = tostring(cfg().AppealLink or ""):sub(1, 200), hideName = cfg().HideName == true })
    if status == 200 and data then
        if data.state == "suspended" and state.serverState ~= "suspended" then
            Util.Warn("ban network: this server's votes are SUSPENDED for review (too many cheat bans in a day). "
                .. "Your own bans still work here. Nothing was deleted.")
        end
        state.serverState, state.counting = data.state, data.counting
        return true
    end
    return false, (data and data.error) or tostring(status)
end

local function clipHashes(hashes)
    local out = {}
    for _, h in ipairs(hashes or {}) do
        if #out >= MAX_HASHES then break end
        out[#out + 1] = h
    end
    return out
end

local function setNetState(id, value)
    db().exec("UPDATE `poggy_bans` SET `net_state` = ? WHERE `id` = ?", { value, id })
end

--- Send one cheat ban as a vote. Left "pending" when the network cannot be reached.
function Net.Vote(ban)
    if not ban or ban.category ~= "cheat" or cfg().Submit == false then return false end
    -- The count from the moment of the ban: by now the banned player has been
    -- dropped, and the network sizes its "too many bans" limit from this number.
    local online = math.max(tonumber(ban.playersOnline) or tonumber(ban.players_online) or 0,
        GetNumPlayerIndices and GetNumPlayerIndices() or 0)
    local status, data = call("/vote", { hashes = clipHashes(ban.hashes), ban = tostring(ban.id), playersOnline = online })
    if status == 200 then
        setNetState(ban.id, "sent")
        if data and data.suspended then Util.Warn("ban network: %s", tostring(data.reason)) end
        return true
    end
    -- 403 = suspended or removed: trying again would change nothing.
    setNetState(ban.id, status == 403 and "refused" or "pending")
    return false, (data and data.error) or tostring(status)
end

function Net.Withdraw(ban)
    if not ban or ban.category ~= "cheat" then return false end
    local status = call("/withdraw", { hashes = clipHashes(ban.hashes) })
    if status == 200 then setNetState(ban.id, "withdrawn") return true end
    return false
end

--- Votes that could not be sent when the ban was made.
function Net.SendPending()
    local list = PoggyCore.Bans.List({ active = true, limit = 200 }) or {}
    for _, ban in ipairs(list) do
        if ban.category == "cheat" and ban.net_state == "pending" then Net.Vote(ban) end
    end
end

--- A timed cheat ban that has run out: the network only knows votes, not how
--- long a ban was for, so the vote is taken back here. Without this a 10 minute
--- ban would count against the player for a year.
function Net.WithdrawExpired()
    local rows = db().query(
        "SELECT * FROM `poggy_bans` WHERE `category` = 'cheat' AND `net_state` = 'sent' AND `revoked_at` IS NULL "
        .. "AND `expires_at` IS NOT NULL AND `expires_at` <= ? ORDER BY `id` LIMIT 50", { now() }) or {}
    for _, r in ipairs(rows) do
        local ok, hashes = pcall(json.decode, r.hashes or "")
        if ok and type(hashes) == "table" then
            Net.Withdraw({ id = tonumber(r.id), category = "cheat", hashes = hashes })
        end
    end
end

-- ---------------------------------------------------------------------------
-- A second chance: let one blocked player into THIS server
-- ---------------------------------------------------------------------------

function Net.Allow(identifier, reason, by)
    local ids, hashes = PoggyCore.Bans.Clean({ identifier })
    if #hashes == 0 then return nil, "that is not an identifier (license:…, fivem:…)" end
    reason = type(reason) == "string" and reason:gsub("^%s+", ""):gsub("%s+$", "") or ""
    if reason == "" then return nil, "say why (it is kept with the record)" end
    local rows = db().query("SELECT 1 AS found FROM `poggy_ban_allow` WHERE `hash` = ? LIMIT 1", { hashes[1] })
    if rows and #rows == 0 then
        db().exec("INSERT INTO `poggy_ban_allow` (`hash`, `identifier`, `reason`, `allowed_by`, `created_at`) VALUES (?, ?, ?, ?, ?)",
            { hashes[1], ids[1], reason:sub(1, 500), tostring(by or "console"):sub(1, 128), now() })
    end
    state.allow[hashes[1]] = true
    Util.Log("ban network: %s is allowed on this server (%s).", ids[1], reason)
    if Net.Live() then call("/vouch", { hashes = hashes }) end
    return true
end

function Net.Disallow(identifier)
    local _, hashes = PoggyCore.Bans.Clean({ identifier })
    if #hashes == 0 then return nil, "that is not an identifier" end
    db().exec("DELETE FROM `poggy_ban_allow` WHERE `hash` = ?", { hashes[1] })
    state.allow[hashes[1]] = nil
    if Net.Live() then call("/unvouch", { hashes = hashes }) end
    return true
end

-- ---------------------------------------------------------------------------
-- Hooks
-- ---------------------------------------------------------------------------

AddEventHandler("poggy_core:ban:added", function(ban)
    if not state.ready or not Net.Live() or type(ban) ~= "table" or ban.category ~= "cheat" then return end
    CreateThread(function() pcall(Net.Vote, ban) end)
end)

AddEventHandler("poggy_core:ban:removed", function(ban)
    if not state.ready or not Net.Live() or type(ban) ~= "table" or ban.category ~= "cheat" then return end
    CreateThread(function() pcall(Net.Withdraw, ban) end)
end)

-- Someone the network knows, but below the block: tell staff. poggy_tickets
-- turns this event into a pop-up for mods and admins.
AddEventHandler("playerJoining", function()
    local src = source
    local ok, err = pcall(function()
        if not state.ready or not Net.Live() or cfg().FlagStaff == false then return end
        local _, hashes = PoggyCore.Bans.IdentifiersOf(src)
        local n = Net.Count(hashes)
        if n < 1 then return end
        Util.Log("ban network: %s joined. Banned for cheating on %d other server(s) (blocked at %d).", tostring(GetPlayerName(src)), n, Net.BlockAt())
        TriggerEvent("poggy_core:ban:networkFlag", { src = src, name = GetPlayerName(src), count = n, blockAt = Net.BlockAt() })
    end)
    if not ok then Util.Debug("ban network: join flag failed: %s", tostring(err)) end
end)

-- ---------------------------------------------------------------------------
-- Console: poggycore bannet | banallow
-- ---------------------------------------------------------------------------

function Net.Command(src, args, say)
    local sub = (args[1] or ""):lower()
    if sub == "banallow" then
        if (args[2] or ""):lower() == "remove" then
            local ok, err = Net.Disallow(args[3] or "")
            say(ok and "^2removed^7: they are subject to the network again." or ("^1not removed^7: " .. tostring(err)))
            return
        end
        if not args[2] then
            local rows = db().query("SELECT `identifier`, `reason`, `allowed_by` FROM `poggy_ban_allow` ORDER BY `created_at` DESC") or {}
            say(("%d player(s) allowed here despite the network"):format(#rows))
            for _, r in ipairs(rows) do say(("  %s  (%s, by %s)"):format(tostring(r.identifier), tostring(r.reason), tostring(r.allowed_by))) end
            say("usage: poggycore banallow <identifier> <reason>   |   poggycore banallow remove <identifier>")
            return
        end
        local by = (src and src > 0 and GetPlayerName(src)) or "console"
        local ok, err = Net.Allow(args[2], table.concat(args, " ", 3), by)
        say(ok and "^2allowed^7: the network will not block them here, and your server counts as one voice for them."
            or ("^1not allowed^7: " .. tostring(err)))
        return
    end

    -- bannet: status. "refresh" reads the list first, so the numbers below are current.
    if api() == "" then say("the ban network is not switched on in this version of poggy_core.") return end
    if (args[2] or ""):lower() == "refresh" then
        local ok, err = Net.Refresh()
        say(ok and "^2list read just now^7" or ("^1list not read^7: " .. tostring(err)))
    end
    say(("enforce: %s   submit: %s   blocks at: %d server(s)"):format(cfg().Enforce ~= false and "^2on^7" or "^1off^7",
        cfg().Submit ~= false and "^2on^7" or "^1off^7", Net.BlockAt()))
    -- One banned player is several entries: each of their identifiers (licence,
    -- Cfx, Steam, Discord) is listed on its own, so any one of them is enough.
    say(("list: %d identifier(s) on the network (one player is usually 3 to 5 of them), %s"):format(state.entries,
        state.fetchedAt and ("read %d minute(s) ago"):format((now() - state.fetchedAt) // 60) or "from the last start (not read yet this start)"))
    say(("this server: %s%s"):format(state.key and "registered" or "^3not registered yet^7",
        state.serverState and (", " .. state.serverState .. (state.counting and ", votes count" or ", votes do not count yet (new servers wait 14 days)")) or ""))
end

-- ---------------------------------------------------------------------------
-- Start
-- ---------------------------------------------------------------------------

--- Said once, loudly: existing owners got this through an update, already on.
local function banner()
    if kvGet("bannet_banner") then return end
    kvSet("bannet_banner", tostring(now()))
    local line = "^3" .. string.rep("=", 78) .. "^7"
    print(line)
    print("^3  POGGY BAN NETWORK — this server is now part of it (new in poggy_core 0.20.0)^7")
    print("")
    print("  Players banned for CHEATING on 3 or more Poggy servers are refused here.")
    print("  Your own CHEAT bans count as one vote on the network. Rule-break bans never leave.")
    print("  Only hashes are shared: no names, no reasons, no identifiers.")
    print("")
    print("  Turn either half off in /poggy → poggy_core → Bans, or in poggy_core/config.lua:")
    print("      PoggyCoreConfig.Bans.Network.Enforce = false     (do not refuse anyone)")
    print("      PoggyCoreConfig.Bans.Network.Submit  = false     (do not share your bans)")
    print("  Let one blocked player in:  poggycore banallow <identifier> <reason>")
    print("  Status:                     poggycore bannet")
    print(line)
end

CreateThread(function()
    if api() == "" then return end                 -- not switched on in this build
    local waited = 0
    while not (PoggyCore.Bans and PoggyCore.Bans.DB and PoggyCore.Bans.DB.ready) and waited < 120000 do
        Wait(1000)
        waited = waited + 1000
    end
    if not (PoggyCore.Bans and PoggyCore.Bans.DB.ready) or not Net.Live() then return end
    if not Net.Init() then return end
    banner()

    local lastBeat = 0
    while true do
        pcall(function()
            if cfg().Enforce ~= false or cfg().FlagStaff ~= false then Net.Refresh() end
            if cfg().Submit ~= false then
                if (now() - lastBeat) >= BEAT_SECONDS then
                    if Net.Heartbeat() then lastBeat = now() end
                end
                Net.SendPending()
                Net.WithdrawExpired()
            end
        end)
        Wait(REFRESH_MS)
    end
end)
