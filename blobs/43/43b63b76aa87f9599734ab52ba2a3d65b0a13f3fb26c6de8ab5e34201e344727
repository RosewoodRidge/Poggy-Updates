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
