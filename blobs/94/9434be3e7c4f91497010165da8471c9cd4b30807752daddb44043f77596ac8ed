# Poggy Auction House

An auction house for RedM on VORP. Players list goods, bid, buy out and collect at auction houses around the map. It also has a supply catalogue for ordering raw goods, and a board for posting want requests.

## Requirements

- `poggy_core` 0.13.0 or newer, on VORP or RSG
- `oxmysql`
- Optional: `poggy_markets`, to ship catalogue orders straight into a shop's storage

## Install

1. Put `poggy_auction` in your resources folder.
2. Add `ensure poggy_auction` to `server.cfg`, below `oxmysql` and `poggy_core`, and below `poggy_markets` if you use it.
3. Start the server. The tables are created automatically when the script starts (`sql/install.sql`). To manage them yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua` and import that file.

## What players get

- **Browse**: every live auction. Filter by category, search and sort. Bid, or buy it out when the seller set a buyout price.
- **Sell**: list from your satchel with a starting price and an optional buyout, per piece. Pick how long it runs. Longer listings take a bigger deposit, and a sales tax comes off the sale.
- **My Auctions** and **My Bids**: your own listings, and whether you are winning or outbid.
- **Mailbox**: winnings, sale money, outbid refunds, returned items and deliveries all land here. Collect one at a time or all at once.
- **Order Catalogue**: order raw goods from lumberjacking, mining, farming, hunting, fishing and crafted suppliers. Prices shift a few percent each time the catalogue opens. Each order goes through processing, transit and delivery, then arrives in the mailbox or in a Poggy Markets shop.
- **Want Requests**: post a buy order and the money is held in escrow. Other players fill it in full or in part and get paid as they deliver. Anything left over comes back when the request ends.

## Setup

Everything is in `config.lua`.

| Setting | What it does |
|---|---|
| `Config.Locations` | Where the auction houses are, the NPC and the blip |
| `Config.Interaction` | Prompt key, prompt distance, NPC spawn distance |
| `Config.Auction` | Durations and deposits, sales tax, price limits, bid steps, listing and mailbox limits |
| `Config.Categories` | The browse categories, and which item groups land in each |
| `Config.BlacklistedItems` | Items that can never be listed |
| `Config.Discord` | Webhook logging, per event |
| `Config.Shipment` | Shipping fee, order limits, how long each stage takes, price swing, shop delivery |
| `Config.ShipmentCatalog` | What can be ordered, with prices and descriptions. Items your server doesn't have are left out on their own. |
| `Config.Requests` | Want-request tax, limits and durations |

The browse categories are built from poggy_core's item registry. Each item has a group (on VORP its `item_group` id, on RSG the item's `category` in `rsg-core/shared/items.lua`), and `Config.Categories` says which category each group belongs to. The stock VORP groups are already mapped; a group that is not mapped shows as a category of its own, so add a row to give it a label and an icon. Player-facing messages are in `translations.lua`.

## Poggy Markets

When `poggy_markets` is running, the order form shows a **Deliver to Shop ID** box. A shop's ID is shown at the top of its manager.

Owners, and staff whose role lets them deposit items, can send an order straight into that shop's storage. If the shop is full when the order arrives, it goes to the mailbox instead.

Without `poggy_markets` the box is hidden and every order goes to the mailbox. Set `Config.Shipment.ShopDelivery = false` to turn shop delivery off.

## The treasury

With Poggy Banking's treasury module running, the auction house takes part in
the server's economy: the sales tax on every sale and want-request fill, the
listing deposit and the catalogue's shipping fee are paid to the treasury
instead of being deleted; sales, listings, orders and fills are reported so
the treasury can build a baseline of normal trade; the treasury's price index
scales catalogue prices, and its policy may add a few points to the sales tax.

Without a treasury nothing changes: the fees are deleted as before, the index
is 1.0 and the reports go nowhere. There is no setting.

## Admin commands

| Command | What it does |
|---|---|
| `/auctionrefresh` | Settles every listing that has run out, without waiting for the next check |
| `/auctionpurge` | Cancels every active listing. The items in them are **not** returned. |
| `/auctiondiscord` | Sends a test message to the Discord webhook |

Admins are characters in the groups `admin`, `superadmin` and `god`. Each command also works from the server console.

## Changelog

- **1.3.0** — Treasury hooks (poggy_core 0.17.0): tax, deposits and shipping fees go to Poggy Banking's treasury when it is installed; sales, listings, orders and fills are reported; catalogue prices follow the price index. No change without a treasury.
- **1.2.1** — Categories and item icons come from poggy_core's item registry instead of VORP's `items` / `item_group` tables, so the auction house opens on RSG (it failed with "Table 'item_group' doesn't exist"). `Config.Categories` maps item groups to categories; the stock VORP list is unchanged.
