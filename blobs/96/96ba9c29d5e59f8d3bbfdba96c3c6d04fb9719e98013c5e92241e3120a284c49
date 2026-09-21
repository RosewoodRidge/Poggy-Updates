--[[===========================================================================
    poggy_crafting · server/craft.lua
    ---------------------------------------------------------------------------
    The craft itself. Nothing the client sends is trusted beyond the recipe's
    name: the recipe is looked up again here, the job is read again here, and
    the ingredients are counted against the player's real inventory.

    Order of events, and it matters:
        validate -> take the ingredients -> reward
    or, for a recipe with a skillcheck:
        validate -> take the ingredients -> client plays it -> reward a share

    Materials come out before the skillcheck on purpose. A failed craft has to
    cost something or there is no reason to aim.
===========================================================================]]--

PoggyCrafting = PoggyCrafting or {}
local PC = PoggyCrafting
local T  = PC.T

-- CurrencyType in a recipe -> the name poggy_core uses.
local CURRENCY = { [0] = 'cash', [1] = 'gold', [2] = 'rol' }

-- Skillchecks in flight, keyed by an id this server made up. A client can only
-- ever report against an id it was handed, and only once.
local pending    = {}
local pendingSeq = 0

-- ===========================================================================
--  Ingredients
-- ===========================================================================

