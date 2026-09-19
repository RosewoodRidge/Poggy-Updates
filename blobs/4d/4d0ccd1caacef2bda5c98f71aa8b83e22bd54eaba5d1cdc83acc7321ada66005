--[[===========================================================================
    poggy_crafting · config/config.lua
    ---------------------------------------------------------------------------
    Everything in this file is safe to edit and is merged forward when the
    script updates itself: lines you add stay, lines Poggy adds appear, and your
    values are never overwritten.

    Three config files, one job each:

        config/config.lua        this file -- benches, props, timing, skillchecks
        config/recipes.lua       Config.Crafting -- what can be made
        config/item_sources.lua  Config.ItemSources -- the "where do I get this?"
                                 tooltip shown on every ingredient

    Nothing here may contain a function. The updater merges this file
    structurally, and a function is code, not data.
===========================================================================]]--

Config = {}

-- ===========================================================================
--  1.  GENERAL
-- ===========================================================================

-- Extra console output while you are setting the script up. Leave false live.
Config.Debug = false

-- Discord webhook for a crafting log. Blank = no log.
-- Must be a https://discord.com/api/webhooks/... URL; anything else is ignored.
Config.Webhook = ''
Config.WebhookAvatar = ''

-- Log every craft to the webhook above. Busy servers may want this off.
Config.WebhookLogCrafts = true

-- ===========================================================================
--  2.  COMMANDS AND KEYS
-- ===========================================================================

-- Set any of these to false to remove the command entirely.
Config.Commands = {
    craft      = 'craft',       -- opens the recipe browser from anywhere
    campfire   = false,         -- place a campfire without the item (testing)
    extinguish = 'extinguish',  -- put out the campfire you placed
    tracker    = 'gt',          -- give the mouse to the gathering tracker to move it
    trackerHide = 'gthide',     -- hide/show the gathering tracker
}

-- The control that opens a bench or campfire. `G` on a keyboard.
-- Control hashes: https://github.com/femga/rdr3_discoveries/tree/master/AI/INPUTS
Config.PromptControl = 0x760A9C6F

-- ===========================================================================
--  3.  THE BROWSER
-- ===========================================================================

Config.Styles = {
    fontSize = 'm',             -- 's' | 'm' | 'l'
    descriptionsidebar = true,  -- show the recipe description panel
}

-- The per-character shopping list (right-hand panel) and the on-screen
-- gathering tracker it feeds. Turning the list off hides both and skips the
-- database table entirely.
Config.ShoppingList = {
    enabled  = true,
    maxItems = 60,   -- per character, across all recipes
}

-- Some items have no label in the inventory database, or a label nobody would
-- recognise (raw fish model names, for instance). Anything named here wins over
-- the database label in the browser. Purely cosmetic.
Config.ItemLabelOverrides = {
    -- ['a_c_fishbluegil_01_sm'] = 'Small Bluegill',
    -- ['a_c_fishperch_01_ms']   = 'Medium Perch',
}

-- ===========================================================================
--  4.  WHERE PEOPLE CRAFT
-- ===========================================================================

-- How close a player must be, in metres.
Config.Distances = {
    campfire  = 1.0,
    locations = 0.75,
}

-- Fixed crafting benches.
--
--   id          used by a recipe's `Location` and a category's `Location`.
--               Never reuse an id.
--   name        shown on the prompt and on the blip.
--   Categories  0 for every category, or a list of category idents this bench
--               offers, e.g. { 'smithing' }.
--   Job         0 for anyone, or a list of job names, e.g. { 'blacksmith' }.
--   Blip        omit for no blip. Hash is a RedM blip sprite name or hash.
--
-- These three are examples. Delete them and add your own.
Config.Locations = {
    {
        id         = 'smithy_valentine',
        name       = 'Blacksmith Anvil',
        x          = -279.16,
        y          = 807.79,
        z          = 119.36,
        Categories = { 'smithing' },
        Job        = 0,
        Blip       = { enable = true, Hash = 'blip_shop_blacksmith' },
    },
    {
        id         = 'still_lemoyne',
        name       = 'Moonshine Still',
        x          = 1790.89,
        y          = -818.54,
        z          = 189.40,
        Categories = { 'moonshine' },
        Job        = 0,
        Blip       = { enable = false, Hash = 'blip_ambient_liquor' },
    },
    {
        id         = 'workbench_general',
        name       = 'Workbench',
        x          = -324.02,
        y          = 773.36,
        z          = 117.62,
        Categories = 0,
        Job        = 0,
    },
}

