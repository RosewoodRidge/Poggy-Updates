/* ============================================================================
   poggy_markets — interface skins
   ----------------------------------------------------------------------------
   Config.UI.skin arrives with every open message (open, custOpen,
   exchange:open, prompt).  "default" is the built-in dark theme; any other
   name loads css/skin-<name>.css on top of it and marks <html data-skin>.

   A skin is CSS and images only.  A missing stylesheet or image leaves the
   default look underneath, so a half-finished skin never breaks a panel.

   Loaded before the other scripts, so the skin is applied before a panel
   renders for the message that carried it.
   ============================================================================ */

(function () {
    'use strict';

    var current = 'default';

    function apply(name) {
        // Folder-safe names only: the value becomes part of a file path.
        name = String(name || '').toLowerCase().replace(/[^a-z0-9_-]/g, '') || 'default';
        if (name === current) return;
        current = name;

        var link = document.getElementById('pm-skin');
        if (name === 'default') {
            if (link) link.parentNode.removeChild(link);
            document.documentElement.removeAttribute('data-skin');
            return;
        }

        if (!link) {
            link = document.createElement('link');
            link.id = 'pm-skin';
            link.rel = 'stylesheet';
            document.head.appendChild(link);
        }
        link.href = 'css/skin-' + name + '.css';
        document.documentElement.setAttribute('data-skin', name);
    }

    window.addEventListener('message', function (event) {
        var d = event.data;
        if (d && typeof d.skin === 'string') apply(d.skin);
    });
}());
