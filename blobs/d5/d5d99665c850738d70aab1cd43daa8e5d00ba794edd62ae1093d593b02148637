# Using the props in Poggy Badge

This pack only adds 3D models. **Poggy Badge** is what pins them on a character.

## 1. Start the pack first

In `server.cfg`, this pack must start before Poggy Badge:

```
ensure poggy_core
ensure poggy_badge_props
ensure poggy_badge
```

## 2. Give a job a badge

1. Open **/poggy** → **Poggy Badge** → **Badges** → **Badge sets**.
2. Open the set for the job (or add one) and pick a grade.
3. Set **Prop model** to a name from this pack, for example `p_poggy_badge_sheriff`.
4. Set **Badge image** to the matching picture: the same name with `poggy_` in place of
   `p_poggy_badge_`, for example `poggy_sheriff`. (Needs Poggy Badge 1.4.0 or newer.)
5. Save & Restart.

Every model name is listed in this pack's README.

## 3. Armbands

An armband goes on an arm, and comes in nine sizes because the game cannot stretch a prop.
For the grade that wears it (turn on advanced settings):

| Field | What to put |
|---|---|
| Prop model | The size players start with, for example `p_poggy_armband_medic_s3` |
| Starting bone | Left upper arm or Right upper arm |
| Sizes | All nine names, `p_poggy_armband_medic_s1` to `_s9`, smallest first |
| Badge image | `poggy_armband_medic` |

Players then move the **Size** slider in `/badge` until it fits their outfit.

## Trying a prop quickly

Admins can type `/bprop <model>` to see any prop on their own character before adding it
to a job.
