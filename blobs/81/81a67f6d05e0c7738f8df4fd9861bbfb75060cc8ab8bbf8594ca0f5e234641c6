--[[
    Balloon Taxi Service Configuration
    All values can be adjusted to customize the balloon ride service
]]

Config = {}

-- ===============================================
-- DEBUG SETTINGS
-- ===============================================
Config.Debug = {
    Enabled = false,            -- Enable/disable debug logging
    Level = 4                  -- 0=OFF, 1=ERROR, 2=WARNING, 3=INFO, 4=DEBUG
}

-- ===============================================
-- FLIGHT (captain controls)
-- ===============================================
-- The game's own burner input (INPUT_VEH_FLY_THROTTLE_UP, Shift) only counts
-- while the game is in its in-vehicle input context, and for the balloon
-- captain it did not register on current game builds: holding Shift did
-- nothing. Ascent is therefore driven by this script from the same key.
-- Speeds are metres per second.
Config.Flight = {
    ScriptedAscent = true,   -- false = rely on the game's own burner only (old behaviour)
    AscentAccel = 0.04,      -- climb speed gained each frame while Shift is held
    AscentMax = 2.5,         -- top climb speed
    DescendKey = true,       -- Ctrl pushes the balloon down (false = it only sinks on its own)
    DescentAccel = 0.04,     -- sink speed gained each frame while Ctrl is held
    DescentMax = 2.0,        -- top sink speed from the key
}

