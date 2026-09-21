# Moving from another emote script

## Before you start

Run **one** emote script. Two that both register `/e` will fight over it, and
which one answers is down to load order. Stop and remove the old one before
starting poggy_emotes.

---

## Coming from js_emotes

Favourites come across on their own. The first time poggy_emotes starts it
looks for the old `emote_favorites` table, and if it is there, folds each
character's starred emotes into its own table. It runs once and is recorded, so
a later restart does not do it again.

The old table is **left exactly as it is**. Nothing is deleted, so you can put
the old script back if you want to.

Emote names are unchanged, so `/e wave`, `/e sitground1` and the rest all still
work. Players keep their muscle memory and any macros they had.

A few differences worth knowing:

| Then | Now |
|---|---|
| Emote data split between `config.lua` and `client.lua` | All in `shared/emotes.lua`, with your own additions in `Config.CustomEmotes` |
| `/e c` to cancel | `/ec`, or Backspace. `Backspace` worked before too. |
| Three emotes in the menu did nothing (`fixwagon1`, `fixwagon2`, `indicate`) | Removed: they had no animation behind them |
| `sleep2` appeared twice | Once |
| Thirteen animations reachable only by typing the name | Now in the menu |

---

## Coming from something else

There is no automatic import, but two things make the move easier.

**Keep your emote names.** If your players are used to `/e dance5` and that
name is not in poggy_emotes, add a row to `Config.CustomEmotes` with
`name = "dance5"` pointing at whichever animation you want. Their macros keep
working.

**Bring your own emotes across.** Most emote scripts store animations in a
shape close to this one: a dictionary, an animation name, and some flags. See
`docs/help/adding-emotes.md` for the field names and the three kinds.

---

## After the move

1. Check the server console on first start. It says how many entries in
   `Config.CustomEmotes` were skipped, and why, if any were.
2. Open `/poggy`, find Poggy Emotes, and look at the Emotes tab.
3. Play a prop emote, then mount a horse. The prop should be put away.
4. Ask a player to check their favourites are still there.
