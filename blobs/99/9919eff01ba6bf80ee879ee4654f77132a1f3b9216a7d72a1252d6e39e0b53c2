fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_animtool'
author 'Poggy'
description 'Poggy AnimTool - Timeline editor for RedM animation scenes (clips and keyframed props) with Lua export.'
version '2.2.0'
poggy_core_min '0.27.0'

dependencies {
    'poggy_core',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
}

client_scripts {
    'client/scene_player.lua',
    'client/client.lua',
}

server_scripts {
    'server/server.lua',
}

-- The screen is shown by poggy_core (0.27.0), not as a page of this script's own:
-- it loads the first time the script opens it, not on every player's game at join.
poggy_ui 'ui/index.html'

files {
    'ui/index.html',
    'ui/css/app.css',
    'ui/css/timeline.css',
    'ui/js/util.js',
    'ui/js/nui.js',
    'ui/js/state.js',
    'ui/js/browser.js',
    'ui/js/timeline.js',
    'ui/js/inspector.js',
    'ui/js/export.js',
    'ui/js/app.js',
    'ui/data/anim_chunk_*.json',
    'ui/data/obj_chunk_*.json',
    'client/scene_player.lua', -- served to the UI for the export dialog
    'docs/icon.png',
}

-- The playback runtime is handed to the user from the export dialog and shipped
-- inside their own resources, so it must stay readable.
escrow_ignore {
    'client/scene_player.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'