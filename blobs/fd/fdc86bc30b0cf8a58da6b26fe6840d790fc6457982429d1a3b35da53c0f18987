fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_multijob'
author 'Poggy'
description 'Multijob System'
version '1.8.0'
poggy_core_min '0.27.0'

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
    'server/joblist.lua',
    'server/main.lua',
    'server/admin.lua',
    'server/api.lua',
    'server/provider.lua',
}

-- The admin screen is shown by poggy_core (0.27.0), not as a page of this
-- script's own: it is loaded only when /mjadmin opens it, and unloaded again
-- after two minutes closed.
poggy_ui 'ui/index.html'
poggy_ui_idle '120'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'