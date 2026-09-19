# Prices, upgrades and expiry

## Buying a storage

A player types `/createstorage` and confirms. They pay the **Creation price**
and the storage appears where they stand, with the **Starting capacity**.

- Use whole dollars for the creation price.
- `0` makes storages free.
- **Storages per character** caps how many one character can own.

## Upgrades

Owners and managers can buy more space from the storage menu. Each upgrade adds
**Slots per upgrade**, and each one costs more than the last:

```
price = first upgrade price x (1 + increase) ^ upgrades already bought
```

The price is rounded down to whole dollars.

**Example** with the shipped values (first upgrade $1.50, increase 0.1,
25 slots, start 200):

| Capacity before | Upgrades so far | Price |
|---|---|---|
| 200 | 0 | $1 |
| 225 | 1 | $1 |
| 450 | 10 | $3 |
| 700 | 20 | $10 |

> The number of upgrades is worked out from the capacity. If you change
> **Starting capacity** or **Slots per upgrade** later, existing storages'
> next upgrade price changes too.

## Expiry

With **Delete unused storages** on, player storages nobody has opened for
**Unused for** days are deleted when the script starts. There is no warning.
Job storages are never deleted.

Turn it off if your players store things for long breaks.
