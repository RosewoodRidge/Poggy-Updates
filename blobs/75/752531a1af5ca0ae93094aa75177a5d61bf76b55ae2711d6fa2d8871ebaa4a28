fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_supplydrops'
author 'Poggy'
description 'Random supply drops and scavenger hunts across the map, with clues, rewards and Discord logging.'
version '1.5.1'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'translations.lua',
}

client_scripts {
    'client/main.lua',
    'client/notifications.lua',
    'client/supply_drops.lua',
    'client/scavenger_hunts.lua',
}

server_scripts {
    'server/players.lua',
    'server/main.lua',
    'server/discord.lua',
    'server/supply_drops.lua',
    'server/scavenger_hunts.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
    'ui/images/*.jpg',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
    'ui/images/*.jpg',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'