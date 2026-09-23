Config = {}

-- The language poggy_emotes speaks. Must be one of the languages in
-- translations.lua. Add your own there and put its key here.
Config.Language = "en"

-- ============================================================================
--  Commands
-- ============================================================================

-- Opens the emote menu. Players type a slash and this word.
Config.MenuCommand = "emotes"

-- Plays one emote by name, e.g. /e wave. On its own it opens the menu.
Config.PlayCommand = "e"

-- Stops whatever emote is playing and puts away any prop.
Config.CancelCommand = "ec"

-- Adds every emote name to the chat autocomplete, so typing /e shows the list.
-- Turn this off if your chat resource gets slow with a few hundred suggestions.
Config.ChatSuggestions = true

-- ============================================================================
--  Keys
-- ============================================================================
-- RedM has no in-game rebinding menu, so the keys are whatever you set here
-- and they are the same for every player.
--
-- Pick keys nothing else on your server uses. A clash means both scripts
-- react to the same press.

Config.Keys = {
    -- The keyboard key that opens the emote menu, and closes it again. It is
    -- read straight from the keyboard, so it works for keys the game has no
    -- control for. One of: END, HOME, INSERT, DELETE, PAGEUP, PAGEDOWN,
    -- F1 to F12, or a single letter or digit. "" turns the key off; the
    -- command always works.
    OpenMenuKey = "END",

    -- A game control that also opens the menu, for owners who would rather
    -- use one (for instance for controller players). 0 turns it off. Names
    -- between backticks come from
    -- https://github.com/femga/rdr3_discoveries/tree/master/Controls
    OpenMenuControl = 0,

    -- Stops the emote you are playing and puts away any prop. Default:
    -- Backspace. This works any time, including while riding.
    Cancel = `INPUT_GAME_MENU_CANCEL`,
}

-- ============================================================================
--  How emotes behave
-- ============================================================================

Config.Behaviour = {
    -- Stop the emote as soon as the player moves. Most roleplay servers want
    -- this on: it stops players sliding around mid-animation. Turn it off if
    -- you want emotes to hold until cancelled.
    CancelOnMove = true,

    -- Stop the emote when the player draws or fires a weapon. Leave this on.
    CancelOnAim = true,

    -- Put weapons away before an emote starts. Off means an emote played with
    -- a rifle out can look wrong, but it keeps the weapon in hand.
    HolsterWeapons = true,

    -- Let emotes play while riding a horse or sitting in a wagon. Most
    -- animations are built for standing and look broken mounted, so this is
    -- off by default. Cancelling always works, mounted or not.
    AllowWhileMounted = false,

    -- Let emotes play while the player is swimming. Off is strongly advised:
    -- an animation in water usually sinks or freezes the character.
    AllowWhileSwimming = false,
}

-- ============================================================================
--  Ragdoll
-- ============================================================================
-- One press drops the character limp on the ground, the next lets them get up.

Config.Ragdoll = {
    -- Turn the ragdoll key and command on. Off removes both.
    Enabled = true,

    -- The key that drops the character, and lets them up again. Default: Z.
    -- It is a game control, so it does nothing while the chat box is open.
    -- 0 turns the key off and leaves the command.
    Key = `INPUT_GAME_MENU_TAB_LEFT_SECONDARY`,

    -- The chat command that does the same as the key. "" turns it off.
    Command = "ragdoll",

    -- The least time a player stays down, in seconds. A second press sooner
    -- than this is ignored. 0 lets them bounce straight back up.
    MinSeconds = 2,

    -- Seconds a player must wait after getting up before they can drop again.
    -- It stops the key being hammered to dodge a lasso or a fist fight.
    -- 0 means no wait.
    Cooldown = 3,

    -- The longest a player can stay down, in seconds, before the script stands
    -- them up. 0 means as long as they like.
    MaxSeconds = 0,

    -- Let players drop while riding a horse or sitting in a wagon, which
    -- throws them off. Off by default.
    AllowWhileMounted = false,
}

-- ============================================================================
--  The menu
-- ============================================================================

