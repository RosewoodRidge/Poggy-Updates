fx_version "cerulean"
games { "rdr3" }
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'
lua54 "yes"

-- The product id the update feed knows this resource by. Never changes.
poggy_id 'poggy_core'
author "Poggy"
description "Poggy Core — one documented framework API for RedM (VORP, RSG, QBR, RedEM:RP, RPX)"
version "0.14.1"

-- VORP, RSG and QBR adapters + standalone fallback. See README.md for what is
-- and is not implemented yet. Adding a framework means adding one file under
-- server/adapters/ and one entry in shared/sh_detect.lua.

shared_scripts {
    "config.lua",
    "shared/sh_api.lua",
    "shared/sh_detect.lua",
    "shared/sh_verbs.lua",
}

client_scripts {
    "client/cl_core.lua",
    "client/cl_dispatch.lua",
    "client/cl_callbacks.lua",
    "client/cl_notify_native.lua",   -- the RDR2 notification natives; before cl_notify.lua
    "client/cl_notify.lua",
    "client/cl_prompt.lua",
    "client/cl_menu.lua",            -- the menu and text input (ui/); after cl_callbacks.lua, which it registers with
}

server_scripts {
    "server/sv_util.lua",
    "server/sv_identity.lua",        -- poggy_id -> folder; before everything that looks a script up
    "server/sv_sql.lua",
    "server/adapters/standalone.lua",
    "server/adapters/vorp.lua",
    "server/adapters/rsg.lua",
    "server/adapters/qbr.lua",
    "server/sv_notify.lua",
    "server/sv_storage.lua",
    "server/sv_usables.lua",         -- usable-item registry; before sv_core.lua, which binds it
    "server/sv_core.lua",
    "server/sv_dispatch.lua",
    "server/sv_callbacks.lua",
    "server/sv_ui.lua",              -- menu / input round trip to a client; after sv_callbacks.lua
    "server/sv_selftest.lua",
    "server/sv_configmerge.lua",
    "server/sv_updates.lua",
    "server/sv_nettest.lua",
    "server/sv_commands.lua",
}

-- The menu and text input every Poggy script uses (menu.open, input.text).
-- Hidden until asked for; nothing in it needs the internet.
ui_page "ui/index.html"

files {
    "template/poggy.lua",
    -- Included by other resources as @poggy_core/client/lib/prompts.lua;
    -- deliberately not a client_script here, it does nothing in poggy_core itself.
    "client/lib/prompts.lua",
    "ui/index.html",
    "ui/style.css",
    "ui/script.js",
}

-- Everything in this resource is meant to be readable by the people building
-- against it. Nothing is escrowed.
escrow_ignore {
    "config.lua",
    "shared/*.lua",
    "client/*.lua",
    "client/lib/*.lua",
    "server/*.lua",
    "server/adapters/*.lua",
    "template/*.lua",
    "ui/*",
}
