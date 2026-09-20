Config = {}

--================================--
--       SHARED CONFIGURATION     --
--================================--

-- Debug: Set to true to enable debug logging
Config.DEBUG = false

--================================--
--       SERVER CONFIGURATION     --
--================================--

-- How the blips work.
--   "hybrid"  A player in range is followed through their ped: the blip is
--             attached to it and the game moves it every frame, at no network
--             cost. A player out of range has no ped on your machine (the
--             server only streams what is near you), so they get a coordinate
--             blip from the server instead. The default.
--   "coords"  Every blip is a coordinate blip moved by the server. How this
--             script worked before 1.3.0.
Config.METHOD = "hybrid"

-- How often positions are sent to admins (milliseconds). With "hybrid" they
-- are only used for players who are out of range, and the blip glides between
-- updates, so 2000 is plenty. With "coords" it is every blip: 500 is smoother.
Config.UPDATE_INTERVAL_MS = 2000

-- Glide an out-of-range blip from one position to the next instead of jumping.
Config.GLIDE = true

-- Staff command for checking the blips without a restart:
--   /abtest status          what each blip really is (attached to a ped, or fed
--                           by coordinates) and how far away. Details print to F8.
--   /abtest hybrid|coords   switch method, for everyone, until the next restart
--   /abtest rate <ms>       /abtest glide on|off
-- Set to false to remove the command.
Config.TEST_COMMAND = "abtest"

-- Pending player retry interval (milliseconds)
Config.PENDING_RETRY_INTERVAL = 5000

-- Admin groups: Which groups can see blips (used when ALLOWED_NAMES is empty)
Config.ADMIN_GROUPS = {
    "admin",
    "superadmin"
}

--================================--
--       CLIENT CONFIGURATION     --
--================================--

-- Whitelist: Only these Steam Names (Display Names) can see blips.
-- They must also be in one of the ADMIN_GROUPS.
-- If empty, all players in ADMIN_GROUPS will see blips
Config.ALLOWED_NAMES = {
}

-- Blip appearance
Config.BLIP_STYLE = "BLIP_STYLE_ENEMY"              -- Style hash for creating the blip
Config.BLIP_SPRITE = "blip_ambient_companion"       -- Sprite/icon for the blip
Config.BLIP_MODIFIER = "BLIP_MODIFIER_DEBUG_GREEN"  -- Color modifier (green)

-- Blip name format: Use {id} and {name} as placeholders
Config.BLIP_NAME_FORMAT = "{id} | {name}"

-- Hide own blip: Set to true to hide your own blip from yourself
Config.HIDE_OWN_BLIP = true

-- Initial wait time before registering (milliseconds)
Config.INIT_WAIT_TIME = 5000

--================================--
--      END CONFIGURATION         --
--================================--
