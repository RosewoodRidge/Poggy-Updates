Config = {}

---------------------------------------------------------------------------
--  GENERAL
---------------------------------------------------------------------------
Config.Debug            = false -- print debug info to console (true/false)
Config.DebugState       = false -- print state-transition audits to F8 console (true/false)
Config.FishingRodItem   = "fishingrod"
Config.ProRodItem       = "fishingrod_pro"
Config.BaitPropDefault  = "p_lgoc_spinner_v4"

Config.HideUndiscoveredFish = true  -- hide fish the player hasn't caught yet from the fishing UI (requires poggy_fishing_journal)

Config.BiteTimeMin = 3
Config.BiteTimeMax = 20

---------------------------------------------------------------------------
--  PRO ROD MODIFIERS
--  Applied when using the Pro Rod.  Changes fishing mode:
--    • No interest/tension phase — fish bites after a random wait
--    • Tension bar runs during the skillcheck phase (must manage both)
--    • Rarer fish are more likely (size weight shift)
--    • Skillcheck gains are reduced, misses hurt more
--    • Better loot drop chance, lower bait loss on escape
---------------------------------------------------------------------------
Config.ProRod = {
    -- Skillcheck multipliers (applied to tier good/great/miss values)
    GoodMult      = 0.65,   -- good hits give 65% of normal progress
    GreatMult     = 0.75,   -- great hits give 75% of normal progress
    MissMult      = 1.5,    -- misses hurt 50% more

    -- Size weight shift: multiplied onto bait sizeWeights / NoBaitSizeWeights
    SizeWeights   = { sm = 0.4, md = 0.8, lg = 2.0, xl = 3.0, legendary = 4.0 },

    -- Bite wait time (seconds) — no interest minigame, just a random wait
    BiteTimeMin   = 4,
    BiteTimeMax   = 15,

    ---------------------------------------------------------------------------
    --  DIRECTION PULL SYSTEM  (runs during the entire fight, alongside skillchecks)
    --
    --  The fish pulls LEFT, RIGHT, or FORWARD.  Direction changes every few sec.
    --  Player must counter by moving their mouse in the matching direction.
    --  Camera look controls are disabled — mouse only moves the pull meter.
    --
    --  Correct pull  → drain is reduced, tension decays
    --  Wrong pull    → drain at full speed, tension rises
    --
    --  Fail states:
    --    Progress ≤ 0   → fish escapes
    --    Tension ≥ 100  → line snaps
    ---------------------------------------------------------------------------

    -- Passive progress drain (fish pulling line back)
    DrainRate         = 0.7,    -- progress lost per second (base, scaled by fish tier)
    DrainOnMiss       = 6.0,    -- extra instant drain on a missed skillcheck

    -- Drain scaling per fish price tier (multiplied onto DrainRate)
    DrainScaling = {
        easy   = 0.7,
        medium = 1.0,
        hard   = 1.3,
        expert    = 1.7,
        legendary = 2.2,
    },

    -- Direction pull: how the fish changes direction
    PullSwitchMin     = 3.0,    -- min seconds before direction changes
    PullSwitchMax     = 6.0,    -- max seconds
    PullDirections    = {"left", "right", "forward"},   -- possible directions

    -- Mouse detection
    MouseSensitivity  = 180,    -- multiplier on raw mouse delta for pull meter speed
    PullReturnForce   = 50,     -- constant pull-back toward center (units/sec) — forward is rest
    NeedleCurve       = 0.6,    -- visual perspective curve (<1 = compress edges, 3D depth feel)
    EdgeSensitivity   = 0.25,   -- mouse sensitivity at full deflection (fraction of full)
    PullThreshold     = 35,     -- pull position must exceed ±this to count as left/right (for anims)
    CorrectThreshold  = 0.20,   -- wrongness below this = "correct" (green highlight)

    -- Drain modifiers based on pull correctness
    CorrectDrainMult  = 0.12,   -- drain * this when pulling correct direction (88% less)
    WrongDrainMult    = 1.2,    -- drain * this when pulling wrong (120% drain)

    -- Tension based on pull correctness
    TensionWrongRate  = 20.0,   -- tension gained per second when pulling wrong
    TensionCorrectDecay = 1.8,  -- tension lost per second when pulling correct
    TensionOnMiss     = 15,     -- extra tension spike on missed skillcheck
    TensionReliefOnHit = 10,    -- tension relieved on successful skillcheck
    TensionSnap       = 100,    -- tension >= this = line snaps

    -- Continuous splash during pull fight
    SplashInterval    = 1400,   -- ms between ambient splashes during pull
    SplashLayers      = 2,      -- layers per ambient splash
    SplashScaleBase   = 0.5,    -- base scale for ambient splashes

    -- Forward jerk: random impulses that shove the pull bar left/right
    -- while the fish is pulling "forward" to keep the player on their toes.
    ForwardJerkMin    = 0.8,    -- min seconds between jerks
    ForwardJerkMax    = 2.0,    -- max seconds between jerks
    ForwardJerkForce  = 20,     -- impulse force added to pullPos (±)

    -- Starting progress (you begin with some line already reeled in)
    StartProgress     = 0,

    -- Loot & bait modifiers
    LootDropExtraRoll = true,   -- roll loot table twice (keep both)
    BaitLoseChanceMult = 0.5,   -- bait loss chance multiplied by 0.5 (halved)
}

