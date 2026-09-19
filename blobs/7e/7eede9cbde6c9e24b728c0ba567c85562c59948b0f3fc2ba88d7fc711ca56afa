-- Initialize Config table if it doesn't exist
Config = Config or {}

Config.LawResponseDebugging = false -- Enable or disable debugging for law response system

Config.LawResponse = {
    Enabled = false,                  -- Master toggle for the law response system
    ResponseTime = 8000,             -- Time in ms between witness report and law arrival (8 seconds). A fixed number: math.random() here would be rolled once at load, not per report
    Duration = 300000,               -- Maximum duration of response before auto-despawn (ms; 300000 = 5 minutes)
    MinOfficers = 3,                 -- Minimum number of officers in a response
    MaxOfficers = 8,                 -- Maximum number of officers in a response
    SpawnDistance = 80.0,            -- Distance from player where officers spawn
    DespawnDistance = 100.0,         -- Distance at which officers despawn when player escapes
    EnableBlips = true,              -- Show officer blips on the map
    BlipSprite = "blip_ambient_law", -- Blip sprite to use for officers
    BlipScale = 1.2,                 -- Size of officer blips
    -- NPC Law Response Priority Settings
    -- When enabled, NPC law will NOT spawn if player LEOs are on duty
    -- This allows player law enforcement to handle all law responses instead
    PlayerLEOPriority = true, -- If true, NPC law only spawns when no player LEOs are on duty
    -- Engaged Crimes: Only these crime types will trigger NPC law enforcement response
    -- Crime types not in this list will still alert player LEOs but won't spawn NPC law
    -- Available crime types: "Shooting", "Threatening", "Melee", "Lassoing", "Trampling", "Hijacking"
    EngagedCrimes = {
        "Shooting",    -- Firearms discharge
        "Hijacking",   -- Stealing horses/wagons
        -- Add or remove crime types as needed:
        -- "Threatening", -- Aiming weapons without shooting
        -- "Melee",       -- Fist fights and melee attacks
        -- "Lassoing",    -- Hogtying/lassoing people  
        -- "Trampling",   -- Running over people with horses
    },
    -- Surrender System Configuration
    AllowSurrender = true,           -- Enable/disable surrender system
    
    -- Replace single JailTime with severity-based table
    JailTime = {
        LOW = 10,     -- 10 minutes for low severity crimes
        MEDIUM = 20,  -- 20 minutes for medium severity crimes
        HIGH = 30     -- 30 minutes for high severity crimes
    },
    
    SurrenderMovementTolerance = 2.0, -- How far player can move while surrendering (in game units)
    
    -- Jail System Configuration
    -- Empty by default: stock VORP has no jail API. vorp_police only jails
    -- through its /jail command and exports no jail function, so there is no
    -- call this resource can make on a stock server. With "" the NPC arrest
    -- still happens (surrender, arrest notice, law response ends) and the
    -- JailTime table above is simply not used.
    -- To add jail time, point this at a jail script that has a server export.
    -- The string is run as Lua on the server. These placeholders are replaced
    -- with the real values first:
    -- src - The player's server ID (source)
    -- charId - The player's character identifier
    -- timeInSeconds - The jail time in seconds
    -- Example for a jail script with an export:
    -- JailExportCall = "exports.another_jail:JailPlayer(src, timeInSeconds, 'Arrested by law')"
    JailExportCall = "",
    
    -- Horse models for mounted officers
    HorseModels = {
        "A_C_HORSE_TENNESSEEWALKER_BLACKRABICANO",
        "a_c_horse_americanstandardbred_black",
        "a_c_horse_missourifoxtrotter_blueroan",
        "a_c_horse_mustang_grullodun"
    },
    
    -- Weapons officers might carry
    Weapons = {
        "WEAPON_SHOTGUN_PUMP",
        "WEAPON_SNIPERRIFLE_CARCANO",
        "WEAPON_SHOTGUN_DOUBLEBARREL",  
        "WEAPON_PISTOL_M1899"
   },
    
    -- assign a severity level per crime type
    CrimeSeverity = {
        Shooting    = "HIGH",
        Threatening = "MEDIUM",
        Melee       = "MEDIUM",
        Lassoing    = "LOW",
        Trampling   = "MEDIUM",
        Hijacking   = "MEDIUM"
    },
    
    -- define behavior per severity
    ResponseTypes = {
        LOW = {
            MinOfficers    = 2,
            MaxOfficers    = 3,
            Aggressiveness = 1,
            MountedPercent = 25,
            Probability    = 100   -- % chance to spawn any officers
        },
        MEDIUM = {
            MinOfficers    = 3,
            MaxOfficers    = 5,
            Aggressiveness = 3,
            MountedPercent = 50,
            Probability    = 100  
        },
        HIGH = {
            MinOfficers    = 5,
            MaxOfficers    = 6,
            Aggressiveness = 5,
            MountedPercent = 75,
            Probability    = 100  
        }
    },

    -- Notification settings (use translation keys)
    Notifications = {
        LawDefeated = {
            title    = "LAW_DEFEATED_TITLE",
            message  = "LAW_DEFEATED_MESSAGE",
            duration = 5000
        },
        LawEscaped = {
            title    = "LAW_ESCAPED_TITLE",
            message  = "LAW_ESCAPED_MESSAGE",
            duration = 5000
        }
    },
}

