--[[===========================================================================
    poggy_markets · shared/locale.lua
    ---------------------------------------------------------------------------
    Looks up the strings in translations.lua.  Kept out of that file so it
    holds nothing but text.
===========================================================================]]--

PM = PM or {}

--- Look up a locale string and fill in its markers.
--- Returns the key itself when a string is missing, so a typo shows up in game
--- as the key rather than as an empty message.
---@param key string
---@param ... any
---@return string
function PM.L(key, ...)
    local str = PM.Locale[key]
    if not str then return tostring(key) end
    if select("#", ...) == 0 then return str end

    -- tostring every argument so numbers and nils cannot break string.format.
    local args = {}
    for i = 1, select("#", ...) do
        args[i] = tostring((select(i, ...)))
    end
    local ok, formatted = pcall(string.format, str, table.unpack(args))
    return ok and formatted or str
end
