Config = Config or {}

-- ============================================================================
-- DEBUG SETTINGS
-- ============================================================================
Config.Debug = false -- Set to true to print debug output
Config.DetailedWitnessDebug = false -- Set to true for more detailed debug output about witnesses

Config.WitnessCooldown = 120000 -- 120 seconds in milliseconds, cooldown between witness report event triggers.
Config.MaxActiveWitnesses = 3   -- Maximum number of witnesses that can be actively tracked at any given time.
                                -- Use a fixed number: the config is read once, so math.random() here would only pick one value per restart.
Config.WitnessSearchRadius = 40.0 -- Radius to search for a witness around the player
Config.AllowPlayersAsVictims = true -- Whether players can be victims in crimes (but never witnesses)

Config.DisableTopNotifications = false -- Set to true to disable top notifications for all alerts (the dramatic WITNESS alerts)

Config.LawImmunityEnabled = true -- Enable/disable law immunity for witnesses. If true, witnesses will not report crimes committed by players with the law job.
Config.DutyCheckEnabled = true -- Enable/disable checking if LEO recipients of alerts are on duty.

-- ============================================================================
-- DUTY CHECK CONFIGURATION
-- ============================================================================
-- Duty comes from poggy_core. On VORP that is vorp_police / vorp_medic: an
-- officer counts as on duty only after going on duty there. When neither runs,
-- nobody counts as on duty, so with DutyCheckEnabled = true law jobs then get
-- no alerts (unless an alert sets overrideDutyCheck = true).
--
-- Optional override: a Lua string that returns true when the player is on duty,
-- for a duty script poggy_core does not read. Leave it "" to use poggy_core.
-- Available placeholders that will be replaced with actual values:
--   playerid   - The player's server ID (number).
--   job        - The player's job name (string, e.g., "police").
--   charId     - The player's character identifier (string or number).
--   identifier - The player's primary identifier, e.g., steam hex (string).
--   jobGrade   - The player's job grade (string or number).
-- Example: Config.DutyExportCall = "return exports['my_duty_script']:IsOnDuty(playerid)"
Config.DutyExportCall = ""
--
-- IMPORTANT: Ensure the call returns true if on duty, false otherwise.
-- Witness Escape Configuration
Config.WitnessEscape = {
    RequiredDistance = 20.0,                -- How far (in meters) a witness must be from player (relative to initial distance) to start reporting
    CheckInterval = 200,                    -- How often (in ms) to check witness position and update their statuses. 
    MaxEscapeTime = 120000,                 -- Maximum time (in ms) allowed for a witness to try to escape before giving up (120 seconds)
    ReportGracePeriod = 8000,               -- Time (in ms) a witness must remain "escaped" before the report is finalized (default: 8 seconds)
    ForcedReportMultiplier = 2,             -- If the witness travels X times the RequiredDistance, they will be forced to report immediately.
    EnableBlips = true,                     -- Enable/disable blips for witnesses
    BlipType = "blip_ambient_eyewitness",   -- Use a standard enemy blip type
    BlipScale = 1.5,                        -- Scale of the blip (increased for better visibility)
    ReactionDelay = 1200,                   -- Time (in ms) an NPC pauses and turns toward the threat before breaking into a flee. Set to 0 to disable.
}

Config.Checks = {
    Shooting = { -- Includes just aiming the weapon
        Enabled = true,
        Command = "witness_shooting_alert", -- Command executed if player shoots
        Probability = 100 -- 100% chance to trigger witness event (default)
    },
    Threatening = { -- Checks for aiming a threatening weapon without shooting
        Enabled = false,
        Command = "witness_threatening_alert", -- Command executed if player aims without shooting
        Probability = 50
    },
    Melee = { -- Punching or fighting with melee weapons 
        Enabled = false,
        Command = "witness_melee_alert",
        Probability = 30
    },
    Lassoing = { 
        Enabled = false,
        Command = "witness_lassoing_alert",
        Probability = 75
    },
    Trampling = {
        Enabled = false,
        MinSpeed = 1.0, -- Minimum speed of the mount to trigger trampling check
        Command = "witness_trampling_alert",
        Probability = 100
    },
    Hijacking = {
        Enabled = true,
        Command = "witness_hijacking_alert",
        Probability = 80
    },
    CarryingHostage = { -- Player is riding with a hogtied NPC attached to their horse
        Enabled = true,
        Command = "witness_carrying_hostage_alert",
        Probability = 90
    }
}

