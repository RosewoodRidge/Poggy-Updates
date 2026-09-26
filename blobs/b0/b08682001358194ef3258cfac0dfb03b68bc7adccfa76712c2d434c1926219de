# Poggy Markets

Player-owned stores for RedM, with living prices and a frontier commodities
exchange.

Players buy a storefront or found their own shop, stock it, price it, hire
staff and read a dashboard that tells them what actually sells. NPC stores
across the map trade from item lists you control. Three optional modules add
supply-and-demand pricing, a commodities exchange and simulated customers.

Runs on **VORP**, **RSG** and **QBR** through poggy_core.

---

## What you get

**Stores that belong to players.** Buy a storefront from the config list, or
found one anywhere with a deed item. Set prices per item, choose what the shop
buys from the public, hide back-stock from customers, upgrade storage, rename
it, move it, and put it on the map or keep it off.

**A dashboard worth opening.** Today's takings, the week's, best sellers by
volume and by revenue, low-stock warnings, and a full transaction history with
filters.

**Staff with real permissions.** Hire anyone online as a manager or an
employee. Employees restock and take deposits. Managers do everything except
change the shop's settings. You can rewrite the permission table.

**No second NPC file.** A store's clerk is part of the store. Delete the store
and the clerk goes too. Clerks spawn when a player is near and are removed when
they leave.

**Nothing to hand-write.** Stand where you want a store, run
`/pmhere gunsmith Valentine Gunsmith`, and paste the block it prints.

