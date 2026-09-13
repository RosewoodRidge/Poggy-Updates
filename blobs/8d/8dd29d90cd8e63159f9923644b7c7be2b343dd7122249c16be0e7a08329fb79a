# Poggy Fishing

A skillcheck-based fishing system for RedM (VORP framework) featuring zone-specific fish, two rod types with distinct playstyles, a bait targeting system, dynamic seasons, and a full NUI interface with sound and music.

## Dependencies

| Resource | Purpose |
|----------|---------|
| `poggy_core` (0.13.0 or newer) | Framework layer: items, baits, fish, notifications |
| `poggy_skillcheck` | Skillcheck UI/logic |
| `oxmysql` | Database access |

## Installation

1. Place `poggy_fishing` in your resources folder.
2. Add `ensure poggy_fishing` to your server.cfg (after all dependencies).
3. Configure `config.lua` to taste. Water zones and the fish found in each are built into the script.
4. Copy the item icons from `docs/item_images/` into your inventory's item image folder (on VORP: `vorp_inventory/html/img/items/`), so fish, bait and rods show their pictures. The journal's icon is in `poggy_fishing_journal/docs/item_images/`.

## Database

The fish, bait, rod and loot items are added to the database automatically when the script starts (`sql/install.sql`); items you already have are never changed. To import the file yourself instead, set `PoggyCoreConfig.Sql.AutoInstall = false` in `poggy_core/config.lua`.

On a framework without an `items` table (RSG), poggy_core skips those rows and the items must be added to the framework's item list by hand (RSG: `rsg-core/shared/items.lua`): the two rods (`fishingrod`, `fishingrod_pro`), the baits and lures (`bait_worm`, `bait_cricket`, `bait_bread`, `bait_crawdad`, `p_lgoc_spinner_v4`, `p_finishedragonflylegendary01x`), every fish in `Config.Fish` (`a_c_fish…`) and the loot drops (`whitepearl`, `redpearl`, `bluepearl`, `goldpearl`, `blackpearl`, `golden_nugget`, `diamond_uncut`). At start the script prints one yellow line naming any of these your item list does not have.

## Controls

| Key | Action |
|-----|--------|
| **LMB** | Cast line |
| **E** | Tug line (interest phase) |
| **G** | Open bait menu |
| **C** | Cycle music track |
| **Backspace** | Stow rod |
| **Mouse** | Counter fish pull direction (Pro Rod only) |

## How It Works

Fishing follows a five-phase state machine:

### 1. Equip Rod
The player uses a `fishingrod` or `fishingrod_pro` item from inventory. A draw animation plays, the rod prop attaches to the hand, and the fishing HUD appears showing the current water zone and available fish.

### 2. Cast Line
LMB triggers a wind-up animation. The line lands with a synced splash effect visible to nearby players, then a brief random wait begins (3–20 seconds, configurable).

### 3. Wait for Bite

**Regular Rod** — An interactive interest/tension minigame:
- Press **E** to tug the line, building both interest (blue bar) and tension (red bar).
- Both bars decay naturally when not pressing.
- Fish hooks when interest reaches 100%. Line snaps if tension hits 100%.
- Difficulty scales per fish tier (easy through expert) — harder fish gain interest slower and lose it faster.

**Pro Rod** — Skips the interest phase entirely. The fish bites after a random wait (4–15 seconds) and you go straight to the fight.

### 4. Fight Phase

**Regular Rod** — Progressive skillchecks:
- 3–10 rounds depending on fish difficulty tier.
- Good/Great hits fill the progress bar; misses drain it.
- Consecutive misses increase speed and shake intensity.
- 4 total misses snaps the line.

**Pro Rod** — Skillchecks plus a real-time direction pull system:
- The fish pulls LEFT, RIGHT, or FORWARD. Direction changes every 3–6 seconds.
- Move your mouse to counter the pull (camera controls are disabled during this phase).
- Correct pull reduces drain by 88% and decays tension. Wrong pull increases drain to 120% and builds tension at 20/sec.
- Random forward jerks keep you on your toes.
- Progress ≤ 0 = fish escapes. Tension ≥ 100 = line snaps.
- Skillcheck gains are reduced (65% good, 75% great) and misses hurt 50% more.

### 5. Fish Caught
A size-appropriate unhook animation plays, the fish ped spawns in the player's hands, and the catch + any bonus loot is awarded to inventory.

## Rod Comparison

| | Regular Rod | Pro Rod |
|---|---|---|
| Approach phase | Interest/tension tug minigame | Skipped (random wait) |
| Fight phase | Skillchecks only | Skillchecks + direction pull |
| Skillcheck difficulty | Standard | Harder (reduced gains, harsher misses) |
| Rare fish chance | Normal | Weighted toward larger/rarer fish |
| Loot rolls per catch | 1 | 2 |
| Bait loss on escape | Standard | Halved |
| Line snap condition | 4 failed skillchecks | Tension ≥ 100 |

## Bait System

Equip bait through the bait menu (**G** key). Each bait has:

- **Bonus** — Reduces effective skillcheck difficulty (0–2 tiers).
- **Target fish** — Specific species this bait attracts. `nil` = works on everything.
- **Lose chance** — Probability (0–100%) the bait is consumed when a fish escapes. Line snaps always consume bait.
- **Size weights** — Per-size multiplier controlling how likely small/medium/large/XL fish are to bite.

