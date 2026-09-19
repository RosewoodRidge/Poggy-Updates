--[[
    poggy_core — the settings hub: saving, history, roles, restarts (0.18.0).

    The second half of PoggyCore.Hub (sv_hub.lua is the first). Spec:
    docs/reference/poggy-hub-spec.md §4.4 to §4.9.

    Saving (§4.4), in this order, all or nothing:
      1. the player holds the script's lock, and the script is started (a
         stopped script has no bridge to write with: err "stopped")
      2. every file the changes touch is read again; if its fingerprint is not
         the one the player was shown, the whole save is refused: err "stale"
         and the files that changed on disk
      3. the changes are applied one by one through PoggyCore.SettingsModel,
         each checked first: the setting exists, is editable, and the value
         has the setting's type and meets hub.json's limits (min, max,
         options, length, webhook and colour formats). Any failure refuses the
         whole save: err "invalid", with the path and the reason
      4. each file's current text is copied to
         poggy_core/update_backups/<folder>__<yyyymmdd-hhmmss>__settings__<file, / as _>
      5. each file is written through the script's own PoggyWriteOwnFile
         (poggy_core's own files: SaveResourceFile) and read back. If one
         fails, the files already written are put back as they were
      6. one poggy_settings_log row per change; the mirror row is rebuilt
      7. the answer: the new fingerprints, applied = n, and restart = true
         when a changed setting is not marked "live" in hub.json

    The database is a mirror and a history. Without oxmysql the hub still
    edits files; it only has no history and no cached defaults.

    Defaults (§4.6) are the config files the running version shipped with,
    from the update feed (Updates.FetchShipped), cached in memory and in
    poggy_settings.defaults. Unavailable (offline, a development build that was
    never published): every default is null and the page hides the badges.

    Restarts (§4.5) use `ensure <folder>` when poggy_core has command.ensure,
    else StopResource and StartResource. They work on a development server
    too: the updater's rule against restarting there is about installing
    escrowed builds, not about settings. poggy_core never restarts itself.

    Roles (§4.7): PoggyCoreConfig.Roles in poggy_core's config.lua. A strings
    setting in any script is linked to a role by `-- poggy:role <name>` on its
    line. Saving the roles rewrites that block of config.lua, then every linked
    setting in every script, then restarts the scripts that changed. Scripts
    someone else is editing are skipped and named.
]]

PoggyCore = PoggyCore or {}

local Hub  = PoggyCore.Hub
local I    = Hub.Internal
local Util = PoggyCore.Util
local SELF = GetCurrentResourceName()

local MAX_STRING = 4096
-- Validation helpers, defined below and used by each other.
local samePath, effectiveKind, inOptions, fieldSegs, fieldMeta, checkFields
local groupValue, valueIn, subMetaFor, nestedKind, refSourcesTo
local MAX_CHANGES = 500
local PAGE = 50

local function M() return PoggyCore.SettingsModel end
local function cfg() return PoggyCoreConfig.Hub or {} end

-- ---------------------------------------------------------------------------
-- Database (§4.9)
--
-- Created at start the way sv_sql.lua creates poggy_migrations: asked of
-- information_schema first, and CREATE TABLE IF NOT EXISTS sent only when the
-- table is missing, so a normal start sends no DDL at all.
-- ---------------------------------------------------------------------------

