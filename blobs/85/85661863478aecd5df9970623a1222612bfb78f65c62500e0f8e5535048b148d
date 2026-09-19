# NPC law response

A posse of NPC lawmen can ride out when a witness reports a crime. It is off
by default.

## Turning it on

1. Open `/poggy` → **Witnesses** → **NPC law response**.
2. Turn on **Enabled**.
3. Pick the **Crimes that send NPC law**. Shooting and Hijacking ship on. The
   names you can add: Shooting, Threatening, Melee, Lassoing, Trampling,
   Hijacking, CarryingHostage.
4. Save and restart Witnesses.

## When it rides out

All of these must be true:

- A witness escaped and reported the crime.
- The crime is in **Crimes that send NPC law**.
- **Player lawmen first** is off, or no player with a law job is on duty.
- The chance roll for that severity passes.

## How big the posse is

Each crime has a **severity**: Low, Medium or High (**Crime severity**). A crime
not listed counts as Medium.

Each severity (**Posse by severity**) sets:

| Setting | What it does |
|---|---|
| Fewest / Most officers | The posse size is picked between these. |
| Aggressiveness | How hard the officers fight. |
| Chance to respond | Percent chance a posse comes at all. |

## Escaping, fighting, surrendering

- Officers appear at least 70 m away (**Spawn distance**).
- Get further than **Escape distance** from them and the response ends.
- Defeat them all and the response ends.
- With **Allow surrender** on, the player can give up and be arrested.

## Jail time

Stock VORP has no jail API, so an arrest does not jail anyone by default. If
you run a jail script with a server export, set **Jail call** in
`config/npc.lua`. It runs as code, so it can only be changed in the file.
**Jail time by severity** is used only when a jail call is set.
