--[[
    poggy_core — other scripts' screens, shown in poggy_core's page (0.27.0)

    RedM gives every resource with a `ui_page` its own frame in one shared
    browser, loaded when the player joins and kept for the whole session, used
    or not. A server with many such resources pays for every one of them all the
    time (Britannia DEV, 6 October 2026: past ~134 pages the game crashed).

    A Poggy script can hand its screen to poggy_core instead. Its manifest drops
    `ui_page` and names the page for poggy_core:

        poggy_ui 'ui/index.html'          -- the same file ui_page named
        poggy_ui_idle '60'                -- optional: unload after 60 s unused

    The script's code does not change. Its bridge (template/poggy.lua) sees the
    manifest and sends SendNUIMessage, SetNuiFocus and SetNuiFocusKeepInput here
    (the exports below). poggy_core's page (ui/hub/host.js) opens the script's
    page in a frame of its own the first time a message arrives, straight from
    the script's folder (https://cfx-nui-<script>/<page>), so the page's files,
    its callbacks and its saved browser data are exactly what they were. The
    page's fetch calls still reach the script's RegisterNUICallback: RedM routes
    them by the address, not by which frame sent them.

    Focus: poggy_core's own screens (menu, input, /poggy, /poggyui) and every
    hosted screen share poggy_core's one frame, so the frame's focus is decided
    here: SetNuiFocus and SetNuiFocusKeepInput are wrapped for poggy_core's own
    files (they keep calling the native names), and the answer is whoever wants
    focus, poggy_core's own screen first.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Host = PoggyCore.Host or {}

local Host = PoggyCore.Host

local nativeFocus = SetNuiFocus
local nativeKeepInput = SetNuiFocusKeepInput

local own = { focus = false, cursor = false, keep = false }   -- poggy_core's own screens
local apps = {}        -- [resource] = { page, focus, cursor, keep }
local order = {}       -- resources by the last time they took focus, newest last
local warned = {}

-- ---------------------------------------------------------------------------
-- Focus
-- ---------------------------------------------------------------------------

local function bump(res)
    for i = #order, 1, -1 do
        if order[i] == res then table.remove(order, i) end
    end
    order[#order + 1] = res
end

--- The hosted screen that has focus, newest first, or nil.
local function topApp()
    for i = #order, 1, -1 do
        local a = apps[order[i]]
        if a and a.focus then return order[i], a end
    end
    return nil
end

local function apply()
    local res, a = topApp()
    local focus = own.focus or a ~= nil
    local cursor = own.cursor or (a ~= nil and a.cursor) or false
    -- Keep-input belongs to whoever is in front: poggy_core's own screen when
    -- it is open (a menu must never let the game through), else the screen.
    local keep
    if own.focus then keep = own.keep else keep = a ~= nil and a.keep or false end
    nativeKeepInput(keep and true or false)
    nativeFocus(focus and true or false, cursor and true or false)
    SendNUIMessage({ poggyHost = { op = "focus", app = (not own.focus) and res or nil,
        cursor = (a ~= nil and a.cursor) or false } })
end

-- poggy_core's own files keep calling the native names.
function SetNuiFocus(focus, cursor)
    own.focus = focus and true or false
    own.cursor = cursor and true or false
    apply()
end

function SetNuiFocusKeepInput(keep)
    own.keep = keep and true or false
    apply()
end

-- ---------------------------------------------------------------------------
-- The hosted screens
-- ---------------------------------------------------------------------------

--- The page a resource asked poggy_core to show, or nil (and a warning once).
local function pageOf(res)
    local a = apps[res]
    if a then return a end
    local page = GetResourceMetadata(res, "poggy_ui", 0)
    if not page or page == "" then
        if not warned[res] then
            warned[res] = true
            print(("^3[Poggy Core] %s sent a screen message but its manifest has no poggy_ui line; ignored.^7"):format(res))
        end
        return nil
    end
    page = page:gsub("^/+", "")
    local idle = tonumber(GetResourceMetadata(res, "poggy_ui_idle", 0) or "") or 0
    a = { page = page, idle = idle, focus = false, cursor = false, keep = false }
    apps[res] = a
    return a
end

local function header(res, a, op)
    return { op = op, app = res, url = ("https://cfx-nui-%s/%s"):format(res, a.page), idle = a.idle }
end

--- SendNUIMessage, for a hosted screen.
local function send(res, data)
    local a = pageOf(res)
    if not a then return false end
    local m = header(res, a, "send")
    m.data = data
    SendNUIMessage({ poggyHost = m })
    return true
end

local function focus(res, f, c)
    local a = pageOf(res)
    if not a then return false end
    a.focus = f and true or false
    a.cursor = c and true or false
    if a.focus then
        bump(res)
        -- The screen is opened if this is the first thing the script did.
        SendNUIMessage({ poggyHost = header(res, a, "ensure") })
    end
    apply()
    return true
end

local function keepInput(res, k)
    local a = pageOf(res)
    if not a then return false end
    a.keep = k and true or false
    apply()
    return true
end

--- Unload a hosted screen now (its next message opens it fresh).
local function close(res)
    local a = apps[res]
    if not a then return false end
    apps[res] = nil
    SendNUIMessage({ poggyHost = { op = "unload", app = res } })
    apply()
    return true
end

Host.Send, Host.Focus, Host.KeepInput, Host.Close = send, focus, keepInput, close

-- Called by template/poggy.lua in the script itself; the caller is the screen's owner.
exports("HostSend", function(data) local r = GetInvokingResource(); return r ~= nil and send(r, data) end)
exports("HostFocus", function(f, c) local r = GetInvokingResource(); return r ~= nil and focus(r, f, c) end)
exports("HostKeepInput", function(k) local r = GetInvokingResource(); return r ~= nil and keepInput(r, k) end)
exports("HostClose", function() local r = GetInvokingResource(); return r ~= nil and close(r) end)

-- A script that stops takes its screen (and any focus it held) with it.
AddEventHandler("onClientResourceStop", function(res)
    if res == GetCurrentResourceName() then return end
    if apps[res] then close(res) end
    warned[res] = nil
end)
