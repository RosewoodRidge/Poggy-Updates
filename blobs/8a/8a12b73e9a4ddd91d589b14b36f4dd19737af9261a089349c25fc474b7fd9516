--[[
    poggy_core — updates for Poggy resources.

        poggycore update <resource|all>                check: compare versions
        poggycore update <resource|all> stage          download changed files as <file>.poggyupdate
        poggycore update <resource|all> apply          back up and replace changed files
        poggycore update <resource> writetest          which ways this server can write files

    Flags:  force              stage/apply even when the versions say there is nothing to do
            feed=Owner/Name    another GitHub feed repository
            url=https://...    a feed served from a website instead
            repo=Owner/Name    read a source repository directly (development)
            branch=main

    Where updates come from (PoggyCoreConfig.Updates.Source):
        github   the feed publish-updates.bat pushes to Updates.Repo: one
                 index.json, the file list of every version, and the files
                 stored by SHA-256. While the repo is private, poggy_core reads it
                 through the GitHub API with `set poggy_github_token "..."` from
                 server.cfg (never sets or setr). With no token it reads the
                 public repo from raw.githubusercontent.com, which costs no API calls.
        website  the same feed served from Updates.Url. Not usable on a .xyz
                 domain: FXServer refuses every .xyz hostname.
        repo=    (flag only) a source repository read directly, for development.
    The token is only ever sent to GitHub.

    The version in fxmanifest.lua decides. A resource is updated only when the
    published version is higher than the one on disk here. Same version: nothing
    is downloaded. Local newer: nothing is touched, so an update never
    downgrades a script. `force` overrides both.

    Identity (0.13.0). The feed, its index and its file lists know a script by
    its poggy_id (fxmanifest.lua: poggy_id 'poggy_scene'); the folder on this
    server may have any name. Everything that touches the LOCAL copy (its state,
    version, files, the bridge that writes them, backups, restarts, writetest)
    uses the folder PoggyCore.Identity finds for the id: the folder that
    registered it (core.register, sent by its bridge), else the one whose
    fxmanifest.lua declares it. An id two folders claim is never updated.
    Console lines show the id, plus "(folder: X)" when the folder differs, and
    `poggycore update <name>` takes the id or a folder name.

    Within an update, only files whose content differs are written, and
    fxmanifest.lua is written last. If any other file fails, the manifest is held
    back, so the old version number stays and the update can simply be run again.

    Config files are merged, never overwritten (sv_configmerge.lua): settings
    that were added are inserted, removed ones removed, reshaped ones replaced,
    and every value the owner set is kept. What was ours and what is theirs is
    told apart with the config of the version they run. If that cannot be found,
    lines are only added. If a merge cannot be proven safe, their file is left as
    it is and the new version is saved beside it as <file>.poggyupdate. stage
    writes the merged result as <file>.poggyupdate so it can be read first.

    A script may only write inside its own resource, so poggy_core does not write
    another resource's files itself. It hands each file to that resource's bridge
    (PoggyWriteOwnFile in @poggy_core/template/poggy.lua, which every Poggy
    resource loads from poggy_core), which saves it. The resource must be
    started and load that bridge. It also cannot create folders (proven
    on a live server): a version that adds one stops before writing anything and
    lists the folders to create by hand. apply copies each replaced original
    into poggy_core/update_backups/ first. The console command never restarts
    anything: after it, run `refresh` then `restart <resource>`.

    Automatic runs (bottom of this file):
      - on start (after StartDelaySeconds) and every CheckIntervalMinutes,
        poggy_core runs `update all check`, or `update all apply` when
        Updates.AutoUpdate is on (ApplyOnStart: apply on start only).
      - after an automatic apply, RestartUpdated runs `refresh` once and restarts
        each updated resource. Updates applied while players are online wait for
        an empty server when RestartWhenEmptyOnly is set; at start they restart
        at once. `refresh` needs `add_ace resource.poggy_core command.refresh allow`;
        without it nothing is restarted and the exact lines are printed.
      - only one update run at a time (Updates.busy). A resource that fails is
        reported and the rest carry on.
      - poggy_core is updated last and never restarts itself (see restartResources).

    Development server (0.13.1). The owner's development server IS the source of
    the scripts, so an update applied there would overwrite source with escrowed
    builds. `set poggy_dev_server 1` in server.cfg (a convar, never a config key:
    config.lua ships to customers and the merge keeps an owner's values, so a
    flag in the development copy would be pulled into every release) makes every
    writing path — apply, stage, writetest, the automatic runs — refuse before
    touching anything, with one yellow line per run. check and nettest still
    work, the start-up check still lists what is newer, and the banner says
    "dev server: updates read-only". See Updates.IsDevServer.
]]

PoggyCore = PoggyCore or {}

local Updates = PoggyCore.Updates or {}
PoggyCore.Updates = Updates

local USER_AGENT = "poggy_core-updater"
local SKIP = { "%.bak", "%.poggyupdate$", "%.fxap$" }

--- .fxap files are per-customer escrow licence tokens. The updater never
--- downloads, compares, writes, backs up or deletes one, in any resource: a
--- feed entry for one is dropped, and writeFile refuses the name outright.
--- A base name ending in .fxap is the same test as a path ending in it.
local function isFxap(path)
    return tostring(path or ""):lower():sub(-5) == ".fxap"
end

-- ---------------------------------------------------------------------------
-- Console
--
-- One line per resource, an icon first, then aligned columns. Colour codes are
-- decoration only: every line still says what happened with them stripped.
-- ---------------------------------------------------------------------------

-- Same prefix as sv_util.lua; repeated so this file also runs on its own (tests).
local PREFIX = PoggyCore.PREFIX or "^5[Poggy Core]^7 "

local ICON = {
    current = "✅", updated = "✅", staged = "✅",
    available = "⬆️ ", newer = "⚠️ ", noversion = "⚠️ ",
    error = "❌", missing = "➖", skipped = "➖", unpublished = "➖",
    duplicate = "❌",
}

local function statusLine(status, resource, current, remote, verdict)
    return ("%s %-24s local v%-9s published v%-9s %s")
        :format(ICON[status] or "  ", resource, current or "?", remote or "?", verdict)
end

--- A config-merge report line, coloured by its leading mark.
local function mergeLine(line)
    local mark = line:sub(1, 1)
    if mark == "+" then return "^2" .. line .. "^7" end
    if mark == "-" then return "^1" .. line .. "^7" end
    if mark == "~" then return "^3" .. line .. "^7" end
    if mark == "!" or line:find("^merge failed") then return "^1" .. line .. "^7" end
    return "^9" .. line .. "^7"
end

--- "3 failed" in red when it is not zero.
local function bad(n, word)
    return n > 0 and ("^1%d %s^7"):format(n, word) or ("%d %s"):format(n, word)
end

local function cfg()
    return PoggyCoreConfig.Updates or {}
end

local function token()
    local t = GetConvar("poggy_github_token", "")
    return t ~= "" and t or nil
end

--- Is this server marked as a development server (`set poggy_dev_server 1` in
--- server.cfg)? 1 or "true" both count. Read on every run, never cached, so
--- `set` from the console takes effect at once. Both natives are called under
--- pcall: the harnesses that run this file on its own do not define them all.
function Updates.IsDevServer()
    local okInt, n = pcall(GetConvarInt, "poggy_dev_server", 0)
    if okInt and tonumber(n) == 1 then return true end
    local okStr, s = pcall(GetConvar, "poggy_dev_server", "")
    if okStr and type(s) == "string" then
        s = s:lower():match("^%s*(.-)%s*$")
        return s == "1" or s == "true"
    end
    return false
end

--- The one line a refused write prints. Once per run, through that run's say().
local DEV_LINE = "^3⚠️  this server is marked as a development server (set poggy_dev_server 1 in server.cfg):"
    .. " updates are reported but never written.^7"

