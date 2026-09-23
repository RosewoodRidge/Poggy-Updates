fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_balloon'
author 'Poggy'
description 'Hot air balloons with an NPC taxi service to waypoints, self-piloted rentals and passenger animations.'
version '1.6.2'
poggy_core_min '0.14.0'

dependencies {
    'poggy_core',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'translations.lua',
}

-- Load order matters: balloon_controls.lua defines the globals BalloonBurnHeld /
-- BalloonDescendHeld that balloonanimations.lua reads.
client_scripts {
    '@poggy_core/client/lib/prompts.lua',
    'client/balloon_controls.lua',
    'client/balloon.lua',
    'client/balloonanimations.lua',
    'client/balloon_taxi.lua',
    'client/balloon_spawn.lua',
    'client/balloon_rental.lua',
}

server_scripts {
    'server/balloon_server.lua',
    'server/balloon_taxi_server.lua',
    'server/balloon_rental_server.lua',
}

files {
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'