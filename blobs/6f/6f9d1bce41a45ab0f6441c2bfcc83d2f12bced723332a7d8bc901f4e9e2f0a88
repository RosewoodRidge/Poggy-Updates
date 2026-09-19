--[[===========================================================================
    poggy_crafting · server/icons.lua
    ---------------------------------------------------------------------------
    Hide recipes the browser cannot draw.

    An item with no PNG in the inventory's icon folder falls back to the
    placeholder. One or two of those is untidy; a page full of them looks
    broken. So at startup the server asks the filesystem, once, whether every
    item every recipe touches actually has an icon, and hides the recipes that
    come up short.

    Then it walks the chain. A recipe can pass the icon check itself and still
    be pointless: if every recipe that produced one of its ingredients has been
    hidden, nothing can make that ingredient any more, so the recipe goes too.
    That repeats until nothing changes.

    Raw materials are deliberately safe from the chain walk. An item that no
    recipe in the config produces is assumed to come from the world -- mined,
    grown, hunted, looted, bought -- so a recipe is never hidden merely for
    depending on one.

    The result is PC.Hidden, keyed by recipe Text. shared/recipes.lua reads it
    in PC.FindRecipe, so a hidden recipe cannot be crafted even by a client
    that asks for it by name. The client fetches the same set when it first
    opens the browser and never renders them.

    Off by default; on in one line: Config.HideRecipesWithoutIcons = true.
    Icons are found through poggy_core (inv.itemInfo with checkImages), so it
    works on VORP, RSG and QBR alike.
===========================================================================]]--

local PC = PoggyCrafting

-- ---------------------------------------------------------------------------
--  Does this item have an icon?
-- ---------------------------------------------------------------------------

-- Where the icons are. By default poggy_core answers for the framework (VORP,
-- RSG, QBR each keep them somewhere else, and RSG and QBR name files by an
-- image field). Config.IconResource + Config.IconPath override it, for icons
-- kept in another resource.
local iconCache = {}   -- item name -> true / false
local iconFile  = {}   -- item name -> "<resource>/<path>", when known

local function overridden()
    return type(Config.IconResource) == 'string' and Config.IconResource ~= ''
end

local function hasIcon(name)
    if type(name) ~= 'string' or name == '' then return false end
    local cached = iconCache[name]
    if cached ~= nil then return cached end

    local found = false
    if overridden() then
        local path = (Config.IconPath or 'html/img/items/%s.png'):format(name)
        local file = LoadResourceFile(Config.IconResource, path)
        found = file ~= nil and #file > 0
        iconFile[name] = ('%s/%s'):format(Config.IconResource, path)
    else
        local ok, info = Poggy('inv.itemInfo', { item = name, checkImages = true })
        found = ok and type(info) == 'table' and info.hasImage == true
        if ok and type(info) == 'table' and type(info.image) == 'string' then
            iconFile[name] = (info.image:gsub('^nui://', ''))
        end
    end

    iconCache[name] = found
    return found
end

-- ---------------------------------------------------------------------------
--  Work out what to hide
-- ---------------------------------------------------------------------------

--- Every item name a recipe touches, and separately just its ingredients.
local function namesOf(recipe)
    local all, ingredients = {}, {}

    for _, item in ipairs(recipe.Items or {}) do
        if item.name then
            all[item.name] = true
            ingredients[item.name] = true
        end
        for _, alt in ipairs(item.AltNames or {}) do
            all[alt] = true
            ingredients[alt] = true
        end
    end

    for _, reward in ipairs(recipe.Reward or {}) do
        if reward.name then all[reward.name] = true end
    end

    -- AltRewards pay out instead of Reward, so they have to be drawable too.
    for _, list in pairs(recipe.AltRewards or {}) do
        for _, reward in ipairs(list) do
            if reward.name then all[reward.name] = true end
        end
    end

    return all, ingredients
end

