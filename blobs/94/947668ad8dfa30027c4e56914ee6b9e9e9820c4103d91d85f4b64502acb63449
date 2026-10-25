# Owned balloons

Players can buy their own balloon at any balloon station and call it to the nearest road from anywhere. One balloon per character, saved in the database.

## How players use it

1. Talk to an operator and pick **Buy a Balloon**. The price is taken from their cash.
2. Near a road, press **B** or type **/balloon**. The balloon is brought to the nearest road and shown on their map.
3. At a station, **Take Out Your Balloon** brings it out there, with no road needed.
4. **/balloon away** puts it away. Calling it again moves it to the road nearest them.
5. **Sell Your Balloon** at a station sells it back for part of what they paid.

Friends ride along as passengers the usual way. Only the owner can take the captain's seat, unless you turn **Only the owner flies it** off.

## Settings

| Setting | Where | Default |
|---|---|---|
| Owned balloons on | General | On |
| Balloon price | Economy & prices | $500 |
| Sell-back share | Economy & prices | 50% (0 removes Sell) |
| Command name | Owned balloons | balloon |
| Call key | Owned balloons | 0x4CC0E2FE (B) |
| Road distance | Owned balloons | 75 metres |
| Call cooldown | Owned balloons | 15 seconds |
| Only the owner flies it | Owned balloons | On |
| Map blip | Owned balloons | On |

## Changing the call key

**Call key** takes a control hash, the same kind as the taxi keys. Find them in the rdr3_discoveries controls list. Set **Call key name** to the new key's name so the messages match. 0 turns the key off; the command still works.

The key does nothing for players who own no balloon, while they sit in a vehicle, or during a taxi ride (the taxi's Take Off key is also B).

## "You're not near a road"

The balloon only comes to a road or trail. If players on your map are often far from one, raise **Road distance**.

## Taking a balloon away from a player

Delete their row from the `poggy_balloons` table (keyed by character id). Turning **Owned balloons on** off hides the feature but keeps every saved balloon.
