fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_trashbins'
author 'Poggy'
description 'Poggy Trash Bins'
version '1.7.0'
poggy_core_min '0.28.0'

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
}

server_scripts {
    'server/main.lua',
}

-- The screen is shown by poggy_core, not as a page of this script's own: it loads
-- the first time a bin is searched, not on every player's game at join. It is
-- drawn with poggy_core's Poggy UI kit (0.28.0): the source is ui-src/ in
-- poggy-src, built into ui/ by tools/ui-vue.
poggy_ui 'ui/index.html'

files {
    'ui/index.html',
    'ui/app.js',
    'ui/app.css',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'