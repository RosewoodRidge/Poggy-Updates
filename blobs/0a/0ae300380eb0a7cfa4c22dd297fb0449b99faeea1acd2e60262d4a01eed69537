# poggy_markets

Player-owned stores for RedM. Standalone, framework agnostic, and configured
from one file.

Players buy a storefront, stock it, price it, hire staff, and watch a dashboard
tell them what actually sells. Everything else — dynamic pricing, a commodities
exchange, simulated customers — is optional and switched off until you want it.

---

## What you get

**Stores that belong to players.** Buy a storefront from the config list or
found one anywhere with a deed item. Set prices per item, decide what the shop
buys from the public, hide back-stock from customers, upgrade storage, rename
it, move it, put it on the map or keep it off.

**A dashboard worth opening.** Today's takings, the week's, best sellers by
volume and by revenue, low-stock warnings, and a full transaction history with
filters. Owners can see which lines are dead and which ones sell out.

**Staff with real permissions.** Hire anyone online as a manager or an employee.
Employees restock and take deposits. Managers do everything except change the
shop's settings. The permission table is in the config and you can rewrite it.

**No second NPC file.** A store's clerk is part of the store. Delete the store
and the clerk goes with it. Clerks spawn when a player is nearby and are deleted
when they leave.

**Nothing to hand-write.** Stand where you want a store, run `/pmhere gunsmith
Valentine Gunsmith`, and paste the block it prints into your config.

---

## Requirements

- **poggy_core** 0.12.0 or newer
- **oxmysql**

poggy_core drives the framework and draws the notifications. There is no
setting to change.

---

## Install

1. Drop `poggy_markets` into your resources folder.
2. Add `ensure poggy_markets` to your server config.
3. Start the server and read the console.

The shop deed item (`shoptoken`) is added by `sql/install.sql`; its inventory
icon is `docs/shoptoken.png`, to copy into your inventory's image folder.

The database tables are created automatically when the script starts
(`sql/install.sql`). To import it yourself instead, set
`PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

On first boot the script prints which framework it found, how many stores it
loaded, and — importantly — any items in the catalog that do not exist in your
`items` table. **Expect that last list to be long.** Every server names items
differently. See the next section.

### Coming from syn_stores

There is nothing to migrate. poggy_markets uses the same `playershops` and
`shop_sales_log` tables, so your shops, ledgers, stock and sales history are
already where it expects them. `sql/install.sql` only adds the columns
your database might be missing and widens one enum — it drops nothing.

To keep your existing storefronts purchasable at the same places and prices,
give each store in `config/stores.lua` a `shopId` pointing at its `playershops`
row; the entries already in that file show the shape.

---

## Configure

Settings live in `config/`, and player-facing text in `translations.lua` at the
root. Most servers only ever touch the first two files.

| File | What it holds |
|---|---|
| `config.lua` | Module switches, player shops, tax, permissions, admin |
| `stores.lua` | Where the stores are, and what kind each one is |
| `catalog_default.lua` | What each kind of store buys and sells |
| `keys.lua` | Control hashes, so you can write `"G"` instead of a number |
| `pricing.lua`, `exchange.lua`, `ghostbuyer.lua` | Settings for the three optional modules |
| `../translations.lua` | Every player-facing string |

### Making the catalog yours

`catalog_default.lua` ships with plausible item names, and plausible is not the
same as correct. Open it, compare it against your `items` table, and fix the
names. The startup warning tells you exactly which ones are wrong.

A store type points at a catalog list, so editing one list changes every store
of that type at once. Nine general stores, one edit.

### Adding a store

Three lines:

```lua
{ type = "blacksmith", name = "Valentine Blacksmith",
  coords = vec3(-127.44, 632.21, 114.02), heading = 88.0 },
