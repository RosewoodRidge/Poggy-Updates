# Poggy Balloon Changelog

## v1.6.0

- **Framework-agnostic.** The station menu and all notifications now go through poggy_core (`menu.open`, `notify.styled`); nothing calls vorp_core or vorp_menu any more. Requires poggy_core 0.14.0. `vorp_menu` is no longer a dependency.

## v1.5.1

### Core Changes
- **Renamed** from `poggy-balloon` to `poggy_balloon`. Change `ensure poggy-balloon` to `ensure poggy_balloon` in your server.cfg. Settings are the same.
- **Climb and descend fixed.** Holding Shift did nothing on current game builds, because the game's own burner input was not registering for the captain. The script now applies the climb itself: hold **Shift** to burn and climb, hold **Ctrl** to descend. Tune it in `Config.Flight`.

### Release Defaults
- Taxi ride `Price = 3`, rental `Price = 50`.
- `Config.Commands.SpawnBalloon.AdminOnly = true`.
- Taxi server logging now follows `Config.Debug` (it printed every ride to the console before).
- `/balloon_server_status` uses the ACE permission `command.balloon_server_status` only.
- Removed the developer diagnostic command.

## v1.5.0

### Cleanup
- Removed leftover framework fallback code. Money and character checks go only through poggy_core.

## v1.4.0 – v1.4.6

### Core Changes
- **Own prompt library (1.4.0).** On-screen prompts now come from poggy_core's prompt library. The third-party `uiprompt` resource is no longer needed or used.
- **Rental balloons climb again (1.4.6).** Rentals were spawned in hover mode, which pinned their height. That call is gone.
- **Spawn notification fixed (1.4.2).** The `/spawnballoon` notification called a function before it was defined, so it never showed.

### Technical Summary
- 1.4.1 to 1.4.4 tried several climb fixes that were withdrawn. 1.4.5 put the flight controls back to the 1.2.1 original, and 1.4.6 changed only the rental hover call.

## v1.3.0

### Core Changes
- **Moved to poggy_core.** Money (`money.get`, `money.remove`) and character checks go through poggy_core, which is now a required dependency.

## v1.2.1

### Discord Post Format
```md
# :balloon: poggy-balloon v1.2.1

## :tools: Core Changes
- Separated Balloon Taxi and Balloon Rental feature toggles so they work independently.
- Taxi can now be disabled while rentals stay enabled.
- Rentals can now be disabled while taxi stays enabled.
- Both can be enabled together or disabled together.

## :gear: Config Behavior
- Config.BalloonTaxi.Enabled controls only taxi rides.
- Config.BalloonRental.Enabled controls only balloon rentals.
- NPC spawn, blips, and menus now load when at least one service is enabled.
- Locations with rentalEnabled = false will not show rental option when taxi is disabled.

## :sparkles: Localization And Configurable Text Pass
- Updated more baked-in user text to use translation keys in `translations.lua`.
- Menu price labels are now translation-driven (including the duration text).
- Removed hardcoded `($price / 1 hour)` menu text from script logic.
- Added translation keys for service-disabled and insufficient-funds reason messages.
- Moved spawn command notifications into translation keys.

## :gear: Translation Keys Added
- Ride menu label with price formatting (`menu_ride_price`)
- Rental menu label with price + duration formatting (`menu_rent_price`, `menu_rental_duration`)
- Service unavailable messages (`taxi_service_disabled`, `rental_service_disabled`, `service_unavailable`)
- Dynamic insufficient funds reasons (`taxi_insufficient_funds_required`, `rental_insufficient_funds_required`)
- Spawn command status text (`spawn_failed_model`, `spawn_failed_create`, `spawn_success`)

## :shield: Safety Checks
- Added server-side validation to deny taxi requests when taxi service is disabled.
- Added server-side validation to deny rental requests when rental service is disabled.
```

### Technical Summary
- Decoupled service startup checks in client taxi logic from taxi-only mode to any-enabled mode.
- Menu options are now built dynamically based on each toggle.
- Added explicit disabled-service messaging for blocked requests.
- Replaced remaining direct player-facing literals in taxi, rental, and spawn flows with T(...) keys.
- Server denial reasons now use translation keys instead of inline strings.
