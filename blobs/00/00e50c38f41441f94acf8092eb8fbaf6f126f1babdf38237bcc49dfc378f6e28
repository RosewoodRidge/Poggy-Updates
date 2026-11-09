# Managing shops and owners

Shops change hands in three ways: an owner hands theirs over, an admin moves
it with `/pmadmin`, or an admin moves it here in **Player shops**. All three
keep the shop's stock, ledger and staff.

## Owners handing a shop over

In the shop's manager, **Settings** tab, the owner sees **Hand Over Shop**.

1. **Find players nearby** lists everyone standing close by.
2. Pick one. Anyone greyed out cannot take it, and the list says why.
3. Type the shop's name to confirm, then **Offer shop**.
4. The other player gets an **Accept / Decline** box. Nothing changes until
   they accept.

Who can receive a shop:

- They must be standing near the owner (**Handover distance**).
- They cannot already own as many shops as **Shops per character** allows.
- A storefront reserved for a job (its **Buyers' jobs**) only goes to
  someone holding that job.

Turn **Owners can hand shops over** off (Player shops tab) and only admins
can move shops.

## The shop admin panel (/pmadmin)

For staff who do not use `/poggy`. Anyone in **Admin groups** can open it,
and so can anyone granted the **Admin ACE** in `server.cfg`:

```
add_ace group.moderator poggy_markets.admin allow
```

The table lists every shop. Search by shop ID, name, owner or job, or filter
to owned, no-owner or repossessed shops. Click a shop to:

| Action | What it does |
|---|---|
| **Route to shop** | Sets a GPS route to it. |
| **Open manager** | Opens its manager as the owner would see it. |
| **Transfer** | Search every character, online or not, by name or character id, pick one, and type the shop ID to confirm. The job and shop limits do not apply here. |
| **Repossess / Restore** | Hides the shop from its owner and staff, or brings it back. Nothing is deleted. Repossess asks for a second click. |
| **Ledger** | Shows the balance. Change it and say why; the difference shows in the shop's ledger as an admin adjustment. |
| **Shop job** | The job the owner and staff are given. Empty = none, `reset` = the job in config. |
| **Staff** | Removes someone from the shop's staff. |

Every change is written to the server console and the admin webhook with
who made it.

## Shop wages

Turn on **Shop wages on** (Player shops tab). Hired staff can then clock in
at the shop and are paid from that shop's ledger. No money is created.

In the manager's **Employees** tab, the owner fills in **Per hour** and, if
they want a limit, **Max/day**, then presses **Save**. Someone with no wage
cannot clock in.

- **Staff with a back office** clock in from the strip at the top of the
  manager.
- **Clerks** have no back office. Pressing the shop's key gives them a small
  card to clock in or out, or to browse the shop as a customer.
- **Owners are not paid**, and neither is anyone who reaches a shop only
  through their job.

Pay arrives every **Pay every** minutes and when someone clocks out. It shows
in the ledger as **Wages**.

The daily limit is per person, per shop. It comes from the wages recorded
today by the database's clock, so a server restart does not reset it.

Someone is clocked out when they reach their limit, stay too far from the
shop, stop moving past the **Idle limit**, are let go, or the shop cannot pay.
With **Keep tax money in the ledger** on, wages stop before the ledger falls
below one tax payment.

Logging out or a server restart ends the shift. Only the unpaid part of the
current pay period is lost.
