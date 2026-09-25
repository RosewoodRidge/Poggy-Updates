# Duty and police alerts

Law alerts only reach lawmen who are on duty. When nobody is on duty, the NPC
law response can ride out instead.

## Pick your duty script

Open `/poggy` → **Witnesses** → **Access & permissions** → **Duty**.

| Duty script | Use it when |
|---|---|
| Automatic | You run one of the scripts below. It finds it. |
| outsider_policeman | Your law clocks in with outsider_policeman. |
| vorp_police | Your law goes on duty in vorp_police. |
| rsg-lawman / qbr-policejob | Duty is your framework's job duty. |
| bcc-law / bcc-society | Off duty renames the job (`offpolice`). |
| My framework | poggy_core's own answer. |
| Everyone with a law job | You have no duty system. |
| Custom | Another duty script: fill in **Your own duty script** (Show advanced). |

**When the duty script cannot say** (none running): "Everyone with a law job"
alerts every law job; "Nobody" alerts none.

Save and restart Witnesses. Then type `/witnessduty` (console or an admin): it
shows the duty script in use and each lawman's duty state.

## Make sure your lawmen get the alert

1. Their job is in **Law jobs** (exact name, case matters).
2. Their grade is in **Law grades** for that job. Grades that start at 1, or go
   above 5, must be added.
3. They are on duty in your duty script.

## One crime, one alert

**Alert cooldown** (Alerts tab) is how long before the same crime by the same
player alerts again: 60 seconds by default. A different crime alerts at once.

## Players in jail

With outsider_policeman or rsg-lawman, a player in jail is never reported for
a crime and no NPC posse goes after them. They can still call for help.

## NPC law when nobody is on duty

In **NPC law response**: turn on **Enabled** and keep **Player lawmen first**
on. The posse then rides out only while no lawman is on duty, for the crimes
in **Crimes that send NPC law**.

## From another script

A robbery or dispatch script can raise a police alert with
`exports.poggy_witnesses:PoliceAlert({ src = source, title = "...", message = "...", crime = "Robbery" })`.
The README lists every field.
