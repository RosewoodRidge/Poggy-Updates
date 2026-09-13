--[[
    poggy_core — database install runner (server).

    Every Poggy script keeps its tables in ONE file, sql/install.sql, and when the
    script starts its bridge asks poggy_core to run it (sql.install). The file is
    written so it can run on every start, on MariaDB and on MySQL 8:

        CREATE TABLE IF NOT EXISTS ...        created when missing
        ALTER TABLE t ADD COLUMN c ...        added when missing
        ALTER TABLE t ADD INDEX i (...)       added when missing (the index needs a name)
        CREATE INDEX i ON t (...)             added when missing
        ALTER TABLE t MODIFY COLUMN c TYPE    run only when the column's TYPE differs
        INSERT IGNORE INTO ...                rows added when missing, never overwritten
        UPDATE ... WHERE ...                  data fixes; the WHERE must make them repeatable
        SELECT ...                            ignored (handy when importing the file by hand)

    MySQL 8 has no ADD COLUMN IF NOT EXISTS (MariaDB does), so the runner asks
    information_schema first and sends only what is missing, with any IF NOT
    EXISTS removed. A second run sends no DDL at all and raises no database error.
    That matters: oxmysql announces every failed query (the oxmysql:error event),
    and an owner's error logger should never hear from a normal start.

    Anything that cannot be repeated safely is refused in install.sql and skipped
    with a yellow line: DROP, RENAME, TRUNCATE, DELETE, a plain INSERT, INSERT ...
    ON DUPLICATE KEY UPDATE (it would overwrite owners' rows on every start), and
    ALTER clauses other than the ones above. Those belong in a migration:

        sql/migrations/001.sql, 002.sql, ...

    Migrations run once each, in order, after install.sql, and are recorded in the
    `poggy_migrations` table. A failed migration is not recorded, so it runs again
    on the next start; write migrations so a re-run is harmless (the ADD checks
    above apply inside them too). A migration may begin with

        -- poggy:only-if-table <name>

    to count as done without running when that table does not exist, e.g. copying
    rows out of a table that a fresh install never had.

    MODIFY compares types only. A change to NOT NULL, a default or a comment alone
    never triggers it; put that in a migration.
]]

PoggyCore = PoggyCore or {}

local Sql = {}
PoggyCore.Sql = Sql

local SELF        = GetCurrentResourceName()
local INSTALL     = "sql/install.sql"
local MIGRATION   = "sql/migrations/%03d.sql"
local BRIDGE      = "@poggy_core/template/poggy.lua"
local DDL_TIMEOUT = 120000

local running = {}               -- resource -> true while its install runs

local function prefix() return PoggyCore.PREFIX or "^5[Poggy Core]^7 " end

-- ---------------------------------------------------------------------------
-- Database access
--
-- Replaceable: tools/test-sql-runner.py swaps Sql.Db for a fake database.
--   Sql.Db.query(sql, params) -> rows            | nil, message
--   Sql.Db.exec(sql, params)  -> affected rows   | nil, message
-- ---------------------------------------------------------------------------

local function call(method, sql, params)
    if GetResourceState("oxmysql") ~= "started" then return nil, "oxmysql is not started" end
    if PoggyCore.Util and PoggyCore.Util.CanYield and not PoggyCore.Util.CanYield() then
        return nil, "the database install must run inside a thread"
    end

    local p = promise.new()
    local settled = false
    local function settle(result, err)
        if settled then return end
        settled = true
        p:resolve({ result = result, err = err })
    end

    local okCall, callErr = pcall(function()
        -- The last two arguments are what oxmysql's own lib/MySQL.lua passes: the
        -- resource to name, and "this call is awaited". With them a failed query
        -- comes back to the callback instead of being printed as a console error.
        exports.oxmysql[method](nil, sql, params or {}, function(result, err)
            settle(result, err)
        end, SELF, true)
    end)
    if not okCall then return nil, tostring(callErr) end

    SetTimeout(DDL_TIMEOUT, function()
        settle(nil, ("no answer from the database after %d s"):format(DDL_TIMEOUT // 1000))
    end)

    local r = Citizen.Await(p)
    if r.err then
        -- oxmysql's text is "<resource> was unable to execute a query!", the query,
        -- the parameters, then the database's own message on the last line.
        local msg = tostring(r.err)
        return nil, msg:match("([^\n]+)%s*$") or msg
    end
    return r.result
end

Sql.Db = {
    query = function(sql, params) return call("query", sql, params) end,
    exec  = function(sql, params)
        local r, err = call("update", sql, params)
        if err then return nil, err end
        return tonumber(r) or 0
    end,
}

-- ---------------------------------------------------------------------------
-- Reading SQL
-- ---------------------------------------------------------------------------

--- Split a file into statements. Semicolons inside quotes, backticks and comments
--- do not end a statement. Returns statements, directives (the text after
--- "-- poggy:" on comment lines).
function Sql.Split(text)
    local statements, directives = {}, {}
    local buf, i, n = {}, 1, #text

    local function flush()
        local s = table.concat(buf):match("^%s*(.-)%s*$")
        if s ~= "" then statements[#statements + 1] = s end
        buf = {}
    end

    while i <= n do
        local c = text:sub(i, i)
        if c == "-" and text:sub(i + 1, i + 1) == "-" and (i + 2 > n or text:sub(i + 2, i + 2):match("%s")) then
            local j = text:find("\n", i, true) or (n + 1)
            local d = text:sub(i, j - 1):match("^%-%-%s*poggy:%s*(.-)%s*$")
            if d and d ~= "" then directives[#directives + 1] = d end
            buf[#buf + 1] = " "
            i = j + 1
        elseif c == "#" then
            local j = text:find("\n", i, true) or (n + 1)
            buf[#buf + 1] = " "
            i = j + 1
        elseif c == "/" and text:sub(i + 1, i + 1) == "*" then
            local j = text:find("*/", i + 2, true)
            buf[#buf + 1] = " "
            i = j and (j + 2) or (n + 1)
        elseif c == "'" or c == '"' or c == "`" then
            local j = i + 1
            while j <= n do
                local d = text:sub(j, j)
                if d == "\\" and c ~= "`" then
                    j = j + 2
                elseif d == c then
                    if text:sub(j + 1, j + 1) == c then j = j + 2 else break end
                else
                    j = j + 1
                end
            end
            buf[#buf + 1] = text:sub(i, j)
            i = j + 1
        elseif c == ";" then
            flush()
            i = i + 1
        else
            local j = text:find("[%-#/'\"`;]", i + 1) or (n + 1)
            buf[#buf + 1] = text:sub(i, j - 1)
            i = j
        end
    end
    flush()
    return statements, directives
end

--- Read an identifier (`quoted` or bare) at pos. schema.table gives the table.
local function readIdent(s, pos)
    pos = s:find("%S", pos) or (#s + 1)
    local name
    if s:sub(pos, pos) == "`" then
        local close = s:find("`", pos + 1, true)
        if not close then return nil, pos end
        name, pos = s:sub(pos + 1, close - 1), close + 1
    else
        local a, b = s:find("^[%w_%$]+", pos)
        if not a then return nil, pos end
        name, pos = s:sub(a, b), b + 1
    end
    if s:sub(pos, pos) == "." then return readIdent(s, pos + 1) end
    return name, pos
end

--- Split at commas that are not inside parentheses or quotes.
local function splitTop(s)
    local parts, depth, quote, start, i = {}, 0, nil, 1, 1
    while i <= #s do
        local c = s:sub(i, i)
        if quote then
            if c == "\\" and quote ~= "`" then i = i + 1
            elseif c == quote then quote = nil end
        elseif c == "'" or c == '"' or c == "`" then quote = c
        elseif c == "(" then depth = depth + 1
        elseif c == ")" then depth = depth - 1
        elseif c == "," and depth == 0 then
            parts[#parts + 1] = s:sub(start, i - 1):match("^%s*(.-)%s*$")
            start = i + 1
        end
        i = i + 1
    end
    parts[#parts + 1] = s:sub(start):match("^%s*(.-)%s*$")
    return parts
end

--- Words at the top level; parentheses and quotes stay inside their word.
local function words(s)
    local out, cur, depth, quote = {}, {}, 0, nil
    for i = 1, #s do
        local c = s:sub(i, i)
        if quote then
            cur[#cur + 1] = c
            if c == quote then quote = nil end
        elseif c == "'" or c == '"' or c == "`" then
            quote = c; cur[#cur + 1] = c
        elseif c == "(" then
            depth = depth + 1; cur[#cur + 1] = c
        elseif c == ")" then
            depth = depth - 1; cur[#cur + 1] = c
        elseif c:match("%s") and depth == 0 then
            if #cur > 0 then out[#out + 1] = table.concat(cur); cur = {} end
        else
            cur[#cur + 1] = c
        end
    end
    if #cur > 0 then out[#out + 1] = table.concat(cur) end
    return out
end

local TYPE_END = {
    NOT = true, NULL = true, DEFAULT = true, COMMENT = true, AFTER = true, FIRST = true,
    AUTO_INCREMENT = true, CHARACTER = true, CHARSET = true, COLLATE = true, PRIMARY = true,
    UNIQUE = true, KEY = true, ON = true, GENERATED = true, AS = true, CHECK = true,
    REFERENCES = true, INVISIBLE = true, VISIBLE = true,
}

--- The type part of a column definition: "ENUM('a','b') NOT NULL DEFAULT 'a'" -> "ENUM('a','b')".
local function columnType(def)
    local out = {}
    for _, w in ipairs(words(def)) do
        local head = w:match("^[%a_]+")
        if head and TYPE_END[head:upper()] then break end
        out[#out + 1] = w
    end
    return table.concat(out, " ")
end

local TYPE_ALIAS = {
    { "^boolean", "tinyint" }, { "^bool", "tinyint" }, { "^integer", "int" },
    { "^numeric", "decimal" }, { "^dec%(", "decimal(" }, { "^fixed", "decimal" }, { "^real", "double" },
}

--- Comparable form of a column type: lower case, no spaces, no integer display
--- widths (MySQL 8 drops them; MariaDB keeps them), aliases resolved.
function Sql.NormType(t)
    t = tostring(t or ""):lower():gsub("%s+", "")
    t = t:gsub("(%a*int)%(%d+%)", "%1")
    for _, a in ipairs(TYPE_ALIAS) do
        local replaced = t:gsub(a[1], a[2])
        t = replaced
    end
    return t
end

-- ---------------------------------------------------------------------------
-- Layout checks
-- ---------------------------------------------------------------------------

local Q_TABLE  = "SELECT 1 AS found FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ? LIMIT 1"
local Q_COLUMN = "SELECT COLUMN_TYPE AS column_type FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ? AND COLUMN_NAME = ? LIMIT 1"
local Q_INDEX  = "SELECT 1 AS found FROM information_schema.STATISTICS WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ? AND INDEX_NAME = ? LIMIT 1"

local function hasTable(db, name)
    local rows, err = db.query(Q_TABLE, { name })
    if not rows then return nil, err end
    return #rows > 0
end

--- The column's current type, false when it does not exist, nil + err on failure.
local function columnInfo(db, tbl, col)
    local rows, err = db.query(Q_COLUMN, { tbl, col })
    if not rows then return nil, err end
    if #rows == 0 then return false end
    return rows[1].column_type or rows[1].COLUMN_TYPE or ""
end

local function hasIndex(db, tbl, idx)
    local rows, err = db.query(Q_INDEX, { tbl, idx })
    if not rows then return nil, err end
    return #rows > 0
end

-- ---------------------------------------------------------------------------
-- Running statements
-- ---------------------------------------------------------------------------

local function snippet(stmt)
    local s = stmt:gsub("%s+", " ")
    if #s > 80 then s = s:sub(1, 77) .. "..." end
    return s
end

local NOT_COLUMN = {
    INDEX = true, KEY = true, UNIQUE = true, PRIMARY = true, CONSTRAINT = true,
    FOREIGN = true, FULLTEXT = true, SPATIAL = true, PARTITION = true, CHECK = true,
}

local INDEX_FORMS = {
    "^ADD%s+UNIQUE%s+INDEX%f[^%w_]", "^ADD%s+UNIQUE%s+KEY%f[^%w_]",
    "^ADD%s+FULLTEXT%s+INDEX%f[^%w_]", "^ADD%s+FULLTEXT%s+KEY%f[^%w_]",
    "^ADD%s+INDEX%f[^%w_]", "^ADD%s+KEY%f[^%w_]",
    "^ADD%s+UNIQUE%f[^%w_]", "^ADD%s+FULLTEXT%f[^%w_]",
}

local IF_NOT_EXISTS = "^%s*IF%s+NOT%s+EXISTS%f[^%w_]"

--- Run (or, when dry, plan) a list of statements. context is "install" or a
--- migration id; migrations may run what install.sql may not.
local function runStatements(db, statements, context, dry, out)
    local migration = context ~= "install"
    local tag = migration and ("migration %s: "):format(context) or ""
    local newTables = {}         -- tables a dry run would create (their ALTERs are not checked)

    local function fail(label, err)
        out.errors[#out.errors + 1] = ("%s%s — %s"):format(tag, label, tostring(err))
        return false
    end

    local function refuse(label, why)
        out.refused[#out.refused + 1] = { label = tag .. label, why = why }
        return true
    end

    --- Send one statement. key counts it in the summary.
    local function send(sql, key, label)
        if dry then
            if key == "rows" then
                out.data = out.data + 1
            else
                out.planned[#out.planned + 1] = tag .. label
            end
            return true
        end
        local affected, err = db.exec(sql)
        if not affected then return fail(label, err) end
        if key == "rows" then
            out.data = out.data + 1
            out.rows = out.rows + (tonumber(affected) or 0)
        elseif key then
            out[key] = out[key] + 1
        end
        return true
    end

    local function alterClause(tbl, clause)
        local cu = clause:upper()

        -- ADD INDEX / KEY / UNIQUE
        for _, form in ipairs(INDEX_FORMS) do
            local _, b = cu:find(form)
            if b then
                local _, y = cu:find(IF_NOT_EXISTS, b + 1)
                local name = readIdent(clause, (y or b) + 1)
                local after = clause:sub((y or b) + 1):match("^%s*(.)")
                if not name or after == "(" then
                    return refuse(("index on %s: %s"):format(tbl, snippet(clause)),
                        "give the index a name so it can be found again")
                end
                if not dry or not newTables[tbl:lower()] then
                    if not (dry and newTables[tbl:lower()]) then
                        local exists, err = hasIndex(db, tbl, name)
                        if exists == nil then return fail("read indexes of " .. tbl, err) end
                        if exists then return true end
                    end
                end
                if dry and newTables[tbl:lower()] then return true end
                local body = y and (clause:sub(1, b) .. clause:sub(y + 1)) or clause
                return send(("ALTER TABLE `%s` %s"):format(tbl, body), "indexes",
                    ("add index %s.%s"):format(tbl, name))
            end
        end

        -- ADD [COLUMN] [IF NOT EXISTS] name ...
        local _, b = cu:find("^ADD%s+COLUMN%f[^%w_]")
        local explicit = b ~= nil
        if not b then _, b = cu:find("^ADD%f[^%w_]") end
        if b then
            local _, y = cu:find(IF_NOT_EXISTS, b + 1)
            local name = readIdent(clause, (y or b) + 1)
            if name and (explicit or not NOT_COLUMN[name:upper()]) then
                if dry and newTables[tbl:lower()] then return true end
                local current, err = columnInfo(db, tbl, name)
                if current == nil then return fail("read columns of " .. tbl, err) end
                if current then return true end
                local body = y and (clause:sub(1, b) .. clause:sub(y + 1)) or clause
                return send(("ALTER TABLE `%s` %s"):format(tbl, body), "added",
                    ("add column %s.%s"):format(tbl, name))
            end
        end

        -- MODIFY [COLUMN] name TYPE ...
        local _, m = cu:find("^MODIFY%s+COLUMN%f[^%w_]")
        if not m then _, m = cu:find("^MODIFY%f[^%w_]") end
        if m then
            local name, pos = readIdent(clause, m + 1)
            if name then
                if dry and newTables[tbl:lower()] then return true end
                local target = columnType(clause:sub(pos))
                local current, err = columnInfo(db, tbl, name)
                if current == nil then return fail("read columns of " .. tbl, err) end
                if current == false then
                    return fail(("change %s.%s"):format(tbl, name), "that column does not exist")
                end
                if Sql.NormType(current) == Sql.NormType(target) then return true end
                return send(("ALTER TABLE `%s` %s"):format(tbl, clause), "modified",
                    ("change %s.%s from %s to %s"):format(tbl, name, current, target))
            end
        end

        if migration then
            return send(("ALTER TABLE `%s` %s"):format(tbl, clause), "other",
                ("alter %s: %s"):format(tbl, snippet(clause)))
        end
        return refuse(("alter %s: %s"):format(tbl, snippet(clause)),
            "not repeatable on every start; put it in sql/migrations/")
    end

    for _, stmt in ipairs(statements) do
        local up = stmt:upper()

        if up:find("^DELIMITER%f[^%w_]") then
            refuse(snippet(stmt), "DELIMITER is not supported; write plain statements")

        elseif up:find("^CREATE%s+TABLE%f[^%w_]") then
            local _, b = up:find("^CREATE%s+TABLE%f[^%w_]")
            local _, y = up:find(IF_NOT_EXISTS, b + 1)
            local name = readIdent(stmt, (y or b) + 1)
            if not name then
                refuse(snippet(stmt), "could not read the table name")
            else
                local exists, err = hasTable(db, name)
                if exists == nil then
                    fail("read tables", err)
                elseif not exists then
                    if dry then newTables[name:lower()] = true end
                    send(stmt, "created", "create table " .. name)
                end
            end

        elseif up:find("^CREATE%s+[%u%s]-INDEX%f[^%w_]") then
            local _, b = up:find("^CREATE%s+[%u%s]-INDEX%f[^%w_]")
            local _, y = up:find(IF_NOT_EXISTS, b + 1)
            local name, pos = readIdent(stmt, (y or b) + 1)
            local _, on = up:find("^%s*ON%f[^%w_]", pos)
            local tbl = on and readIdent(stmt, on + 1)
            if not name or not tbl then
                refuse(snippet(stmt), "could not read the index or table name")
            elseif not (dry and newTables[tbl:lower()]) then
                local exists, err = hasIndex(db, tbl, name)
                if exists == nil then
                    fail("read indexes of " .. tbl, err)
                elseif not exists then
                    local body = y and (stmt:sub(1, b) .. stmt:sub(y + 1)) or stmt
                    send(body, "indexes", ("add index %s.%s"):format(tbl, name))
                end
            end

        elseif up:find("^ALTER%s+TABLE%f[^%w_]") then
            local _, b = up:find("^ALTER%s+TABLE%f[^%w_]")
            local tbl, pos = readIdent(stmt, b + 1)
            if not tbl then
                refuse(snippet(stmt), "could not read the table name")
            else
                local present = dry and newTables[tbl:lower()]
                if not present then
                    local exists, err = hasTable(db, tbl)
                    if exists == nil then
                        fail("read tables", err)
                    elseif not exists then
                        fail("alter " .. tbl, "that table does not exist")
                    else
                        present = true
                    end
                end
                if present then
                    for _, clause in ipairs(splitTop(stmt:sub(pos))) do
                        if clause ~= "" then alterClause(tbl, clause) end
                        if #out.errors > 0 then break end
                    end
                end
            end

        elseif up:find("^INSERT%s+IGNORE%f[^%w_]") then
            send(stmt, "rows", snippet(stmt))

        elseif up:find("^UPDATE%f[^%w_]") then
            send(stmt, "rows", snippet(stmt))

        elseif up:find("^SET%f[^%w_]") then
            send(stmt, nil, snippet(stmt))

        elseif up:find("^SELECT%f[^%w_]") or up:find("^SHOW%f[^%w_]") then
            -- A report for someone importing the file by hand; nothing to do here.

        elseif migration then
            send(stmt, "other", snippet(stmt))

        elseif up:find("^INSERT%f[^%w_]") or up:find("^REPLACE%f[^%w_]") then
            if up:find("ON%s+DUPLICATE%s+KEY%s+UPDATE") then
                refuse(snippet(stmt), "would overwrite owners' rows on every start; use INSERT IGNORE")
            else
                refuse(snippet(stmt), "use INSERT IGNORE so a re-run adds nothing twice")
            end

        else
            refuse(snippet(stmt), "not repeatable on every start; put it in sql/migrations/")
        end

        if #out.errors > 0 then break end
    end
end

local TRACKING = [[
CREATE TABLE IF NOT EXISTS `poggy_migrations` (
    `resource`   VARCHAR(64) NOT NULL,
    `migration`  VARCHAR(64) NOT NULL,
    `applied_at` TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`resource`, `migration`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4]]

--- The script's poggy_id (sv_identity.lua), or the folder name without it.
local function identityOf(resource)
    local I = PoggyCore.Identity
    local id = I and I.Id(resource) or resource
    return (type(id) == "string" and id ~= "") and id or resource
end

local function runMigrations(db, resource, dry, out)
    local files = {}
    for i = 1, 999 do
        local text = LoadResourceFile(resource, MIGRATION:format(i))
        if not text then break end
        files[#files + 1] = { id = ("%03d"):format(i), text = text }
    end
    if #files == 0 then return end

    local tracked, err = hasTable(db, "poggy_migrations")
    if tracked == nil then
        out.errors[#out.errors + 1] = "read tables — " .. tostring(err)
        return
    end
    if not tracked and not dry then
        local ok, e = db.exec(TRACKING)
        if not ok then
            out.errors[#out.errors + 1] = "create table poggy_migrations — " .. tostring(e)
            return
        end
        tracked = true
    end

    -- 0.13.0: a migration is recorded under the script's poggy_id, so renaming
    -- the folder does not run it again. Rows recorded under the folder name
    -- (poggy_core 0.12.x, or a folder renamed before poggy_id existed) still count.
    local id = identityOf(resource)
    local applied = {}
    if tracked then
        local keys = id ~= resource and { id, resource } or { id }
        for _, key in ipairs(keys) do
            local rows, e = db.query("SELECT `migration` FROM `poggy_migrations` WHERE `resource` = ?", { key })
            if not rows then
                out.errors[#out.errors + 1] = "read poggy_migrations — " .. tostring(e)
                return
            end
            for _, r in ipairs(rows) do applied[tostring(r.migration)] = true end
        end
    end

    for _, f in ipairs(files) do
        if not applied[f.id] then
            local statements, directives = Sql.Split(f.text)
            local onlyIf
            for _, d in ipairs(directives) do
                onlyIf = onlyIf or d:match("^only%-if%-table%s+`?([%w_%$]+)`?")
            end

            local nothingToDo = false
            if onlyIf then
                local has, e = hasTable(db, onlyIf)
                if has == nil then
                    out.errors[#out.errors + 1] = ("migration %s: read tables — %s"):format(f.id, tostring(e))
                    return
                end
                nothingToDo = not has
            end

            if dry then
                out.planned[#out.planned + 1] = nothingToDo
                    and ("migration %s: nothing to migrate (no %s table); recorded as done"):format(f.id, onlyIf)
                    or ("run migration %s (%d statement%s)"):format(f.id, #statements, #statements == 1 and "" or "s")
            else
                if not nothingToDo then
                    runStatements(db, statements, f.id, false, out)
                    if #out.errors > 0 then return end      -- not recorded: runs again next start
                end
                local ok, e = db.exec("INSERT IGNORE INTO `poggy_migrations` (`resource`, `migration`) VALUES (?, ?)",
                    { id, f.id })
                if not ok then
                    out.errors[#out.errors + 1] = ("migration %s: record it — %s"):format(f.id, tostring(e))
                    return
                end
                out.migrations[#out.migrations + 1] = nothingToDo and (f.id .. " (nothing to migrate)") or f.id
            end
        end
    end
end

-- ---------------------------------------------------------------------------
-- Reporting
-- ---------------------------------------------------------------------------

local function describe(out)
    local parts = {}
    local function add(n, one, many)
        if n > 0 then parts[#parts + 1] = (n == 1 and one or many):format(n) end
    end
    add(out.created,  "created %d table",        "created %d tables")
    add(out.added,    "added %d column",         "added %d columns")
    add(out.indexes,  "added %d index",          "added %d indexes")
    add(out.modified, "changed %d column type",  "changed %d column types")
    add(out.rows,     "wrote %d row",            "wrote %d rows")
    if #out.migrations > 0 then
        parts[#parts + 1] = "ran migration " .. table.concat(out.migrations, ", ")
    end
    return table.concat(parts, ", ")
end

local function report(out, dry, say)
    local name = ("%-24s"):format(out.resource)
    local emit = say or function(msg) print(prefix() .. msg) end

    for _, r in ipairs(out.refused) do
        emit(("^3⚠️  %s database: skipped %s ^9— %s^7"):format(name, r.label, r.why))
    end
    for _, e in ipairs(out.errors) do
        emit(("^1❌ %s database: %s^7"):format(name, e))
    end
    if #out.errors > 0 then
        emit(("   ^9nothing after it ran. Fix it, then restart %s or run: poggycore sql install %s^7")
            :format(out.resource, out.resource))
    end

    if dry then
        if #out.planned == 0 then
            if #out.errors == 0 then emit(("✅ %s database: nothing to change"):format(name)) end
        else
            emit(("⬆️  %s database: %d change%s to make"):format(name, #out.planned, #out.planned == 1 and "" or "s"))
            for _, p in ipairs(out.planned) do emit("   ^9" .. p .. "^7") end
        end
        if out.data > 0 then
            emit(("   ^9plus %d data statement%s (INSERT IGNORE / UPDATE) that run on every start and only fill in what is missing^7")
                :format(out.data, out.data == 1 and "" or "s"))
        end
        return
    end

    local text = describe(out)
    if text ~= "" then
        emit(("^2✅ %s database: %s^7"):format(name, text))
    elseif say and #out.errors == 0 then
        emit(("✅ %s database: up to date"):format(name))
    end
end

-- ---------------------------------------------------------------------------
-- Public
-- ---------------------------------------------------------------------------

--- Run a resource's sql/install.sql and pending migrations (dry: plan only).
--- say: a function for command output; without it only changes and problems are
--- printed, so a normal start with nothing to do is silent.
function Sql.Install(resource, dry, say)
    local out = {
        resource = resource, ok = true,
        created = 0, added = 0, indexes = 0, modified = 0, other = 0, rows = 0, data = 0,
        migrations = {}, planned = {}, refused = {}, errors = {},
    }

    local text = LoadResourceFile(resource, INSTALL)
    if not text then
        out.none = true
        return out
    end
    if running[resource] then
        out.ok, out.busy = false, true
        out.errors[1] = "an install for this resource is already running"
        report(out, dry, say)
        return out
    end

    running[resource] = true
    local okRun, crash = pcall(function()
        local db = Sql.Db
        runStatements(db, (Sql.Split(text)), "install", dry, out)
        if #out.errors == 0 then runMigrations(db, resource, dry, out) end
    end)
    running[resource] = nil

    if not okRun then out.errors[#out.errors + 1] = "runner error — " .. tostring(crash) end
    out.ok = #out.errors == 0
    report(out, dry, say)
    return out
end

--- The sql.install / sql.check verbs. resource is the caller, from the dispatcher.
function Sql.Verb(resource, dry)
    local Err = PoggyCore.Err or {}
    if type(resource) ~= "string" or resource == "" then
        return false, nil, Err.BAD_ARG or "bad_argument"
    end
    if not dry and PoggyCoreConfig and PoggyCoreConfig.Sql and PoggyCoreConfig.Sql.AutoInstall == false then
        return true, { resource = resource, disabled = true }, nil
    end
    local out = Sql.Install(resource, dry, nil)
    if out.ok then return true, out, nil end
    return false, out, Err.FRAMEWORK_ERR or "framework_error"
end

local function isPoggyScript(name)
    local manifest = LoadResourceFile(name, "fxmanifest.lua")
    return manifest ~= nil and manifest:find(BRIDGE, 1, true) ~= nil
end

--- Every Poggy script (loads the bridge) that has sql/install.sql, sorted.
function Sql.Resources()
    local list = {}
    for i = 0, GetNumResources() - 1 do
        local name = GetResourceByFindIndex(i)
        if name and name ~= SELF and isPoggyScript(name) and LoadResourceFile(name, INSTALL) then
            list[#list + 1] = name
        end
    end
    table.sort(list)
    return list
end

--- poggycore sql check|install <resource|all>
function Sql.Command(mode, target, say)
    local dry = mode == "check"
    if not target or target == "" then
        say("name a resource, or all: poggycore sql " .. mode .. " <resource|all>")
        return
    end

    local names
    if target:lower() == "all" then
        names = Sql.Resources()
        if #names == 0 then
            say("➖ no Poggy script with " .. INSTALL .. " is on this server.")
            return
        end
    else
        -- A poggy_id works as well as a folder name.
        local I = PoggyCore.Identity
        if I and not LoadResourceFile(target, "fxmanifest.lua") then
            I.Refresh()
            local okFind, folder = pcall(I.Locate, I.Resolve(target))
            if okFind and folder then target = folder end
        end
        if not LoadResourceFile(target, "fxmanifest.lua") then
            say(("^1❌ there is no resource called %s.^7"):format(target))
            return
        end
        if not isPoggyScript(target) then
            say(("^1❌ %s is not a Poggy script. poggy_core only runs SQL for scripts that load its bridge.^7"):format(target))
            return
        end
        if not LoadResourceFile(target, INSTALL) then
            say(("➖ %s has no %s."):format(target, INSTALL))
            return
        end
        names = { target }
    end

    local failed = 0
    for _, name in ipairs(names) do
        local out = Sql.Install(name, dry, say)
        if not out.ok then failed = failed + 1 end
    end
    if #names > 1 then
        say(("%d script%s checked%s."):format(#names, #names == 1 and "" or "s",
            failed > 0 and (", ^1" .. failed .. " with errors^7") or ""))
    end
end
