# Editing what stores trade

What NPC stores sell and buy lives in `config/catalog_default.lua`.
In the hub it is the **Catalogs** tab: one entry per catalog, each with a
**Sells to players** and a **Buys from players** list.

## How it fits together

- A **catalog** (for example `GeneralStore`) has two parts:
  **Sells to players** and **Buys from players**.
- A **store type** (**Locations** → **Store types**) points at one catalog for
  selling and one for buying. The catalog shows **Used by** for every type and
  store that points at it.
- Every store of that type uses those lists.

So one edit to the `GeneralStore` catalog changes all nine general stores.

To add a catalog of your own, press **Add catalog** on the **Catalogs** tab,
name it, fill its lists, then choose it in a store type's **Sells from catalog**
or **Buys from catalog**.

## Fix the item names first

The shipped lists use common RedM item names. Your server may name things
differently. At start the console lists every item it cannot find.
Fix or remove those names; a missing item is skipped, never sold.

Run `/pmdiag` to see which lists loaded and how many items each has.

## An item, field by field

| Field | What it does |
|---|---|
| Item | The name in your framework's item list. |
| Label | The name players see. |
| Price | Price per unit. |
| Kind | `Item`, or `Weapon` for guns and tools that are weapons. |
| Stock | Starting stock. Leave it out for unlimited. |
| Hidden | Stocked but never shown, bought or sold. |

## A store that only buys, or only sells

In **Locations** → **Store types**, set **Sells from catalog** or
**Buys from catalog** to `false`.
The butcher ships this way: it buys pelts and sells nothing.

## Items nobody may trade

Add them to **Items no store may trade**, at the top of the **Catalogs** tab.
They are blocked everywhere, in every store and every player shop.
