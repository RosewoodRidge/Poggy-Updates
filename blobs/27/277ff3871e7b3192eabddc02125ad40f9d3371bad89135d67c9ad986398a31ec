Config = {}

-- ============================================================================
-- Admins bypass every job restriction, and are the only ones who can use the
-- custom ped box and the character browser.
--
-- ACE is checked first and works on every framework.  Add to server.cfg:
--     add_ace group.admin poggy_transform.admin allow
-- ============================================================================
Config.AdminAce = "poggy_transform.admin"

-- Framework admin groups, checked after ACE.  The group comes from poggy_core
-- (perms.group), so this works on any framework poggy_core supports.  On a
-- framework poggy_core does not drive yet, use the ACE permission above.
Config.AdminGroups = { "admin", "superadmin", "god" }

-- ============================================================================
-- Anti-abuse
--
-- Attacks are relayed through the server, so both ends validate: the server
-- won't relay faster than AttackMinInterval, and the target won't accept an
-- attack from further away than the animal's reach plus the tolerance below.
-- Raise the tolerance if players report attacks missing on a laggy server.
-- ============================================================================
Config.AttackMinInterval     = 300   -- ms; minimum gap between relayed attacks
Config.AttackRangeTolerance  = 2.0   -- metres of slack on top of attackRadius

-- ============================================================================
-- Command to open the transform menu
-- ============================================================================
Config.Command = "transform"

-- ============================================================================
-- Debug mode
-- ============================================================================
Config.Debug = false

-- ============================================================================
-- Attack cooldown (ms) — time between attacks for aggressive animal peds
-- ============================================================================
Config.AttackCooldown = 2000

-- ============================================================================
-- Attack input mappings — which buttons trigger which attack slot
--
--   "primary"  = INPUT_ATTACK        (LMB / RT)         — basic attack
--   "heavy"    = INPUT_MELEE_ATTACK   (R / RS click)     — pounce / heavy
--   "special"  = INPUT_HORSE_MELEE    (RB / R1)          — NOTE: also crouch toggle
--                                                          when no target in range
--   "taunt"    = INPUT_CONTEXT_B      (G / D-Pad Right)  — growl / howl / taunt
--
-- These are checked in the main loop. Each maps to an attacks[] key.
-- ============================================================================
Config.AttackInputs = {
    { slot = "pounce",  input = `INPUT_ENTER` },   -- heavy / pounce
    { slot = "ground",  input = `INPUT_ATTACK` },   -- heavy / pounce
    { slot = "taunt",   input = `INPUT_RELOAD` },      -- growl / taunt
}

-- ============================================================================
-- Categories — define the tab/section order and appearance in the UI
--
--   id    : matches the 'category' field on each entry
--   label : display name in the UI tab
--   icon  : emoji/icon shown next to the tab label
-- ============================================================================
Config.Categories = {
    { id = "predator",   label = "Predators",        icon = "🐺" },
    { id = "prey",       label = "Prey & Wildlife",  icon = "🦌" },
    { id = "bird",       label = "Birds",            icon = "🦅" },
    { id = "aquatic",    label = "Aquatic & Reptile", icon = "🐊" },
    { id = "legendary",  label = "Legendary",        icon = "⭐" },
    { id = "people",     label = "People",           icon = "🤠" },
}

