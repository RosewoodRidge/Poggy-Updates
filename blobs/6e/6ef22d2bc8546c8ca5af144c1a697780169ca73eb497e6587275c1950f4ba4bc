fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_transform'
author 'Poggy'
description 'Poggy Transform - Become any animal, any ped, or any character on your server.'
version '1.3.0'
poggy_core_min '0.27.0'

dependencies {
    'poggy_core',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
}

client_scripts {
    'client/client.lua',
}

server_scripts {
    'server/permissions.lua',
    'server/server.lua',
}

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- it loads the first time the script opens it, not on every player's game at join.
poggy_ui 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
    'ui/images/*.png',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'server/permissions.lua',
    'ui/style.css',
    'ui/images/*.png',
    'README.md',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'