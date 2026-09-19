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

## A streamed badge pack faces the wrong way

Stock `s_` badges are fine with the shipped rotation. Custom packs often are not.

1. Find the start of the pack's prop names, for example `kh_`.
2. In **Badges** → **Rotation overrides by prop name**, add that prefix.
3. Set the rotation that makes the badge face forward.
4. Save and restart.

Every prop whose name starts with that prefix now starts with that rotation.

## Why values end in .1

The game resets a rotation axis that is set to a whole number. The script adds 0.1 to whole numbers to stop that. You will not see a difference.
