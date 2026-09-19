# Managing live auctions, orders and requests

Three tables in `/poggy` show what players are doing in the auction house right now. Changes apply at once. No restart is needed.

Open `/poggy` → **Auction House**, then:

| Table | Tab | What it shows |
|---|---|---|
| **Live auctions** | Auctions | Every active listing, from all auction houses |
| **Shipment orders** | Supply catalogue | Catalogue orders on their way, and those closed in the last three days |
| **Want requests** | Want requests | Open requests, and those closed in the last three days |

Everything here moves items and money the way the game does. Anything a player gets back goes to their **mailbox**, whether they are online or not. Players online also get a message.

## Live auctions

Each row shows the item, the seller, the start price, the leading bid and bidder, how many bids it has, and when it ends.

- **Cancel and return item** (type the listing number to confirm). The listing closes. The item goes to the seller's mailbox. The leading bid goes back to the bidder's mailbox. This is what happens when a seller cancels their own listing. The deposit is not returned, as in game.
- **Extend**. Adds whole hours (1 to 720) to the listing. A listing whose time is already up cannot be extended: the expiry check is settling it.
- A row tinted yellow has run out of time. The next expiry check settles it, or run `/auctionrefresh`.

### A listing's bids

Click a listing to see its bids, newest first.

- Only the **leading** bid holds money. Every other bid was beaten and refunded to its bidder's mailbox at that moment.
- **Remove bid** on the leading bid: its money goes back to the bidder's mailbox, and the listing has **no bid** now. The bids it beat do not come back, because those players already have their money.
- **Remove bid** on an outbid bid only takes it out of the history. No money moves.
- The bid counter players see never goes down. Removing the leading bid adds one to it, so a bid placed at the same moment cannot land on the old price.

Bids on a closed listing are settled and cannot be removed.

## Shipment orders

Each row shows the player, the supplier, what was ordered, what they paid, the stage, and where it will be delivered.

- **Cancel and refund** (type the order number to confirm). Only while the order is still **Processing**, as in game. Everything the player paid, shipping included, goes to their mailbox.
- **Deliver now**. The order goes through its remaining stages at once and is delivered as the delivery timer would: into the chosen Poggy Markets shop, or the mailbox when there is no shop or the shop is full.

## Want requests

Each row shows who asked, for what, how many are wanted and filled, the price, the escrow still held and when it expires.

- Click a request to see who filled it, how many, and what they were paid.
- **Cancel and refund** (type the request number to confirm). The request closes. The escrow not yet spent goes to the requester's mailbox, as when a request expires. Items already delivered stay delivered.

## Good to know

- Every change is saved in the hub's **History**, and printed in the server console with the name of the staff member.
- Cancelling a listing and removing a leading bid are also sent to the Discord log, when it is on.
- If someone acts on the same row at the same moment (a bid, a fill, the delivery timer), the hub refuses the change and asks you to refresh. Nothing is paid twice.
