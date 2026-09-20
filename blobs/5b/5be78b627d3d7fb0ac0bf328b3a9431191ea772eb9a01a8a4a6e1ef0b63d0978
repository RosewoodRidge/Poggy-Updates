--[[===========================================================================
    poggy_crafting · server/main.lua
    ---------------------------------------------------------------------------
    Startup, the usable campfire item, the webhook, and the callbacks the
    browser needs before it can draw anything.
===========================================================================]]--

PoggyCrafting = PoggyCrafting or {}
local PC = PoggyCrafting

local RESOURCE = GetCurrentResourceName()

-- ===========================================================================
--  Small helpers other server files use
-- ===========================================================================

--- One notification call for the whole script, so a style change is one edit.
--- `advanced` keeps the boxed icon notification the browser was built around.
function PC.Notify(src, text, kind)
    Poggy('notify.styled', {
        src      = src,
        style    = 'advanced',
        text     = text,
        dict     = 'BLIPS',
        icon     = kind == 'error' and 'blip_cash_bag' or 'blip_craft',
        color    = kind == 'error' and 'COLOR_RED' or 'COLOR_GREEN',
        duration = 4000,
    })
end

--- The item's label, preferring the config override. Falls back to the raw
--- name so a missing item still reads as something rather than nothing.
function PC.ItemLabel(name)
    if Config.ItemLabelOverrides and Config.ItemLabelOverrides[name] then
        return Config.ItemLabelOverrides[name]
    end
    local ok, info = Poggy('inv.itemInfo', { item = name })
    if ok and info and info.label and info.label ~= '' then return info.label end
    return name
end

--- Discord log. Silently does nothing without a webhook, which is how it
--- ships: Config.Webhook is blank.
function PC.Webhook(title, description)
    local url = Config.Webhook
    if not url or url == '' then return end
    if not url:find('^https://discord%.com/api/webhooks/') then return end

    PerformHttpRequest(url, function() end, 'POST', json.encode({
        username   = RESOURCE,
        avatar_url = (Config.WebhookAvatar ~= '' and Config.WebhookAvatar) or nil,
        embeds     = { {
            title       = title,
            description = description,
            color       = 3066993,
            footer      = { text = os.date('%Y-%m-%d %H:%M:%S') },
        } },
    }), { ['Content-Type'] = 'application/json' })
end

function PC.Debug(...)
    if Config.Debug then print(('^3[%s]^7'):format(RESOURCE), ...) end
end

-- ===========================================================================
--  Startup
-- ===========================================================================

CreateThread(function()
    -- Registers with poggy_core, runs sql/install.sql and any migrations, then
    -- waits until the framework has resolved. False means the running
    -- poggy_core is older than poggy_core_min and this script must do nothing.
    if not PoggyReady() then return end

    -- The campfire item. Registered through poggy_core so it survives an
    -- inventory restart, which a direct registration does not.
    if Config.CampfireItem then
        Poggy('inv.registerUsable', {
            item = Config.CampfireItem,
            fn   = function(data)
                local src = data and data.source
                if not src then return end
                Poggy('inv.remove', { src = src, item = Config.CampfireItem, qty = 1 })
                TriggerClientEvent('poggy_crafting:placeCampfire', src)
            end,
        })
    end

    -- A Job or Location that can never match (Job = 8) locks something for
    -- everyone without a word. One line per offender, every start, so it is
    -- found in the console rather than in a support ticket.
    for _, line in ipairs(PC.CheckRestrictions()) do
        print(('^3[%s] config:^7 %s'):format(RESOURCE, line))
    end

    -- /campfire places a free campfire, so it is staff only. The server
    -- decides; the client only builds the fire when told to.
    if Config.Commands.campfire then
        RegisterCommand(Config.Commands.campfire, function(src)
            if src == 0 then return end
            local okAdmin, admin = Poggy('perms.isAdmin', { src = src })
            if not okAdmin or not admin then
                return PC.Notify(src, PC.T('no_permission'), 'error')
            end
            TriggerClientEvent('poggy_crafting:placeCampfire', src)
        end, false)
    end

    PC.Debug(('ready with %d recipes in %d categories')
        :format(#Config.Crafting, #Config.Categories))
end)

-- ===========================================================================
--  Callbacks the browser opens on
-- ===========================================================================

-- Item labels and carry limits change only when the item database does, and
-- collecting them costs one lookup per distinct item in every recipe. Built
-- once, on the first request, and handed to everyone after that.
local itemCache = nil

local function buildItemCache()
    local labels, limits = {}, {}

    local function record(name)
        if not name or labels[name] ~= nil then return end
        if Config.ItemLabelOverrides and Config.ItemLabelOverrides[name] then
            labels[name] = Config.ItemLabelOverrides[name]
        end
        local ok, info = Poggy('inv.itemInfo', { item = name })
        if ok and info then
            labels[name] = labels[name] or info.label or name
            limits[name] = tonumber(info.limit) or 0
        else
            labels[name] = labels[name] or name
            limits[name] = 0
        end
    end

    for _, recipe in ipairs(Config.Crafting) do
        for _, item in ipairs(recipe.Items or {}) do
            record(item.name)
            for _, alt in ipairs(item.AltNames or {}) do record(alt) end
        end
        for _, reward in ipairs(recipe.Reward or {}) do record(reward.name) end
    end

    local okBase, imageBase = Poggy('inv.imageBase', {})

    return {
        labels    = labels,
        limits    = limits,
        imageBase = okBase and imageBase or nil,
    }
end

CreateThread(function()
    if not PoggyReady() then return end

    -- poggy_core callbacks RETURN their answer; there is no callback argument.
    -- The handler runs on its own thread, so a yielding call inside one is fine.

    -- The player's job name, read fresh. The client asks for this every time
    -- the browser opens so a job change from another script (a multijob, a
    -- duty toggle) is picked up without a relog.
    Poggy('callback.register', {
        name = 'poggy_crafting:job',
        fn   = function(src)
            local ok, job = Poggy('job.get', { src = src })
            return ok and job and job.name or nil
        end,
    })

    -- Everything the browser has to know about items: label, carry limit, and
    -- the URL its icons live under. Collected from the recipes rather than the
    -- whole item registry, so this stays small however big the database is.
    Poggy('callback.register', {
        name = 'poggy_crafting:items',
        fn   = function()
            itemCache = itemCache or buildItemCache()
            return itemCache
        end,
    })

    -- What the player is carrying, name -> count, summed across stacks.
    Poggy('callback.register', {
        name = 'poggy_crafting:inventory',
        fn   = function(src)
            return PC.InventoryCounts(src)
        end,
    })
end)

--- name -> total count. The browser does all its "can I make this" maths on
--- this shape, so it is built in one place.
function PC.InventoryCounts(src)
    local counts = {}
    local ok, items = Poggy('inv.get', { src = src })
    if not ok or type(items) ~= 'table' then return counts end
    for _, item in pairs(items) do
        if item.name then
            counts[item.name] = (counts[item.name] or 0) + (tonumber(item.amount) or 0)
        end
    end
    return counts
end

--- Push a fresh inventory to one player's browser.
function PC.PushInventory(src)
    TriggerClientEvent('poggy_crafting:inventory', src, PC.InventoryCounts(src))
end
