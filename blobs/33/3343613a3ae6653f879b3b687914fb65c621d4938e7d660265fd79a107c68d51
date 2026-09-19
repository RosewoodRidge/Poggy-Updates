# Poggy Crafting

A crafting system for RedM, built on [poggy_core](https://github.com/RosewoodRidge/-Poggy-Scripts-).
Benches, campfires and world props open a recipe browser that shows the whole
tree behind an item: what it takes, what makes those, what they are used in,
and where in the world each ingredient comes from.

Runs on **VORP**, **RSG** and **QBR**. Every framework call goes through
poggy_core, so there is one copy of this script and it does not care which one
you run.

---

## What it does

**A recipe browser, not a list.** Categories down the side, a search that
matches recipe names, ingredients and rewards, and two filters: *materials
only* (what you can make right now) and *only accessible* (hide what your job
cannot touch). A recipe you have not unlocked still appears, greyed out under
**Requires Access**, so people can see what a job would open up.

**Recipe chains.** Open a recipe and the browser draws the tree beneath it:
each ingredient, whether it can itself be crafted, and how far you are from
having enough. **Used In** shows the other way: what this item feeds into.

**Where do I get this?** Hover any ingredient and a card says how it is
obtained, from `config/item_sources.lua`. Fill that in once and your Discord
stops getting asked where sulfur comes from.

**A shopping list and a gathering tracker.** Add a recipe's ingredients to a
per-character list. Tick one off, or pin the list to a small draggable window
that stays on screen while you collect them. The counts fill in as items land
in your inventory.

**Skillchecks, optionally.** A recipe can ask for timed rounds instead of a
progress bar (needs [poggy_skillcheck](https://rosewoodridge.xyz/store/poggy-skillcheck), free).
Pass them all for the full reward, pass some for a share, hit the clean band
for a bonus. A recipe can be dangerous: miss a round on dynamite and it goes off
in your hands; miss one on a moonshine run and you catch fire.

**Craft what you were never taught.** A recipe locked to a job can stay open to
everyone through `jobSkillcheck`. The tradesman crafts it straight away; anyone
else has to earn it on a much harder skillcheck.

**Alternative ingredients.** One slot can accept a list of items ("any fish",
"any pelt"), and `AltRewards` pays out by which one was actually used.

**Currency recipes.** A recipe can charge money, pay money, or both, in cash,
gold or role tokens.

**A placeable campfire.** The `campfire` item builds a campfire in front of the
player. `/extinguish` puts it out.

---

## Requirements

| | |
|---|---|
| **poggy_core** | 0.16.0 or newer. Install it first. 0.18.0 or newer adds the in-game settings editor. |
| **Framework** | VORP, RSG or QBR. |
| **oxmysql** | For the shopping list. |
| **poggy_skillcheck** | Optional, free. Only needed if your recipes use `skillcheck` or `jobSkillcheck`. |

No progress bar resource. No `uiprompt`. No menu resource. The script draws its
own.

---

## Install

1. Put `poggy_crafting` in your resources folder.
2. Add `ensure poggy_crafting` to `server.cfg`, **after** `poggy_core`.
3. Copy `docs/item_images/campfire.png` into your inventory's item image folder
   (`vorp_inventory/html/img/items/`, or your framework's equivalent).

There is no SQL to import. poggy_core creates the table and adds the campfire
item the first time the script starts. To manage the database yourself, set
`PoggyCoreConfig.Sql.AutoInstall = false` in poggy_core's config and import
`sql/install.sql`.

Then write your recipes. The thirteen that ship are examples, each showing one
feature with stock item names. They are meant to be deleted.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/craft` | Everyone | Opens the browser anywhere. Recipes that need a bench or prop are still out of reach. |
| `/gt` | Everyone | Lends the mouse to the gathering tracker so you can drag it somewhere else. |
| `/gthide` | Everyone | Hides or shows the tracker. |
| `/extinguish` | Everyone | Puts out the campfire you placed. |
| `/campfire` | Everyone | Places a campfire without the item. **Off by default**; for testing only. |

Every command name is set in `Config.Commands`. Set one to `false` to remove it.

---

## Configuration

**Every setting can be changed in game.** Type `/poggy`, open
**Poggy Crafting**, change what you need, save, and restart the script. The
editor needs poggy_core 0.18.0 or newer. You can still edit the files by hand.

| File | What is in it |
|---|---|
| `config/config.lua` | Benches, world props, campfires, timing, categories, animations, skillcheck difficulty, the progress bar. |
| `config/recipes.lua` | `Config.Crafting`: every recipe, with the full field reference at the top. |
| `config/item_sources.lua` | The "where do I get this?" tooltip. |
| `translations.lua` | Every string a player can see, including the browser. |

All four are readable, and the updater merges them forward: your values survive
an update and new settings appear alongside them.

### Highlights

| Setting | Default | What it does |
|---|---|---|
| `Config.CraftTime` | `5000` | How long a craft takes, in milliseconds. A recipe can override it. |
| `Config.CraftingPropsEnabled` | `true` | Craft at world campfires and ovens. Off if you only use benches. |
| `Config.CampfireItem` | `'campfire'` | The item that places a campfire. `false` for none. |
| `Config.ShoppingList.enabled` | `true` | The shopping list and gathering tracker. |
| `Config.Skillcheck.enabled` | `true` | `false` turns every skillcheck into a progress bar. |
| `Config.Webhook` | `''` | Discord webhook for a log of every craft. Blank = no log. |

### Where people craft

Three things can open the browser, and a recipe can require any of them:

- **Benches** in `Config.Locations`: a fixed point with a name, an optional
  blip, an optional job lock, and optionally only certain categories.
- **World props** in `Config.CraftingProps`: every campfire, oven and forge the
  game already places. Walk up to one and the prompt appears.
- **A placed campfire**: the `campfire` item, used from the inventory.

`Config.CraftingPropsEnabled = false` turns the prop scan off. It is the most
expensive thing this script does.

### Recipe fields

Documented in full at the top of `config/recipes.lua`. The short version:
`Items` in, `Reward` out, `Job` and `Location` decide who and where, `Category`
decides where it appears, and the skillcheck fields decide how hard it is.

The hub's Help tab has step-by-step guides: writing a recipe, adding a bench,
and skillcheck recipes.

---

## Permissions

There are no admin commands and no ACEs. Access is decided by jobs and places:

| Field | Where | Effect |
|---|---|---|
| `Job` | a category | Only these jobs can craft in it. |
| `Job` | a recipe | Only these jobs can craft it, unless it has a `jobSkillcheck`. |
| `Job` | a bench | Only these jobs see the bench's prompt. |
| `Config.CampfireJobLock` | all props and campfires | Only these jobs can craft at them. |
| `Location` | a category or recipe | Only at these benches or props. |

`0` means anyone, or anywhere. Otherwise, give a list: `{ 'blacksmith' }`.

---

## Troubleshooting

| Problem | What to check |
|---|---|
| "That recipe needs poggy_skillcheck" | Start poggy_skillcheck, or set `Config.Skillcheck.enabled = false`. |
| No prompt at a campfire | `Config.CraftingPropsEnabled` is on, the prop is in `Config.CraftingProps`, and `Config.CampfireJobLock` allows your job. |
| A recipe is greyed out | Its category or recipe `Job`, or its `Location`, does not match. `/craft` never reaches recipes that need a place. |
| An ingredient shows its raw name | The item has no label in your inventory. Add it to `Config.ItemLabelOverrides`. |
| An ingredient says "Unknown" | Add it to `config/item_sources.lua`. |
| Shopping list entries vanished | A recipe's `Text` was renamed. `Text` is the recipe's id. |
| A config change did nothing | Restart `poggy_crafting`. |

---

## Notes

**`canUseDecay` is VORP-only.** It refuses an ingredient below a condition
threshold. VORP tracks item condition; RSG and QBR do not, so there the check
passes and the item is accepted. Also, the *acceptance* is per stack but the
*removal* is by item name: on a mixed inventory the game may take a different
stack of the same item than the one that passed the check.

**Recipe names are identities.** The server matches a craft request on a
recipe's `Text`, and the shopping list stores it. Renaming a recipe orphans
anything already on a list. Treat it as an id.

**Nothing the browser sends is trusted.** The page sends a recipe name and a
quantity. The server looks the recipe up in its own config, reads the job
again, counts the real inventory, and only then takes anything.

**Materials are taken before a skillcheck.** That is deliberate: a failed craft
has to cost something, or there is no reason to aim.

**One craft is at most 100 at once.** Larger requests are ignored.

---

## Credits and licence

The original `vorp_crafting` was written by the VORP team (@blue) and is
distributed under the **GNU General Public License v2**; see `LICENSE`. This is
a fork of it: the interface, the shopping list, the gathering tracker, the
skillcheck system, the chain view and the poggy_core layer are new, and the
licence carries over to the whole of it.

Redesign and rework by **Poggy** / [Rosewood Ridge](https://rosewoodridge.xyz).
