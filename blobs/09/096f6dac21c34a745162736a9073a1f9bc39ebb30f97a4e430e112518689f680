fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_chess'
author 'Poggy'
description 'Chess and checkers tables: play a friend or the AI, four checkers variants and house rules, wagers held safely, and a record of every game with a move-by-move review.'
version '2.3.0'
poggy_core_min '0.27.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'translations.lua',
    'shared/sh_locale.lua',
    'shared/sh_board.lua',
    'shared/sh_variants.lua',
}

client_scripts {
    '@poggy_core/client/lib/prompts.lua',
    'client/cl_objects.lua',
    'client/cl_camera.lua',
    'client/cl_npc.lua',
    'client/cl_seat.lua',
    'client/cl_ui.lua',
    'client/cl_session.lua',
    'client/cl_main.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/sv_database.lua',
    'server/sv_chess.lua',
    'server/sv_checkers.lua',
    'server/sv_engine.lua',
    'server/sv_ai.lua',
    'server/sv_wager.lua',
    'server/sv_ratings.lua',
    'server/sv_clock.lua',
    'server/sv_game.lua',
    'server/sv_records.lua',
    'server/sv_main.lua',
}

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- it loads the first time the script opens it, not on every player's game at join.
poggy_ui 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/app.js',
    'ui/board.js',
    'ui/screens.js',
    'ui/play.js',
    'ui/record.js',
    'ui/img/*.png',
    'ui/sfx/chess/*.mp3',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'translations.lua',
    'sql/*.sql',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'