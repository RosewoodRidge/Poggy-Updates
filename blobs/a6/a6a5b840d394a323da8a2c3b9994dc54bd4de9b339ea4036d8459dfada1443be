--[[
    poggy_core — network test for the updater.

        poggycore update nettest [url=https://...]

    PerformHttpRequest reported "Failure handling HTTP request" for the update site
    while the same URL loads everywhere else from the same machine. This asks the
    server itself for a handful of addresses and prints what came back for each, so
    the cause shows up as a pattern:

        everything fails                -> the server has no outbound HTTP at all
        only https fails, http answers  -> TLS between FXServer and that host
        the site fails, others work     -> something specific to the site or its CDN
        the IP answers, the name fails  -> DNS as the server resolves it

    Nothing is written anywhere.
]]

PoggyCore = PoggyCore or {}
PoggyCore.Updates = PoggyCore.Updates or {}

local function request(url, headers)
    local p = promise.new()
    local started = GetGameTimer()
    PerformHttpRequest(url, function(status, body, respHeaders, errorData)
        p:resolve({
            status = status,
            size = body and #body or 0,
            err = errorData,
            ms = GetGameTimer() - started,
            headers = respHeaders,
        })
    end, "GET", "", headers or { ["User-Agent"] = "poggy_core-updater" })
    return Citizen.Await(p)
end

function PoggyCore.Updates.NetTest(say, opts)
    opts = opts or {}
    local site = "rosewoodridge.xyz"
    local feed = (opts.url or (PoggyCoreConfig.Updates or {}).Url or ("https://" .. site .. "/api/updates/scripts"))
        :gsub("/+$", "") .. "/index.json"

    local byIp = { ["Host"] = site, ["User-Agent"] = "poggy_core-updater" }
    local up = PoggyCoreConfig.Updates or {}
    local feedRepo, feedBranch = up.Repo or "RosewoodRidge/Poggy-Updates", up.Branch or "main"
    local tok = GetConvar("poggy_github_token", "")
    local tests = {
        { "GitHub feed via API" .. (tok ~= "" and " + token" or " (no token)"),
          ("https://api.github.com/repos/%s/contents/index.json?ref=%s"):format(feedRepo, feedBranch),
          { ["User-Agent"] = "poggy_core-updater", ["Accept"] = "application/vnd.github.raw",
            ["Authorization"] = tok ~= "" and ("Bearer " .. tok) or nil } },
        { "GitHub feed via raw (public)",
          ("https://raw.githubusercontent.com/%s/%s/index.json"):format(feedRepo, feedBranch) },
        { "update feed (https)",           feed },
        { "site file (https)",             "https://" .. site .. "/robots.txt" },
        { "site file (plain http)",        "http://" .. site .. "/robots.txt" },
        { "www. subdomain (https)",        "https://www." .. site .. "/robots.txt" },
        { "explicit port 443 (https)",     "https://" .. site .. ":443/robots.txt" },
        { "another .xyz site (https)",     "https://abc.xyz/" },
        { "site by IP, no DNS (http)",     "http://159.198.67.238/robots.txt", byIp },
        { "feed by IP, no DNS (http)",     "http://159.198.67.238/api/updates/scripts/index.json", byIp },
        { "site by IP (https)",            "https://159.198.67.238/robots.txt", byIp },
        { "known https by IP (1.1.1.1)",   "https://1.1.1.1/cdn-cgi/trace" },
        { "GitHub API (worked before)",    "https://api.github.com/zen" },
        { "Cloudflare (https, ECDSA)",     "https://www.cloudflare.com/cdn-cgi/trace" },
        { "Let's Encrypt (https)",         "https://letsencrypt.org/robots.txt" },
    }

    say("network test from the server (nothing is written)")
    for _, t in ipairs(tests) do
        local r = request(t[2], t[3])
        local ok = r.status and r.status > 0
        say(("  %s %-30s status=%s  bytes=%d  %d ms%s"):format(
            ok and "^2answered^7" or "^1FAILED  ^7", t[1], tostring(r.status), r.size, r.ms,
            (not ok and r.err) and ("  error: " .. tostring(r.err)) or ""))
        say("           " .. t[2])
    end
    say("paste this whole block; the pattern of answers says where the problem is.")
end
