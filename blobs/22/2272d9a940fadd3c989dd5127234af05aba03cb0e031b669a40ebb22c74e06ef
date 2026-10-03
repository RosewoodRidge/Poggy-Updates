# Poggy Chess & Checkers

Chess and checkers tables for RedM. Players sit down at a table, choose a game, and play a friend or the AI on a real board of props. Checkers comes in four variants plus house rules. Wagers are held safely until the game ends, and every game is kept for a record and a move-by-move review.

Runs on VORP, RSG and QBR through poggy_core.

## Features

- **Chess**, with every rule: castling (never out of or through check), en passant, promotion to any piece, and draws by stalemate, repetition, the fifty-move rule or insufficient material.
- **A marble chess set**: a black and white marble board and pieces, every piece veined differently, in the `poggy_chess_props` resource that comes with the script. One switch in `/poggy` goes back to the game's wooden set.
- **Checkers** in American, Russian, Brazilian and Pool rules, or house rules picked at the table. Captures are picked one jump at a time, so a capture that can go two ways is the player's choice. Pieces that must capture are ringed, and the rules of the game are always on screen.
- **Against a friend or the AI.** Chess uses the server's own engine and, for the strong levels, Stockfish (chess-api.com), with the server's engine as the fallback. Everything runs on the server, and the player's game never picks the AI's move.
- **A chess clock** for chess and checkers: the player who sets up the game picks the time (5+0, 3+2, 15+10 and the rest, or their own), the clocks show beside the names, and running out of time loses. The server keeps the time. The times on offer are set in `/poggy`.
- **Wagers** between players. Both stakes are taken when the game starts and paid out when it ends. Leaving a game loses it, and an idle opponent can be claimed against, so money can never be dodged or held hostage.
- **Characters walk to the chair and sit down** the way the game's own people do. If the way is blocked, they are placed in the chair.
- **Deaths** end the game for both players and stand them up. By default no result is recorded and the stakes are returned.
- **Elo ratings and a leaderboard**, one for chess and one for checkers. Only games between two players are rated, never games against the AI. Ratings show beside the names, the change shows when a game ends, and past games are rated on the first start.
- **My record**: every game per character, with wins, draws and losses, opponents, filters for chess, checkers and AI games, and a replay of any game move by move.
- Saved games (without a wager) can be picked up later from the same chair.
- Everything in `/poggy`, in the Poggy look, and in any language (translations.lua).

## Installation

1. Put `poggy_chess` and `poggy_chess_props` in your resources. `ensure` them **after** `poggy_core` and `oxmysql`, with `poggy_chess_props` before `poggy_chess`.
2. Restart the server. The database tables are created on the first start; there is nothing to import.
3. Add your tables in `/poggy` → Chess & Checkers → Tables, or in `config.lua`.

Requires poggy_core 0.23.0 or newer, and oxmysql.

## Playing

- Walk up to a chair and press **G** to sit. The first player seated chooses the game.
- The player in the other chair is invited, and sees the rules, the clock and any wager before accepting. Sitting alone, a player can play the AI instead.
- Click a piece, then where it goes, on the table or on the small board. **G** picks the square under the mouse. **F** or a right-click lets go, or takes back the last jump of a capture.
- **Z** offers a draw, **H** asks for a hint (against the AI), **C** changes the view, hold **R** to resign, and **E** stands up.

## For owners

| Where | What |
|---|---|
| `config.lua` | Tables, games, the AI levels, wagers, sitting, records, the screen. Every setting is also in `/poggy`. |
| `translations.lua` | Every line players read, including the table screens (the NUI block). |
| `/chesstable` | Admins and the console: lists the tables; `reset <id>` ends a stuck game with no result and empties the chairs; `rerate` works every rating out again. |

Database tables:

- `poggy_chess_games`: one row per game, with the whole move list for the review.
- `poggy_chess_ratings`: each character's Elo rating per game. It can always be rebuilt from the games.
- `poggy_chess_payouts`: money owed to players who were away (a wager returned after a restart). It is paid when they are next online.

Upgrading from 1.x: the old games table is extended in place. Old games keep their results, and those saved before 2.0.0 can still be reviewed (rebuilt from their notation). Records are now kept per character.
