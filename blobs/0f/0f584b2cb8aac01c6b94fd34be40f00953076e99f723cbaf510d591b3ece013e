# Locking crafting to a job or a place

Every recipe, category and bench has two locks: **Jobs** (who) and **Where**
(at which bench or prop). Each is either open to everyone, or a list of
names. Nothing here needs the config file: it is all done in `/poggy`.

## The two switches

Open any recipe, category or bench in `/poggy` and you will see:

- **Jobs**: **Anyone** / **Only these jobs**. Choose the second and add each
  job name with the job picker.
- **Where**: **Anywhere** / **Only at these places**. Choose the second and
  add the places: a bench's **ID** (from **Locations** → **Benches**) or a prop
  group's title in lower case (`campfire`, `oven`, `forge`). **Add** lists
  every bench and prop place you have (poggy_core 0.20.0 or newer); a bench you
  have just added shows up straight away, before you save. A place the list
  does not know is marked with a warning.

A category's locks apply to every recipe in it. A recipe's own locks are
checked as well, so a recipe can be stricter than its category, never looser.

If a field shows a warning such as **"8 is neither Anyone nor a list of
names"**, someone typed a number into it. That locks it for everyone. Pick
**Anyone** or **Only these**, save, and it works again.

## Example: the blacksmith only crafts in the smithy

The blacksmith has the job, but is crafting at a campfire by the mine. Make
the blacksmith recipes work only at the shop.

1. Stand at the anvil in the smithy and note your position.
2. Type `/poggy`. Open **Poggy Crafting**.
3. Open the **Locations** tab, then **Benches**. Click **Add bench**.
4. Fill it in:
   - **ID**: `smithy` (any short name; write it down, step 6 needs it).
   - **Name**: `Smithy Anvil`.
   - **X, Y, Z**: your position (the coords picker has **Use my position**).
   - **Categories**: **Only these**, and add the blacksmith category.
   - **Jobs**: **Only these jobs**, and add `blacksmith` (or **Anyone**, if the
     category is already locked to the job).
5. Open the **Recipes** tab, then **Categories**, and open the blacksmith
   category.
6. Set **Where** to **Only at these places**, press **Add** and pick the
   bench from step 4 (its ID is `smithy`).
7. **Save**, then **Save & Restart** (or `restart poggy_crafting`).

Now every blacksmith recipe works at that anvil and nowhere else. At the
campfire they show greyed out, with **Craft at: Smithy Anvil** on the badge,
and `/craft` shows the same. Recipes in other categories are not affected.

To allow one recipe elsewhere as well, open that recipe and give it its own
**Where** list. To lock a single recipe rather than a whole category, leave
the category on **Anywhere** and set **Where** on the recipe instead.

## Example: a job-locked recipe "is not picking up my job"

A player has the job, but the recipe stays grey. Look at the badge:

- **Requires: bwcafe**: the job really does not match. Check the exact
  spelling and case of the job name in **Jobs**, against what the server
  reports for the player (`poggycore do job.get src=<id>` in the console).
- **Craft at: Campfire, Oven**: the job is fine; they are in the wrong place.
  `/craft` counts as nowhere, so a recipe with a **Where** list never unlocks
  through the command. Go to one of the places named.
- **Unavailable**: the field holds a number. Open the recipe (or its
  category) in `/poggy` and pick **Anyone** or **Only these**.

## What each lock means

| Where the lock is | What it does |
|---|---|
| **Category → Jobs** | Only those jobs see the category open. Others see it under **Requires Access**. |
| **Category → Where** | Every recipe in the category only works at those places. |
| **Recipe → Jobs** | Only those jobs may craft it. With a **job skillcheck** set, anyone may try it on a hard skillcheck. |
| **Recipe → Where** | The recipe only works at those places. |
| **Bench → Jobs** | Only those jobs get the prompt at the bench. |
| **Bench → Categories** | The bench offers only those categories. |
| **Prop job lock** (Locations tab) | Only those jobs may use campfires, ovens and forges. |

## The world props

Campfires, ovens and forges the game already places are crafting spots too.
Their titles, in lower case, are place IDs: `campfire`, `oven`, `forge`.
The forge (`p_furnace01x`) is its own group because furnaces also stand at
mines; a recipe locked to `oven` does not work at one.
