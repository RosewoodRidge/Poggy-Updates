# Poggy's Supply Drops & Scavenger Hunts

A dynamic resource for RedM servers that adds randomly spawning supply drops and engaging scavenger hunts throughout the world. Players can discover, collect, and earn rewards from these activities, enhancing server economy and exploration.

## Features

- **Framework Agnostic**: Every framework call goes through poggy_core
- **Dynamic Supply Drops**: Randomly spawning supply crates at configured locations throughout the map
- **Scavenger Hunts**: Treasure hunts with clues that players can solve to find hidden rewards
- **Configurable Rewards**: Fully customizable item drops and monetary rewards
- **Visual Feedback**: Custom blips, props, and notifications for all activities
- **Automatic Scheduling**: Server-side timers to spawn events at configured intervals
- **Admin Commands**: Manually spawn supply drops or scavenger hunts
- **Discord Integration**: Webhook notifications for new events and player collections
- **Penalty System**: Optional penalties for revealing scavenger hunt clues
- **Performance Optimized**: Distance-based rendering and efficient resource usage
- **Extensive Configuration**: Highly customizable via config.lua

## Dependencies

- poggy_core 0.13.0 or newer (it detects and drives your framework)

## Installation

1. Extract the `poggy_supplydrops` folder into your server's `resources` directory
2. Add `ensure poggy_supplydrops` to your server.cfg, after `ensure poggy_core`
3. Configure the script by editing config.lua
4. Place any custom images for scavenger hunts in the ui/images/ folder
5. Restart your server

## Configuration

The script is highly configurable through the `config.lua` file:

- **Global Settings**: Debug mode, notification settings, and collection notifications
- **Discord Webhook Settings**: Configure separate webhooks for supply drops and scavenger hunts
- **Feature Toggles**: Enable/disable supply drops and scavenger hunts independently
- **Supply Drop Options**: Configure intervals, locations, timeout periods, and item rewards
- **Scavenger Hunt Options**: Set up hunt intervals, clue system, penalties, and rewards
- **Notification Appearance**: Customize colors and icons for success notifications

## How It Works

### Supply Drops
1. Supply drops spawn at random intervals at configured locations
2. Players are notified when a new supply drop appears
3. Players can travel to the drop location (marked on the map)
4. Players can interact with the drop to collect the supplies
5. Drops disappear after collection or when they timeout

### Scavenger Hunts
1. Scavenger hunts spawn at random intervals at configured locations
2. Players are notified when a new hunt begins
3. Players can use the `/clue` command to see information about the hunt
4. Players can choose to reveal more specific clues (with a potential reward penalty)
5. When players find the hunt location, they can interact to collect the reward

## Admin Commands

- `/supplydrop` - Spawn a supply drop (admin only)
- `/scavengerhunt` - Spawn a scavenger hunt (admin only)
- `/testdiscord [supply|scavenger]` - Test Discord webhook functionality (console only)

## Discord Integration

Configure Discord webhooks to receive notifications about:
- New supply drops spawning
- Players collecting supply drops
- New scavenger hunts beginning
- Players completing scavenger hunts (with or without using clues)

## License & Credits

© 2023 Poggy's Scripts - All Rights Reserved

This resource is for use on your RedM server only. Redistribution, resale, or modification beyond configuration is not permitted without explicit permission.

Created by Poggy for RedM VORP Framework.