-- ---------------------------------------------------------------------------
-- Identity: feed id -> the folder on this server (sv_identity.lua)
--
-- Without sv_identity.lua loaded (a test that runs this file on its own) an id
-- is its own folder, which is how every version before 0.13.0 behaved.
-- ---------------------------------------------------------------------------

--- folder, problem, claimants. problem: nil | "missing" | "duplicate".
local function localFolder(id)
    local I = PoggyCore.Identity
    if not I then return id end
    return I.Locate(id)
end

--- The poggy_id of a local folder.
local function idOf(folder)
    local I = PoggyCore.Identity
    return I and I.Id(folder) or folder
end

--- "poggy_scene", or "poggy_scene (folder: my_scene)" when they differ.
local function displayName(id, folder)
    if folder and folder ~= id then return ("%s (folder: %s)"):format(tostring(id), tostring(folder)) end
    return tostring(id)
end

local function shownFolder(folder)
    return displayName(idOf(folder), folder)
end

-- Duplicate ids already reported, so a periodic run names each one once.
local duplicateSeen = {}

-- ---------------------------------------------------------------------------
-- HTTP
-- ---------------------------------------------------------------------------

-- curl's error text for the most recent request that got no HTTP response at all.
local lastHttpError

--- GET a URL. Yields; call from a thread. The GitHub token is attached only
--- when withToken is set, which only the GitHub source does.
local function get(url, accept, withToken)
    local headers = {
        ["User-Agent"] = USER_AGENT,
        ["Accept"] = accept or "*/*",
    }
    local t = withToken and token()
    if t then headers["Authorization"] = "Bearer " .. t end

    local p = promise.new()
    -- The fourth argument is FXServer's own error text (curl's), set when no HTTP
    -- response came back at all. Without it "no response" says nothing useful.
    PerformHttpRequest(url, function(status, body, respHeaders, errorData)
        p:resolve({ status = status, body = body, headers = respHeaders or {}, err = errorData })
    end, "GET", "", headers)
    local res = Citizen.Await(p)
    lastHttpError = (res.status == 0 or res.status == nil) and res.err or nil
    return res
end

--- Percent-encode a path, keeping the slashes.
local function encodePath(path)
    return (path:gsub("[^%w%-%._~/]", function(c)
        return ("%%%02X"):format(c:byte())
    end))
end

local function explain(status, what)
    if status == 404 then
        return what .. ": 404. " .. (token()
            and "The token cannot see this repository, the branch name is wrong, or the path does not exist."
            or  "The repository is private (set poggy_github_token), the branch name is wrong, or the path does not exist.")
    elseif status == 401 then
        return what .. ": 401. poggy_github_token is set but GitHub rejected it (expired or mistyped)."
    elseif status == 403 or status == 429 then
        return what .. (": %d. Rate limited (60 calls an hour without a token, 5000 with one), or the token lacks Contents: read."):format(status)
    elseif status == 0 or status == nil then
        return what .. ": no response. The server could not reach GitHub."
            .. (lastHttpError and (" FXServer says: " .. tostring(lastHttpError)) or "")
    end
    return what .. (": HTTP %s."):format(tostring(status))
end

local function explainWeb(status, what)
    if status == 404 then
        return what .. ": 404, not found. Has it been published with publish-updates.bat?"
    elseif status == 0 or status == nil then
        return what .. ": no response. The server could not reach the update site."
            .. (lastHttpError and (" FXServer says: " .. tostring(lastHttpError)) or "")
    end
    return what .. (": HTTP %s."):format(tostring(status))
end

-- ---------------------------------------------------------------------------
-- Writing
--
-- Proven on this server by `writetest`: SaveResourceFile works only into the
-- resource that calls it, and io.open cannot write anywhere. So poggy_core
-- writes its own files directly and asks every other resource's bridge to
-- write its own, then reads each file back to confirm what landed on disk.
-- ---------------------------------------------------------------------------

