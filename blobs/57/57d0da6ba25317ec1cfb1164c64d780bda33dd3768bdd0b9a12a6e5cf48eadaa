# Who can see the blips

Two settings decide who gets player blips. Both are on the **Access & permissions** tab.

## Admin groups

A player gets blips when they are in one of the **Admin groups**.

- The groups come from your framework, through poggy_core (for example `admin` or `superadmin` on VORP).
- Upper or lower case does not matter.
- The same list decides who may use `/ahb`.

To add a moderator group, add `moderator` to the list and save.

## Allowed player names (optional)

Leave this list empty to give blips to every admin.

When it has names in it, blips go **only** to those players, and they must **also** be in an Admin group. A name on its own is not enough.

- Use the exact Steam / RedM display name.
- Names are case-sensitive: `Poggy` and `poggy` are different.

## When a change takes effect

The check runs once, a few seconds after a player joins. After you change either list:

1. Save and restart the script (or use **Save & Restart**).
2. Admins who are already online get blips again a few seconds after the restart.

## Blips still missing?

1. Turn on **Debug output** (Advanced tab) and restart the script.
2. Rejoin and watch the server console. It says which groups it found for you and whether you were registered.
3. If it shows no groups, your character may be loading slowly. Raise **Wait after joining**.
4. Turn **Debug output** off again when you are done.
