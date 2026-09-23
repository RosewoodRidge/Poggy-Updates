fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_skillcheck'
author 'Poggy'
description 'Poggy Skillcheck - Standalone DBD-style circular skillcheck system.'
version '1.0.3'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
}

client_scripts {
    'client/cl_skillcheck.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/skillcheck.css',
    'ui/skillcheck.js',
    'ui/sfx/skillcheck/*.mp3',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'ui/sfx/skillcheck/*.mp3',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'