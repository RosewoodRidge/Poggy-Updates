# Poggy Fishing

Rod fishing for RedM. Players cast, tug the line to hook a fish, then win a skillcheck fight to land it.

- 28 fish across 34 waters, with day and night feeders.
- Bait that changes what bites.
- Random seasons, so the catch changes every restart.
- A Pro Rod with a mouse-driven pull fight.
- Bonus loot on every fish kept.
- A full HUD with sound, music and two looks (brass or default).

Works on every framework poggy_core supports (VORP, RSG, QBR).

## Requirements

| Resource | Why |
|---|---|
| `poggy_core` 0.13.0 or newer | Framework layer: items, money, notifications |
| `poggy_skillcheck` | The skillcheck circle |
| `oxmysql` | Database |
| `poggy_fishing_journal` (optional) | Records discoveries; needed for "hide fish not yet caught" |

## Installation

1. Put `poggy_fishing` in your resources folder.
2. Add `ensure poggy_fishing` to `server.cfg`, after its requirements.
3. Copy the item icons from `docs/item_images/` into your inventory's image folder.
   On VORP that is `vorp_inventory/html/img/items/`.
4. Restart the server. The items are added to the database for you.

### Database

`sql/install.sql` runs by itself when the script starts. It adds the rods, baits, fish and loot items.
Items you already have are never changed.
To import it yourself, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

On RSG (no `items` table) the items must go in `rsg-core/shared/items.lua` by hand:

- Rods: `fishingrod`, `fishingrod_pro`
- Baits and lures: `bait_worm`, `bait_cricket`, `bait_bread`, `bait_crawdad`, `p_lgoc_spinner_v4`, `p_finishedragonflylegendary01x`
- Every fish item in the Fish list (`a_c_fish…`)
- Loot: `whitepearl`, `redpearl`, `bluepearl`, `goldpearl`, `blackpearl`, `golden_nugget`, `diamond_uncut`

