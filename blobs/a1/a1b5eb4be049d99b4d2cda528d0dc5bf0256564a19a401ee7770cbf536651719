# Diagnostics commands

Run these in the **server console** as `poggycore …`, or in chat as `/poggycore …`.

In chat you need the ACE `command` or `poggycore`, or to be a framework admin. Commands that write files or change the database work in the console only.

## Is poggy_core working?

| Command | What it shows |
|---|---|
| `poggycore` | Version, framework, capabilities, registered containers. |
| `poggycore detect` | Why the framework did or did not resolve. |
| `poggycore resolve` | Runs detection again, without a restart. |
| `poggycore caps` | What your framework can and cannot do. |
| `poggycore scripts` | Every Poggy script that registered, with version. |
| `poggycore dependents` | Scripts that stop with poggy_core, and their state. |

## Does it work with my framework?

| Command | What it does |
|---|---|
| `/poggycore test` | Read-only check against your own character. Run it in game. |
| `poggycore selftest` | Tests every verb, read-only. |
| `poggycore selftest full` | Also tests the verbs that change things, and puts everything back. |
| `poggycore verbs [word]` | Lists the verbs scripts use. |
| `poggycore do <verb> key=value …` | Runs one verb for real. `src=me` means you. |

Example: `/poggycore do money.get src=me`

`do` really does what the verb says. `money.add` really gives money.

## Updates (console only)

| Command | What it does |
|---|---|
| `poggycore update all` | Lists scripts with a newer version. |
| `poggycore update all apply` | Installs them. Your config values are kept; backups go to `poggy_core/update_backups/`. |
| `poggycore update <script> apply` | Installs one. |
| `poggycore update nettest` | Checks the server can reach the update feed. |
| `poggycore update <script> writetest` | Checks poggy_core can write into that script's folder. |
| `poggycore catalog` | Published Poggy scripts you do not have, with store links. |

## Database (console only)

| Command | What it does |
|---|---|
| `poggycore sql check <script\|all>` | Shows what would be added to the database. |
| `poggycore sql install <script\|all>` | Adds it now. |

## Settings hub (console)

| Command | What it does |
|---|---|
| `poggycore settings` | Scripts, who is editing what. |
| `poggycore settings show <id>` | Every setting of a script, with file and line. |
| `poggycore settings set <id> <path> <value>` | Changes one setting, like the hub. Text values in double quotes. |
| `poggycore settings unlock <id>` | Frees a script someone is stuck editing. |

## Bans

| Command | What it does |
|---|---|
| `poggycore ban <id or identifier> <time> [cheat] <reason>` | Bans the account and removes the player. Time: `30m`, `2h`, `7d`, `1w`, `perm`. |
| `poggycore unban <ban number> <reason>` | Lifts a ban. It stays on record. |
| `poggycore baninfo [id, identifier or ban number]` | Active bans, or every ban on record for one player. |
| `poggycore bannet [refresh]` | The ban network's status on this server. |
| `poggycore banallow <identifier> <reason>` | Let one network-blocked player into your server. |

More on the **Bans** help page.

## Common fixes

- **Framework shows as standalone:** run `poggycore detect`. Your framework core may have started after poggy_core, or *Force a framework* is set.
- **Updates never install:** run `poggycore update nettest`, then check *Automatic updates* is on.
- **Scripts do not restart after an update or a hub save:** add `add_ace resource.poggy_core command.ensure allow` (and `command.refresh` for updates) to `server.cfg`.
