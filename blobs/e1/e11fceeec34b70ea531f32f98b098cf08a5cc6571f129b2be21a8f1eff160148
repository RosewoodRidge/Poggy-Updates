# Roles, staff chat and ready-made replies

All three are edited in two places, and both change the same thing:

- In the staff panel: **Staff** tab → **Roles** or **Ready-made replies**. Rooms are made on the **Staff chat** tab.
- In `/poggy`: Tickets → **Roles, chat & replies**.

You need a role with the power **Roles, chat rooms and replies**. Admin always has it. Changes apply at once: no restart.

## Roles

A new server starts with four: **Admin**, **Mod**, **Helper**, **Developer**. They are only a starting point. Rename them, change them, delete them, or make your own: *Staff Manager*, *Community Manager*, *Event Team*.

For each role you choose:

| | |
|---|---|
| **Name and colour** | What staff see on chips and in chat. |
| **Rank** | 0 to 99. Higher is more senior. It only orders lists. |
| **May do** | Tick each power: claim, reply, close, go to a player, change kind, change priority, give a ticket to someone, escalate, mark the answer, warn, kick, ban, lift bans, staff notes, archive, add staff, read the audit trail, manage roles. |
| **Sees these kinds of ticket** | A role only ever sees the kinds ticked here. |
| **Also reads the staff chat of** | Whose role room it can read, besides its own. |
| **Told about "I need help right now"** | Who gets the help strip and its pop-up. |
| **Discord webhook** | Copies the role's staff chat to a Discord channel. |

One person can hold several roles. They get everything any of their roles allows.

**Admin is locked.** It holds every power, sees every ticket and reads every room, and it cannot be deleted or stripped, so nobody can lock the whole team out. Its name, colour and webhook are still yours. Anyone your framework counts as an admin is an Admin here without being added.

Deleting a role takes it off everyone who held it. Someone who held only that role stops being staff.

### Updating from 1.0

The first time 1.1.0 starts, it copies your old **Who sees which kind** and **Who hears help calls** settings into the roles, once. After that those two settings do nothing: the roles are the truth.

## Deleting a ticket

**Archive** is a power a role can hold. An archived ticket leaves every list but the **Archived** filter, which only people who may archive can see. Nothing is destroyed, and **Bring back** undoes it.

**Delete for good** is not a role power. Only the server owner can allow it, in `server.cfg`:

```
add_ace identifier.steam:110000112345678 poggy_tickets.delete allow
```

Use that person's own identifier, or a group such as `group.owner`. Even then, only a ticket that is already archived can be deleted, so it always takes two steps. The audit trail keeps one line saying who deleted which ticket and when; what the ticket said is gone.

## Staff chat

Every role has a room of its own. With the roles a new server starts with:

- **Admins** read every room.
- **Mods** read Mod and Helper.
- **Helpers** read Helper.
- **Developers** have a room to themselves.

Change that on each role (*Also reads the staff chat of*). Make extra rooms for any mix of roles with **New room**: a room for an event, for managers, for one project.

**Staff chat is never saved.** Messages live in the server's memory and are gone at the next restart. There is no chat log to keep, to leak or to answer for. If a room needs a record, give it a Discord webhook: every message is copied to that channel, and Discord keeps it.

A webhook address is only ever written. The panel shows that one is set, never what it is.

## Ready-made replies

Short replies staff pick instead of typing them again. On a ticket they sit above the reply box: the ones whose **keywords** appear in what the player wrote come first, and **Replies …** opens the whole list. Picking one fills the box; you can still change it before you send.

- `{player}` becomes the player's first name, `{staff}` yours, `{id}` the ticket number.
- Keywords are a comma list: `lost, items, horse`. It is plain word matching, nothing clever.
- The script ships with eight. Change them or delete them; a deleted one does not come back.
