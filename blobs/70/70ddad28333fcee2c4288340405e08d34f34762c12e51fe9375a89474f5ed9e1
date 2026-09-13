# Poggy's Balloon - Enhanced Hot Air Balloon System for RedM

A comprehensive hot air balloon system for RedM featuring NPC-piloted taxi services, player-controlled rentals, enhanced flight animations, and multi-language support. No more passengers ragdolling in the basket and captains awkwardly standing still!

## Features

### 🎈 Balloon Taxi Service
- **NPC-Piloted Rides**: Talk to balloon operators at designated locations across the map
- **Waypoint Navigation**: Set a waypoint and the AI pilot flies you there automatically
- **Terrain Avoidance**: Smart altitude adjustments to avoid mountains and obstacles
- **Manual Take-Off**: Board the balloon and press **B** when you're ready to depart
- **In-Flight Controls**:
  - **Q** - Request early stop (land at current position)
  - **R** - Change destination (reroute to new waypoint)
  - **A** - "Step on it!" (toggle speed boost - 22 m/s → 30 m/s)
  - **C** - Change seat (cycle through seats 0-3)
  - **F** - Disembark (teleport safely out of balloon)
- **Multiple Locations**: Strawberry, Rhodes, Valentine, Annesburg, Saint Denis, Blackwater, Armadillo, Tumbleweed

### 🎫 Balloon Rental System
- **Self-Piloted Rentals**: Rent a balloon for 1 hour and fly it yourself
- **Rental Timer**: Visual warnings before rental expires
- **Available at All Taxi Locations**: Where rentals are enabled in config

### 🌍 Multi-Language Support
- **4 Languages Included**: English, French, Spanish, German
- **Easy to Extend**: Add new languages in `translations.lua`
- **Change Language**: Set `Config.Language` in translations.lua

### 🔔 VORP Notification System
- **Context-Aware Icons**:
  - `blip_location_higher` - Ascending
  - `blip_location_lower` - Descending/landing
  - `blip_cash_arthur` - Purchase made
  - `blip_poi` - General notifications
  - `blip_region_hunting` - Rerouting
  - `blip_destroy` - Errors/cancellations

### ✨ Core Balloon Features
- **Enhanced Balloon Controls**: Camera-relative movement options make flying more intuitive
- **Multiple Passenger Support**: Up to 4 passengers can ride in the balloon basket safely
- **Captain Animation System**: Realistic burner pull animation with rope visual
- **Passenger Animations**: Proper sitting animations for passengers
- **Altitude Lock**: Lock balloon height for stable horizontal navigation
- **Invisible Safety Floor**: Prevents passengers from ragdolling inside the basket
- **Server Synchronization**: All players see consistent passenger positions and animations
- **Prompt System**: Clear UI prompts for entering/exiting and controlling the balloon
- **Boost & Brake Controls**: Fine-tune your balloon's speed with dedicated controls

## Dependencies

- **poggy_core**: Money, character checks and UI prompts
- **vorp_core**: Bottom and right-hand notifications
- **vorp_menu**: For NPC interaction menus

## Installation

1. Extract the `poggy_balloon` folder into your server's `resources` directory
2. Add `ensure poggy_balloon` to your server.cfg (after poggy_core and the vorp dependencies)
3. Configure settings in `config.lua`
4. Set your preferred language in `translations.lua`
5. Restart your server

### Upgrading from poggy-balloon

The folder was renamed from `poggy-balloon` to `poggy_balloon`. Delete the old `poggy-balloon` folder, drop in `poggy_balloon`, and change `ensure poggy-balloon` to `ensure poggy_balloon` in your server.cfg. Keep your old `config.lua` and `translations.lua` if you changed them; the settings are the same.

## Configuration

### config.lua
```lua
Config.BalloonTaxi = {
    Enabled = true,
    Price = 3,                    -- Cost for taxi ride
    CruisingAltitude = 80.0,      -- Flight altitude in meters
    TravelSpeed = 22.0,           -- Normal speed (m/s)
    TravelSpeedFast = 30.0,       -- "Step on it" speed (m/s)
    -- ... more settings
}

Config.BalloonRental = {
    Enabled = true,
    Price = 50,                   -- Rental cost
    RentalDurationMinutes = 60,   -- 1 hour
    WarningTimeMinutes = 5,       -- 5 minute warning
}
```

### translations.lua
```lua
Config.Language = "en"  -- Options: "en", "fr", "es", "de"
```

