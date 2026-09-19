# Poggy's Supply Drops & Scavenger Hunts

World events for RedM that send players racing across the map.

- **Supply drops** float down under a hot air balloon at a random place. The first player to reach the crate claims the goods.
- **Scavenger hunts** hide a treasure somewhere in the world. Players get a photo of the hiding place and a riddle. Revealing the riddle costs part of the reward.

Both run on their own schedule. Admins can also start them by command.

## Features

- Balloon drops that descend, release the crate and fade away.
- 10 drop locations and 30 scavenger hunts, each with its own photo and riddle, ready to use.
- Weighted loot tables for drops, with weapons supported.
- Cash, gold or item rewards for hunts.
- A clue penalty: reveal the riddle and lose part of the reward.
- Map blips, notifications and five colour themes for the hunt window.
- Separate Discord webhooks for drops and hunts.
- Works on every framework poggy_core supports.

## Requirements

| Resource | Needed |
|---|---|
| `poggy_core` 0.13.0 or newer | Yes. It runs the framework for this script. |

No database tables are used.

## Installation

1. Put the `poggy_supplydrops` folder in your resources folder.
2. Add `ensure poggy_supplydrops` to `server.cfg`, after `ensure poggy_core`.
3. Check that every item in the drop loot and hunt rewards exists on your server (see [Troubleshooting](#troubleshooting)).
4. Restart the server.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/supplydrop` | Admins, console | Starts a supply drop now. It also starts the drop cooldown. |
| `/scavengerhunt` | Admins, console | Starts a scavenger hunt now. It also starts the hunt cooldown. |
| `/nextdrop` | Everyone | Shows how long until the next drop and hunt. Admins can add `detailed` to see the clock time. |
| `/clue` | Everyone | Opens the current hunt: photo, reward and the hidden riddle. |
| `testdiscord [supply\|scavenger]` | Server console | Sends a test message to a Discord webhook. |

**Admins** are decided by poggy_core (your framework's admin groups). There is no separate setting in this script.

## How it plays

### Supply drops

1. A drop starts at a random location. Every player gets a notification and a map blip.
2. Players close by see the balloon come down. Players far away find the crate already landed.
3. The first player to hold **G** at the crate collects it.
4. An uncollected drop disappears after 15 minutes.

### Scavenger hunts

1. A hunt starts at a random hunt location. Every player gets a notification and a map blip.
2. Players type `/clue` to see the photo and the reward. The riddle is blurred.
3. Revealing the riddle lowers the reward (30% by default).
4. The first player to hold **G** at the treasure claims it.
5. An unclaimed hunt disappears after 45 minutes.

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy Hub). You can also edit `config.lua` and `translations.lua` by hand. Restart the script after a change.

| Setting | What it controls |
|---|---|
| `Config.EnableSupplyDrops`, `Config.EnableScavengerHunts` | Turn each feature on or off. |
| `Config.RandomEvent` | The automatic schedule: waits and cooldowns, in milliseconds. |
| `Config.Supply` | Balloon behaviour, blip, distances, how long a drop lasts, the loot and the locations. |
| `Config.Scavenger` | Hunt window theme, clue penalty, blip, distances, how long a hunt lasts, the rewards and the hunts. |
| `Config.NotificationDuration`, `Config.ShowCollectedByNotifications` | How long collect messages stay, and whether everyone hears who collected a drop. |
| `Config.Discord` | One webhook for drops, one for hunts. |
| `translations.lua` | Every message. English, Spanish and French ship; pick one with `Config.Language`. |

All times in `Config.RandomEvent`, `DropTimeout` and `HuntTimeout` are in **milliseconds** (60000 = 1 minute).

The Hub has help pages for the schedule, adding drop locations and loot, and adding scavenger hunts.

### Loot weights

The `chance` of each loot or reward entry is a **weight**, not a percent. An entry with 20 is twice as likely as one with 10. Use whole numbers.

### Hunt photos

Put each hunt's photo in `ui/images/` as a **.jpg** file, and set the hunt's `image` to the file name. Only `.jpg` files are sent to players.

## Discord

Each webhook can post:

- New supply drops, and who collected them.
- New scavenger hunts, and who found them (with or without the clue).

Turn on `Enabled` and paste a real webhook URL. Nothing is sent while it still says `YOUR DISCORD WEBHOOK HERE`.

## Troubleshooting

**No drops or hunts ever start.**
Check `Config.RandomEvent.Enabled` and the two feature switches. Events start only after the first player has loaded in. Use `/nextdrop` to see the timers.

**A drop cannot be collected: the prompt keeps coming back.**
The item probably does not exist in your items table, or the player's inventory is full. Every loot `name` must be a real item.

**The hunt photo is a broken image.**
The file is missing from `ui/images/`, is not a `.jpg`, or the name in the hunt does not match. Restart the script after adding a photo.

**Some scavenger hunts never start.**
Check the money amounts in the hunt rewards. Use whole dollars or halves (16 or 16.5). Some cent values cannot be rolled, and the hunt fails to start.

**The crate is invisible.**
The `prop` name is wrong. Use a valid RedM object model.

## Changelog

- **1.5.1** — Poggy Hub support: settings, lists and help pages for `/poggy`. The timings in `config.lua` are written as plain milliseconds instead of sums (`600000` rather than `10 * 60 * 1000`); the values are the same. The comments now give the real cooldown ranges.

## License

© 2023 Poggy's Scripts. All rights reserved.

This resource is for use on your RedM server only. Redistribution, resale, or modification beyond configuration is not permitted without explicit permission.
