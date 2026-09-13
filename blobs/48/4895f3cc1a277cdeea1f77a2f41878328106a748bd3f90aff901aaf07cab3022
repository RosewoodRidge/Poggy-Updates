--[[
    poggy_core — config merge.

    When a resource updates, its config.lua changes too, but the server owner's
    copy holds their settings. This brings the owner's file in line with the new
    version's STRUCTURE while keeping every one of their VALUES:

        a setting we added          -> the line is inserted, with its default
        a setting we removed        -> the line is removed
        a setting whose shape we    -> that setting's value is replaced
          changed (number -> table)
        a setting only they have    -> left alone
        a setting in both           -> their value is kept, always

    "Removed by us" is told apart from "added by the owner" with the config as
    it was shipped in the version they are running (the base). Without a base,
    nothing is removed or replaced: lines are only added, and anything that
    looks obsolete is reported instead.

    Lists of entries (locations, stalls, job tables, anything with positional or
    [bracketed] keys) are one opaque value in a config. They are never merged
    item by item, because their contents are the owner's data: re-adding a job
    they deliberately removed would hand that job access again.

    Translation and locale files are different: their ["key"] = "text" entries
    are strings the script needs. Merge them with { dictKeys = true } and a
    bracketed string key counts as a setting of its own, so a string we added
    arrives and one the owner deleted comes back.

    The result is loaded as Lua and checked before it is returned: every setting
    the new version defines must exist, and every owner value that was not
    deliberately removed or replaced must be unchanged. If any check fails, no
    merged text is returned and the caller keeps the owner's file untouched.

    Plain Lua, no natives, so it can be tested outside the server.

        local merged, report = PoggyCore.ConfigMerge.Merge(baseText, newText, userText, opts)
        -- opts:   { dictKeys = true } for translation and locale files
        -- merged: string, or nil on failure
        -- report: list of human-readable lines (what changed, or why it failed)
]]

PoggyCore = PoggyCore or {}

local M = {}
PoggyCore.ConfigMerge = M

-- ---------------------------------------------------------------------------
-- Lexer: just enough Lua to find where things start and end
-- ---------------------------------------------------------------------------

