-- ============================================================================
--  poggy_chess · settings
--  Everything here can also be changed in game: /poggy → Chess & Checkers.
--  Words players read are in translations.lua.
-- ============================================================================

Config = Config or {}

-- Language of every message and screen: a key of Translations in translations.lua.
Config.Language = "en"

-- Prints what the script is doing to the server console. Leave off on a live server.
Config.Debug = false

-- Which games the tables offer. Turn one off to hide it from the start menu.
Config.Games = {
    Chess = true,
    Checkers = true,
}

-- ─── Tables ──────────────────────────────────────────────────────────────────
-- One row per table. The script places the table, the board and both chairs
-- itself at `origin` (the middle of the table, on the floor). `heading` turns
-- the whole table: at 0 the white chair is on the east side.
-- `id` is saved with every game played there: set it once and keep it.
Config.Tables = {
    {
        id      = "blackwater_courtyard",
        label   = "Blackwater Courtyard",
        origin  = vector3(-817.98, -1224.31, 42.98),
        heading = 90.0,
        blip    = { enabled = true, sprite = "blip_player", color = "BLIP_MODIFIER_MP_COLOR_32", scale = 0.25 },
    },
    {
        id      = "saint_denis_saloon",
        label   = "Saint Denis Saloon",
        origin  = vector3(2635.58, -1227.82, 58.59),
        heading = 270.0,
        blip    = { enabled = true, sprite = "blip_player", color = "BLIP_MODIFIER_MP_COLOR_32", scale = 0.25 },
    },
    {
        id      = "rhodes_saloon",
        label   = "Rhodes Saloon",
        origin  = vector3(1348.31, -1373.31, 79.49),
        heading = 171.23,
        blip    = { enabled = true, sprite = "blip_player", color = "BLIP_MODIFIER_MP_COLOR_32", scale = 0.25 },
    },
    {
        id      = "armadillo_saloon",
        label   = "Armadillo Saloon",
        origin  = vector3(-3703.11, -2591.65, -14.32),
        heading = 180.0,
        blip    = { enabled = true, sprite = "blip_player", color = "BLIP_MODIFIER_MP_COLOR_32", scale = 0.25 },
    },
    {
        id      = "sisika_prison",
        label   = "Sisika Prison",
        origin  = vector3(3350.78, -698.27, 43.0),
        heading = 226.89,
        blip    = { enabled = true, sprite = "blip_player", color = "BLIP_MODIFIER_MP_COLOR_32", scale = 0.25 },
    },
}

-- ─── Getting to the table ────────────────────────────────────────────────────
Config.Interaction = {
    -- Metres from a chair at which "Sit" shows.
    SitRadius = 1.8,
    -- Metres from a table at which its furniture and pieces appear, and a game
    -- in progress there can be watched.
    FurnitureRadius = 20.0,
    -- A seated player further than this many metres from the table (teleported,
    -- dragged) is stood up by the server.
    LeaveDistance = 8.0,
    -- Keys (control hashes). Sit and Select share a key: they are never on screen together.
    KeySit      = 0x760A9C6F,   -- G
    KeySelect   = 0x760A9C6F,   -- G: pick the square under the mouse
    KeyBack     = 0x3B24C470,   -- F: undo the last step of a move
    KeyStand    = 0xD51B784F,   -- E
    KeyCamera   = 0x9959A6F0,   -- C
    KeyResign   = 0x27D1C284,   -- R
    KeyDraw     = 0x26E9DC00,   -- Z
    KeyHint     = 0x84543902,   -- H
    KeyClaim    = 0x8CC9CD42,   -- X: claim the win when the opponent has stopped playing
}

-- How a player sits down and stands up.
Config.Seating = {
    -- true: the character walks to the chair and sits down the way the game's
    -- own people do. false: they are placed straight in the chair.
    Walk = true,
    -- If walking there takes longer than this (ms), the character is placed in the chair.
    WalkTimeoutMs = 6000,
    -- The sitting scenario, how far above the chair it plays, and its turn
    -- against the chair. Change these only if characters sit too high, too low
    -- or facing the wrong way.
    Scenario = "GENERIC_SEAT_BENCH_SCENARIO",
    ScenarioHeight = 0.5,
    ScenarioTurn = 180.0,
}

