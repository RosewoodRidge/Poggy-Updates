# Setting up: staff, routing and Discord

## 1. Add your staff

Anyone your framework counts as an admin is already an **Admin** here.

1. Type `/tickets`.
2. Open the **Staff** tab.
3. Search a character name. It finds everyone, online or not.
4. Pick them, tick their roles, press **Save**.

A role is on the **account**, so it covers every character that person plays. One person can have several roles.

These four are only what a new server starts with. See **Roles, staff chat and ready-made replies** to change them or make your own.

| Starts as | Sees | Can |
|---|---|---|
| Admin | everything | everything |
| Mod | Cheater, Player report, Help, Stuck, Other | claim, assign, escalate, reply, close, teleport, mark the answer, warn, kick, staff notes |
| Helper | Help, Stuck | claim, escalate, reply, close, teleport |
| Developer | Bug, Other | claim, escalate, reply, close, teleport, mark the answer |

## 2. Change who sees what

- **Kinds of ticket**: `/poggy` → Tickets → **Kinds & routing**. Add, rename or remove kinds. Never change a kind's **id** once tickets exist.
- **Who sees which kind** is part of each **role** now: staff panel → **Staff** → **Roles** → Edit, or `/poggy` → Tickets → **Roles, chat & replies**. Made a new kind? Tick it on the roles that should see it, or only admins will.

## 3. Discord

1. In Discord: channel settings → Integrations → Webhooks → New webhook → Copy URL.
2. `/poggy` → Tickets → **Discord** → paste it into **Webhook**.
3. Restart the script.

Each ticket is **one post**. It is red when open, amber when claimed, green when closed. The post is edited, never deleted.

Want bugs in their own channel? Add `bug` and that channel's webhook under **Webhook per kind**.

## 4. Keys

Page Up opens the form; Home opens the staff panel. Change them in `/poggy` → Tickets → **Opening & keys**. Players cannot rebind them, so pick keys nothing else uses. `0` turns a key off.

## Bans

Bans belong to **poggy_core**, not this script. They keep working if this script is stopped. You can also ban from the console: `poggycore ban`, `poggycore unban`, `poggycore baninfo`.
