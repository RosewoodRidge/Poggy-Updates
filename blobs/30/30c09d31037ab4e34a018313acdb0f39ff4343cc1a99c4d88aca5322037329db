# Running the economy modules

Three optional systems sit on top of the stores. Switch each one on or off
in the **General** tab, under **Optional modules**. Tune them in the
**Economy & prices** tab. Restart the script after a change.

## Dynamic pricing

Every tracked item has a multiplier that starts at 1.0.

- Players **selling** an item to stores push its price **down**.
- Players **buying** it push the price **up**.
- Left alone, the price drifts back toward normal.

Items are grouped into **markets**. Dumping one pelt softens the price of
every pelt in the same market.

**Start small.** This changes your whole server's economy. Track one market,
watch it for a week, then add more.

| To do this | Change this |
|---|---|
| Track a new kind of item | Add a market to **Price markets**, with the catalogs or name patterns it covers. |
| Stop prices crashing so far | Raise **Lowest multiplier**. |
| Make prices recover faster | Raise **Recovery per hour**. |
| Keep items independent | Set **Spillover** to 0. |

**Live prices.** **Economy & prices** → **Market prices** lists every
tracked item with its price now, base, multiplier, trend and last trade.
Type a new price to nudge it (it stays between the lowest and highest
multiplier), or use **Reset to base**. Open an item to see its price points
for the last week. Changes apply at once, no restart, and go in the history.

## Commodities exchange

Players pay into an exchange balance, then hold positions on a market:
long if they think prices will rise. Positions settle in cash.

It needs dynamic pricing on.

- **Tradable markets** decides what can be traded.
- **Largest position** and **Exchange fee** are the brakes. Raise them carefully.
- **Allow short selling** is off at first, because a short can lose more than it staked.

Add or move the desks in **Locations** → **Exchange desks**.

**Accounts.** **Economy & prices** → **Exchange accounts** lists every
account with its balance and open positions. **Correct balance** adds or
takes an amount (give a reason; it is logged and shows in the player's trade
log). Open an account to see its positions; **Close position** settles one
at the live price, exactly as the player closing it would.

## Ghost buyers

Unseen customers now and then buy a few items from player shops,
so owners on a quiet server still see some income.

The defaults are stingy on purpose: a few dollars a day.

- **Most expensive item bought** and **Most per sale** stop an owner pricing
  one item at $999,999 and printing money.
- **Pay into: Shop ledger** keeps ghost income on the same path as real income.
- **Tell the owner** is off, so the customers feel real.
