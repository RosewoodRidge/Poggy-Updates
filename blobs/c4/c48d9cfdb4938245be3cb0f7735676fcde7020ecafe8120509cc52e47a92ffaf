--[[===========================================================================
    poggy_crafting · shared/recipes.lua
    ---------------------------------------------------------------------------
    The small amount of logic both sides need: translation lookup, and the
    questions "may this player use this recipe here?" asked in exactly one
    place so the client's greying-out and the server's refusal can never
    disagree.
===========================================================================]]--

PoggyCrafting = PoggyCrafting or {}
local PC = PoggyCrafting

--- Translate. Missing keys come back as the key itself, which is ugly on
--- screen but tells you which line of translations.lua to add.
function PC.T(key, ...)
    local text = PC.Locale and PC.Locale[key]
    if not text then return tostring(key) end
    if select('#', ...) == 0 then return text end
    local ok, out = pcall(string.format, text, ...)
    return ok and out or text
end

--- Is `value` inside `list`? A list of 0 (or nil) means "no restriction", which
--- is how every Job and Location field in the config says "anyone, anywhere".
function PC.Allows(list, value)
    if list == nil or list == 0 then return true end
    if type(list) ~= 'table' then return list == value end
    for _, v in pairs(list) do
        if v == value then return true end
    end
    return false
end

--- The recipe with this Text (and Category, when given), or nil. The client
--- sends both and the server looks the real recipe up again, so a tampered
--- payload can only ever name a recipe that exists.
---
--- Text alone is not unique: a server may have a blacksmith "Shovel" and a
--- lumber "Shovel". Matching on Text only handed every craft of the second one
--- the first one's recipe, and its job refused them (2.0.15).
function PC.FindRecipe(text, category)
    if type(text) ~= 'string' then return nil end
    -- A hidden recipe is not findable, so the server refuses to craft it even
    -- if a client asks for it by name.
    if PC.Hidden and PC.Hidden[text] then return nil end
    if type(category) ~= 'string' then category = nil end
    for _, recipe in ipairs(Config.Crafting) do
        if recipe.Text == text and (not category or recipe.Category == category) then
            return recipe
        end
    end
    return nil
end

--- The category table for an ident, or nil.
function PC.FindCategory(ident)
    for _, cat in ipairs(Config.Categories) do
        if cat.ident == ident then return cat end
    end
    return nil
end

--- May `job` craft `recipe` at `locationId`?
---
--- Returns allowed, needsJobSkillcheck. The second value is the whole point of
--- this function: a player without the job is not simply refused when the
--- recipe offers `jobSkillcheck`, they are let through to earn it.
function PC.CanCraft(recipe, job, locationId)
    if not recipe then return false, false end

    local cat = PC.FindCategory(recipe.Category)
    if cat then
        if not PC.Allows(cat.Job, job) then return false, false end
        if not PC.Allows(cat.Location, locationId) then return false, false end
    end

    if not PC.Allows(recipe.Location, locationId) then return false, false end

    if PC.Allows(recipe.Job, job) then return true, false end

    local bypass = tonumber(recipe.jobSkillcheck) or 0
    if bypass > 0 and Config.Skillcheck and Config.Skillcheck.enabled then
        return true, true
    end

    return false, false
end

--- Why can `job` not craft `recipe` at `locationId`? nil when they can.
---
--- Returns reason, detail. The checks run in CanCraft's order, so the reason
--- is the first one that fails:
---   'category_job'    the category's Job list       detail = that list
---   'category_place'  the category's Location list  detail = that list
---   'recipe_place'    the recipe's Location list    detail = that list
---   'recipe_job'      the recipe's Job list         detail = that list
---
--- The server's refusal and the browser's grey badge both come from this, so
--- they always agree. Naming the wrong check costs real time: a cafe worker
--- told "you do not know how to make that" while standing in the wrong place
--- went looking for a job bug.
function PC.WhyLocked(recipe, job, locationId)
    if not recipe then return 'unknown', nil end

    local cat = PC.FindCategory(recipe.Category)
    if cat then
        if not PC.Allows(cat.Job, job) then return 'category_job', cat.Job end
        if not PC.Allows(cat.Location, locationId) then return 'category_place', cat.Location end
    end

    if not PC.Allows(recipe.Location, locationId) then return 'recipe_place', recipe.Location end
    if PC.Allows(recipe.Job, job) then return nil, nil end

    local bypass = tonumber(recipe.jobSkillcheck) or 0
    if bypass > 0 and Config.Skillcheck and Config.Skillcheck.enabled then return nil, nil end

    return 'recipe_job', recipe.Job
end

