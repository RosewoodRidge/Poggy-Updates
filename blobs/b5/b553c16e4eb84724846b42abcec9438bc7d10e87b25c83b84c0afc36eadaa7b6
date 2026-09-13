Config = {}

---------------------------------------------------------------------------
--  SKILLCHECK SYSTEM
--  Dead-by-Daylight style circular skillcheck rendered in HTML/CSS/JS.
--  Triggered via export:  exports.poggy_skillcheck:StartSkillCheck(options)
--  Returns: { success = bool, great = bool, greatCount = number }
--
--  Parameters (all optional, defaults shown):
--    speed       (1-5)        Needle rotation speed. 1=slow, 5=very fast.
--    difficulty  (1-5)        Size of the success zone. 1=large, 5=small.
--    repetition  (1-10)       How many consecutive skillchecks must be passed.
--    randomizer  (0-5)        Random position offset. 0=centered, 5=up to 50%.
--    shake       (boolean)    Whether the skillcheck shakes.
--    shakeSpeed  (1-5)        How fast it shakes.
--    shakeDist   (1-5)        How far it shakes from origin.
--    timeBetween (100-500)    Ms between repetitions (only if repetition > 1).
--    direction   ("cw"|"ccw"|"rand")  Needle direction.
--    great       (0-100)      Percentage of success zone that is "great".
--    startOffset (0-50)       Random needle start offset (% of circle) from
--                             the point opposite the zone. 0=always exact, 8=±8%.
---------------------------------------------------------------------------
Config.SkillCheck = {
    Enabled = true,
    TestCommand = false, -- Enable /skillcheck test command (set true to enable)

    Defaults = {
        speed       = 3,
        difficulty  = 3,
        repetition  = 1,
        randomizer  = 0,
        shake       = false,
        shakeSpeed  = 2,
        shakeDist   = 2,
        timeBetween = 250,
        direction   = "cw",
        great       = 30,
        startOffset = 20,
    },

    Sounds = {
        Volume        = 0.1,
        IncomingBoost = 1,
        Incoming      = "incoming.mp3",
        Good          = "good.mp3",
        Great         = "great.mp3",
    },
}