---------------------------------------------------------------------------
--  RANDOM LOOT DROPS  (rolled on successful catch)
--  Exactly 1 item is picked per catch, weighted by `chance`.
--  Higher chance = more likely to be selected.
---------------------------------------------------------------------------
Config.LootDrops = {
    { item = "whitepearl",                        label = "White Pearl",              chance = 10.0  },
    { item = "redpearl",                          label = "Red Pearl",                chance = 6.5  },
    { item = "bluepearl",                         label = "Blue Pearl",               chance = 4.5  },
    { item = "goldpearl",                         label = "Gold Pearl",               chance = 2.0  },
    { item = "golden_nugget",                      label = "Golden Nugget",            chance = 2.0  },
    { item = "golden_nugget",                      label = "Golden Nugget",            chance = 2.0  },
    { item = "blackpearl",                        label = "Black Pearl",              chance = 0.5  },
    { item = "diamond_uncut",                     label = "Uncut Diamond",          chance = 0.5  },
    { item = "bait_crawdad",                      label = "Cricket Bait",             chance = 25.0  },
    { item = "p_finishedragonflylegendary01x",    label = "Legendary Dragon Fly Lure", chance = 1.0 },
}

---------------------------------------------------------------------------
--  SKILLCHECK SYSTEM  (one-at-a-time, progressive difficulty)
--  Speed: 3 minimum, 5 maximum.  Rounds: 3 minimum, 10 maximum.
--  Good/Great/Miss affect percentage fill of the progress bar.
--  Consecutive misses escalate difficulty (speed + shake).
---------------------------------------------------------------------------
Config.SkillCheck = {
    SpeedMin      = 3,
    SpeedMax      = 5,
    MaxFails      = 4,          -- total fails before line breaks
    MissSpeedBump = 1,          -- speed increase per consecutive miss

    Tiers = {
        easy   = { speed = 3, diff = 1, rounds = 3,  good = 28, great = 38, miss = -12, shake = true, greatPct = 30, randomizer = 2, shakeSpeed = 1, shakeDist = 2, timeBetween = 250, direction = "cw"   },
        medium = { speed = 3, diff = 2, rounds = 4,  good = 18, great = 26, miss = -8,  shake = true, greatPct = 25, randomizer = 2, shakeSpeed = 2, shakeDist = 2, timeBetween = 250, direction = "cw"   },
        hard   = { speed = 4, diff = 3, rounds = 8,  good = 16, great = 23, miss = -6,  shake = true,  greatPct = 20, randomizer = 2, shakeSpeed = 3, shakeDist = 2, timeBetween = 200, direction = "rand" },
        expert    = { speed = 4, diff = 4, rounds = 10, good = 12, great = 18, miss = -5,  shake = true,  greatPct = 15, randomizer = 2, shakeSpeed = 3, shakeDist = 2, timeBetween = 150, direction = "rand" },
        legendary = { speed = 5, diff = 5, rounds = 12, good = 10, great = 14, miss = -6,  shake = true,  greatPct = 10, randomizer = 3, shakeSpeed = 4, shakeDist = 2, timeBetween = 120, direction = "rand" },
    },
}

