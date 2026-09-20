# Bans

A ban is on the player's **account**. It follows them to every character they make.

## Ban someone

Three ways. They all do the same thing.

- **In `/poggy`:** open poggy_core, then **Bans**, then **Ban a player**.
- **In the console or chat:** `poggycore ban 12 7d combat logging`
- **From a script** that uses it, such as poggy_tickets.

The player is removed at once and reads your reason.

## How long

`30m`, `2h`, `7d`, `1w`, or `perm` for permanent.

## Local or cheating

- **Local** is a rule break on your server.
- **Cheating** is for cheats and exploits. Add the word `cheat`:
  `poggycore ban 12 perm cheat aimbot`

## Someone who already left

Ban their identifier: `poggycore ban license:1a2b3c perm cheat left before the ban`

## Lift a ban

- In `/poggy`: **Lift ban** on that row. You must say why.
- In the console: `poggycore unban 14 appeal accepted`

The ban stays in the list, marked as lifted, so you keep the history.

## Look someone up

- `poggycore baninfo` lists the active bans.
- `poggycore baninfo 12` shows every ban on record for player 12.

## The ban network

A **cheating** ban is shared with every server running poggy_core, as one vote. A player with votes from **3 servers** is refused on all of them. A **local** ban never leaves your server.

- Only scrambled identifiers are shared. Never a name or a reason.
- Lifting a ban takes your vote back.
- A new server's votes start counting after 14 days.
- It is **on by default**. Turn off either half in `/poggy` → poggy_core → **Bans**: *Refuse network-banned players*, *Share my cheating bans*.

**Someone the network blocks, that you want to let in:** the console prints their identifier when they are refused.

`poggycore banallow license:1a2b3c cleared on appeal`

**Status:** `poggycore bannet`

A blocked player is sent to **rosewoodridge.xyz/appeal**, where they log in with Cfx and see which servers banned them and how to appeal to each.

## If something breaks

Bans never lock your server. If the database cannot be read, **nobody is refused** and the console says so.
