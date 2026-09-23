# Adding a store

A store needs three things: a **type**, a **name** and a **position**.
The type brings the item lists, the map blip and the clerk.

## The quick way

1. Stand where the player should get the prompt.
2. Face the way the clerk should face.
3. Run `/pmhere <type> <name>`, for example `/pmhere gunsmith Valentine Gunsmith`.
4. Copy the block from the server console into `config/stores.lua`.
5. Restart `poggy_markets`.

Run `/pmhere` with no type to see every type name in the console.

## In the hub

1. Open `/poggy`, then **Poggy Markets**, then **Locations** → **Stores**.
2. Click **Add store**.
3. Pick the **Store type** (from the **Store types** list on the same tab) and set the **Name**.
4. Stand on the spot and press **Use my position** on **Prompt position**.
5. Save, then restart the script.

## Three positions, not one

| Field | What it is |
|---|---|
| Prompt position (`coords`) | Where the **player** stands to get the prompt. |
| Clerk position (`npc.coords`) | Where the **clerk** stands, behind the counter. Left out, the clerk stands on the prompt. |
| For-sale position (`purchaseCoords`) | Where the **FOR SALE** prompt appears. |

Most town shops need no clerk: the game already has a shopkeeper there.
Set **Clerk** to `false` for those.

## Selling a storefront to players

A storefront can only be bought when **both** are true:

- it has a **Shop ID** that points at a row in the `playershops` table;
- that row is unowned or repossessed.

Then set **For sale** on, and a **Sale price**. The FOR SALE prompt
disappears as soon as someone buys it.

To sell it only to the holders of a job (a stable to the stable's job), fill
**Buyers' jobs**. The job they are wearing counts, and with poggy_multijob so
does every job on their list. Everyone else is told which job they need.
Admins can always buy, and can hand the shop over in **Player shops** →
**Transfer** instead.

## Only some jobs may use it

Fill **Job lock** with job names, for example `sheriff`.
Everyone else is turned away.

## Giving the shop's staff a job

With **Give shop jobs** on (Shops tab), a shop's owner and everyone they hire
get the shop's job, at the grade for their role, and lose it when they leave.

Only server staff set a shop's job. Owners and their staff cannot; they see it
in the staff tab as a locked title ("Shop job: ... Set by staff").

- **In the hub (the main way):** **Player shops** tab → **Shop jobs**. Every
  shop is listed; edit its **Shop job** cell and it applies at once, no
  restart. Clear the cell for no job. The same table is on the Poggy Multijob
  page.
- In the file: the store's **Job** field (`job` in `config/stores.lua`), used
  until staff set another.
- Optional command: `/pmshopjob <shopId> <job>` (admins, or the server
  console). `none` removes it, `reset` goes back to the file's job, and with no
  job it shows the current one. `/pmshops` lists shop IDs.