```

The type supplies the item lists, the map icon, and the clerk. Override any of
them per store if you want to; see the commented example at the bottom of
`config/stores.lua` for every available field.

### Three locations, not one

A shop involves up to three different points, and treating them as one is how
you end up with a clerk standing inside a customer, or behind glass you cannot
reach through:

| Field | What it is |
|---|---|
| `coords` | Where the **player** stands to get the prompt |
| `npc.coords` | Where the **clerk** stands — behind the counter, through a service window, across a stall. Absolute, not an offset. |
| `purchaseCoords` | Where the **FOR SALE** prompt appears |

Leave `npc.coords` out and the clerk spawns on top of the prompt, which is
wrong almost everywhere. `npc.offset` works as a relative alternative.

The FOR SALE point resolves as `purchaseCoords` → the `playershops` row's own
`coords` → the prompt. A store that declares a `shopId` may leave `coords` out
entirely and take its position from that row.

Blacksmiths, saloons and horse supply stores ship as *types* but without
locations, because the right spot depends on which interiors and map edits you
run. Use `/pmhere` to place them.

---

## Optional modules

All three are off by default. A server that never switches them on carries no
cost for them — the files return immediately.

### Dynamic pricing

Prices move with supply and demand. Every tracked item carries a multiplier:
players flooding the market push it down, scarcity lets it drift back up.
Quoted price is the config price times that multiplier, clamped to a band you
set.

You declare which items are tracked, grouped into markets. Items in the same
market influence each other, so dumping one kind of pelt softens prices on all
of them. Pelts and fish are set up as examples; ore is there commented out.

**This changes the economy of your whole server, not just one shop.** Start with
one market and watch it for a week.

`Config.Modules.dynamicPricing = true` · settings in `config/pricing.lua`

### Commodities exchange

A trading floor NPC where players speculate on those same markets. They hold
positions rather than goods — long if they think pelts are cheap, short if they
think a crash is coming — and settle in cash against the live price.

It is a period-appropriate commodities desk, not a stock market: the only things
traded are goods your players already handle.

Requires dynamic pricing. Short selling is off by default, because a short can
lose more than it staked.

`Config.Modules.exchange = true` · settings in `config/exchange.lua`

### Ghost buyers

On a quiet server a shop can sit untouched for days, which makes owning one feel
pointless. Ghost buyers simulate the trickle of custom a real store would get:
every so often an unseen customer buys a few items, stock goes down, the ledger
goes up.

The defaults are deliberately stingy — a few dollars a day, not a living. There
are price caps, stock minimums, and a per-sale ceiling so nobody stocks one
absurdly-priced item and farms it.

`Config.Modules.ghostBuyer = true` · settings in `config/ghostbuyer.lua`

---

## Jobs

**poggy_markets does not touch anyone's job unless you tell it to.**

This is deliberate. If another resource manages jobs, two scripts writing to the
same character means the last writer wins and neither behaves predictably. With
`Config.ShopJobs.enabled = false` (the default), shop access comes purely from
ownership and the staff list, and nothing fights.

Turn it on and buying a storefront that declares a `job` also sets the buyer's
job, and characters whose job matches a store get in without being on its staff
list. Only do this if poggy_markets is the thing that owns job assignment.

---

## Admin commands

| Command | What it does |
|---|---|
| `/pmhere <type> <name>` | Print a ready-to-paste store block for where you stand |
| `/pmdiag` | Report what loaded: framework, catalogs, modules, tracked markets |
| `/pmshops` | List every shop with its id, owner and ledger |
| `/pmreload` | Rebuild the store index and reload shops, no restart |
| `/pmgiveshop <serverId>` | Found a shop at a player's feet, no deed needed |
| `/pmdelshop <shopId>` | Delete a shop permanently |
| `/pmmoveshop <shopId> [x y z]` | Move a shop; no coordinates means where you stand |
| `/pmreposhop <shopId>` | Hide a shop from players, keeping all its data |
| `/pmunreposhop <shopId>` | Restore a repossessed shop |

Command names are configurable in `config.lua`; set one to `false` to remove it.

---

## Tax and repossession

Shops pay tax from their ledger on a schedule you set. A shop that cannot pay is
flagged as repossessed: it disappears from the map and the owner loses access.

**Nothing is deleted.** Stock, ledger and staff list all survive, and
`/pmunreposhop` puts it back. Set `Config.Tax.enabled = false` to switch the
whole thing off.

---

## Exports

```lua
-- Server
exports.poggy_markets:GetPrice(itemName, basePrice, "sell")  -- live price, or nil
exports.poggy_markets:GetMarket("pelts")                     -- every item in a market
exports.poggy_markets:IsTracked(itemName)                    -- boolean
```

---

## Frameworks

Every framework call goes straight to poggy_core; the few helpers that shape
its answers live in `server/player.lua` and `server/inventory.lua`. Nothing in
the script knows or cares which framework you run; a framework poggy_core
supports is a framework poggy_markets supports.

---

## Notes on behaviour

A few decisions that are easier to read here than to infer from the code:

- **The server owns every price.** A price arriving from the client is never
  used as the amount charged, only to notice the displayed price went stale.
- **Capacity is checked before money.** A full satchel is not a paid mistake.
- **Ledgers are debited before payouts.** A duplicated event cannot withdraw the
  same balance twice.
- **Roles are re-derived on every privileged call.** What the interface believes
  about a player's role is never trusted.
- **Owners cannot trade with their own shops.** It would move money in a circle
  and poison the analytics.
- **Webhook URLs must be Discord.** Otherwise the field is a way to make your
  server POST to an arbitrary host.
- **A missing item is skipped, not fatal.** Catalog entries that do not exist on
  your server are reported at startup and ignored at trade time.
- **A missing clerk model is skipped, not fatal.** A model name that does not
  exist is reported once and replaced with a generic clerk, rather than leaving
  an empty counter and a repeating console error.
