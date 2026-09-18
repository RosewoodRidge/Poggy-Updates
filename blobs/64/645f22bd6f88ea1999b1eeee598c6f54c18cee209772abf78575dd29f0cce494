/*
    Poggy Hub — core: DOM helpers, icons, value and path helpers, the NUI
    bridge, and the shared overlays (toasts, modals, popovers, tooltip).

    Loaded first. Every hub module hangs off window.PoggyHub (PH); there is no
    build step and no framework. Contract: docs/reference/poggy-hub-spec.md.

    Browser support: RedM's CEF is an older Chromium, so nothing newer than
    ES2019 in script (no ?. or ??) and no :has(), nesting or container queries
    in CSS.
*/
(function () {
    'use strict';

    var PH = window.PoggyHub = window.PoggyHub || {};

    // Inside the game the page belongs to poggy_core; in a normal browser
    // (dev/mock.html) there is no GetParentResourceName, and the mock answers
    // the same URLs.
    var RESOURCE = 'poggy_core';
    try { if (typeof GetParentResourceName === 'function') RESOURCE = GetParentResourceName(); } catch (e) { /* browser */ }
    PH.RESOURCE = RESOURCE;

    // ------------------------------------------------------------------ DOM --

    /**
     * h('div.ph-card.is-on', { onclick: fn, title: 'x', dataset: { id: 1 } }, [children...])
     * Children may be nodes, strings (as text, never HTML), arrays or null.
     */
    function h(sel, attrs, children) {
        var parts = String(sel).split('.');
        var el = document.createElement(parts[0] || 'div');
        if (parts.length > 1) el.className = parts.slice(1).join(' ');
        if (attrs && (Array.isArray(attrs) || typeof attrs === 'string' || attrs instanceof Node)) {
            children = attrs;
            attrs = null;
        }
        if (attrs) {
            Object.keys(attrs).forEach(function (k) {
                var v = attrs[k];
                if (v === undefined || v === null || v === false) return;
                if (k === 'class') el.className += (el.className ? ' ' : '') + v;
                else if (k === 'text') el.textContent = v;
                else if (k === 'html') el.innerHTML = v;          // only ever given markup the hub built
                else if (k === 'style' && typeof v === 'object') Object.keys(v).forEach(function (s) { el.style[s] = v[s]; });
                else if (k === 'dataset') Object.keys(v).forEach(function (d) { el.dataset[d] = v[d]; });
                else if (k.slice(0, 2) === 'on' && typeof v === 'function') el.addEventListener(k.slice(2), v);
                else if (k === 'value') el.value = v;
                else if (k === 'checked' || k === 'disabled' || k === 'readOnly' || k === 'hidden') el[k] = !!v;
                else if (v === true) el.setAttribute(k, '');
                else el.setAttribute(k, v);
            });
        }
        append(el, children);
        return el;
    }

    function append(el, children) {
        if (children === undefined || children === null || children === false) return el;
        if (Array.isArray(children)) { children.forEach(function (c) { append(el, c); }); return el; }
        if (children instanceof Node) { el.appendChild(children); return el; }
        el.appendChild(document.createTextNode(String(children)));
        return el;
    }

    function clear(el) { while (el && el.firstChild) el.removeChild(el.firstChild); return el; }

    function $(sel, root) { return (root || document).querySelector(sel); }
    function $$(sel, root) { return Array.prototype.slice.call((root || document).querySelectorAll(sel)); }

    PH.h = h; PH.append = append; PH.clear = clear; PH.$ = $; PH.$$ = $$;

    // ---------------------------------------------------------------- icons --
    // Drawn for the hub on a 24px grid, stroked in currentColor.

    var ICONS = {
        search:   '<circle cx="11" cy="11" r="6.5"/><path d="M20 20l-4.2-4.2"/>',
        x:        '<path d="M6 6l12 12M18 6L6 18"/>',
        back:     '<path d="M19 12H5M11 18l-6-6 6-6"/>',
        restart:  '<path d="M20 12a8 8 0 1 1-2.6-5.9"/><path d="M20 4v5h-5"/>',
        play:     '<path d="M8 5.5v13l10.5-6.5z" fill="currentColor" stroke="none"/>',
        stop:     '<rect x="6.5" y="6.5" width="11" height="11" rx="1.5" fill="currentColor" stroke="none"/>',
        lock:     '<rect x="5" y="11" width="14" height="9.5" rx="2"/><path d="M8 11V8a4 4 0 0 1 8 0v3"/>',
        unlock:   '<rect x="5" y="11" width="14" height="9.5" rx="2"/><path d="M8 11V8a4 4 0 0 1 7.6-1.7"/>',
        info:     '<circle cx="12" cy="12" r="9"/><path d="M12 11v5.5"/><path d="M12 7.6v.2" stroke-width="2.4"/>',
        reset:    '<path d="M9 13.5L4.5 9 9 4.5"/><path d="M4.5 9H14a5.5 5.5 0 0 1 0 11h-3.5"/>',
        save:     '<path d="M5 4h11l3 3v13H5z"/><path d="M8.5 4v4.5h6V4M8 20v-6h8v6"/>',
        plus:     '<path d="M12 5v14M5 12h14"/>',
        minus:    '<path d="M5 12h14"/>',
        copy:     '<rect x="9" y="9" width="11" height="11" rx="2"/><path d="M5 15V6a2 2 0 0 1 2-2h8"/>',
        trash:    '<path d="M4 7h16M10 11v6M14 11v6M6.5 7l1 13h9l1-13M9 7V4h6v3"/>',
        up:       '<path d="M12 19V5M6 11l6-6 6 6"/>',
        down:     '<path d="M12 5v14M6 13l6 6 6-6"/>',
        grip:     '<circle cx="9" cy="6" r="1.3" fill="currentColor"/><circle cx="15" cy="6" r="1.3" fill="currentColor"/><circle cx="9" cy="12" r="1.3" fill="currentColor"/><circle cx="15" cy="12" r="1.3" fill="currentColor"/><circle cx="9" cy="18" r="1.3" fill="currentColor"/><circle cx="15" cy="18" r="1.3" fill="currentColor"/>',
        right:    '<path d="M9 6l6 6-6 6"/>',
        left:     '<path d="M15 6l-6 6 6 6"/>',
        chevdown: '<path d="M6 9l6 6 6-6"/>',
        check:    '<path d="M5 12.5l4.5 4.5L19 7"/>',
        alert:    '<path d="M12 3.5l9.5 17h-19z"/><path d="M12 10v5"/><path d="M12 17.6v.2" stroke-width="2.4"/>',
        target:   '<circle cx="12" cy="12" r="7"/><path d="M12 2.5v4M12 17.5v4M2.5 12h4M17.5 12h4"/><circle cx="12" cy="12" r="1.4" fill="currentColor"/>',
        user:     '<circle cx="12" cy="8" r="4"/><path d="M4.5 20.5a7.5 7.5 0 0 1 15 0"/>',
        users:    '<circle cx="9" cy="8.5" r="3.5"/><path d="M2.5 20a6.5 6.5 0 0 1 13 0"/><path d="M15.5 5.2a3.5 3.5 0 0 1 0 6.6M17.5 14.2a6.5 6.5 0 0 1 4 5.8"/>',
        shield:   '<path d="M12 3l8 3v6c0 5-3.4 8-8 9.2C7.4 20 4 17 4 12V6z"/>',
        sliders:  '<path d="M4 6h9M17 6h3M4 12h3M11 12h9M4 18h11M19 18h1"/><circle cx="15" cy="6" r="2"/><circle cx="9" cy="12" r="2"/><circle cx="17" cy="18" r="2"/>',
        list:     '<path d="M9 6h11M9 12h11M9 18h11"/><circle cx="4.5" cy="6" r="1.2" fill="currentColor"/><circle cx="4.5" cy="12" r="1.2" fill="currentColor"/><circle cx="4.5" cy="18" r="1.2" fill="currentColor"/>',
        terminal: '<rect x="3" y="4.5" width="18" height="15" rx="2"/><path d="M7 9.5l3 2.5-3 2.5M12.5 15H17"/>',
        book:     '<path d="M3 5.5h6a3 3 0 0 1 3 3V20a2.5 2.5 0 0 0-2.5-2.5H3z"/><path d="M21 5.5h-6a3 3 0 0 0-3 3V20a2.5 2.5 0 0 1 2.5-2.5H21z"/>',
        clock:    '<circle cx="12" cy="12" r="9"/><path d="M12 7v5l3.2 2"/>',
        history:  '<path d="M3.5 12a8.5 8.5 0 1 0 2.6-6.1L3.5 8.5"/><path d="M3.5 3.5v5h5M12 7.5V12l3 2"/>',
        map:      '<path d="M9 4.5L3.5 6.5v13l5.5-2 6 2 5.5-2v-13l-5.5 2z"/><path d="M9 4.5v13M15 6.5v13"/>',
        box:      '<path d="M3.5 7.5L12 3.5l8.5 4v9L12 20.5l-8.5-4z"/><path d="M3.5 7.5L12 11.5l8.5-4M12 11.5v9"/>',
        coins:    '<ellipse cx="9" cy="7" rx="5.5" ry="2.5"/><path d="M3.5 7v4.5c0 1.4 2.5 2.5 5.5 2.5s5.5-1.1 5.5-2.5V7"/><path d="M9.5 14.2v2.3c0 1.4 2.5 2.5 5.5 2.5s5.5-1.1 5.5-2.5V12c0-1.3-2-2.3-4.8-2.5"/>',
        star:     '<path d="M12 3.5l2.6 5.4 5.9.8-4.3 4.1 1 5.8L12 16.8l-5.2 2.8 1-5.8-4.3-4.1 5.9-.8z"/>',
        tag:      '<path d="M3.5 12.5V4.5h8l9 9-8 8z"/><circle cx="8" cy="9" r="1.4" fill="currentColor"/>',
        globe:    '<circle cx="12" cy="12" r="9"/><path d="M3 12h18M12 3c2.6 2.5 3.8 5.5 3.8 9s-1.2 6.5-3.8 9c-2.6-2.5-3.8-5.5-3.8-9S9.4 5.5 12 3z"/>',
        filter:   '<path d="M4 5h16l-6.2 7.7V19l-3.6-1.6v-4.7z"/>',
        sort:     '<path d="M7 4v16M3.8 16.8L7 20l3.2-3.2M17 20V4M13.8 7.2L17 4l3.2 3.2"/>',
        dots:     '<circle cx="5.5" cy="12" r="1.6" fill="currentColor" stroke="none"/><circle cx="12" cy="12" r="1.6" fill="currentColor" stroke="none"/><circle cx="18.5" cy="12" r="1.6" fill="currentColor" stroke="none"/>',
        pencil:   '<path d="M4 20h4.2L19.5 8.7l-4.2-4.2L4 15.8z"/><path d="M13.5 6.3l4.2 4.2"/>',
        link:     '<path d="M10 14a4 4 0 0 0 5.7 0l3-3a4 4 0 0 0-5.7-5.7l-1.1 1.1"/><path d="M14 10a4 4 0 0 0-5.7 0l-3 3a4 4 0 0 0 5.7 5.7l1.1-1.1"/>',
        unlink:   '<path d="M15.5 13.2l3.2-3.2a4 4 0 0 0-5.7-5.7l-1.1 1.1M8.5 10.8L5.3 14a4 4 0 0 0 5.7 5.7l1.1-1.1"/><path d="M4 4l16 16"/>',
        eye:      '<path d="M2.5 12S6 5.5 12 5.5 21.5 12 21.5 12 18 18.5 12 18.5 2.5 12 2.5 12z"/><circle cx="12" cy="12" r="3"/>',
        file:     '<path d="M6 3h8l4.5 4.5V21H6z"/><path d="M14 3v4.5h4.5"/>',
        hash:     '<path d="M5 9h14M5 15h14M10 4L8 20M16 4l-2 16"/>',
        bag:      '<path d="M5 8h14l-1.2 12.5H6.2z"/><path d="M9 8V6.5a3 3 0 0 1 6 0V8"/>',
        badge:    '<rect x="3" y="7" width="18" height="13" rx="2"/><path d="M9 7V5h6v2M3 12.5h18"/>',
        compass:  '<circle cx="12" cy="12" r="9"/><path d="M15.6 8.4l-2.2 5-5 2.2 2.2-5z"/>',
        palette:  '<path d="M12 3.5a8.5 8.5 0 0 0 0 17c1.2 0 1.8-.8 1.8-1.7 0-1.4-1.2-1.6-1.2-2.8 0-1 .8-1.6 1.8-1.6h2.4a3.7 3.7 0 0 0 3.7-3.7C20.5 6.8 16.7 3.5 12 3.5z"/><circle cx="7.8" cy="11" r="1.2" fill="currentColor"/><circle cx="10.5" cy="7.3" r="1.2" fill="currentColor"/><circle cx="15" cy="7.8" r="1.2" fill="currentColor"/>',
        keyboard: '<rect x="2.5" y="6" width="19" height="12" rx="2"/><path d="M6 10h.5M9.5 10h.5M13 10h.5M16.5 10h1M7 14h10"/>',
        bolt:     '<path d="M13 3L5 13.5h6L10 21l8-10.5h-6z"/>',
        home:     '<path d="M4 11l8-7 8 7"/><path d="M6 9.5V20h12V9.5"/>',
        cog:      '<circle cx="12" cy="12" r="3"/><path d="M12 2.8v2.4M12 18.8v2.4M4.8 4.8l1.7 1.7M17.5 17.5l1.7 1.7M2.8 12h2.4M18.8 12h2.4M4.8 19.2l1.7-1.7M17.5 6.5l1.7-1.7"/>',
        text:     '<path d="M5 6.5V5h14v1.5M12 5v14M9 19h6"/>',
        hat:      '<path d="M3 16.5c3 1.5 15 1.5 18 0"/><path d="M6.5 16.2L8 8.5c.3-1.4 1.7-2 3-1.3l1 .5 1-.5c1.3-.7 2.7-.1 3 1.3l1.5 7.7"/>',
        fish:     '<path d="M3 12c3-4.5 8-6 12.5-3.5L20 5.5v13l-4.5-3C11 18 6 16.5 3 12z"/><circle cx="8" cy="11" r="1" fill="currentColor"/>',
        anvil:    '<path d="M3 7h14c0 2.5 2 3.5 4 3.5v1.5H13l-1 3h3v3H7v-3h3l-1-3c-3.5 0-6-2-6-5z"/>',
        balloon:  '<path d="M12 3a6 6 0 0 1 6 6.5c0 3.8-3.2 6.5-6 6.5s-6-2.7-6-6.5A6 6 0 0 1 12 3z"/><path d="M10 16l1 2.5h2L14 16M11 18.5v2.5h2v-2.5"/>',
        chat:     '<path d="M4 5h16v11H9l-5 4z"/>',
        bell:     '<path d="M6 16.5V11a6 6 0 0 1 12 0v5.5l1.5 2H4.5z"/><path d="M10 20.5a2 2 0 0 0 4 0"/>',
        clipboard:'<rect x="5" y="4.5" width="14" height="16.5" rx="2"/><path d="M9 4.5V3h6v1.5M8.5 10h7M8.5 14h7M8.5 18h4"/>',
        move:     '<path d="M12 3v18M3 12h18M12 3l-3 3M12 3l3 3M12 21l-3-3M12 21l3-3M3 12l3-3M3 12l3 3M21 12l-3-3M21 12l-3 3"/>',
        wrench:   '<path d="M14.5 6.5a4 4 0 0 0 5 5l-9 9a2.1 2.1 0 0 1-3-3l9-9a4 4 0 0 0-2-2z"/><path d="M14.5 6.5l3-3a4 4 0 0 1 2.5 5.5l-3 .5z"/>',
        gavel:    '<path d="M13.5 4.5l6 6M11 7l6 6M8.5 9.5l5-5 6 6-5 5z"/><path d="M11 12l-7.5 7.5M3 21h9"/>',
        truck:    '<path d="M2.5 6.5h11v9h-11zM13.5 9.5h4l3 3.5v2.5h-7z"/><circle cx="6.5" cy="17.5" r="1.8"/><circle cx="17" cy="17.5" r="1.8"/>',
        bug:      '<rect x="7" y="7.5" width="10" height="12" rx="5"/><path d="M9.5 7.5a2.5 2.5 0 0 1 5 0M3.5 12h3.5M17 12h3.5M4.5 18l3-1.5M19.5 18l-3-1.5M4.5 6.5l3 2M19.5 6.5l-3 2M12 11v8"/>',
        navigation:'<path d="M12 2.5l7 18-7-4-7 4z"/>',
        wind:     '<path d="M3 9h11a3 3 0 1 0-3-3M3 13h15a3 3 0 1 1-3 3M3 17h7"/>',
        cpu:      '<rect x="6" y="6" width="12" height="12" rx="1.5"/><rect x="9.5" y="9.5" width="5" height="5"/><path d="M9 2.5V6M15 2.5V6M9 18v3.5M15 18v3.5M2.5 9H6M2.5 15H6M18 9h3.5M18 15h3.5"/>',
        download: '<path d="M12 3.5v12M7 10.5l5 5 5-5M4 20.5h16"/>',
        activity: '<path d="M2.5 12h4l3-7.5 5 15 3-7.5h4"/>',
        chart:    '<path d="M3.5 3.5v17h17"/><path d="M7.5 15l4-4.5 3 3 5.5-6"/>',
        volume:   '<path d="M4 9.5h4l5-4v13l-5-4H4z"/><path d="M16.5 9a4 4 0 0 1 0 6M19 6.5a7.5 7.5 0 0 1 0 11"/>',
        swords:   '<path d="M14.5 17.5L3.5 6.5V3.5h3l11 11M13 19l6-6M16 16l4 4M19 21l2-2M9.5 17.5l11-11V3.5h-3l-11 11M11 19l-6-6M8 16l-4 4M5 21l-2-2"/>',
        paw:      '<circle cx="7" cy="9" r="2"/><circle cx="12" cy="6.5" r="2"/><circle cx="17" cy="9" r="2"/><path d="M8 17c0-3 1.8-5.5 4-5.5s4 2.5 4 5.5c0 1.6-1.4 2.5-4 2.5s-4-.9-4-2.5z"/>',
        music:    '<path d="M9 18V5.5l11-2V16"/><circle cx="6.5" cy="18" r="2.5"/><circle cx="17.5" cy="16" r="2.5"/>',
        gift:     '<rect x="3.5" y="9" width="17" height="11.5" rx="1"/><path d="M3 9h18M12 9v11.5M12 9c-1.5-3.5-5.5-4-5.5-1.5S12 9 12 9zm0 0c1.5-3.5 5.5-4 5.5-1.5S12 9 12 9z"/>',
        // Kinds of list in the rail: a plain list, a keyed table, a collection
        // (rows that hold lists of their own) and a key table.
        rows:     '<rect x="3.5" y="4.5" width="17" height="15" rx="1.5"/><path d="M3.5 9.5h17M3.5 14.5h17M9 4.5v15"/>',
        grid:     '<rect x="4" y="4" width="7" height="7" rx="1"/><rect x="13" y="4" width="7" height="7" rx="1"/><rect x="4" y="13" width="7" height="7" rx="1"/><rect x="13" y="13" width="7" height="7" rx="1"/>',
        open:     '<path d="M13.5 4.5h6v6M19.5 4.5L11 13"/><path d="M17.5 14v5.5h-13v-13H10"/>',
        folder:   '<path d="M3.5 6.5a1.5 1.5 0 0 1 1.5-1.5h4.2l2 2.2H19a1.5 1.5 0 0 1 1.5 1.5v9.3A1.5 1.5 0 0 1 19 19.5H5a1.5 1.5 0 0 1-1.5-1.5z"/>',
    };
    // hub.json tab icons use their own names; map them onto the set above.
    var ICON_ALIAS = {
        settings: 'sliders', general: 'sliders', config: 'sliders', gear: 'cog',
        'map-pin': 'target', pin: 'target', location: 'target', locations: 'map', coords: 'target',
        money: 'coins', economy: 'coins', dollar: 'coins', cash: 'coins', bank: 'coins',
        items: 'bag', item: 'bag', inventory: 'bag', shop: 'bag', store: 'bag', cart: 'bag', package: 'box', crate: 'box',
        jobs: 'badge', job: 'badge', briefcase: 'badge', people: 'users', group: 'users', groups: 'users', team: 'users',
        permissions: 'shield', access: 'shield', security: 'shield', admin: 'shield', lock: 'lock',
        time: 'clock', timing: 'clock', timer: 'clock', schedule: 'clock', text: 'text', translations: 'globe',
        language: 'globe', lang: 'globe', locale: 'globe', messages: 'chat', notifications: 'chat', discord: 'chat',
        webhook: 'link', webhooks: 'link', commands: 'terminal', help: 'book', docs: 'book', history: 'history',
        list: 'list', lists: 'list', debug: 'bolt', advanced: 'cog', ui: 'palette', look: 'palette', appearance: 'palette',
        colour: 'palette', color: 'palette', rewards: 'gift', loot: 'gift', crafting: 'anvil', recipes: 'anvil',
        fishing: 'fish', world: 'globe', events: 'bolt', key: 'keyboard', keys: 'keyboard', controls: 'keyboard',
        message: 'chat', type: 'text', parachute: 'balloon', zap: 'bolt', package: 'box', store: 'bag', tags: 'tag',
        notification: 'bell', alerts: 'bell', alert: 'bell', sound: 'volume', audio: 'volume', stats: 'chart',
        tools: 'wrench', tool: 'wrench', law: 'gavel', court: 'gavel', transport: 'truck', delivery: 'truck',
        animals: 'paw', animal: 'paw', combat: 'swords', weapons: 'swords', updates: 'download', performance: 'cpu',
    };

    function icon(name, cls) {
        // An icon name the hub does not draw (a new hub.json tab icon) gets a
        // neutral one rather than nothing; 'dot' is the only deliberate dot.
        var key = ICONS[name] ? name : (ICON_ALIAS[String(name || '').toLowerCase()] || (name === 'dot' ? 'dot' : 'sliders'));
        var body = ICONS[key] || '<circle cx="12" cy="12" r="3" fill="currentColor" stroke="none"/>';
        var span = document.createElement('span');
        span.className = 'ph-ico' + (cls ? ' ' + cls : '');
        span.setAttribute('aria-hidden', 'true');
        span.innerHTML = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" ' +
            'stroke-linecap="round" stroke-linejoin="round">' + body + '</svg>';
        return span;
    }
    PH.icon = icon;
    PH.hasIcon = function (name) { return !!(ICONS[name] || ICON_ALIAS[String(name || '').toLowerCase()]); };

    // --------------------------------------------------------------- values --

    function clone(v) { return v === undefined ? undefined : JSON.parse(JSON.stringify(v)); }

    function deepEqual(a, b) {
        if (a === b) return true;
        if (typeof a !== typeof b || a === null || b === null || typeof a !== 'object') return false;
        if (Array.isArray(a) !== Array.isArray(b)) return false;
        if (Array.isArray(a)) {
            if (a.length !== b.length) return false;
            for (var i = 0; i < a.length; i++) if (!deepEqual(a[i], b[i])) return false;
            return true;
        }
        var ka = Object.keys(a), kb = Object.keys(b);
        if (ka.length !== kb.length) return false;
        for (var j = 0; j < ka.length; j++) if (!deepEqual(a[ka[j]], b[ka[j]])) return false;
        return true;
    }

    function isVec(v) { return !!v && typeof v === 'object' && /^vec[234]$/.test(v.__type); }
    function isHash(v) { return !!v && typeof v === 'object' && v.__type === 'hash'; }
    /** A cell the server cannot rewrite (built from code): { __type: 'code', source, reason }. */
    function isCode(v) { return !!v && typeof v === 'object' && v.__type === 'code'; }
    function isPlainObj(v) { return !!v && typeof v === 'object' && !Array.isArray(v) && !isVec(v) && !isHash(v) && !isCode(v); }
    function vecAxes(v) { return v.__type === 'vec2' ? ['x', 'y'] : v.__type === 'vec3' ? ['x', 'y', 'z'] : ['x', 'y', 'z', 'w']; }

    function num(n) {
        if (typeof n !== 'number' || !isFinite(n)) return String(n);
        return Math.round(n * 100) / 100 === n ? String(n) : String(Math.round(n * 1000) / 1000);
    }

    /** A short one-line rendering of any value, for tables, diffs and history. */
    function fmtValue(v, max) {
        max = max || 80;
        var s;
        if (v === undefined) s = '—';
        else if (v === null) s = 'nil';
        else if (typeof v === 'boolean') s = v ? 'On' : 'Off';
        else if (typeof v === 'number') s = num(v);
        else if (typeof v === 'string') s = v === '' ? '(empty)' : v;
        else if (isVec(v)) s = vecAxes(v).map(function (a) { return num(v[a]); }).join(', ');
        else if (isHash(v)) s = v.name;
        else if (isCode(v)) s = String(v.source || 'code');
        else if (Array.isArray(v)) {
            if (!v.length) s = '(none)';
            else if (v.every(function (x) { return typeof x !== 'object' || x === null; })) s = v.map(function (x) { return fmtValue(x); }).join(', ');
            else s = v.length + (v.length === 1 ? ' entry' : ' entries');
        } else if (typeof v === 'object') {
            var keys = Object.keys(v).filter(function (k) { return k !== '__int_keys'; });
            s = keys.length ? '{ ' + keys.slice(0, 4).join(', ') + (keys.length > 4 ? ', …' : '') + ' }' : '(empty)';
        } else s = String(v);
        return s.length > max ? s.slice(0, max - 1) + '…' : s;
    }

    function typeOf(v) {
        if (typeof v === 'boolean') return 'boolean';
        if (typeof v === 'number') return 'number';
        if (typeof v === 'string') return 'string';
        if (isVec(v)) return v.__type === 'vec2' ? 'vector2' : v.__type === 'vec3' ? 'vector3' : 'vector4';
        if (isHash(v)) return 'hash';
        if (isCode(v)) return 'code';
        if (Array.isArray(v)) return 'array';
        if (v && typeof v === 'object') return 'table';
        return 'nil';
    }

    /** MinBid -> "Min bid", DROP_TIMEOUT -> "Drop timeout", item_sources -> "Item sources". */
    function readable(key) {
        var s = String(key == null ? '' : key);
        if (/^\d+$/.test(s)) return '#' + s;
        s = s.replace(/[_\-]+/g, ' ')
             .replace(/([a-z0-9])([A-Z])/g, '$1 $2')
             .replace(/([A-Z]+)([A-Z][a-z])/g, '$1 $2')
             .trim();
        if (!s) return String(key);
        // All-caps words (URL, ID, NPC) stay as they are unless the whole key was shouting.
        var shouting = s === s.toUpperCase() && /[A-Z]{2}/.test(s);
        var words = s.split(/\s+/).map(function (w, i) {
            if (!shouting && w.length > 1 && w === w.toUpperCase() && /[A-Z]/.test(w)) return w;
            w = w.toLowerCase();
            return i === 0 ? w.charAt(0).toUpperCase() + w.slice(1) : w;
        });
        return words.join(' ');
    }

    PH.clone = clone; PH.deepEqual = deepEqual; PH.isVec = isVec; PH.isHash = isHash; PH.isCode = isCode;
    PH.isPlainObj = isPlainObj; PH.vecAxes = vecAxes; PH.fmtValue = fmtValue; PH.typeOf = typeOf;
    PH.readable = readable; PH.num = num;

    // ---------------------------------------------------------------- paths --
    // Lua-shaped: Config.Stores[3].label, Config.Lang["en"].greeting.
    // Segments: strings for keys, numbers for 1-based array indices.

    function parsePath(path) {
        var s = String(path), segs = [], i = 0, n = s.length;
        while (i < n) {
            var c = s.charAt(i);
            if (c === '.') { i++; continue; }
            if (c === '[') {
                var q = s.charAt(i + 1);
                if (q === '"' || q === "'") {
                    var j = i + 2, out = '';
                    while (j < n && s.charAt(j) !== q) {
                        if (s.charAt(j) === '\\' && j + 1 < n) { out += s.charAt(j + 1); j += 2; continue; }
                        out += s.charAt(j); j++;
                    }
                    segs.push(out);
                    i = s.indexOf(']', j) + 1;
                    if (i <= 0) break;
                } else {
                    var close = s.indexOf(']', i);
                    if (close < 0) break;
                    var inner = s.slice(i + 1, close).trim();
                    segs.push(/^-?\d+$/.test(inner) ? parseInt(inner, 10) : inner);
                    i = close + 1;
                }
                continue;
            }
            var m = /^[^.\[]+/.exec(s.slice(i));
            segs.push(m[0]);
            i += m[0].length;
        }
        return segs;
    }

    var IDENT = /^[A-Za-z_][A-Za-z0-9_]*$/;

    function fmtSeg(seg, bracketStrings) {
        if (typeof seg === 'number') return '[' + seg + ']';
        if (!bracketStrings && IDENT.test(seg)) return '.' + seg;
        return '["' + String(seg).replace(/\\/g, '\\\\').replace(/"/g, '\\"') + '"]';
    }

    /**
     * Append one segment to a path, in the server's own spelling (I.canon in
     * sv_hub.lua): identifier-like keys as .key, other strings as ["key"],
     * integers as [n]. The page compares paths as strings (pending badges,
     * data-path lookups), so every path it builds must be spelled one way.
     * The third argument is kept for older callers and no longer changes anything.
     */
    function joinPath(base, seg) {
        return base + fmtSeg(seg, false);
    }

    var canonMemo = {};
    var canonCount = 0;
    /** One spelling for a path: Config.Lang["en"] and Config.Lang.en are the same key. */
    function canon(path) {
        if (typeof path !== 'string') return path;
        var hit = canonMemo[path];
        if (hit !== undefined) return hit;
        var segs = parsePath(path);
        var out = segs.length ? String(segs[0]) : path;
        for (var i = 1; i < segs.length; i++) out += fmtSeg(segs[i], false);
        if (++canonCount > 20000) { canonMemo = {}; canonCount = 0; }
        canonMemo[path] = out;
        return out;
    }

    function segKey(segs) {
        return segs.map(function (s) { return (typeof s === 'number' ? '#' : '$') + s; }).join('');
    }

    /** Config.Zones[3].loot[2].item -> loot[].item (relative to a row, for hub.json field keys). */
    function fieldPattern(segs) {
        var out = '';
        segs.forEach(function (s) {
            if (typeof s === 'number') out += '[]';
            else out += (out ? '.' : '') + s;
        });
        return out;
    }

    PH.parsePath = parsePath; PH.joinPath = joinPath; PH.segKey = segKey; PH.fieldPattern = fieldPattern; PH.canon = canon;
    /** A CSS attribute selector value for a path, in its one spelling. */
    PH.pathSel = function (path) { return String(canon(path)).replace(/(["\\])/g, '\\$1'); };
    PH.IDENT = IDENT;

    // ------------------------------------------------------------------ NUI --

    function post(name, body) {
        return fetch('https://' + RESOURCE + '/' + name, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json; charset=UTF-8' },
            body: JSON.stringify(body || {}),
        }).then(function (r) { return r.json(); });
    }

    /**
     * One server call through client/cl_hub.lua.
     * Resolves (never rejects) to { ok: true, value } or { ok: false, err, ... }.
     */
    function api(call) {
        var args = Array.prototype.slice.call(arguments, 1);
        return post('hub', { call: call, args: args }).then(function (r) {
            if (r && typeof r.ok === 'boolean') return r;
            return { ok: false, err: 'bad_answer' };
        }, function () {
            return { ok: false, err: 'network' };
        });
    }

    PH.post = post;
    PH.api = api;

    /** Plain words for the error codes the server and client can return. */
    /** The plain-English reason of a refused answer: the server's own message when it gave one. */
    PH.errMsg = function (r) {
        if (r && typeof r.message === 'string' && r.message) return r.message;
        return PH.errText(r && r.err);
    };

    /** poggy_core itself: never restarted from the hub. */
    PH.isCore = function (id) {
        var c = PH.cardById ? PH.cardById(id) : null;
        return id === 'poggy_core' || !!(c && c.self);
    };

    PH.errText = function (err) {
        var map = {
            timeout: 'The server did not answer in time. It may still be working on it.',
            network: 'The request did not reach the game client.',
            no_answer: 'The server has no answer for this yet (is poggy_core up to date?).',
            forbidden: 'You do not have permission to do that.',
            denied: 'You do not have permission to do that.',
            missing: 'That script is not on this server any more.',
            self: 'Restart poggy_core by hand; it restarts every Poggy script.',
            unavailable: 'That is not available on this server.',
            write_failed: 'The file could not be written. Nothing was changed.',
            restart_failed: 'The script did not restart. See the server console.',
            no_permission: 'You do not have permission to do that.',
            locked: 'Someone else is editing this script.',
            not_locked: 'You are not holding this script’s edit lock any more.',
            stopped: 'The script is stopped. Start it first: a stopped script cannot write its own files.',
            stale: 'The config files changed on disk since you opened them.',
            invalid: 'One of the values is not valid.',
            not_found: 'That script is not on this server any more.',
            closed: 'The hub was closed.',
            bad_argument: 'The request was malformed.',
        };
        return map[err] || ('Failed: ' + (err || 'unknown error'));
    };

    // ------------------------------------------------------------ clipboard --

    PH.copyText = function (text) {
        // navigator.clipboard is unreliable inside CEF; the textarea trick works everywhere.
        var ta = h('textarea', { style: { position: 'fixed', left: '-9999px', top: '0' } });
        ta.value = String(text);
        document.body.appendChild(ta);
        ta.select();
        var ok = false;
        try { ok = document.execCommand('copy'); } catch (e) { ok = false; }
        document.body.removeChild(ta);
        return ok;
    };

    // ------------------------------------------------------------------ time --

    function toDate(at) {
        if (at === null || at === undefined || at === '') return null;
        if (typeof at === 'number') return new Date(at < 1e12 ? at * 1000 : at);
        if (/^\d+$/.test(String(at))) return toDate(Number(at));
        var d = new Date(String(at).replace(' ', 'T'));
        return isNaN(d.getTime()) ? null : d;
    }

    PH.when = function (at) {
        var d = toDate(at);
        if (!d) return { rel: String(at || ''), abs: '' };
        var diff = (Date.now() - d.getTime()) / 1000;
        var rel;
        if (diff < 45) rel = 'just now';
        else if (diff < 90) rel = 'a minute ago';
        else if (diff < 3600) rel = Math.round(diff / 60) + ' minutes ago';
        else if (diff < 5400) rel = 'an hour ago';
        else if (diff < 86400) rel = Math.round(diff / 3600) + ' hours ago';
        else if (diff < 172800) rel = 'yesterday';
        else if (diff < 86400 * 30) rel = Math.round(diff / 86400) + ' days ago';
        else rel = d.toLocaleDateString(undefined, { day: 'numeric', month: 'short', year: 'numeric' });
        var abs = d.toLocaleString(undefined, { day: 'numeric', month: 'short', year: 'numeric', hour: '2-digit', minute: '2-digit' });
        return { rel: rel, abs: abs };
    };

    // --------------------------------------------------------------- layers --
    // Everything that floats (modal, drawer, palette, popover) registers here so
    // Escape closes the top one first, and only then reaches the page.

    var layers = [];
    PH.layers = layers;
    PH.pushLayer = function (layer) { layers.push(layer); return layer; };
    PH.popLayer = function (layer) {
        var i = layers.indexOf(layer);
        if (i !== -1) layers.splice(i, 1);
    };
    PH.topLayer = function () { return layers[layers.length - 1] || null; };

    function overlayRoot() { return PH.els && PH.els.overlays ? PH.els.overlays : document.body; }

    // --------------------------------------------------------------- toasts --

    /**
     * PH.toast({ kind: 'success'|'error'|'info'|'warning', title, text, timeout, actions: [{ label, onClick, primary }] })
     * Returns { close, set(text) }.
     */
    PH.toast = function (opts) {
        opts = opts || {};
        var host = PH.els && PH.els.toasts;
        if (!host) return { close: function () {}, set: function () {} };
        var kind = opts.kind || 'info';
        var textEl = h('div.ph-toast__text', opts.text || '');
        var el = h('div.ph-toast.ph-toast--' + kind, { role: 'status' }, [
            icon(kind === 'success' ? 'check' : kind === 'error' ? 'alert' : kind === 'warning' ? 'alert' : 'info', 'ph-toast__ico'),
            h('div.ph-toast__body', [
                opts.title ? h('div.ph-toast__title', opts.title) : null,
                opts.text ? textEl : null,
                opts.actions && opts.actions.length ? h('div.ph-toast__actions', opts.actions.map(function (a) {
                    return h('button.ph-btn.ph-btn--sm' + (a.primary ? '.ph-btn--primary' : '.ph-btn--ghost'), {
                        type: 'button',
                        onclick: function () { if (a.onClick) a.onClick(); if (a.keep !== true) close(); },
                    }, a.label);
                })) : null,
            ]),
            h('button.ph-iconbtn.ph-toast__close', { type: 'button', title: 'Dismiss', onclick: function () { close(); } }, icon('x')),
        ]);
        // Keep the stack short: the oldest toast goes when a fifth arrives.
        var live = PH.$$('.ph-toast:not(.is-out)', host);
        if (live.length >= 4 && live[0].__phClose) live[0].__phClose();
        host.appendChild(el);
        requestAnimationFrame(function () { el.classList.add('is-in'); });
        var timer = null;
        var timeout = opts.timeout === 0 ? 0 : (opts.timeout || (kind === 'error' ? 9000 : 4500));
        if (timeout) timer = setTimeout(close, timeout);
        var closed = false;
        function close() {
            if (closed) return;
            closed = true;
            if (timer) clearTimeout(timer);
            el.classList.remove('is-in');
            el.classList.add('is-out');
            setTimeout(function () { if (el.parentNode) el.parentNode.removeChild(el); }, 260);
            if (opts.onClose) opts.onClose();
        }
        el.__phClose = close;
        return {
            close: close,
            el: el,
            set: function (text) { textEl.textContent = text; if (!textEl.parentNode) el.querySelector('.ph-toast__body').appendChild(textEl); },
        };
    };

    // --------------------------------------------------------------- modals --

    /**
     * PH.modal({ title, icon, body: Node|string, actions: [{ id, label, kind, onClick }], wide, tone, dismissable })
     * An action's onClick may return false to keep the modal open, or a Promise.
     * Returns a Promise of the chosen action id (null when dismissed), with .close().
     */
    PH.modal = function (opts) {
        opts = opts || {};
        var resolveFn;
        var p = new Promise(function (r) { resolveFn = r; });
        var dismissable = opts.dismissable !== false;

        var foot = h('div.ph-modal__foot');
        var box = h('div.ph-modal' + (opts.wide ? '.ph-modal--wide' : '') + (opts.tone ? '.ph-modal--' + opts.tone : ''), {
            role: 'dialog', 'aria-modal': 'true',
        }, [
            h('div.ph-modal__head', [
                opts.icon ? icon(opts.icon, 'ph-modal__ico') : null,
                h('h2.ph-modal__title', opts.title || ''),
                dismissable ? h('button.ph-iconbtn', { type: 'button', title: 'Close (Esc)', onclick: function () { done(null); } }, icon('x')) : null,
            ]),
            h('div.ph-modal__body', typeof opts.body === 'string' ? h('p', opts.body) : opts.body),
            foot,
        ]);
        var scrim = h('div.ph-scrim', {
            onmousedown: function (e) { if (e.target === scrim && dismissable) done(null); },
        }, box);

        (opts.actions || [{ id: 'ok', label: 'OK', kind: 'primary' }]).forEach(function (a) {
            var btn = h('button.ph-btn.ph-btn--' + (a.kind || 'ghost'), { type: 'button' }, [a.icon ? icon(a.icon) : null, a.label]);
            btn.addEventListener('click', function () {
                if (!a.onClick) { done(a.id); return; }
                var r = a.onClick();
                if (r === false) return;
                if (r && typeof r.then === 'function') {
                    btn.disabled = true;
                    btn.classList.add('is-busy');
                    r.then(function (keep) {
                        btn.disabled = false;
                        btn.classList.remove('is-busy');
                        if (keep !== false) done(a.id);
                    });
                    return;
                }
                done(a.id);
            });
            if (a.align === 'left') btn.classList.add('ph-modal__left');
            foot.appendChild(btn);
        });

        var layer = { kind: 'modal', close: function () { if (dismissable) done(null); } };
        PH.pushLayer(layer);
        overlayRoot().appendChild(scrim);
        requestAnimationFrame(function () { scrim.classList.add('is-in'); });
        // Focus the safe primary action so Enter does the expected thing and Tab
        // starts inside. A destructive button is never the default: Enter on a
        // "Discard" or "Delete" dialog must not destroy anything.
        setTimeout(function () {
            var focusEl = box.querySelector('[autofocus]') || foot.querySelector('.ph-btn--primary') ||
                foot.querySelector('.ph-btn--go') || foot.querySelector('.ph-btn--ghost') || foot.querySelector('.ph-btn');
            if (focusEl) focusEl.focus();
        }, 30);

        var finished = false;
        function done(id) {
            if (finished) return;
            finished = true;
            PH.popLayer(layer);
            scrim.classList.remove('is-in');
            setTimeout(function () { if (scrim.parentNode) scrim.parentNode.removeChild(scrim); }, 200);
            if (opts.onClose) opts.onClose(id);
            resolveFn(id);
        }
        p.close = function () { done(null); };
        p.box = box;
        return p;
    };

    /** Yes/no. Resolves true when the confirming action is chosen. */
    PH.confirm = function (opts) {
        return PH.modal({
            title: opts.title,
            icon: opts.icon || (opts.danger ? 'alert' : 'info'),
            tone: opts.danger ? 'danger' : null,
            body: opts.body,
            actions: [
                { id: 'no', label: opts.cancel || 'Cancel', kind: 'ghost' },
                { id: 'yes', label: opts.ok || 'OK', kind: opts.danger ? 'danger' : 'primary', icon: opts.okIcon },
            ],
        }).then(function (id) { return id === 'yes'; });
    };

    /** One line of text. Resolves the string, or null when cancelled. */
    PH.prompt = function (opts) {
        var input = h('input.ph-input', { type: 'text', value: opts.value || '', placeholder: opts.placeholder || '', autofocus: true, spellcheck: 'false' });
        var err = h('div.ph-field__error');
        var result = null;
        var m = PH.modal({
            title: opts.title,
            icon: opts.icon || 'pencil',
            body: h('div.ph-prompt', [opts.text ? h('p', opts.text) : null, input, err]),
            actions: [
                { id: 'no', label: 'Cancel', kind: 'ghost' },
                { id: 'yes', label: opts.ok || 'OK', kind: 'primary', onClick: function () {
                    var v = input.value;
                    var bad = opts.validate ? opts.validate(v) : null;
                    if (bad) { err.textContent = bad; input.focus(); return false; }
                    result = v;
                } },
            ],
        });
        input.addEventListener('keydown', function (e) {
            if (e.key === 'Enter') { e.preventDefault(); m.box.querySelector('.ph-btn--primary').click(); }
        });
        setTimeout(function () { input.focus(); input.select(); }, 40);
        return m.then(function () { return result; });
    };

    // ------------------------------------------------------------- popovers --

    /**
     * A floating panel under (or above) an anchor. Closes on outside click,
     * Escape, scroll of the page, or close(). Returns { el, close }.
     */
    PH.popover = function (anchor, content, opts) {
        opts = opts || {};
        var el = h('div.ph-pop' + (opts.cls ? '.' + opts.cls : ''), { role: 'dialog' }, content);
        overlayRoot().appendChild(el);
        var layer = { kind: 'popover', close: function () { close(); } };
        PH.pushLayer(layer);

        function place() {
            var r = anchor.getBoundingClientRect();
            var vw = window.innerWidth, vh = window.innerHeight;
            var w = opts.width || Math.max(r.width, opts.minWidth || 220);
            el.style.width = w + 'px';
            var left = opts.alignRight ? r.right - w : r.left;
            left = Math.max(12, Math.min(left, vw - w - 12));
            el.style.left = left + 'px';
            var below = vh - r.bottom, above = r.top;
            var maxH = opts.maxHeight || 380;
            if (below < Math.min(maxH, el.scrollHeight) + 16 && above > below) {
                el.style.top = '';
                el.style.bottom = (vh - r.top + 6) + 'px';
                el.style.maxHeight = Math.min(maxH, above - 20) + 'px';
            } else {
                el.style.bottom = '';
                el.style.top = (r.bottom + 6) + 'px';
                el.style.maxHeight = Math.min(maxH, below - 20) + 'px';
            }
        }
        place();
        requestAnimationFrame(function () { el.classList.add('is-in'); });

        function onDown(e) {
            if (el.contains(e.target) || anchor.contains(e.target)) return;
            close();
        }
        function onScroll(e) {
            if (el.contains(e.target)) return;
            close();
        }
        setTimeout(function () {
            document.addEventListener('mousedown', onDown, true);
            document.addEventListener('scroll', onScroll, true);
        }, 0);
        window.addEventListener('resize', close);

        var closed = false;
        function close() {
            if (closed) return;
            closed = true;
            PH.popLayer(layer);
            document.removeEventListener('mousedown', onDown, true);
            document.removeEventListener('scroll', onScroll, true);
            window.removeEventListener('resize', close);
            if (el.parentNode) el.parentNode.removeChild(el);
            if (opts.onClose) opts.onClose();
        }
        return { el: el, close: close, place: place };
    };

    /**
     * A menu of actions under an anchor.
     * items: [{ label, icon, onClick, danger, disabled, hint }] or '-' for a divider.
     */
    PH.menu = function (anchor, items, opts) {
        var pop;
        var list = h('div.ph-menu', { role: 'menu' });
        items.forEach(function (it) {
            if (it === '-') { list.appendChild(h('div.ph-menu__sep')); return; }
            if (!it) return;
            list.appendChild(h('button.ph-menu__item' + (it.danger ? '.is-danger' : '') + (it.active ? '.is-active' : ''), {
                type: 'button', role: 'menuitem', disabled: it.disabled, title: it.title || null,
                onclick: function () { pop.close(); if (it.onClick) it.onClick(); },
            }, [it.icon ? icon(it.icon) : h('span.ph-ico'), h('span.ph-menu__label', it.label), it.hint ? h('span.ph-menu__hint', it.hint) : null]));
        });
        pop = PH.popover(anchor, list, Object.assign({ minWidth: 220 }, opts || {}));
        var first = list.querySelector('.ph-menu__item:not([disabled])');
        if (first) first.focus();
        list.addEventListener('keydown', function (e) {
            var btns = PH.$$('.ph-menu__item:not([disabled])', list);
            var i = btns.indexOf(document.activeElement);
            if (e.key === 'ArrowDown') { e.preventDefault(); btns[(i + 1) % btns.length].focus(); }
            if (e.key === 'ArrowUp') { e.preventDefault(); btns[(i - 1 + btns.length) % btns.length].focus(); }
        });
        return pop;
    };

    // -------------------------------------------------------------- tooltip --
    // One shared tooltip. An element opts in with PH.tip(el, fnOrNode).

    var tipEl = null, tipTimer = null, tipFor = null;

    function showTip(target) {
        var content = target.__phTip;
        if (!content) return;
        if (!tipEl) {
            tipEl = h('div.ph-tip', { role: 'tooltip' });
            overlayRoot().appendChild(tipEl);
        }
        clear(tipEl);
        var c = typeof content === 'function' ? content() : content;
        append(tipEl, typeof c === 'string' ? h('div.ph-tip__text', c) : c);
        tipEl.classList.add('is-in');
        tipFor = target;
        var r = target.getBoundingClientRect();
        var tw = tipEl.offsetWidth, th = tipEl.offsetHeight;
        var left = Math.max(10, Math.min(r.left + r.width / 2 - tw / 2, window.innerWidth - tw - 10));
        var top = r.top - th - 10;
        tipEl.classList.toggle('is-below', top < 8);
        if (top < 8) top = r.bottom + 10;
        tipEl.style.left = left + 'px';
        tipEl.style.top = top + 'px';
    }

    function hideTip() {
        clearTimeout(tipTimer);
        tipFor = null;
        if (tipEl) tipEl.classList.remove('is-in');
    }

    PH.tip = function (el, content) {
        el.__phTip = content;
        if (el.__phTipBound) return el;
        el.__phTipBound = true;
        el.addEventListener('mouseenter', function () { clearTimeout(tipTimer); tipTimer = setTimeout(function () { showTip(el); }, 220); });
        el.addEventListener('mouseleave', hideTip);
        el.addEventListener('focus', function () { showTip(el); });
        el.addEventListener('blur', hideTip);
        el.addEventListener('mousedown', hideTip);
        return el;
    };
    PH.hideTip = hideTip;

    // ----------------------------------------------------------------- misc --

    PH.debounce = function (fn, ms) {
        var t = null;
        return function () {
            var args = arguments, self = this;
            clearTimeout(t);
            t = setTimeout(function () { fn.apply(self, args); }, ms);
        };
    };

    PH.plural = function (n, one, many) { return n + ' ' + (n === 1 ? one : (many || pluralOf(one))); };

    /** "store" -> "stores", "entry" -> "entries", "catalog" -> "catalogs". */
    function pluralOf(word) {
        var w = String(word);
        if (/[^aeiou]y$/i.test(w)) return w.slice(0, -1) + 'ies';
        if (/(s|x|z|ch|sh)$/i.test(w)) return w + 'es';
        return w + 's';
    }
    PH.pluralOf = pluralOf;

    /** "Catalogs" -> "catalog", "Store types" -> "store type": an item name from a list's label. */
    PH.singular = function (label) {
        var s = String(label || '').trim().toLowerCase();
        if (/ies$/.test(s)) return s.slice(0, -3) + 'y';
        if (/(ss|us)$/.test(s)) return s;
        if (/(ches|shes|xes)$/.test(s)) return s.slice(0, -2);
        if (/s$/.test(s)) return s.slice(0, -1);
        return s;
    };

    /**
     * Per-viewer conveniences (which rail tabs are open). Storage can be
     * missing or throw inside CEF, so every call is guarded and the page
     * works the same without it.
     */
    PH.store = {
        get: function (key, fallback) {
            try {
                var raw = window.localStorage && window.localStorage.getItem('poggyhub:' + key);
                return raw ? JSON.parse(raw) : fallback;
            } catch (e) { return fallback; }
        },
        set: function (key, value) {
            try { if (window.localStorage) window.localStorage.setItem('poggyhub:' + key, JSON.stringify(value)); } catch (e) { /* storage off */ }
        },
    };
    /** "was" or "were", "is" or "are", to go with a count. */
    PH.verb = function (n, one, many) { return n === 1 ? one : many; };

    /** Case-insensitive "does any of these strings contain every word of q". */
    PH.matches = function (q, strings) {
        if (!q) return true;
        var hay = strings.filter(function (s) { return s !== null && s !== undefined; }).join('  ').toLowerCase();
        return q.toLowerCase().split(/\s+/).every(function (w) { return !w || hay.indexOf(w) !== -1; });
    };

    /** Scroll an element into the middle of its scroll container and flash it. */
    PH.flash = function (el) {
        if (!el) return;
        el.scrollIntoView({ behavior: 'smooth', block: 'center' });
        el.classList.remove('is-flash');
        void el.offsetWidth;   // restart the animation
        el.classList.add('is-flash');
        setTimeout(function () { el.classList.remove('is-flash'); }, 2200);
    };

    // Category colours for the fallback monogram tiles and chips.
    PH.CATEGORY_COLORS = {
        Economy: '#c2a15c', Jobs: '#7fa66a', World: '#5d8fa6', Roleplay: '#b4514e',
        Admin: '#8c73b3', Utility: '#9a8c74', Framework: '#cc7459',
    };
    PH.categoryColor = function (cat) {
        if (PH.CATEGORY_COLORS[cat]) return PH.CATEGORY_COLORS[cat];
        // Anything unknown gets a stable colour from its name.
        var s = String(cat || ''), hsh = 0;
        for (var i = 0; i < s.length; i++) hsh = (hsh * 31 + s.charCodeAt(i)) | 0;
        return 'hsl(' + (Math.abs(hsh) % 360) + ', 32%, 56%)';
    };
})();
