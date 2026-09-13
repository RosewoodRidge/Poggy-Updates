--[[
    poggy_core — VORP adapter.   Verified against vorp_core 2.8 / vorp_inventory 4.0.

    Framework traps this file exists to absorb, all confirmed in the installed source:

    * The core object comes from exports.vorp_core:GetCore() and nowhere else.
      vorp_core also exposes vorpAPI(), which is deprecated and returns a
      ten-method object with NO getUser. Asking for that one first is the bug in
      poggy_util today.
    * user.getUsedCharacter is a VALUE, not a function. Calling it errors.
    * Currencies are indexed by number: 0 money, 1 gold, 2 rol.

    * There IS a bank on a VORP server, but not in vorp_core. It lives in the
      separate `vorp_banking` resource, which registers no exports at all and
      only proximity-gated events, so the single route to a balance is its
      `bank_users` table. A character holds one row per branch (Valentine,
      Blackwater, Saint Denis, Rhodes), which means "the bank balance" is a sum
      and "add to the bank" is meaningless until a branch is named.

      `money.bank` is therefore false, and Core.Money refuses it. That is the
      honest answer for poggy_core today: none of the Poggy scripts bank, so
      reaching into another resource's money table would be risk with no
      return. If a script ever needs it, read this paragraph first.
    * getItemCount takes its callback as the SECOND argument, not the last.
    * Callback position varies per function and is NOT always last. Every call
      in this file has been checked against inventoryApiService.lua one by one;
      do the same for any new one. Getting it wrong does not error — the
      callback lands in a data parameter and your resolve is simply never
      called, so the call times out and the query runs against a function.
    * addItemsToCustomInventory takes an optional 5th `identifier`, which is how
      a NON-SHARED container is addressed. removeItemFromCustomInventory has no
      such parameter, so programmatic removal from a non-shared container is not
      reachable through this API. Shared containers have no such problem.
    * Custom inventory definitions are memory-only. Nothing persists across a
      restart, so every consumer must re-register on boot.
    * Usable-item handlers are memory-only too, one per item, last registration
      wins, and vorp_inventory never drops one when its owner stops. sv_usables.lua
      keeps the registry and replays it; see invRegisterUsable.
    * The DB columns are lowercase (charidentifier, jobgrade, joblabel) while the
      in-memory object is camelCase (charIdentifier, jobGrade, jobLabel).
]]

PoggyCore = PoggyCore or {}
PoggyCore.Adapters = PoggyCore.Adapters or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

--- Canonical currency name -> VORP's numeric index.
local CURRENCY_INDEX = {
    cash = 0,
    gold = 1,
    rol  = 2,
}

--- Canonical currency name -> the field on the character object.
local CURRENCY_FIELD = {
    cash = "money",
    gold = "gold",
    rol  = "rol",
}

local VORP = {
    id    = "vorp",
    label = "VORP Core",

    --- The resource that holds usable-item handlers (and container definitions)
    --- in memory. sv_usables.lua watches it start and re-registers everything.
    inventoryResource = "vorp_inventory",

    caps = {
        ["money.cash"]           = true,
        ["money.bank"]           = false,  -- vorp_core genuinely has no bank
        ["money.gold"]           = true,
        ["money.rol"]            = true,
        ["char.onduty"]          = false,  -- flipped to true in init if vorp_police is present
        ["job.registry"]         = false,  -- no shared jobs table with labels/grades
        ["job.duty"]             = false,
        ["job.event"]            = true,
        ["inventory.items"]      = true,
        ["inventory.weapons"]    = true,
        ["inventory.metadata"]   = true,
        ["inventory.carrycheck"] = true,
        ["storage"]              = true,
        ["storage.persist"]      = false,  -- definitions are memory-only
        ["storage.permissions"]  = true,
        ["storage.weapons"]      = true,
        ["menu.native"]          = true,
        ["permissions"]          = true,
        -- 0.11.0; all three read or write VORP's tables through oxmysql
        ["char.offline"]         = true,   -- false in init without oxmysql
        ["job.persist"]          = true,   -- false in init without oxmysql
        ["inventory.registry"]   = true,   -- false in init without oxmysql
    },
}

-- ---------------------------------------------------------------------------
-- Lifecycle
-- ---------------------------------------------------------------------------

