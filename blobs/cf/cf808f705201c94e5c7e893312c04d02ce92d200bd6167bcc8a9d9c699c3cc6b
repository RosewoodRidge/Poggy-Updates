# The AI opponent and its levels

A player sitting alone can play the AI. It takes the other chair (the ped in **AI ped model**) and plays the other colour, so a player in the black chair plays black and the AI moves first.

## Chess

Each row in **Chess levels** is one difficulty in the start menu.

- **local** levels play on the server's own engine. It looks a few moves ahead and makes deliberate slips (**Mistake chance**) so that it can lose.
- **api** levels ask Stockfish at chess-api.com. They pick among Stockfish's suggestions to aim at a **Target winning chance**, so even these can be beaten. If chess-api.com cannot be reached in time, the server's own engine plays that move instead, and the game never stalls.
- Everything happens on the server. A player's own game never chooses the AI's move.

## Checkers

Each row in **Checkers levels** is one difficulty. The engine plays every variant by its own rules. It thinks for at most **Thinking time**, and pauses for the server every few milliseconds, so a long think never stalls the server. **Looseness** lets the easier levels choose among good moves instead of always the best.

## Draw offers

The AI takes a draw only after **Draws from move** moves each, and only when it is not too far ahead (**Takes a draw up to**). Easier levels take draws readily; the hardest rarely do.

## Hints

In games against the AI, a player can ask for a hint (H, or the Hint button). The suggested move is lit on the board. Hints have a cooldown, and are never offered in games between two players.

## Names

The name shown for a level comes from `level_<id>` in the NUI block of translations.lua. A new level with no line there shows its id.