--- Returns true, method  or  false, reason.
local function writeFile(resource, rel, data)
    -- The last lock on a development server. Run() and Auto() already turn every
    -- writing mode into a check there; this makes sure nothing below this line can
    -- run even if a caller gets that wrong. Its folders are the scripts' source.
    if Updates.IsDevServer() then
        return false, "refused: development server (set poggy_dev_server 1 in server.cfg); update files are never written here"
    end
    if isFxap(rel) then
        return false, "refused: .fxap files are per-customer licence tokens and are never written"
    end

    local own = GetCurrentResourceName()
    local how

    if resource == own then
        if not SaveResourceFile(own, rel, data, #data) then
            return false, "SaveResourceFile returned false (does the folder exist?)"
        end
        how = "SaveResourceFile"
    else
        local state = GetResourceState(resource)
        if state ~= "started" then
            return false, ("'%s' is %s; it must be started to save its own files"):format(resource, state)
        end
        local okCall, res = pcall(function()
            return exports[resource]:PoggyWriteOwnFile(rel, data)
        end)
        if not okCall then
            return false, ("%s has no PoggyWriteOwnFile. It must load the bridge from poggy_core"
                .. " (shared_script '@poggy_core/template/poggy.lua' in its fxmanifest.lua) and be restarted;"
                .. " if it already does, update poggy_core. (%s)"):format(resource, tostring(res))
        end
        if type(res) ~= "table" or not res.ok then
            return false, tostring(type(res) == "table" and res.err or res)
        end
        how = "own bridge"
    end

    local back = LoadResourceFile(resource, rel)
    if not back or #back ~= #data then
        return false, ("saved, but reading it back gave %s byte(s), expected %d")
            :format(back and tostring(#back) or "no", #data)
    end
    return true, how
end

-- ---------------------------------------------------------------------------
-- Versions
-- ---------------------------------------------------------------------------

local function manifestVersion(text)
    return text and (text:match("\n%s*version%s+[\"']([^\"']+)[\"']")
        or text:match("^%s*version%s+[\"']([^\"']+)[\"']"))
end

--- The version in the fxmanifest.lua on disk. Read from the file rather than
--- GetResourceMetadata, which keeps the old value until `refresh`.
local function localVersion(resource)
    return manifestVersion(LoadResourceFile(resource, "fxmanifest.lua"))
        or GetResourceMetadata(resource, "version", 0)
end

--- -1, 0 or 1. Numeric parts are compared as numbers, so 1.10.0 > 1.9.0.
local function compare(a, b)
    local pa, pb = {}, {}
    for n in tostring(a):gmatch("%d+") do pa[#pa + 1] = tonumber(n) end
    for n in tostring(b):gmatch("%d+") do pb[#pb + 1] = tonumber(n) end
    for i = 1, math.max(#pa, #pb) do
        local x, y = pa[i] or 0, pb[i] or 0
        if x ~= y then return x < y and -1 or 1 end
    end
    return 0
end

--- The `poggy_core_min '0.11.0'` a manifest declares, or nil.
local function manifestCoreMin(text)
    return text and (text:match("\n%s*poggy_core_min%s+[\"']([^\"']+)[\"']")
        or text:match("^%s*poggy_core_min%s+[\"']([^\"']+)[\"']"))
end

--- Does the version of this resource now on disk need a newer poggy_core than
--- the one running? Returns (min, running) when it does, nil otherwise.
---
--- PoggyCore.VERSION is the version this poggy_core started as, so it stays
--- the old one after an update of poggy_core itself has been installed: that is
--- the point. Restarting such a resource would load a bridge that refuses every
--- call against the running core (template/poggy.lua), so it is left alone.
local function needsNewerCore(resource)
    local running = PoggyCore.VERSION
    if not running then return nil end
    local min = manifestCoreMin(LoadResourceFile(resource, "fxmanifest.lua"))
    if min and compare(running, min) < 0 then return min, running end
    return nil
end

--- Translation and locale files: translations*.lua, locale*.lua, lang*.lua.
--- Merged like configs, and with their ["key"] = "text" entries merged one by
--- one, so strings added in an update reach the owner's copy.
local function isTranslation(rel)
    local base = (rel:lower():gsub("\\", "/"):match("([^/]+)$")) or ""
    return base:sub(-4) == ".lua"
        and (base:find("^translation") or base:find("^locale") or base:find("^lang")) ~= nil
end

--- Files a server owner edits: config*.lua anywhere, anything under config/
--- (except a future config/default*.lua), and translation and locale files.
--- These are merged, never overwritten.
local function isConfig(rel)
    local lower = rel:lower():gsub("\\", "/")
    if lower:find("^config/default") or lower:find("/config/default") then return false end
    if lower:find("^config/") or lower:find("/config/") then return true end
    if isTranslation(rel) then return true end
    local base = lower:match("([^/]+)$") or lower
    return base:find("^config") ~= nil
end

-- ---------------------------------------------------------------------------
-- Source: the feed written by publish-updates.bat
--
--   index.json                    { resources = { name = { version, files } } }
--   <resource>/<version>.json     the file list of every version ever published
--   blobs/<ab>/<sha256>           each file, stored once by its hash
--
-- The same layout is read from a website or from a GitHub repository; only how a
-- path is fetched differs, so that part is passed in.
-- ---------------------------------------------------------------------------

--- fetchPath(path, isJson) returns a get() result. explainFn(status, what) words a failure.
local function feedSource(label, fetchPath, explainFn)
    local src = { label = label }
    local index

    local function fetchJson(path)
        local r = fetchPath(path, true)
        if r.status ~= 200 then return nil, explainFn(r.status, path), r.status end
        local ok, data = pcall(json.decode, r.body or "")
        if not ok or type(data) ~= "table" then return nil, path .. " could not be read" end
        return data
    end

    local function fileList(entry)
        local files = entry and entry.files or {}
        if files.path then files = { files } end   -- a one-file list written as an object
        return files
    end

    local function blobPath(sha)
        return ("blobs/%s/%s"):format(sha:sub(1, 2), sha)
    end

    function src.load()
        local data, err, status = fetchJson("index.json")
        if not data then return false, err, status end
        if type(data.resources) ~= "table" then return false, "index.json lists no resources" end
        index = data
        return true
    end

    function src.names()
        local names = {}
        for name in pairs(index.resources) do names[#names + 1] = name end
        table.sort(names)
        return names
    end

    function src.find(resource)
        local entry = index.resources[resource]
        if not entry then return nil, "not published in " .. label, "unpublished" end
        local files, ignoredFxap = {}, 0
        for _, f in ipairs(fileList(entry)) do
            if isFxap(f.path) then
                ignoredFxap = ignoredFxap + 1   -- never fetched, never compared
            elseif type(f.sha256) == "string" and #f.sha256 == 64 then
                files[#files + 1] = { rel = f.path, size = tonumber(f.size) or -1, blob = blobPath(f.sha256) }
            end
        end
        table.sort(files, function(a, b) return a.rel < b.rel end)
        return { version = entry.version, files = files, ignoredFxap = ignoredFxap }
    end

    function src.fetch(file)
        local r = fetchPath(file.blob)
        if r.status ~= 200 then return r.status, nil, explainFn(r.status, file.rel) end
        return 200, r.body
    end

    local lists = {}
    function src.base(resource, version, rel)
        local key = resource .. "@" .. tostring(version)
        if lists[key] == nil then
            local data, err = fetchJson(resource .. "/" .. tostring(version) .. ".json")
            lists[key] = data or false
            if not data then return nil, err end
        end
        if not lists[key] then return nil, "the v" .. tostring(version) .. " file list is not published" end
        for _, f in ipairs(fileList(lists[key])) do
            if f.path == rel then
                local r = fetchPath(blobPath(f.sha256))
                if r.status == 200 then return r.body end
                return nil, explainFn(r.status, "the v" .. tostring(version) .. " copy of " .. rel)
            end
        end
        return nil, rel .. " did not exist in v" .. tostring(version)
    end

    return src
end

--- The feed on a website. Not usable on .xyz: FXServer refuses those hostnames.
local function websiteSource(url)
    url = url:gsub("/+$", "")
    return feedSource(url, function(path, isJson)
        local u = url .. "/" .. encodePath(path)
        -- The query string keeps a CDN from answering with an old copy.
        if isJson then u = u .. "?t=" .. os.time() end
        return get(u, isJson and "application/json" or nil)
    end, explainWeb)
end

-- ---------------------------------------------------------------------------
-- Source: a GitHub repository (development)
-- ---------------------------------------------------------------------------

local function contentsUrl(repo, ref, path)
    return ("https://api.github.com/repos/%s/contents/%s?ref=%s"):format(repo, encodePath(path), ref)
end
local RAW = "application/vnd.github.raw"

local function skipped(path)
    for _, pattern in ipairs(SKIP) do
        if path:find(pattern) then return true end
    end
    return false
end

local function fetchTree(repo, branch)
    local url = ("https://api.github.com/repos/%s/git/trees/%s?recursive=1"):format(repo, branch)
    local res = get(url, "application/vnd.github+json", true)
    if res.status ~= 200 then
        local msg = explain(res.status, "repository listing for " .. repo .. "@" .. branch)
        if token() then
            local who = get("https://api.github.com/user", "application/vnd.github+json", true)
            local okWho, user = pcall(json.decode, who.body or "")
            local login = (who.status == 200 and okWho and type(user) == "table") and user.login or nil
            local meta = get(("https://api.github.com/repos/%s"):format(repo), "application/vnd.github+json", true)
            local okMeta, info = pcall(json.decode, meta.body or "")
            msg = msg .. ("\n    token belongs to: %s"):format(login or ("unknown (HTTP " .. tostring(who.status) .. ")"))
            if meta.status == 200 and okMeta and type(info) == "table" then
                msg = msg .. ("\n    token CAN see %s (private=%s, default branch=%s) but cannot list its files:"
                    .. " give the token Repository permissions > Contents: Read-only%s.")
                    :format(repo, tostring(info.private), tostring(info.default_branch),
                        info.default_branch ~= branch and (", or use branch=" .. tostring(info.default_branch)) or "")
            else
                msg = msg .. ("\n    token CANNOT see %s at all (HTTP %s). The token's Resource owner must be %s,"
                    .. " and Repository access must include it.")
                    :format(repo, tostring(meta.status), repo:match("^([^/]+)") or "?")
            end
        end
        return nil, msg
    end
    local ok, tree = pcall(json.decode, res.body or "")
    if not ok or type(tree) ~= "table" or type(tree.tree) ~= "table" then
        return nil, "GitHub answered, but the listing could not be read."
    end
    return tree.tree, tree.truncated == true
end

local function locate(entries, resource)
    local prefix
    for _, entry in ipairs(entries) do
        local p = entry.path
        if entry.type == "blob" and (p == resource .. "/fxmanifest.lua"
            or p:sub(-(#resource + 16)) == "/" .. resource .. "/fxmanifest.lua") then
            prefix = p:sub(1, #p - #"fxmanifest.lua")
            break
        end
    end
    if not prefix then
        for _, entry in ipairs(entries) do
            if entry.type == "blob" and entry.path == "fxmanifest.lua" then prefix = "" break end
        end
    end
    if not prefix then
        return nil, ("no folder named '%s' with an fxmanifest.lua in the repository."):format(resource), "unpublished"
    end
    local files = {}
    for _, entry in ipairs(entries) do
        if entry.type == "blob" and entry.path:sub(1, #prefix) == prefix then
            local rel = entry.path:sub(#prefix + 1)
            if rel ~= "" and not skipped(rel) and not isFxap(rel) then
                files[#files + 1] = { path = entry.path, rel = rel, size = entry.size or -1 }
            end
        end
    end
    table.sort(files, function(a, b) return a.rel < b.rel end)
    return { prefix = prefix, files = files }
end

local function githubSource(repo, branch)
    local src = { label = "GitHub " .. repo .. "@" .. branch }
    local entries

    function src.load()
        local e, extra = fetchTree(repo, branch)
        if not e then return false, extra end
        entries = e
        src.truncated = extra == true
        return true
    end

    function src.names()
        local seen, names = {}, {}
        for _, entry in ipairs(entries) do
            if entry.type == "blob" then
                local folder = entry.path:match("([^/]+)/fxmanifest%.lua$")
                if folder and not seen[folder] then
                    seen[folder] = true
                    names[#names + 1] = folder
                end
            end
        end
        table.sort(names)
        return names
    end

    function src.find(resource)
        local found, err, why = locate(entries, resource)
        if not found then return nil, err, why end
        local manifest = get(contentsUrl(repo, branch, found.prefix .. "fxmanifest.lua"), RAW, true)
        if manifest.status ~= 200 then return nil, explain(manifest.status, "remote fxmanifest.lua") end
        for _, f in ipairs(found.files) do
            if f.rel == "fxmanifest.lua" then f.body = manifest.body end
        end
        return { version = manifestVersion(manifest.body), files = found.files, prefix = found.prefix }
    end

    function src.fetch(file)
        if file.body then return 200, file.body end
        local r = get(contentsUrl(repo, branch, file.path), RAW, true)
        if r.status ~= 200 then return r.status, nil, explain(r.status, file.rel) end
        return 200, r.body
    end

    -- The newest commit whose fxmanifest.lua carries the version a server runs.
    local shas = {}
    function src.base(resource, version, rel, info)
        if shas[resource] == nil then
            shas[resource] = false
            local url = ("https://api.github.com/repos/%s/commits?path=%s&sha=%s&per_page=30")
                :format(repo, encodePath(info.prefix .. "fxmanifest.lua"), branch)
            local res = get(url, "application/vnd.github+json", true)
            if res.status ~= 200 then return nil, explain(res.status, "commit history") end
            local ok, commits = pcall(json.decode, res.body or "")
            if ok and type(commits) == "table" then
                local looked = 0
                for _, c in ipairs(commits) do
                    if looked >= 20 then break end
                    if type(c) == "table" and c.sha then
                        looked = looked + 1
                        local m = get(contentsUrl(repo, c.sha, info.prefix .. "fxmanifest.lua"), RAW, true)
                        if m.status == 200 and manifestVersion(m.body) == version then
                            shas[resource] = c.sha
                            break
                        end
                    end
                end
            end
        end
        if not shas[resource] then
            return nil, "no commit found with version " .. tostring(version)
        end
        local r = get(contentsUrl(repo, shas[resource], info.prefix .. rel), RAW, true)
        if r.status == 200 then return r.body end
        return nil, rel .. " did not exist in v" .. tostring(version)
    end

    return src
end

--- When a token is set but GitHub answers 403/404, say which of the token's
--- settings is wrong. Never prints the token.
local function tokenDiagnosis(repo, branch)
    local who = get("https://api.github.com/user", "application/vnd.github+json", true)
    local okWho, user = pcall(json.decode, who.body or "")
    local login = (who.status == 200 and okWho and type(user) == "table") and user.login or nil
    local meta = get(("https://api.github.com/repos/%s"):format(repo), "application/vnd.github+json", true)
    local okMeta, info = pcall(json.decode, meta.body or "")
    local msg = ("\n    token belongs to: %s"):format(login or ("unknown (HTTP " .. tostring(who.status) .. ")"))
    if meta.status == 200 and okMeta and type(info) == "table" then
        return msg .. ("\n    token CAN see %s (private=%s, default branch=%s) but cannot read its files:"
            .. " give the token Repository permissions > Contents: Read-only%s.")
            :format(repo, tostring(info.private), tostring(info.default_branch),
                info.default_branch ~= branch and (", or use branch=" .. tostring(info.default_branch)) or "")
    end
    return msg .. ("\n    token CANNOT see %s at all (HTTP %s). The token's Resource owner must be %s,"
        .. " and its Repository access must include %s.")
        :format(repo, tostring(meta.status), repo:match("^([^/]+)") or "?", repo)
end

--- The feed in a GitHub repository. With a token (private repo) every read goes
--- through the contents API; without one it reads the public repo from
--- raw.githubusercontent.com, which does not count against the API limit.
local function githubFeedSource(repo, branch)
    local withToken = token() ~= nil
    local src = feedSource(("GitHub %s@%s"):format(repo, branch), function(path)
        if withToken then
            return get(contentsUrl(repo, branch, path), RAW, true)
        end
        return get(("https://raw.githubusercontent.com/%s/%s/%s"):format(repo, branch, encodePath(path)))
    end, explain)

    local load = src.load
    function src.load()
        local ok, err, status = load()
        if not ok and withToken and (status == 403 or status == 404) then
            err = tostring(err) .. tokenDiagnosis(repo, branch)
        end
        return ok, err
    end
    return src
end

-- ---------------------------------------------------------------------------
-- One resource
--
-- Returns a status: "available" "current" "newer" "noversion" "updated"
-- "staged" "missing" "unpublished" "error".
-- ---------------------------------------------------------------------------

local function runOne(resource, mode, opts, say, source, quiet, run)
    -- `resource` is the feed id: the feed, index and file lists use it. The copy
    -- on this server is `localRes`, the folder that id lives in here.
    local localRes, problem, claimants = localFolder(resource)
    if problem == "duplicate" then
        local key = resource .. "|" .. table.concat(claimants or {}, ",")
        local fresh = not duplicateSeen[key]
        duplicateSeen[key] = true
        if fresh and run then run.newDuplicates = (run.newDuplicates or 0) + 1 end
        -- A periodic run names it once; a start run or a command every time.
        if fresh or not opts.onlyChanges then
            say(("%s %-24s ^1not updated: poggy_id '%s' is declared by %s. poggy_core never guesses between them:"
                .. " remove or stop all but one, then run the update again.^7")
                :format(ICON.duplicate, resource, resource, table.concat(claimants or {}, " and ")))
        end
        return "duplicate"
    end

    local state = localRes and GetResourceState(localRes) or "missing"
    if state == "missing" or state == "unknown" then
        if not quiet then say(("%s %-24s ^9not a resource on this server^7"):format(ICON.missing, resource)) end
        return "missing"
    end
    local shown = displayName(resource, localRes)

    local info, err, why = source.find(resource)
    if not info then
        if why == "unpublished" and quiet then return "unpublished" end
        say(("%s %-24s ^1%s^7"):format(ICON.error, shown, tostring(err)))
        return why == "unpublished" and "unpublished" or "error"
    end

    local remote = info.version
    local current = localVersion(localRes)
    local order = (remote and current) and compare(current, remote) or nil

    local status, verdict
    if not remote then
        status, verdict = "noversion", "^3no version line in the published fxmanifest.lua^7"
    elseif not current then
        status, verdict = "noversion", "^3no version line in the local fxmanifest.lua^7"
    elseif order < 0 then
        status, verdict = "available", "^3update available^7"
    elseif order == 0 then
        status, verdict = "current", "^2up to date^7"
    else
        status, verdict = "newer", "^3newer locally than published^7"
    end

    -- Periodic runs (opts.onlyChanges) leave out what is already up to date.
    if not (quiet and opts.onlyChanges and status == "current") then
        say(statusLine(status, shown, current, remote, verdict))
    end

    if mode == "check" then
        if not quiet and status == "available" then
            say(("   ^9next: poggycore update %s apply, then refresh and restart %s^7"):format(resource, localRes))
        end
        return status
    end

    if status ~= "available" then
        if not opts.force then
            if not quiet then
                say(("   ^9nothing to %s. Add 'force' to %s anyway.^7"):format(mode, mode))
            end
            return status
        end
        say("   ^3force: continuing anyway^7")
    end

    -- Only a started resource can save files: its own bridge does the writing.
    if localRes ~= GetCurrentResourceName() and state ~= "started" then
        say(("%s %-24s ^3skipped: it is %s, and only a started resource can receive files."
            .. " Start it, then run the update again.^7"):format(ICON.skipped, shown, tostring(state)))
        return "skipped"
    end

    -- .fxap licence tokens: the sources already leave them out, and this is the
    -- second line of defence. Never downloaded, compared, written or backed up.
    local ignoredFxap = tonumber(info.ignoredFxap) or 0

    -- fxmanifest.lua last: if anything before it fails, the old version number
    -- stays on disk and the same update can be run again.
    local files, manifestFile = {}, nil
    for _, f in ipairs(info.files) do
        if isFxap(f.rel) then
            ignoredFxap = ignoredFxap + 1
        elseif f.rel == "fxmanifest.lua" then
            manifestFile = f
        else
            files[#files + 1] = f
        end
    end
    if ignoredFxap > 0 then
        say("   ^9ignored .fxap in feed (licence tokens are never updated)^7")
    end
    if manifestFile then files[#files + 1] = manifestFile end

    local own = GetCurrentResourceName()
    local stamp = os.date("%Y%m%d-%H%M%S")
    local wrote, unchanged, failed, mismatch, keptConfigs, mergedConfigs = 0, 0, 0, 0, 0, 0
    local baseNoted = false

    -- A server script cannot create folders (writetest on a live server,
    -- 13 September 2026). Before anything else is written, try the first new file
    -- in every folder this update writes to that has no published file here yet.
    -- If a folder is missing, stop with the list: nothing is half-installed, the
    -- version stays as it was, and the owner knows exactly what to create.
    local known, prewritten, missingFolders = {}, {}, {}
    local function markKnown(rel)
        local folder = rel:match("^(.*)/[^/]+$")
        while folder do
            known[folder] = true
            folder = folder:match("^(.*)/[^/]+$")
        end
    end
    for _, f in ipairs(files) do
        if LoadResourceFile(localRes, f.rel) then markKnown(f.rel) end
    end
    for _, f in ipairs(files) do
        local folder = f.rel:match("^(.*)/[^/]+$")
        if folder and not known[folder] and f ~= manifestFile then
            local probeStatus, probeBody = source.fetch(f)
            if probeStatus == 200 and probeBody and (f.size < 0 or #probeBody == f.size) then
                local target = mode == "stage" and (f.rel .. ".poggyupdate") or f.rel
                if writeFile(localRes, target, probeBody) then
                    markKnown(f.rel)
                    prewritten[f.rel] = true
                    wrote = wrote + 1
                    say(("   ^2+ added^7 ^9%s (%d bytes)^7"):format(target, #probeBody))
                else
                    missingFolders[#missingFolders + 1] = folder
                    known[folder] = true
                end
            end
        end
    end
    if #missingFolders > 0 then
        say(("%s %-24s ^1stopped: this version adds folder(s) this server does not have, and a server script cannot create folders:^7")
            :format(ICON.error, shown))
        for _, folder in ipairs(missingFolders) do
            say("      ^3" .. localRes .. "/" .. folder .. "^7")
        end
        say("   ^9create them by hand (empty folders are enough), then run the update again. "
            .. (wrote > 0 and "Only brand-new files were added so far; " or "Nothing was changed; ")
            .. "the version is still v" .. tostring(current) .. ".^7")
        return "error"
    end

    for _, f in ipairs(files) do
        if prewritten[f.rel] then goto continue end
        if f.rel == "fxmanifest.lua" and mode == "apply" and (failed > 0 or mismatch > 0) then
            say("   ^3held back fxmanifest.lua^7 ^9— earlier files failed, so the version is unchanged and this update can be retried^7")
            break
        end

        local httpStatus, body, fetchErr = source.fetch(f)
        body = body or ""

        if httpStatus ~= 200 then
            failed = failed + 1
            say("   ^1download failed " .. f.rel .. " — " .. tostring(fetchErr or httpStatus) .. "^7")
        elseif f.size >= 0 and #body ~= f.size then
            mismatch = mismatch + 1
            say(("   ^1size mismatch %s — published as %d bytes, received %d; not written^7")
                :format(f.rel, f.size, #body))
        else
            local existing = LoadResourceFile(localRes, f.rel)
            if existing == body then
                unchanged = unchanged + 1
            else
                local target, proceed, content = f.rel, true, body
                local keepConfig, mergedConfig = false, false

                if existing ~= nil and isConfig(f.rel) then
                    local baseText, baseWhy = source.base(resource, current, f.rel, info)
                    if not baseText and not baseNoted then
                        baseNoted = true
                        say("   ^3note^7 ^9the config of v" .. tostring(current) .. " is not available (" .. tostring(baseWhy)
                            .. "), so config lines will only be added, never removed^7")
                    end

                    local merged, report
                    if PoggyCore.ConfigMerge then
                        merged, report = PoggyCore.ConfigMerge.Merge(baseText, body, existing,
                            { dictKeys = isTranslation(f.rel) })
                    else
                        report = { "merge unavailable: sv_configmerge.lua is not loaded" }
                    end
                    for _, line in ipairs(report or {}) do
                        say("      ^9" .. f.rel .. "^7  " .. mergeLine(line))
                    end

                    if merged == nil then
                        keepConfig = true
                    elseif merged == existing then
                        proceed = false
                        unchanged = unchanged + 1
                        say("   ^2config current^7 ^9" .. f.rel .. " already has every setting this version needs^7")
                    else
                        mergedConfig = true
                        content = merged
                    end
                end

                if not proceed then
                    -- nothing to write
                elseif mode == "stage" or keepConfig then
                    target = f.rel .. ".poggyupdate"
                elseif existing then
                    -- Backups never go beside the file. poggy_core keeps them in
                    -- its own update_backups folder, one flat file per original.
                    local bak = ("update_backups/%s__%s__%s"):format(localRes, stamp, (f.rel:gsub("[/\\]", "__")))
                    local bakOk, bakWhy = writeFile(own, bak, existing)
                    if not bakOk then
                        failed = failed + 1
                        proceed = false
                        say("   ^1backup failed " .. f.rel .. " — left untouched: " .. tostring(bakWhy) .. "^7")
                    end
                end

                if proceed then
                    local okWrite, how = writeFile(localRes, target, content)
                    if okWrite and keepConfig then
                        keptConfigs = keptConfigs + 1
                        say(("   ^3config kept^7 %s is yours and was not changed ^9— the new version is beside it as %s^7")
                            :format(f.rel, target))
                    elseif okWrite and mergedConfig then
                        mergedConfigs = mergedConfigs + 1
                        wrote = wrote + 1
                        say(("   ^2merged^7 %s%s ^9— your values kept (%s)^7"):format(
                            target, mode == "stage" and " (preview)" or "", how))
                    elseif okWrite and existing then
                        wrote = wrote + 1
                        say(("   ^9wrote %s (%d bytes, %s)^7"):format(target, #content, how))
                    elseif okWrite then
                        wrote = wrote + 1
                        say(("   ^2+ added^7 ^9%s (%d bytes, %s)^7"):format(target, #content, how))
                    else
                        failed = failed + 1
                        local folder = target:match("^(.*)/[^/]+$")
                        local hint = ""
                        if folder and not LoadResourceFile(localRes, folder .. "/") and tostring(how):find("folder") then
                            hint = (" — this version adds the folder '%s', and a server script cannot create folders."
                                .. " Create %s/%s by hand, then run the update again"):format(folder, localRes, folder)
                        end
                        say(("   ^1write failed %s — %s%s^7"):format(target, tostring(how), hint))
                    end
                end
            end
        end
        ::continue::
    end

    local broken = failed > 0 or mismatch > 0
    say(("%s %-24s %s: %d written (%d config merge(s)), %d unchanged, %d config file(s) kept, %s, %s")
        :format(broken and ICON.error or ICON.updated, shown,
            broken and ("^1" .. mode .. " incomplete^7") or ("^2" .. mode .. " done^7"),
            wrote, mergedConfigs, unchanged, keptConfigs,
            bad(failed, "failed"), bad(mismatch, "size mismatch(es)")))

    if broken then return "error" end
    if mode == "stage" then
        if wrote > 0 then
            say(("   ^9%d staged file(s) sit beside the originals as *.poggyupdate; delete them when done.^7"):format(wrote))
        end
        return "staged"
    end
    if wrote > 0 then
        say(("   ^9backups: poggy_core/update_backups/%s__%s__*^7"):format(localRes, stamp))
        if not opts.auto then
            local min, running = needsNewerCore(localRes)
            if min then
                say(("   ^3needs poggy_core %s^7 ^9(v%s is running): restart poggy_core first"
                    .. " (restart the server, or run: refresh, ensure poggy_core), then restart %s^7")
                    :format(min, running, localRes))
            else
                say(("   ^9next: refresh, then restart %s^7"):format(localRes))
            end
        end
        return "updated", localRes
    end
    return "current"
end

-- ---------------------------------------------------------------------------
-- The command
-- ---------------------------------------------------------------------------

--- The body of a run. Returns { counts = {status = n}, updated = {folders},
--- newDuplicates = n }, or nil plus a reason when nothing could be checked.
--- updated holds local FOLDER names: they are what gets restarted.
local function runAll(resource, mode, opts, say)
    local c = cfg()

    if not resource or resource == "" then
        say("usage: poggycore update <resource|all> [check|stage|apply] [force] [feed=Owner/Name] [url=...] [repo=Owner/Name] [branch=main]")
        return nil, "usage"
    end

    -- Which folder holds which id is looked up afresh for every run: scripts
    -- register and stop while the server runs.
    local I = PoggyCore.Identity
    if I then I.Refresh() end
    local all = resource == "all"
    local source
    local kind = opts.repo and "repo" or opts.url and "website" or opts.feed and "github" or (c.Source or "github")
    local what = ("update %s %s"):format(mode, resource)

    if kind == "repo" then
        local repo = opts.repo or (not all and (c.Repos or {})[resource]) or c.SourceRepo
        if not repo then
            say("^1❌ no source repository set. Pass repo=Owner/Name.^7")
            return nil, "no repo"
        end
        source = githubSource(repo, opts.branch or c.SourceBranch or "main")
        say(("^9%s · source repository %s (token: %s)^7"):format(
            what, source.label, token() and "set" or "none"))
    elseif kind == "website" then
        local url = opts.url or c.Url
        if not url or url == "" then
            say("^1❌ no update address set. Add PoggyCoreConfig.Updates.Url.^7")
            return nil, "no url"
        end
        source = websiteSource(url)
        say(("^9%s · %s^7"):format(what, source.label))
    else
        local repo = opts.feed or c.Repo
        if not repo or repo == "" then
            say("^1❌ no update repository set. Add PoggyCoreConfig.Updates.Repo.^7")
            return nil, "no repo"
        end
        source = githubFeedSource(repo, opts.branch or c.Branch or "main")
        say(("^9%s · %s (%s)^7"):format(what, source.label,
            token() and "token set" or "no token: public repo"))
    end

    local ok, err = source.load()
    if not ok then
        say("^1❌ " .. tostring(err) .. "^7")
        return nil, tostring(err)
    end
    if source.truncated then
        say("^3⚠️  GitHub truncated the listing; very large repositories may miss files.^7")
    end

    -- A name typed in a command may be the id or a folder name. A published name
    -- is the id; anything else is read as a folder and becomes its id.
    if not all and I then
        local published = false
        for _, name in ipairs(source.names()) do
            if name == resource then published = true break end
        end
        if not published then resource = I.Resolve(resource) end
    end

    local run = { newDuplicates = 0 }

    if not all then
        local status, folder = runOne(resource, mode, opts, say, source, false, run)
        return { counts = { [status] = 1 }, updated = status == "updated" and { folder } or {},
            newDuplicates = run.newDuplicates }
    end

    -- poggy_core last, so the updater is not replaced while it is still working.
    -- The feed names it by its id.
    local names, selfName, hasSelf = {}, idOf(GetCurrentResourceName()), false
    for _, name in ipairs(source.names()) do
        if name == selfName then hasSelf = true else names[#names + 1] = name end
    end
    if hasSelf then names[#names + 1] = selfName end

    local counts, updated = {}, {}
    for _, name in ipairs(names) do
        -- One resource going wrong must not stop the others.
        local okOne, status, folder = pcall(runOne, name, mode, opts, say, source, true, run)
        if not okOne then
            say(("%s %-24s ^1stopped by an error: %s^7"):format(ICON.error, name, tostring(status)))
            status = "error"
        end
        counts[status] = (counts[status] or 0) + 1
        if status == "updated" then updated[#updated + 1] = folder or name end
    end

    local parts = {}
    local function part(k, word, colour)
        local v = counts[k] or 0
        if v > 0 then parts[#parts + 1] = ("%s%s %d %s^7"):format(colour, ICON[k], v, word) end
    end
    part("available", "update(s) available", "^3")
    part("updated", "updated", "^2")
    part("staged", "staged", "^2")
    part("current", "up to date", "^2")
    part("newer", "newer locally", "^3")
    part("noversion", "without a version", "^3")
    part("skipped", "skipped (not started)", "^3")
    part("error", "error(s)", "^1")
    part("duplicate", "declared by two folders (not updated)", "^1")
    part("missing", "not on this server", "^9")
    say(("%d published resource(s)  ·  %s"):format(#names, #parts > 0 and table.concat(parts, "  ·  ") or "nothing to report"))

    if mode == "check" and (counts.available or 0) > 0 then
        if opts.auto then
            say("   ^9install with: poggycore update all apply, or set Updates.AutoUpdate = true in poggy_core/config.lua^7")
        else
            say("   ^9install with: poggycore update all apply   (then refresh, and restart each updated resource)^7")
        end
    end
    return { counts = counts, updated = updated, newDuplicates = run.newDuplicates }
end

--- resource: a name, or "all". mode: "check" | "stage" | "apply".
--- opts: { force, feed, url, repo, branch }. say(msg) prints.
--- Returns what runAll returns. Only one run at a time: a second one is refused.
function Updates.Run(resource, mode, opts, say)
    mode = mode or "check"
    opts = opts or {}
    if Updates.busy then
        say(("^3⏳ an update run is already in progress (%s). Try again when it finishes.^7"):format(tostring(Updates.busy)))
        return nil, "busy"
    end
    -- A development server never writes: stage and apply become a check, so the
    -- owner still sees what is newer, and nothing on disk is touched.
    if mode ~= "check" and Updates.IsDevServer() then
        say(DEV_LINE)
        mode = "check"
    end
    Updates.busy = opts.label or ("update " .. tostring(resource) .. " " .. mode)
    local ok, result, why = pcall(runAll, resource, mode, opts, say)
    Updates.busy = false
    if not ok then
        say("^1❌ the update run stopped with an error: " .. tostring(result) .. "^7")
        return nil, tostring(result)
    end
    return result, why
end

-- ---------------------------------------------------------------------------
-- Automatic updates
-- ---------------------------------------------------------------------------

--- One banner line: where updates come from and whether they install themselves.
function Updates.Describe()
    local c = cfg()
    local from
    if (c.Source or "github") == "website" then
        from = ("website ^5%s^7"):format(tostring(c.Url))
    else
        from = ("GitHub ^5%s@%s^7 ^9(%s)^7"):format(tostring(c.Repo), tostring(c.Branch or "main"),
            token() and "token set" or "public, no token")
    end
    local auto = c.AutoUpdate and "^2on^7" or c.ApplyOnStart and "^3at start only^7" or "^9off^7"
    local interval = math.max(0, tonumber(c.CheckIntervalMinutes) or 60)
    local checks = interval > 0 and ("checks every %d min"):format(interval)
        or (c.CheckOnStart ~= false and "checks at start only" or "no automatic checks")
    local dev = Updates.IsDevServer() and "  ·  ^3dev server: updates read-only^7" or ""
    return ("updates: %s  ·  auto-update: %s  ·  %s%s"):format(from, auto, checks, dev)
end

local function aceAllowed(object)
    if IsPrincipalAceAllowed == nil then return false end
    local ok, allowed = pcall(IsPrincipalAceAllowed, "resource." .. GetCurrentResourceName(), object)
    return ok and allowed == true
end

--- The server.cfg lines poggy_core still needs to restart resources itself.
local function missingAceLines()
    local lines = {}
    for _, cmd in ipairs({ "refresh", "ensure" }) do
        if not aceAllowed("command." .. cmd) then
            lines[#lines + 1] = ("add_ace resource.%s command.%s allow"):format(GetCurrentResourceName(), cmd)
        end
    end
    return lines
end

--- `refresh` once, then restart each updated resource.
---
--- refresh has no native, so it goes through ExecuteCommand and needs the ACE
--- command.refresh; without it nothing is restarted (a restart without refresh
--- keeps the old manifest) and the exact server.cfg lines are printed. Restarts
--- use `ensure` when command.ensure is allowed, otherwise the StopResource /
--- StartResource natives. Every restart is checked afterwards.
---
--- poggy_core never restarts itself. Stopping it stops every resource that
--- depends on it, which is every Poggy script, and it would be torn down in the
--- middle of this very function, so nothing could bring them back. Its new
--- files are written last and take effect on the next server restart, or on
--- `refresh` then `ensure poggy_core` by hand; the console says so.
---
--- A resource whose new fxmanifest.lua declares a poggy_core_min above the
--- running poggy_core is not restarted either (see needsNewerCore): it is
--- installed, and takes effect once poggy_core has been restarted.
local function restartResources(names, say)
    -- A development server is never refreshed or restarted by the updater.
    if Updates.IsDevServer() then
        say(DEV_LINE)
        return
    end
    local own = GetCurrentResourceName()
    local others, held, hasSelf = {}, {}, false
    for _, name in ipairs(names) do
        if name == own then
            hasSelf = true
        else
            local min, running = needsNewerCore(name)
            if min then
                held[#held + 1] = { name = name, min = min, running = running }
            else
                others[#others + 1] = name
            end
        end
    end

    if #others > 0 then
        if not aceAllowed("command.refresh") then
            say("^1❌ cannot restart the updated resources: poggy_core is not allowed to run refresh.^7")
            say("   add these lines to server.cfg, then restart the server once:")
            for _, line in ipairs(missingAceLines()) do say("      ^3" .. line .. "^7") end
            say(("   ^9until then, run by hand: refresh, then ensure %s^7"):format(table.concat(others, ", ensure ")))
        else
            ExecuteCommand("refresh")
            Wait(1000)
            local useEnsure = aceAllowed("command.ensure")
            for _, name in ipairs(others) do
                local state = GetResourceState(name)
                if state ~= "started" then
                    say(("%s %-24s ^3not restarted: it is %s now^7"):format(ICON.skipped, shownFolder(name), tostring(state)))
                else
                    local how, refused
                    if useEnsure then
                        how = "ensure"
                        ExecuteCommand("ensure " .. name)
                    else
                        how = "StopResource/StartResource"
                        local okStop, stopped = pcall(StopResource, name)
                        if not okStop or stopped == false then
                            refused = "StopResource was refused"
                        else
                            Wait(250)
                            local okStart, started = pcall(StartResource, name)
                            if not okStart or started == false then refused = "StartResource was refused" end
                        end
                    end
                    Wait(1000)
                    local after = GetResourceState(name)
                    if not refused and after == "started" then
                        say(("%s %-24s ^2restarted^7 ^9(%s)^7"):format(ICON.updated, shownFolder(name), how))
                    else
                        say(("%s %-24s ^1restart failed: %s (state: %s)^7"):format(ICON.error, shownFolder(name),
                            refused or (how .. " did not bring it back"), tostring(after)))
                        if not useEnsure then
                            say(("      ^3add_ace resource.%s command.ensure allow^7 ^9lets poggy_core use ensure; for now run: ensure %s^7")
                                :format(own, name))
                        end
                    end
                end
            end
        end
    end

    for _, h in ipairs(held) do
        say(("%s %-24s ^3installed, not restarted^7 ^9— needs poggy_core %s and v%s is running."
            .. " It takes effect once poggy_core is restarted: on the next server restart, or run: refresh, then ensure %s, then ensure %s^7")
            :format(ICON.available, shownFolder(h.name), h.min, h.running, own, h.name))
    end

    if hasSelf then
        say(("%s %-24s ^3installed, not restarted^7 ^9— poggy_core does not restart itself (that would stop every Poggy script)."
            .. " It takes effect on the next server restart, or run: refresh, then ensure %s^7"):format(ICON.available, own, own))
    end
end

local pending, watching = {}, false

local function pendingNames()
    local names = {}
    for name in pairs(pending) do names[#names + 1] = name end
    table.sort(names)
    return names
end

local function labelled(label)
    return function(msg) print(PREFIX .. "^9" .. label .. "^7 " .. msg) end
end

--- Resources whose update is installed but whose restart waits for an empty server.
function Updates.PendingRestarts()
    return pendingNames()
end

--- Run the waiting restarts if the server is empty. Returns true when it did.
function Updates.ProcessPending()
    local names = pendingNames()
    if #names == 0 or Updates.busy or GetNumPlayerIndices() > 0 then return false end
    pending = {}
    local say = labelled("[pending restart]")
    say("server is empty: restarting " .. table.concat(names, ", "))
    Updates.busy = "pending restart"
    local ok, err = pcall(restartResources, names, say)
    Updates.busy = false
    if not ok then say("^1❌ restart stopped with an error: " .. tostring(err) .. "^7") end
    return true
end

local function watchPending()
    if watching then return end
    watching = true
    CreateThread(function()
        while next(pending) do
            Wait(30000)
            Updates.ProcessPending()
        end
        watching = false
    end)
end

--- One automatic run. reason: "startup" or "periodic".
function Updates.Auto(reason)
    local c = cfg()
    local wantApply = c.AutoUpdate == true or (reason == "startup" and c.ApplyOnStart == true)
    -- On a development server an automatic apply runs as a check: what is newer
    -- is still listed, nothing is written or restarted.
    local dev = wantApply and Updates.IsDevServer()
    local apply = wantApply and not dev
    local mode = apply and "apply" or "check"
    local label = ("[%s %s]"):format(reason, apply and "update" or "check")
    local say = labelled(label)

    if Updates.busy then
        say(("^9skipped: another update run is in progress (%s)^7"):format(tostring(Updates.busy)))
        return nil
    end

    local restartWanted = apply and c.RestartUpdated ~= false
    if reason == "startup" and restartWanted and not aceAllowed("command.refresh") then
        say("^3⚠️  auto-update cannot restart what it installs: poggy_core is not allowed to run refresh. Add to server.cfg:^7")
        for _, line in ipairs(missingAceLines()) do say("      ^3" .. line .. "^7") end
    end

    -- A periodic run only speaks when there is something to say.
    local periodic = reason ~= "startup"
    local held = {}
    local out = periodic and function(msg) held[#held + 1] = msg end or say
    if dev then out(DEV_LINE) end
    local result = Updates.Run("all", mode, { auto = true, onlyChanges = periodic, label = label }, out)
    if periodic then
        local n = result and result.counts or {}
        local newDuplicates = result and tonumber(result.newDuplicates) or 0
        if not result or (n.available or 0) + (n.updated or 0) + (n.error or 0) + (n.skipped or 0) + newDuplicates > 0 then
            for _, msg in ipairs(held) do say(msg) end
        end
    end
    if not result or not apply or #result.updated == 0 then return result end

    if c.RestartUpdated == false then
        say(("^9RestartUpdated is off. To load the new versions: refresh, then restart %s^7")
            :format(table.concat(result.updated, ", ")))
        return result
    end

    local players = GetNumPlayerIndices()
    if periodic and c.RestartWhenEmptyOnly ~= false and players > 0 then
        local own = GetCurrentResourceName()
        local waiting = {}
        for _, name in ipairs(result.updated) do
            if name == own or needsNewerCore(name) then
                -- Neither is ever restarted here, so neither waits for an
                -- empty server: this only prints what to do.
                restartResources({ name }, say)
            else
                pending[name] = true
                waiting[#waiting + 1] = name
            end
        end
        if #waiting > 0 then
            say(("^3⏳ restart pending^7 for %s ^9— %d player(s) online. The files are installed; the restart runs when the server is empty.^7")
                :format(table.concat(waiting, ", "), players))
            watchPending()
        end
        return result
    end

    -- Anything still waiting from an earlier run goes along with this restart.
    local names, seen = {}, {}
    for _, name in ipairs(pendingNames()) do names[#names + 1] = name; seen[name] = true end
    for _, name in ipairs(result.updated) do
        if not seen[name] then names[#names + 1] = name end
    end
    pending = {}
    -- poggy_core, if it is in the list, stays last.
    table.sort(names, function(a, b)
        local own = GetCurrentResourceName()
        if (a == own) ~= (b == own) then return b == own end
        return a < b
    end)

    Updates.busy = label .. " restart"
    local ok, err = pcall(restartResources, names, say)
    Updates.busy = false
    if not ok then say("^1❌ restart stopped with an error: " .. tostring(err) .. "^7") end
    return result
end

-- Start check, then a check every CheckIntervalMinutes. Labelled, so an
-- automatic run cannot be mistaken for a command someone just typed.
CreateThread(function()
    local c = cfg()
    local atStart = c.CheckOnStart ~= false or c.AutoUpdate == true or c.ApplyOnStart == true
    local interval = math.max(0, tonumber(c.CheckIntervalMinutes) or 60)
    if not atStart and interval == 0 then return end
    Wait(math.max(0, tonumber(c.StartDelaySeconds) or 20) * 1000)
    if atStart then Updates.Auto("startup") end
    while interval > 0 do
        Wait(interval * 60 * 1000)
        Updates.Auto("periodic")
    end
end)

-- ---------------------------------------------------------------------------
-- Write test
--
-- `poggycore update <resource> writetest` tries every way this server might
-- let a script put a file in a resource folder, and says which ones work.
-- Everything it writes, it removes again where it can.
-- ---------------------------------------------------------------------------

function Updates.WriteTest(resource, say)
    -- Everything this does is a write; on a development server none of it runs.
    if Updates.IsDevServer() then
        say(DEV_LINE)
        return
    end
    local own = GetCurrentResourceName()
    resource = resource or own
    -- A folder name is tested as it is; any other name is read as a poggy_id and
    -- the test runs against the folder that holds it.
    local I = PoggyCore.Identity
    if I and resource ~= own and GetResourceState(resource) == "missing" then
        I.Refresh()
        local folder, problem, claimants = I.Locate(resource)
        if problem == "duplicate" then
            say(("^1poggy_id '%s' is declared by %s; name one of the folders instead.^7")
                :format(resource, table.concat(claimants or {}, " and ")))
            return
        end
        if folder then
            say(("poggy_id '%s' is the folder '%s'"):format(resource, folder))
            resource = folder
        end
    end
    if GetResourceState(resource) == "missing" then
        say(("'%s' is not a resource on this server."):format(resource))
        return
    end

    local name = "_poggy_writetest.txt"
    local data = "poggy_core write test " .. os.date("%Y-%m-%d %H:%M:%S")
    local rawPath = GetResourcePath(resource)
    local clean = (rawPath:gsub("//+", "/"))
    local back = (clean:gsub("/", "\\"))
    local relative = clean:match("(resources/.*)$")
    local passed, leftovers = {}, {}

    say(("write test for '%s'"):format(resource))
    say(("  GetResourcePath: %s"):format(rawPath))
    say(("  io.open: %s   os.remove: %s   os.tmpname: %s"):format(
        tostring(io ~= nil and io.open ~= nil),
        tostring(os ~= nil and os.remove ~= nil),
        tostring(os ~= nil and os.tmpname ~= nil)))

    local function remove(path)
        if os and os.remove then
            local ok, removed = pcall(os.remove, path)
            return ok and removed ~= nil
        end
        return false
    end

    local function result(ok, label, detail)
        if ok then passed[#passed + 1] = label end
        say(("  %s %-40s %s"):format(ok and "^2OK  ^7" or "^1FAIL^7", label, detail or ""))
    end

    for _, res in ipairs(resource == own and { own } or { own, resource }) do
        local label = "SaveResourceFile -> " .. res
        local ok = SaveResourceFile(res, name, data, #data)
        result(ok, label, ok and "" or "returned false")
        if ok then
            local path = (GetResourcePath(res) .. "/" .. name):gsub("//+", "/")
            if not remove(path) then leftovers[#leftovers + 1] = path end
        end
    end

    if resource ~= own then
        local okB, whyB = writeFile(resource, name, data)
        result(okB, "own bridge -> " .. resource, okB and "read back OK" or tostring(whyB))
        if okB then
            local p = (GetResourcePath(resource) .. "/" .. name):gsub("//+", "/")
            if not remove(p) then leftovers[#leftovers + 1] = p end
        end

        -- Can an update create a folder? If not, a version that adds one needs
        -- the folder made by hand on each server.
        local folderFile = "_poggy_newfolder_test/probe.txt"
        local okD, whyD = writeFile(resource, folderFile, data)
        result(okD, "new folder via bridge", okD and "folders can be created" or ("cannot: " .. tostring(whyD)))
        if okD then
            leftovers[#leftovers + 1] = (GetResourcePath(resource) .. "/_poggy_newfolder_test"):gsub("//+", "/")
        end

        -- The largest shipped file is about 12 MB (poggy_animtool's animations.json).
        local big = string.rep("P", 12 * 1024 * 1024)
        local started = GetGameTimer()
        local okL, whyL = writeFile(resource, "_poggy_large_test.bin", big)
        result(okL, "12 MB file via bridge", okL and (("%d ms"):format(GetGameTimer() - started)) or tostring(whyL))
        if okL then
            writeFile(resource, "_poggy_large_test.bin", "")
            local p = (GetResourcePath(resource) .. "/_poggy_large_test.bin"):gsub("//+", "/")
            if not remove(p) then leftovers[#leftovers + 1] = p .. " (emptied)" end
        end
        big = nil
    end

    local spellings = {
        { "raw",       rawPath },
        { "cleaned",   clean },
        { "backslash", back },
    }
    if relative then spellings[#spellings + 1] = { "relative", relative } end

    if io and io.open then
        for _, s in ipairs(spellings) do
            local label, base = s[1], s[2]
            local sep = label == "backslash" and "\\" or "/"

            local rf, rerr = io.open(base .. sep .. "fxmanifest.lua", "rb")
            if rf then
                local chunk = rf:read(16)
                rf:close()
                result(true, "io.open READ  (" .. label .. ")", ("read %d byte(s)"):format(chunk and #chunk or 0))
            else
                result(false, "io.open READ  (" .. label .. ")", tostring(rerr))
            end

            local target = base .. sep .. name
            local wf, werr = io.open(target, "wb")
            if wf then
                local ok, e = wf:write(data)
                wf:close()
                result(ok ~= nil and ok ~= false, "io.open WRITE (" .. label .. ")", ok and target or tostring(e))
                if ok and not remove(target) then leftovers[#leftovers + 1] = target end
            else
                result(false, "io.open WRITE (" .. label .. ")", tostring(werr))
            end
        end

        if os and os.tmpname then
            local okName, tmp = pcall(os.tmpname)
            if okName and tmp then
                local tf, terr = io.open(tmp, "wb")
                if tf then
                    tf:write(data)
                    tf:close()
                    result(true, "io.open WRITE (temp file)", tmp)
                    remove(tmp)
                else
                    result(false, "io.open WRITE (temp file)", tostring(terr))
                end
            end
        end
    end

    say(("write test: %d method(s) worked%s"):format(#passed,
        #passed > 0 and (": " .. table.concat(passed, ", ")) or ""))
    if #leftovers > 0 then
        say("could not remove these test files; delete them by hand:")
        for _, p in ipairs(leftovers) do say("  " .. p) end
    end
end