local TABLES = {
    poggy_settings = [[
CREATE TABLE IF NOT EXISTS `poggy_settings` (
  `poggy_id`    VARCHAR(64)  NOT NULL,
  `folder`      VARCHAR(128) NOT NULL,
  `version`     VARCHAR(32)  NULL,
  `fingerprint` VARCHAR(64)  NULL,
  `toggles`     LONGTEXT     NULL,
  `inputs`      LONGTEXT     NULL,
  `lists`       LONGTEXT     NULL,
  `defaults`    LONGTEXT     NULL,
  `updated_at`  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`poggy_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
    poggy_settings_log = [[
CREATE TABLE IF NOT EXISTS `poggy_settings_log` (
  `id`              BIGINT       NOT NULL AUTO_INCREMENT,
  `poggy_id`        VARCHAR(64)  NOT NULL,
  `file`            VARCHAR(128) NOT NULL,
  `path`            VARCHAR(255) NOT NULL,
  `op`              VARCHAR(16)  NOT NULL,
  `old_value`       LONGTEXT     NULL,
  `new_value`       LONGTEXT     NULL,
  `changed_by`      VARCHAR(64)  NULL,
  `changed_by_name` VARCHAR(64)  NULL,
  `changed_at`      TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  INDEX `idx_script_time` (`poggy_id`, `changed_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]],
}
local TABLE_ORDER = { "poggy_settings", "poggy_settings_log" }

local DB = { ready = false }
Hub.DB = DB

--- Replaceable for tests: DB.query(sql, params) -> rows | nil, err;
--- DB.exec(sql, params) -> affected | nil, err. Both wait: call from a thread.
DB.query = function(sql, params)
    local Sql = PoggyCore.Sql
    if not Sql or not Sql.Db then return nil, "no database" end
    return Sql.Db.query(sql, params)
end
DB.exec = function(sql, params)
    local Sql = PoggyCore.Sql
    if not Sql or not Sql.Db then return nil, "no database" end
    return Sql.Db.exec(sql, params)
end

local function cut(s, n)
    if s == nil then return nil end
    s = tostring(s)
    return #s > n and s:sub(1, n) or s
end

--- Create the tables when missing and prune old history. Returns true when
--- the database can be used.
function Hub.InitDatabase()
    for _, name in ipairs(TABLE_ORDER) do
        local rows, err = DB.query(
            "SELECT 1 AS found FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ? LIMIT 1",
            { name })
        if not rows then
            Util.Warn("settings hub: could not read the database (%s); history and cached defaults are off.", tostring(err))
            return false
        end
        if #rows == 0 then
            local ok, e = DB.exec(TABLES[name])
            if not ok then
                Util.Error("settings hub: could not create %s: %s", name, tostring(e))
                return false
            end
            Util.Log("^2✅ settings hub: created table %s^7", name)
        end
    end
    local days = math.floor(tonumber(cfg().HistoryDays) or 90)
    if days > 0 then
        local removed = DB.exec(("DELETE FROM `poggy_settings_log` WHERE `changed_at` < (NOW() - INTERVAL %d DAY)"):format(days))
        if tonumber(removed) and tonumber(removed) > 0 then
            Util.Log("settings hub: removed %d history row(s) older than %d days", tonumber(removed), days)
        end
    end
    DB.ready = true
    return true
end

-- ---------------------------------------------------------------------------
-- Mirror (poggy_settings): rebuilt from the files, never read back
-- ---------------------------------------------------------------------------

local mirrorQueue, mirrorWorker = {}, false

--- The mirror row of one script, from its files as they are now.
function Hub.RebuildMirror(script)
    local Model = M()
    if not DB.ready or not Model or not Model.Mirror then return false end
    local b = Hub.Build(script)
    local toggles, inputs, lists, fps = {}, {}, {}, {}
    for _, f in ipairs(b.files) do
        local l = b.loaded[f.file]
        if l and l.model then
            local okM, m = pcall(Model.Mirror, l.model)
            if okM and type(m) == "table" then
                for k, v in pairs(m.toggles or {}) do toggles[k] = v end
                for k, v in pairs(m.inputs or {}) do inputs[k] = v end
                for k, v in pairs(m.lists or {}) do lists[k] = v end
            end
        end
        fps[#fps + 1] = f.file .. "=" .. tostring(f.fingerprint)
    end
    local fp = Model.Fingerprint(table.concat(fps, "\n"))
    local version = cut(script.version, 32)
    -- Cached defaults belong to one version; an update makes them stale.
    DB.exec("UPDATE `poggy_settings` SET `defaults` = NULL WHERE `poggy_id` = ? AND (`version` IS NULL OR `version` <> ?)",
        { script.id, version or "" })
    local ok, err = DB.exec([[
INSERT INTO `poggy_settings` (`poggy_id`, `folder`, `version`, `fingerprint`, `toggles`, `inputs`, `lists`)
VALUES (?, ?, ?, ?, ?, ?, ?)
ON DUPLICATE KEY UPDATE `folder` = VALUES(`folder`), `version` = VALUES(`version`), `fingerprint` = VALUES(`fingerprint`),
  `toggles` = VALUES(`toggles`), `inputs` = VALUES(`inputs`), `lists` = VALUES(`lists`)]],
        { script.id, cut(script.folder, 128), version, cut(fp, 64),
          I.encode(toggles) or "{}", I.encode(inputs) or "{}", I.encode(lists) or "{}" })
    if not ok then Util.Debug("hub: mirror of %s not written: %s", script.id, tostring(err)) end
    return ok ~= nil
end

--- Rebuild a script's mirror soon (debounced, one at a time).
function Hub.QueueMirror(id)
    if type(id) ~= "string" then return end
    mirrorQueue[id] = true
    if mirrorWorker then return end
    mirrorWorker = true
    CreateThread(function()
        Wait(2000)
        while next(mirrorQueue) do
            if not DB.ready then Wait(5000) else
                local id2 = next(mirrorQueue)
                mirrorQueue[id2] = nil
                local s = Hub.Find(id2)
                if s then
                    local ok, err = pcall(Hub.RebuildMirror, s)
                    if not ok then Util.Debug("hub: mirror of %s failed: %s", id2, tostring(err)) end
                end
                Wait(50)
            end
        end
        mirrorWorker = false
    end)
end

-- ---------------------------------------------------------------------------
-- History (poggy_settings_log)
-- ---------------------------------------------------------------------------

local function logRows(script, rows, who)
    if not DB.ready or #rows == 0 then return end
    for _, r in ipairs(rows) do
        DB.exec([[
INSERT INTO `poggy_settings_log` (`poggy_id`, `file`, `path`, `op`, `old_value`, `new_value`, `changed_by`, `changed_by_name`)
VALUES (?, ?, ?, ?, ?, ?, ?, ?)]], {
            script.id, cut(r.file, 128), cut(r.path, 255), cut(r.op, 16),
            r.old ~= nil and I.encode(r.old) or nil, r.new ~= nil and I.encode(r.new) or nil,
            cut(who.identifier, 64), cut(who.name, 64),
        })
    end
end

--- §10: one data panel write, so History shows it beside the config changes.
--- file = "panel:<resource>:<panelId>", path = "<key>.<field>".
function I.logPanel(script, panel, folder, key, field, old, new, who)
    logRows(script, { {
        file = "panel:" .. tostring(folder) .. ":" .. panel.id,
        path = tostring(key) .. "." .. tostring(field),
        op = "set", old = old, new = new,
    } }, who or I.who(0))
end

--- §10.5: a panel action. op = "action:<id>" (cut to the column), path = the
--- row key ("" for the panel); new = { action, input } keeps the whole id.
function I.logPanelAction(script, panel, folder, action, key, input, who)
    logRows(script, { {
        file = "panel:" .. tostring(folder) .. ":" .. panel.id,
        path = key ~= nil and tostring(key) or "",
        op = "action:" .. action.id,
        new = { action = action.id, input = input },
    } }, who or I.who(0))
end

--- §10.5: a change to a container's contents. file = "container:<id>",
--- path = the item ("*" when emptied), op = add | remove | empty.
function I.logContainer(script, containerId, item, op, old, new, who)
    logRows(script, { {
        file = "container:" .. tostring(containerId),
        path = tostring(item), op = op, old = old, new = new,
    } }, who or I.who(0))
end

local function unixSeconds(v)
    local n = tonumber(v)
    if n then return n > 1e11 and math.floor(n / 1000) or math.floor(n) end
    return v
end

--- One page (1-based, 50 rows) of a script's history, newest first.
function Hub.History(id, page)
    page = math.max(1, math.floor(tonumber(page) or 1))
    if not DB.ready then return { rows = {}, more = false, unavailable = true } end
    local rows = DB.query(("SELECT `id`, `file`, `path`, `op`, `old_value`, `new_value`, `changed_by_name`, `changed_at`"
        .. " FROM `poggy_settings_log` WHERE `poggy_id` = ? ORDER BY `id` DESC LIMIT %d OFFSET %d")
        :format(PAGE + 1, (page - 1) * PAGE), { id }) or {}
    local out, built = {}, false
    for i, r in ipairs(rows) do
        if i > PAGE then break end
        local row = { id = tonumber(r.id), file = r.file, path = r.path, op = r.op,
            by = r.changed_by_name, at = unixSeconds(r.changed_at) }
        -- A stored false is a value; only a missing one is nil.
        if r.old_value ~= nil then row.old = I.decode(r.old_value) end
        if r.new_value ~= nil then row.new = I.decode(r.new_value) end
        -- §10: a data panel write reads "Shop jobs · valentine_general · job".
        if type(r.file) == "string" and (r.file:sub(1, 6) == "panel:" or r.file:sub(1, 10) == "container:") and I.panelLogInfo then
            if built == false then
                local s = Hub.Find(id)
                local okB, b = false, nil
                if s then okB, b = pcall(Hub.Build, s) end
                built = okB and b or nil
            end
            local info = I.panelLogInfo(built, r.file, r.path, r.op)
            if info then
                row.panel, row.container, row.label = info.panel, info.container, info.label
                row.panelKey, row.panelField = info.panelKey, info.panelField
                row.canUndo = info.canUndo
            end
        end
        out[#out + 1] = row
    end
    return { rows = out, more = #rows > PAGE }
end

-- ---------------------------------------------------------------------------
-- Defaults (§4.6): the shipped config of the running version
-- ---------------------------------------------------------------------------

local shipped = {}          -- id -> { version, files = { rel = text } | nil, failedAt, loading, stamp, models }
local RETRY_SECONDS = 600
local fetchQueue, fetchWorker = {}, false

--- The shipped files of a script's running version, or nil when not known (yet).
function Hub.DefaultsFor(script)
    local e = shipped[script.id]
    if e and e.version == script.version and e.files then return e end
    return nil
end

--- The shipped copy of one config file, parsed. nil when it did not ship.
--- opts: the live file's options (Hub.LoadFile), so the shipped file is read
--- with the same kinds (a collection is a map in both, and defaults line up).
function Hub.ShippedModel(script, rel, dictKeys, opts)
    local e = Hub.DefaultsFor(script)
    if not e or type(e.files[rel]) ~= "string" then return nil end
    e.models = e.models or {}
    e.optsOf = e.optsOf or {}
    if e.models[rel] == nil or e.optsOf[rel] ~= opts then
        local Model = M()
        local ok, model = false, nil
        local o = opts or (I.modelOpts(Hub.Meta(script.folder), dictKeys))
        if Model then ok, model = pcall(Model.Parse, e.files[rel], o) end
        e.models[rel] = (ok and model) or false
        e.optsOf[rel] = opts
    end
    return e.models[rel] or nil
end

local function loadDefaults(script)
    local e = shipped[script.id]
    local rels = {}
    for _, f in ipairs(Hub.ConfigFiles(script, Hub.Meta(script.folder))) do rels[#rels + 1] = f.file end

    local files = nil
    if DB.ready then
        local rows = DB.query("SELECT `version`, `defaults` FROM `poggy_settings` WHERE `poggy_id` = ? LIMIT 1", { script.id })
        local r = rows and rows[1]
        if r and r.version == script.version and type(r.defaults) == "string" then
            local decoded = I.decode(r.defaults)
            if type(decoded) == "table" and next(decoded) then files = decoded end
        end
    end

    if not files then
        local U = PoggyCore.Updates
        if U and U.FetchShipped and script.version and #rels > 0 then
            local got, why = U.FetchShipped(script.id, script.version, rels)
            if got and next(got) then
                files = got
                if DB.ready then
                    DB.exec([[
INSERT INTO `poggy_settings` (`poggy_id`, `folder`, `version`, `defaults`) VALUES (?, ?, ?, ?)
ON DUPLICATE KEY UPDATE `defaults` = VALUES(`defaults`), `version` = VALUES(`version`)]],
                        { script.id, cut(script.folder, 128), cut(script.version, 32), I.encode(got) })
                end
            else
                Util.Debug("hub: no shipped config for %s v%s: %s", script.id, tostring(script.version), tostring(why))
            end
        end
    end

    if files then
        shipped[script.id] = { version = script.version, files = files, stamp = I.now() }
        I.forgetBuilt(script.folder)
    else
        shipped[script.id] = { version = script.version, failedAt = I.now() }
    end
    if e then e.loading = false end
end

--- Start fetching a script's defaults when they are not known; with waitMs,
--- wait that long for them.
function Hub.WarmDefaults(script, waitMs)
    local e = shipped[script.id]
    if e and e.version == script.version then
        if e.files then return true end
        if e.failedAt and I.now() - e.failedAt < RETRY_SECONDS then return false end
    end
    if not fetchQueue[script.id] then
        shipped[script.id] = { version = script.version, loading = true }
        fetchQueue[script.id] = script
        if not fetchWorker then
            fetchWorker = true
            CreateThread(function()
                while next(fetchQueue) do
                    local id, s = next(fetchQueue)
                    local ok, err = pcall(loadDefaults, s)
                    if not ok then
                        Util.Debug("hub: defaults for %s failed: %s", id, tostring(err))
                        shipped[id] = { version = s.version, failedAt = I.now() }
                    end
                    fetchQueue[id] = nil
                end
                fetchWorker = false
            end)
        end
    end
    if waitMs and waitMs > 0 and I.CanWait() then
        local deadline = GetGameTimer() + waitMs
        while fetchQueue[script.id] and GetGameTimer() < deadline do Wait(100) end
    end
    return Hub.DefaultsFor(script) ~= nil
end

function I.CanWait()
    return Util.CanYield()
end

-- ---------------------------------------------------------------------------
-- Validation (§4.4 step 3)
-- ---------------------------------------------------------------------------

local VEC = { vec2 = { "x", "y" }, vec3 = { "x", "y", "z" }, vec4 = { "x", "y", "z", "w" } }
local VEC_TYPE = { vec2 = "vector2", vec3 = "vector3", vec4 = "vector4" }

--- The §3.1 type of an encoded value.
function I.typeOf(v)
    if type(v) == "table" then
        if VEC_TYPE[v.__type] then return VEC_TYPE[v.__type] end
        if v.__type == "hash" then return "hash" end
        return "table"
    end
    return type(v)
end

local function finite(n) return type(n) == "number" and n == n and n ~= math.huge and n ~= -math.huge end

--- Only plain data: finite numbers, strings under the cap, valid vectors and
--- hashes, no deeper than 16, no more than 20000 values. Returns nil or a reason.
local function checkData(v, depth, budget)
    budget.n = budget.n + 1
    if budget.n > 20000 then return "too much data" end
    if depth > 16 then return "nested too deeply" end
    local t = type(v)
    if t == "number" then
        if not finite(v) then return "not a usable number" end
    elseif t == "string" then
        if #v > MAX_STRING then return ("longer than %d characters"):format(MAX_STRING) end
    elseif t == "boolean" then
        -- fine
    elseif t == "table" then
        if VEC[v.__type] then
            for _, k in ipairs(VEC[v.__type]) do
                if not finite(v[k]) then return "a position needs numbers for " .. table.concat(VEC[v.__type], ", ") end
            end
        elseif v.__type == "hash" then
            if type(v.name) ~= "string" or not v.name:match("^[%w_%-%.]+$") or #v.name > 128 then
                return "not a valid key or hash name"
            end
        elseif v.__type ~= nil then
            return "unknown value type " .. tostring(v.__type)
        else
            for k, child in pairs(v) do
                if type(k) ~= "string" and math.type(k) ~= "integer" then return "a table key must be text or a whole number" end
                if type(k) == "string" and #k > 256 then return "a key is too long" end
                local why = checkData(child, depth + 1, budget)
                if why then return why end
            end
        end
    else
        return "values cannot be " .. t
    end
    return nil
end

local function isArray(v)
    if type(v) ~= "table" or v.__type then return false end
    local n = 0
    for k in pairs(v) do
        if math.type(k) ~= "integer" or k < 1 then return false end
        n = n + 1
    end
    return n == #v
end

--- hub.json limits on one value: min, max, options, maxLength, pickers.
local function checkLimits(v, m)
    if type(m) ~= "table" then return nil end
    if type(v) == "number" then
        if tonumber(m.min) and v < tonumber(m.min) then return ("below the minimum (%s)"):format(tostring(m.min)) end
        if tonumber(m.max) and v > tonumber(m.max) then return ("above the maximum (%s)"):format(tostring(m.max)) end
    elseif type(v) == "string" then
        local cap = math.min(tonumber(m.maxLength) or MAX_STRING, MAX_STRING)
        if #v > cap then return ("longer than %d characters"):format(cap) end
        if m.picker == "webhook" and v ~= "" then
            local okHook = v:match("^https://discord%.com/api/webhooks/%d+/[%w_%-]+$")
                or v:match("^https://discordapp%.com/api/webhooks/%d+/[%w_%-]+$")
                or v:match("^https://ptb%.discord%.com/api/webhooks/%d+/[%w_%-]+$")
                or v:match("^https://canary%.discord%.com/api/webhooks/%d+/[%w_%-]+$")
            if not okHook then return "not a Discord webhook address (https://discord.com/api/webhooks/...)" end
        elseif m.picker == "color" and v ~= "" then
            if not (v:match("^#%x%x%x$") or v:match("^#%x%x%x%x%x%x$") or v:match("^#%x%x%x%x%x%x%x%x$")) then
                return "not a colour (#rrggbb)"
            end
        end
    end
    if type(m.options) == "table" and #m.options > 0 then
        local function allowed(x) return inOptions(x, m) end
        -- A whole value that is itself an option (a list option) is fine;
        -- otherwise every entry of a list must be one.
        if allowed(v) then
            -- fine
        elseif isArray(v) then
            for _, x in ipairs(v) do
                if not allowed(x) then return ("%s is not one of the allowed choices"):format(tostring(x)) end
            end
        elseif not allowed(v) then
            return "not one of the allowed choices"
        end
    end
    return nil
end

--- A list field key as segments ("loot[].count" -> { "loot", "[]", "count" })
--- and the field meta for a row-relative position: sv_hub.lua's, shared with
--- the organisation (§8.4) so both read field keys the same way.
fieldSegs = I.fieldSegs
fieldMeta = I.fieldMeta

--- Check every value in a row against the list's field meta. segs is the
--- row-relative position so far. Returns nil or reason, where.
checkFields = function(fields, v, segs)
    if type(fields) ~= "table" then return nil end
    if #segs > 0 then
        local m = fieldMeta(fields, segs)
        if m then
            local why = checkLimits(v, m)
            if why then return why, table.concat(segs, ".") end
        end
    end
    if type(v) == "table" and v.__type == nil and #segs < 16 then
        for k, child in pairs(v) do
            if k ~= "__int_keys" then
                local next_ = {}
                for i2 = 1, #segs do next_[i2] = segs[i2] end
                next_[#next_ + 1] = tostring(k)
                local why, where = checkFields(fields, child, next_)
                if why then return why, where end
            end
        end
    end
    return nil
end

--- The value must fit the node it replaces (§3 kinds and types). kind is
--- the effective kind (hub.json may force list or map).
local function checkShape(node, kind, v)
    if kind == "value" then
        local want, got = node.type, I.typeOf(v)
        if want and want ~= "table" and got ~= want then
            return ("expects %s, got %s"):format(want, got)
        end
    elseif kind == "strings" then
        if not isArray(v) then return "expects a list of names" end
        for _, x in ipairs(v) do
            if type(x) ~= "string" and type(x) ~= "number" then return "expects a list of names or numbers" end
        end
    elseif kind == "list" then
        if not isArray(v) then return "expects a list of rows" end
        for _, row in ipairs(v) do
            if type(row) ~= "table" or row.__type then return "every row must be a table" end
        end
    elseif kind == "map" then
        -- Rows keyed by name; a row may be a table or a single value (a map
        -- of item -> category is a map of strings).
        if type(v) ~= "table" or v.__type or (next(v) ~= nil and isArray(v)) then return "expects rows keyed by name" end
    elseif kind == "group" then
        return "a group is edited one setting at a time"
    elseif kind == "readonly" then
        return "this setting can only be edited in the file"
    end
    return nil
end

--- Where a path sits in a model. Returns nil for a path that cannot be read, else
---   exact      the node at the path, or nil
---   chain      every model node at or above the path, nearest first: { node, meta }
---   list       the nearest list or map (hub.json may force the kind) at or above it
---   listMeta, listKind
---   whole      true when the path IS that list
---   rowKey     the row's index or key when the path is inside a row
---   rowSegs    the row-relative field segments below that row ({} for the row itself)
---   rowPath    the row's own path
local function locate(model, b, path)
    local segs = I.splitPath(path)
    if not segs then return nil end
    local out = { chain = {} }
    local paths = { path }
    for _, anc in ipairs(I.ancestors(path)) do paths[#paths + 1] = anc end
    for i, p in ipairs(paths) do
        local n = I.nodeAt(model.nodes, p)
        if n then
            local bn = I.nodeAt(b.byPath, n.path)
            local m = bn and bn.meta or nil
            out.chain[#out.chain + 1] = { node = n, meta = m }
            if i == 1 then out.exact = n end
            if not out.list then
                local k = effectiveKind(n, m or {})
                if k == "list" or k == "map" then
                    out.list, out.listMeta, out.listKind, out.listDepth = n, m or {}, k, #segs - i + 1
                end
            end
        end
    end
    if out.list then
        if out.listDepth == #segs then
            out.whole = true
        else
            local row = segs[out.listDepth + 1]
            out.rowKey = row.key
            out.rowPath = I.canon(out.list.path) .. (row.index and ("[%d]"):format(row.key)
                or (tostring(row.key):match("^[%a_][%w_]*$") and ("." .. row.key) or ("[%q]"):format(row.key)))
            out.rowSegs = {}
            for j = out.listDepth + 2, #segs do out.rowSegs[#out.rowSegs + 1] = tostring(segs[j].key) end
        end
    end
    return out
end

--- hub.json readonly or hidden: never changed from here, whatever the page sends.
local function protected(m)
    return type(m) == "table" and (m.readonly == true or m.hidden == true)
end

local PROTECTED_REASON = "read-only here: edit it in the file"

--- A node or meta on the way down to the path that forbids editing it.
local function chainRefusal(loc)
    for _, c in ipairs(loc.chain) do
        if c.node.kind == "readonly" or c.node.editable == false then return c.node.reason or PROTECTED_REASON end
        if protected(c.meta) then return PROTECTED_REASON end
    end
    return nil
end

--- A node below the path (a row the model marks read-only, say) that a
--- replacement of the whole value would rewrite.
local function descendantRefusal(model, b, path)
    local root = I.canon(path)
    for p, n in pairs(model.nodes) do
        local c = I.canon(p)
        if #c > #root and c:sub(1, #root) == root then
            local nextChar = c:sub(#root + 1, #root + 1)
            if nextChar == "." or nextChar == "[" then
                local bn = I.nodeAt(b.byPath, n.path)
                if n.kind == "readonly" or n.editable == false or protected(bn and bn.meta) then
                    return ("%s cannot be changed here (edit it in the file)"):format(n.path)
                end
            end
        end
    end
    return nil
end

--- A row field on the way down to rowSegs that is readonly or hidden.
local function fieldRefusal(fields, rowSegs)
    for k = 1, #rowSegs do
        local prefix = {}
        for j = 1, k do prefix[j] = rowSegs[j] end
        if protected(fieldMeta(fields, prefix)) then
            return ("%s is read-only here: edit it in the file"):format(table.concat(prefix, "."))
        end
    end
    return nil
end

local function plainTable(v) return type(v) == "table" and v.__type == nil end

--- Every position in `new` (or `old`) that a readonly or hidden field covers
--- must hold what it held before. Returns the first position that differs.
local function protectedDiff(fields, old, new, segs)
    if type(fields) ~= "table" then return nil end
    if #segs > 0 and protected(fieldMeta(fields, segs)) then
        if I.equal(old, new) then return nil end
        return table.concat(segs, ".")
    end
    if #segs >= 16 then return nil end
    local keys = {}
    if plainTable(new) then for k in pairs(new) do keys[k] = true end end
    if plainTable(old) then for k in pairs(old) do keys[k] = true end end
    for k in pairs(keys) do
        if k ~= "__int_keys" then
            local o, nw = nil, nil
            if plainTable(old) then o = old[k] end
            if plainTable(new) then nw = new[k] end
            local s2 = {}
            for j = 1, #segs do s2[j] = segs[j] end
            s2[#s2 + 1] = tostring(k)
            local where = protectedDiff(fields, o, nw, s2)
            if where then return where end
        end
    end
    return nil
end

--- protectedDiff row by row, for a whole list or map.
local function protectedRows(fields, old, new)
    if type(fields) ~= "table" then return nil end
    local keys = {}
    if plainTable(new) then for k in pairs(new) do keys[k] = true end end
    if plainTable(old) then for k in pairs(old) do keys[k] = true end end
    for k in pairs(keys) do
        if k ~= "__int_keys" then
            local o, nw = nil, nil
            if plainTable(old) then o = old[k] end
            if plainTable(new) then nw = new[k] end
            local where = protectedDiff(fields, o, nw, {})
            if where then return where end
        end
    end
    return nil
end

-- Every path hub.json marks readonly or hidden, whether or not the model has
-- a node there (a map row, a field of a list row): { canon, sub }. sub = a
-- ".*" rule, which covers everything below its path.
local protectedCache = setmetatable({}, { __mode = "k" })

local function protectedEntries(meta)
    if type(meta) ~= "table" then return {} end
    local c = protectedCache[meta]
    if c then return c end
    c = {}
    for key, m in pairs(type(meta.settings) == "table" and meta.settings or {}) do
        if type(key) == "string" and protected(m) then
            if key:sub(-2) == ".*" then
                c[#c + 1] = { canon = I.canon(key:sub(1, -3)), sub = true }
            else
                c[#c + 1] = { canon = I.canon(key), sub = false }
            end
        end
    end
    for key, m in pairs(type(meta.lists) == "table" and meta.lists or {}) do
        if type(key) == "string" and protected(m) then c[#c + 1] = { canon = I.canon(key), sub = false } end
    end
    protectedCache[meta] = c
    return c
end

--- Does entry e protect the canonical path cp itself?
local function covers(e, cp)
    if e.sub then return I.under(cp, e.canon) end
    return cp == e.canon or I.under(cp, e.canon)
end

--- Does entry e protect something strictly inside cp?
local function coversInside(e, cp)
    return I.under(e.canon, cp) or (e.sub and e.canon == cp)
end

--- Walk a value by segments ({ key, index }) with int/text key fallbacks.
local function walk(v, segs)
    for _, sg in ipairs(segs) do
        if type(v) ~= "table" or v.__type ~= nil then return nil end
        local k = sg.key
        local nextV = v[k]
        if nextV == nil and math.type(k) == "integer" then nextV = v[tostring(k)] end
        if nextV == nil and type(k) == "string" and math.tointeger(tonumber(k)) then nextV = v[math.tointeger(tonumber(k))] end
        v = nextV
    end
    return v
end

--- A change at path, replacing old with new: refused when hub.json protects
--- the path or anything above it, or when it changes something protected
--- inside it. Returns nil or the reason.
local function protectedRefusal(meta, path, old, new, replaces)
    local cp = I.canon(path)
    for _, e in ipairs(protectedEntries(meta)) do
        if covers(e, cp) then return PROTECTED_REASON end
        if replaces and coversInside(e, cp) then
            local rel = e.canon:sub(#cp + 1)
            local segs = rel ~= "" and I.splitPath("R" .. rel) or { {} }
            if segs then
                table.remove(segs, 1)
                local a, b2 = walk(old, segs), walk(new, segs)
                if not I.equal(a, b2) then
                    return ("%s is read-only here: edit it in the file"):format(e.canon)
                end
            end
        end
    end
    return nil
end

--- Is anything strictly inside a list protected by position? Inserting,
--- removing or moving rows would shift those positions onto other rows.
local function protectedByPosition(meta, listPath)
    local cp = I.canon(listPath)
    for _, e in ipairs(protectedEntries(meta)) do
        if I.under(e.canon, cp) and e.canon:sub(#cp + 1, #cp + 1) == "[" and e.canon:sub(#cp + 2, #cp + 2):match("%d") then
            return true
        end
    end
    return false
end

--- The value at row-relative segments inside a row.
local function valueAt(row, segs)
    local v = row
    for _, sg in ipairs(segs) do
        if not plainTable(v) then return nil end
        local nextV = v[sg]
        if nextV == nil and tonumber(sg) then nextV = v[math.tointeger(tonumber(sg)) or tonumber(sg)] end
        v = nextV
    end
    return v
end

--- The current value of a row (by index or key).
local function rowValue(list, key)
    if not plainTable(list) then return nil end
    local v = list[key]
    if v == nil and type(key) == "string" and tonumber(key) then v = list[math.tointeger(tonumber(key)) or tonumber(key)] end
    -- An int-keyed map travels with string keys ("__int_keys", spec 3.1).
    if v == nil and math.type(key) == "integer" then v = list[tostring(key)] end
    return v
end

local function toInt(x)
    local n = tonumber(x)
    return n and math.tointeger(n) or nil
end

local function countRows(v)
    if type(v) ~= "table" then return 0 end
    if isArray(v) then return #v end
    local n = 0
    for _ in pairs(v) do n = n + 1 end
    return n
end

--- Two spellings of one path?
samePath = function(a, b)
    return a == b or I.canon(a) == I.canon(b)
end

--- The kind a node is edited as. hub.json's kinds reach the model through its
--- options, so a table the model could read as rows already has that kind;
--- between list and map hub.json's word is taken, and a group stays a group
--- (the model could not read it as rows). An empty {} that a collection uses
--- as a sublist (its meta says list, §8.4) is an empty list of rows, not an
--- empty list of names.
effectiveKind = function(node, meta)
    local forced = type(meta) == "table" and meta.kind or nil
    if (forced == "list" or forced == "map") and (node.kind == "list" or node.kind == "map") then
        return forced
    end
    if forced == "list" and node.kind == "strings" and type(node.value) == "table" and next(node.value) == nil then
        return "list"
    end
    return node.kind
end

--- Is the value one of hub.json's options for this setting?
inOptions = function(v, meta)
    if type(meta) ~= "table" or type(meta.options) ~= "table" or #meta.options == 0 then return false end
    for _, o in ipairs(meta.options) do
        local ov = o
        if type(o) == "table" and (o.value ~= nil or o.label ~= nil) then ov = o.value end
        if I.equal(ov, v) then return true end
    end
    return false
end

-- ---------------------------------------------------------------------------
-- Lists inside rows, collections, references (§8.4, §8.5)
-- ---------------------------------------------------------------------------

--- The value of a group of settings in a model, put together from its nodes
--- (a collection row kept as nodes). nil when part of it is code.
function groupValue(model, path)
    local node = I.nodeAt(model.nodes, path)
    if not node then return nil end
    local kids = {}
    for _, p in ipairs(model.order or {}) do
        local n = model.nodes[p]
        if n and n.parent then
            local l = kids[n.parent]
            if not l then l = {}; kids[n.parent] = l end
            l[#l + 1] = n
        end
    end
    local function build(n, depth)
        if depth > 16 or n.kind == "readonly" then return nil end
        if n.kind ~= "group" then return n.value end
        local out = {}
        for _, c in ipairs(kids[n.path] or {}) do
            local v = build(c, depth + 1)
            if v == nil then return nil end
            out[c.key] = v
        end
        return out
    end
    return build(node, 0)
end

--- The current value at any path of a model: a node's, a group's put
--- together, or a position inside a list or map row.
function valueIn(model, b, path)
    local n = I.nodeAt(model.nodes, path)
    if n then
        if n.kind == "group" then return groupValue(model, n.path) end
        return n.value
    end
    local loc = locate(model, b, path)
    if loc and loc.list and loc.rowKey ~= nil then
        local row = rowValue(loc.list.value, loc.rowKey)
        if #loc.rowSegs == 0 then return row end
        return valueAt(row, loc.rowSegs)
    end
    return nil
end

--- The meta of a list that sits inside the rows of another (a collection's
--- sublist): hub.json's `sublists` entry for it, plus every field key of the
--- outer list below that position ("sell[].price" is "price" in the sublist
--- "sell"). readonly or hidden on the position itself carries over.
function subMetaFor(listMeta, rowSegs)
    listMeta = listMeta or {}
    local out = {}
    local subs = plainTable(listMeta.sublists) and listMeta.sublists or nil
    if #rowSegs == 1 and subs and plainTable(subs[rowSegs[1]]) then
        for k, v in pairs(subs[rowSegs[1]]) do out[k] = v end
    end
    local fields = {}
    for k, fm in pairs(plainTable(out.fields) and out.fields or {}) do fields[k] = fm end
    for k, fm in pairs(plainTable(listMeta.fields) and listMeta.fields or {}) do
        local segs = fieldSegs(k)
        if #segs > #rowSegs + 1 and segs[#rowSegs + 1] == "[]" then
            local match = true
            for i2 = 1, #rowSegs do
                if segs[i2] ~= "[]" and segs[i2] ~= tostring(rowSegs[i2]) then match = false break end
            end
            if match then
                local rest = {}
                for i2 = #rowSegs + 2, #segs do rest[#rest + 1] = segs[i2] end
                local key = I.fieldKeyOf(rest)
                if fields[key] == nil then fields[key] = fm end
            end
        end
    end
    out.fields = fields
    if protected(fieldMeta(listMeta.fields, rowSegs)) then out.readonly = true end
    return out
end

--- What a table inside a row is edited as: rows (list), names (strings) or keyed rows (map).
function nestedKind(v, lm)
    if next(v) == nil then return (type(lm) == "table" and lm.kind == "map") and "map" or "list" end
    if isArray(v) then
        for _, x in ipairs(v) do if type(x) == "table" then return "list" end end
        return "strings"
    end
    return "map"
end

--- The references (§8.5) that name the keys of the table at path.
function refSourcesTo(b, path)
    local out, c = {}, I.canon(path)
    for _, rs in ipairs(b.refSourceList or {}) do
        if I.canon(rs.target) == c and (rs.refField == nil or rs.refField == "") then out[#out + 1] = rs end
    end
    return out
end

-- ---------------------------------------------------------------------------
-- Writing
-- ---------------------------------------------------------------------------

local function aceAllowed(object)
    local ok, allowed = pcall(IsPrincipalAceAllowed, "resource." .. SELF, object)
    return ok and allowed == true
end

--- Write one config file: poggy_core's own directly, any other script's
--- through its bridge. Then read it back. Returns true or false, reason.
function I.writeFile(folder, rel, text)
    if folder == SELF then
        if not SaveResourceFile(SELF, rel, text, #text) then return false, "SaveResourceFile returned false" end
    else
        if GetResourceState(folder) ~= "started" then return false, folder .. " is not started" end
        local okCall, res = pcall(function()
            -- The third argument marks a settings write, so a bridge that
            -- guards a development server against updates can let it through.
            return exports[folder]:PoggyWriteOwnFile(rel, text, "settings")
        end)
        if not okCall then
            return false, ("%s has no PoggyWriteOwnFile (it must load @poggy_core/template/poggy.lua): %s")
                :format(folder, tostring(res))
        end
        if type(res) ~= "table" or not res.ok then
            return false, tostring(type(res) == "table" and res.err or res)
        end
    end
    local back = LoadResourceFile(folder, rel)
    if back ~= text then
        return false, ("saved, but reading it back gave %s byte(s), expected %d"):format(back and #back or "no", #text)
    end
    return true
end

--- Copy a file's current text into poggy_core/update_backups/.
function I.backup(folder, rel, text, stamp)
    local file = (rel:gsub("[/\\]", "_"))
    local name = ("update_backups/%s__%s__settings__%s"):format(folder, stamp, file)
    -- Two saves in the same second: the second backup gets -2, -3 ... so the
    -- first one (the text before either save) is never overwritten.
    local n = 1
    while LoadResourceFile(SELF, name) ~= nil and n < 100 do
        n = n + 1
        name = ("update_backups/%s__%s-%d__settings__%s"):format(folder, stamp, n, file)
    end
    if not SaveResourceFile(SELF, name, text, #text) then return false, "could not write " .. name end
    return true
end

-- ---------------------------------------------------------------------------
-- Applying changes (§4.4)
-- ---------------------------------------------------------------------------

local OPS = { set = true, reset = true, insert = true, remove = true, move = true,
    renameKey = true, duplicate = true, linkRole = true, unlinkRole = true }

local function invalid(path, reason, extra)
    local e = extra or {}
    e.path, e.reason = path, reason
    return I.fail("invalid", ("%s: %s"):format(tostring(path), tostring(reason)), e)
end

--- Apply `changes` to `script`. opts: { who, fingerprints, logOp, skipLock }.
--- Returns an answer: I.ok({ fingerprints, restart, applied }) or I.fail(...).
function Hub.Apply(src, script, changes, opts)
    opts = opts or {}
    local Model = M()
    if not Model then return I.fail("unavailable", "The settings model is not loaded.") end
    local who = opts.who or I.who(src)

    if type(changes) ~= "table" or #changes == 0 then return I.fail("invalid", "Nothing to save.", { reason = "no changes" }) end
    if #changes > MAX_CHANGES then return I.fail("invalid", "Too many changes in one save.", { reason = "too many changes" }) end

    -- 1. the lock, and a running script
    if not opts.skipLock then
        local holder = Hub.IsLocked(script.id)
        if tonumber(src) == 0 then
            if holder then
                return I.fail("locked", ("%s is editing it in /poggy."):format(holder.name),
                    { holder = { name = holder.name, since = holder.since } })
            end
        elseif not I.holdsLock(src, script.id) then
            return I.fail("locked", holder and ("%s is editing it now."):format(holder.name)
                or "You no longer hold this script's lock. Open it again.",
                { holder = holder and { name = holder.name, since = holder.since } or nil })
        end
        I.touch(src, script.id)
    end
    local state = GetResourceState(script.folder)
    if script.folder ~= SELF and state ~= "started" then
        return I.fail("stopped", "The script is stopped, so it cannot save its files. Start it, then save again.")
    end

    local b = Hub.Build(script)
    local known = {}
    for _, f in ipairs(b.files) do known[f.file] = b.loaded[f.file] end

    -- 2. every touched file, re-read and compared with what the player saw.
    -- A rename that updates references (§8.5) also touches the files those
    -- references are in.
    local order, texts, originals, stale = {}, {}, {}, {}
    local shown = opts.fingerprints or I.remembered(src, script.id) or {}
    local function readFile(file, path)
        if texts[file] then return nil end
        local l = known[file]
        local text = LoadResourceFile(script.folder, file)
        if not text then return invalid(path, "the file could not be read") end
        if Model.Fingerprint(text) ~= shown[file] then stale[#stale + 1] = file end
        if not l.model then return invalid(path, "the file could not be read as settings: " .. tostring(l.err)) end
        texts[file], originals[file] = text, text
        order[#order + 1] = file
        return nil
    end
    for _, ch in ipairs(changes) do
        if type(ch) ~= "table" or type(ch.file) ~= "string" or not known[ch.file] then
            return invalid(type(ch) == "table" and ch.path or "?", "not one of this script's config files")
        end
        if not OPS[ch.op] then return invalid(ch.path, "unknown operation " .. tostring(ch.op)) end
        if type(ch.path) ~= "string" or not I.splitPath(ch.path) then return invalid(ch.path, "not a setting path") end
        local failure = readFile(ch.file, ch.path)
        if failure then return failure end
        if ch.op == "renameKey" and ch.updateRefs == true then
            for _, rs in ipairs(refSourcesTo(b, ch.path)) do
                if known[rs.node.file] then
                    failure = readFile(rs.node.file, ch.path)
                    if failure then return failure end
                end
            end
        end
    end
    if #stale > 0 then
        table.sort(stale)
        return I.fail("stale", "Changed on disk since you opened it: " .. table.concat(stale, ", ")
            .. ". Your changes were not saved; reload the script to see the file as it is now.", { files = stale })
    end

    -- 3. apply in order, each checked first against the file as it is by then
    local models, log, restart = {}, {}, false
    local function modelOf(file)
        if not models[file] then
            local l = known[file]
            local ok, model, err = pcall(Model.Parse, texts[file], l.opts)
            if not ok or not model then return nil, tostring(ok and err or model) end
            models[file] = model
        end
        return models[file]
    end

    local applyOne

    --- After a map row was renamed with updateRefs: every reference to the
    --- old key (§8.5), in any config file of the script, is set to the new
    --- key, each as a change of its own, checked like any other.
    local function rewriteRefs(listPath, oldKey, newKey)
        local oldText = tostring(oldKey)
        for _, rs in ipairs(refSourcesTo(b, listPath)) do
            local file = rs.node.file
            local model = known[file] and modelOf(file)
            if model then
                local cur = valueIn(model, b, rs.node.path)
                if rs.kind == "setting" then
                    if rs.node.kind == "strings" and plainTable(cur) then
                        local list, hit = {}, false
                        for i2, x in ipairs(cur) do
                            if type(x) == "string" and x == oldText then list[i2] = newKey; hit = true else list[i2] = x end
                        end
                        if hit then
                            local failure = applyOne({ file = file, op = "set", path = rs.node.path, value = list })
                            if failure then return failure end
                        end
                    elseif type(cur) == "string" and cur == oldText then
                        local failure = applyOne({ file = file, op = "set", path = rs.node.path, value = newKey })
                        if failure then return failure end
                    end
                elseif plainTable(cur) then
                    local hits = {}
                    I.eachRefValue({ kind = "field", segs = rs.segs,
                        node = { path = rs.node.path, value = cur, meta = rs.node.meta } }, function(p, v)
                        if type(v) == "string" and v == oldText then hits[#hits + 1] = p end
                    end)
                    for _, p in ipairs(hits) do
                        local failure = applyOne({ file = file, op = "set", path = p, value = newKey })
                        if failure then return failure end
                    end
                end
            end
        end
        return nil
    end

    --- The list a row operation works on, or nil, reason. Covers a list or
    --- map setting, a list of names, the rows of a collection written as
    --- separate lines (§8.4), and a list inside a row (a collection's
    --- sublist), which may not exist yet: inserting into it creates it.
    local function listFor(model, loc, target, meta, path, fopts)
        if loc.whole then
            return { path = loc.list.path, value = loc.list.value, kind = loc.listKind, meta = loc.listMeta, node = loc.list }
        end
        if target and target.kind == "strings" then
            return { path = target.path, value = target.value, kind = "strings", meta = meta, node = target }
        end
        if target and target.kind == "group" and fopts.statementRows and fopts.statementRows[target.path] then
            local bn = I.nodeAt(b.byPath, target.path)
            local lm = I.copy(bn and bn.meta or meta)
            lm.fields = I.widenFields(lm)
            return { path = target.path, value = groupValue(model, target.path) or {}, kind = "map", meta = lm,
                node = target, statements = true }
        end
        if loc.list and loc.rowSegs and #loc.rowSegs >= 1 then
            -- A list inside a row of a list or map.
            local row = rowValue(loc.list.value, loc.rowKey)
            if row == nil then return nil, "no such row" end
            local cur = valueAt(row, loc.rowSegs)
            local lm = subMetaFor(loc.listMeta, loc.rowSegs)
            if cur == nil then
                return { path = path, value = {}, kind = lm.kind == "map" and "map" or "list", meta = lm, missing = true }
            end
            if not plainTable(cur) then return nil, "no such list" end
            return { path = path, value = cur, kind = nestedKind(cur, lm), meta = lm, nested = true }
        end
        -- A list a collection row (kept as nodes) does not have yet.
        local holder = loc.chain[1] and loc.chain[1].node
        if not target and holder and holder.kind == "group" then
            local segs = I.splitPath(path)
            local hsegs = I.splitPath(holder.path)
            local bn = I.nodeAt(b.byPath, holder.path)
            if segs and hsegs and #segs == #hsegs + 1 and bn and bn.item and not segs[#segs].index then
                local coll = I.nodeAt(b.byPath, bn.item)
                local field = segs[#segs].key
                local sm = coll and plainTable(coll.meta.sublists) and coll.meta.sublists[field] or nil
                if bn.row ~= nil and bn.key == bn.row and coll and coll.collection then
                    local lm = plainTable(sm) and I.copy(sm) or {}
                    local fm = fieldMeta(coll.meta.fields, { field })
                    if protected(fm) then lm.readonly = true end
                    return { path = path, value = {}, kind = "list", meta = lm, missing = true, holder = holder }
                end
            end
        end
        return nil, "no such list"
    end

    applyOne = function(ch)
        local file, path, op = ch.file, ch.path, ch.op
        local model, perr = modelOf(file)
        if not model then return invalid(path, "the file could not be read as settings: " .. tostring(perr)) end
        local fopts = known[file].opts
        local loc = locate(model, b, path)
        if not loc then return invalid(path, "not a setting path") end

        -- readonly / hidden / not editable, anywhere on the way down: refused
        -- before anything else, whatever the page sent (§4.2: nothing from the
        -- client is trusted). Some of these are Lua or SQL the script runs.
        local refusal = chainRefusal(loc) or protectedRefusal(b.meta, path, nil, nil, false)
        if refusal then return invalid(path, refusal) end
        local listFields = loc.list and loc.listMeta.fields or nil
        if loc.rowSegs then
            refusal = fieldRefusal(listFields, loc.rowSegs)
            if refusal then return invalid(path, refusal) end
        end

        local target = loc.exact
        local meta = {}
        if target then
            local bn = I.nodeAt(b.byPath, target.path)
            meta = bn and bn.meta or {}
        end
        local liveMeta = (target and next(meta) and meta) or (loc.list and loc.listMeta) or meta

        if op == "set" or op == "reset" then
            local value = ch.value
            if op == "reset" then
                if not target then return invalid(path, "only a whole setting can be reset") end
                local sm = Hub.ShippedModel(script, file, known[file].dictKeys, fopts)
                local sn = sm and I.nodeAt(sm.nodes, path)
                if not sn then return invalid(path, "the shipped value is not known") end
                value = sn.value
            end
            if value == nil then return invalid(path, "no value") end
            local why = checkData(value, 0, { n = 0 })
            if why then return invalid(path, why) end

            local old
            if loc.whole then
                -- A whole list or map.
                why = checkShape(loc.list, loc.listKind, value) or descendantRefusal(model, b, path)
                if not why then
                    for _, row in pairs(value) do
                        local w, where = checkFields(listFields, row, {})
                        if w then why = (where and (where .. ": ") or "") .. w break end
                    end
                end
                if not why then
                    local where = protectedRows(listFields, loc.list.value, value)
                    if where then why = where .. " is read-only here: edit it in the file" end
                end
                old = loc.list.value
            elseif loc.rowSegs then
                -- A row, or a value inside one: the list's field limits.
                local oldRow = rowValue(loc.list.value, loc.rowKey)
                if #loc.rowSegs == 0 then
                    if loc.listKind == "list" and (type(value) ~= "table" or value.__type) then why = "every row must be a table" end
                    if not why and oldRow == nil and loc.listKind == "list" then why = "no such row" end
                    old = oldRow
                else
                    if oldRow == nil then why = "no such row" end
                    why = why or checkLimits(value, fieldMeta(listFields, loc.rowSegs))
                    old = valueAt(oldRow, loc.rowSegs)
                end
                if not why then
                    local w, where = checkFields(listFields, value, loc.rowSegs)
                    if w then why = (where and (where .. ": ") or "") .. w end
                end
                if not why then
                    local where = protectedDiff(listFields, old, value, loc.rowSegs)
                    if where then why = where .. " is read-only here: edit it in the file" end
                end
                -- Anything read-only below what is replaced: a code cell of
                -- the row (name = T('X')) when the whole row is set.
                if not why then why = descendantRefusal(model, b, path) end
            elseif target then
                -- An ordinary setting. hub.json options define the allowed values,
                -- types included (false or "vorp"): a listed value is never
                -- refused for its type.
                local kind = effectiveKind(target, meta)
                if not inOptions(value, meta) then why = checkShape(target, kind, value) end
                why = why or checkLimits(value, meta)
                old = target.value
            else
                why = "no such setting"
            end
            if not why then why = protectedRefusal(b.meta, path, old, value, true) end
            if why then return invalid(path, why) end

            local newText, err = Model.Set(texts[file], path, value, fopts)
            if not newText then return invalid(path, err or "could not be written") end
            texts[file] = newText
            log[#log + 1] = { file = file, path = path, op = opts.logOp or op, old = old, new = value }
            if target and not loc.whole and (target.kind == "value" or target.kind == "strings") then
                -- Structure unchanged: keep the model, note the new value.
                target.value = value
            else
                models[file] = nil
            end
            if liveMeta.live ~= true then restart = true end

        elseif op == "insert" or op == "remove" or op == "move" or op == "renameKey" or op == "duplicate" then
            local L, lwhy = listFor(model, loc, target, meta, path, fopts)
            if not L then return invalid(path, lwhy) end
            local kind, lmeta = L.kind, L.meta or {}
            if protected(lmeta) then return invalid(path, PROTECTED_REASON) end
            if L.missing and op ~= "insert" then return invalid(path, "no such list") end
            local rows = countRows(L.value)
            local function rowPathOf(key)
                if type(key) == "string" then return I.canon(L.path) .. ("[%q]"):format(key) end
                return I.canon(L.path) .. ("[%d]"):format(key)
            end
            local newText, err
            if kind ~= "map" and op ~= "renameKey" and protectedByPosition(b.meta, L.path) then
                return invalid(path, "some rows here are read-only by position; add, remove or reorder them in the file")
            end
            if op == "insert" then
                local value = ch.value
                local why = value == nil and "no value" or checkData(value, 0, { n = 0 })
                if not why then
                    if kind == "strings" then
                        if type(value) ~= "string" and type(value) ~= "number" then why = "expects a name" end
                        why = why or checkLimits(value, lmeta)
                    else
                        -- A map row may be a single value (item -> category).
                        if kind == "list" and (type(value) ~= "table" or value.__type) then why = "a row must be a table" end
                        if not why then
                            local w, where = checkFields(lmeta.fields, value, {})
                            if w then why = (where and (where .. ": ") or "") .. w end
                        end
                        if not why then
                            -- A new row may carry a read-only field only as the
                            -- hub.json template has it.
                            local where = protectedDiff(lmeta.fields, lmeta.template, value, {})
                            if where then why = where .. " is read-only here: edit it in the file" end
                        end
                    end
                end
                if why then return invalid(path, why) end
                local index = ch.index
                if kind == "map" then
                    -- Text, or a whole number for a number-keyed map.
                    if type(index) == "number" then index = toInt(index) end
                    if (type(index) ~= "string" and math.type(index) ~= "integer") or index == "" or #tostring(index) > 128 then
                        return invalid(path, "a new row needs a key")
                    end
                    if rowValue(L.value, index) ~= nil then return invalid(path, "that key is already used") end
                    local why2 = protectedRefusal(b.meta, rowPathOf(index), nil, value, true)
                    if why2 then return invalid(path, why2) end
                elseif index ~= nil then
                    index = toInt(index)
                    if not index or index < 1 or index > rows + 1 then return invalid(path, "row position out of range") end
                end
                if L.missing and L.holder then
                    -- The row (a group of settings) gains the list: the row is
                    -- rewritten with one field more, nothing else changes.
                    local rowNow = groupValue(model, L.holder.path)
                    if rowNow == nil then return invalid(path, "this row has code in it; add the list in the file") end
                    local key = I.splitPath(path)
                    key = key[#key].key
                    local newRow = I.copy(rowNow)
                    newRow[key] = kind == "map" and { [tostring(index)] = value } or { value }
                    newText, err = Model.Set(texts[file], L.holder.path, newRow, fopts)
                elseif L.missing then
                    local first = kind == "map" and { [tostring(index)] = value } or { value }
                    newText, err = Model.Set(texts[file], path, first, fopts)
                else
                    newText, err = Model.Insert(texts[file], path, index, value, fopts)
                end
            elseif op == "remove" then
                local index = ch.index
                if kind == "map" then
                    if index == nil then index = ch.oldKey end
                    if type(index) == "number" then index = toInt(index) end
                    if (type(index) ~= "string" and math.type(index) ~= "integer") or rowValue(L.value, index) == nil then
                        return invalid(path, "no such row")
                    end
                else
                    index = toInt(index)
                    if not index or index < 1 or index > rows then return invalid(path, "no such row") end
                end
                local rowNode = I.nodeAt(model.nodes, rowPathOf(index))
                if rowNode and (rowNode.kind == "readonly" or rowNode.editable == false) then
                    return invalid(path, "that row can only be edited in the file")
                end
                local why2 = protectedRefusal(b.meta, rowPathOf(index), rowValue(L.value, index), nil, true)
                if why2 then return invalid(path, why2) end
                newText, err = Model.Remove(texts[file], path, index, fopts)
            elseif op == "duplicate" then
                -- A verbatim copy of a row the file already has (its code
                -- cells too): list rows go after the original, keyed rows
                -- take newKey.
                if kind == "strings" then return invalid(path, "a list of names has nothing to copy; add the name instead") end
                if kind == "map" then
                    local oldKey, newKey = ch.oldKey, ch.newKey
                    if oldKey == nil then oldKey = ch.index end
                    if type(oldKey) == "number" then oldKey = toInt(oldKey) end
                    if type(newKey) == "number" then newKey = toInt(newKey) end
                    if oldKey == nil or rowValue(L.value, oldKey) == nil then return invalid(path, "no such row") end
                    if (type(newKey) ~= "string" and math.type(newKey) ~= "integer") or newKey == "" or #tostring(newKey) > 128 then
                        return invalid(path, "the copy needs a new key")
                    end
                    if rowValue(L.value, newKey) ~= nil then return invalid(path, "that key is already used") end
                    local why2 = protectedRefusal(b.meta, rowPathOf(newKey), nil, rowValue(L.value, oldKey), true)
                    if why2 then return invalid(path, why2) end
                    newText, err = Model.Duplicate(texts[file], path, oldKey, newKey, fopts)
                else
                    local index = toInt(ch.index)
                    if not index or index < 1 or index > rows then return invalid(path, "no such row") end
                    newText, err = Model.Duplicate(texts[file], path, index, nil, fopts)
                end
            elseif op == "move" then
                if kind == "map" then return invalid(path, "rows keyed by name have no order") end
                local from, to = toInt(ch.from), toInt(ch.to)
                if not from or not to or from < 1 or to < 1 or from > rows or to > rows then
                    return invalid(path, "row position out of range")
                end
                newText, err = Model.Move(texts[file], path, from, to, fopts)
            else
                if kind ~= "map" then return invalid(path, "only rows keyed by name can be renamed") end
                local oldKey, newKey = ch.oldKey, ch.newKey
                if type(oldKey) == "number" then oldKey = toInt(oldKey) end
                if type(newKey) == "number" then newKey = toInt(newKey) end
                if oldKey == nil or rowValue(L.value, oldKey) == nil then return invalid(path, "no such row") end
                if (type(newKey) ~= "string" and math.type(newKey) ~= "integer") or newKey == "" or #tostring(newKey) > 128 then
                    return invalid(path, "the new key is empty or too long")
                end
                if rowValue(L.value, newKey) ~= nil then return invalid(path, "that key is already used") end
                local rowNode = I.nodeAt(model.nodes, rowPathOf(oldKey))
                if rowNode and (rowNode.kind == "readonly" or rowNode.editable == false) then
                    return invalid(path, "that row can only be edited in the file")
                end
                local moving = rowValue(L.value, oldKey)
                local why2 = protectedRefusal(b.meta, rowPathOf(oldKey), moving, nil, true)
                    or protectedRefusal(b.meta, rowPathOf(newKey), nil, moving, true)
                if why2 then return invalid(path, why2) end
                newText, err = Model.RenameKey(texts[file], path, oldKey, newKey, fopts)
            end
            if not newText then return invalid(path, err or "could not be written") end
            local old = L.missing and nil or L.value
            texts[file] = newText
            models[file] = nil
            local after = modelOf(file)
            log[#log + 1] = { file = file, path = path, op = opts.logOp or op, old = old,
                new = after and valueIn(after, b, path) or nil }
            if lmeta.live ~= true then restart = true end
            if op == "renameKey" and ch.updateRefs == true then
                local failure = rewriteRefs(path, ch.oldKey, ch.newKey)
                if failure then return failure end
            end

        elseif op == "linkRole" or op == "unlinkRole" then
            if not target or target.kind ~= "strings" or loc.rowSegs then
                return invalid(path, "only a list of names can be linked to a role")
            end
            local old = target.value
            local newText, err = texts[file], nil
            local newValue = old
            if op == "linkRole" then
                local roles = PoggyCoreConfig.Roles or {}
                local role = type(ch.role) == "string" and roles[ch.role] or nil
                if type(role) ~= "table" or type(role.list) ~= "table" then return invalid(path, "no such role") end
                if not I.equal(old, role.list) then
                    local why = checkLimits(role.list, meta)
                    if why then return invalid(path, why) end
                    newText, err = Model.Set(newText, path, role.list, fopts)
                    newValue = role.list
                end
                if newText then newText, err = Model.SetRole(newText, path, ch.role, fopts) end
            else
                newText, err = Model.SetRole(newText, path, nil, fopts)
            end
            if not newText then return invalid(path, err or "could not be written") end
            texts[file] = newText
            models[file] = nil
            log[#log + 1] = { file = file, path = path, op = opts.logOp or op, old = old, new = newValue }
            if not I.equal(old, newValue) and meta.live ~= true then restart = true end
        end
        return nil
    end

    for _, ch in ipairs(changes) do
        local failure = applyOne(ch)
        if failure then return failure end
    end

    -- Nothing actually changed (every value was what the file already held).
    local changedFiles = {}
    for _, file in ipairs(order) do
        if texts[file] ~= originals[file] then changedFiles[#changedFiles + 1] = file end
    end

    -- 4. backups, all before any write
    local stamp = os.date("%Y%m%d-%H%M%S")
    for _, file in ipairs(changedFiles) do
        local ok, why = I.backup(script.folder, file, originals[file], stamp)
        if not ok then return I.fail("write_failed", "The backup could not be written, so nothing was saved: " .. tostring(why)) end
    end

    -- 5. write; if one fails, put back the ones already written
    local written = {}
    for _, file in ipairs(changedFiles) do
        local ok, why = I.writeFile(script.folder, file, texts[file])
        if not ok then
            for _, done in ipairs(written) do I.writeFile(script.folder, done, originals[done]) end
            I.forgetModels(script.folder)
            Util.Error("settings hub: %s/%s could not be saved: %s", script.folder, file, tostring(why))
            return I.fail("write_failed", ("%s could not be saved: %s. Nothing was changed."):format(file, tostring(why)))
        end
        written[#written + 1] = file
    end
    I.forgetModels(script.folder)

    -- 6. history and mirror
    if #changedFiles > 0 then
        logRows(script, log, who)
        Hub.QueueMirror(script.id)
        Util.Log("settings hub: %s saved %d change(s) to %s (%s)", tostring(who.name), #log,
            script.id, table.concat(changedFiles, ", "))
    end

    -- 7. the answer
    local fresh = Hub.Build(script)
    local fps = {}
    for _, f in ipairs(fresh.files) do fps[f.file] = f.fingerprint end
    if tonumber(src) and tonumber(src) > 0 then I.remember(src, script.id, fresh.files) end
    return I.ok({ fingerprints = fps, restart = restart and #changedFiles > 0, applied = #log })
end

--- hub:save — id, { fingerprints = { file = fp }, changes = { change... } }
function Hub.Save(src, id, payload)
    local s, failure = I.script(id)
    if not s then return failure end
    if type(payload) ~= "table" then return I.fail("invalid", "Nothing to save.", { reason = "no payload" }) end
    local fps = type(payload.fingerprints) == "table" and payload.fingerprints or nil
    return Hub.Apply(src, s, payload.changes, { fingerprints = fps })
end

--- The current fingerprints of a script's files (the console and undo save
--- against the files as they are now).
local function currentFingerprints(script)
    local fps = {}
    for _, f in ipairs(Hub.Build(script).files) do fps[f.file] = f.fingerprint end
    return fps
end

--- `poggycore settings set <id> <path> <json value>`, logged as the console.
function Hub.ConsoleSet(id, path, value)
    local s, failure = I.script(id)
    if not s then return failure end
    local b = Hub.Build(s)
    local file
    local node = I.nodeAt(b.byPath, path)
    if node then file = node.file end
    if not file then
        for _, a in ipairs(I.ancestors(path)) do
            local anc = I.nodeAt(b.byPath, a)
            if anc then file = anc.file break end
        end
    end
    if not file then return I.fail("invalid", "no such setting", { path = path, reason = "no such setting" }) end
    return Hub.Apply(0, s, { { file = file, op = "set", path = path, value = value } },
        { fingerprints = currentFingerprints(s), who = I.who(0) })
end

--- hub:undo — put a logged change's old value back, as a new change.
function Hub.Undo(src, logId)
    if not DB.ready then return I.fail("unavailable", "History needs the database (oxmysql).") end
    local n = toInt(logId)
    if not n then return I.fail("invalid", "No such history entry.", { reason = "bad id" }) end
    local rows = DB.query("SELECT `poggy_id`, `file`, `path`, `op`, `old_value` FROM `poggy_settings_log` WHERE `id` = ? LIMIT 1", { n })
    local r = rows and rows[1]
    if not r then return I.fail("invalid", "No such history entry.", { reason = "not found" }) end
    local s, failure = I.script(r.poggy_id)
    if not s then return failure end
    -- §10: a data panel write is undone through the panel, not the files.
    -- Actions, container changes and drill-down writes are not undone here.
    if type(r.file) == "string" and (r.file:sub(1, 6) == "panel:" or r.file:sub(1, 10) == "container:") then
        local okB, b = pcall(Hub.Build, s)
        local info = I.panelLogInfo(okB and b or nil, r.file, r.path, r.op)
        if not info or not info.canUndo then
            return I.fail("invalid", "That change cannot be undone from History; make it again in the panel.",
                { reason = "not undoable" })
        end
        if r.old_value == nil then
            return I.fail("invalid", "That cell was empty before; there is no earlier value to go back to.",
                { path = r.path, reason = "no earlier value" })
        end
        return Hub.PanelWrite(src, s.id, info.panel, info.panelKey, info.panelField, I.decode(r.old_value))
    end
    if r.old_value == nil then
        return I.fail("invalid", "That change added the setting; there is no earlier value to go back to.",
            { path = r.path, reason = "no earlier value" })
    end
    local old = I.decode(r.old_value)
    if old == nil then return I.fail("invalid", "The earlier value could not be read.", { path = r.path, reason = "unreadable" }) end
    return Hub.Apply(src, s, { { file = r.file, op = "set", path = r.path, value = old } },
        { fingerprints = currentFingerprints(s), logOp = "undo" })
end

-- ---------------------------------------------------------------------------
-- Restart and start (§4.5)
-- ---------------------------------------------------------------------------

local function waitStarted(folder, ms)
    local deadline = GetGameTimer() + ms
    repeat
        Wait(250)
        if GetResourceState(folder) == "started" then return true end
    until GetGameTimer() >= deadline
    return GetResourceState(folder) == "started"
end

--- Restart (or start) one folder. Returns true or false, reason.
function I.restartFolder(folder, startOnly)
    if aceAllowed("command.ensure") then
        ExecuteCommand("ensure " .. folder)
    else
        if not startOnly then
            local okStop, stopped = pcall(StopResource, folder)
            if not okStop or stopped == false then return false, "StopResource was refused" end
            Wait(250)
        end
        local okStart, started = pcall(StartResource, folder)
        if not okStart or started == false then
            return false, ("StartResource was refused; add_ace resource.%s command.ensure allow lets poggy_core use ensure"):format(SELF)
        end
    end
    if waitStarted(folder, 10000) then return true end
    return false, "it did not start (state: " .. tostring(GetResourceState(folder)) .. "); see the server console"
end

local SELF_RESTART = "Restart poggy_core by hand; it restarts every Poggy script."

function Hub.Restart(src, id, startOnly)
    local s, failure = I.script(id)
    if not s then return failure end
    if s.folder == SELF then return I.fail("self", SELF_RESTART) end
    local state = GetResourceState(s.folder)
    if startOnly and state == "started" then return I.ok({ running = true }) end
    if not startOnly and state ~= "started" then
        return I.fail("stopped", "The script is stopped; start it instead.")
    end
    local who = I.who(src)
    Util.Log("settings hub: %s %s %s", tostring(who.name), startOnly and "started" or "restarted", s.id)
    local ok, why = I.restartFolder(s.folder, startOnly)
    I.pushViewers("scriptState", { id = s.id, running = GetResourceState(s.folder) == "started" })
    if not ok then return I.fail("restart_failed", tostring(why)) end
    return I.ok({ running = true })
end

-- ---------------------------------------------------------------------------
-- Roles (§4.7)
-- ---------------------------------------------------------------------------

--- Every role and where each is linked: { roles, usage = { role = { {id, path} } } }.
function Hub.Roles()
    local usage = {}
    for name in pairs(PoggyCoreConfig.Roles or {}) do usage[name] = {} end
    for _, s in ipairs(Hub.Scripts()) do
        local okB, b = pcall(Hub.Build, s)
        if okB and b then
            for _, n in ipairs(b.nodes) do
                if n.role then
                    usage[n.role] = usage[n.role] or {}
                    table.insert(usage[n.role], { id = s.id, path = n.path, file = n.file })
                end
            end
        end
    end
    return { roles = PoggyCoreConfig.Roles or {}, usage = usage }
end

--- Check and tidy roles sent by the page. Returns the clean table or nil, reason.
function I.cleanRoles(roles)
    if type(roles) ~= "table" then return nil, "no roles" end
    local out, count = {}, 0
    for name, r in pairs(roles) do
        count = count + 1
        if count > 64 then return nil, "too many roles" end
        if type(name) ~= "string" or not name:match("^[%a_][%w_]*$") or #name > 32 then
            return nil, ("%s is not a usable role name (letters, digits and _)"):format(tostring(name))
        end
        if type(r) ~= "table" then return nil, name .. " is not a role" end
        local label = r.label == nil and I.readable(name) or r.label
        if type(label) ~= "string" or #label > 64 or label:find("[%c]") then return nil, name .. ": the label is not usable" end
        local kind = r.kind == nil and "jobs" or r.kind
        if kind ~= "jobs" and kind ~= "groups" then return nil, name .. ": kind must be jobs or groups" end
        if type(r.list) ~= "table" or (next(r.list) ~= nil and not isArray(r.list)) then return nil, name .. ": the list is not a list" end
        if #r.list > 500 then return nil, name .. ": the list is too long" end
        local list, seen = {}, {}
        for _, v in ipairs(r.list) do
            if type(v) ~= "string" then return nil, name .. ": every entry must be a name" end
            v = v:match("^%s*(.-)%s*$")
            if v == "" or #v > 64 or v:find("[%c]") then return nil, name .. ": " .. v .. " is not a usable name" end
            if not seen[v] then seen[v] = true; list[#list + 1] = v end
        end
        out[name] = { label = label, kind = kind, list = list }
    end
    return out
end

--- The PoggyCoreConfig.Roles block, as it is written into config.lua.
function I.rolesBlock(roles)
    local names = {}
    for name in pairs(roles) do names[#names + 1] = name end
    table.sort(names)
    local width = 0
    for _, name in ipairs(names) do width = math.max(width, #name) end
    local lines = { "PoggyCoreConfig.Roles = {" }
    for _, name in ipairs(names) do
        local r, items = roles[name], {}
        for _, v in ipairs(r.list) do items[#items + 1] = ("%q"):format(v) end
        lines[#lines + 1] = ("    %s = { label = %q, kind = %q, list = { %s } },")
            :format(name .. (" "):rep(width - #name), r.label, r.kind, table.concat(items, ", "))
    end
    lines[#lines + 1] = "}"
    return table.concat(lines, "\n")
end

--- The index of the "}" that closes the "{" at `open`, skipping strings and comments.
local function closingBrace(text, open)
    local depth, i, n = 0, open, #text
    while i <= n do
        local c = text:sub(i, i)
        if c == "-" and text:sub(i + 1, i + 1) == "-" then
            local eq = text:match("^%-%-%[(=*)%[", i)
            if eq then
                local close = text:find("]" .. eq .. "]", i, true)
                i = close and (close + #eq + 2) or (n + 1)
            else
                i = (text:find("\n", i, true) or n) + 1
            end
        elseif c == '"' or c == "'" then
            local j = i + 1
            while j <= n do
                local d = text:sub(j, j)
                if d == "\\" then j = j + 2
                elseif d == c or d == "\n" then break
                else j = j + 1 end
            end
            i = j + 1
        elseif c == "[" and text:match("^%[=*%[", i) then
            local eq = text:match("^%[(=*)%[", i)
            local close = text:find("]" .. eq .. "]", i, true)
            i = close and (close + #eq + 2) or (n + 1)
        else
            if c == "{" then
                depth = depth + 1
            elseif c == "}" then
                depth = depth - 1
                if depth == 0 then return i end
            end
            i = i + 1
        end
    end
    return nil
end

--- Load config text in a sandbox and return the globals it assigned.
local function loadGlobals(text)
    local Model = M()
    if Model and Model.Load then
        local ok, env = pcall(Model.Load, text)
        if ok and type(env) == "table" then return env end
    end
    local env = setmetatable({}, { __index = function(_, k)
        if k == "vector2" or k == "vector3" or k == "vector4" then
            return function(...) return { ... } end
        end
        return nil
    end })
    local fn = load((text:gsub("`[^`\n]*`", "0")), "=config", "t", env)
    if not fn or not pcall(fn) then return nil end
    return env
end

--- config.lua with its PoggyCoreConfig.Roles block replaced (added at the end
--- when there is none), checked: only Roles differs afterwards.
function I.rewriteRoles(text, roles)
    local block = I.rolesBlock(roles)
    local newText
    local s = text:find("\nPoggyCoreConfig%.Roles%s*=%s*{")
    local start = s and (s + 1) or (text:find("^PoggyCoreConfig%.Roles%s*=%s*{") and 1 or nil)
    if start then
        local open = text:find("{", start, true)
        local close = closingBrace(text, open)
        if not close then return nil, "the Roles block in config.lua could not be read" end
        newText = text:sub(1, start - 1) .. block .. text:sub(close + 1)
    else
        newText = text .. (text:sub(-1) == "\n" and "" or "\n") .. "\n" .. block .. "\n"
    end
    local before, after = loadGlobals(text), loadGlobals(newText)
    if not before or not after or type(after.PoggyCoreConfig) ~= "table" then
        return nil, "config.lua would not load after the change"
    end
    if not I.equal(after.PoggyCoreConfig.Roles, roles) then return nil, "the roles did not come out as written" end
    for k, v in pairs(before.PoggyCoreConfig or {}) do
        if k ~= "Roles" and not I.equal(v, after.PoggyCoreConfig[k]) then
            return nil, "the change would have touched PoggyCoreConfig." .. tostring(k)
        end
    end
    return newText
end

--- hub:saveRoles. Rewrites poggy_core's config.lua, then every linked setting,
--- then restarts the scripts that changed.
function Hub.SaveRoles(src, roles)
    local clean, why = I.cleanRoles(roles)
    if not clean then return I.fail("invalid", why, { reason = why }) end
    local who = I.who(src)
    local holder = Hub.IsLocked(I.coreId())
    if holder and holder.src ~= tonumber(src) then
        return I.fail("locked", ("%s is editing Poggy Core's settings; the roles live there."):format(holder.name),
            { holder = { name = holder.name, since = holder.since } })
    end

    local text = LoadResourceFile(SELF, "config.lua")
    if not text then return I.fail("write_failed", "poggy_core's config.lua could not be read.") end
    local newText, err = I.rewriteRoles(text, clean)
    if not newText then return I.fail("write_failed", "The roles could not be written: " .. tostring(err)) end
    if newText ~= text then
        local stamp = os.date("%Y%m%d-%H%M%S")
        local okB, whyB = I.backup(SELF, "config.lua", text, stamp)
        if not okB then return I.fail("write_failed", "The backup could not be written: " .. tostring(whyB)) end
        local okW, whyW = I.writeFile(SELF, "config.lua", newText)
        if not okW then
            I.writeFile(SELF, "config.lua", text)
            return I.fail("write_failed", "config.lua could not be saved: " .. tostring(whyW))
        end
        I.forgetModels(SELF)
        local core = Hub.Find(I.coreId())
        if core then
            logRows(core, { { file = "config.lua", path = "PoggyCoreConfig.Roles", op = "roles",
                old = PoggyCoreConfig.Roles, new = clean } }, who)
            Hub.QueueMirror(core.id)
        end
    end
    PoggyCoreConfig.Roles = clean

    -- Every linked setting in every script.
    local rewritten, skipped, restartList = {}, {}, {}
    for _, s in ipairs(Hub.Scripts(true)) do
        local b = Hub.Build(s)
        local changes, paths = {}, {}
        for _, n in ipairs(b.nodes) do
            if n.role then
                local role = clean[n.role]
                if role then
                    if not I.equal(n.value, role.list) then
                        changes[#changes + 1] = { file = n.file, op = "set", path = n.path, value = role.list }
                        paths[#paths + 1] = n.path
                    end
                else
                    changes[#changes + 1] = { file = n.file, op = "unlinkRole", path = n.path }
                    paths[#paths + 1] = n.path
                end
            end
        end
        if #changes > 0 then
            local lock = Hub.IsLocked(s.id)
            if lock and lock.src ~= tonumber(src) then
                skipped[#skipped + 1] = { id = s.id, holder = { name = lock.name, since = lock.since } }
            elseif s.folder ~= SELF and GetResourceState(s.folder) ~= "started" then
                skipped[#skipped + 1] = { id = s.id, reason = "stopped" }
            else
                local fps = {}
                for _, f in ipairs(b.files) do fps[f.file] = f.fingerprint end
                local res = Hub.Apply(src, s, changes, { fingerprints = fps, who = who, logOp = "role", skipLock = true })
                if res.ok then
                    for _, p in ipairs(paths) do rewritten[#rewritten + 1] = { id = s.id, path = p } end
                    if res.value.restart and s.folder ~= SELF then restartList[#restartList + 1] = s end
                else
                    skipped[#skipped + 1] = { id = s.id, reason = res.message or res.err }
                end
            end
        end
    end

    local restarted = {}
    for _, s in ipairs(restartList) do
        local ok = I.restartFolder(s.folder, false)
        I.pushViewers("scriptState", { id = s.id, running = GetResourceState(s.folder) == "started" })
        if ok then restarted[#restarted + 1] = s.id end
    end
    Util.Log("settings hub: %s saved the roles: %d setting(s) rewritten, %d script(s) skipped, %d restarted",
        tostring(who.name), #rewritten, #skipped, #restarted)
    return I.ok({ rewritten = rewritten, skipped = skipped, restarted = restarted })
end

--- poggy_core's own id (its poggy_id).
function I.coreId()
    local Id = PoggyCore.Identity
    return Id and Id.Id(SELF) or SELF
end

-- ---------------------------------------------------------------------------
-- Callbacks
-- ---------------------------------------------------------------------------

local register = Hub.Register

register("save", function(src, id, payload) return Hub.Save(src, id, payload) end)
register("undo", function(src, logId) return Hub.Undo(src, logId) end)
register("restart", function(src, id) return Hub.Restart(src, id, false) end)
register("start", function(src, id) return Hub.Restart(src, id, true) end)

register("history", function(src, id, page)
    local s, failure = I.script(id)
    if not s then return failure end
    return I.ok(Hub.History(s.id, page))
end)

register("roles", function() return I.ok(Hub.Roles()) end)
register("saveRoles", function(src, roles) return Hub.SaveRoles(src, roles) end)

-- ---------------------------------------------------------------------------
-- Start: the tables, old history, and every mirror
-- ---------------------------------------------------------------------------

CreateThread(function()
    -- oxmysql may start after poggy_core.
    local waited = 0
    while GetResourceState("oxmysql") ~= "started" and waited < 120 do
        Wait(1000)
        waited = waited + 1
    end
    if GetResourceState("oxmysql") ~= "started" then
        Util.Warn("settings hub: oxmysql is not started, so /poggy keeps no history. Editing still works.")
        return
    end
    local ok, err = pcall(Hub.InitDatabase)
    if not ok then
        Util.Error("settings hub: database setup failed: %s", tostring(err))
        return
    end
    if DB.ready then
        for _, e in ipairs((PoggyCore.Identity and PoggyCore.Identity.List()) or {}) do Hub.QueueMirror(e.id) end
    end
end)
