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
    item_unavailable = "That item is not available on this server.",
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

    -- ── Shop jobs (Config.ShopJobs) ────────────────────────────────────────
    shopjob_given    = "You have been given the job: %s.",
    shopjob_taken    = "You no longer have the job: %s.",
    shopjob_unknown  = "There is no job called %s on this server.",
    shopjob_bad_name = "A job name may only use letters, digits, _ - and . (at most 64).",
    shopjob_saved_off = "Saved, but shop jobs are off (Config.ShopJobs.enabled), so nobody is given it yet.",
    shopjob_admin_show = "Shop #%s (%s): job %s (%s).",
    shopjob_from_config = "From stores.lua",
    shopjob_from_none   = "None",

    -- Poggy hub (/poggy), the "Shop jobs" panel.  Seen by server staff.
    hub_kind_storefront = "Buyable storefront",
    hub_kind_config     = "Storefront",
    hub_kind_player     = "Player shop",
    hub_note_nojob      = "Has staff but no shop job: they are given no job.",
    hub_note_repo       = "Repossessed: its owner and staff hold no shop job until it is restored.",
    hub_note_norow      = "No playershops row yet (no shopId, or never sold), so its job cannot be changed here; it uses the job in stores.lua.",
    hub_only_job        = "Only the shop job can be changed here.",
    hub_shopjobs_loading = "poggy_markets is still loading its shops. Try again in a moment.",
    hub_shopjobs_off    = "Shop jobs are off (Config.ShopJobs.enabled): jobs set here are saved, but nobody is given them until it is on.",
    hub_shopjobs_no_multijob = "poggy_multijob is not running: shop jobs are set directly (replacing the current job), online players only.",

    -- Poggy hub (/poggy): player shops, prices and exchange panels.
    hub_bad_charid      = "Enter a character id (VORP: the number; RSG/QBR: the citizenid).",
    hub_same_owner      = "That character already owns this shop.",
    hub_no_character    = "There is no character with the id %s.",
    hub_transferred     = "%s now belongs to %s.",
    hub_already_repo    = "That shop is already repossessed.",
    hub_not_repo        = "That shop is not repossessed.",
    hub_repossessed     = "%s has been repossessed.",
    hub_restored        = "%s has been restored.",
    hub_bad_amount      = "Enter an amount that is not zero.",
    hub_need_reason     = "Give a reason; it goes in the log.",
    hub_ledger_negative = "The ledger holds only %s; it cannot go below zero.",
    hub_ledger_done     = "Done. The ledger now holds %s.",
    hub_not_staff       = "That character is not on this shop's staff any more.",
    hub_only_price      = "Only the price can be changed here.",
    hub_bad_price       = "Enter a price of zero or more.",
    hub_no_item         = "That item is not there any more. Refresh the panel.",
    hub_price_done      = "%s now costs %s.",
    hub_price_clamped   = "%s is kept within the floor and ceiling in config/pricing.lua: it now costs %s.",
    hub_no_base         = "This item has no base price, so its price cannot be set.",
    hub_pricing_off     = "Dynamic pricing is off (Config.Modules.dynamicPricing), so there are no live prices.",
    hub_pricing_empty   = "No items are tracked. Check the markets in config/pricing.lua.",
    hub_history_note    = "Price points for %s over the last 7 days, newest first.",
    hub_exchange_off    = "The commodities exchange is off (Config.Modules.exchange, which also needs dynamic pricing).",
    hub_exchange_negative = "Negative balance: a short position lost more than its collateral.",
    hub_balance_done    = "Done. %s's exchange balance is now %s.",
    hub_positions_note  = "Open positions of %s. Closing one settles it at the live price, as the player would.",
    hub_no_position     = "That position is not open any more. Refresh the panel.",
    hub_no_mark         = "That item has no live price right now, so the position cannot be settled.",
    hub_position_closed = "Closed %sx %s, profit/loss %s.",

    -- Staff tab: the shop's job, locked (only server staff change it)
    ui_shopjob_locked  = "Shop job: %s",
    ui_shopjob_set_by  = "Set by staff",
    ui_shopjob_tooltip = "Only server staff can change this, in /poggy (Shop jobs) or with %s.",
    ui_shopjob_none    = "No shop job — ask server staff to set one.",
    ui_shopjob_grade   = "grade %s",

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