-- ===========================================================================
--  5.  CAMPFIRES AND WORLD PROPS
-- ===========================================================================

-- Walking up to one of these world props opens the browser, no bench needed.
-- Set to false if you would rather people only craft at Config.Locations --
-- the prop scan is the most expensive thing this script does.
Config.CraftingPropsEnabled = true

-- `title` is both the prompt text and the location id a recipe can match on,
-- lower-cased: Location = { 'campfire' } means "only at a campfire".
Config.CraftingProps = {
    {
        title = 'Campfire',
        prop  = {
            'p_campfire01x', 'p_campfire03x', 'p_campfire05x',
            'p_campfirefresh01x', 'p_campfirecombined01x', 's_splitfirelog01x',
        },
    },
    {
        title = 'Oven',
        prop  = {
            'p_furnace01x', 'p_breadoven01x', 'p_ambstove01x',
            'p_stove01x', 'p_stove04x', 'p_stove05x', 'p_stove06x',
            'p_stove07x', 'p_stove09x', 'p_woodstove01x', 'p_gen_stove01x_tc01',
        },
    },
}

-- Restrict every campfire and prop to certain jobs. 0 = anyone.
Config.CampfireJobLock = 0

-- The usable inventory item that places a campfire, and the prop it places.
-- Set Config.CampfireItem to false if you do not want a placeable campfire;
-- sql/install.sql still adds the item, but nothing registers it.
Config.CampfireItem      = 'campfire'
Config.PlaceableCampfire = 'p_campfire05x'

-- Kneel while the browser is open, the way the game's own crafting does.
Config.KneelingAnimation = true

-- ===========================================================================
--  6.  TIMING AND THE PROGRESS BAR
-- ===========================================================================

-- How long a craft takes, in milliseconds. A recipe may override this with its
-- own `CraftTime`.
Config.CraftTime = 5000

-- The progress bar is drawn by this script's own UI, so there is no progress
-- bar resource to install and it looks the same on every framework.
Config.ProgressBar = {
    enabled  = true,
    position = 'bottom',   -- 'bottom' | 'top'
}

-- ===========================================================================
--  7.  CATEGORIES
-- ===========================================================================

-- Every recipe names one of these in its `Category`. A category the player
-- cannot reach still appears in the browser, greyed out under "Requires
-- Access", so people can see what a job or a bench would unlock.
--
--   ident     must match the recipe's Category exactly.
--   text      what the player reads.
--   Location  0 for anywhere, or a list of Config.Locations ids / prop titles.
--   Job       0 for anyone, or a list of job names.
Config.Categories = {
    {
        ident    = 'food',
        text     = 'Food & Cooking',
        Location = 0,
        Job      = 0,
    },
    {
        ident    = 'supplies',
        text     = 'Supplies',
        Location = 0,
        Job      = 0,
    },
    {
        ident    = 'smithing',
        text     = 'Blacksmithing',
        Location = { 'smithy_valentine' },
        Job      = { 'blacksmith' },
    },
    {
        ident    = 'moonshine',
        text     = 'Moonshine',
        Location = { 'still_lemoyne' },
        -- Open to everyone on purpose: the recipes inside are job-locked
        -- individually and let a non-distiller through on a hard skillcheck.
        Job      = 0,
    },
}

-- ===========================================================================
--  8.  ANIMATIONS
-- ===========================================================================

-- A recipe picks one by name with `Animation = 'knifecooking'`. A recipe with
-- no Animation uses 'craft'.
--
--   dict/name  the animation dictionary and clip.
--   flag       27 plays once, 17 loops. Looping clips suit long crafts.
--   type       'standard' (an animation) or 'scenario' (then set `hash`).
--   prop       optional object in the player's hand, with an optional subprop.
Config.Animations = {

    ['craft'] = {
        dict = 'mech_inventory@crafting@fallbacks',
        name = 'full_craft_and_stow',
        flag = 27,
        type = 'standard',
    },

    ['campfire'] = {
        dict = 'script_campfire@lighting_fire@male_male',
        name = 'light_fire_b_p2_male_b',
        flag = 17,
        type = 'standard',
    },

    ['knifecooking'] = {
        dict = 'amb_camp@world_player_fire_cook_knife@male_a@wip_base',
        name = 'wip_base',
        flag = 17,
        type = 'standard',
        prop = {
            model  = 'w_melee_knife06',
            bone   = 'SKEL_R_Finger13',
            coords = { x = -0.01, y = -0.02, z = 0.02, xr = 190.0, yr = 0.0, zr = 0.0 },
            subprop = {
                model  = 'p_redefleshymeat01xa',
                coords = { x = 0.00, y = 0.02, z = -0.20, xr = 0.0, yr = 0.0, zr = 0.0 },
            },
        },
    },

    ['spindlecook'] = {
        dict = 'amb_camp@world_camp_fire_cooking@male_d@wip_base',
        name = 'wip_base',
        flag = 17,
        type = 'standard',
        prop = {
            model  = 'p_stick04x',
            bone   = 'SKEL_R_Finger13',
            coords = { x = 0.2, y = 0.04, z = 0.12, xr = 170.0, yr = 50.0, zr = 0.0 },
            subprop = {
                model  = 's_meatbit_chunck_medium01x',
                coords = { x = -0.30, y = -0.08, z = -0.30, xr = 0.0, yr = 0.0, zr = 70.0 },
            },
        },
    },
}

