fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_character_storage'
author 'Poggy'
description 'Poggy Storage - Player-owned storage for RedM.'
version '1.5.2'
poggy_core_min '0.14.0'

-- poggy_core provides character data, money, containers, notifications, the
-- menus and the text prompts. Nothing here talks to a framework directly.
dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
}

client_scripts {
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
    'server/hub.lua',
}

-- The card icon the /poggy settings hub shows.
files {
    'docs/icon.png',
}

-- Readable by owners and by the updater's config merge. The translations live
-- inside config.lua.
escrow_ignore {
    'config.lua',
}

-- Metadata only: the exports server/storage.lua and server/hub.lua register.
server_exports {
    'GetDatabaseAPI',
    'GetAuthorizedUsers',
    'RegisterAllStorageInventories',
    'RefreshAllPlayerStorages',
    'HubPanel',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'