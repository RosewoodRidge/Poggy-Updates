# Giving staff access

`/animtool` only opens for players with the ACE permission `command.animtool`. Everyone else gets no reply.

AnimTool has no config file, so access is set in `server.cfg`.

## Give it to a group

```
add_ace group.admin command.animtool allow
```

Players in `group.admin` can now use `/animtool`.

Many servers already give `group.admin` the ACE `command` (every command). That includes `command.animtool`, so admins may already have access.

## Give it to one player

```
add_principal identifier.license:<their licence> group.animators
add_ace group.animators command.animtool allow
```

## After changing server.cfg

Restart the server, or run the same `add_ace` / `add_principal` lines in the server console to apply them now.

## Good to know

- Projects are saved **on each player's own PC** (client-side storage), not on the server. Use **Export → Project JSON** and **Import** to share a project with someone else.
- While the editor is open, your game time is held at 10:00 so lighting stays the same. Only you see this. Weather sync is asked to resend when you close it.