Config.Menu = {
    -- How many emotes to keep in the Recent category, per character.
    -- 0 turns Recent off. Above about 30 the category stops being useful.
    RecentCount = 12,

    -- Let players mark emotes as favourites. Favourites are saved per
    -- character in the database and survive a reconnect.
    Favourites = true,

    -- Keep the menu open after playing an emote, so a player can try a few in
    -- a row. This is only where the switch at the bottom of the menu starts:
    -- each player can flip it, and their choice is remembered.
    StayOpenAfterPlay = false,

    -- Hide emotes that are not for the player's character gender. Off shows
    -- every emote to everyone, which is what you want if your server lets
    -- players pick any animation regardless of their model.
    FilterByGender = true,

    -- Which categories appear, and in what order. Remove one to hide it.
    -- 'favourites' and 'recent' are built by the script; the rest come from
    -- the category field in shared/emotes.lua and Config.CustomEmotes.
    CategoryOrder = {
        "nearby",
        "favourites",
        "recent",
        "gestures",
        "conversation",
        "idle",
        "sitting",
        "working",
        "consume",
        "dance",
        "injury",
    },
}

-- ============================================================================
--  Nearby: what the player can do with what is around them
-- ============================================================================
-- The Nearby category lists actions for the things around the player: sit on
-- that chair, lean on that railing, stand at the bar, pump that water. Pick
-- one and the character walks over and does it. Nothing is teleported.
--
-- Most of it comes from the game itself. Chairs, benches, pianos, bars, pumps,
-- stoves, campfires and railings carry scenario points that the game's own
-- NPCs use; the script lists the ones nearby and lets the player use them the
-- way an NPC would. The keyword and wall lists below add actions for things
-- the game has no point for.

