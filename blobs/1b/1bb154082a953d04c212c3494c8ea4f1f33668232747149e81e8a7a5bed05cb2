# Writing a recipe

Recipes live in `config/recipes.lua`. In the hub they are the **Recipes** list,
on the **Recipes** tab, next to **Categories**.
The thirteen that ship are examples. Delete them and write your own.

## The smallest recipe

```lua
{
    Text     = 'Bread Bait',
    Items    = { { name = 'bread', count = 1 } },
    Reward   = { { name = 'bait_bread', count = 5 } },
    Type     = 'item',
    Category = 'supplies',
    Job      = 0,
    Location = 0,
},
```

Two ingredients in, one reward out, anyone, anywhere.

## The fields you will use most

| Field | What it does |
|---|---|
| `Text` | The recipe's name **and** its id. It must be unique. Renaming it empties it from shopping lists. |
| `SubText`, `Desc` | The short line and the long description in the browser. |
| `Items` | What one craft uses. `take = false` keeps an item, like a tool. |
| `Reward` | What one craft gives. |
| `Type` | `'item'`, or `'weapon'` for a real weapon with a serial. |
| `Category` | A category from the **Categories** list (the hub offers a dropdown). |
| `Job` | **Anyone**, or **Only these jobs** and the job names. In the file: `0` or `{ 'blacksmith' }`. |
| `Location` | **Anywhere**, or **Only at these places**: bench IDs and prop titles. In the file: `0` or `{ 'campfire' }`. |
| `Animation` | A key from the **Animations** list (**Advanced** tab). Default `'craft'`. |
| `CraftTime` | How long it takes, in milliseconds. |

## "Any fish" in one slot

Give an ingredient `AltNames`. Any of those items fills the slot.
Add `AltRewards` to pay out by which one was used:

```lua
Items = { { name = 'fish', count = 1, AltNames = { 'a_c_fishperch_01_ms' } } },
AltRewards = {
    ['a_c_fishperch_01_ms'] = { { name = 'fish_filet', count = 4 } },
},
```

## Charging or paying money

Set `UseCurrencyMode = true`. A reward entry with **no name** is money:

- `{ count = -2 }` charges the player $2.
- `{ count = 5 }` pays the player $5.

`CurrencyType` picks the money: `0` cash, `1` gold, `2` role tokens.

## Tell players where ingredients come from

Add each ingredient to **Where items come from** (`config/item_sources.lua`).
Hovering the ingredient in the browser then shows a badge and one line.

## Check it

Restart `poggy_crafting`, then open `/craft`. A recipe you cannot reach shows
greyed out under **Requires Access**, with the reason on its badge: the job it
needs, or **Craft at:** and the places where it works.
