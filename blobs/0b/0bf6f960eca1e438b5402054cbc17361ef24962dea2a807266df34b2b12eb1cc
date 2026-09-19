# Setting up armor protection

Armor protection is **off** when poggy_util is installed.

## What it does

- A player wearing something in the native **armor** clothing slot is protected.
- Shots to the torso (pelvis to upper spine) are blocked.
- Each blocked shot uses one hit. After **Hits the armor absorbs** hits, the armor breaks.
- Broken armor blocks nothing until the player uses the **repair kit**.
- The kit is used up only when the repair finishes.

Head, arm and leg shots are never blocked.

## Turning it on

1. Open `/poggy` and pick **Poggy Util**.
2. On the **General** tab, under **Utilities**, turn on **Armor protection**.
3. On the **Combat** tab, check **Repair kit item**. On VORP the `armor_kit` item is added for you.
   On RSG add it to `rsg-core/shared/items.lua` by hand.
4. Copy `docs/armor_kit.png` into your inventory's image folder.
5. Save and restart poggy_util.

Players need clothing in the armor slot. The clothing stores that use that slot (vorp_character, jo_libs, jo_clothingstore) work.

## The HUD icon

While armor is worn, an icon shows what is left in 10 stages.

- **Hits per HUD stage** × 10 should equal **Hits the armor absorbs** (the default is 2 × 10 = 10 hits).
- Players move the icon for themselves with `/movearmor`, then click to save. It is saved per character.

## For doctor and injury scripts

Other scripts can ask poggy_util, on the client:

| Export | Answer |
|---|---|
| `ArmorIsWearing()` | wearing armor |
| `ArmorGetShots()` | hits left |
| `ArmorIsBroken()` | no hits left |
| `ArmorShouldBlockBleeding()` | wearing, not broken, and the last hit was on the torso |
