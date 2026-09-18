--[[
    poggy_core — QBCore RedM (QBR) adapter.   Written against qbr-core 1.0.3 / qbr-inventory 1.0.1.
    NOT YET VERIFIED IN GAME. Every call below was read from the installed source
    on the QBR test server (file:line noted where it matters); none has run live.

    Framework traps this file exists to absorb, all confirmed in the installed source:

    * There is no core object. Every qbr-core API is a flat export on the
      resource (server/functions.lua, server/player.lua, shared/main.lua,
      server/exports.lua); the probe in sh_detect.lua hands back a marker table
      and this file talks to exports['qbr-core'] directly.
    * Every Player is a COPY. exports['qbr-core']:GetPlayer(src) returns the
      QBCore.Players[src] table through a funcref call, so it is msgpack-
      serialised on the way over: Player.PlayerData is a snapshot taken at
      call time and Player.Functions.* are funcrefs that act on the real
      object inside qbr-core (functions.lua:35-38). Reading PlayerData is fine;
      WRITING to it does nothing. Every mutation here goes through a
      Player.Functions.* call, and every read fetches the player again.
    * Money types are named: PlayerData.money.cash and .bank by default
      (config.lua:13, QBConfig.Money.MoneyTypes; a server may add more, e.g.
      gold, and init reads that list). RemoveMoney refuses a negative balance
      only for the types in DontAllowMinus — cash alone by default
      (config.lua:14, player.lua:256-261) — so the bank may go negative; the
      no_funds guard here applies to every currency. AddMoney / RemoveMoney /
      SetMoney answer true, false (unknown type), or NOTHING for a negative
      amount (player.lua:231, 252, 280). There are no money items.
    * job.grade is a TABLE: { name, level }. SetJob(job, grade) lower-cases
      the job, does tostring(grade) and indexes QBShared.Jobs[job].grades by
      that STRING key (player.lua:146-159): a nil grade becomes "nil" and lands
      on 'No Grades' level 0, so this file always passes a whole number and
      re-uses the current level when the job name is unchanged. SetJob also
      resets onduty to the job's defaultDuty (player.lua:153) and returns false
      for a job the shared table does not know. Labels come from QBShared.Jobs.
    * qbr-core fires NO server event on a job change. SetJob sends only the
      client event QBCore:Client:OnJobUpdate (player.lua:171), and SetJobDuty
      sends nothing but a PlayerData push. sv_core.lua therefore keeps the last
      job seen per player and re-checks it (a) after this adapter's own jobSet
      and (b) when the player's client relays QBCore:Client:OnJobUpdate as a
      "look again" poke (client/cl_core.lua); the job itself is always read
      back from qbr-core, never taken from the client. QBCore:Server:PlayerLoaded
      (player.lua:469) does fire locally, with the Player object, and seeds it.
    * Inventory is inside qbr-core, not qbr-inventory. Player.Functions.AddItem
      (player.lua:332-371) checks the weight against QBConfig.Player.MaxWeight,
      stacks only type = 'item' non-unique items, otherwise takes the first free
      slot up to QBConfig.Player.MaxInvSlots, and answers true, false (no room)
      or NOTHING for an unknown item after notifying the player. There is no
      CanAddItem, so invCanCarry computes weight and slots from the snapshot.
      RemoveItem without a slot takes from ONE stack that holds the whole amount
      and never spans stacks (player.lua:389-405), so invRemove removes slot by
      slot itself. A stack's info is '' (a string) when none was given.
    * qbr-inventory exposes NO server exports. Its stashes live in a local
      `Stashes` table that is filled from the `stashitems` row when a player
      opens the stash (server/main.lua:349-400) and written back when the
      player closes it (SaveStashItems, main.lua:90-104). Opening is a NET
      event the player's client must trigger (main.lua:349, uses `source`), so
      stOpen asks the player's poggy_core client to send it, the way every QBR
      script does (qbr-policejob client/job.lua:171-172). Register/items/add/
      remove/delete work on the `stashitems` row directly through oxmysql; a
      write made while a player has that stash open is overwritten when they
      close it. The label shown is always "Stash-<id>" (main.lua:374); the
      caller's label, slots and maxweight are kept in this adapter and passed
      at open time (main.lua:369-372). No resource on the server ships a
      CREATE TABLE for `stashitems`; init checks it exists and refuses storage
      honestly, with the statement to run, when it does not.
    * Weapons are ordinary inventory items with type = 'weapon' and a serial
      that AddItem stamps into info.serie when no info is given (player.lua:
      341-345). There is no separate loadout, so the weapon verbs here are a
      view over inventory items keyed by serial. Passing `{}` as info would
      suppress the serial, so weaponAdd passes nothing.
    * Item names are lower-case and QBShared.Items is keyed by name. qbr-core
      lower-cases the name it is given in AddItem (player.lua:334) but not in
      the shared lookup exports (shared/main.lua:59), and the Poggy scripts pass
      VORP's weapon spelling (WEAPON_REVOLVER_CATTLEMAN), so every item and
      weapon verb goes through itemName(), which tries the exact key and then
      the lower-cased one, and answers not_found for a name the table lacks.
    * Usable-item handlers live in qbr-core (QBCore.UseableItems,
      functions.lua:95-97) and are called as callback(source, item)
      (events.lua:99-105). The Poggy scripts expect the VORP shape, a table
      with .source, so the handler is wrapped. `inventoryResource` is
      therefore qbr-core: that is the resource whose restart loses handlers.
    * Permissions are ACE. exports['qbr-core']:HasPermission(src, level) is
      IsPlayerAceAllowed(src, level) (functions.lua:173-177); the levels are
      QBConfig.Permissions (config.lua:10: god, admin, mod) and a server grants
      them with `add_ace qbcore.<level> <level> allow` plus add_principal lines
      in server.cfg. The QBR test server's server.cfg has the principals but
      not those add_ace lines, so every player reads as 'user' until they exist.
    * Notifications: qbr-core's client Notify(id, text, duration, subtext, ...)
      takes a numeric style 1-9 (client/functions.lua:291-320), reached through
      the QBCore:Notify net event (client/events.lua:187). Used only when config
      selects the 'framework' renderer; poggy_core's native renderer is the default.
    * Item icons: qbr-inventory/html/images/<item.image>, referenced by the NUI
      as "images/" .. item.image (html/js/app.js:654). The file name is the
      item's `image` field, which is often NOT name .. '.png'
      (bread -> consumable_bread_roll.png).
]]

