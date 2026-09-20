# Adding a bench

A bench is a fixed spot where players craft. Benches live in
`Config.Locations` in `config/config.lua`. In the hub they are the
**Benches** list, in the **Locations** tab.

## Steps

1. Stand at the spot. Note your position (x, y, z).
2. Open `/poggy`, then **Poggy Crafting**, then **Locations** → **Benches**.
3. Click **Add bench** and fill it in:

| Field | What to put |
|---|---|
| ID | A unique id, for example `smithy_rhodes`. Recipes use it. |
| Name | Shown on the prompt and the map. |
| X, Y, Z | The position. |
| Categories | **Every category**, or **Only these** and pick them from **Recipes** → **Categories**. |
| Jobs | **Anyone**, or **Only these jobs** and add each job name. |
| Map blip | Optional. `Show blip` on, and a sprite name such as `blip_shop_blacksmith`. |

4. Save, then restart the script.

## Lock recipes to the bench

Open the category (or one recipe), set **Where** to **Only at these places**,
and add the bench's **ID**. That category (or recipe) then only works at that
bench. In the file it is `Location = { 'smithy_rhodes' }`.
The full walk-through is the help page **Locking crafting to a job or a place**.

## Campfires, ovens and other props

You do not need a bench for these. Every prop model in **Crafting props**
opens the browser when a player walks up to it. The prop group's **Title**,
in lower case, is its location id: `Location = { 'campfire' }`.

The prop scan is the most expensive thing this script does.
If you only use benches, switch off **Craft at world props**.

## Never reuse an ID

Two benches with the same ID confuse every recipe that names it.
Renaming an ID breaks the recipes and categories that use the old one.
