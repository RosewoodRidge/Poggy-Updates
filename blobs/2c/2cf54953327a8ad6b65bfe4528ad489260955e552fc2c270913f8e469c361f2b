--[[===========================================================================
    poggy_crafting · config/recipes.lua
    ---------------------------------------------------------------------------
    Config.Crafting -- every recipe in the script.

    THESE ARE EXAMPLES. Thirteen of them, chosen to demonstrate one feature
    each, using item names that ship with a stock VORP inventory. Delete the
    lot and write your own; nothing else in the script refers to them by name.

    ---------------------------------------------------------------------------
    A recipe, field by field
    ---------------------------------------------------------------------------

    Text            REQUIRED. The recipe's identity. The server matches the
                    craft request on this string, the shopping list stores it,
                    and it must be unique. Changing it orphans shopping-list
                    entries, so treat it as an id, not a label.

    SubText         A short line under the title in the browser.
    Desc            The long description. Write the recipe out in words.

    Items           REQUIRED. What it consumes.
        name        the inventory item name.
        count       how many, per craft.
        take        false leaves the item in the inventory -- a tool, not an
                    ingredient. Defaults to true.
        canUseDecay a number 0-100. The item is only accepted when its
                    condition is at least this. Omit to accept any condition.
                    VORP only: on frameworks with no item decay the check
                    passes and the item is accepted.
        AltNames    other items that satisfy this slot. One recipe can then
                    take "any fish" or "any pelt". The player's own stock is
                    counted across the primary name and every alt.

    Reward          REQUIRED. What it produces.
        name        the item to give. Omit `name` in UseCurrencyMode and the
                    entry becomes money instead (see Bottle Deposit below).
        count       how many, per craft.

    AltRewards      Optional. Keyed by the alt ingredient actually consumed,
                    each holding a full Reward list that replaces the default.
                    Lets one recipe pay out by size or quality.

    Type            'item' or 'weapon'. A weapon reward is created as a weapon,
                    with its own serial, not as a stackable item.

    Category        REQUIRED. A Config.Categories ident.
    Job             0 for anyone, or a list of job names.
    Location        0 for anywhere, or a list of Config.Locations ids and
                    lower-cased Config.CraftingProps titles.

    Animation       A Config.Animations key. Defaults to 'craft'.
    CraftTime       ms, overriding Config.CraftTime for this recipe.

    UseCurrencyMode true makes money a first-class part of the recipe.
    CurrencyType    0 money, 1 gold, 2 rol.

    skillcheck      N rounds instead of the progress bar.
    jobSkillcheck   N HARD rounds for a player who lacks `Job`, instead of
                    refusing them. Having the job skips the skillcheck.
    useHardSkillcheck  use the Hard difficulty for `skillcheck` too.
    explodeOnFail   a missed round detonates on the crafter.
    catchFireOnFail a missed round sets the crafter alight.

    See config/config.lua section 9 for how the skillcheck reward is worked out.
===========================================================================]]--

