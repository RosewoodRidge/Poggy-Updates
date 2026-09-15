/* ============================================================================
   poggy_fishing — interface skins
   ----------------------------------------------------------------------------
   Config.UI.skin arrives with every message that opens a panel (show,
   showBaitMenu).  "default" is the built-in dark water theme; any other name
   loads ui/skin-<name>.css on top of it and marks <html data-skin>.

   A skin is CSS and images only.  A missing stylesheet or image leaves the
   default look underneath, so a half-finished skin never breaks the HUD.

   Loaded before app.js, so the skin is applied before the HUD renders for
   the message that carried it.  Same mechanism as poggy_markets/ui/js/skin.js.
   ============================================================================ */

(function () {
    'use strict';

    var current = 'default';

    function apply(name) {
        // Folder-safe names only: the value becomes part of a file path.
        name = String(name || '').toLowerCase().replace(/[^a-z0-9_-]/g, '') || 'default';
        if (name === current) return;
        current = name;

        var link = document.getElementById('pf-skin');
        if (name === 'default') {
            if (link) link.parentNode.removeChild(link);
            document.documentElement.removeAttribute('data-skin');
            return;
        }

        if (!link) {
            link = document.createElement('link');
            link.id = 'pf-skin';
            link.rel = 'stylesheet';
            document.head.appendChild(link);
        }
        link.href = 'skin-' + name + '.css';
        document.documentElement.setAttribute('data-skin', name);
    }

    window.addEventListener('message', function (event) {
        var d = event.data;
        if (d && typeof d.skin === 'string') apply(d.skin);
    });
}());
