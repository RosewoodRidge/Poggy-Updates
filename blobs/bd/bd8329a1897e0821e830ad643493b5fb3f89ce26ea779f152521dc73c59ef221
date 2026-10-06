# Fixing badge placement and rotation

Players place their own badge with the editor. These settings only decide **where a new badge starts**.

## A player's badge is in the wrong place

Nothing to change on the server. The player should:

1. Type `/badge`.
2. Move the sliders until it looks right.
3. Type a name and click **Save**. Next time they click the preset to load it.

## Every new badge starts in the wrong place

Change the starting values in **/poggy** → **Poggy Badge** → **Badges** → **Starting position on the chest**:

| Setting | What it changes |
|---|---|
| Starting bone | Which part of the body the badge is pinned to |
| Starting offset | Position from that bone, in metres (advanced) |
| Starting rotation | Pitch, roll and yaw, in degrees (advanced) |

A quick way to find good numbers: put a badge where you want it with the editor, then copy the slider values into the settings.

## An armband should start on the arm

An armband from the **Poggy Badge Props** pack starts on the left upper arm, the right way up, by itself. For the other arm, a different start, or another pack's armband, a grade can have its own starting position.

1. Open **Badges** → **Badge sets**, then the set and the grade that uses the armband.
2. Turn on advanced settings and set **Starting bone** to **Left upper arm** or **Right upper arm**.
3. Set that grade's **Starting offset** and **Starting rotation** (find the numbers with the editor, as above).

Players can still pick either arm in the editor's bone list.

## An armband is too tight or too loose

The game cannot stretch a prop, so an armband comes as several models, one per size.

- **Players:** type `/badge` and move the **Size** slider. Save a preset per outfit: a coat needs a bigger size than a shirt.
- **Owners:** the slider shows when the grade lists its **Sizes** (advanced), smallest first. **Prop model** is the size players start with.

## An armband wearer should show a badge, not the armband

**Show Badge** holds up the prop the player wears. For an armband that looks odd: a medic should hold up a medic's badge.

1. Open **Badges** → **Badge sets**, then the set and the grade that uses the armband.
2. Turn on advanced settings and put the badge's model name in **Prop held up by Show Badge**.
3. Save and restart.

The armband stays on the arm; only the prop in the hand changes. **Badge image** is still the picture nearby players see.

## A streamed badge pack faces the wrong way

Stock `s_` badges are fine with the shipped rotation. Custom packs often are not.

1. Find the start of the pack's prop names, for example `kh_`.
2. In **Badges** → **Rotation overrides by prop name**, add that prefix.
3. Set the rotation that makes the badge face forward.
4. Save and restart.

Every prop whose name starts with that prefix now starts with that rotation.

## Why values end in .1

The game resets a rotation axis that is set to a whole number. The script adds 0.1 to whole numbers to stop that. You will not see a difference.
