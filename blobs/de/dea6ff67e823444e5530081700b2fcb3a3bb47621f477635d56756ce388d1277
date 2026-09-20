<1.5.2>
- Fixed Add player failing with "menu error: bad argument #2 to 'format'" on VORP whenever someone on the server was still at character select; they are left out of the list
- A value that does not fit a message's placeholder (text, or a decimal price, in %d) is now shown as text instead of closing the menu

<1.5.1>
- The Storages panel in /poggy (poggy_core's settings hub, data panels): every player and job storage in one table with owner, location, slots, upgrades, access and jobs
- Rename and resize player storages from the table; Move to my position, Change owner and Delete on each row; each storage's access list (give, change level, remove); the hub's Contents viewer on every storage (container character_storage_<id>)
- /movestorage and /deletestorage share one code path with the panel (CS.Admin.Move / CS.Admin.Delete); a refused move now says so. Deleting still unregisters the container and leaves the items in the inventory database
- New export HubPanel; new DB.UpdateStorageOwner

<1.4.0>
- Requires poggy_core 0.14.0 or newer
- Framework-agnostic: nothing in the resource talks to VORP any more
- The storage menus and text prompts are drawn by poggy_core (menu.open / input.text); the resource's own NUI (ui/, client/ui.lua) is gone
- The armory shops and /adminshop are poggy_core menus: price on the right, description underneath, a quantity prompt for stackable items. The purchase logic is unchanged (money before item, job re-checked on every take, refund when the give fails) and now also checks the player can carry the item before charging
- /adminshop lists every item in the registry, not only those with an icon
- Removed the "needs VORP" gate on the shops, and vorp_inputs / vorp_inventory / vorp_menu from the dependencies

<1.3.0>
- Requires poggy_core 0.11.0 or newer; poggy_util is no longer used
- Removed the old per-framework fallbacks; every framework call goes through poggy_core
- Fixed storages that could look empty after a restart: container ids are switched to raw before the first registration
- Fixed storage registration running outside a thread after a database callback
- Money is now taken before a storage is created, upgraded, deposited or an armory item is given; nothing is granted when the payment fails, and a failed step refunds
- Armory: the job is re-checked on every take, and a refused give refunds the price
- Fixed "View / Remove Player" never replacing the "Loading player..." rows
- Discord tracking reads container contents through poggy_core and stays off when a webhook is empty
- Webhook URLs in config.lua are blank by default; add your own
- Added escrow_ignore for config.lua

<1.1.0>
- Added Access Level System: players added to a storage can now be assigned "basic", "member", or "manager" roles
  - Managers can deposit, withdraw, view ledger, and upgrade storage
  - Members can deposit and view ledger (no withdraw)
  - Basic users can only open the storage items
  - Storage owners can change a player's access level at any time
- Added configurable job-grade-to-access-level mapping (Config.ManagerJobGrades, Config.MemberJobGrades)
- Added NPC Clerk system (client/npc.lua): spawns unkillable, persistent clerk NPCs at configurable locations with distance-based spawn/despawn
- Added Armory Shop system (client/shop.lua): VORP Inventory-based job-locked shops with configurable items, NPC vendors, blips, and prompts
- Added Discord Webhook integration (server/discord.lua): real-time inventory and activity tracking for preset storages and armory shops
  - Inventory summary embeds updated on a configurable interval
  - Activity log embeds tracking who took/added items with timestamps
  - Automatic restore of activity history from existing Discord messages on restart
  - Armory purchase logging to Discord
  - Configurable per-storage and per-armory webhook settings
- Added Config.WeaponArmoryItems: master list of weapons, ammo, and supplies for armory shops
- Added Config.ArmoryShops: configurable armory shop locations with job-lock, NPC, blip, and Discord tracking
- Added Config.NPCs section for clerk NPC spawning at storage locations
- Added Config.DiscordDebug toggle for Discord webhook debug logging
- Added Config.adminShopCommand for admin all-items shop
- Added public_access option for preset storages (e.g., hotel rooms anyone can open)
- Added non-linked preset storages with id_prefix/name_template pattern (e.g., individual hotel rooms)
- Added Blackwater Hotel Rooms as a non-linked preset storage example
- Added Blackwater Hotel Keys armory shop example
- Added new translation keys for armory, admin shop, and access level system (English and Spanish)
- Improved storage initialization: SendStoragesToAllPlayers now called inside LoadAllStorages callback instead of fixed timeout
- Improved client startup: storages now requested on resource start to cover restart scenarios
- Improved blip system: storages with blipSprite = false are explicitly skipped (no blip created)
- Improved authorized_users format: migrated from plain number array to objects with id and level fields
- Updated police_evidence preset: changed its authorized jobs; updated Blackwater location coordinates
- Updated doctor_main preset: updated Blackwater Doctor location coordinates; added Discord tracking
- Fixed duplicate CreateStorageBlips function definition in old client.lua
- Bumped version from 1.0.8 to 1.1.0

<1.0.9>
- Fixed an issue with duplicate definitions.

<1.0.8>
- Added ability for players to store money in their character storages.

<1.0.6>
- Added automatic deletion of unused storages after configurable time period (default 60 days)

<1.0.5>
- Made the isPreset flag automatic - no longer needs to be specified in config
- Improved reliability of storage permission system
- Fixed an issue with "Search Nearby Players" not finding players. Changed language to "Search players", and now finds all players currently online instead.
- Added option to only show blips for storages the player has access to
- Added admin command to show/hide all storage blips regardless of access
- Improved permission checking for storage blip visibility
- Removed dependency on syn_inputs. Now uses VORP native inputs

<1.0.4>
- Added option to create preconfigured storages including linked storages (useful for shared police or doctor storages for example)
- Added option for players to give specified job access to their storages
- Improved data consistency for preset storage blip information sent from server to client.
- Modified script to by default allow shared storage visibility (was off for testing purposes)

<1.0.3>
- Fixed an issue with storages not loading on server start (requiring a manual restart of the script for storages to appear)
- Fixed an issue where after deleting a storage, the blip on the map remained and the prompt was interactable

<1.0.2>
- Fixed some language issues
- Added a configurable multiplier to upgrading the storage

<1.0.1>
- Added version checking system
- Created README documentation
- Fixed bug with player removal from storage access list
- Code optimizations and documentation improvements

<1.0.0>
- Initial release
- Player storage creation system
- Storage access management
- Storage capacity upgrades
- Multi-language support (English and Spanish)
- Storage renaming system
- Admin commands for storage management
