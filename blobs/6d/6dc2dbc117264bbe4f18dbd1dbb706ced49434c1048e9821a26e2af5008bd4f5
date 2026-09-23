--[[
    poggy_core — the Poggy theme: presets (0.23.0)

    One look for every Poggy script's screens, the Poggy inventory's: dark
    panels, a thin light edge, square corners, cream text, one accent colour.
    The server owner picks a preset (or "custom") in PoggyCoreConfig.Theme;
    every script's page follows it through ui/hub/theme.js.

    A preset is a base palette only. theme.js works every other shade out of
    these (hover, glow, borders, wells), so a preset or a custom theme is
    always complete.

        background  the panel colour
        text        the main text colour
        accent      titles' highlights, selection, primary buttons, focus
        success / danger / warning / info   what those words say
        opacity     how solid a panel is, 0.5 to 1
        font        "fell" (period serif titles, plain numbers: the default),
                    "serif" (one book serif throughout), "clean" (all sans)
        corners     corner rounding in pixels, 0 (square, the default) to 16

    Loaded on both sides: the server resolves what it sends, the client
    resolves a script's theme for its page.
]]

PoggyCore = PoggyCore or {}

local Themes = {}
PoggyCore.Themes = Themes

--- In the order the settings hub lists them.
Themes.Order = {
    "rosewood", "blackwater", "lemoyne", "saint_denis", "ambarino",
    "tumbleweed", "outlaw", "silver", "ledger", "custom",
}

Themes.Presets = {
    rosewood = {
        label = "Rosewood (black and gold)",
        background = "#141414", text = "#f6efe3", accent = "#d6ad68",
        success = "#8fbf7f", danger = "#d48a8a", warning = "#e0b35a", info = "#8fb3d9",
        opacity = 0.94, font = "fell", corners = 0,
    },
    blackwater = {
        label = "Blackwater Steel (slate and steel blue)",
        background = "#14171b", text = "#e8edf1", accent = "#9db6cc",
        success = "#86b89a", danger = "#d98c8c", warning = "#d9b46a", info = "#8fb3d9",
        opacity = 0.94, font = "fell", corners = 0,
    },
    lemoyne = {
        label = "Lemoyne Moss (dark green and moss)",
        background = "#121712", text = "#eef0e0", accent = "#a8c07a",
        success = "#8fbf7f", danger = "#d48a8a", warning = "#d9b85f", info = "#8fb8c9",
        opacity = 0.94, font = "fell", corners = 0,
    },
    saint_denis = {
        label = "Saint Denis Wine (burgundy and rose)",
        background = "#1a1013", text = "#f5e8e6", accent = "#d88a8e",
        success = "#93bf8a", danger = "#e07a6e", warning = "#e0b35a", info = "#9ab0d6",
        opacity = 0.94, font = "fell", corners = 0,
    },
    ambarino = {
        label = "Ambarino Frost (midnight and ice)",
        background = "#0f151c", text = "#eaf3f7", accent = "#8fd0e6",
        success = "#8fc9a8", danger = "#e08c8c", warning = "#e6c170", info = "#a7c4ec",
        opacity = 0.94, font = "fell", corners = 0,
    },
    tumbleweed = {
        label = "Tumbleweed Copper (leather and copper)",
        background = "#1a1511", text = "#f3e7d9", accent = "#d18a52",
        success = "#9bbf7f", danger = "#d8837a", warning = "#e3b25c", info = "#90b0cf",
        opacity = 0.94, font = "fell", corners = 0,
    },
    outlaw = {
        label = "Outlaw Crimson (black and blood red)",
        background = "#0e0d0d", text = "#f2ebe5", accent = "#c9463d",
        success = "#8fbf7f", danger = "#ef8a6b", warning = "#e0b35a", info = "#8fb3d9",
        opacity = 0.95, font = "fell", corners = 0,
    },
    silver = {
        label = "Silver Dollar (charcoal and silver)",
        background = "#151515", text = "#f0f0f0", accent = "#c9ccd1",
        success = "#8fbf7f", danger = "#d48a8a", warning = "#e0b35a", info = "#8fb3d9",
        opacity = 0.94, font = "fell", corners = 0,
    },
    ledger = {
        label = "Ledger (light parchment and ink)",
        background = "#efe6d2", text = "#2b2118", accent = "#8a5a22",
        success = "#3f7a3a", danger = "#a8423a", warning = "#9a6a12", info = "#2f5f8a",
        opacity = 0.97, font = "fell", corners = 0,
    },
}

Themes.Default = "rosewood"

Themes.Fonts = { fell = true, serif = true, clean = true }

local function hex(v, fallback)
    if type(v) == "string" then
        local h = v:match("^%s*#?(%x%x%x%x%x%x)%s*$") or v:match("^%s*#?(%x%x%x)%s*$")
        if h then return "#" .. h:lower() end
    end
    return fallback
end

--- The palette a theme config names: a preset's, or the custom one, each
--- colour checked and anything missing taken from Rosewood.
---   cfg  PoggyCoreConfig.Theme (or the server's copy of it)
--- Returns preset id, palette.
function Themes.Palette(cfg)
    cfg = type(cfg) == "table" and cfg or {}
    local id = type(cfg.Preset) == "string" and cfg.Preset:lower() or Themes.Default
    local base = Themes.Presets[Themes.Default]
    local src
    if id == "custom" then
        local c = type(cfg.Custom) == "table" and cfg.Custom or {}
        src = {
            background = c.Background, text = c.Text, accent = c.Accent,
            success = c.Success, danger = c.Danger, warning = c.Warning, info = c.Info,
            opacity = c.Opacity, font = c.Font, corners = c.Corners,
        }
    else
        src = Themes.Presets[id]
        if not src then id, src = Themes.Default, base end
    end

    local opacity = tonumber(src.opacity) or base.opacity
    if opacity < 0.5 then opacity = 0.5 elseif opacity > 1 then opacity = 1 end
    local corners = math.floor(tonumber(src.corners) or 0)
    if corners < 0 then corners = 0 elseif corners > 16 then corners = 16 end
    local font = type(src.font) == "string" and src.font:lower() or "fell"
    if not Themes.Fonts[font] then font = "fell" end

    return id, {
        background = hex(src.background, base.background),
        text       = hex(src.text,       base.text),
        accent     = hex(src.accent,     base.accent),
        success    = hex(src.success,    base.success),
        danger     = hex(src.danger,     base.danger),
        warning    = hex(src.warning,    base.warning),
        info       = hex(src.info,       base.info),
        opacity    = opacity,
        font       = font,
        corners    = corners,
    }
end

--- Does this script keep its own look? cfg.OwnLook lists poggy_ids.
function Themes.KeepsOwnLook(cfg, poggyId)
    local list = type(cfg) == "table" and cfg.OwnLook or nil
    if type(list) ~= "table" or not poggyId then return false end
    for _, v in ipairs(list) do
        if v == poggyId then return true end
    end
    return false
end

--- What a script's page is sent (ui/hub/theme.js reads it).
---   { id, enabled, preset, rev, palette }
function Themes.For(cfg, poggyId, rev)
    local preset, palette = Themes.Palette(cfg)
    if Themes.KeepsOwnLook(cfg, poggyId) then
        return { id = poggyId, enabled = false, preset = preset, rev = rev }
    end
    return { id = poggyId, enabled = true, preset = preset, rev = rev, palette = palette }
end