local function lex(src)
    local toks, i, n = {}, 1, #src
    local function push(t, s, e) toks[#toks + 1] = { t = t, s = s, e = e } end

    while i <= n do
        local c = src:sub(i, i)
        if c:match("%s") then
            i = i + 1
        elseif src:sub(i, i + 1) == "--" then
            local lvl = src:match("^%[(=*)%[", i + 2)
            if lvl then
                local close = "]" .. lvl .. "]"
                local e = src:find(close, i + 4 + #lvl, true)
                i = e and (e + #close) or (n + 1)
            else
                local e = src:find("\n", i, true)
                i = e or (n + 1)
            end
        elseif c == '"' or c == "'" or c == "`" then
            local j = i + 1
            while j <= n do
                local d = src:sub(j, j)
                if d == "\\" then
                    j = j + 2
                elseif d == c or d == "\n" then
                    break
                else
                    j = j + 1
                end
            end
            push("string", i, math.min(j, n))
            i = j + 1
        elseif c == "[" and src:match("^%[=*%[", i) then
            local lvl = src:match("^%[(=*)%[", i)
            local close = "]" .. lvl .. "]"
            local e = src:find(close, i + 2 + #lvl, true)
            local stop = e and (e + #close - 1) or n
            push("string", i, stop)
            i = stop + 1
        elseif c:match("[%a_]") then
            local s, e = src:find("^[%w_]+", i)
            push("name", s, e)
            i = e + 1
        elseif c:match("%d") or (c == "." and src:sub(i + 1, i + 1):match("%d")) then
            local s, e = src:find("^0[xX][%x%.]*[pP]?[%+%-]?%x*", i)
            if not s then s, e = src:find("^%d*%.?%d*[eE][%+%-]?%d+", i) end
            if not s then s, e = src:find("^%d*%.?%d*", i) end
            push("number", s, e)
            i = e + 1
        else
            local two = src:sub(i, i + 1)
            if src:sub(i, i + 2) == "..." then
                push("op", i, i + 2); i = i + 3
            elseif two == "==" or two == "~=" or two == "<=" or two == ">=" or two == ".."
                or two == "::" or two == "//" or two == "<<" or two == ">>" then
                push("op", i, i + 1); i = i + 2
            else
                push("op", i, i); i = i + 1
            end
        end
    end
    return toks
end

-- ---------------------------------------------------------------------------
-- Parser: finds every setting, its key, value span and separator
-- ---------------------------------------------------------------------------

local KEYWORDS = {}
for w in ("and break do else elseif end false for function goto if in local nil not or repeat return then true until while")
    :gmatch("%a+") do KEYWORDS[w] = true end

local UNOP = { ["-"] = true, ["not"] = true, ["#"] = true, ["~"] = true }
local BINOP = {}
for _, o in ipairs({ "+", "-", "*", "/", "//", "%", "^", "..", "==", "~=", "<", "<=", ">", ">=",
    "and", "or", "&", "|", "~", "<<", ">>" }) do BINOP[o] = true end

local function tk(S, k) return S.toks[S.p + (k or 0)] end
local function tv(S, k)
    local t = tk(S, k)
    return t and S.src:sub(t.s, t.e)
end

--- The text of a quoted or long-bracket string literal, or nil (e.g. a `hash`).
local function unquote(lit)
    local q = lit:sub(1, 1)
    if (q == '"' or q == "'") and #lit >= 2 and lit:sub(-1) == q then
        local body = lit:sub(2, -2)
        if body:find("\\", 1, true) then
            body = (body:gsub("\\(.)", { n = "\n", t = "\t", ["\\"] = "\\", ['"'] = '"', ["'"] = "'" }))
        end
        return body
    end
    local lvl = lit:match("^%[(=*)%[")
    if lvl then return lit:sub(#lvl + 3, -(#lvl + 3)) end
    return nil
end

--- A key path segment for a bracketed string key, identical however it was quoted.
local function bracketSegment(key)
    return '["' .. key:gsub("\\", "\\\\"):gsub('"', '\\"') .. '"]'
end

local function extend(list, item)
    local out = {}
    for i, v in ipairs(list or {}) do out[i] = v end
    out[#out + 1] = item
    return out
end

--- From an opener (function, if, do, repeat), past its matching end/until.
local function skipBlock(S)
    local depth = 0
    while tk(S) do
        local t = tk(S)
        if t.t == "name" then
            local v = tv(S)
            if v == "function" or v == "if" or v == "do" or v == "repeat" then
                depth = depth + 1
            elseif v == "end" or v == "until" then
                depth = depth - 1
                if depth == 0 then
                    S.p = S.p + 1
                    return t.e
                end
            end
        end
        S.p = S.p + 1
    end
    error("a block is never closed")
end

local parseExpr, parseTable

local function parseParen(S)
    S.p = S.p + 1
    while tk(S) and tv(S) ~= ")" do
        parseExpr(S)
        if tv(S) == "," then S.p = S.p + 1 end
    end
    local t = tk(S)
    if not t then error("a parenthesis is never closed") end
    S.p = S.p + 1
    return t.e
end

local function parseSuffix(S, last)
    while tk(S) do
        local t, v = tk(S), tv(S)
        local nxt = tk(S, 1)
        if (v == "." or v == ":") and t.t == "op" and nxt and nxt.t == "name" then
            S.p = S.p + 2
            last = nxt.e
        elseif v == "[" and t.t == "op" then
            S.p = S.p + 1
            parseExpr(S)
            if tv(S) == "]" then
                last = tk(S).e
                S.p = S.p + 1
            end
        elseif v == "(" and t.t == "op" then
            last = parseParen(S)
        elseif t.t == "string" then
            S.p = S.p + 1
            last = t.e
        elseif v == "{" and t.t == "op" then
            last = parseTable(S, nil)
        else
            break
        end
    end
    return last
end

function parseExpr(S)
    local t = tk(S)
    if not t then error("unexpected end of file") end
    local v = tv(S)
    local last

    if t.t ~= "string" and UNOP[v] then
        S.p = S.p + 1
        return parseExpr(S)
    end

    if t.t == "string" or t.t == "number" or v == "nil" or v == "true" or v == "false" or v == "..." then
        S.p = S.p + 1
        last = t.e
    elseif v == "function" then
        last = skipBlock(S)
    elseif v == "{" and t.t == "op" then
        last = parseTable(S, nil)
    elseif v == "(" and t.t == "op" then
        last = parseSuffix(S, parseParen(S))
    elseif t.t == "name" then
        S.p = S.p + 1
        last = parseSuffix(S, t.e)
    else
        S.p = S.p + 1
        last = t.e
    end

    local nt = tk(S)
    if nt and nt.t ~= "string" and BINOP[tv(S)] then
        S.p = S.p + 1
        last = parseExpr(S)
    end
    return last
end

local function classify(S)
    if tv(S) == "function" then return "function" end
    return "value"
end

--- At "{". Returns closePos, kind ("map" | "data" | "empty"), children, openPos.
--- path/parts/chain describe the table itself; nil when it is not being indexed.
function parseTable(S, path, parts, chain)
    local openTok = tk(S)
    S.p = S.p + 1
    local children, isData, count, order = {}, false, 0, 0

    while tk(S) and not (tv(S) == "}" and tk(S).t == "op") do
        local t = tk(S)
        local field
        local key, segment

        -- A setting's key: `name =`, or in dictionary mode `["text"] =`.
        if t.t == "name" and not KEYWORDS[tv(S)] and tv(S, 1) == "=" then
            key = S.src:sub(t.s, t.e)
            segment = "." .. key
            S.p = S.p + 2
        elseif S.dict and tv(S) == "[" and t.t == "op" and tk(S, 1) and tk(S, 1).t == "string"
            and tv(S, 2) == "]" and tv(S, 3) == "=" then
            key = unquote(tv(S, 1))
            if key then
                segment = bracketSegment(key)
                S.p = S.p + 4
            end
        end

        if key then
            order = order + 1
            local childPath = path and (path .. segment) or segment
            local vt = tk(S)
            field = {
                path = childPath, parent = path, group = path, order = order,
                keyS = t.s, valS = vt and vt.s,
                parts = extend(parts, key), chain = extend(chain, childPath),
            }
            if vt and tv(S) == "{" and vt.t == "op" then
                local close, kind, sub, openPos = parseTable(S, childPath, field.parts, field.chain)
                field.valE, field.kind, field.open = close, kind, openPos
                children[#children + 1] = field
                for _, c in ipairs(sub) do children[#children + 1] = c end
            else
                field.kind = classify(S)
                field.valE = parseExpr(S)
                children[#children + 1] = field
            end
        elseif tv(S) == "[" and t.t == "op" then
            isData = true
            S.p = S.p + 1
            parseExpr(S)
            if tv(S) == "]" then S.p = S.p + 1 end
            if tv(S) == "=" then S.p = S.p + 1 end
            parseExpr(S)
        else
            isData = true
            parseExpr(S)
        end

        count = count + 1
        local sep = tk(S)
        if sep and sep.t == "op" and (tv(S) == "," or tv(S) == ";") then
            if field then field.sepE = sep.e end
            S.p = S.p + 1
        end
    end

    local closeTok = tk(S)
    if not closeTok then error("a table is never closed") end
    S.p = S.p + 1

    local kind = isData and "data" or (count == 0 and "empty" or "map")
    return closeTok.e, kind, (kind == "map") and children or {}, openTok.s
end

--- Every setting in a chunk, top-level statements and table fields alike.
local function parseChunk(src, dict)
    local S = { src = src, toks = lex(src), p = 1, dict = dict }
    local entries, order = {}, 0

    while tk(S) do
        local t, v = tk(S), tv(S)

        if t.t == "name" and not KEYWORDS[v] then
            local parts, q = { v }, S.p + 1
            while S.toks[q] and S.toks[q + 1]
                and S.src:sub(S.toks[q].s, S.toks[q].e) == "." and S.toks[q].t == "op"
                and S.toks[q + 1].t == "name" do
                parts[#parts + 1] = S.src:sub(S.toks[q + 1].s, S.toks[q + 1].e)
                q = q + 2
            end
            local eq = S.toks[q]
            if eq and eq.t == "op" and S.src:sub(eq.s, eq.e) == "=" then
                S.p = q + 1
                order = order + 1
                local chain, acc = {}, nil
                for _, part in ipairs(parts) do
                    acc = acc and (acc .. "." .. part) or part
                    chain[#chain + 1] = acc
                end
                local path = acc
                local vt = tk(S)
                if not vt then error("an assignment to " .. path .. " has no value") end
                local e = {
                    path = path, parent = "(top)", group = "(top)", order = order,
                    keyS = t.s, valS = vt.s, parts = parts, chain = chain,
                }
                if tv(S) == "{" and vt.t == "op" then
                    local close, kind, sub, openPos = parseTable(S, path, parts, chain)
                    e.valE, e.kind, e.open = close, kind, openPos
                    entries[#entries + 1] = e
                    for _, c in ipairs(sub) do entries[#entries + 1] = c end
                else
                    e.kind = classify(S)
                    e.valE = parseExpr(S)
                    entries[#entries + 1] = e
                end
                if tk(S) and tv(S) == ";" then
                    e.sepE = tk(S).e
                    S.p = S.p + 1
                end
            else
                parseExpr(S)
            end
        elseif v == "local" then
            S.p = S.p + 1
            if tv(S) == "function" then
                skipBlock(S)
            else
                while tk(S) and tk(S).t == "name" do
                    S.p = S.p + 1
                    if tv(S) == "<" then
                        while tk(S) and tv(S) ~= ">" do S.p = S.p + 1 end
                        S.p = S.p + 1
                    end
                    if tv(S) == "," then S.p = S.p + 1 else break end
                end
                if tv(S) == "=" then
                    S.p = S.p + 1
                    parseExpr(S)
                    while tv(S) == "," do
                        S.p = S.p + 1
                        parseExpr(S)
                    end
                end
            end
        elseif v == "function" or v == "if" or v == "do" or v == "repeat" then
            skipBlock(S)
        elseif v == "for" or v == "while" then
            while tk(S) and tv(S) ~= "do" do S.p = S.p + 1 end
            skipBlock(S)
        elseif v == "return" then
            S.p = S.p + 1
            if tk(S) and not KEYWORDS[tv(S)] then parseExpr(S) end
        else
            S.p = S.p + 1
        end
    end
    return entries
end

local function index(src, dict)
    local list = parseChunk(src, dict)
    local map = {}
    for i, e in ipairs(list) do
        e.idx = i
        if not map[e.path] then map[e.path] = e end
    end
    return { src = src, list = list, map = map }
end

-- ---------------------------------------------------------------------------
-- Text helpers
-- ---------------------------------------------------------------------------

local function lineStart(src, pos)
    local i = pos
    while i > 1 and src:sub(i - 1, i - 1) ~= "\n" do i = i - 1 end
    return i
end

local function lineEnd(src, pos)
    return src:find("\n", pos, true) or #src
end

local function cleanBefore(src, pos)
    return src:sub(lineStart(src, pos), pos - 1):match("^%s*$") ~= nil
end

local function cleanAfter(src, pos)
    local rest = src:sub(pos + 1, lineEnd(src, pos + 1))
    return rest:match("^%s*$") ~= nil or rest:match("^%s*%-%-") ~= nil
end

local function spanEnd(e) return e.sepE or e.valE end

--- Comment lines sitting directly above a line, stopping at a blank line or a
--- section banner.
local function attachedComments(src, ls)
    local start = ls
    while start > 1 do
        local prevEnd = start - 1
        local prevStart = lineStart(src, prevEnd)
        local line = src:sub(prevStart, prevEnd - 1)
        if line:match("^%s*%-%-") and not line:find("===", 1, true) and not line:find("%-%-%[=*%[") then
            start = prevStart
        else
            break
        end
    end
    return start
end

--- The text to insert for a setting from the new file. Returns text, isWholeLines.
local function blockText(N, n, U)
    local src = N.src
    local stop = spanEnd(n)
    if cleanBefore(src, n.keyS) and cleanAfter(src, stop) then
        local ls = lineStart(src, n.keyS)
        local comments = ""
        local cs = attachedComments(src, ls)
        if cs < ls then
            for line in src:sub(cs, ls - 1):gmatch("[^\n]*\n") do
                local trimmed = line:match("^%s*(.-)%s*$")
                if trimmed ~= "" and not U.src:find(trimmed, 1, true) then
                    comments = comments .. line
                end
            end
        end
        local le = lineEnd(src, stop)
        local body
        if n.sepE or n.group == "(top)" then
            body = src:sub(ls, le)
        else
            body = src:sub(ls, n.valE) .. "," .. src:sub(n.valE + 1, le)
        end
        if body:sub(-1) ~= "\n" then body = body .. "\n" end
        return comments .. body, true
    end
    return src:sub(n.keyS, n.valE), false
end

local function newlineBefore(src, pos)
    if pos > 1 and src:sub(pos - 1, pos - 1) ~= "\n" then return "\n" end
    return ""
end

-- ---------------------------------------------------------------------------
-- Planning
-- ---------------------------------------------------------------------------

local function family(e)
    if e.kind == "map" or e.kind == "data" or e.kind == "empty" then return "table" end
    return e.kind
end

local function shapeDiffers(a, b)
    if family(a) ~= family(b) then return true end
    if family(a) == "table" and a.kind ~= "empty" and b.kind ~= "empty" and a.kind ~= b.kind then
        return true
    end
    return false
end

--- Edits that insert new setting n into the user's file, or nil, reason.
local function planAdd(N, U, n)
    local src = U.src

    if n.group == "(top)" then
        local text = blockText(N, n, U)
        for i = n.idx - 1, 1, -1 do
            local c = N.list[i]
            if c.group == "(top)" then
                local u = U.map[c.path]
                if u and u.group == "(top)" then
                    local pos = lineEnd(src, spanEnd(u)) + 1
                    return { { pos, pos - 1, newlineBefore(src, pos) .. text } }
                end
            end
        end
        for i = n.idx + 1, #N.list do
            local c = N.list[i]
            if c.group == "(top)" then
                local u = U.map[c.path]
                if u and u.group == "(top)" then
                    local pos = lineStart(src, u.keyS)
                    return { { pos, pos - 1, text } }
                end
            end
        end
        local pos = #src + 1
        return { { pos, pos - 1, newlineBefore(src, pos) .. text } }
    end

    local parentU = U.map[n.parent]
    if not parentU or not parentU.open or (parentU.kind ~= "map" and parentU.kind ~= "empty") then
        return nil, ("its parent %s is not a settings table in your file"):format(tostring(n.parent))
    end

    local text, whole = blockText(N, n, U)
    local piece = whole and N.src:sub(n.keyS, n.valE) or text

    for i = n.idx - 1, 1, -1 do
        local c = N.list[i]
        if c.group == n.group and c.path ~= n.path then
            local u = U.map[c.path]
            if u and u.group == n.parent then
                local edits = {}
                local stop = spanEnd(u)
                if not u.sepE then
                    edits[#edits + 1] = { u.valE + 1, u.valE, ",", comma = true }
                end
                if whole and cleanAfter(src, stop) then
                    local pos = lineEnd(src, stop) + 1
                    edits[#edits + 1] = { pos, pos - 1, newlineBefore(src, pos) .. text }
                else
                    edits[#edits + 1] = { stop + 1, stop, " " .. piece .. "," }
                end
                return edits
            end
        end
    end

    for i = n.idx + 1, #N.list do
        local c = N.list[i]
        if c.group == n.group then
            local u = U.map[c.path]
            if u and u.group == n.parent then
                if whole and cleanBefore(src, u.keyS) then
                    local pos = lineStart(src, u.keyS)
                    return { { pos, pos - 1, text } }
                end
                return { { u.keyS, u.keyS - 1, piece .. ", " } }
            end
        end
    end

    local open = parentU.open
    if whole and cleanAfter(src, open) then
        local pos = lineEnd(src, open) + 1
        return { { pos, pos - 1, newlineBefore(src, pos) .. text } }
    end
    return { { open + 1, open, " " .. piece .. "," } }
end

local function planRemove(U, u)
    local src = U.src
    local stop = spanEnd(u)
    if cleanBefore(src, u.keyS) and cleanAfter(src, stop) then
        return { lineStart(src, u.keyS), lineEnd(src, stop), "" }
    end
    local e2 = stop
    while src:sub(e2 + 1, e2 + 1) == " " do e2 = e2 + 1 end
    return { u.keyS, e2, "" }
end

local function applyEdits(src, edits)
    table.sort(edits, function(a, b)
        if a[1] ~= b[1] then return a[1] > b[1] end
        local la, lb = a[2] - a[1], b[2] - b[1]
        if la ~= lb then return la > lb end
        return a.seq > b.seq
    end)
    local limit = math.huge
    for _, e in ipairs(edits) do
        if e[2] >= limit then
            return nil, "two changes overlap"
        end
        src = src:sub(1, e[1] - 1) .. e[3] .. src:sub(e[2] + 1)
        limit = e[1]
    end
    return src
end

-- ---------------------------------------------------------------------------
-- Verification: run the configs and compare values
-- ---------------------------------------------------------------------------

local function makeStub()
    local s
    s = setmetatable({}, {
        __index = function() return s end,
        __call = function(_, ...) return ... end,
    })
    return s
end

local function evaluate(text, name)
    local env = {}
    env.math = setmetatable({ random = function(a) return a or 0 end }, { __index = math })
    setmetatable(env, {
        __index = function(_, k)
            local g = _G[k]
            if g ~= nil then return g end
            return makeStub()
        end,
    })
    local fn, err = load(text, "=" .. name, "t", env)
    if not fn then return nil, err end
    local ok, runErr = pcall(fn)
    if not ok then return nil, runErr end
    return env
end

--- The value a setting holds once the chunk has run, walked key by key.
local function valueAt(env, e)
    local cur = env
    for _, part in ipairs(e.parts) do
        if type(cur) ~= "table" then return nil end
        cur = rawget(cur, part)
    end
    return cur
end

local function same(a, b, depth)
    depth = depth or 0
    if depth > 30 then return true end
    local ta, tb = type(a), type(b)
    if ta ~= tb then return false end
    if ta == "function" then return true end
    if ta ~= "table" then return a == b end
    for k, v in pairs(a) do
        if not same(v, rawget(b, k), depth + 1) then return false end
    end
    for k in pairs(b) do
        if rawget(a, k) == nil then return false end
    end
    return true
end

--- Is e, or anything it sits inside, in the set?
local function underAny(e, set)
    for _, p in ipairs(e.chain) do
        if set[p] then return true end
    end
    return false
end

--- Does anything in the list sit strictly inside e?
local function containsAny(e, list)
    for _, x in ipairs(list) do
        if x.path ~= e.path then
            for _, p in ipairs(x.chain) do
                if p == e.path then return true end
            end
        end
    end
    return false
end

-- ---------------------------------------------------------------------------
-- Merge
-- ---------------------------------------------------------------------------

local function containedIn(spans, s, e)
    for _, sp in ipairs(spans) do
        if s >= sp[1] and e <= sp[2] and not (s == sp[1] and e == sp[2]) then return true end
    end
    return false
end

function M.Merge(baseText, newText, userText, opts)
    local report = {}
    local dict = opts and opts.dictKeys and true or false

    local okN, N = pcall(index, newText, dict)
    if not okN then return nil, { "the new config could not be read: " .. tostring(N) } end
    local okU, U = pcall(index, userText, dict)
    if not okU then return nil, { "your config could not be read: " .. tostring(U) } end
    local B
    if baseText then
        local okB, b = pcall(index, baseText, dict)
        if okB then
            B = b
        else
            report[#report + 1] = "note: the previous version's config could not be read, so nothing will be removed"
        end
    else
        report[#report + 1] = "note: the previous version's config is unknown, so nothing will be removed"
    end

    local edits, seq = {}, 0
    local claimedN, claimedU = {}, {}
    local changed, skipped = {}, {}      -- path -> entry / true
    local touched = {}                   -- entries added, removed or replaced

    local function push(list)
        for _, e in ipairs(list) do
            seq = seq + 1
            e.seq = seq
            edits[#edits + 1] = e
        end
    end

    -- Settings in the owner's file that the new version no longer has.
    for _, u in ipairs(U.list) do
        if not N.map[u.path] and not containedIn(claimedU, u.keyS, spanEnd(u)) then
            if B and B.map[u.path] then
                push({ planRemove(U, u) })
                claimedU[#claimedU + 1] = { u.keyS, spanEnd(u) }
                changed[u.path] = u
                touched[#touched + 1] = u
                report[#report + 1] = "- removed " .. u.path
            elseif not B then
                claimedU[#claimedU + 1] = { u.keyS, spanEnd(u) }
                report[#report + 1] = "? kept " .. u.path .. " (not in the new version; remove it if unused)"
            end
        end
    end

    -- Settings the new version has.
    for _, n in ipairs(N.list) do
        if not containedIn(claimedN, n.keyS, spanEnd(n)) then
            local u = U.map[n.path]
            if not u then
                local list, why = planAdd(N, U, n)
                claimedN[#claimedN + 1] = { n.keyS, spanEnd(n) }
                if list then
                    push(list)
                    touched[#touched + 1] = n
                    report[#report + 1] = "+ added " .. n.path
                else
                    skipped[n.path] = true
                    report[#report + 1] = "! could not add " .. n.path .. ": " .. why
                end
            elseif shapeDiffers(n, u) then
                local b = B and B.map[n.path]
                claimedN[#claimedN + 1] = { n.keyS, spanEnd(n) }
                if b and shapeDiffers(b, n) and not shapeDiffers(b, u)
                    and not containedIn(claimedU, u.keyS, spanEnd(u)) then
                    push({ { u.valS, u.valE, N.src:sub(n.valS, n.valE) } })
                    claimedU[#claimedU + 1] = { u.keyS, spanEnd(u) }
                    changed[n.path] = u
                    touched[#touched + 1] = u
                    report[#report + 1] = "~ replaced " .. n.path .. " (its format changed in this version)"
                else
                    skipped[n.path] = true
                    report[#report + 1] = "! left " .. n.path .. " as it is: its format in your file differs from the new version"
                end
            end
        end
    end

    if #edits == 0 then
        return userText, report
    end

    -- Commas planned twice for the same sibling collapse to one.
    local seen, unique = {}, {}
    for _, e in ipairs(edits) do
        local key = e.comma and ("," .. e[1]) or nil
        if not key or not seen[key] then
            if key then seen[key] = true end
            unique[#unique + 1] = e
        end
    end

    local merged, applyErr = applyEdits(userText, unique)
    if not merged then
        report[#report + 1] = "merge failed: " .. applyErr
        return nil, report
    end

    -- Verify by running all three.
    local envM, errM = evaluate(merged, "merged config")
    if not envM then
        report[#report + 1] = "merge failed: the merged config does not load: " .. tostring(errM)
        return nil, report
    end
    local envU, errU = evaluate(userText, "your config")
    if not envU then
        report[#report + 1] = "merge failed: your current config does not load, so the result cannot be checked: " .. tostring(errU)
        return nil, report
    end
    local envN = evaluate(newText, "new config")

    -- A setting that CONTAINS something added, removed or replaced is expected to
    -- differ as a whole (e.g. `Config = Config or {}`); its untouched children are
    -- still compared one by one.
    for _, u in ipairs(U.list) do
        if u.kind ~= "map" and u.kind ~= "empty" and not underAny(u, changed)
            and not containsAny(u, touched) then
            if not same(valueAt(envU, u), valueAt(envM, u)) then
                report[#report + 1] = "merge failed: your value for " .. u.path .. " would have changed"
                return nil, report
            end
        end
    end
    if envN then
        for _, n in ipairs(N.list) do
            if not underAny(n, skipped) and valueAt(envN, n) ~= nil and valueAt(envM, n) == nil then
                report[#report + 1] = "merge failed: " .. n.path .. " is still missing after the merge"
                return nil, report
            end
        end
    end
    for path, e in pairs(changed) do
        if not N.map[path] and valueAt(envM, e) ~= nil then
            report[#report + 1] = "merge failed: " .. path .. " was meant to be removed but is still set"
            return nil, report
        end
    end

    return merged, report
end

-- Exposed for tests.
M._index = index

return M
