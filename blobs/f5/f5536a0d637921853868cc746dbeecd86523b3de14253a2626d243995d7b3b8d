fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_markets'
author 'Poggy'
description 'Player-owned stores, dynamic pricing and a commodities exchange for RedM.'
version '1.2.0'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config/config.lua',
    'config/keys.lua',
    'config/stores.lua',
    'config/catalog_default.lua',
    'config/pricing.lua',
    'config/exchange.lua',
    'config/ghostbuyer.lua',
    'translations.lua',
    'shared/math.lua',
}

client_scripts {
    'client/core.lua',
    'client/ui.lua',
    'client/exchange.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/player.lua',
    'server/inventory.lua',
    'server/core.lua',
    'server/trade.lua',
    'server/shops.lua',
    'server/ui.lua',
    'server/admin.lua',
    'server/pricing.lua',
    'server/exchange.lua',
    'server/ghostbuyer.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/css/manager.css',
    'ui/css/exchange.css',
    'ui/js/manager.js',
    'ui/js/exchange.js',
    'ui/js/prompt.js',
    'ui/lib/chart.min.js',
    'ui/lib/chart-annotation.min.js',
}

escrow_ignore {
    'config/*.lua',
    'translations.lua',
    'sql/*.sql',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'