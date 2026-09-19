fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_witnesses'
author 'Poggy'
description 'Witness system for RedM (runs on poggy_core)'
version '1.3.1'
poggy_core_min '0.14.0'

dependencies {
    'poggy_core',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'translations.lua', -- before config: config/config.lua calls T() while it loads
    'config/config.lua',
    'config/npc.lua',
    'shared/api.lua', -- public CreateWitness API (server export + client handler)
    '@PolyZone/client.lua', -- Download here: https://github.com/mkafrin/PolyZone
    '@PolyZone/CircleZone.lua',
}

client_scripts {
    'client/client_action_detection.lua',
    'client/client_blips.lua',
    'client/client_event_handlers.lua',
    'client/client_global.lua',
    'client/client_init.lua',
    'client/client_jacking_detection.lua',
    'client/client_job_alerts.lua',
    'client/client_job_integration.lua',
    'client/client_notifications_override.lua',
    'client/client_npc.lua',
    'client/client_utils.lua',
    'client/client_witness_core.lua',
    'client/client_witness_helpers.lua',
    'client/client_witness_tracking.lua',
}

server_scripts {
    'server/server_core.lua',
    'server/server_npc.lua',
}

files {
    'docs/icon.png',
}

escrow_ignore {
    'config/config.lua',
    'config/npc.lua',
    'translations.lua',
    'shared/api.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'