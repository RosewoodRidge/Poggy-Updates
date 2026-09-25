# NPC law response

A posse of NPC lawmen can ride out when a witness reports a crime and no
player lawman is on duty. It is off by default.

## Turning it on

1. Open `/poggy` → **Witnesses** → **NPC law response**.
2. Turn on **Enabled**.
3. Pick the **Crimes that send NPC law**. Shooting and Hijacking ship on. The
   names you can add: Shooting, Threatening, Melee, Lassoing, Trampling,
   Hijacking, CarryingHostage, Looting, Poaching.
4. Save and restart Witnesses.

## When it rides out

All of these must be true:

- A witness escaped and reported the crime (or another script sent a police
  alert for an engaged crime).
- The crime is in **Crimes that send NPC law**.
- **Player lawmen first** is off, or no lawman is on duty (see *Duty and
  police alerts*).
- The chance roll for that severity passes.

The player is told the law is coming at once. The posse sets out after
**Response time** and hunts for at most **Response duration**.

## How big the posse is

Each crime has a **severity**: Low, Medium or High (**Crime severity**). A crime
not listed counts as Medium.

Each severity (**Posse by severity**) sets:

| Setting | What it does |
|---|---|
| Fewest / Most officers | The posse size is picked between these. |
| Mounted officers | Percent who ride in on horseback. |
| Aggressiveness | How hard they fight: their aim, and how long they warn. |
| Chance to respond | Percent chance a posse comes at all. |

## What the officers do

- They appear at least 70 m away (**Spawn distance**), out of the player's
  sight, on a road where there is one. Riders arrive in the saddle.
- Within about 25 m they call out and hold the player at gunpoint for a few
  seconds. Shooting, drawing on them, running or waiting it out starts the
  fight.
- They fight only the wanted player: never bystanders or other lawmen.
- Riders get off their horses to chase a player on foot.
- When it is over they ride or walk away and vanish once out of sight.

## Escaping, fighting, surrendering

- Get further than **Escape distance** from every officer, for about 10
  seconds, and the player has escaped.
- Defeat them all and the response ends.
- With **Allow surrender** on, a player on foot near the posse can hold the
  surrender prompt. The officers hold fire and walk in with their guns up;
  one calls out, walks round behind the player and cuffs them there.
  Shooting starts the fight. Moving more than **Surrender movement
  tolerance**, or drawing a weapon, puts the officers back to their warning.

## Jail time

After an arrest the player is jailed for **Jail time by severity** minutes,
when something on the server can jail them:

- **Jail call** in `config/npc.lua` (runs as code, so it can only be changed in
  the file), or
- your jail script's export, added to `config/config.lua` as
  `JailExport` under **Your own duty script** (the README shows how), or
- **Use the built-in Sisika jail**, when neither of those is set.

None of the law scripts Witnesses knows has a way for another script to jail
a player (outsider_policeman, vorp_police, rsg-lawman, qbr-policejob,
bcc-law): with nothing set, the officer takes the cuffs off after a moment,
the player is let go with a warning, and the posse leaves.

## The built-in Sisika jail

Off by default, so it never fights a jail script you already run. Turn on
**Use the built-in Sisika jail** (NPC law response → Surrender and jail),
save and restart Witnesses. Then an arrested player:

- fades out and wakes up in Sisika Penitentiary, in the prison uniform, with
  their weapons put away;
- is kept on the prison grounds (walk off and they are brought back) and sees
  the time left at the bottom of the screen;
- serves the time only while online: log off and the rest waits for them,
  also after a server restart;
- gets their own clothes back when the time is up, and fades out to the far
  bank of the river, south-east of Saint Denis.

A player in this jail is never reported for a crime and no posse is sent
after them.

Staff commands (ACE, or the server console):

| Command | What it does |
|---|---|
| `/jail <id\|me> <minutes> [reason]` | Jails a player (at most 240 minutes). `me` jails yourself, to try it. |
| `/unjail <id\|me>` | Releases them at once. |

To try it: turn it on, then `/jail me 1`. You should land in Sisika in
the uniform with a timer, and a minute later be across the river in your own
clothes. On RSG or QBR, poggy_core cannot reload clothes yet: add
`RestoreCommand = "your reload command"` under `BuiltinJail` in
`config/npc.lua`.

Other optional keys (coordinates, grounds size, escapes, the most minutes) go
under `BuiltinJail` in `config/npc.lua`; the README lists them.

A player already in jail (outsider_policeman and rsg-lawman can say so) is
never reported for a crime and no posse is sent after them.

## Fine-tuning

More settings (warning time, accuracy, speech and others) can be added by hand
to `config/npc.lua`. The README lists them with their defaults.
