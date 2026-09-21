fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_emotes'
author 'Poggy'
description 'A searchable emote menu with 330+ animations, favourites, recents and a live preview.'
version '1.0.1'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'translations.lua',
    'shared/emotes.lua',
    'shared/catalogue.lua',
}

client_scripts {
    'client/player.lua',
    'client/preview.lua',
    'client/ragdoll.lua',
    'client/main.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
    'docs/icon.png',
}

-- Open where an owner needs it: settings, words, the emote list (so its field
-- reference can be read) and the database file. The rest is escrowed.
escrow_ignore {
    'config.lua',
    'translations.lua',
    'shared/emotes.lua',
    'sql/*.sql',
    'sql/**',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'