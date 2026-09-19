# How the schedule works

Supply drops and scavenger hunts each run on their own timer. The two do not affect each other.

## The cycle

For each kind of event:

1. An event starts (by itself, or when an admin uses a command).
2. A **cooldown** begins: the **Cooldown** time plus a random extra of up to **Extra random cooldown**.
3. When the cooldown ends, the script waits a random time between **Shortest wait** and **Longest wait**.
4. The next event starts. Back to step 2.

## Default timings

| | Cooldown | Then a wait of | So a new one every |
|---|---|---|---|
| Supply drop | 60 to 90 min | 10 to 45 min | about 70 to 135 min |
| Scavenger hunt | 90 to 150 min | 20 to 60 min | about 110 to 210 min |

Nothing starts by itself until the first player has loaded in. After that, the first supply drop comes after a **Shortest wait** to **Longest wait** (10 to 45 minutes), and the first scavenger hunt after its own wait (20 to 60 minutes).

## Times are in milliseconds

Every time on the **Timing & schedule** tab is in milliseconds:

| Minutes | Milliseconds |
|---|---|
| 1 | 60000 |
| 10 | 600000 |
| 30 | 1800000 |
| 60 | 3600000 |

## Useful changes

- **More drops:** lower the supply drop **Cooldown** and **Longest wait**.
- **Only by command:** turn off **Automatic events**. Admins can still use `/supplydrop` and `/scavengerhunt`.
- **Turn one off:** use **Supply drops** or **Scavenger hunts** on the General tab.

Players can see how long is left with `/nextdrop`.
