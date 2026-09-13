--[[
    Poggy Trash Bins - translations

    Every player-facing string lives here. ~t6~, ~t8~ and ~d~ are RDR2 text
    colour codes; keep them in place when translating.
]]

-- Current language (change this to switch languages)
Config.Language = "en" -- Options: "en"

Translations = {
    ["en"] = {
        bin_name          = "Trash Bin",
        searching         = "Searching...",
        prompt_search     = "Search the Trash Bin",
        prompt_storage    = "Take a look inside the Trash Bin",
        found_prefix      = "You have found~t6~",
        found_nothing     = "~t8~You didnt find anything",
        already_searched  = "~t8~It looks like someone has already messed around here.",
        player_too_close  = "~d~Someone is standing too close to you!",
        wipe_command_help = "Trash Bins Wipe Command (Admin Only)",
        list_separator    = " , ",
        wiped_all         = "~t6~You have Wiped all Trash Bins on the server",
    },
}

-- Get a string for the current language; extra arguments fill %s markers.
function T(key, ...)
    local lang = Config.Language or "en"
    local set = Translations[lang] or Translations["en"]
    local text = set[key] or Translations["en"][key] or key
    if select('#', ...) > 0 then
        return string.format(text, ...)
    end
    return text
end