---------------------------------------------------------------------------
--  INTEREST / TENSION MECHANIC  (during wait-for-bite phase)
--  Player presses [E] to tug the line — builds both interest and tension.
--  Interest = how attracted the fish is (blue bar, 100% = hooked!)
--  Tension  = stress on the line (red bar, 100% = snap!)
--  Both decay over time when not pressing E.
---------------------------------------------------------------------------
Config.Interest = {
    InitialInterest = 20,
    GracePeriod     = 5000,
    GainPerPress    = 4,        -- was 12 (÷3)
    BaitBonus       = 2,        -- was 6 (÷3)
    RightBaitBonus  = 3.33,     -- was 10 (÷3)
    InterestDecay   = 1.33,     -- was 4 (÷3)
    TensionPerPress = 3.33,     -- was 10 (÷3)
    TensionDecay    = 4,        -- was 12 (÷3)
    PressCooldown   = 150,
    BoreTimeout     = 5000,

    NoBait = {
        interestGain  = 0.3,
        interestDecay = 0.9,
        tensionGain   = 1.2,
        tensionDecay  = 1.25,
        boreTimeout   = 0.6,
        gracePeriod   = 0.6,

        TensionBonus = {
            minTension = 20,
            maxTension = 85,
            bonusMult  = 3.0,
        },
    },

    Scaling = {
        easy   = { interestGain = 1.0,  interestDecay = 1.0,  tensionGain = 1.0,  tensionDecay = 1.0,  boreTimeout = 1.0,  gracePeriod = 1.0  },
        medium = { interestGain = 0.8,  interestDecay = 1.3,  tensionGain = 1.2,  tensionDecay = 0.8,  boreTimeout = 0.8,  gracePeriod = 0.8  },
        hard   = { interestGain = 0.6,  interestDecay = 1.6,  tensionGain = 1.45, tensionDecay = 0.6,  boreTimeout = 0.6,  gracePeriod = 0.6  },
        expert    = { interestGain = 0.45, interestDecay = 2.0,  tensionGain = 1.7,  tensionDecay = 0.45, boreTimeout = 0.4,  gracePeriod = 0.45 },
        legendary = { interestGain = 0.30, interestDecay = 2.5,  tensionGain = 2.0,  tensionDecay = 0.30, boreTimeout = 0.3,  gracePeriod = 0.35 },
    },
}

