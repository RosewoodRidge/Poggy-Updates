fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_multijob'
author 'Poggy'
description 'Multijob System'
version '1.6.0'
poggy_core_min '0.13.0'

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
    'client/main.lua',
    'client/menu.lua',
    'client/nui.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua',
    'server/admin.lua',
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
}

dependency '/assetpacks'
dependency '/assetpacks-redm'