--[[
    poggy_core — native RDR2 notification renderer, client side.

    Until 0.11.0 poggy_core drew notifications by calling poggy_util's exports.
    poggy_util is being retired, so the renderer lives here now. These are the
    same natives, with the same struct layouts, that poggy_util (and before it
    vorp_core) used, so every style looks exactly as it did. No NUI, no
    dependency: the game's own UI feed draws all of it.

    Nothing here is called by scripts directly. cl_notify.lua picks a renderer
    and calls PoggyCore.NativeNotify.<Style>(...).

    DataView by gottfriedleibniz:
    https://gist.github.com/gottfriedleibniz/8ff6e4f38f97dd43354a60f8494eedff
]]

PoggyCore = PoggyCore or {}

-- ---------------------------------------------------------------------------
-- DataView: packs the int64 structs the UI feed natives read
-- ---------------------------------------------------------------------------

local _strblob = string.blob or function(length)
    return string.rep("\0", math.max(40 + 1, length))
end

local DataView = {
    EndBig = ">",
    EndLittle = "<",
    Types = {
        Int8 = { code = "i1", size = 1 },
        Uint8 = { code = "I1", size = 1 },
        Int16 = { code = "i2", size = 2 },
        Uint16 = { code = "I2", size = 2 },
        Int32 = { code = "i4", size = 4 },
        Uint32 = { code = "I4", size = 4 },
        Int64 = { code = "i8", size = 8 },
        Uint64 = { code = "I8", size = 8 },
        LuaInt = { code = "j", size = 8 },
        UluaInt = { code = "J", size = 8 },
        LuaNum = { code = "n", size = 8 },
        Float32 = { code = "f", size = 4 },
        Float64 = { code = "d", size = 8 },
        String = { code = "z", size = -1 },
    },
    FixedTypes = {
        String = { code = "c", size = -1 },
        Int = { code = "i", size = -1 },
        Uint = { code = "I", size = -1 },
    },
}
DataView.__index = DataView

local function _ib(o, l, t) return ((t.size < 0 and true) or (o + (t.size - 1) <= l)) end
local function _ef(big) return (big and DataView.EndBig) or DataView.EndLittle end

local SetFixed = nil

function DataView.ArrayBuffer(length)
    return setmetatable({ offset = 1, length = length, blob = _strblob(length) }, DataView)
end

function DataView.Wrap(blob)
    return setmetatable({ offset = 1, blob = blob, length = blob:len() }, DataView)
end

function DataView:Buffer() return self.blob end
function DataView:ByteLength() return self.length end
function DataView:ByteOffset() return self.offset end

function DataView:SubView(offset)
    return setmetatable({ offset = offset, blob = self.blob, length = self.length }, DataView)
end

for label, datatype in pairs(DataView.Types) do
    DataView["Get" .. label] = function(self, offset, endian)
        local o = self.offset + offset
        if _ib(o, self.length, datatype) then
            local v, _ = string.unpack(_ef(endian) .. datatype.code, self.blob, o)
            return v
        end
        return nil
    end

    DataView["Set" .. label] = function(self, offset, value, endian)
        local o = self.offset + offset
        if _ib(o, self.length, datatype) then
            return SetFixed(self, o, value, _ef(endian) .. datatype.code)
        end
        return self
    end
end

for label, _ in pairs(DataView.FixedTypes) do
    DataView["GetFixed" .. label] = function(self, offset, typelen, endian)
        local o = self.offset + offset
        if o + (typelen - 1) <= self.length then
            local code = _ef(endian) .. "c" .. tostring(typelen)
            local v, _ = string.unpack(code, self.blob, o)
            return v
        end
        return nil
    end
    DataView["SetFixed" .. label] = function(self, offset, typelen, value, endian)
        local o = self.offset + offset
        if o + (typelen - 1) <= self.length then
            local code = _ef(endian) .. "c" .. tostring(typelen)
            return SetFixed(self, o, value, code)
        end
        return self
    end
end

