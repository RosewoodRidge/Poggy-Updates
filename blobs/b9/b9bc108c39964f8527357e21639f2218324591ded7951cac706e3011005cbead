fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_tickets'
author 'Poggy'
description 'Player tickets and help requests, with a staff panel: your own staff roles, staff chat, claim, escalate, reply, teleport, warn, kick, ban, Discord.'
version '1.1.1'
poggy_core_min '0.19.0'        -- ban.add, ban.remove, ban.list, player.kick, player.identifiers

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'translations.lua',
    'shared/sh_tickets.lua',
}

client_scripts {
    'client/cl_place.lua',   -- the game-specific bits; before cl_main.lua
    'client/cl_main.lua',
    'client/cl_staff.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/sv_database.lua',
    'server/sv_roles.lua',
    'server/sv_staff.lua',
    'server/sv_discord.lua',
    'server/sv_chat.lua',
    'server/sv_canned.lua',
    'server/sv_tickets.lua',
    'server/sv_moderation.lua',
    'server/sv_web.lua',
    'server/sv_hubpanel.lua',
    'server/sv_callbacks.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/app.js',
    'ui/sfx/incoming.mp3',
    'ui/fonts/*.woff2',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'