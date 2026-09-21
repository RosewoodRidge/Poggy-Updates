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
| `/tickets` | Staff | Opens the staff panel. Home does the same. |
| `/mod [id]` | Staff whose role may warn, kick or ban | Opens the **Players menu**: everyone online with their record, and Warn, Kick, Ban. No ticket needed. |
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

### Roles are yours

A new server starts with Admin, Mod, Helper and Developer, but they are only rows you can change. Make *Staff Manager* or *Event Team*, name and colour it, and tick:

- what it **may do** (18 powers, from *claim* to *manage roles*),
- which **kinds of ticket** it sees,
- whose **staff chat** it reads,
- whether it hears **help requests**.

Edit roles in the panel (**Staff → Roles**) or in `/poggy` (**Roles, chat & replies**). No restart. One person can hold several roles. **Admin is locked**: it cannot be deleted or stripped, so nobody can lock the team out, and anyone your framework counts as an admin holds it.

| Starts as | Sees | Can |
|---|---|---|
| Admin | everything | everything |
| Mod | Cheater, Player report, Help, Stuck, Other | claim, assign, escalate, reply, close, teleport, mark the answer, warn, kick, staff notes |
| Helper | Help, Stuck | claim, escalate, reply, close, teleport |
| Developer | Bug, Other | claim, escalate, reply, close, teleport, mark the answer |

### On a ticket

- **Escalate** adds another role to a ticket without taking it off its owner. **Assign** hands it over; escalate brings help in.
- **Internal notes**, in dark red, are for staff. They are removed on the server before anything is sent to the player.
- **Ready-made replies**, offered by keyword from what the player wrote. Eight ship with the script.
- **Mark the answer**: pinned in green for staff and player.
- **Red bubbles** on the chips and tabs count what is still waiting.
- **Full screen**, per staff member.

### Staff chat

A room per role, plus any rooms you make. Admins read all; mods read Mod and Helper; helpers read Helper; developers have their own. All of that is on the role, so change it. **Messages are never saved**: memory only, gone at the next restart. A room with a Discord webhook is copied there.

### Solved questions

Staff can make a closed ticket with a marked answer **public**. Any player can then read it, with every name hidden, and the form suggests matching answers while a player types, so a question already answered never becomes a ticket. Off by default for every ticket, never possible for a ticket about a player, and `Config.PublicAnswers = false` removes the feature.

### Archive, and delete

**Archive** is a role power: the ticket leaves every list but the Archived filter, and nothing is destroyed. **Delete** is not a role power. Only the server owner can grant it, in `server.cfg`, and only an archived ticket can be deleted:

```
add_ace identifier.steam:110000112345678 poggy_tickets.delete allow
```

The audit trail keeps who deleted which ticket, and when.

### Tickets on the website (off by default)

At **rosewoodridge.xyz/tickets**, signed in with Cfx.re, with your community's one ID (shown to everyone at the top of `/ticket` and `/tickets`):

- **players** read and answer their own tickets, under each character, and write new ones when they are not in game;
- **anyone with a ticket role** gets the staff desk as well: read, reply, internal notes, claim, release, close. **Warn, kick and ban stay in game** unless you switch on `Config.Web.Moderation`. With `Config.Web.VerifyDevices` (on by default) each browser must be confirmed once with a link code from the game before it opens the staff desk, so a stolen Cfx.re login is not enough.
- **banned players** can appeal to you by name at **rosewoodridge.xyz/appeal**; an appeal arrives as a *Ban appeal* ticket.

A person is linked to their Cfx.re account by itself at login when their game names one, or with a six-character **link code** from the **Website** tab of `/ticket`, typed on the website. Turn it on with `Config.Web.Enabled`, and read the note above it first: your server cannot talk to a website directly, so copies of your **open tickets** are held on a relay run by Rosewood Ridge while they are open and for 7 days after, with the Cfx.re id, name and character names of each person who uses the website. Switch it off and the relay forgets your community at once. Your own server checks every web action again. See `docs/help/web.md`.

### Players

The record shows what the **ladder** suggests next (a suggestion, never an action), which old warnings **no longer count**, **staff notes** the player never sees, and a **watch** flag that tells staff when that player logs in.

## Config

Everything is in `config.lua`, with a comment on every setting, and in `/poggy`.

| Setting | Default | What |
|---|---|---|
| `Config.OpenKey` / `Config.StaffKey` | Page Up / Home | The keys. `0` turns one off. (The staff key was Page Down until 1.1.0; vorp_admin uses that one.) |
| `Config.PlayerList` | `"nearby"` | Who can be picked as the reported player: `nearby`, `all`, or `ids`. |
| `Config.NearbyRadius` | `100` | Metres. |
| `Config.TicketCooldownMinutes` | `15` | |
| `Config.MaxOpenTickets` | `2` | |
| `Config.HelpCooldownMinutes` | `5` | |
| `Config.Categories` | six kinds | The kinds of ticket. Never change an `id` once tickets exist. |
| `Config.Routing`, `Config.HelpRoles` | | **Old.** Read once when 1.1.0 first starts, then the roles are the truth. |
| `Config.ShowExpectedWait` | `true` | Tell players how long their kind of ticket usually waits. |
| `Config.PublicAnswers`, `Config.SuggestAnswers`, `Config.PublicKinds` | on | Solved questions: see below. |
| `Config.NudgeAfterHours` / `Config.AutoCloseAfterHours` | `24` / `48` | "Still need help?", then close as *No response*. `0` turns either off. |
| `Config.StaffFullscreen` | `false` | How the panel starts. Each staff member can switch it. |
| `Config.ChatPopups` | `true` | A pop-up for a new staff chat message. |
| `Config.WarningDecayDays` | `90` | When a warning stops counting toward the ladder. `0` is never. |
| `Config.Ladder` | warn, warn, kick, ban 1d, 7d, perm | What to *suggest* for a player's next offence. |
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
