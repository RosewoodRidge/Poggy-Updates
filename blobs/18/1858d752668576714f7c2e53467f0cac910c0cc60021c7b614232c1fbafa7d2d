# Poggy Auction House

A player market for RedM.

Players list goods at auction houses around the map. Others bid on them or buy them out. Everything a player wins, earns or gets back lands in a mailbox.

The auction house also has a **supply catalogue** for ordering raw goods, and a **want-request board** for posting buy orders.

## Requirements

| Resource | Needed |
|---|---|
| `poggy_core` 0.17.0 or newer | Yes. It runs the framework (VORP or RSG). |
| `oxmysql` | Yes |
| `poggy_markets` | Optional. Lets catalogue orders go straight into a shop's storage. |
| Poggy Banking treasury | Optional. See [The treasury](#the-treasury). |

## Installation

1. Put `poggy_auction` in your resources folder.
2. Add `ensure poggy_auction` to `server.cfg`. Put it below `oxmysql` and `poggy_core`, and below `poggy_markets` if you use it.
3. Start the server.

The database tables are created for you when the script starts (`sql/install.sql`). You do not need to import anything.

To manage the tables yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`, then import `sql/install.sql`.

## What players get

| Tab | What it does |
|---|---|
| **Browse** | Every live auction. Filter by category, search and sort. Bid, or buy it now when the seller set a buyout price. |
| **Sell** | List an item from your satchel. Set a starting price, an optional buyout, and how long it runs. Longer listings take a bigger deposit. A sales tax comes off the sale. |
| **My Auctions** / **My Bids** | Your own listings, and whether you are winning or outbid. |
| **Mailbox** | Winnings, sale money, outbid refunds, returned items and deliveries. Collect one at a time, or all at once. |
| **Order Catalogue** | Order raw goods from lumberjacking, mining, farming, hunting, fishing and crafted suppliers. Prices shift a little each time the catalogue opens. Each order goes through Processing, In Transit and Out for Delivery, then arrives in the mailbox or a Poggy Markets shop. |
| **Want Requests** | Post a buy order. The money is held until the request ends. Other players fill it in full or in part and get paid as they deliver. Anything unused comes back at the end. |

Players open the auction house by walking up to the auctioneer and pressing **G**.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/auctionrefresh` | Admins, console | Settles every listing whose time has run out, now. |
| `/auctionpurge` | Admins, console | Cancels every active listing. The items in them are **not** returned. To cancel one listing and return its item, use **Live auctions** in `/poggy`. |
| `/auctiondiscord` | Admins, console | Sends a test message to the Discord webhook. |

**Admins** here means characters in the `admin`, `superadmin` or `god` group. This list is fixed in the script. Every command also works from the server console.

## Staff tools in /poggy

Server staff can see and manage live auction house data in **`/poggy`** → **Auction House**. Changes apply at once, with no restart.

| Table | Tab | Actions |
|---|---|---|
| **Live auctions** | Auctions | Cancel a listing and return the item, extend it by some hours, open it to see its bids and remove one |
| **Shipment orders** | Supply catalogue | Cancel and refund (while processing), deliver now |
| **Want requests** | Want requests | Open it to see who filled it, cancel and refund |

Items and money go where the game would send them, which is the player's mailbox. Every change is logged in the hub's History and the server console. See the help page **Managing live auctions, orders and requests** (`docs/help/live-data.md`). This needs a poggy_core with data panel actions (the settings hub).

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy Hub). You can also edit `config.lua` and `translations.lua` by hand. Restart the script after a change.

