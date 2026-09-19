fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 'yes'

poggy_id 'poggy_util'
author 'Poggy'
description 'Optional server utilities: AOP, armor, clock, duty count, music zones, object removal, unstuck, weapon jam, help menu, stipend.'
version '2.0.6'
poggy_core_min '0.13.0'

dependencies {
    'poggy_core',
    'oxmysql',
}

shared_scripts {
    '@poggy_core/template/poggy.lua',
    'config.lua',
    'config/help.lua',
    'shared/helpers.lua',
}

client_scripts {
    'client/aop.lua',
    'client/armor.lua',
    'client/clock_weather.lua',
    'client/duty_count.lua',
    'client/help.lua',
    'client/music.lua',
    'client/object_removal.lua',
    'client/unstuck.lua',
    'client/weapon_jam.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/aop.lua',
    'server/armor.lua',
    'server/clock_weather.lua',
    'server/duty_count.lua',
    'server/object_removal.lua',
    'server/stipend.lua',
}

ui_page 'ui/index.html'

files {
    'ui/index.html',
    'ui/clock.css',
    'ui/clock.js',
    'ui/help.css',
    'ui/help.js',
    'ui/music.css',
    'ui/sounds.js',
    'ui/img/armor.png',
    'ui/img/empty_bg.png',
    'ui/img/meter/rpg_tank_1.png',
    'ui/img/meter/rpg_tank_2.png',
    'ui/img/meter/rpg_tank_3.png',
    'ui/img/meter/rpg_tank_4.png',
    'ui/img/meter/rpg_tank_5.png',
    'ui/img/meter/rpg_tank_6.png',
    'ui/img/meter/rpg_tank_7.png',
    'ui/img/meter/rpg_tank_8.png',
    'ui/img/meter/rpg_tank_9.png',
    'ui/img/meter/rpg_tank_10.png',
    'ui/sfx/weaponjam/gun_empty1.wav',
    'ui/sfx/weaponjam/gun_empty2.wav',
    'ui/sfx/weaponjam/gun_empty3.wav',
    'docs/icon.png',
}

escrow_ignore {
    'config.lua',
    'config/help.lua',
}

dependency '/assetpacks'
dependency '/assetpacks-redm'