local function build()
    local hidden = {}

    if not Config.HideRecipesWithoutIcons then
        PC.Hidden = hidden
        return 0, 0, 0
    end

    local allNames, ingredientsOf, producedBy = {}, {}, {}
    local missing = {}

    for _, recipe in ipairs(Config.Crafting) do
        local all, ingredients = namesOf(recipe)
        allNames[recipe.Text]      = all
        ingredientsOf[recipe.Text] = ingredients

        for _, reward in ipairs(recipe.Reward or {}) do
            if reward.name then
                producedBy[reward.name] = producedBy[reward.name] or {}
                producedBy[reward.name][recipe.Text] = true
            end
        end
        for _, list in pairs(recipe.AltRewards or {}) do
            for _, reward in ipairs(list) do
                if reward.name then
                    producedBy[reward.name] = producedBy[reward.name] or {}
                    producedBy[reward.name][recipe.Text] = true
                end
            end
        end
    end

    -- Pass one: anything the browser cannot draw.
    local noIcon = 0
    for _, recipe in ipairs(Config.Crafting) do
        for name in pairs(allNames[recipe.Text]) do
            if not hasIcon(name) then
                if not missing[name] then
                    missing[name] = true
                    noIcon = noIcon + 1
                end
                hidden[recipe.Text] = true
            end
        end
    end

    local direct = 0
    for _ in pairs(hidden) do direct = direct + 1 end

    -- Pass two onwards: ingredients nothing can make any more.
    if Config.HideBrokenChains then
        local changed = true
        while changed do
            changed = false
            for _, recipe in ipairs(Config.Crafting) do
                if not hidden[recipe.Text] then
                    for name in pairs(ingredientsOf[recipe.Text]) do
                        local makers = producedBy[name]
                        if makers then
                            local anyLeft = false
                            for text in pairs(makers) do
                                if not hidden[text] then anyLeft = true break end
                            end
                            if not anyLeft then
                                hidden[recipe.Text] = true
                                changed = true
                                break
                            end
                        end
                    end
                end
            end
        end
    end

    PC.Hidden = hidden

    local total = 0
    for _ in pairs(hidden) do total = total + 1 end
    return total, direct, noIcon
end

-- ---------------------------------------------------------------------------
--  Startup
-- ---------------------------------------------------------------------------

