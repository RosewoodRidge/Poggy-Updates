# Hiding emotes you do not want

Not every server wants all 330. `Config.HiddenEmotes` takes them away.

```lua
Config.HiddenEmotes = { "pee", "vomitkneel", "spit" }
```

A hidden emote is gone from the menu **and** from `/e`, so a player who already
knows the name still cannot play it.

---

## Use the typed name, not the label

The list wants the name in brackets in the menu, the one after `/e`. It is
lower case and has no spaces.

| In the menu | Put this in the list |
|---|---|
| Passed Out Drunk 2 `/e drunkpassout2` | `"drunkpassout2"` |
| Smoke (Nervous) `/e smoke3` | `"smoke3"` |

A name that does not match anything is ignored, so a typo hides nothing rather
than breaking the script. If an emote you hid is still showing, check the
spelling first.

---

## Hiding a whole category

Faster than listing every emote in it: take the category out of
`Config.Menu.CategoryOrder` on the **Menu** tab.

```lua
Config.Menu.CategoryOrder = {
    "favourites",
    "recent",
    "gestures",
    "conversation",
    "idle",
    "sitting",
    "working",
    "consume",
    "dance",
    -- "injury",   -- removed: this server does not want injury emotes
}
```

Those emotes are then gone from the menu and from `/e`, the same as hiding them
one by one.

---

## What this is not

This hides emotes from **everyone**. There is no per-job or per-rank emote
locking, and no admin-only emotes.

If you want an emote for one group only, the usual answer is to leave it hidden
here and trigger it from the script that owns that content.
