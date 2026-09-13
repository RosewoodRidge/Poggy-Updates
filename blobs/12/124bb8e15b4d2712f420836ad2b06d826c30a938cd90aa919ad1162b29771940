--[[===========================================================================
    poggy_markets · translations.lua
    ---------------------------------------------------------------------------
    Every player-facing string.  Translate this file and you have translated
    the script.

    %s markers are filled in order by whatever the caller passes.  Keep them in
    place -- reorder them for your language using positional markers if needed:
        bought = "%1$s x %2$s gekauft"
===========================================================================]]--

PM = PM or {}
PM.Locale = {

    -- ── Interaction prompts ────────────────────────────────────────────────
    open_store       = "Open Store",
    open_manager     = "Manage Store",
    buy_store        = "Purchase Storefront",

    -- ── Trading ────────────────────────────────────────────────────────────
    bought           = "Bought %sx %s for %s",
    sold             = "Sold %sx %s for %s",
    cannot_afford    = "You cannot afford that.",
    inventory_full   = "You cannot carry any more of that.",
    not_enough_stock = "The store does not have that many in stock.",
    not_enough_items = "You do not have that many.",
    shop_not_buying  = "This shop is not buying that.",
    shop_cannot_afford = "This shop does not have enough money in its ledger.",
    nothing_to_sell  = "You have nothing this shop wants.",
    cannot_buy_own   = "You cannot buy from your own shop.",
    cannot_sell_own  = "You cannot sell to your own shop.",
    transaction_failed = "That transaction could not be completed.",
    store_joblocked  = "You are not permitted in this store.",

    -- ── Shop ownership ─────────────────────────────────────────────────────
    shop_created     = "Your shop has been opened.",
    shop_purchased   = "You are now the owner of %s.",
    shop_sold        = "You have sold %s.",
    shop_deleted     = "That shop has been deleted.",
    shop_renamed     = "Shop renamed to %s.",
    shop_moved       = "Shop relocated.",
    shop_in_use      = "Someone else is using that shop right now.",
    shop_too_close   = "That is too close to another shop.",
    shop_max_reached = "You already own the maximum number of shops.",
    shop_no_deed     = "You need a shop deed to do that.",
    shop_invalid_name = "That is not a valid shop name.",
    shop_not_for_sale = "That storefront is not for sale.",
    shop_already_owned = "That storefront already has an owner.",

    -- ── Ledger ─────────────────────────────────────────────────────────────
    ledger_deposited = "Deposited %s into the shop ledger.",
    ledger_withdrew  = "Withdrew %s from the shop ledger.",
    ledger_short     = "The ledger does not hold that much.",
    ledger_invalid   = "Enter a valid amount.",

    -- ── Storage ────────────────────────────────────────────────────────────
    storage_full     = "The shop has no room for that.",
    weapons_shelf_only = "Guns can only go on the shelves, not into storage.",
    slots_upgraded   = "Storage upgraded to %s slots.",

    -- ── Staff ──────────────────────────────────────────────────────────────
    staff_hired      = "%s has been hired as %s.",
    staff_fired      = "%s has been let go.",
    staff_not_found  = "No such character is nearby.",
    staff_already    = "That person already works here.",

    -- ── Tax ────────────────────────────────────────────────────────────────
    tax_paid         = "%s paid %s in tax.",
    tax_repossessed  = "%s could not pay its tax and has been repossessed.",

    -- ── Permissions ────────────────────────────────────────────────────────
    no_permission    = "You do not have permission to do that.",

    -- ── Admin ──────────────────────────────────────────────────────────────
    admin_only       = "That command is for administrators.",
    admin_usage      = "Usage: %s",
    admin_no_shop    = "No shop with that ID.",
    admin_done       = "Done.",
}

--- Look up a locale string and fill in its markers.
--- Returns the key itself when a string is missing, so a typo shows up in game
--- as the key rather than as an empty message.
---@param key string
---@param ... any
---@return string
function PM.L(key, ...)
    local str = PM.Locale[key]
    if not str then return tostring(key) end
    if select("#", ...) == 0 then return str end

    -- tostring every argument so numbers and nils cannot break string.format.
    local args = {}
    for i = 1, select("#", ...) do
        args[i] = tostring((select(i, ...)))
    end
    local ok, formatted = pcall(string.format, str, table.unpack(args))
    return ok and formatted or str
end