CreateThread(function()
    if not PoggyReady() then return end

    local hiddenCount, direct, missingIcons = build()
    local visible = #Config.Crafting - hiddenCount

    if hiddenCount > 0 then
        PC.Debug(('%d of %d recipes hidden: %d touch one of %d items with no icon, %d more further down the chain. %d recipes visible.')
            :format(hiddenCount, #Config.Crafting, direct, missingIcons,
                    hiddenCount - direct, visible))
    else
        PC.Debug(('every one of %d recipes has icons for all its items')
            :format(#Config.Crafting))
    end

    -- The client asks for this once, the first time it opens the browser.
    Poggy('callback.register', {
        name = 'poggy_crafting:hidden',
        fn   = function()
            return PC.Hidden
        end,
    })
end)

--- Re-run the check and print the result. poggy_core reads each icon once, and
--- the inventory only serves the files it found when it started, so a PNG
--- dropped in while the server runs shows after the next server restart; with
--- Config.IconResource set, the file is looked at again here.
RegisterCommand('poggycrafting_icons', function(source)
    if source ~= 0 then return end   -- console only
    CreateThread(function()
        iconCache, iconFile = {}, {}
        local hiddenCount, direct, missingIcons = build()
        print(('[poggy_crafting] %d of %d recipes hidden (%d direct, %d items with no icon)')
            :format(hiddenCount, #Config.Crafting, direct, missingIcons))
    end)
end, true)

--[[===========================================================================
    Why each recipe is hidden, for the Poggy hub (/poggy)
    ---------------------------------------------------------------------------
    PC.Hidden says WHICH recipes are hidden; an owner looking at the recipe
    list in the hub also needs to know WHY, and what to do about it. So
    poggy_core's settings hub calls the export below when the owner opens
    this script (docs/hub.json names it: "diagnostics": "HubDiagnostics"),
    and marks each hidden recipe's row with the reason:

      missing_icon  the recipe itself uses an item with no icon. Lists every
                    such item and the exact file each one needs: the one
                    poggy_core names for the framework, or the one built from
                    Config.IconResource and Config.IconPath when those are set.
      chain         the recipe has every icon it needs, but one of its
                    ingredients can no longer be made: every recipe that
                    produced it is hidden. Lists those ingredients.

    Nothing here changes what is hidden. The reasons are worked out from
    PC.Hidden and the icon answers the startup check already made, at the
    moment the hub asks, so they always describe the config the server is
    running (they change after the owner saves and restarts the script, or
    runs poggycrafting_icons).

    Rows are named the way the hub names them: Config.Crafting[<n>], n being
    the recipe's 1-based position in Config.Crafting.
===========================================================================]]--

--- The file an item's icon should be at, as "<resource>/<path>". An item the
--- inventory does not know has no file to name.
local function iconFileOf(name)
    if iconFile[name] then return iconFile[name] end
    if overridden() then
        return ('%s/%s'):format(Config.IconResource, (Config.IconPath or 'html/img/items/%s.png'):format(name))
    end
    return ('%s (not an item on this server)'):format(name)
end

--- "a", "a and b", "a, b and c".
local function joinNames(names)
    if #names <= 1 then return names[1] or '' end
    return table.concat(names, ', ', 1, #names - 1) .. ' and ' .. names[#names]
end

--- The sorted keys of a set.
local function sortedKeys(set)
    local out = {}
    for k in pairs(set) do out[#out + 1] = k end
    table.sort(out)
    return out
end

--- { summary, rows = { ['Config.Crafting[n]'] = { entry } } } for the hub.
local function hubDiagnostics()
    local total = #Config.Crafting

    if not Config.HideRecipesWithoutIcons then
        return {
            summary = ('Recipes are never hidden for missing icons (Config.HideRecipesWithoutIcons is off); all %d are shown in game.')
                :format(total),
            rows = {},
        }
    end
    if PC.Hidden == nil then
        return { summary = 'The icon check has not run yet: the script is still starting.', rows = {} }
    end

    -- Who makes what, exactly as build() works it out.
    local producedBy = {}
    for _, recipe in ipairs(Config.Crafting) do
        local function made(reward)
            if reward.name then
                producedBy[reward.name] = producedBy[reward.name] or {}
                producedBy[reward.name][recipe.Text] = true
            end
        end
        for _, reward in ipairs(recipe.Reward or {}) do made(reward) end
        for _, list in pairs(recipe.AltRewards or {}) do
            for _, reward in ipairs(list) do made(reward) end
        end
    end

    local rows = {}
    local hiddenCount, direct, chain = 0, 0, 0
    local noIcon = {}

    for index, recipe in ipairs(Config.Crafting) do
        if PC.Hidden[recipe.Text] then
            hiddenCount = hiddenCount + 1
            local all, ingredients = namesOf(recipe)

            local missing = {}
            for name in pairs(all) do
                if not hasIcon(name) then missing[name] = true; noIcon[name] = true end
            end
            missing = sortedKeys(missing)

            local entry
            if #missing > 0 then
                direct = direct + 1
                local files = {}
                for i, name in ipairs(missing) do files[i] = iconFileOf(name) end
                entry = {
                    level   = 'warning',
                    code    = 'missing_icon',
                    message = ('Hidden in game: no icon for %s'):format(joinNames(missing)),
                    items   = missing,
                    files   = files,
                }
            else
                chain = chain + 1
                local dead = {}
                for name in pairs(ingredients) do
                    local makers = producedBy[name]
                    if makers then
                        local anyLeft = false
                        for text in pairs(makers) do
                            if not PC.Hidden[text] then anyLeft = true break end
                        end
                        if not anyLeft then dead[name] = true end
                    end
                end
                dead = sortedKeys(dead)
                entry = {
                    level   = 'warning',
                    code    = 'chain',
                    message = #dead > 0
                        and ('Hidden in game: nothing can make %s any more (every recipe for %s is hidden)')
                            :format(joinNames(dead), #dead == 1 and 'it' or 'them')
                        or 'Hidden in game: a recipe it depends on is hidden',
                    items   = dead,
                }
            end
            rows[('Config.Crafting[%d]'):format(index)] = { entry }
        end
    end

    local summary
    if hiddenCount == 0 then
        summary = ('Every one of %d recipes has icons for all its items; none are hidden in game.'):format(total)
    else
        summary = ('%d of %d recipes are hidden in game: %d use one of %d items with no icon, %d more further down the chain.')
            :format(hiddenCount, total, direct, #sortedKeys(noIcon), chain)
    end
    return { summary = summary, rows = rows }
end

exports('HubDiagnostics', hubDiagnostics)
