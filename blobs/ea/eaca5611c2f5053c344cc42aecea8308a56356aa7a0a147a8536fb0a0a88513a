--[[===========================================================================
    poggy_crafting · client/ui.lua
    ---------------------------------------------------------------------------
    Opening and closing the browser, and the NUI callbacks behind it.

    The browser is told everything it needs in one message so it can draw
    immediately: the recipes, the categories, the player's job and inventory,
    item labels and carry limits, and the URL its icons live under. The icon URL
    comes from poggy_core rather than being hard-coded, which is what lets the
    same page work on VORP, RSG and QBCore.
===========================================================================]]--

PoggyCrafting = PoggyCrafting or {}
local PC = PoggyCrafting
local T  = PC.T

PC.uiOpen = false

local itemData = nil   -- labels, limits and imageBase, fetched once per session
local hiddenFetched = false   -- has the hidden-recipe set been asked for yet?
local recipesSent = false     -- has the page been given the recipe list yet?

--- Call a poggy_core server callback and unpack its answer.
local function ask(name, args)
    local ok, packed = Poggy('callback.await', {
        name = name,
        args = args and { n = 1, args } or { n = 0 },
    })
    if not ok or type(packed) ~= 'table' then return nil end
    return packed[1]
end
PC.Ask = ask

--- Which categories this location offers. A bench with `Categories = 0`, and
--- the /craft command, offer all of them.
local function categoriesFor(location)
    if not location or location.Categories == nil or location.Categories == 0 then
        return Config.Categories
    end
    local out = {}
    for _, ident in ipairs(location.Categories) do
        for _, cat in ipairs(Config.Categories) do
            if cat.ident == ident then out[#out + 1] = cat break end
        end
    end
    return out
end

--- Open the browser at `location`. `location.id` is what recipes and
--- categories match their Location against.
function PC.OpenUI(location)
    if PC.uiOpen then return end

    itemData = itemData or ask('poggy_crafting:items') or {}
    local inventory = ask('poggy_crafting:inventory') or {}
    local job       = ask('poggy_crafting:job')

    -- Which recipes the server has hidden for want of an item icon. Asked for
    -- once per session; the answer only changes when the resource restarts.
    if not hiddenFetched then
        hiddenFetched = true
        PC.Hidden = ask('poggy_crafting:hidden') or {}
    end

    PC.uiOpen = true

    if Config.KneelingAnimation then
        PC.ForceRestScenario(true)
    end

    -- The recipe list goes to the page once per session. It is by far the
    -- largest thing this script sends -- hundreds of kilobytes on a server with
    -- hundreds of recipes -- and it cannot change until the resource restarts,
    -- which reloads the page as well. Every later open sends only what can
    -- change: the place, the job, the inventory.
    if not recipesSent then
        SendNUIMessage({ type = 'recipes', craftables = PC.VisibleRecipes() })
        recipesSent = true
    end

    SendNUIMessage({
        type        = 'open',
        categories  = categoriesFor(location),
        location    = location,
        job         = job,
        crafttime   = Config.CraftTime,
        style       = Config.Styles,
        language    = PC.Locale,
        itemLabels  = itemData.labels    or {},
        itemLimits  = itemData.limits    or {},
        imageBase   = itemData.imageBase or '',
        inventory   = inventory,
        itemSources = Config.ItemSources or {},
        placeNames  = PC.PlaceNameMap(),
        shoppingList = Config.ShoppingList and Config.ShoppingList.enabled or false,
    })

    SetNuiFocus(true, true)
end

--- Close it, from either side. `fromUI` means the page has already torn itself
--- down and does not need telling.
function PC.CloseUI(fromUI)
    if not PC.uiOpen then return end
    PC.uiOpen = false

    if not fromUI then
        SendNUIMessage({ type = 'close' })
    end

    SetNuiFocus(false, false)

    if Config.KneelingAnimation then
        PC.ForceRestScenario(false)
    end
end

-- ===========================================================================
--  Messages in
-- ===========================================================================

RegisterNetEvent('poggy_crafting:inventory', function(inventory)
    SendNUIMessage({ type = 'inventory', inventory = inventory })
end)

-- ===========================================================================
--  NUI callbacks
-- ===========================================================================

RegisterNUICallback('close', function(_, cb)
    PC.CloseUI(true)
    cb('ok')
end)

-- The page answering "I am gone" after it was told to close. client/craft.lua
-- waits on this before handing the screen to poggy_skillcheck.
PC.uiConfirmedClosed = false
RegisterNUICallback('closed', function(_, cb)
    PC.uiConfirmedClosed = true
    cb('ok')
end)

RegisterNUICallback('craft', function(data, cb)
    local quantity = math.floor(tonumber(data and data.quantity) or 0)
    if quantity < 1 then
        Poggy('notify.styled', { style = 'tip', text = T('invalid_amount'), duration = 4000 })
        return cb('invalid')
    end

    TriggerServerEvent('poggy_crafting:craft',
        data.recipe, quantity, data.location and data.location.id or nil)
    cb('ok')
end)

RegisterNUICallback('trackerRelease', function(_, cb)
    if not PC.uiOpen then SetNuiFocus(false, false) end
    cb('ok')
end)

-- ── Shopping list ───────────────────────────────────────────────────────
-- Each one is a straight pass-through to the server callback of the same name.

local LIST_CALLBACKS = {
    ['list:get']    = 'poggy_crafting:list:get',
    ['list:add']    = 'poggy_crafting:list:add',
    ['list:remove'] = 'poggy_crafting:list:remove',
    ['list:toggle'] = 'poggy_crafting:list:toggle',
    ['list:qty']    = 'poggy_crafting:list:qty',
    ['list:clear']  = 'poggy_crafting:list:clear',
}

for nuiName, serverName in pairs(LIST_CALLBACKS) do
    RegisterNUICallback(nuiName, function(data, cb)
        if not (Config.ShoppingList and Config.ShoppingList.enabled) then return cb({}) end
        cb(ask(serverName, data) or {})
    end)
end

-- ===========================================================================
--  Tracker commands
-- ===========================================================================

-- The gathering tracker floats over the game with no focus of its own. This
-- lends it the mouse for a moment so it can be dragged somewhere else.
if Config.Commands.tracker then
    RegisterCommand(Config.Commands.tracker, function()
        if PC.uiOpen then return end
        SendNUIMessage({ type = 'trackerFocus' })
        SetNuiFocus(true, true)
    end, false)
end

if Config.Commands.trackerHide then
    RegisterCommand(Config.Commands.trackerHide, function()
        SendNUIMessage({ type = 'trackerToggle' })
    end, false)
end

-- ===========================================================================
--  Live inventory
-- ===========================================================================

-- While the browser is open, refresh what the player is carrying every few
-- seconds so a trade, a pickup or another script's reward shows up without
-- reopening. Nothing runs while it is closed.
CreateThread(function()
    while true do
        if PC.uiOpen then
            local inventory = ask('poggy_crafting:inventory')
            if inventory then
                SendNUIMessage({ type = 'inventory', inventory = inventory })
            end
            Wait(3000)
        else
            Wait(1000)
        end
    end
end)

AddEventHandler('onResourceStop', function(resource)
    if resource ~= GetCurrentResourceName() then return end
    SetNuiFocus(false, false)
    if Config.KneelingAnimation then PC.ForceRestScenario(false) end
end)