---------------------------------------------------------------------------
--  BAIT / LURE ITEMS
--  Each bait can grant:
--    - bonus       : reduces skillcheck difficulty by this many levels (1-2)
--    - targetFish  : table of fish keys this bait specifically attracts
--                    (nil = general purpose, works on everything)
--    - loseChance  : 0-100 percent chance the bait is lost when a fish
--                    escapes ("the fish took your bait!").  Line snaps
--                    ALWAYS lose the bait regardless of this value.
--    - sizeWeights : per-size-tier weight multiplier that controls how
--                    likely each size class is to bite.  Higher = more
--                    likely.  Omitted sizes default to 1.
---------------------------------------------------------------------------
Config.Baits = {
    -----------------------------------------------------------------
    -- GENERAL PURPOSE (works everywhere, no fish preference)
    -----------------------------------------------------------------
    ["spinner_v4"] = {
        label       = "Spinner Lure (V4)",
        item        = "p_lgoc_spinner_v4",             -- inventory item name
        prop        = "p_lgoc_spinner_v4",      -- prop model swapped onto line
        bonus       = 0,                        -- no skillcheck bonus
        targetFish  = nil,                      -- catches anything
        loseChance  = 20,                       -- 20% chance fish takes the lure
        sizeWeights = { sm = 5, md = 3, lg = 1, xl = 0.3 },  -- mostly small/med
    },

    -----------------------------------------------------------------
    -- WORM BAIT — attracts catfish, bass, perch (consumed)
    -----------------------------------------------------------------
    ["bait_worm"] = {
        label       = "Worm Bait",
        item        = "bait_worm",
        prop        = "p_baitWorm01x",
        bonus       = 1,                       -- slightly easier checks
        targetFish  = {
            "bluegill_sm", "bluegill_md",
            "bullhead_sm", "bullhead_md",
            "perch_sm", "perch_md",
            "largemouth_md", "largemouth_lg",
            "smallmouth_md", "smallmouth_lg",
        },
        loseChance  = 75,                       -- 75% chance fish takes the worm
        sizeWeights = { sm = 3, md = 4, lg = 2, xl = 0.5 },  -- good all-rounder
    },

    -----------------------------------------------------------------
    -- CRICKET BAIT — attracts pickerel, rock bass, trout (consumed)
    -----------------------------------------------------------------
    ["bait_cricket"] = {
        label       = "Cricket Bait",
        item        = "bait_cricket",
        prop        = "p_baitCricket01x",
        bonus       = 1,
        targetFish  = {
            "chain_pickerel_sm", "chain_pickerel_md",
            "redfin_pickerel_sm", "redfin_pickerel_md",
            "rock_bass_sm", "rock_bass_md",
            "rainbow_trout_md",
            "sockeye_salmon_md",
        },
        loseChance  = 70,                       -- 70% chance fish takes the cricket
        sizeWeights = { sm = 3, md = 4, lg = 2, xl = 0.5 },  -- good all-rounder
    },

    -----------------------------------------------------------------
    -- BREAD — cheap, attracts small / common fish (consumed)
    -----------------------------------------------------------------
    ["bait_bread"] = {
        label       = "Bread Bait",
        item        = "bait_bread",
        prop        = "p_baitBread01x",
        bonus       = 0,
        targetFish  = {
            "bluegill_sm", "bluegill_md",
            "perch_sm", "perch_md",
            "rock_bass_sm",
            "bullhead_sm",
        },
        loseChance  = 80,                       -- 80% chance fish takes the bread
        sizeWeights = { sm = 6, md = 2, lg = 0.5, xl = 0 },  -- almost all small
    },

    -----------------------------------------------------------------
    -- CRAWDAD BAIT — attracts large bottom-feeders (consumed)
    -----------------------------------------------------------------
    ["bait_crawdad"] = {
        label       = "Crawdad Bait",
        item        = "bait_crawdad",
        prop        = "p_crawdad01x",
        bonus       = 2,                      -- big bonus for targeting big fish
        targetFish  = {
            "longnose_gar",
            "northern_pike",
        },
        loseChance  = 60,                       -- 60% chance fish takes the crawdad
        sizeWeights = { sm = 0.5, md = 2, lg = 5, xl = 3 },  -- targets big fish
    },

    -----------------------------------------------------------------
    -- LEGENDARY LURE — dragon fly pattern (reusable, rare drop)
    -----------------------------------------------------------------
    ["p_finishedragonflylegendary01x"] = {
        label       = "Legendary Dragon Fly Lure",
        item        = "p_finishedragonflylegendary01x",
        prop        = "p_finishedragonflylegendary01x",
        bonus       = 2,
        targetFish  = {
            "longnose_gar",
            "northern_pike",
            "sockeye_salmon_md", "sockeye_salmon_ml", "sockeye_salmon_lg",
            "largemouth_lg",
            "smallmouth_lg",
            -- Legendary fish (only lure that can hook them)
            "legendary_rainbow_trout",
            "legendary_muskie",
            "legendary_sturgeon",
            "legendary_channel_catfish",
        },
        loseChance  = 5,                       -- 5% chance (legendary is hard to lose)
        sizeWeights = { sm = 0.3, md = 1, lg = 4, xl = 5, legendary = 3 },  -- legendaries possible but not guaranteed
    },
}

