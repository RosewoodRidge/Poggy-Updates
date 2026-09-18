# Using the settings hub

Type **`/poggy`** in chat to open the hub. Every Poggy script on the server has a card.

## Who can open it

- Players with the ACE `poggy.settings`, **or**
- anyone your framework counts as an admin.

```
add_ace group.admin poggy.settings allow
```

To restart scripts from the hub, `server.cfg` also needs:

```
add_ace resource.poggy_core command.ensure allow
```

## Changing a setting

1. Click a script's card.
2. Pick a tab on the left, or search at the top.
3. Change the value. Hover the **ⓘ** to read what it does.
4. Click **Save**, or **Save & Restart** to load the change now.

Most changes need the script restarted before they take effect. The hub tells you when.

- **Advanced** settings are hidden until you turn on *Show advanced*.
- A **changed** badge means the value differs from the script's shipped default. **Reset** puts the default back.
- **History** lists every change: who, when, old and new value. You can undo from there.

## What the hub does to your files

The hub changes the script's own `config.lua` (and any other config file). It changes only the lines you edited. Every comment and every other line stays exactly as it was.

Before each save, the old file is copied to `poggy_core/update_backups/`.

## One editor at a time

Only one person can edit a script at a time. If someone else has it open, you see it read-only with their name.

- After a few idle minutes the script is freed and unsaved changes are dropped. You are warned first.
- Players with the ACE `poggy.settings.takeover` can take a script over. The other editor's unsaved changes are dropped.
- The console command `poggycore settings unlock <id>` frees a stuck script.

**Do not edit a script's config file by hand while someone is editing it in the hub.** The hub refuses to save over a file that changed on disk, and you would have to start again.

## poggy_core itself

poggy_core's own settings are saved like any other, but the hub never restarts poggy_core. Restart it by hand when the server is quiet: restarting poggy_core restarts every Poggy script.
