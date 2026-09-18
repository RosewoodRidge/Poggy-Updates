# Roles and job lists

Many Poggy scripts have their own job lists: who counts as a lawman, a doctor, or staff. A **role** is one master list you can link those settings to, so you change the names in one place.

## The roles

Open **Roles** from the top of the hub. poggy_core ships with three:

| Role | Kind | Names |
|---|---|---|
| Lawmen | jobs | sheriff, deputy, marshal |
| Medics | jobs | doctor |
| Staff groups | groups | admin, superadmin, god |

- **jobs** roles hold job names. **groups** roles hold admin group names.
- Names must match your framework exactly.
- You can add your own roles.

## Linking a setting to a role

1. Open a script and find a job or group list (it shows as chips).
2. Click **Link to role** and pick the role.
3. Save.

The setting's list is replaced by the role's list.

## Changing a role

1. Open **Roles** and edit the names.
2. Click **Save**. The hub lists every script that will change.
3. Confirm.

Every linked setting in every script is rewritten, and those scripts restart. A script that someone else is editing is skipped, and you are told which.

## Unlinking

Click **Unlink** on the setting and save. The list keeps its current names, and later role changes no longer touch it.

## How the link is stored

A link is a short comment at the end of the setting's line in the script's config, for example:

```lua
jobs = { "sheriff", "deputy" }, -- poggy:role lawmen
```

The scripts never read it; only the hub does. Delete the comment to unlink by hand.

## Law jobs and medical jobs in poggy_core

`Law jobs` and `Medical jobs` on poggy_core's **Access & permissions** tab are used by every Poggy script that asks "is this player a lawman / a medic?". You can link them to a role too.