PoggyCore = PoggyCore or {}
PoggyCore.Adapters = PoggyCore.Adapters or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

--- Canonical currency name -> QBR money type. Everything else refuses. `gold`
--- is only honoured when the server's QBConfig.Money.MoneyTypes has it (init).
local CURRENCY = {
    cash = "cash",
    bank = "bank",
    gold = "gold",
}

--- The stashitems table qbr-inventory reads and writes but never creates.
local STASH_TABLE_SQL =
    "CREATE TABLE IF NOT EXISTS `stashitems` (`id` INT NOT NULL AUTO_INCREMENT, " ..
    "`stash` VARCHAR(255) NOT NULL, `items` LONGTEXT NULL, PRIMARY KEY (`id`), UNIQUE KEY `stash` (`stash`))"

local QBR = {
    id    = "qbr",
    label = "QBCore RedM",

    --- Usable handlers live in qbr-core (QBCore.UseableItems), so that is the
    --- resource whose restart loses them and after which sv_usables.lua
    --- replays. Stash definitions live in this adapter's own table, so
    --- sv_storage.lua's replay on poggy_core:ready is all storage needs.
    inventoryResource = "qbr-core",

    caps = {
        ["money.cash"]           = true,
        ["money.bank"]           = true,
        ["money.gold"]           = false,  -- true in init when QBConfig.Money.MoneyTypes has gold
        ["money.rol"]            = false,  -- no such currency on QBR
        ["char.onduty"]          = true,   -- PlayerData.job.onduty
        ["job.registry"]         = true,   -- QBShared.Jobs has labels and grades
        ["job.duty"]             = true,   -- Player.Functions.SetJobDuty
        ["job.event"]            = true,   -- no server event; sv_core.lua re-checks after jobSet and on a client poke
        ["inventory.items"]      = true,
        ["inventory.weapons"]    = true,   -- weapon-type items, keyed by serial
        ["inventory.metadata"]   = true,   -- item.info
        ["inventory.carrycheck"] = false,  -- no CanAddItem; weight and slots are computed here
        ["storage"]              = true,   -- false in init without oxmysql, qbr-inventory or the stashitems table
        ["storage.persist"]      = false,  -- contents persist (stashitems row); label/slots/maxweight live here
        ["storage.permissions"]  = false,  -- no job/char access model on a QBR stash
        ["storage.weapons"]      = true,
        ["menu.native"]          = false,  -- true in init when qbr-menu is started
        ["permissions"]          = true,
        ["char.offline"]         = true,   -- false in init without oxmysql
        ["job.persist"]          = true,   -- false in init without oxmysql
        ["inventory.registry"]   = true,   -- QBShared.Items through GetItems(), no database needed
    },
}

-- ---------------------------------------------------------------------------
-- Lifecycle
-- ---------------------------------------------------------------------------

function QBR:init(core)
    self.core    = core
    self.qb      = exports["qbr-core"]
    self.stashes = {}     -- id -> { label, slots, maxWeight }; see the storage section

    local ok, list = pcall(function() return self.qb:GetPlayers() end)
    if not ok or type(list) ~= "table" then
        return false, "qbr-core did not answer GetPlayers()"
    end
    local okItems, items = pcall(function() return self.qb:GetItems() end)
    if not okItems or type(items) ~= "table" then
        return false, "qbr-core did not answer GetItems()"
    end

    -- Money types are whatever the server configured; gold is declared only
    -- when it is really there, so money.gold never lies.
    local cfg = self:config()
    local types = type(cfg.Money) == "table" and cfg.Money.MoneyTypes or nil
    self.caps["money.gold"] = type(types) == "table" and types.gold ~= nil
    if type(types) == "table" then
        self.caps["money.cash"] = types.cash ~= nil
        self.caps["money.bank"] = types.bank ~= nil
    end

    self.caps["menu.native"] = GetResourceState("qbr-menu") == "started"

    if GetResourceState("qbr-inventory") ~= "started" then
        Util.Warn("qbr-inventory is not started. Storage calls will refuse; items still work through qbr-core.")
        self.caps["storage"] = false
    end

    if not Util.DbAvailable() then
        Util.Warn("oxmysql is not started. char.offline, char.list, job.set persist and storage will refuse.")
        self.caps["char.offline"] = false
        self.caps["job.persist"]  = false
        self.caps["storage"]      = false
    elseif self.caps["storage"] and Util.CanYield() then
        -- qbr-inventory reads and writes `stashitems` but nothing on a QBR
        -- server creates it. Say so now rather than at the first storage call.
        local rows = Util.DbQuery(
            "SELECT COUNT(*) AS n FROM information_schema.tables WHERE table_schema = DATABASE() AND table_name = 'stashitems'")
        local n = rows and rows[1] and tonumber(rows[1].n) or nil
        if n == 0 then
            Util.Warn("the `stashitems` table qbr-inventory uses does not exist, so storage will refuse. Create it with:")
            Util.Warn("  %s", STASH_TABLE_SQL)
            self.caps["storage"] = false
        end
    end

    return true
end

-- ---------------------------------------------------------------------------
-- Identity
-- ---------------------------------------------------------------------------

--- The QBR player object for a source: a fresh snapshot every call.
---@return table|nil player
---@return table|nil playerData
function QBR:raw(src)
    local n = tonumber(src)
    if not n then return nil, nil end
    local ok, player = pcall(function() return self.qb:GetPlayer(n) end)
    if not ok or type(player) ~= "table" or type(player.PlayerData) ~= "table" then return nil, nil end
    return player, player.PlayerData
end

--- QBConfig as GetConfig() hands it over (config.lua:130). Money types,
--- carry limits and permission levels are read from here; all plain data.
function QBR:config()
    local ok, cfg = pcall(function() return self.qb:GetConfig() end)
    if ok and type(cfg) == "table" then
        self.cfgCache = cfg
        return cfg
    end
    return self.cfgCache or {}
end

--- A fresh copy of QBShared.Items (shared/main.lua:65). Items added later
--- through AddItems would be missing from any snapshot, so the registry is
--- read again on each (cached, sv_core.lua) request.
function QBR:sharedItems()
    local ok, items = pcall(function() return self.qb:GetItems() end)
    if ok and type(items) == "table" then return items end
    return {}
end

local function fullName(first, last)
    return ((first or "") .. " " .. (last or "")):gsub("^%s+", "")
end