Config.EnforceTownLimits = true -- Set to true to only detect actions within defined town zones. Detection may cause performance issues if set to true, though none have been noticed in testing.
Config.EnforceBlacklistTowns = true -- Set to true to disable witness detection in blacklisted towns (overrides whitelist)

Config.Towns = {
    { name = "Saint Denis",     coords = vector3(2610.13, -1196.84, 53.33), detectionRadius = 450.0, enabled = true },
    { name = "Rhodes",          coords = vector3(1337.33, -1263.98, 77.64), detectionRadius = 240.0, enabled = true }, 
    { name = "Annesburg",       coords = vector3(2883.7,  1352.5,   61.91), detectionRadius = 170.0, enabled = true },
    { name = "Valentine",       coords = vector3(-302.08, 750.93,  118.00), detectionRadius = 290.0, enabled = true }, 
    { name = "Emerald Ranch",   coords = vector3(1419.21, 329.99,   88.49), detectionRadius = 190.0, enabled = true },
    { name = "Strawberry",      coords = vector3(-1793.61,-414.31, 152.76), detectionRadius = 180.0, enabled = true },
    { name = "Blackwater",      coords = vector3(-859.48, -1330.21, 43.41), detectionRadius = 220.0, enabled = true },
    { name = "Tumbleweed",      coords = vector3(-5500.97,-2947.17, -1.54), detectionRadius = 180.0, enabled = true },
    { name = "Armadillo",       coords = vector3(-3663.32,-2595.45,-10.08), detectionRadius = 170.0, enabled = true },
    { name = "Colter",          coords = vector3(-1329.25,2443.97, 300.57), detectionRadius = 150.0, enabled = true },
    { name = "Thieves Landing", coords = vector3(-1420.86, -2306.92, 43.06), detectionRadius = 80.0, enabled = true }
    -- Add more towns here if needed, following the same format
}

-- Towns where witness system will be disabled when Config.EnforceBlacklistTowns is true
Config.BlacklistedTowns = {
    { name = "Sisika Penitentary",        coords = vector3(3232.98, -626.49, 43.45), detectionRadius = 350.0, enabled = true }, -- Sisika Penitentiary
    { name = "Van Horn",  coords = vector3(2979.23, 499.11, 45.83), detectionRadius = 140.0, enabled = true },
    { name = "Wapiti",     coords = vector3(453.97, 2229.16, 247.22), detectionRadius = 100.0, enabled = true } -- Example disabled area
    -- Add more blacklisted areas as needed
}

Config.VictimDetectionRadius = 3.0 -- Radius for detecting victims for trampling

Config.PoliceJobList = {
    "police",
    "sheriff",
    "marshal"
} -- Add any other law job names here (exact job names, case-sensitive)

Config.PoliceJobRanks = { -- List of police ranks that will receive alerts. Add your job ranks here.
    police = {0,1,2,3,4,5},
    sheriff = {0,1,2,3,4,5},
    marshal = {0,1,2,3,4,5}
}

Config.BlipRadius = 50.0 -- Radius in meters for alert blips

-- Persistent Blip Settings (for alerts with persistentBlip = true)
Config.PersistentBlipEnabled = true -- Master toggle for persistent blip feature
Config.PersistentBlipClearDistance = 25.0 -- Distance in meters at which persistent blips are cleared when player arrives
Config.PersistentBlipCheckInterval = 1000 -- How often (in ms) to check player distance from persistent blips