--- Walk the player's inventory and work out whether the recipe's slots are
--- filled, and by which items.
---
--- Returns ok, taking, consumedAlt
---     taking       array of { name, qty } to remove
---     consumedAlt  the alt name that filled a slot, for AltRewards
local function gatherIngredients(src, recipe, quantity)
    local items = select(2, Poggy('inv.get', { src = src })) or {}

    -- slot key -> what it still needs, and which names can fill it
    local slots = {}
    for _, item in ipairs(recipe.Items or {}) do
        local take = item.take ~= false
        slots[item.name] = {
            -- A tool (take = false) is not used up, so it is needed ONCE however
            -- many are crafted. It used to be count x quantity: one hatchet
            -- capped every craft at one (2.0.11).
            need        = take and item.count * quantity or item.count,
            found       = 0,
            take        = take,
            canUseDecay = tonumber(item.canUseDecay),
            fill        = {},
            -- With AltRewards the payout follows the ONE alternative used, so a
            -- batch is filled from a single name: 3 pine + 2 oak is not 5 of
            -- anything. `names` is the order they are tried in, the recipe's
            -- own item first.
            names       = (take and recipe.AltRewards and item.AltNames and #item.AltNames > 0)
                and { item.name, table.unpack(item.AltNames) } or nil,
        }
    end

    -- alt name -> the slot it belongs to
    local altOf = {}
    for _, item in ipairs(recipe.Items or {}) do
        for _, alt in ipairs(item.AltNames or {}) do altOf[alt] = item.name end
    end

    --- canUseDecay: VORP marks degradable items with a condition percentage. A
    --- framework that has no such idea reports neither field, and the item is
    --- accepted -- the check can only ever make a recipe stricter, never block
    --- one on a framework without decay.
    local function usableIn(slot, entry)
        if not slot.canUseDecay then return true end
        local native = entry.native
        if native and native.isDegradable then
            return (tonumber(native.percentage) or 100) >= slot.canUseDecay
        end
        return true
    end

    -- Single-name slots: which one name covers the whole batch on its own?
    local stock = {}   -- slot key -> name -> usable amount
    for _, entry in pairs(items) do
        local key  = altOf[entry.name] or entry.name
        local slot = slots[key]
        if slot and slot.names and usableIn(slot, entry) then
            stock[key] = stock[key] or {}
            stock[key][entry.name] = (stock[key][entry.name] or 0) + (tonumber(entry.amount) or 0)
        end
    end
    for key, slot in pairs(slots) do
        if slot.names then
            for _, name in ipairs(slot.names) do
                if ((stock[key] or {})[name] or 0) >= slot.need then slot.only = name break end
            end
            if not slot.only then return false, nil, nil end
        end
    end

    local consumedAlt = nil

    for _, entry in pairs(items) do
        local key  = altOf[entry.name] or entry.name
        local slot = slots[key]
        if slot and slot.found < slot.need and (not slot.only or slot.only == entry.name) then
            local usable = usableIn(slot, entry)

            if usable then
                local take = math.min(tonumber(entry.amount) or 0, slot.need - slot.found)
                if take > 0 then
                    slot.found = slot.found + take
                    if slot.take then
                        slot.fill[#slot.fill + 1] = { name = entry.name, qty = take }
                    end
                    if altOf[entry.name] and not consumedAlt then
                        consumedAlt = entry.name
                    end
                end
            end
        end
    end

    local taking = {}
    for _, slot in pairs(slots) do
        if slot.found < slot.need then return false, nil, nil end
        for _, fill in ipairs(slot.fill) do taking[#taking + 1] = fill end
    end

    return true, taking, consumedAlt
end

--- Everything the reward would add, and whether the player can hold it.
local function canCarryReward(src, recipe, reward, quantity)
    if recipe.Type == 'weapon' then
        for _, entry in ipairs(reward) do
            local ok, can = Poggy('weapon.canCarry', {
                src = src, qty = entry.count * quantity, weapon = entry.name,
            })
            if not ok or not can then return false end
        end
        return true
    end

    for _, entry in ipairs(reward) do
        if entry.name then
            local ok, can = Poggy('inv.canCarry', {
                src = src, item = entry.name, qty = entry.count * quantity,
            })
            if not ok or not can then return false end
        end
    end
    return true
end

-- ===========================================================================
--  Rewards
-- ===========================================================================

--- Hand over one reward entry. `scale` is 1 for a normal craft and the
--- skillcheck's share otherwise.
local function giveReward(src, recipe, entry, quantity, scale)
    local currency = CURRENCY[recipe.CurrencyType or 0] or 'cash'

    -- No name, in currency mode, means money: negative is a price, positive
    -- is a payment.
    if recipe.UseCurrencyMode and not entry.name then
        local amount = math.floor(entry.count * quantity * scale + 0.5)
        if amount == 0 then return end
        if amount < 0 then
            Poggy('money.remove', {
                src = src, amount = math.abs(amount), currency = currency,
                reason = 'poggy_crafting:' .. recipe.Text,
            })
            PC.Notify(src, T('paid', math.abs(amount)), 'error')
        else
            Poggy('money.add', {
                src = src, amount = amount, currency = currency,
                reason = 'poggy_crafting:' .. recipe.Text,
            })
            PC.Notify(src, T('earned', amount))
        end
        return
    end

    if not entry.name then return end

    local amount = math.floor(entry.count * quantity * scale + 0.5)
    if amount < 1 then
        -- A partial skillcheck that rounds to nothing still owes them one.
        if scale < 1 and scale > 0 then amount = 1 else return end
    end

    if recipe.Type == 'weapon' then
        for _ = 1, amount do
            Poggy('weapon.add', { src = src, weapon = entry.name })
        end
    else
        Poggy('inv.add', { src = src, item = entry.name, qty = amount })
    end

    PC.Notify(src, T('crafted', amount, PC.ItemLabel(entry.name)))

    if Config.WebhookLogCrafts then
        PC.Webhook('Crafted', ('**%s** made %sx %s'):format(
            GetPlayerName(src) or src, amount, entry.name))
    end
end

local function giveAllRewards(src, recipe, reward, quantity, scale)
    for _, entry in ipairs(reward) do
        giveReward(src, recipe, entry, quantity, scale)
    end
    PC.PushInventory(src)
end

-- ===========================================================================
--  Where the player says they are
-- ===========================================================================

-- A bench is a fixed point the server knows, so it checks the player is really
-- there, that the bench's job lock lets them use it, and that the bench offers
-- the recipe's category. World props and placed campfires are drawn by the
-- game on each client and the server cannot see them, so for those it checks
-- what it can: Config.CampfireJobLock. Anything else is treated as "no place",
-- which only reaches recipes that need none.

-- Slack on top of the prompt distance: the player may shuffle while crafting,
-- and positions reach the server a moment late.
local BENCH_SLACK = 5.0

local function benchById(id)
    for _, loc in ipairs(Config.Locations or {}) do
        if loc.id == id then return loc end
    end
end

local function isPropTitle(id)
    for _, p in ipairs(Config.CraftingProps or {}) do
        if type(p.title) == 'string' and p.title:lower() == id then return true end
    end
    return false
end

--- The location id the server accepts for this craft, or nil for none.
--- Returns false when the player claims a place they may not use.
local function checkedLocation(src, locationId, jobName, recipe)
    if type(locationId) ~= 'string' or locationId == '' then return nil end

    local bench = benchById(locationId)
    if bench then
        if not PC.Allows(bench.Job, jobName) then return false end
        if bench.Categories and bench.Categories ~= 0
            and not PC.Allows(bench.Categories, recipe.Category) then
            return false
        end
        local ped = GetPlayerPed(src)
        local pos = ped and ped ~= 0 and GetEntityCoords(ped) or nil
        -- Without OneSync the server has no position (0,0,0): skip the check
        -- rather than refuse every bench.
        if pos and (pos.x ~= 0.0 or pos.y ~= 0.0 or pos.z ~= 0.0) then
            local reach = (tonumber(Config.Distances and Config.Distances.locations) or 1.0) + BENCH_SLACK
            local dx, dy, dz = pos.x - bench.x, pos.y - bench.y, pos.z - bench.z
            if dx * dx + dy * dy + dz * dz > reach * reach then return false end
        end
        return locationId
    end

    if isPropTitle(locationId) then
        if not Config.CraftingPropsEnabled then return false end
        if not PC.Allows(Config.CampfireJobLock, jobName) then return false end
        return locationId
    end

    return nil
end

-- ===========================================================================
--  The craft request
-- ===========================================================================

RegisterNetEvent('poggy_crafting:craft', function(recipeName, quantity, locationId)
    local src = source

    quantity = math.floor(tonumber(quantity) or 0)
    if quantity < 1 or quantity > 100 then return end

    local recipe = PC.FindRecipe(recipeName)
    if not recipe then
        return PC.Notify(src, T('unknown_recipe'), 'error')
    end

    local okJob, job = Poggy('job.get', { src = src })
    local jobName = okJob and job and job.name or nil

    locationId = checkedLocation(src, locationId, jobName, recipe)
    if locationId == false then
        return PC.Notify(src, T('not_here'), 'error')
    end

    local allowed, needsJobSkillcheck = PC.CanCraft(recipe, jobName, locationId)
    if not allowed then
        -- Name the check that failed. A missing place and a missing job are
        -- different problems, and the player can only fix one of them.
        local why, detail = PC.WhyLocked(recipe, jobName, locationId)
        if why == 'category_place' or why == 'recipe_place' then
            -- Each of these lines is newer than the one before it; a
            -- translations.lua that predates it falls back to the older wording
            -- rather than showing the key.
            local L = PC.Locale or {}
            local places = PC.PlaceNames(detail)
            local text
            if places ~= '' and L.not_here_at then text = T('not_here_at', places)
            elseif L.not_here then text = T('not_here')
            else text = T('not_job') end
            return PC.Notify(src, text, 'error')
        end
        return PC.Notify(src, T('not_job'), 'error')
    end

    -- Money is checked before the ingredients so a player who cannot pay is
    -- told that, rather than being told they are short of something they hold.
    local reward   = recipe.Reward or {}
    local currency = CURRENCY[recipe.CurrencyType or 0] or 'cash'
    if recipe.UseCurrencyMode then
        local cost = 0
        for _, entry in ipairs(reward) do
            if not entry.name and entry.count < 0 then
                cost = cost + math.abs(entry.count * quantity)
            end
        end
        if cost > 0 then
            local okMoney, balance = Poggy('money.get', { src = src, currency = currency })
            if not okMoney or (tonumber(balance) or 0) < cost then
                return PC.Notify(src, T('no_money', cost), 'error')
            end
        end
    end

    local haveAll, taking, consumedAlt = gatherIngredients(src, recipe, quantity)
    if not haveAll then
        return PC.Notify(src, T('not_enough'), 'error')
    end

    -- AltRewards: which specific alternative filled the slot decides the payout.
    if consumedAlt and recipe.AltRewards and recipe.AltRewards[consumedAlt] then
        reward = recipe.AltRewards[consumedAlt]
    end

    if not canCarryReward(src, recipe, reward, quantity) then
        return PC.Notify(src,
            recipe.Type == 'weapon' and T('weapons_full') or T('too_full'), 'error')
    end

    -- Point of no return: take the ingredients.
    for _, fill in ipairs(taking) do
        Poggy('inv.remove', { src = src, item = fill.name, qty = fill.qty })
    end

    local reps, profile = PC.SkillcheckFor(recipe, needsJobSkillcheck, quantity)

    if reps > 0 then
        pendingSeq = pendingSeq + 1
        local id = pendingSeq

        pending[id] = {
            src        = src,
            recipe     = recipe,
            reward     = reward,
            quantity   = quantity,
            reps       = reps,
            expires    = os.time() + 300,
            explode    = recipe.explodeOnFail or false,
            catchFire  = recipe.catchFireOnFail or false,
        }

        TriggerClientEvent('poggy_crafting:skillcheck', src, {
            id        = id,
            reps      = reps,
            profile   = profile,
            animation = recipe.Animation or 'craft',
            explode   = recipe.explodeOnFail or false,
            catchFire = recipe.catchFireOnFail or false,
        })
        return
    end

    giveAllRewards(src, recipe, reward, quantity, 1)
    TriggerClientEvent('poggy_crafting:play', src,
        recipe.Animation or 'craft',
        tonumber(recipe.CraftTime) or Config.CraftTime)
end)

-- ===========================================================================
--  Skillcheck result
-- ===========================================================================

RegisterNetEvent('poggy_crafting:skillcheckResult', function(id, passed, greats)
    local src = source

    id = tonumber(id)
    local job = id and pending[id]
    if not job then return end

    -- One report per session, from the player who started it.
    pending[id] = nil
    if job.src ~= src then return end

    passed = math.max(0, math.min(math.floor(tonumber(passed) or 0), job.reps))
    greats = math.max(0, math.min(math.floor(tonumber(greats) or 0), passed))

    if passed == 0 then
        if job.explode then
            TriggerClientEvent('poggy_crafting:explode', src)
            PC.Notify(src, T('skillcheck_exploded'), 'error')
        elseif job.catchFire then
            TriggerClientEvent('poggy_crafting:catchFire', src)
            PC.Notify(src, T('skillcheck_fire'), 'error')
        else
            PC.Notify(src, T('skillcheck_failed'), 'error')
        end
        PC.PushInventory(src)
        return
    end

    local bonus = tonumber(Config.Skillcheck.GreatBonusPct) or 0
    local scale = (passed / job.reps) * (1 + greats * bonus)

    giveAllRewards(src, job.recipe, job.reward, job.quantity, scale)

    PC.Notify(src, greats > 0
        and T('skillcheck_greats', passed, job.reps, greats)
        or  T('skillcheck_summary', passed, job.reps))
end)

-- A player who logs out mid-skillcheck simply loses the materials; leaving the
-- session behind would let them claim it on their next character.
AddEventHandler('playerDropped', function()
    local src = source
    for id, job in pairs(pending) do
        if job.src == src then pending[id] = nil end
    end
end)

-- Sessions nobody ever reported on. Five minutes is far longer than any
-- skillcheck and short enough that the table cannot grow.
CreateThread(function()
    while true do
        Wait(60000)
        local now = os.time()
        for id, job in pairs(pending) do
            if job.expires < now then pending[id] = nil end
        end
    end
end)