Config.Nearby = {
    -- Show the Nearby category at all. Off removes it from the menu.
    Enabled = true,

    -- How far around the player to look, in metres.
    Radius = 4.0,

    -- The most rows the category shows at once. Closest things first.
    MaxRows = 40,

    -- Seconds the character may go without getting any closer to the spot
    -- before the action is cancelled with a message. Counted from the last
    -- step of progress, not from the press, so a long way round a table is
    -- fine; standing against a wall for this long is not.
    WalkTimeout = 10,

    -- The most scenario points the game is asked to report in one scan.
    PointLimit = 48,

    -- Only scenario types whose name starts with one of these are offered.
    -- Everything else (animal, ambient-only, cutscene types) is hidden.
    Prefixes = { "PROP_HUMAN_", "WORLD_HUMAN_", "PROP_PLAYER_", "WORLD_PLAYER_", "PROP_CAMP_", "MP_LOBBY_" },

    -- Lua patterns. A scenario type matching any of them is never listed.
    Hidden = { "_SPAWN", "_DEAD", "_CORPSE", "PROP_HUMAN_HITCH", "ANIMAL" },

    -- List scenario types the game reports but the script has no name for
    -- (shown as a hex number). Useful when hunting for one to label.
    ShowUnknown = false,

    -- Also ask the game whether this character may use the point. Turn on if
    -- rows appear that the character then refuses; off if the list is empty.
    RequireUsable = false,

    -- Blocking from in game. A dev sees an x on every Nearby row; clicking it
    -- and confirming takes that action off the tab for everyone, at once, and
    -- writes it into the Blocked list below, so it is kept with the config,
    -- survives updates, and ports with the file. Some of the game's scenarios
    -- are badly broken in custom-built areas, and this is how they are found
    -- and removed in play. A blocked row stays visible to devs, crossed out,
    -- so it can be allowed again from the same place.
    --
    -- Who is a dev: anyone poggy_core says is an admin (when AdminsCanBlock is
    -- on), and anyone with the ACE below in server.cfg, e.g.
    --   add_ace group.dev poggy_emotes.nearby.block allow
    AdminsCanBlock = true,
    BlockAce = "poggy_emotes.nearby.block",

    -- A block with x, y, z is for that one spot only (within BlockRadius
    -- metres); one without is for that scenario type everywhere. The x in
    -- the menu offers both. Rows may also be added by hand.
    BlockRadius = 0.75,
    Blocked = {
    },

    -- Friendly names for scenario types. Anything not listed is named from the
    -- scenario itself ("PROP_HUMAN_SEAT_CHAIR_DRINKING" -> "Seat Chair Drinking").
    Labels = {
        { scenario = "PROP_HUMAN_SEAT_CHAIR",                label = "Sit down" },
        { scenario = "PROP_HUMAN_SEAT_CHAIR_DRINKING",       label = "Sit and drink" },
        { scenario = "PROP_HUMAN_SEAT_CHAIR_TABLE",          label = "Sit at the table" },
        { scenario = "PROP_HUMAN_SEAT_CHAIR_PORCH",          label = "Sit on the porch" },
        { scenario = "PROP_HUMAN_SEAT_CHAIR_READING",        label = "Sit and read" },
        { scenario = "PROP_HUMAN_SEAT_CHAIR_KNITTING",       label = "Sit and knit" },
        { scenario = "PROP_HUMAN_SEAT_BENCH",                label = "Sit on the bench" },
        { scenario = "PROP_HUMAN_SEAT_BENCH_DRINKING",       label = "Sit and drink" },
        { scenario = "PROP_HUMAN_SEAT_BENCH_SMOKING",        label = "Sit and smoke" },
        { scenario = "PROP_HUMAN_SEAT_BENCH_PORCH",          label = "Sit on the porch" },
        { scenario = "PROP_HUMAN_SEAT_BENCH_PORCH_DRINKING", label = "Sit and drink" },
        { scenario = "PROP_HUMAN_SEAT_BENCH_PORCH_SMOKING",  label = "Sit and smoke" },
        { scenario = "PROP_HUMAN_PIANO",                     label = "Play the piano" },
        { scenario = "PROP_HUMAN_ABIGAIL_PIANO",             label = "Play the piano" },
        { scenario = "PROP_HUMAN_SLEEP_BED_PILLOW",          label = "Sleep" },
        { scenario = "PROP_HUMAN_SLEEP_BED_PILLOW_HIGH",     label = "Sleep" },
        { scenario = "PROP_HUMAN_PUMP_WATER",                label = "Pump water" },
        { scenario = "PROP_HUMAN_WOOD_CHOP",                 label = "Chop wood" },
        { scenario = "PROP_HUMAN_GRINDSTONE",                label = "Use the grindstone" },
        { scenario = "WORLD_HUMAN_BARCUSTOMER",              label = "Stand at the bar" },
        { scenario = "WORLD_HUMAN_LEAN_RAILING",             label = "Lean on the railing" },
        { scenario = "WORLD_HUMAN_LEAN_BACK_RAILING",        label = "Lean back on the railing" },
        { scenario = "WORLD_HUMAN_LEAN_BARREL",              label = "Lean on the barrel" },
        { scenario = "WORLD_HUMAN_LEAN_BACK_WALL",           label = "Lean on the wall" },
        { scenario = "WORLD_HUMAN_LEAN_WALL_LEFT",           label = "Lean on the wall (left)" },
        { scenario = "WORLD_HUMAN_LEAN_WALL_RIGHT",          label = "Lean on the wall (right)" },
        { scenario = "WORLD_HUMAN_FIRE_STAND",               label = "Stand by the fire" },
        { scenario = "WORLD_HUMAN_FIRE_SIT",                 label = "Sit by the fire" },
        { scenario = "WORLD_HUMAN_CAULDRON_STIR",            label = "Stir the pot" },
        { scenario = "WORLD_HUMAN_STIR_SOUP",                label = "Stir the soup" },
        { scenario = "WORLD_HUMAN_CLEAN_TABLE",              label = "Wipe the table" },
        { scenario = "WORLD_HUMAN_SHOP_BROWSE_COUNTER",      label = "Browse" },
        { scenario = "WORLD_HUMAN_WASH_FACE_BUCKET_GROUND",  label = "Wash your face" },
        { scenario = "WORLD_HUMAN_WASHBOARD_BASIN",          label = "Wash clothes" },
        { scenario = "WORLD_HUMAN_KNOCK_DOOR",               label = "Knock" },
        { scenario = "WORLD_HUMAN_CLEAN_WINDOW",             label = "Clean the window" },
        { scenario = "WORLD_HUMAN_GRAVE_MOURNING",           label = "Mourn" },
        { scenario = "WORLD_HUMAN_GRAVE_MOURNING_KNEEL",     label = "Kneel and mourn" },
        { scenario = "WORLD_HUMAN_SAW_WOOD",                 label = "Saw wood" },
        { scenario = "WORLD_HUMAN_PLANE_WOOD",               label = "Plane wood" },
        { scenario = "WORLD_HUMAN_HAMMER_TABLE",             label = "Hammer" },
        { scenario = "WORLD_HUMAN_PITCH_HAY_SCOOP",          label = "Pitch hay" },
        { scenario = "WORLD_HUMAN_SIT_GROUND",               label = "Sit on the ground" },
        { scenario = "WORLD_PLAYER_SLEEP_GROUND",            label = "Sleep on the ground" },
        { scenario = "WORLD_PLAYER_SLEEP_BEDROLL",           label = "Sleep in the bedroll" },
    },

    -- Actions for props the game has no scenario point for, matched on the
    -- prop's model name. keywords = plain text that must appear in the name;
    -- distance = metres from the prop's edge to stand; face = "toward" the
    -- prop or "away" from it (back to it, for leaning and sitting on edges).
    Keywords = {
        { keywords = { "barrel", "keg" },                 label = "Lean on it",          scenario = "WORLD_HUMAN_LEAN_BARREL",       distance = 0.1, face = "toward" },
        { keywords = { "rail", "fence", "balustrade" },   label = "Lean on the railing", scenario = "WORLD_HUMAN_LEAN_RAILING",      distance = 0.1, face = "toward" },
        { keywords = { "rail", "fence" },                 label = "Lean back on it",     scenario = "WORLD_HUMAN_LEAN_BACK_RAILING", distance = 0.1, face = "away" },
        { keywords = { "post", "pole", "pillar", "column", "lamp" }, label = "Lean on it (left)",  scenario = "WORLD_HUMAN_LEAN_POST_LEFT",  distance = 0.15, face = "away" },
        { keywords = { "post", "pole", "pillar", "column", "lamp" }, label = "Lean on it (right)", scenario = "WORLD_HUMAN_LEAN_POST_RIGHT", distance = 0.15, face = "away" },
        { keywords = { "table", "desk", "counter" },      label = "Wipe it down",        scenario = "WORLD_HUMAN_CLEAN_TABLE",       distance = 0.2, face = "toward" },
        { keywords = { "table", "desk" },                 label = "Write in a notebook", scenario = "WORLD_HUMAN_WRITE_NOTEBOOK",    distance = 0.2, face = "toward" },
        { keywords = { "table", "desk" },                 label = "Lean and read paper", scenario = "WORLD_HUMAN_LEAN_READ_PAPER",   distance = 0.2, face = "toward" },
        { keywords = { "counter", "shelf", "shelves", "display", "cabinet" }, label = "Browse", scenario = "WORLD_HUMAN_SHOP_BROWSE_COUNTER", distance = 0.3, face = "toward" },
        { keywords = { "crate", "box", "chest", "trunk", "sack", "bag", "basket", "cabinet", "drawer", "cupboard", "wardrobe", "suitcase", "luggage" },
          label = "Rummage through it", scenario = "WORLD_HUMAN_CROUCH_INSPECT", distance = 0.4, face = "toward" },
        { keywords = { "window" },                        label = "Clean the window",    scenario = "WORLD_HUMAN_CLEAN_WINDOW",      distance = 0.3, face = "toward" },
        { keywords = { "door" },                          label = "Knock",               scenario = "WORLD_HUMAN_KNOCK_DOOR",        distance = 0.4, face = "toward" },
        { keywords = { "campfire", "fire", "brazier", "firepit" }, label = "Stand by the fire", scenario = "WORLD_HUMAN_FIRE_STAND", distance = 0.5, face = "toward" },
        { keywords = { "campfire", "fire", "brazier", "firepit" }, label = "Sit by the fire",   scenario = "WORLD_HUMAN_FIRE_SIT",   distance = 0.6, face = "toward" },
        { keywords = { "campfire", "fire", "brazier", "firepit" }, label = "Tend the fire",     scenario = "WORLD_HUMAN_FIRE_TEND_KNEEL", distance = 0.5, face = "toward" },
        { keywords = { "cauldron", "pot", "stove", "kettle" }, label = "Stir the pot",    scenario = "WORLD_HUMAN_CAULDRON_STIR",     distance = 0.3, face = "toward" },
        { keywords = { "trough", "bucket", "basin", "washtub", "tub" }, label = "Wash your face", scenario = "WORLD_HUMAN_WASH_FACE_BUCKET_GROUND", distance = 0.3, face = "toward" },
        { keywords = { "washboard", "washtub", "basin" }, label = "Wash clothes",        scenario = "WORLD_HUMAN_WASHBOARD_BASIN",   distance = 0.3, face = "toward" },
        { keywords = { "grindstone" },                    label = "Use the grindstone",  scenario = "PROP_HUMAN_GRINDSTONE",         distance = 0.2, face = "toward" },
        { keywords = { "stump", "chopblock", "chopping" }, label = "Chop wood",          scenario = "PROP_HUMAN_WOOD_CHOP",          distance = 0.3, face = "toward" },
        { keywords = { "log", "lumber", "plank" },        label = "Saw wood",            scenario = "WORLD_HUMAN_SAW_WOOD",          distance = 0.3, face = "toward" },
        { keywords = { "anvil", "forge" },                label = "Hammer",              scenario = "WORLD_HUMAN_HAMMER_GROUND",     distance = 0.3, face = "toward" },
        { keywords = { "grave", "tombstone", "headstone" }, label = "Mourn",             scenario = "WORLD_HUMAN_GRAVE_MOURNING",    distance = 0.5, face = "toward" },
        { keywords = { "grave", "tombstone", "headstone" }, label = "Kneel and mourn",   scenario = "WORLD_HUMAN_GRAVE_MOURNING_KNEEL", distance = 0.5, face = "toward" },
        { keywords = { "step", "stair", "porch" },        label = "Sit on the steps",    scenario = "WORLD_HUMAN_SEAT_STEPS",        distance = 0.1, face = "away" },
        { keywords = { "ledge", "crate", "box", "barrel", "trunk", "log", "rock" }, label = "Sit on the edge", scenario = "WORLD_HUMAN_SEAT_LEDGE", distance = 0.1, face = "away" },
    },

    -- Some scenario points are the first of a chain: pick a bale up, carry
    -- it, set it down at the linked point. On, those rows run the whole
    -- chain (marked "carry" in the list) and what is set down stays. Off,
    -- they run as a single point, which leaves the character holding it.
    Chains = true,

    -- Print what the walk sees, what the ped's movement state is at every
    -- emote start and stop, and why anything stops, in the player's F8
    -- console. Leave off on a live server.
    Debug = false,

    -- The marker drawn on the thing the highlighted row belongs to, so the
    -- player can see which chair "Sit down" means.
    Marker = {
        Enabled = true,
        Type = 0x94FDAE17,                 -- a marker hash; this one is a flat cylinder
        Colour = { 254, 127, 156, 128 },   -- red, green, blue, alpha (0-255)
        Scale = 0.5,                       -- metres across
        Height = 0.0,                      -- lift above the spot, in metres
    },

    -- A wall found by a short ray scan around the player.
    -- headingOffset: 0 = back flat against the wall, -90 = left shoulder on it,
    -- 90 = right shoulder, 180 = facing it. distance = metres from the surface.
    Wall = {
        Enabled = true,
        Distance = 1.5,      -- how close a wall has to be, in metres
        Rays = 12,           -- directions checked; more finds walls at more angles
        Actions = {
            { label = "Lean back on the wall",         scenario = "WORLD_HUMAN_LEAN_BACK_WALL",            headingOffset = 0.0,   distance = 0.30 },
            { label = "Lean back, smoke",              scenario = "WORLD_HUMAN_LEAN_BACK_WALL_SMOKING",    headingOffset = 0.0,   distance = 0.30 },
            { label = "Lean back, drink",              scenario = "WORLD_HUMAN_LEAN_BACK_WALL_DRINKING",   headingOffset = 0.0,   distance = 0.30 },
            { label = "Lean on the wall (left side)",  scenario = "WORLD_HUMAN_LEAN_WALL_LEFT",            headingOffset = -90.0, distance = 0.35 },
            { label = "Lean on the wall (right side)", scenario = "WORLD_HUMAN_LEAN_WALL_RIGHT",           headingOffset = 90.0,  distance = 0.35 },
            { label = "Guard the wall",                scenario = "WORLD_HUMAN_GUARD_LEAN_WALL",           headingOffset = 0.0,   distance = 0.30 },
            { label = "Brace on the wall (drunk)",     scenario = "WORLD_HUMAN_DRUNK_BRACE_WALL_NO_VOMIT", headingOffset = 180.0, distance = 0.45 },
        },
    },
}

