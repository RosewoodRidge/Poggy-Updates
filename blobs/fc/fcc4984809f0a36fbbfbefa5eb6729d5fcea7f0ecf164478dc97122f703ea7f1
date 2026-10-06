fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_skillcheck'
author 'Poggy'
description 'Poggy Skillcheck - Standalone DBD-style circular skillcheck system.'
version '1.1.0'
poggy_core_min '0.27.0'

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

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- it loads the first time the script opens it, not on every player's game at join.
poggy_ui 'ui/index.html'

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