fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_crafting'
author 'Poggy'
description 'Crafting benches, campfire cooking and a recipe browser with chains, a shopping list and a gathering tracker.'
version '2.1.0'
poggy_core_min '0.27.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

-- poggy_skillcheck is OPTIONAL. Recipes that set `skillcheck` or `jobSkillcheck`
-- need it; every other recipe works without it. See Config.Skillcheck.

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config/config.lua',
    'config/recipes.lua',
    'config/item_sources.lua',
    'translations.lua',
    'shared/recipes.lua',
}

client_scripts {
    '@poggy_core/client/lib/prompts.lua',
    'client/animations.lua',
    'client/campfire.lua',
    'client/ui.lua',
    'client/craft.lua',
    'client/main.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua',
    'server/icons.lua',
    'server/craft.lua',
    'server/shoppinglist.lua',
}

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- it is loaded only when a player first opens crafting, instead of on every
-- player's game from the moment they join. Same files, same look.
poggy_ui 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/app.js',
    'ui/vendor/vue.js',
    'ui/img/placeholder.png',
    'docs/icon.png',
}

-- Free and open source (GPL v2, from VORP's vorp_crafting): nothing is
-- encrypted when this goes through the Cfx portal.
escrow_ignore {
    'config/*.lua',
    'translations.lua',
    'sql/*.sql',
    'client/*.lua',
    'server/*.lua',
    'shared/*.lua',
    'ui/*',
    'ui/**',
}
