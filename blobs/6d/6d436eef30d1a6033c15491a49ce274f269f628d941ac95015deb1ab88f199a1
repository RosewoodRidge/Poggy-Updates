fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_fishing'
author 'Poggy'
description 'Skillcheck-based rod fishing with zone-specific fish.'
version '1.3.3'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
    'oxmysql',
    'poggy_skillcheck',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'shared/fishing.lua',
}

client_scripts {
    'client/client.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/server.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/app.js',
    'ui/skin.js',
    'ui/skin-*.css',
    'ui/skin-*.png',
    'ui/img/*.png',
    'ui/sfx/*.mp3',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'