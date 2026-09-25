/* ================================================================
   poggy_core — the Poggy theme loader (0.23.0)

   One line in a Poggy script's page, at the end of its <head>:

       <script src="https://cfx-nui-poggy_core/ui/hub/theme.js"></script>

   and that page follows the theme the server owner chose in poggy_core
   (PoggyCoreConfig.Theme; /poggy → Poggy Core → Theme). This file:

     1. puts ui/hub/theme.css (the tokens) and ui/hub/theme-<poggy_id>.css
        (that script's skin) at the end of <head>, after the script's own
        stylesheets, and class "pg-themed" on <html>;
     2. asks the script's own bridge for the theme: a POST to
        https://<this resource>/poggy_core:theme, answered by the bridge
        poggy_core/template/poggy.lua registers in every Poggy script;
     3. sets the base colours on <html>; theme.css derives the rest;
     4. asks when the page loads, and again (0.24.0) when its script sends
        it a message, at most every few seconds: so a theme or a player's
        setting changed while the page was loaded shows the next time the
        screen opens. The stylesheets are attached once and never moved or
        re-attached: re-attaching them is what made pages flash their old
        look in 0.23.0 (crafting, 23 September 2026), not the asking.

   A script on the owner's "keep its own look" list gets nothing: no skin,
   no class, no colours. With no answer at all (an older poggy_core or
   bridge), the page keeps the Rosewood defaults in theme.css.

   poggy_core's own page (ui/index.html) loads this file too; its client
   answers the same callback.

   The player's own settings (0.24.0, /poggyui). The same answer carries
   `scale` (the size the player chose) and `motion` ("less": no animations),
   and the palette is already the player's Light or chosen preset when they
   picked one. See "the screen size" and "less motion" below.
   ================================================================ */

