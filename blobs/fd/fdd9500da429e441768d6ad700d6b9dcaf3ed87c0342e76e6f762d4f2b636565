# Hidden recipes

With **Hide recipes with no item icon** on, the server checks every item each
recipe uses when it starts. A recipe whose items have no icon is hidden from the
browser, and it cannot be crafted.

## Why a recipe is hidden

The Recipes list here marks each hidden recipe:

| Mark | What it means | What to do |
|---|---|---|
| **No icon for ...** | The recipe uses an item with no icon file. The mark names the file. | Add that PNG to your inventory's icon folder, then restart the server. |
| **Nothing can make ... any more** | Every recipe that made one of its ingredients is hidden. | Fix the recipe that makes the ingredient; this one comes back with it. |

Raw materials (anything no recipe makes) never hide a recipe this way: they are
assumed to come from the world.

## Checking again

`poggycrafting_icons` in the server console runs the check and prints how many
recipes are hidden. Icons added while the server runs show after a server
restart, because the inventory only serves the files it found when it started.

## Icons in another resource

Poggy Core finds your framework's icons (VORP, RSG or QBR). If yours are
somewhere else, set **Config.IconResource** and **Config.IconPath** in
`config/config.lua`.