function QBR:getChar(src)
    local player, pd = self:raw(src)
    if not pd then return nil end

    local info  = type(pd.charinfo) == "table" and pd.charinfo or {}
    local job   = type(pd.job) == "table" and pd.job or {}
    local grade = type(job.grade) == "table" and job.grade or {}
    local money = type(pd.money) == "table" and pd.money or {}

    return {
        charId        = tostring(pd.citizenid or ""),
        ownerId       = tostring(pd.license or ""),
        source        = tonumber(src),
        firstName     = info.firstname or "",
        lastName      = info.lastname or "",
        fullName      = fullName(info.firstname, info.lastname),
        job           = job.name or "unemployed",
        jobLabel      = job.label or job.name or "",
        jobGrade      = tonumber(grade.level) or 0,
        jobGradeLabel = grade.name,
        onDuty        = job.onduty == true,
        group         = self:permGroup(src),
        gender        = PoggyCore.NormaliseGender(info.gender),
        money         = { cash = money.cash, bank = money.bank, gold = money.gold },
        native        = player,
    }
end

function QBR:getCharByCharId(charId)
    if charId == nil then return nil end
    local ok, player = pcall(function() return self.qb:GetPlayerByCitizenId(tostring(charId)) end)
    if not ok or type(player) ~= "table" or type(player.PlayerData) ~= "table" then return nil end
    local src = player.PlayerData.source
    if not src then return nil end
    return self:getChar(src)
end