(function () {
    'use strict';

    if (window.__poggyTheme) return;             // loaded twice: once is enough
    window.__poggyTheme = true;

    var me = document.currentScript;
    var BASE = (me && me.src) ? me.src.replace(/[^\/]*$/, '') : 'https://cfx-nui-poggy_core/ui/hub/';
    var RESOURCE = (typeof GetParentResourceName === 'function') ? GetParentResourceName() : null;

    // The theme's fonts. Keys are what PoggyCoreConfig.Theme's Font setting takes.
    var FONTS = {
        fell:  { display: "'IM Fell English', 'Source Serif 4', Georgia, 'Times New Roman', serif",
                 body:    "'Segoe UI', Arial, Helvetica, sans-serif" },
        serif: { display: "'Source Serif 4', Georgia, 'Times New Roman', serif",
                 body:    "'Source Serif 4', Georgia, 'Times New Roman', serif" },
        clean: { display: "'Segoe UI', Arial, Helvetica, sans-serif",
                 body:    "'Segoe UI', Arial, Helvetica, sans-serif" },
    };

    var state = { id: null, rev: null, enabled: null, key: null };

    // ── colour helpers ────────────────────────────────────────────────
    function parseHex(hex) {
        var m = /^#?([0-9a-f]{3}|[0-9a-f]{6})$/i.exec(String(hex || '').trim());
        if (!m) return null;
        var h = m[1];
        if (h.length === 3) h = h[0] + h[0] + h[1] + h[1] + h[2] + h[2];
        return [parseInt(h.slice(0, 2), 16), parseInt(h.slice(2, 4), 16), parseInt(h.slice(4, 6), 16)];
    }
    function clamp(n) { return Math.max(0, Math.min(255, Math.round(n))); }
    function mix(a, b, t) { return [clamp(a[0] + (b[0] - a[0]) * t), clamp(a[1] + (b[1] - a[1]) * t), clamp(a[2] + (b[2] - a[2]) * t)]; }
    function lum(c) {
        var s = c.map(function (v) { v /= 255; return v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4); });
        return 0.2126 * s[0] + 0.7152 * s[1] + 0.0722 * s[2];
    }
    function triple(c) { return c[0] + ', ' + c[1] + ', ' + c[2]; }

    // The base palette (what the server sends) → the --pg-*-rgb tokens.
    function tokens(p) {
        var bg     = parseHex(p.background) || [20, 20, 20];
        var text   = parseHex(p.text)       || [246, 239, 227];
        var accent = parseHex(p.accent)     || [214, 173, 104];
        var dark   = lum(bg) < lum(text);                     // a dark theme: text lighter than the panel
        var away   = dark ? [0, 0, 0] : [255, 255, 255];     // "deeper" than the panel
        var font   = FONTS[p.font] || FONTS.fell;
        var alpha  = Number(p.opacity);
        if (!(alpha >= 0.5 && alpha <= 1)) alpha = 0.94;
        var corners = Math.max(0, Math.min(16, Number(p.corners) || 0));

        var t = {
            'bg-rgb':        triple(bg),
            'deep-rgb':      triple(mix(bg, away, 0.30)),
            'raise-rgb':     triple(mix(bg, text, 0.04)),
            'text-rgb':      triple(text),
            'accent-rgb':    triple(accent),
            'accent-hi-rgb': triple(mix(accent, dark ? [255, 255, 255] : [0, 0, 0], 0.25)),
            'on-accent-rgb': triple(lum(accent) > 0.4 ? mix(bg, [0, 0, 0], 0.3) : [255, 255, 255]),
            'success-rgb':   triple(parseHex(p.success) || [143, 191, 127]),
            'danger-rgb':    triple(parseHex(p.danger)  || [212, 138, 138]),
            'warning-rgb':   triple(parseHex(p.warning) || [224, 179, 90]),
            'info-rgb':      triple(parseHex(p.info)    || [143, 179, 217]),
            'panel-alpha':   String(alpha),
            'font-display':  font.display,
            'font-body':     font.body,
            'radius':        corners + 'px',
            'radius-sm':     Math.round(corners / 2) + 'px',
        };
        return t;
    }

    // ── the page ──────────────────────────────────────────────────────
    function link(id, href) {
        var el = document.getElementById(id);
        if (!href) { if (el) el.parentNode.removeChild(el); return; }
        if (!el) {
            el = document.createElement('link');
            el.id = id;
            el.rel = 'stylesheet';
        }
        if (el.getAttribute('href') !== href) el.setAttribute('href', href);
        // Attached once and never moved: moving a <link> makes the browser
        // drop and reload its sheet, and the page flashes unthemed.
        if (!el.isConnected) document.head.appendChild(el);
    }

    // Once, when the document is complete: if the page's own stylesheets were
    // parsed after ours, put ours back at the end so a skin wins over sheets
    // of the same specificity. Only then, and only if something follows them.
    function toEnd() {
        ['pg-theme-base', 'pg-theme-skin'].forEach(function (id) {
            var el = document.getElementById(id);
            if (!el || !el.isConnected) return;
            var n = el.nextElementSibling, later = false;
            while (n) { if (n.id !== 'pg-theme-base' && n.id !== 'pg-theme-skin' && (n.tagName === 'STYLE' || (n.tagName === 'LINK' && /stylesheet/i.test(n.rel)))) later = true; n = n.nextElementSibling; }
            if (later) document.head.appendChild(el);
        });
    }

    function skinName(id) {
        return String(id || '').replace(/[^A-Za-z0-9_-]/g, '');
    }

    // ── less motion (0.24.0) ───────────────────────────────────────────
    // The player asked for no animations (/poggyui): every animation and
    // transition on the page runs in a blink. Not "none": a page that waits
    // for animationend or transitionend still gets it.
    var CALM_CSS = 'html[data-pg-motion="less"] *, html[data-pg-motion="less"] *::before, ' +
        'html[data-pg-motion="less"] *::after { animation-duration: 0.001ms !important; ' +
        'animation-delay: 0s !important; animation-iteration-count: 1 !important; ' +
        'transition-duration: 0.001ms !important; transition-delay: 0s !important; ' +
        'scroll-behavior: auto !important; }';

    function motion(kind) {
        var root = document.documentElement;
        if (kind === 'less') {
            if (!document.getElementById('pg-motion')) {
                var st = document.createElement('style');
                st.id = 'pg-motion';
                st.textContent = CALM_CSS;
                (document.head || root).appendChild(st);
            }
            root.setAttribute('data-pg-motion', 'less');
        } else {
            root.removeAttribute('data-pg-motion');
        }
    }

    function apply(theme) {
        if (!theme || typeof theme !== 'object') return;
        // Size and motion are the player's, not the look's: a script that
        // keeps its own colours still follows them.
        if (theme.scale) Scale.set(theme.scale);
        motion(theme.motion);
        var root = document.documentElement;
        var id = skinName(theme.id || RESOURCE);

        if (theme.enabled === false) {
            root.classList.remove('pg-themed');
            root.removeAttribute('data-pg-theme');
            root.removeAttribute('data-pg-tone');
            link('pg-theme-base', null);
            link('pg-theme-skin', null);
            if (state.key) {
                Object.keys(tokens({})).forEach(function (k) { root.style.removeProperty('--pg-' + k); });
            }
            state.enabled = false; state.key = null; state.id = id;
            return;
        }

        var key = JSON.stringify(theme.palette || {}) + '|' + id;
        root.classList.add('pg-themed');
        root.setAttribute('data-pg-theme', String(theme.preset || 'rosewood'));
        // light or dark, for the rare rule that must differ (a rarity colour
        // that reads on black but not on parchment): html[data-pg-tone="light"]
        var pal = theme.palette || {};
        var bgc = parseHex(pal.background) || [20, 20, 20], txc = parseHex(pal.text) || [246, 239, 227];
        root.setAttribute('data-pg-tone', lum(bgc) < lum(txc) ? 'dark' : 'light');
        if (key !== state.key) {
            var t = tokens(theme.palette || {});
            Object.keys(t).forEach(function (k) { root.style.setProperty('--pg-' + k, t[k]); });
            state.key = key;
        }
        // skin === false: poggy_core ships no skin for this script (nothing to
        // ask for); undefined (the defaults, before the answer): try it.
        var sid = skinName(theme.skinId) || id;           // poggy_tickets_fivem wears poggy_tickets' skin
        var skin = (sid && theme.skin !== false) ? BASE + 'theme-' + sid + '.css' : null;
        link('pg-theme-base', BASE + 'theme.css');
        link('pg-theme-skin', skin);
        state.enabled = true; state.id = id; state.rev = theme.rev; state.skin = skin;
        try { window.dispatchEvent(new CustomEvent('poggy:theme', { detail: theme })); } catch (e) { /* old engine */ }
    }

    var lastAsk = 0;

    function ask() {
        if (!RESOURCE) return;
        lastAsk = Date.now();
        fetch('https://' + RESOURCE + '/poggy_core:theme', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json; charset=UTF-8' },
            body: '{}',
        }).then(function (r) { return r.json(); })
          .then(function (t) {
              if (!t || typeof t !== 'object') return;
              if (t.palette || t.enabled === false) apply(t);
          })
          .catch(function () { /* no bridge answer: keep what is on the page */ });
    }

    // ── the screen size (0.24.0) ──────────────────────────────────────
    //
    // A player picks how big Poggy screens are (/poggyui; 100% is the size
    // each page was made at). Chromium's own zoom (Ctrl +) cannot be reached
    // from a page, so this does the next best thing: CSS `zoom` on <html>,
    // then makes the page behave as if the browser itself were zoomed, so
    // that no script has to know it is scaled:
    //
    //   - viewport units (vw, vh, vmin, vmax) in the page's stylesheets and
    //     style attributes are divided by the zoom. Chromium zooms them too,
    //     so a panel 88vw wide would otherwise run off the screen;
    //   - media queries on width and height are moved by the zoom, so a page
    //     that rearranges itself on a small screen still does;
    //   - mouse coordinates (clientX, pageX, offsetX, movementX, ...),
    //     innerWidth / innerHeight, the root's clientWidth / clientHeight and
    //     elementFromPoint are given in the page's own zoomed pixels: the ones
    //     getBoundingClientRect, offsetLeft and style.left already use in the
    //     game's Chromium (103). A newer Chromium (128+) reports
    //     getBoundingClientRect in screen pixels instead; that is detected
    //     and turned back into the page's pixels, so both agree;
    //   - devicePixelRatio is multiplied by the zoom, so a canvas that honours
    //     it (the market's charts) is drawn sharp instead of stretched.
    //
    // At 100% nothing is touched: no rewrite, no patch. Once a page has been
    // scaled, the rewritten units and the patched readings hold at any size,
    // 100% included.
    //
    // The size never makes a page's own area smaller than 1280 × 720 (a 1080p
    // screen goes to 150% at most, 1440p to 200%), so a page made for 1080p
    // always fits. "Fit to screen" is min(width / 1920, height / 1080): what
    // makes a page made at 1080p cover the same part of any screen.
    var Scale = (function () {
        var MIN = 0.5, MAX = 3, FLOOR_W = 1280, FLOOR_H = 720;
        var s = 1;                 // the zoom in force
        var last = null;           // the last answer, to work out again on a resize
        var ready = false;         // patches installed
        var standard = false;      // a Chromium whose getBoundingClientRect includes the zoom
        var real = {};             // original getters: the real screen size
        var sheetsDone = typeof WeakSet === 'function' ? new WeakSet() : null;
        var medias = [];           // [{ list: MediaList, text: its original text }]

        function realW() { return real.w ? real.w.call(window) : window.innerWidth; }
        function realH() { return real.h ? real.h.call(window) : window.innerHeight; }

        function fit() { return Math.min(realW() / 1920, realH() / 1080) || 1; }

        function cap() {
            return Math.max(1, Math.min(MAX, realW() / FLOOR_W, realH() / FLOOR_H));
        }

        // The answer: { player: percent or null, default: percent, fit: bool }
        function resolve(a) {
            if (!a || typeof a !== 'object') return 1;
            var want;
            var p = Number(a.player);
            if (p > 0) want = p / 100;
            else {
                var d = Number(a['default']);
                if (!(d > 0)) d = 100;
                want = (a.fit === true ? fit() : 1) * d / 100;
            }
            if (!(want > 0)) want = 1;
            want = Math.max(MIN, Math.min(want, cap()));
            return Math.round(want * 1000) / 1000;
        }

        // ── viewport units and media queries ────────────────────────
        var VP = /(^|[^\w.-])(-?(?:\d*\.)?\d+)(vw|vh|vmin|vmax)\b/g;

        function fixValue(v) {
            if (!v || !/\d(vw|vh|vmin|vmax)\b/.test(v)) return null;
            if (v.indexOf('--pg-zoom') >= 0 || /url\(|["']/.test(v)) return null;
            var out = v.replace(VP, function (m, pre, n, u) {
                return pre + 'calc(' + n + u + ' / var(--pg-zoom, 1))';
            });
            return out === v ? null : out;
        }

        function fixStyle(st) {
            if (!st || !st.length) return;
            var todo = [];
            for (var i = 0; i < st.length; i++) {
                var p = st[i], nv = fixValue(st.getPropertyValue(p));
                if (nv) todo.push([p, nv, st.getPropertyPriority(p)]);
            }
            for (var j = 0; j < todo.length; j++) {
                try { st.setProperty(todo[j][0], todo[j][1], todo[j][2]); } catch (e) { /* leave it */ }
            }
        }

        var MQ = /\b(min|max)-(width|height)\s*:\s*(\d*\.?\d+)px/g;

        function moveMedia(entry) {
            var text = entry.text.replace(MQ, function (m, mm, dim, n) {
                return mm + '-' + dim + ': ' + (Math.round(Number(n) * s * 100) / 100) + 'px';
            });
            try { if (entry.list.mediaText !== text) entry.list.mediaText = text; } catch (e) { /* leave it */ }
        }

        function fixRules(rules) {
            if (!rules) return;
            for (var i = 0; i < rules.length; i++) {
                var r = rules[i];
                if (r.style) fixStyle(r.style);
                if (r.media && r.media.mediaText) {
                    MQ.lastIndex = 0;
                    if (MQ.test(r.media.mediaText)) {
                        var entry = { list: r.media, text: r.media.mediaText };
                        medias.push(entry);
                        moveMedia(entry);
                    }
                    MQ.lastIndex = 0;
                }
                if (r.styleSheet) fixSheet(r.styleSheet);
                if (r.cssRules) fixRules(r.cssRules);
            }
        }

        function fixSheet(sheet) {
            if (!sheet) return;
            if (sheetsDone && sheetsDone.has(sheet)) return;
            var rules;
            try { rules = sheet.cssRules; } catch (e) { return; }   // another origin: poggy_core's own sheets, a web font
            if (!rules) return;
            if (sheetsDone) sheetsDone.add(sheet);
            fixRules(rules);
        }

        function fixNode(n) {
            if (!n || n.nodeType !== 1) return;
            if (n.tagName === 'STYLE') fixSheet(n.sheet);
            else if (n.tagName === 'LINK') {
                if (n.sheet) fixSheet(n.sheet);
                else n.addEventListener('load', function () { fixSheet(n.sheet); });
            }
            if (n.hasAttribute && n.hasAttribute('style')) fixStyle(n.style);
            if (n.firstElementChild && n.querySelectorAll) {
                var list = n.querySelectorAll('[style], style, link[rel~="stylesheet"]');
                for (var i = 0; i < list.length; i++) fixNode(list[i]);
            }
        }

        function rewriteAll() {
            for (var i = 0; i < document.styleSheets.length; i++) fixSheet(document.styleSheets[i]);
            fixNode(document.documentElement);
            if (typeof MutationObserver !== 'function') return;
            // What the page adds or restyles later: a <style> its JS writes,
            // a style="height: 40vh" it sets.
            new MutationObserver(function (records) {
                for (var i = 0; i < records.length; i++) {
                    var m = records[i];
                    if (m.type === 'attributes') { if (m.target.style) fixStyle(m.target.style); continue; }
                    if (m.target && m.target.tagName === 'STYLE') { fixSheet(m.target.sheet); continue; }
                    for (var j = 0; j < m.addedNodes.length; j++) fixNode(m.addedNodes[j]);
                }
            }).observe(document.documentElement, {
                subtree: true, childList: true, attributes: true, attributeFilter: ['style'],
            });
        }

        // ── readings, in the page's own pixels ──────────────────────
        function getter(obj, name, fn) {
            var d = Object.getOwnPropertyDescriptor(obj, name);
            if (!d || !d.get || !d.configurable) return null;
            Object.defineProperty(obj, name, {
                configurable: true, enumerable: d.enumerable,
                get: function () { return fn(d.get.call(this), this); },
                set: d.set,
            });
            return d.get;
        }

        function down(v) { return (s === 1 || typeof v !== 'number') ? v : v / s; }

        function scaledRect(r) {
            if (s === 1 || !r) return r;
            return new DOMRect(r.x / s, r.y / s, r.width / s, r.height / s);
        }

        // Does getBoundingClientRect include the zoom (standardized zoom,
        // Chromium 128+), or leave it out (the game's Chromium 103)?
        function probe() {
            var host = document.body || document.documentElement;
            var d = document.createElement('div');
            d.style.cssText = 'position:fixed;left:0;top:0;width:100px;height:1px;zoom:2;visibility:hidden;pointer-events:none';
            host.appendChild(d);
            var w = d.getBoundingClientRect().width;
            host.removeChild(d);
            return w > 150;
        }

        function install() {
            if (ready) return;
            ready = true;
            try { standard = probe(); } catch (e) { standard = false; }

            var ME = window.MouseEvent && MouseEvent.prototype;
            if (ME) {
                ['clientX', 'clientY', 'pageX', 'pageY', 'x', 'y', 'offsetX', 'offsetY',
                 'layerX', 'layerY', 'movementX', 'movementY'].forEach(function (n) {
                    getter(ME, n, down);
                });
            }
            real.w = getter(window, 'innerWidth', down);
            real.h = getter(window, 'innerHeight', down);
            getter(window, 'devicePixelRatio', function (v) { return v * s; });

            // The root answers with the screen's size: give it in the page's pixels.
            ['clientWidth', 'clientHeight'].forEach(function (n) {
                getter(Element.prototype, n, function (v, el) { return el === document.documentElement ? down(v) : v; });
            });

            // A point in the page's pixels, to the screen's.
            ['elementFromPoint', 'elementsFromPoint', 'caretRangeFromPoint'].forEach(function (n) {
                var f = Document.prototype[n];
                if (typeof f !== 'function') return;
                Document.prototype[n] = function (x, y) {
                    var args = Array.prototype.slice.call(arguments);
                    if (s !== 1) { args[0] = x * s; args[1] = y * s; }
                    return f.apply(this, args);
                };
            });

            if (standard) {
                [Element.prototype, window.Range && Range.prototype].forEach(function (P) {
                    if (!P) return;
                    var one = P.getBoundingClientRect, many = P.getClientRects;
                    if (one) P.getBoundingClientRect = function () { return scaledRect(one.call(this)); };
                    if (many) P.getClientRects = function () {
                        var list = many.call(this);
                        if (s === 1) return list;
                        var out = [];
                        for (var i = 0; i < list.length; i++) out.push(scaledRect(list[i]));
                        out.item = function (k) { return out[k] || null; };
                        return out;
                    };
                });
            }

            rewriteAll();
        }

        function zoomTo(next) {
            if (next === s) return;
            if (next !== 1) install();
            s = next;
            var root = document.documentElement;
            if (s === 1) {
                root.style.removeProperty('zoom');
                root.style.removeProperty('--pg-zoom');
                root.removeAttribute('data-pg-scale');
            } else {
                root.style.setProperty('zoom', String(s));
                root.style.setProperty('--pg-zoom', String(s));
                root.setAttribute('data-pg-scale', String(Math.round(s * 100)));
            }
            for (var i = 0; i < medias.length; i++) moveMedia(medias[i]);
            // Pages that place things from JS do it again on a resize.
            try { window.dispatchEvent(new Event('resize')); } catch (e) { /* old engine */ }
            try { window.dispatchEvent(new CustomEvent('poggy:scale', { detail: s })); } catch (e) { /* old engine */ }
        }

        function set(answer) {
            if (!answer || typeof answer !== 'object') return;
            last = answer;
            zoomTo(resolve(answer));
        }

        // The game changed resolution: the cap, and a fitted size, move with it.
        window.addEventListener('resize', function (e) {
            if (last && e.isTrusted) zoomTo(resolve(last));
        });

        return {
            set: set,
            // Shows a size without keeping it (the /poggyui preview), in percent;
            // null goes back to the last answer.
            preview: function (percent) {
                if (percent == null) zoomTo(resolve(last));
                else zoomTo(resolve({ player: percent }));
            },
            current: function () { return s; },
            fitPercent: function () { return Math.round(fit() * 100); },
            maxPercent: function () { return Math.floor(cap() * 100); },
            resolve: function (a) { return resolve(a); },
            answer: function () { return last; },
        };
    }());

    function start() {
        // Until the answer comes, the defaults: Rosewood, this script's skin.
        apply({ id: RESOURCE, enabled: true, preset: 'rosewood', palette: {} });
        ask();
    }

    // Only poggy_core's own page is ever sent the theme (its menus, /poggy
    // and /poggyui follow a change at once). A script's page never gets this
    // message: it asks again, at most every few seconds, when its script
    // sends it a message (opening a screen, most often), so a new theme, size
    // or motion setting shows on each screen the next time it opens. The
    // same answer applied again changes nothing on the page.
    var IS_CORE = RESOURCE === 'poggy_core';
    window.addEventListener('message', function (e) {
        var d = e && e.data;
        if (d && typeof d === 'object' && d.poggyTheme) { apply(d.poggyTheme); return; }
        if (!IS_CORE && Date.now() - lastAsk > 2500) ask();
    });

    // Exposed for poggy_core's own page and for the preview tool.
    window.PoggyTheme = { apply: apply, ask: ask, tokens: tokens, scale: Scale, motion: motion };

    if (document.head) start();
    else document.addEventListener('DOMContentLoaded', start);
    document.addEventListener('DOMContentLoaded', toEnd);
}());
