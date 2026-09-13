Config = {}
Config.keys = {
    ["G"] = 0x760A9C6F,
    ["4"] = 0x8F9F9E58,
    ["DOWN"] = 0x05CA7C52,
    ["UP"] = 0x6319DB71,
    ["LEFT"] = 0xA65EBAB4,
    ["RIGHT"] = 0xDEB34313,
    ["1"] = 0xE6F612E4,
    ["2"] = 0x1CE6D9EB,
    ["B"] = 0x4CC0E2FE,
}
Config.denysceneinhideout = false 
Config.scenecommand = "scene" -- /scene
Config.mecommand = "me" -- /me
Config.ooccommand = "do" -- /do
Config.statuscommand = "status" -- /status
Config.stopdisplay = "cstatus" -- /cstatus stop displaying status.
Config.id = "id" -- /id
Config.cash = "cash" -- /cash
Config.scenevisabilitydistance = 5
Config.removejobs = {"police", "sheriff", "marshal"} -- jobs that can remove any scene (exact job names, case-sensitive)
Config.webhook = "YOUR DISCORD WEBHOOK HERE"
Config.joblock = false
Config.allowjobs = {}
-- ── /me Display Box ──────────────────────────────────────────────────────
-- Displays nearby /me messages in a textbox at the top-center of the screen.
Config.meboxcommand = "mebox"          -- /mebox auto | persist | off
Config.meboxdistance = 6              -- max distance (units) to see /me messages
Config.meboxtimeout = 30               -- seconds before auto-fade (auto mode)
Config.meboxmaxlines = 7               -- max visible lines before scrolling
Config.meboxJobColors = {            -- name color by job (exact job names, case-sensitive)
    ["police"]          = "#A0C4FF",
    ["sheriff"]         = "#A0C4FF",
    ["marshal"]         = "#97c796",
    ["doctor"]          = "#F4A0A0",
}
Config.meboxDefaultNameColor = "#eeebe4" -- fallback name color

-- ── Overhead /me Display (floating text above peds) ──────────────────────
Config.meoverheadduration = 10   -- seconds overhead /me, /do text stays visible above ped
Config.meoverheadradius   = 20   -- world-unit radius to see overhead /me texts

-- ── ME Box History & Dissipation ─────────────────────────────────────────
Config.meboxhistory       = 50   -- max messages kept in scroll-back buffer
Config.meboxdissipateafter = 300  -- seconds before an individual message is permanently removed

-- ── Scene / Status Color Swatches ────────────────────────────────
-- Hex values approximate in-game RDR2 text colors.
-- Order here = order shown in the editor toolbar (rainbow → neutrals).
Config.sceneColors = {
    -- reds / oranges
    { code = "~e~",  hex = "#c42020", title = "Red" },          -- COLOR_ENEMY
    { code = "~t8~", hex = "#b43547", title = "Crimson" },      -- COLOR_NET_PLAYER10
    { code = "~d~",  hex = "#d07830", title = "Orange" },       -- COLOR_ORANGE
    { code = "~t2~", hex = "#d07d44", title = "Warm Orange" },  -- COLOR_NET_PLAYER4
    -- yellows
    { code = "~t4~", hex = "#d0c777", title = "Mustard" },      -- COLOR_NET_PLAYER6
    { code = "~o~",  hex = "#d4b830", title = "Gold" },         -- COLOR_OBJECTIVE
    -- greens
    { code = "~t6~", hex = "#47a960", title = "Green" },        -- COLOR_NET_PLAYER8
    -- blues / cyans / teals
    { code = "~t3~", hex = "#78cbc9", title = "Cyan" },         -- COLOR_NET_PLAYER5
    { code = "~t7~", hex = "#085e6d", title = "Dark Teal" },    -- COLOR_NET_PLAYER9
    { code = "~pa~", hex = "#498eb2", title = "Blue" },         -- COLOR_POSSE_ALLY
    -- purples / pinks
    { code = "~t1~", hex = "#7f5490", title = "Purple" },       -- COLOR_NET_PLAYER3
    { code = "~u~",  hex = "#d801d8", title = "Magenta" },      -- COLOR_SCRIPT_VARIABLE2
    { code = "~t5~", hex = "#d293ae", title = "Light Pink" },   -- COLOR_NET_PLAYER7
    -- neutrals (dark → light)
    { code = "~v~",  hex = "#000000", title = "Black" },        -- COLOR_SCRIPT_VARIABLE
    { code = "~t~",  hex = "#928e8d", title = "Dark Grey" },    -- COLOR_MENU_GREY
    { code = "~m~",  hex = "#808080", title = "Grey" },         -- COLOR_MID_GREY_MP
    { code = "~f~",  hex = "#afaead", title = "Light Grey" },   -- COLOR_FRIENDLY
    { code = "~p~",  hex = "#e8e8e8", title = "Off-White" },    -- COLOR_RADAR_PICKUP
    { code = "~q~",  hex = "#ffffff", title = "White" },        -- COLOR_PURE_WHITE
}

-- ── UI Theme ──────────────────────────────────────────────────────────────
-- Controls the color scheme of the Scene / Status editor panels.
-- Available themes (the color in the name tells you what you get):
--   "amber-frontier"   Old West sepia amber & gold (default)
--   "steel-blue"       Modern dark navy with steel blue accents
--   "ivory-noir"       Black & white, no color tint
--   "lavender-dusk"    Soft pastel lavender & purple
--   "crimson-dusk"     Deep red, dramatic
--   "emerald-ridge"    Forest green, earthy
Config.uitheme = "amber-frontier"

Config.Language = {
    helloworld = "Aim And Press G To Register Scene",
    confirm = "Confirm",
    displaytext = "Display Text",
    pressup = "Press Up To Move Text",
    pressleft = "Press Left To Move Text",
    pressright = "Press Right To Move Text",
    pressdown = "Press Down To Move Text",
    press1 = "Press 1 To Move Text",
    press2 = "Press 2 To Move Text",
    pressg = "Press G To Confirm",
    pressg2 = "Press 4 To Remove",
    pressb = "Press B To Cancel",
    scenereg = "Scene Registered",
    sceneremoved = "Scene Removed",
    ooc = "~n~<font size=\"22\">~e~",
    stat = "~n~~n~Status: ",
    cash = "~n~~n~~n~Cash: ~t6~$ ",
    id = "~n~~n~~n~~n~~t3~ID: ",
    notinhideout = "Cant place scene in hideout",
    notjob = "You Dont Have The Right Job",
}