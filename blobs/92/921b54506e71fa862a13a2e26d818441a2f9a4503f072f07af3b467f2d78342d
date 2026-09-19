# Setting up a job storage

A job storage is a stash you define in the config, such as police evidence or a
doctor's cabinet. Nobody buys it. It never expires. The job decides who opens it.

## Add one

1. Open the **Locations** tab, then **Job storages**, and choose **Add job storage**.
2. Give it a unique **Id**, such as `ranch_hands`.
3. Give it a **Name**. Players see it on the prompt and the blip.
4. Add a **Location** for every spot it can be opened from. Stand there and use
   "use my position".
5. Set the **Capacity**.
6. Save and restart the script.

> Do not change the **Id** later. The items are stored under it, so a new id
> shows an empty storage.

## Who can open it

| Setting | What it does |
|---|---|
| **Jobs with access** | `["sheriff"] = { all_grades = true }` lets every sheriff in. `{ grades = { 2 } }` lets in grade 2 and above. Job names are case-sensitive. |
| **Character ids with access** | Lets named characters in, whatever their job. |
| **Open to everyone** | `public_access = true` lets anyone in. |

The player's job grade then sets what they can do inside:

| Grade is in | Access |
|---|---|
| **Manager grades** (Access & permissions tab) | Open, deposit, withdraw, ledger, upgrade |
| **Member grades** | Open, deposit, ledger |
| Neither | Open only |

## One inventory or one per location?

**One shared inventory** on (the default) means every location opens the same
stash. That is what most job storages want.

Turning it off gives each location its own stash. That needs a different
layout, so edit it in `config.lua`. The commented example at the end of
`Config.DefaultStorages` shows it (`id_prefix`, `name_template`, and locations
written as `{ coords = vector3(...), name_detail = "Valentine" }`).

## Discord logging

Each job storage can post its contents and a who-took-what log to Discord.

1. Paste a channel webhook into **Webhook** and turn **Discord tracking on**.
2. Restart the script.
3. The server console prints two message ids. Paste them into
   **Inventory message id** and **Activity message id**, so the same messages
   are edited from then on.

## Clerks

The **Clerks** list places NPCs for decoration, such as a clerk behind the
evidence desk. They cannot be hurt and do not move. Opening the storage is
still done at its location.
