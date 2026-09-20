--- poggy_core — the verb table.
---
--- Every Poggy script talks to poggy_core through exactly one call:
---
---     local ok, value, err = Poggy('money.add', { src = src, amount = 50 })
---
--- This file is the list of things that call can ask for. It is pure data: the
--- handlers live in sv_dispatch.lua and cl_dispatch.lua. Keeping the list
--- separate from the handlers means both sides, and `/poggycore verbs`, read
--- from one source, and a typo in a verb name is caught here rather than
--- silently doing nothing.
---
--- Fields:
---   side      'server' | 'client' | 'both' — where the verb may be called
---   args      required payload keys; a missing one fails before dispatch
---   optional  keys that are read if present
---   returns   what `value` holds on success, for the docs and /poggycore verbs
---
--- Adding a verb here plus a handler in the dispatcher is the ONLY change
--- needed to give all eighteen resources a new capability. Nothing in any
--- script changes, and nothing has to be re-uploaded to Cfx. That is the whole
--- reason this indirection exists.

PoggyCore = PoggyCore or {}

PoggyCore.Verbs = {

    -- ---------------------------------------------------------- character --
    ["char.get"]        = { side = "both",   args = {},        optional = {"src"}, returns = "character table" },
    ["char.byId"]       = { side = "server", args = {"charId"},                    returns = "character table" },
    ["char.id"]         = { side = "both",   args = {},        optional = {"src"}, returns = "character id" },
    ["players.list"]    = { side = "server", args = {},                            returns = "array of server ids" },
    -- 0.11.0. A character that may be offline, read from the framework's own
    -- table (online characters come back live). appearance = true adds
    -- appearance = { skin, comps, tints }, the stored values as they are.
    ["char.offline"]    = { side = "server", yields = true, args = {"charId"}, optional = {"appearance"},       returns = "character table with online = boolean" },
    -- 0.11.0. Every character, online or not, sorted by name. search matches
    -- the full name; limit/offset page through it.
    ["char.list"]       = { side = "server", yields = true, args = {}, optional = {"search", "limit", "offset"}, returns = "array of { charId, ownerId, firstName, lastName, fullName }" },
    -- 0.11.0. Re-apply the local player's saved appearance and clothing.
    ["char.reloadSkin"] = { side = "client", args = {},                            returns = "true" },
    -- 0.11.0. Players who are on duty, optionally only those with `job` (a
    -- name or an array) and at least `minGrade`. Duty nobody can report counts
    -- as off duty, so this never returns a player who might not be working.
    ["players.onDuty"]  = { side = "server", args = {}, optional = {"job", "minGrade"},                        returns = "array of server ids" },

    -- -------------------------------------------------------------- money --
    -- raw = true (0.17.0) reads or sets the framework's own field even when a
    -- bank provider is registered; only the provider itself has a use for it.
    ["money.get"]       = { side = "server", args = {"src"},   optional = {"currency", "raw"},               returns = "number" },
    ["money.add"]       = { side = "server", args = {"src", "amount"}, optional = {"currency", "reason"},    returns = "true" },
    ["money.remove"]    = { side = "server", args = {"src", "amount"}, optional = {"currency", "reason"},    returns = "true" },
    ["money.set"]       = { side = "server", args = {"src", "amount"}, optional = {"currency", "reason", "raw"}, returns = "true" },
    ["money.supports"]  = { side = "both",   args = {"currency"},                                           returns = "boolean" },

    -- ---------------------------------------------------- providers (0.17.0) --
    -- A Poggy script supplies what the framework lacks and registers once at
    -- start; poggy_core routes the matching verbs to it (server/sv_providers.lua).
    -- fns is a table of functions, each answering ok, value, err.
    --   bank:     get(src) add(src, amount, reason) remove(...) set(...)
    --             -> money.* with currency = 'bank' go there on every framework.
    --   treasury: collect(p) index() rates() state() disburse(p) balance(account) report(p)
    ["bank.register"]     = { side = "server", args = {"fns"},                                                 returns = "true" },
    ["treasury.register"] = { side = "server", args = {"fns"},                                                 returns = "true" },
    -- Every treasury verb refuses with 'unsupported' when no treasury is
    -- installed; callers default (a fee is deleted, the index is 1.0).
    ["treasury.collect"]  = { side = "server", args = {"kind", "amount"}, optional = {"src", "charId", "source", "meta"}, returns = "true" },
    ["treasury.index"]    = { side = "server", args = {},                                                       returns = "number, the price index" },
    ["treasury.rates"]    = { side = "server", args = {},                                                       returns = "{ withdrawalLevy, transferFee, depositInterest, loanRate, salesTaxAdjust }" },
    ["treasury.state"]    = { side = "server", args = {},                                                       returns = "{ verdict, event, index, supply, reserveWeeks, warmup }" },
    ["treasury.disburse"] = { side = "server", args = {"account", "amount", "src"}, optional = {"reason"},     returns = "true" },
    ["treasury.balance"]  = { side = "server", args = {"account"},                                             returns = "number" },
    -- Activity, fire-and-forget: what players did, so the treasury can build
    -- a baseline. metric is a short name ('shop_sale'); value a number.
    ["treasury.report"]   = { side = "server", args = {"metric", "value"}, optional = {"source", "meta"},       returns = "true" },

    -- --------------------------------------------------------------- jobs --
    ["job.get"]         = { side = "both",   args = {},        optional = {"src"},
                            returns = "{ name, grade, label, gradeLabel, onDuty }" },
    -- persist = true (0.11.0) also writes the job to the framework's database,
    -- so resources that read the table see it now. From a thread the write is
    -- awaited and a failure returns false (the in-memory job is still set);
    -- off a thread it runs on its own and a failure is logged.
    ["job.set"]         = { side = "server", args = {"src", "job"}, optional = {"grade", "label", "persist"}, returns = "true" },
    ["job.duty"]        = { side = "server", args = {"src", "onDuty"},                                      returns = "true" },
    ["job.has"]         = { side = "both",   args = {"job"},   optional = {"src", "minGrade"},              returns = "boolean" },
    ["job.isLaw"]       = { side = "both",   args = {},        optional = {"src"},                          returns = "boolean" },
    ["job.isMedical"]   = { side = "both",   args = {},        optional = {"src"},                          returns = "boolean" },
    -- 0.18.0. Every job the server knows, for pickers (the settings hub).
    -- RSG and QBR: the core's shared jobs table. VORP keeps jobs as free text,
    -- so it is the distinct jobs in the characters table, labels = names, best
    -- effort. Standalone: an empty list.
    ["jobs.list"]       = { side = "server", yields = true, args = {},                                      returns = "array of { name, label, grades = { { grade, label }, ... } }" },

    -- ---------------------------------------------------------- inventory --
    ["inv.add"]         = { side = "server", yields = true, args = {"src", "item"}, optional = {"qty", "meta"},            returns = "true" },
    ["inv.remove"]      = { side = "server", yields = true, args = {"src", "item"}, optional = {"qty", "meta"},            returns = "true" },
    ["inv.count"]       = { side = "server", yields = true, args = {"src", "item"}, optional = {"meta"},                   returns = "number" },
    ["inv.has"]         = { side = "server", yields = true, args = {"src", "item"}, optional = {"qty"},                    returns = "boolean" },
    ["inv.get"]         = { side = "server", yields = true, args = {"src"},                                               returns = "array of items" },
    ["inv.canCarry"]    = { side = "server", yields = true, args = {"src", "item"}, optional = {"qty"},                    returns = "boolean" },
    -- 0.16.0. How many more of `item` the player can hold, up to `cap` (default
    -- 1000). Found from the framework's own carry check, so it follows VORP's
    -- per-item limit and RSG's and QBR's weight and slots alike.
    ["inv.maxCarry"]    = { side = "server", yields = true, args = {"src", "item"}, optional = {"cap"},                    returns = "number" },
    ["inv.setMeta"]     = { side = "server", yields = true, args = {"src", "itemId", "meta"}, optional = {"amount"},       returns = "true" },
    -- 0.11.0. The item registry: every item the framework knows, read once at
    -- start and kept (0.18.2). search matches name or label.
    -- checkImages = true adds hasImage (whether the icon file exists).
    ["inv.items"]       = { side = "server", yields = true, args = {}, optional = {"search", "limit", "checkImages"},     returns = "array of { name, label, desc, weight, limit, type, usable, group, image }" },
    ["inv.itemInfo"]    = { side = "server", yields = true, args = {"item"}, optional = {"checkImages"},                  returns = "{ name, label, desc, weight, limit, type, usable, group, image }" },
    -- 0.11.0. The URL prefix item icons live under, for NUI: prefix .. name .. '.png'.
    ["inv.imageBase"]   = { side = "both",   args = {},                                                                  returns = "URL prefix string" },
    -- 0.11.0. Close the player's own inventory screen (not a storage).
    ["inv.close"]       = { side = "server", args = {"src"},                                                             returns = "true" },
    -- These two register a function rather than asking a question, so the
    -- payload carries a callback. That works because both sides are in the same
    -- Lua state; it is still the same one call shape, so scripts do not need a
    -- second way of talking to poggy_core just for registrations.
    ["inv.registerUsable"] = { side = "server", args = {"item", "fn"},                                    returns = "true" },

    -- ------------------------------------------------------------ weapons --
    ["weapon.add"]      = { side = "server", args = {"src", "weapon"}, optional = {"ammo", "components"},   returns = "true" },
    ["weapon.remove"]   = { side = "server", args = {"src", "weaponId"},                                   returns = "true" },
    ["weapon.get"]      = { side = "server", yields = true, args = {"src"},                                               returns = "array of weapons" },
    -- 0.11.0. Like inv.canCarry, for weapons: false comes with err 'no_space'.
    ["weapon.canCarry"] = { side = "server", yields = true, args = {"src"}, optional = {"qty", "weapon"},                returns = "boolean" },
    -- 0.16.0. Like inv.maxCarry, for weapons (of `weapon`, when given).
    ["weapon.maxCarry"] = { side = "server", yields = true, args = {"src"}, optional = {"weapon", "cap"},               returns = "number" },

    -- ----------------------------------------------------------- database --
    -- 0.12.0. Runs the CALLING resource's sql/install.sql, then any pending
    -- sql/migrations/NNN.sql. Tables, columns and indexes that already exist are
    -- skipped, so it is safe on every start. The bridge calls it once at start;
    -- scripts never need to. See server/sv_sql.lua.
    ["sql.install"]     = { side = "server", yields = true, args = {},                                               returns = "summary table" },
    -- 0.12.0. The same checks, but nothing is run: what sql.install would do.
    ["sql.check"]       = { side = "server", yields = true, args = {},                                               returns = "summary table" },

    -- ------------------------------------------------------------ storage --
    ["storage.register"]   = { side = "server", args = {"id"}, optional = {"opts"},                         returns = "true" },
    ["storage.registered"] = { side = "server", yields = true, args = {"id"},                                              returns = "boolean" },
    ["storage.open"]       = { side = "server", args = {"src", "id"},                                       returns = "true" },
    ["storage.close"]      = { side = "server", args = {"src", "id"},                                       returns = "true" },
    -- charId is optional in this contract but REQUIRED on VORP, which
    -- refuses a container add without a valid one. Pass it whenever you
    -- have it; a missing one shows up as framework_error, not bad_argument.
    ["storage.addItem"]    = { side = "server", yields = true, args = {"id", "item"}, optional = {"qty", "meta", "charId"},returns = "true" },
    ["storage.removeItem"] = { side = "server", yields = true, args = {"id", "item"}, optional = {"qty", "meta"},          returns = "true" },
    -- Reads straight from the database (a SELECT against
    -- character_inventories), not from memory, so it costs a query every call.
    -- A run where an add appeared not to land could not be reproduced once the
    -- container started empty -- the change was visible in 0ms -- so treat this
    -- as "cheap to get wrong under load", not as a known stale-read hazard.
    -- The self-test polls rather than reads once, which is cheap insurance.
    ["storage.items"]      = { side = "server", yields = true, args = {"id"},                                              returns = "array of items" },
    ["storage.capacity"]   = { side = "server", args = {"id"}, optional = {"slots", "maxWeight"},           returns = "true" },
    ["storage.unregister"] = { side = "server", args = {"id"},                                              returns = "true" },
    ["storage.delete"]     = { side = "server", yields = true, args = {"id"},                                              returns = "true" },
    ["storage.rawIds"]     = { side = "server", args = {"enabled"},                                         returns = "true" },
    -- 0.11.0. Weapons inside a container. not_found when it is not registered.
    ["storage.weapons"]    = { side = "server", yields = true, args = {"id"},                                              returns = "array of { id, name, label, serial, desc }" },

    -- ------------------------------------------------------ notifications --
    -- kind: 'info' | 'success' | 'error' | 'warning'
    ["notify"]          = { side = "both", args = {"text"}, optional = {"src", "kind", "duration"},         returns = "true" },
    ["notify.rich"]     = { side = "both", args = {},       optional = {"src", "title", "description", "kind", "duration", "dict", "icon", "color"},
                            returns = "true" },
    -- style: 'tip' | 'right' | 'objective' | 'top' | 'advanced' — keeps a
    -- migrated script looking exactly as it did before. Since 0.11.0 also
    -- 'location' | 'left' | 'leftRank' | 'basicTop' | 'center' | 'bottomRight'
    -- | 'fail' | 'dead' | 'update' | 'warning' (the rest of poggy_util's styles).
    -- An unknown style now fails with bad_argument instead of returning true.
    ["notify.styled"]   = { side = "both", args = {"style"}, optional = {"src", "text", "title", "subtitle", "duration", "dict", "icon", "color",
                                                                          "location", "audioRef", "audioName", "quality", "showQuality"},
                            returns = "true" },

    -- ------------------------------------------------------------- ui (0.14.0) --
    -- poggy_core draws these itself (ui/ in poggy_core), so no script needs
    -- vorp_menu or vorp_inputs and every framework gets the same screens.
    --
    -- menu.open: a list menu. items = array of { label, value, desc?, right?,
    -- disabled? }. Yields until the player picks an item (value = { value,
    -- index, item }) or closes it (false, 'closed'). Opening a menu while one
    -- is open replaces it. On the server, src names the player and the verb
    -- round-trips through poggy_core's callbacks.
    ["menu.open"]       = { side = "both", yields = true, args = {"title", "items"},
                            optional = {"src", "subtitle", "cursor", "closeText"},
                            returns = "{ value, index, item }" },
    ["menu.close"]      = { side = "both", args = {}, optional = {"src"},                              returns = "true" },
    -- input.text: one text (or number) box. Yields until submitted (the string,
    -- or a number when numeric = true) or cancelled (false, 'closed').
    ["input.text"]      = { side = "both", yields = true, args = {"title"},
                            optional = {"src", "placeholder", "default", "maxLength", "numeric", "submitText"},
                            returns = "string | number" },

    -- -------------------------------------------------------- permissions --
    -- ---------------------------------------------------------- callbacks --
    ["callback.register"] = { side = "server", args = {"name", "fn"},                                      returns = "true" },
    -- 0.11.0. Call a server callback from a client and wait for the answer.
    -- args is a list ({ n = count, ... } keeps nils); the value is the packed
    -- results { n = count, ... }. A timeout fails with 'timeout'.
    ["callback.await"]    = { side = "client", yields = true, args = {"name"}, optional = {"args"},        returns = "packed results { n, ... }" },

    ["perms.group"]     = { side = "server", args = {"src"},                                                returns = "group name" },
    ["perms.isAdmin"]   = { side = "server", args = {"src"},                                                returns = "boolean" },
    -- Every group poggy_core can see for this player, lower-cased and deduped.
    -- The character record and the framework's own user object do not always
    -- agree, and admin checks have historically consulted both. Doing that here
    -- keeps the duplication in one place instead of in every script.
    ["perms.groups"]    = { side = "server", args = {"src"},                                                returns = "array of group names" },

    -- --------------------------------------------------------------- bans --
    -- 0.19.0. A ban is on the ACCOUNT (every identifier, hashed), so it follows
    -- the player across characters. These verbs do not check who is asking:
    -- the calling script decides who may ban. `by` is the staff member's
    -- server id (or a name); duration is seconds, absent = permanent;
    -- category is "local" or "cheat". Lifting a ban keeps the row.
    ["ban.add"]            = { side = "server", yields = true, args = {"reason"}, optional = {"src", "identifiers", "name", "category", "duration", "evidence", "by", "byName", "source", "sourceRef"}, returns = "ban id" },
    ["ban.remove"]         = { side = "server", yields = true, args = {"id", "reason"}, optional = {"by", "byName"},          returns = "true" },
    ["ban.check"]          = { side = "server", args = {}, optional = {"src", "identifiers"},                                returns = "{ banned, ban?, network = { count, blocked } }" },
    ["ban.list"]           = { side = "server", yields = true, args = {}, optional = {"search", "active", "limit", "offset"}, returns = "array of bans, newest first" },
    ["player.kick"]        = { side = "server", args = {"src"}, optional = {"reason"},                                        returns = "true" },
    -- Every account identifier of an online player and the hash of each: what a
    -- report stores so the player can still be banned after they leave.
    ["player.identifiers"] = { side = "server", args = {"src"},                                                               returns = "{ name, identifiers, hashes }" },

    -- --------------------------------------------------------------- core --
    ["core.ready"]      = { side = "both", args = {},                                                       returns = "boolean" },
    ["core.framework"]  = { side = "both", args = {},                                                       returns = "{ id, label }" },
    ["core.has"]        = { side = "both", args = {"capability"},                                           returns = "boolean" },
    ["core.version"]    = { side = "both", args = {},                                                       returns = "version string" },
    ["core.caps"]       = { side = "both", args = {},                                                       returns = "capability map" },
    -- 0.13.0. The bridge announces its script once it is ready: id is the
    -- poggy_id in its fxmanifest.lua (its folder name when it has none). The
    -- folder is the calling resource, never taken from the payload, and an id
    -- the manifest does not declare is refused. See server/sv_identity.lua.
    ["core.register"]   = { side = "server", args = {}, optional = {"id", "version"},                        returns = "{ id, folder, version }" },
}

--- Verb names, sorted. Used by /poggycore verbs and by the startup self-check.
function PoggyCore.VerbNames()
    local out = {}
    for name in pairs(PoggyCore.Verbs) do out[#out + 1] = name end
    table.sort(out)
    return out
end

--- Is this verb callable on this side? Returns ok, reason.
function PoggyCore.VerbAllowed(name, side)
    local spec = PoggyCore.Verbs[name]
    if not spec then return false, "unknown_verb" end
    if spec.side ~= "both" and spec.side ~= side then return false, "wrong_side" end
    return true, nil
end

--- Check a payload against the verb's required keys. Returns ok, missingKey.
function PoggyCore.VerbCheck(name, payload)
    local spec = PoggyCore.Verbs[name]
    if not spec then return false, "unknown_verb" end
    payload = payload or {}
    for _, key in ipairs(spec.args) do
        if payload[key] == nil then return false, key end
    end
    return true, nil
end
