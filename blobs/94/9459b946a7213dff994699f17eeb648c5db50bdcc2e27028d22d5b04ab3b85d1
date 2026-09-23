--[[
    poggy_core — the RedM frameworks, and how to reach each one.

    Split out of sh_detect.lua on 22 September 2026, when poggy_core gained a
    FiveM build: the detection LOGIC is identical on both games, only this table
    differs. The game layer supplies the table; common/shared/sh_detect.lua uses
    it. Loaded before sh_detect.lua by the manifest.
]]

PoggyCore = PoggyCore or {}

--- Which resource name identifies each framework, and how to reach its core.
--- `probe` runs on the server only; the client trusts the server's answer, which
--- avoids five different client-side handshakes that can all race resource start.
PoggyCore.Frameworks = {
    vorp = {
        resource = "vorp_core",
        label    = "VORP Core",
        -- Where vorp_inventory keeps item icons. Shared, so inv.imageBase gives
        -- the same answer on both sides without a round trip.
        itemImageBase = "nui://vorp_inventory/html/img/items/",
        probe    = function()
            -- GetCore() only. vorp_core also exposes vorpAPI(), which is
            -- deprecated and, more importantly, returns an object with no
            -- getUser — so every later character lookup against it fails and
            -- silently falls back to a database read. poggy_util calls vorpAPI()
            -- first today; that is the bug this line exists to avoid.
            local ok, core = pcall(function()
                return exports.vorp_core:GetCore()
            end)
            if not ok or type(core) ~= "table" or core.getUser == nil then
                return nil
            end
            if PoggyCore.IsCallable(core.getUser) then return core end

            -- Funcref metatables are not always inspectable, so if the cheap
            -- shape check is inconclusive, settle it by actually calling
            -- something harmless. getUsers() is a plain read.
            if pcall(function() return core.getUsers() end) then return core end
            return nil
        end,
    },
    rsg = {
        resource = "rsg-core",
        label    = "RSG Core",
        -- Where rsg-inventory keeps item icons: the NUI loads "images/" ..
        -- item.image (html/app.js). Note the file name is the item's `image`
        -- field, which is usually, not always, name .. '.png'.
        itemImageBase = "nui://rsg-inventory/html/images/",
        probe    = function()
            local ok, core = pcall(function()
                return exports["rsg-core"]:GetCoreObject()
            end)
            if ok and type(core) == "table" and core.Functions then return core end
            return nil
        end,
    },
    qbr = {
        resource = "qbr-core",
        label    = "QBCore RedM",
        -- Where qbr-inventory keeps item icons: the NUI loads "images/" ..
        -- item.image (html/js/app.js:654). The file name is the item's `image`
        -- field, which is often NOT name .. '.png' (bread -> consumable_bread_roll.png).
        itemImageBase = "nui://qbr-inventory/html/images/",
        probe    = function()
            -- QBR has no core object at all; every API is a flat export on
            -- qbr-core (server/functions.lua, server/player.lua, shared/main.lua).
            -- The handshake is the two exports the adapter cannot live without:
            -- GetPlayers (functions.lua:7-14) and GetItems (shared/main.lua:65),
            -- both plain reads that answer a table. The marker table stands in
            -- for the core object; the adapter talks to exports['qbr-core'].
            local ok, players, items = pcall(function()
                return exports["qbr-core"]:GetPlayers(), exports["qbr-core"]:GetItems()
            end)
            if ok and type(players) == "table" and type(items) == "table" then
                return { flat = true, resource = "qbr-core" }
            end
            return nil
        end,
    },
    redem = {
        resource = "redem_roleplay",
        label    = "RedEM:RP",
        probe    = function()
            local ok, core = pcall(function()
                return exports["redem_roleplay"]:RedEM()
            end)
            if ok and type(core) == "table" then return core end
            return nil
        end,
    },
    rpx = {
        resource = "rpx-core",
        label    = "RPX",
        probe    = function()
            local ok, core = pcall(function()
                return exports["rpx-core"]:GetObject()
            end)
            if ok and type(core) == "table" then return core end
            return nil
        end,
    },
}
