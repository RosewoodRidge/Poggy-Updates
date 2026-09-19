# Poggy Multijob

Let players hold several jobs and switch between them in one command.

Every job a character is given is remembered. Players switch from a menu or with `/mj <number>`. Staff get a full admin panel for online and offline characters, with one-click job presets.

---

## Features

- **Keeps every job.** Every job change on the server, from any script, admin command or boss menu, is saved to the character's list: both the job they left and the job they took. The job is also saved on join and each time they open the menu.
- **Server API.** Other scripts can add, remove and re-grade jobs on a character's list, online or offline, without switching their active job (see [For developers](#for-developers)). poggy_markets uses it to give shop staff their shop's job.
- **Shop jobs in /poggy.** With Poggy Markets installed, `/poggy` → **Poggy Multijob** → **Shop jobs** lists every shop and the job it gives its staff. Change a shop's job there and it applies at once: everyone hired at that shop gets the job added to their list here (their current job is left alone), and loses it when let go. Only server staff can change it (the same table is on the Poggy Markets page; `/pmshopjob` is the optional command).
- **Menu.** `/multijob` lists every job held, marks the active one, and offers Unemployed, Quit Job and Quit All Jobs.
- **Quick switch.** `/mj 1` jumps straight to another job.
- **Admin panel** (`/mjadmin`):
  - **Edit Job** — every character with saved jobs, online and offline. Set the active job, add, edit or remove jobs.
  - **All Jobs** — every saved job in one table. Search, filter by job, sort, see last login, remove.
  - **Job Presets** — ready-made jobs grouped by department. Pick one and apply it to a player.
- **Offline changes.** The admin panel can change the active job of an offline character on VORP, RSG and QBR.
- **Any framework.** Online job changes go through poggy_core, which tells other scripts about the change.

---

## Requirements

