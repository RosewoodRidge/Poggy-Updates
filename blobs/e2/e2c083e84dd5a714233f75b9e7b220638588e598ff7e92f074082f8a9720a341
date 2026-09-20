# Poggy Tickets

Players ask for help. Staff claim it. Works on VORP, RSG Core and QBCore RedM through poggy_core.

- A **ticket window** for players: a key (Page Up) or `/ticket`. It opens on two big buttons, **I need help right now** and **Send a ticket**, so the choice is obvious.
- A **staff panel**, a large window and not full screen: live ticket ages, green for online players and grey for offline, colour for every kind and priority, filters, claim, assign, a chat with the player, one-press teleports.
- An **audit timeline** for admins: grouped by day, filtered by area, person or ticket.
- **Warn, kick and ban**, from a ticket or with no ticket at all: the **Players menu** (`/mod`, the Players tab, or the button in `/poggy`). Bans are poggy_core's account bans.
- **Help requests**: one button, no form, calls staff to the player.
- **Discord**: one post per ticket, edited as the ticket moves, never deleted.
- **Roles**: admin, mod, helper, developer. Staff are added in the panel.

## Requirements

- **poggy_core 0.19.0 or newer**
- oxmysql
- OneSync (for "Go to player")

## Install

1. Put `poggy_tickets` in your resources folder.
2. Add `ensure poggy_tickets` to `server.cfg`, after `poggy_core`.
3. Start the server. The database tables are made for you.
4. In game, type `/tickets` and add your staff on the **Staff** tab. Anyone your framework counts as an admin can already do this.

## Commands

| Command | Who | What |
|---|---|---|
| `/ticket` | Everyone | Opens the ticket form: send a ticket, call for help, read and answer your tickets. Page Up does the same. |
| `/tickets` | Staff | Opens the staff panel. Page Down does the same. |
| `/mod [id]` | Mods, admins | Opens the **Players menu**: everyone online with their record, and Warn, Kick, Ban. No ticket needed. |
| `/invis` | Staff | Turns your invisibility on or off. You cannot be hurt while invisible. Logged. |
| `/staffduty` | Staff | Off duty, or back on. Always on again after a relog. |

Every command name and both keys can be changed in `config.lua` or in `/poggy`.

## For players

- Pick what it is about and how urgent, say what happened. For reports, pick the player from those nearby (100 m) and add a video link if you have one.
- Your position is saved with the ticket.
- If no staff are on, you are told, and the ticket is still saved.
- A pop-up at the bottom right, with a sound, tells you when staff take, answer or close your ticket.
- **I need help right now** calls staff without a ticket: for being stuck, downed or lost.
- Limits: one ticket every 15 minutes, two open at a time, one help request every 5 minutes.

## For staff

| Role | Sees | Can |
|---|---|---|
| Admin | everything | everything, and ban, lift bans, add staff, read the audit trail |
| Mod | Cheater, Player report, Help, Stuck, Other | claim, assign, reply, close, teleport, warn, kick |
| Helper | Help, Stuck | claim, reply, close, teleport |
| Developer | Bug, Other | claim, reply, close, teleport |

- A role is on the **account**, so it covers every character. One person can hold several roles.
- **Claim** makes a ticket yours so only one person works it. If you log off, it frees itself after 15 minutes.
- **Go to player** and **Go to location** take one press. You arrive behind the player, invisible and unable to be hurt. **Return** takes you back.
- **Duty**: on by default. Off duty means no sounds, pop-ups or badge. It is never saved: a relog puts you back on duty.
- **Warn** shows the player a full-screen warning they must acknowledge (they cannot be hurt while it shows). **Kick** and **Ban** need a reason. The reported player is never told who reported them.
- The **audit trail** records every staff action. Only admins read it.

## Config

Everything is in `config.lua`, with a comment on every setting, and in `/poggy`.

| Setting | Default | What |
|---|---|---|
| `Config.OpenKey` / `Config.StaffKey` | Page Up / Page Down | The keys. `0` turns one off. |
| `Config.PlayerList` | `"nearby"` | Who can be picked as the reported player: `nearby`, `all`, or `ids`. |
| `Config.NearbyRadius` | `100` | Metres. |
| `Config.TicketCooldownMinutes` | `15` | |
| `Config.MaxOpenTickets` | `2` | |
| `Config.HelpCooldownMinutes` | `5` | |
| `Config.Categories` | six kinds | The kinds of ticket. Never change an `id` once tickets exist. |
| `Config.Routing` | | Which roles see which kind. |
| `Config.StaleClaimMinutes` | `15` | |
| `Config.SoundVolume` | `0.4` | |
| `Config.PurgeClosedAfterDays` | `0` | `0` keeps closed tickets forever. |
| `Config.Discord.Webhook` | blank | Your channel's webhook. |

## Discord

Paste a channel webhook into `Config.Discord.Webhook`. Each ticket is one post: red when open, amber when claimed, green when closed, with who closed it and how long it took. Player text can never ping anyone. `Config.Discord.CategoryWebhooks` sends one kind of ticket to its own channel.

## Bans

Bans are poggy_core's: on the account, reversible, and enforced even when this script is stopped. Admins lift them on the panel's **Bans** tab or with `poggycore unban`. See the **Bans** help page in poggy_core.

## Database

`poggy_tickets`, `poggy_ticket_staff`, `poggy_ticket_warnings`, `poggy_ticket_audit`, `poggy_ticket_help`. Made and kept up to date by poggy_core from `sql/install.sql`.
