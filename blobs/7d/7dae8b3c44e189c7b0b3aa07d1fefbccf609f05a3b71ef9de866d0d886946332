# Adding an alert command

Alert commands let players call a job to their position, like `/callpolice`
or `/calldoctor`. You can add your own, for example `/callblacksmith`.

## In game

1. Open `/poggy` and choose **Witnesses**.
2. Open the **Alerts** tab, then the **Job alerts** list.
3. Select an alert that is close to what you want, such as **NEED DOCTOR**,
   and choose **Duplicate**. Or choose **Add alert**.
4. Fill in the new row:

| Field | What to put |
|---|---|
| Title | The heading the job sees, e.g. `NEED BLACKSMITH`. |
| Command | The command name without the slash, e.g. `callblacksmith`. |
| Message to responders | What the job sees. |
| Message to the caller | What the player who called sees. |
| Jobs alerted | The job names, exactly as in your framework. |
| Grades alerted | For each job, the grades that get it, e.g. `blacksmith = 0,1,2,3`. |
| Cooldown | Seconds before the same player can call again. |
| Stay until reached | On if the blip should stay until the job arrives. |

5. Save, then restart Witnesses when asked.

## Good to know

- **Anyone can type any alert command.** For an alert that only another
  script should fire, give it an odd name that players will not guess.
- **Every command must be unique.** Two alerts with the same command clash.
- **Law jobs and duty.** If a job in the list is also in the **Law jobs**
  list, and **Only alert on-duty law** is on (both in **Access &
  permissions**), it only gets the alert while on duty. Turn on
  **Ignore duty** in the alert to skip that.
- **A job with no grades listed** only alerts grade 0.
- **Where and when.** Every alert says where it happened (the town, or the
  nearest one) and the in-game time.
- **Seeing a crime alert yourself.** `/witnesstestalert [crime]` (ACE
  `command.witnesstestalert`) puts one real crime alert on your own screen,
  as an on-duty lawman gets it, whatever your job. Nobody else is alerted.
- **The player who was seen** is told the law was called when a lawman got
  the alert or the NPC posse rides out; with nobody to answer, they are told
  that instead.

## From another script

Fire an alert from a server script without a command:

```lua
exports.poggy_witnesses:TriggerAlertForPlayer(src, "callblacksmith")
```

A police alert with its own text, no row needed (on-duty law only, with the
alert cooldown):

```lua
exports.poggy_witnesses:PoliceAlert({ src = src, title = "BANK ROBBERY", message = "The bank is being robbed!", crime = "Robbery" })
```
