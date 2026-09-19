--[[
    poggy_core — the settings hub, server side (0.18.0).  PoggyCore.Hub

    /poggy opens a full-screen hub in game (client/cl_hub.lua, ui/hub/). Every
    Poggy script on the server gets a card; opening one shows its settings,
    lists, commands, README and history, and lets an admin change them. The
    contract is docs/reference/poggy-hub-spec.md; this file and sv_hub_save.lua
    are its §4 and the server half of §6.

    The config file is the source of truth. The hub edits a script's config
    files in place through PoggyCore.SettingsModel (sv_settings_model.lua), so
    every comment and every line it does not change stays byte-identical, and
    writes them through the script's own bridge (PoggyWriteOwnFile). There is no
    override layer. The database (poggy_settings, poggy_settings_log) is a
    mirror and a history, rebuilt from the files, never read back into them.

    This file holds:
      - discovery: which scripts exist, which of their files are config files,
        their docs/hub.json, README and help pages (all read with
        LoadResourceFile, which works for stopped scripts too)
      - the model cache: each config file parsed once per fingerprint
      - hub.json metadata merged onto each setting (exact paths, ".*" prefix
        rules, inheritance), the card, and the global search index
      - permissions: the ACE poggy.settings, or a framework admin; taking over
        someone's lock needs poggy.settings.takeover. Every call re-checks.
      - locks: one editor per script, idle timeout with a warning, takeover,
        release on close and on disconnect. Hub.IsLocked(id) is what the
        updater asks before it writes a script's files.
      - item checks (§9.1): every item name a script's config uses, checked
        against the server's item registry (exact case) and for its icon
        file, when the script's hub.json says "itemCheck": true; the whole
        registry for type-ahead (`itemList`) and checks of newly typed names
        (`itemCheck`)
      - script diagnostics (§9.2): a running script's own report on its rows,
        from the server export its hub.json names ("diagnostics")
      - data panels (§10): live data a script owns (its database), shown as a
        table and edited cell by cell, with actions and drill-down, through
        that script's own export (`panel`, `panelWrite`, `panelAction`), and
        container contents built from the storage verbs (`container`,
        `containerAction`); every change is logged to History
      - the callbacks poggy_core:hub:<call> and the push event
        poggy_core:hub:event (type, data) the client relays to the page
      - `poggycore settings` for the server console (routed from sv_commands.lua)

    sv_hub_save.lua holds saving, validation, backups, history, the database
    mirror, defaults from the update feed, restarts, and roles.

    Every answer to the page has one shape:
        { ok = true, value = ... }
        { ok = false, err = "code", message = "plain English", ...extra }
]]

PoggyCore = PoggyCore or {}

local Hub = PoggyCore.Hub or {}
PoggyCore.Hub = Hub

local Util = PoggyCore.Util
local SELF = GetCurrentResourceName()

--- Internal functions shared with sv_hub_save.lua. Not an API.
local I = Hub.Internal or {}
Hub.Internal = I

local EVENT = "poggy_core:hub:event"

local CATEGORIES = { "Economy", "Jobs", "World", "Roleplay", "Admin", "Utility", "Framework" }

local function cfg() return PoggyCoreConfig.Hub or {} end
local function M() return PoggyCore.SettingsModel end

--- Seconds since the epoch. One place, so a test can move the clock.
function I.now() return os.time() end

-- ---------------------------------------------------------------------------
-- Small helpers
-- ---------------------------------------------------------------------------

--- Deep equality for plain data (encoded values, decoded JSON).
function I.equal(a, b, depth)
    if a == b then return true end
    if type(a) ~= "table" or type(b) ~= "table" then return false end
    depth = (depth or 0) + 1
    if depth > 32 then return false end
    for k, v in pairs(a) do
        if not I.equal(v, b[k], depth) then return false end
    end
    for k in pairs(b) do
        if a[k] == nil then return false end
    end
    return true
end

--- A shallow copy.
function I.copy(t)
    local out = {}
    for k, v in pairs(t or {}) do out[k] = v end
    return out
end

--- json.decode that never throws. Returns value or nil, error.
function I.decode(text)
    if type(text) ~= "string" then return nil, "not text" end
    local ok, value = pcall(json.decode, text)
    if not ok then return nil, tostring(value) end
    return value
end

function I.encode(value)
    local ok, text = pcall(json.encode, value)
    if ok then return text end
    return nil
end