-- What happens when a seated player dies. Either way the game ends at once for
-- both players and they are stood up.
--   "abandon": no result is recorded, and any wager goes back to both players.
--   "forfeit": the player who died loses (a game against the AI is abandoned).
Config.Death = {
    Outcome = "abandon",
}

-- ─── Games ───────────────────────────────────────────────────────────────────
Config.Game = {
    -- Minutes a player may take over one move in a game against another player
    -- before their opponent can claim the win. 0 turns the claim off.
    IdleClaimMinutes = 5,
}

-- The chess clock, for chess and checkers. Whoever sets up a game picks the
-- time: minutes each for the whole game, plus seconds added to a player's
-- clock after each of their moves ("5+3" is 5 minutes and 3 seconds a move).
-- A clock starts with the first move. A player whose time runs out loses; in
-- chess it is a draw when the other side has only a king, or a king and one
-- bishop or knight. While a clock runs, the idle claim is not needed and is off.
Config.Clock = {
    Enabled = true,
    -- What the start menu opens on: "none" for no clock, or a time from Presets, such as "10+0".
    Default = "none",
    -- Offer the clock in games against the AI too (the AI's time runs while it thinks).
    AgainstAI = true,
    -- Let the player type their own minutes and seconds, within the limits below.
    Custom = true,
    MaxMinutes = 180,
    MaxIncrement = 60,
    -- Seconds left at which a clock turns red and shows tenths.
    LowSeconds = 20,
    -- The times offered as buttons, in this order.
    Presets = {
        { minutes = 1,  increment = 0 },
        { minutes = 3,  increment = 2 },
        { minutes = 5,  increment = 0 },
        { minutes = 10, increment = 0 },
        { minutes = 15, increment = 10 },
        { minutes = 30, increment = 0 },
    },
}

Config.Checkers = {
    -- The variant the start menu opens on: american, russian, brazilian, pool or custom.
    DefaultVariant = "american",
    -- Automatic draws. Repetition: the same position, same player to move, this
    -- many times. QuietMoves: this many moves by each player with no capture and
    -- no man moved (only kings moving about). 0 turns one off.
    Draws = {
        Repetition = 3,
        QuietMoves = 40,
    },
}

-- ─── The AI opponent ─────────────────────────────────────────────────────────
-- Who sits in the other chair in a game against the AI.
Config.AIOpponent = {
    Name = "Local Patron",
    Model = "a_m_m_blwupperclass_01",
}

-- The chess AI. Local levels play on the server's own engine; "api" levels ask
-- chess-api.com (Stockfish) and use the server's engine when it cannot be reached.
--   engine          "local" or "api"
--   localDepth      moves the server's engine looks ahead (1-3)
--   timeMs          most time the server's engine may think, in ms
--   depth           Stockfish search depth (1-18)
--   apiThinkMs      Stockfish time limit, in ms (1-100)
--   variants        moves Stockfish offers to choose from (1-5)
--   targetWinChance the AI steers for this winning chance of its own, in %; 0 = always the best move
--   noise           random spread on that target, in %
--   mistakeRate     chance (0-1) of a deliberate slip on any move
--   mistakeMinCp / mistakeMaxCp  size of a slip, in hundredths of a pawn
--   delayMs         shortest time the AI takes over a move (for feel)
--   drawMinMoves    moves each before it considers a draw offer
--   drawAcceptCp    it takes a draw when it is ahead by no more than this (hundredths of a pawn)
Config.ChessAI = {
    Enabled = true,
    DefaultLevel = "club",
    ApiUrl = "https://chess-api.com/v1",
    ApiTimeoutMs = 4000,
    Levels = {
        { id = "beginner",    engine = "local", localDepth = 1, timeMs = 400,  mistakeRate = 0.60, mistakeMinCp = 150, mistakeMaxCp = 600, delayMs = 500,  drawMinMoves = 0,  drawAcceptCp = 9999 },
        { id = "novice",      engine = "local", localDepth = 1, timeMs = 500,  mistakeRate = 0.38, mistakeMinCp = 100, mistakeMaxCp = 350, delayMs = 700,  drawMinMoves = 3,  drawAcceptCp = 9999 },
        { id = "casual",      engine = "local", localDepth = 2, timeMs = 700,  mistakeRate = 0.18, mistakeMinCp = 50,  mistakeMaxCp = 200, delayMs = 900,  drawMinMoves = 5,  drawAcceptCp = 200 },
        { id = "club",        engine = "local", localDepth = 3, timeMs = 1200, mistakeRate = 0.06, mistakeMinCp = 25,  mistakeMaxCp = 100, delayMs = 1100, drawMinMoves = 8,  drawAcceptCp = 100 },
        { id = "advanced",    engine = "api", depth = 6,  apiThinkMs = 50,  variants = 5, targetWinChance = 60, noise = 3, localDepth = 3, timeMs = 1500, mistakeRate = 0.02, mistakeMinCp = 10, mistakeMaxCp = 50, delayMs = 1300, drawMinMoves = 12, drawAcceptCp = 0 },
        { id = "expert",      engine = "api", depth = 12, apiThinkMs = 80,  variants = 5, targetWinChance = 70, noise = 0, localDepth = 3, timeMs = 1500, mistakeRate = 0.01, mistakeMinCp = 10, mistakeMaxCp = 10, delayMs = 1700, drawMinMoves = 15, drawAcceptCp = -100 },
        { id = "grandmaster", engine = "api", depth = 18, apiThinkMs = 100, variants = 1, targetWinChance = 0,  noise = 0, localDepth = 3, timeMs = 2000, mistakeRate = 0,    mistakeMinCp = 0,  mistakeMaxCp = 0,  delayMs = 2200, drawMinMoves = 20, drawAcceptCp = -100 },
    },
}

-- The checkers AI (always the server's own engine).
--   depth            moves it looks ahead at most
--   timeMs           most time it may think, in ms
--   noise            how loosely it picks among good moves (0 = always the best)
--   delayMs          shortest time it takes over a move (for feel)
--   drawMinMoves     moves each before it considers a draw offer
--   drawAcceptMargin it takes a draw when it is ahead by no more than this many men (a king counts extra)
Config.CheckersAI = {
    Enabled = true,
    DefaultLevel = "medium",
    Levels = {
        { id = "easy",   depth = 2,  timeMs = 400,  noise = 0.6,  delayMs = 500,  drawMinMoves = 0,  drawAcceptMargin = 99 },
        { id = "medium", depth = 6,  timeMs = 900,  noise = 0.15, delayMs = 900,  drawMinMoves = 5,  drawAcceptMargin = 0 },
        { id = "hard",   depth = 12, timeMs = 1800, noise = 0,    delayMs = 1400, drawMinMoves = 10, drawAcceptMargin = -99 },
    },
}

-- Hints, in games against the AI only.
Config.Hint = {
    Enabled = true,
    -- Shortest time between two hints, in ms.
    CooldownMs = 15000,
    -- How long a hint stays on the board, in ms.
    DisplayMs = 8000,
    -- Stockfish depth and time for a chess hint.
    Depth = 12,
    ThinkMs = 50,
}

-- ─── Wagers ──────────────────────────────────────────────────────────────────
-- Between two players only. The stake is taken from both when the game starts
-- and the winner receives both; a draw or an abandoned game gives each their
-- own back. A player who stands up or leaves the server mid-game loses.
Config.Wagers = {
    Enabled = true,
    -- "cash" or "gold".
    Currency = "cash",
    Min = 1,
    Max = 500,
    -- Seconds between checks for money owed to players who were away (a wager
    -- refunded after a restart). They are paid when next online.
    PayoutCheckSeconds = 120,
}

-- ─── Ratings ─────────────────────────────────────────────────────────────────
-- Elo ratings, one for chess and one for checkers, per character. Only
-- finished games between two players are rated; games against the AI never are.
Config.Ratings = {
    Enabled = true,
    -- Where every character starts.
    Start = 1200,
    -- A game needs at least this many moves (by both players together) to be rated.
    MinMoves = 2,
    -- How far a rating moves (FIDE's values): KProvisional for a player's
    -- first ProvisionalGames games, KMaster from MasterRating up, K otherwise.
    ProvisionalGames = 30,
    KProvisional = 40,
    K = 20,
    MasterRating = 2400,
    KMaster = 10,
    -- Rated games a player needs before they appear on the leaderboard.
    LeaderboardMinGames = 5,
}

-- ─── Records ─────────────────────────────────────────────────────────────────
Config.Records = {
    -- "My Record" opens with games against the AI hidden; players can show them.
    HideAIByDefault = true,
    -- Unfinished games nobody has picked up for this many days are closed
    -- (recorded as abandoned) when the script starts. 0 keeps them forever.
    AbandonAfterDays = 30,
}

-- ─── Screen ──────────────────────────────────────────────────────────────────
Config.UI = {
    -- Show the evaluation bar in chess games against the AI.
    EvalBar = true,
    -- How long notifications stay, in ms.
    NotifyMs = 4000,
    -- How much of each square a highlight fills (0.5-1.0).
    SquareFill = 0.85,
}

Config.Sounds = {
    -- 0 = silent, 1 = full.
    Volume = 0.6,
}

-- A warm light over the table while a game is on (only the players see it).
Config.TableLight = {
    Enabled = true,
    Height = 1.5,
    R = 255,
    G = 210,
    B = 140,
    Intensity = 20.0,
    Range = 3.0,
}

-- ─── Models and layout (advanced) ────────────────────────────────────────────
-- Change these only to use other props. Distances are in metres from the
-- table's origin: x towards the white chair, y towards the h-file, z up.
Config.Models = {
    Table = "p_endtable03x",
    Board = "p_chessset01x",
    Chair = "p_chair12x",
    Chess = {
        P = "p_wpawn_01x",   p = "p_bpawn_01x",
        N = "p_wknight_01x", n = "p_bknight_01x",
        B = "p_wbishop_01x", b = "p_bbishop_01x",
        R = "p_wrook_01x",   r = "p_brook_01x",
        Q = "p_wqueen_01x",  q = "p_bqueen_01x",
        K = "p_wking_01x",   k = "p_bking_01x",
    },
    -- r / b are men, R / B are kings (a double stack).
    Checkers = {
        r = "prop_chip_red_x2",   R = "prop_chip_red_x4",
        b = "prop_chip_black_x2", B = "prop_chip_black_x4",
    },
    -- Extra turn, in degrees, for pieces that face one way (the knights).
    PieceTurn = { N = 180.0, n = 0.0 },
}

Config.Geometry = {
    -- Height of a piece's base above the origin, and the top of a moving piece's arc.
    PieceHeight = 0.850005,
    LiftHeight = 0.910014,
    -- The board prop sits this far below the pieces, turned this many degrees.
    BoardDrop = 0.04,
    BoardTurn = -90.0,
    -- The centres of the four corner squares.
    BoardCorners = {
        r1a = { x = 0.217282, y = -0.220408 },
        r1h = { x = 0.215839, y = 0.221546 },
        r8a = { x = -0.216006, y = -0.218982 },
        r8h = { x = -0.217813, y = 0.221424 },
    },
    Chairs = {
        white = { x = 0.737882, y = 0.0, z = 0.037834, heading = 270.0 },
        black = { x = -0.737882, y = 0.0, z = 0.037834, heading = 90.0 },
    },
}

-- Cameras for each seat, cycled with the camera key. pos is where the camera
-- is; it looks at `look`, or is turned by `rot` when that is given.
Config.Cameras = {
    white = {
        { pos = vector3(0.60, 0.16, 1.300005), look = vector3(0.0, 0.0, 0.850005) },
        { pos = vector3(0.00, 0.00, 1.560005), rot = vector3(-90.0, 0.0, 90.0) },
        { pos = vector3(0.54, 0.79, 1.780005), look = vector3(0.0, 0.0, 0.850005) },
    },
    black = {
        { pos = vector3(-0.60, 0.16, 1.300005), look = vector3(0.0, 0.0, 0.850005) },
        { pos = vector3(0.00, 0.00, 1.560005), rot = vector3(-90.0, 0.0, 270.0) },
        { pos = vector3(-0.54, 0.79, 1.780005), look = vector3(0.0, 0.0, 0.850005) },
    },
}

-- How pieces move on the board.
Config.MoveAnim = {
    DurationMs = 600,
    TickMs = 16,
}
