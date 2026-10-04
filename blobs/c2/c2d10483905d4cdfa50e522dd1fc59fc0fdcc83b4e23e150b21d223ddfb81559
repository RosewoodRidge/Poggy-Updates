// Making a folder (0.26.2).
//
// A Lua server script cannot create a folder: SaveResourceFile only writes into
// folders that already exist. A JavaScript server script can, with one limit the
// server enforces itself: only inside the resource the script is running in
// (proven on a live server, 4 October 2026; a write into any other resource is
// refused with ERR_ACCESS_DENIED).
//
// So this file makes folders in ITS OWN resource and nowhere else. poggy_core
// lists it, which covers poggy_core's folders (update_backups, ui/hub/...). Any
// other script that lists it in its manifest,
//     server_script '@poggy_core/server/sv_folders.js'
// gets the same export under its own name, and the updater can then install a
// version of that script that adds a folder.
//
//     exports[<resource>]:PoggyMakeOwnFolder("docs/help")  ->  true | "why not"

const fs = require('fs');
const path = require('path');

exports('PoggyMakeOwnFolder', (rel) => {
    try {
        if (typeof rel !== 'string' || rel === '' || rel.length > 200) return 'not a folder name';
        const clean = rel.replace(/\\/g, '/').replace(/\/+$/, '');
        if (clean === '' || clean.startsWith('/') || /^[A-Za-z]:/.test(clean)) return 'not a relative path';
        if (clean.split('/').some((part) => part === '' || part === '.' || part === '..')) return 'not a plain relative path';
        const base = path.resolve(GetResourcePath(GetCurrentResourceName()));
        const target = path.resolve(base, clean);
        if (!target.startsWith(base + path.sep)) return 'outside the resource';
        fs.mkdirSync(target, { recursive: true });
        return true;
    } catch (e) {
        return String((e && e.message) || e);
    }
});