-- ============================================================================
--  The preview character
-- ============================================================================
-- A stand-in character appears beside the player while the menu is open and
-- acts out whatever emote is highlighted. Only the player sees it.

Config.Preview = {
    -- Show the preview character at all. Turn this off on a busy server, or
    -- if you would rather players tried emotes on themselves.
    Enabled = true,

    -- The character model used for the preview. Any valid ped model works.
    Model = "u_m_m_sdphotographer_01",

    -- How see-through the preview is, from 0 (invisible) to 255 (solid).
    Alpha = 200,

    -- Where the preview stands, in metres from the player. Right moves it
    -- across the screen, Forward moves it away from the camera (smaller is
    -- closer and so looks bigger), Height lifts or drops it.
    OffsetRight = 1.0,
    OffsetForward = 0.75,
    OffsetHeight = 0.0,

    -- Shine a soft light on the preview so it can still be seen at night.
    LightEnabled = true,

    -- The light's colour, as red, green and blue from 0 to 255.
    LightColour = { 250, 250, 250 },

    -- How far the light reaches, in metres.
    LightRange = 3.0,

    -- How bright the light is. Above about 25 it washes the character out.
    LightIntensity = 10.0,
}

-- ============================================================================
--  Your own emotes
-- ============================================================================
-- Everything below is yours. An update never changes it.