| Resource | Why |
|---|---|
| **poggy_core** 0.14.0 or newer | Characters, jobs, groups, the menu and notifications. Start it first. |
| [oxmysql](https://github.com/overextended/oxmysql) | Stores each character's jobs |
| A framework poggy_core supports | VORP, RSG or QBR |

No menu resource (such as vorp_menu) is needed.

---

## Installation

1. Put the `poggy_multijob` folder in your `resources` folder.
2. Add it to `server.cfg`, after poggy_core and oxmysql:
   ```
   ensure poggy_core
   ensure poggy_multijob
   ```
3. Restart the server.

The `poggy_multijob` table is created for you when the script starts. There is nothing to import.

If you manage the database yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`, then import `sql/install.sql` (and `sql/migrations/001.sql` when upgrading from 1.3.0 or earlier).

### Upgrading

**From the `multijob` folder (before 1.6.0)**

1. Copy your settings out of the old `shared/config.lua` and `shared/locales.lua`.
2. Delete the old `multijob` folder and add `poggy_multijob`.
3. In `server.cfg`, change `ensure multijob` to `ensure poggy_multijob`.
4. Put your settings into `config.lua` and `translations.lua` (both sit next to `fxmanifest.lua`).

Players keep their jobs: the table is still `poggy_multijob`, and the commands are unchanged. If your own scripts call `exports.multijob:GetJobs()` or listen for `multijob:` events, change them to `exports.poggy_multijob:GetJobs()` and `poggy_multijob:`.

**From 1.3.0 or earlier** — rows are copied from `marshal_multi_jobs` on the first start. The old table is kept so you can roll back; delete it when you are happy.

**From before 1.7.1** — the `cid` column changes from a number to text (`VARCHAR(64)`) so RSG citizen ids fit. This happens on the first start and keeps your data.

---

## Commands

| Command | Who | What it does |
|---|---|---|
| `/multijob` | Everyone | Opens the job menu. |
| `/mj <number>` | Everyone | Switches to one of your other jobs. `1` is the first job that is not your current one. Unemployed is always last. |
| `/mjadmin` | Groups in `Config.AdminGroups` | Opens the admin panel. |

`/multijob` and `/mj` can be renamed (`Config.Command`, `Config.SwitchCommand`). `/mjadmin` is fixed.

---

## Configuration

Every setting can be changed in game with **/poggy** (Poggy Hub). You can also edit `config.lua` and `translations.lua` by hand. Restart the script after a change.

| Setting | Default | What it does |
|---|---|---|
| `Config.Command` | `'multijob'` | Command that opens the job menu. |
| `Config.SwitchCommand` | `'mj'` | Command that switches job by number. |
| `Config.DefaultJob` | `'unemployed'` | Job given after Quit Job or Quit All Jobs. |
| `Config.DefaultGrade` | `0` | Grade given with it. |
| `Config.DefaultJobLabel` | `'Unemployed'` | Label given with it. |
| `Config.AdminGroups` | `admin`, `superadmin`, `moderator` | Groups that may use `/mjadmin`. |
| `Config.JobPresets` | Law, Medical, Business, Shops | Ready-made jobs for the admin panel. |
| `Config.Debug` | `false` | Prints extra detail to the consoles. |

These settings are in the file but **not used** by the current version:

| Setting | Note |
|---|---|
| `Config.MaxJobs` | No limit is enforced. Remove jobs in the admin panel if needed. |
| `Config.Webhook` | Nothing is sent to Discord. |
| `Config.CommandDescription`, `Config.SwitchCommandDescription` | No chat suggestions are registered. |

### Job presets

Presets are grouped. Each group has a `name` and a list of `jobs`:

```lua
Config.JobPresets = {
    police = {
        name = "Law Enforcement",
        jobs = {
            { job = "sheriff", label = "DEPUTY", grade = 1, category = "Sheriff" },
            { job = "sheriff", label = "SHERIFF", grade = 6, category = "Sheriff" },
        }
    },
}
```

| Field | What it is |
|---|---|
| `job` | Job name. Must match your server's job exactly (case-sensitive on VORP). |
| `label` | Name shown for the job. |
| `grade` | Grade (rank number). |
| `category` | Subheading inside the group, for example `Sheriff` or `Saloons`. |

The shipped **shop owner** jobs are examples, not stock jobs. Rename or delete them to match your server.

The panel's category filter knows the group keys `police`, `medical`, `business` and `shops`. A group with any other key still shows under **All Categories**.

### Text

All player messages are in `translations.lua` (`Locales`). Keep every `%s`: the script fills in the job name or number there.

---

## Permissions

`/mjadmin` and every admin action are checked on the server. A player is allowed when any of their groups (account or character) is in `Config.AdminGroups`. Group names are not case-sensitive.

No ACE is needed.

---

## Troubleshooting

**A new job does not show in the menu.**
Every job change is saved as it happens (poggy_core reports it), and the current job is saved again on joining and each time `/multijob` or `/mj` is used. A job given while poggy_multijob was stopped is picked up the next time the character opens `/multijob` while wearing it.

**Jobs are not saving.**
- Look for a red `poggy_multijob database:` line in the server console when the script starts.
- Check that oxmysql is running.
- Turn on `Config.Debug` for more detail.

**The admin panel does not open.**
Your account or character group must be in `Config.AdminGroups`.

**"Offline job changes are not supported on this framework."**
Online players can be changed on every framework. Offline players are written straight into the framework's own table, which works on VORP (`characters`), RSG and QBR (`players`) only. Their multijob entries are still saved and apply next time they switch.

---

## For developers

- Export: `exports.poggy_multijob:GetJobs()` (client) returns the cached job list.
- Job changes go through poggy_core's `job.set` with `persist = true`. poggy_core fires its own job-changed events for other scripts.
- poggy_multijob listens to poggy_core's server event `poggy_core:jobChanged` (`src, job, grade, oldJob, oldGrade`), which poggy_core raises for every framework (VORP's job and grade events, RSG's `OnJobUpdate`, QBR's re-check). Both jobs are saved to the list; unemployed and empty jobs are skipped.

### Server API

poggy_multijob owns every character's job list. Other scripts ask it through
these server exports instead of writing the `poggy_multijob` table:

```lua
local ok, reason = exports.poggy_multijob:AddJob(target, job, grade, label)
local ok, reason = exports.poggy_multijob:RemoveJob(target, job)
local ok, reason = exports.poggy_multijob:SetJobGrade(target, job, grade)
local has, reason = exports.poggy_multijob:HasJob(target, job)
```

| Export | What it does |
|---|---|
| `AddJob(target, job, grade?, label?)` | Adds the job to the list **without switching the active job**. `grade` defaults to 0, `label` to the job name. Already on the list: its grade (and label, when given) are updated, as `SetJobGrade` does. |
| `RemoveJob(target, job)` | Removes the job from the list. If it is the job the character wears, they are switched to `Config.DefaultJob`: online through poggy_core, offline in the framework's own table (VORP, RSG, QBR). |
| `SetJobGrade(target, job, grade)` | Changes the job's grade on the list, and the active grade when it is the job they wear (online or offline). |
| `HasJob(target, job)` | `true` when the job is on the list, or is the job an online player wears. |

**`target`** is either an online player's **server id (a number)** or a
**character id (a string)**: the VORP `charidentifier` as text, or the RSG/QBR
`citizenid`. A character id also works for offline characters. A VORP
character id must be passed as a string, `tostring(charId)`; a number is always
read as a server id.

**Call them from a thread** (`CreateThread`): they wait on the database.

**Return values.** Every export returns `ok` and a reason:
`'added'`, `'updated'`, `'unchanged'`, `'removed'` on success;
`'not_ready'`, `'bad_target'`, `'bad_job'`, `'bad_grade'`, `'not_online'`
(a server id nobody is using), `'no_character'` (no such character),
`'not_found'` (not on the list) or `'db_error'` on failure. `HasJob` gives a
reason only when it returns `false`.

Unemployed (and `none`, `unknown`, empty) can never be added. `Config.MaxJobs`
is not enforced here either.

---

## Changelog

- **1.7.3** — Every job change is captured: poggy_multijob listens to poggy_core's job-change relay (any framework, any script or admin command) and saves both the job left and the job taken. New server API for other scripts: `AddJob`, `RemoveJob`, `SetJobGrade`, `HasJob`, taking a server id or a character id (offline characters too). A removed or quit job is no longer saved straight back by the job change that follows it. Offline active-job changes now also work on QBR. The `/poggy` page has a **Shop jobs** tab showing Poggy Markets' shop jobs (the data and edits belong to poggy_markets). Saving a job is a single statement, so two saves at once cannot collide.
- **1.7.2** — Poggy Hub support: settings, commands and help pages in `/poggy`. No gameplay changes.
- **1.7.1** — `cid` is `VARCHAR(64)` (was `INT(11)`) so RSG citizenids fit; an existing table is converted on the first start and the server compares character ids as text everywhere.
- **1.7.0** — Framework-agnostic: the `/multijob` menu is drawn by poggy_core (`menu.open`, poggy_core 0.14.0), so vorp_menu is no longer required; every job change for an online player goes through poggy_core (`job.set` with `persist`), which also relays the change to other scripts; the admin panel now updates offline players' active job too (VORP and RSG).
- **1.6.0** — Renamed to `poggy_multijob`; poggy_core creates the table and copies rows from `marshal_multi_jobs`.

---

## Support

Join the Poggy Scripts Discord: https://discord.com/invite/rBarFeuzFj

## Credits

- Original concept by Marshal
- Modernised and extended by Poggy
