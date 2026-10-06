/*
  poggy_core — other scripts' screens, shown in this page (0.27.0)

  client/cl_host.lua sends { poggyHost: { op, app, url, idle, data } }:

    send    open the app's frame if it is not open, then hand it `data`
            exactly as SendNUIMessage would have (queued until it has loaded)
    ensure  open the app's frame if it is not open
    focus   `app` is the screen in front (nil: poggy_core's own screen is)
    unload  remove the app's frame

  Each app gets a frame of its own, loaded from the script's own folder
  (https://cfx-nui-<script>/<page>), set up the way RedM sets up a script's
  frame (GetParentResourceName answers the script's name), so the page runs
  as if it were the script's own ui_page. Nothing is loaded until the script
  first uses its screen. An app whose manifest sets poggy_ui_idle is removed
  again after that many seconds without focus or messages.
*/
(function () {
    'use strict';

    var apps = {};            // name -> { frame, loaded, queue, last, idle }
    var front = null;         // the app that has focus, or null
    var layer = null;

    function getLayer() {
        if (layer) return layer;
        layer = document.createElement('div');
        layer.id = 'pg-host';
        layer.style.cssText = 'position:fixed;inset:0;pointer-events:none;z-index:0;';
        // First in the body: poggy_core's own panels paint over hosted screens.
        document.body.insertBefore(layer, document.body.firstChild);
        return layer;
    }

    function named(frame, name) {
        try {
            frame.contentWindow.GetParentResourceName = function () { return name; };
        } catch (e) { /* the page will still load; its calls may go astray */ }
    }

    function open(name, url, idle) {
        var app = apps[name];
        if (app) {
            app.idle = idle || 0;
            return app;
        }
        var frame = document.createElement('iframe');
        frame.name = name;
        frame.src = url;
        frame.tabIndex = -1;
        frame.setAttribute('allowtransparency', 'true');
        frame.allow = 'microphone *; camera *;';
        frame.style.cssText = 'position:absolute;inset:0;width:100%;height:100%;border:0;' +
            'background:transparent;pointer-events:none;visibility:hidden;';

        app = { frame: frame, loaded: false, queue: [], last: Date.now(), idle: idle || 0 };
        apps[name] = app;

        getLayer().appendChild(frame);
        // As RedM's own root page does: set right after the frame is added, so
        // the page's first script already gets the script's name.
        named(frame, name);

        frame.addEventListener('load', function () {
            if (apps[name] !== app) return;
            named(frame, name);
            try {
                var said = frame.contentWindow.GetParentResourceName();
                if (said !== name) console.warn('[poggy_core] hosted screen ' + name + ' reports its name as ' + said);
            } catch (e) { /* cross-origin read refused: nothing to check */ }
            frame.style.visibility = 'visible';
            app.loaded = true;
            var q = app.queue;
            app.queue = [];
            for (var i = 0; i < q.length; i++) deliver(app, q[i]);
            if (front === name) focusFrame(app);
        });
        return app;
    }

    function deliver(app, data) {
        try { app.frame.contentWindow.postMessage(data, '*'); } catch (e) { console.error(e); }
    }

    function focusFrame(app) {
        try { app.frame.contentWindow.focus(); } catch (e) { /* not loaded yet */ }
    }

    function unload(name) {
        var app = apps[name];
        if (!app) return;
        delete apps[name];
        if (front === name) front = null;
        if (app.frame.parentNode) app.frame.parentNode.removeChild(app.frame);
    }

    function setFront(name, cursor) {
        front = name || null;
        for (var n in apps) {
            if (!Object.prototype.hasOwnProperty.call(apps, n)) continue;
            var on = n === front;
            apps[n].frame.style.pointerEvents = on && cursor ? 'auto' : 'none';
            if (on) {
                apps[n].last = Date.now();
                if (apps[n].loaded) focusFrame(apps[n]);
            }
        }
        if (!front && document.activeElement && document.activeElement.tagName === 'IFRAME') {
            document.activeElement.blur();
        }
    }

    window.addEventListener('message', function (event) {
        var h = event.data && event.data.poggyHost;
        if (!h) return;
        switch (h.op) {
            case 'send': {
                var app = open(h.app, h.url, h.idle);
                app.last = Date.now();
                if (app.loaded) deliver(app, h.data);
                else app.queue.push(h.data);
                break;
            }
            case 'ensure':
                open(h.app, h.url, h.idle).last = Date.now();
                break;
            case 'focus':
                setFront(h.app, h.cursor);
                break;
            case 'unload':
                unload(h.app);
                break;
            default:
                break;
        }
    });

    // Screens that asked to be unloaded when unused.
    setInterval(function () {
        var now = Date.now();
        for (var n in apps) {
            if (!Object.prototype.hasOwnProperty.call(apps, n)) continue;
            var app = apps[n];
            if (app.idle > 0 && n !== front && now - app.last > app.idle * 1000) unload(n);
        }
    }, 5000);
})();