-- ============================================================================
-- Transformables — every ped the player can become
--
--   id         : unique string identifier
--   label      : display name in the UI
--   model      : RDR3 ped model name
--   image      : filename inside ui/images/  (will show in the card)
--   category   : must match a Config.Categories[].id
--   job        : required job name ('' = no restriction). Admins always bypass.
--   aggressive : if true, the ped can attack other peds/players
--   brawlStyle : (optional) hash for _SET_PED_BRAWLING_STYLE
--   scale      : (optional) ped scale override, defaults to 1.0
--
-- ATTACK SYSTEM:
--   attackAnim   : primary attack { dict, name } — triggered by INPUT_ATTACK (LMB / RT)
--   attackRadius : max range to find a target
--   attackForce  : ragdoll velocity multiplier
--   attackDamage : damage dealt per hit
--   attacks      : (optional) table of extra attack moves, each with:
--       key      : identifier ("pounce", "ground", "jump", "swipe", "taunt", etc.)
--       label    : display-friendly name
--       anim     : { dict, name }
--       damage   : override damage (nil = use attackDamage)
--       force    : override force (nil = use attackForce)
--       radius   : override radius (nil = use attackRadius)
--       cooldown : override cooldown ms (nil = use Config.AttackCooldown)
--       input    : input hash name — which button triggers it
--                  Mapped in client: see Config.AttackInputs below
-- ============================================================================
Config.Animals = {

    -- =====================================================================
    -- PREDATORS
    -- =====================================================================
    {
        id         = "wolf",
        label      = "Wolf",
        model      = "A_C_Wolf",
        image      = "wolf.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0xA8F023D4, -- BS_WOLF
        attackAnim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 3.0,
        attackDamage = 30,
        attacks = {
            { key = "pounce",  label = "Pounce Kill",   anim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "jump_attack_0" },   damage = 50, force = 5.0, radius = 5.0, cooldown = 4000 },
            { key = "ground",  label = "Ground Attack", anim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "ground_attack_0" }, damage = 40, force = 2.0 },
            { key = "taunt",   label = "Howl/Growl",    anim = { dict = "creatures_mammal@wolf@melee@taunts", name = "taunt_a" }, damage = 0, force = 0, radius = 0 },
        },
        jumpAnim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "jump_attack_0" },
        emotes = {
            { cmd = "howl",  label = "Howl",  loop = false, anim = { dict = "amb_creature_mammal@world_wolf_howling@wolf_medium@base",          name = "base"    } },
            { cmd = "growl", label = "Growl", loop = false, anim = { dict = "creatures_mammal@wolf@melee@taunts",                              name = "taunt_b" } },
            { cmd = "sniff", label = "Sniff", loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sniffing_ground@wolf_medium@base",  name = "base"   } },
            { cmd = "sit",   label = "Sit",   loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sitting@wolf_medium@base",          name = "base"   } },
            { cmd = "rest",  label = "Rest",  loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_resting@wolf_medium@idle",          name = "idle_a" } },
            { cmd = "sleep", label = "Sleep", loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sleeping@idle",                    name = "idle_a" } },
        },
    },
    {
        id         = "wolf_small",
        label      = "Small Wolf",
        model      = "A_C_Wolf_Small",
        image      = "wolf_small.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0xA8F023D4, -- BS_WOLF
        attackAnim = { dict = "creatures_mammal@wolf_small@melee@attacks@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 3.0,
        attackDamage = 20,
        attacks = {
            { key = "pounce",  label = "Pounce Kill",   anim = { dict = "creatures_mammal@wolf_small@melee@attacks@streamed_core", name = "jump_attack_0" },   damage = 35, force = 4.0, radius = 5.0, cooldown = 4000 },
            { key = "ground",  label = "Ground Attack", anim = { dict = "creatures_mammal@wolf_small@melee@attacks@streamed_core", name = "ground_attack_0" }, damage = 30, force = 2.0 },
            { key = "taunt",   label = "Howl/Growl",    anim = { dict = "creatures_mammal@wolf_small@melee@taunts", name = "taunt_a" }, damage = 0, force = 0, radius = 0 },
        },
        jumpAnim = { dict = "creatures_mammal@wolf_small@melee@attacks@streamed_core", name = "jump_attack_0" },
        emotes = {
            { cmd = "howl",  label = "Howl",  loop = false, anim = { dict = "amb_creature_mammal@world_wolf_howling@wolf_small@base",           name = "base"    } },
            { cmd = "growl", label = "Growl", loop = false, anim = { dict = "creatures_mammal@wolf_small@melee@taunts",                        name = "taunt_b" } },
            { cmd = "sniff", label = "Sniff", loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sniffing_ground@wolf_medium@base",  name = "base"   } },
            { cmd = "sit",   label = "Sit",   loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sitting@wolf_medium@base",          name = "base"   } },
            { cmd = "rest",  label = "Rest",  loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_resting@wolf_small@idle",           name = "idle_a" } },
            { cmd = "sleep", label = "Sleep", loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sleeping@idle",                    name = "idle_a" } },
        },
    },
    {
        id         = "bear_grizzly",
        label      = "Grizzly Bear",
        model      = "A_C_Bear_01",
        image      = "bear.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x0BC66E35, -- BS_BEAR
        attackAnim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 5.0,
        attackDamage = 30,
        attacks = {
            { key = "pounce",  label = "Running Swipe",  anim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "att_run_swipe_v01" },       damage = 45, force = 6.0, radius = 4.0, cooldown = 3500 },
            { key = "ground",  label = "Double Swipe",   anim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "att_swipe_both_v01" },      damage = 40, force = 4.0 },
            { key = "taunt",   label = "Stand & Roar",   anim = { dict = "creatures_mammal@bear@melee@fightidle", name = "idle" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "bear_black",
        label      = "Black Bear",
        model      = "A_C_BearBlack_01",
        image      = "bear_black.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x0BC66E35, -- BS_BEAR
        attackAnim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 5.0,
        attackDamage = 25,
        attacks = {
            { key = "pounce",  label = "Running Swipe",  anim = { dict = "creatures_mammal@bearblack@melee@streamed_core", name = "att_run_swipe_v01" },   damage = 40, force = 5.0, radius = 4.0, cooldown = 3500 },
            { key = "ground",  label = "Left Swipe",     anim = { dict = "creatures_mammal@bearblack@melee@streamed_core", name = "att_swipe_left_v01" },  damage = 35, force = 3.0 },
            { key = "taunt",   label = "Stand & Roar",   anim = { dict = "creatures_mammal@bear@melee@fightidle", name = "idle" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "cougar",
        label      = "Cougar",
        model      = "A_C_Cougar_01",
        image      = "cougar.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x9DAA7CCB, -- BS_COUGAR
        attackAnim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "attack" },
        attackRadius = 2.0,
        attackForce  = 3.0,
        attackDamage = 20,
        attacks = {
            { key = "pounce",  label = "Pounce Takedown", anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "takedown_from_front" }, damage = 55, force = 5.0, radius = 5.0, cooldown = 5000 },
            { key = "ground",  label = "Ground Maul",     anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "ground_attack_0" },      damage = 35, force = 2.0 },
            { key = "taunt",   label = "Growl",           anim = { dict = "creatures_mammal@cougar@melee@streamed_taunts", name = "growling" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "panther",
        label      = "Florida Panther",
        model      = "A_C_Panther_01",
        image      = "panther.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x9DAA7CCB, -- BS_COUGAR
        attackAnim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "attack" },
        attackRadius = 2.0,
        attackForce  = 3.0,
        attackDamage = 20,
        attacks = {
            { key = "pounce",  label = "Pounce Takedown", anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "takedown_from_back" },  damage = 55, force = 5.0, radius = 5.0, cooldown = 5000 },
            { key = "ground",  label = "Ground Maul",     anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "ground_attack_0" },     damage = 35, force = 2.0 },
            { key = "taunt",   label = "Growl",           anim = { dict = "creatures_mammal@cougar@melee@streamed_taunts", name = "growling" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "coyote",
        label      = "Coyote",
        model      = "A_C_Coyote_01",
        image      = "coyote.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0xA448EB69, -- BS_COYOTE
        attackAnim = { dict = "creatures_mammal@coyote@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 2.0,
        attackDamage = 25,
        attacks = {
            { key = "pounce",  label = "Leap Attack",   anim = { dict = "creatures_mammal@coyote@melee@streamed_core", name = "jump_attack_0" },   damage = 35, force = 3.0, radius = 4.5, cooldown = 4000 },
            { key = "ground",  label = "Ground Attack", anim = { dict = "creatures_mammal@coyote@melee@streamed_core", name = "ground_attack_0" }, damage = 30, force = 2.0 },
        },
    },
    {
        id         = "boar",
        label      = "Wild Boar",
        model      = "MP_A_C_BOAR_01",
        image      = "boar.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x176A5831, -- BS_BOAR
        attackAnim = { dict = "creatures_mammal@boar@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 3.0,
        attackDamage = 25,
    },
    {
        id         = "alligator",
        label      = "Alligator",
        model      = "A_C_Alligator_02",
        image      = "alligator.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x7A5548ED, -- BS_ALLIGATOR
        attackAnim = { dict = "amb_creatures_reptile@gator_giant@nip_attack", name = "nip" },
        attackRadius = 3.0,
        attackForce  = 2.0,
        attackDamage = 25,
        attacks = {
            { key = "pounce",  label = "Lunge Left",  anim = { dict = "amb_creatures_reptile@gator_giant@nip_attack", name = "nip_left" },  damage = 35, force = 3.0, radius = 3.5, cooldown = 3000 },
            { key = "ground",  label = "Lunge Right", anim = { dict = "amb_creatures_reptile@gator_giant@nip_attack", name = "nip_right" }, damage = 35, force = 3.0 },
        },
    },
    {
        id         = "alligator_large",
        label      = "Large Alligator",
        model      = "A_C_Alligator_03",
        image      = "alligator_large.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x368EC7CB, -- BS_ALLIGATOR_LARGE
        attackAnim = { dict = "creatures_reptile@alligator@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 2.0,
        attackDamage = 25,
        attacks = {
            { key = "pounce",  label = "Lunge Forward", anim = { dict = "creatures_reptile@alligator@melee@streamed_core", name = "lunge_attack_forward" }, damage = 40, force = 4.0, radius = 4.0, cooldown = 3500 },
            { key = "ground",  label = "Snap Left",     anim = { dict = "creatures_reptile@alligator@melee@streamed_core", name = "attack_left" },          damage = 30, force = 2.0 },
        },
    },
    {
        id         = "snake",
        label      = "Snake",
        model      = "A_C_Snake_01",
        image      = "snake.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x82BEBC4B, -- BS_SNAKE
        attackAnim = { dict = "creatures_reptile@snake@melee@streamed_core", name = "attack_high_close" },
        attackRadius = 2.0,
        attackForce  = 1.0,
        attackDamage = 15,
        attacks = {
            { key = "pounce",  label = "Strike Far",  anim = { dict = "creatures_reptile@snake@melee@streamed_core", name = "attack_high_far" },   damage = 20, force = 1.5, radius = 3.0, cooldown = 2500 },
            { key = "ground",  label = "Low Strike",  anim = { dict = "creatures_reptile@snake@melee@streamed_core", name = "attack_low_close" },  damage = 15, force = 1.0 },
        },
    },
    {
        id         = "badger",
        label      = "Badger",
        model      = "A_C_Badger_01",
        image      = "badger.png",
        category   = "predator",
        job        = "",
        aggressive = true,
        brawlStyle = 0x7E7C3F53, -- BS_BADGER
        attackAnim = { dict = "creatures_mammal@badger@melee", name = "nip_attack" },
        attackRadius = 2.0,
        attackForce  = 1.0,
        attackDamage = 15,
    },

    -- =====================================================================
    -- PREY & WILDLIFE
    -- =====================================================================
    {
        id         = "buck",
        label      = "Buck",
        model      = "A_C_Buck_01",
        image      = "buck.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "deer",
        label      = "Deer",
        model      = "A_C_Deer_01",
        image      = "deer.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "elk",
        label      = "Elk",
        model      = "A_C_Elk_01",
        image      = "elk.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "moose",
        label      = "Moose",
        model      = "A_C_Moose_01",
        image      = "moose.png",
        category   = "prey",
        job        = "",
        aggressive = false,
        brawlStyle = 0x968917AB, -- BS_MOOSE
    },
    {
        id         = "bison",
        label      = "Bison",
        model      = "A_C_Buffalo_01",
        image      = "bison.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "bull",
        label      = "Bull",
        model      = "A_C_Bull_01",
        image      = "bull.png",
        category   = "prey",
        job        = "",
        aggressive = true,
        brawlStyle = 0x4E50C5D2, -- BS_BULL
        attackAnim = { dict = "creatures_mammal@bull@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 4.0,
        attackDamage = 25,
    },
    {
        id         = "bighorn_ram",
        label      = "Bighorn Ram",
        model      = "MP_A_C_BIGHORNRAM_01",
        image      = "bighorn_ram.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "pronghorn",
        label      = "Pronghorn",
        model      = "A_C_Pronghorn_01",
        image      = "pronghorn.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "goat",
        label      = "Goat",
        model      = "A_C_Goat_01",
        image      = "goat.png",
        category   = "prey",
        job        = "",
        aggressive = false,
        brawlStyle = 0x078E649F, -- BS_GOAT
    },
    {
        id         = "sheep",
        label      = "Sheep",
        model      = "A_C_Sheep_01",
        image      = "sheep.png",
        category   = "prey",
        job        = "",
        aggressive = false,
        brawlStyle = 0x6827CCCF, -- BS_SHEEP
    },
    {
        id         = "pig",
        label      = "Pig",
        model      = "A_C_Pig_01",
        image      = "pig.png",
        category   = "prey",
        job        = "",
        aggressive = false,
        brawlStyle = 0x22EAD110, -- BS_PIG
    },
    {
        id         = "beaver",
        label      = "Beaver",
        model      = "MP_A_C_BEAVER_01",
        image      = "beaver.png",
        category   = "prey",
        job        = "",
        aggressive = true,
        brawlStyle = 0x4E313783, -- BS_BEAVER
        attackAnim = { dict = "creatures_mammal@beaver@melee", name = "nip_attack" },
        attackRadius = 2.0,
        attackForce  = 1.0,
        attackDamage = 15,
    },
    {
        id         = "fox",
        label      = "Fox",
        model      = "A_C_Fox_01",
        image      = "fox.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "rabbit",
        label      = "Rabbit",
        model      = "A_C_Rabbit_01",
        image      = "rabbit.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "raccoon",
        label      = "Raccoon",
        model      = "A_C_Raccoon_01",
        image      = "raccoon.png",
        category   = "prey",
        job        = "",
        aggressive = true,
        brawlStyle = 0x505F8917, -- BS_RACCOON
        attackAnim = { dict = "creatures_mammal@raccoon@melee", name = "nip_attack" },
        attackRadius = 2.0,
        attackForce  = 1.0,
        attackDamage = 15,
    },
    {
        id         = "muskrat",
        label      = "Muskrat",
        model      = "A_C_Muskrat_01",
        image      = "muskrat.png",
        category   = "prey",
        job        = "",
        aggressive = true,
        brawlStyle = 0x1EDC33AC, -- BS_MUSKRAT
        attackAnim = { dict = "creatures_mammal@muskrat@melee", name = "nip_attack" },
        attackRadius = 2.0,
        attackForce  = 1.0,
        attackDamage = 15,
    },
    {
        id         = "cat",
        label      = "Cat",
        model      = "A_C_Cat_01",
        image      = "cat.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "dog_husky",
        label      = "Husky",
        model      = "A_C_DogHusky_01",
        image      = "husky.png",
        category   = "prey",
        job        = "",
        aggressive = true,
        brawlStyle = 0x5A4155C4, -- BS_DOG
        attackAnim = { dict = "creatures_mammal@dog_pers@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 2.0,
        attackDamage = 20,
        jumpAnim = { dict = "creatures_mammal@dog_pers@melee@attacks@basic", name = "jump_attack_0" },
        emotes = {
            { cmd = "bark",    label = "Bark",    loop = false, anim = { dict = "creatures_mammal@dog_pers@melee@streamed_taunts",              name = "taunt_01"  } },
            { cmd = "growl",   label = "Growl",   loop = false, anim = { dict = "creatures_mammal@dog_pers@melee@streamed_taunts",              name = "taunt_02"  } },
            { cmd = "happy",   label = "Happy",   loop = true,  anim = { dict = "creatures_mammal@dog_pers@happy@idle@idle_variation@var_a",   name = "idle"      } },
            { cmd = "sniff",   label = "Sniff",   loop = true,  anim = { dict = "creatures_mammal@dog_pers@searching@idle@idle_variation@var_a", name = "idle"    } },
            { cmd = "scratch", label = "Scratch", loop = false, anim = { dict = "creatures_mammal@dog_pers@agitated@idle@variation@fidget_02", name = "idle"      } },
            { cmd = "shake",   label = "Shake",   loop = false, anim = { dict = "creatures_mammal@dog_pers@wet@idle@variations@drying_off_a",  name = "idle"      } },
            { cmd = "sad",     label = "Sad",     loop = true,  anim = { dict = "creatures_mammal@dog_pers@sad@idle@idle_variation@var_a",     name = "idle"      } },
            { cmd = "roll",    label = "Roll",    loop = true,  anim = { dict = "amb_creature_mammal@world_dog_roll_ground@idle",              name = "idle_a"    } },
            { cmd = "sleep",   label = "Sleep",   loop = true,  anim = { dict = "amb_creature_mammal@world_dog_sleeping@idle",                name = "idle_a"    } },
        },
    },
    {
        id         = "dog_foxhound",
        label      = "Fox Hound",
        model      = "a_c_dogamericanfoxhound_01",
        image      = "husky.png",
        category   = "prey",
        job        = "",
        aggressive = true,
        brawlStyle = 0x5A4155C4, -- BS_DOG
        attackAnim = { dict = "creatures_mammal@dog_pers@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 2.0,
        attackDamage = 20,

        -- ---- Emotes: /ae <cmd>  (foxhound only) ----
        --   loop = true  → animation loops until /ae stop
        --   loop = false → plays once and returns to idle
        jumpAnim = { dict = "creatures_mammal@dog_pers@melee@attacks@basic", name = "jump_attack_0" },
        emotes = {
            { cmd = "bark",    label = "Bark",    loop = false, anim = { dict = "creatures_mammal@dog_pers@melee@streamed_taunts",              name = "taunt_01"  } },
            { cmd = "growl",   label = "Growl",   loop = false, anim = { dict = "creatures_mammal@dog_pers@melee@streamed_taunts",              name = "taunt_02"  } },
            { cmd = "happy",   label = "Happy",   loop = true,  anim = { dict = "creatures_mammal@dog_pers@happy@idle@idle_variation@var_a",   name = "idle"      } },
            { cmd = "sniff",   label = "Sniff",   loop = true,  anim = { dict = "creatures_mammal@dog_pers@searching@idle@idle_variation@var_a", name = "idle"    } },
            { cmd = "scratch", label = "Scratch", loop = false, anim = { dict = "creatures_mammal@dog_pers@agitated@idle@variation@fidget_02", name = "idle"      } },  -- fidget_01 was bark; try fidget_03 if this still barks
            { cmd = "shake",   label = "Shake",   loop = false, anim = { dict = "creatures_mammal@dog_pers@wet@idle@variations@drying_off_a",  name = "idle"      } },
            { cmd = "sad",     label = "Sad",     loop = true,  anim = { dict = "creatures_mammal@dog_pers@sad@idle@idle_variation@var_a",     name = "idle"      } },
            { cmd = "roll",    label = "Roll",    loop = true,  anim = { dict = "amb_creature_mammal@world_dog_roll_ground@idle",              name = "idle_a"    } },
            { cmd = "sleep",   label = "Sleep",   loop = true,  anim = { dict = "amb_creature_mammal@world_dog_sleeping@idle",                name = "idle_a"    } },
        },
    },
    {
        id         = "armadillo",
        label      = "Armadillo",
        model      = "A_C_Armadillo_01",
        image      = "armadillo.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "gila_monster",
        label      = "Gila Monster",
        model      = "A_C_GilaMonster_01",
        image      = "gila_monster.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },
    {
        id         = "chicken",
        label      = "Chicken",
        model      = "A_C_Chicken_01",
        image      = "chicken.png",
        category   = "prey",
        job        = "",
        aggressive = false,
    },

    -- =====================================================================
    -- BIRDS
    -- =====================================================================
    {
        id         = "eagle",
        label      = "Eagle",
        model      = "A_C_Eagle_01",
        image      = "eagle.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "hawk",
        label      = "Hawk",
        model      = "A_C_Hawk_01",
        image      = "hawk.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "owl",
        label      = "Owl",
        model      = "A_C_Owl_01",
        image      = "owl.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "crow",
        label      = "Crow",
        model      = "A_C_Crow_01",
        image      = "crow.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "vulture",
        label      = "Vulture",
        model      = "A_C_Vulture_01",
        image      = "vulture.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "pelican",
        label      = "Pelican",
        model      = "A_C_Pelican_01",
        image      = "pelican.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "heron",
        label      = "Great Blue Heron",
        model      = "A_C_Heron_01",
        image      = "heron.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "parrot",
        label      = "Parrot",
        model      = "A_C_Parrot_01",
        image      = "parrot.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },
    {
        id         = "bat",
        label      = "Bat",
        model      = "a_c_bat_01",
        image      = "bat.png",
        category   = "bird",
        job        = "",
        aggressive = false,
    },

    -- =====================================================================
    -- AQUATIC & REPTILE
    -- =====================================================================
    {
        id         = "turtle_snapping",
        label      = "Snapping Turtle",
        model      = "A_C_TurtleSnapping_01",
        image      = "turtle.png",
        category   = "aquatic",
        job        = "",
        aggressive = false,
    },
    {
        id         = "frog",
        label      = "Bullfrog",
        model      = "A_C_FrogBull_01",
        image      = "frog.png",
        category   = "aquatic",
        job        = "",
        aggressive = false,
    },
    {
        id         = "crab",
        label      = "Crab",
        model      = "A_C_Crab_01",
        image      = "crab.png",
        category   = "aquatic",
        job        = "",
        aggressive = false,
    },

    -- =====================================================================
    -- LEGENDARY ANIMALS
    -- =====================================================================
    {
        id         = "legendary_wolf",
        label      = "Legendary Wolf",
        model      = "A_C_Wolf",
        image      = "wolf.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0xA8F023D4, -- BS_WOLF
        scale      = 1.3,
        attackAnim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "attack" },
        attackRadius = 3.5,
        attackForce  = 4.0,
        attackDamage = 45,
        attacks = {
            { key = "pounce",  label = "Pounce Kill",   anim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "jump_attack_0" },   damage = 70, force = 7.0, radius = 6.0, cooldown = 4000 },
            { key = "ground",  label = "Ground Maul",   anim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "ground_attack_0" }, damage = 55, force = 3.0 },
            { key = "taunt",   label = "Alpha Howl",    anim = { dict = "creatures_mammal@wolf@melee@taunts", name = "taunt_a" }, damage = 0, force = 0, radius = 0 },
        },
        jumpAnim = { dict = "creatures_mammal@wolf@melee@attacks@streamed_core", name = "jump_attack_0" },
        emotes = {
            { cmd = "howl",  label = "Howl",  loop = false, anim = { dict = "amb_creature_mammal@world_wolf_howling@wolf_medium@base",          name = "base"    } },
            { cmd = "growl", label = "Growl", loop = false, anim = { dict = "creatures_mammal@wolf@melee@taunts",                              name = "taunt_b" } },
            { cmd = "sniff", label = "Sniff", loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sniffing_ground@wolf_medium@base",  name = "base"   } },
            { cmd = "sit",   label = "Sit",   loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sitting@wolf_medium@base",          name = "base"   } },
            { cmd = "rest",  label = "Rest",  loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_resting@wolf_medium@idle",          name = "idle_a" } },
            { cmd = "sleep", label = "Sleep", loop = true,  anim = { dict = "amb_creature_mammal@world_wolf_sleeping@idle",                    name = "idle_a" } },
        },
    },
    {
        id         = "legendary_bear",
        label      = "Legendary Bear",
        model      = "A_C_Bear_01",
        image      = "bear.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x0BC66E35, -- BS_BEAR
        scale      = 1.5,
        attackAnim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "attack" },
        attackRadius = 3.5,
        attackForce  = 6.0,
        attackDamage = 50,
        attacks = {
            { key = "pounce",  label = "Charging Swipe", anim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "att_run_swipe_v01" },  damage = 75, force = 8.0, radius = 5.0, cooldown = 3500 },
            { key = "ground",  label = "Double Swipe",   anim = { dict = "creatures_mammal@bear@melee@streamed_core", name = "att_swipe_both_v01" }, damage = 60, force = 5.0 },
            { key = "taunt",   label = "Stand & Roar",   anim = { dict = "creatures_mammal@bear@melee@fightidle", name = "idle" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "legendary_panther",
        label      = "Legendary Panther",
        model      = "A_C_Panther_01",
        image      = "panther.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x9DAA7CCB, -- BS_COUGAR
        scale      = 1.3,
        attackAnim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 4.0,
        attackDamage = 35,
        attacks = {
            { key = "pounce",  label = "Pounce Takedown", anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "takedown_from_back" },  damage = 70, force = 6.0, radius = 6.0, cooldown = 5000 },
            { key = "ground",  label = "Ground Maul",     anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "ground_attack_0" },     damage = 50, force = 3.0 },
            { key = "taunt",   label = "Growl",           anim = { dict = "creatures_mammal@cougar@melee@streamed_taunts", name = "growling" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "legendary_alligator",
        label      = "Legendary Alligator",
        model      = "A_C_Alligator_03",
        image      = "alligator.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x368EC7CB, -- BS_ALLIGATOR_LARGE
        scale      = 1.5,
        attackAnim = { dict = "creatures_reptile@alligator@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 3.0,
        attackDamage = 40,
        attacks = {
            { key = "pounce",  label = "Lunge Forward", anim = { dict = "creatures_reptile@alligator@melee@streamed_core", name = "lunge_attack_forward" }, damage = 60, force = 5.0, radius = 4.5, cooldown = 3500 },
            { key = "ground",  label = "Snap Left",     anim = { dict = "creatures_reptile@alligator@melee@streamed_core", name = "attack_left" },          damage = 45, force = 3.0 },
        },
    },
    {
        id         = "legendary_cougar",
        label      = "Legendary Cougar",
        model      = "A_C_Cougar_01",
        image      = "cougar.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x9DAA7CCB, -- BS_COUGAR
        scale      = 1.3,
        attackAnim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "attack" },
        attackRadius = 2.5,
        attackForce  = 4.0,
        attackDamage = 35,
        attacks = {
            { key = "pounce",  label = "Pounce Takedown", anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "takedown_from_front" }, damage = 70, force = 6.0, radius = 6.0, cooldown = 5000 },
            { key = "ground",  label = "Ground Maul",     anim = { dict = "creatures_mammal@cougar@melee@streamed_core", name = "ground_attack_0" },      damage = 50, force = 3.0 },
            { key = "taunt",   label = "Growl",           anim = { dict = "creatures_mammal@cougar@melee@streamed_taunts", name = "growling" }, damage = 0, force = 0, radius = 0 },
        },
    },
    {
        id         = "legendary_buck",
        label      = "Legendary Buck",
        model      = "MP_A_C_BUCK_01",
        image      = "buck.png",
        category   = "legendary",
        job        = "",
        aggressive = false,
        scale      = 1.4,
    },
    {
        id         = "legendary_moose",
        label      = "Legendary Moose",
        model      = "A_C_Moose_01",
        image      = "moose.png",
        category   = "legendary",
        job        = "",
        aggressive = false,
        brawlStyle = 0x968917AB, -- BS_MOOSE
        scale      = 1.5,
    },
    {
        id         = "legendary_boar",
        label      = "Legendary Boar",
        model      = "MP_A_C_BOAR_01",
        image      = "boar.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x176A5831, -- BS_BOAR
        scale      = 1.4,
        attackAnim = { dict = "creatures_mammal@boar@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 4.0,
        attackDamage = 40,
    },
    {
        id         = "legendary_bison",
        label      = "Legendary Bison",
        model      = "A_C_Buffalo_01",
        image      = "bison.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x4E50C5D2, -- BS_BULL (closest)
        scale      = 1.5,
        attackAnim = { dict = "creatures_mammal@bull@melee@streamed_core", name = "attack" },
        attackRadius = 3.5,
        attackForce  = 5.0,
        attackDamage = 45,
    },
    {
        id         = "legendary_ram",
        label      = "Legendary Ram",
        model      = "MP_A_C_BIGHORNRAM_01",
        image      = "ram.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        scale      = 1.3,
        attackAnim = { dict = "creatures_mammal@bull@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 3.0,
        attackDamage = 35,
    },
    {
        id         = "legendary_beaver",
        label      = "Legendary Beaver",
        model      = "MP_A_C_BEAVER_01",
        image      = "beaver.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0x4E313783, -- BS_BEAVER
        scale      = 1.5,
        attackAnim = { dict = "creatures_mammal@beaver@melee", name = "nip_attack" },
        attackRadius = 2.5,
        attackForce  = 2.0,
        attackDamage = 25,
    },
    {
        id         = "legendary_coyote",
        label      = "Legendary Coyote",
        model      = "A_C_Coyote_01",
        image      = "coyote.png",
        category   = "legendary",
        job        = "",
        aggressive = true,
        brawlStyle = 0xA448EB69, -- BS_COYOTE
        scale      = 1.3,
        attackAnim = { dict = "creatures_mammal@coyote@melee@streamed_core", name = "attack" },
        attackRadius = 3.0,
        attackForce  = 3.0,
        attackDamage = 40,
        attacks = {
            { key = "pounce",  label = "Leap Attack",   anim = { dict = "creatures_mammal@coyote@melee@streamed_core", name = "jump_attack_0" },   damage = 55, force = 4.0, radius = 5.0, cooldown = 4000 },
            { key = "ground",  label = "Ground Attack", anim = { dict = "creatures_mammal@coyote@melee@streamed_core", name = "ground_attack_0" }, damage = 50, force = 3.0 },
        },
    },

    -- =====================================================================
    -- PEOPLE PEDS
    -- =====================================================================

    -- ---- Story Characters ----
    {
        id         = "dutch",
        label      = "Dutch van der Linde",
        model      = "cs_dutch",
        image      = "dutch.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "micah",
        label      = "Micah Bell",
        model      = "cs_micahbell",
        image      = "micah.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "john",
        label      = "John Marston",
        model      = "cs_johnmarston",
        image      = "john.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "hosea",
        label      = "Hosea Matthews",
        model      = "cs_hoseamatthews",
        image      = "hosea.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "charles",
        label      = "Charles Smith",
        model      = "cs_charlessmith",
        image      = "charles.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "bill",
        label      = "Bill Williamson",
        model      = "cs_billwilliamson",
        image      = "bill.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "javier",
        label      = "Javier Escuella",
        model      = "cs_javierescuella",
        image      = "javier.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "kieran",
        label      = "Kieran Duffy",
        model      = "cs_kieran",
        image      = "kieran.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "josiah",
        label      = "Josiah Trelawny",
        model      = "cs_josiahtrelawny",
        image      = "josiah.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "reverend",
        label      = "Reverend Swanson",
        model      = "cs_reverendswanson",
        image      = "reverend.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "colm",
        label      = "Colm O'Driscoll",
        model      = "cs_colmodriscoll",
        image      = "colm.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },

    -- ---- Lawmen ----
    {
        id         = "edgar_ross",
        label      = "Edgar Ross",
        model      = "cs_edgarross",
        image      = "edgar_ross.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "pinkerton",
        label      = "Pinkerton Agent",
        model      = "cs_pinkertongoon",
        image      = "pinkerton.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "sheriff_valentine",
        label      = "Valentine Sheriff",
        model      = "cs_valsheriff",
        image      = "sheriff_valentine.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "sheriff_strawberry",
        label      = "Strawberry Sheriff",
        model      = "cs_strsheriff_01",
        image      = "sheriff_strawberry.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "sheriff_blackwater",
        label      = "Blackwater Sheriff",
        model      = "cs_bwsheriff",
        image      = "sheriff_blackwater.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "sheriff_owens",
        label      = "Sheriff Owens",
        model      = "cs_sheriffowens",
        image      = "sheriff_owens.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "bounty_hunter",
        label      = "Bounty Hunter",
        model      = "cs_mp_bountyhunter",
        image      = "bounty_hunter.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "sd_police",
        label      = "Saint Denis Police",
        model      = "s_m_m_ambientsdpolice_01",
        image      = "sd_police.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },

    -- ---- Guards & Military ----
    {
        id         = "cornwall_guard",
        label      = "Cornwall Guard",
        model      = "MP_S_M_M_CornwallGuard_01",
        image      = "cornwall_guard.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "jameson_guard",
        label      = "Jameson Guard",
        model      = "a_m_m_jamesonguard_01",
        image      = "jameson_guard.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "skp_guard",
        label      = "Saint Denis Guard",
        model      = "s_m_m_skpguard_01",
        image      = "skp_guard.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "orp_guard",
        label      = "Orp Guard",
        model      = "s_m_m_orpguard_01",
        image      = "orp_guard.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },

    -- ---- Civilians & Oddities ----
    {
        id         = "wildman",
        label      = "Wild Man",
        model      = "RE_WILDMAN_01",
        image      = "wildman.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "swamp_freak",
        label      = "Swamp Freak",
        model      = "CS_SwampFreak",
        image      = "swamp_freak.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "naked_swimmer",
        label      = "Naked Swimmer",
        model      = "RE_NAKEDSWIMMER_MALES_01",
        image      = "naked_swimmer.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "circus_wagon",
        label      = "Circus Performer",
        model      = "U_M_M_CircusWagon_01",
        image      = "circus.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "gus",
        label      = "Gus MacMillan",
        model      = "CS_MP_GUS_MACMILLAN",
        image      = "gus.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "butcher",
        label      = "Butcher",
        model      = "S_M_M_UNIBUTCHERS_01",
        image      = "butcher.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "vampire",
        label      = "Vampire",
        model      = "cs_vampire",
        image      = "vampire.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "photographer",
        label      = "Photographer",
        model      = "mp_u_m_m_nat_photographer_02",
        image      = "photographer.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
    {
        id         = "obese_woman",
        label      = "Obese Woman",
        model      = "a_f_m_btcobesewomen_01",
        image      = "obese_woman.png",
        category   = "people",
        job        = "",
        aggressive = false,
    },
}
