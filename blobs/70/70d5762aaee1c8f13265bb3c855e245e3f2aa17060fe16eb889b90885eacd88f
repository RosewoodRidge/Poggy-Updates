fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_auction'
author 'Poggy'
description 'Auction house where players list, bid on, buy out and collect items, with a supply catalogue and want requests.'
version '1.3.0'
poggy_core_min '0.17.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'translations.lua',
}

client_scripts {
    'client/utils.lua',
    'client/callbacks.lua',
    'client/main.lua',
    'client/nui.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/players.lua',
    'server/callbacks.lua',
    'server/database.lua',
    'server/auction.lua',
    'server/main.lua',
    'server/discord.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'