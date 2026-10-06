fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_scenes_2'
author 'Poggy'
description 'Poggy Scenes 2.0 - real sign boards you type on, scene text, status tags and the /me box for RedM.'
version '2.1.0'
poggy_core_min '0.27.0'

dependencies {
    'poggy_core',
    'oxmysql',
    '/assetpacks',
    '/assetpacks-redm',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'client/sign_metrics.lua',   -- letter widths and what fits a sign: both sides measure with it
}

client_scripts {
    'client/signs.lua',
    'client/main.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua',
    'server/hub.lua',
}

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- an on-screen display, so it loads when the player joins, as before.
poggy_ui 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
    'ui/sym_*.png',              -- the sign symbols, as the editor shows them
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'client/main.lua',
}

-- Metadata only: the export server/hub.lua registers (the /poggy Scenes panel).
server_exports {
    'HubPanel',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'