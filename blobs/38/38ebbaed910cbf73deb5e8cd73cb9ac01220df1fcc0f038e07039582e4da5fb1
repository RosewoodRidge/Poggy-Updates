--[[
    poggy_core — providers (0.17.0).

    A provider is a Poggy script that supplies something the framework does
    not: a bank with per-branch accounts, a treasury that collects levies. It
    registers a table of functions once at start (bank.register,
    treasury.register), and from then on poggy_core routes the matching verbs
    to it:

      * bank      money.get / money.add / money.remove / money.set with
                  currency = 'bank' go to the provider instead of the adapter,
                  and money.supports 'bank' answers true even on VORP, which
                  has no bank of its own.
      * treasury  every treasury.* verb.

    Without a provider nothing changes: the adapter answers for 'bank' where
    the framework has one (RSG, QBR) and refuses where it does not (VORP), and
    treasury.* refuses with 'unsupported', so a script that calls it defaults
    quietly and carries on.

    Contract for a provider function: the verb contract, ok, value, err. Every
    call goes through pcall; a provider that throws is reported once by name
    and the caller sees framework_error, never a crash.

    One provider per kind. A second resource registering while the first is
    still started is refused with a red line naming both, the same rule as
    script identity (sv_identity.lua). The owner is the calling resource from
    the dispatcher, never anything a payload could name, and stopping it drops
    its registration.
]]

PoggyCore = PoggyCore or {}

local Util = PoggyCore.Util
local Err  = PoggyCore.Err

local Providers = {}
PoggyCore.Providers = Providers

--- kind -> the functions a provider must supply.
Providers.KINDS = {
    bank     = { "get", "add", "remove", "set" },
    treasury = { "collect", "index", "rates", "state", "disburse", "balance", "report" },
}

--- kind -> { resource, fns, registeredAt }
local registry = {}
PoggyCore.ProviderRegistry = registry

local function started(folder)
    local ok, state = pcall(GetResourceState, folder)
    return ok and state == "started"
end

--- Register `resource` as the provider for `kind`. Returns ok, err.
function Providers.Register(resource, kind, fns)
    local need = Providers.KINDS[kind]
    if not need then return false, Err.BAD_ARG end
    if type(resource) ~= "string" or resource == "" then return false, Err.BAD_ARG end
    if type(fns) ~= "table" then
        Util.Warn("%s: %s.register needs a table of functions.", resource, kind)
        return false, Err.BAD_ARG
    end
    for _, name in ipairs(need) do
        if not PoggyCore.IsCallable(fns[name]) then
            Util.Warn("%s: %s.register is missing '%s'. Nothing was registered.", resource, kind, name)
            return false, Err.BAD_ARG
        end
    end

    local old = registry[kind]
    if old and old.resource ~= resource and started(old.resource) then
        Util.Error("%s wants to be the %s provider, but %s already is and is still running. Refused; stop one of them.",
            resource, kind, old.resource)
        return false, Err.DUPLICATE_ID
    end

    registry[kind] = { resource = resource, fns = fns, registeredAt = os.time() }
    if not old or old.resource ~= resource then
        Util.Log("^2%s is the %s provider.^7", resource, kind)
    end
    return true
end

--- Is there a provider for `kind`?
function Providers.Has(kind)
    return registry[kind] ~= nil
end

--- The resource providing `kind`, or nil.
function Providers.Owner(kind)
    local e = registry[kind]
    return e and e.resource or nil
end

--- Call one provider function. Returns the verb contract: ok, value, err.
--- 'unsupported' when there is no provider; a throw inside the provider is
--- reported once per function and comes back as framework_error.
local complained = {}
function Providers.Call(kind, name, ...)
    local e = registry[kind]
    if not e then return false, nil, Err.UNSUPPORTED end
    local fn = e.fns[name]
    if not PoggyCore.IsCallable(fn) then return false, nil, Err.NOT_IMPL end
    local ok, a, b, c = pcall(fn, ...)
    if not ok then
        local key = kind .. "." .. name
        if not complained[key] then
            complained[key] = true
            Util.Warn("%s (the %s provider) errored in '%s': %s", e.resource, kind, name, tostring(a))
        end
        return false, nil, Err.FRAMEWORK_ERR
    end
    -- A provider may answer with the plain shape (value) for reads or (true)
    -- for writes; normalise to the contract.
    if a == true or a == false then return a, b, c end
    if a == nil then return false, nil, b or Err.NOT_FOUND end
    return true, a, nil
end

--- Drop every registration a resource owns (it stopped).
function Providers.Forget(resource)
    for kind, e in pairs(registry) do
        if e.resource == resource then
            registry[kind] = nil
            complained[kind] = nil
            Util.Log("^9%s stopped; no %s provider until it is back.^7", resource, kind)
        end
    end
end

--- Every registration, for /poggycore status.
function Providers.List()
    local out = {}
    for kind, e in pairs(registry) do
        out[#out + 1] = { kind = kind, resource = e.resource, registeredAt = e.registeredAt }
    end
    table.sort(out, function(x, y) return x.kind < y.kind end)
    return out
end

AddEventHandler("onResourceStop", function(resource)
    Providers.Forget(resource)
end)