-- ===============================================
-- BALLOON TAXI SERVICE SETTINGS
-- ===============================================
Config.BalloonTaxi = {
    Enabled = true,             -- Enable/disable the balloon taxi service
    
    -- Pricing
    Price = 3,                  -- Cost in dollars for a balloon ride
    
    -- Flight Settings
    CruisingAltitude = 80.0,    -- Target altitude in meters above ground
    AltitudeTolerance = 10.0,   -- Tolerance range (-10m from cruising altitude)
    HeightCheckInterval = 500, -- How often to check/adjust altitude (milliseconds)
    
    -- Movement Settings (velocity in meters per second)
    TravelSpeed = 22.0,          -- Normal horizontal travel speed (m/s)
    TravelSpeedFast = 30.0,      -- Fast travel speed when "Step on it" is active (m/s)
    AscentSpeed = 5.0,           -- Ascending speed (m/s)
    DescentSpeed = 20.0,          -- Descending speed (m/s) - gentler for smooth landing
    PositionAdjustSpeed = 20.0,   -- Speed for gentle position adjustments at destination
    
    -- Arrival Settings
    FinalApproachDistance = 100.0, -- Distance from waypoint to start calculating final ground height
    ArrivalDistance = 5.0,         -- Distance to waypoint XY to consider "arrived"
    PositionLockSmoothness = 0.05, -- How gently to lock position (lower = smoother)
    
    -- Landing & Exit Settings
    LandingHeightOffset = 2.0,     -- Height above ground to land (for balloon basket clearance)
    ExitWaitTime = 30000,          -- Time to wait for player to exit after landing (ms)
    DespawnRiseTime = 30000,        -- Time to rise before despawning (ms)
    DespawnDelay = 10000,          -- Time after rising before despawn (ms)
    
    -- Balloon Model
    BalloonModel = "hotairballoon01",
    
    -- Pilot Configuration (fallback if location doesn't specify)
    PilotModel = "cs_balloonoperator", -- NPC model for the balloon pilot (uses location npcModel if available)
    PilotScenario = nil,               -- Optional scenario to play (nil = use balloon captain animations)
}

-- ===============================================
-- CONTROL KEY BINDINGS
-- ===============================================
-- Control hashes for prompt actions. Change these to rebind keys.
-- Find control hashes at: https://github.com/femga/rdr3_discoveries/blob/master/Controls/controls.lua
Config.Controls = {
    -- NPC Interaction
    Speak = 0x760A9C6F,           -- G key (INPUT_FRONTEND_SOCIAL_CLUB_SECONDARY)
    
    -- Pre-Flight
    TakeOff = 0x4CC0E2FE,         -- B key (INPUT_SPECIAL_ABILITY_SECONDARY)
    
    -- In-Flight Controls
    Stop = 0x06052D11,            -- Q key (INPUT_COVER)
    Reroute = 0xE30CD707,         -- R key (INPUT_RELOAD)
    SpeedBoost = 0x7065027D,      -- A key (INPUT_MOVE_LEFT_ONLY)
    ChangeSeat = 0x26E9DC00,      -- C key (INPUT_LOOK_BEHIND)
    Exit = 0x06052D11,            -- F key (INPUT_ENTER)
}

-- ===============================================
-- BALLOON RENTAL SETTINGS
-- ===============================================
Config.BalloonRental = {
    Enabled = true,                    -- Master toggle for balloon rentals
    Price = 50,                        -- Price to rent a balloon
    RentalDurationMinutes = 60,       -- Rental duration in minutes (1 hour = 60)
    WarningTimeMinutes = 5,            -- Time before expiry to warn player (5 minutes)
    BalloonModel = "hotairballoon01",  -- Balloon model for rentals
}

-- ===============================================
-- BALLOON TAXI NPC LOCATIONS
-- ===============================================
Config.BalloonTaxi.Locations = {
    {
        id = "strawberry_balloon",
        name = "Strawberry Balloon Tours",
        coords = vector3(-1812.9, -677.43, 149.6),
        heading = 180.0,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(-1810.09, -673.38, 150.55),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true -- Allow balloon rentals at this location
    },
    {
        id = "rhodes_balloon",
        name = "Rhodes Balloon Tours",
        coords = vector3(1425.01, -1273.0, 77.27),
        heading = 176.31,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(1423.63, -1283.8, 76.97),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    },
    {
        id = "valentine_balloon",
        name = "Valentine Balloon Tours",
        coords = vector3(-145.7, 611.94, 114.39),
        heading = 121.62,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(-144.74, 618.11, 114.02),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    },
    {
        id = "annesburg_balloon",
        name = "Annesburg Balloon Tours",
        coords = vector3(2865.34, 1427.64, 67.41),
        heading = 125.07,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(2866.08, 1423.03, 67.18),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    },
    {
        id = "saintdenis_balloon",
        name = "Saint Denis Balloon Tours",
        coords = vector3(2717.29, -1511.52, 43.19),
        heading = 4.08,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(2718.64, -1518.06, 43.17),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    },
    {
        id = "blackwater_balloon",
        name = "Blackwater Balloon Tours",
        coords = vector3(-697.71, -1285.4, 42.22),
        heading = 49.31,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(-710.54, -1284.07, 42.36),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    },
    {
        id = "armadillo_balloon",
        name = "Armadillo Balloon Tours",
        coords = vector3(-3738.5, -2589.45, -14.3),
        heading = 182.43,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(-3740.74, -2594.05, -14.27),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    },
    {
        id = "tumbleweed_balloon",
        name = "Tumbleweed Balloon Tours",
        coords = vector3(-5468.53, -2962.53, -1.4),
        heading = 59.85,
        npcModel = "cs_balloonoperator",
        balloonSpawnCoords = vector3(-5473.68, -2966.36, -1.87),
        balloonHeading = 0.0,
        blipEnabled = true,
        blipSprite = 531267562,
        blipName = "Balloon Tours",
        rentalEnabled = true
    }
}

-- ===============================================
-- ADMIN COMMAND SETTINGS
-- ===============================================
Config.Commands = {
    SpawnBalloon = {
        Name = "spawnballoon",
        Enabled = true,
        AdminOnly = true,       -- Set to false to let everyone use it
        AcePermission = nil     -- Set to "command.spawnballoon" to use ACE permissions
    }
}

return Config
