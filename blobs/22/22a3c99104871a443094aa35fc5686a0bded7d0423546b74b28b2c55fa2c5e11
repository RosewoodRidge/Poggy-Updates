# Wagers, deaths and leaving a game

## Wagers

Two players can stake money on a game. The player who sets up the game enters an amount. The other player sees it before they accept.

- When the other player accepts, the stake is taken from **both** players at once. If either cannot pay, nothing is taken and the game does not start.
- The script holds both stakes until the game ends:
  - the winner receives both stakes;
  - a draw returns each player their own stake;
  - a game that ends with no result (a death, an admin reset) also returns both stakes.
- Standing up or leaving the server in the middle of a wagered game loses it, and the stake with it.
- If the opponent stops moving, the waiting player can claim the win after **Idle claim** minutes (X, or the Claim button). So walking away cannot hold the money hostage.
- If the server restarts during a wagered game, both stakes are returned. A player who is offline at the time is paid the next time they are on (see **Payout check**).

Money the script owes is kept in the `poggy_chess_payouts` table until it is paid, and nothing is ever deleted from it.

## Deaths

When a seated player dies, the game ends at once for both players, and both are stood up.

- With **When a player dies** set to *abandon* (the default), no result is recorded and any wager is returned.
- With *forfeit*, the player who died loses the game and the wager. This makes killing an opponent a way to win their stake, so think before choosing it.

## Leaving

- Standing up from a game **without** a wager saves it. Either player can pick it up again later from the start menu, sitting in the same chair.
- Games against the AI are saved the same way.
- A saved game nobody picks up is closed after **Close unfinished games after** days.
