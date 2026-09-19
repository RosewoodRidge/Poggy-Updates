# Managing storages in /poggy

The **Storages** tab is the main admin tool. It lists every storage on the
server and changes them at once: no restart, no commands to remember.

Open **/poggy** → **Poggy Storage** → **Storages**.

## What you see

| Column | What it is |
|---|---|
| **ID** | The storage's number (player storages) or its config id (job storages). |
| **Name** | What players see on the prompt and the inventory window. |
| **Kind** | **Player** for a bought storage. **Job**, **Public** or **Preset (config)** for the ones in `Config.DefaultStorages`. |
| **Owner** | The owner's name and character id. |
| **Location** | Where it is. A job storage with several places shows the first and how many more. |
| **Slots** | How much it holds. |
| **Upgrades** | How many upgrades the owner has bought. |
| **Access** | How many characters have a key. |
| **Jobs** | Jobs whose members can open it. |

Search and sort work as in any list.

## Change a player storage

- **Rename:** click the name and type. Up to 50 characters; `'`, `;` and `-` are removed.
- **Resize:** click the slots and type a number, at least 1. Lowering it below
  what is inside keeps the items, but nothing more fits until it is under the size.
- **Move to my position:** stand where it should be, press the row's button and
  choose "Use my position". Same as `/movestorage`.
- **Change owner:** give a character id. You type the storage id to confirm.
  The new owner loses any separate key they had, because they own it now.
- **Delete storage:** you type the storage id to confirm. Same as `/deletestorage`.

> **Deleting does not empty it.** Like `/deletestorage`, it unregisters the
> container: the items stay in the inventory database, but nobody can reach
> them. If they should go, open **Contents** first and choose **Empty**.

## Who has a key

Click a storage to open its access list.

- **Give access:** a character id and a level (Basic, Member or Manager).
- **Change a level:** pick it in the **Level** column.
- **Remove access:** the row's button.

The characters' storage menus update at once if they are online, and they get
the same "access granted" or "access removed" message as when the owner does it.
Job rules are shown but are set by the owner in the storage menu.

## What is inside

**Contents** on any row, including job storages, shows the items (and weapons)
in that storage. You can add an item, remove some, or empty it. This is drawn by
poggy_core and works on every storage.

## Job storages

Job storages come from `config.lua` (**Locations** tab → **Job storages**). In
this panel they are read-only apart from **Contents**: change their name, size,
places and access in the config, then restart the script.

## Good to know

- Only people who may edit settings in `/poggy` can use this panel.
- Every change is printed to the server console with who made it, and shows in
  `/poggy`'s **History**.
- Player storages nobody opens for **Unused for** days are deleted when the
  script starts, if **Delete unused storages** is on (General tab → Expiry).