-- ===========================================================================
--  9.  SKILLCHECKS  (optional -- needs poggy_skillcheck)
-- ===========================================================================
--
--  A recipe can ask for a skillcheck instead of the progress bar. There are
--  three ways in, and they are checked in this order:
--
--    1. jobSkillcheck = N   The player does NOT have the recipe's Job.
--                           Instead of being refused they get N HARD rounds.
--                           Someone who does have the job crafts normally.
--    2. useHardSkillcheck   The recipe is dangerous for everyone. N comes from
--       + skillcheck = N    `skillcheck`, the difficulty from Hard below.
--    3. skillcheck = N      N ordinary rounds.
--
--  Materials are taken BEFORE the skillcheck, so a failure costs them.
--
--  Reward maths, per round passed:
--      amount = floor( base * count * (passed / total) * (1 + greats * GreatBonusPct) + 0.5 )
--
--      0 of N passed -> nothing at all, materials gone.
--      K of N passed -> that fraction of the reward, minimum 1.
--      A "great" hit (the small bright band) adds GreatBonusPct on top.
--
--  Crafting more at once is harder: +1 round for every `RepsPerExtra` items.
--
--  Two optional penalties, for recipes that deserve them:
--      explodeOnFail   = true  a missed round detonates on the crafter
--      catchFireOnFail = true  a missed round sets the crafter alight
--  Either one means a single miss ends the craft with nothing.
--
Config.Skillcheck = {
    -- false disables every skillcheck: those recipes fall back to the progress
    -- bar and nobody needs poggy_skillcheck installed.
    enabled = true,

    -- Extra round per this many items in one craft. 0 = never scale.
    RepsPerExtra = 3,

    -- Reward bonus per great hit, as a fraction. 0.25 = +25% each.
    GreatBonusPct = 0.25,

    -- Milliseconds between the browser closing and the first round, so the
    -- screen is clear before the needle appears.
    StartDelay = 500,

    -- ── Normal ──────────────────────────────────────────────────────────
    Normal = {
        speed       = 3,        -- needle speed, 1-5
        difficulty  = 2,        -- success band, 1-5 (higher = smaller)
        great       = 25,       -- great band, % of the success band
        randomizer  = 1,        -- how much the band moves between rounds
        shake       = false,
        shakeSpeed  = 1,
        shakeDist   = 1,
        timeBetween = 200,      -- ms between rounds
        direction   = 'rand',   -- 'rand' | 'cw' | 'ccw'
    },

    -- ── Job bypass: crafting something you were not trained for ─────────
    JobBypass = {
        speed       = 4,
        difficulty  = 4,
        great       = 20,
        randomizer  = 2,
        shake       = true,
        shakeSpeed  = 1,
        shakeDist   = 1,
        timeBetween = 100,
        direction   = 'rand',
    },

    -- ── Hard: explosives, volatile chemistry ────────────────────────────
    Hard = {
        speed       = 4,
        difficulty  = 4,
        great       = 20,
        randomizer  = 2,
        shake       = true,
        shakeSpeed  = 3,
        shakeDist   = 2,
        timeBetween = 100,
        direction   = 'rand',
    },
}

-- What `explodeOnFail = true` does.
Config.CraftingExplosion = {
    ExplosionType   = 25,      -- 25 = dynamite
    ExplosionDamage = 1000.0,
    ExplosionShake  = 1.0,
}

-- What `catchFireOnFail = true` does. RDR3 puts a ped out after ~2 seconds, so
-- the script re-lights them until the duration is up.
Config.CraftingFire = {
    FireDuration = 10000,   -- ms
}