Config.Alerts = {
    -- Witness: SHOOTING
    {
        name                = T('ALERT_SHOOTING_NAME'),
        command             = Config.Checks.Shooting.Command,
        message             = T('ALERT_SHOOTING_MESSAGE'),
        alerterNotification = T('ALERT_SHOOTING_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_RED",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_eyewitness",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true, -- ADD THIS
        witnessType         = "Shooting" -- ADD THIS (must match client's actionType)
    },
    -- Witness: MELEE FIGHT
    {
        name                = T('ALERT_MELEE_NAME'),
        command             = Config.Checks.Melee.Command,
        message             = T('ALERT_MELEE_MESSAGE'),
        alerterNotification = T('ALERT_MELEE_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_YELLOW",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_eyewitness",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true, -- ADD THIS
        witnessType         = "Melee" -- ADD THIS
    },
    -- Witness: HOGTIED / LASSOING
    {
        name                = T('ALERT_LASSOING_NAME'),
        command             = Config.Checks.Lassoing.Command,
        message             = T('ALERT_LASSOING_MESSAGE'),
        alerterNotification = T('ALERT_LASSOING_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_ORANGE",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_eyewitness",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true, -- ADD THIS
        witnessType         = "Lassoing" -- ADD THIS
    },
    -- Witness: TRAMPLING
    {
        name                = T('ALERT_TRAMPLING_NAME'),
        command             = Config.Checks.Trampling.Command,
        message             = T('ALERT_TRAMPLING_MESSAGE'),
        alerterNotification = T('ALERT_TRAMPLING_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_BLUE",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_eyewitness",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true, -- ADD THIS
        witnessType         = "Trampling" -- ADD THIS
    },
    -- Witness: THREATENING WITH A WEAPON
    {
        name                = T('ALERT_THREATENING_NAME'),
        command             = Config.Checks.Threatening.Command,
        message             = T('ALERT_THREATENING_MESSAGE'),
        alerterNotification = T('ALERT_THREATENING_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_ORANGE",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_eyewitness",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true, -- ADD THIS
        witnessType         = "Threatening" -- ADD THIS
    },
    -- Witness: HIJACKING (horse / wagon)
    {
        name                = T('ALERT_HIJACKING_NAME'),
        command             = Config.Checks.Hijacking.Command,
        message             = T('ALERT_HIJACKING_MESSAGE'),
        alerterNotification = T('ALERT_HIJACKING_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_ORANGE",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_horse",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true, -- ADD THIS
        witnessType         = "Hijacking" -- ADD THIS
    },
    -- Witness: CARRYING HOSTAGE (riding with a hogtied NPC on horseback)
    {
        name                = T('ALERT_CARRYING_HOSTAGE_NAME'),
        command             = Config.Checks.CarryingHostage.Command,
        message             = T('ALERT_CARRYING_HOSTAGE_MESSAGE'),
        alerterNotification = T('ALERT_CARRYING_HOSTAGE_ALERTER_NOTIFICATION'),
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "tick",
        color               = "COLOR_RED",
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_horse",
        radius              = 5.0,
        blipTime            = 120000,
        blipDelay           = 3000,
        waypoint            = true,
        triggerWitness      = true,
        witnessType         = "CarryingHostage"
    },
    -- Other job alerts. You can use these to specify different alerts for other scripts.
    -- Break-in alert triggered server-side by gs_doorlocks (obfuscated command so players cannot call it manually)
    {
        name = 'BREAK-IN ATTEMPT',
        command = 'breakin9x7k2m4p8q', -- Obfuscated: triggered by gs_doorlocks server only, not intended for player use
        message = "Someone is breaking and entering! Investigate the location.",
        alerterNotification = "Someone noticed your break-in! The law has been alerted.",
        messageTime = 5000,
        jobs = Config.PoliceJobList,
        jobgrade = Config.PoliceJobRanks,
        icon = "lock",
        color = "COLOR_RED",
        texturedict = "generic_textures",
        blipName = "blip_proc_home_locked",
        radius = 5.0,
        blipTime = 120000,
        blipDelay = 0,
        waypoint = true,
        triggerWitness = false,
    },
    {
        name = 'ROBBERY IN PROGRESS', --The name of the alert
        command = 'robbery198sd99720', -- the command, this is what players will use with /
        message = "A bank robbery is in progress! Respond to the location.", -- Message to show to the police
        alerterNotification = "You've triggered an alert! Authorities are aware of the robbery.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "stamp_gold", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_cash_bag", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'TEST ALERT IGNORE', --The name of the alert
        command = 'testalert4068', -- the command, this is what players will use with /
        message = "This is only a test. You may ignore this alert.", -- Message to show to the police
        alerterNotification = "You've triggered an alert! Authorities are aware of the test.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "stamp_gold", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_cash_bag", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'CANNIBAL SALES', --The name of the alert
        command = 'cannibal3098084', -- the command, this is what players will use with /
        message = "Someone was caught selling human organs to an underground market!", -- Message to show to the police
        alerterNotification = "Someone saw your shady sales!", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "stamp_gold", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_cash_bag", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'BOUNTY SPOTTED', --The name of the alert
        command = 'bounty198sd99720', -- the command, this is what players will use with /
        message = "A bounty has been spotted! Respond to the location.", -- Message to show to the police
        alerterNotification = "The locals recognize you! Authorities are aware of your bounty.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "temp_pedshot", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_cash_bag", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'NEED POLICE', --The name of the alert
        command = 'callpolice', -- the command, this is what players will use with /
        message = "A citizen is requesting law assistance!", -- Message to show to the police
        alerterNotification = "The locals have sent for the law to come!", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_BLUE', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_ambient_sheriff", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true, -- Whether to set a waypoint to the alert location
        cooldown = 120 -- Cooldown time in seconds before the alert can be triggered again
    },
    {
        name = 'NEED DOCTOR', --The name of the alert
        command = 'calldoctor', -- the command, this is what players will use with /
        message = "An injured citizen needs your help!", -- Message to show to the police
        alerterNotification = "The locals have sent for the doctor!", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = {"doctor"}, -- Job the alert is for
        jobgrade = {doctor = {0,1,2,3,4,5}},
        icon = "star", -- The icon the alert will use
        color = 'COLOR_YELLOW', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_mp_travelling_saleswoman", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds) - Used as fallback if persistentBlip fails
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true, -- Whether to set a waypoint to the alert location
        cooldown = 120, -- Cooldown time in seconds before the alert can be triggered again
        persistentBlip = true -- If true, blip stays until player reaches the location (within Config.PersistentBlipClearDistance meters)
    },
    {
        name = 'PRESIDENT HURT', --The name of the alert
        command = 'pd', -- the command, this is what players will use with /
        message = "The President has been hurt, doctor!", -- Message to show to the police
        alerterNotification = "The locals have sent for the President's Doctor!", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = {"doctor"}, -- Job the alert is for
        jobgrade = {doctor = {3,4,5}},
        icon = "star", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_mp_travelling_saleswoman", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds) - Used as fallback if persistentBlip fails
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true, -- Whether to set a waypoint to the alert location
        cooldown = 120, -- Cooldown time in seconds before the alert can be triggered again
        persistentBlip = true -- If true, blip stays until player reaches the location (within Config.PersistentBlipClearDistance meters)
    },
    {
        name = 'BACKUP NEEDED', --The name of the alert
        command = 'buaasuydikhjg', -- the command, this is what players will use with /
        message = "Officer requesting backup! All available units respond.", -- Message to show to the police
        alerterNotification = "Backup request sent! Nearby units alerted.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_ambient_sheriff", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'PRESIDENT IN DANGER', --The name of the alert
        command = 'pc2114', -- the command, this is what players will use with /
        message = "The President is in danger! All available units respond.", -- Message to show to the police
        alerterNotification = "You have alerted nearby units!", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_ambient_sheriff", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'DRUG SALE', --The name of the alert
        command = 'drugsaleaasuydikhjg', -- the command, this is what players will use with /
        message = "A suspected drug sale has been reported. Investigate the area.", -- Message to show to the police
        alerterNotification = "A local saw your drug sale! The Sheriffs have been informed.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_GREEN', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_mg_five_finger_fillet", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 0, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'EXPLOSIVES', --The name of the alert
        command = 'explosivesudiubieubsj', -- the command, this is what players will use with /
        message = "Report of someone transporting explosives. Approach with caution.", -- Message to show to the police
        alerterNotification = "Someone tipped off the law about the explosives! Sheriffs notified.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_teamsters", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 3000, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'UNDERTAKER', --The name of the alert
        command = 'undertaker', -- the command, this is what players will use with /
        message = "Some dead locals need to be picked up by the undertaker.", -- Message to show to the police
        alerterNotification = "Locals have gone to get the undertakers.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = {"undertaker"}, -- Job the alert is for
        jobgrade = {undertaker = {0,1,2,3,4,5}},
        icon = "temp_pedshot", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_teamsters", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 3000, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'UNDERTAKER', --The name of the alert
        command = 'alertundertaker', -- the command, this is what players will use with /
        message = "Some dead locals need to be picked up by the undertaker.", -- Message to show to the police
        alerterNotification = "Locals have gone to get the undertakers.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = {"undertaker"}, -- Job the alert is for
        jobgrade = {undertaker = {0,1,2,3,4,5}},
        icon = "temp_pedshot", -- The icon the alert will use
        color = 'COLOR_RED', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_teamsters", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 3000, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'KIDNAPPING', --The name of the alert
        command = 'kidnappingsudiubieubsj', -- the command, this is what players will use with /
        message = "Urgent: A kidnapping has been reported! Immediate response required.", -- Message to show to the police
        alerterNotification = "Someone tipped off the law about the kidnapping! Sheriffs notified.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_BLUE', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_honor_bad", -- Alternative blip name for better reliability
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 3000, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'PRISONER ESCAPE', --The name of the alert
        command = 'prisonersudiubieubsj', -- the command, this is what players will use with /
        message = "Alert: Prisoner escape reported! Secure the area and search for the escapee.", -- Message to show to the police
        alerterNotification = "The prison guards are aware of the escapee! Sheriffs notified.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_YELLOWSTRONG', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_mp_game_vip", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 3000, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'TRAIN ROBBERY', --The name of the alert
        command = 'trainsudiubieubsj', -- the command, this is what players will use with /
        message = "A train robbery is reportedly in progress! All units converge.", -- Message to show to the police
        alerterNotification = "The conductor reported the train robbery! Sheriffs notified.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = Config.PoliceJobList, -- Job the alert is for
        jobgrade = Config.PoliceJobRanks,
        icon = "star", -- The icon the alert will use
        color = 'COLOR_PURPLE', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_ambient_train", -- The code of the blip
        radius = 5.0,
        blipTime = 120000,
        blipDelay = 3000,
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    {
        name = 'PASSENGER PICKUP', --The name of the alert
        command = 'calltrain', -- the command, this is what players will use with /
        message = "Someone is requesting a ride, Mr. Conductor.", -- Message to show to the police
        alerterNotification = "The conductor has been notified of your request.", -- Message to show players they have alerted
        messageTime = 5000, -- Time the message will stay on screen (miliseconds)
        jobs = {"conductor"}, -- Job the alert is for
        jobgrade = {conductor = {0,1,2,3,4,5}},
        icon = "star", -- The icon the alert will use
        color = 'COLOR_PURPLE', -- The color of the icon / https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/colours
        texturedict = "generic_textures", --https://github.com/femga/rdr3_discoveries/tree/master/useful_info_from_rpfs/textures/menu_textures
        blipName = "blip_ambient_train", -- The code of the blip
        radius = 5.0, -- The size of the radius blip
        blipTime = 120000, -- How long the blip will stay for the job (miliseconds)
        blipDelay = 3000, -- Delay time before the job is notified (miliseconds)
        waypoint = true -- Whether to set a waypoint to the alert location
    },
    -- Grave Robbing: fired server-side by another resource via exports.poggy_witnesses:TriggerAlertForPlayer
    {
        name                = 'GRAVE ROBBING',
        command             = 'graverobbing8x2kp19',   -- obfuscated – the calling resource must use this exact command
        message             = "Grave robbing reported at the local cemetery!",
        alerterNotification = "Your grave robbing has been noticed by the locals!",
        messageTime         = 5000,
        jobs                = Config.PoliceJobList,
        jobgrade            = Config.PoliceJobRanks,
        icon                = "star",
        color               = 'COLOR_PURPLE',
        texturedict         = "generic_textures",
        blipName            = "blip_ambient_graveyard",
        radius              = 5.0,
        blipTime            = 90000,
        blipDelay           = 0,
        waypoint            = true,
        triggerWitness      = false,
    }
}

Config.NonThreateningAimItems = { -- Aiming with these items will not trigger the threatening weapon check
    "WEAPON_LASSO",
    "WEAPON_KIT_CAMERA",
    "WEAPON_KIT_BINOCULARS",
    "WEAPON_MELEE_LANTERN",
    "WEAPON_MELEE_DAVY_LANTERN",
    "WEAPON_MELEE_LANTERN_ELECTRIC",
    "WEAPON_MELEE_TORCH", -- Note: GetHashKey("WEAPON_MELEE_TORCH") is 0xD696159F, not 0x67DC3FDE. Using the common name.
    "WEAPON_FISHINGROD",
    "WEAPON_LASSO_REINFORCED",
    "WEAPON_KIT_BINOCULARS_IMPROVED",
    "WEAPON_KIT_CAMERA_ADVANCED",
    "WEAPON_MELEE_LANTERN_HALLOWEEN",
    "WEAPON_KIT_METAL_DETECTOR"
    -- Add other non-threatening items here by their string name (e.g., "WEAPON_UNARMED")
}

Config.BlacklistNPCs = { -- NPCs that will not trigger the witness system
-- NPCs from Bodyguard Script
    "cs_captainmonroe", 
    "a_m_m_armtownfolk_01",
    "MP_U_M_M_LBT_COACHDRIVER_01",
    "cs_pinkertongoon",
    "cs_sheriffowens",
    "MP_U_M_M_ARMSHERIFF_01",
    "cs_valsheriff",
    "cs_bwsheriff",
    "cs_rhodes_sheriff",
    "cs_strsheriff_01",
    "a_m_m_jamesonguard_01",
    "MP_S_M_M_CornwallGuard_01",
    "s_m_m_orpguard_01",
    "s_m_m_skpguard_01",
    "cs_mp_bountyhunter",
    "g_m_m_bountyhunters_01",
    "cs_dutch",
    "cs_micahbell",
    "cs_billwilliamson",
    "cs_charlessmith",
    "cs_johnmarston",
    "cs_hoseamatthews",
    "cs_kieran",
    "cs_javierescuella",
    "cs_josiahtrelawny",
    "cs_reverendswanson",
    "cs_colmodriscoll",
    "cs_edgarross",
-- END NPCs from Bodyguard Script
    "cs_mp_travellingsaleswoman",
    "mp_fm_knownbounty_informants_females_01",
    "mp_fm_knownbounty_informants_males_01",
    "mp_u_f_m_legendarybounty_03",
    "mp_u_f_m_bountytarget_001",
    "mp_u_f_m_bountytarget_002",
    "mp_u_f_m_bountytarget_003",
    "mp_u_f_m_bountytarget_004",
    "mp_u_f_m_bountytarget_005",
    "mp_u_f_m_bountytarget_006",
    "mp_u_f_m_bountytarget_007",
    "mp_u_f_m_bountytarget_008",
    "mp_u_f_m_bountytarget_009",
    "mp_u_f_m_bountytarget_010",
    "mp_u_f_m_bountytarget_011",
    "mp_u_m_m_bountyinjuredman_01",
    "mp_fm_bounty_horde_law_01",
    "mp_u_m_m_legendarybounty_08",
    "mp_u_m_m_legendarybounty_09",
    "mp_u_f_m_legendarybounty_001",
    "mp_u_f_m_legendarybounty_002",
    "g_m_m_bountyhunters_01",
    "g_m_m_unibanditos_01",
    "re_street_fight_males_01",
    "S_M_M_AmbientSDPolice_01",
    "CS_MARSHALL_THURWELL",
    "CS_MP_MARSHALL_DAVIES",
    "S_M_M_AmbientBlWPolice_01",
    "U_M_O_BlWPoliceChief_01",
    "U_M_M_ValSheriff_01",
    "s_m_m_bankclerk_01",
    "mp_u_m_m_animalpoacher_01",
    "mp_u_m_m_animalpoacher_02",
    "mp_u_m_m_animalpoacher_03",
    "mp_u_m_m_animalpoacher_04",
    "mp_u_m_m_animalpoacher_05",
    "MP_G_M_M_ANIMALPOACHERS_01",

    -- newly added law-officer models:
    "mp_fm_track_sd_lawman_01",
    "s_m_m_ambientsdpolice_01",
    "u_m_m_sdpolicechief_01",
    "cs_rhodeputy_01",
    "cs_rhodeputy_02",
    "u_m_m_rhdsheriff_01",
    "mp_u_m_m_lawcamp_lawman_02",
    "u_m_m_rhdbackupdeputy_02",
    "mp_u_m_m_lawcamp_lawman_01",
    "a_m_m_valdeputyresident_01",
    "cs_valsheriff",
    "cs_valdeputy_01",
    "s_m_m_valdeputy_01",
    "u_m_m_valsheriff_01",
    "cs_strsheriff_01",
    "a_m_m_strdeputyresident_01",
    "cs_strdeputy_01",
    "cs_strdeputy_02",
    "cs_mp_sherifffreeman",
    "MP_S_M_M_PinLaw_01",
    "cs_pinkertongoon",
    "s_m_m_pinlaw_01",
    "a_m_m_armdeputyresident_01",
    "MP_U_M_M_ARMSHERIFF_01",
    "s_m_m_tumdeputies_01",
    "MP_RESCUE_COLTER_MALES_01",
    "MP_G_M_M_BOUNTYHUNTERS_01",
    "u_m_m_unibountyhunter_01",
    "u_m_m_unibountyhunter_02",

    -- Job NPCs from xakra_jobs (location NPCs):
    "A_M_M_DELIVERYTRAVELERS_COOL_01",
    "CS_VALSHERIFF",

    -- Delivery job NPCs:
    "A_F_M_BiVFancyTravellers_01",
    "CS_AberdeenPigFarmer",
    "MP_ASN_BRAITHWAITEMANOR_MALES_03",
    "U_M_M_HtlForeman_01",
    "U_M_O_BHT_DOCWORMWOOD",

    -- Liqueur delivery NPCs:
    "A_M_M_RkrFancyTravellers_01",
    "CS_SDSALOONDRUNK_01",
    "A_M_M_RANCHER_01",
    "mp_chu_kid_armadillo_males_01",
    "A_M_M_BLWObeseMen_01",
    "A_M_M_BtcHillbilly_01",

    -- Explosives delivery NPCs:
    "mp_chu_kid_tumbleweed_males_01",
    "A_M_M_NbxDockWorkers_01",
    "S_M_Y_Army_01",
    "mp_u_m_m_lbt_accomplice_01",
    "A_M_M_RHDDEPUTYRESIDENT_01",

    -- Contract job clients and delivery NPCs:
    "A_M_M_FARMTRAVELERS_WARM_01",
    "A_M_M_BTCObeseMen_01",
    "A_F_M_BTCObeseWomen_01",
    "RE_INJUREDRIDER_MALES_01",
    "CS_JAMIE",
    "G_M_M_UNILANGSTONBOYS_01",
    "MES_FINALE2_MALES_01",
    "CS_SUNWORSHIPPER",
    "S_F_M_BwmWorker_01",
    "G_F_M_UNIDUSTER_01",

    -- Kidnapped victims:
    "MSP_BOUNTYHUNTER1_FEMALES_01",
    "A_F_M_VhtProstitute_01",
    "mp_re_kidnapped_females_01",
    "RE_KIDNAPPEDVICTIM_FEMALES_01",
    "A_F_O_WAPTOWNFOLK_01",
    "A_F_M_ValProstitute_01",

    -- Prisoners:
    "A_M_M_SkpPrisoner_01",
    "A_M_M_SkpPrisonLine_01",
    "CS_Mickey",
    "CS_Meredith",
    "CS_Magnifico",
    "CS_GermanSon",

    -- Bandits and enemies (these shouldn't be witnesses anyway):
    "A_M_M_HUNTERTRAVELERS_WARM_01",
    "mp_g_m_m_uniinbred_01",
    "G_M_M_UniBanditos_01",
    "mp_g_m_m_unibanditos_01",
    "A_M_M_WapWarriors_01",
    "G_M_M_UniInbred_01",
    "G_M_M_UniDuster_04",
    "G_M_M_UniInbred_01",
    "A_M_M_huntertravelers_cool_01",
    "A_M_M_GriSurvivalist_01",
    "A_M_M_MOONSHINERS_01",
    "A_M_M_BynFancyDRIVERS_01",
    "A_M_M_BynRoughTravellers_01",
    "G_M_M_UniMountainMen_01",
    "mp_g_m_m_unimountainmen_01",
    "A_M_M_RhdTownfolk_02",
    "G_M_M_UNIDUSTER_05",
    "G_M_O_UniExConfeds_01",
    "G_M_M_UniCriminals_02",
    "G_M_M_UniCriminals_01",
    "G_M_Y_UniExConfeds_01",
    "G_M_Y_UNIEXCONFEDS_02",

    -- Guards and law enforcement from jobs:
    "mp_s_m_m_cornwallguard_01",
    "A_M_M_JamesonGuard_01",
    "A_M_M_UniCoachGuards_01",
    "S_M_M_ValBankGuards_01",
    "S_M_M_UniTrainGuards_01",
    "S_M_M_Army_01",
    "S_M_M_MARSHALLSRURAL_01",
    "S_M_M_FussarHenchman_01"
}