| Bait | Bonus | Targets | Lose % | Best For |
|------|-------|---------|--------|----------|
| Spinner Lure (V4) | 0 | All | 20% | General purpose |
| Worm | 1 | Bass, catfish, perch, bluegill | 75% | Medium fish |
| Cricket | 1 | Pickerel, trout, salmon, rock bass | 70% | Medium-hard fish |
| Bread | 0 | Small fish (bluegill, perch, rock bass) | 80% | Beginners |
| Crawdad | 2 | Large bottom-feeders (catfish, sturgeon, gar, pike, muskie) | 60% | Trophy fish |
| Legendary Dragon Fly Lure | 2 | Trophy fish (sturgeon, gar, pike, muskie, salmon, bass) | 5% | Rare drop, nearly impossible to lose |

**No-bait fishing** is possible but much harder — interest gain is drastically reduced. A risk/reward mechanic grants a 3× interest bonus when tension is between 20–85%.

## Fish

32 species across 4 size classes. Difficulty is determined by base sale price:

| Price Range | Tier | Example Species |
|---|---|---|
| < $1.00 | Easy | Bluegill, Perch, Rock Bass, Bullhead (sm) |
| $1.00 – $2.99 | Medium | Largemouth Bass, Smallmouth Bass, Rainbow Trout, Salmon (md) |
| $3.00 – $5.99 | Hard | Large bass, Large trout, Sockeye Salmon (lg) |
| ≥ $6.00 | Expert | Channel Catfish, Northern Pike, Longnose Gar, Muskie, Lake Sturgeon |

Many fish have **time-of-day windows** (e.g., Bullhead Catfish is nocturnal 8PM–6AM, Chain Pickerel feeds 5AM–12PM). Fish outside their active hours cannot be caught.

## Water Zones

25+ zones mapped to RDR2's native water-map-zone hashes. Each zone has a curated fish list:

| Zone Type | Examples | Typical Fish |
|---|---|---|
| **Lakes** | Flat Iron Lake, O'Creagh's Run, Owanjila, Lake Isabella, Elysian Pool | Bass, sturgeon, muskie, trout, salmon |
| **Rivers** | Dakota River, Kamassa River, Lannahechee River, Upper/Lower Montana | Pickerel, salmon, pike, gar |
| **Swamps** | Bayou Nwa | Gar, catfish, bullhead |
| **Creeks** | Dewberry Creek, Ringneck Creek | Small fish only (beginner zones) |

Some trophy fish are zone-exclusive: Lake Sturgeon only appears in Flat Iron Lake and O'Creagh's Run. Muskie is limited to Flat Iron Lake and Owanjila.

## Loot Drops

Every successful catch rolls a bonus loot table (Pro Rod rolls twice):

| Item | Drop Weight |
|------|-------------|
| Cricket Bait | 25.0 |
| White Pearl | 10.0 |
| Red Pearl | 6.5 |
| Blue Pearl | 4.5 |
| Gold Pearl | 2.0 |
| Golden Nugget | 2.0 |
| Black Pearl | 0.5 |
| Uncut Diamond | 0.5 |
| Legendary Dragon Fly Lure | 0.2 |

## Season System

On server start, a random 80% of all fish species are marked "in season." Out-of-season fish cannot be caught. Seasons reset every server restart or via the `/rerollseasons` command.

## Music

Press **C** to cycle through ambient music tracks while fishing:

| Track | Label |
|-------|-------|
| — | Off |
| fishingMusic_1 | Peaceful Waters |
| fishingMusic_2 | Serene Lakes |
| fishingMusic_3 | Circle of Stones |
| fishingMusic_4 | Mountain Banjo |

## Configuration

All mechanics are tunable in `config.lua`:

| Section | What It Controls |
|---------|-----------------|
| `Config.ProRod` | Pro rod drain, tension, pull sensitivity, skillcheck multipliers |
| `Config.SkillCheck` | Difficulty tiers, speed, rounds, good/great/miss values |
| `Config.Interest` | Interest/tension gains, decay rates, scaling per tier |
| `Config.Baits` | Bait bonuses, target fish lists, lose chances, size weights |
| `Config.Fish` | All 32 species — price, weight range, model, time windows |
| `Config.LootDrops` | Bonus item table and drop weights |
| `Config.Sounds` | SFX files and volumes |
| `Config.MusicTracks` | Available music tracks |
| `Config.Season` | Percentage of fish active per season cycle |

Zone definitions and fish-per-zone assignments are built into the script and are not user-editable; the Water Zones section above lists what is included. Everything owners tune lives in `config.lua`.

## File Structure

```
poggy_fishing/
├── fxmanifest.lua          # Resource manifest & dependencies
├── config.lua              # All mechanics configuration
├── README.md
├── shared/
│   └── fishing.lua         # Water zones, fish availability, helpers (built in, escrowed)
├── sql/
│   └── install.sql         # Item registration (applied automatically on start)
├── client/
│   └── client.lua          # Client state machine & animations
├── server/
│   └── server.lua          # Catch handling, item awards, season logic
└── ui/
    ├── index.html           # NUI overlay structure
    ├── style.css            # Styling & animations
    ├── app.js               # NUI logic (skillcheck, pull bar, HUD)
    ├── img/                 # Fishing item icons
    └── sfx/                 # Sound effects & music (mp3)
```

## Debug

Set `Config.Debug = true` for console logging. Set `Config.DebugState = true` to print state-transition audits to the F8 console.

## Changelog

- **1.2.1** — Runs on frameworks without an `items` table (RSG): the item rows in `sql/install.sql` are skipped there instead of stopping the install, and the script prints one yellow line at start naming the items your framework's item list lacks.