| Setting | What it controls |
|---|---|
| `Config.Locations` | Where the auction houses are, the auctioneer NPC and the map blip. |
| `Config.Interaction` | Prompt distance, NPC spawn distance, open key. |
| `Config.Auction` | Durations and deposits, sales tax, price limits, bid steps, listing and mailbox limits. |
| `Config.Categories` | The browse categories, and which item groups land in each. |
| `Config.CategoryIcons`, `Config.ItemCategories` | Icons for other groups, and per-item category overrides. |
| `Config.BlacklistedItems` | Items that can never be listed. |
| `Config.Shipment` | Shipping fee, order limits, how long each delivery stage takes, price swing, shop delivery. |
| `Config.ShipmentCompanies` | The suppliers shown in the catalogue sidebar. |
| `Config.ShipmentCatalog` | What can be ordered, with prices and descriptions. |
| `Config.Requests` | Want-request tax, limits and durations. |
| `Config.Discord` | Webhook logging, per event. |
| `translations.lua` | Every message and every word in the auction house window. |

### Categories

The browse categories come from poggy_core's item list. Each item has a group:

- On **VORP** it is the item's `item_group` id (stock ids 1 to 11).
- On **RSG** it is the item's `category` in `rsg-core/shared/items.lua`.

`Config.Categories` says which category each group belongs to. The stock VORP groups are already set up. A group that is not listed still shows, as a category of its own. Add a row to give it a proper label and icon.

### The catalogue

A catalogue item only shows when its item name exists in your items table. Items your server does not have are left out on their own. Several of the shipped items come from other scripts; `config.lua` lists them.

## Poggy Markets

When `poggy_markets` is running, the order form shows a **Deliver to Shop ID** box. A shop's ID is shown at the top of its manager.

- Owners, and staff whose role lets them deposit items, can send an order straight into that shop's storage.
- If the shop is full when the order arrives, it goes to the mailbox instead.
- Without `poggy_markets`, the box is hidden and every order goes to the mailbox.
- Set `Config.Shipment.ShopDelivery = false` to turn shop delivery off.

## The treasury

With Poggy Banking's treasury module running, the auction house joins the server's economy:

- The sales tax, want-request tax, listing deposit and shipping fee go to the treasury instead of disappearing.
- Sales, listings, orders and fills are reported, so the treasury can learn what normal trade looks like.
- The treasury's price index scales catalogue prices.
- Its policy may add a few points to the sales tax.

Without a treasury nothing changes: fees disappear as before and prices are not scaled. There is no setting for this.

## Troubleshooting

**The auction house opens with no categories, or says a table does not exist.**
Update `poggy_core`. Categories come from poggy_core's item list, not from VORP's `item_group` table.

**A catalogue item is missing.**
Its item name is not in your items table. Add the item, or change the entry's `name` to an item you have.

**Discord messages do not arrive.**
Turn on `Config.Discord.Enabled` and paste a real webhook URL. Nothing is sent while it still says `YOUR DISCORD WEBHOOK HERE`. Test with `/auctiondiscord` (it posts as a "Listing" event, so keep that event on).

**A listing's timer reached zero but it has not ended.**
Listings are settled every `Config.Auction.ExpirationCheckInterval` seconds (60 by default). Use `/auctionrefresh` to settle them now.

**Players cannot list anything.**
Check they are under `MaxActiveListings`, and under `MaxMailboxItems` uncollected mailbox entries. Ask them to collect their mailbox.

## Changelog

- **1.3.2** — Staff tables in `/poggy`: live auctions (cancel and return, extend, see and remove bids), shipment orders (cancel and refund, deliver now) and want requests (see fills, cancel and refund). They reuse the game's own cancel, delivery and refund paths; refunds go to the mailbox. Players are told when staff change their listing, bid, order or request (new lines in `translations.lua`). No other change in game.
- **1.3.1** — Poggy Hub support: settings, lists and help pages for `/poggy`. No change in game.
- **1.3.0** — Treasury hooks (poggy_core 0.17.0): tax, deposits and shipping fees go to Poggy Banking's treasury when it is installed; sales, listings, orders and fills are reported; catalogue prices follow the price index. No change without a treasury.
- **1.2.1** — Categories and item icons come from poggy_core's item registry instead of VORP's `items` / `item_group` tables, so the auction house opens on RSG (it failed with "Table 'item_group' doesn't exist"). `Config.Categories` maps item groups to categories; the stock VORP list is unchanged.