function VORP:init(core)
    self.core = core
    if not core or not PoggyCore.IsCallable(core.getUser) then
        return false, "vorp_core returned an object without getUser"
    end

    self.inv = exports.vorp_inventory
    if GetResourceState("vorp_inventory") ~= "started" then
        Util.Warn("vorp_inventory is not started. Item and storage calls will fail.")
        self.caps["inventory.items"]   = false
        self.caps["inventory.weapons"] = false
        self.caps["storage"]           = false
    end

    -- On-duty is only answerable when vorp_police or vorp_medic is present. If
    -- neither is, the capability stays false and Core.GetChar().onDuty returns
    -- nil rather than defaulting to true, which is what poggy_util did and
    -- means an off-duty check passes for everybody. See dutyOf.
    if GetResourceState("vorp_police") == "started" or GetResourceState("vorp_medic") == "started" then
        self.caps["char.onduty"] = true
    end

    if not Util.DbAvailable() then
        Util.Warn("oxmysql is not started. char.offline, char.list, inv.items, inv.itemInfo and job.set persist will refuse.")
        self.caps["char.offline"]       = false
        self.caps["job.persist"]        = false
        self.caps["inventory.registry"] = false
    end

    return true
end

-- ---------------------------------------------------------------------------
-- Identity
-- ---------------------------------------------------------------------------

--- Fetch the raw VORP character object for a source.
---@return table|nil user
---@return table|nil character
function VORP:raw(src)
    local user = self.core.getUser(src)
    if not user then return nil, nil end
    -- A VALUE, not a call. This is the single most common mistake against VORP.
    local char = user.getUsedCharacter
    if not char then return user, nil end
    return user, char
end

function VORP:getChar(src)
    local user, char = self:raw(src)
    if not char then return nil end

    local onDuty = self:dutyOf(src)

    local money = { cash = char.money, gold = char.gold, rol = char.rol }

    return {
        charId        = tostring(char.charIdentifier),
        ownerId       = tostring(char.identifier or (user and user.getIdentifier and user.getIdentifier()) or ""),
        source        = src,
        firstName     = char.firstname or "",
        lastName      = char.lastname or "",
        fullName      = ((char.firstname or "") .. " " .. (char.lastname or "")):gsub("^%s+", ""),
        job           = char.job or "unemployed",
        jobLabel      = char.jobLabel or char.job or "",
        jobGrade      = tonumber(char.jobGrade) or 0,
        jobGradeLabel = nil,     -- VORP has no per-grade label
        onDuty        = onDuty,  -- nil when unanswerable, deliberately
        group         = char.group or "user",
        gender        = PoggyCore.NormaliseGender(char.gender),
        money         = money,
        native        = char,
    }
end

function VORP:getCharByCharId(charId)
    if not PoggyCore.IsCallable(self.core.getUserByCharId) then return nil end
    local user = self.core.getUserByCharId(tonumber(charId) or charId)
    if not user then return nil end
    local src = user.source
    if not src then return nil end
    return self:getChar(src)
end

