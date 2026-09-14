Config = Config or {}

-- ============================================================================
-- poggy_util - optional server utilities
-- Every utility has its own Enabled switch below. Framework features
-- (characters, jobs, duty, money, inventory, notifications) come from
-- poggy_core, so there is no framework setting here.
-- ============================================================================

-- Print debug lines from every utility to the console (F8 on clients).
Config.Debug = false

-- ── Help Menu ────────────────────────────────────────────────────────────────
-- /help opens the searchable guide. Its content is in config/help.lua (example
-- content: replace it with your own).
Config.Help = {
    Enabled = true,
    ServerName = "Your Server Name",  -- shown in the guide's header
    Logo = "",                        -- optional image left of the name: a URL
                                      -- (https://example.com/logo.png) or a file
                                      -- in ui/ that you also list under files {}
                                      -- in fxmanifest.lua. Empty = no logo.
}

Config.WeaponJam = {
    Enabled = true, -- Master switch for the weapon jamming feature
    JamCheckInterval = 250, -- Milliseconds between jam checks when player is shooting
    JamStartThreshold = 0.2, -- Degradation value (0.0 to 1.0) at which jamming can start. Default: 0.1
    JamChanceExponent = 1.5, -- Controls how steeply the jam chance increases with degradation. Higher values = steeper curve.
    MaxJamProbability = 0.25, -- Maximum probability of a jam occurring (0.0 to 1.0). Default: 0.5 (50%)
    CleanlinessCheckInterval = 5000, -- Milliseconds between checking if jammed weapons are now clean
    UnjamThreshold = 0.05, -- If weapon degradation is below this value, a jammed weapon will be automatically unjammed
    
    -- Jam Sound Configuration
    JamSound = {
        Enabled = true, -- Play sound when weapon jams
        Gain = 0.3, -- Volume/gain of the jam sound (0.0 to 1.0)
        MaxDistance = 50.0, -- Maximum distance (in meters) at which the jam sound can be heard
        Falloff = 2.0, -- Volume falloff exponent (2.0 = realistic inverse square law, 1.0 = linear)
        Sounds = { -- List of sound files to randomly play on jam (relative to ui/sfx/weaponjam/)
            "gun_empty1.wav",
            "gun_empty2.wav",
            "gun_empty3.wav"
        }
    },
    
    GunsToJam = {
        "WEAPON_PISTOL_SEMIAUTO", "WEAPON_PISTOL_MAUSER", "WEAPON_PISTOL_VOLCANIC",
        "WEAPON_PISTOL_M1899", "WEAPON_REVOLVER_SCHOFIELD", "WEAPON_REVOLVER_NAVY",
        "WEAPON_REVOLVER_NAVY_CROSSOVER", "WEAPON_REVOLVER_LEMAT", "WEAPON_REVOLVER_DOUBLEACTION",
        "WEAPON_REVOLVER_CATTLEMAN", "WEAPON_REVOLVER_CATTLEMAN_MEXICAN", "WEAPON_RIFLE_VARMINT",
        "WEAPON_REPEATER_WINCHESTER", "WEAPON_REPEATER_HENRY", "WEAPON_REPEATER_EVANS",
        "WEAPON_REPEATER_CARBINE", "WEAPON_SNIPERRIFLE_ROLLINGBLOCK", "WEAPON_SNIPERRIFLE_CARCANO",
        "WEAPON_RIFLE_SPRINGFIELD", "WEAPON_RIFLE_ELEPHANT", "WEAPON_RIFLE_BOLTACTION",
        "WEAPON_SHOTGUN_SEMIAUTO", "WEAPON_SHOTGUN_SAWEDOFF", "WEAPON_SHOTGUN_REPEATING",
        "WEAPON_SHOTGUN_DOUBLEBARREL_EXOTIC", "WEAPON_SHOTGUN_PUMP", "WEAPON_SHOTGUN_DOUBLEBARREL"
    }
}

-- Government Stipend Configuration
Config.Stipend = {
    Enabled = false,
    IntervalMinutes = 60,              -- How often to check/pay (in minutes)
    BasePay = 5.00,                    -- Base payment amount
    IncreaseAmount = 0.25,             -- Amount to increase pay by
    IncreaseIntervalDays = 30,         -- Days required for each pay increase
    MinimumTenureDays = 30,            -- Minimum days on server to be eligible
    UnemployedJobName = "unemployed",  -- Job name to check for (case-insensitive)
    LegacyDate = "2025-11-01 00:00:00" -- Creation date given to characters that existed before poggy_util (YYYY-MM-DD HH:MM:SS).
                                       -- Informational: the date is written once by sql/install.sql when it adds
                                       -- characters.created_at. To use another date, change it in that file too,
                                       -- before poggy_util first starts.
}

-- Object removal configuration
Config.ObjectRemoval = {
    Enabled = true,                   -- Master toggle for object removal feature
    IntervalMinutes = 1,              -- Check every X minutes for objects to remove
    Objects = {
        "s_coachlock02x"              -- Default object model to remove
    },
    AdminGroups = {"admin", "superadmin", "moderator"}, -- Groups that can use the removeobjects command
    
    -- Dead entity cleanup settings
    DeadEntityCleanup = {
        Enabled = false,               -- Master toggle for dead entity cleanup
        IntervalMinutes = 60,         -- Check every X minutes for dead entities
        CleanPeds = true,             -- Remove dead NPCs (not players)
        CleanHorses = true,           -- Remove dead horses
        CleanAnimals = true,          -- Remove dead animals (deer, wolves, etc.)
        CleanWagons = true            -- Remove abandoned wagons
    }
}

-- Armor Protection configuration
-- When enabled, any shot that lands on the torso / chest / abdomen bone zone
-- while the player has a clothing component equipped in the native "armor" slot
-- is fully blocked (health is immediately restored to its pre-shot value).
-- The "armor" slot is the dedicated RDR3 body-armour clothing category used by
-- vorp_character, jo_libs, and the jo_clothingstore / kd systems.
Config.ArmorProtection = {
    Enabled = false,               -- Master toggle for the armor protection feature

    -- Minimum HP drop (before it's treated as a real shot rather than a
    -- health-tick fluctuation).  Keep this low (1–2) for all bullet types.
    MinDamageThreshold = 1,

    -- Player-facing notification when armor absorbs a hit
    NotifyPlayer     = false,
    NotifyMessage    = "Your armor absorbed the hit!",
    NotifyDuration   = 2500,       -- ms to display the notification (absorbed hit)
    NotifyCooldown   = 3000,       -- ms between notifications (prevents spam on rapid fire)

    -- ── Durability system ──────────────────────────────────────────────────
    -- The armor starts at MaxShots. Each absorbed chest/abdomen shot costs 1.
    -- When shots reach 0 the armor stops blocking damage entirely.
    -- To repair, the player uses RepairKitItem from their inventory.
    MaxShots     = 10,             -- Total absorbed hits before armor breaks
    ShotsPerStage = 2,             -- Absorbed hits consumed per tank-image stage (10 stages total)
    RepairKitItem = "armor_kit",   -- Inventory item name for the repair kit (sql/install.sql adds armor_kit)
    RepairTime    = 8000,          -- ms for the repair animation + progress bar

    -- ── HUD – armor durability indicator ─────────────────────────────────
    -- Displays rpg_tank_1 … rpg_tank_10 (or empty_bg when broken) at a
    -- fixed corner of the screen.  All position values are CSS strings.
    HUD = {
        Enabled     = true,
        Bottom      = "80px",      -- distance from the bottom edge
        Left        = "30px",      -- distance from the left edge
        Right       = nil,         -- set this (and clear Left) to anchor to the right side
        ImageWidth  = "44px",      -- reduced circle size (was 80px)
        ImageHeight = "44px",
        ShieldSize  = "25px",      -- size of the armor.png shield icon in the center
    },

    -- ── Repair progress bar position ──────────────────────────────────────
    RepairBar = {
        Bottom = "90px",           -- distance from the bottom edge (centered horizontally)
        Width  = "300px",          -- width of the progress bar widget
    },
}

-- Music Zone configuration
Config.MusicZones = {
    Enabled = true,                    -- Master toggle for music zones
    UnloadDistance = 100.0,            -- Distance to unload the video completely (save resources)
    Quality = 'small',                 -- YouTube quality: 'small' (240p), 'medium' (360p), 'large' (480p), 'hd720', 'hd1080', 'highres'
    VolumeMultiplier = 0.5,            -- Global volume multiplier (0.0 - 1.0)
    Falloff = 2.0,                     -- Volume falloff exponent (2.0 = realistic inverse square law, 1.0 = linear)
    DirectionalAudio = true,           -- Simulate 3D audio by lowering volume when looking away
    Zones = {
        {
            name = "Sousa Marches",
            youtubeId = "h8RnD9Qz1tI",
            radius = 25.0,
            volume = 0.25,
            loop = true,
            timestamps = {
                0,    -- 0:00 - Semper Fidelis (1888)
                164,  -- 2:44 - The Thunderer (1889)
                328,  -- 5:28 - The Washington Post (1889)
                481,  -- 8:01 - The High School Cadets (1890)
                635,  -- 10:35 - The Liberty Bell (1893)
                848,  -- 14:08 - Manhattan Beach (1893)
                981,  -- 16:21 - El Capitan (1896)
                1118, -- 18:38 - The Stars and Stripes Forever (1896)
                1331, -- 22:11 - Hands Across the Sea (1899)
                1497  -- 24:57 - The Invincible Eagle (1901)
            },
            locations = {
                vector3(2401.08, -1115.61, 46.52)
            }
        },
        -- Add more zones by copying the block above (name, youtubeId, radius, volume, loop, timestamps, locations).
    }
}

-- Unstuck configuration
Config.Unstuck = {
    Enabled = true,                    -- Master toggle for unstuck feature
    Command = "unstuck",               -- Command name to trigger unstuck
    DelaySeconds = 10,                 -- Seconds to wait before teleporting (player must stay still)
    MoveTolerance = 2.0                -- Distance (meters) player can drift before cancelling
}

-- Area of Play (AOP) configuration
Config.AOP = {
    Enabled = true,                    -- Master toggle for AOP display
    UpdateInterval = 20,               -- Seconds between automatic zone updates
    SearchRadius = 1000.0,              -- Meters to search for nearest zone node
    Zones = {
        {
            name = "Valentine Zone",
            coords = vector3(-300.32, 790.24, 118.16)
        },
        {
            name = "Strawberry Zone",
            coords = vector3(-1790.0, -390.0, 160.0)
        },
        {
            name = "Rhodes Zone",
            coords = vector3(1360.0, -1300.0, 77.0)
        },
        {
            name = "Blackwater Zone",
            coords = vector3(-813.0, -1324.0, 44.0)
        },
        {
            name = "Saint Denis Zone",
            coords = vector3(2439.69, -1185.52, 46.66)
        },
        {
            name = "Armadillo Zone",
            coords = vector3(-3685.0, -2623.0, -13.0)
        },
        {
            name = "Tumbleweed Zone",
            coords = vector3(-5512.0, -2950.0, -2.0)
        },
        {
            name = "Emerald Ranch Zone",
            coords = vector3(1426.88, 328.73, 88.44)
        },
        {
            name = "Van Horn Zone",
            coords = vector3(2974.0, 570.0, 44.0)
        },
        {
            name = "Annesburg Zone",
            coords = vector3(2930.0, 1290.0, 44.0)
        }
    }
}

-- ── Duty Count HUD ───────────────────────────────────────────────────────────
-- Displays on-duty law and medical personnel counts inside the AOP display.
-- Law officers are shown in pastel red; doctors in pastel blue.
-- The display respects the /aop toggle and side setting.
Config.DutyCount = {
    Enabled = true,
    UpdateInterval = 10,  -- Seconds between server-side count refreshes

    -- Jobs that count as active law enforcement in the HUD display.
    -- Must match the job strings stored in character data (case-sensitive).
    LawJobs = {
        police = true,
        sheriff = true,
        marshal = true,
    },
}

-- ── Clock & Weather Widget ───────────────────────────────────────────────────
-- Displays an animated clock with sun/moon and weather icon below the AOP.
-- Reads game time and weather from the weathersync resource.
Config.ClockWeather = {
    Enabled = true,
    UpdateInterval = 5,  -- Seconds between server→client time/weather broadcasts
}