--- Place id -> what the player reads: a bench's name, or a prop group's
--- title. Location lists hold ids; nobody should have to read 'still_lemoyne'.
function PC.PlaceNameMap()
    local names = {}
    for _, loc in ipairs(Config.Locations or {}) do
        if loc.id then names[loc.id] = loc.name or loc.id end
    end
    for _, group in ipairs(Config.CraftingProps or {}) do
        if group.title then names[group.title:lower()] = group.title end
    end
    return names
end

--- A Location list as readable text: "Blacksmith Anvil, Forge".
function PC.PlaceNames(list)
    if type(list) ~= 'table' then return '' end
    local names, out = PC.PlaceNameMap(), {}
    for _, id in ipairs(list) do out[#out + 1] = names[id] or tostring(id) end
    return table.concat(out, ', ')
end

--- Job and Location fields that can never match anything: a number other
--- than 0 (Job = 8), or a list holding something that is not a name. Each
--- one locks its recipe, category or bench for everyone, silently, so the
--- server names them at start. Returns an array of lines.
function PC.CheckRestrictions()
    local problems = {}

    local function check(where, field, v)
        if v == nil or v == 0 then return end
        if type(v) == 'number' then
            problems[#problems + 1] = ('%s: %s = %s matches nothing. Use 0 for "any", or a list of names.')
                :format(where, field, tostring(v))
        elseif type(v) ~= 'table' then
            problems[#problems + 1] = ('%s: %s = %s is not a list.'):format(where, field, tostring(v))
        else
            for _, x in pairs(v) do
                if type(x) ~= 'string' and type(x) ~= 'number' then
                    problems[#problems + 1] = ('%s: %s holds a %s, not a name.'):format(where, field, type(x))
                    break
                end
            end
        end
    end

    check('Config.CampfireJobLock', 'CampfireJobLock', Config.CampfireJobLock)
    for _, cat in ipairs(Config.Categories or {}) do
        local where = ('category %q'):format(tostring(cat.ident))
        check(where, 'Job', cat.Job)
        check(where, 'Location', cat.Location)
    end
    for _, loc in ipairs(Config.Locations or {}) do
        local where = ('bench %q'):format(tostring(loc.id))
        check(where, 'Job', loc.Job)
        check(where, 'Categories', loc.Categories)
    end
    local seen = {}
    for _, recipe in ipairs(Config.Crafting or {}) do
        local where = ('recipe %q'):format(tostring(recipe.Text))
        check(where, 'Job', recipe.Job)
        check(where, 'Location', recipe.Location)
        -- Name + category is what a craft asks for; two recipes sharing both
        -- are one recipe to the server, and the second can never be made.
        local key = tostring(recipe.Text) .. '\0' .. tostring(recipe.Category)
        if seen[key] then
            problems[#problems + 1] = ('%s: a second recipe with this name in category %q. Only the first can be crafted; rename one.')
                :format(where, tostring(recipe.Category))
        end
        seen[key] = true
    end
    return problems
end

--- How many rounds, and which difficulty profile, this craft asks for.
--- Returns reps, profileName ('Normal' | 'JobBypass' | 'Hard'), or 0 when the
--- recipe uses the ordinary progress bar.
function PC.SkillcheckFor(recipe, needsJobSkillcheck, quantity)
    if not (Config.Skillcheck and Config.Skillcheck.enabled) then return 0, nil end

    local reps, profile
    if needsJobSkillcheck then
        reps    = tonumber(recipe.jobSkillcheck) or 0
        profile = 'JobBypass'
    else
        reps    = tonumber(recipe.skillcheck) or 0
        profile = recipe.useHardSkillcheck and 'Hard' or 'Normal'
    end

    if reps <= 0 then return 0, nil end

    -- Crafting a batch is harder than crafting one.
    local per = tonumber(Config.Skillcheck.RepsPerExtra) or 0
    if per > 0 then
        reps = reps + math.floor((tonumber(quantity) or 1) / per)
    end

    return reps, profile
end

-- ===========================================================================
--  HIDDEN RECIPES
-- ===========================================================================

-- Filled in on the server by server/icons.lua, and sent to each client when
-- it first opens the browser. Keyed by recipe Text, the recipe's id.
--
-- Empty on both sides until that happens, which is the safe default: nothing
-- is hidden unless something positively said to hide it.
PC.Hidden = PC.Hidden or {}

--- Is this recipe hidden? Takes the recipe or its Text.
function PC.IsHidden(recipe)
    local text = type(recipe) == 'table' and recipe.Text or recipe
    if type(text) ~= 'string' then return false end
    return PC.Hidden[text] == true
end

--- Every recipe the player should be able to see. The browser is built from
--- this rather than Config.Crafting, so a hidden recipe is not merely greyed
--- out, it is not sent to the page at all.
function PC.VisibleRecipes()
    if next(PC.Hidden) == nil then return Config.Crafting end
    local out = {}
    for _, recipe in ipairs(Config.Crafting) do
        if not PC.Hidden[recipe.Text] then out[#out + 1] = recipe end
    end
    return out
end
