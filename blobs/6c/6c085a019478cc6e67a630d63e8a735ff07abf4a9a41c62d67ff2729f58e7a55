# Tuning animal attacks

An animal can attack when **Can attack** is on and it has a **Basic attack
animation**.

## The keys

| Key | Does |
|---|---|
| Left click | The basic attack. |
| Keys in **Attack inputs** (Combat tab) | An extra move, by slot. |
| Space | The jump animation, if the animal has one. |
| R1 / RB | Crouch. |

The shipped **Attack inputs**:

| Slot | Key |
|---|---|
| `pounce` | `INPUT_ENTER` |
| `ground` | `INPUT_ATTACK` (left click, so it fires with the basic attack) |
| `taunt` | `INPUT_RELOAD` |

An animal only uses a slot if it has an **Extra attack move** with that key.

## Numbers for one attack

| Setting | What it does |
|---|---|
| **Damage** | Health taken per hit. |
| **Knockback** | How hard the target is thrown. `0` = none. |
| **Reach** | How far away a target can be, in metres. |
| **Cooldown** | Wait before that move works again, in ms. |

An extra move that leaves a number empty uses the animal's basic value. An
empty cooldown uses **Attack cooldown** from the Combat tab.

**Taunts:** a move with damage `0` and reach `0` just plays its animation.
It needs no target.

## Hitting other players

Attacks on players go through the server and are only applied when:

- PvP is on for the target, and
- the attacker is within the animal's **Reach** plus **Range tolerance**.

The server also drops attacks sent faster than **Server attack limit**.

If attacks miss on a laggy server, raise **Range tolerance** a little. Keep
**Server attack limit** below your shortest cooldown.

Two things to know about hits that go through the server (on players, and on
NPCs another player's game controls):

- The target checks the attacker's basic **Reach**, even for a longer-reach
  extra move. If a pounce misses players at full range, raise **Range
  tolerance**.
- The hit uses the animal's basic **Damage** and **Knockback**. An extra
  move's own numbers apply to nearby NPCs only.