-- Law officer models for specific towns
Config.TownLawModels = {
    ["Saint Denis"] = {
        "mp_fm_track_sd_lawman_01", 
        "s_m_m_ambientsdpolice_01", 
        "u_m_m_sdpolicechief_01"
    },
    ["Rhodes"] = {
        "cs_rhodeputy_01", 
        "cs_rhodeputy_02", 
        "u_m_m_rhdsheriff_01", 
        "mp_u_m_m_lawcamp_lawman_02", 
        "u_m_m_rhdbackupdeputy_02"
    },
    ["Annesburg"] = {
        "mp_u_m_m_lawcamp_lawman_01", 
        "mp_u_m_m_lawcamp_lawman_02", 
        "a_m_m_valdeputyresident_01", 
        "cs_valsheriff"
    },
    ["Valentine"] = {
        "cs_valdeputy_01", 
        "cs_valsheriff", 
        "s_m_m_valdeputy_01", 
        "u_m_m_valsheriff_01"
    },
    ["Emerald Ranch"] = {
        "cs_rhodeputy_01", 
        "cs_rhodeputy_02", 
        "u_m_m_rhdsheriff_01", 
        "mp_u_m_m_lawcamp_lawman_02", 
        "cs_valdeputy_01", 
        "cs_valsheriff", 
        "s_m_m_valdeputy_01", 
        "u_m_m_valsheriff_01"
    },
    ["Strawberry"] = {
        "cs_strsheriff_01", 
        "a_m_m_strdeputyresident_01", 
        "cs_strdeputy_01", 
        "cs_strdeputy_02"
    },
    ["Blackwater"] = {
        "cs_mp_sherifffreeman", 
        "MP_S_M_M_PinLaw_01", 
        "cs_pinkertongoon", 
        "s_m_m_pinlaw_01"
    },
    ["Tumbleweed"] = {
        "a_m_m_armdeputyresident_01", 
        "MP_U_M_M_ARMSHERIFF_01", 
        "s_m_m_tumdeputies_01"
    },
    ["Armadillo"] = {
        "a_m_m_armdeputyresident_01", 
        "MP_U_M_M_ARMSHERIFF_01", 
        "s_m_m_tumdeputies_01"
    },
    ["Colter"] = {
        "MP_RESCUE_COLTER_MALES_01", 
        "g_m_m_bountyhunters_01", 
        "MP_G_M_M_BOUNTYHUNTERS_01", 
        "u_m_m_unibountyhunter_01", 
        "u_m_m_unibountyhunter_02", 
        "mp_fm_bounty_horde_law_01"
    },
    ["Thieves Landing"] = {
        "cs_mp_sherifffreeman", 
        "MP_S_M_M_PinLaw_01", 
        "cs_pinkertongoon", 
        "s_m_m_pinlaw_01"
    }
}

-- Default models for when player is not in a specific town
Config.DefaultLawModels = {
    "MP_RESCUE_COLTER_MALES_01", 
    "g_m_m_bountyhunters_01", 
    "MP_G_M_M_BOUNTYHUNTERS_01", 
    "u_m_m_unibountyhunter_01", 
    "u_m_m_unibountyhunter_02", 
    "mp_fm_bounty_horde_law_01"
}
