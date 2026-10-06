fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_emotes'
author 'Poggy'
description 'A searchable emote menu with 330+ animations, favourites, recents, a live preview, and a Nearby tab for the chairs, bars and props around you.'
version '1.2.0'
poggy_core_min '0.27.0'

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
    'client/dataview.lua',
    'client/object_names.lua',
    'client/scenario_names.lua',
    'client/player.lua',
    'client/nearby.lua',
    'client/preview.lua',
    'client/ragdoll.lua',
    'client/main.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua',
}

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- it loads the first time the script opens it, not on every player's game at join.
poggy_ui 'ui/index.html'

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