SetFixed = function(self, offset, value, code)
    local fmt = {}
    local values = {}
    if self.offset < offset then
        local size = offset - self.offset
        fmt[#fmt + 1] = "c" .. tostring(size)
        values[#values + 1] = self.blob:sub(self.offset, size)
    end
    fmt[#fmt + 1] = code
    values[#values + 1] = value
    local ps = string.packsize(fmt[#fmt])
    if (offset + ps) <= self.length then
        local newoff = offset + ps
        local size = self.length - newoff + 1
        fmt[#fmt + 1] = "c" .. tostring(size)
        values[#values + 1] = self.blob:sub(newoff, self.length)
    end
    self.blob = string.pack(table.concat(fmt, ""), table.unpack(values))
    self.length = self.blob:len()
    return self
end

-- ---------------------------------------------------------------------------
-- Helpers
-- ---------------------------------------------------------------------------

local function loadTexture(dict)
    if not dict or HasStreamedTextureDictLoaded(dict) then return true end
    RequestStreamedTextureDict(dict, true)
    local tries = 0
    repeat
        Wait(0)
        tries = tries + 1
    until HasStreamedTextureDictLoaded(dict) or tries > 100
    return HasStreamedTextureDictLoaded(dict)
end

local function bigInt(value)
    local buf = DataView.ArrayBuffer(16)
    buf:SetInt64(0, value)
    return buf:GetInt64(0)
end

local function str(text)
    return bigInt(VarString(10, "LITERAL_STRING", tostring(text or "")))
end

local function hash(name)
    return bigInt(GetHashKey(tostring(name)))
end

local function ms(duration, default)
    return tonumber(duration) or default or 3000
end

-- ---------------------------------------------------------------------------
-- Styles
--
-- One function per on-screen style. Names match the style strings accepted by
-- the notify.styled verb, capitalised. The "timed" styles (Fail, Dead, Update,
-- Warning) have to be dismissed by hand after `duration`, so they wait; the
-- caller in cl_notify.lua runs them on their own thread.
-- ---------------------------------------------------------------------------

local N = {}

--- Bottom-centre tip.
function N.Tip(text, duration)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 3)
    data:SetUint64(8 * 1, str(text))
    Citizen.InvokeNative(0x049D5C615BD38BAD, cfg:Buffer(), data:Buffer(), 1)
end

--- Right-hand tip. The default for Poggy('notify').
function N.Right(text, duration)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 3)
    data:SetInt64(8 * 1, str(text))
    Citizen.InvokeNative(0xB2920B9760F0F36B, cfg:Buffer(), data:Buffer(), 1)
end

--- Bottom objective banner (poggy_util NotifyObjective, VORP vorp:TipBottom).
function N.Objective(text, duration)
    Citizen.InvokeNative(0xDD1232B332CBB9E7, 3, 1, 0)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 3)
    data:SetInt64(8 * 1, str(text))
    Citizen.InvokeNative(0xCEDBF17EFCC0E4A4, cfg:Buffer(), data:Buffer(), 1)
end

--- Top banner with a title and a subtitle (poggy_util NotifySimpleTop).
function N.Top(title, subtitle, duration)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 7)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(subtitle))
    Citizen.InvokeNative(0xA6F4216AB10EB08E, cfg:Buffer(), data:Buffer(), 1, 1)
end

--- Top location banner: a message under a place name (poggy_util NotifyTop).
function N.Location(text, location, duration)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 5)
    data:SetInt64(8 * 1, str(location))
    data:SetInt64(8 * 2, str(text))
    Citizen.InvokeNative(0xD05590C1AB38F068, cfg:Buffer(), data:Buffer(), 0, 1)
end

--- Right-hand notification with an icon (poggy_util NotifyAdvanced).
function N.Advanced(text, dict, icon, color, duration, quality, showQuality)
    loadTexture(dict)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    cfg:SetInt64(8 * 1, str("Transaction_Feed_Sounds"))
    cfg:SetInt64(8 * 2, str("Transaction_Positive"))
    local data = DataView.ArrayBuffer(8 * 10)
    data:SetInt64(8 * 1, str(text))
    data:SetInt64(8 * 2, str(dict))
    data:SetInt64(8 * 3, hash(icon))
    data:SetInt64(8 * 5, hash(color or "COLOR_WHITE"))
    if showQuality then
        data:SetInt32(8 * 6, tonumber(quality) or 1)
    end
    Citizen.InvokeNative(0xB249EBCB30DD88E0, cfg:Buffer(), data:Buffer(), 1)
    if dict then Citizen.InvokeNative(0x4ACA10A91F66F1E2, dict) end
end

--- Left-hand notification with an icon, a title and a subtitle (poggy_util NotifyLeft).
function N.Left(title, subtitle, dict, icon, duration, color)
    loadTexture(dict)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 8)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(subtitle))
    data:SetInt32(8 * 3, 0)
    data:SetInt64(8 * 4, hash(dict))
    data:SetInt64(8 * 5, hash(icon))
    data:SetInt64(8 * 6, hash(color or "COLOR_WHITE"))
    Citizen.InvokeNative(0x26E87218390E6729, cfg:Buffer(), data:Buffer(), 1, 1)
    if dict then Citizen.InvokeNative(0x4ACA10A91F66F1E2, dict) end
