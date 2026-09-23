fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_badge'
author 'Poggy'
description 'Poggy Badge – custom badge attachment, editor & display'
version '1.3.2'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
}

client_scripts {
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
    'ui/images/*.png',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'