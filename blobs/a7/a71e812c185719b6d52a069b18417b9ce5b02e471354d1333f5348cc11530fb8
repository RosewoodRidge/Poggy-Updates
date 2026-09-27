# Ratings and the leaderboard

Every character has an Elo rating for chess and one for checkers (all checkers variants count as checkers). Players see them in **My record**: on the Summary tab, as a +/- on each game, and on the **Leaderboard** tab. During a game they show beside the names, and the change shows when the game ends.

## What is rated

- Only **finished games between two players**. A game against the AI never changes a rating.
- A win, a loss or a draw (by agreement, repetition, stalemate and so on) all count. So do resigning, leaving a wagered game, and an idle claim.
- A game that ends with no result (a death, an admin reset) is not rated. Nor is a game with fewer than **Moves before a game counts** moves, so an instant resign changes nothing.
- A saved game is rated when it is finished.

## How it is worked out

It is the standard Elo formula:

- Everyone starts at **Starting rating** (1200).
- A win against a stronger player gains more than a win against a weaker one, and a loss to a weaker player costs more.
- How far a rating moves in one game:
  - 40 for a player's first 30 games, so a new player finds their level quickly;
  - then 20;
  - 10 from 2400 up.

  These are FIDE's values, and all of them are in the settings.

## The leaderboard

- Players appear after **Games to be listed** rated games (5 by default).
- It is ordered by rating, then by games played.
- A player who is not listed yet is told how many more games they need.
- The name shown is the character's name at their last rated game.

## Changing the settings

Changes apply from the next game. To work every rating out again under new settings, run `/chesstable rerate` (admins, or the server console). A rebuild uses the games themselves, so nothing is lost. The ratings were also worked out once from every game already played, the first time the script started with ratings.