--- "MinBid" -> "Min bid", "drop_interval" -> "Drop interval", "HTTPPort" -> "HTTP port".
--- Keys are config text: explicit ASCII ranges only (%a, %u, %l and %s follow
--- the C locale, which on Windows can match UTF-8 bytes and split a character).
function I.readable(key)
    local s = tostring(key or "")
    s = s:gsub("^%[ *[\"']?", ""):gsub("[\"']? *%]$", "")
    s = s:gsub("[_%-]+", " ")
    s = s:gsub("([a-z])([A-Z])", "%1 %2")
    s = s:gsub("([A-Z])([A-Z][a-z])", "%1 %2")
    s = s:gsub("([A-Za-z])([0-9])", "%1 %2")
    s = s:gsub("[ \t\r\n]+", " "):match("^ ?(.-) ?$")
    local words = {}
    for w in s:gmatch("[^ ]+") do
        if #words > 0 and not w:match("^[A-Z][A-Z]+$") then
            w = (w:gsub("[A-Z]", function(c) return string.char(c:byte() + 32) end))
        end
        words[#words + 1] = w
    end
    s = table.concat(words, " ")
    return (s:gsub("^[a-z]", function(c) return string.char(c:byte() - 32) end))
end

--- The label a script shows when its hub.json names none.
function I.folderLabel(folder)
    if folder == "poggy_core" then return "Poggy Core" end
    local base = tostring(folder):gsub("^poggy[_%-]", "")
    return I.readable(base)
end

-- ---------------------------------------------------------------------------
-- Paths
--
-- Config.Stores[3].label   Config.Lang["en"].greeting   Config.badges.sheriff
-- ---------------------------------------------------------------------------

--- The segments of a path: { { key = "Config" }, { key = 3, index = true }, ... }.
--- Returns nil for a path it cannot read.
function I.splitPath(path)
    if type(path) ~= "string" or path == "" or #path > 512 then return nil end
    local segs, i, n = {}, 1, #path
    local first = path:match("^[%a_][%w_]*")
    if not first then return nil end
    segs[1] = { key = first }
    i = #first + 1
    while i <= n do
        local c = path:sub(i, i)
        if c == "." then
            local id = path:match("^[%a_][%w_]*", i + 1)
            if not id then return nil end
            segs[#segs + 1] = { key = id }
            i = i + 1 + #id
        elseif c == "[" then
            local num = path:match("^%[(%-?%d+)%]", i)
            if num then
                segs[#segs + 1] = { key = tonumber(num), index = true }
                i = i + #num + 2
            else
                local q = path:sub(i + 1, i + 1)
                if q ~= '"' and q ~= "'" then return nil end
                local j, buf = i + 2, {}
                while j <= n do
                    local d = path:sub(j, j)
                    if d == "\\" then
                        buf[#buf + 1] = path:sub(j + 1, j + 1)
                        j = j + 2
                    elseif d == q then
                        break
                    else
                        buf[#buf + 1] = d
                        j = j + 1
                    end
                end
                if path:sub(j, j + 1) ~= q .. "]" then return nil end
                segs[#segs + 1] = { key = table.concat(buf), bracket = true }
                i = j + 2
            end
        else
            return nil
        end
    end
    return segs
end

--- Every proper ancestor of a path, nearest first, in canonical spelling
--- (I.canon): look them up with I.nodeAt, not by string.
function I.ancestors(path)
    local out, segs = {}, I.splitPath(path)
    if not segs then return out end
    local parts = {}
    for idx, s in ipairs(segs) do
        if idx == 1 then
            parts[idx] = s.key
        elseif s.index then
            parts[idx] = ("[%d]"):format(s.key)
        elseif tostring(s.key):match("^[%a_][%w_]*$") then
            parts[idx] = "." .. s.key
        else
            parts[idx] = ('[%q]'):format(s.key)
        end
    end
    for k = #parts - 1, 1, -1 do out[#out + 1] = table.concat(parts, "", 1, k) end
    return out
end

local canonIndex = setmetatable({}, { __mode = "k" })   -- nodes table -> { canonical path -> node }

--- The node at a path in a nodes table (model.nodes, or a build's byPath),
--- whatever spelling either side uses.
function I.nodeAt(nodes, path)
    if type(nodes) ~= "table" or type(path) ~= "string" then return nil end
    local direct = nodes[path]
    if direct then return direct end
    local idx = canonIndex[nodes]
    if not idx then
        idx = {}
        for p, n in pairs(nodes) do idx[I.canon(p)] = n end
        canonIndex[nodes] = idx
    end
    return idx[I.canon(path)]
end

--- One spelling for every path, so `Config.Lang["en"]` and `Config.Lang.en`
--- (the same Lua key) match: identifier-like string keys as .key, other
--- strings as ["key"], integers as [n]. Unreadable paths come back as they are.
function I.canon(path)
    local segs = I.splitPath(path)
    if not segs then return path end
    local parts = { segs[1].key }
    for idx = 2, #segs do
        local sg = segs[idx]
        if sg.index then
            parts[idx] = ("[%d]"):format(sg.key)
        elseif tostring(sg.key):match("^[%a_][%w_]*$") then
            parts[idx] = "." .. sg.key
        else
            parts[idx] = ("[%q]"):format(sg.key)
        end
    end
    return table.concat(parts)
end

--- "loot[2].count" -> "loot[].count": the form list field meta is keyed by.
function I.fieldKey(rel)
    return (tostring(rel):gsub("%[%-?%d+%]", "[]"):gsub("^%.", ""))
end

-- ---------------------------------------------------------------------------
-- Pushes (§6.3): the client relays them to the page as { action = 'hub:event', type, data }
-- ---------------------------------------------------------------------------

local viewers = {}   -- src -> true while that player has the hub open

function I.push(target, kind, data)
    if not target then return end
    TriggerClientEvent(EVENT, target, kind, data or {})
end

function I.pushViewers(kind, data)
    for src in pairs(viewers) do I.push(src, kind, data) end
end

function I.viewers() return viewers end

-- ---------------------------------------------------------------------------
-- Permissions (§4.2)
-- ---------------------------------------------------------------------------

local function ace(src, object)
    local ok, allowed = pcall(IsPlayerAceAllowed, src, object)
    return ok and allowed == true
end

--- May this player open the hub and edit? The console (0) always may.
function Hub.Allowed(src)
    src = tonumber(src)
    if src == 0 then return true end
    if not src or src < 0 then return false end
    if ace(src, "poggy.settings") then return true end
    local ok, isAdmin = PoggyCore.Do("perms.isAdmin", { src = src })
    return ok == true and isAdmin == true
end

--- May this player take someone else's lock? The ACE only.
function Hub.CanTakeover(src)
    src = tonumber(src)
    if src == 0 then return true end
    return src ~= nil and ace(src, "poggy.settings.takeover")
end

--- { name, identifier } of a player, or the console.
function I.who(src)
    src = tonumber(src)
    if not src or src == 0 then return { src = 0, name = "console", identifier = "console" } end
    local name = GetPlayerName(src) or ("player " .. src)
    local identifier
    if GetPlayerIdentifierByType then
        local ok, id = pcall(GetPlayerIdentifierByType, src, "license")
        if ok and type(id) == "string" and id ~= "" then identifier = id end
    end
    if not identifier and GetNumPlayerIdentifiers then
        for i = 0, (GetNumPlayerIdentifiers(src) or 0) - 1 do
            local id = GetPlayerIdentifier(src, i)
            if type(id) == "string" and id:find("^license:") then identifier = id break end
        end
    end
    return { src = src, name = name, identifier = identifier }
end

-- ---------------------------------------------------------------------------
-- Discovery (§4.1)
-- ---------------------------------------------------------------------------

local scanAt = nil
local SCAN_SECONDS = 30

local function manifestField(text, key)
    local Id = PoggyCore.Identity
    return Id and Id.ManifestField(text, key) or nil
end

--- Every Poggy script on the server, running or not, sorted by id:
--- { id, folder, version, running }. Scripts whose id two folders claim
--- resolve to the one that is running, else are left out.
function Hub.Scripts(fresh)
    local Id = PoggyCore.Identity
    if not Id then return {} end
    if fresh or not scanAt or I.now() - scanAt >= SCAN_SECONDS then
        Id.Refresh()
        scanAt = I.now()
    end
    local seen, out = {}, {}
    local function add(id, folder)
        if seen[id] or not folder then return end
        seen[id] = true
        local okState, state = pcall(GetResourceState, folder)
        state = okState and state or "missing"
        if state == "missing" or state == "unknown" then return end
        local text = LoadResourceFile(folder, "fxmanifest.lua")
        out[#out + 1] = {
            id = id,
            folder = folder,
            version = manifestField(text, "version") or GetResourceMetadata(folder, "version", 0),
            running = state == "started",
            state = state,
        }
    end
    local registered = Id.List()
    for _, e in ipairs(registered) do add(e.id, e.folder) end
    local scan = Id.Scan()
    for id in pairs(scan.claims or {}) do
        if not seen[id] then
            local okLoc, folder = pcall(Id.Locate, id)
            if okLoc and folder then add(id, folder) end
        end
    end
    table.sort(out, function(a, b) return a.id < b.id end)
    return out
end

--- One script by id (or folder name), or nil plus "missing".
function Hub.Find(id)
    if type(id) ~= "string" or id == "" then return nil, "missing" end
    for _, s in ipairs(Hub.Scripts()) do
        if s.id == id then return s end
    end
    for _, s in ipairs(Hub.Scripts(true)) do
        if s.id == id or s.folder == id then return s end
    end
    return nil, "missing"
end

--- Glob from a manifest (`config/*.lua`, `ui/**`) as a Lua pattern.
local function globPattern(glob)
    local p = glob:gsub("[%^%$%(%)%%%.%[%]%+%-%?]", "%%%0")
    p = p:gsub("%*%*", "\1"):gsub("%*", "[^/]*"):gsub("\1", ".*")
    return "^" .. p .. "$"
end

--- Every entry of one manifest key, from the metadata the server scanned.
local function metadataList(folder, key)
    local out = {}
    local okN, n = pcall(GetNumResourceMetadata, folder, key)
    for i = 0, ((okN and tonumber(n)) or 0) - 1 do
        local ok, v = pcall(GetResourceMetadata, folder, key, i)
        if ok and type(v) == "string" and v ~= "" then out[#out + 1] = (v:gsub("\\", "/")) end
    end
    return out
end

--- Is rel listed in the manifest's files (exactly or by a glob)?
function I.manifestListsFile(folder, rel)
    for _, entry in ipairs(metadataList(folder, "file")) do
        if entry == rel or (entry:find("*", 1, true) and rel:match(globPattern(entry))) then return true end
    end
    return false
end

local function isConfigPath(rel)
    local lower = rel:lower()
    return lower == "config.lua" or lower == "translations.lua" or lower:match("^config/[^/]+%.lua$") ~= nil
end

local function isTranslationPath(rel)
    local base = (rel:lower():match("([^/]+)$")) or ""
    return base:find("^translation") ~= nil or base:find("^locale") ~= nil or base:find("^lang") ~= nil
end

--- The config files of a script (§4.1): hub.json `files` when given, else
--- every config.lua, config/*.lua and translations.lua the manifest loads.
--- Returns { { file, dictKeys } } in manifest order.
function Hub.ConfigFiles(script, meta)
    local out, seen = {}, {}
    local function add(rel)
        rel = tostring(rel):gsub("\\", "/"):gsub("^%./", "")
        if seen[rel] or rel == "" or rel:find("..", 1, true) or rel:find(":", 1, true) or rel:find("^/") then return end
        if rel:sub(-4):lower() ~= ".lua" then return end
        if not LoadResourceFile(script.folder, rel) then return end
        seen[rel] = true
        out[#out + 1] = { file = rel, dictKeys = isTranslationPath(rel) }
    end
    if meta and type(meta.files) == "table" and #meta.files > 0 then
        for _, rel in ipairs(meta.files) do
            if type(rel) == "string" then add(rel) end
        end
        return out
    end
    local candidates = {}
    for _, key in ipairs({ "shared_script", "server_script", "client_script" }) do
        for _, rel in ipairs(metadataList(script.folder, key)) do candidates[#candidates + 1] = rel end
    end
    -- A glob in the manifest (config/*.lua) cannot be listed from here; the
    -- manifest text names the files a glob stands for often enough to find them.
    local text = LoadResourceFile(script.folder, "fxmanifest.lua") or ""
    for quoted in text:gmatch("[\"']([%w_%-%./]+%.lua)[\"']") do candidates[#candidates + 1] = quoted end
    for _, rel in ipairs(candidates) do
        if not rel:find("*", 1, true) and not rel:find("^@") and isConfigPath(rel) then add(rel) end
    end
    return out
end

-- ---------------------------------------------------------------------------
-- docs/hub.json, README, help pages
-- ---------------------------------------------------------------------------

local metaCache = {}    -- folder -> { text, meta }
I.metaIndex = setmetatable({}, { __mode = "k" })   -- meta -> { settings, lists } by canonical path
local metaWarned = {}   -- folder .. length -> true

--- The script's docs/hub.json as a table (empty when missing or broken; a
--- broken one is logged once).
function Hub.Meta(folder)
    local text = LoadResourceFile(folder, "docs/hub.json")
    local cached = metaCache[folder]
    if cached and cached.text == text then return cached.meta end
    local meta = {}
    if type(text) == "string" and text:match("%S") then
        local value, err = I.decode((text:gsub("^\239\187\191", "")))
        if type(value) == "table" then
            meta = value
        else
            local key = folder .. "|" .. #text
            if not metaWarned[key] then
                metaWarned[key] = true
                Util.Warn("%s/docs/hub.json could not be read (%s); the hub shows the script without it.",
                    folder, tostring(err or "not a JSON object"))
            end
        end
    end
    if type(meta.settings) ~= "table" then meta.settings = {} end
    if type(meta.lists) ~= "table" then meta.lists = {} end
    -- Lookups by canonical path (I.canon), built once per hub.json. Kept out of
    -- what is sent to the page (they are functions of the same data).
    local settingsCanon, listsCanon = {}, {}
    for k, v in pairs(meta.settings) do
        if type(k) == "string" and k:sub(-2) ~= ".*" then settingsCanon[I.canon(k)] = v end
    end
    for k, v in pairs(meta.lists) do
        if type(k) == "string" then listsCanon[I.canon(k)] = v end
    end
    I.metaIndex[meta] = { settings = settingsCanon, lists = listsCanon }
    metaCache[folder] = { text = text, meta = meta }
    return meta
end

local function safeDocPath(rel)
    return type(rel) == "string" and rel ~= "" and not rel:find("..", 1, true)
        and not rel:find(":", 1, true) and not rel:find("^[/\\]") and #rel <= 200
end

function I.readme(script, meta)
    local rel = safeDocPath(meta.readme) and meta.readme or "README.md"
    return LoadResourceFile(script.folder, rel)
end

function I.helpPages(script, meta)
    local out = {}
    if type(meta.help) ~= "table" then return out end
    for _, h in ipairs(meta.help) do
        if type(h) == "table" and safeDocPath(h.file) then
            local text = LoadResourceFile(script.folder, h.file)
            if text then
                out[#out + 1] = { title = tostring(h.title or I.readable((h.file:match("([^/]+)%.%w+$")) or h.file)), markdown = text }
            end
        end
    end
    return out
end

local iconCache = {}   -- folder -> { at, value }

--- card.icon (§8.8), in order:
---   1. the script's own docs/icon.png, when its manifest lists it in files
---      (else nui:// cannot serve it) and it is there
---   2. the icon poggy_core ships for that poggy_id, ui/hub/img/icons/<id>.png,
---      so older installs without their own icon still show one
---   3. nil: the page draws a monogram tile
function I.icon(script)
    local c = iconCache[script.folder]
    if c and I.now() - c.at < 300 then return c.value end
    local value = nil
    if I.manifestListsFile(script.folder, "docs/icon.png") and LoadResourceFile(script.folder, "docs/icon.png") then
        value = ("nui://%s/docs/icon.png"):format(script.folder)
    elseif type(script.id) == "string" and script.id:find("^[A-Za-z0-9_%-]+$") then
        local rel = ("ui/hub/img/icons/%s.png"):format(script.id)
        if LoadResourceFile(SELF, rel) then value = ("nui://%s/%s"):format(SELF, rel) end
    end
    iconCache[script.folder] = { at = I.now(), value = value }
    return value
end

-- ---------------------------------------------------------------------------
-- Models: each config file parsed once per fingerprint
-- ---------------------------------------------------------------------------

local modelCache = {}   -- folder .. "|" .. file -> { fp, sig, model, err }

-- The kinds hub.json may force. "collection" (§8.4) is a map to the model:
-- its rows are keyed, whatever style the keys are written in.
local FORCIBLE = { list = "list", map = "map", strings = "strings", group = "group", collection = "map" }
local kindsCache = setmetatable({}, { __mode = "k" })   -- meta -> { kinds, sig }

--- The options every SettingsModel call for a file takes: dictKeys for
--- translation files, and the kinds hub.json forces ("kind": "map" on a list
--- entry or a setting), so an edit sees the same kinds the page was shown.
--- These are the base options; Hub.LoadFile adds the kinds it works out
--- itself (§8.4 collections, §8.9 key tables) and every edit then uses the
--- file's own `opts` from LoadFile.
function I.modelOpts(meta, dictKeys)
    meta = type(meta) == "table" and meta or {}
    local c = kindsCache[meta]
    if not c then
        local kinds, parts = {}, {}
        local function take(src)
            for p, m in pairs(type(src) == "table" and src or {}) do
                if type(p) == "string" and p:sub(-2) ~= ".*" and type(m) == "table" and FORCIBLE[m.kind] then
                    kinds[p] = FORCIBLE[m.kind]
                    parts[#parts + 1] = p .. "=" .. FORCIBLE[m.kind]
                end
            end
        end
        take(meta.settings)
        take(meta.lists)
        table.sort(parts)
        c = { kinds = kinds, sig = table.concat(parts, ";") }
        kindsCache[meta] = c
    end
    return { dictKeys = dictKeys and true or false, kinds = c.kinds }, c.sig
end

-- ---------------------------------------------------------------------------
-- Shapes (§8.4, §8.9): what a table's rows look like. Pure data checks, used
-- both on a parsed model (to decide which tables the model should read as
-- maps) and on the built script (collections, key tables).
--
-- Keys and values here are config text, so no %a/%w/%u/%c classes: on Windows
-- those follow the C locale and can match UTF-8 bytes. Explicit ranges only.
-- ---------------------------------------------------------------------------

--- A plain data table (not a vector or a hash).
local function plain(v) return type(v) == "table" and v.__type == nil end
I.plain = plain

--- ASCII-only lower case (string.lower follows the locale).
function I.lower(s)
    return (tostring(s):gsub("[A-Z]", function(c) return string.char(c:byte() + 32) end))
end

--- A key that names a control: G, ENTER, LEFTBRACKET, INPUT_CONTEXT, "1".
local function controlName(k)
    return type(k) == "string" and #k >= 1 and #k <= 64 and k:find("^[A-Z0-9_]+$") ~= nil
end

--- A control hash: a backtick hash, or a whole number the size of a 32-bit
--- hash (0x760A9C6F). Small numbers are ordinary settings (MAX = 5).
local function controlValue(v)
    if type(v) == "table" then return v.__type == "hash" end
    return math.type(v) == "integer" and (v >= 0x10000 or v <= -0x10000)
end

--- The rows of a map's or list's value, in a stable order: { { key, value } }.
--- A list keeps its order; a map's keys sort (numbers as numbers). Keys of a
--- number-keyed map (which travels with text keys and __int_keys, §3.1) come
--- back as integers.
function I.rowsOf(v)
    local out = {}
    if not plain(v) then return out end
    local n, count = #v, 0
    for k in pairs(v) do if k ~= "__int_keys" then count = count + 1 end end
    if n == count then
        for i = 1, n do out[i] = { key = i, value = v[i] } end
        return out
    end
    local ints = v.__int_keys == true
    for k, x in pairs(v) do
        if k ~= "__int_keys" then
            local key = k
            if ints and type(k) == "string" then key = math.tointeger(tonumber(k)) or k end
            out[#out + 1] = { key = key, value = x }
        end
    end
    table.sort(out, function(a, b)
        local ta, tb = type(a.key), type(b.key)
        if ta ~= tb then return ta == "number" end
        return a.key < b.key
    end)
    return out
end

--- What a value inside a row is, as far as collections care:
---   "list"   an array of tables (rows)          "empty"  {}
---   "map"    a number-keyed map of tables, or 2+ named tables
---   nil      anything else (a value, a vector, a list of names, a nested
---            table of settings such as npc = { model = ..., coords = ... })
function I.sublistShape(v)
    if not plain(v) then return nil end
    local count, arr, tables = 0, true, true
    for k, x in pairs(v) do
        if k ~= "__int_keys" then
            count = count + 1
            if math.type(k) ~= "integer" then arr = false end
            if not plain(x) then tables = false end
        end
    end
    if count == 0 then return "empty" end
    if not tables then return nil end
    if arr and count == #v then return "list" end
    if v.__int_keys == true or count >= 2 then return "map" end
    return nil
end

--- Is a map's value a key table (§8.9): control names to control hashes?
function I.isKeyTableValue(v)
    if not plain(v) or v.__int_keys then return false end
    local n = 0
    for k, x in pairs(v) do
        if not controlName(k) or not controlValue(x) then return false end
        n = n + 1
    end
    return n >= 2
end

--- Children of every node by parent path, in file order, for one model.
function I.childrenIndex(nodes, order)
    local kids = {}
    for _, p in ipairs(order) do
        local n = nodes[p]
        if n and n.parent then
            local list = kids[n.parent]
            if not list then list = {}; kids[n.parent] = list end
            list[#list + 1] = n
        end
    end
    return kids
end

--- A group whose children are all tables of settings (groups), at least half
--- of which hold a list or map: a collection written with named keys.
--- Returns rows, hits (rows holding a list) or nil.
function I.groupRows(node, kids)
    local rows = kids[node.path]
    if not rows or #rows == 0 then return nil end
    local hits = 0
    for _, r in ipairs(rows) do
        if r.kind ~= "group" then return nil end
        for _, c in ipairs(kids[r.path] or {}) do
            if c.kind == "list" or c.kind == "map" then hits = hits + 1 break end
        end
    end
    return rows, hits
end

--- A group whose children are all control hashes (a key table written with
--- plain names: Keys = { G = 0x760A9C6F, ... }).
function I.isKeyTableGroup(node, kids)
    local rows = kids[node.path]
    if not rows or #rows < 2 then return false end
    for _, r in ipairs(rows) do
        if r.kind ~= "value" or not controlName(r.key) or not controlValue(r.value) then return false end
    end
    return true
end

--- Paths hub.json marks "kind": "collection" (canonical), per hub.json.
local collForcedCache = setmetatable({}, { __mode = "k" })
function I.collectionPaths(meta)
    meta = type(meta) == "table" and meta or {}
    local c = collForcedCache[meta]
    if c then return c end
    c = {}
    for _, src in ipairs({ meta.settings, meta.lists }) do
        for p, m in pairs(type(src) == "table" and src or {}) do
            if type(p) == "string" and p:sub(-2) ~= ".*" and type(m) == "table" and m.kind == "collection" then
                c[I.canon(p)] = true
            end
        end
    end
    collForcedCache[meta] = c
    return c
end

--- The tables the model should read as maps although it sees named keys
--- (groups): collections (§8.4) and key tables (§8.9) written as
--- `X = { a = { ... }, b = { ... } }`. Only groups hub.json says nothing about
--- (an explicit "kind" wins), and never a file's root. { path = "map" }.
function I.autoKinds(model, meta, baseKinds)
    local out = {}
    if type(model) ~= "table" or type(model.nodes) ~= "table" then return out end
    local explicit = {}
    for p in pairs(baseKinds or {}) do explicit[I.canon(p)] = true end
    local kids = I.childrenIndex(model.nodes, model.order or {})
    local skip = {}
    for _, path in ipairs(model.order or {}) do
        local n = model.nodes[path]
        if n and n.kind == "group" and n.parent and not skip[n.parent] then
            local c = I.canon(path)
            if not explicit[c] then
                local rows, hits = I.groupRows(n, kids)
                if (rows and hits >= 1 and hits * 2 >= #rows) or I.isKeyTableGroup(n, kids) then
                    out[path] = "map"
                    skip[path] = true
                end
            end
        end
        if n and skip[n.parent] then skip[path] = true end
    end
    return out
end

--- Read and parse one config file. Returns { file, text, fingerprint, lines,
--- model, err, opts }. text is nil when the file is gone. meta: the script's
--- hub.json (for forced kinds); optional.
---
--- opts are the options every edit of this file must use: hub.json's kinds,
--- plus the groups this file turns out to hold as collections or key tables
--- (I.autoKinds), read as maps. A kind the model cannot honour (a group
--- filled in by separate `Config.X.y = ...` lines, or with code in a row)
--- is dropped again, hub.json's own included: the table is then shown as the
--- group it is, and a collection of it is kept by its nodes (§8.4).
function Hub.LoadFile(folder, rel, dictKeys, meta)
    local out = { file = rel }
    local text = LoadResourceFile(folder, rel)
    if not text then
        out.err = "the file could not be read"
        return out
    end
    local Model = M()
    out.text = text
    local _, newlines = text:gsub("\n", "")
    out.lines = newlines + ((text ~= "" and text:sub(-1) ~= "\n") and 1 or 0)
    if not Model then
        out.err = "the settings model (sv_settings_model.lua) is not loaded"
        return out
    end
    out.fingerprint = Model.Fingerprint(text)
    local base, sig = I.modelOpts(meta, dictKeys)
    local key = folder .. "|" .. rel
    local cached = modelCache[key]
    if cached and cached.fp == out.fingerprint and cached.sig == sig then
        out.model, out.err, out.opts = cached.model, cached.err, cached.opts
        return out
    end
    local model, err, opts = I.parseWithKinds(Model, text, base, meta)
    if not model then
        Util.Warn("%s/%s could not be read as settings: %s", folder, rel, tostring(err))
    end
    modelCache[key] = { fp = out.fingerprint, sig = sig, model = model, opts = opts,
        err = model == nil and tostring(err) or nil }
    out.model, out.err, out.opts = model, modelCache[key].err, opts
    return out
end

--- Parse a config text with hub.json's kinds, then with the kinds the file
--- itself calls for (I.autoKinds). Returns model, err, opts: the options the
--- model was finally read with, which every edit of the file must use.
---
--- A forced "map" the model cannot honour comes back read-only (a group
--- filled in by separate lines, a row with code in it). Such a kind is
--- dropped and the file read again, so the table stays the group it is. A
--- group that is a collection (§8.4) and is filled in by separate lines
---     Config.Catalog = {}
---     Config.Catalog.GeneralStore = { sell = { ... }, buy = { ... } }
--- is named in opts.statementRows: its rows are those lines, and the model
--- adds, removes and renames them as whole statements.
function I.parseWithKinds(Model, text, base, meta)
    local function parse(o)
        local ok, m, e = pcall(Model.Parse, text, o)
        if not ok then return nil, tostring(m) end
        return m, e
    end
    local model, err = parse(base)
    if not model then return nil, err, base end

    local kinds = {}
    for p, k in pairs(base.kinds or {}) do kinds[p] = k end
    local auto = I.autoKinds(model, meta, base.kinds)
    local added = false
    for p, k in pairs(auto) do kinds[p] = k; added = true end
    local opts = { dictKeys = base.dictKeys, kinds = kinds }
    if added then
        local m2 = parse(opts)
        if m2 then model = m2 end
    end
    -- A forced map (hub.json's or ours) that came back read-only or as
    -- something else is dropped, and the file read again.
    for _ = 1, 4 do
        local failed = false
        for p, k in pairs(kinds) do
            local n = model.nodes[p] or I.nodeAt(model.nodes, p)
            if k == "map" and n and n.kind ~= "map" then
                kinds[p] = nil
                failed = true
            end
        end
        if not failed then break end
        local m3 = parse(opts)
        if not m3 then break end
        model = m3
    end

    -- Collections that stayed groups: filled in by separate lines, or with
    -- rows the model reads as groups of settings.
    local forced = I.collectionPaths(meta)
    local kids = I.childrenIndex(model.nodes, model.order or {})
    for _, path in ipairs(model.order or {}) do
        local n = model.nodes[path]
        if n and n.kind == "group" and n.parent then
            local c = I.canon(path)
            local rows, hits = I.groupRows(n, kids)
            local explicitOff = false
            for p, k in pairs(base.kinds or {}) do
                if I.canon(p) == c and k ~= "map" then explicitOff = true end
            end
            if not explicitOff and (forced[c] or (rows and hits >= 1 and hits * 2 >= #rows)) then
                opts.statementRows = opts.statementRows or {}
                opts.statementRows[path] = true
            end
        end
    end
    return model, nil, opts
end

--- Forget cached models of a folder (after a write).
function I.forgetModels(folder)
    for key in pairs(modelCache) do
        if key:sub(1, #folder + 1) == folder .. "|" then modelCache[key] = nil end
    end
    I.forgetBuilt(folder)
end

-- ---------------------------------------------------------------------------
-- hub.json metadata onto settings (§5)
-- ---------------------------------------------------------------------------

--- The ".*" prefix rules of a hub.json, longest last: { { prefix, meta } }.
local function prefixRules(meta)
    local rules = {}
    for key, m in pairs(meta.settings or {}) do
        if type(key) == "string" and key:sub(-2) == ".*" and type(m) == "table" then
            rules[#rules + 1] = { prefix = I.canon(key:sub(1, -3)), meta = m }
        end
    end
    table.sort(rules, function(a, b) return #a.prefix < #b.prefix end)
    return rules
end

local function under(path, prefix)
    if #path <= #prefix or path:sub(1, #prefix) ~= prefix then return false end
    local c = path:sub(#prefix + 1, #prefix + 1)
    return c == "." or c == "["
end

I.under = under

local INHERITED = { "tab", "live", "advanced", "hidden", "readonly" }

--- The merged meta of one node: prefix rules (shorter first), the list entry
--- for a list or map, the exact entry; then what is inherited from the parent,
--- then labels and tooltips filled in from the key and the config comment.
---
--- Returns m, plus `own`: the keys hub.json itself set for this node (by a
--- prefix rule, a list entry or an exact entry), before anything was inherited
--- or filled in. The organisation (§8) needs to know which labels, groups and
--- tabs the author chose and which were made up. `tab` is not defaulted here:
--- a node hub.json gives no tab (not even through a parent) is placed by the
--- fallback rules of §8.2 once the whole script is read.
local function resolveMeta(meta, rules, node, parentMeta, parentNode)
    local m, own = {}, {}
    local canon = I.canon(node.path)
    local index = I.metaIndex[meta] or { settings = meta.settings, lists = meta.lists }
    for _, r in ipairs(rules) do
        if under(canon, r.prefix) then
            for k, v in pairs(r.meta) do m[k] = v; own[k] = true end
        end
    end
    local list = index.lists[canon]
    if type(list) == "table" then
        for k, v in pairs(list) do m[k] = v; own[k] = true end
        m.isList = true
    end
    local exact = index.settings[canon]
    if type(exact) == "table" then
        for k, v in pairs(exact) do m[k] = v; own[k] = true end
    end
    if parentMeta then
        for _, k in ipairs(INHERITED) do
            if m[k] == nil and parentMeta[k] ~= nil then m[k] = parentMeta[k] end
        end
        -- A group's children sit under a heading named after the nearest group.
        if m.group == nil then
            if parentNode and parentNode.kind == "group" then
                m.group = parentMeta.label
            else
                m.group = parentMeta.group
            end
        end
    end
    if m.label == nil then m.label = I.readable(node.key) end
    if m.tooltip == nil and type(node.comment) == "string" and node.comment ~= "" then m.tooltip = node.comment end
    return m, own
end

-- ---------------------------------------------------------------------------
-- A script, built: every file's nodes with meta, defaults and changed flags
-- ---------------------------------------------------------------------------

local built = {}   -- folder -> { key, value }

function I.forgetBuilt(folder)
    if folder then built[folder] = nil else built = {} end
end

--- Copy of a model node with the hub's fields added.
local function outNode(node, file, m, default, hasDefault)
    local n = {
        path = node.path, parent = node.parent, key = node.key, kind = node.kind, type = node.type,
        value = node.value, line = node.line, comment = node.comment, role = node.role,
        editable = node.editable ~= false, reason = node.reason, source = node.source,
        file = file, meta = m,
        -- A code cell inside a list or map row (name = T('X')): read-only,
        -- shown with its source in that row (§3 readonly, cell level).
        cell = node.cell or nil,
    }
    if m.readonly and n.editable then
        n.editable = false
        n.reason = "read-only here: edit it in the file"
    end
    -- Forced kinds (hub.json lists: kind = "list" | "map") reach the model
    -- through I.modelOpts, so the model's kind is already the forced one when
    -- it could honour it. Between list and map the page follows hub.json; a
    -- group stays a group (the model could not read it as rows).
    if (m.kind == "list" or m.kind == "map") and (node.kind == "list" or node.kind == "map") then
        n.kind = m.kind
    end
    if hasDefault then
        n.default = default
        n.changed = not I.equal(node.value, default)
    else
        n.changed = false
    end
    return n
end

--- Build a script's settings. Returns
---   { script, meta, files = { { file, fingerprint, lines, err } }, nodes = { node... },
---     byPath = { path = node }, order = { path... }, tabs, defaultsKnown,
---     settingsCount, changedCount, loaded = { file = LoadFile result } }
function Hub.Build(script)
    local meta = Hub.Meta(script.folder)
    local files = Hub.ConfigFiles(script, meta)
    local loaded, keyParts = {}, {}
    for _, f in ipairs(files) do
        local l = Hub.LoadFile(script.folder, f.file, f.dictKeys, meta)
        l.dictKeys = f.dictKeys
        loaded[#loaded + 1] = l
        keyParts[#keyParts + 1] = f.file .. "=" .. tostring(l.fingerprint)
    end
    local defaults = Hub.DefaultsFor and Hub.DefaultsFor(script) or nil
    local metaText = metaCache[script.folder] and metaCache[script.folder].text or ""
    local key = table.concat(keyParts, ";") .. "|" .. tostring(metaText and #metaText) .. "|"
        .. tostring(defaults and defaults.stamp or "none") .. "|" .. tostring(script.version)
    local cached = built[script.folder]
    if cached and cached.key == key then return cached.value end

    local rules = prefixRules(meta)
    local out = {
        script = script, meta = meta, files = {}, nodes = {}, byPath = {}, order = {},
        defaultsKnown = defaults ~= nil and defaults.files ~= nil,
        settingsCount = 0, changedCount = 0, loaded = {},
    }
    -- What the organisation (§8) needs and the page does not: per file, its
    -- nodes and lookups; per node, the keys hub.json set for it itself.
    local priv = { files = {}, own = {} }

    for _, l in ipairs(loaded) do
        out.loaded[l.file] = l
        out.files[#out.files + 1] = { file = l.file, fingerprint = l.fingerprint, lines = l.lines, err = l.err }
        local model = l.model
        if model then
            local F = { file = l.file, dictKeys = l.dictKeys, opts = l.opts or {}, nodes = {}, at = {} }
            priv.files[#priv.files + 1] = F
            local shipped = nil
            if out.defaultsKnown then
                shipped = Hub.ShippedModel and Hub.ShippedModel(script, l.file, l.dictKeys, l.opts) or nil
            end
            local metas = {}
            for _, path in ipairs(model.order or {}) do
                local node = model.nodes[path]
                if node then
                    local parentNode = node.parent and model.nodes[node.parent] or nil
                    local m, own = resolveMeta(meta, rules, node, parentNode and metas[node.parent] or nil, parentNode)
                    metas[path] = m
                    local default, hasDefault = nil, false
                    if shipped then
                        local sn = shipped.nodes[path]
                        if sn and node.kind ~= "group" then default, hasDefault = sn.value, true end
                    end
                    local n = outNode(node, l.file, m, default, hasDefault)
                    out.nodes[#out.nodes + 1] = n
                    out.order[#out.order + 1] = path
                    if not out.byPath[path] then out.byPath[path] = n end
                    F.nodes[#F.nodes + 1] = n
                    F.at[path] = n
                    priv.own[n] = own
                end
            end
        end
    end

    -- Tabs, sections, items, collections, labels, references (§8).
    I.organise(out, priv)

    for _, n in ipairs(out.nodes) do
        if n.kind ~= "group" and not n.meta.hidden and not n.cell then
            out.settingsCount = out.settingsCount + 1
            if n.changed then out.changedCount = out.changedCount + 1 end
        end
    end

    built[script.folder] = { key = key, value = out }
    return out
end

-- ---------------------------------------------------------------------------
-- Navigation and organisation (§8)
--
-- The first test on a real server (older script versions, no hub.json) put
-- 238 settings under "General" and twenty lists called "Sell" and "Buy" in the
-- rail. So the server works out the structure itself, with or without
-- hub.json, and the page renders what it is sent:
--
--   tabs      hub.json's; else one per config file (several files) or one per
--             big top-level group (one file); translations always in
--             "Text & language"                                          §8.2
--   sections  the setting groups on a tab page, in file order           §8.1
--   items     the lists, maps and collections under a tab               §8.1
--   labels    a list never shows a bare generic name ("Sell")           §8.3
--   collections  a map or list whose rows hold lists (catalogs with sell
--             and buy): ONE item, its inner lists are its sublists      §8.4
--   refs      settings and fields that name a key of another table     §8.5
--   key tables  control name -> control hash maps, shown in two columns §8.9
--
-- Every node gets `tab`, and `section` when it is a setting shown on a tab
-- page, or `item` (+ `row`) when it lives inside a collection kept as nodes.
-- ---------------------------------------------------------------------------

local GENERIC = { sell = true, buy = true, items = true, list = true, entries = true, data = true,
    rows = true, options = true, values = true, loot = true, rewards = true }

local SETTING = { value = true, strings = true, readonly = true }

--- A key or table name that holds translations.
local function translationKey(k)
    if type(k) ~= "string" then return false end
    local l = I.lower(k)
    return l:find("^lang") ~= nil or l:find("^locale") ~= nil or l:find("^translation") ~= nil
        or l:find("^i18n") ~= nil or l == "text" or l == "texts" or l == "strings"
end

local function squash(s) return (I.lower(tostring(s)):gsub("[^a-z0-9]", "")) end

local function slug(s)
    local out = I.lower(tostring(s)):gsub("[^a-z0-9]+", "_"):gsub("^_+", ""):gsub("_+$", "")
    return out ~= "" and out or "tab"
end

--- The path of one key below a path, spelled as I.canon spells it.
function I.child(base, key)
    if math.type(key) == "integer" then return ("%s[%d]"):format(base, key) end
    key = tostring(key)
    if key:find("^[A-Za-z_][A-Za-z0-9_]*$") then return base .. "." .. key end
    return base .. ("[%q]"):format(key)
end

--- One key of a table as data holds it (int keys of a number-keyed map travel as text).
function I.keyed(t, key)
    if not plain(t) then return nil end
    local v = t[key]
    if v == nil and math.type(key) == "integer" then v = t[tostring(key)] end
    if v == nil and type(key) == "string" and math.tointeger(tonumber(key)) then v = t[math.tointeger(tonumber(key))] end
    return v
end

local TITLE_FIELDS = { "name", "label", "title", "id", "key", "type" }

--- The words a row is known by: hub.json's titleField, else a name, label,
--- title, id, key or type field. With bare, nil when the row has none of
--- them; otherwise its key ("catalog 3" for a list row).
function I.rowTitle(meta, row, key, bare)
    if plain(row) then
        local tf = type(meta) == "table" and meta.titleField or nil
        local v = type(tf) == "string" and row[tf] or nil
        if (type(v) == "string" and v ~= "") or type(v) == "number" then return tostring(v) end
        for _, f in ipairs(TITLE_FIELDS) do
            v = row[f]
            if type(v) == "string" and v ~= "" then return v end
        end
    end
    if bare then return nil end
    if type(key) == "string" then return key end
    local what = type(meta) == "table" and type(meta.itemLabel) == "string" and meta.itemLabel or "row"
    return ("%s %s"):format((what:gsub("^[a-z]", string.upper)), tostring(key))
end

-- ---------------------------------------------------------------------------
-- List field keys: "loot[].count" is the count of every loot row. Shared with
-- sv_hub_save.lua, which checks edits against them.
-- ---------------------------------------------------------------------------

--- A list field key as segments: "loot[].count" -> { "loot", "[]", "count" }.
function I.fieldSegs(key)
    local segs = {}
    key = tostring(key):gsub("^%.", "")
    local i, n = 1, #key
    while i <= n do
        local c = key:sub(i, i)
        if c == "." then
            i = i + 1
        elseif c == "[" then
            local close = key:find("]", i, true) or n
            local inner = key:sub(i + 1, close - 1)
            if inner == "" then
                segs[#segs + 1] = "[]"
            else
                segs[#segs + 1] = tostring(inner:match('^["\'](.*)["\']$') or inner)
            end
            i = close + 1
        else
            local word = key:match("^[^%.%[]+", i)
            segs[#segs + 1] = word
            i = i + #word
        end
    end
    return segs
end

--- Segments back to a field key: { "loot", "[]", "count" } -> "loot[].count".
function I.fieldKeyOf(segs)
    local out = ""
    for _, s in ipairs(segs) do
        if s == "[]" then out = out .. "[]" elseif out == "" then out = s else out = out .. "." .. s end
    end
    return out
end

local fieldCache = setmetatable({}, { __mode = "k" })   -- fields -> { { segs, meta } }

--- The field meta for a row-relative position (segments). "[]" in a field
--- key matches any one segment: every element of an array, or every value of
--- a map (int-keyed or not). An exact key wins over one with "[]".
function I.fieldMeta(fields, segs)
    if type(fields) ~= "table" or #segs == 0 then return nil end
    local parsed = fieldCache[fields]
    if not parsed then
        parsed = {}
        for k, m in pairs(fields) do
            if type(k) == "string" and type(m) == "table" then parsed[#parsed + 1] = { segs = I.fieldSegs(k), meta = m } end
        end
        fieldCache[fields] = parsed
    end
    local best, bestWild = nil, math.huge
    for _, f in ipairs(parsed) do
        if #f.segs == #segs then
            local wild, match = 0, true
            for i2 = 1, #segs do
                local want = f.segs[i2]
                if want == "[]" then
                    wild = wild + 1
                elseif want ~= tostring(segs[i2]) then
                    match = false
                    break
                end
            end
            if match and wild < bestWild then best, bestWild = f.meta, wild end
        end
    end
    return best
end

--- Fill m from extra where hub.json did not set that key for the node
--- itself. readonly and hidden only ever tighten: true from anywhere wins.
local function mergeInto(m, own, extra)
    if not plain(extra) then return end
    for k, v in pairs(extra) do
        if k == "readonly" or k == "hidden" then
            if v == true then m[k] = true end
        elseif not own[k] then
            m[k] = v
        end
    end
end

--- A node whose meta turned read-only after it was built.
local function lockIfReadonly(n)
    if n.meta.readonly and n.editable then
        n.editable = false
        n.reason = "read-only here: edit it in the file"
    end
end

--- Rows of a value that hold at least one list: rows, hits.
local function valueCollectionHits(value)
    local rows = I.rowsOf(value)
    local hits = 0
    for _, r in ipairs(rows) do
        if plain(r.value) then
            for _, x in pairs(r.value) do
                local s = I.sublistShape(x)
                if s == "list" or s == "map" then hits = hits + 1 break end
            end
        end
    end
    return rows, hits
end

--- The field keys of a set of rows, in a stable order: hub.json's columns
--- first, then the order they were first seen in the file (when known), then
--- the rest sorted.
local function orderedFields(names, meta, firstSeen)
    local out, seen = {}, {}
    if type(meta) == "table" and type(meta.columns) == "table" then
        for _, c in ipairs(meta.columns) do
            local f = type(c) == "table" and c.field or c
            if type(f) == "string" and names[f] and not seen[f] then seen[f] = true; out[#out + 1] = f end
        end
    end
    for _, f in ipairs(firstSeen or {}) do
        if names[f] and not seen[f] then seen[f] = true; out[#out + 1] = f end
    end
    local rest = {}
    for f in pairs(names) do if not seen[f] then rest[#rest + 1] = f end end
    table.sort(rest, function(a, b) return tostring(a) < tostring(b) end)
    for _, f in ipairs(rest) do out[#out + 1] = f end
    return out
end

--- The sublists hub.json names for a collection: { field = meta }.
local function hubSublists(m)
    return plain(m.sublists) and m.sublists or {}
end

--- A collection's field meta widened with every sublist's: the sublist "sell"
--- with field "price" is the row field "sell[].price". The save path checks
--- edits inside a row against these (readonly and hidden included), exactly
--- as it checks a list's own fields. Returns a new table; hub.json's stays.
function I.widenFields(m)
    local fields = {}
    for k, v in pairs(plain(m.fields) and m.fields or {}) do fields[k] = v end
    for field, sm in pairs(plain(m.sublists) and m.sublists or {}) do
        if type(field) == "string" and plain(sm) then
            if fields[field] == nil then
                fields[field] = { label = sm.label, tooltip = sm.tooltip, readonly = sm.readonly,
                    hidden = sm.hidden, advanced = sm.advanced }
            elseif sm.readonly == true or sm.hidden == true then
                local copy = I.copy(fields[field])
                if sm.readonly == true then copy.readonly = true end
                if sm.hidden == true then copy.hidden = true end
                fields[field] = copy
            end
            for fk, fm in pairs(plain(sm.fields) and sm.fields or {}) do
                if type(fk) == "string" then
                    local key = field .. "[]" .. (fk:sub(1, 1) == "[" and "" or ".") .. fk
                    if fields[key] == nil then fields[key] = fm end
                end
            end
        end
    end
    return fields
end

--- The descriptor of one sublist.
local function sublistEntry(field, kind, sm)
    return {
        field = field,
        label = plain(sm) and type(sm.label) == "string" and sm.label or I.readable(field),
        kind = kind,
        meta = plain(sm) and sm or nil,
    }
end

--- How many rows a table value has.
local function countRowsOf(v)
    if not plain(v) then return 0 end
    local n = 0
    for k in pairs(v) do if k ~= "__int_keys" then n = n + 1 end end
    return n
end

--- A collection kept in one value (a map or list whose rows hold lists):
--- the rows, their sublist counts, and the field meta the save path checks
--- edits inside the rows against.
local function makeValueCollection(n)
    local m = n.meta
    local subsMeta = hubSublists(m)
    local rows = I.rowsOf(n.value)
    local kinds, scalars = {}, {}
    for _, r in ipairs(rows) do
        if plain(r.value) then
            for k, x in pairs(r.value) do
                local s = I.sublistShape(x)
                if (s == "list" or s == "map") and type(k) == "string" then kinds[k] = kinds[k] or s end
            end
        end
    end
    for k, sm in pairs(subsMeta) do
        if type(k) == "string" and not kinds[k] then
            kinds[k] = (plain(sm) and (sm.kind == "map" or sm.kind == "list") and sm.kind) or "list"
        end
    end
    for _, r in ipairs(rows) do
        if plain(r.value) then
            for k in pairs(r.value) do
                if type(k) == "string" and not kinds[k] and k ~= "__int_keys" then scalars[k] = true end
            end
        end
    end
    local sublists = {}
    for _, f in ipairs(orderedFields(kinds, nil)) do sublists[#sublists + 1] = sublistEntry(f, kinds[f], subsMeta[f]) end
    local scalarFields = {}
    for _, f in ipairs(orderedFields(scalars, m)) do
        local fm = I.fieldMeta(m.fields, { f })
        scalarFields[#scalarFields + 1] = { field = f, label = fm and type(fm.label) == "string" and fm.label or I.readable(f), meta = fm }
    end

    m.fields = I.widenFields(m)

    local defaultRows = n.default
    local outRows = {}
    for _, r in ipairs(rows) do
        local counts = {}
        for _, s in ipairs(sublists) do counts[s.field] = countRowsOf(plain(r.value) and r.value[s.field] or nil) end
        local changed = false
        if n.default ~= nil then changed = not I.equal(r.value, I.keyed(defaultRows, r.key)) end
        outRows[#outRows + 1] = {
            key = r.key, path = I.child(n.path, r.key),
            title = I.rowTitle(m, r.value, r.key, true), counts = counts, changed = changed, usedBy = 0,
        }
    end
    local isList = n.kind == "list"
    local ok = n.editable ~= false
    local template = nil
    if m.template == nil then
        template = {}
        for _, s in ipairs(sublists) do template[s.field] = {} end
    end
    n.collection = {
        sublists = sublists, scalarFields = scalarFields, rows = outRows, rowSource = "value",
        rowOps = { add = ok, remove = ok, duplicate = ok, move = ok and isList, rename = ok and not isList },
        template = template,
    }
end

--- The encoded value of a group of settings, from its nodes. A part that is
--- code (a read-only node) comes back as { __type = "code", source, reason },
--- as code cells of a list do, and makes the second result true.
local function valueFromNodes(n, kidsOf, depth)
    if n.kind ~= "group" then
        if n.kind == "readonly" then
            return { __type = "code", source = n.source, reason = n.reason }, true
        end
        return n.value, false
    end
    depth = (depth or 0) + 1
    if depth > 16 then return {}, true end
    local out, code = {}, false
    for _, c in ipairs(kidsOf(n)) do
        local v, cc = valueFromNodes(c, kidsOf, depth)
        if cc then code = true end
        if v ~= nil then out[c.key] = v end
    end
    return out, code
end

--- A collection kept as nodes: a group of groups (rows) that hold lists,
--- read by the model as settings because its keys are names, or because it
--- is filled in by separate lines (Config.Catalog.GeneralStore = {...}).
--- The node becomes a map on the page, with its rows' value put together
--- from their nodes; every row field is still a node of its own, and hub.json's
--- sublist and field meta are merged onto those nodes (the save path checks
--- edits against the nodes' meta, so readonly and hidden hold here too).
local function makeNodeCollection(n, kidsOf, own, statementRows)
    local m = n.meta
    local subsMeta = hubSublists(m)
    local rows = kidsOf(n)
    local kinds, seenOrder = {}, {}
    for _, r in ipairs(rows) do
        for _, c in ipairs(kidsOf(r)) do
            if type(c.key) == "string" then seenOrder[#seenOrder + 1] = c.key end
            if (c.kind == "list" or c.kind == "map") and type(c.key) == "string" then kinds[c.key] = kinds[c.key] or c.kind end
        end
    end
    for k, sm in pairs(subsMeta) do
        if type(k) == "string" and not kinds[k] then
            kinds[k] = (plain(sm) and (sm.kind == "map" or sm.kind == "list") and sm.kind) or "list"
        end
    end
    local scalars = {}
    local sublists = {}
    for _, f in ipairs(orderedFields(kinds, nil, seenOrder)) do sublists[#sublists + 1] = sublistEntry(f, kinds[f], subsMeta[f]) end
    local subBy = {}
    for _, s in ipairs(sublists) do subBy[s.field] = s end

    local value, outRows, anyChanged = {}, {}, false
    for _, r in ipairs(rows) do
        local counts = {}
        for _, s in ipairs(sublists) do counts[s.field] = 0 end
        local rowChanged = false
        -- Every node inside the row: merge the collection's meta onto it.
        local function visit(node, segs)
            local sub = #segs == 1 and subBy[segs[1]] or nil
            if sub then
                -- {} where other rows have a list is an empty list, not a list of names.
                if node.kind == "strings" and plain(node.value) and next(node.value) == nil then
                    node.kind = "list"
                    node.meta.kind = "list"
                end
                mergeInto(node.meta, own(node), sub.meta)
                if not own(node).label then
                    node.meta.label = I.readable(r.key) .. " · " .. sub.label
                end
                counts[sub.field] = countRowsOf(node.value)
            elseif SETTING[node.kind] then
                scalars[segs[1]] = true
            end
            mergeInto(node.meta, own(node), I.fieldMeta(m.fields, segs))
            lockIfReadonly(node)
            if node.changed then rowChanged = true end
            if node.kind == "group" then
                for _, c in ipairs(kidsOf(node)) do
                    local s2 = {}
                    for i2, x in ipairs(segs) do s2[i2] = x end
                    s2[#s2 + 1] = tostring(c.key)
                    visit(c, s2)
                end
            end
        end
        for _, c in ipairs(kidsOf(r)) do visit(c, { tostring(c.key) }) end
        local rv, code = valueFromNodes(r, kidsOf)
        value[r.key] = rv
        if rowChanged then anyChanged = true end
        outRows[#outRows + 1] = {
            key = r.key, path = r.path, title = I.rowTitle(m, rv, r.key, true),
            counts = counts, changed = rowChanged, usedBy = 0, hasCode = code or nil,
        }
    end
    local scalarFields = {}
    for _, f in ipairs(orderedFields(scalars, m, seenOrder)) do
        local fm = I.fieldMeta(m.fields, { f })
        scalarFields[#scalarFields + 1] = { field = f, label = fm and type(fm.label) == "string" and fm.label or I.readable(f), meta = fm }
    end
    local template = nil
    if m.template == nil then
        template = {}
        for _, s in ipairs(sublists) do template[s.field] = {} end
    end
    local ok = n.editable ~= false and statementRows and true or false
    n.kind, n.type, n.value = "map", "table", value
    n.changed = anyChanged
    n.collection = {
        sublists = sublists, scalarFields = scalarFields, rows = outRows, rowSource = "nodes",
        rowOps = { add = ok, remove = ok, duplicate = ok, rename = ok, move = false },
        rowOpsReason = not ok and "The rows of this table are written as settings in the file; add, remove or rename them there." or nil,
        template = template,
    }
end

--- Every node of a script placed: tab, section, item. Builds b.nav, b.tabs
--- and b.refs. priv: { files = { { file, dictKeys, opts, nodes, at } }, own = { node -> keys } }.
function I.organise(b, priv)
    local meta = b.meta
    local F = priv.files
    local ownOf = priv.own
    local fileOf = {}
    for _, f in ipairs(F) do
        f.kids = {}
        for _, n in ipairs(f.nodes) do
            fileOf[n] = f
            if n.parent then
                local l = f.kids[n.parent]
                if not l then l = {}; f.kids[n.parent] = l end
                l[#l + 1] = n
            end
        end
    end
    local EMPTY = {}
    local function own(n) return ownOf[n] or EMPTY end
    local function parentOf(n) return n.parent and fileOf[n].at[n.parent] or nil end
    local function kidsOf(n) return fileOf[n].kids[n.path] or EMPTY end

    -- 1. Collections (§8.4) and key tables (§8.9).
    local inside, rowOf, holds = {}, {}, {}
    for _, f in ipairs(F) do
        for _, n in ipairs(f.nodes) do
            local p = parentOf(n)
            if n.cell and p then
                -- A code cell of a list or map: part of that table's rows.
                inside[n] = inside[p] or p
                rowOf[n] = inside[p] and rowOf[p] or n.row
            elseif p and (holds[p] or inside[p]) then
                inside[n] = holds[p] and p or inside[p]
                rowOf[n] = holds[p] and n.key or rowOf[p]
            else
                local m = n.meta
                local explicit = own(n).kind and m.kind or nil
                if (n.kind == "list" or n.kind == "map") and plain(n.value) then
                    local rows, hits = valueCollectionHits(n.value)
                    if m.kind == "collection" or (explicit == nil and hits >= 1 and hits * 2 >= #rows) then
                        makeValueCollection(n)
                    elseif n.kind == "map" and (m.display == "keytable"
                        or (m.display == nil and I.isKeyTableValue(n.value))) then
                        n.display = "keytable"
                    end
                elseif n.kind == "group" and n.parent then
                    local stmt = f.opts.statementRows and f.opts.statementRows[n.path] or false
                    local rows, hits = I.groupRows(n, f.kids)
                    if stmt or m.kind == "collection"
                        or (explicit == nil and rows and hits >= 1 and hits * 2 >= #rows) then
                        makeNodeCollection(n, kidsOf, own, stmt)
                        holds[n] = true
                    end
                end
            end
        end
    end

    -- 2. Labels that never repeat (§8.3), for every list, map and collection
    -- the rail shows. Sublists inside a collection were named with their row.
    local itemsAll = {}
    for _, f in ipairs(F) do
        for _, n in ipairs(f.nodes) do
            if (n.kind == "list" or n.kind == "map") and not inside[n] then itemsAll[#itemsAll + 1] = n end
        end
    end
    local function countLabels()
        local c = {}
        for _, n in ipairs(itemsAll) do c[n.meta.label] = (c[n.meta.label] or 0) + 1 end
        return c
    end
    local counts = countLabels()
    for _, n in ipairs(itemsAll) do
        if not own(n).label and (GENERIC[I.lower(tostring(n.key))] or counts[n.meta.label] > 1) then
            local p = parentOf(n)
            local prefix = p and I.readable(p.key) or I.readable((n.file:match("([^/]+)%.[Ll][Uu][Aa]$")) or n.file)
            n.meta.label = prefix .. " · " .. n.meta.label
        end
    end
    counts = countLabels()
    for _, n in ipairs(itemsAll) do
        if not own(n).label and counts[n.meta.label] > 1 then
            n.meta.label = ("%s (%s)"):format(n.meta.label, n.file)
        end
    end
    counts = countLabels()
    local seenLabel = {}
    for _, n in ipairs(itemsAll) do
        if not own(n).label and counts[n.meta.label] > 1 then
            local base = n.meta.label
            seenLabel[base] = (seenLabel[base] or 0) + 1
            if seenLabel[base] > 1 then n.meta.label = ("%s %d"):format(base, seenLabel[base]) end
        end
    end

    -- 3. Tabs (§8.2).
    local declared, declaredOrder = {}, {}
    if type(meta.tabs) == "table" then
        for _, t in ipairs(meta.tabs) do
            if type(t) == "table" and type(t.id) == "string" and t.id ~= "" and not declared[t.id] then
                declared[t.id] = { label = type(t.label) == "string" and t.label or I.readable(t.id), icon = t.icon }
                declaredOrder[#declaredOrder + 1] = t.id
            end
        end
    end
    local textId = "text"
    for _, id in ipairs(declaredOrder) do
        if declared[id].label == "Text & language" then textId = id break end
    end
    local usedIds = { general = true, advanced = true }
    usedIds[textId] = true
    for _, id in ipairs(declaredOrder) do usedIds[id] = true end
    local function uniqueId(base)
        local id, i = base, 2
        while usedIds[id] do id = base .. "_" .. i; i = i + 1 end
        usedIds[id] = true
        return id
    end

    local tabs, seq = {}, 0
    local function useTab(id, label, icon)
        local t = tabs[id]
        if not t then
            seq = seq + 1
            local d = declared[id]
            if id == "general" then label, icon = label or "General", icon or "sliders" end
            if id == textId then label, icon = label or "Text & language", icon or "language" end
            t = { id = id, label = d and d.label or label or I.readable(id), icon = d and d.icon or icon, first = seq }
            tabs[id] = t
        end
        return id
    end

    -- Settings and lists below each node, for rule 3.
    local below = {}
    for _, f in ipairs(F) do
        for i = #f.nodes, 1, -1 do
            local n = f.nodes[i]
            local c = below[n]
            if not c then c = { s = 0, l = 0 }; below[n] = c end
            local p = parentOf(n)
            if p then
                local pc = below[p]
                if not pc then pc = { s = 0, l = 0 }; below[p] = pc end
                pc.s = pc.s + c.s + ((SETTING[n.kind] and not n.meta.hidden) and 1 or 0)
                pc.l = pc.l + c.l + ((n.kind == "list" or n.kind == "map") and 1 or 0)
            end
        end
    end

    local configFiles = 0
    for _, f in ipairs(F) do
        if not f.dictKeys and #f.nodes > 0 then configFiles = configFiles + 1 end
    end
    local multi = configFiles > 1
    local hasDeclared = #declaredOrder > 0

    local fileTabs, groupTabs, tabOwner = {}, {}, {}
    local function fileTab(f)
        if fileTabs[f] then return fileTabs[f] end
        local base = f.file:match("([^/]+)%.[Ll][Uu][Aa]$") or f.file
        local id, label
        if f.dictKeys then
            id = textId
        elseif squash(base) == "config" or squash(base) == "general" then
            id = "general"
        else
            -- "ghostbuyer.lua" holding Config.GhostBuyer reads "Ghost buyer".
            local owner
            for _, n in ipairs(f.nodes) do
                local p = parentOf(n)
                if p and not p.parent and squash(n.key) == squash(base) then
                    label = own(n).label and n.meta.label or I.readable(n.key)
                    owner = n
                    break
                end
            end
            label = label or I.readable(base)
            id = uniqueId(slug(base))
            tabOwner[id] = owner
        end
        fileTabs[f] = useTab(id, label)
        return fileTabs[f]
    end
    local function groupTab(n)
        if groupTabs[n] then return groupTabs[n] end
        local id = uniqueId(slug(n.key))
        groupTabs[n] = useTab(id, n.meta.label)
        tabOwner[id] = n
        return id
    end
    local function ruleTab(n, f, p)
        if f.dictKeys then return useTab(textId) end
        if translationKey(p and p.key or n.key) then return useTab(textId) end
        if p and n.kind ~= "value" and translationKey(n.key) then return useTab(textId) end
        if hasDeclared then return useTab("general") end
        if multi then return fileTab(f) end
        if p and n.kind == "group" then
            local c = below[n]
            if c.s >= 8 or c.l > 0 then return groupTab(n) end
        end
        return useTab("general")
    end

    local tabOf = {}
    for _, f in ipairs(F) do
        for _, n in ipairs(f.nodes) do
            local p = parentOf(n)
            local id
            if inside[n] then
                id = tabOf[inside[n]]
            elseif type(n.meta.tab) == "string" and n.meta.tab ~= "" then
                id = useTab(n.meta.tab)   -- hub.json's, on the node or a parent
            elseif p and p.parent then
                id = tabOf[p]
            else
                id = ruleTab(n, f, p)
            end
            tabOf[n] = id
        end
    end
    for _, f in ipairs(F) do
        for _, n in ipairs(f.nodes) do
            n.tab = tabOf[n]
            n.meta.tab = tabOf[n]
            if inside[n] then
                n.item = inside[n].path
                n.row = rowOf[n]
            end
        end
    end

    -- 4. Sections and items, per tab, in file order.
    local nav = {}
    local function navTab(id)
        local t = nav[id]
        if not t then
            t = { id = id, label = tabs[id].label, icon = tabs[id].icon, count = 0, changed = 0,
                sections = {}, items = {}, secBy = {}, first = tabs[id].first }
            nav[id] = t
        end
        return t
    end
    local function groupLabel(g)
        local o = own(g)
        if o.label then return g.meta.label end
        if o.group and type(g.meta.group) == "string" then return g.meta.group end
        local pp = parentOf(g)
        local base = I.readable(g.key)
        if pp and pp.parent and tabOwner[tabOf[g]] ~= pp then
            return groupLabel(pp) .. " · " .. base
        end
        return base
    end
    local sectionParent = {}
    for _, f in ipairs(F) do
        for _, n in ipairs(f.nodes) do
            if not n.meta.hidden and not inside[n] then
                local t = navTab(tabOf[n])
                if SETTING[n.kind] then
                    local sid, slabel, sp
                    local og = own(n).group and n.meta.group or nil
                    local p = parentOf(n)
                    if type(og) == "string" and og ~= "" then
                        sid, slabel = "group:" .. og, og
                    elseif not p or not p.parent then
                        sid, slabel = t.id, t.label
                    else
                        sid, slabel, sp = p.path, groupLabel(p), parentOf(p)
                    end
                    local s = t.secBy[sid]
                    if not s then
                        s = { id = sid, label = slabel, count = 0, changed = 0 }
                        t.secBy[sid] = s
                        t.sections[#t.sections + 1] = s
                        sectionParent[s] = sp
                    end
                    s.count = s.count + 1
                    if n.changed then s.changed = s.changed + 1 end
                    n.section = sid
                    t.count = t.count + 1
                    if n.changed then t.changed = t.changed + 1 end
                elseif n.kind == "list" or n.kind == "map" then
                    local count, changed = 0, 0
                    if n.collection then
                        count = #n.collection.rows
                        for _, r in ipairs(n.collection.rows) do if r.changed then changed = changed + 1 end end
                    else
                        local rows = I.rowsOf(n.value)
                        count = #rows
                        if n.default ~= nil then
                            local seen = {}
                            for _, r in ipairs(rows) do
                                seen[tostring(r.key)] = true
                                if not I.equal(r.value, I.keyed(n.default, r.key)) then changed = changed + 1 end
                            end
                            for _, r in ipairs(I.rowsOf(n.default)) do
                                if not seen[tostring(r.key)] then changed = changed + 1 end
                            end
                        end
                    end
                    if n.changed and changed == 0 then changed = 1 end
                    t.items[#t.items + 1] = {
                        path = n.path, label = n.meta.label, file = n.file,
                        kind = n.collection and "collection" or n.kind,
                        display = n.display, count = count, changed = changed,
                    }
                    t.count = t.count + count
                    t.changed = t.changed + changed
                end
            end
        end
    end
    -- Two sections of one tab with the same name: say whose they are.
    for _, t in pairs(nav) do
        local c = {}
        for _, s in ipairs(t.sections) do c[s.label] = (c[s.label] or 0) + 1 end
        for _, s in ipairs(t.sections) do
            if c[s.label] > 1 and sectionParent[s] and sectionParent[s].parent then
                s.label = groupLabel(sectionParent[s]) .. " · " .. s.label
            end
        end
        t.secBy = nil
    end

    -- 4b. Data panels (§10): each is an item of kind "panel" under its tab.
    -- A tab hub.json declares (or one the settings already use) takes it,
    -- even when it has nothing else; any other tab id goes to "Data".
    b.panels = I.panelDecls(meta)
    for _, p in ipairs(b.panels) do
        local id
        if p.tab and (declared[p.tab] or tabs[p.tab]) then
            id = useTab(p.tab)
        else
            id = useTab("data", "Data", "database")
        end
        p.tab = id
        local t = navTab(id)
        t.items[#t.items + 1] = { path = "panel:" .. p.id, panel = p.id, label = p.label, kind = "panel" }
    end

    -- Order: General, hub.json's tabs as declared, the rest in file order,
    -- then Text & language and Advanced. Empty tabs are left out.
    local list, pushed = {}, {}
    local function push(id)
        local t = nav[id]
        if t and not pushed[id] and (#t.sections > 0 or #t.items > 0) then
            pushed[id] = true
            list[#list + 1] = t
        end
    end
    push("general")
    for _, id in ipairs(declaredOrder) do
        if id ~= textId and id ~= "advanced" then push(id) end
    end
    local rest = {}
    for id, t in pairs(nav) do
        if id ~= textId and id ~= "advanced" then rest[#rest + 1] = t end
    end
    table.sort(rest, function(a, c) return a.first < c.first end)
    for _, t in ipairs(rest) do push(t.id) end
    push(textId)
    push("advanced")
    local tabList = {}
    for _, t in ipairs(list) do
        t.first = nil
        tabList[#tabList + 1] = { id = t.id, label = t.label, icon = t.icon }
    end
    b.nav = list
    b.tabs = tabList

    -- 5. References (§8.5).
    b.refs = I.buildRefs(b, {
        kidsOf = function(n) return fileOf[n] and kidsOf(n) or EMPTY end,
        inside = inside, rowOf = rowOf, tabs = tabs,
    })
end

-- ---------------------------------------------------------------------------
-- References (§8.5)
--
-- hub.json can say a setting or a list field names a key of another table:
--     "Config.Stores": { "fields": { "sell": { "ref": "Config.Catalog" } } }
--     "Config.Recipes": { "fields": { "Category": { "ref": "Config.Categories", "refField": "name" } } }
-- The script answer carries, per target, its keys and who uses each one, so
-- the page can offer a dropdown, flag a name that is not a key, and show
-- "Used by N" on the target's rows. The server validates nothing extra: a
-- value that is not a key is allowed (the owner may be about to add it).
-- ---------------------------------------------------------------------------

--- Every place hub.json declares a reference: { kind = "setting" | "field",
--- node, target, refField, segs }. Collections kept as nodes are skipped as
--- a whole: their rows' own nodes carry the merged field meta.
function I.refSources(b)
    local out = {}
    for _, n in ipairs(b.nodes) do
        local m = n.meta
        if (n.kind == "value" or n.kind == "strings") and type(m.ref) == "string" and m.ref ~= "" then
            out[#out + 1] = { kind = "setting", node = n, target = m.ref, refField = m.refField }
        end
        if (n.kind == "list" or n.kind == "map") and plain(n.value) and plain(m.fields)
            and not (n.collection and n.collection.rowSource == "nodes") then
            for fk, fm in pairs(m.fields) do
                if type(fk) == "string" and plain(fm) and type(fm.ref) == "string" and fm.ref ~= "" then
                    out[#out + 1] = { kind = "field", node = n, target = fm.ref, refField = fm.refField,
                        segs = I.fieldSegs(fk) }
                end
            end
        end
    end
    return out
end

--- Walk a row by field segments; call fn(path, value, title) for every value
--- reached. A list of names at the end is walked name by name.
local function walkRef(v, segs, i, path, title, subMeta, fn)
    if i > #segs then
        if plain(v) then
            local n = countRowsOf(v)
            if n > 0 and #v == n then
                for j, x in ipairs(v) do
                    if not plain(x) then fn(I.child(path, j), x, title) end
                end
            end
        else
            fn(path, v, title)
        end
        return
    end
    if not plain(v) then return end
    local s = segs[i]
    if s == "[]" then
        -- An inner row (a sublist's): its own title first, the outer after.
        local sm = subMeta and subMeta[segs[i - 1]] or nil
        for _, r in ipairs(I.rowsOf(v)) do
            local t2 = title
            if i < #segs then
                local inner = I.rowTitle(sm, r.value, nil, true)
                if inner then t2 = inner .. " · " .. title end
            end
            walkRef(r.value, segs, i + 1, I.child(path, r.key), t2, subMeta, fn)
        end
    else
        walkRef(I.keyed(v, s), segs, i + 1, I.child(path, s), title, subMeta, fn)
    end
end

--- Every value a reference source holds: calls fn(path, value, label, where).
function I.eachRefValue(src, fn, ctx)
    local n = src.node
    if src.kind == "setting" then
        local label = n.meta.label
        if ctx and ctx.rowOf and ctx.rowOf[n] ~= nil then label = I.readable(ctx.rowOf[n]) .. " · " .. label end
        local where = ctx and ctx.inside and ctx.inside[n] and ctx.inside[n].meta.label
            or (ctx and ctx.tabs and n.tab and ctx.tabs[n.tab] and ctx.tabs[n.tab].label) or nil
        if n.kind == "strings" and plain(n.value) then
            for j, x in ipairs(n.value) do fn(I.child(n.path, j), x, label, where) end
        else
            fn(n.path, n.value, label, where)
        end
        return
    end
    local subMeta = hubSublists(n.meta)
    for _, r in ipairs(I.rowsOf(n.value)) do
        local title = I.rowTitle(n.meta, r.value, r.key)
        walkRef(r.value, src.segs, 1, I.child(n.path, r.key), title, subMeta, function(path, v, t)
            fn(path, v, t, n.meta.label)
        end)
    end
end

--- The keys of a reference target: a map's keys, a group's keys, a
--- collection's row keys, or with refField that field of every row.
function I.refKeys(b, target, refField, kidsOf)
    local t = I.nodeAt(b.byPath, target)
    if not t then return {}, false end
    local keys, seen = {}, {}
    local function add(k)
        if k == nil or (type(k) ~= "string" and type(k) ~= "number") then return end
        k = tostring(k)
        if not seen[k] then seen[k] = true; keys[#keys + 1] = k end
    end
    local field = type(refField) == "string" and refField ~= "" and refField or nil
    if t.collection and t.collection.rowSource == "nodes" then
        for _, r in ipairs(t.collection.rows) do
            if field then
                local rv = I.keyed(t.value, r.key)
                add(plain(rv) and rv[field] or nil)
            else
                add(r.key)
            end
        end
    elseif (t.kind == "list" or t.kind == "map") and plain(t.value) then
        for _, r in ipairs(I.rowsOf(t.value)) do
            if field then
                add(plain(r.value) and r.value[field] or nil)
            elseif t.kind == "map" then
                add(r.key)
            end
        end
    elseif t.kind == "strings" and plain(t.value) then
        for _, x in ipairs(t.value) do add(x) end
    elseif t.kind == "group" and kidsOf then
        for _, c in ipairs(kidsOf(t)) do
            if field then
                for _, g in ipairs(kidsOf(c)) do
                    if g.key == field then add(g.value) end
                end
            else
                add(c.key)
            end
        end
    end
    return keys, true
end

--- refs for the script answer: { target = { keys, usedBy = { key = { {path,
--- label, where} } }, unknown = { {path, label, where, value} }, found } }.
function I.buildRefs(b, ctx)
    local refs = {}
    local sources = I.refSources(b)
    b.refSourceList = sources
    for _, src in ipairs(sources) do
        local r = refs[src.target]
        if not r then
            local keys, found = I.refKeys(b, src.target, src.refField, ctx.kidsOf)
            local set = {}
            for _, k in ipairs(keys) do set[k] = true end
            r = { keys = keys, usedBy = {}, unknown = {}, found = found, set = set }
            refs[src.target] = r
        end
        I.eachRefValue(src, function(path, v, label, where)
            if type(v) ~= "string" and type(v) ~= "number" then return end   -- false = none
            local k = tostring(v)
            if r.set[k] then
                local list = r.usedBy[k]
                if not list then list = {}; r.usedBy[k] = list end
                list[#list + 1] = { path = path, label = label, where = where }
            else
                r.unknown[#r.unknown + 1] = { path = path, label = label, where = where, value = v }
            end
        end, ctx)
    end
    -- "Used by N" on the target's rows.
    for target, r in pairs(refs) do
        r.set = nil
        local t = I.nodeAt(b.byPath, target)
        if t and t.collection then
            for _, row in ipairs(t.collection.rows) do
                local list = r.usedBy[tostring(row.key)]
                row.usedBy = list and #list or 0
            end
        end
    end
    return refs
end

-- ---------------------------------------------------------------------------
-- Item checks (§9.1)
--
-- Item names are typed by hand, and the first real use (BritanniaRP) found
-- them wrong in case or spelling with nothing to say so, and crafting hid
-- recipes whose items had no icon without the hub knowing. So, for a script
-- whose hub.json says "itemCheck": true, the `script` answer carries every
-- item name the script's config uses, each checked against the server:
--
--   exists   the name is in the item registry, EXACTLY (case matters: the
--            inventory looks items up by exact name). A weapon name
--            (WEAPON_...) counts as existing: weapons are not items in the
--            registry, and a recipe may still reward one.
--   suggest  when it does not exist, the registry name that matches it
--            case-insensitively, if there is one ("steel_Bar" -> "steel_bar")
--   icon     whether the icon file is there, checked the way the game checks
--            it: LoadResourceFile(resource, pattern with the name in it)
--   label    the registry's label, when it exists
--   file     "<resource>/<path>" the icon should be at, when it is missing
--
-- The registry is read once with Poggy('inv.items', { limit = 100000 }) and
-- kept for five minutes (or read again on demand, `itemList` with fresh);
-- icon answers are kept per file for as long as the registry is.
--
-- Where icons live: the inventory's image base (inv.imageBase, e.g.
-- nui://vorp_inventory/html/img/items/ -> resource vorp_inventory, pattern
-- html/img/items/%s.png). A script that looks icons up itself names its own
-- settings in hub.json, "itemIcons": { "resourcePath": "Config.IconResource",
-- "pathPath": "Config.IconPath" }, so the hub checks exactly the file the
-- script checks. A setting that is missing from the config falls back to the
-- inventory's answer.
--
-- Which names: every string value of every field whose meta says
-- picker = "item": settings, lists of names, list fields ("Items[].name"),
-- names inside a list field ("Items[].AltNames[]"), and the sublists of
-- collections. Empty strings and non-strings (false = none) are skipped.
-- ---------------------------------------------------------------------------

local REGISTRY_SECONDS = 300        -- how long the item registry is kept
local REGISTRY_RETRY_SECONDS = 30   -- how soon a registry that could not be read is tried again
local MAX_CHECK_NAMES = 500         -- itemCheck(names): names per call

local registry = { at = nil }       -- { at, ttl, list, byName, byLower, err }
local iconSeen = {}                 -- resource .. "|" .. rel -> boolean

--- The server's item registry: { list = { {name, label, image} } sorted by
--- name, byName, byLower (ASCII lower case -> first item with that name),
--- err }. list and byName are nil when the registry cannot be read (no
--- framework, or one without an item table): nothing is then called missing.
function Hub.ItemRegistry(fresh)
    local age = registry.at and (I.now() - registry.at) or nil
    if not fresh and age and age < (registry.ttl or REGISTRY_SECONDS) then return registry end
    if fresh and PoggyCore.DropItemCache then pcall(PoggyCore.DropItemCache) end
    local ok, list, err = PoggyCore.Do("inv.items", { limit = 100000 })
    local r = { at = I.now(), ttl = REGISTRY_SECONDS }
    if ok and type(list) == "table" then
        r.list, r.byName, r.byLower = {}, {}, {}
        for _, it in ipairs(list) do
            if type(it) == "table" and type(it.name) == "string" and it.name ~= "" and not r.byName[it.name] then
                local e = {
                    name = it.name,
                    label = type(it.label) == "string" and it.label ~= "" and it.label or it.name,
                    image = type(it.image) == "string" and it.image or nil,
                }
                r.list[#r.list + 1] = e
                r.byName[e.name] = e
                local low = I.lower(e.name)
                if r.byLower[low] == nil then r.byLower[low] = e end
            end
        end
        table.sort(r.list, function(a, b)
            local la, lb = I.lower(a.name), I.lower(b.name)
            if la ~= lb then return la < lb end
            return a.name < b.name
        end)
    else
        r.err = tostring(err or "unsupported")
        r.ttl = REGISTRY_RETRY_SECONDS
    end
    registry = r
    iconSeen = {}   -- icons are looked at again with every fresh registry
    return r
end

--- A weapon name: weapons are not rows of the item registry.
local function isWeaponName(name)
    return name:find("^[Ww][Ee][Aa][Pp][Oo][Nn]_") ~= nil
end

--- An item name that is safe inside a file path (no folders, no ..).
local function pathSafeName(name)
    return #name <= 100 and not name:find("..", 1, true) and not name:find("[/\\:%c]")
end

--- Where a script's item icons are: { resource, pattern, base, source =
--- "config" | "inventory" } or nil when nothing says. pattern has one %s.
function I.iconLocation(b)
    local loc
    local okBase, base = PoggyCore.Do("inv.imageBase", {})
    if okBase and type(base) == "string" then
        local res, dir = base:match("^nui://([^/]+)/(.*)$")
        if res then loc = { resource = res, pattern = dir .. "%s.png", base = base, source = "inventory" } end
    end
    local ic = b and b.meta and plain(b.meta.itemIcons) and b.meta.itemIcons or nil
    if ic then
        local function setting(path)
            if type(path) ~= "string" or path == "" then return nil end
            local n = I.nodeAt(b.byPath, path)
            return n and type(n.value) == "string" and n.value ~= "" and n.value or nil
        end
        local res, pattern = setting(ic.resourcePath), setting(ic.pathPath)
        if pattern and not pattern:find("%s", 1, true) then pattern = nil end
        if res or pattern then
            loc = {
                resource = res or (loc and loc.resource), pattern = pattern or (loc and loc.pattern),
                base = loc and loc.base, source = "config",
            }
            if not loc.resource or not loc.pattern then loc = nil end
        end
    end
    return loc
end

--- The icon file of one name, relative to loc.resource, or nil when the name
--- cannot be part of a path. The inventory's own image for a registry item
--- when icons come from the inventory (RSG and QBR name files by an image
--- field, not always <name>.png); the pattern otherwise.
local function iconRel(loc, name, item)
    if not pathSafeName(name) then return nil end
    if loc.source == "inventory" and item and item.image then
        local prefix = "nui://" .. loc.resource .. "/"
        if item.image:sub(1, #prefix) == prefix then return item.image:sub(#prefix + 1) end
    end
    return (loc.pattern:gsub("%%s", function() return name end, 1))
end

--- Is this icon file there? Cached with the registry.
local function iconExists(resource, rel)
    local key = resource .. "|" .. rel
    local seen = iconSeen[key]
    if seen == nil then
        local okLoad, data = pcall(LoadResourceFile, resource, rel)
        seen = okLoad and type(data) == "string" and #data > 0 or false
        iconSeen[key] = seen
    end
    return seen
end

--- The §9.1 check of a set of names: { name = { exists, icon, label, suggest,
--- weapon, file } }, plus counts { names, missing, wrongCase, noIcon }.
--- exists is absent when the registry cannot be read; icon when no icon
--- location is known.
function I.itemValues(names, loc)
    local reg = Hub.ItemRegistry()
    local values, counts = {}, { names = 0, missing = 0, wrongCase = 0, noIcon = 0 }
    for _, name in ipairs(names) do
        if type(name) == "string" and name ~= "" and values[name] == nil then
            local v = {}
            local item = reg.byName and reg.byName[name] or nil
            if reg.byName then
                if item then
                    v.exists, v.label = true, item.label
                elseif isWeaponName(name) then
                    v.exists, v.weapon = true, true
                else
                    v.exists = false
                    local near = reg.byLower[I.lower(name)]
                    if near then v.suggest = near.name end
                    counts.missing = counts.missing + 1
                    if near then counts.wrongCase = counts.wrongCase + 1 end
                end
            end
            if loc then
                local rel = iconRel(loc, name, item)
                v.icon = rel ~= nil and iconExists(loc.resource, rel) or false
                if not v.icon then
                    if rel then v.file = loc.resource .. "/" .. rel end
                    counts.noIcon = counts.noIcon + 1
                end
            end
            values[name] = v
            counts.names = counts.names + 1
        end
    end
    return values, counts
end

--- Every item name a script's config uses (picker "item"), sorted, once per build.
function I.itemNames(b)
    if b.itemNames then return b.itemNames end
    local seen, out = {}, {}
    local function add(v)
        if type(v) == "string" and v ~= "" and not seen[v] then seen[v] = true; out[#out + 1] = v end
    end
    for _, n in ipairs(b.nodes) do
        local m = n.meta or {}
        if m.picker == "item" and not n.cell then
            if n.kind == "value" then
                add(n.value)
            elseif n.kind == "strings" and plain(n.value) then
                for _, x in ipairs(n.value) do add(x) end
            end
        end
        -- List fields. A collection kept as nodes is skipped as a whole: its
        -- rows are nodes of their own and carry the merged field meta.
        if (n.kind == "list" or n.kind == "map") and plain(n.value) and plain(m.fields)
            and not (n.collection and n.collection.rowSource == "nodes") then
            for fk, fm in pairs(m.fields) do
                if type(fk) == "string" and plain(fm) and fm.picker == "item" then
                    local segs = I.fieldSegs(fk)
                    for _, r in ipairs(I.rowsOf(n.value)) do
                        walkRef(r.value, segs, 1, I.child(n.path, r.key), "", nil, function(_, v) add(v) end)
                    end
                end
            end
        end
    end
    table.sort(out)
    b.itemNames = out
    return out
end

--- The `itemCheck` block of the script answer (§9.1), or nil when hub.json
--- does not ask for it.
function I.itemCheck(b)
    if b.meta.itemCheck ~= true then return nil end
    local loc = I.iconLocation(b)
    local values, counts = I.itemValues(I.itemNames(b), loc)
    local reg = Hub.ItemRegistry()
    return {
        available = reg.byName ~= nil,     -- false: the registry could not be read; nothing is "missing"
        reason = reg.byName == nil and ("The item list is not available on this server (%s)."):format(reg.err or "unsupported") or nil,
        iconResource = loc and loc.resource or nil,
        iconPattern = loc and loc.pattern or nil,
        values = values,
        counts = counts,
    }
end

--- `itemList` (§9.1): the whole registry for type-ahead. image is sent only
--- for an item whose icon is not imageBase .. name .. ".png".
function I.itemList(fresh)
    local reg = Hub.ItemRegistry(fresh == true)
    local okBase, base = PoggyCore.Do("inv.imageBase", {})
    base = okBase and type(base) == "string" and base or nil
    local items = {}
    for _, it in ipairs(reg.list or {}) do
        local e = { name = it.name, label = it.label }
        if it.image and (not base or it.image ~= base .. it.name .. ".png") then e.image = it.image end
        items[#items + 1] = e
    end
    return {
        items = items,
        imageBase = base,
        available = reg.list ~= nil,
        reason = reg.list == nil and ("The item list is not available on this server (%s)."):format(reg.err or "unsupported") or nil,
    }
end

-- ---------------------------------------------------------------------------
-- Script diagnostics (§9.2)
--
-- Some things only the script knows: poggy_crafting hides a recipe when one
-- of its items has no icon, and hides the recipes further down the chain that
-- can no longer be made. A script says so through a server export that
-- hub.json names ("diagnostics": "HubDiagnostics"), returning
--   { summary = "...", rows = { ["Config.Crafting[37]"] = { { level, code,
--     message, items = {...}, files = {...} } } } }
-- It reflects the config the script is RUNNING, so it is asked only when the
-- script is started, and changes after save + restart. The call is pcall'd;
-- a failure is logged once per script and reason, and the hub opens without it.
-- What comes back crosses a resource boundary, so it is copied into a known
-- shape (strings and lists of strings, capped) before it goes to the page.
-- ---------------------------------------------------------------------------

local diagWarned = {}   -- folder .. "|" .. export .. "|" .. reason -> true
local DIAG_LEVELS = { error = true, warning = true, info = true }

--- A string or number as text of at most about n bytes, never cut inside a
--- UTF-8 character.
local function diagText(v, n)
    if type(v) ~= "string" and type(v) ~= "number" then return nil end
    local text = tostring(v)
    if #text <= n then return text end
    local cut = n - 1
    while cut > 0 do
        local byte = text:byte(cut + 1)
        if not byte or byte < 0x80 or byte > 0xBF then break end
        cut = cut - 1
    end
    return text:sub(1, cut) .. "…"
end

local function diagStrings(t, most)
    if type(t) ~= "table" then return nil end
    local out = {}
    for _, x in ipairs(t) do
        local s = type(x) == "string" and diagText(x, 200) or nil
        if s then out[#out + 1] = s end
        if #out >= most then break end
    end
    return #out > 0 and out or nil
end

--- The export's answer in the §9.2 shape: { summary, rows = { path = { entry } } }.
function I.cleanDiagnostics(raw)
    local out = { summary = diagText(raw.summary, 400), rows = {} }
    local rows, n = type(raw.rows) == "table" and raw.rows or {}, 0
    for path, entries in pairs(rows) do
        if type(path) == "string" and type(entries) == "table" then
            local list = {}
            for _, e in ipairs(entries) do
                if type(e) == "table" and diagText(e.message, 400) then
                    list[#list + 1] = {
                        level = DIAG_LEVELS[e.level] and e.level or "warning",
                        code = diagText(e.code, 40),
                        message = diagText(e.message, 400),
                        items = diagStrings(e.items, 50),
                        files = diagStrings(e.files, 50),
                    }
                end
                if #list >= 20 then break end
            end
            if #list > 0 then
                out.rows[I.canon(path)] = list
                n = n + 1
                if n >= 5000 then break end
            end
        end
    end
    return out
end

--- diagnostics for the script answer: value, note. The note is plain
--- English for the page when hub.json asks for diagnostics and there are none.
function I.diagnostics(s, b)
    local name = b.meta.diagnostics
    if type(name) ~= "string" or not name:find("^[%a_][%w_]*$") then return nil end
    if GetResourceState(s.folder) ~= "started" then
        return nil, "Start the script to see what it reports about its rows."
    end
    local okCall, res = pcall(function() return exports[s.folder][name]() end)
    local why = not okCall and tostring(res) or (type(res) ~= "table" and ("it returned " .. type(res)) or nil)
    if why then
        local key = s.folder .. "|" .. name .. "|" .. why
        if not diagWarned[key] then
            diagWarned[key] = true
            Util.Warn("hub: %s's diagnostics export %s could not be read (%s); the hub opens without it.",
                s.folder, name, why)
        end
        return nil, "The script's own report could not be read; the server console says why."
    end
    return I.cleanDiagnostics(res)
end

-- ---------------------------------------------------------------------------
-- Data panels (§10)
--
-- Some things owners manage live in a script's database, not its config
-- (poggy_markets' shop jobs: which job each shop gives its staff). hub.json
-- declares a panel:
--     "panels": [ { "id": "shopjobs", "label": "Shop jobs", "tab": "shopjobs",
--                   "resource": "poggy_markets", "export": "HubPanel",
--                   "itemLabel": "shop", "tooltip": "..." } ]
-- and the script that OWNS the data (resource; default the declaring script)
-- answers through a server export:
--     HubPanel(panelId, "read", payload)                      -> { columns, rows, note, actions }
--     HubPanel(panelId, "write", { key, field, value }, who)  -> true, "message" | false, "reason"
--     HubPanel(panelId, "action", { action, key, input }, who) -> true, "message" | false, "reason"
-- A panel may sit in another script's hub.json (poggy_multijob shows markets'
-- shop jobs). The hub adds nothing of its own: it asks, checks the shape,
-- enforces the permission (register), refuses a write to a column the latest
-- read does not call editable and an action the latest read does not offer,
-- and logs every change to History. No edit lock: each change is one small
-- step the script validates. Every export call is pcall'd and its answer
-- copied into a known shape, so a failing script never breaks the hub.
--
-- §10.5 on top:
--   actions     buttons per row or on the toolbar, with input fields asked
--               first and a confirmation for dangerous ones (typed: the row's
--               key, or CONFIRM for the panel), checked here as well
--   drill-down  a row with `detail = "<panelId>"` opens that panel, read with
--               payload { parent = <row key> }. The page sends the declared
--               panel it started from (`root`), whose script and export answer.
--   contents    a row with `container = "<id>"` gets Contents, built here
--               from the storage verbs (storage.items / storage.weapons,
--               storage.addItem / storage.removeItem): no script code.
-- ---------------------------------------------------------------------------

local PANEL_ROWS = 5000        -- rows sent to the page at most
local PANEL_COLUMNS = 40
local PANEL_ACTIONS = 20
local PANEL_INPUTS = 12
local PANEL_STRING = 4096      -- longest string a cell, a write or an input may carry
local PANEL_LEVELS = { warning = true, error = true, info = true }
local PANEL_PICKERS = { job = true, item = true, group = true, role = true, key = true, text = true,
    multiline = true, color = true, coords = true }
local INPUT_TYPES = { number = true, text = true, boolean = true }
local CONFIRMS = { typed = true, simple = true }
local panelWarned = {}         -- folder .. "|" .. panel .. "|" .. reason -> true

local function scalar(v) return type(v) == "string" or type(v) == "number" or type(v) == "boolean" end
local function finiteNumber(v) return type(v) == "number" and v == v and v ~= math.huge and v ~= -math.huge end
local function idLike(v, n) return type(v) == "string" and v ~= "" and #v <= (n or 64) and v:find("^[%w_%-]+$") ~= nil end
local function rowKeyLike(v)
    return (type(v) == "string" and v ~= "" and #v <= 200) or (type(v) == "number" and v == v)
end

--- hub.json `panels`, checked: { id, label, tab, itemLabel, tooltip, resource, export }.
function I.panelDecls(meta)
    local out, seen = {}, {}
    if type(meta) ~= "table" or type(meta.panels) ~= "table" then return out end
    for _, p in ipairs(meta.panels) do
        if type(p) == "table" and idLike(p.id) and not seen[p.id] then
            local export = type(p.export) == "string" and p.export ~= "" and p.export or "HubPanel"
            if export:find("^[%a_][%w_]*$") then
                seen[p.id] = true
                out[#out + 1] = {
                    id = p.id,
                    label = type(p.label) == "string" and p.label ~= "" and diagText(p.label, 80) or I.readable(p.id),
                    tab = type(p.tab) == "string" and p.tab ~= "" and p.tab or nil,
                    itemLabel = type(p.itemLabel) == "string" and p.itemLabel ~= "" and diagText(p.itemLabel, 40) or nil,
                    tooltip = diagText(p.tooltip, 600),
                    resource = type(p.resource) == "string" and p.resource:find("^[%w_%-%.]+$") and p.resource or nil,
                    export = export,
                }
            end
        end
    end
    return out
end

--- The folder of the script that owns a panel's data: the declaring script,
--- or the named one (a poggy_id or a folder, so a renamed folder still works).
function I.panelFolder(s, p)
    if not p.resource or p.resource == s.id or p.resource == s.folder then return s.folder end
    for _, x in ipairs(Hub.Scripts()) do
        if x.id == p.resource or x.folder == p.resource then return x.folder end
    end
    return p.resource
end

--- folder, available, reason (plain English when it is not available).
function I.panelState(s, p)
    local folder = I.panelFolder(s, p)
    local okState, state = pcall(GetResourceState, folder)
    state = okState and state or "missing"
    if state == "missing" or state == "unknown" then
        return folder, false, ("%s is not on this server, so there is nothing to show here."):format(folder)
    end
    if state ~= "started" then
        return folder, false, ("%s is not running. Start it to see and change this data."):format(folder)
    end
    -- As CFX: asking a resource for an export it does not have is an error.
    local okEx, fn = pcall(function() return exports[folder][p.export] end)
    if not okEx or fn == nil then
        return folder, false, ("%s has no %s export. Update it to a version that has this panel."):format(folder, p.export)
    end
    return folder, true
end

--- The panel as the script answer and the panel call describe it.
function I.panelView(p, folder, available, reason)
    return {
        id = p.id, label = p.label, tab = p.tab, itemLabel = p.itemLabel, tooltip = p.tooltip,
        resource = folder, available = available and true or false, reason = reason,
    }
end

local function panelWarn(folder, p, why)
    local key = folder .. "|" .. p.id .. "|" .. why
    if panelWarned[key] then return end
    panelWarned[key] = true
    Util.Warn("hub: %s's %s export failed for the panel '%s' (%s).", folder, p.export, p.id, why)
end

--- exports[folder][export](panelId, action, payload, who), pcall'd: ok, ...
local function callPanel(folder, p, action, payload, who)
    return pcall(function()
        local ex = exports[folder]
        return ex[p.export](ex, p.id, action, payload, who)
    end)
end

--- A value as plain JSON: strings (clipped), finite numbers, booleans, and
--- small tables of those. Anything else is nil.
local function panelValue(v, depth)
    local t = type(v)
    if t == "string" then return #v <= PANEL_STRING and v or diagText(v, PANEL_STRING) end
    if t == "number" then return finiteNumber(v) and v or nil end
    if t == "boolean" then return v end
    if t == "table" and depth < 3 then
        local out, n = {}, 0
        for k, x in pairs(v) do
            if type(k) == "string" or type(k) == "number" then
                n = n + 1
                if n > 100 then break end
                out[k] = panelValue(x, depth + 1)
            end
        end
        return out
    end
    return nil
end
I.panelValue = function(v) return panelValue(v, 0) end

local function cleanOptions(list)
    if type(list) ~= "table" then return nil end
    local opts = {}
    for _, o in ipairs(list) do
        if type(o) == "table" and scalar(o.value) then
            opts[#opts + 1] = { value = panelValue(o.value, 0), label = diagText(o.label, 120) or tostring(o.value) }
        elseif scalar(o) then
            opts[#opts + 1] = { value = panelValue(o, 0), label = tostring(o) }
        end
        if #opts >= 500 then break end
    end
    return #opts > 0 and opts or nil
end

--- An action's input fields: { field, label, picker, options, type, min, max, step, default, required, tooltip, placeholder }.
local function cleanInputs(list)
    local out, seen = {}, {}
    if type(list) ~= "table" then return out end
    for _, f in ipairs(list) do
        if type(f) == "table" and type(f.field) == "string" and f.field:find("^[%w_%-%.]+$") and #f.field <= 64 and not seen[f.field] then
            seen[f.field] = true
            out[#out + 1] = {
                field = f.field,
                label = diagText(f.label, 80) or I.readable(f.field),
                picker = type(f.picker) == "string" and PANEL_PICKERS[f.picker] and f.picker or nil,
                options = cleanOptions(f.options),
                type = INPUT_TYPES[f.type] and f.type or nil,
                min = finiteNumber(f.min) and f.min or nil,
                max = finiteNumber(f.max) and f.max or nil,
                step = finiteNumber(f.step) and f.step > 0 and f.step or nil,
                default = scalar(f.default) and panelValue(f.default, 0) or nil,
                required = f.required == true or nil,
                tooltip = diagText(f.tooltip, 300),
                placeholder = diagText(f.placeholder, 120),
            }
            if #out >= PANEL_INPUTS then break end
        end
    end
    return out
end

--- `actions` of a read answer (§10.5).
local function cleanActions(list)
    local out, seen = {}, {}
    if type(list) ~= "table" then return out end
    for _, a in ipairs(list) do
        if type(a) == "table" and idLike(a.id, 40) and not seen[a.id] then
            seen[a.id] = true
            local danger = a.danger == true
            local confirm = CONFIRMS[a.confirm] and a.confirm or (danger and "simple" or nil)
            out[#out + 1] = {
                id = a.id,
                label = diagText(a.label, 60) or I.readable(a.id),
                icon = type(a.icon) == "string" and #a.icon <= 24 and a.icon:find("^[%w_%-]+$") and a.icon or nil,
                scope = a.scope == "panel" and "panel" or "row",
                input = cleanInputs(a.input),
                danger = danger or nil,
                confirm = confirm,
                tooltip = diagText(a.tooltip, 300),
            }
            if #out >= PANEL_ACTIONS then break end
        end
    end
    return out
end

--- The export's read answer in the §10.2 / §10.5 shape, or nil and why.
function I.cleanPanel(raw)
    if type(raw) ~= "table" then return nil, "it returned " .. type(raw) .. ", not a table" end
    if type(raw.columns) ~= "table" then return nil, "the answer has no columns" end
    if raw.rows ~= nil and type(raw.rows) ~= "table" then return nil, "rows is not a list" end
    local columns, byField = {}, {}
    for _, c in ipairs(raw.columns) do
        if type(c) == "table" and type(c.field) == "string" and c.field ~= "" and #c.field <= 64 and not byField[c.field] then
            local col = {
                field = c.field,
                label = diagText(c.label, 80) or I.readable(c.field),
                editable = c.editable == true,
                picker = type(c.picker) == "string" and PANEL_PICKERS[c.picker] and c.picker or nil,
                options = cleanOptions(c.options),
                type = INPUT_TYPES[c.type] and c.type or nil,
                min = finiteNumber(c.min) and c.min or nil,
                max = finiteNumber(c.max) and c.max or nil,
                step = finiteNumber(c.step) and c.step > 0 and c.step or nil,
                tooltip = diagText(c.tooltip, 400),
                width = (type(c.width) == "number" and c.width > 0 and c.width <= 2000 and c.width)
                    or (type(c.width) == "string" and #c.width <= 12 and c.width) or nil,
            }
            byField[col.field] = col
            columns[#columns + 1] = col
            if #columns >= PANEL_COLUMNS then break end
        end
    end
    -- No columns is fine while there is nothing to show (a script still
    -- loading answers { columns = {}, rows = {}, note = "..." }).
    if #columns == 0 and type(raw.rows) == "table" and raw.rows[1] ~= nil then
        return nil, "the answer has rows but no usable columns"
    end
    local actions = cleanActions(raw.actions)
    local rows, seen, total = {}, {}, 0
    for _, r in ipairs(raw.rows or {}) do
        local key = type(r) == "table" and r.key or nil
        if rowKeyLike(key) then
            local k = tostring(key)
            if not seen[k] then
                seen[k] = true
                total = total + 1
                if #rows < PANEL_ROWS then
                    local cells = {}
                    if type(r.cells) == "table" then
                        for f in pairs(byField) do cells[f] = panelValue(r.cells[f], 0) end
                    end
                    local rowActions
                    if type(r.actions) == "table" then
                        rowActions = {}
                        for _, a in ipairs(r.actions) do
                            if idLike(a, 40) then rowActions[#rowActions + 1] = a end
                            if #rowActions >= PANEL_ACTIONS then break end
                        end
                    end
                    local details
                    if type(r.details) == "table" then
                        details = {}
                        for _, d in ipairs(r.details) do
                            if idLike(d) then details[#details + 1] = d end
                            if #details >= 10 then break end
                        end
                        if #details == 0 then details = nil end
                    end
                    rows[#rows + 1] = {
                        key = key, cells = cells, note = diagText(r.note, 300),
                        level = PANEL_LEVELS[r.level] and r.level or nil,
                        actions = rowActions,
                        detail = idLike(r.detail) and r.detail or nil,   -- what a click on the row opens
                        details = details,                              -- every drill-down it offers
                        container = type(r.container) == "string" and r.container ~= "" and #r.container <= 128 and r.container or nil,
                    }
                end
            end
        end
    end
    return {
        columns = columns, rows = rows, actions = actions, note = diagText(raw.note, 600),
        label = diagText(raw.label, 80),   -- optional: a drill-down panel's own title
        total = total, truncated = total > #rows or nil,
    }
end

--- The script and the declared panel, or nil, nil, failure.
function I.findPanel(id, panelId)
    local s, failure = I.script(id)
    if not s then return nil, nil, failure end
    local b = Hub.Build(s)
    for _, p in ipairs(b.panels or {}) do
        if p.id == panelId then return s, p end
    end
    return nil, nil, I.fail("missing", ("This script has no data panel called %s."):format(tostring(panelId)))
end

--- Which panel a call is about. ctx (from the page, optional) = { parent =
--- <row key>, root = <declared panel id> }: a drill-down panel (§10.5) is
--- answered by its root's script and export, and read with { parent }.
--- Returns s, p, parent, or nil, nil, nil, failure.
function I.resolvePanel(id, panelId, ctx)
    ctx = type(ctx) == "table" and ctx or {}
    local parent = rowKeyLike(ctx.parent) and ctx.parent or nil
    local root = idLike(ctx.root) and ctx.root or panelId
    local s, p, failure = I.findPanel(id, root)
    if not s and root == panelId and parent ~= nil then
        -- A drill-down panel with no root named: when every panel the script
        -- declares is answered by one export, that export answers this too.
        local sx = I.script(id)
        local decl = sx and (Hub.Build(sx).panels or {}) or {}
        local same = decl[1] ~= nil
        for _, d in ipairs(decl) do
            if d.export ~= decl[1].export or d.resource ~= decl[1].resource then same = false break end
        end
        if same then s, p = sx, decl[1] end
    end
    if not s then return nil, nil, nil, failure end
    if panelId ~= p.id then
        if not idLike(panelId) then
            return nil, nil, nil, I.fail("invalid", "That is not a panel name.", { reason = "bad panel id" })
        end
        if parent == nil then
            return nil, nil, nil, I.fail("invalid", "A drill-down panel is opened from a row; the row is missing.", { reason = "no parent" })
        end
        -- Sub-panels are not declared: the declared panel's script and export answer them.
        p = { id = panelId, label = I.readable(panelId), export = p.export, resource = p.resource, tab = p.tab, root = p.id }
    end
    return s, p, parent
end

--- Ask the owning script for its data: the clean read, or nil and a failure.
function I.readPanel(s, p, folder, who, parent)
    local okCall, res = callPanel(folder, p, "read", parent ~= nil and { parent = parent } or nil, who)
    if not okCall then
        panelWarn(folder, p, tostring(res))
        return nil, I.fail("panel_error", ("%s could not read this data: %s"):format(folder, diagText(tostring(res), 300)))
    end
    local clean, why = I.cleanPanel(res)
    if not clean then
        panelWarn(folder, p, why)
        return nil, I.fail("panel_error", ("%s answered in a shape the hub cannot show (%s)."):format(folder, why))
    end
    return clean
end

--- s, p, parent, folder, data (the latest read), or nil + failure. Every
--- change starts here: the columns and actions are the ones the script
--- offers now, not the ones the page was shown.
local function freshRead(src, id, panelId, ctx)
    local s, p, parent, failure = I.resolvePanel(id, panelId, ctx)
    if not s then return nil, failure end
    local folder, available, reason = I.panelState(s, p)
    if not available then return nil, I.fail("unavailable", reason, { reason = reason }) end
    local data, fail = I.readPanel(s, p, folder, I.who(src), parent)
    if not data then return nil, fail end
    return { s = s, p = p, parent = parent, folder = folder, data = data }
end

local function rowOf(data, key)
    for _, r in ipairs(data.rows) do
        if tostring(r.key) == tostring(key) then return r end
    end
    return nil
end

--- The export's answer to a write or an action: done, message.
local function outcome(a, b)
    if type(a) == "table" then return a.ok == true, diagText(a.message or a.reason, 400) end
    return a == true, diagText(b, 400)
end

--- panel(id, panelId, ctx): the panel with its columns, rows and actions.
function Hub.Panel(src, id, panelId, ctx)
    local s, p, parent, failure = I.resolvePanel(id, panelId, ctx)
    if not s then return failure end
    local folder, available, reason = I.panelState(s, p)
    local out = I.panelView(p, folder, available, reason)
    out.parent, out.root = parent, p.root
    -- A drill-down has no declared name; the page names it from its row
    -- unless the script's answer carries a `label`.
    if p.root then out.label = nil end
    if not available then
        out.columns, out.rows, out.actions = {}, {}, {}
        return I.ok(out)
    end
    local data, fail = I.readPanel(s, p, folder, I.who(src), parent)
    if not data then return fail end
    for k, v in pairs(data) do out[k] = v end
    out.readAt = I.now()
    return I.ok(out)
end

--- panelWrite(id, panelId, key, field, value, ctx): one cell, through the script.
function Hub.PanelWrite(src, id, panelId, key, field, value, ctx)
    if not rowKeyLike(key) then
        return I.fail("invalid", "Which row? The change named none.", { reason = "no row key" })
    end
    if type(field) ~= "string" or field == "" then
        return I.fail("invalid", "Which column? The change named none.", { reason = "no field" })
    end
    if type(value) == "string" and #value > PANEL_STRING then
        return I.fail("invalid", ("That text is too long (the most is %d characters)."):format(PANEL_STRING), { reason = "too long" })
    end
    local clean = panelValue(value, 0)
    if value ~= nil and clean == nil then
        return I.fail("invalid", "That value cannot be stored.", { reason = "bad value" })
    end
    local f, failure = freshRead(src, id, panelId, ctx)
    if not f then return failure end
    local col
    for _, c in ipairs(f.data.columns) do if c.field == field then col = c break end end
    if not col then
        return I.fail("invalid", ("This panel has no %s column any more. Refresh it."):format(field), { reason = "no such column" })
    end
    if not col.editable then
        return I.fail("readonly", ("%s cannot be changed here."):format(col.label), { reason = "read-only column" })
    end
    local row = rowOf(f.data, key)
    if not row then
        return I.fail("invalid", "That row is not there any more. Refresh the panel.", { reason = "no such row" })
    end
    if col.options then
        local known = false
        for _, o in ipairs(col.options) do if I.equal(o.value, clean) then known = true break end end
        if not known then
            return I.fail("invalid", ("That is not one of the choices for %s."):format(col.label), { reason = "not an option" })
        end
    end
    if col.type == "number" then
        local n = tonumber(clean)
        if not finiteNumber(n) then
            return I.fail("invalid", ("%s must be a number."):format(col.label), { reason = "not a number" })
        end
        if col.min and n < col.min then
            return I.fail("invalid", ("%s must be at least %s."):format(col.label, tostring(col.min)), { reason = "below min" })
        end
        if col.max and n > col.max then
            return I.fail("invalid", ("%s must be at most %s."):format(col.label, tostring(col.max)), { reason = "above max" })
        end
        clean = n
    end
    local who = I.who(src)
    local old = row.cells[field]
    local okCall, a, b = callPanel(f.folder, f.p, "write",
        { key = row.key, field = field, value = clean, parent = f.parent }, who)
    if not okCall then
        panelWarn(f.folder, f.p, tostring(a))
        return I.fail("panel_error", ("%s could not save it: %s"):format(f.folder, diagText(tostring(a), 300)))
    end
    local done, message = outcome(a, b)
    if not done then
        return I.fail("refused", message or "The script refused the change.", { reason = message or "refused" })
    end
    if I.logPanel then
        local okLog, err = pcall(I.logPanel, f.s, f.p, f.folder, row.key, field, old, clean, who)
        if not okLog then Util.Error("hub: could not log a panel change: %s", tostring(err)) end
    end
    return I.ok({ ok = true, message = message or ("%s saved."):format(col.label), key = row.key, field = field, old = old, value = clean })
end

--- An action's input, checked against its declared fields. Unknown fields
--- are dropped. Returns the clean input, or nil, message.
function I.cleanActionInput(fields, input)
    input = type(input) == "table" and input or {}
    local out = {}
    for _, f in ipairs(fields or {}) do
        local v = input[f.field]
        if v == nil or v == "" then
            if f.required then return nil, ("%s is needed."):format(f.label) end
        elseif f.picker == "coords" then
            if type(v) ~= "table" or not finiteNumber(v.x) or not finiteNumber(v.y) or not finiteNumber(v.z) then
                return nil, ("%s needs a position (x, y, z)."):format(f.label)
            end
            out[f.field] = { x = v.x, y = v.y, z = v.z, heading = finiteNumber(v.heading) and v.heading or nil }
        elseif f.type == "number" then
            local n = tonumber(v)
            if not finiteNumber(n) then return nil, ("%s must be a number."):format(f.label) end
            if f.min and n < f.min then return nil, ("%s must be at least %s."):format(f.label, tostring(f.min)) end
            if f.max and n > f.max then return nil, ("%s must be at most %s."):format(f.label, tostring(f.max)) end
            out[f.field] = n
        elseif f.type == "boolean" then
            out[f.field] = v == true
        else
            if not scalar(v) then return nil, ("%s must be plain text."):format(f.label) end
            if type(v) == "string" and #v > PANEL_STRING then return nil, ("%s is too long."):format(f.label) end
            out[f.field] = v
        end
        if f.options and out[f.field] ~= nil then
            local known = false
            for _, o in ipairs(f.options) do if I.equal(o.value, out[f.field]) then known = true break end end
            if not known then return nil, ("That is not one of the choices for %s."):format(f.label) end
        end
    end
    return out
end

--- The confirmation a danger action needs (§10.5): typed = the row's key,
--- or CONFIRM for the panel; simple = yes. Returns nil when it is given.
function I.confirmRefusal(action, key, given)
    if action.confirm == "typed" then
        local want = action.scope == "panel" and "CONFIRM" or tostring(key)
        if type(given) ~= "string" and type(given) ~= "number" or tostring(given) ~= want then
            return ("Type %s to confirm %s."):format(want, action.label)
        end
    elseif action.confirm == "simple" or action.danger then
        if given == nil or given == false or given == "" then
            return ("%s needs confirming."):format(action.label)
        end
    end
    return nil
end

--- panelAction(id, panelId, actionId, key, input, ctx): run one action
--- through the script. ctx = { parent, root, confirm }.
function Hub.PanelAction(src, id, panelId, actionId, key, input, ctx)
    if not idLike(actionId, 40) then
        return I.fail("invalid", "Which action? The request named none.", { reason = "no action" })
    end
    ctx = type(ctx) == "table" and ctx or {}
    local f, failure = freshRead(src, id, panelId, ctx)
    if not f then return failure end
    local action
    for _, a in ipairs(f.data.actions) do if a.id == actionId then action = a break end end
    if not action then
        return I.fail("invalid", "This panel does not offer that any more. Refresh it.", { reason = "no such action" })
    end
    local row
    if action.scope == "row" then
        if not rowKeyLike(key) then
            return I.fail("invalid", ("%s needs a row."):format(action.label), { reason = "no row key" })
        end
        row = rowOf(f.data, key)
        if not row then
            return I.fail("invalid", "That row is not there any more. Refresh the panel.", { reason = "no such row" })
        end
        if row.actions then
            local offered = false
            for _, a in ipairs(row.actions) do if a == actionId then offered = true break end end
            if not offered then
                return I.fail("invalid", ("%s is not offered on that row."):format(action.label), { reason = "not offered on row" })
            end
        end
        key = row.key
    else
        key = nil
    end
    local refusal = I.confirmRefusal(action, key, ctx.confirm)
    if refusal then return I.fail("confirm", refusal, { reason = "confirmation" }) end
    local clean, bad = I.cleanActionInput(action.input, input)
    if not clean then return I.fail("invalid", bad, { reason = bad }) end
    local who = I.who(src)
    local okCall, a, b = callPanel(f.folder, f.p, "action",
        { action = actionId, key = key, input = clean, parent = f.parent }, who)
    if not okCall then
        panelWarn(f.folder, f.p, tostring(a))
        return I.fail("panel_error", ("%s could not do it: %s"):format(f.folder, diagText(tostring(a), 300)))
    end
    local done, message = outcome(a, b)
    if not done then
        return I.fail("refused", message or "The script refused it.", { reason = message or "refused" })
    end
    if I.logPanelAction then
        local okLog, err = pcall(I.logPanelAction, f.s, f.p, f.folder, action, key, clean, who)
        if not okLog then Util.Error("hub: could not log a panel action: %s", tostring(err)) end
    end
    return I.ok({ ok = true, message = message or ("%s done."):format(action.label), action = actionId, key = key })
end

-- ---------------------------------------------------------------------------
-- Container contents (§10.5): generic, from the storage verbs
-- ---------------------------------------------------------------------------

local CONTAINER_MAX_QTY = 100000

--- The container a panel row names, found by reading the panel again (the
--- id never comes from the page). Returns f (freshRead) + containerId, or nil + failure.
local function containerOf(src, id, panelId, key, ctx)
    if not rowKeyLike(key) then return nil, nil, I.fail("invalid", "Which row? The request named none.", { reason = "no row key" }) end
    local f, failure = freshRead(src, id, panelId, ctx)
    if not f then return nil, nil, failure end
    local row = rowOf(f.data, key)
    if not row then return nil, nil, I.fail("invalid", "That row is not there any more. Refresh the panel.", { reason = "no such row" }) end
    if not row.container then
        return nil, nil, I.fail("invalid", "That row has no container.", { reason = "no container" })
    end
    return f, row.container
end

--- Items and weapons in a container, as panel rows. Returns rows, items, weapons, err.
local function containerRows(cid)
    local ok, items, err = PoggyCore.Do("storage.items", { id = cid })
    if not ok then return nil, nil, nil, err or "unsupported" end
    local reg = Hub.ItemRegistry()
    local rows, list, seenName = {}, {}, {}
    for _, it in ipairs(type(items) == "table" and items or {}) do
        if type(it) == "table" and type(it.name) == "string" and it.name ~= "" then
            seenName[it.name] = (seenName[it.name] or 0) + 1
            local key = it.name .. "#" .. seenName[it.name]
            local amount = tonumber(it.amount) or 0
            local known = reg.byName and reg.byName[it.name]
            list[key] = { name = it.name, amount = amount, meta = it.meta }
            local hasMeta = type(it.meta) == "table" and next(it.meta) ~= nil
            rows[#rows + 1] = {
                key = key,
                cells = {
                    item = it.name,
                    label = diagText(it.label, 80) or (known and known.label) or it.name,
                    amount = amount,
                    kind = hasMeta and "Item (with metadata)" or "Item",
                },
                actions = { "remove" },
            }
        end
    end
    local weapons = 0
    local okW, ws = PoggyCore.Do("storage.weapons", { id = cid })
    if okW and type(ws) == "table" then
        for i, w in ipairs(ws) do
            if type(w) == "table" and type(w.name) == "string" then
                weapons = weapons + 1
                rows[#rows + 1] = {
                    key = "weapon:" .. tostring(w.id or i),
                    cells = {
                        item = w.name, label = diagText(w.label, 80) or w.name, amount = 1,
                        kind = "Weapon", serial = diagText(w.serial, 60),
                    },
                    actions = {},
                }
            end
        end
    end
    return rows, list, weapons
end

local CONTAINER_ACTIONS = {
    { id = "add", label = "Add item", icon = "plus", scope = "panel", input = {
        { field = "item", label = "Item", picker = "item", required = true },
        { field = "amount", label = "Amount", type = "number", min = 1, max = CONTAINER_MAX_QTY, step = 1, default = 1, required = true },
    } },
    { id = "remove", label = "Remove", icon = "minus", scope = "row", input = {
        { field = "amount", label = "Amount", type = "number", min = 1, max = CONTAINER_MAX_QTY, step = 1, default = 1, required = true },
    } },
    { id = "empty", label = "Empty", icon = "trash", scope = "panel", danger = true, confirm = "typed",
      tooltip = "Removes every item. Weapons stay: the hub cannot take them out." },
}

--- container(id, panelId, key, ctx): what is inside the container a row names.
function Hub.Container(src, id, panelId, key, ctx)
    local f, cid, failure = containerOf(src, id, panelId, key, ctx)
    if not f then return failure end
    local rows, _, weapons, err = containerRows(cid)
    if not rows then
        return I.fail("unsupported", ("The contents of %s cannot be read on this server (%s)."):format(cid, tostring(err)))
    end
    local columns = {
        { field = "item", label = "Item", picker = "item" },
        { field = "label", label = "Name" },
        { field = "amount", label = "Amount" },
        { field = "kind", label = "Kind" },
    }
    if weapons > 0 then columns[#columns + 1] = { field = "serial", label = "Serial" } end
    return I.ok({
        id = "contents", container = cid, key = key, label = "Contents", itemLabel = "item",
        available = true, columns = columns, rows = rows, actions = I.copy(CONTAINER_ACTIONS),
        total = #rows, readAt = I.now(),
        note = weapons > 0 and "Weapons are listed but cannot be taken out from here." or nil,
    })
end

--- containerAction(id, panelId, key, actionId, rowKey, input, ctx): Add item,
--- Remove, Empty, through the storage verbs. ctx = { parent, root, confirm }.
function Hub.ContainerAction(src, id, panelId, key, actionId, rowKey, input, ctx)
    ctx = type(ctx) == "table" and ctx or {}
    local action
    for _, a in ipairs(CONTAINER_ACTIONS) do if a.id == actionId then action = a break end end
    if not action then return I.fail("invalid", "Contents cannot do that.", { reason = "no such action" }) end
    local f, cid, failure = containerOf(src, id, panelId, key, ctx)
    if not f then return failure end
    local refusal = I.confirmRefusal(action, rowKey, ctx.confirm)
    if refusal then return I.fail("confirm", refusal, { reason = "confirmation" }) end
    local clean, bad = I.cleanActionInput(action.input, input)
    if not clean then return I.fail("invalid", bad, { reason = bad }) end
    local who = I.who(src)
    local _, list = containerRows(cid)
    list = list or {}
    local file = "container:" .. cid

    if actionId == "add" then
        local name = clean.item
        if type(name) ~= "string" or name == "" or #name > 100 then return I.fail("invalid", "Choose an item.", { reason = "no item" }) end
        local qty = math.floor(clean.amount)
        if qty < 1 then return I.fail("invalid", "Add at least one.", { reason = "amount" }) end
        local reg = Hub.ItemRegistry()
        if reg.byName and not reg.byName[name] then
            local near = reg.byLower and reg.byLower[I.lower(name)]
            return I.fail("invalid", near and ("There is no item called %s. Did you mean %s?"):format(name, near.name)
                or ("There is no item called %s."):format(name), { reason = "no such item", suggest = near and near.name or nil })
        end
        local before = 0
        for _, x in pairs(list) do if x.name == name then before = before + x.amount end end
        local charId
        if tonumber(src) and tonumber(src) > 0 then
            local okC, cidv = PoggyCore.Do("char.id", { src = src })
            if okC then charId = cidv end
        end
        local ok, _, err = PoggyCore.Do("storage.addItem", { id = cid, item = name, qty = qty, charId = charId })
        if not ok then
            return I.fail("refused", ("%s could not be added (%s). A full container, or an item the inventory refuses."):format(name, tostring(err)),
                { reason = tostring(err) })
        end
        if I.logContainer then pcall(I.logContainer, f.s, cid, name, "add", before, before + qty, who) end
        return I.ok({ ok = true, message = ("Added %d × %s to %s."):format(qty, name, cid) })
    end

    if actionId == "remove" then
        local x = type(rowKey) == "string" and list[rowKey] or nil
        if not x then
            return I.fail("invalid", "That item is not in the container any more. Refresh it.", { reason = "no such item" })
        end
        local qty = math.floor(clean.amount)
        if qty < 1 then return I.fail("invalid", "Remove at least one.", { reason = "amount" }) end
        if qty > x.amount then
            return I.fail("invalid", ("There are only %d × %s."):format(x.amount, x.name), { reason = "not that many" })
        end
        local ok, _, err = PoggyCore.Do("storage.removeItem", { id = cid, item = x.name, qty = qty, meta = x.meta })
        if not ok then
            return I.fail("refused", ("%s could not be removed (%s)."):format(x.name, tostring(err)), { reason = tostring(err) })
        end
        if I.logContainer then pcall(I.logContainer, f.s, cid, x.name, "remove", x.amount, x.amount - qty, who) end
        return I.ok({ ok = true, message = ("Removed %d × %s from %s."):format(qty, x.name, cid) })
    end

    -- empty
    local removed, failed, before = 0, {}, {}
    local keys = {}
    for k in pairs(list) do keys[#keys + 1] = k end
    table.sort(keys)
    for _, k in ipairs(keys) do
        local x = list[k]
        before[x.name] = (before[x.name] or 0) + x.amount
        if x.amount > 0 then
            local ok = PoggyCore.Do("storage.removeItem", { id = cid, item = x.name, qty = x.amount, meta = x.meta })
            if ok then removed = removed + x.amount else failed[#failed + 1] = x.name end
        end
    end
    if I.logContainer then pcall(I.logContainer, f.s, cid, "*", "empty", before, {}, who) end
    if #failed > 0 then
        return I.fail("partial", ("Removed %d item(s); these could not be removed: %s."):format(removed, table.concat(failed, ", ")),
            { reason = "partial" })
    end
    return I.ok({ ok = true, message = removed > 0 and ("Emptied %s: %d item(s) removed."):format(cid, removed)
        or ("%s was already empty."):format(cid) })
end

--- For History: what a logged panel or container change was, or nil when the
--- row is neither. Panel: file = "panel:<resource>:<panelId>", path =
--- "<key>.<field>" (an action: path = key, op = "action:<id>"). Container:
--- file = "container:<id>", path = item name ("*" when emptied).
function I.panelLogInfo(b, file, path, op)
    if type(file) ~= "string" then return nil end
    local cid = file:match("^container:(.+)$")
    if cid then
        return { container = cid, canUndo = false,
            label = ("Contents of %s · %s"):format(cid, path == "*" and "everything" or tostring(path)) }
    end
    local pid = file:match("^panel:[^:]*:(.+)$")
    if not pid then return nil end
    local label, declared = I.readable(pid), false
    for _, p in ipairs(b and b.panels or {}) do
        if p.id == pid then label, declared = p.label, true break end
    end
    if type(op) == "string" and op:sub(1, 7) == "action:" then
        return { panel = pid, canUndo = false,
            label = label .. (path ~= nil and path ~= "" and (" · " .. tostring(path)) or "") .. " · " .. I.readable(op:sub(8)) }
    end
    local key, field = tostring(path or ""):match("^(.*)%.([^%.]+)$")
    return {
        panel = pid, panelKey = key or path, panelField = field,
        canUndo = declared and field ~= nil,
        label = label .. " · " .. tostring(key or path) .. (field and (" · " .. field) or ""),
    }
end

-- ---------------------------------------------------------------------------
-- Cards, commands, search index
-- ---------------------------------------------------------------------------

--- The commands hub.json lists, with the live name when it comes from the config.
function I.commands(b)
    local out = {}
    if type(b.meta.commands) ~= "table" then return out end
    for _, c in ipairs(b.meta.commands) do
        if type(c) == "table" then
            local cmd = I.copy(c)
            if type(c.configPath) == "string" then
                local node = b.byPath[c.configPath]
                if node and (type(node.value) == "string" or type(node.value) == "number") then
                    cmd.declared = c.command
                    local live = tostring(node.value)
                    cmd.command = live:sub(1, 1) == "/" and live or ("/" .. live)
                end
            end
            out[#out + 1] = cmd
        end
    end
    return out
end

function I.lockView(id)
    local l = Hub.IsLocked(id)
    if not l then return nil end
    return { name = l.name, since = l.since }
end

--- §6.2 card.
function Hub.Card(b)
    local s, meta = b.script, b.meta
    local lock = I.lockView(s.id)
    return {
        id = s.id,
        folder = s.folder,
        label = type(meta.label) == "string" and meta.label or I.folderLabel(s.folder),
        tagline = meta.tagline,
        category = type(meta.category) == "string" and meta.category
            or (s.folder == SELF and "Framework" or "Utility"),
        description = meta.description,
        version = s.version,
        running = s.running,
        icon = I.icon(s),
        lock = lock and { holder = lock } or nil,
        settingsCount = b.settingsCount,
        changedCount = b.changedCount,
        defaultsKnown = b.defaultsKnown,
        self = s.folder == SELF or nil,
    }
end

local function clip(text, n)
    if type(text) ~= "string" then return nil end
    if #text <= n then return text end
    -- n counts bytes; step back off any UTF-8 continuation byte (0x80-0xBF)
    -- so a clipped tooltip never ends in half a character.
    local cut = n - 1
    while cut > 0 do
        local b = text:byte(cut + 1)
        if not b or b < 0x80 or b > 0xBF then break end
        cut = cut - 1
    end
    return text:sub(1, cut) .. "…"
end

--- §6.2 searchEntry: every setting and list, and every command. Each points
--- where the rail (§8.1) shows it: tab, and section (a setting on a tab
--- page) or item + row (inside a collection kept as nodes).
function I.searchEntries(b, into)
    local id = b.script.id
    for _, n in ipairs(b.nodes) do
        if n.kind ~= "group" and not n.meta.hidden and not n.cell then
            into[#into + 1] = {
                id = id, path = n.path, label = n.meta.label, tooltip = clip(n.meta.tooltip, 160),
                tab = n.tab, section = n.section, item = n.item, row = n.row,
                kind = n.collection and "collection" or n.kind,
            }
        end
    end
    -- §10: data panels by name (their rows are live data, not indexed).
    for _, p in ipairs(b.panels or {}) do
        into[#into + 1] = {
            id = id, panel = p.id, path = "panel:" .. p.id, label = p.label,
            tooltip = clip(p.tooltip, 160), tab = p.tab, kind = "panel",
        }
    end
    for _, c in ipairs(I.commands(b)) do
        if c.command then
            into[#into + 1] = { id = id, command = c.command, description = clip(c.description, 160), kind = "command" }
        end
    end
end

-- ---------------------------------------------------------------------------
-- Locks (§4.3)
-- ---------------------------------------------------------------------------

local locks = {}   -- id -> { src, name, identifier, since, lastActive, warned, label }

--- The holder of a script's lock ({ src, name, identifier, since }) or nil.
--- The updater calls this before it writes a script's files.
function Hub.IsLocked(id)
    local l = locks[id]
    if not l then return nil end
    return { src = l.src, name = l.name, identifier = l.identifier, since = l.since }
end

function I.holdsLock(src, id)
    local l = locks[id]
    return l ~= nil and l.src == tonumber(src)
end

--- Any edit, save or heartbeat counts as activity.
function I.touch(src, id)
    local l = locks[id]
    if l and l.src == tonumber(src) then
        l.lastActive = I.now()
        l.warned = false
        return true
    end
    return false
end

local function labelOf(id)
    local s = Hub.Find(id)
    if not s then return id end
    local meta = Hub.Meta(s.folder)
    return type(meta.label) == "string" and meta.label or I.folderLabel(s.folder)
end

--- Take a script's lock if it is free (or already ours). Returns true, or
--- false plus the holder.
function I.acquire(src, id)
    src = tonumber(src)
    local l = locks[id]
    if l and l.src ~= src then return false, { name = l.name, since = l.since } end
    if l then
        I.touch(src, id)
        return true
    end
    local who = I.who(src)
    locks[id] = {
        src = src, name = who.name, identifier = who.identifier,
        since = I.now(), lastActive = I.now(), warned = false,
    }
    print(("[poggy] %s is editing %s in /poggy. Do not edit its config files by hand until they finish.")
        :format(who.name, labelOf(id)))
    I.pushViewers("lock", { id = id, holder = { name = who.name, since = locks[id].since } })
    return true
end

--- Release a lock. reason: 'release' | 'closed' | 'idle' | 'dropped' | 'console'.
--- The holder is told for idle (released) and console (kicked); everyone with
--- the hub open sees the lock go.
function I.release(id, reason, by)
    local l = locks[id]
    if not l then return false end
    locks[id] = nil
    if reason == "idle" then
        I.push(l.src, "released", { id = id, reason = "idle" })
    elseif reason == "console" then
        I.push(l.src, "kicked", { id = id, by = by or "the server console" })
    end
    I.pushViewers("lock", { id = id })
    Util.Debug("hub: %s released %s (%s)", tostring(l.name), id, tostring(reason))
    return true
end

--- Release every lock a player holds.
function I.releaseAll(src, reason)
    src = tonumber(src)
    local ids = {}
    for id, l in pairs(locks) do
        if l.src == src then ids[#ids + 1] = id end
    end
    for _, id in ipairs(ids) do I.release(id, reason) end
    return #ids
end

--- Take a lock from its holder. The holder is told who took it.
function I.takeover(src, id)
    src = tonumber(src)
    local l = locks[id]
    local who = I.who(src)
    if l and l.src ~= src then
        I.push(l.src, "kicked", { id = id, by = who.name })
        locks[id] = nil
    end
    return I.acquire(src, id)
end

--- The idle check (every few seconds on its own thread; a test calls it).
function Hub.CheckIdle()
    local c = cfg()
    local idle = math.max(1, tonumber(c.IdleMinutes) or 10) * 60
    local warn = math.max(0, tonumber(c.IdleWarnMinutes) or 8) * 60
    if warn >= idle then warn = math.max(0, idle - 60) end
    local t = I.now()
    local expired = {}
    for id, l in pairs(locks) do
        local quiet = t - l.lastActive
        if quiet >= idle then
            expired[#expired + 1] = id
        elseif quiet >= warn and not l.warned then
            l.warned = true
            I.push(l.src, "idleWarning", { id = id, secondsLeft = math.max(0, idle - quiet) })
        end
    end
    for _, id in ipairs(expired) do I.release(id, "idle") end
end

CreateThread(function()
    while true do
        Wait(5000)
        local ok, err = pcall(Hub.CheckIdle)
        if not ok then Util.Error("hub idle check: %s", tostring(err)) end
    end
end)

AddEventHandler("playerDropped", function()
    local src = tonumber(source)
    viewers[src] = nil
    I.releaseAll(src, "dropped")
    if Hub.ForgetSession then Hub.ForgetSession(src) end
end)

--- The hub closed: every lock that player holds goes.
local function closed(src)
    src = tonumber(src)
    if not src then return end
    viewers[src] = nil
    I.releaseAll(src, "closed")
    if Hub.ForgetSession then Hub.ForgetSession(src) end
end

RegisterNetEvent("poggy_core:hub:closed", function()
    closed(source)
end)

-- ---------------------------------------------------------------------------
-- Sessions: the fingerprints each player was shown (§4.4 step 1)
-- ---------------------------------------------------------------------------

local sessions = {}   -- src -> { id -> { file -> fingerprint } }

function I.remember(src, id, files)
    src = tonumber(src)
    if not src then return end
    sessions[src] = sessions[src] or {}
    local fps = {}
    for _, f in ipairs(files) do fps[f.file] = f.fingerprint end
    sessions[src][id] = fps
end

function I.remembered(src, id)
    local s = sessions[tonumber(src)]
    return s and s[id] or nil
end

function Hub.ForgetSession(src)
    sessions[tonumber(src)] = nil
end

-- ---------------------------------------------------------------------------
-- Script state pushes
-- ---------------------------------------------------------------------------

--- A script registered (its bridge is ready): tell open hubs it runs, and
--- rebuild its database mirror (sv_hub_save.lua).
function Hub.OnRegister(entry)
    if type(entry) ~= "table" or not entry.id then return end
    I.forgetBuilt(entry.folder)
    I.pushViewers("scriptState", { id = entry.id, running = true })
    if Hub.QueueMirror then Hub.QueueMirror(entry.id) end
end

AddEventHandler("onResourceStop", function(resource)
    if resource == SELF then return end
    local Id = PoggyCore.Identity
    if not Id or next(viewers) == nil then return end
    local declared = Id.DeclaredId(resource)
    local text = LoadResourceFile(resource, "fxmanifest.lua")
    if declared or (text and text:find("@poggy_core/template/poggy.lua", 1, true)) then
        I.pushViewers("scriptState", { id = declared or resource, running = false })
    end
end)

-- ---------------------------------------------------------------------------
-- Answers
-- ---------------------------------------------------------------------------

function I.ok(value) return { ok = true, value = value } end

--- { ok = false, err, message, ...extra }
function I.fail(err, message, extra)
    local out = extra and I.copy(extra) or {}
    out.ok, out.err, out.message = false, err, message
    return out
end

local MESSAGES = {
    missing = "That script is not on this server any more.",
    denied  = "You do not have permission to use the settings hub.",
}

function I.script(id)
    local s = Hub.Find(id)
    if not s then return nil, I.fail("missing", MESSAGES.missing) end
    return s
end

-- ---------------------------------------------------------------------------
-- open (§6.2): every card and the search index
-- ---------------------------------------------------------------------------

function Hub.Open(src)
    if tonumber(src) and tonumber(src) > 0 then viewers[tonumber(src)] = true end
    local scripts, index, used = {}, {}, {}
    for _, s in ipairs(Hub.Scripts(true)) do
        local okBuild, b = pcall(Hub.Build, s)
        if okBuild and b then
            local card = Hub.Card(b)
            scripts[#scripts + 1] = card
            used[card.category] = true
            I.searchEntries(b, index)
            if not b.defaultsKnown and Hub.WarmDefaults then Hub.WarmDefaults(s) end
        else
            Util.Error("hub: %s could not be read: %s", s.id, tostring(b))
        end
    end
    table.sort(scripts, function(a, b) return tostring(a.label):lower() < tostring(b.label):lower() end)
    local categories, seen = {}, {}
    for _, c in ipairs(CATEGORIES) do
        if used[c] then categories[#categories + 1] = c; seen[c] = true end
    end
    local extra = {}
    for c in pairs(used) do if not seen[c] then extra[#extra + 1] = c end end
    table.sort(extra)
    for _, c in ipairs(extra) do categories[#categories + 1] = c end

    return {
        me = { name = I.who(src).name },
        canTakeover = Hub.CanTakeover(src),
        command = cfg().Command or "poggy",
        coreVersion = PoggyCore.VERSION,
        categories = categories,
        scripts = scripts,
        index = index,
    }
end

-- ---------------------------------------------------------------------------
-- script (§6.2): one script, in full. Opening it asks for its lock.
-- ---------------------------------------------------------------------------

function Hub.Script(src, id)
    local s, failure = I.script(id)
    if not s then return failure end
    -- The shipped config, when it can be had quickly; otherwise the next open has it.
    if Hub.DefaultsFor and not Hub.DefaultsFor(s) and Hub.WarmDefaults then Hub.WarmDefaults(s, 2500) end
    local b = Hub.Build(s)
    local mine, holder = I.acquire(src, s.id)
    local l = locks[s.id]
    I.remember(src, s.id, b.files)
    -- §9: item checks and the script's own report. Neither may stop the
    -- script opening: a failure is logged and the answer goes without it.
    local itemCheck, diagnostics, diagnosticsNote
    if b.meta.itemCheck == true then
        local okCheck, res = pcall(I.itemCheck, b)
        if okCheck then itemCheck = res else Util.Error("hub: item check for %s failed: %s", s.id, tostring(res)) end
    end
    if b.meta.diagnostics ~= nil then
        local okDiag, res, note = pcall(I.diagnostics, s, b)
        if okDiag then diagnostics, diagnosticsNote = res, note
        else Util.Error("hub: diagnostics for %s failed: %s", s.id, tostring(res)) end
    end
    -- §10: data panels, each with whether its owning script can answer now.
    local panels = {}
    for _, p in ipairs(b.panels or {}) do
        local okState, folder, available, reason = pcall(I.panelState, s, p)
        if okState then
            panels[#panels + 1] = I.panelView(p, folder, available, reason)
        else
            panels[#panels + 1] = I.panelView(p, p.resource or s.folder, false, "The hub could not check this panel's script.")
        end
    end
    return I.ok({
        panels = panels,                     -- §10
        itemCheck = itemCheck,               -- §9.1
        diagnostics = diagnostics,           -- §9.2
        diagnosticsNote = diagnosticsNote,   -- why there are none, when hub.json asks for them
        card = Hub.Card(b),
        meta = b.meta,
        tabs = b.tabs,
        nav = b.nav,     -- §8.1: tabs with their sections and items, for the rail
        refs = b.refs,   -- §8.5: reference targets, their keys and who uses them
        files = b.files,
        nodes = b.nodes,
        order = b.order,
        lock = {
            mine = mine and true or false,
            holder = mine and { name = l.name, since = l.since } or holder,
        },
        readme = I.readme(s, b.meta),
        help = I.helpPages(s, b.meta),
        commands = I.commands(b),
        defaultsKnown = b.defaultsKnown,
        restartNote = s.folder == SELF
            and "Restart poggy_core by hand; it restarts every Poggy script." or nil,
    })
end

-- ---------------------------------------------------------------------------
-- Callbacks: poggy_core:hub:<call>
-- ---------------------------------------------------------------------------

local Api = PoggyCore.BuildCallbackApi(SELF)

--- Register one call. Every call re-checks the permission (nothing from the
--- client is trusted) and answers { ok, value | err, message }.
--- A refused /poggy says why in the console: which checks failed, and which
--- identifiers the player connected with (an `add_ace identifier.discord:...`
--- line does nothing when Discord was not running as they joined).
function I.explainDenied(src)
    local types = {}
    local discord
    for i = 0, (GetNumPlayerIdentifiers(src) or 0) - 1 do
        local id = GetPlayerIdentifier(src, i)
        if type(id) == "string" then
            local kind = id:match("^([^:]+):")
            if kind then types[#types + 1] = kind end
            if kind == "discord" then discord = id end
        end
    end
    -- Separate "the cfg line is not loaded" from "the player does not inherit
    -- the identifier principal" from "the check itself is wrong".
    local function raw(fn, ...)
        local ok, v = pcall(fn, ...)
        if not ok then return "error: " .. tostring(v) end
        return tostring(v)
    end
    local probes = {
        "player(num)=" .. raw(IsPlayerAceAllowed, src, "poggy.settings"),
        "player(str)=" .. raw(IsPlayerAceAllowed, tostring(src), "poggy.settings"),
        "player command=" .. raw(IsPlayerAceAllowed, src, "command"),
    }
    if discord then
        probes[#probes + 1] = "principal " .. discord .. "=" .. raw(IsPrincipalAceAllowed, "identifier." .. discord, "poggy.settings")
    end
    probes[#probes + 1] = "principal group.admin=" .. raw(IsPrincipalAceAllowed, "group.admin", "poggy.settings")
    Util.Warn("ACE probes for %s: %s", GetPlayerName(src) or "?", table.concat(probes, "  "))

    local okAdmin, isAdmin = PoggyCore.Do("perms.isAdmin", { src = src })
    local okGroups, groups = PoggyCore.Do("perms.groups", { src = src })
    groups = (okGroups == true and type(groups) == "table" and #groups > 0) and table.concat(groups, ", ") or "none found"
    Util.Warn("%s (player %d) was refused /poggy: ACE poggy.settings = %s, framework admin = %s "
        .. "(framework groups: %s; admin means one of admin, superadmin, god, owner, headadmin, developer). "
        .. "Connected with: %s%s. Grant it in the server.cfg the server actually runs: "
        .. "add_ace identifier.<type>:<id> poggy.settings allow (or add_ace group.admin poggy.settings allow), then restart the server.",
        GetPlayerName(src) or "?", src, tostring(ace(src, "poggy.settings")),
        tostring(okAdmin == true and isAdmin == true), groups, table.concat(types, ", "),
        discord and (" (" .. discord .. ")") or " (no discord identifier)")
end

local function register(call, fn, open)
    Api.Register("poggy_core:hub:" .. call, function(src, ...)
        src = tonumber(src)
        if not open and not Hub.Allowed(src) then
            if call == "open" then pcall(I.explainDenied, src) end
            return I.fail("denied", MESSAGES.denied)
        end
        local okCall, res = pcall(fn, src, ...)
        if not okCall then
            Util.Error("hub call '%s' failed: %s", call, tostring(res))
            return I.fail("error", "Something went wrong on the server: " .. tostring(res))
        end
        return res
    end)
end
Hub.Register = register

--- One console line per /poggy: proof the server answered, how big the answer
--- is, and whether it survives msgpack (what the network uses) intact.
function I.traceOpen(src, answer)
    local okPack, packed = pcall(msgpack.pack, { n = 1, answer })
    local size = (okPack and type(packed) == "string") and #packed or -1
    local trip
    if not okPack then
        trip = "PACK FAILED: " .. tostring(packed)
    else
        local okUnpack, back = pcall(msgpack.unpack, packed)
        if okUnpack and type(back) == "table" and back.n == 1 and type(back[1]) == "table" and back[1].ok == true then
            trip = "ok"
        else
            trip = "UNPACK FAILED: " .. tostring(back)
        end
    end
    local scripts = type(answer.value) == "table" and type(answer.value.scripts) == "table" and #answer.value.scripts or 0
    print(("%s/%s opened for %s: %d scripts, answer %d bytes, msgpack round trip %s")
        :format(PoggyCore.PREFIX, tostring((PoggyCoreConfig.Hub or {}).Command or "poggy"),
            GetPlayerName(src) or tostring(src), scripts, size, trip))
end

register("open", function(src)
    local answer = I.ok(Hub.Open(src))
    -- Diagnostics only: never let the trace stop the hub opening.
    if type(msgpack) == "table" then pcall(I.traceOpen, src, answer) end
    return answer
end)

--- A client that could not open the hub says why (see client/cl_hub.lua).
local lastDiag = {}
RegisterNetEvent("poggy_core:hub:diag", function(report)
    local src = source
    local now = GetGameTimer()
    if lastDiag[src] and now - lastDiag[src] < 2000 then return end
    lastDiag[src] = now
    local okJson, text = pcall(json.encode, report)
    text = okJson and tostring(text) or "(unreadable report)"
    if #text > 1500 then text = text:sub(1, 1500) .. "…" end
    Util.Warn("%s could not open the hub. Client report: %s", GetPlayerName(src) or tostring(src), text)
end)

register("script", function(src, id) return Hub.Script(src, id) end)

register("lock", function(src, id)
    local s, failure = I.script(id)
    if not s then return failure end
    local mine, holder = I.acquire(src, s.id)
    if mine then return I.ok({ mine = true }) end
    return I.ok({ mine = false, holder = holder })
end)

register("release", function(src, id)
    if type(id) == "string" and I.holdsLock(src, id) then I.release(id, "release") end
    return I.ok(true)
end)

register("takeover", function(src, id)
    local s, failure = I.script(id)
    if not s then return failure end
    if not Hub.CanTakeover(src) then
        return I.fail("denied", "Taking over needs the ACE poggy.settings.takeover.")
    end
    I.takeover(src, s.id)
    return I.ok({ mine = true })
end)

register("heartbeat", function(src, id)
    if type(id) == "string" then I.touch(src, id) end
    return I.ok(true)
end)

-- The hub closed. Answered for anyone: releasing your own locks needs no permission.
register("closed", function(src)
    closed(src)
    return I.ok(true)
end, true)

register("items", function(src, search)
    local ok, list, err = PoggyCore.Do("inv.items", {
        search = type(search) == "string" and search:sub(1, 64) or nil, limit = 50,
    })
    if not ok then return I.fail(err or "unsupported", "The item list is not available on this server.") end
    local out = {}
    for i, it in ipairs(list or {}) do
        if i > 50 then break end
        out[#out + 1] = { name = it.name, label = it.label, image = it.image }
    end
    return I.ok(out)
end)

-- §9.1: the whole item registry, for type-ahead. fresh = read it again now.
register("itemList", function(src, fresh)
    return I.ok(I.itemList(fresh == true))
end)

-- §9.1: names typed since the script was loaded, checked like the script
-- answer's itemCheck.values. id (optional): the script whose icon location
-- applies (hub.json itemIcons); without it, the inventory's.
register("itemCheck", function(src, names, id)
    if type(names) ~= "table" then return I.fail("invalid", "itemCheck takes a list of item names.") end
    local list = {}
    for _, name in ipairs(names) do
        if type(name) == "string" and name ~= "" and #name <= 100 then list[#list + 1] = name end
        if #list >= MAX_CHECK_NAMES then break end
    end
    local b = nil
    if type(id) == "string" then
        local s = Hub.Find(id)
        if s then b = Hub.Build(s) end
    end
    local values = I.itemValues(list, I.iconLocation(b))
    return I.ok({ values = values })
end)

register("jobs", function()
    local ok, list, err = PoggyCore.Do("jobs.list", {})
    if not ok then return I.fail(err or "unsupported", "The job list is not available on this server.") end
    return I.ok(list or {})
end)

-- §10: data panels. No edit lock (each write is one change the owning script
-- validates), but register() still requires the permission to edit.
-- ctx (optional, last) = { parent, root, confirm }: a drill-down panel's row
-- and declared root, and a danger action's confirmation (§10.5).
register("panel", function(src, id, panelId, ctx) return Hub.Panel(src, id, panelId, ctx) end)
register("panelWrite", function(src, id, panelId, key, field, value, ctx)
    return Hub.PanelWrite(src, id, panelId, key, field, value, ctx)
end)
register("panelAction", function(src, id, panelId, actionId, key, input, ctx)
    return Hub.PanelAction(src, id, panelId, actionId, key, input, ctx)
end)
-- §10.5: a row's container, built by poggy_core from the storage verbs.
register("container", function(src, id, panelId, key, ctx) return Hub.Container(src, id, panelId, key, ctx) end)
register("containerAction", function(src, id, panelId, key, actionId, rowKey, input, ctx)
    return Hub.ContainerAction(src, id, panelId, key, actionId, rowKey, input, ctx)
end)

-- save, restart, start, history, undo, roles, saveRoles: sv_hub_save.lua.

-- ---------------------------------------------------------------------------
-- Console: poggycore settings ... (§4.10), routed from sv_commands.lua
-- ---------------------------------------------------------------------------

local function short(value, n)
    local text = type(value) == "string" and ("%q"):format(value) or I.encode(value) or tostring(value)
    return clip(text, n or 80)
end

--- args: the words after `settings`. say(msg) prints. src: 0 for the console.
function Hub.Command(src, args, say)
    local sub = (args[1] or ""):lower()
    local console = tonumber(src) == 0

    if sub == "" or sub == "list" then
        local list = Hub.Scripts(true)
        say(("%d Poggy script(s)"):format(#list))
        for _, s in ipairs(list) do
            local b = Hub.Build(s)
            local l = Hub.IsLocked(s.id)
            local fps = {}
            for _, f in ipairs(b.files) do
                fps[#fps + 1] = ("%s %s"):format(f.file, f.err and "^1(unreadable)^7" or ("^9" .. tostring(f.fingerprint):sub(1, 12) .. "^7"))
            end
            say(("  %-24s %-8s %s%s"):format(s.id, s.running and "^2running^7" or "^3stopped^7",
                l and ("^3edited by " .. l.name .. "^7  ") or "",
                #fps > 0 and table.concat(fps, ", ") or "^9no config files^7"))
        end
        say("^9poggycore settings show <id> | set <id> <path> <json value> | unlock <id>^7")
        return
    end

    if sub == "show" then
        local s = Hub.Find(args[2] or "")
        if not s then say(("^1❌ no Poggy script called %s.^7"):format(tostring(args[2]))) return end
        local b = Hub.Build(s)
        say(("%s v%s — %d setting(s)%s"):format(s.id, tostring(s.version), b.settingsCount,
            b.defaultsKnown and (", " .. b.changedCount .. " changed from the shipped config") or ""))
        for _, f in ipairs(b.files) do
            if f.err then say(("  ^1%s: %s^7"):format(f.file, f.err)) end
        end
        for _, n in ipairs(b.nodes) do
            if n.kind ~= "group" then
                say(("  %-44s %s  ^9%s:%s%s%s^7"):format(n.path, short(n.value, 70), n.file, tostring(n.line),
                    n.changed and "  changed" or "", n.editable and "" or "  read-only"))
            end
        end
        return
    end

    if sub == "set" then
        if not console then say("settings set runs from the server console only.") return end
        local id, path = args[2], args[3]
        if not id or not path or not args[4] then
            say("usage: poggycore settings set <id> <path> <json value>   e.g. set poggy_markets Config.Debug true")
            return
        end
        local raw = table.concat(args, " ", 4)
        local value, derr = I.decode(raw)
        if value == nil then
            -- Plain text that is not JSON is taken as a string; broken JSON is refused.
            if raw:match('^%s*[%[{"]') or raw == "null" then
                say(("^1❌ that value is not usable JSON: %s^7"):format(tostring(derr or "null")))
                return
            end
            value = raw
        end
        local res = Hub.ConsoleSet(id, path, value)
        if res.ok then
            say(("^2✅ %s %s = %s^7%s"):format(id, path, short(value, 80),
                res.value.restart and "  ^9— restart the script to use it^7" or ""))
        else
            say(("^1❌ %s: %s^7"):format(tostring(res.err), tostring(res.message or res.reason or "")))
            if res.files then say("   ^9changed on disk: " .. table.concat(res.files, ", ") .. "^7") end
        end
        return
    end

    if sub == "unlock" then
        if not console then say("settings unlock runs from the server console only.") return end
        local s = Hub.Find(args[2] or "")
        local id = s and s.id or args[2]
        local l = id and Hub.IsLocked(id)
        if not l then say(("%s is not being edited."):format(tostring(id))) return end
        I.release(id, "console")
        say(("^2✅ released %s (was %s)^7"):format(id, l.name))
        return
    end

    say("usage: poggycore settings [show <id> | set <id> <path> <json value> | unlock <id>]")
end
