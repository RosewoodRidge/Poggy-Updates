# Tickets on the website, and ban appeals

Both are **off** until you turn them on. `/poggy` → Tickets → **Web panel**.

## What it is

Tickets in a browser, at **rosewoodridge.xyz/tickets**. Everyone signs in with their Cfx.re account and types your **community ID**.

- **Players** read and answer their **own** tickets, under each of their characters, and write new ones when they are not in game.
- **Anyone with a ticket role** gets the **staff desk** as well. Nothing to set up: it follows their role.
- **Banned players** can appeal to you at **rosewoodridge.xyz/appeal**. Each appeal arrives as a ticket of the kind **Ban appeal**.

## Read this before you turn it on

Your server cannot talk to a website directly, and a website cannot talk to your server. So both talk to a **relay** run by Rosewood Ridge.

That means copies of your **open tickets** leave your server: names, what players wrote, the conversations, internal notes.

- They are kept while a ticket is open, and for **7 days** after it closes.
- Archive or delete a ticket and its copy is taken back at once.
- Turn the web panel off and the relay is told to **forget your community**, at once.
- If you stop syncing for a month, the relay forgets you by itself.
- For each person who uses the website, the relay also keeps their Cfx.re account id, their name and their character names on your server. Nothing else about them.

If that is not right for your community, leave it off. Nothing else in the script needs it.

## What staff can do from the web

Read, reply, write internal notes, claim, release, close.

**Warn, kick and ban stay in game unless you switch them on** (below).

Every web action is checked against that person's ticket role on **your** server, exactly like an in-game press. The relay never decides anything. In the audit trail a web action shows as *Name (web)*.

## Turn it on

1. `/poggy` → Tickets → **Web panel** → switch it on. Restart the script.
2. Your **community ID** is made for you. You do not type it or choose it. It looks like `ABCD-EFGH`. There is **one** for your whole community, and no two communities get the same one.
3. **Everyone sees it** in game: at the top of `/tickets` and `/ticket`, and on the **Website** tab of both. Nothing to hand out. It is not a secret: on its own it opens nothing.

## How a person links their account

Your server knows people by their game account. The website knows them by their Cfx.re account. Each person joins the two **once**.

**By itself.** If their game is signed in to Cfx.re, they are linked the moment they log in. The **Website** tab says *You are linked*.

**With a link code**, when that did not happen, or they use a different Cfx.re account on the website:

1. In game: `/ticket` (or `/tickets`) → **Website** → **Get my link code**. Six letters and numbers, good for ten minutes, once.
2. At rosewoodridge.xyz/tickets: sign in, type the community ID, type the code.
3. Your server checks the code within a few seconds and tells them in game.

A code must **never be given to anyone**: whoever types it gets that person's tickets, and their staff desk if they have one. The tab says so in red.

A link made with a code is kept. Anyone can **unlink** on the same tab.

People who played before you turned this on: no framework keeps the Cfx.re id, so they are linked the next time they log in, or straight away if an old ticket of theirs carries it.

## What a player sees

Their own tickets, open and closed in the last 7 days, with a chip per **character**. They can answer, and write a new ticket as one of their characters. The same cooldown and open-ticket limit as in game apply. A banned player is sent to the appeal page instead.

They never see internal notes, and never anyone else's ticket.

## Staff on the website

Staff get a second tab, **Staff desk**, for as long as their account holds a role: read, reply, internal notes, claim, release, close. Take the role away in game and the desk is gone at the next sync.

- A staff link runs out after 30 days without a login (**Staff must log in every**), so someone who left cannot keep it.

### Confirm each browser (on by default)

A Cfx.re login alone does **not** open the staff desk. The first time in a new browser, the page asks for a **link code**:

1. In game: `/tickets` → **Website** → **Get my link code**.
2. Type it on the page. Once per browser.

So a stolen Cfx.re login is useless without also being in your game. A code only works for the person it was given to.

Lost a device, or worried? `/tickets` → **Website** → **Sign out my browsers**. Every browser loses the staff desk until it is confirmed again.

Linking with a code confirms that browser in the same step. Players' own tickets never ask for this.

### Warn, kick and ban from the website (off by default)

Switch on **Warn, kick and ban from the website** and staff get three more buttons on a ticket that **reports a player**, if their role has the power in game.

- A reason is always asked for. The player reads it.
- A ban from the website is a **local** ban. Bans for the shared ban network are made in game.
- Your server checks each one again, and the audit trail says *Name (web)*.
- Leave **Confirm each browser** on if you use this.

## Ban appeals

Switch on **Take ban appeals from the website**. This lists your community **by name** on the appeal page, so only do it if you want to be found there.

- A player signs in with Cfx.re, picks you, and writes their appeal.
- It becomes a **Ban appeal** ticket. Every role that may **lift bans** sees it, with nothing to set up. Tick the kind on other roles if you want them to see it too.
- Click the player's name on the ticket to see their record. Their bans show there, because a Cfx.re account is one of the identifiers a ban is on.
- Reply in the ticket. The player reads it on the website and can answer.
- They never see internal notes.
- One open appeal per player at a time.

Lift the ban the usual way: the **Bans** tab, or `poggycore unban` in the console.
