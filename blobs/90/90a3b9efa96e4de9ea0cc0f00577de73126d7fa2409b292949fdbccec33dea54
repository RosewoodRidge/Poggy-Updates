# Skillcheck recipes

A recipe can ask for timed rounds instead of the progress bar.
This needs **poggy_skillcheck** (free) running on the server.

**The ingredients are taken before the rounds start.** A miss costs them.

## Three ways to ask for one

| Recipe field | What happens |
|---|---|
| `skillcheck = 3` | Three rounds at **Normal** difficulty. |
| `skillcheck = 2` and `useHardSkillcheck = true` | Two rounds at **Hard** difficulty. |
| `jobSkillcheck = 4` | Players **with** the recipe's job craft normally. Players **without** it may still try, on four rounds at **Untrained** difficulty. |

## The reward

- Pass every round: the full reward.
- Pass some: that share of the reward, at least 1.
- Pass none: nothing, and the materials are gone.
- Each hit in the bright band adds **Great-hit bonus** on top (25% by default).

Crafting several at once adds a round for every **Extra round every** items.

## Dangerous recipes

| Field | What a miss does |
|---|---|
| `explodeOnFail = true` | An explosion on the crafter. |
| `catchFireOnFail = true` | Sets the crafter alight for **Burn time**. |

With either one, a single miss ends the craft with nothing.

## Tuning the difficulty

The **Skillchecks** tab has three difficulty groups: Normal, Untrained and Hard.

- **Needle speed** and **Band size** do most of the work (1 easy, 5 hard).
- **Shake** makes the dial harder to read.

## Switching them all off

Turn **Skillchecks on** off. Skillcheck recipes then use the progress bar,
and nobody needs poggy_skillcheck. Job-locked recipes with `jobSkillcheck`
are then refused to players without the job.
