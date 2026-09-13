--[[
    poggy_core — verb self-test.

    /poggycore selftest        read-only. Nothing is modified.
    /poggycore selftest full   also exercises writes, each paired with its
                               inverse and verified back to the starting value.

    Every check goes through PoggyCore.Do, the same path a script takes, so this
    tests the dispatcher, the verb table, the adapter and the framework together
    rather than any one of them in isolation.

    Three outcomes, and the difference matters:
      PASS  the verb did what it claims
      FAIL  the verb is broken, or the contract is not what it says
      SKIP  the framework genuinely cannot do this, and said so honestly

    A SKIP is a pass for the capability map. A verb that returns ok for
    something the framework cannot do would be a FAIL, and a worse one than a
    crash, because it loses data quietly.
]]

PoggyCore = PoggyCore or {}

local R = { pass = 0, fail = 0, skip = 0, lines = {}, seen = {} }

local function reset()
    R = { pass = 0, fail = 0, skip = 0, lines = {}, seen = {} }
end

local function note(kind, verb, detail)
    R[kind] = R[kind] + 1
    -- Track which verbs were actually reached. Counting checks and calling the
    -- result "verbs exercised" overstated coverage, which is the one thing a
    -- test must never do about itself.
    if PoggyCore.Verbs[verb] then R.seen[verb] = true end
    local colour = kind == "pass" and "^2PASS^7" or (kind == "fail" and "^1FAIL^7" or "^3SKIP^7")
    R.lines[#R.lines + 1] = ("%s %-20s %s"):format(colour, verb, detail or "")
end

--- Render a value compactly enough for chat.
local function show(v, depth)
    depth = depth or 0
    if type(v) ~= "table" then return tostring(v) end
    if depth > 0 then return "{...}" end
    local n = 0
    for _ in pairs(v) do n = n + 1 end
    return ("table(%d)"):format(n)
end

--- Run a verb and judge the result.
--- expect: 'ok'        must succeed
---         'fail'      must fail, for any reason
---         'refuse'    must fail with one of the listed errors (honest refusal)
local function check(verb, payload, expect, allowedErrs, describe)
    local ok, value, err = PoggyCore.Do(verb, payload)

    if expect == "ok" then
        if ok then
            note("pass", verb, describe and describe(value) or ("-> " .. show(value)))
        elseif allowedErrs and allowedErrs[err] then
            note("skip", verb, "refused honestly: " .. tostring(err))
        else
            note("fail", verb, "err = " .. tostring(err))
        end
        return ok, value

    elseif expect == "fail" then
        if ok then
            note("fail", verb, "succeeded but should not have")
        else
            note("pass", verb, "refused: " .. tostring(err))
        end
        return ok, value

    elseif expect == "refuse" then
        if ok then
            note("fail", verb, "returned ok for something it cannot do")
        elseif allowedErrs and not allowedErrs[err] then
            note("fail", verb, ("refused with '%s', expected one of the honest errors"):format(tostring(err)))
        else
            note("pass", verb, "refused: " .. tostring(err))
        end
        return ok, value
    end
end

-- ---------------------------------------------------------------------------

local UNSUPPORTED = { unsupported = true, not_implemented = true }
local NOCHAR      = { no_char = true, not_found = true }

local function runTests(src, full)
    reset()

    -- --- core ---------------------------------------------------------------
    check("core.ready", {}, "ok", nil, function(v) return "ready = " .. tostring(v) end)
    check("core.version", {}, "ok", nil, function(v) return "v" .. tostring(v) end)
    check("core.framework", {}, "ok", nil, function(v)
        return ("%s (%s)"):format(v and v.id or "?", v and v.label or "?")
    end)
    check("core.caps", {}, "ok")
    check("core.has", { capability = "inventory.items" }, "ok", nil,
        function(v) return "inventory.items = " .. tostring(v) end)

    -- --- character ----------------------------------------------------------
    local okChar, char = check("char.get", { src = src }, "ok", NOCHAR, function(v)
        if type(v) ~= "table" then return "?" end
        return ("charId=%s name=%s job=%s"):format(
            tostring(v.charId), tostring(v.fullName), tostring(v.job))
    end)

    if okChar and type(char) == "table" then
        -- The normalised shape is part of the contract, so check it is really there.
        local missing = {}
        for _, f in ipairs({ "charId", "ownerId", "firstName", "lastName", "job", "jobGrade" }) do
            if char[f] == nil then missing[#missing + 1] = f end
        end
        if #missing > 0 then
            note("fail", "char shape", "missing: " .. table.concat(missing, ", "))
        else
            note("pass", "char shape", "all documented fields present")
        end

        check("char.byId", { charId = char.charId }, "ok", NOCHAR, function(v)
            return v and ("round-tripped charId " .. tostring(v.charId)) or "?"
        end)
    end

    check("char.id", { src = src }, "ok", NOCHAR)
    check("players.list", {}, "ok", nil, function(v)
        return ("%d online"):format(type(v) == "table" and #v or 0)
    end)

    -- --- money --------------------------------------------------------------
    local supported = {}
    for _, cur in ipairs({ "cash", "bank", "gold", "rol" }) do
        local _, yes = PoggyCore.Do("money.supports", { currency = cur })
        supported[cur] = yes and true or false
    end
    note("pass", "money.supports", ("cash=%s bank=%s gold=%s rol=%s"):format(
        tostring(supported.cash), tostring(supported.bank),
        tostring(supported.gold), tostring(supported.rol)))

    for cur, yes in pairs(supported) do
        if yes then
            check("money.get", { src = src, currency = cur }, "ok", NOCHAR,
                function(v) return ("%s = %s"):format(cur, tostring(v)) end)
        else
            -- The important one. An unsupported currency must refuse, not lie.
            check("money.get", { src = src, currency = cur }, "refuse", UNSUPPORTED)
        end
    end

    -- --- jobs ---------------------------------------------------------------
    local okJob, job = check("job.get", { src = src }, "ok", NOCHAR, function(v)
        if type(v) ~= "table" then return "?" end
        return ("%s grade %s (%s)"):format(tostring(v.name), tostring(v.grade), tostring(v.label))
    end)
    if okJob and type(job) == "table" and job.name then
        check("job.has", { src = src, job = job.name }, "ok", nil, function(v)
            return v and "true (own job)" or "FALSE for its own job"
        end)
    end
    check("job.has", { src = src, job = "definitely_not_a_job" }, "ok", nil,
        function(v) return "false for a made-up job = " .. tostring(v) end)
    check("job.isLaw", { src = src }, "ok")
    check("job.isMedical", { src = src }, "ok")

    -- --- 0.11.0 reads -------------------------------------------------------
    check("players.onDuty", {}, "ok", nil, function(v)
        return ("%d player(s) on duty"):format(type(v) == "table" and #v or 0)
    end)
    local selfId = (type(char) == "table") and char.charId or nil
    if selfId then
        check("char.offline", { charId = selfId }, "ok", UNSUPPORTED, function(v)
            return type(v) == "table" and ("%s, online = %s"):format(tostring(v.fullName), tostring(v.online)) or "?"
        end)
    else
        note("skip", "char.offline", "no character id to look up")
    end
    check("char.list", { limit = 5 }, "ok", UNSUPPORTED, function(v)
        return ("%d character(s) (limit 5)"):format(type(v) == "table" and #v or 0)
    end)

    if full and okJob and type(job) == "table" and job.name then
        -- Re-setting the job the player already has. Exercises the write path
        -- without changing anything about them.
        check("job.set", { src = src, job = job.name, grade = job.grade, label = job.label },
              "ok", UNSUPPORTED)
        -- The same no-op, written to the database: the row gets the values it has.
        check("job.set", { src = src, job = job.name, grade = job.grade, label = job.label, persist = true },
              "ok", UNSUPPORTED, function() return "persist = true (same job written back)" end)
        local _, j2 = PoggyCore.Do("job.get", { src = src })
        if type(j2) == "table" and j2.name ~= job.name then
            note("fail", "job.set", ("no-op set changed the job from %s to %s")
                :format(tostring(job.name), tostring(j2.name)))
        end
        if job.onDuty ~= nil then
            check("job.duty", { src = src, onDuty = job.onDuty }, "ok", UNSUPPORTED)
        else
            note("skip", "job.duty", "this framework cannot report duty, so nothing to restore")
        end
    end

    -- --- inventory ----------------------------------------------------------
    local okInv, items = check("inv.get", { src = src }, "ok", UNSUPPORTED, function(v)
        return ("%d stack(s)"):format(type(v) == "table" and #v or 0)
    end)

    -- The probe item is used for the add/remove round trips below. Skip the
    -- items that ARE money on a framework which represents cash as inventory
    -- items (RSG: dollar, cent, blood_dollar, blood_cent, gold): adding one of
    -- those changes the balance, and the money checks further down would then
    -- be measuring this test rather than the framework. Weapons are skipped
    -- too, because a unique item does not stack and the count arithmetic
    -- assumes it does.
    local MONEY_ITEMS = { dollar = true, cent = true, blood_dollar = true, blood_cent = true, gold = true }
    local probeStack = nil
    if okInv and type(items) == "table" then
        for _, it in ipairs(items) do
            if it.name and not MONEY_ITEMS[it.name] and it.type ~= "weapon" then
                probeStack = it
                break
            end
        end
        probeStack = probeStack or items[1]
    end
    local probe = probeStack and probeStack.name or nil
    if probe then
        -- inv.count sums every stack of the item; inv.get lists stacks. Compare
        -- against the sum, not the first stack, or a split stack fails this.
        local expected = 0
        for _, it in ipairs(items) do
            if it.name == probe then expected = expected + (tonumber(it.amount) or 0) end
        end
        local _, n = check("inv.count", { src = src, item = probe }, "ok", UNSUPPORTED,
            function(v) return ("%s x%s"):format(probe, tostring(v)) end)
        if n ~= nil and expected > 0 and n ~= expected then
            note("fail", "inv.count", ("said %s, inv.get said %s"):format(tostring(n), tostring(expected)))
        end
        check("inv.has", { src = src, item = probe, qty = 1 }, "ok", UNSUPPORTED)
        check("inv.canCarry", { src = src, item = probe, qty = 1 }, "ok", UNSUPPORTED)
    else
        note("skip", "inv.count", "no items to probe with")
    end

    check("weapon.get", { src = src }, "ok", UNSUPPORTED, function(v)
        return ("%d weapon(s)"):format(type(v) == "table" and #v or 0)
    end)
    check("weapon.canCarry", { src = src, qty = 1 }, "ok", UNSUPPORTED, function(v)
        return "room for one more = " .. tostring(v)
    end)

    check("inv.imageBase", {}, "ok", UNSUPPORTED, function(v) return tostring(v) end)
    local okItems, registry = check("inv.items", { limit = 5 }, "ok", UNSUPPORTED, function(v)
        return ("%d item(s) (limit 5)"):format(type(v) == "table" and #v or 0)
    end)
    if okItems and type(registry) == "table" and registry[1] then
        check("inv.itemInfo", { item = registry[1].name }, "ok", UNSUPPORTED, function(v)
            return type(v) == "table" and ("%s = %s"):format(tostring(v.name), tostring(v.label)) or "?"
        end)
    else
        note("skip", "inv.itemInfo", "the registry gave no item to look up")
    end

    -- --- storage ------------------------------------------------------------
    -- Two scratch containers, both created here.
    --
    -- The first carries the add/remove round trip and is DELETED at the end, so
    -- nothing this test creates outlives it. That matters: an earlier run left
    -- an item behind, because unregister deliberately keeps contents, and the
    -- leftover then broke the next run's arithmetic.
    --
    -- The second exists only to exercise unregister, and is left empty so that
    -- forgetting it costs nothing.
    local scratch  = "poggycore_selftest"
    local scratch2 = "poggycore_selftest_unreg"

    --- How many of `name` are in this item list. Counting entries instead of
    --- summing amounts is what made the round trip look broken: identical items
    --- stack, so adding one to an existing stack never changes the entry count.
    local function amountOf(items, name)
        if type(items) ~= "table" then return 0 end
        local total = 0
        for _, it in ipairs(items) do
            if it.name == name then total = total + (tonumber(it.amount) or 0) end
        end
        return total
    end

    local okReg = check("storage.register", {
        id = scratch,
        -- Shared, because vorp_inventory refuses a remove from a non-shared
        -- container without an identifier the storage API cannot pass. On RSG
        -- a stash is one flat container and the flag is ignored.
        opts = { label = "poggy_core self-test", slots = 4, maxWeight = 200000, shared = true },
    }, "ok", UNSUPPORTED)

    if okReg then
        check("storage.registered", { id = scratch }, "ok", nil,
            function(v) return "registered = " .. tostring(v) end)
        check("storage.capacity", { id = scratch, slots = 6 }, "ok", UNSUPPORTED)

        local _, opening = PoggyCore.Do("storage.items", { id = scratch })
        local leftover = type(opening) == "table" and #opening or 0
        check("storage.items", { id = scratch }, "ok", UNSUPPORTED, function()
            return leftover == 0 and "empty, as expected"
                or ("%d stack(s) left over from an earlier run; cleaned up below"):format(leftover)
        end)
        check("storage.weapons", { id = scratch }, "ok", UNSUPPORTED, function(v)
            return ("%d weapon(s)"):format(type(v) == "table" and #v or 0)
        end)

        -- charId is not decoration: vorp_inventory refuses a container add
        -- outright without a valid one ("charid is not valid"). Other
        -- frameworks (RSG) accept and ignore it.
        local cid = (type(char) == "table") and char.charId or nil
        if full and probe and cid then
            local base = amountOf(opening, probe)

            -- storage.items is not read-your-writes. GET_ALL_ITEMS runs a fresh
            -- SELECT against character_inventories, while the add that preceded
            -- it answers as soon as the in-memory state is updated and leaves
            -- the DB write in flight -- one of those took 238ms in a previous
            -- run. Reading once immediately after a write therefore sees the
            -- OLD number, which is what made this look like a failed add when
            -- the database plainly showed amount = 2.
            --
            -- So poll, and report how long it actually took. A number here is
            -- evidence about the API; a pass/fail on a single read is a coin toss.
            local function settle(want, budgetMs)
                local waited = 0
                while waited <= (budgetMs or 2000) do
                    local _, items = PoggyCore.Do("storage.items", { id = scratch })
                    local n = amountOf(items, probe)
                    if n == want then return n, waited end
                    Wait(100); waited = waited + 100
                end
                local _, items = PoggyCore.Do("storage.items", { id = scratch })
                return amountOf(items, probe), waited
            end

            check("storage.addItem",
                  { id = scratch, item = probe, qty = 1, charId = cid }, "ok", UNSUPPORTED)
            local m, tAdd = settle(base + 1)

            check("storage.removeItem", { id = scratch, item = probe, qty = 1 }, "ok", UNSUPPORTED)
            local a, tRem = settle(base)

            if m == base + 1 and a == base then
                note("pass", "storage round-trip",
                     ("%s %d -> %d -> %d (visible after %dms / %dms)")
                     :format(probe, base, m, a, tAdd, tRem))
            else
                note("fail", "storage round-trip",
                     ("%s went %d -> %d -> %d, expected %d -> %d -> %d (waited %dms / %dms)")
                     :format(probe, base, m, a, base, base + 1, base, tAdd, tRem))
            end
        end

        -- Delete, not unregister: this container may hold items this test put
        -- there, and they must not survive the run. Everything in it was created
        -- by the test, so nothing a player owns is destroyed.
        check("storage.delete", { id = scratch }, "ok", UNSUPPORTED)

        -- Unregister on an empty container, so the difference between forgetting
        -- a container and destroying its contents cannot cost anything here.
        if PoggyCore.Do("storage.register", { id = scratch2, opts = { slots = 1, shared = true } }) then
            check("storage.unregister", { id = scratch2 }, "ok", UNSUPPORTED)
        end
    end

    -- --- registrations ------------------------------------------------------
    -- Both hand poggy_core a function to hold rather than asking a question.
    -- Registering against a name nothing uses is inert.
    check("callback.register", {
        name = "poggy_core:selftest:noop",
        fn   = function() return true end,
    }, "ok", UNSUPPORTED)

    check("inv.registerUsable", {
        item = "poggycore_selftest_item",
        fn   = function() end,
    }, "ok", UNSUPPORTED)

    check("notify.rich", {
        src = src, title = "poggy_core", description = "self-test", kind = "info", duration = 3000,
    }, "ok")

    -- --- permissions --------------------------------------------------------
    check("perms.group", { src = src }, "ok", NOCHAR, function(v) return tostring(v) end)
    check("perms.groups", { src = src }, "ok", NOCHAR, function(v)
        return type(v) == "table" and table.concat(v, ", ") or "?"
    end)
    check("perms.isAdmin", { src = src }, "ok", nil, function(v) return tostring(v) end)

    -- --- notifications ------------------------------------------------------
    check("notify", { src = src, text = "poggy_core self-test", kind = "info", duration = 3000 }, "ok")
    check("notify.styled", { src = src, style = "objective", text = "self-test styled", duration = 3000 }, "ok")

    -- --- the guardrails -----------------------------------------------------
    -- These must fail. A dispatcher that accepts nonsense is worse than one
    -- that rejects something valid, because nonsense fails silently later.
    check("no.such.verb", {}, "fail")
    check("money.add", { src = src }, "fail")               -- no amount
    check("char.byId", {}, "fail")                          -- no charId

    -- --- writes, paired -----------------------------------------------------
    if full then
        local cur = supported.cash and "cash" or (supported.gold and "gold" or nil)
        if not cur then
            note("skip", "money write", "no writable currency")
        else
            local _, before = PoggyCore.Do("money.get", { src = src, currency = cur })
            before = tonumber(before) or 0

            local addOk = check("money.add", { src = src, currency = cur, amount = 10, reason = "selftest" }, "ok")
            local _, mid = PoggyCore.Do("money.get", { src = src, currency = cur })

            if addOk and (tonumber(mid) or 0) ~= before + 10 then
                note("fail", "money round-trip", ("%s -> %s, expected %s"):format(
                    tostring(before), tostring(mid), tostring(before + 10)))
            end

            check("money.remove", { src = src, currency = cur, amount = 10, reason = "selftest" }, "ok")
            local _, after = PoggyCore.Do("money.get", { src = src, currency = cur })

            if (tonumber(after) or 0) == before then
                note("pass", "money round-trip", ("%s back to %s"):format(cur, tostring(after)))
            else
                note("fail", "money round-trip", ("started %s, ended %s — MONEY WAS LOST OR CREATED")
                    :format(tostring(before), tostring(after)))
            end

            -- Overdraw must be refused rather than taking the player negative.
            check("money.remove", { src = src, currency = cur, amount = before + 1000000 },
                  "refuse", { no_funds = true })

            -- Setting the balance to what it already is exercises the verb
            -- without moving a penny.
            check("money.set", { src = src, currency = cur, amount = before, reason = "selftest" },
                  "ok", UNSUPPORTED)
            local _, settled = PoggyCore.Do("money.get", { src = src, currency = cur })
            if (tonumber(settled) or 0) ~= before then
                note("fail", "money.set", ("no-op set changed %s from %s to %s")
                    :format(cur, tostring(before), tostring(settled)))
            end
        end

        if probe then
            local _, n0 = PoggyCore.Do("inv.count", { src = src, item = probe })
            n0 = tonumber(n0) or 0
            check("inv.add", { src = src, item = probe, qty = 1 }, "ok", UNSUPPORTED)
            local _, n1 = PoggyCore.Do("inv.count", { src = src, item = probe })
            check("inv.remove", { src = src, item = probe, qty = 1 }, "ok", UNSUPPORTED)
            local _, n2 = PoggyCore.Do("inv.count", { src = src, item = probe })

            if (tonumber(n1) or 0) == n0 + 1 and (tonumber(n2) or 0) == n0 then
                note("pass", "inv round-trip", ("%s %d -> %d -> %d"):format(probe, n0, n1, n2))
            else
                note("fail", "inv round-trip", ("%s started %d, went %s, ended %s")
                    :format(probe, n0, tostring(n1), tostring(n2)))
            end
        end
    end

    return R
end

-- ---------------------------------------------------------------------------

--- Public so sv_commands.lua can drive it.
function PoggyCore.SelfTest(src, full, chat)
    if not src or src == 0 then
        chat("the self-test needs a player: run it in game, not from the console.")
        return
    end

    -- Everything also goes to the server console, without colour codes, because
    -- chat is where you read it and the console is where you can select it.
    local function reply(msg)
        chat(msg)
        print("[poggy_core:selftest] " .. tostring(msg):gsub("%^%d", ""))
    end

    print("[poggy_core:selftest] ================ begin ================")
    reply(("running self-test (%s)..."):format(full and "^3full — includes writes^7" or "read-only"))

    local res = runTests(src, full)
    for _, line in ipairs(res.lines) do reply(line) end

    local all      = PoggyCore.VerbNames()
    local seen, untested = 0, {}
    for _, v in ipairs(all) do
        if res.seen[v] then seen = seen + 1 else untested[#untested + 1] = v end
    end

    reply(("^2%d passed^7, %s%d failed^7, ^3%d skipped^7  —  %d checks across %d of %d verbs")
        :format(res.pass,
                res.fail > 0 and "^1" or "^2",
                res.fail, res.skip,
                res.pass + res.fail + res.skip,
                seen, #all))

    -- Say why, not just which. A verb left out on purpose is a different thing
    -- from one nobody noticed, and only this file knows the difference.
    local WHY = {
        ["storage.open"]   = "opens a container on the player's screen",
        ["storage.close"]  = "only meaningful after storage.open",
        ["storage.rawIds"] = "a server-wide flag; flipping it would affect every resource",
        ["weapon.add"]     = "cannot be paired safely — a failed removal leaves a real weapon",
        ["weapon.remove"]  = "same, and it needs a weapon id this test has no safe way to obtain",
        ["inv.setMeta"]    = "would rewrite metadata on an item the player actually owns",
        ["inv.close"]      = "closes the player's inventory screen",
        ["callback.await"] = "client-side; this self-test runs on the server",
        ["char.reloadSkin"] = "client-side, and it re-dresses the player's ped",
        ["core.register"]  = "every script's bridge calls it at start; poggycore scripts lists the result",
        ["sql.install"]    = "every script's bridge runs it at start; poggycore sql check all covers it",
        ["sql.check"]      = "same; run poggycore sql check all to see it",
        -- 0.14.0: the three ui verbs wait for a person to press something, so
        -- there is no way to exercise them without one. Open a menu in game.
        ["menu.open"]      = "opens a menu on the player's screen and waits for a choice",
        ["menu.close"]     = "only meaningful after menu.open or input.text",
        ["input.text"]     = "opens a text box on the player's screen and waits for an answer",
    }
    if #untested > 0 then
        reply(("^3%d verb(s) deliberately not exercised:^7"):format(#untested))
        for _, v in ipairs(untested) do
            reply(("   %-20s %s"):format(v, WHY[v] or "^1no reason recorded — check the self-test^7"))
        end
    end

    if res.fail == 0 then
        reply(full and "^2everything passed, and every write came back to where it started.^7"
                    or "^2everything passed. Run '/poggycore selftest full' to exercise writes.^7")
    else
        reply("^1something is wrong. The FAIL lines above say what.^7")
    end
    print("[poggy_core:selftest] ================= end =================")
end