Config.Crafting = {

    -- ═══════════════════════════════════════════════════════════════════
    --  FOOD & COOKING
    -- ═══════════════════════════════════════════════════════════════════

    -- The plainest recipe there is: two items in, one out.
    {
        Text     = 'Seasoned Small Game',
        SubText  = 'Cooked over a fire',
        Desc     = 'Salt and a spit. Small game keeps a day longer this way.',
        Items    = {
            { name = 'game', count = 1, AltNames = { 'bird' } },
            { name = 'salt', count = 1 },
        },
        Reward   = {
            { name = 'cookedsmallgame', count = 2 },
        },
        Type      = 'item',
        Category  = 'food',
        Job       = 0,
        Location  = 0,
        Animation = 'knifecooking',
    },

    -- Location-locked: only works at a campfire or an oven, because the
    -- Location list matches Config.CraftingProps titles, lower-cased.
    {
        Text     = 'Wheat Flour',
        SubText  = 'Ground by hand',
        Desc     = 'Three heads of wheat, milled down to two measures of flour.',
        Items    = {
            { name = 'wheat', count = 3 },
        },
        Reward   = {
            { name = 'flour', count = 2 },
        },
        Type      = 'item',
        Category  = 'food',
        Job       = 0,
        Location  = { 'campfire', 'oven', 'workbench_general' },
        Animation = 'craft',
    },

    -- Five ingredients, and a tool that is NOT consumed (`take = false`).
    {
        Text     = 'Apple Pie',
        SubText  = 'Feeds four',
        Desc     = 'Apple, water, sugar, an egg and a measure of flour. The pot stays with you.',
        Items    = {
            { name = 'apple', count = 1 },
            { name = 'water', count = 1 },
            { name = 'sugar', count = 1 },
            { name = 'egg',   count = 1 },
            { name = 'flour', count = 1 },
            { name = 'pot',   count = 1, take = false },
        },
        Reward   = {
            { name = 'consumable_applepie', count = 1 },
        },
        Type      = 'item',
        Category  = 'food',
        Job       = 0,
        Location  = { 'oven' },
        Animation = 'craft',
        CraftTime = 8000,
    },

    -- AltNames and AltRewards together: one recipe takes any of three fish and
    -- pays out by size. Extend the two tables in step for a full fish list.
    {
        Text     = 'Fish Fillet',
        SubText  = 'Any fish',
        Desc     = 'Knife any caught fish into fillets. A bigger fish gives more.',
        Items    = {
            {
                name     = 'fish',
                count    = 1,
                AltNames = {
                    'a_c_fishbluegil_01_sm',
                    'a_c_fishperch_01_ms',
                    'a_c_fishlargemouthbass_01_lg',
                },
            },
        },
        Reward     = {
            { name = 'fish_filet', count = 4 },
        },
        AltRewards = {
            ['a_c_fishbluegil_01_sm']       = { { name = 'fish_filet', count = 2 } },
            ['a_c_fishperch_01_ms']         = { { name = 'fish_filet', count = 4 } },
            ['a_c_fishlargemouthbass_01_lg'] = { { name = 'fish_filet', count = 6 } },
        },
        Type      = 'item',
        Category  = 'food',
        Job       = 0,
        Location  = 0,
        Animation = 'knifecooking',
    },

    -- ═══════════════════════════════════════════════════════════════════
    --  SUPPLIES
    -- ═══════════════════════════════════════════════════════════════════

    -- One in, many out.
    {
        Text     = 'Bread Bait',
        SubText  = 'Five casts',
        Desc     = 'A loaf, torn up and rolled. Fish are not fussy.',
        Items    = {
            { name = 'bread', count = 1 },
        },
        Reward   = {
            { name = 'bait_bread', count = 5 },
        },
        Type     = 'item',
        Category = 'supplies',
        Job      = 0,
        Location = 0,
    },

    -- canUseDecay: cloth in poor condition will not do for a bandage.
    {
        Text     = 'Bandage',
        SubText  = 'Clean cloth only',
        Desc     = 'Two lengths of clean cloth, boiled. Anything grubbier is no use.',
        Items    = {
            { name = 'cloth', count = 2, canUseDecay = 60 },
            { name = 'water', count = 1 },
        },
        Reward   = {
            { name = 'bandage', count = 2 },
        },
        Type     = 'item',
        Category = 'supplies',
        Job      = 0,
        Location = 0,
    },

    -- A chain link: this feeds the Lantern below, and the browser draws the
    -- two as a tree.
    {
        Text     = 'Tallow',
        SubText  = 'Rendered fat',
        Desc     = 'Animal fat, rendered slowly. Burns clean enough for a lamp.',
        Items    = {
            { name = 'fat', count = 3 },
        },
        Reward   = {
            { name = 'tallow', count = 2 },
        },
        Type      = 'item',
        Category  = 'supplies',
        Job       = 0,
        Location  = { 'campfire', 'oven' },
        Animation = 'spindlecook',
    },

    {
        Text     = 'Lantern',
        SubText  = 'Burns for hours',
        Desc     = 'Glass, a strip of iron and two measures of tallow.',
        Items    = {
            { name = 'glass',     count = 1 },
            { name = 'ironingot', count = 1 },
            { name = 'tallow',    count = 2 },
        },
        Reward   = {
            { name = 'lantern', count = 1 },
        },
        Type     = 'item',
        Category = 'supplies',
        Job      = 0,
        Location = 0,
    },

    -- UseCurrencyMode with a NEGATIVE reward entry and no `name`: the player
    -- pays for the craft. A positive one pays them instead.
    {
        Text            = 'Bottle Deposit',
        SubText         = 'Costs $2.00',
        Desc            = 'Hand over two dollars at the counter and take five clean bottles.',
        Items           = {},
        Reward          = {
            { count = -2 },                    -- no name = money, negative = a cost
            { name = 'bottle', count = 5 },
        },
        Type            = 'item',
        Category        = 'supplies',
        Job             = 0,
        Location        = { 'workbench_general' },
        UseCurrencyMode = true,
        CurrencyType    = 0,
    },

    -- ═══════════════════════════════════════════════════════════════════
    --  BLACKSMITHING  -- the category is job-locked, so these are too
    -- ═══════════════════════════════════════════════════════════════════

    {
        Text     = 'Iron Ingot',
        SubText  = 'Smelted at the forge',
        Desc     = 'Three of ore and a measure of coal, held in the fire until it runs.',
        Items    = {
            { name = 'ironore', count = 3 },
            { name = 'coal',    count = 1 },
        },
        Reward   = {
            { name = 'ironingot', count = 1 },
        },
        Type      = 'item',
        Category  = 'smithing',
        Job       = { 'blacksmith' },
        Location  = { 'smithy_valentine' },
        CraftTime = 10000,
    },

    -- An ordinary skillcheck: three rounds, part marks for a partial pass.
    {
        Text       = 'Horseshoes',
        SubText    = 'Three rounds at the anvil',
        Desc       = 'Two ingots drawn out over the horn. Strike true or waste the iron.',
        Items      = {
            { name = 'ironingot', count = 2 },
        },
        Reward     = {
            { name = 'horseshoes', count = 4 },
        },
        Type       = 'item',
        Category   = 'smithing',
        Job        = { 'blacksmith' },
        Location   = { 'smithy_valentine' },
        skillcheck = 3,
    },

    -- Type = 'weapon': the reward is created as a real weapon with a serial.
    {
        Text     = 'Hunting Knife',
        SubText  = 'A proper blade',
        Desc     = 'One ingot, one handle of hardwood, and an evening at the grindstone.',
        Items    = {
            { name = 'ironingot', count = 1 },
            { name = 'hwood',     count = 1 },
        },
        Reward   = {
            { name = 'WEAPON_MELEE_KNIFE', count = 1 },
        },
        Type      = 'weapon',
        Category  = 'smithing',
        Job       = { 'blacksmith' },
        Location  = { 'smithy_valentine' },
        CraftTime = 12000,
    },

    -- ═══════════════════════════════════════════════════════════════════
    --  MOONSHINE  -- open category, job-locked recipes with a way through
    -- ═══════════════════════════════════════════════════════════════════

    -- jobSkillcheck: a distiller makes this with no skillcheck at all. Anyone
    -- else is let through on four HARD rounds rather than being refused.
    {
        Text          = 'Corn Mash',
        SubText       = 'Distillers craft this freely',
        Desc          = 'Five ears of corn and two of water, left to sour. Anyone may try; only a distiller finds it easy.',
        Items         = {
            { name = 'corn',  count = 5 },
            { name = 'water', count = 2 },
        },
        Reward        = {
            { name = 'mash', count = 2 },
        },
        Type          = 'item',
        Category      = 'moonshine',
        Job           = { 'distiller' },
        Location      = { 'still_lemoyne' },
        jobSkillcheck = 4,
    },

    -- The dangerous one: hard difficulty for everybody, and a missed round
    -- sets the still -- and the crafter -- on fire.
    {
        Text              = 'Bottled Moonshine',
        SubText           = 'Two rounds. Mind the vapour.',
        Desc              = 'Mash and a clean bottle, run off the still. Let it catch and you go up with it.',
        Items             = {
            { name = 'mash',   count = 2 },
            { name = 'bottle', count = 1 },
        },
        Reward            = {
            { name = 'moonshine', count = 1 },
        },
        Type              = 'item',
        Category          = 'moonshine',
        Job               = 0,
        Location          = { 'still_lemoyne' },
        skillcheck        = 2,
        useHardSkillcheck = true,
        catchFireOnFail   = true,
    },
}
