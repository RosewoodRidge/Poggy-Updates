fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_character_storage'
author 'Poggy'
description 'Poggy Storage - Player-owned storage for RedM.'
version '1.3.0'
poggy_core_min '0.13.0'

-- poggy_core provides character data, money, containers and notifications.
-- The menu and the text prompts are this resource's own, so vorp_menu and
-- vorp_inputs are not needed. The armory and /adminshop use vorp_inventory's
-- store window directly.
dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
}

client_scripts {
    'client/ui.lua',
    'client/storage.lua',
    'client/menu.lua',
    'client/npc.lua',
    'client/shop.lua',
}

server_scripts {
    'server/containers.lua',
    'server/player.lua',
    'server/database.lua',
    'server/storage.lua',
    'server/shop.lua',
    'server/discord.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/style.css',
    'ui/script.js',
}

-- Readable by owners and by the updater's config merge. The translations live
-- inside config.lua.
escrow_ignore {
    'config.lua',
}

-- Metadata only: the exports server/storage.lua registers.
server_exports {
    'GetDatabaseAPI',
    'GetAuthorizedUsers',
    'RegisterAllStorageInventories',
    'RefreshAllPlayerStorages',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'