function QBR:getPlayers()
    -- qbr-core's list holds only players with a loaded character
    -- (functions.lua:7-13), which is what every caller of Core.GetPlayers wants.
    local ok, list = pcall(function() return self.qb:GetPlayers() end)
    local out = {}
    if ok and type(list) == "table" then
        for _, src in pairs(list) do out[#out + 1] = tonumber(src) end
        return out
    end
    for _, src in ipairs(GetPlayers()) do out[#out + 1] = tonumber(src) end
    return out
end

--- Duty is part of the job on QBR (PlayerData.job.onduty), so it is always
--- answerable: true, false, or nil only when there is no character.
function QBR:dutyOf(src)
    local _, pd = self:raw(src)
    if not pd or type(pd.job) ~= "table" then return nil end
    return pd.job.onduty == true
end

--- JSON columns come back as strings from oxmysql; decode defensively.
local function decodeJson(v)
    if type(v) == "table" then return v end
    if type(v) ~= "string" or v == "" then return {} end
    local ok, t = pcall(json.decode, v)
    return (ok and type(t) == "table") and t or {}
end

--- A character by citizenid, online or not.
---
--- Online characters come back live. Offline ones are read from the `players`
--- table (qbcore.sql:1-19), whose charinfo and job columns are JSON. Appearance
--- lives in qbr-clothing's `playerskins` table (model, skin, clothes, active;
--- qbr-clothing server/sv_main.lua:18), so asking for it costs one more query.
function QBR:charOffline(charId, withAppearance)
    local id = charId ~= nil and tostring(charId) or nil
    if not id or id == "" then return nil, Err.BAD_ARG end

    local live = self:getCharByCharId(id)
    if live and not withAppearance then
        live.online = true
        return live
    end

    local out = live
    if not out then
        local rows, err = Util.DbQuery(
            "SELECT citizenid, license, charinfo, job FROM players WHERE citizenid = ? LIMIT 1", { id })
        if not rows then return nil, err end
        local r = rows[1]
        if not r then return nil, Err.NOT_FOUND end

        local info  = decodeJson(r.charinfo)
        local job   = decodeJson(r.job)
        local grade = type(job.grade) == "table" and job.grade or {}
        out = {
            charId        = tostring(r.citizenid),
            ownerId       = tostring(r.license or ""),
            source        = nil,
            firstName     = info.firstname or "",
            lastName      = info.lastname or "",
            fullName      = fullName(info.firstname, info.lastname),
            job           = job.name or "unemployed",
            jobLabel      = job.label or job.name or "",
            jobGrade      = tonumber(grade.level) or 0,
            jobGradeLabel = grade.name,
            onDuty        = nil,
            group         = "user",   -- ACE is per connection; nothing to read for an offline player
            gender        = PoggyCore.NormaliseGender(info.gender),
            native        = r,
        }
    end
    out.online = live ~= nil

    if withAppearance then
        local rows, err = Util.DbQuery(
            "SELECT model, skin, clothes FROM playerskins WHERE citizenid = ? AND active = 1 LIMIT 1", { id })
        if not rows then return nil, err end
        local r = rows[1]
        -- Two JSON blobs plus the ped model, decoded; the shape follows the data
        -- the way RSG's { skin, clothes } does.
        out.appearance = r and { model = r.model, skin = decodeJson(r.skin), clothes = decodeJson(r.clothes) } or nil
    end
    return out
end

--- Every character in the players table, sorted by name. Names are inside the
--- charinfo JSON column, so they are extracted in SQL (MariaDB 10.2+ / MySQL 5.7+).
function QBR:charList(opts)
    local first = "JSON_UNQUOTE(JSON_EXTRACT(charinfo, '$.firstname'))"
    local last  = "JSON_UNQUOTE(JSON_EXTRACT(charinfo, '$.lastname'))"
    local sql = ("SELECT citizenid, license, %s AS firstname, %s AS lastname FROM players"):format(first, last)
    local params = {}
    if type(opts.search) == "string" and opts.search ~= "" then
        sql = sql .. (" WHERE CONCAT(%s, ' ', %s) LIKE ?"):format(first, last)
        params[1] = "%" .. opts.search .. "%"
    end
    sql = sql .. " ORDER BY firstname, lastname"
    local limit, offset = tonumber(opts.limit), tonumber(opts.offset)
    if limit then
        -- Numbers written into the SQL, never bound: LIMIT cannot take a bound
        -- string, and flooring them is what keeps this safe.
        sql = sql .. (" LIMIT %d"):format(math.max(0, math.floor(limit)))
        if offset then sql = sql .. (" OFFSET %d"):format(math.max(0, math.floor(offset))) end
    end

    local rows, err = Util.DbQuery(sql, params)
    if not rows then return nil, err end
    local out = {}
    for _, r in ipairs(rows) do
        out[#out + 1] = {
            charId    = tostring(r.citizenid),
            ownerId   = tostring(r.license or ""),
            firstName = r.firstname or "",
            lastName  = r.lastname or "",
            fullName  = fullName(r.firstname, r.lastname),
        }
    end
    return out
end

-- ---------------------------------------------------------------------------
-- Money
-- ---------------------------------------------------------------------------

--- The money type for a canonical currency, or nil when this server has no
--- such account. money.cash / money.bank / money.gold in caps say the same.
function QBR:moneyType(currency)
    local mtype = CURRENCY[currency]
    if not mtype or not self.caps["money." .. currency] then return nil end
    return mtype
end

function QBR:moneyGet(src, currency)
    local mtype = self:moneyType(currency)
    if not mtype then return nil, Err.UNSUPPORTED end
    local _, pd = self:raw(src)
    if not pd then return nil, Err.NO_CHAR end
    local v = type(pd.money) == "table" and pd.money[mtype] or nil
    if v == nil then return nil, Err.UNSUPPORTED end
    return tonumber(v) or 0
end

--- AddMoney/RemoveMoney/SetMoney answer true, false, or nothing at all for a
--- negative amount (player.lua:231). Util.Amount already refused negatives, so
--- anything but `true` here is the framework refusing.
local function moneyResult(ok, res)
    if not ok or res ~= true then return false, Err.FRAMEWORK_ERR end
    return true
end

function QBR:moneyAdd(src, currency, amount, reason)
    local mtype = self:moneyType(currency)
    if not mtype then return false, Err.UNSUPPORTED end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    if type(pd.money) ~= "table" or pd.money[mtype] == nil then return false, Err.UNSUPPORTED end
    return moneyResult(pcall(player.Functions.AddMoney, mtype, amount, reason or "poggy_core"))
end

function QBR:moneyRemove(src, currency, amount, reason)
    local mtype = self:moneyType(currency)
    if not mtype then return false, Err.UNSUPPORTED end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local balance = type(pd.money) == "table" and tonumber(pd.money[mtype]) or nil
    if balance == nil then return false, Err.UNSUPPORTED end

    -- QBR refuses a negative balance for cash only (DontAllowMinus) and lets
    -- the bank go under. Refuse every overdraw here instead, so `ok` means the
    -- same thing on every framework.
    if balance < amount then return false, Err.NO_FUNDS end

    return moneyResult(pcall(player.Functions.RemoveMoney, mtype, amount, reason or "poggy_core"))
end

function QBR:moneySet(src, currency, amount, reason)
    local mtype = self:moneyType(currency)
    if not mtype then return false, Err.UNSUPPORTED end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    if type(pd.money) ~= "table" or pd.money[mtype] == nil then return false, Err.UNSUPPORTED end
    return moneyResult(pcall(player.Functions.SetMoney, mtype, amount, reason or "poggy_core"))
end

-- ---------------------------------------------------------------------------
-- Jobs
-- ---------------------------------------------------------------------------

function QBR:jobSet(src, name, grade, label)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end

    -- SetJob does tostring(grade) and indexes the grades table by that string
    -- (player.lua:148-155): nil becomes "nil" and 2.0 becomes "2.0", neither
    -- of which is a grade key. Always hand it a whole number. A caller that
    -- names the job the player already has and no grade means "keep the
    -- grade", which is what VORP does with a nil grade.
    local before = type(pd.job) == "table" and pd.job or {}
    if grade == nil and before.name == name:lower() then
        grade = type(before.grade) == "table" and before.grade.level or 0
    end
    grade = math.floor(tonumber(grade) or 0)

    -- `label` is not a thing QBR lets a caller set: labels come from
    -- QBShared.Jobs (player.lua:152). It is accepted and ignored.
    local ok, res = pcall(player.Functions.SetJob, name, grade)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NOT_FOUND end   -- job not in QBShared.Jobs

    -- qbr-core fires no server event for this; tell the relay to look again,
    -- with the job as it was in case this player was never seen loading.
    if PoggyCore.QbrJobCheck then
        PoggyCore.QbrJobCheck(src, {
            name  = before.name,
            grade = tonumber(type(before.grade) == "table" and before.grade.level) or 0,
        })
    end
    return true
end

function QBR:jobSetDuty(src, onDuty)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local ok = pcall(player.Functions.SetJobDuty, onDuty and true or false)   -- returns nothing (player.lua:205)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Write the job to the players table.
---
--- QBR saves the whole row itself every UpdateInterval minutes and on drop
--- (player.lua:107-129, events.lua:10), so an in-memory SetJob normally reaches
--- the DB on its own. This writes the job column now, from the in-memory job
--- the set just produced, so anything reading the table sees it immediately.
function QBR:jobPersist(src)
    local _, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    if type(pd.job) ~= "table" or not pd.citizenid then return false, Err.FRAMEWORK_ERR end

    local affected, err = Util.DbExecute("UPDATE players SET job = ? WHERE citizenid = ?",
        { json.encode(pd.job), tostring(pd.citizenid) })
    if not affected then return false, err end
    return true
end

-- ---------------------------------------------------------------------------
-- Inventory
-- ---------------------------------------------------------------------------

--- Normalise one qbr-core item entry to PoggyItem. `info` is '' when the item
--- was created without any (player.lua:353), so only a table counts as meta.
local function normaliseItem(it)
    return {
        name   = it.name,
        amount = tonumber(it.amount) or 0,
        meta   = type(it.info) == "table" and it.info or nil,
        label  = it.label,
        weight = tonumber(it.weight),
        slot   = it.slot,
        type   = it.type,
        unique = it.unique == true,
        native = it,
    }
end

--- Does every key of `want` match in `info`? Used to honour a `meta` argument
--- the way VORP does, since qbr-core itself matches items by name only.
local function metaMatches(info, want)
    if type(want) ~= "table" or next(want) == nil then return true end
    if type(info) ~= "table" then return false end
    for k, v in pairs(want) do
        if info[k] ~= v then return false end
    end
    return true
end

local function sameName(a, b)
    return type(a) == "string" and type(b) == "string" and a:lower() == b:lower()
end

--- The QBShared.Items key for a name a script passed, and its definition.
---
--- QBR item names are lower-case (`weapon_revolver_cattleman`); the Poggy
--- scripts were written against VORP, where weapons are named the way the game
--- hashes them (`WEAPON_REVOLVER_CATTLEMAN`). GetItem (shared/main.lua:59)
--- indexes the table verbatim. The exact key wins when it exists; otherwise the
--- lower-cased one. Returns nil, nil for a name the table does not know at all.
---@param name any
---@return string|nil key
---@return table|nil def
function QBR:itemName(name)
    if name == nil then return nil, nil end
    local key = tostring(name)
    local ok, def = pcall(function() return self.qb:GetItem(key) end)
    if not ok or type(def) ~= "table" then
        key = key:lower()
        ok, def = pcall(function() return self.qb:GetItem(key) end)
    end
    if not ok or type(def) ~= "table" then return nil, nil end
    return key, def
end

--- Tell the player's screen about an add or a remove, the way every QBR script
--- does (qbr-inventory server/main.lua:848). Cosmetic; never fails a call.
function QBR:itemBox(src, item, action, qty)
    local _, info = self:itemName(item)
    if type(info) ~= "table" then return end
    TriggerClientEvent("inventory:client:ItemBox", src, info, action, qty)
end

--- What AddItem would check (player.lua:346-366), computed from the snapshot:
--- the total weight against MaxWeight, then a stack to join (type 'item',
--- not unique) or one free slot below MaxInvSlots.
---@return boolean ok
---@return string|nil err
function QBR:room(pd, def, qty)
    local cfg    = self:config()
    local player = type(cfg.Player) == "table" and cfg.Player or {}
    local maxWeight = tonumber(player.MaxWeight) or 120000
    local maxSlots  = tonumber(player.MaxInvSlots) or 41
    local items  = type(pd.items) == "table" and pd.items or {}

    local weight, used, hasStack = 0, {}, false
    for k, it in pairs(items) do
        if type(it) == "table" then
            weight = weight + (tonumber(it.weight) or 0) * (tonumber(it.amount) or 0)
            local slot = tonumber(it.slot) or tonumber(k)
            if slot then used[slot] = true end
            if sameName(it.name, def.name) then hasStack = true end
        end
    end
    if weight + (tonumber(def.weight) or 0) * qty > maxWeight then return false, Err.NO_SPACE end

    local stacks = def.type == "item" and def.unique ~= true
    if stacks and hasStack then return true end
    for i = 1, maxSlots do
        if not used[i] then return true end
    end
    return false, Err.NO_SPACE
end

function QBR:invCanCarry(src, item, qty)
    local key, def = self:itemName(item)
    if not key then return false, Err.NOT_FOUND end
    local _, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    return self:room(pd, def, qty)
end

function QBR:invAdd(src, item, qty, meta)
    local key, def = self:itemName(item)
    if not key then return false, Err.NOT_FOUND end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local can, cerr = self:room(pd, def, qty)
    if not can then return false, cerr end

    -- `false` for the slot means "first free slot" (tonumber(false) is nil,
    -- player.lua:340). `meta` is last and may be nil: qbr-core then stores ''
    -- for an item, and stamps a serial for a weapon (player.lua:341-345),
    -- which an empty table would suppress.
    local ok, res = pcall(player.Functions.AddItem, key, qty, false, meta)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res == nil then return false, Err.FRAMEWORK_ERR end   -- unknown item; cannot happen after itemName
    if res ~= true then return false, Err.NO_SPACE end
    self:itemBox(src, key, "add", qty)
    return true
end

--- Remove `qty` of `item`, stack by stack. qbr-core's RemoveItem without a
--- slot only ever takes from one stack that holds the whole amount
--- (player.lua:389-405), so the slots are walked here, largest first.
function QBR:invRemove(src, item, qty, meta)
    local key = self:itemName(item) or tostring(item)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end

    local stacks = {}
    local total = 0
    for k, it in pairs(type(pd.items) == "table" and pd.items or {}) do
        if type(it) == "table" and sameName(it.name, key) and metaMatches(it.info, meta) then
            local slot = tonumber(it.slot) or tonumber(k)
            local amount = tonumber(it.amount) or 0
            if slot and amount > 0 then
                stacks[#stacks + 1] = { slot = slot, amount = amount, name = it.name }
                total = total + amount
            end
        end
    end
    if total < qty then return false, Err.NOT_FOUND end
    table.sort(stacks, function(x, y) return x.amount > y.amount end)

    local left = qty
    for _, st in ipairs(stacks) do
        if left <= 0 then break end
        local take = math.min(left, st.amount)
        local ok, res = pcall(player.Functions.RemoveItem, st.name, take, st.slot)
        if not ok or res ~= true then return false, Err.FRAMEWORK_ERR end   -- partial: earlier stacks are gone
        left = left - take
    end
    self:itemBox(src, key, "remove", qty)
    return true
end

function QBR:invCount(src, item, meta)
    local key = self:itemName(item) or tostring(item)
    local _, pd = self:raw(src)
    if not pd or type(pd.items) ~= "table" then return 0 end
    local n = 0
    for _, it in pairs(pd.items) do
        if type(it) == "table" and sameName(it.name, key) and metaMatches(it.info, meta) then
            n = n + (tonumber(it.amount) or 0)
        end
    end
    return n
end

function QBR:invGet(src)
    local _, pd = self:raw(src)
    if not pd or type(pd.items) ~= "table" then return {} end
    local out = {}
    for _, it in pairs(pd.items) do
        if type(it) == "table" and it.name then out[#out + 1] = normaliseItem(it) end
    end
    table.sort(out, function(x, y) return (tonumber(x.slot) or 0) < (tonumber(y.slot) or 0) end)
    return out
end

--- itemId is the SLOT on QBR (the only per-stack id it has). `meta` is merged
--- over the stack's info; `amount` is accepted for the interface and ignored,
--- because a stack cannot be split by metadata here. The stack is written back
--- through SetInventory(data, slot) (player.lua:410-412), which sets the slot
--- without telling the client, so UpdatePlayerItems(slot) (player.lua:136)
--- follows it.
function QBR:invSetMeta(src, itemId, meta, amount)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local slot = tonumber(itemId)
    if not slot or type(pd.items) ~= "table" then return false, Err.BAD_ARG end

    local it
    for k, v in pairs(pd.items) do
        if type(v) == "table" and (tonumber(v.slot) == slot or tonumber(k) == slot) then it = v; break end
    end
    if type(it) ~= "table" then return false, Err.NOT_FOUND end

    local info = type(it.info) == "table" and it.info or {}
    for k, v in pairs(meta or {}) do info[k] = v end
    it.info = info
    it.slot = slot

    local ok = pcall(player.Functions.SetInventory, it, slot)
    if not ok then return false, Err.FRAMEWORK_ERR end
    pcall(player.Functions.UpdatePlayerItems, slot)
    return true
end

function QBR:invRegisterUsable(item, fn, resource)
    -- qbr-core calls the handler as callback(source, item) (events.lua:102),
    -- keyed by item.name, which is the registry's lower-case key. The Poggy
    -- scripts, written against VORP, read data.source from a single table
    -- argument; hand them that shape, with the QBR item alongside, so one
    -- handler works on every framework.
    local key = self:itemName(item) or tostring(item):lower()
    local wrapped = function(a, b)
        local src, data
        if type(a) == "number" or tonumber(a) then
            src, data = tonumber(a), b
        elseif type(a) == "table" then
            src, data = tonumber(a.source), a
        end
        data = type(data) == "table" and data or {}
        fn({
            source   = src,
            name     = data.name or key,
            label    = data.label,
            amount   = data.amount,
            slot     = data.slot,
            metadata = type(data.info) == "table" and data.info or nil,
            item     = data,
            resource = resource,
        })
    end
    -- pcall: with qbr-core stopped the export proxy throws, and the registry
    -- must keep the entry for when it is back.
    local ok = pcall(function() self.qb:CreateUseableItem(key, wrapped) end)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Drop a handler. qbr-core has no remover; CreateUseableItem with no callback
--- stores nil, which is the same thing (functions.lua:95-97).
function QBR:invUnregisterUsable(item)
    local key = self:itemName(item) or tostring(item):lower()
    local ok = pcall(function() self.qb:CreateUseableItem(key, nil) end)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Is qbr-core answering? Handlers live there, so that is what a replay
--- needs. GetItem is a pure read and returns nil for a name it does not know.
function QBR:invReady()
    if GetResourceState(self.inventoryResource) ~= "started" then return false end
    local ok = pcall(function() return self.qb:GetItem("poggy_core_probe") end)
    return ok
end

--- Every item in QBShared.Items. Weapons ARE items on QBR (type 'weapon'), so
--- they are included; a caller that wants items only filters on `type`.
function QBR:itemRegistry()
    local items = self:sharedItems()
    local base = self:imageBase()
    local out = {}
    for name, it in pairs(items) do
        if type(it) == "table" then
            out[#out + 1] = {
                name   = it.name or name,
                label  = it.label or it.name or name,
                desc   = it.description,
                weight = tonumber(it.weight),
                limit  = nil,                       -- QBR has no per-item stack limit
                type   = it.type,
                usable = it.useable == true,
                group  = nil,                       -- QBR items carry no category
                image  = base .. (it.image or ((it.name or name) .. ".png")),
            }
        end
    end
    table.sort(out, function(x, y) return tostring(x.name) < tostring(y.name) end)
    return out
end

function QBR:imageBase()
    return PoggyCore.Frameworks.qbr.itemImageBase
end

--- Does qbr-inventory ship an icon for this item? The file name is the item's
--- `image` field, which is often not name .. '.png'. Reads the file, so
--- callers cache it.
function QBR:itemImageExists(name)
    local key, it = self:itemName(name)
    local file = (type(it) == "table" and it.image) or (tostring(key or name) .. ".png")
    return LoadResourceFile("qbr-inventory", "html/images/" .. file) ~= nil
end

--- Close the player's own inventory screen. qbr-inventory has no server
--- export or event for it; its client registers the `closeinv` command
--- (client/main.lua:243), which poggy_core's client runs on request.
function QBR:invClose(src)
    if GetResourceState("qbr-inventory") ~= "started" then return false, Err.UNSUPPORTED end
    TriggerClientEvent("poggy_core:qbr:closeInventory", src)
    return true
end

-- ---------------------------------------------------------------------------
-- Weapons — a view over inventory items with type = 'weapon'
-- ---------------------------------------------------------------------------

--- Normalise a weapon-type item. `id` is the serial qbr-core stamps into
--- info.serie on creation (player.lua:343), or the slot when there is none.
local function normaliseWeapon(it)
    local info = type(it.info) == "table" and it.info or {}
    return {
        id     = info.serie or it.slot,
        name   = it.name,
        label  = it.label or it.name,
        serial = info.serie,
        desc   = it.description,
        slot   = it.slot,
        meta   = info,
        native = it,
    }
end

local function isWeaponItem(it)
    return type(it) == "table" and it.type == "weapon"
end

function QBR:weaponsGet(src)
    local _, pd = self:raw(src)
    if not pd or type(pd.items) ~= "table" then return {} end
    local out = {}
    for _, it in pairs(pd.items) do
        if isWeaponItem(it) then out[#out + 1] = normaliseWeapon(it) end
    end
    return out
end

--- Give a weapon. `ammo` and `components` are accepted for the interface and
--- NOT applied: on QBR ammunition is a separate item the player loads and
--- components are qbr-weapons' business. Neither has a server export.
---
--- The name is the VORP spelling the scripts use (`WEAPON_REVOLVER_CATTLEMAN`)
--- or the QBR one; itemName() maps it. not_found when the item list has no
--- such weapon (or the name is an item that is not a weapon), never a silent
--- false. No info is passed, so qbr-core stamps the serial itself.
function QBR:weaponAdd(src, weaponName, ammo, components)
    local key, def = self:itemName(weaponName)
    if not key or not isWeaponItem(def) then return false, Err.NOT_FOUND end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local can, cerr = self:room(pd, def, 1)
    if not can then return false, cerr or Err.NO_SPACE end
    local ok, res = pcall(player.Functions.AddItem, key, 1, false)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NO_SPACE end
    self:itemBox(src, key, "add", 1)
    return true
end

--- Remove one weapon by the id weaponsGet returned: its serial, or its slot.
function QBR:weaponRemove(src, weaponId)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    if weaponId == nil then return false, Err.BAD_ARG end

    local found, foundSlot
    for k, it in pairs(type(pd.items) == "table" and pd.items or {}) do
        if isWeaponItem(it) then
            local serial = type(it.info) == "table" and it.info.serie or nil
            local slot = tonumber(it.slot) or tonumber(k)
            if (serial ~= nil and tostring(serial) == tostring(weaponId))
                or (serial == nil and slot == tonumber(weaponId)) then
                found, foundSlot = it, slot
                break
            end
        end
    end
    if not found or not foundSlot then return false, Err.NOT_FOUND end

    local ok, res = pcall(player.Functions.RemoveItem, found.name, 1, foundSlot)
    if not ok or res ~= true then return false, Err.FRAMEWORK_ERR end
    self:itemBox(src, found.name, "remove", 1)
    return true
end

--- Room for `qty` more weapons? With a name, the real weight-and-slots check;
--- without one only slots can be counted, because the weight depends on which.
function QBR:weaponCanCarry(src, qty, weapon)
    if weapon then
        local key, def = self:itemName(weapon)
        if not key or not isWeaponItem(def) then return false, Err.NOT_FOUND end
        local _, pd = self:raw(src)
        if not pd then return false, Err.NO_CHAR end
        return self:room(pd, def, qty)
    end
    local _, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local cfg = self:config()
    local maxSlots = type(cfg.Player) == "table" and tonumber(cfg.Player.MaxInvSlots) or 41
    local used = {}
    for k, it in pairs(type(pd.items) == "table" and pd.items or {}) do
        local slot = type(it) == "table" and (tonumber(it.slot) or tonumber(k)) or nil
        if slot then used[slot] = true end
    end
    local free = 0
    for i = 1, maxSlots do
        if not used[i] then free = free + 1 end
    end
    if free >= qty then return true end
    return false, Err.NO_SPACE
end

-- ---------------------------------------------------------------------------
-- Storage (qbr-inventory stashes, through the stashitems row)
-- ---------------------------------------------------------------------------
-- qbr-inventory has no export for any of this. A stash is a `stashitems` row
-- (stash, items JSON keyed by slot) that qbr-inventory loads into memory when a
-- player opens it and writes back when they close it. This adapter keeps the
-- definition (label, slots, maxWeight) itself, passes it at open time, and
-- reads and writes the row directly for everything else. Every read and write
-- below waits on oxmysql, so it needs a thread.

--- The stash row's items, keyed by slot, or nil, err. An absent row is an
--- empty stash (qbr-inventory treats it the same, server/main.lua:61-88).
function QBR:stRead(id)
    local rows, err = Util.DbQuery("SELECT items FROM stashitems WHERE stash = ? LIMIT 1", { id })
    if not rows then return nil, err end
    local items = {}
    local r = rows[1]
    if r then
        for _, it in pairs(decodeJson(r.items)) do
            local slot = type(it) == "table" and tonumber(it.slot) or nil
            if slot and it.name then items[slot] = it end
        end
    end
    return items
end

--- Write the row, the way qbr-inventory does (server/main.lua:96-99).
function QBR:stWrite(id, items)
    local list = {}
    for slot, it in pairs(items) do
        it.slot = slot
        it.description = nil   -- qbr-inventory strips it on save too
        list[#list + 1] = it
    end
    table.sort(list, function(x, y) return x.slot < y.slot end)
    local affected, err = Util.DbExecute(
        "INSERT INTO stashitems (stash, items) VALUES (?, ?) ON DUPLICATE KEY UPDATE items = VALUES(items)",
        { id, json.encode(list) })
    if not affected then return false, err end
    return true
end

--- A stash entry the way qbr-inventory builds one (server/main.lua:113-127).
local function stashEntry(def, amount, info, slot)
    return {
        name        = def.name,
        amount      = amount,
        info        = info or "",
        label       = def.label,
        description = def.description or "",
        weight      = def.weight,
        type        = def.type,
        unique      = def.unique,
        useable     = def.useable,
        image       = def.image,
        slot        = slot,
    }
end

function QBR:stIsRegistered(id)
    return self.stashes[id] ~= nil
end

function QBR:stRegister(id, opts)
    if not self.caps["storage"] then return false, Err.UNSUPPORTED end
    -- Nothing to create: the row appears on the first write or the first
    -- close. jobAccess, charAccess, shared, allowWeapons and whitelistItems
    -- have no counterpart on a QBR stash; storage.permissions is false.
    self.stashes[id] = {
        label     = opts.label or id,
        slots     = tonumber(opts.slots),
        maxWeight = tonumber(opts.maxWeight),
    }
    return true
end

function QBR:stOpen(src, id)
    if not self.caps["storage"] then return false, Err.UNSUPPORTED end
    local def = self.stashes[id]
    if not def then return false, Err.NOT_FOUND end
    -- qbr-inventory refuses with a notify while the player's own inventory is
    -- busy (server/main.lua:353); say so instead of opening nothing.
    local okState, busy = pcall(function() return Player(src).state.inv_busy end)
    if okState and busy then return false, Err.FRAMEWORK_ERR end
    -- The open is a net event that reads `source`, so the player's client
    -- sends it (client/cl_core.lua), with the definition as `other`.
    TriggerClientEvent("poggy_core:qbr:openStash", src, id, { slots = def.slots, maxweight = def.maxWeight })
    return true
end

function QBR:stClose(src, id)
    if not self.caps["storage"] then return false, Err.UNSUPPORTED end
    -- Closing the screen makes the NUI post CloseInventory, which saves the
    -- stash (client/main.lua:261-279). Same route as invClose.
    TriggerClientEvent("poggy_core:qbr:closeInventory", src)
    return true
end

function QBR:stAddItem(id, item, qty, meta, charId)
    -- charId is accepted for the interface; a QBR stash does not attribute items.
    if not self.caps["storage"] then return false, Err.UNSUPPORTED end
    local def = self.stashes[id]
    if not def then return false, Err.NOT_FOUND end
    local key, info = self:itemName(item)
    if not key then return false, Err.NOT_FOUND end

    local items, err = self:stRead(id)
    if not items then return false, err end

    -- The same limits qbr-inventory shows in its UI: the slots and maxweight
    -- the definition carries (the NUI refuses a drop past either).
    local weight, top = 0, 0
    for slot, it in pairs(items) do
        weight = weight + (tonumber(it.weight) or 0) * (tonumber(it.amount) or 0)
        if slot > top then top = slot end
    end
    if def.maxWeight and weight + (tonumber(info.weight) or 0) * qty > def.maxWeight then
        return false, Err.NO_SPACE
    end

    local stacks = info.type == "item" and info.unique ~= true
    local target
    if stacks then
        for slot, it in pairs(items) do
            if sameName(it.name, key) and metaMatches(type(it.info) == "table" and it.info or nil, meta) then
                target = slot
                break
            end
        end
    end
    if target then
        items[target].amount = (tonumber(items[target].amount) or 0) + qty
    else
        local limit = def.slots or math.max(top, 50)
        for i = 1, limit do
            if not items[i] then target = i; break end
        end
        if not target then return false, Err.NO_SPACE end
        items[target] = stashEntry(info, qty, meta, target)
    end
    return self:stWrite(id, items)
end

function QBR:stRemoveItem(id, item, qty, meta)
    if not self.caps["storage"] then return false, Err.UNSUPPORTED end
    if not self.stashes[id] then return false, Err.NOT_FOUND end
    local key = self:itemName(item) or tostring(item)

    local items, err = self:stRead(id)
    if not items then return false, err end

    local slots, total = {}, 0
    for slot, it in pairs(items) do
        if sameName(it.name, key) and metaMatches(type(it.info) == "table" and it.info or nil, meta) then
            slots[#slots + 1] = slot
            total = total + (tonumber(it.amount) or 0)
        end
    end
    if total < qty then return false, Err.NOT_FOUND end
    table.sort(slots, function(x, y) return (tonumber(items[x].amount) or 0) > (tonumber(items[y].amount) or 0) end)

    local left = qty
    for _, slot in ipairs(slots) do
        if left <= 0 then break end
        local have = tonumber(items[slot].amount) or 0
        local take = math.min(left, have)
        if take >= have then items[slot] = nil else items[slot].amount = have - take end
        left = left - take
    end
    return self:stWrite(id, items)
end

function QBR:stGetItems(id)
    if not self.caps["storage"] then return {}, Err.UNSUPPORTED end
    local items, err = self:stRead(id)
    if not items then return {}, err end
    local out = {}
    for _, it in pairs(items) do
        if type(it) == "table" and it.name then out[#out + 1] = normaliseItem(it) end
    end
    table.sort(out, function(x, y) return (tonumber(x.slot) or 0) < (tonumber(y.slot) or 0) end)
    return out
end

--- Weapons stored in a stash: its weapon-type items. not_found for an id that
--- was never registered, matching the VORP adapter.
function QBR:stGetWeapons(id)
    if not self.caps["storage"] then return nil, Err.UNSUPPORTED end
    if not self.stashes[id] then return nil, Err.NOT_FOUND end
    local items, err = self:stRead(id)
    if not items then return nil, err end
    local out = {}
    for _, it in pairs(items) do
        if isWeaponItem(it) then out[#out + 1] = normaliseWeapon(it) end
    end
    return out
end

function QBR:stSetCapacity(id, slots, maxWeight)
    if not slots and not maxWeight then return false, Err.BAD_ARG end
    local def = self.stashes[id]
    if not def then return false, Err.NOT_FOUND end
    if slots then def.slots = tonumber(slots) end
    if maxWeight then def.maxWeight = tonumber(maxWeight) end
    return true
end

--- Drop the definition without touching the contents. The row stays and the
--- next Register brings it back.
function QBR:stUnregister(id)
    self.stashes[id] = nil
    return true
end

--- Destroy the stash AND everything in it. Not reversible. A player who has
--- the stash open at that moment writes its contents back on close; there is
--- no export to evict them first.
function QBR:stDelete(id)
    if not self.caps["storage"] then return false, Err.UNSUPPORTED end
    if not self.stashes[id] then return false, Err.NOT_FOUND end
    local affected, err = Util.DbExecute("DELETE FROM stashitems WHERE stash = ?", { id })
    if not affected then return false, err end
    self.stashes[id] = nil
    return true
end

-- ---------------------------------------------------------------------------
-- Permissions
-- ---------------------------------------------------------------------------

--- The permission levels a server configured (config.lua:10), highest first.
--- QBConfig.Permissions is written highest first in the stock file; that
--- order is kept.
function QBR:permLevels()
    local cfg = self:config()
    local levels = cfg.Permissions
    if type(levels) ~= "table" or #levels == 0 then
        levels = { "god", "admin", "mod" }
    end
    return levels
end

--- Every level this player holds, highest first. The same test qbr-core's
--- HasPermission makes: IsPlayerAceAllowed(src, level) (functions.lua:173).
--- A QBR server grants these with `add_ace qbcore.<level> <level> allow` and
--- add_principal lines in server.cfg; a player with none of them is 'user'.
function QBR:permGroups(src)
    local out = {}
    local n = tonumber(src)
    if not n or n <= 0 then return out end
    for _, level in ipairs(self:permLevels()) do
        if IsPlayerAceAllowed(n, level) then out[#out + 1] = level end
    end
    return out
end

function QBR:permGroup(src)
    local groups = self:permGroups(src)
    return groups[1] or "user"
end

-- ---------------------------------------------------------------------------
-- Notifications
-- ---------------------------------------------------------------------------

--- qbr-core's own notification, through its QBCore:Notify net event
--- (client/events.lua:187 -> client/functions.lua:291): a numeric style, no
--- kind. 4 is ShowBasicTopNotification, 7 ShowTopNotification with a subtitle.
--- Used only when config selects the 'framework' renderer; the default is
--- poggy_core's native renderer, which draws the same natives itself.
function QBR:notify(src, text, kind, duration)
    if GetResourceState("qbr-core") ~= "started" then return false end
    TriggerClientEvent("QBCore:Notify", src, 4, text, duration)
    return true
end

function QBR:notifyRich(src, opts)
    if GetResourceState("qbr-core") ~= "started" then return false end
    if opts.title and opts.description then
        TriggerClientEvent("QBCore:Notify", src, 7, opts.title, opts.duration, opts.description)
    else
        TriggerClientEvent("QBCore:Notify", src, 4, opts.title or opts.description or "", opts.duration)
    end
    return true
end

-- ---------------------------------------------------------------------------
-- Job list (0.18.0)
-- ---------------------------------------------------------------------------

--- Every job in QBShared.Jobs through qbr-core's GetJobs() (shared/main.lua:69).
--- Keyed by name, no name field; grades keyed '0', '1' ... each { name, payment }.
function QBR:jobsList()
    local ok, jobs = pcall(function() return self.qb:GetJobs() end)
    if not ok or type(jobs) ~= "table" then return nil, Err.FRAMEWORK_ERR end
    local out = {}
    for key, job in pairs(jobs) do
        if type(job) == "table" then
            local grades = {}
            for g, def in pairs(type(job.grades) == "table" and job.grades or {}) do
                local n = tonumber(g)
                if n then
                    grades[#grades + 1] = { grade = n, label = type(def) == "table" and def.name or tostring(n) }
                end
            end
            table.sort(grades, function(a, b) return a.grade < b.grade end)
            local name = tostring(job.name or key)
            out[#out + 1] = { name = name, label = job.label or name, grades = grades }
        end
    end
    table.sort(out, function(a, b) return a.name < b.name end)
    return out
end

-- ---------------------------------------------------------------------------
-- Escape hatch
-- ---------------------------------------------------------------------------

--- There is no core object on QBR. What a script gets here is the export
--- proxy for qbr-core, so `Core.Native():GetPlayer(src)` works.
function QBR:nativeCore()
    return self.qb
end

PoggyCore.Adapters.qbr = QBR
