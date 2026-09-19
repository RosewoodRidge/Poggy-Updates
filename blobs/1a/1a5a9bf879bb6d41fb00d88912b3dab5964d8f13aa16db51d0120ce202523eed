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

--- The recipe with this Text, or nil. Text is the recipe's id: the client
--- sends it and the server looks the real recipe up again, so a tampered
--- payload can only ever name a recipe that exists.
function PC.FindRecipe(text)
    if type(text) ~= 'string' then return nil end
    -- A hidden recipe is not findable, so the server refuses to craft it even
    -- if a client asks for it by name.
    if PC.Hidden and PC.Hidden[text] then return nil end
    for _, recipe in ipairs(Config.Crafting) do
        if recipe.Text == text then return recipe end
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
