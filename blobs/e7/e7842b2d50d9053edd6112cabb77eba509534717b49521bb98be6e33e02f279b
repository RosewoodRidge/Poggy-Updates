# The "brass" skin — images

The optional brass skin for the fishing HUD (`ui/skin-brass.css`, chosen with
`Config.UI.skin = "brass"`) draws every image at exactly one size, 1:1. Nothing
is stretched, sliced or repeated: the HUD has a fixed layout in this skin, so
each piece has one size, and each piece is generated at that size from the
whole-HUD template that set the look.

## How the set is built (15 September 2026)

Nothing in the current build was generated at its final size; the owner's
template-derived pieces were larger, and `tools/build-fishing-skin.py`
derives each exact-size file from them without stretching or repeating:
scaled down whole where the aspect matches, the plain middle cut shorter or
extended where it does not. The plate is composed from the owner's frame with
the reel plate in its corner (`1_b.png`), the walnut (`2.png`, scaled and
cropped, never tiled) and the owner's recoloured full-width crossbar (`3_c.png`) laid
across as the two row seams. The bait-menu plate is cut from the reel-free
frame (`1.png`). The reel's gear and handle are the owner's recoloured
sprites (`skin-brass-gear.png`, `skin-brass-handle.png`), turning over the
hub built into the frame.

The per-piece prompts in `ui-skin-brass-piece-prompts.md` are kept for
regenerating any piece at its exact size; prompt 1 there also adds the faint
tackle on the wood, which the composed plate does not have.

## The pieces

| File | Size (px) | Drawn as |
|---|---|---|
| `skin-brass-plate.png` | 518 × 595 | the whole backplate: frame with the reel corner, walnut, two seams |
| `skin-brass-plaque.png` | 283 × 53 | the zone nameplate |
| `skin-brass-slot.png` | 64 × 64 | each fish slot (24 to 42 CSS px, sized to the fish count) and each bait-row icon; made large so it is only ever scaled down |
| `skin-brass-window.png` | 451 × 120 | the water window frame, centre transparent |
| `skin-brass-track-wide.png` + `-back.png` | 437 × 29 | the progress bar channel: hollow brass front over the fill, steel back under it |
| `skin-brass-track.png` + `-back.png` | 194 × 26 | the interest and tension channels |
| `skin-brass-track-pull.png` + `-back.png` | 446 × 31 | the left / forward / right pull channel |
| `skin-brass-track-slim.png` + `-back.png` | 353 × 22 | the Pro Rod tension channel |
| `skin-brass-key.png` | 31 × 31 | the E key |
| `skin-brass-key-wide.png` | 53 × 35 | the LMB, G, Bksp and D keys |
| `skin-brass-needle.png` | 12 × 19 | the pull-bar pointer |
| `skin-brass-menu.png` | 360 × 384 | the bait menu's plate |
| `skin-brass-button.png` | 283 × 48 | the Remove Bait button |
| `skin-brass-gear.png`, `skin-brass-handle.png` | 800 × 800 | the turning reel parts, drawn in 124 / 123 CSS px boxes on the hub at CSS (385.5, 428.5) |

## The fixed layout (CSS px)

Plate 432 × 496. Rows from the top: rail 18; the frame's own rivet bar and
gap to 44; plaque 236 × 44 at y 44; fish slots row 42 tall at y 96 (slots sized to
the count: 42 px up to 8 fish, down to 24 px at 14, within 380 px); window
and the rest follow; the **stage** runs to 344 (window 376 × 84 at y 201 with the tug bars below it while waiting;
the progress bar at y 197 with the pull bars below it in a Pro Rod fight;
the window alone while idle; the window sits above the status line, 376 × 100); controls row 53 tall at y 344; music row 31
tall at y 397; time row 24 tall at y 428; the frame's bottom bar and rail
below. The two seams are centred at y 344 and y 397 (CSS), y 413 and y 476
in the 595-tall file. The reel hub is at CSS (385.5, 428.5). Each gauge is a steel back under the fill and a hollow brass front over it; fills, zone tints and the needle are confined to the hollow (4 to 8 % in from the ends, per channel).

If `--hud-scale` in style.css ever changes, every file size changes with it
(CSS size × scale) and the set has to be regenerated.