**Optional economy modules.** Dynamic pricing, a commodities exchange and ghost
buyers. See [Optional modules](#optional-modules).

---

## Requirements

| | |
|---|---|
| **poggy_core** | 0.17.0 or newer. Install it first. 0.18.0 or newer adds the in-game settings editor. |
| **oxmysql** | For shops, sales and prices. |
| **Framework** | VORP, RSG or QBR. |

poggy_core talks to the framework and draws the notifications. There is
nothing to set.

---

## Install

1. Put `poggy_markets` in your resources folder.
2. Add `ensure poggy_markets` to `server.cfg`, **after** `poggy_core`.
3. Copy `docs/shoptoken.png` into your inventory's item image folder.
4. Start the server and read the console.

### Database

The tables are created when the script starts (`sql/install.sql`). You import
nothing. To import it yourself instead, set
`PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

`sql/install.sql` also adds the shop deed item (`shoptoken`). On a framework
without an `items` table (RSG), that row is skipped: add `shoptoken` to the
framework's item list by hand (RSG: `rsg-core/shared/items.lua`). The script
warns at start while it is missing.

### First start

The console says which framework it found, how many stores it loaded, and every
catalog item that does not exist on your server. **Expect that list to be
long.** Every server names items differently. See
[Make the catalog yours](#make-the-catalog-yours).

### Coming from syn_stores

There is nothing to migrate. poggy_markets uses the same `playershops` and
`shop_sales_log` tables, so shops, ledgers, stock and sales history are already
where it expects them. `sql/install.sql` only adds missing columns and widens
one enum. It drops nothing.

To keep your storefronts purchasable at the same places and prices, give each
store in `config/stores.lua` a `shopId` pointing at its `playershops` row.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/pmhere <type> <name>` | Admins | Prints a ready-to-paste store block for where you stand. In game only. |
| `/pmdiag` | Admins, console | Reports what loaded: framework, stores, catalog lists, modules, tracked markets. |
| `/pmshops` | Admins, console | Lists every shop with its ID, owner and ledger. |
| `/pmreload` | Admins, console | Rebuilds the store index and reloads shops from the database. Config changes still need a restart. |
| `/pmmanage [shopId]` | Admins | Opens any shop's manager. No ID means the shop you are in. |
| `/pmadmin` | Admins | The shop admin panel: find any shop, route to it, open its manager, transfer, repossess or restore, adjust the ledger, remove staff, set its job. See [Managing shops](#managing-shops). |
| `/pmgiveshop <serverId>` | Admins, console | Founds a shop at a player's feet. No deed needed. |
| `/pmdelshop <shopId>` | Admins, console | Deletes a shop permanently. |
| `/pmmoveshop <shopId> [x y z]` | Admins, console | Moves a shop. No coordinates means where you stand. |
| `/pmreposhop <shopId>` | Admins, console | Hides a shop from players and keeps all its data. |
| `/pmunreposhop <shopId>` | Admins, console | Restores a repossessed shop. |
| `/pmshopjob <shopId> <job\|none\|reset>` | Admins, console | Sets the job a shop gives its owner and staff. Optional: the Shop jobs panel in `/poggy` does the same (see [Shop jobs](#shop-jobs)). No job shows the current one. |
| `/openstore` | Everyone | Opens the "name your store" prompt where the deed cannot be used from the inventory. The deed is still used up. |

**Admins** are characters whose framework group is in `Config.Admin.groups`
(`admin`, `superadmin`, `god`, `owner` by default). `/pmadmin` also opens for
anyone granted the ACE in `Config.Admin.ace` (`poggy_markets.admin`).

The eight commands in `Config.Admin.commands` can be renamed, or removed by
setting them to `false`. `/pmdiag`, `/pmshops`, `/pmreload`, `/pmmanage` and
`/openstore` have fixed names.

---

## Configuration

**Every setting can be changed in game.** Type `/poggy`, open
**Poggy Markets**, change what you need, save, and restart the script. The
editor needs poggy_core 0.18.0 or newer. You can still edit the files by hand.

**Live data in /poggy.** Four tables edit the script's own data directly
and apply at once, with no restart. Every change is checked by the script,
logged to the console and the admin webhook with who made it, and shows in
the hub's history. They need the hub's edit permission.

| Panel | Tab | What you can do |
|---|---|---|
| **Shop jobs** | Player shops | Set or clear the job each shop gives its staff (see [Shop jobs](#shop-jobs)). |
| **Player shops** | Player shops | See every shop's owner, stock, ledger, staff and repossession. Transfer ownership (type the shop id to confirm), repossess or restore, adjust the ledger with a reason. Open a shop for its staff (remove someone) and its shelf stock (change a price). |
| **Market prices** | Economy & prices | Nudge an item's current price or reset it to base; open it for the last week's price points. Needs dynamic pricing. |
| **Exchange accounts** | Economy & prices | Correct a balance with a reason; open an account to close a position at the live price. Needs the exchange. |

Each uses the same code as the game: a transfer behaves like a storefront
purchase (shop jobs follow the new owner), removing staff like the manager's
Remove button, a closed position settles as the player's own close would.

| File | What it holds |
|---|---|
| `config/config.lua` | Module switches, player shops, tax, staff roles, admin, interface skin |
| `config/stores.lua` | Where the stores are, and the store types |
| `config/catalog_default.lua` | What each kind of store sells and buys |
| `config/keys.lua` | Key names, so you can write `"G"` instead of a control hash |
| `config/pricing.lua` | Dynamic pricing |
| `config/exchange.lua` | The commodities exchange |
| `config/ghostbuyer.lua` | Ghost buyers |
| `translations.lua` | Every message players see |

### Highlights

| Setting | Default | What it does |
|---|---|---|
| `Config.openKey` | `"G"` | Key that opens a store, shop or exchange desk. |
| `Config.purchaseKey` | `"B"` | Key that buys a storefront. It asks for a second press before paying. |
| `Config.currency` | `"$"` | Symbol shown before every price. |
| `Config.blacklistedItems` | empty | Items no shop may ever trade. |
| `Config.PlayerShops.creationItem` | `"shoptoken"` | Deed item that founds a shop. `false` stops player-founded shops. |
| `Config.PlayerShops.maxPerPlayer` | `2` | Shops one character may own. |
| `Config.Tax.amount` | `75` | Tax taken from each shop's ledger on collection day. |
| `Config.UI.skin` | `"leather"` | `"default"` dark theme, or `"leather"`. |

### Make the catalog yours

`catalog_default.lua` ships with plausible item names. Plausible is not the
same as correct. Compare it with your item list and fix the names. The start-up
warning names every wrong one.

A store type points at a catalog list, so one edit changes every store of that
type. Nine general stores, one edit.

### Adding a store

```lua
{ type = "blacksmith", name = "Valentine Blacksmith",
  coords = vector3(-127.44, 632.21, 114.02), heading = 88.0 },
```

The type supplies the item lists, the map blip and the clerk. Any of them can be
overridden per store. The commented example at the bottom of
`config/stores.lua` shows every field.

Blacksmiths, saloons and horse supply stores ship as types without locations,
because the right spot depends on your interiors and map edits. Use `/pmhere`
to place them.

### Three locations, not one

| Field | What it is |
|---|---|
| `coords` | Where the **player** stands to get the prompt. |
| `npc.coords` | Where the **clerk** stands: behind the counter, through a window, across a stall. Absolute, not an offset. |
| `purchaseCoords` | Where the **FOR SALE** prompt appears. |

Leave `npc.coords` out and the clerk spawns on the prompt, which is wrong
almost everywhere. `npc.offset` works as a relative alternative.

The FOR SALE point resolves as `purchaseCoords`, then the `playershops` row's
own coordinates, then the prompt. A store with a `shopId` may leave `coords`
out and take its position from that row.

### Selling a storefront

A storefront is for sale only when it has `purchasable = true`, a `shopId`
pointing at a `playershops` row, and that row is unowned or repossessed. None
of the shipped stores are for sale: which ones to sell, and for how much,
depends on your own rows.

`purchaseJobs = { "valstables" }` sells it only to characters holding one of
those jobs: the job they wear, or any job on their poggy_multijob list. Anyone
else who tries is told which job they need. Admins are never refused.

### Interface skin

`Config.UI.skin` sets the look of the shop, the store manager and the exchange.
`"default"` is the dark theme. `"leather"` is stitched leather and parchment,
like the game's own satchel.

A skin is a stylesheet plus images in `ui/css`: `skin-<name>.css` and
`skin-<name>-*.png`. To make your own, copy `skin-leather.css` under a new
name, change the image names in it, and set `skin` to that name. Every image is
optional; a missing one leaves that piece in plain colour. Image sizes are
listed at the top of `skin-leather.css`.

---

## Optional modules

All three ship switched **on** in `Config.Modules`. Turn off any you do not
want. A module that is off costs nothing: its files return straight away.

### Dynamic pricing

Prices move with supply and demand. Every tracked item has a multiplier.
Players flooding the market push it down; scarcity lets it recover. The quoted
price is the catalog price times that multiplier, held inside a band you set.

You choose which items are tracked, grouped into markets. Items in one market
move together, so dumping one kind of pelt softens every pelt. Pelts and fish
ship as examples; ore is there, commented out.

**This changes the economy of your whole server.** Start with one market and
watch it for a week.

Settings: `config/pricing.lua`.

### Commodities exchange

Trading desks where players speculate on those same markets. They hold
positions, not goods: long if they think pelts are cheap, short if they expect
a crash. Positions settle in cash against the live price.

It needs dynamic pricing. Short selling ships **off**, because a short can lose
more than it staked.

Settings: `config/exchange.lua`.

### Ghost buyers

On a quiet server a shop can sit untouched for days. Ghost buyers are unseen
customers who now and then buy a few items: stock goes down, the ledger goes up.

The defaults are stingy on purpose: a few dollars a day. Price caps, stock
minimums and a per-sale ceiling stop anyone farming one absurdly priced item.

Settings: `config/ghostbuyer.lua`.

---

## Staff, jobs and permissions

**Staff roles.** `Config.Roles` sets what an owner, a manager and an employee
can do in the manager. Admins can do everything in every shop.

**poggy_markets does not touch anyone's job unless you tell it to.** With
`Config.ShopJobs.enabled = false` (the default), shop access comes only from
ownership and the staff list, and nothing fights another jobs script.

### Shop jobs

Turn on `Config.ShopJobs.enabled` and a shop that has a job gives it to its
owner and everyone the owner hires, so job-locked content elsewhere (crafting
recipes, doors, boss menus) works for shop staff. No admin has to hand out
jobs.

**Only server staff set a shop's job.** No player can, owner included. It
comes from, first match wins:

1. Server staff, saved in the database (`playershops.job`), so it survives
   restarts:
   - **The main way:** `/poggy` → **Poggy Markets** (Player shops tab) or
     **Poggy Multijob** (Shop jobs tab) → **Shop jobs**. One table lists every
     shop (buyable storefronts and player-founded shops) with its owner, job,
     where the job came from and staff count; a shop with staff but no job is
     flagged. Edit the **Shop job** cell and it applies at once: staff get or
     lose the job straight away, no restart. Clearing the cell means no job.
     Needs the hub's edit permission.
   - **Optional, the command** (admins and the server console):
     `/pmshopjob <shopId> <job>` sets it, `none` means no job even if the
     config names one, `reset` goes back to the config, and with no job it
     shows the current one.
2. The store's `job` field in `config/stores.lua`, the file default.

A buyable storefront with no `playershops` row yet (no `shopId`, never sold)
is listed in the panel but uses its `stores.lua` job until it has a row.

The staff tab shows the job to everyone who can open it (owner, managers,
admins) as a locked title: *Shop job: General Store 🔒 Set by staff*, with the
tooltip "Only server staff can change this, in /poggy (Shop jobs) or with
/pmshopjob 7." A shop without
one shows *No shop job — ask server staff to set one.* There is nothing to
click. `/pmshops` lists each shop's job too.

**Grades.** Each role has a job grade:

```lua
Config.ShopJobs.grades = { employee = 1, manager = 2, owner = 3 }
```

The owner gets `grades.owner`, unless the older `ownerGrade` setting holds a
number (it is `false` in a new config and still honoured in an old one).
Promoting or demoting someone in the staff tab moves their grade with them.

**When it happens.**

| Event | Result |
|---|---|
| Hire | Gets the job at their role's grade |
| Promote / demote (staff tab role + Save) | Grade changes |
| Let go | Loses the job |
| Storefront bought | New owner gets it; the previous owner loses it |
| Shop repossessed or deleted | Owner and staff lose it (restored: they get it back) |
| Shop's job changed | Old job taken, new job given |

**With poggy_multijob** (1.7.3 or newer) running, the job is *added to the
character's job list* and their current job is left alone; they switch with
`/multijob`. It works for offline characters too. Without poggy_multijob the
job is set directly with poggy_core (it replaces the current job), online
players only; a firing that happens while someone is offline is applied when
they next load in, moving them to `fallbackJob` if they still wear it.

**poggy_markets only takes away jobs it gave.** Every job it hands out is
recorded in the `poggy_markets_job_grants` table. Someone who already had the
job before being hired keeps it, at their own grade, when they leave. Someone
on the staff of two shops with the same job keeps it while either employs
them, at the higher grade.

**It checks itself.** On every start, on `/pmreload`, when poggy_multijob
starts, and for one character whenever they load in, every shop's owner and
staff are compared with what has been given: missing jobs are given, wrong
grades fixed, and jobs of people no longer on the staff taken away. The start
prints one line:

```
[poggy_markets] shop jobs (start, via poggy_multijob): 4 shop(s) with a job, 11 holder(s) checked; 0 given, 1 grade(s) fixed, 2 taken away, 0 waiting, 0 failed
```

Running it again changes nothing. Turning `enabled` off freezes everything as
it is: nothing is given or taken while it is off.

**Access by job.** With `grantAccessByJob`, a character whose job matches the
shop's job gets in without being hired, by grade: `grades.owner` and up as
owner, `grades.manager` as manager, `grades.employee` as employee. Because
only staff set shop jobs, it reaches exactly as far as staff make it: give two
shops the same job and that job's holders (including the other shop's staff)
get into both, owner-grade holders with ledger access. Give each shop its own
job, or set `grantAccessByJob = false`, if that is not what you want.

**Job names** must be real jobs. On RSG and QBR the job has to exist in the
core's shared jobs (an unknown name is refused); on VORP jobs are free text.

---

## Managing shops

**Owners hand shops over.** In the manager's **Settings** tab the owner picks a
player standing near them, types the shop's name to confirm, and the other
player gets an Accept / Decline box. Stock, ledger and staff go with the shop.
A storefront reserved for a job (`purchaseJobs`) only goes to someone holding
that job, and nobody gets more shops than `Config.PlayerShops.maxPerPlayer`.
Settings: `Config.Transfers` (`enabled`, `distance`, `offerSeconds`).

**`/pmadmin` is the shop admin panel** for staff who do not use `/poggy`: every
shop in one table, searched by shop ID, name, owner or job. Pick one to set a
GPS route to it, open its manager, transfer it (search every character,
online or not, by name or id, then type the shop ID to confirm; the job and
shop limits do not apply), repossess or restore it, set its ledger balance
with a reason, remove staff or set its job. The `/poggy` Player shops panel runs the same code, and every
change is logged to the console and the admin webhook with who made it.

## Tax and repossession

Shops pay tax from their ledger on a schedule you set, by the server's clock.
A shop that cannot pay is repossessed: it leaves the map and the owner loses
access.

**Nothing is deleted.** Stock, ledger and staff list all survive, and
`/pmunreposhop` puts it back. `Config.Tax.enabled = false` switches the whole
thing off.

---

## The treasury

With Poggy Banking's treasury module running, poggy_markets joins the server's
economy:

- the shop tax is paid to the treasury instead of being deleted;
- every sale and purchase, every ghost sale and each market's average price
  multiplier are reported to it, so it can learn what normal trade looks like;
- the treasury's **price index** scales every config store's prices, buying
  and selling. Player shops set their own prices and are never touched.

Without a treasury nothing changes: the index is 1.0 and the reports go
nowhere.

---

## Exports

Available while dynamic pricing is on.

```lua
-- Server
exports.poggy_markets:GetPrice(itemName, basePrice, "sell")  -- live price, or nil
exports.poggy_markets:GetMarket("pelts")                     -- every item in a market
exports.poggy_markets:IsTracked(itemName)                    -- boolean
```

---

## Troubleshooting

| Problem | What to check |
|---|---|
| A store shows no items | Run `/pmdiag`. The store type's list name must match a catalog list, and the item names must exist on your server. |
| "Item not available on this server" | The item name is not in your framework's item list. Fix it in the catalog. |
| Nobody can found a shop | The deed item is missing from your item list (the console says so), or `Config.PlayerShops.enabled` is off. |
| The FOR SALE prompt never appears | The store needs `purchasable = true`, a `shopId`, and an unowned or repossessed `playershops` row. |
| The clerk stands on the prompt | Give the store an `npc.coords` behind the counter. |
| A clerk model is wrong | A missing model is replaced by `Config.fallbackClerkModel` and reported once in the client console. |
| The exchange is empty | `Config.Exchange.tradableMarkets` must name markets in `Config.Pricing.markets`, and those markets must match catalog items. `/pmdiag` shows both. |
| A config change did nothing | Restart `poggy_markets`. `/pmreload` reloads shops, not config files. |

---

## Notes on behaviour

- **The server owns every price.** A price from the client is never charged; it
  only shows that the displayed price went stale.
- **Capacity is checked before money.** A full satchel is not a paid mistake.
- **Ledgers are debited before payouts.** A duplicated event cannot withdraw
  the same balance twice.
- **Roles are re-checked on every privileged action.** What the interface
  believes about a player's role is never trusted.
- **Owners cannot trade with their own shops.** It would move money in a circle
  and spoil the analytics.
- **Webhook URLs must be Discord.** Otherwise the field could make your server
  post to any host.
- **A missing item is skipped, not fatal.** It is reported at start and ignored
  at trade time.
- **A missing clerk model is skipped, not fatal.** It is reported once and
  replaced with a generic clerk.

---

## Changelog

- **1.6.1** — The manager's ledger shows money in as well as money out: every entry is signed and coloured, and ghost sales count as income (on the dashboard, in revenue, the chart and the best sellers). The ledger filter gains Ghost Sales and Tax, and its date range now works.
- **1.6.0** — Shop owners can hand their shop to a player standing near them from the manager's Settings tab; the other player has to accept, and a storefront reserved for a job only goes to someone with that job (`Config.Transfers`). New `/pmadmin` panel for admins who do not use `/poggy`: every shop in a searchable table, with route, open manager, transfer to any character (searched by name or id, online or not), repossess / restore, the ledger balance edited in place with a reason, staff removal and shop job. Opens for `Config.Admin.groups` or the ACE `poggy_markets.admin`. The `/poggy` Player shops panel now uses the same code, and a new owner who was on the shop's staff is taken off it.
- **1.5.0** — A storefront can be reserved for the holders of certain jobs: `purchaseJobs = { "valstables" }` in `config/stores.lua` (**Buyers' jobs** in `/poggy`). The job the buyer is wearing counts, and with poggy_multijob so does every job on their list, so a stable owner does not have to switch jobs to buy the stable. Anyone else is told which job they need and nothing is charged. Admins are never refused. Left out, a storefront sells to anyone, as before. Two fixes to the shop prompt: when two shops are within reach, the nearest one owns the prompt (before, one keypress opened both), and a storefront with nothing to trade shows only its FOR SALE prompt instead of an "open" prompt that led to an empty shop. A player shop's blip sprite may also be a hash written as a number (`playershops.blipsprite = "623069873"`), for sprites that have no known name.
- **1.4.1** — A refused visit no longer traps the player. Pressing the prompt at a shop someone else is managing (or one the job lock keeps you out of) left you frozen with a cursor and no panel to close: the client had taken focus before the server answered, and the refusal only sent a notification. Every refusal now releases the player, and the client lets go by itself if no panel arrives within eight seconds.
- **1.4.0** — Shop jobs (`Config.ShopJobs`, off by default). A shop with a job gives it to its owner and staff at the grade for their role (`grades = { employee = 1, manager = 2, owner = 3 }`) and takes it back when they are let go or the shop is sold, repossessed or deleted; promotions and demotions move the grade. With poggy_multijob 1.7.3 running the job is added to the character's job list (online or offline) instead of replacing their current job. Only server staff set a shop's job, saved in the database: the new **Shop jobs** panel in `/poggy` (on the Poggy Markets and Poggy Multijob pages, applies at once, exported as `HubPanel`), the optional `/pmshopjob <shopId> <job|none|reset>` (admins and console), or the store's `job` in `config/stores.lua`; the staff tab shows it to everyone as a locked title. Grants are recorded in the new `poggy_markets_job_grants` table, only jobs poggy_markets gave are ever taken, and every start and every character load checks and repairs them (one summary line in the console). Access by job now uses the configured grades. Three more `/poggy` data panels: **Player shops** (owner, stock, ledger, staff, repossession; transfer, repossess/restore, ledger adjustment, staff removal, shelf prices), **Market prices** (nudge or reset a live price, price history) and **Exchange accounts** (balance correction, close a position). The staff tab keeps RSG/QBR character ids as text, so hiring, role changes and letting go work there.
- **1.3.1** — Ready for the Poggy Hub: every setting, store, catalog list and message can be edited in game with `/poggy` (poggy_core 0.18.0), with help pages for adding stores, the catalog and the economy modules. Store types name their blip (`blipSprite = "shop"`) instead of pointing at `Config.BlipSprites.shop`; a sprite hash still works. The config files and `/pmhere` now write `vector3(...)`. No change in game.
- **1.3.0** — Interface skins. `Config.UI.skin = "leather"` restyles the shop, store manager, exchange and naming prompt: a stitched leather frame and header flap, satchel-style item slots with the stock count in the corner, parchment tooltips and a pocket of coins and letters under the shop. `"default"` (the shipped setting) looks exactly as before. A server can add its own skin as `ui/css/skin-<name>.css` without touching any code. The Max button now offers what you can actually buy: the store's stock, capped by how many more you can carry, as poggy_core reports it for your framework (VORP's item limit, RSG's and QBR's weight and slots). It used to offer 999 at every unlimited store. Needs poggy_core 0.16.0.
- **1.2.2** — Buying a weapon on RSG works: poggy_core's RSG adapter now maps the catalog's `WEAPON_...` names to rsg-core's lower-case weapon items (needs poggy_core 0.14.0). When a hand-over is refused the console names the item and the reason, and the player is told "not available on this server" for an item the framework does not know, instead of a vague "could not be completed". The start-up item check now covers weapons on frameworks that list them as items (RSG), and shelving a gun from your loadout matches its name regardless of case.
- **1.2.1** — Runs on frameworks without an `items` table (RSG): the shop deed row in `sql/install.sql` moved to the end of the file and is skipped there instead of stopping the install (which left `poggy_markets_prices` and the other tables uncreated); the start-up check now also names the deed item when your framework's item list lacks it.
