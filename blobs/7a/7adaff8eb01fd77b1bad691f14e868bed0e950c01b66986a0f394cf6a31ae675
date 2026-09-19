# Writing your /help guide

`/help` opens a searchable guide for players. It ships with **example content**. Replace it with your own.

## Where it lives

- Open `/poggy`, pick **Poggy Util**, then the **Help pages** tab. Open **Guide pages**, pick a page on the left; its sections are on the right.
- Or edit `config/help.lua` by hand.

The server name and logo in the guide's header are on the same tab, under **Guide header**. The guide's on/off switch is on the **General** tab, under **Utilities**.

## How it is built

The guide is a list of **pages** (the file calls them categories). Each page is a tile with:

| Field | What it is |
|---|---|
| Id | Unique: letters, digits and underscores |
| Title | Tile label and page heading |
| Icon | One of the 16 built-in icons |
| Colour | Tile and heading colour, like `#d4c5a0` |
| Sections | The blocks on the page, in order |

The page with id `getting_started` is the large first tile. The others are sorted by title.

## Section types

| Type | Looks like |
|---|---|
| `info` | Plain bullets |
| `steps` | Numbered steps |
| `tips` | Bullets with a light bulb |
| `commands` | Plain bullets; write them as `/cmd — what it does` |

Two advanced types, `drug-crafting` and `medicine-grid`, draw item guides with inventory icons.
Their fields are described at the top of `config/help.lua`. Edit those in the file.

## Tips

- Search looks through every title and every line, so use the words players will type.
- The **I'M STUCK** button on the front page runs the unstuck command.
- Save, then restart poggy_util to see the new guide.