-- Emotes to add. A row here with the same name as one in shared/emotes.lua
-- replaces it, which is how you re-label, re-categorise or re-time a shipped
-- emote without editing code.
--
-- The fields are the same as shared/emotes.lua. The three shapes are:
--
--   { name = "mysalute", label = "Salute", category = "gestures", gender = "any",
--     kind = "kit", hash = 901097731 },
--
--   { name = "mylean", label = "Lean on a Post", category = "idle", gender = "any",
--     kind = "scenario", scenario = "WORLD_HUMAN_LEAN_POST_LEFT" },
--
--   { name = "mybook", label = "Read a Book", category = "sitting", gender = "any",
--     kind = "anim", dict = "amb_rest_sit@world_human_sit_ground@read_book@male_a@base",
--     anim = "base", speed = 2.0, speedX = 2.0, duration = -1, flags = 1,
--     prop = { model = "p_journal_open01x", bone = 22798,
--              x = 0.1, y = 0.07, z = -0.08, rx = 0.0, ry = 180.0, rz = 70.0 } },
--
-- A category that is not in Config.Menu.CategoryOrder will not be shown, so
-- add yours there too if you invent one.
Config.CustomEmotes = {}

-- Emote names to hide from the menu and from /e. Use this to drop emotes you
-- do not want on your server, e.g. { "pee", "vomitkneel" }.
Config.HiddenEmotes = {}