## How It Works

### 🚖 Using the Taxi Service
1. Find a balloon tour NPC (marked with blips on map)
2. Press **G** to talk to the operator
3. Set a waypoint on your map before requesting a ride
4. Select "Take a Ride" from the menu
5. Enter the balloon as a passenger when prompted
6. Enjoy the ride! Use Q/R/G for in-flight controls

### 🎈 Renting a Balloon
1. Talk to any balloon operator with rentals enabled
2. Select "Rent a Balloon" from the menu
3. Enter the balloon as the pilot
4. Fly anywhere you want for 1 hour!

### 🎮 Manual Flying (Captain)
1. Enter the balloon as normal using the game's vehicle entry system
2. Controls while piloting:
   - **W/S**: Move forward/backward 
   - **A/D**: Move left/right 
   - **Shift** (hold): Burn and climb
   - **Ctrl** (hold): Descend
   - **F**: Boost (increase speed)
   - **R**: Brake (slow down)
   - **Space**: Toggle altitude lock

### 👥 Riding as Passenger
1. Approach a balloon and press **F** when prompted
2. Press **F** again to exit
3. Up to 4 passengers in designated basket positions

## Taxi Locations

| Location | Coordinates | Rental Available |
|----------|-------------|------------------|
| Strawberry | (-1812.9, -677.43, 149.6) | ✅ |
| Rhodes | (1425.01, -1273.0, 77.27) | ✅ |
| Valentine | (-145.7, 611.94, 114.39) | ✅ |
| Annesburg | (2865.34, 1427.64, 67.41) | ✅ |
| Saint Denis | (2717.29, -1511.52, 43.19) | ✅ |
| Blackwater | (-697.71, -1285.4, 42.22) | ✅ |
| Armadillo | (-3738.5, -2589.45, -14.3) | ✅ |
| Tumbleweed | (-5468.53, -2962.53, -1.4) | ✅ |

## Debugging

Debug commands for troubleshooting:

- `/balloon_debug_main [level|toggle|status]` - Main module debug
- `/balloon_debug_anim [level|toggle]` - Animation module debug
- `/balloon_debug_ctrl [level|toggle]` - Controls module debug
- `/balloon_server_status` - View balloon occupancy (admin)
- `/balloon_debug_ride` - Show current taxi ride state

Debug levels: 0 (OFF) → 4 (DEBUG), default 3 (INFO)

## File Structure

```
poggy_balloon/
├── fxmanifest.lua
├── config.lua              # All configuration settings
├── translations.lua        # Multi-language text
├── README.md
├── CHANGELOG.md
├── client/
│   ├── balloon_controls.lua
│   ├── balloon.lua         # Core balloon mechanics
│   ├── balloonanimations.lua
│   ├── balloon_taxi.lua    # Taxi service
│   ├── balloon_spawn.lua
│   └── balloon_rental.lua  # Rental system
└── server/
    ├── balloon_server.lua
    ├── balloon_taxi_server.lua   # Taxi server logic
    └── balloon_rental_server.lua # Rental server logic
```

## Technical Details

### Taxi Flight System
- Uses RDR2 AITRANSPORT natives for balloon-specific vehicle handling
- Intelligent terrain look-ahead for mountain avoidance
- Smooth velocity interpolation for realistic flight feel
- State machine: WAITING_FOR_ENTRY → ASCENDING → CRUISING → APPROACHING → LOCKING_POSITION → WAITING_EXIT

### Passenger Detection
Uses multiple methods for reliable balloon occupancy detection:
- `_IS_PED_ON_TRANSPORT_ENTITY` (0x159EF5B6EDCE00E8)
- Attachment and distance checks as fallbacks

### Animation System
- **Captains**: Animated burner controls with rope visuals
- **Passengers**: Proper sitting animations to prevent T-poses
- **AI Pilots**: Idle burner animations during taxi flights

## Credits

This resource combines and enhances code from:
- [kibook's side saddle](https://github.com/kibook/redm-sidesaddle)
- [kibook's balloon control scripts](https://github.com/kibook/redm-ballooncontrols)
- [PersePixel's balloon crap](https://github.com/PersePixels/balloon-crap)

Enhanced with taxi service, rental system, and translations by Poggy.

## License & Legal

This is a free resource, modified from other open source projects.
Feel free to use and modify for your server, but please credit the original authors.

## Video Example

https://medal.tv/games/red-dead-2/clips/kizULvk7B3R3miNy1