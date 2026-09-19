# Adding a transformation

Every card in the `/transform` menu is one row in the **Transformations** list.

## Add one

1. Open **Transformations** and choose **Add transformation**, or duplicate a
   similar one (duplicating a wolf keeps its attacks and emotes).
2. Give it a unique **Id**, such as `my_wolf`.
3. Set the **Name** players see on the card.
4. Set the **Ped model**, such as `A_C_Wolf` or `cs_dutch`.
5. Pick a **Category**. It decides the menu tab.
6. Save and restart the script.

## Card art

Put a PNG in `ui/images/` and type its file name into **Card image**. Art
around 300 x 200 or larger looks best. With no file, the card shows a category
icon, so a missing image is never broken.

## Lock it behind a job

Type a job name into **Job lock**. Only that job sees the card. Leave it empty
for everyone. Admins always see every card.

Only one job per entry. To offer the same animal to two jobs, add it twice
with different ids.

## Make it bigger

**Size** scales the ped: `1.0` is normal, `1.5` is half as big again. The
legendary animals use `1.3` to `1.5`.

## Let it attack

Turn on **Can attack** and fill in the **Basic attack animation**, **Reach**,
**Knockback** and **Damage**. Copying a similar animal is the quickest way to
get working animation names. See **Tuning animal attacks** for the rest.

## Emotes

Add rows to **Emotes**. Each has a **Command** (the word after `/ae`), a name,
an animation, and **Loops** (plays until `/ae stop`) or not (plays once).
