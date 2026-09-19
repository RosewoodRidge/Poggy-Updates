# Tuning baits and what bites

Open `/poggy`, pick **Poggy Fishing**, then the **Baits & fish** tab.
It holds three lists: **Bonus loot**, **Baits and lures** and **Fish**.

## How a fish is picked

When a line goes in, the script looks at the fish that live in that water.
It skips any fish that is out of season or outside its active hours.
Each fish left gets a weight:

1. Start with the bait's **size weight** for the fish's size.
2. If the fish is one of the bait's **target fish**, multiply by 3.
3. With the Pro Rod, multiply by the Pro Rod's **Fish size odds** too (Pro Rod tab).

A fish with a bigger weight is more likely to bite. A weight of 0 means it never bites on that bait.

## Size weights

| Size | Used by |
|---|---|
| `sm` | small fish |
| `md` | medium fish |
| `lg` | large fish (pike and gar are large) |
| `xl` | extra large (no shipped fish uses it) |
| `legendary` | the four legendary fish |

A size you leave out of a bait counts as **1**, not 0.
To keep legendary fish off a bait, give it `legendary = 0`.
With no bait at all, legendary fish never bite.

## Target fish

List fish ids from the **Fish** list, such as `bluegill_sm` or `northern_pike`.
Target fish are three times as likely to bite.
With the normal rod, if a target fish lives in the water, each tug also gains extra interest.

## Losing bait

- A snapped line always loses the bait.
- When a fish gets away, the bait is lost by its **lose chance**.
- The Pro Rod multiplies the lose chance by **Bait loss ×** (0.5 halves it), under **Loot and bait** on the Pro Rod tab.
- A caught fish never uses up bait.

## Adding a bait

1. Add the item to your framework's item list first.
2. Click **Add bait**. Set its name, item, bonus, lose chance and size weights.
3. Save and restart the script. The new item becomes usable on its own.

## Fish and waters

Which fish live in which water is built into the script.
You can change a fish's name, item, price, size, weights and hours.
A new fish, or a fish id you rename, will not appear in any water.
