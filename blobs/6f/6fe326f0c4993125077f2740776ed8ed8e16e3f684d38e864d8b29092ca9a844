# Adding drop locations and loot

Each supply drop picks one **location** at random and one **loot** entry by weight.

## Add a drop location

1. Go to an open spot on the ground with clear sky above it (the balloon comes down from 40 m).
2. Open `/poggy` → **Supply Drops & Scavenger Hunts** → **Supply drops**.
3. In **Drop locations**, press **Add location**.
4. Give it a **Name**. Players read it in the notification ("…descending over Blackwater Docks").
5. On **Position**, press **use my position**.
6. Save and restart.

## Add loot

1. In **Drop loot**, press **Add loot**.
2. Pick the **Item**. It must exist in your items table.
3. Set **Amount (min, max)**: the fewest and most the crate holds.
4. Set a **Weight** (see below).
5. Set a **Crate prop**: the object players see, such as `p_crateapple01b` or `p_boxlrgmeat01x`.

### Weapons

Use a weapon name such as `WEAPON_REVOLVER_CATTLEMAN` as the item. The player gets one weapon; the amount is ignored.

### How weight works

Weights are compared with each other. They are not percents.

- Coal has weight 20 and a rifle has weight 1, so coal is 20 times as likely as the rifle.
- Add up all the weights. An entry's chance is its weight divided by that total.
- Use whole numbers.

## Test it

Use `/supplydrop` in game (admins) to start a drop now.
