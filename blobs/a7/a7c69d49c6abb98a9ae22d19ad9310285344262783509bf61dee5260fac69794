--[[
    poggy_core — RSG Core adapter.   Written against rsg-core 2.3.13 / rsg-inventory 2.8.5.
    NOT YET VERIFIED IN GAME. Every call below was read from the installed source
    (file:line noted where it matters); none has run on a live RSG server.

    Framework traps this file exists to absorb, all confirmed in the installed source:

    * The core object is a COPY. exports['rsg-core']:GetCoreObject() returns the
      RSGCore table through a funcref call, so it is msgpack-serialised on the
      way over: data fields are a snapshot, functions become funcrefs. The same
      is true of every Player that RSGCore.Functions.GetPlayer hands back —
      Player.PlayerData is a snapshot taken at call time and Player.Functions.*
      are funcrefs that act on the real object inside rsg-core. Reading
      PlayerData is fine (every RSG script does it); WRITING to it does nothing.
      Every mutation here goes through a Player.Functions.* call or an
      rsg-inventory export, and every read fetches the player again.
    * Anything in rsg-core that yields (GetOfflinePlayerByCitizenId uses
      MySQL.prepare.await, player.lua:38) cannot be called through a funcref
      from another resource. Offline reads go to the `players` table through
      oxmysql instead, exactly as vorp.lua reads `characters`.
    * Money types are named, not numbered: PlayerData.money.cash/bank/gold plus
      valbank, rhobank, blkbank, armbank, bloodmoney (config.lua:9). Gold must be
      an integer (player.lua:328). RemoveMoney refuses to take cash, gold and
      bloodmoney negative but lets bank go to -5000 (config.lua:10-11); the
      no_funds guard here applies to every currency, so `ok` means the same
      thing on every framework.
    * Money items are ON by default (config.lua:14): cash is re-derived from
      `dollar` and `cent` inventory items in SynchronizeMoneyItems every time
      PlayerData is pushed (moneyitems.lua:260). AddMoney('cash') therefore
      ends in an AddItem of dollar items, and rsg-inventory drops an item that
      does not fit ON THE GROUND (ForceDropItem, exports.lua:656). So money.add
      for an item-backed currency checks CanAddItem first, and every money write
      is verified by reading the balance back, because AddMoney returns true
      before the item side has happened.
    * job.grade is a TABLE: { level, name, payment, isboss }. SetJob(job, grade)
      takes the grade as a number or a string key and resets it to 0 when
      omitted (player.lua:187-220); labels come from RSGShared.Jobs, never from
      the caller. It returns false for a job the shared table does not know.
    * RSGCore:Server:OnJobUpdate(source, job) fires for BOTH SetJob and
      SetJobDuty (player.lua:215, 259) and carries no old job, so sv_core.lua
      keeps the previous job per player and only relays a real change.
    * Stashes live in rsg-inventory's `Inventories` table in memory. The
      `inventories` DB rows are loaded at start with ITEMS ONLY (main.lua:11-35):
      label, slots and maxweight exist only after CreateInventory or
      OpenInventory sets them, and AddItem into a stash whose `slots` is nil
      errors inside GetFirstFreeSlot (functions.lua:57). Register therefore
      always calls CreateInventory, which is idempotent (exports.lua:895-911).
      AddItem/RemoveItem on a stash never write the DB; a player closing the
      stash does (events.lua:49-50), and so does SaveStash, which this file
      calls after each of its own writes.
    * A stash is single-occupancy: `isOpen` holds the opening source and a
      second opener is refused with a notify and no return value
      (exports.lua:520-523). OpenInventory also returns silently when the
      player's own inventory is busy (exports.lua:503).
    * Weapons are ordinary inventory items with type = 'weapon' and a serial in
      info.serie (exports.lua:717-731). There is no separate loadout, so the
      weapon verbs here are a view over inventory items keyed by serial.
    * Item names are lower-case and RSGShared.Items is indexed verbatim. The
      scripts pass VORP's weapon spelling (WEAPON_REVOLVER_CATTLEMAN), so every
      item and weapon verb goes through itemName(), which tries the exact key
      and then the lower-cased one, and answers not_found for a name the table
      lacks instead of rsg-inventory's bare false.
    * Usable-item handlers live in rsg-core (RSGCore.UsableItems,
      functions.lua:431) and are called as callback(source, itemData)
      (rsg-inventory events.lua:104). The Poggy scripts expect the VORP shape,
      a table with .source, so the handler is wrapped.
    * Permissions are ACE. RSGCore.Functions.HasPermission(src, 'admin') is
      IsPlayerAceAllowed(src, 'admin') (functions.lua:531-541); the group
      levels are RSGCore.Config.Server.Permissions (config.lua:104-111) and a
      server grants them with add_ace / add_principal in server.cfg.
    * Item icons: rsg-inventory/html/images/<item.image>, referenced by the NUI
      as "images/" .. item.image (app.js:1045). The file name comes from the
      item's `image` field, which usually but not always equals name .. '.png'.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Adapters = PoggyCore.Adapters or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

--- Canonical currency name -> RSG money type. Everything else refuses.
local CURRENCY = {
    cash = "cash",
    bank = "bank",
    gold = "gold",
}

local RSG = {
    id    = "rsg",
    label = "RSG Core",

    --- Stash definitions (label, slots, maxweight) live in rsg-inventory's
    --- memory, so sv_storage.lua replays every registration when it restarts.
    --- Usable handlers live in rsg-core, not here; replaying them after an
    --- rsg-inventory restart is harmless.
    inventoryResource = "rsg-inventory",

    caps = {
        ["money.cash"]           = true,
        ["money.bank"]           = true,
        ["money.gold"]           = true,   -- integer only; see moneyAdd
        ["money.rol"]            = false,  -- no such currency on RSG
        ["char.onduty"]          = true,   -- PlayerData.job.onduty
        ["job.registry"]         = true,   -- RSGShared.Jobs has labels and grades
        ["job.duty"]             = true,   -- Player.Functions.SetJobDuty
        ["job.event"]            = true,   -- RSGCore:Server:OnJobUpdate, relayed in sv_core.lua
        ["inventory.items"]      = true,
        ["inventory.weapons"]    = true,   -- weapon-type items, keyed by serial
        ["inventory.metadata"]   = true,   -- item.info
        ["inventory.carrycheck"] = true,   -- rsg-inventory CanAddItem
        ["storage"]              = true,
        ["storage.persist"]      = false,  -- contents persist; label/slots/maxweight do not
        ["storage.permissions"]  = false,  -- no job/char access model on an RSG stash
        ["storage.weapons"]      = true,
        ["menu.native"]          = false,  -- true in init when ox_lib is started
        ["permissions"]          = true,
        ["char.offline"]         = true,   -- false in init without oxmysql
        ["job.persist"]          = true,   -- false in init without oxmysql
        ["inventory.registry"]   = true,   -- RSGShared.Items, no database needed
    },
}

-- ---------------------------------------------------------------------------
-- Lifecycle
-- ---------------------------------------------------------------------------

function RSG:init(core)
    self.core = core
    if not core or type(core.Functions) ~= "table" or not PoggyCore.IsCallable(core.Functions.GetPlayer) then
        return false, "rsg-core returned an object without Functions.GetPlayer"
    end

    self.inv = exports["rsg-inventory"]
    if GetResourceState("rsg-inventory") ~= "started" then
        Util.Warn("rsg-inventory is not started. Item, weapon and storage calls will fail.")
        self.caps["inventory.items"]   = false
        self.caps["inventory.weapons"] = false
        self.caps["storage"]           = false
    end

    self.caps["menu.native"] = GetResourceState("ox_lib") == "started"

    if not Util.DbAvailable() then
        Util.Warn("oxmysql is not started. char.offline, char.list and job.set persist will refuse.")
        self.caps["char.offline"] = false
        self.caps["job.persist"]  = false
    end

    return true
end

-- ---------------------------------------------------------------------------
-- Identity
-- ---------------------------------------------------------------------------

--- The RSG player object for a source: a fresh snapshot every call.
---@return table|nil player
---@return table|nil playerData
function RSG:raw(src)
    local n = tonumber(src)
    if not n then return nil, nil end
    local ok, player = pcall(self.core.Functions.GetPlayer, n)
    if not ok or type(player) ~= "table" or type(player.PlayerData) ~= "table" then return nil, nil end
    return player, player.PlayerData
end

--- The shared config as the core object carries it. Permission levels and money
--- switches are read from here; both are plain data that a server sets at boot.
function RSG:config()
    return type(self.core.Config) == "table" and self.core.Config or {}
end

--- A fresh copy of RSGShared.Items. The core object we hold is a snapshot
--- taken at resolve, and items added later through AddItems would be missing
--- from it, so the registry is read again on each (cached) request.
function RSG:sharedItems()
    local ok, core = pcall(function() return exports["rsg-core"]:GetCoreObject() end)
    if ok and type(core) == "table" and type(core.Shared) == "table" and type(core.Shared.Items) == "table" then
        return core.Shared.Items
    end
    return (type(self.core.Shared) == "table" and self.core.Shared.Items) or {}
end

local function fullName(first, last)
    return ((first or "") .. " " .. (last or "")):gsub("^%s+", "")
end

function RSG:getChar(src)
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

function RSG:getCharByCharId(charId)
    if charId == nil then return nil end
    local ok, player = pcall(self.core.Functions.GetPlayerByCitizenId, tostring(charId))
    if not ok or type(player) ~= "table" or type(player.PlayerData) ~= "table" then return nil end
    local src = player.PlayerData.source
    if not src then return nil end
    return self:getChar(src)
end

function RSG:getPlayers()
    -- rsg-core's own list holds only players with a loaded character, which
    -- is what every caller of Core.GetPlayers wants.
    local ok, list = pcall(self.core.Functions.GetPlayers)
    local out = {}
    if ok and type(list) == "table" then
        for _, src in pairs(list) do out[#out + 1] = tonumber(src) end
        return out
    end
    for _, src in ipairs(GetPlayers()) do out[#out + 1] = tonumber(src) end
    return out
end

--- Duty is part of the job on RSG (PlayerData.job.onduty), so it is always
--- answerable: true, false, or nil only when there is no character.
function RSG:dutyOf(src)
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
--- table, whose charinfo and job columns are JSON. Appearance lives in
--- rsg-appearance's `playerskins` table (skin and clothes, both JSON), so
--- asking for it costs one more query.
function RSG:charOffline(charId, withAppearance)
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
        local rows, err = Util.DbQuery("SELECT skin, clothes FROM playerskins WHERE citizenid = ? LIMIT 1", { id })
        if not rows then return nil, err end
        local r = rows[1]
        -- The shape differs from VORP's { skin, comps, tints } because the
        -- data does: RSG keeps two JSON blobs. Both are handed over decoded.
        out.appearance = r and { skin = decodeJson(r.skin), clothes = decodeJson(r.clothes) } or nil
    end
    return out
end

--- Every character in the players table, sorted by name. Names are inside the
--- charinfo JSON column, so they are extracted in SQL (MariaDB 10.2+ / MySQL 5.7+).
function RSG:charList(opts)
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

--- Is this currency represented by inventory items on this server?
--- moneyitems.lua:234-241, read from the config the core object carries.
function RSG:moneyIsItem(currency)
    local cfg = self:config()
    if currency == "gold" then
        return type(cfg.Gold) == "table" and cfg.Gold.EnableGoldItems == true
    end
    return currency == "cash" and type(cfg.Money) == "table" and cfg.Money.EnableMoneyItems == true
end

--- The inventory items an item-backed currency turns into (moneyitems.lua:5-17),
--- and how many of each `amount` needs, so the add can be capacity-checked.
function RSG:moneyItemsFor(currency, amount)
    if currency == "gold" then
        local cfg = self:config()
        local unit = (type(cfg.Money) == "table" and cfg.Money.GoldItem) or "gold"
        return { { name = unit, qty = amount } }
    end
    local dollars, frac = math.modf(amount)
    local cents = math.floor(frac * 100 + 0.5)
    local out = {}
    if dollars > 0 then out[#out + 1] = { name = "dollar", qty = dollars } end
    if cents > 0 then out[#out + 1] = { name = "cent", qty = cents } end
    return out
end

function RSG:moneyGet(src, currency)
    local mtype = CURRENCY[currency]
    if not mtype then return nil, Err.UNSUPPORTED end
    local _, pd = self:raw(src)
    if not pd then return nil, Err.NO_CHAR end
    local v = type(pd.money) == "table" and pd.money[mtype] or nil
    -- A server can remove a type from RSGCore.Config.Money.MoneyTypes; the
    -- field is then absent and every AddMoney for it returns false.
    if v == nil then return nil, Err.UNSUPPORTED end
    return tonumber(v) or 0
end

function RSG:moneyAdd(src, currency, amount, reason)
    local mtype = CURRENCY[currency]
    if not mtype then return false, Err.UNSUPPORTED end
    if mtype == "gold" and amount % 1 ~= 0 then return false, Err.BAD_ARG end   -- player.lua:328
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local before = type(pd.money) == "table" and tonumber(pd.money[mtype]) or nil
    if before == nil then return false, Err.UNSUPPORTED end

    -- Item-backed money ends in AddItem, and an add that does not fit is
    -- dropped on the ground while AddMoney still answers true. Refuse first.
    if self:moneyIsItem(mtype) and self.caps["inventory.items"] then
        for _, it in ipairs(self:moneyItemsFor(mtype, amount)) do
            local can = self.inv:CanAddItem(src, it.name, it.qty)
            if not can then return false, Err.NO_SPACE end
        end
    end

    local ok, res = pcall(player.Functions.AddMoney, mtype, amount, reason or "poggy_core")
    if not ok or res ~= true then return false, Err.FRAMEWORK_ERR end
    return true
end

function RSG:moneyRemove(src, currency, amount, reason)
    local mtype = CURRENCY[currency]
    if not mtype then return false, Err.UNSUPPORTED end
    if mtype == "gold" and amount % 1 ~= 0 then return false, Err.BAD_ARG end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local balance = type(pd.money) == "table" and tonumber(pd.money[mtype]) or nil
    if balance == nil then return false, Err.UNSUPPORTED end

    -- RSG lets the bank go to -5000 and refuses the rest itself. Refuse every
    -- overdraw here instead, so `ok` means the same thing on every framework.
    if balance < amount then return false, Err.NO_FUNDS end

    local ok, res = pcall(player.Functions.RemoveMoney, mtype, amount, reason or "poggy_core")
    if not ok or res ~= true then return false, Err.FRAMEWORK_ERR end
    return true
end

function RSG:moneySet(src, currency, amount, reason)
    local mtype = CURRENCY[currency]
    if not mtype then return false, Err.UNSUPPORTED end
    if mtype == "gold" and amount % 1 ~= 0 then return false, Err.BAD_ARG end
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    if type(pd.money) ~= "table" or pd.money[mtype] == nil then return false, Err.UNSUPPORTED end

    local ok, res = pcall(player.Functions.SetMoney, mtype, amount, reason or "poggy_core")
    if not ok or res ~= true then return false, Err.FRAMEWORK_ERR end
    return true
end

-- ---------------------------------------------------------------------------
-- Jobs
-- ---------------------------------------------------------------------------

function RSG:jobSet(src, name, grade, label)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end

    -- SetJob resets the grade to 0 when none is given. A caller that names the
    -- job the player already has and no grade means "keep the grade", which is
    -- what VORP does with a nil grade, so read the current one back.
    if grade == nil and type(pd.job) == "table" and pd.job.name == name:lower() then
        grade = type(pd.job.grade) == "table" and pd.job.grade.level or 0
    end

    -- `label` is not a thing RSG lets a caller set: labels come from
    -- RSGShared.Jobs, and CheckPlayerData rewrites the job from that table on
    -- every login anyway. It is accepted and ignored.
    local ok, res = pcall(player.Functions.SetJob, name, tonumber(grade) or 0)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NOT_FOUND end   -- job not in RSGShared.Jobs
    return true
end

function RSG:jobSetDuty(src, onDuty)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local ok = pcall(player.Functions.SetJobDuty, onDuty and true or false)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Write the job to the players table.
---
--- RSG saves the whole row itself every UpdateInterval minutes and on drop
--- (player.lua:534-558), so an in-memory SetJob normally reaches the DB on its
--- own. This writes the job column now, from the in-memory job the set just
--- produced, so anything reading the table sees it immediately. grade and
--- label are already applied by jobSet; only the column is written here.
function RSG:jobPersist(src)
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

--- The export proxy throws when rsg-inventory is stopped; every call goes
--- through here so that becomes an error code rather than a crash.
function RSG:call(name, ...)
    local ok, a, b = pcall(function(...) return self.inv[name](self.inv, ...) end, ...)
    if not ok then
        Util.Debug("rsg-inventory %s threw: %s", name, tostring(a))
        return false, nil, Err.FRAMEWORK_ERR
    end
    return true, a, b
end

--- Normalise one rsg-inventory item entry to PoggyItem.
local function normaliseItem(it)
    return {
        name   = it.name,
        amount = tonumber(it.amount) or 0,
        meta   = type(it.info) == "table" and it.info or nil,
        label  = it.label,
        weight = tonumber(it.weight),
        slot   = it.slot,
        type   = it.type,
        native = it,
    }
end

--- Does every key of `want` match in `info`? Used to honour a `meta` argument
--- the way VORP does, since rsg-inventory itself matches items by name only.
local function metaMatches(info, want)
    if type(want) ~= "table" or next(want) == nil then return true end
    if type(info) ~= "table" then return false end
    for k, v in pairs(want) do
        if info[k] ~= v then return false end
    end
    return true
end

--- The RSGShared.Items key for a name a script passed, and its definition.
---
--- RSG item names are lower-case (`weapon_revolver_cattleman`); the Poggy
--- scripts were written against VORP, where weapons are named the way the
--- game hashes them (`WEAPON_REVOLVER_CATTLEMAN`). rsg-inventory indexes the
--- table verbatim, so the VORP spelling finds nothing and every export answers
--- a bare false. The exact key wins when it exists; otherwise the lower-cased
--- one. Returns nil, nil for a name the table does not know at all.
---@param name any
---@return string|nil key
---@return table|nil def
function RSG:itemName(name)
    if name == nil then return nil, nil end
    local items = self:sharedItems()
    local key = tostring(name)
    local def = items[key]
    if type(def) ~= "table" then
        key = key:lower()
        def = items[key]
    end
    if type(def) ~= "table" then return nil, nil end
    return key, def
end

--- Tell the player's screen about an add or a remove, the way every RSG
--- script does (rsg-inventory commands.lua:22). Cosmetic; never fails a call.
function RSG:itemBox(src, item, action, qty)
    local _, info = self:itemName(item)
    if type(info) ~= "table" then return end
    TriggerClientEvent("rsg-inventory:client:ItemBox", src, info, action, qty)
end

function RSG:invCanCarry(src, item, qty)
    -- An unknown name is answered here, honestly, rather than by CanAddItem's
    -- bare false (exports.lua:300), which cannot be told from "no room".
    local key = self:itemName(item)
    if not key then return false, Err.NOT_FOUND end
    local ok, can = self:call("CanAddItem", src, key, qty)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if can then return true end
    return false, Err.NO_SPACE
end

function RSG:invAdd(src, item, qty, meta)
    local key = self:itemName(item)
    if not key then return false, Err.NOT_FOUND end
    -- `false` for the slot, not nil: a nil in the middle of an export's
    -- argument list does not survive the trip, and AddItem treats false as
    -- "first free slot" (exports.lua:674). Every RSG script passes it this way.
    local ok, res = self:call("AddItem", src, key, qty, false, meta or {}, "poggy_core inv.add")
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NO_SPACE end   -- the only false paths are weight and slots
    self:itemBox(src, key, "add", qty)
    return true
end

function RSG:invRemove(src, item, qty, meta)
    item = self:itemName(item) or item
    local count = self:invCount(src, item, meta)
    if count < qty then return false, Err.NOT_FOUND end

    if type(meta) == "table" and next(meta) ~= nil then
        -- rsg-inventory removes by name across every stack; honouring `meta`
        -- means picking the matching stacks ourselves, slot by slot.
        local ok, stacks = self:call("GetItemsByName", src, item)
        if not ok or type(stacks) ~= "table" then return false, Err.FRAMEWORK_ERR end
        local left = qty
        for _, it in pairs(stacks) do
            if left <= 0 then break end
            if metaMatches(it.info, meta) then
                local take = math.min(left, tonumber(it.amount) or 0)
                if take > 0 then
                    local rok, res = self:call("RemoveItem", src, item, take, it.slot, "poggy_core inv.remove")
                    if not rok or res ~= true then return false, Err.FRAMEWORK_ERR end
                    left = left - take
                end
            end
        end
        if left > 0 then return false, Err.NOT_FOUND end
        self:itemBox(src, item, "remove", qty)
        return true
    end

    local ok, res = self:call("RemoveItem", src, item, qty, false, "poggy_core inv.remove")
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NOT_FOUND end
    self:itemBox(src, item, "remove", qty)
    return true
end

function RSG:invCount(src, item, meta)
    item = self:itemName(item) or item
    if type(meta) == "table" and next(meta) ~= nil then
        local ok, stacks = self:call("GetItemsByName", src, item)
        if not ok or type(stacks) ~= "table" then return 0 end
        local n = 0
        for _, it in pairs(stacks) do
            if metaMatches(it.info, meta) then n = n + (tonumber(it.amount) or 0) end
        end
        return n
    end
    local ok, count = self:call("GetItemCount", src, item)
    if not ok then return 0 end
    return tonumber(count) or 0
end

function RSG:invGet(src)
    local _, pd = self:raw(src)
    if not pd or type(pd.items) ~= "table" then return {} end
    local out = {}
    for _, it in pairs(pd.items) do
        if type(it) == "table" and it.name then out[#out + 1] = normaliseItem(it) end
    end
    table.sort(out, function(x, y) return (tonumber(x.slot) or 0) < (tonumber(y.slot) or 0) end)
    return out
end

--- itemId is the SLOT on RSG (the only per-stack id it has). `meta` is merged
--- over the stack's info; `amount` is accepted for the interface and ignored,
--- because a stack cannot be split by metadata here.
function RSG:invSetMeta(src, itemId, meta, amount)
    local player, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    local slot = tonumber(itemId)
    if not slot or type(pd.items) ~= "table" then return false, Err.BAD_ARG end

    local items = pd.items   -- our snapshot; handed back whole, as SetInventory does
    local it = items[slot]
    if type(it) ~= "table" then
        -- msgpack may hand a sparse array back with string keys; look again.
        for k, v in pairs(items) do
            if tonumber(k) == slot and type(v) == "table" then it = v; break end
        end
    end
    if type(it) ~= "table" then return false, Err.NOT_FOUND end

    local info = type(it.info) == "table" and it.info or {}
    for k, v in pairs(meta or {}) do info[k] = v end
    it.info = info

    local ok = pcall(player.Functions.SetPlayerData, "items", items)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

function RSG:invRegisterUsable(item, fn, resource)
    -- rsg-inventory calls the handler as callback(source, itemData)
    -- (events.lua:104). The Poggy scripts, written against VORP, read
    -- data.source from a single table argument; hand them that shape, with the
    -- RSG item data alongside, so one handler works on both frameworks.
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
            name     = data.name or item,
            label    = data.label,
            amount   = data.amount,
            slot     = data.slot,
            metadata = data.info,
            item     = data,
            resource = resource,
        })
    end
    -- pcall: with rsg-core stopped the funcref throws, and the registry must
    -- keep the entry for when it is back.
    local ok = pcall(self.core.Functions.CreateUseableItem, item, wrapped)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Drop a handler. rsg-core has no remover; CreateUseableItem with no data
--- stores nil, which is the same thing (functions.lua:431-433).
function RSG:invUnregisterUsable(item)
    local ok = pcall(self.core.Functions.CreateUseableItem, item, nil)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Are rsg-inventory's exports answering? GetItemWeight is a pure read of the
--- shared table and returns nil for a name it does not know.
function RSG:invReady()
    if GetResourceState(self.inventoryResource) ~= "started" then return false end
    local ok = pcall(function() return self.inv:GetItemWeight("poggy_core_probe") end)
    return ok
end

--- Every item in RSGShared.Items. Weapons ARE items on RSG (type 'weapon'), so
--- they are included; a caller that wants items only filters on `type`.
function RSG:itemRegistry()
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
                limit  = nil,                       -- RSG has no per-item stack limit
                type   = it.type,
                usable = it.useable == true,
                group  = it.category,
                image  = base .. (it.image or ((it.name or name) .. ".png")),
            }
        end
    end
    table.sort(out, function(x, y) return tostring(x.name) < tostring(y.name) end)
    return out
end

function RSG:imageBase()
    return PoggyCore.Frameworks.rsg.itemImageBase
end

--- Does rsg-inventory ship an icon for this item? The file name is the item's
--- `image` field, which is not always name .. '.png'. Reads the file, so
--- callers cache it.
function RSG:itemImageExists(name)
    local key, it = self:itemName(name)
    local file = (type(it) == "table" and it.image) or (tostring(key or name) .. ".png")
    return LoadResourceFile("rsg-inventory", "html/images/" .. file) ~= nil
end

--- Close the player's own inventory screen.
function RSG:invClose(src)
    local ok = self:call("CloseInventory", src)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

-- ---------------------------------------------------------------------------
-- Weapons — a view over inventory items with type = 'weapon'
-- ---------------------------------------------------------------------------

--- Normalise a weapon-type item. `id` is the serial rsg-inventory stamps into
--- info.serie on creation (exports.lua:718), or the slot when there is none.
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

function RSG:weaponsGet(src)
    local _, pd = self:raw(src)
    if not pd or type(pd.items) ~= "table" then return {} end
    local out = {}
    for _, it in pairs(pd.items) do
        if isWeaponItem(it) then out[#out + 1] = normaliseWeapon(it) end
    end
    return out
end

--- Give a weapon. `ammo` and `components` are accepted for the interface and
--- NOT applied: on RSG ammunition is a separate item the player loads, and
--- components are rsg-weaponcomp's business. Neither has a server export.
---
--- The name is the VORP spelling the scripts use (`WEAPON_REVOLVER_CATTLEMAN`)
--- or the RSG one; itemName() maps it. not_found when the item list has no
--- such weapon (or the name is an item that is not a weapon), never a silent
--- false: that is what made a shop purchase take the money and give nothing.
function RSG:weaponAdd(src, weaponName, ammo, components)
    local key, def = self:itemName(weaponName)
    if not key or not isWeaponItem(def) then return false, Err.NOT_FOUND end
    local can, cerr = self:invCanCarry(src, key, 1)
    if not can then return false, cerr or Err.NO_SPACE end
    local ok, res = self:call("AddItem", src, key, 1, false, {}, "poggy_core weapon.add")
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NO_SPACE end
    self:itemBox(src, key, "add", 1)
    return true
end

--- Remove one weapon by the id weaponsGet returned: its serial, or its slot.
function RSG:weaponRemove(src, weaponId)
    local _, pd = self:raw(src)
    if not pd then return false, Err.NO_CHAR end
    if weaponId == nil then return false, Err.BAD_ARG end

    local found
    for _, it in pairs(type(pd.items) == "table" and pd.items or {}) do
        if isWeaponItem(it) then
            local serial = type(it.info) == "table" and it.info.serie or nil
            if (serial ~= nil and tostring(serial) == tostring(weaponId))
                or (serial == nil and tonumber(it.slot) == tonumber(weaponId)) then
                found = it
                break
            end
        end
    end
    if not found then return false, Err.NOT_FOUND end

    local ok, res = self:call("RemoveItem", src, found.name, 1, found.slot, "poggy_core weapon.remove")
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.FRAMEWORK_ERR end
    self:itemBox(src, found.name, "remove", 1)
    return true
end

--- Room for `qty` more weapons? With a name, the real weight-and-slots check;
--- without one only slots can be counted, because the weight depends on which.
function RSG:weaponCanCarry(src, qty, weapon)
    if weapon then
        local key, def = self:itemName(weapon)
        if not key or not isWeaponItem(def) then return false, Err.NOT_FOUND end
        return self:invCanCarry(src, key, qty)
    end
    local ok, _, free = self:call("GetSlots", src)
    if not ok then return false, Err.FRAMEWORK_ERR end
    free = tonumber(free) or 0
    if free >= qty then return true end
    return false, Err.NO_SPACE
end

-- ---------------------------------------------------------------------------
-- Storage (rsg-inventory stashes)
-- ---------------------------------------------------------------------------

--- The in-memory stash, or nil. GetInventory returns nil for an unknown id.
function RSG:stash(id)
    local ok, inv = self:call("GetInventory", id)
    if not ok or type(inv) ~= "table" then return nil end
    return inv
end

function RSG:stIsRegistered(id)
    return self:stash(id) ~= nil
end

function RSG:stRegister(id, opts)
    -- Always CreateInventory: it creates a missing stash and, for one that
    -- exists (including one loaded from the DB with items only), sets the
    -- label, slots and maxweight that AddItem and the UI need. Idempotent.
    --
    -- jobAccess, charAccess, shared, allowWeapons and whitelistItems have no
    -- counterpart on an RSG stash: it is one flat container that anyone who
    -- reaches it can open. storage.permissions is false for that reason.
    local ok = self:call("CreateInventory", id, {
        label     = opts.label or id,
        slots     = opts.slots,
        maxweight = opts.maxWeight,
    })
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

function RSG:stOpen(src, id)
    -- OpenInventory returns nothing on either refusal, so check the two
    -- conditions it refuses on first (exports.lua:503 and 520).
    local okState, busy = pcall(function() return Player(src).state.inv_busy end)
    if okState and busy then return false, Err.FRAMEWORK_ERR end
    local inv = self:stash(id)
    if inv and inv.isOpen and tonumber(inv.isOpen) ~= tonumber(src) then
        return false, Err.FRAMEWORK_ERR   -- single occupancy; someone else has it open
    end
    local ok = self:call("OpenInventory", src, id)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

function RSG:stClose(src, id)
    local ok = self:call("CloseInventory", src, id)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Write the stash row now. AddItem/RemoveItem on a stash change memory only;
--- rsg-inventory writes the row when a player closes the stash, and not before.
function RSG:stSave(id)
    self:call("SaveStash", id)
end

function RSG:stAddItem(id, item, qty, meta, charId)
    -- charId is accepted for the interface; an RSG stash does not attribute items.
    if not self:stIsRegistered(id) then return false, Err.NOT_FOUND end
    local ok, can = self:call("CanAddItem", id, item, qty)
    if not ok then return false, Err.FRAMEWORK_ERR end
    if not can then
        if self:sharedItems()[item] == nil then return false, Err.NOT_FOUND end
        return false, Err.NO_SPACE
    end
    local aok, res = self:call("AddItem", id, item, qty, false, meta or {}, "poggy_core storage.addItem")
    if not aok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NO_SPACE end   -- out of slots (weight was checked above)
    self:stSave(id)
    return true
end

function RSG:stRemoveItem(id, item, qty, meta)
    if not self:stIsRegistered(id) then return false, Err.NOT_FOUND end
    local ok, res = self:call("RemoveItem", id, item, qty, false, "poggy_core storage.removeItem")
    if not ok then return false, Err.FRAMEWORK_ERR end
    if res ~= true then return false, Err.NOT_FOUND end
    self:stSave(id)
    return true
end

function RSG:stGetItems(id)
    local inv = self:stash(id)
    if not inv or type(inv.items) ~= "table" then return {} end
    local out = {}
    for _, it in pairs(inv.items) do
        if type(it) == "table" and it.name then out[#out + 1] = normaliseItem(it) end
    end
    table.sort(out, function(x, y) return (tonumber(x.slot) or 0) < (tonumber(y.slot) or 0) end)
    return out
end

--- Weapons stored in a stash: its weapon-type items. not_found for an id
--- rsg-inventory does not know, matching the VORP adapter.
function RSG:stGetWeapons(id)
    local inv = self:stash(id)
    if not inv then return nil, Err.NOT_FOUND end
    local out = {}
    for _, it in pairs(type(inv.items) == "table" and inv.items or {}) do
        if isWeaponItem(it) then out[#out + 1] = normaliseWeapon(it) end
    end
    return out
end

function RSG:stSetCapacity(id, slots, maxWeight)
    if not slots and not maxWeight then return false, Err.BAD_ARG end
    -- CreateInventory on an existing stash updates only the fields given.
    local ok = self:call("CreateInventory", id, { slots = slots, maxweight = maxWeight })
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Drop the definition without touching the contents.
---
--- DeleteInventory forgets the stash in memory only; the `inventories` row
--- stays and is loaded again at the next start, or by the next Register. The
--- row is written first so nothing added since the last close is lost.
function RSG:stUnregister(id)
    if not self:stIsRegistered(id) then return true end
    self:stSave(id)
    local ok = self:call("DeleteInventory", id)
    return ok
end

--- Destroy the stash AND everything in it. Not reversible.
---
--- ClearStash empties the items and writes the emptied row; DeleteInventory
--- drops the definition. The row itself is removed when oxmysql is available,
--- so a later start does not load an empty ghost.
function RSG:stDelete(id)
    if not self:stIsRegistered(id) then return false, Err.NOT_FOUND end
    local ok = self:call("ClearStash", id)
    if not ok then return false, Err.FRAMEWORK_ERR end
    self:call("DeleteInventory", id)
    if Util.DbAvailable() and Util.CanYield() then
        Util.DbExecute("DELETE FROM inventories WHERE identifier = ?", { id })
    end
    if self:stIsRegistered(id) then return false, Err.FRAMEWORK_ERR end
    return true
end

-- ---------------------------------------------------------------------------
-- Permissions
-- ---------------------------------------------------------------------------

--- The permission levels a server configured, highest first (config.lua:104).
function RSG:permLevels()
    local cfg = self:config()
    local levels = type(cfg.Server) == "table" and cfg.Server.Permissions or nil
    if type(levels) ~= "table" or #levels == 0 then
        levels = { "god", "developer", "headadmin", "admin", "mod", "helper" }
    end
    return levels
end

--- Every level this player holds, highest first. The same test rsg-core's
--- HasPermission makes: IsPlayerAceAllowed(src, level). An RSG server grants
--- these with `add_ace rsgcore.<level> <level> allow` and principals in
--- server.cfg; a player with none of them is 'user'.
function RSG:permGroups(src)
    local out = {}
    local n = tonumber(src)
    if not n or n <= 0 then return out end
    for _, level in ipairs(self:permLevels()) do
        if IsPlayerAceAllowed(n, level) then out[#out + 1] = level end
    end
    return out
end

function RSG:permGroup(src)
    local groups = self:permGroups(src)
    return groups[1] or "user"
end

-- ---------------------------------------------------------------------------
-- Notifications
-- ---------------------------------------------------------------------------

--- RSG has no server-side notify of its own; its scripts fire ox_lib's client
--- event (events.lua:170). Used only when config selects the 'framework'
--- renderer and ox_lib is running; otherwise false, and sv_notify moves on.
local OX_TYPE = { info = "inform", success = "success", error = "error", warning = "warning" }

function RSG:notify(src, text, kind, duration)
    if GetResourceState("ox_lib") ~= "started" then return false end
    TriggerClientEvent("ox_lib:notify", src, {
        title = text, type = OX_TYPE[kind] or "inform", duration = duration,
    })
    return true
end

function RSG:notifyRich(src, opts)
    if GetResourceState("ox_lib") ~= "started" then return false end
    TriggerClientEvent("ox_lib:notify", src, {
        title       = opts.title or opts.description or "",
        description = opts.title and opts.description or nil,
        type        = OX_TYPE[opts.kind] or "inform",
        duration    = opts.duration,
    })
    return true
end

-- ---------------------------------------------------------------------------
-- Escape hatch
-- ---------------------------------------------------------------------------

function RSG:nativeCore()
    return self.core
end

PoggyCore.Adapters.rsg = RSG