---------------------------------------------------------------------------
--  FISH DEFINITIONS
--  key           : unique ID used everywhere in the script
--  label         : display name
--  item          : VORP inventory item awarded on catch
--  model         : ped / entity model for the "fish in hands" display
--  price         : base sale value — ALSO drives difficulty:
--                  price < 1.00   → easy      (speed 3, diff 1)
--                  price < 3.00   → medium    (speed 3, diff 2)
--                  price < 6.00   → hard      (speed 4, diff 3)
--                  price < 20.00  → expert    (speed 4, diff 4)
--                  price >= 20.00 → legendary (speed 5, diff 5)
--  size          : "sm", "md", "lg", "xl" — used for unhook animation pick
--  weightMin     : minimum random weight in lbs
--  weightMax     : maximum random weight in lbs
--  hours         : (optional) {start, end} — 24-hour range when this fish
--                  is available.  Wraps midnight, e.g. {20, 5} = 8PM–5AM.
--                  Omit for fish that are available all day.
---------------------------------------------------------------------------
Config.Fish = {
    -----------------------------------------------------------------
    -- SMALL / EASY
    -----------------------------------------------------------------
    ["bluegill_sm"] = {
        label  = "Bluegill (Small)",
        item   = "a_c_fishbluegil_01_sm",
        model  = "A_C_FISHBLUEGIL_01_SM",
        price  = 0.40,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 4.12,
    },
    ["perch_sm"] = {
        label  = "Perch (Small)",
        item   = "a_c_fishperch_01_sm",
        model  = "A_C_FISHPERCH_01_SM",
        price  = 0.35,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 2.1,
    },
    ["rock_bass_sm"] = {
        label  = "Rock Bass (Small)",
        item   = "a_c_fishrockbass_01_sm",
        model  = "A_C_FISHROCKBASS_01_SM",
        price  = 0.40,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 1.5,
    },
    ["redfin_pickerel_sm"] = {
        label  = "Redfin Pickerel (Small)",
        item   = "a_c_fishredfinpickerel_01_sm",
        model  = "A_C_FISHREDFINPICKEREL_01_SM",
        price  = 0.50,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 1.0,
    },
    ["bullhead_sm"] = {
        label  = "Bullhead Catfish (Small)",
        item   = "a_c_fishbullheadcat_01_sm",
        model  = "A_C_FISHBULLHEADCAT_01_SM",
        price  = 0.45,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 7.6,
        hours  = {20, 6},  -- nocturnal: 8PM–6AM
    },
    ["chain_pickerel_sm"] = {
        label  = "Chain Pickerel (Small)",
        item   = "a_c_fishchainpickerel_01_sm",
        model  = "A_C_FISHCHAINPICKEREL_01_SM",
        price  = 0.55,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 4.0,
        hours  = {5, 12},  -- morning feeder: 5AM–12PM
    },
    ["sockeye_salmon_sm"] = {
        label  = "Sockeye Salmon (Small)",
        item   = "a_c_fishsalmonsockeye_01_sm",
        model  = "A_C_FISHSALMONSOCKEYE_01_MS",
        price  = 0.60,
        size   = "sm",
        weightMin = 0.1,
        weightMax = 3.0,
        hours  = {4, 10},  -- dawn/morning: 4AM–10AM
    },

    -----------------------------------------------------------------
    -- MEDIUM
    -----------------------------------------------------------------
    ["bluegill_md"] = {
        label  = "Bluegill (Medium)",
        item   = "a_c_fishbluegil_01_ms",
        model  = "A_C_FISHBLUEGIL_01_MS",
        price  = 0.75,
        size   = "md",
        weightMin = 0.1,
        weightMax = 4.12,
    },
    ["perch_md"] = {
        label  = "Perch (Medium)",
        item   = "a_c_fishperch_01_ms",
        model  = "A_C_FISHPERCH_01_MS",
        price  = 0.80,
        size   = "md",
        weightMin = 2.2,
        weightMax = 4.3,
    },
    ["rock_bass_md"] = {
        label  = "Rock Bass (Medium)",
        item   = "a_c_fishrockbass_01_ms",
        model  = "A_C_FISHROCKBASS_01_MS",
        price  = 0.85,
        size   = "md",
        weightMin = 1.6,
        weightMax = 3.0,
    },
    ["bullhead_md"] = {
        label  = "Bullhead Catfish (Medium)",
        item   = "a_c_fishbullheadcat_01_ms",
        model  = "A_C_FISHBULLHEADCAT_01_MS",
        price  = 1.00,
        size   = "md",
        weightMin = 7.7,
        weightMax = 11.0,
        hours  = {20, 6},  -- nocturnal: 8PM–6AM
    },
    ["redfin_pickerel_md"] = {
        label  = "Redfin Pickerel (Medium)",
        item   = "a_c_fishredfinpickerel_01_ms",
        model  = "A_C_FISHREDFINPICKEREL_01_MS",
        price  = 1.10,
        size   = "md",
        weightMin = 1.1,
        weightMax = 2.1,
        hours  = {15, 22},  -- evening feeder: 3PM–10PM
    },
    ["chain_pickerel_md"] = {
        label  = "Chain Pickerel (Medium)",
        item   = "a_c_fishchainpickerel_01_ms",
        model  = "A_C_FISHCHAINPICKEREL_01_MS",
        price  = 1.25,
        size   = "md",
        weightMin = 4.1,
        weightMax = 9.6,
        hours  = {5, 12},  -- morning feeder: 5AM–12PM
    },
    ["largemouth_md"] = {
        label  = "Largemouth Bass (Medium)",
        item   = "a_c_fishlargemouthbass_01_ms",
        model  = "A_C_FISHLARGEMOUTHBASS_01_MS",
        price  = 1.50,
        size   = "md",
        weightMin = 2.1,
        weightMax = 9.0,
        hours  = {4, 11},  -- dawn feeder: 4AM–11AM
    },
    ["smallmouth_md"] = {
        label  = "Smallmouth Bass (Medium)",
        item   = "a_c_fishsmallmouthbass_01_ms",
        model  = "A_C_FISHSMALLMOUTHBASS_01_MS",
        price  = 1.60,
        size   = "md",
        weightMin = 2.0,
        weightMax = 6.0,
        hours  = {6, 14},  -- morning/midday: 6AM–2PM
    },
    ["sockeye_salmon_md"] = {
        label  = "Sockeye Salmon (Medium)",
        item   = "a_c_fishsalmonsockeye_01_ms",
        model  = "A_C_FISHSALMONSOCKEYE_01_MS",
        price  = 2.25,
        size   = "md",
        weightMin = 3.0,
        weightMax = 6.0,
        hours  = {4, 10},  -- dawn/morning: 4AM–10AM
    },
    ["rainbow_trout_md"] = {
        label  = "Rainbow Trout (Medium)",
        item   = "a_c_fishrainbowtrout_01_ms",
        model  = "A_C_FISHRAINBOWTROUT_01_MS",
        price  = 1.20,
        size   = "md",
        weightMin = 5.0,
        weightMax = 24.0,
        hours  = {5, 21},  -- daytime: 5AM–9PM
    },

    -----------------------------------------------------------------
    -- LARGE / HARD
    -----------------------------------------------------------------
    ["largemouth_lg"] = {
        label  = "Largemouth Bass (Large)",
        item   = "a_c_fishlargemouthbass_01_lg",
        model  = "A_C_FISHLARGEMOUTHBASS_01_LG",
        price  = 3.50,
        size   = "lg",
        weightMin = 9.1,
        weightMax = 22.0,
        hours  = {4, 10},  -- dawn: 4AM–10AM
    },
    ["smallmouth_lg"] = {
        label  = "Smallmouth Bass (Large)",
        item   = "a_c_fishsmallmouthbass_01_lg",
        model  = "A_C_FISHSMALLMOUTHBASS_01_LG",
        price  = 4.00,
        size   = "lg",
        weightMin = 6.1,
        weightMax = 11.15,
        hours  = {5, 11},  -- early morning: 5AM–11AM
    },

    ["sockeye_salmon_lg"] = {
        label  = "Sockeye Salmon (Large)",
        item   = "a_c_fishsalmonsockeye_01_lg",
        model  = "A_C_FISHSALMONSOCKEYE_01_LG",
        price  = 5.00,
        size   = "lg",
        weightMin = 10.1,
        weightMax = 15.3,
        hours  = {4, 10},  -- dawn/morning: 4AM–10AM
    },
    ["sockeye_salmon_ml"] = {
        label  = "Sockeye Salmon (Medium-Large)",
        item   = "a_c_fishsalmonsockeye_01_ml",
        model  = "A_C_FISHSALMONSOCKEYE_01_ML",
        price  = 3.50,
        size   = "lg",
        weightMin = 6.1,
        weightMax = 10.0,
        hours  = {4, 10},  -- dawn/morning: 4AM–10AM
    },
    ["steelhead_trout_lg"] = {
        label  = "Steelhead Trout (Large)",
        item   = "a_c_fishsteelheadtrout",
        model  = "A_C_FISHRAINBOWTROUT_01_LG",
        price  = 4.00,
        size   = "lg",
        weightMin = 8.0,
        weightMax = 20.0,
        hours  = {5, 11},  -- dawn/morning: 5AM–11AM
    },

    -----------------------------------------------------------------
    -- EXTRA LARGE / EXPERT  (trophy fish — exclusive locations)
    -----------------------------------------------------------------

    ["northern_pike"] = {
        label  = "Northern Pike (Large)",
        item   = "a_c_fishnorthernpike_01_lg",
        model  = "A_C_FISHNORTHERNPIKE_01_LG",
        price  = 7.00,
        size   = "lg",
        weightMin = 15.0,
        weightMax = 55.1,
        hours  = {4, 11},  -- early morning: 4AM–11AM
    },
    ["longnose_gar"] = {
        label  = "Longnose Gar (Large)",
        item   = "a_c_fishlongnosegar_01_lg",
        model  = "A_C_FISHLONGNOSEGAR_01_LG",
        price  = 7.50,
        size   = "lg",
        weightMin = 10.0,
        weightMax = 56.2,
        hours  = {20, 6},  -- nocturnal: 8PM–6AM
    },



    -----------------------------------------------------------------
    -- LEGENDARY  (requires Dragon Fly Lure — size "legendary" has
    -- zero weight on all other baits/no-bait, so only the legendary
    -- lure can hook them.  Even then they're rare, not guaranteed.)
    -----------------------------------------------------------------
    ["legendary_rainbow_trout"] = {
        label  = "Legendary Rainbow Trout",
        item   = "a_c_fishrainbowtrout_01_lg",
        model  = "A_C_FISHRAINBOWTROUT_01_LG",
        price  = 35.00,
        size   = "legendary",
        weightMin = 50.0,
        weightMax = 96.0,
        hours  = {5, 9},   -- brief dawn window: 5AM–9AM
    },
    ["legendary_muskie"] = {
        label  = "Legendary Muskellunge",
        item   = "a_c_fishmuskie_01_lg",
        model  = "A_C_FISHMUSKIE_01_LG",
        price  = 35.00,
        size   = "legendary",
        weightMin = 70.0,
        weightMax = 120.0,
        hours  = {4, 7},   -- narrow dawn twilight: 4AM–7AM
    },
    ["legendary_sturgeon"] = {
        label  = "Legendary Lake Sturgeon",
        item   = "a_c_fishlakesturgeon_01_lg",
        model  = "A_C_FISHLAKESTURGEON_01_LG",
        price  = 35.00,
        size   = "legendary",
        weightMin = 80.0,
        weightMax = 150.0,
        hours  = {20, 23},  -- brief night window: 8PM–11PM
    },
    ["legendary_channel_catfish"] = {
        label  = "Legendary Channel Catfish",
        item   = "a_c_fishchannelcatfish_01_xl",
        model  = "a_c_fishchannelcatfish_01_lg",
        price  = 35.00,
        size   = "legendary",
        weightMin = 60.0,
        weightMax = 110.0,
        hours  = {22, 2},   -- deep night: 10PM–2AM
    },
}

---------------------------------------------------------------------------
--  SOUND EFFECTS (played via NUI — mp3 files in ui/sfx/)
--  Each entry: { file = "sfx/filename.mp3", volume = 0.4 }
--  Lua sends  SendNUIMessage({ action="playSound", id="cast", ... })
--  Music entries have loop = true.
---------------------------------------------------------------------------
Config.Sounds = {
    -- UI / feedback
    cast        = { file = "sfx/cast.mp3",         volume = 0.05 },
    splash      = { file = "sfx/splash.mp3",       volume = 0.2 },
    bite        = { file = "sfx/bite.mp3",          volume = 0.2 },
    tug         = { file = "sfx/tug.mp3",           volume = 0.4 },
    reel        = { file = "sfx/reel.mp3",          volume = 0.1 },
    caught      = { file = "sfx/caught.mp3",        volume = 0.4 },
    escaped     = { file = "sfx/escaped.mp3",       volume = 0.4 },
    lineSnap    = { file = "sfx/lineSnap.mp3",      volume = 0.2 },
    scared      = { file = "sfx/scared.mp3",         volume = 0.4 },
    equipRod    = { file = "sfx/equiprod.mp3",       volume = 0.4 },
    stowRod     = { file = "sfx/stowrod.mp3",        volume = 0.4 },
    baitSelect  = { file = "sfx/baitSelect.mp3",    volume = 0.3 },
    fishSplash  = { file = "sfx/fishSplash.mp3",    volume = 0.25 },
    keepFish    = { file = "sfx/keep_fish.mp3",     volume = 0.4 },
    throwBack   = { file = "sfx/throw_back.mp3",    volume = 0.4 },

    -- Music (loops)
    fishingMusic_1 = { file = "sfx/fishingMusic_1.mp3", volume = 0.15, loop = true },
    fishingMusic_2 = { file = "sfx/fishingMusic_2.mp3", volume = 0.07, loop = true },
    fishingMusic_3 = { file = "sfx/fishingMusic_3.mp3", volume = 0.07, loop = true },
    fishingMusic_4 = { file = "sfx/fishingMusic_4.mp3", volume = 0.08, loop = true },
}

---------------------------------------------------------------------------
--  MUSIC TRACKS  (cycled with C key while fishing)
--  Each entry: { id = sound id, label = display name }
--  The first entry is always "Off" (no music).
--  Sound ids must match a key in Config.Sounds above.
---------------------------------------------------------------------------
Config.MusicTracks = {
    { id = nil,                label = "Off" },
    { id = "fishingMusic_1",   label = "Peaceful Waters" },
    { id = "fishingMusic_2",   label = "Serene Lakes" },
    { id = "fishingMusic_3",   label = "Circle of Stones" },
    { id = "fishingMusic_4",   label = "Mountain Banjo" },
}

---------------------------------------------------------------------------
--  SEASON SYSTEM
--  On server start, the server randomly picks ActivePercent% of all fish
--  to be "in season."  Out-of-season fish cannot be caught.
--  Resets every server restart or via /rerollseasons command.
---------------------------------------------------------------------------
Config.Season = {
    ActivePercent = 80,   -- % of fish randomly active each restart
}

