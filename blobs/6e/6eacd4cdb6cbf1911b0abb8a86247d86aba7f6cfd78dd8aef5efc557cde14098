# Adding bins

There are two kinds of bin:

| Kind | Where it comes from |
|---|---|
| **Listed bins** | **Bins**, on the **Locations** tab. You place them. They can have a map blip. |
| **Found bins** | Bins already standing in the game world. Found automatically when **Find world bins** is on. |

## Add a listed bin

1. Walk to the spot.
2. Open `/poggy` → **Trash Bins** → **Locations** → **Bins**.
3. Press **Add bin**.
4. On **Position**, press **use my position**.
5. Choose **Spawn a bin here**:
   - **On** if there is no bin at this spot. The script places a street trash can.
   - **Off** if a bin already stands here in the world.
6. Give it a **Storage id** that no other bin uses, such as the next number (`34`, `35` …).
7. Save and restart.

## Storage ids matter

Each bin's storage is saved under its storage id.

- Two bins with the same id share one storage.
- Changing an id loses whatever players left in that bin.

## Found bins

With **Find world bins** on, every object in **Bin models to find** becomes a bin as players come near it. Each gets its own storage, named from its position (`auto_…`). Found bins use the shared loot and the **Found bins** slot count. They never get a map blip.

To stop a model counting as a bin, remove it from **Bin models to find**.
