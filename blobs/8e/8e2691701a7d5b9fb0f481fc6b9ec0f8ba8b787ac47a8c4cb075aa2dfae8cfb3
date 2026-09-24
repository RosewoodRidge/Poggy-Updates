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
     4. asks ONCE, when the page loads (0.23.1). A theme changed in /poggy
        reaches a script's page when that script restarts or the player
        reconnects. Re-asking while the page was open, and re-attaching the
        stylesheets on each answer, made the browser reload them: the page
        flashed its old look (crafting, 23 September 2026).

   A script on the owner's "keep its own look" list gets nothing: no skin,
   no class, no colours. With no answer at all (an older poggy_core or
   bridge), the page keeps the Rosewood defaults in theme.css.

   poggy_core's own page (ui/index.html) loads this file too; its client
   answers the same callback.
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

    function apply(theme) {
        if (!theme || typeof theme !== 'object') return;
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

    function ask() {
        if (!RESOURCE) return;
        fetch('https://' + RESOURCE + '/poggy_core:theme', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json; charset=UTF-8' },
            body: '{}',
        }).then(function (r) { return r.json(); })
          .then(function (t) { if (t && (t.palette || t.enabled === false)) apply(t); })
          .catch(function () { /* no bridge answer: keep what is on the page */ });
    }

    function start() {
        // Until the answer comes, the defaults: Rosewood, this script's skin.
        apply({ id: RESOURCE, enabled: true, preset: 'rosewood', palette: {} });
        ask();
    }

    // Only poggy_core's own page is ever sent the theme (its menus and /poggy
    // follow a change at once). A script's page never gets this message.
    window.addEventListener('message', function (e) {
        var d = e && e.data;
        if (d && typeof d === 'object' && d.poggyTheme) apply(d.poggyTheme);
    });

    // Exposed for poggy_core's own page and for the preview tool.
    window.PoggyTheme = { apply: apply, ask: ask, tokens: tokens };

    if (document.head) start();
    else document.addEventListener('DOMContentLoaded', start);
    document.addEventListener('DOMContentLoaded', toEnd);
}());