At start the script prints one yellow line naming any item your list is missing.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/rerollseasons` | ACE `command.rerollseasons`, or the server console | Picks a new random set of in-season fish now |

To let a group use it: `add_ace group.admin command.rerollseasons allow`

## Controls

| Key | Action |
|---|---|
| Use the rod item | Equip the rod |
| **Left mouse** | Cast |
| **E** | Tug the line (normal rod) |
| **G** | Bait menu |
| **D** | Next music track |
| **Backspace** | Stow the rod |
| **Mouse** | Counter the fish's pull (Pro Rod) |
| **E** / **Backspace** after a catch | Keep the fish / throw it back |

## How a catch works

1. **Equip.** Use `fishingrod` or `fishingrod_pro`. The HUD shows the water and its fish.
2. **Cast.** Left mouse. The splash is seen by nearby players.
3. **Bite.**
   - Normal rod: after 3 to 20 seconds the tug game starts. Tap **E** to raise interest (blue bar).
     Each tap adds tension (red bar). Interest 100 hooks the fish. Tension 100 snaps the line.
   - Pro Rod: no tug game. A fish bites after 4 to 15 seconds.
4. **Fight.** Skillchecks fill the progress bar. The fish is landed at 100.
   - Misses take progress away. At 0 the fish escapes.
   - Misses in a row make the needle faster.
   - Normal rod: 4 misses snap the line.
   - Pro Rod: the fish also pulls left, right or forward. Move the mouse to counter it.
     Countering well slows the drain and eases tension. Tension 100 snaps the line.
5. **Keep or throw back.** A kept fish goes to the inventory with a random weight, plus a loot roll.

### Normal rod and Pro Rod

| | Normal rod | Pro Rod |
|---|---|---|
| Before the bite | Tug game | Random wait |
| Fight | Skillchecks | Skillchecks and pull fight |
| Progress per hit | Full | 65% good, 75% great |
| Miss penalty | Normal | 50% more |
| Bigger fish | Normal odds | Shifted toward large and legendary |
| Loot rolls | 1 | 2 |
| Bait loss when a fish escapes | Normal | Halved |
| Line snaps on | 4 misses | Tension 100 |

## Baits

Pick a bait with **G**. Each bait has:

- **Bonus**: makes the skillcheck zone larger (0 to 2). 0 counts like no bait.
- **Target fish**: three times as likely to bite.
- **Lose chance**: chance it is lost when a fish escapes. A snapped line always loses it.
- **Size weights**: how likely each fish size is to bite.

| Bait | Bonus | Targets | Lose chance |
|---|---|---|---|
| Spinner Lure (V4) | 0 | anything | 20% |
| Worm | 1 | bluegill, bullhead, perch, bass | 75% |
| Cricket | 1 | pickerel, rock bass, rainbow trout, sockeye | 70% |
| Bread | 0 | small bluegill, perch, rock bass, bullhead | 80% |
| Crawdad | 2 | longnose gar, northern pike | 60% |
| Legendary Dragon Fly Lure | 2 | pike, gar, sockeye, large bass, the four legendary fish | 5% |

Fishing with no bait works, but interest rises slowly and legendary fish never bite.
With no bait, taps give more interest while the line is tense (a risk and reward).

## Fish

A fish's price sets its difficulty.

| Price | Tier | Examples |
|---|---|---|
| under $1 | Easy | small bluegill, perch, rock bass, bullhead, pickerel |
| $1 to $2.99 | Medium | medium bass, pickerel, sockeye, rainbow trout |
| $3 to $5.99 | Hard | large bass, large sockeye, steelhead trout |
| $6 to $19.99 | Expert | northern pike, longnose gar |
| $20 or more | Legendary | rainbow trout, muskellunge, lake sturgeon, channel catfish |

Many fish only bite at certain hours. Bullhead feed at night (8 PM to 6 AM). Chain pickerel feed in the morning (5 AM to noon).

## Waters

34 waters are built in: lakes, rivers, the bayou, creeks, ponds and the sea. Each has its own fish list.

| Fish | Where |
|---|---|
| Legendary lake sturgeon | Flat Iron Lake |
| Legendary rainbow trout | O'Creagh's Run |
| Legendary muskellunge | Owanjila |
| Legendary channel catfish | Bayou Nwa |
| Northern pike | Owanjila, Lake Isabella, Dakota River, Upper and Lower Montana River |
| Longnose gar | Elysian Pool, Kamassa, Lannahechee and San Luis rivers, Bayou Nwa, Arroyo de la Vibora, Bahia de la Paz |

The water list is part of the script and cannot be edited. Fish, baits and everything else can.

## Loot

Every kept fish gives one bonus item, picked by weight. The Pro Rod rolls twice.

| Item | Weight |
|---|---|
| `bait_crawdad` | 25 |
| White pearl | 10 |
| Red pearl | 6.5 |
| Blue pearl | 4.5 |
| Golden nugget (listed twice) | 2 + 2 |
| Gold pearl | 2 |
| Legendary Dragon Fly Lure | 1 |
| Black pearl | 0.5 |
| Uncut diamond | 0.5 |

A weight is not a percentage. An item's odds are its weight divided by the total.

## Seasons

At every start, a random 80% of the fish are in season. Out-of-season fish cannot be caught.
`/rerollseasons` picks again without a restart.

## Music

Press **D** while fishing to cycle: Off, Peaceful Waters, Serene Lakes, Circle of Stones, Mountain Banjo.
Peaceful Waters plays when the rod comes out.

## Configuration

Every setting can be changed in game with **`/poggy`** (the Poggy Hub), then a restart of the script.
You can also edit `config.lua` by hand.

| Section | What it controls |
|---|---|
| General | Rod items, bite wait, seasons, interface skin, hide fish not yet caught |
| `Config.SkillCheck` | Fight difficulty per tier |
| `Config.Interest` | The normal rod's tug game |
| `Config.ProRod` | The Pro Rod: drain, tension, mouse feel, loot and bait changes |
| `Config.Baits` | Baits and lures |
| `Config.Fish` | Fish: name, item, price, size, weight range, hours |
| `Config.LootDrops` | Bonus loot table |
| `Config.Sounds`, `Config.MusicTracks` | Sound files, volumes and music |

The hub has two guides: *Tuning baits and what bites* and *Making fishing easier or harder*.

### Interface skin

`Config.UI.skin` sets the look of the HUD and bait menu.

- `"brass"`: riveted brass and walnut, like the reel plate.
- `"default"`: the dark water theme.

To make your own, copy `ui/skin-brass.css` as `ui/skin-<name>.css`, change the image names in it, and set `skin` to `<name>`.
Images are optional; a missing one leaves that piece in plain colour.
Image sizes are listed at the top of `skin-brass.css`. The brass images are described in `docs/ui-skin-brass-prompts.md`.

## Troubleshooting

**"Nothing seems to be biting here..."**
No fish of this water is available right now. It may be the wrong hour, out of season, or the bait's size weights are 0 for every fish here. Try another bait, another time, or `/rerollseasons`.

**A yellow line at start names missing items.**
Add those items to your framework's item list (RSG: `rsg-core/shared/items.lua`).

**The HUD shows no fish icons.**
Copy `docs/item_images/` into your inventory's image folder. With "hide fish not yet caught" on, a fish only shows after it is caught in that water.

**Nothing happens when using the rod.**
Check `poggy_core` and `poggy_skillcheck` are started before `poggy_fishing`.

## Debug

`Config.Debug = true` prints catch and fight lines. `Config.DebugState = true` prints every state change to F8.

## Changelog

- **1.3.2**: Poggy Hub support. Every setting has a label and help text in `/poggy`, with two guides. README rewritten; the music key is **D** (it said C).
- **1.3.1**: Legendary fish are fairer. Their skillchecks no longer appear further off centre than any other fish's, where they could land behind the fishing HUD and be impossible to hit. The great zone is larger and each hit fills more of the bar, so a legendary fight is about one hit shorter on the rod and two on the Pro Rod. Every other tier is unchanged.
- **1.3.0**: The fish window now means something: a hook hangs from the surface on a line, the fish swims in from the right as interest rises and backs off as it falls, the line reddens and jitters with tension, and on the bite the fish lunges onto the hook and the line jerks before the fight starts. There is no fish until the line is in the water; it then swims in over the random time until the bite (with the Pro Rod it reaches the hook exactly as the fish bites; with the normal rod it arrives where the interest game takes over). Every fill bar (progress, interest, tension, pull tension) carries its water texture across the whole bar and loops without a jump; the wave along the top of the HUD loops cleanly too. The LEFT / FORWARD / RIGHT labels stay sharp while the pull needle moves. Pro Rod: when the fish pulls FORWARD it now shoves the bar steadily off centre (`ForwardPushForce`) and the jerks in `ForwardJerk*` actually fire; before, the push and the return force cancelled out and the bar just jittered at centre. Interface skins: `Config.UI.skin = "brass"` restyles the HUD and bait menu in riveted brass and gunmetal to match the reel plate; `"default"` looks as before. A server can add its own skin as `ui/skin-<name>.css` without touching any code.
- **1.2.1**: Runs on frameworks without an `items` table (RSG): the item rows in `sql/install.sql` are skipped there instead of stopping the install, and the script prints one yellow line at start naming the items your framework's item list lacks.