function VORP:getPlayers()
    local out = {}
    for _, src in ipairs(GetPlayers()) do
        out[#out + 1] = tonumber(src)
    end
    return out
end

--- Is this player on duty? true, false, or nil when nothing on the server can say.
---
--- vorp_police exports `isOnDuty` (lower case). Until 0.11.0 this adapter called
--- `IsPlayerOnDuty`, which does not exist, so the pcall failed quietly and
--- onDuty was nil for every player on every server. vorp_police and vorp_medic
--- also publish duty as the replicated state bags isPoliceDuty / isMedicDuty
--- (nil, not false, when off duty); vorp_billing and vorp_paycheck read those,
--- so they count here too.
function VORP:dutyOf(src)
    local police = GetResourceState("vorp_police") == "started"
    local medic  = GetResourceState("vorp_medic") == "started"
    if police then
        local ok, on = pcall(function() return exports.vorp_police:isOnDuty(src) end)
        if ok and on then return true end
    end
    local okState, state = pcall(function() return Player(src).state end)
    if okState and state and (state.isPoliceDuty or state.isMedicDuty) then return true end
    if police or medic then return false end
    return nil
end

--- A character by id, online or not.
---
--- Online characters come back live from vorp_core. Offline ones are read from
--- the characters table (the DB columns are lower case). Appearance is only in
--- the table, so asking for it always costs one query.
function VORP:charOffline(charId, withAppearance)
    local id = tonumber(charId)
    if not id then return nil, Err.BAD_ARG end

    local live = self:getCharByCharId(id)
    if live and not withAppearance then
        live.online = true
        return live
    end

    local cols = "charidentifier, identifier, firstname, lastname, job, joblabel, jobgrade, `group`, gender"
    if withAppearance then cols = cols .. ", skinPlayer, compPlayer, compTints" end
    local rows, err = Util.DbQuery("SELECT " .. cols .. " FROM characters WHERE charidentifier = ? LIMIT 1", { id })
    if not rows then return nil, err end
    local r = rows[1]
    if not r then return nil, Err.NOT_FOUND end

    local out = live or {
        charId        = tostring(r.charidentifier),
        ownerId       = tostring(r.identifier or ""),
        source        = nil,
        firstName     = r.firstname or "",
        lastName      = r.lastname or "",
        fullName      = ((r.firstname or "") .. " " .. (r.lastname or "")):gsub("^%s+", ""),
        job           = r.job or "unemployed",
        jobLabel      = r.joblabel or r.job or "",
        jobGrade      = tonumber(r.jobgrade) or 0,
        jobGradeLabel = nil,
        onDuty        = nil,
        group         = r.group or "user",
        gender        = PoggyCore.NormaliseGender(r.gender),
        native        = r,
    }
    out.online = live ~= nil
    if withAppearance then
        out.appearance = { skin = r.skinPlayer, comps = r.compPlayer, tints = r.compTints }
    end
    return out
end

--- Every character in the characters table, sorted by name.
function VORP:charList(opts)
    local sql, params = "SELECT charidentifier, identifier, firstname, lastname FROM characters", {}
    if type(opts.search) == "string" and opts.search ~= "" then
        sql = sql .. " WHERE CONCAT(firstname, ' ', lastname) LIKE ?"
        params[1] = "%" .. opts.search .. "%"
    end
    sql = sql .. " ORDER BY firstname, lastname"
    local limit, offset = tonumber(opts.limit), tonumber(opts.offset)
    if limit then
        -- Numbers written into the SQL, never strings: LIMIT cannot take a bound
        -- string, and flooring them is what keeps this safe.
        sql = sql .. (" LIMIT %d"):format(math.max(0, math.floor(limit)))
        if offset then sql = sql .. (" OFFSET %d"):format(math.max(0, math.floor(offset))) end
    end

    local rows, err = Util.DbQuery(sql, params)
    if not rows then return nil, err end
    local out = {}
    for _, r in ipairs(rows) do
        out[#out + 1] = {
            charId    = tostring(r.charidentifier),
            ownerId   = tostring(r.identifier or ""),
            firstName = r.firstname or "",
            lastName  = r.lastname or "",
            fullName  = ((r.firstname or "") .. " " .. (r.lastname or "")):gsub("^%s+", ""),
        }
    end
    return out
end

-- ---------------------------------------------------------------------------
-- Money
-- ---------------------------------------------------------------------------

function VORP:moneyGet(src, currency)
    local field = CURRENCY_FIELD[currency]
    if not field then return nil, Err.UNSUPPORTED end
    local _, char = self:raw(src)
    if not char then return nil, Err.NO_CHAR end
    return tonumber(char[field]) or 0
end

function VORP:moneyAdd(src, currency, amount)
    local idx = CURRENCY_INDEX[currency]
    if not idx then return false, Err.UNSUPPORTED end
    local _, char = self:raw(src)
    if not char then return false, Err.NO_CHAR end
    char.addCurrency(idx, amount)
    return true
end

function VORP:moneyRemove(src, currency, amount)
    local idx   = CURRENCY_INDEX[currency]
    local field = CURRENCY_FIELD[currency]
    if not idx then return false, Err.UNSUPPORTED end
    local _, char = self:raw(src)
    if not char then return false, Err.NO_CHAR end

    -- VORP will happily take a player negative. Refuse here instead, so every
    -- framework behaves the same way and a caller can trust `ok`.
    local balance = tonumber(char[field]) or 0
    if balance < amount then return false, Err.NO_FUNDS end

    char.removeCurrency(idx, amount)
    return true
end

function VORP:moneySet(src, currency, amount)
    local _, char = self:raw(src)
    if not char then return false, Err.NO_CHAR end
    if currency == "cash" then
        char.setMoney(amount)
    elseif currency == "gold" then
        char.setGold(amount)
    elseif currency == "rol" then
        char.setRol(amount)
    else
        return false, Err.UNSUPPORTED
    end
    return true
end

-- ---------------------------------------------------------------------------
-- Jobs
-- ---------------------------------------------------------------------------

function VORP:jobSet(src, name, grade, label)
    local _, char = self:raw(src)
    if not char then return false, Err.NO_CHAR end

    -- Two calls on VORP, one on our API. The second argument is a "fire the
    -- change event" flag; anything other than false fires it, and we want it.
    char.setJob(name, true)
    if grade ~= nil then
        char.setJobGrade(tonumber(grade) or 0, true)
    end
    if label and PoggyCore.IsCallable(char.setJobLabel) then
        char.setJobLabel(label)
    end
    return true
end

function VORP:jobSetDuty()
    -- vorp_core has no duty concept; vorp_police owns it and exposes no setter.
    return false, Err.UNSUPPORTED
end

--- Write the job to the characters table.
---
--- VORP's setters change only the in-memory character. Anything that reads
--- the table (vorp_crafting, other scripts) sees the old job until the
--- character is saved on disconnect, which is why poggy_multijob wrote the columns
--- itself. A grade or label left nil keeps what the row already has.
function VORP:jobPersist(src, name, grade, label)
    local _, char = self:raw(src)
    if not char then return false, Err.NO_CHAR end

    -- Built up rather than bound with nils: a Lua array with holes does not
    -- survive the trip into oxmysql intact.
    local sets, params = { "job = ?" }, { name }
    if grade ~= nil then
        sets[#sets + 1] = "jobgrade = ?"
        params[#params + 1] = tonumber(grade) or 0
    end
    if label ~= nil then
        sets[#sets + 1] = "joblabel = ?"
        params[#params + 1] = tostring(label)
    end
    params[#params + 1] = tonumber(char.charIdentifier)

    local affected, err = Util.DbExecute(
        "UPDATE characters SET " .. table.concat(sets, ", ") .. " WHERE charidentifier = ?", params)
    if not affected then return false, err end
    return true
end

-- ---------------------------------------------------------------------------
-- Inventory
-- ---------------------------------------------------------------------------

--- Did the framework actually do it?
---
--- Util.Await tells us the callback FIRED, not that the work succeeded.
--- vorp_inventory answers `false` when it refuses (71 places in
--- inventoryApiService.lua do exactly that) and `nil` when it simply has no
--- value to hand back, so only an explicit false counts as a refusal.
---
--- Ignoring this is how storage.addItem came back `true` for an add that
--- vorp_inventory had rejected outright with "charid is not valid". A write
--- that reports success it did not achieve is the worst kind of bug in this
--- codebase, because the caller then takes the money.
local function accepted(answer)
    return answer ~= false
end

function VORP:invCanCarry(src, item, qty)
    local ok, can = Util.Await(function(resolve)
        self.inv:canCarryItem(src, item, qty, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    return can and true or false, can and nil or Err.NO_SPACE
end

function VORP:invAdd(src, item, qty, meta)
    local ok, answer = Util.Await(function(resolve)
        self.inv:addItem(src, item, qty, meta or {}, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    if not accepted(answer) then return false, Err.FRAMEWORK_ERR end
    return true
end

function VORP:invRemove(src, item, qty, meta)
    -- Check first: subItem does not tell us whether the player actually had it.
    local count = self:invCount(src, item, meta)
    if count < qty then return false, Err.NOT_FOUND end

    local ok, answer = Util.Await(function(resolve)
        self.inv:subItem(src, item, qty, meta or {}, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    if not accepted(answer) then return false, Err.FRAMEWORK_ERR end
    return true
end

function VORP:invCount(src, item, meta)
    -- The callback is the SECOND argument here, not the last. This signature is
    -- inconsistent with every other function in the same file.
    local ok, count = Util.Await(function(resolve)
        self.inv:getItemCount(src, resolve, item, meta)
    end)
    if not ok then return 0 end
    return tonumber(count) or 0
end

function VORP:invGet(src)
    local ok, items = Util.Await(function(resolve)
        self.inv:getUserInventoryItems(src, resolve)
    end)
    if not ok or type(items) ~= "table" then return {} end

    local out = {}
    for _, it in pairs(items) do
        out[#out + 1] = {
            name   = it.name,
            amount = tonumber(it.count) or tonumber(it.amount) or 0,
            meta   = it.metadata,
            label  = it.label,
            weight = it.weight,
            native = it,
        }
    end
    return out
end

function VORP:invSetMeta(src, itemId, meta, amount)
    local ok, answer = Util.Await(function(resolve)
        self.inv:setItemMetadata(src, itemId, meta or {}, amount or 1, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    if not accepted(answer) then return false, Err.FRAMEWORK_ERR end
    return true
end

function VORP:invRegisterUsable(item, fn, resource)
    -- The third argument is the registering resource. vorp_inventory keeps it
    -- for a debug print fifteen seconds after its own start and for nothing
    -- else: it does NOT drop the handler when that resource stops (nothing in
    -- vorp_inventory's server listens for onResourceStop), and a restart of
    -- vorp_inventory empties its handler table outright. sv_usables.lua covers
    -- both, which is why the registry, not this call, is the source of truth.
    --
    -- pcall: with vorp_inventory stopped the export proxy throws, and the
    -- registry must keep the entry for when it is back.
    local ok = pcall(function() self.inv:registerUsableItem(item, fn, resource) end)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Drop a handler. VORP never does this on its own when the owner stops, so a
--- stopped script's dead funcref would otherwise sit there until replaced.
function VORP:invUnregisterUsable(item)
    local ok = pcall(function() self.inv:unRegisterUsableItem(item) end)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

--- Is vorp_inventory answering its exports? Right after onResourceStart the
--- proxy can still throw, so re-registration waits on this. With no callback,
--- isCustomInventoryRegistered answers synchronously (its respond() returns the
--- value when cb is nil) and changes nothing.
function VORP:invReady()
    if GetResourceState(self.inventoryResource) ~= "started" then return false end
    local ok = pcall(function() return self.inv:isCustomInventoryRegistered("poggy_core:probe") end)
    return ok
end

--- Every item in the items table, the same read vorp_inventory makes at start.
--- sv_core caches the result; this runs once per ItemCacheSeconds at most.
function VORP:itemRegistry()
    local rows, err = Util.DbQuery("SELECT * FROM items", {})
    if not rows then return nil, err end
    local base = self:imageBase()
    local out = {}
    for _, r in ipairs(rows) do
        if r.item then
            out[#out + 1] = {
                name   = r.item,
                label  = r.label or r.item,
                desc   = r.desc,
                weight = tonumber(r.weight),
                limit  = tonumber(r.limit),
                type   = r.type,
                usable = r.usable == 1 or r.usable == true,
                group  = r.groupId,
                image  = base .. r.item .. ".png",
            }
        end
    end
    return out
end

function VORP:imageBase()
    return PoggyCore.Frameworks.vorp.itemImageBase
end

--- Does vorp_inventory ship an icon for this item? Reads the file, so callers cache it.
function VORP:itemImageExists(name)
    return LoadResourceFile("vorp_inventory", "html/img/items/" .. tostring(name) .. ".png") ~= nil
end

--- Close the player's own inventory screen. No id: an id closes a container.
function VORP:invClose(src)
    local ok = pcall(function() self.inv:closeInventory(src) end)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return true
end

-- ---------------------------------------------------------------------------
-- Weapons
-- ---------------------------------------------------------------------------

function VORP:weaponsGet(src)
    local ok, weapons = Util.Await(function(resolve)
        self.inv:getUserInventoryWeapons(src, resolve)
    end)
    if not ok or type(weapons) ~= "table" then return {} end
    return weapons
end

function VORP:weaponAdd(src, weaponName, ammo, components)
    -- Deliberately WITHOUT a callback.
    --
    -- vorp_inventory relays createWeapon's reply callback through an event,
    -- positioned after two nil arguments, and a CFX event drops every argument
    -- following a nil. The callback is therefore never delivered: the weapon is
    -- created, but anything waiting on the callback waits forever. Passing no
    -- callback makes the export return a boolean synchronously instead.
    --
    -- This was found the hard way in poggy_markets; the comment there records a
    -- purchase that took the money, created the gun, and then never updated
    -- stock or told the player, because the wrapper never resumed.
    local ok, res = pcall(function()
        return self.inv:createWeapon(src, weaponName, ammo or {}, components or {}, {})
    end)
    if not ok then return false, Err.FRAMEWORK_ERR end
    return res == true
end

function VORP:weaponRemove(src, weaponId)
    local ok, answer = Util.Await(function(resolve)
        self.inv:subWeapon(src, weaponId, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    if not accepted(answer) then return false, Err.FRAMEWORK_ERR end
    return true
end

function VORP:weaponCanCarry(src, qty, weapon)
    -- (source, amount, callback, weaponName): the callback is THIRD here, and
    -- the weapon name after it. Checked against inventoryApiService.lua.
    local ok, can = Util.Await(function(resolve)
        self.inv:canCarryWeapons(src, qty, resolve, weapon)
    end)
    if not ok then return false, Err.TIMEOUT end
    return can and true or false, can and nil or Err.NO_SPACE
end

-- ---------------------------------------------------------------------------
-- Storage (VORP custom inventories)
-- ---------------------------------------------------------------------------

function VORP:stIsRegistered(id)
    local ok, registered = Util.Await(function(resolve)
        self.inv:isCustomInventoryRegistered(id, resolve)
    end)
    if not ok then return false end
    return registered and true or false
end

function VORP:stRegister(id, opts)
    -- Idempotent by contract. VORP keeps definitions in memory only, so every
    -- consumer re-registers at boot; registering twice in one session would
    -- otherwise reset the container.
    if self:stIsRegistered(id) then
        -- Capacity may still have changed between boots.
        if opts.slots then self.inv:updateCustomInventorySlots(id, opts.slots) end
        return true
    end

    -- DO NOT add a key here whose name matches a method on vorp_inventory's
    -- customInventory class.
    --
    -- vorp_lib's class:New does `setmetatable(data, cls)` when handed a single
    -- table: the payload IS the object. Methods resolve through __index, so any
    -- key you pass shadows the method of the same name, permanently, on that
    -- container.
    --
    -- `useWeight = <boolean>` used to be in this list. The class stores the
    -- value in `self.useweight` (lower case) and exposes a `useWeight()` getter,
    -- so passing the documented field name replaced the getter with a boolean
    -- and every openInventory on the container died with
    -- "attempt to call a boolean value (method 'useWeight')". Removing the key
    -- is the whole fix: the constructor defaults `useweight` to false, which is
    -- what every slot-limited container wants anyway.
    --
    -- The method names as of vorp_inventory 4.x include getLimit, getName,
    -- getWeight, getWebhook, isShared, isInUse, setInUse and useWeight. Check
    -- server/models/customInventory.lua before adding anything.
    --
    -- opts.maxWeight is therefore ignored here and the slot limit applies. That
    -- used to print a warning per container, which said nothing an operator
    -- could act on, so it is documented in the README instead.

    self.inv:registerInventory({
        id                   = id,
        name                 = opts.label or id,
        limit                = opts.slots or 50,
        acceptWeapons        = opts.allowWeapons and true or false,
        shared               = opts.shared ~= false,
        ignoreItemStackLimit = opts.ignoreStackLimit and true or false,
        whitelistItems       = opts.whitelistItems and true or false,
        UsePermissions       = (opts.jobAccess or opts.charAccess) and true or false,
        UseBlackList         = false,
        webhook              = opts.webhook or false,
    })

    for job, grade in pairs(opts.jobAccess or {}) do
        self.inv:AddPermissionMoveToCustom(id, job, grade)
        self.inv:AddPermissionTakeFromCustom(id, job, grade)
    end
    for _, charId in ipairs(opts.charAccess or {}) do
        self.inv:AddCharIdPermissionMoveToCustom(id, tonumber(charId), true)
        self.inv:AddCharIdPermissionTakeFromCustom(id, tonumber(charId), true)
    end

    return true
end

function VORP:stOpen(src, id)
    self.inv:openInventory(src, id)
    return true
end

function VORP:stClose(src, id)
    self.inv:closeInventory(src, id)
    return true
end

function VORP:stAddItem(id, item, qty, meta, charId)
    local ok, answer = Util.Await(function(resolve)
        self.inv:addItemsToCustomInventory(id, {
            { name = item, amount = qty, metadata = meta or {} },
        }, charId and tonumber(charId) or nil, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    if not accepted(answer) then return false, Err.FRAMEWORK_ERR end
    return true
end

function VORP:stRemoveItem(id, item, qty)
    local ok, answer = Util.Await(function(resolve)
        -- Five parameters, not four: (id, name, amount, item_crafted_id, callback).
        -- Passing resolve in fourth place put a FUNCTION where the crafted-item
        -- id goes, so vorp_inventory took the "specific instance" branch and
        -- queried `WHERE item_crafted_id = <function>` — hence the SQL error
        -- "val.slice is not a function" — while `callback` stayed nil, so our
        -- resolve was never called and every remove timed out.
        -- nil means "any instance of this item", which is what we want.
        self.inv:removeItemFromCustomInventory(id, item, qty, nil, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end
    if not accepted(answer) then return false, Err.FRAMEWORK_ERR end
    return true
end

function VORP:stGetItems(id)
    local ok, items = Util.Await(function(resolve)
        self.inv:getCustomInventoryItems(id, resolve)
    end)
    if not ok or type(items) ~= "table" then return {} end

    local out = {}
    for _, it in pairs(items) do
        out[#out + 1] = {
            name   = it.name,
            amount = tonumber(it.count) or tonumber(it.amount) or 0,
            meta   = it.metadata,
            label  = it.label,
            native = it,
        }
    end
    return out
end

--- Weapons stored in a container. vorp_inventory answers false for an id it
--- does not know, which is reported as not_found rather than as an empty list.
function VORP:stGetWeapons(id)
    local ok, weapons = Util.Await(function(resolve)
        self.inv:getCustomInventoryWeapons(id, resolve)
    end)
    if not ok then return nil, Err.TIMEOUT end
    if weapons == false then return nil, Err.NOT_FOUND end
    if type(weapons) ~= "table" then return {} end

    local out = {}
    for _, w in pairs(weapons) do
        out[#out + 1] = {
            id     = w.id,
            name   = w.name,
            label  = w.custom_label or w.label or w.name,
            serial = w.serial_number,
            desc   = w.custom_desc or w.desc,
            native = w,
        }
    end
    return out
end

function VORP:stSetCapacity(id, slots)
    if not slots then return false, Err.BAD_ARG end
    self.inv:updateCustomInventorySlots(id, slots)
    return true
end

--- Drop the definition without touching the contents.
---
--- Distinct from stDelete on purpose. VORP's removeInventory forgets the
--- container in memory while its items stay in character_inventories, so the
--- contents come back when it is registered again. deleteCustomInventory
--- destroys them. Conflating the two is how a restart eats a player's stash.
function VORP:stUnregister(id)
    local ok = pcall(function() self.inv:removeInventory(id) end)
    return ok
end

--- Destroy the container AND everything in it. Not reversible.
function VORP:stDelete(id)
    local ok = Util.Await(function(resolve)
        self.inv:deleteCustomInventory(id, resolve)
    end)
    if not ok then return false, Err.TIMEOUT end

    -- deleteCustomInventory's answer cannot be trusted either way. Read its
    -- SECONDARY.DELETE in inventoryApiService.lua: when the service call returns
    -- falsy it removes the container anyway and THEN answers `false`. So a
    -- `false` here can mean "deleted". Ask whether the container still exists
    -- instead of believing what we were told.
    local stillThere = self:stIsRegistered(id)
    if stillThere then return false, Err.FRAMEWORK_ERR end
    return true
end

-- ---------------------------------------------------------------------------
-- Permissions
-- ---------------------------------------------------------------------------

function VORP:permGroup(src)
    local _, char = self:raw(src)
    if not char then return "user" end
    return char.group or "user"
end

-- ---------------------------------------------------------------------------
-- Notifications
-- ---------------------------------------------------------------------------

--- Server-side notify through VORP's own API. sv_core only reaches this when the
--- 'framework' renderer is selected in config.
function VORP:notify(src, text, kind, duration)
    local core = self.core
    if kind == "error" then
        core.NotifyRightTip(src, text, duration)
    elseif kind == "success" then
        core.NotifyRightTip(src, text, duration)
    else
        core.NotifyRightTip(src, text, duration)
    end
    return true
end

function VORP:notifyRich(src, opts)
    local core = self.core
    if opts.title and opts.description then
        core.NotifySimpleTop(src, opts.title, opts.description, opts.duration)
    elseif opts.dict and opts.icon then
        core.NotifyLeft(src, opts.title or "", opts.description or "",
            opts.dict, opts.icon, opts.duration, opts.color or "COLOR_WHITE")
    else
        core.NotifyRightTip(src, opts.title or opts.description or "", opts.duration)
    end
    return true
end

-- ---------------------------------------------------------------------------
-- Escape hatch
-- ---------------------------------------------------------------------------

function VORP:nativeCore()
    return self.core
end

PoggyCore.Adapters.vorp = VORP
