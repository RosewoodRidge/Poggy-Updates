--[[
    poggy_core — settings model.

    The hub (/poggy) shows each script's config file as settings an owner can
    change in game. The config file stays the source of truth: nothing is kept
    anywhere else, and the hub edits the file itself. This file is the part that
    understands a config file. Plain Lua, no natives, so it is tested outside the
    server (tools/test-settings-model.py).

    Reading. Parse turns a config into settings, in file order, each with a path
    (Config.Auction.MinBid, Config.Stores[3], Translations["en"].greeting), a kind,
    its value, its line, and the comments written beside it:

        value     a boolean, number, text, vector (vector3 / vec3 ...) or `hash`
        group     a table of named settings; its children are settings too
        list      an array of tables or other values (stores, recipes): rows
        map       a table keyed by data (["valentine"] = {...}, [0] = 70): rows
                  with a key column; the values may be tables or plain values
        strings   an array of text or numbers (job lists): chips
        readonly  code rather than data: a function call, a reference to another
                  value, a concatenation, a function, a table that mixes a list
                  with named keys. `reason` says which, `source` shows the text.

    Positional or [bracketed] keys make a table data (list/map); identifier keys
    make it settings (group), as in sv_configmerge.lua. With { dictKeys = true }
    (translation files) a ["text"] key is a setting of its own, so a language
    table is a group of strings. { kinds = { [path] = "map" | "list" | ... } }
    forces a kind (hub.json's "kind").

    Writing. Every edit changes the smallest span it can and leaves every other
    byte alone: comments, spacing, alignment, order, tabs, CRLF. New text follows
    the value it replaces or the row beside it: quote style, decimals, hex,
    vec3 or vector3, one line or several, the siblings' indentation and key
    order. Only data is ever written. A value that would need code (a function,
    anything that is not a boolean, number, text, vector, hash or table of those)
    is refused.

    Every edit checks itself before returning. The new text must parse and run in
    the sandbox; every setting other than the edited one must hold exactly what
    it held before; the edited one must hold exactly what was asked for, both as
    written and as Lua sees it once the whole file has run (so a line further
    down that overwrites it is caught). If any of that fails, the edit returns
    nil and a reason, and the caller writes nothing.

        local M = PoggyCore.SettingsModel
        local model, err = M.Parse(text, { dictKeys = false })
        local newText, err = M.Set(text, "Config.Auction.MinBid", 10)

    Every edit takes the same opts as Parse as its last argument; pass them, so
    the edit sees the same kinds (read-only, map...) the hub showed.

    Reuses sv_configmerge.lua's lexer (long comments, backtick hashes...), its
    expression skipper and its sandbox runner, so this file loads after it.
]]

PoggyCore = PoggyCore or {}

local CM = PoggyCore.ConfigMerge
assert(CM and CM._lex, "sv_settings_model.lua must load after sv_configmerge.lua")

local lex, parseExpr, skipBlock = CM._lex, CM._parseExpr, CM._skipBlock
local KEYWORDS, BINOP = CM._KEYWORDS, CM._BINOP
local evaluate, same = CM._evaluate, CM._same

local M = {}
PoggyCore.SettingsModel = M

-- ---------------------------------------------------------------------------
-- Errors: a refusal is a table { msg }, turned into nil, msg at the API edge
-- ---------------------------------------------------------------------------

local function fail(msg)
    error({ msg = msg }, 0)
end

local function cleanErr(e)
    return (tostring(e):gsub("^[^\n]-:%d+: ", ""))
end

local function guard(fn, ...)
    local res = table.pack(pcall(fn, ...))
    if res[1] then return table.unpack(res, 2, res.n) end
    local e = res[2]
    if type(e) == "table" and e.msg then return nil, e.msg end
    return nil, "internal error: " .. cleanErr(e)
end

-- ---------------------------------------------------------------------------
-- Lines
-- ---------------------------------------------------------------------------

local function computeLines(src)
    local starts, p = { 1 }, 1
    while true do
        local nl = src:find("\n", p, true)
        if not nl then break end
        starts[#starts + 1] = nl + 1
        p = nl + 1
    end
    return starts
end

--- 1-based line number of a byte position.
local function lineOf(D, pos)
    local L = D.lines
    local lo, hi = 1, #L
    while lo < hi do
        local mid = (lo + hi + 1) // 2
        if L[mid] <= pos then lo = mid else hi = mid - 1 end
    end
    return lo
end

local function lineStart(D, pos) return D.lines[lineOf(D, pos)] end

--- The position of the newline that ends the line (or the last byte of the file).
local function lineEnd(D, pos)
    local nxt = D.lines[lineOf(D, pos) + 1]
    return nxt and (nxt - 1) or #D.src
end

--- The last byte of a line's content, before any "\r\n".
local function contentEnd(D, li)
    local nxt = D.lines[li + 1]
    local e = nxt and (nxt - 2) or #D.src
    if e >= D.lines[li] and D.src:sub(e, e) == "\r" then e = e - 1 end
    return e
end

local function lineText(D, li)
    return D.src:sub(D.lines[li], contentEnd(D, li))
end

local function lineIndent(D, pos)
    return D.src:match("^[ \t]*", lineStart(D, pos))
end

--- The newline a line ends with, so an inserted line matches its neighbours.
local function lineNl(D, pos)
    local nxt = D.lines[lineOf(D, pos) + 1]
    if nxt then
        return D.src:sub(nxt - 2, nxt - 2) == "\r" and "\r\n" or "\n"
    end
    return D.nl
end

local function cleanBefore(D, pos)
    return D.src:sub(lineStart(D, pos), pos - 1):match("^[ \t\r\n]*$") ~= nil
end

--- Nothing but spaces or a comment after pos on its line.
local function cleanAfter(D, pos)
    local rest = D.src:sub(pos + 1, contentEnd(D, lineOf(D, pos)))
    return rest:match("^[ \t\r]*$") ~= nil or rest:match("^[ \t]*%-%-") ~= nil
end

local function isCommentLine(line)
    return line:match("^[ \t]*%-%-") ~= nil and line:match("^[ \t]*%-%-%[=*%[") == nil
end

--- `-- =====` and the like: a banner rule, not words.
local function isRuleLine(line)
    local body = line:match("^[ \t]*%-%-(.-)[ \t\r]*$")
    return body ~= nil and body:match("^[ \t=%-%*#~_]*$") ~= nil and body:match("[=%-%*#~_][=%-%*#~_][=%-%*#~_]") ~= nil
end

local function commentWords(line)
    return (line:match("^[ \t]*%-%-%-?[ \t]?(.-)[ \t\r]*$") or "")
end

--- The last token that touches a line.
local function lastTokenOnLine(D, li)
    local ce = contentEnd(D, li)
    local toks = D.toks
    local lo, hi = 1, #toks
    if hi == 0 or toks[1].s > ce then return nil end
    while lo < hi do
        local mid = (lo + hi + 1) // 2
        if toks[mid].s <= ce then lo = mid else hi = mid - 1 end
    end
    local t = toks[lo]
    if t.e < D.lines[li] then return nil end
    return t
end

--- The comment at the end of a line: its words, where "--" starts, and
--- whether it is a long comment. nil when the line has none.
local function trailingComment(D, li)
    local t = lastTokenOnLine(D, li)
    local ce = contentEnd(D, li)
    if not t or t.e > ce then return nil end
    local rest = D.src:sub(t.e + 1, ce)
    local off = rest:match("^[ \t]*()%-%-")
    if not off then return nil end
    local body = rest:sub(off)
    local long = body:match("^%-%-%[=*%[") ~= nil
    local words
    if long then
        words = body:gsub("^%-%-%[=*%[", ""):gsub("%]=*%]%-*[ \t]*$", "")
    else
        words = body:gsub("^%-+[ \t]?", "")
    end
    return (words:gsub("[ \t\r]+$", "")), t.e + off, long
end

local ROLE_PAT = "poggy:role[ \t]+([A-Za-z0-9_%-]+)"

local function stripRole(words)
    local role = words:match(ROLE_PAT)
    if role then
        words = words:gsub("[ \t]*%-*[ \t]*poggy:role[ \t]+[A-Za-z0-9_%-]+", ""):gsub("^[ \t]+", ""):gsub("[ \t]+$", "")
    end
    return words, role
end

--- Comment lines directly above a setting (banner rules skipped over, a blank
--- line ends them), plus the comment at the end of its first and last line.
local function commentsFor(D, pos, endPos)
    local out, role = {}, nil
    if cleanBefore(D, pos) then
        local li = lineOf(D, pos) - 1
        local above, taken = {}, 0
        while li >= 1 and taken < 80 do
            local text = lineText(D, li)
            if not isCommentLine(text) then break end
            if not isRuleLine(text) then
                local w, r = stripRole(commentWords(text))
                table.insert(above, 1, w)
            end
            li = li - 1
            taken = taken + 1
        end
        while above[1] == "" do table.remove(above, 1) end
        while above[#above] == "" do table.remove(above) end
        for _, l in ipairs(above) do out[#out + 1] = l end
    end
    local first, last = lineOf(D, pos), lineOf(D, endPos)
    for _, li in ipairs(first == last and { first } or { first, last }) do
        local words = trailingComment(D, li)
        if words then
            local w, r = stripRole(words)
            role = role or r
            if w ~= "" then out[#out + 1] = w end
        end
    end
    local text = table.concat(out, "\n")
    return text ~= "" and text or nil, role
end

--- Indentation unit of the file: a tab, or the smallest step of spaces seen.
local function detectUnit(D)
    local best
    for i = 1, math.min(#D.lines, 400) do
        local ws = D.src:match("^([ \t]+)%S", D.lines[i])
        if ws then
            if ws:sub(1, 1) == "\t" then return "\t" end
            if #ws >= 2 and (not best or #ws < best) then best = #ws end
        end
    end
    return (" "):rep(best and math.min(best, 8) or 4)
end

-- ---------------------------------------------------------------------------
-- Values as written: strings, numbers
-- ---------------------------------------------------------------------------

local SIMPLE_ESC = {
    a = "\a", b = "\b", f = "\f", n = "\n", r = "\r", t = "\t", v = "\v",
    ["\\"] = "\\", ['"'] = '"', ["'"] = "'", ["\n"] = "\n",
}

--- A string literal's value exactly as Lua reads it, and its quote. nil when
--- the literal is malformed.
local function decodeString(lit)
    local q = lit:sub(1, 1)
    if q == "[" then
        local lvl = lit:match("^%[(=*)%[")
        if not lvl then return nil end
        local close = "]" .. lvl .. "]"
        if #lit < #lvl * 2 + 4 or lit:sub(-#close) ~= close then return nil end
        local body = lit:sub(#lvl + 3, -(#lvl + 3))
        local nlAfter = false
        if body:sub(1, 2) == "\r\n" or body:sub(1, 2) == "\n\r" then
            body, nlAfter = body:sub(3), true
        elseif body:sub(1, 1) == "\n" or body:sub(1, 1) == "\r" then
            body, nlAfter = body:sub(2), true
        end
        body = body:gsub("\r\n", "\n"):gsub("\n\r", "\n"):gsub("\r", "\n")
        return body, "[" .. lvl .. "[", nlAfter
    end
    if (q ~= '"' and q ~= "'") or #lit < 2 or lit:sub(-1) ~= q then return nil end
    local body = lit:sub(2, -2)
    if not body:find("\\", 1, true) then return body, q end
    local out, i, n = {}, 1, #body
    while i <= n do
        local c = body:sub(i, i)
        if c ~= "\\" then
            local j = body:find("\\", i, true) or (n + 1)
            out[#out + 1] = body:sub(i, j - 1)
            i = j
        else
            local d = body:sub(i + 1, i + 1)
            if SIMPLE_ESC[d] then
                out[#out + 1] = SIMPLE_ESC[d]
                i = i + 2
            elseif d == "\r" then
                out[#out + 1] = "\n"
                i = i + 2
                if body:sub(i, i) == "\n" then i = i + 1 end
            elseif d == "x" then
                local h = body:match("^%x%x", i + 2)
                if not h then return nil end
                out[#out + 1] = string.char(tonumber(h, 16))
                i = i + 4
            elseif d == "z" then
                local _, e = body:find("^[ \t\r\n\v\f]*", i + 2)
                i = e + 1
            elseif d:match("%d") then
                local ds = body:match("^%d%d?%d?", i + 1)
                local v = tonumber(ds)
                if v > 255 then return nil end
                out[#out + 1] = string.char(v)
                i = i + 1 + #ds
            elseif d == "u" then
                local hx = body:match("^{(%x+)}", i + 2)
                if not hx then return nil end
                local cp = tonumber(hx, 16)
                if cp > 0x7FFFFFFF then return nil end
                out[#out + 1] = utf8.char(cp)
                i = i + 4 + #hx
            else
                return nil
            end
        end
    end
    return table.concat(out), q
end

local function quoteString(s, q)
    q = q or '"'
    -- Explicit byte ranges: %c follows the C locale, and on Windows it matches
    -- some UTF-8 continuation bytes, which would break multi-byte characters.
    local out = s:gsub('[\0-\31\127"\'\\]', function(c)
        if c == "\\" then return "\\\\" end
        if c == q then return "\\" .. q end
        if c == '"' or c == "'" then return c end
        if c == "\n" then return "\\n" end
        if c == "\r" then return "\\r" end
        if c == "\t" then return "\\t" end
        return ("\\%03d"):format(c:byte())
    end)
    return q .. out .. q
end

local function shortestFloat(v)
    local s
    for p = 1, 17 do
        s = ("%." .. p .. "g"):format(v)
        if tonumber(s) == v then break end
    end
    if not s:find("[%.eEn]") then s = s .. ".0" end
    return s
end

local function isIntegral(v)
    return math.type(v) == "integer" or (v == math.floor(v) and math.abs(v) < 2 ^ 53)
end

--- A number in the style of the literal it replaces (decimals, hex, float or
--- integer). floatish: write whole numbers with ".0" when there is no hint.
local function fmtNumber(v, hintSrc, floatish)
    if v ~= v or v == math.huge or v == -math.huge then fail("a number must be finite") end
    local whole = isIntegral(v)
    local int = whole and math.tointeger(v) or nil
    if hintSrc then
        local hs = hintSrc:gsub("^%-[ \t]*", "")
        local prefix, digits = hs:match("^(0[xX])(%x+)$")
        if prefix and int and int >= 0 then
            local upper = digits:match("%u") ~= nil or not digits:match("%l")
            return prefix .. ("%0" .. #digits .. (upper and "X" or "x")):format(int)
        end
        local dec = hs:match("^%d*%.(%d*)$")
        if dec then
            local s = ("%." .. math.max(#dec, 1) .. "f"):format(v)
            if tonumber(s) == v then return s end
            return shortestFloat(v)
        end
        if hs:match("^%d+$") then
            if int then return ("%d"):format(int) end
            return shortestFloat(v)
        end
    end
    if int then
        return floatish and ("%.1f"):format(v) or ("%d"):format(int)
    end
    return shortestFloat(v)
end

-- ---------------------------------------------------------------------------
-- Parser: the file as statements and value trees, with exact spans
-- ---------------------------------------------------------------------------

local VECTORS = { vector2 = 2, vector3 = 3, vector4 = 4, vec2 = 2, vec3 = 3, vec4 = 4 }
local COMP = { "x", "y", "z", "w" }
local ARITH = {}
for _, o in ipairs({ "+", "-", "*", "/", "//", "%", "^", "(", ")" }) do ARITH[o] = true end
local SUFFIX = { ["."] = true, [":"] = true, ["["] = true, ["("] = true }

local function tk(S, k) return S.toks[S.p + (k or 0)] end
local function tv(S, k)
    local t = tk(S, k)
    return t and S.src:sub(t.s, t.e)
end
local function ttext(S, i)
    local t = S.toks[i]
    return t and S.src:sub(t.s, t.e)
end

local function shorten(s, n)
    s = s:gsub("[ \t\r\n]+", " ")
    if #s > n then s = s:sub(1, n - 1) .. "…" end
    return s
end

--- A code span: why it cannot be edited, or a constant calculation's value.
local function codeNode(S, s, e, p1, p2)
    local src = S.src:sub(s, e)
    local node = { t = "code", s = s, e = e, source = src }
    local first = ttext(S, p1)

    local calc = true
    for i = p1, p2 do
        local t = S.toks[i]
        if not (t.t == "number" or (t.t == "op" and ARITH[ttext(S, i)])) then calc = false break end
    end
    if calc then
        local fn = load("return " .. src, "=calc", "t", {})
        local ok, v = false, nil
        if fn then ok, v = pcall(fn) end
        if ok and type(v) == "number" and v == v and v ~= math.huge and v ~= -math.huge then
            return { t = "number", value = v, s = s, e = e, src = src, calc = true, source = src }
        end
    end

    local concat, logic, call, chain = false, false, false, true
    for i = p1, p2 do
        local t, x = S.toks[i], ttext(S, i)
        if t.t == "op" and x == ".." then concat = true end
        if t.t == "name" and (x == "and" or x == "or" or x == "not") then logic = true end
        if t.t == "op" and x == "(" and i > p1 then call = true end
        if not (t.t == "name" and not KEYWORDS[x]) and not (t.t == "op" and x == ".") then chain = false end
    end
    local short = shorten(src, 80)
    if first == "function" then
        node.reason = "a function"
    elseif concat then
        node.reason = "text joined together with .. (" .. short .. ")"
    elseif chain then
        node.reason = "a reference to another value (" .. short .. ")"
    elseif logic then
        node.reason = "worked out by a condition (" .. short .. ")"
    elseif call then
        node.reason = "worked out by a function call (" .. short .. ")"
    else
        node.reason = "worked out by code (" .. short .. ")"
    end
    return node
end

local parseValue

local function tryVector(S)
    local nameTok = tk(S)
    local ctor = tv(S)
    local arity = VECTORS[ctor]
    local open = S.toks[S.p + 1]
    local q = S.p + 2
    local comps, compSrc, firstS, lastE, seps = {}, {}, nil, nil, {}
    while true do
        local t = S.toks[q]
        if not t then return nil end
        local x = S.src:sub(t.s, t.e)
        if #comps == 0 and t.t == "op" and x == ")" then return nil end
        local cs = t.s
        local neg = false
        if t.t == "op" and x == "-" then
            neg = true
            q = q + 1
            t = S.toks[q]
            if not t then return nil end
            x = S.src:sub(t.s, t.e)
        end
        if t.t ~= "number" then return nil end
        local num = tonumber(x)
        if not num then return nil end
        comps[#comps + 1] = neg and -num or num
        compSrc[#compSrc + 1] = S.src:sub(cs, t.e)
        firstS = firstS or cs
        lastE = t.e
        q = q + 1
        local sep = S.toks[q]
        if not sep or sep.t ~= "op" then return nil end
        local st = S.src:sub(sep.s, sep.e)
        if st == ")" then break end
        if st ~= "," then return nil end
        seps[#seps + 1] = { sep.s, sep.e }
        q = q + 1
    end
    if #comps ~= arity then return nil end
    local close = S.toks[q]
    local node = {
        t = "vector", ctor = ctor, arity = arity, comps = comps, compSrc = compSrc,
        s = nameTok.s, e = close.e,
        pre = S.src:sub(open.e + 1, firstS - 1),
        post = S.src:sub(lastE + 1, close.s - 1),
    }
    if seps[1] then
        local nextComp = S.src:find("[%-%d%.]", seps[1][2] + 1)
        node.sep = S.src:sub(seps[1][1], (nextComp or seps[1][2] + 1) - 1)
    end
    S.p = q + 1
    return node
end

--- Shape of a parsed table: "empty", "pos" (a list), "keyed", or "bad" + why.
local function tableShape(T)
    local fields = T.fields
    if #fields == 0 then return "empty" end
    local pos, keyed, ints, strs = 0, 0, 0, 0
    local seen = {}
    for _, f in ipairs(fields) do
        if f.kind == "computed" then
            return "bad", "has a key worked out by code (" .. shorten(f.keySrc, 60) .. ")"
        elseif f.kind == "pos" then
            pos = pos + 1
        else
            keyed = keyed + 1
            local k = f.key
            if type(k) == "number" then
                if math.type(k) ~= "integer" then return "bad", "has a number key with decimals" end
                ints = ints + 1
            else
                strs = strs + 1
                if k == "__type" or k == "__int_keys" then return "bad", "uses the reserved key " .. k end
            end
            if seen[k] then return "bad", "has the key " .. tostring(k) .. " twice" end
            seen[k] = true
        end
    end
    if pos > 0 and keyed > 0 then return "bad", "mixes a list with named keys" end
    if pos > 0 then return "pos" end
    if ints > 0 and strs > 0 then return "bad", "mixes number keys and text keys" end
    return "keyed"
end

local function parseTable(S)
    local open = tk(S)
    S.p = S.p + 1
    local T = { t = "table", s = open.s, fields = {} }
    T.pad = S.src:sub(open.e + 1, open.e + 1) == " " and " " or ""
    local pos = 0
    while true do
        local t = tk(S)
        if not t then error("a table is never closed") end
        local v = tv(S)
        if t.t == "op" and v == "}" then
            T.e, T.closeS = t.e, t.s
            S.p = S.p + 1
            break
        end
        local f = { s = t.s }
        local nt = tk(S, 1)
        if t.t == "name" and not KEYWORDS[v] and nt and nt.t == "op" and tv(S, 1) == "=" then
            f.kind, f.key, f.keyS, f.keyE, f.eqS = "name", v, t.s, t.e, nt.s
            S.p = S.p + 2
        elseif t.t == "op" and v == "[" then
            local keyVal, lastP, quote
            if nt and nt.t == "string" and tv(S, 2) == "]" and tv(S, 3) == "=" then
                local lit = tv(S, 1)
                if lit:sub(1, 1) ~= "`" then
                    keyVal, quote = decodeString(lit)
                end
                lastP = S.p + 2
            elseif nt and nt.t == "number" and tv(S, 2) == "]" and tv(S, 3) == "=" then
                keyVal = tonumber(tv(S, 1))
                lastP = S.p + 2
            elseif nt and nt.t == "op" and tv(S, 1) == "-" and tk(S, 2) and tk(S, 2).t == "number"
                and tv(S, 3) == "]" and tv(S, 4) == "=" then
                keyVal = tonumber(tv(S, 2))
                keyVal = keyVal and -keyVal
                lastP = S.p + 3
            end
            if keyVal ~= nil then
                if math.type(keyVal) == "float" and keyVal == math.floor(keyVal) and math.abs(keyVal) < 2 ^ 53 then
                    keyVal = math.tointeger(keyVal)
                end
                f.kind, f.key, f.keyS, f.keyE = "bracket", keyVal, t.s, S.toks[lastP].e
                f.keyQuote = quote
                f.eqS = S.toks[lastP + 1].s
                S.p = lastP + 2
            else
                S.p = S.p + 1
                parseExpr(S)
                if tv(S) ~= "]" then error("a [key] is never closed") end
                f.keyE = tk(S).e
                S.p = S.p + 1
                if tv(S) ~= "=" then error("a [key] has no =") end
                f.eqS = tk(S).s
                S.p = S.p + 1
                f.kind, f.keyS = "computed", t.s
                f.keySrc = S.src:sub(t.s, f.keyE)
            end
        else
            pos = pos + 1
            f.kind, f.key = "pos", pos
        end
        f.v = parseValue(S)
        f.e = f.v.e
        local sep = tk(S)
        if sep and sep.t == "op" and (tv(S) == "," or tv(S) == ";") then
            f.sepS, f.sepE = sep.s, sep.e
            S.p = S.p + 1
        end
        T.fields[#T.fields + 1] = f
    end
    T.shape, T.why = tableShape(T)
    return T
end

--- One value: a literal, a vector, a table, or a span of code.
function parseValue(S)
    local t = tk(S)
    if not t then error("unexpected end of file") end
    local startP = S.p
    local v = tv(S)
    local node

    if t.t == "op" and v == "-" and tk(S, 1) and tk(S, 1).t == "number" then
        local nt = tk(S, 1)
        local num = tonumber(S.src:sub(nt.s, nt.e))
        if num then
            node = { t = "number", value = -num, s = t.s, e = nt.e, src = S.src:sub(t.s, nt.e) }
            S.p = S.p + 2
        end
    elseif t.t == "number" then
        local num = tonumber(v)
        if num then
            node = { t = "number", value = num, s = t.s, e = t.e, src = v }
            S.p = S.p + 1
        end
    elseif t.t == "string" then
        if v:sub(1, 1) == "`" then
            if #v >= 2 and v:sub(-1) == "`" then
                node = { t = "hash", name = v:sub(2, -2), s = t.s, e = t.e }
            end
        else
            local val, q, nlAfter = decodeString(v)
            if val then node = { t = "string", value = val, quote = q, nlAfter = nlAfter, s = t.s, e = t.e } end
        end
        if node then S.p = S.p + 1 end
    elseif t.t == "name" and (v == "true" or v == "false") then
        node = { t = "boolean", value = (v == "true"), s = t.s, e = t.e }
        S.p = S.p + 1
    elseif t.t == "name" and v == "nil" then
        node = { t = "nil", s = t.s, e = t.e, source = "nil" }
        S.p = S.p + 1
    elseif t.t == "op" and v == "{" then
        node = parseTable(S)
    elseif t.t == "name" and VECTORS[v] and tv(S, 1) == "(" then
        node = tryVector(S)
        if not node then S.p = startP end
    end

    if node then
        local nt = tk(S)
        if nt and nt.t ~= "string" then
            local x = tv(S)
            if (nt.t == "op" and (BINOP[x] or SUFFIX[x])) or (nt.t == "name" and (x == "and" or x == "or")) then
                node = nil
            end
        elseif nt and nt.t == "string" and node.t == "table" then
            node = nil
        end
    end
    if not node then
        S.p = startP
        local e = parseExpr(S)
        node = codeNode(S, t.s, e, startP, S.p - 1)
    end
    return node
end

--- Every top-level statement: plain assignments become settings; other
--- statements and blocks that run at load are kept for a scan of what they write.
local function parseStatements(S)
    local stmts, scans = {}, {}
    while tk(S) do
        local t, v = tk(S), tv(S)
        local startP = S.p
        if t.t == "name" and not KEYWORDS[v] then
            -- The left-hand side: Config.Catalog.GeneralStore, or with a
            -- bracketed text key, Config.Catalog["General Store"] (a row
            -- added from the hub under a name that is not an identifier).
            local parts, q = { v }, S.p + 1
            while S.toks[q] and S.toks[q + 1] and S.toks[q].t == "op" do
                local x = ttext(S, q)
                if x == "." and S.toks[q + 1].t == "name" then
                    parts[#parts + 1] = ttext(S, q + 1)
                    q = q + 2
                elseif x == "[" and S.toks[q + 1].t == "string" and S.toks[q + 2]
                    and S.toks[q + 2].t == "op" and ttext(S, q + 2) == "]" then
                    local lit = ttext(S, q + 1)
                    local key = lit:sub(1, 1) ~= "`" and decodeString(lit) or nil
                    if type(key) ~= "string" then break end
                    parts[#parts + 1] = key
                    q = q + 3
                else
                    break
                end
            end
            local eq = S.toks[q]
            if eq and eq.t == "op" and ttext(S, q) == "=" then
                S.p = q + 1
                local st = { keys = parts, s = t.s, T = parseValue(S) }
                if tk(S) and tv(S) == ";" then
                    st.sepE = tk(S).e
                    S.p = S.p + 1
                end
                stmts[#stmts + 1] = st
            else
                parseExpr(S)
                while tv(S) == "," do
                    S.p = S.p + 1
                    parseExpr(S)
                end
                if tk(S) and tk(S).t == "op" and tv(S) == "=" then
                    S.p = S.p + 1
                    parseExpr(S)
                    while tv(S) == "," do
                        S.p = S.p + 1
                        parseExpr(S)
                    end
                end
                if S.p == startP then S.p = S.p + 1 end
                scans[#scans + 1] = { startP, S.p - 1 }
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
                scans[#scans + 1] = { startP, S.p - 1 }
            end
        elseif v == "function" then
            skipBlock(S)
        elseif v == "if" or v == "do" or v == "repeat" then
            skipBlock(S)
            scans[#scans + 1] = { startP, S.p - 1 }
        elseif v == "for" or v == "while" then
            while tk(S) and tv(S) ~= "do" do S.p = S.p + 1 end
            skipBlock(S)
            scans[#scans + 1] = { startP, S.p - 1 }
        elseif v == "return" then
            S.p = S.p + 1
            if tk(S) and not KEYWORDS[tv(S)] then
                parseExpr(S)
                while tv(S) == "," do
                    S.p = S.p + 1
                    parseExpr(S)
                end
            end
        else
            S.p = S.p + 1
        end
    end
    return stmts, scans
end

-- ---------------------------------------------------------------------------
-- Paths and keys
-- ---------------------------------------------------------------------------

local function isIdent(k)
    return type(k) == "string" and k:match("^[A-Za-z_][A-Za-z0-9_]*$") ~= nil and not KEYWORDS[k]
end

--- A key path segment for a bracketed string key, identical however it was quoted.
local function bracketSegment(key)
    return '["' .. key:gsub("\\", "\\\\"):gsub('"', '\\"') .. '"]'
end

local function segment(key, asName)
    if math.type(key) == "integer" then return "[" .. key .. "]" end
    if asName then return "." .. key end
    return bracketSegment(key)
end

--- "Config.Lang[\"en\"].greeting" -> { "Config", "Lang", "en", "greeting" }
local function parsePath(path)
    if type(path) ~= "string" then fail("a path must be text") end
    local root = path:match("^[A-Za-z_][A-Za-z0-9_]*")
    if not root then fail("a path starts with a name, e.g. Config.Debug (got " .. path .. ")") end
    local keys, i, n = { root }, #root + 1, #path
    while i <= n do
        local c = path:sub(i, i)
        if c == "." then
            local name = path:match("^[A-Za-z_][A-Za-z0-9_]*", i + 1)
            if not name then fail("bad path near " .. path:sub(i)) end
            keys[#keys + 1] = name
            i = i + 1 + #name
        elseif c == "[" then
            local q = path:sub(i + 1, i + 1)
            if q == '"' or q == "'" then
                local j, buf = i + 2, {}
                while j <= n do
                    local d = path:sub(j, j)
                    if d == "\\" then
                        local e = path:sub(j + 1, j + 1)
                        buf[#buf + 1] = (e == "n" and "\n") or (e == "t" and "\t") or e
                        j = j + 2
                    elseif d == q then
                        break
                    else
                        buf[#buf + 1] = d
                        j = j + 1
                    end
                end
                if path:sub(j, j) ~= q or path:sub(j + 1, j + 1) ~= "]" then fail("bad path near " .. path:sub(i)) end
                keys[#keys + 1] = table.concat(buf)
                i = j + 2
            else
                local num = path:match("^%-?%d+", i + 1)
                if not num or path:sub(i + 1 + #num, i + 1 + #num) ~= "]" then fail("bad path near " .. path:sub(i)) end
                keys[#keys + 1] = math.tointeger(tonumber(num))
                i = i + 2 + #num
            end
        else
            fail("bad path near " .. path:sub(i))
        end
    end
    return keys
end

--- The path of the first n keys of a statement, as the model spells paths:
--- Config.Catalog.GeneralStore, Config.Catalog["General Store"].
local function keysPath(keys, n)
    local out = { tostring(keys[1]) }
    for i = 2, n or #keys do
        local k = keys[i]
        out[i] = segment(k, isIdent(k))
    end
    return table.concat(out)
end

local function canon(keys, n)
    local out = {}
    for i = 1, n or #keys do
        local k = keys[i]
        out[i] = (math.type(k) == "integer" and "i" or "s") .. tostring(k)
    end
    return table.concat(out, "\1")
end

local function extend(list, item)
    local out = {}
    for i, v in ipairs(list) do out[i] = v end
    out[#out + 1] = item
    return out
end

local function fieldEnd(f) return f.sepE or f.v.e end

-- ---------------------------------------------------------------------------
-- Encoding: parsed values -> JSON-friendly values (spec §3.1)
-- ---------------------------------------------------------------------------

local function scalarType(T)
    if T.t == "vector" then return "vector" .. T.arity end
    return T.t
end

local function relSeg(f)
    if f.kind == "name" then return "." .. f.key end
    if type(f.key) == "number" then return "[" .. f.key .. "]" end
    return bracketSegment(f.key)
end

--- A parsed value as JSON-side data (§3.1), or nil, why.
---
--- cells: when given (a list), code inside the table (a cell such as
--- `name = T('X')` or `jobs = Config.PoliceJobList`) does not make the whole
--- value unreadable. It is encoded as { __type = "code", source, reason } and
--- listed in cells as { rel, keys, T }, so the rest of the table stays
--- editable and only those cells are read-only. The value itself (rel "")
--- being code is still an error. relKeys: the keys down to T, with cells.
local function encodeT(D, T, rel, cells, relKeys)
    local t = T.t
    if t == "string" or t == "number" or t == "boolean" then return T.value end
    if t == "vector" then
        local v = { __type = "vec" .. T.arity }
        for i = 1, T.arity do v[COMP[i]] = T.comps[i] end
        return v
    end
    if t == "hash" then return { __type = "hash", name = T.name } end
    local where = (rel == nil or rel == "") and "it" or ("entry " .. rel)
    where = where .. " (line " .. lineOf(D, T.s) .. ")"
    if t == "nil" then return nil, where .. " has no value (nil)" end
    if t == "code" then
        if cells and rel ~= nil and rel ~= "" then
            cells[#cells + 1] = { rel = rel, keys = relKeys or {}, T = T }
            return { __type = "code", source = shorten(T.source, 400), reason = T.reason }
        end
        return nil, where .. " is " .. T.reason
    end
    local out = {}
    if T.shape == "empty" then return out end
    if T.shape == "bad" then return nil, where .. " " .. T.why end
    local intKeys = false
    for i, f in ipairs(T.fields) do
        -- `key = nil` in a keyed table: Lua has no such key, so neither does the value.
        if not (T.shape == "keyed" and f.v.t == "nil") then
            local v, err = encodeT(D, f.v, (rel or "") .. relSeg(f), cells, cells and extend(relKeys or {}, f.key))
            if v == nil then return nil, err end
            if T.shape == "pos" then
                out[i] = v
            elseif math.type(f.key) == "integer" then
                intKeys = true
                out[tostring(f.key)] = v
            else
                out[f.key] = v
            end
        end
    end
    if intKeys then out.__int_keys = true end
    return out
end

--- What kind of thing a JSON-side value is.
local function valueKind(v)
    local t = type(v)
    if t == "string" or t == "boolean" then return t end
    if t == "number" then
        if v ~= v or v == math.huge or v == -math.huge then return nil, "a number that is not finite" end
        return "number"
    end
    if t ~= "table" then return nil, "a " .. t end
    local tag = rawget(v, "__type")
    if tag ~= nil then
        if tag == "hash" then return "hash" end
        if tag == "vec2" or tag == "vec3" or tag == "vec4" then return "vector" end
        return nil, "a value tagged " .. tostring(tag)
    end
    local n, count = #v, 0
    for _ in pairs(v) do count = count + 1 end
    if count == 0 or n == count then return "array" end
    return "object"
end

--- Equality of JSON-side values: numbers by value (5 == 5.0), int keys as text.
local function eqEnc(a, b, depth)
    depth = depth or 0
    if depth > 60 then return false end
    local ta, tb = type(a), type(b)
    if ta ~= tb then return false end
    if ta ~= "table" then return a == b end
    local ca, cb = 0, 0
    for k, v in pairs(a) do
        if k ~= "__int_keys" then
            ca = ca + 1
            local w = rawget(b, k)
            if w == nil then
                if type(k) == "string" then
                    local n = math.tointeger(tonumber(k))
                    if n then w = rawget(b, n) end
                elseif math.type(k) == "integer" then
                    w = rawget(b, tostring(k))
                end
            end
            if w == nil or not eqEnc(v, w, depth + 1) then return false end
        end
    end
    for k in pairs(b) do
        if k ~= "__int_keys" then cb = cb + 1 end
    end
    return ca == cb
end

local function deepCopy(v)
    if type(v) ~= "table" then return v end
    local out = {}
    for k, x in pairs(v) do out[k] = deepCopy(x) end
    return out
end

-- ---------------------------------------------------------------------------
-- The model
-- ---------------------------------------------------------------------------

local function classify(T, dict, forced, isRoot)
    local shape = T.shape
    if forced == "readonly" then return "readonly", "marked read-only" end
    if shape == "bad" then return "readonly", T.why end
    if shape == "empty" then
        if forced == "list" or forced == "strings" or forced == "map" or forced == "group" then return forced end
        return isRoot and "group" or "strings"
    end
    if shape == "pos" then
        local scalars = true
        for _, f in ipairs(T.fields) do
            local t = f.v.t
            if t ~= "string" and t ~= "number" then scalars = false break end
        end
        if forced == "list" then return "list" end
        if forced == "strings" and scalars then return "strings" end
        return scalars and "strings" or "list"
    end
    -- keyed
    if forced == "map" or forced == "group" then return forced end
    local bracket, int = false, false
    for _, f in ipairs(T.fields) do
        if f.kind == "bracket" then bracket = true end
        if math.type(f.key) == "integer" then int = true end
    end
    if dict then return int and "map" or "group" end
    return bracket and "map" or "group"
end

local function buildModel(D, opts)
    local dict = opts.dictKeys and true or false
    local forcedKinds = {}
    if type(opts.kinds) == "table" then
        for p, k in pairs(opts.kinds) do
            local ok, keys = pcall(parsePath, p)
            if ok then forcedKinds[canon(keys)] = k end
        end
    end

    local nodes, order, roots, rootSeen = {}, {}, {}, {}
    local byKeys, info = {}, {}
    D.model = { roots = roots, order = order, nodes = nodes }
    D.byKeys, D.info = byKeys, info

    local function register(node, keys, T, pos, endPos)
        nodes[node.path] = node
        order[#order + 1] = node.path
        byKeys[canon(keys)] = node
        info[node.path] = { T = T, keys = keys, pos = pos, endPos = endPos }
    end

    local function makeReadonly(node, reason)
        node.kind = "readonly"
        node.editable = false
        node.reason = reason
        node.value = nil
    end

    local function describe(path, parent, key, keys, T, pos, endPos, isRoot)
        local node = { path = path, parent = parent, key = key, line = lineOf(D, pos), editable = true }
        local cells
        node.comment, node.role = commentsFor(D, pos, endPos)
        local forced = forcedKinds[canon(keys)]
        if T.t == "code" then
            node.source = T.source
            if isRoot then
                node.kind, node.type = "group", "table"
            else
                makeReadonly(node, T.reason)
            end
        elseif T.t == "nil" then
            node.source = "nil"
            makeReadonly(node, "has no value (nil)")
        elseif T.t ~= "table" then
            node.kind, node.type = "value", scalarType(T)
            node.value = encodeT(D, T)
            if T.calc then node.source = T.source end
            if forced == "readonly" then makeReadonly(node, "marked read-only") end
        else
            node.type = "table"
            local kind, why = classify(T, dict, forced, isRoot)
            if kind == "group" then
                node.kind = "group"
                register(node, keys, T, pos, endPos)
                info[path].empty = (T.shape == "empty")
                for _, f in ipairs(T.fields) do
                    local asName = f.kind == "name"
                    describe(path .. segment(f.key, asName), path, f.key, extend(keys, f.key), f.v, f.s, fieldEnd(f), false)
                end
                return node
            elseif kind == "readonly" then
                node.source = shorten(D.src:sub(T.s, T.e), 8000)
                makeReadonly(node, why)
            else
                local val, err = encodeT(D, T, "")
                if val == nil and (kind == "list" or kind == "map") then
                    -- Code in some cells (name = T('X')) keeps the rest
                    -- editable: only those cells are read-only.
                    cells = {}
                    val = encodeT(D, T, "", cells, {})
                    if val == nil then cells = nil end
                end
                if val == nil then
                    node.source = D.src:sub(T.s, T.e)
                    if #node.source > 8000 then node.source = node.source:sub(1, 7999) .. "…" end
                    makeReadonly(node, err)
                else
                    node.kind, node.value = kind, val
                end
            end
        end
        register(node, keys, T, pos, endPos)
        -- Each code cell is a read-only node of its own, below the table:
        -- the page shows its source, and no edit may replace it.
        for _, c in ipairs(cells or {}) do
            local ckeys = {}
            for i, k in ipairs(keys) do ckeys[i] = k end
            for _, k in ipairs(c.keys) do ckeys[#ckeys + 1] = k end
            local cell = {
                path = path .. c.rel, parent = path, key = c.keys[#c.keys], line = lineOf(D, c.T.s),
                kind = "readonly", editable = false, reason = c.T.reason,
                source = shorten(c.T.source, 8000), cell = true, row = c.keys[1],
            }
            register(cell, ckeys, c.T, c.T.s, c.T.e)
        end
        if node.kind == "strings" and T.shape == "empty" and not forced then info[path].emptyDefault = true end
        return node
    end

    local function dropDescendants(keys)
        local prefix = canon(keys) .. "\1"
        local keep = {}
        for _, p in ipairs(order) do
            local k = canon(info[p].keys)
            if k:sub(1, #prefix) == prefix then
                byKeys[k] = nil
                nodes[p] = nil
                info[p] = nil
            else
                keep[#keep + 1] = p
            end
        end
        for i = #order, 1, -1 do order[i] = nil end
        for i, p in ipairs(keep) do order[i] = p end
    end

    for _, st in ipairs(D.stmts) do
        local keys = st.keys
        local root = keys[1]
        if not rootSeen[root] then
            rootSeen[root] = true
            roots[#roots + 1] = root
        end
        local path = keysPath(keys)
        local line = lineOf(D, st.s)
        local endPos = st.sepE or st.T.e
        local existing = byKeys[canon(keys)]

        -- `X = X or {}` again, for a table already declared: harmless.
        local redeclare = #keys == 1 and st.T.t == "code"
            and st.T.source:match("^" .. root .. "[ \t\r\n]+or[ \t\r\n]+{[ \t\r\n]*}$") ~= nil

        if existing and redeclare and existing.kind == "group" then
            -- nothing to do
        elseif existing then
            local first = existing.line
            dropDescendants(keys)
            makeReadonly(existing, ("set more than once in this file (lines %d and %d)"):format(first, line))
            existing.source = shorten(D.src:sub(st.T.s, st.T.e), 8000)
            existing.value = nil
        else
            local ok = true
            for i = 1, #keys - 1 do
                local sub = { table.unpack(keys, 1, i) }
                local pnode = byKeys[canon(sub)]
                if not pnode then
                    local pp = keysPath(sub)
                    local implicit = {
                        path = pp, parent = i > 1 and keysPath(sub, i - 1) or nil,
                        key = keys[i], kind = "group", type = "table", line = line, editable = true,
                    }
                    register(implicit, sub, nil, st.s, st.s)
                    info[pp].implicit = true
                    pnode = implicit
                end
                if pnode.kind ~= "group" then
                    local pinfo = info[pnode.path]
                    if pinfo and pinfo.emptyDefault then
                        pnode.kind, pnode.value = "group", nil
                        pinfo.emptyDefault = nil
                    else
                        if pnode.kind ~= "readonly" then
                            makeReadonly(pnode, "changed again at line " .. line)
                        end
                        ok = false
                        break
                    end
                end
                info[pnode.path].stmtChildren = true
            end
            if ok then
                describe(path, #keys > 1 and keysPath(keys, #keys - 1) or nil,
                    keys[#keys], keys, st.T, st.s, endPos, #keys == 1)
            end
        end
    end

    -- Statements and blocks that write into a setting at load time make it
    -- read-only: the file would not be the whole truth about its value.
    local toks, src = D.toks, D.src
    local function txt(i) local t = toks[i] return t and src:sub(t.s, t.e) end
    for _, sc in ipairs(D.scans) do
        for q = sc[1], sc[2] do
            local t = toks[q]
            if t.t == "name" and rootSeen[txt(q)] and txt(q - 1) ~= "." then
                local chain, r = { txt(q) }, q + 1
                while toks[r] and txt(r) == "." and toks[r + 1] and toks[r + 1].t == "name" do
                    chain[#chain + 1] = txt(r + 1)
                    r = r + 2
                end
                local written = false
                local nx = txt(r)
                if nx == "=" and toks[r].t == "op" then
                    written = true
                elseif nx == "[" or nx == "." then
                    local depth = 0
                    while toks[r] and r <= sc[2] do
                        local x = txt(r)
                        if x == "[" then depth = depth + 1
                        elseif x == "]" then depth = depth - 1
                        elseif depth == 0 and not (x == "." or toks[r].t == "name") then break end
                        r = r + 1
                    end
                    written = txt(r) == "=" and toks[r] and toks[r].t == "op"
                end
                if not written and txt(q - 1) == "(" and (txt(q - 2) == "insert" or txt(q - 2) == "remove")
                    and txt(q - 3) == "." and txt(q - 4) == "table" then
                    written = true
                end
                if written and #chain >= 2 then
                    for n = #chain, 2, -1 do
                        local node = byKeys[canon(chain, n)]
                        if node then
                            if node.kind ~= "group" and node.kind ~= "readonly" then
                                makeReadonly(node, "changed again by code at line " .. lineOf(D, t.s))
                            end
                            break
                        end
                    end
                end
            end
        end
    end
end

local function parseDoc(text, opts)
    if type(text) ~= "string" then fail("the file text is missing") end
    opts = type(opts) == "table" and opts or {}
    local D = { src = text }
    local ok, err = pcall(function()
        D.toks = lex(text)
        D.lines = computeLines(text)
        D.nl = text:find("\r\n", 1, true) and "\r\n" or "\n"
        D.unit = detectUnit(D)
        local S = { src = text, toks = D.toks, p = 1, dict = opts.dictKeys and true or false }
        D.stmts, D.scans = parseStatements(S)
    end)
    if not ok then
        if type(err) == "table" and err.msg then error(err, 0) end
        fail("the file could not be read: " .. cleanErr(err))
    end
    buildModel(D, opts)
    return D
end

--- The value tree at a path: the latest plain assignment that covers it, then
--- down through table fields. nil, depth, parentTable when a key is missing.
local function resolve(D, keys)
    local best, bestLen
    for _, st in ipairs(D.stmts) do
        local sk = st.keys
        if #sk <= #keys and (not bestLen or #sk >= bestLen) then
            local match = true
            for i = 1, #sk do
                if sk[i] ~= keys[i] then match = false break end
            end
            if match then best, bestLen = st, #sk end
        end
    end
    if not best then return nil, 1 end
    local T, field, container, idx = best.T, nil, nil, nil
    for i = bestLen + 1, #keys do
        if T.t ~= "table" then return nil, i end
        local found, fi
        for j, f in ipairs(T.fields) do
            if f.kind ~= "computed" and f.key == keys[i] then found, fi = f, j break end
        end
        if not found then return nil, i, T end
        container, field, idx, T = T, found, fi, found.v
    end
    return { T = T, field = field, container = container, index = idx, stmt = best, depth = bestLen }
end

--- The deepest setting on a path, refusing anything read-only on the way.
--- deep = the path goes inside that setting's value (a row, a row's field).
local function target(D, keys)
    local node, depth
    for i = 1, #keys do
        local n = D.byKeys[canon(keys, i)]
        if n then
            if n.kind == "readonly" then
                fail(n.path .. " is read-only here (" .. tostring(n.reason) .. "); edit it in the file")
            end
            node, depth = n, i
        end
    end
    if not node then fail("there is no setting at " .. table.concat(keys, ".")) end
    local deep = depth < #keys
    if deep and (node.kind == "group" or node.kind == "value") then
        fail("there is no setting at that path; new settings are added in the file")
    end
    return node, deep
end

-- ---------------------------------------------------------------------------
-- The sandbox: run a config and compare values
-- ---------------------------------------------------------------------------

local function joaat(s)
    s = tostring(s):lower()
    local h = 0
    for i = 1, #s do
        h = (h + s:byte(i)) & 0xFFFFFFFF
        h = (h + (h << 10)) & 0xFFFFFFFF
        h = h ~ (h >> 6)
    end
    h = (h + (h << 3)) & 0xFFFFFFFF
    h = h ~ (h >> 11)
    h = (h + (h << 15)) & 0xFFFFFFFF
    if h >= 0x80000000 then h = h - 0x100000000 end
    return h
end

--- Callable, indexable stand-in for anything the file uses but does not define
--- (T(), Citizen, exports...). Numeric keys read as nil so ipairs stops.
local function newStub()
    local s = {}
    local mt = {
        __index = function(_, k) if type(k) == "number" then return nil end return s end,
        __call = function() return s end,
        __concat = function() return "" end,
        __len = function() return 0 end,
        __lt = function() return false end,
        __le = function() return false end,
        __tostring = function() return "?" end,
    }
    for _, m in ipairs({ "__add", "__sub", "__mul", "__div", "__mod", "__pow", "__unm", "__idiv",
        "__band", "__bor", "__bxor", "__shl", "__shr", "__bnot" }) do
        mt[m] = function() return 0 end
    end
    return setmetatable(s, mt)
end

local function vec(n)
    return function(x, y, z, w)
        local v = { __type = "vec" .. n, x = x, y = y }
        if n >= 3 then v.z = z end
        if n == 4 then v.w = w end
        return v
    end
end

--- A fresh sandbox: pure helpers only, vectors as the tagged tables of §3.1,
--- and a stub for every other global, remembered once read. Nothing the file
--- does can reach the server, and two runs of one text are identical.
local function makeEnv()
    local base = {
        math = setmetatable({ random = function(a) return a or 0 end, randomseed = function() end }, { __index = math }),
        string = string, table = table, utf8 = utf8,
        pairs = pairs, ipairs = ipairs, next = next, select = select, type = type,
        tostring = tostring, tonumber = tonumber, rawget = rawget, rawset = rawset,
        rawequal = rawequal, rawlen = rawlen, setmetatable = setmetatable, getmetatable = getmetatable,
        assert = assert, error = error, pcall = pcall, unpack = table.unpack,
        print = function() end,
        GetHashKey = joaat, joaat = joaat,
        vector2 = vec(2), vector3 = vec(3), vector4 = vec(4), vec2 = vec(2), vec3 = vec(3), vec4 = vec(4),
    }
    return setmetatable({}, {
        __index = function(t, k)
            local b = base[k]
            if b ~= nil then return b end
            local s = newStub()
            rawset(t, k, s)
            return s
        end,
    })
end

local function loadEnv(text)
    local env, err = evaluate(text, "config", makeEnv())
    if not env then return nil, cleanErr(err) end
    return env
end

local function evalData(src)
    local fn, err = load("return " .. src, "=value", "t", makeEnv())
    if not fn then return nil, err end
    local ok, v = pcall(fn)
    if not ok then return nil, v end
    return v
end

local function valueAt(env, keys)
    local cur = env
    for _, k in ipairs(keys) do
        if type(cur) ~= "table" then return nil end
        cur = rawget(cur, k)
    end
    return cur
end

--- A tree of key paths that may differ between two runs.
local function exemptTree(list)
    local root = {}
    for _, keys in ipairs(list) do
        local t = root
        for _, k in ipairs(keys) do
            t.kids = t.kids or {}
            local c = t.kids[k]
            if not c then
                c = {}
                t.kids[k] = c
            end
            t = c
        end
        t.stop = true
    end
    return root
end

--- a and b are the same everywhere except under the exempt paths.
local function sameExcept(a, b, tree)
    if tree.stop then return true end
    if not tree.kids or type(a) ~= "table" or type(b) ~= "table" then return same(a, b) end
    for key, va in pairs(a) do
        local sub = tree.kids[key]
        if sub then
            if not sameExcept(va, rawget(b, key), sub) then return false end
        elseif not same(va, rawget(b, key)) then
            return false
        end
    end
    for key, vb in pairs(b) do
        if rawget(a, key) == nil then
            local sub = tree.kids[key]
            if not sub or not sameExcept(nil, vb, sub) then return false end
        end
    end
    return true
end

local REMOVED = {}

--- Does an encoded value hold a code cell ({ __type = "code" })?
local function hasCode(v, depth)
    if type(v) ~= "table" then return false end
    if rawget(v, "__type") == "code" then return true end
    depth = (depth or 0) + 1
    if depth > 40 then return false end
    for _, x in pairs(v) do
        if hasCode(x, depth) then return true end
    end
    return false
end

--- A code cell at or below keys: the node, or nil.
local function codeUnder(D, keys)
    local prefix = canon(keys)
    for k, n in pairs(D.byKeys) do
        if n.cell and (k == prefix or k:sub(1, #prefix + 1) == prefix .. "\1") then return n end
    end
    return nil
end

--- Run both texts and prove the edit did what it said and nothing else.
---   D         the parsed file before the edit
---   keys      the edited path (nil: nothing may change at all)
---   expected  what the file must now hold there (JSON-side), REMOVED, or nil
--- Read-only settings (code) may change: they are worked out from others, so
--- `jobs = Config.PoliceJobList` follows an edit of Config.PoliceJobList.
local function verify(D, newText, keys, expected, opts)
    local oldEnv, e1 = loadEnv(D.src)
    if not oldEnv then fail("the file does not run as it is, so a change cannot be checked (" .. e1 .. ")") end
    local newEnv, e2 = loadEnv(newText)
    if not newEnv then fail("the change would break the file (" .. e2 .. "); nothing was written") end
    local exempt = {}
    if keys then exempt[1] = keys end
    for path, node in pairs(D.model.nodes) do
        if node.kind == "readonly" then exempt[#exempt + 1] = D.info[path].keys end
    end
    if not sameExcept(oldEnv, newEnv, exemptTree(exempt)) then
        fail("the change would alter other settings; nothing was written")
    end
    local D2 = parseDoc(newText, opts)
    if keys and expected ~= nil then
        local slot = resolve(D2, keys)
        if expected == REMOVED then
            if slot then fail("the entry is still there after removing it") end
            if valueAt(newEnv, keys) ~= nil then fail("the entry is still set once the file has run") end
        else
            if not slot then fail("the change did not land; nothing was written") end
            local enc = encodeT(D2, slot.T, "", {}, {})
            if enc == nil or not eqEnc(enc, expected) then
                fail("the file would not hold the value asked for; nothing was written")
            end
            -- A value with code cells in it (moved or kept verbatim) cannot be
            -- run on its own; its data was compared above and everything
            -- around it by sameExcept.
            if not hasCode(expected) then
                local want, err = evalData(D2.src:sub(slot.T.s, slot.T.e))
                if err then fail("the new value does not run (" .. cleanErr(err) .. ")") end
                if not same(valueAt(newEnv, keys), want) then
                    fail("this setting is changed again further down the file, so the edit would not take effect")
                end
            end
        end
    end
    return D2
end

-- ---------------------------------------------------------------------------
-- Writing values (the serialiser)
-- ---------------------------------------------------------------------------

local serValue

local function serString(s, hint)
    local q = hint and hint.t == "string" and hint.quote or '"'
    if q:sub(1, 1) == "[" then
        local lvl = q:match("^%[(=*)%[")
        local close = "]" .. lvl .. "]"
        if not s:find("\r", 1, true) and not (s .. "]"):find(close, 1, true) then
            local lead = (hint.nlAfter or s:sub(1, 1) == "\n") and "\n" or ""
            return q .. lead .. s .. close
        end
        q = '"'
    end
    return quoteString(s, q)
end

local function serHash(v)
    local name = rawget(v, "name")
    if type(name) ~= "string" or name == "" or name:find("[`\\\r\n]") then
        fail("a hash needs a plain name, e.g. INPUT_CONTEXT")
    end
    return "`" .. name .. "`"
end

local function serVector(v, hint)
    local n = tonumber(rawget(v, "__type"):sub(4))
    local family = "vector"
    local h = hint and hint.t == "vector" and hint or nil
    if h and h.ctor:sub(1, 6) ~= "vector" then family = "vec" end
    local parts = {}
    for i = 1, n do
        local c = rawget(v, COMP[i])
        if type(c) ~= "number" then fail("a vector needs a number for " .. COMP[i]) end
        parts[i] = fmtNumber(c, h and h.compSrc[i], true)
    end
    local pre, sep, post = "", ", ", ""
    if h then pre, sep, post = h.pre, h.sep or ", ", h.post end
    return family .. n .. "(" .. pre .. table.concat(parts, sep) .. post .. ")"
end

local function keyText(key, style)
    if math.type(key) == "integer" then return "[" .. key .. "]" end
    if style and style.kind == "bracket" and type(style.key) == "string" then
        return "[" .. quoteString(key, style.keyQuote or '"') .. "]"
    end
    if isIdent(key) then return key end
    return "[" .. quoteString(key, '"') .. "]"
end

--- JSON-side object entries with Lua keys, in the hint's order then sorted.
local function objectEntries(v, hint)
    if hint and hint.shape ~= "keyed" then hint = nil end
    local intKeys = rawget(v, "__int_keys") == true
    local hintInts = hint and math.type(hint.fields[1].key) == "integer"
    local list, seen = {}, {}
    for k, val in pairs(v) do
        if k ~= "__int_keys" then
            local key = k
            if type(k) == "string" then
                if intKeys or hintInts then
                    local n = math.tointeger(tonumber(k))
                    if n then key = n elseif intKeys then fail("the key " .. k .. " is not a whole number") end
                end
            elseif type(k) == "number" then
                key = math.tointeger(k)
                if not key then fail("a key cannot have decimals") end
            else
                fail("a key must be text or a whole number")
            end
            if seen[key] then fail("the key " .. tostring(key) .. " appears twice") end
            seen[key] = true
            list[#list + 1] = { key = key, value = val }
        end
    end
    local rank = {}
    if hint then
        for i, f in ipairs(hint.fields) do
            if f.key ~= nil and rank[f.key] == nil then rank[f.key] = i end
        end
    end
    table.sort(list, function(a, b)
        local ra, rb = rank[a.key], rank[b.key]
        if ra and rb then return ra < rb end
        if ra then return true end
        if rb then return false end
        local ta, tb = type(a.key), type(b.key)
        if ta ~= tb then return ta == "number" end
        return a.key < b.key
    end)
    return list
end

local function findField(T, key)
    if not T or T.t ~= "table" then return nil end
    for i, f in ipairs(T.fields) do
        if f.kind ~= "computed" and f.key == key then return f, i end
    end
end

--- Spaces between a new key and its "=", lined up with the siblings when they
--- are aligned in a column.
local function gapFor(D, keyTxt, indent, hf, sibling)
    if hf and hf.kind ~= "pos" and hf.eqS then
        local g = D.src:sub(hf.keyE + 1, hf.eqS - 1)
        if g:match("^[ \t]*$") and #g > 0 then
            if #g == 1 then return g end
            -- aligned: keep the column
            local col = hf.eqS - lineStart(D, hf.eqS)
            local want = col - #indent - #keyTxt
            if cleanBefore(D, hf.keyS) and want >= 1 then return (" "):rep(want) end
            return " "
        end
        return g:match("^[ \t]*$") and g or " "
    end
    if sibling and sibling.kind ~= "pos" and sibling.eqS and cleanBefore(D, sibling.keyS) then
        local g = D.src:sub(sibling.keyE + 1, sibling.eqS - 1)
        if g:match("^ +$") and #g > 1 then
            local col = sibling.eqS - lineStart(D, sibling.eqS)
            local want = col - #indent - #keyTxt
            if want >= 1 then return (" "):rep(want) end
        end
    end
    return " "
end

local function afterEq(D, f)
    if f and f.eqS and f.v then
        local g = D.src:sub(f.eqS + 1, f.v.s - 1)
        if g:match("^[ \t]+$") then return g end
    end
    return " "
end

local function serTable(v, ctx, hint, depth)
    local D = ctx.D
    local kind = valueKind(v)
    local entries = {}
    if hint and hint.t ~= "table" then hint = nil end
    if kind == "array" then
        local posHint = hint and hint.shape == "pos" and hint or nil
        for i = 1, #v do
            local h
            if posHint then
                local f = posHint.fields[i] or posHint.fields[#posHint.fields]
                h = f.v
            end
            entries[i] = { value = v[i], hint = h }
        end
    else
        local keyedHint = hint and hint.shape == "keyed" and hint or nil
        for _, kv in ipairs(objectEntries(v, keyedHint)) do
            local hf = keyedHint and findField(keyedHint, kv.key) or nil
            entries[#entries + 1] = { key = kv.key, value = kv.value, hint = hf and hf.v, hf = hf,
                keyTxt = keyText(kv.key, hf or (keyedHint and keyedHint.fields[1])) }
        end
    end
    if #entries == 0 then return "{}" end

    local inline = ctx.inline
    if inline == nil then
        if hint and #hint.fields > 0 then
            inline = lineOf(D, hint.s) == lineOf(D, hint.e)
        else
            -- One line when short: a list of plain values, or named values that
            -- hold at most a small table of plain values ({ a = 1, b = { 1, 2 } }).
            -- A list of tables is rows: one per line.
            inline = true
            for _, e in ipairs(entries) do
                local ek = valueKind(e.value)
                if ek == "array" or ek == "object" then
                    if kind == "array" then inline = false break end
                    for _, x in pairs(e.value) do
                        local xk = valueKind(x)
                        if xk == "array" or xk == "object" then inline = false break end
                    end
                end
                if not inline then break end
            end
        end
    end

    if inline then
        local sub = { D = D, indent = ctx.indent, unit = ctx.unit, nl = ctx.nl, inline = true }
        local parts = {}
        for i, e in ipairs(entries) do
            local val = serValue(e.value, sub, e.hint, depth)
            parts[i] = e.keyTxt and (e.keyTxt .. " = " .. val) or val
        end
        local body = table.concat(parts, ", ")
        if hint or ctx.inline or #body <= 100 then
            local pad = hint and hint.pad or " "
            return "{" .. pad .. body .. pad .. "}"
        end
    end

    local childIndent = ctx.indent .. ctx.unit
    if hint and #hint.fields > 0 and cleanBefore(D, hint.fields[1].s) then
        local hi = lineIndent(D, hint.fields[1].s)
        local bi = lineIndent(D, hint.s)
        if hi:sub(1, #bi) == bi and #hi > #bi then childIndent = ctx.indent .. hi:sub(#bi + 1) end
    end
    local sub = { D = D, indent = childIndent, unit = ctx.unit, nl = ctx.nl }
    local lines = {}
    for i, e in ipairs(entries) do
        local val = serValue(e.value, sub, e.hint, depth)
        local head = ""
        if e.keyTxt then
            head = e.keyTxt .. gapFor(D, e.keyTxt, childIndent, e.hf, hint and hint.fields[1]) .. "=" .. afterEq(D, e.hf)
        end
        lines[i] = childIndent .. head .. val .. ","
    end
    return "{" .. ctx.nl .. table.concat(lines, ctx.nl) .. ctx.nl .. ctx.indent .. "}"
end

--- Lua source for a JSON-side value, in the style of hint (the parsed value it
--- replaces, or a sibling). Refuses anything that is not data.
function serValue(v, ctx, hint, depth)
    depth = (depth or 0) + 1
    if depth > 40 then fail("the value is nested too deeply") end
    local kind, why = valueKind(v)
    if not kind then fail(why .. " cannot be written into a config file; only data can") end
    if kind == "string" then return serString(v, hint) end
    if kind == "number" then return fmtNumber(v, hint and hint.t == "number" and hint.src or nil) end
    if kind == "boolean" then return tostring(v) end
    if kind == "vector" then return serVector(v, hint) end
    if kind == "hash" then return serHash(v) end
    return serTable(v, ctx, hint, depth)
end

--- Check a value is plain data all the way down, before any work is done.
local function checkData(v, depth)
    depth = depth or 0
    if depth > 40 then fail("the value is nested too deeply") end
    local kind, why = valueKind(v)
    if not kind then fail(why .. " cannot be written into a config file; only data can") end
    if kind == "array" or kind == "object" then
        for k, x in pairs(v) do
            if k ~= "__int_keys" then
                if type(k) ~= "string" and math.type(k) ~= "integer" then fail("a key must be text or a whole number") end
                checkData(x, depth + 1)
            end
        end
    end
end

-- ---------------------------------------------------------------------------
-- Text edits
-- ---------------------------------------------------------------------------

--- Apply { s, e, text } replacements (an insertion has e = s - 1).
local function applyEdits(src, edits)
    table.sort(edits, function(a, b)
        if a[1] ~= b[1] then return a[1] > b[1] end
        return a[2] > b[2]
    end)
    local limit = math.huge
    for _, ed in ipairs(edits) do
        if ed[2] >= limit then error("two changes overlap") end
        src = src:sub(1, ed[1] - 1) .. ed[3] .. src:sub(ed[2] + 1)
        limit = ed[1]
    end
    return src
end

--- Where a field's own comment lines start (the lines directly above it).
--- A comment above the first field counts only when other fields have their
--- own too; otherwise it describes the whole table and stays put.
local function rawLead(D, f)
    if not cleanBefore(D, f.s) then return f.s, false end
    local li = lineOf(D, f.s)
    local start, found = D.lines[li], false
    local j = li - 1
    while j >= 1 do
        local text = lineText(D, j)
        if isCommentLine(text) and not isRuleLine(text) then
            start, found = D.lines[j], true
            j = j - 1
        else
            break
        end
    end
    return start, found
end

local function leadingStart(D, T, i)
    local f = T.fields[i]
    local start, found = rawLead(D, f)
    if found and i == 1 then
        local other = false
        for j = 2, #T.fields do
            local _, fj = rawLead(D, T.fields[j])
            if fj then other = true break end
        end
        if not other then return lineStart(D, f.s) end
    end
    return start
end

local function hasCommentsInside(D, T)
    local pos = T.s + 1
    for _, f in ipairs(T.fields) do
        if D.src:sub(pos, f.s - 1):find("--", 1, true) then return true end
        pos = f.e + 1
    end
    return D.src:sub(pos, T.closeS - 1):find("--", 1, true) ~= nil
end

--- Every field of T on lines of its own.
local function ownLines(D, T)
    for _, f in ipairs(T.fields) do
        if not cleanBefore(D, f.s) or not cleanAfter(D, fieldEnd(f)) then return false end
    end
    return #T.fields > 0
end

--- Remove fields i..j of T cleanly: their lines (with their own comments)
--- when they have lines to themselves, otherwise just their text and comma.
local function removeRange(D, T, i, j, edits)
    local src, fields = D.src, T.fields
    if i == 1 and j == #fields and not hasCommentsInside(D, T) then
        edits[#edits + 1] = { T.s, T.e, "{}" }
        return
    end
    local first, last = fields[i], fields[j]
    local s0, e0 = first.s, fieldEnd(last)
    -- The last field has no comma of its own: take the one before it instead,
    -- when nothing but spaces sits between (a comment there stays).
    if not last.sepE and i > 1 then
        local prev = fields[i - 1]
        if src:sub(prev.sepE + 1, first.s - 1):match("^[ \t]*$") then
            edits[#edits + 1] = { prev.sepS, e0, "" }
            return
        end
    end
    if cleanBefore(D, s0) and cleanAfter(D, e0) then
        edits[#edits + 1] = { leadingStart(D, T, i), lineEnd(D, e0), "" }
        -- The file leaves its last row without a comma: so does the new last.
        if j == #fields and not last.sepE and i > 1 and fields[i - 1].sepE then
            local prev = fields[i - 1]
            edits[#edits + 1] = { prev.sepS, prev.sepE, "" }
        end
        return
    end
    local e1 = e0
    while src:sub(e1 + 1, e1 + 1):match("^[ \t]$") do e1 = e1 + 1 end
    local nxt = src:sub(e1 + 1, e1 + 1)
    if nxt == "" or nxt == "\n" or nxt == "\r" then
        while s0 > 1 and src:sub(s0 - 1, s0 - 1):match("^[ \t]$") do s0 = s0 - 1 end
    end
    edits[#edits + 1] = { s0, e1, "" }
end

--- Insert new fields at index (1..n+1) of T, laid out like their siblings.
--- items = { { key = nil|key, value = v, hint = T|nil } }
local function insertFields(D, T, index, items, edits)
    local src, fields = D.src, T.fields
    local n = #fields
    local unit = D.unit
    local style = fields[math.min(math.max(index, 1), math.max(n, 1))]

    local function render(item, indent, nl, inline)
        local ctx = { D = D, indent = indent, unit = unit, nl = nl, inline = inline }
        -- item.raw: source text copied verbatim from another row (Duplicate).
        local val = item.raw or serValue(item.value, ctx, item.hint)
        if item.key ~= nil then
            local kt = keyText(item.key, style and style.kind ~= "pos" and style or nil)
            local gap = inline and " " or gapFor(D, kt, indent, nil, style)
            return kt .. gap .. "=" .. afterEq(D, style) .. val
        end
        return val
    end

    if n == 0 then
        local inner = src:sub(T.s + 1, T.closeS - 1)
        local nl = lineNl(D, T.s)
        if inner:find("\n", 1, true) then
            if cleanBefore(D, T.closeS) then
                local indent = lineIndent(D, T.closeS) .. unit
                local buf = {}
                for _, it in ipairs(items) do buf[#buf + 1] = indent .. render(it, indent, nl) .. "," .. nl end
                local pos = lineStart(D, T.closeS)
                edits[#edits + 1] = { pos, pos - 1, table.concat(buf) }
            else
                local parts = {}
                for _, it in ipairs(items) do parts[#parts + 1] = render(it, lineIndent(D, T.s) .. unit, nl) end
                edits[#edits + 1] = { T.closeS, T.closeS - 1, " " .. table.concat(parts, ", ") .. " " }
            end
            return
        end
        local base = lineIndent(D, T.s)
        local multi = false
        for _, it in ipairs(items) do
            local k = valueKind(it.value)
            if (k == "array" or k == "object") and next(it.value) ~= nil then multi = true end
        end
        local text
        if multi then
            local buf = {}
            for _, it in ipairs(items) do
                buf[#buf + 1] = base .. unit .. render(it, base .. unit, nl) .. "," .. nl
            end
            text = "{" .. nl .. table.concat(buf) .. base .. "}"
        else
            local parts = {}
            for _, it in ipairs(items) do parts[#parts + 1] = render(it, base, nl) end
            text = "{ " .. table.concat(parts, ", ") .. " }"
        end
        edits[#edits + 1] = { T.s, T.e, text }
        return
    end

    local ref = fields[math.min(index, n)]
    local lineMode = cleanBefore(D, ref.s) and cleanAfter(D, fieldEnd(ref))
    local nl = lineNl(D, ref.s)
    local indent = lineIndent(D, ref.s)

    if lineMode then
        -- Appending after a last row with no comma: it gets one, and the new
        -- last row goes without, as the file had it.
        local bareLast = index > n and not fields[n].sepE
        local buf = {}
        for k, it in ipairs(items) do
            local comma = (bareLast and k == #items) and "" or ","
            buf[#buf + 1] = indent .. render(it, indent, nl) .. comma .. nl
        end
        local text = table.concat(buf)
        if index <= n then
            local pos = leadingStart(D, T, index)
            edits[#edits + 1] = { pos, pos - 1, text }
        else
            local last = fields[n]
            if not last.sepE then edits[#edits + 1] = { last.v.e + 1, last.v.e, "," } end
            local pos = lineEnd(D, fieldEnd(last)) + 1
            if src:sub(pos - 1, pos - 1) ~= "\n" then text = nl .. text end
            edits[#edits + 1] = { pos, pos - 1, text }
        end
        return
    end

    local parts = {}
    for _, it in ipairs(items) do parts[#parts + 1] = render(it, indent, nl) end
    local joined = table.concat(parts, ", ")
    if index <= n then
        edits[#edits + 1] = { ref.s, ref.s - 1, joined .. ", " }
    else
        local last = fields[n]
        if last.sepE then
            edits[#edits + 1] = { last.sepE + 1, last.sepE, " " .. joined .. "," }
        else
            edits[#edits + 1] = { last.v.e + 1, last.v.e, ", " .. joined }
        end
    end
end

--- The context to write a value that replaces the one at pos.
local function ctxAt(D, pos)
    return { D = D, indent = lineIndent(D, pos), unit = D.unit, nl = lineNl(D, pos) }
end

--- Edits that turn parsed value T into want, touching as little as possible.
--- A table that must both lose and gain fields loses them first; pending.more
--- asks for another pass on the new text.
local function patch(D, T, want, edits, pending)
    local wk = valueKind(want)
    if T.t == "code" then
        fail("part of this value is worked out by code (" .. shorten(T.source, 60) .. "); edit it in the file")
    end
    if T.t ~= "table" then
        if T.t ~= "nil" then
            local cur = encodeT(D, T)
            if cur ~= nil and eqEnc(cur, want) then return end
        end
        edits[#edits + 1] = { T.s, T.e, serValue(want, ctxAt(D, T.s), T) }
        return
    end
    if wk ~= "array" and wk ~= "object" then
        edits[#edits + 1] = { T.s, T.e, serValue(want, ctxAt(D, T.s), T) }
        return
    end
    local fields = T.fields
    local empty = next(want) == nil
    if T.shape == "empty" then
        if empty then return end
        edits[#edits + 1] = { T.s, T.e, serValue(want, ctxAt(D, T.s), nil) }
        return
    end
    if T.shape == "bad" then
        fail("this table " .. T.why .. "; edit it in the file")
    end
    if T.shape == "pos" and wk == "array" then
        local n, m = #fields, #want
        for i = 1, math.min(n, m) do patch(D, fields[i].v, want[i], edits, pending) end
        if m > n then
            local items = {}
            for i = n + 1, m do items[#items + 1] = { value = want[i], hint = fields[n].v } end
            insertFields(D, T, n + 1, items, edits)
        elseif m < n then
            removeRange(D, T, m + 1, n, edits)
        end
        return
    end
    if T.shape == "keyed" and (wk == "object" or empty) then
        local entries = empty and {} or objectEntries(want, T)
        local wantBy = {}
        for _, e in ipairs(entries) do wantBy[e.key] = e end
        local removed, used = {}, {}
        for i, f in ipairs(fields) do
            local e = wantBy[f.key]
            if e then
                used[f.key] = true
                patch(D, f.v, e.value, edits, pending)
            elseif f.v.t ~= "nil" then
                removed[#removed + 1] = i
            end
        end
        local additions = {}
        for _, e in ipairs(entries) do
            if not used[e.key] then additions[#additions + 1] = { key = e.key, value = e.value } end
        end
        if #removed > 0 then
            local runStart = removed[1]
            for k = 2, #removed + 1 do
                if removed[k] ~= removed[k - 1] + 1 then
                    removeRange(D, T, runStart, removed[k - 1], edits)
                    runStart = removed[k]
                end
            end
            if #additions > 0 then pending.more = true end
        elseif #additions > 0 then
            insertFields(D, T, #fields + 1, additions, edits)
        end
        return
    end
    -- the shape changed (a list became a table of settings, or the reverse)
    edits[#edits + 1] = { T.s, T.e, serValue(want, ctxAt(D, T.s), T) }
end

--- Patch the value at keys until the file holds want (normally one pass).
local function settle(text, keys, want, opts)
    for _ = 1, 8 do
        local D = parseDoc(text, opts)
        local slot = resolve(D, keys)
        if not slot then fail("the setting disappeared while it was being written") end
        local edits, pending = {}, {}
        patch(D, slot.T, want, edits, pending)
        if #edits == 0 then return text end
        text = applyEdits(text, edits)
    end
    fail("the change did not settle; nothing was written")
end

local function normOpts(opts)
    return type(opts) == "table" and opts or {}
end

--- The list or map table at a path, for the row operations.
local function rowTarget(D, keys, wantKinds)
    local node, deep = target(D, keys)
    if not deep and not wantKinds[node.kind] then
        fail(node.path .. " is a " .. node.kind .. ", not a list")
    end
    local slot = resolve(D, keys)
    if not slot or slot.T.t ~= "table" then fail("there is no table at that path") end
    if slot.T.shape == "bad" then fail("this table " .. slot.T.why .. "; edit it in the file") end
    -- Code cells come back as { __type = "code" } and move with their rows.
    local cur, err = encodeT(D, slot.T, "", {}, {})
    if cur == nil then fail(err) end
    return slot.T, cur, node, deep
end

local function keyOf(T, key)
    for i, f in ipairs(T.fields) do
        if f.key == key then return f, i end
        if type(key) == "string" and math.type(f.key) == "integer" and tonumber(key) == f.key then return f, i end
        if math.type(key) == "integer" and f.key == tostring(key) then return f, i end
    end
end

local function encKey(k)
    return math.type(k) == "integer" and tostring(k) or k
end

-- ---------------------------------------------------------------------------
-- Statement rows (hub spec §8.4): a table filled in by separate lines
--
--     Config.Catalog = {}
--     Config.Catalog.GeneralStore = { sell = { ... }, buy = { ... } }
--     Config.Catalog.Gunsmith     = { sell = { ... }, buy = { ... } }
--
-- is a collection whose rows are those statements. The caller names such a
-- table in opts.statementRows = { [path] = true } (the hub does, once it has
-- seen the table is a collection); only then do Insert, Remove and RenameKey
-- treat its rows as statements:
--   Insert     appends `Config.Catalog.<Key> = { ... }` after the last row
--              (`Config.Catalog["Key with spaces"]` when the key is not a
--              name), written in the style of the last row
--   Remove     deletes the row's statement with the comment lines directly
--              above it
--   RenameKey  rewrites the last segment of the row's left-hand side
-- Every one is verified like any other edit: the file runs, and nothing but
-- that row changed.
-- ---------------------------------------------------------------------------

--- Is keys a table the caller declared as statement rows?
local function isStatementTable(keys, opts)
    local sr = opts.statementRows
    if type(sr) ~= "table" then return false end
    local want = canon(keys)
    for p, on in pairs(sr) do
        if on then
            local ok, k2 = pcall(parsePath, p)
            if ok and canon(k2) == want then return true end
        end
    end
    return false
end

--- The row statements of the table at keys: rows = { { key, st } } in file
--- order, deeper = { key -> line } for rows written into again further down
--- (Config.Catalog.X.sell = ...), anchor = the table's own statement or nil.
--- nil when keys is not a statement table.
local function statementRows(D, keys, opts)
    if not isStatementTable(keys, opts) then return nil end
    local node = D.byKeys[canon(keys)]
    if not node or node.kind ~= "group" then return nil end
    local info = D.info[node.path]
    if info and info.T and info.T.t == "table" and info.T.shape ~= "empty" then
        fail("some rows of " .. node.path .. " are written inside its own table; change them in the file")
    end
    local rows, deeper, anchor, n = {}, {}, nil, #keys
    for _, st in ipairs(D.stmts) do
        local sk = st.keys
        if #sk >= n then
            local match = true
            for i = 1, n do
                if sk[i] ~= keys[i] then match = false break end
            end
            if match then
                if #sk == n then
                    anchor = st
                elseif #sk == n + 1 then
                    rows[#rows + 1] = { key = sk[n + 1], st = st }
                else
                    deeper[sk[n + 1]] = deeper[sk[n + 1]] or lineOf(D, st.s)
                end
            end
        end
    end
    return rows, deeper, anchor
end

--- Lua text for a path of keys: Config.Catalog["General Store"].
local function keysText(keys)
    local out = { tostring(keys[1]) }
    for i = 2, #keys do
        local k = keys[i]
        if math.type(k) == "integer" then
            out[i] = "[" .. k .. "]"
        elseif isIdent(k) then
            out[i] = "." .. k
        else
            out[i] = "[" .. quoteString(k, '"') .. "]"
        end
    end
    return table.concat(out)
end

local function checkRowKey(key)
    if type(key) ~= "string" or key == "" then fail("a new row needs a name") end
    if key == "__type" or key == "__int_keys" then fail("that key is reserved") end
    if #key > 128 then fail("the name is too long") end
    if key:find("[\r\n]") then fail("a name cannot hold a line break") end
end

--- Append a row statement.
local function stmtInsert(D, keys, rows, anchor, key, value, opts)
    checkRowKey(key)
    for _, r in ipairs(rows) do
        if r.key == key then fail("the key " .. key .. " is already there") end
    end
    local full = extend(keys, key)
    if D.byKeys[canon(full)] then fail("the key " .. key .. " is already there") end
    local last = rows[#rows] and rows[#rows].st or anchor
    local src = D.src
    local indent = last and lineIndent(D, last.s) or ""
    local nl = last and lineNl(D, last.s) or D.nl
    local hint = rows[#rows] and rows[#rows].st.T or nil
    local val = serValue(value, { D = D, indent = indent, unit = D.unit, nl = nl }, hint)
    local stmt = indent .. keysText(full) .. " = " .. val
    local newText
    if last then
        local le = lineEnd(D, last.sepE or last.T.e)
        if src:sub(le, le) == "\n" then
            newText = src:sub(1, le) .. nl .. stmt .. nl .. src:sub(le + 1)
        else
            newText = src .. nl .. nl .. stmt .. nl
        end
    else
        newText = src .. ((src == "" or src:sub(-1) == "\n") and "" or nl) .. nl .. stmt .. nl
    end
    verify(D, newText, full, value, opts)
    return newText
end

--- The row statement for key, refusing a row the file writes into again.
local function stmtRow(D, keys, rows, deeper, key)
    local found
    for _, r in ipairs(rows) do
        if r.key == key then found = r break end
    end
    if not found then fail("there is no row " .. tostring(key)) end
    if deeper[key] then
        fail(("the row %s is changed again at line %d; change it in the file"):format(tostring(key), deeper[key]))
    end
    local node = D.byKeys[canon(extend(keys, key))]
    if node and node.kind == "readonly" then
        fail(node.path .. " is read-only here (" .. tostring(node.reason) .. "); edit it in the file")
    end
    return found
end

--- Delete a row statement with the comment lines directly above it.
local function stmtRemove(D, keys, rows, deeper, key, opts)
    local row = stmtRow(D, keys, rows, deeper, key)
    local st = row.st
    local s0, e0 = st.s, st.sepE or st.T.e
    if not cleanBefore(D, s0) or not cleanAfter(D, e0) then
        fail("this row shares its line with other code; remove it in the file")
    end
    local li = lineOf(D, s0)
    local j = li - 1
    while j >= 1 and isCommentLine(lineText(D, j)) do j = j - 1 end
    local lastLine = lineOf(D, e0)
    local from = D.lines[j + 1]
    local to = D.lines[lastLine + 1] and (D.lines[lastLine + 1] - 1) or #D.src
    -- A blank line above and a blank line (or the end of the file) below:
    -- take the one above too, so no double gap is left behind.
    if j >= 1 and lineText(D, j):match("^[ \t]*$") then
        local below = D.lines[lastLine + 1] and lineText(D, lastLine + 1) or ""
        if below:match("^[ \t]*$") then from = D.lines[j] end
    end
    local newText = D.src:sub(1, from - 1) .. D.src:sub(to + 1)
    verify(D, newText, extend(keys, key), REMOVED, opts)
    return newText
end

--- The token index of the token that starts at pos.
local function tokenAt(D, pos)
    local toks = D.toks
    local lo, hi = 1, #toks
    while lo < hi do
        local mid = (lo + hi) // 2
        if toks[mid].s < pos then lo = mid + 1 else hi = mid end
    end
    return lo
end

--- Rename a row statement: only the last segment of its left-hand side changes.
local function stmtRename(D, keys, rows, deeper, oldKey, newKey, opts)
    checkRowKey(newKey)
    local row = stmtRow(D, keys, rows, deeper, oldKey)
    if newKey == oldKey then return D.src end
    for _, r in ipairs(rows) do
        if r.key == newKey then fail("the key " .. newKey .. " is already there") end
    end
    if D.byKeys[canon(extend(keys, newKey))] then fail("the key " .. newKey .. " is already there") end
    local st = row.st
    local toks, src = D.toks, D.src
    local k = tokenAt(D, st.s)
    local segS, prevE
    while toks[k] and toks[k].s < st.T.s do
        local t = toks[k]
        local x = src:sub(t.s, t.e)
        if t.t == "op" and x == "=" then break end
        if t.t == "op" and (x == "." or x == "[") then segS = t.s end
        prevE = t.e
        k = k + 1
    end
    if not segS or not prevE then fail("the row's name could not be found on its line") end
    local oldSeg = src:sub(segS, prevE)
    local newSeg
    if isIdent(newKey) and oldSeg:sub(1, 1) == "." then
        newSeg = "." .. newKey
    else
        local q = oldSeg:match('^%[[ \t]*(["\'])') or '"'
        newSeg = "[" .. quoteString(newKey, q) .. "]"
    end
    local newText = src:sub(1, segS - 1) .. newSeg .. src:sub(prevE + 1)
    -- Proof: the file runs, the row moved from the old key to the new one,
    -- and nothing else changed.
    local oldEnv, e1 = loadEnv(src)
    if not oldEnv then fail("the file does not run as it is, so a change cannot be checked (" .. e1 .. ")") end
    local newEnv, e2 = loadEnv(newText)
    if not newEnv then fail("the change would break the file (" .. e2 .. "); nothing was written") end
    local oldFull, newFull = extend(keys, oldKey), extend(keys, newKey)
    local exempt = { oldFull, newFull }
    for path, node in pairs(D.model.nodes) do
        if node.kind == "readonly" then exempt[#exempt + 1] = D.info[path].keys end
    end
    if not sameExcept(oldEnv, newEnv, exemptTree(exempt)) then
        fail("the change would alter other settings; nothing was written")
    end
    if valueAt(newEnv, oldFull) ~= nil then fail("the old name is still set once the file has run") end
    if not same(valueAt(newEnv, newFull), valueAt(oldEnv, oldFull)) then
        fail("the row would not hold what it held before; nothing was written")
    end
    local D2 = parseDoc(newText, opts)
    if not D2.byKeys[canon(newFull)] then fail("the renamed row could not be read back; nothing was written") end
    return newText
end

-- ---------------------------------------------------------------------------
-- API
-- ---------------------------------------------------------------------------

--- The file as settings: { roots, order, nodes } (see the header), or nil, err.
function M.Parse(text, opts)
    return guard(function()
        return parseDoc(text, normOpts(opts)).model
    end)
end

--- Set the value at path. value is JSON-side (§3.1). A path inside a list or
--- map row may name a key the row does not have yet; it is added. Setting the
--- value it already has returns the text unchanged.
function M.Set(text, path, value, opts)
    return guard(function()
        opts = normOpts(opts)
        local keys = parsePath(path)
        checkData(value)
        local D = parseDoc(text, opts)
        local node, deep = target(D, keys)
        local cell = codeUnder(D, keys)
        if cell then
            fail(cell.path .. " is " .. tostring(cell.reason) .. "; this change would replace it, so make it in the file")
        end
        local slot, missingAt, parentT = resolve(D, keys)
        local work = text
        if not slot then
            if not deep or missingAt ~= #keys or not parentT or parentT.t ~= "table"
                or (parentT.shape ~= "keyed" and parentT.shape ~= "empty") then
                fail("there is no setting at " .. path)
            end
            local mapNode = (#keys - 1 == #D.info[node.path].keys) and node.kind == "map"
            local sib = parentT.fields[#parentT.fields]
            local edits = {}
            insertFields(D, parentT, #parentT.fields + 1,
                { { key = keys[#keys], value = value, hint = mapNode and sib and sib.v or nil } }, edits)
            work = applyEdits(text, edits)
        elseif not deep and D.info[node.path].stmtChildren then
            fail(path .. " is filled in by separate lines of the file; set its settings one by one")
        end
        local newText = settle(work, keys, value, opts)
        if newText == text then return text end
        verify(D, newText, keys, value, opts)
        return newText
    end)
end

--- Insert a row into a list at index (1-based; nil appends). For a map, index
--- is the new row's key (text, or a whole number for a number-keyed map) and
--- the row goes at the end of the map.
function M.Insert(text, listPath, index, value, opts)
    return guard(function()
        opts = normOpts(opts)
        local keys = parsePath(listPath)
        checkData(value)
        local D = parseDoc(text, opts)
        local srows, _, anchor = statementRows(D, keys, opts)
        if srows then return stmtInsert(D, keys, srows, anchor, index, value, opts) end
        local T, cur, node, deep = rowTarget(D, keys, { list = true, strings = true, map = true })
        local isMap = T.shape == "keyed" or (T.shape == "empty" and (node.kind == "map" or type(index) == "string"))
        if isMap then
            if index == nil then fail("a map row needs a key") end
            if type(index) ~= "string" and math.type(index) ~= "integer" then fail("a key must be text or a whole number") end
            if type(index) == "string" and index == "" then fail("a key cannot be empty") end
            local key = index
            if T.shape == "keyed" and math.type(T.fields[1].key) == "integer" then
                key = math.tointeger(tonumber(index))
                if not key then fail("this map's keys are whole numbers") end
            end
            if keyOf(T, key) then fail("the key " .. tostring(key) .. " is already there") end
            if key == "__type" or key == "__int_keys" then fail("that key is reserved") end
            local sib = T.fields[#T.fields]
            local edits = {}
            insertFields(D, T, #T.fields + 1, { { key = key, value = value, hint = sib and sib.v } }, edits)
            local newText = applyEdits(text, edits)
            local expected = deepCopy(cur)
            expected[encKey(key)] = value
            if math.type(key) == "integer" then expected.__int_keys = true end
            verify(D, newText, keys, expected, opts)
            return newText
        end
        local n = #T.fields
        if index == nil then index = n + 1 end
        index = math.tointeger(index)
        if not index or index < 1 or index > n + 1 then fail("the position must be between 1 and " .. (n + 1)) end
        local sib = T.fields[math.min(index, n)]
        local edits = {}
        insertFields(D, T, index, { { value = value, hint = sib and sib.v } }, edits)
        local newText = applyEdits(text, edits)
        local expected = deepCopy(cur)
        table.insert(expected, index, value)
        verify(D, newText, keys, expected, opts)
        return newText
    end)
end

--- Remove a list row by index, or a map row by key, with its own comments.
function M.Remove(text, listPath, index, opts)
    return guard(function()
        opts = normOpts(opts)
        local keys = parsePath(listPath)
        local D = parseDoc(text, opts)
        local srows, deeper = statementRows(D, keys, opts)
        if srows then return stmtRemove(D, keys, srows, deeper, index, opts) end
        local T, cur = rowTarget(D, keys, { list = true, strings = true, map = true })
        local expected = deepCopy(cur)
        local edits = {}
        if T.shape == "keyed" then
            local f, fi = keyOf(T, index)
            if not f then fail("there is no row " .. tostring(index)) end
            removeRange(D, T, fi, fi, edits)
            expected[encKey(f.key)] = nil
        else
            local i = math.tointeger(index)
            if not i or i < 1 or i > #T.fields then fail("there is no row " .. tostring(index)) end
            removeRange(D, T, i, i, edits)
            table.remove(expected, i)
        end
        local newText = applyEdits(text, edits)
        verify(D, newText, keys, expected, opts)
        return newText
    end)
end

--- Move a list row from one position to another; its comments go with it.
function M.Move(text, listPath, from, to, opts)
    return guard(function()
        opts = normOpts(opts)
        local keys = parsePath(listPath)
        local D = parseDoc(text, opts)
        local T, cur = rowTarget(D, keys, { list = true, strings = true })
        if T.shape ~= "pos" then fail("only list rows can be moved") end
        local n = #T.fields
        from, to = math.tointeger(from), math.tointeger(to)
        if not from or not to or from < 1 or from > n or to < 1 or to > n then
            fail("positions must be between 1 and " .. n)
        end
        if from == to then return text end
        local src, fields = D.src, T.fields
        local edits = {}
        if ownLines(D, T) then
            -- Each row is a block of lines (its own comments, the row, its
            -- trailing comment). The blocks trade places; whatever sits between
            -- them (blank lines, banners) stays where it is. Rows keep their
            -- commas, except that when the file leaves its last row without one,
            -- whichever row ends up last goes without.
            local bareLast = not fields[n].sepE
            local slots = {}
            for i = 1, n do
                slots[i] = { leadingStart(D, T, i), lineEnd(D, fieldEnd(fields[i])) }
            end
            local function block(i, comma)
                local f = fields[i]
                local bs, be = slots[i][1], slots[i][2]
                local text
                if comma and not f.sepE then
                    text = src:sub(bs, f.v.e) .. "," .. src:sub(f.v.e + 1, be)
                elseif not comma and f.sepE then
                    text = src:sub(bs, f.sepS - 1) .. src:sub(f.sepE + 1, be)
                else
                    text = src:sub(bs, be)
                end
                if text:sub(-1) ~= "\n" and be < #src then text = text .. lineNl(D, f.s) end
                return text
            end
            local order = {}
            for i = 1, n do order[i] = i end
            table.insert(order, to, table.remove(order, from))
            for slot = 1, n do
                local row = order[slot]
                local comma = not (bareLast and slot == n)
                local wantText = block(row, comma)
                if row ~= slot or wantText ~= src:sub(slots[slot][1], slots[slot][2]) then
                    edits[#edits + 1] = { slots[slot][1], slots[slot][2], wantText }
                end
            end
        else
            local vals = {}
            for i, f in ipairs(fields) do vals[i] = src:sub(f.v.s, f.v.e) end
            local moved = table.remove(vals, from)
            table.insert(vals, to, moved)
            for i, f in ipairs(fields) do
                if vals[i] ~= src:sub(f.v.s, f.v.e) then edits[#edits + 1] = { f.v.s, f.v.e, vals[i] } end
            end
        end
        local newText = applyEdits(text, edits)
        local expected = deepCopy(cur)
        table.insert(expected, to, table.remove(expected, from))
        verify(D, newText, keys, expected, opts)
        return newText
    end)
end

--- Copy a row verbatim: its source text, code cells and comments inside it
--- included. A list row goes right after the one copied (key = its index);
--- a map row, or a row of a statement table, gets newKey and goes at the end.
--- Nothing new is written from outside: the text is the file's own.
function M.Duplicate(text, listPath, key, newKey, opts)
    return guard(function()
        opts = normOpts(opts)
        local keys = parsePath(listPath)
        local D = parseDoc(text, opts)
        local srows, deeper, anchor = statementRows(D, keys, opts)
        if srows then
            checkRowKey(newKey)
            local row = stmtRow(D, keys, srows, deeper, key)
            for _, r in ipairs(srows) do
                if r.key == newKey then fail("the key " .. newKey .. " is already there") end
            end
            local full = extend(keys, newKey)
            if D.byKeys[canon(full)] then fail("the key " .. newKey .. " is already there") end
            local cur, err = encodeT(D, row.st.T, "", {}, {})
            if cur == nil then fail(err) end
            local last = srows[#srows].st
            local nl = lineNl(D, last.s)
            local stmt = lineIndent(D, row.st.s) .. keysText(full) .. " = " .. D.src:sub(row.st.T.s, row.st.T.e)
            local le = lineEnd(D, last.sepE or last.T.e)
            local newText
            if D.src:sub(le, le) == "\n" then
                newText = D.src:sub(1, le) .. nl .. stmt .. nl .. D.src:sub(le + 1)
            else
                newText = D.src .. nl .. nl .. stmt .. nl
            end
            verify(D, newText, full, cur, opts)
            return newText
        end
        local T, cur = rowTarget(D, keys, { list = true, map = true })
        local edits = {}
        local expected = deepCopy(cur)
        if T.shape == "pos" then
            local i = math.tointeger(key)
            if not i or i < 1 or i > #T.fields then fail("there is no row " .. tostring(key)) end
            local f = T.fields[i]
            insertFields(D, T, i + 1, { { raw = D.src:sub(f.v.s, f.v.e), value = cur[i] } }, edits)
            table.insert(expected, i + 1, deepCopy(cur[i]))
        elseif T.shape == "keyed" then
            local f = keyOf(T, key)
            if not f then fail("there is no row " .. tostring(key)) end
            local nk = newKey
            if math.type(f.key) == "integer" then
                nk = math.tointeger(tonumber(newKey))
                if not nk then fail("this map's keys are whole numbers") end
            elseif type(nk) ~= "string" or nk == "" then
                fail("a new row needs a key")
            end
            if nk == "__type" or nk == "__int_keys" then fail("that key is reserved") end
            if keyOf(T, nk) then fail("the key " .. tostring(nk) .. " is already there") end
            insertFields(D, T, #T.fields + 1, { { key = nk, raw = D.src:sub(f.v.s, f.v.e), value = cur[encKey(f.key)] } }, edits)
            expected[encKey(nk)] = deepCopy(expected[encKey(f.key)])
        else
            fail("there is no row to copy")
        end
        local newText = applyEdits(text, edits)
        verify(D, newText, keys, expected, opts)
        return newText
    end)
end

--- Rename a map row's key. The key keeps its written style ([ "x" ] or x).
function M.RenameKey(text, mapPath, oldKey, newKey, opts)
    return guard(function()
        opts = normOpts(opts)
        local keys = parsePath(mapPath)
        local D = parseDoc(text, opts)
        local srows, deeper = statementRows(D, keys, opts)
        if srows then return stmtRename(D, keys, srows, deeper, oldKey, newKey, opts) end
        local T, cur = rowTarget(D, keys, { map = true })
        if T.shape ~= "keyed" then fail("only map rows have keys to rename") end
        local f = keyOf(T, oldKey)
        if not f then fail("there is no row " .. tostring(oldKey)) end
        local nk = newKey
        if math.type(f.key) == "integer" then
            nk = math.tointeger(tonumber(newKey))
            if not nk then fail("this map's keys are whole numbers") end
        elseif type(nk) ~= "string" then
            nk = tostring(nk)
        end
        if nk == "" then fail("a key cannot be empty") end
        if nk == "__type" or nk == "__int_keys" then fail("that key is reserved") end
        if nk == f.key then return text end
        if keyOf(T, nk) then fail("the key " .. tostring(nk) .. " is already there") end
        local kt
        if f.kind == "name" and isIdent(nk) then
            kt = nk
        elseif math.type(nk) == "integer" then
            kt = "[" .. nk .. "]"
        else
            kt = "[" .. quoteString(nk, f.keyQuote or '"') .. "]"
        end
        local newText = applyEdits(text, { { f.keyS, f.keyE, kt } })
        local expected = deepCopy(cur)
        expected[encKey(nk)] = expected[encKey(f.key)]
        expected[encKey(f.key)] = nil
        verify(D, newText, keys, expected, opts)
        return newText
    end)
end

--- Link a setting to a master role (-- poggy:role <name> on its line), or
--- unlink it with role = nil. Only the comment changes.
function M.SetRole(text, path, role, opts)
    return guard(function()
        opts = normOpts(opts)
        if role ~= nil and (type(role) ~= "string" or not role:match("^[A-Za-z0-9_%-]+$") or #role > 64) then
            fail("a role name is letters, digits, _ and - only")
        end
        local keys = parsePath(path)
        local D = parseDoc(text, opts)
        local node = D.byKeys[canon(keys)]
        if not node then fail("there is no setting at " .. path) end
        if node.kind == "group" or node.kind == "readonly" then fail(path .. " cannot be linked to a role") end
        if node.role == role then return text end
        local inf = D.info[node.path]
        local first, last = lineOf(D, inf.pos), lineOf(D, inf.endPos)
        local src = D.src
        local edits = {}
        local done = false
        for _, li in ipairs(first == last and { first } or { first, last }) do
            local words, cs, long = trailingComment(D, li)
            if words and not long and words:match(ROLE_PAT) then
                local ce = contentEnd(D, li)
                local ctext = src:sub(cs, ce)
                if role then
                    local a, b = ctext:find("poggy:role[ \t]+[A-Za-z0-9_%-]+")
                    local name = ctext:sub(a, b):match("poggy:role[ \t]+()")
                    edits[#edits + 1] = { cs + a - 1 + name - 1, cs + b - 1, role }
                elseif ctext:match("^%-%-[ \t]*poggy:role[ \t]+[A-Za-z0-9_%-]+[ \t]*$") then
                    local s0 = cs
                    while s0 > 1 and src:sub(s0 - 1, s0 - 1):match("^[ \t]$") do s0 = s0 - 1 end
                    edits[#edits + 1] = { s0, ce, "" }
                else
                    local a, b = ctext:find("[ \t]*%-%-[ \t]*poggy:role[ \t]+[A-Za-z0-9_%-]+")
                    if not a then a, b = ctext:find("[ \t]*poggy:role[ \t]+[A-Za-z0-9_%-]+") end
                    edits[#edits + 1] = { cs + a - 1, cs + b - 1, "" }
                end
                done = true
                break
            end
        end
        if not done then
            if not role then return text end
            for _, li in ipairs(first == last and { first } or { first, last }) do
                local t = lastTokenOnLine(D, li)
                local ce = contentEnd(D, li)
                local words, _, long = trailingComment(D, li)
                if t and t.e <= ce and not long then
                    local pos = ce
                    while pos >= D.lines[li] and src:sub(pos, pos):match("^[ \t]$") do pos = pos - 1 end
                    edits[#edits + 1] = { pos + 1, pos, " -- poggy:role " .. role }
                    done = true
                    break
                end
            end
            if not done then fail("there is no safe place on that line for the role marker") end
        end
        local newText = applyEdits(text, edits)
        local D2 = verify(D, newText, nil, nil, opts)
        local n2 = D2.byKeys[canon(keys)]
        if not n2 or n2.role ~= role then fail("the role marker did not land; nothing was written") end
        return newText
    end)
end

--- Lua source for a value (§3.1), indented for the given level, or nil, err.
--- Only data: a function or anything else that is not data is refused.
function M.Serialize(value, indentLevel)
    return guard(function()
        checkData(value)
        local D = { src = "", lines = { 1 }, toks = {}, nl = "\n", unit = "    " }
        local ctx = { D = D, indent = ("    "):rep(math.max(0, math.tointeger(indentLevel) or 0)), unit = "    ", nl = "\n" }
        return serValue(value, ctx, nil)
    end)
end

--- Run the file in the sandbox: the table of globals it assigned, or nil, err.
--- Vectors come back as { __type = "vec3", x, y, z }; `hash` as the number.
function M.Load(text)
    return guard(function()
        if type(text) ~= "string" then fail("the file text is missing") end
        local env, err = loadEnv(text)
        if not env then fail(err) end
        local out = {}
        for k, v in pairs(env) do out[k] = v end
        return out
    end)
end

--- Fingerprint of a file's exact bytes: FNV-1a 64 plus the length. Tells
--- whether a file changed; not a security measure.
function M.Fingerprint(text)
    text = tostring(text or "")
    local h = 0xcbf29ce484222325
    local prime = 0x100000001b3
    local n, i = #text, 1
    while i <= n do
        local j = math.min(i + 4095, n)
        local bytes = { text:byte(i, j) }
        for k = 1, #bytes do
            h = (h ~ bytes[k]) * prime
        end
        i = j + 1
    end
    return ("%016x-%x"):format(h, n)
end

--- The model flattened for the database mirror.
function M.Mirror(model)
    local out = { toggles = {}, inputs = {}, lists = {} }
    if type(model) ~= "table" or type(model.nodes) ~= "table" then return out end
    for _, path in ipairs(model.order or {}) do
        local n = model.nodes[path]
        if n then
            if n.kind == "value" then
                if n.type == "boolean" then out.toggles[path] = n.value else out.inputs[path] = n.value end
            elseif n.kind == "strings" then
                out.inputs[path] = n.value
            elseif n.kind == "list" or n.kind == "map" then
                out.lists[path] = n.value
            end
        end
    end
    return out
end

-- Exposed for tests.
M._internal = {
    parseDoc = parseDoc, resolve = resolve, parsePath = parsePath, eqEnc = eqEnc,
    loadEnv = loadEnv, valueAt = valueAt, joaat = joaat,
}

return M
