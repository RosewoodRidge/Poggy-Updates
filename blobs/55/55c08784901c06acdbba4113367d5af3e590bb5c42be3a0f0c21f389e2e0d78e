# Setting up an armory

An armory is a job-locked shop with endless stock, like a sheriff's gun rack.
Players walk up, press the key, and pick what they need.

## Add one

1. Open **Armories** and choose **Add armory**.
2. Set the **Position** where players press the key.
3. Give it a unique **Id** and a **Name**.
4. Put the jobs that may use it in **Jobs**. Names are exact and
   case-sensitive. Leave it empty to open it to everyone.
5. Set **Item list** to the name of an item list, such as `WeaponArmoryItems`.
6. Optional: set a **Clerk** model and position, and a blip.
7. Save and restart the script.

## What it sells

The **Armory items** list holds what is on the shelf.

| Field | What it does |
|---|---|
| **Item or weapon** | The item name, or `WEAPON_...` for a weapon. |
| **Price** | Per item. `0` shows as Free. |
| **Type** | Weapons are handed out one at a time. Items ask "How many?". |

Prices are charged in the **Currency** you pick (cash or gold).

## How a purchase works

1. The job is checked when the armory opens, and again on every take.
2. The script checks the player can carry the item.
3. The money is taken.
4. The item is given. If the inventory refuses it, the money goes back.

## Discord logging

Paste a webhook into **Webhook**, turn **Discord tracking on**, and restart.
The console then prints an activity message id. Paste it into **Activity
message id** so the same message is edited from then on.