end

--- Left-hand rank-style notification (poggy_util NotifyLeftRank).
function N.LeftRank(title, subtitle, dict, icon, duration, color)
    dict = dict or "TOASTS_MP_GENERIC"
    loadTexture(dict)
    local cfg = DataView.ArrayBuffer(8 * 8)
    cfg:SetInt32(8 * 0, ms(duration, 5000))
    local data = DataView.ArrayBuffer(8 * 10)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(subtitle))
    data:SetInt64(8 * 4, hash(dict))
    data:SetInt64(8 * 5, hash(icon or "toast_mp_standalone_sp"))
    data:SetInt64(8 * 6, hash(color or "COLOR_WHITE"))
    data:SetInt32(8 * 7, 1)
    Citizen.InvokeNative(0x3F9FDDBA79117C69, cfg:Buffer(), data:Buffer(), 1, 1)
    Citizen.InvokeNative(0x4ACA10A91F66F1E2, dict)
end

--- Plain top notification (poggy_util NotifyBasicTop).
function N.BasicTop(text, duration)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 7)
    data:SetInt64(8 * 1, str(text))
    Citizen.InvokeNative(0x7AE0589093A2E088, cfg:Buffer(), data:Buffer(), 1)
end

--- Centre-screen text (poggy_util NotifyCenter).
function N.Center(text, duration, color)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 4)
    data:SetInt64(8 * 1, str(text))
    data:SetInt64(8 * 2, hash(color or "COLOR_PURE_WHITE"))
    Citizen.InvokeNative(0x893128CDB4B81FBB, cfg:Buffer(), data:Buffer(), 1)
end

--- Bottom-right text (poggy_util NotifyBottomRight).
function N.BottomRight(text, duration)
    local cfg = DataView.ArrayBuffer(8 * 7)
    cfg:SetInt32(8 * 0, ms(duration))
    local data = DataView.ArrayBuffer(8 * 5)
    data:SetInt64(8 * 1, str(text))
    Citizen.InvokeNative(0x2024F4F333095FB1, cfg:Buffer(), data:Buffer(), 1)
end

--- Draws a timed banner, waits, then clears it. Blocks for `duration`.
local function timed(native, data, duration)
    local cfg = DataView.ArrayBuffer(8 * 5)
    local handle = Citizen.InvokeNative(native, cfg:Buffer(), data:Buffer(), 1)
    Wait(ms(duration))
    Citizen.InvokeNative(0x00A15B94CBA4F76F, handle)
end

--- "Mission failed" banner (poggy_util NotifyFail). Blocks for `duration`.
function N.Fail(title, subtitle, duration)
    local data = DataView.ArrayBuffer(8 * 9)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(subtitle))
    timed(0x9F2CC2439A04E7BA, data, duration)
end

--- "Player dead" banner with a sound (poggy_util NotifyDead). Blocks for `duration`.
function N.Dead(title, audioRef, audioName, duration)
    local data = DataView.ArrayBuffer(8 * 9)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(audioRef))
    data:SetInt64(8 * 3, str(audioName))
    timed(0x815C4065AE6E6071, data, duration)
end

--- "Mission update" banner (poggy_util NotifyUpdate). Blocks for `duration`.
function N.Update(title, subtitle, duration)
    local data = DataView.ArrayBuffer(8 * 9)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(subtitle))
    timed(0x339E16B41780FC35, data, duration)
end

--- Warning banner with a sound (poggy_util NotifyWarning). Blocks for `duration`.
function N.Warning(title, subtitle, audioRef, audioName, duration)
    local data = DataView.ArrayBuffer(8 * 9)
    data:SetInt64(8 * 1, str(title))
    data:SetInt64(8 * 2, str(subtitle))
    data:SetInt64(8 * 3, str(audioRef))
    data:SetInt64(8 * 4, str(audioName))
    timed(0x339E16B41780FC35, data, duration)
end

PoggyCore.NativeNotify = N
