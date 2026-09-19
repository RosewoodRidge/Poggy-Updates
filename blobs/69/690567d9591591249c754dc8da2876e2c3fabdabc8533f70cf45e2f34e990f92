--[[===========================================================================
    poggy_crafting · config/item_sources.lua
    ---------------------------------------------------------------------------
    Config.ItemSources -- the "where do I get this?" tooltip.

    Hovering an ingredient anywhere in the browser shows a card with the item's
    icon, a source badge and one line of explanation. An item that is not listed
    here shows "Unknown" and the line in translations.lua, which is the honest
    answer and costs you nothing -- so filling this in is entirely optional.

    It is, though, the single highest-value thing you can fill in. It is the
    difference between a player asking in Discord where to find sulfur and the
    game telling them.

        ['item_name'] = { source = 'Badge', description = 'One sentence.' }

    source       a short badge. Reuse the same wording across related items --
                 the browser colours the badge from the text, so 'Hunting' on
                 twenty items looks like one system.
    description  one line, no full stop needed. Say the verb: where they go and
                 what they do.

    The entries below cover the example recipes in config/recipes.lua and
    nothing else. Group yours by the job or activity that yields them.
===========================================================================]]--

Config.ItemSources = {

    -- ── Hunting ─────────────────────────────────────────────────────────
    ['game']            = { source = 'Hunting',  description = 'Skinned from rabbits, squirrels and other small game' },
    ['bird']            = { source = 'Hunting',  description = 'Skinned from any bird you bring down' },
    ['fat']             = { source = 'Hunting',  description = 'Rendered from larger carcasses when you skin them' },

    -- ── Fishing ─────────────────────────────────────────────────────────
    ['fish']            = { source = 'Fishing',  description = 'Any fish caught with a rod and bait' },
    ['bait_bread']      = { source = 'Crafting', description = 'Made here from a loaf of bread' },

    -- ── Farming ─────────────────────────────────────────────────────────
    ['wheat']           = { source = 'Farming',  description = 'Grown on a farm plot from wheat seed' },
    ['corn']            = { source = 'Farming',  description = 'Grown on a farm plot from corn seed' },
    ['apple']           = { source = 'Farming',  description = 'Picked from an orchard tree' },

    -- ── Mining ──────────────────────────────────────────────────────────
    ['ironore']         = { source = 'Mining',   description = 'Dug from an iron vein with a pickaxe' },
    ['coal']            = { source = 'Mining',   description = 'Dug from a coal seam with a pickaxe' },
    ['salt']            = { source = 'Mining',   description = 'Cut from a salt deposit with a pickaxe' },

    -- ── Lumberjacking ───────────────────────────────────────────────────
    ['hwood']           = { source = 'Lumber',   description = 'Hardwood, split from felled trees' },

    -- ── Bought from a store ─────────────────────────────────────────────
    ['water']           = { source = 'Store',    description = 'Any general store, a few cents a canteen' },
    ['sugar']           = { source = 'Store',    description = 'Sold at general stores and bakeries' },
    ['egg']             = { source = 'Store',    description = 'General stores, or a henhouse if you keep one' },
    ['bread']           = { source = 'Store',    description = 'Baked goods counter at any general store' },
    ['cloth']           = { source = 'Store',    description = 'Sold by the yard at general stores and tailors' },
    ['glass']           = { source = 'Store',    description = 'Sold at general stores' },
    ['bottle']          = { source = 'Store',    description = 'Empty bottles, sold cheaply or returned for deposit' },
    ['pot']             = { source = 'Store',    description = 'A cooking pot. Buy it once -- crafting does not use it up' },

    -- ── Made at a bench ─────────────────────────────────────────────────
    ['flour']           = { source = 'Crafting', description = 'Milled here from wheat' },
    ['tallow']          = { source = 'Crafting', description = 'Rendered here from animal fat' },
    ['ironingot']       = { source = 'Crafting', description = 'Smelted at a blacksmith forge from ore and coal' },
    ['mash']            = { source = 'Crafting', description = 'Soured at a moonshine still from corn and water' },
}
