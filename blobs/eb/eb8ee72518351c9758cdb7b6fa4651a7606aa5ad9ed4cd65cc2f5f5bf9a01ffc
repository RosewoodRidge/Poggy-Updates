/*
    Poggy Hub — the radial builder (0.26.0).

    A list whose hub.json entry says "view": "radial" is edited on a mock of
    the wheel instead of in a table: the ring as players see it on top, the
    option picked on it below. Click an option to edit it, click a submenu
    again to go inside, click the middle to go back, drag an option onto
    another to swap their places.

    The list's rows are menu options in poggy_menu's shape (its config.lua
    says what each field means):
        id label description icon tone items
        command | event | serverEvent | export { resource name args } | builtin, args
        jobs minGrade onDuty admin ace resource when showLocked lockedReason
        confirm cooldown keepOpen badge disabled disabledReason hidden

    hub.json, on the list:
        "view": "radial",
        "radial": {
            "icons":       "ui/icons.js",              the script's icon library (window.PoggyIcons)
            "perRing":     "Config.Ui.MaxPerRing",      options on one ring before "More"
            "iconColours": "Config.Ui.IconColours",     false: every icon the text colour
            "builtins":    "Config.Builtins",           a built-in switched off hides its options
            "adminAce":    "Config.AdminAce"            named in the "Staff only" help
        }
    Every entry is optional. Labels, tooltips and the choices for `tone`,
    `builtin` and `when` come from the list's `fields`, as for the table.

    Every edit is an ordinary change on the working copy (hub-state.js), so
    Save, Discard, History and Undo work as for any list. A field that is
    emptied is taken out of its row (the whole row is set without it), so
    the file never collects command = "" lines.
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var h = PH.h, icon = PH.icon, S = PH.S, F = PH.F;
    var SVGNS = 'http://www.w3.org/2000/svg';

    // The game's geometry (poggy_menu ui/menu.js), in a 1000-wide SVG centred on 0,0.
    var R_OUT = 470, R_IN = 196, GAP = 5, R_ITEM = 333, R_SUB = 449, R_KEY = 222;
    var MAX_DEPTH = 6;

    var ACTION_KEYS = ['command', 'event', 'serverEvent', 'export', 'builtin', 'args', 'items'];
    var ACTIONS = [
        { id: 'command', label: 'Command', icon: 'terminal', tip: 'Runs a chat command, as if the player typed it.' },
        { id: 'export', label: 'Export', icon: 'link', tip: 'Calls a client export of another script.' },
        { id: 'event', label: 'Client event', icon: 'bolt', tip: 'Fires an event on the player\'s own game.' },
        { id: 'serverEvent', label: 'Server event', icon: 'globe', tip: 'Sends an event to the server. The script that listens must check who sent it.' },
        { id: 'builtin', label: 'Built-in', icon: 'star', tip: 'Something the menu does itself.' },
        { id: 'submenu', label: 'Submenu', icon: 'grid', tip: 'Opens more options.' },
    ];
    var TONE_LABELS = {
        '': 'Auto', law: 'Law', medical: 'Medical', alert: 'Alert', money: 'Money',
        travel: 'Travel', social: 'Social', none: 'None',
    };
    var WHEN_LABELS = { alive: 'Alive', dead: 'Down', any: 'Any' };

    // Icon guesses for an option made from a command.
    var GUESS = [
        [/ticket/, 'ticket'], [/poggy|setting|config/, 'gear'], [/admin|staff|mod\b/, 'user-shield'],
        [/shop|store|market/, 'store'], [/job/, 'tools'], [/blip|map|gps|waypoint/, 'marker'],
        [/emote|anim/, 'emote'], [/craft/, 'hammer'], [/bank|money|cash/, 'bank'], [/duty/, 'badge'],
        [/storage|stash/, 'chest'], [/invis/, 'eye'], [/help/, 'help'], [/drop|crate/, 'box'],
        [/hunt|clue/, 'treasure'], [/trash|bin/, 'trash'], [/dead|body/, 'skull'], [/police|law|jail/, 'badge'],
        [/doctor|medic/, 'medical'], [/horse|stable/, 'horse'], [/scene|status/, 'scroll'], [/transform/, 'mask'],
        [/badge/, 'star'], [/chess|game/, 'dice'], [/balloon/, 'travel'], [/fish/, 'fishing'],
    ];

    function isObj(v) { return v !== null && typeof v === 'object' && !Array.isArray(v); }
    function list(v) {
        if (v === undefined || v === null || v === '') return [];
        return Array.isArray(v) ? v.filter(function (x) { return x !== undefined && x !== null && x !== ''; }) : [v];
    }
    function kids(it) {
        if (!it) return null;
        if (Array.isArray(it.items)) return it.items;
        if (isObj(it.items)) return Object.keys(it.items).length ? null : [];   // an empty table arrives as {}
        return null;
    }
    function hasItems(it) { var k = kids(it); return !!k && k.length > 0; }
    function lower(s) { return typeof s === 'string' ? s.toLowerCase() : s; }
    function q(s) { return '"' + String(s == null ? '' : s).replace(/\\/g, '\\\\').replace(/"/g, '\\"') + '"'; }

    /** What an option does, in poggy_menu's own order (cl_actions.lua). */
    function actionOf(it) {
        if (!isObj(it)) return 'none';
        if (hasItems(it)) return 'submenu';
        if (it.builtin !== undefined) return 'builtin';
        if (it.command !== undefined) return 'command';
        if (it.event !== undefined) return 'event';
        if (it.serverEvent !== undefined) return 'serverEvent';
        if (it.export !== undefined) return 'export';
        if (kids(it)) return 'submenu';
        return 'none';
    }

    /** Arguments as the owner types them: `valentine, 2, true`. */
    function argsText(args) {
        return list(args).map(function (a) { return typeof a === 'string' ? a : JSON.stringify(a); }).join(', ');
    }
    function parseArgs(text) {
        text = String(text || '').trim();
        if (!text) return undefined;
        if (text.charAt(0) === '[') {
            try { var arr = JSON.parse(text); if (Array.isArray(arr)) return arr; } catch (e) { /* fall through to the comma list */ }
        }
        return text.split(',').map(function (s) {
            s = s.trim();
            if (/^-?\d+(\.\d+)?$/.test(s)) return Number(s);
            if (s === 'true') return true;
            if (s === 'false') return false;
            return s.replace(/^["']|["']$/g, '');
        });
    }
    function argsCode(args) {
        return list(args).map(function (a) { return typeof a === 'string' ? q(a) : JSON.stringify(a); }).join(', ');
    }

    function slug(s) {
        return String(s || '').toLowerCase().replace(/^\//, '').replace(/[^a-z0-9]+/g, '_').replace(/^_+|_+$/g, '') || 'option';
    }

    // ------------------------------------------------------------ icons --

    var libs = {};   // url -> Promise<PoggyIcons | null>

    /** A script's icon library. `global` is the name it sets on window: PoggyIcons,
     *  or another (a fork's own, hub.json radial "iconsGlobal"). */
    function loadLib(url, global) {
        global = /^[A-Za-z_$][\w$]*$/.test(String(global || '')) ? global : 'PoggyIcons';
        if (window[global]) return Promise.resolve(window[global]);
        if (!url) return Promise.resolve(null);
        if (!libs[url]) {
            libs[url] = new Promise(function (resolve) {
                var s = document.createElement('script');
                s.src = url;
                s.onload = function () { resolve(window[global] || null); };
                s.onerror = function () { resolve(null); };
                document.head.appendChild(s);
            });
        }
        return libs[url];
    }

    // ------------------------------------------------ icons and colours --

    var live = null;
    S.on(function () { if (live && live.el.isConnected) live.changed(); });

    PH.Radial = {};

    /** The icon library a script's hub.json names: a radial list's "icons", or a field's "icons". */
    function libGlobal(fieldMeta) {
        if (fieldMeta && fieldMeta.iconsGlobal) return fieldMeta.iconsGlobal;
        var lists = (S.cur && S.cur.meta && S.cur.meta.lists) || {}, g = null;
        Object.keys(lists).some(function (k) { g = lists[k] && lists[k].radial && lists[k].radial.iconsGlobal; return !!g; });
        return g;
    }
    function libUrl(fieldMeta) {
        var cur = S.cur, folder = cur && cur.card && cur.card.folder;
        if (!folder) return null;
        var rel = fieldMeta && fieldMeta.icons;
        if (!rel) {
            var lists = (cur.meta && cur.meta.lists) || {};
            Object.keys(lists).some(function (k) { rel = lists[k] && lists[k].radial && lists[k].radial.icons; return !!rel; });
        }
        return rel ? 'https://cfx-nui-' + folder + '/' + String(rel).replace(/^\//, '') : null;
    }

    function iconNode(lib, name, cls) {
        var span = h('span.' + (cls || 'ph-rb-ico'));
        if (lib && typeof lib.svg === 'function') span.innerHTML = lib.svg(name || 'dot');   // the script's own drawn icons
        else span.appendChild(icon(PH.hasIcon(name) ? name : 'star'));
        return span;
    }

    /** Icon names matching what was typed: exact, then starts-with, then contains; aliases say what they stand for. */
    function iconMatches(lib, q) {
        q = String(q || '').trim().toLowerCase();
        var names = lib.names(), out = [], seen = {};
        function add(name, via) { if (!seen[name]) { seen[name] = true; out.push({ name: name, via: via }); } }
        var groupOf = {};
        Object.keys(lib.groups || {}).forEach(function (g) { lib.groups[g].forEach(function (n) { if (!groupOf[n]) groupOf[n] = g; }); });
        if (!q) names.forEach(function (n) { add(n); });
        else {
            var aliases = lib.aliases || {};
            [function (x) { return x === q; }, function (x) { return x.indexOf(q) === 0; }, function (x) { return x.indexOf(q) > 0; }].forEach(function (test) {
                names.forEach(function (n) { if (test(n)) add(n); });
                Object.keys(aliases).forEach(function (a) { if (test(a)) add(aliases[a], a); });
            });
        }
        out.forEach(function (o) { o.group = groupOf[o.name] || ''; });
        return out;
    }

    /** Every icon in a grid, grouped and searchable. onPick(name). */
    /**
     * Every icon, in a window of its own: a search box, a button per category,
     * and a grid of fixed-size tiles (icon and name) that cannot be squeezed.
     * Clicking a tile picks it; Enter picks the first match; what is typed can
     * be kept as it is when no icon has that name. onPick(name).
     */
    function gridPicker(anchor, lib, cur, onPick) {
        var input = h('input.ph-input.ph-rb-imodal__search', {
            type: 'text', placeholder: 'Search 160 icons: police, horse, money, doctor…', spellcheck: 'false', autocomplete: 'off', autofocus: true,
        });
        var cats = h('div.ph-rb-imodal__cats');
        var grid = h('div.ph-rb-imodal__grid');
        var foot = h('div.ph-rb-imodal__foot');
        var group = '';
        var m = PH.modal({
            title: 'Choose an icon', icon: 'grid', wide: true,
            body: h('div.ph-rb-imodal', [h('div.ph-rb-imodal__bar', [icon('search'), input]), cats, grid, foot]),
            actions: [{ id: 'cancel', label: 'Cancel', kind: 'ghost' }],
        });
        m.box.classList.add('ph-modal--icons');
        function pick(name) { m.close(); onPick(name); }
        function tile(name, via) {
            return h('button.ph-rb-itile' + (name === cur ? '.is-on' : ''), {
                type: 'button', title: name + (via ? ' (found by “' + via + '”)' : ''),
                onclick: function () { pick(name); },
                onmouseenter: function () { foot.textContent = name + (via ? '   ·   found by “' + via + '”' : ''); },
            }, [iconNode(lib, name, 'ph-rb-itile__ico'), h('span.ph-rb-itile__name', name)]);
        }
        function drawCats() {
            PH.clear(cats);
            if (!lib || !lib.groups) return;
            [''].concat(Object.keys(lib.groups)).forEach(function (g) {
                cats.appendChild(h('button.ph-rb-imodal__cat' + (group === g ? '.is-on' : ''), {
                    type: 'button', onclick: function () { group = g; drawCats(); draw(); input.focus(); },
                }, g ? PH.readable(g) + ' ' + lib.groups[g].length : 'All ' + lib.names().length));
            });
        }
        function draw() {
            PH.clear(grid);
            if (!lib) { grid.appendChild(h('p.ph-rb-icons__none', 'The icon library is not loaded (is the script running?). Type an icon name in the box instead.')); return; }
            var qq = input.value.trim().toLowerCase();
            var hits = iconMatches(lib, qq);
            if (group && lib.groups && lib.groups[group]) hits = hits.filter(function (x) { return lib.groups[group].indexOf(x.name) >= 0; });
            hits.forEach(function (x) { grid.appendChild(tile(x.name, x.via)); });
            if (!hits.length) {
                grid.appendChild(h('div.ph-rb-imodal__none', [
                    h('p', 'No icon matches “' + qq + '”' + (group ? ' in ' + PH.readable(group) : '') + '.'),
                    qq ? h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { pick(qq); } }, 'Use “' + qq + '” anyway (it shows a dot until the library has it)') : null,
                ]));
            }
            foot.textContent = hits.length + (hits.length === 1 ? ' icon' : ' icons') + (cur ? '   ·   now: ' + cur : '');
        }
        input.addEventListener('input', PH.debounce(draw, 60));
        input.addEventListener('keydown', function (e) {
            if (e.key === 'Enter') { e.preventDefault(); var first = grid.querySelector('.ph-rb-itile'); if (first) pick(first.title.split(' (')[0]); }
        });
        drawCats();
        draw();
        setTimeout(function () { input.focus(); var on = grid.querySelector('.ph-rb-itile.is-on'); if (on) on.scrollIntoView({ block: 'center' }); }, 60);
    }

    /**
     * The icon box: type a name and a list drops down as you type, each row with
     * the icon drawn small and its name (an alias says what it stands for). Arrow
     * keys and Enter pick; the grid button shows every icon. What is typed is
     * kept even when it is not in the library (it shows a dot until it is).
     * opts: { value, lib: Promise, readonly, onPick(name|undefined) }
     */
    PH.Radial.iconCombo = function (opts) {
        var lib = null;
        var preview = h('span.ph-rb-icombo__ico');
        var input = h('input.ph-input.ph-input--mono.ph-rb-icombo__in', {
            type: 'text', value: opts.value || '', placeholder: 'Type: badge, horse, money…', spellcheck: 'false',
            readOnly: !!opts.readonly, autocomplete: 'off', dataset: { rbKey: 'icon' },
        });
        var gridBtn = h('button.ph-iconbtn', { type: 'button', title: 'Every icon', disabled: !!opts.readonly }, icon('grid'));
        var el = h('div.ph-rb-icombo', [preview, input, gridBtn]);
        var pop = null, list = null, rows = [], sel = 0, committed = opts.value || '';

        function showPreview(name) { PH.clear(preview); preview.appendChild(iconNode(lib, name || 'dot', 'ph-rb-icombo__svg')); }
        function commit(name) {
            name = String(name || '').trim();
            if (name === committed) return;
            committed = name;
            opts.onPick(name || undefined);
        }
        function close() { if (pop) { var p2 = pop; pop = null; p2.close(); } }
        function open() {
            if (opts.readonly || !lib) return;
            if (!pop) {
                list = h('div.ph-rb-isug', { role: 'listbox' });
                pop = PH.popover(el, list, { minWidth: 300, maxHeight: 340, cls: 'ph-pop--pick', onClose: function () { pop = null; } });
            }
            draw();
        }
        function draw() {
            if (!list) return;
            rows = iconMatches(lib, input.value).slice(0, 80);
            sel = 0;
            PH.clear(list);
            if (!rows.length) list.appendChild(h('div.ph-rb-isug__none', 'No icon is called that. Enter keeps “' + input.value.trim() + '” (it shows a dot).'));
            rows.forEach(function (m, i) {
                list.appendChild(h('button.ph-rb-isug__row' + (i === sel ? '.is-sel' : '') + (m.name === committed ? '.is-cur' : ''), {
                    type: 'button', role: 'option',
                    onmousedown: function (e) { e.preventDefault(); },          // keep the focus in the box
                    onclick: function () { choose(i); },
                    onmouseenter: function () { mark(i); },
                }, [
                    iconNode(lib, m.name, 'ph-rb-isug__ico'),
                    h('span.ph-rb-isug__name', m.name),
                    m.via ? h('span.ph-rb-isug__via', '“' + m.via + '”') : null,
                    m.group ? h('span.ph-rb-isug__grp', PH.readable(m.group)) : null,
                ]));
            });
            if (pop) pop.place();
        }
        function mark(i) {
            sel = i;
            PH.$$('.ph-rb-isug__row', list).forEach(function (r, k) { r.classList.toggle('is-sel', k === i); });
            var r = list.children[i];
            if (r && r.scrollIntoView) r.scrollIntoView({ block: 'nearest' });
        }
        function choose(i) {
            var m = rows[i];
            if (!m) return;
            input.value = m.name;
            showPreview(m.name);
            commit(m.name);
            close();
        }
        input.addEventListener('focus', open);
        input.addEventListener('input', function () { showPreview(input.value.trim()); open(); });
        input.addEventListener('keydown', function (e) {
            if (e.key === 'ArrowDown') { e.preventDefault(); if (!pop) open(); else mark(Math.min(rows.length - 1, sel + 1)); }
            else if (e.key === 'ArrowUp') { e.preventDefault(); mark(Math.max(0, sel - 1)); }
            else if (e.key === 'Enter') {
                e.preventDefault();
                if (pop && rows[sel]) choose(sel);
                else { commit(input.value); close(); }
            } else if (e.key === 'Escape' && pop) { e.stopPropagation(); close(); }
            else if (e.key === 'Tab') close();
        });
        // Leaving the box keeps a real icon name (or an emptied box); anything else
        // half-typed goes back to the icon it had. Enter keeps whatever is typed.
        input.addEventListener('blur', function () {
            setTimeout(function () {
                if (document.activeElement === input) return;
                var typed = input.value.trim();
                if (!typed || !lib || lib.has(typed)) commit(typed);
                else { input.value = committed; showPreview(committed); }
                close();
            }, 150);
        });
        gridBtn.addEventListener('click', function () {
            close();
            gridPicker(gridBtn, lib, committed, function (name) { input.value = name; showPreview(name); commit(name); });
        });
        showPreview(opts.value);
        Promise.resolve(opts.lib).then(function (l) { lib = l; showPreview(input.value.trim()); });
        el.set = function (v) { committed = v || ''; input.value = committed; showPreview(committed); };
        el.disable = function (d) { input.readOnly = d; gridBtn.disabled = d; };
        return el;
    };

    /** The same box as a hub field (picker "icon"), for the table and the row drawer. */
    PH.Radial.iconControl = function (spec, commit) {
        var box = PH.Radial.iconCombo({
            value: spec.value, readonly: spec.readonly || S.isReadOnly(),
            lib: loadLib(libUrl(spec.meta), libGlobal(spec.meta)),
            onPick: function (name) { commit(name === undefined ? '' : name); },
        });
        return { el: box, set: function (v) { box.set(v); }, disable: function (d) { box.disable(d); } };
    };

    // ------------------------------------------------------- colour picker --
    // A colour square (saturation across, brightness down), a hue slider, the
    // hex, a few ready colours and Clear. Colours here are the owner's data.

    function hsvToHex(hh, ss, vv) {
        var f2 = function (n) {
            var k = (n + hh / 60) % 6;
            var c = vv - vv * ss * Math.max(0, Math.min(k, 4 - k, 1));
            return ('0' + Math.round(c * 255).toString(16)).slice(-2);
        };
        return '#' + f2(5) + f2(3) + f2(1);
    }
    function hexToHsv(hex) {
        var m = /^#?([0-9a-f]{6})$/i.exec(String(hex || '').trim());
        if (!m) return null;
        var n = parseInt(m[1], 16), r = (n >> 16 & 255) / 255, g = (n >> 8 & 255) / 255, b = (n & 255) / 255;
        var max = Math.max(r, g, b), min = Math.min(r, g, b), d = max - min, hh = 0;
        if (d) {
            if (max === r) hh = ((g - b) / d) % 6;
            else if (max === g) hh = (b - r) / d + 2;
            else hh = (r - g) / d + 4;
            hh = (hh * 60 + 360) % 360;
        }
        return { h: hh, s: max ? d / max : 0, v: max };
    }
    function normHex(v) {
        var s2 = String(v || '').trim().replace(/^#/, '');
        if (/^[0-9a-f]{3}$/i.test(s2)) s2 = s2.replace(/(.)/g, '$1$1');
        return /^[0-9a-f]{6}$/i.test(s2) ? '#' + s2.toLowerCase() : null;
    }
    var READY = ['#e0b060', '#c9a84c', '#b4514e', '#e0645a', '#7aa6bd', '#4f7fa8', '#7fa66a', '#55934a',
                 '#cc7459', '#a07cc0', '#f3e7d8', '#8a8078', '#3a2a22', '#2a3440', '#243a2a', '#141210'];

    /** { value, readonly, onPick(hex|undefined), emptyLabel } */
    PH.Radial.colorBox = function (opts) {
        var cur = normHex(opts.value);
        var sw = h('button.ph-rb-cbox__sw', { type: 'button', disabled: !!opts.readonly, title: 'Pick a colour' });
        var hex = h('input.ph-input.ph-input--mono.ph-rb-cbox__hex', { type: 'text', value: cur || '', placeholder: opts.emptyLabel || 'Theme', spellcheck: 'false', readOnly: !!opts.readonly });
        var clear = h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Back to the theme', disabled: !!opts.readonly }, icon('x'));
        var el = h('div.ph-rb-cbox', [sw, hex, clear]);
        function show(v) { cur = v; sw.style.background = v || ''; sw.classList.toggle('is-empty', !v); hex.value = v || ''; clear.style.display = v ? '' : 'none'; }
        function set(v) { show(v); opts.onPick(v || undefined); }
        hex.addEventListener('input', function () {
            var v = normHex(hex.value);
            hex.classList.toggle('is-bad', !!hex.value.trim() && !v);
            if (v) { cur = v; sw.style.background = v; sw.classList.remove('is-empty'); clear.style.display = ''; opts.onPick(v); }
        });
        hex.addEventListener('change', function () { if (!hex.value.trim()) set(undefined); });
        clear.addEventListener('click', function () { set(undefined); });
        sw.addEventListener('click', function () {
            var hsv = hexToHsv(cur) || { h: 30, s: 0.6, v: 0.8 };
            var sv = h('div.ph-rb-sv'), knob = h('i.ph-rb-sv__knob');
            sv.appendChild(knob);
            var hue = h('input.ph-rb-hue', { type: 'range', min: '0', max: '360', step: '1', value: String(Math.round(hsv.h)) });
            var out = h('span.ph-rb-cpop__hex');
            var readyRow = h('div.ph-rb-cpop__ready', READY.map(function (c) {
                return h('button.ph-rb-cpop__dot', { type: 'button', title: c, style: { background: c }, onclick: function () { hsv = hexToHsv(c); paint(true); } });
            }));
            var emit = PH.debounce(function () { set(hsvToHex(hsv.h, hsv.s, hsv.v)); }, 50);
            function paint(send) {
                sv.style.background = 'hsl(' + hsv.h + ', 100%, 50%)';
                knob.style.left = (hsv.s * 100) + '%';
                knob.style.top = ((1 - hsv.v) * 100) + '%';
                hue.value = String(Math.round(hsv.h));
                var c = hsvToHex(hsv.h, hsv.s, hsv.v);
                out.textContent = c;
                sw.style.background = c;
                if (send) emit();
            }
            function fromPointer(e) {
                var r = sv.getBoundingClientRect();
                hsv.s = Math.max(0, Math.min(1, (e.clientX - r.left) / r.width));
                hsv.v = 1 - Math.max(0, Math.min(1, (e.clientY - r.top) / r.height));
                paint(true);
            }
            sv.addEventListener('mousedown', function (e) {
                e.preventDefault();
                fromPointer(e);
                function mv(ev) { fromPointer(ev); }
                function up() { document.removeEventListener('mousemove', mv); document.removeEventListener('mouseup', up); }
                document.addEventListener('mousemove', mv);
                document.addEventListener('mouseup', up);
            });
            hue.addEventListener('input', function () { hsv.h = Number(hue.value); paint(true); });
            PH.Radial.busy = true;
            PH.popover(sw, h('div.ph-rb-cpop', [sv, hue, h('div.ph-rb-cpop__foot', [h('span.ph-rb-cpop__lab', 'Colour'), out]), readyRow]), {
                width: 250, maxHeight: 420,
                onClose: function () { PH.Radial.busy = false; if (live && live.el.isConnected) live.changed(); },
            });
            paint(false);
        });
        show(cur);
        return el;
    };

    // --------------------------------------------------------- the view --

    /** Does this list want the radial builder? */
    PH.Radial.wants = function (node) {
        var m = F.metaFor(node);
        return !!node && m.view === 'radial';
    };

    PH.Radial.section = function (node, opts) {
        opts = opts || {};
        var meta = F.metaFor(node);
        var rm = meta.radial || {};
        var fields = meta.fields || {};
        var itemLabel = meta.itemLabel || 'option';
        var storeKey = 'radial:' + ((S.cur && S.cur.id) || '') + ':' + node.path;
        var view = PH.store.get(storeKey + ':view', 'wheel');
        var st = {
            trail: [],          // 1-based indexes of the submenus we are inside
            sel: null,          // 1-based index on this ring
            hover: null,        // slot index under the pointer
            page: 0,
            mode: PH.store.get(storeKey + ':mode', 'all'),   // 'all' | 'player'
            ctx: PH.store.get(storeKey + ':ctx', { staff: false, dead: false, duty: true, job: '', grade: 0 }),
            look: PH.store.get(storeKey + ':look', false),
            cards: PH.store.get(storeKey + ':cards', {}),
            res: {},            // resource -> state, from the server
            cmds: null,         // every script's commands (loaded on first use)
            lib: null,
            slots: [],
        };

        var el = h('section.ph-rb', { dataset: { path: PH.canon(node.path) } });
        var folder0 = S.cur && S.cur.card && S.cur.card.folder;
        var libReady = loadLib(rm.icons && folder0 ? 'https://cfx-nui-' + folder0 + '/' + String(rm.icons).replace(/^\//, '') : libUrl(null), rm.iconsGlobal || libGlobal(null));

        if (view === 'table') {
            el.appendChild(viewBar());
            el.appendChild(PH.L.section(node, opts));
            return el;
        }

        var head = h('div.ph-rb-head');
        var lookEl = h('div.ph-rb-look');
        var notes = h('div.ph-rb-notes');
        var wheel = h('div.ph-rb-wheel', { tabindex: '0', 'aria-label': 'The menu, as players see it' });
        var svg = document.createElementNS(SVGNS, 'svg');
        svg.setAttribute('viewBox', '-500 -500 1000 1000');
        svg.setAttribute('class', 'ph-rb-ring');
        var gWedges = document.createElementNS(SVGNS, 'g');
        var gOrbit = document.createElementNS(SVGNS, 'g');
        svg.appendChild(gOrbit);
        svg.appendChild(gWedges);
        var itemsEl = h('div.ph-rb-items');
        var hub = h('button.ph-rb-hub', { type: 'button' });
        wheel.appendChild(svg);
        wheel.appendChild(itemsEl);
        wheel.appendChild(hub);
        var hint = h('div.ph-rb-hint');
        var stage = h('div.ph-rb-stage', [wheel, hint]);
        var editor = h('div.ph-rb-editor');

        // The wheel on the left (it stays in view while the side scrolls), and
        // on the right the sidebar: Wheel look, then the picked option's editor.
        var side = h('aside.ph-rb-side', [lookEl, editor]);
        el.appendChild(viewBar());
        el.appendChild(head);
        el.appendChild(notes);
        el.appendChild(h('div.ph-rb-body', [stage, side]));

        function ro() { return S.isReadOnly(); }
        function fmeta(key) { return fields[key] || {}; }
        function flabel(key, fallback) { return fmeta(key).label || fallback || PH.readable(key); }
        function ftip(key) { return fmeta(key).tooltip || ''; }
        function options(key) { return fmeta(key).options || []; }
        function setting(path, fallback) {
            if (!path) return fallback;
            var v = S.get(path);
            return v === undefined ? fallback : v;
        }
        function perRing() {
            var n = parseInt(setting(rm.perRing, 8), 10);
            return n >= 3 && n <= 12 ? n : 8;
        }
        function coloursOn() { return setting(rm.iconColours, true) !== false; }
        function builtinOn(name) {
            if (!rm.builtins) return true;
            var b = S.get(rm.builtins);
            return !(isObj(b) && b[name] === false);
        }

        // -------------------------------------------------------- paths --

        function ringPath(trail) {
            var p = node.path;
            trail.forEach(function (i) { p += '[' + i + '].items'; });
            return p;
        }
        function rowPath(trail, n) { return ringPath(trail) + '[' + n + ']'; }
        function ringItems(trail) {
            var v = S.get(ringPath(trail));
            return Array.isArray(v) ? v : [];
        }
        function rowAt(trail, n) { var v = S.get(rowPath(trail, n)); return isObj(v) ? v : null; }
        function selected() { return st.sel ? rowAt(st.trail, st.sel) : null; }

        /** The trail still points at submenus (a Discard or an undo can take them away). */
        function fixTrail() {
            var t = [];
            for (var i = 0; i < st.trail.length; i++) {
                var row = rowAt(t, st.trail[i]);
                if (!row || !kids(row)) break;
                t.push(st.trail[i]);
            }
            if (t.length !== st.trail.length) { st.trail = t; st.sel = null; st.page = 0; }
            if (st.sel && !rowAt(st.trail, st.sel)) st.sel = null;
        }

        /** The tone the submenus above pass down. */
        function trailTone() {
            var t = null, cur = [];
            st.trail.forEach(function (i) {
                var row = rowAt(cur, i);
                if (row && row.tone) t = row.tone;
                cur = cur.concat([i]);
            });
            return t;
        }

        function allIds() {
            var ids = {};
            (function walk(items, depth) {
                if (!Array.isArray(items) || depth > MAX_DEPTH) return;
                items.forEach(function (it) {
                    if (!isObj(it)) return;
                    if (it.id !== undefined && it.id !== '') ids[String(it.id)] = (ids[String(it.id)] || 0) + 1;
                    walk(kids(it), depth + 1);
                });
            })(S.get(node.path), 0);
            return ids;
        }
        function uniqueId(base) {
            var ids = allIds();
            base = slug(base);
            if (!ids[base]) return base;
            for (var i = 2; i < 999; i++) if (!ids[base + '_' + i]) return base + '_' + i;
            return base + '_' + Date.now();
        }

        // ------------------------------------------- who sees it (mirror) --
        // poggy_menu client/cl_context.lua and cl_tree.lua, on the owner's
        // screen. The server knows which resources run; the rest is the
        // "As a player" choices.

        function running(res) {
            var s = st.res[res];
            return s === undefined || s === 'started';
        }

        /** { show, locked, off: [why hidden for everyone], who: [who may see it] } */
        function judge(it, isSub, depth) {
            var out = { show: true, locked: null, off: [], who: [] };
            if (it.hidden === true) out.off.push('Hidden');
            list(it.resource).forEach(function (r) { if (!running(r)) out.off.push(r + ' is not running'); });
            if (it.builtin !== undefined && !isSub && !builtinOn(it.builtin)) out.off.push('The built-in is switched off');

            var when = it.when || (isSub ? 'any' : 'alive');
            if (when === 'dead') out.who.push('while down');
            var jobs = list(it.jobs);
            if (jobs.length) out.who.push(jobs.join(', ') + (it.minGrade ? ' (rank ' + it.minGrade + '+)' : ''));
            if (it.onDuty === true) out.who.push('on duty');
            if (it.admin === true) out.who.push('staff');
            list(it.ace).forEach(function (a) { out.who.push('ace ' + a); });

            if (out.off.length) { out.show = false; return out; }

            if (st.mode === 'player') {
                var c = st.ctx, reason = null;
                if (when === 'dead' && !c.dead) { out.show = false; return out; }
                if (when === 'alive' && c.dead) reason = 'Not while you are down.';
                if (!reason && jobs.length) {
                    var mine = lower(c.job || '');
                    var match = jobs.some(function (j) { return lower(String(j)) === mine; });
                    if (!match) reason = 'Only for: ' + jobs.join(', ') + '.';
                    else if (Number(it.minGrade) && (Number(c.grade) || 0) < Number(it.minGrade)) reason = 'Needs rank ' + it.minGrade + '.';
                }
                if (!reason && it.onDuty === true && !c.duty) reason = 'Only on duty.';
                if (!reason && (it.admin === true || list(it.ace).length) && !c.staff) reason = 'Staff only.';
                if (reason) {
                    if (it.showLocked !== true) { out.show = false; return out; }
                    out.locked = (typeof it.lockedReason === 'string' && it.lockedReason) || reason;
                }
            }

            if (isSub) {
                var inside = (kids(it) || []).filter(function (k) {
                    return isObj(k) && depth < MAX_DEPTH && judge(k, hasItems(k), depth + 1).show;
                });
                if (!inside.length) {
                    out.show = false;
                    out.off.push(kids(it).length ? 'Nothing inside shows' : 'Empty');
                }
                out.inside = inside.length;
            }
            return out;
        }

        // --------------------------------------------------------- data --

        function loadRemote() {
            var names = {};
            (function walk(items, depth) {
                if (!Array.isArray(items) || depth > MAX_DEPTH) return;
                items.forEach(function (it) {
                    if (!isObj(it)) return;
                    list(it.resource).forEach(function (r) { names[r] = true; });
                    if (isObj(it.export) && it.export.resource) names[it.export.resource] = true;
                    walk(kids(it), depth + 1);
                });
            })(S.get(node.path), 0);
            return PH.api('radial', Object.keys(names)).then(function (r) {
                if (!r.ok || !r.value) return;
                st.res = r.value.resources || {};
                st.cmds = Array.isArray(r.value.commands) ? r.value.commands : [];
                if (el.isConnected) drawAll();
            });
        }

        // -------------------------------------------------------- edits --

        /** Set one field of a row; empty takes the line out of the row. */
        function commit(trail, n, key, value) {
            if (ro()) return false;
            var rp = rowPath(trail, n);
            var row = S.get(rp);
            if (!isObj(row)) return false;
            var empty = value === undefined || value === null || value === '' || value === false ||
                (Array.isArray(value) && !value.length);
            if (empty) {
                if (!(key in row)) return true;
                var copy = PH.clone(row);
                delete copy[key];
                return S.queue({ op: 'set', path: rp, value: copy });
            }
            return S.set(rp + '.' + key, value);
        }
        function commitSel(key, value) { return st.sel ? commit(st.trail, st.sel, key, value) : false; }
        function setRow(trail, n, value) { return !ro() && S.queue({ op: 'set', path: rowPath(trail, n), value: value }); }

        function addOption(value) {
            if (ro()) return;
            var path = ringPath(st.trail);
            var len = ringItems(st.trail).length;
            if (S.queue({ op: 'insert', path: path, value: value })) {
                st.sel = len + 1;
                var per = perRing();
                st.page = len + 1 > per ? Math.floor(len / (per - 1)) : 0;
                drawAll();
            }
        }

        function newOption() {
            var v = meta.template !== undefined ? PH.clone(meta.template) : { label: 'New option', icon: 'star', command: '' };
            v.id = uniqueId(v.id || 'new_option');
            addOption(v);
        }
        function newSubmenu() {
            addOption({ id: uniqueId('new_menu'), label: 'New submenu', description: 'More options.', icon: 'grid', items: [] });
        }

        function fromCommand(c) {
            var cmd = String(c.command || '').replace(/^\//, '').split(/\s+/)[0];
            var ic = 'star';
            for (var i = 0; i < GUESS.length; i++) if (GUESS[i][0].test(cmd.toLowerCase())) { ic = GUESS[i][1]; break; }
            var desc = String(c.description || '').split(/(?<=\.)\s/)[0];
            if (desc.length > 110) desc = desc.slice(0, 107).replace(/\s+\S*$/, '') + '…';
            var v = {
                id: uniqueId(c.script + '_' + cmd),
                label: labelFor(c, cmd),
                description: desc || undefined,
                icon: ic,
                command: cmd,
                resource: c.folder,
            };
            if (staffOnly(c.who)) v.admin = true;
            if (!v.description) delete v.description;
            return v;
        }
        /** A name for an option made from a command: "Opens the admin panel: ..." gives
         *  "Admin panel"; a short command (/ahb) takes its script's name. */
        function labelFor(c, cmd) {
            var m = /^(?:Opens|Shows|Starts|Runs)\s+(?:the|a|an|your)\s+([^:.,(]{3,28})[:.,(]/i.exec(String(c.description || ''));
            var text = m ? m[1].trim() : cmd.length <= 5 && c.label ? c.label : PH.readable(cmd);
            return text.charAt(0).toUpperCase() + text.slice(1);
        }
        function staffOnly(who) { return /admin|staff|ace|moderat|console/i.test(String(who || '')); }
        function consoleOnly(who) { return /^\s*(server\s+)?console\s*$/i.test(String(who || '')); }

        function removeSel() {
            var row = selected();
            if (!row || ro()) return;
            var inside = kids(row);
            PH.confirm({
                title: 'Delete “' + (row.label || row.id || itemLabel) + '”?', danger: true, ok: 'Delete', okIcon: 'trash',
                body: h('div', [
                    inside && inside.length ? h('p.ph-warnline', [icon('alert'), 'It is a submenu: the ' + PH.plural(inside.length, 'option') + ' inside go with it.']) : null,
                    h('p', 'Nothing is written until you save, and Discard brings it back.'),
                ]),
            }).then(function (yes) {
                if (!yes) return;
                if (S.queue({ op: 'remove', path: ringPath(st.trail), index: st.sel })) { st.sel = null; drawAll(); }
            });
        }

        function duplicateSel() {
            var row = selected();
            if (!row || ro()) return;
            var n = st.sel;
            if (S.queue({ op: 'duplicate', path: ringPath(st.trail), index: n })) {
                st.sel = n + 1;
                var copy = rowAt(st.trail, n + 1);
                if (copy) {
                    S.set(rowPath(st.trail, n + 1) + '.id', uniqueId(String(row.id || row.label || 'option')));
                }
                drawAll();
            }
        }

        function moveSel(delta) {
            if (!st.sel || ro()) return;
            moveTo(st.sel, st.sel + delta);
        }
        function moveTo(from, to) {
            var len = ringItems(st.trail).length;
            if (to < 1 || to > len || from === to) return;
            if (S.queue({ op: 'move', path: ringPath(st.trail), from: from, to: to })) {
                st.sel = to;
                drawAll();
            }
        }

        /** Every submenu, for "Move to…": [{ trail, label, depth }]. */
        function submenus() {
            var out = [{ trail: [], label: 'Top of the menu', depth: 0 }];
            (function walk(trail, depth) {
                if (depth > MAX_DEPTH) return;
                ringItems(trail).forEach(function (it, i) {
                    if (!isObj(it) || !kids(it)) return;
                    var t = trail.concat([i + 1]);
                    out.push({ trail: t, label: it.label || it.id || 'Submenu', depth: depth + 1 });
                    walk(t, depth + 1);
                });
            })([], 0);
            return out;
        }
        function startsWith(a, b) { return b.length <= a.length && b.every(function (x, i) { return a[i] === x; }); }

        function moveElsewhere(anchor) {
            var row = selected();
            if (!row || ro()) return;
            var self = st.trail.concat([st.sel]);
            var here = st.trail.join('.');
            var items = submenus().map(function (m) {
                var blocked = startsWith(m.trail, self) || m.trail.join('.') === here;
                return {
                    label: new Array(m.depth + 1).join('   ') + m.label,
                    icon: m.depth ? 'grid' : 'home',
                    disabled: blocked,
                    onClick: function () {
                        // Insert first: the removal below cannot move what it points at.
                        var value = PH.clone(row);
                        var len = ringItems(m.trail).length;
                        if (!S.queue({ op: 'insert', path: ringPath(m.trail), value: value })) return;
                        if (!S.queue({ op: 'remove', path: ringPath(st.trail), index: st.sel })) return;
                        // The target's own path may have shifted down by one.
                        var t = m.trail.slice();
                        if (t.length > st.trail.length && startsWith(t, st.trail) && t[st.trail.length] > st.sel) t[st.trail.length]--;
                        st.trail = t;
                        st.sel = len + 1;
                        st.page = 0;
                        drawAll();
                    },
                };
            });
            PH.menu(anchor, items, { minWidth: 260 });
        }

        function setAction(kind) {
            var row = selected();
            if (!row || ro()) return;
            var was = actionOf(row);
            if (was === kind) return;
            var go = function () {
                var v = PH.clone(row);
                ACTION_KEYS.forEach(function (k) { delete v[k]; });
                if (kind === 'command') v.command = '';
                if (kind === 'event') v.event = '';
                if (kind === 'serverEvent') v.serverEvent = '';
                if (kind === 'export') v.export = { resource: '', name: '' };
                if (kind === 'builtin') v.builtin = (options('builtin')[0] || {}).value || 'showId';
                if (kind === 'submenu') v.items = [];
                setRow(st.trail, st.sel, v);
                drawAll();
            };
            var inside = kids(row);
            if (was === 'submenu' && inside && inside.length) {
                PH.confirm({
                    title: 'Turn this submenu into one option?', danger: true, ok: 'Turn it into an option',
                    body: 'The ' + PH.plural(inside.length, 'option') + ' inside it are deleted. Move them out first to keep them (Move to…).',
                }).then(function (yes) { if (yes) go(); });
                return;
            }
            go();
        }

        // ----------------------------------------------------- drawing --

        function svgEl(tag, attrs) {
            var e = document.createElementNS(SVGNS, tag);
            Object.keys(attrs || {}).forEach(function (k) { e.setAttribute(k, attrs[k]); });
            return e;
        }
        function rad(d) { return d * Math.PI / 180; }
        function pt(r, d) { return [r * Math.cos(rad(d)), r * Math.sin(rad(d))]; }
        function f(n) { return Math.round(n * 100) / 100; }

        // The shape (Config.Ui.Roundness), as poggy_menu ui/menu.js draws it: 1 round,
        // towards 0 each edge straightens until the ring is a polygon. Fewer than
        // four options stay round.
        // One polygon for every ring (poggy_menu ui/menu.js): as many sides as the
        // top ring has options (at least four), so a submenu of two is the top
        // ring's octagon cut in two.
        function sides() {
            var top = ringItems([]).filter(function (it) {
                return isObj(it) && (st.mode === 'player' ? judge(it, !!kids(it), 0).show : it.hidden !== true);
            }).length;
            return Math.max(4, Math.min(top, perRing()));
        }
        function shapeT() {
            var t = parseFloat(setting(rm.roundness, 100));
            if (!(t >= 0 && t <= 100)) t = 100;
            return t / 100;
        }
        function edgeR(base, theta, outer, t, S) {
            if (t >= 1) return base;
            var step = 360 / S, half = step / 2;
            var c = -90 + Math.round((theta + 90) / step) * step;
            var cosd = Math.cos(rad(theta - c)), ch = Math.cos(rad(half));
            var poly = outer ? Math.max(1, 430 / (R_OUT * ch)) * ch / cosd : 1 / cosd;
            return base * ((1 - t) * poly + t);
        }
        function edgePts(base, a0, a1, outer, t, S) {
            var step = 360 / S, lo = Math.min(a0, a1), hi = Math.max(a0, a1), marks = [];
            for (var k = Math.ceil((lo + 90) / step - 0.5); k <= Math.floor((hi + 90) / step - 0.5); k++) {
                var corner = -90 + (k + 0.5) * step;
                if (corner > lo && corner < hi) marks.push(corner);
            }
            if (a1 < a0) marks.reverse();
            marks = [a0].concat(marks, [a1]);
            var out = [];
            for (var m = 0; m < marks.length - 1; m++) {
                for (var j = (m ? 1 : 0); j <= 8; j++) {
                    var a = marks[m] + (marks[m + 1] - marks[m]) * j / 8;
                    out.push(pt(edgeR(base, a, outer, t, S), a));
                }
            }
            return out;
        }
        function polyline(pts, move) {
            return pts.map(function (q2, k) { return (k === 0 && move ? 'M ' : ' L ') + f(q2[0]) + ' ' + f(q2[1]); }).join('');
        }
        function outline(base, outer, t, S) { return polyline(edgePts(base, -90, 270, outer, t, S), true) + ' Z'; }

        function wedgePath(i, n) {
            var t = shapeT(), S = sides();
            if (t < 1 && n === 1) return outline(R_OUT, true, t, S) + ' ' + outline(R_IN, false, t, S);
            if (n > 1 && t < 1) {
                var sl = 360 / n, cc = -90 + i * sl, hh = sl / 2;
                var dO = Math.asin(Math.min(1, GAP / R_OUT)) * 180 / Math.PI, dI = Math.asin(Math.min(1, GAP / R_IN)) * 180 / Math.PI;
                return polyline(edgePts(R_OUT, cc - hh + dO, cc + hh - dO, true, t, S), true) +
                       polyline(edgePts(R_IN, cc + hh - dI, cc - hh + dI, false, t, S), false) + ' Z';
            }
            if (n === 1) {
                return 'M ' + R_OUT + ' 0 A ' + R_OUT + ' ' + R_OUT + ' 0 1 1 ' + (-R_OUT) + ' 0 A ' + R_OUT + ' ' + R_OUT + ' 0 1 1 ' + R_OUT + ' 0 Z ' +
                       'M ' + R_IN + ' 0 A ' + R_IN + ' ' + R_IN + ' 0 1 0 ' + (-R_IN) + ' 0 A ' + R_IN + ' ' + R_IN + ' 0 1 0 ' + R_IN + ' 0 Z';
            }
            var slice = 360 / n, c = -90 + i * slice, half = slice / 2;
            var dOut = Math.asin(GAP / R_OUT) * 180 / Math.PI, dIn = Math.asin(GAP / R_IN) * 180 / Math.PI;
            var o0 = pt(R_OUT, c - half + dOut), o1 = pt(R_OUT, c + half - dOut);
            var i1 = pt(R_IN, c + half - dIn), i0 = pt(R_IN, c - half + dIn);
            var large = (2 * half - 2 * dOut) > 180 ? 1 : 0, largeIn = (2 * half - 2 * dIn) > 180 ? 1 : 0;
            return 'M ' + f(o0[0]) + ' ' + f(o0[1]) +
                ' A ' + R_OUT + ' ' + R_OUT + ' 0 ' + large + ' 1 ' + f(o1[0]) + ' ' + f(o1[1]) +
                ' L ' + f(i1[0]) + ' ' + f(i1[1]) +
                ' A ' + R_IN + ' ' + R_IN + ' 0 ' + largeIn + ' 0 ' + f(i0[0]) + ' ' + f(i0[1]) + ' Z';
        }

        function iconEl(name, cls) { return iconNode(st.lib, name, cls); }
        function toneOf(it, inherited) {
            if (!coloursOn()) return null;
            var lib = st.lib;
            var t = it.tone || inherited;
            if (t && lib && lib.toneName) t = lib.toneName(t);
            if (!t && lib && lib.tone) t = lib.tone(it.icon);
            return t && t !== 'neutral' && t !== 'none' ? t : null;
        }

        /** The slots on this ring: every option (or only what the player sees), paged like the game. */
        function buildSlots() {
            var items = ringItems(st.trail);
            var all = [];
            items.forEach(function (it, i) {
                if (!isObj(it)) return;
                var sub = !!kids(it);
                var j = judge(it, sub, st.trail.length);
                if (st.mode === 'player' && !j.show) return;
                all.push({ n: i + 1, item: it, j: j, sub: sub });
            });
            var per = perRing();
            if (all.length <= per) { st.page = 0; return { slots: all, pages: 1, total: all.length }; }
            var perPage = per - 1, pages = Math.ceil(all.length / perPage);
            if (st.sel) {
                var at = all.findIndex(function (s) { return s.n === st.sel; });
                if (at >= 0 && Math.floor(at / perPage) !== st.page && !st.keepPage) st.page = Math.floor(at / perPage);
            }
            st.keepPage = false;
            if (st.page >= pages) st.page = pages - 1;
            var shown = all.slice(st.page * perPage, st.page * perPage + perPage);
            shown.push({ more: true });
            return { slots: shown, pages: pages, total: all.length };
        }

        function drawWheel() {
            fixTrail();
            var built = buildSlots();
            var slots = st.slots = built.slots;
            st.pages = built.pages;
            var n = slots.length;
            var inherited = trailTone();
            gWedges.textContent = '';
            gOrbit.textContent = '';
            PH.clear(itemsEl);
            wheel.style.fontSize = Math.max(8, (wheel.clientWidth || 520) / 38) + 'px';

            var gp = parseFloat(setting(rm.gap, 5));
            GAP = gp >= 0 && gp <= 20 ? gp : 5;
            var isz = parseFloat(setting(rm.iconSize, 100));
            wheel.style.setProperty('--rb-icon-scale', String(isz >= 60 && isz <= 150 ? isz / 100 : 1));
            var tNow = shapeT(), sNow = sides();
            var popN = parseFloat(setting(rm.hoverPop, 7));
            popN = popN >= 0 && popN <= 30 ? popN : 7;
            var growN = parseFloat(setting(rm.hoverGrow, 106));
            wheel.style.setProperty('--rb-grow', String(growN >= 100 && growN <= 130 ? growN / 100 : 1.06));
            wheel.classList.toggle('no-glow', setting(rm.hoverGlow, true) === false);
            var labs = setting(rm.labels, 'always');
            wheel.classList.toggle('labels-pointed', labs === 'pointed');
            wheel.classList.toggle('labels-never', labs === 'never');
            var lsz = parseFloat(setting(rm.labelSize, 100));
            wheel.style.setProperty('--rb-label-scale', String(lsz >= 70 && lsz <= 140 ? lsz / 100 : 1));
            var isFlat = n > 0 && tNow < 1;
            wheel.classList.toggle('is-flat', isFlat);
            gOrbit.appendChild(isFlat ? svgEl('path', { d: outline(R_OUT + 16, true, tNow, sNow), class: 'ph-rb-orbit' }) : svgEl('circle', { r: R_OUT + 16, class: 'ph-rb-orbit' }));
            if (isFlat) gOrbit.appendChild(svgEl('path', { d: outline(170, false, tNow, sNow), class: 'ph-rb-hubshape' }));

            if (!n) {
                itemsEl.appendChild(h('div.ph-rb-empty', st.mode === 'player'
                    ? 'This player sees nothing here.'
                    : 'Nothing here yet. Add the first ' + itemLabel + ' with the buttons above.'));
            }

            var slice = 360 / Math.max(1, n);
            slots.forEach(function (s, i) {
                var active = !s.more && s.n === st.sel;
                var hot = st.hover === i;
                var cls = ['ph-rb-wedge'];
                if (active) cls.push('is-sel');
                if (hot) cls.push('is-hot');
                if (s.more) cls.push('is-more');
                if (!s.more && !s.j.show) cls.push('is-off');
                if (!s.more && (s.j.locked || s.item.disabled === true)) cls.push('is-locked');
                if (st.drag && st.drag.over === i && st.drag.from !== s.n) cls.push('is-drop');
                if (st.drag && st.drag.from === s.n && st.drag.moving) cls.push('is-dragged');
                var it0 = s.item || {};
                var g = svgEl('g', { class: cls.join(' '), 'data-slot': String(i) });
                g.appendChild(svgEl('path', { d: wedgePath(i, n), class: 'ph-rb-wedge__base' }));
                if (n > 1) {
                    var c = -90 + i * slice, half = slice / 2 - Math.asin(Math.min(1, (GAP + 1) / R_OUT)) * 180 / Math.PI;
                    var a0 = pt(R_OUT - 4, c - half), a1 = pt(R_OUT - 4, c + half);
                    g.appendChild(svgEl('path', {
                        d: tNow < 1
                            ? polyline(edgePts(R_OUT - 4, c - half, c + half, true, tNow, sNow), true)
                            : 'M ' + f(a0[0]) + ' ' + f(a0[1]) + ' A ' + (R_OUT - 4) + ' ' + (R_OUT - 4) + ' 0 ' + (2 * half > 180 ? 1 : 0) + ' 1 ' + f(a1[0]) + ' ' + f(a1[1]),
                        class: 'ph-rb-wedge__rim',
                    }));
                }
                // The owner's own wedge colour (data, set as a variable).
                if (!s.more && typeof it0.color === 'string' && /^#?[0-9a-f]{3}([0-9a-f]{3})?$/i.test(it0.color)) {
                    g.classList.add('has-fill');
                    g.style.setProperty('--rb-fill', it0.color.charAt(0) === '#' ? it0.color : '#' + it0.color);
                }
                // The pointed-at (or picked) wedge steps out, as in game.
                if ((hot || (active && st.hover === null)) && n > 1 && popN) {
                    var pp = pt(popN, -90 + i * slice);
                    g.setAttribute('transform', 'translate(' + f(pp[0]) + ' ' + f(pp[1]) + ')');
                }
                gWedges.appendChild(g);

                var ang = n === 1 ? -90 : -90 + i * slice;
                var p = pt(n === 1 ? 0 : R_ITEM, ang);
                var it = s.item || {};
                var tone = s.more ? null : toneOf(it, inherited);
                var tag = null;
                if (!s.more && st.mode === 'all') {
                    if (s.j.off.length) tag = h('span.ph-rb-tag.is-off', s.j.off[0].indexOf('not running') >= 0 ? 'Not running' : s.j.off[0]);
                    else if (it.admin === true || list(it.ace).length) tag = h('span.ph-rb-tag', 'Staff');
                    else if (list(it.jobs).length) tag = h('span.ph-rb-tag', list(it.jobs).length === 1 ? String(list(it.jobs)[0]) : 'Jobs');
                    else if (it.when === 'dead') tag = h('span.ph-rb-tag', 'Down');
                }
                var ownColor = !s.more && typeof it.iconColor === 'string' && /^#?[0-9a-f]{3}([0-9a-f]{3})?$/i.test(it.iconColor)
                    ? (it.iconColor.charAt(0) === '#' ? it.iconColor : '#' + it.iconColor) : null;
                var fx = !s.more && /^(glow|pulse|bounce)$/.test(String(it.effect || '')) ? it.effect : null;
                var itemEl = h('div.ph-rb-item' + (tone ? '.tone-' + tone : '') + (ownColor ? '.has-color' : '') + (fx ? '.fx-' + fx : '') +
                    (active ? '.is-sel' : '') + (hot ? '.is-hot' : ''), {
                    style: { left: (50 + p[0] / 10) + '%', top: (50 + p[1] / 10) + '%' },
                }, [
                    h('span.ph-rb-item__icon', [
                        iconEl(s.more ? 'more' : (it.icon || (s.sub ? 'more' : 'dot'))),
                        it.badge !== undefined && it.badge !== false && it.badge !== '' && !s.more ? h('span.ph-rb-badge', String(it.badge).slice(0, 8)) : null,
                        s.j && s.j.locked ? h('span.ph-rb-lock', icon('lock')) : null,
                    ]),
                    h('span.ph-rb-item__label', s.more ? 'More' : String(it.label || it.id || '?')),
                    tag,
                ]);
                if (ownColor) itemEl.style.setProperty('--rb-ico', ownColor);
                itemsEl.appendChild(itemEl);

                if (s.sub && !s.more && n > 1) {
                    var sp = pt(R_SUB, ang);
                    itemsEl.appendChild(h('span.ph-rb-sub' + (active ? '.is-sel' : ''), {
                        style: { left: (50 + sp[0] / 10) + '%', top: (50 + sp[1] / 10) + '%', transform: 'translate(-50%, -50%) rotate(' + (ang + 90) + 'deg)' },
                    }, icon('up')));
                }
                if (n > 1) {
                    var kp = pt(R_KEY, ang);
                    itemsEl.appendChild(h('span.ph-rb-key' + (active ? '.is-sel' : ''), {
                        style: { left: (50 + kp[0] / 10) + '%', top: (50 + kp[1] / 10) + '%' },
                    }, String(i + 1)));
                }
            });
            st.built = built;
            drawHub(built);
            drawHead(built);
        }

        function drawHub(built) {
            PH.clear(hub);
            var s = st.hover !== null ? st.slots[st.hover] : null;
            if (!s && st.sel) s = st.slots.filter(function (x) { return x.n === st.sel; })[0] || null;
            hub.classList.toggle('is-back', !!st.trail.length);
            if (st.trail.length) hub.appendChild(h('span.ph-rb-hub__back', [icon('left'), 'Back']));
            var crumb = crumbs().slice(1).map(function (c) { return c.label; }).join(' › ');
            if (s && s.more) {
                hub.appendChild(h('span.ph-rb-hub__title', 'More'));
                hub.appendChild(h('span.ph-rb-hub__desc', 'Page ' + (st.page + 1) + ' of ' + built.pages + '. Click to turn.'));
            } else if (s) {
                if (crumb) hub.appendChild(h('span.ph-rb-hub__crumb', crumb));
                hub.appendChild(h('span.ph-rb-hub__title', String(s.item.label || s.item.id || '?')));
                if (s.item.description) hub.appendChild(h('span.ph-rb-hub__desc', String(s.item.description)));
                var note = null;
                if (s.item.disabled === true) note = h('span.ph-rb-hub__note.is-reason', s.item.disabledReason || 'Not available.');
                else if (s.j.locked) note = h('span.ph-rb-hub__note.is-reason', s.j.locked);
                else if (s.item.confirm) note = h('span.ph-rb-hub__note.is-confirm', typeof s.item.confirm === 'string' ? s.item.confirm : 'Pick again to confirm.');
                else if (!s.j.show && st.mode === 'all') note = h('span.ph-rb-hub__note.is-off', 'Players do not see it: ' + s.j.off.join(', ').toLowerCase() + '.');
                if (note) hub.appendChild(note);
            } else {
                if (crumb) hub.appendChild(h('span.ph-rb-hub__crumb', crumb));
                hub.appendChild(h('span.ph-rb-hub__title', st.trail.length ? crumbs().slice(-1)[0].label : (opts.label || meta.label || 'Menu')));
                hub.appendChild(h('span.ph-rb-hub__desc', built.total ? PH.plural(built.total, itemLabel) + (st.mode === 'player' ? ' this player sees' : '') : ''));
            }
            if (built.pages > 1) {
                var dots = h('span.ph-rb-hub__pages');
                for (var i = 0; i < built.pages; i++) dots.appendChild(h('i' + (i === st.page ? '.is-on' : '')));
                hub.appendChild(dots);
            }
            hint.textContent = ro()
                ? 'Read only: click around to look. Take the edit lock to change it.'
                : st.trail.length
                    ? 'Click an option to edit it on the right · click a submenu again to open it · drag to swap · the middle goes back'
                    : 'Click an option to edit it on the right · click a submenu again to open it · drag one onto another to swap them';
        }

        function crumbs() {
            var out = [{ trail: [], label: opts.label || meta.label || 'Menu' }];
            var t = [];
            st.trail.forEach(function (i) {
                var row = rowAt(t, i);
                t = t.concat([i]);
                out.push({ trail: t.slice(), label: row ? String(row.label || row.id || 'Submenu') : '?' });
            });
            return out;
        }

        function go(trail, sel) {
            st.trail = trail.slice();
            st.sel = sel || null;
            st.hover = null;
            st.page = 0;
            drawAll();
        }

        function viewBar() {
            var seg = h('div.ph-seg');
            [['wheel', 'Wheel'], ['table', 'Table']].forEach(function (v) {
                seg.appendChild(h('button.ph-seg__btn' + (view === v[0] ? '.is-on' : ''), {
                    type: 'button',
                    onclick: function () {
                        if (view === v[0]) return;
                        PH.store.set(storeKey + ':view', v[0]);
                        var fresh = PH.Radial.section(node, opts);
                        if (el.parentNode) el.parentNode.replaceChild(fresh, el);
                    },
                }, v[1]));
            });
            return h('div.ph-rb-viewbar', [
                h('div.ph-rb-viewbar__title', [icon('target'), h('span', opts.label || meta.label || 'Menu')]),
                h('span.ph-rb-viewbar__sub', view === 'wheel' ? 'Click the wheel to edit. Save when you are done, then restart the script.' : 'Every option as a table.'),
                seg,
            ]);
        }

        function drawHead(built) {
            PH.clear(head);
            var cr = h('nav.ph-rb-crumbs');
            crumbs().forEach(function (c, i, all) {
                if (i) cr.appendChild(h('span.ph-rb-crumbs__sep', '›'));
                var last = i === all.length - 1;
                cr.appendChild(h('button.ph-rb-crumbs__btn' + (last ? '.is-on' : ''), {
                    type: 'button', disabled: last,
                    onclick: function () { go(c.trail, st.trail[c.trail.length] || null); },
                }, [i ? icon('grid') : icon('home'), c.label]));
            });

            var mode = h('div.ph-seg');
            [['all', 'All options'], ['player', 'As a player']].forEach(function (m) {
                mode.appendChild(h('button.ph-seg__btn' + (st.mode === m[0] ? '.is-on' : ''), {
                    type: 'button',
                    onclick: function () { st.mode = m[0]; PH.store.set(storeKey + ':mode', m[0]); st.hover = null; drawAll(); },
                }, m[1]));
            });
            PH.tip(mode, 'All options: everything in the file, with what players would not see marked. As a player: only what a player like the one you describe sees, as they see it.');

            var add = ro() ? null : h('div.ph-rb-add', [
                h('button.ph-btn.ph-btn--primary.ph-btn--sm', { type: 'button', onclick: newOption }, [icon('plus'), 'Option']),
                h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: newSubmenu }, [icon('grid'), 'Submenu']),
                h('button.ph-btn.ph-btn--ghost.ph-btn--sm', {
                    type: 'button', title: 'Pick a command from any script on this server',
                    onclick: function (e) { pickCommand(e.currentTarget, function (c) { addOption(fromCommand(c)); }); },
                }, [icon('terminal'), 'From a script…']),
            ]);

            var lookBtn = h('button.ph-btn.ph-btn--ghost.ph-btn--sm' + (st.look ? '.is-on' : ''), {
                type: 'button', title: 'Shape, gap, icon size, wheel size',
                onclick: function () { st.look = !st.look; PH.store.set(storeKey + ':look', st.look); drawLook(); drawHead(st.built || { pages: 1, total: 0 }); },
            }, [icon('palette'), 'Wheel look']);
            head.appendChild(h('div.ph-rb-head__row', [cr, h('div.ph-rb-head__side', [mode, lookBtn, add])]));
            if (st.mode === 'player') head.appendChild(playerBar());

            // Notes about the ring and the whole menu.
            PH.clear(notes);
            var per = perRing();
            if (st.mode === 'all') {
                var shows = ringItems(st.trail).filter(function (it) { return isObj(it) && it.hidden !== true; }).length;
                if (shows > per) notes.appendChild(h('div.ph-rb-note', [icon('alert'), PH.plural(shows, itemLabel) + ' on this ring, and a ring holds ' + per + ': players see ' + (per - 1) + ' at a time and a More wedge. Put some in a submenu.']));
            }
            if (st.mode === 'player' && st.trail.length) {
                var up = above();
                if (up.blocked) notes.appendChild(h('div.ph-rb-note', [icon('lock'), 'This player cannot open “' + up.blocked + '”: it is shown here because you opened it in All options.']));
            }
            var ids = allIds(), dup = Object.keys(ids).filter(function (k) { return ids[k] > 1; });
            if (dup.length) notes.appendChild(h('div.ph-rb-note.is-bad', [icon('alert'), 'The same ID is used twice: ' + dup.join(', ') + '. Other scripts find options by ID, so give each its own.']));
        }

        /** Sliders for the whole wheel's look. Built on its own (not with the head), so a
         *  slider being dragged is never replaced under the pointer. */
        function drawLook() {
            PH.clear(lookEl);
            lookEl.hidden = !st.look;
            if (!st.look) return;
            lookEl.appendChild(h('div.ph-rb-look__head', [
                icon('palette'), h('span', 'Wheel look'),
                h('button.ph-iconbtn.ph-iconbtn--sm', {
                    type: 'button', title: 'Close',
                    onclick: function () { st.look = false; PH.store.set(storeKey + ':look', false); drawLook(); drawHead(st.built || { pages: 1, total: 0 }); },
                }, icon('x')),
            ]));
            var rows = [
                { path: rm.roundness, label: 'Shape', def: 100, min: 0, max: 100, step: 5, unit: '%', ends: ['Flat sides', 'Round'],
                  tip: '100 is round. Lower flattens each wedge until the wheel is a polygon with straight sides. Fewer than four options stay round.' },
                { path: rm.gap, label: 'Gap', def: 5, min: 0, max: 20, step: 1, unit: '', ends: ['Joined', 'Wide'], tip: 'The space between two wedges.' },
                { path: rm.iconSize, label: 'Icon size', def: 100, min: 60, max: 150, step: 5, unit: '%', ends: ['Small', 'Large'], tip: 'How big the icons are on the wheel.' },
                { path: rm.size, label: 'Wheel size', def: 76, min: 40, max: 95, step: 1, unit: '% of the screen', ends: ['Small', 'Large'], tip: 'How big the wheel is in game, as a share of the screen height. The picture here keeps its size.' },
                { path: rm.perRing, label: 'Options per ring', def: 8, min: 3, max: 12, step: 1, unit: '', ends: ['3', '12'], tip: 'The most options on one ring before the last wedge becomes More.' },
                { path: rm.hoverPop, label: 'Pointed: pop out', def: 7, min: 0, max: 30, step: 1, unit: '', ends: ['Still', 'Far'], tip: 'How far the wedge under the pointer steps out.' },
                { path: rm.hoverGrow, label: 'Pointed: icon grows', def: 106, min: 100, max: 130, step: 1, unit: '%', ends: ['Same', 'Big'], tip: 'How much the icon under the pointer grows.' },
                { path: rm.hoverGlow, label: 'Pointed: glow', kind: 'switch', tip: 'The wedge under the pointer lights up. Off: only its accent edge shows it.' },
                { path: rm.labels, label: 'Names', kind: 'choice', choices: [['always', 'Always'], ['pointed', 'Only the pointed one'], ['never', 'Icons only']],
                  tip: 'The names under the icons. Icons only: the middle of the wheel still names the option pointed at.' },
                { path: rm.labelSize, label: 'Name size', def: 100, min: 70, max: 140, step: 5, unit: '%', ends: ['Small', 'Large'], tip: 'How big the names under the icons are.' },
            ].filter(function (r) { return !!r.path; });
            rows.forEach(function (r) {
                if (r.kind === 'switch' || r.kind === 'choice') { lookEl.appendChild(lookChoice(r)); return; }
                var node2 = S.node(r.path);
                var v = Number(setting(r.path, r.def !== undefined ? r.def : r.min));
                var out = h('span.ph-rb-look__val', String(v) + (r.unit ? (r.unit.charAt(0) === '%' ? '' : ' ') + r.unit : ''));
                var range = h('input.ph-rb-look__range', { type: 'range', min: String(r.min), max: String(r.max), step: String(r.step), value: String(v), disabled: ro() || !node2 });
                range.addEventListener('input', function () {
                    var n2 = Number(range.value);
                    out.textContent = String(n2) + (r.unit ? (r.unit.charAt(0) === '%' ? '' : ' ') + r.unit : '');
                    S.set(r.path, n2);
                });
                var lab = h('span.ph-rb-look__lab', r.label);
                PH.tip(lab, r.tip);
                lookEl.appendChild(h('div.ph-rb-look__row', [
                    lab, out, range,
                    node2 ? null : h('span.ph-rb-look__miss', 'Not in this config.lua yet: it arrives with the update.'),
                ]));
            });
            lookEl.appendChild(h('p.ph-rb-look__note', 'The wheel follows these as you drag. Colours, effects and the icon of one option are in its editor: click it on the wheel.'));
        }

        /** A Wheel look row that is a switch (on / off) or a choice of words. */
        function lookChoice(r) {
            var node2 = S.node(r.path);
            var cur = setting(r.path, r.kind === 'switch' ? true : r.choices[0][0]);
            var choices = r.kind === 'switch' ? [[true, 'On'], [false, 'Off']] : r.choices;
            var seg = h('div.ph-seg.ph-rb-seg');
            choices.forEach(function (c) {
                seg.appendChild(h('button.ph-seg__btn' + (cur === c[0] ? '.is-on' : ''), {
                    type: 'button', disabled: ro() || !node2,
                    onclick: function () { S.set(r.path, c[0]); drawLook(); },
                }, c[1]));
            });
            var lab = h('span.ph-rb-look__lab', r.label);
            PH.tip(lab, r.tip);
            return h('div.ph-rb-look__row.is-choice', [lab, seg,
                node2 ? null : h('span.ph-rb-look__miss', 'Not in this config.lua yet: it arrives with the update.')]);
        }

        function playerBar() {
            var c = st.ctx;
            function save() { PH.store.set(storeKey + ':ctx', c); drawAll(); }
            function chip(key, label, tip) {
                var b = h('button.ph-rb-chip' + (c[key] ? '.is-on' : ''), { type: 'button', onclick: function () { c[key] = !c[key]; save(); } },
                    [icon(c[key] ? 'check' : 'x'), label]);
                PH.tip(b, tip);
                return b;
            }
            var jobBtn = h('button.ph-rb-chip.is-job', {
                type: 'button',
                onclick: function (e) {
                    F.pick({
                        anchor: e.currentTarget, fetch: function (qq) {
                            return F.fetchers.job(qq).then(function (rows) { return [{ value: '', label: 'No job', icon: 'x' }].concat(rows); });
                        },
                        allowFree: true, freeLabel: 'Use job', placeholder: 'Search jobs…',
                        onPick: function (v) { c.job = v || ''; save(); },
                    });
                },
            }, [icon('badge'), c.job ? 'Job: ' + c.job : 'No job']);
            var grade = h('input.ph-input.ph-input--num.ph-rb-grade', { type: 'number', min: '0', max: '99', value: String(c.grade || 0), title: 'Their rank (job grade)' });
            grade.addEventListener('change', function () { c.grade = Math.max(0, parseInt(grade.value, 10) || 0); save(); });
            return h('div.ph-rb-player', [
                h('span.ph-rb-player__lead', 'A player who is'),
                chip('staff', 'Staff', 'A framework admin, or has the admin ace' + (rm.adminAce && S.get(rm.adminAce) ? ' (' + S.get(rm.adminAce) + ')' : '') + '. Also stands in for any ace an option names.'),
                chip('dead', 'Down', 'Dead or dying: only options for "Down" or "Any" show.'),
                chip('duty', 'On duty', 'Options marked "On duty only" need it.'),
                jobBtn,
                h('label.ph-rb-player__grade', ['rank', grade]),
            ]);
        }

        function pickCommand(anchor, done) {
            var ready = st.cmds ? Promise.resolve() : loadRemote();
            ready.then(function () {
                F.pick({
                    anchor: anchor, width: 460, placeholder: 'Search commands, scripts, what they do…',
                    emptyText: 'No script on this server lists a command.',
                    fetch: function (qq) {
                        return (st.cmds || []).filter(function (c) {
                            return !consoleOnly(c.who) && PH.matches(qq, [c.command, c.label, c.description, c.who]);
                        }).map(function (c) {
                            return {
                                value: c, label: c.command,
                                sub: c.label + (c.who ? ' · ' + c.who : '') + (c.description ? ' · ' + c.description : ''),
                                icon: staffOnly(c.who) ? 'shield' : 'terminal',
                                badge: c.running ? null : 'Stopped',
                            };
                        });
                    },
                    onPick: function (c) { done(c); },
                });
            });
        }

        // ------------------------------------------------------ editor --

        function field(label, tip, control, opts2) {
            opts2 = opts2 || {};
            var lab = h('label.ph-rb-f__label', label);
            var top = h('div.ph-rb-f__top', [lab]);
            if (tip) {
                var info = h('button.ph-info', { type: 'button', 'aria-label': 'About ' + label }, icon('info'));
                PH.tip(info, tip);
                top.appendChild(info);
            }
            return h('div.ph-rb-f' + (opts2.wide ? '.is-wide' : ''), [top, control, opts2.note ? h('div.ph-rb-f__note', opts2.note) : null]);
        }

        function textIn(key, opts2) {
            opts2 = opts2 || {};
            var row = selected() || {};
            var cur = opts2.get ? opts2.get(row) : row[key];
            var input = h('input.ph-input' + (opts2.mono ? '.ph-input--mono' : ''), {
                type: 'text', value: cur === undefined || cur === null ? '' : String(cur),
                placeholder: opts2.placeholder || fmeta(key).placeholder || '', spellcheck: 'false', readOnly: ro(),
                dataset: { rbKey: key },
            });
            input.addEventListener('input', function () {
                var v = input.value;
                if (opts2.set) opts2.set(v);
                else commitSel(key, opts2.keep ? v : (v.trim() === '' ? undefined : v));
            });
            return input;
        }

        function numIn(key, opts2) {
            opts2 = opts2 || {};
            var row = selected() || {};
            var m = fmeta(key);
            var input = h('input.ph-input.ph-input--num', {
                type: 'number', value: row[key] === undefined ? '' : String(row[key]),
                min: m.min !== undefined ? String(m.min) : '0', max: m.max !== undefined ? String(m.max) : null,
                step: m.step !== undefined ? String(m.step) : '1', placeholder: opts2.placeholder || '0', readOnly: ro(),
            });
            input.addEventListener('input', function () {
                var n = parseFloat(input.value);
                commitSel(key, isNaN(n) || n <= 0 ? undefined : n);
            });
            return m.unit || opts2.unit ? h('div.ph-rb-unit', [input, h('span', m.unit || opts2.unit)]) : input;
        }

        function toggleIn(key, onFlip) {
            var row = selected() || {};
            var on = row[key] === true;
            var btn = h('button.ph-toggle' + (on ? '.is-on' : ''), { type: 'button', role: 'switch', disabled: ro(), 'aria-checked': on ? 'true' : 'false' },
                [h('span.ph-toggle__track', h('span.ph-toggle__knob')), h('span.ph-toggle__text', on ? 'On' : 'Off')]);
            btn.addEventListener('click', function () {
                commitSel(key, !on ? true : undefined);
                if (onFlip) onFlip(!on);
                drawEditor();
            });
            return btn;
        }

        function segIn(key, choices, fallback) {
            var row = selected() || {};
            var cur = row[key] === undefined ? fallback : row[key];
            var seg = h('div.ph-seg.ph-rb-seg');
            choices.forEach(function (c) {
                var on = cur === undefined ? c.value === undefined : String(cur) === String(c.value);
                seg.appendChild(h('button.ph-seg__btn' + (on ? '.is-on' : ''), {
                    type: 'button', disabled: ro(),
                    onclick: function () { commitSel(key, c.value === fallback ? undefined : c.value); drawEditor(); },
                }, c.label));
            });
            return seg;
        }

        function codeLine(row) {
            var a = actionOf(row);
            var say, code = null;
            if (a === 'command') {
                var cmd = String(row.command || '').replace(/^\s*\//, '');
                if (!cmd) return { say: 'Type the command. Nothing runs until you do.', bad: true };
                say = 'Runs /' + cmd + ' as if the player typed it. The command checks its own permission.';
                code = 'ExecuteCommand(' + q(cmd) + ')';
            } else if (a === 'export') {
                var ex = isObj(row.export) ? row.export : {};
                if (!ex.resource || !ex.name) return { say: 'Name the script (resource) and its export.', bad: true };
                var args = isObj(row.export) && row.export.args !== undefined ? row.export.args : row.args;
                say = 'Calls the client export ' + ex.name + ' of ' + ex.resource + '.';
                code = 'exports[' + q(ex.resource) + ']:' + ex.name + '(' + argsCode(args) + ')';
            } else if (a === 'event') {
                if (!row.event) return { say: 'Type the event name.', bad: true };
                say = 'Fires this event on the player\'s own game.';
                code = 'TriggerEvent(' + [q(row.event)].concat(argsCode(row.args) ? [argsCode(row.args)] : []).join(', ') + ')';
            } else if (a === 'serverEvent') {
                if (!row.serverEvent) return { say: 'Type the event name.', bad: true };
                say = 'Sends this event to the server. The script that listens must check who sent it.';
                code = 'TriggerServerEvent(' + [q(row.serverEvent)].concat(argsCode(row.args) ? [argsCode(row.args)] : []).join(', ') + ')';
            } else if (a === 'builtin') {
                var o = options('builtin').filter(function (x) { return x.value === row.builtin; })[0];
                say = (o ? o.label : row.builtin) + ': the menu does it itself, on every framework.' + (builtinOn(row.builtin) ? '' : ' It is switched off on the Built-ins tab, so this option is hidden.');
            } else if (a === 'submenu') {
                var n = (kids(row) || []).length;
                say = n ? 'Opens ' + PH.plural(n, itemLabel) + '.' : 'An empty submenu is left out of the menu. Open it and add options.';
                return { say: say, bad: !n };
            } else {
                return { say: 'It does nothing yet: players see it greyed out. Pick what it does above.', bad: true };
            }
            return { say: say, code: code };
        }

        /** The submenus above this ring: what each asks of the player, and whether this player gets in. */
        function above() {
            var out = { who: [], off: [], blocked: null }, t = [];
            st.trail.forEach(function (i) {
                var row = rowAt(t, i);
                if (row) {
                    var j = judge(row, true, t.length);
                    var name = String(row.label || row.id || 'the submenu');
                    j.who.forEach(function (w) { out.who.push(w + ' (' + name + ')'); });
                    j.off.forEach(function (w) { out.off.push(w + ' (' + name + ')'); });
                    if (!j.show && !out.blocked) out.blocked = name;
                }
                t = t.concat([i]);
            });
            return out;
        }

        function whoLine(row, sub) {
            var j = judge(row, sub, st.trail.length);
            var up = above();
            var off = j.off.concat(up.off.filter(function (w) { return !/^(Nothing inside shows|Empty)/.test(w); }));
            if (off.length) return { text: 'Nobody right now: ' + off.join(', ').toLowerCase() + '.', bad: true };
            var who = j.who.length ? j.who.join(' · ') : (row.when === 'any' ? 'everyone, alive or down' : sub ? 'everyone' : 'everyone, while alive');
            var text = who.charAt(0).toUpperCase() + who.slice(1) + (row.showLocked === true ? '; the rest see it greyed out' : '') + '.';
            if (up.who.length) text += ' And, from the submenus above: ' + up.who.join(' · ') + '.';
            return { text: text };
        }

        function drawEditor() {
            var focusKey = document.activeElement && editor.contains(document.activeElement) ? document.activeElement.dataset.rbKey : null;
            PH.clear(editor);
            var row = selected();
            if (!row) {
                editor.appendChild(h('div.ph-rb-blank', [
                    icon('target', 'ph-rb-blank__ico'),
                    h('p', 'Click an option on the wheel to edit it here.'),
                    h('p.ph-rb-blank__sub', 'Wheel look (at the top) changes the whole wheel.'),
                    h('p.ph-rb-blank__sub', 'What it is called, its icon, what it runs, and who sees it.'),
                ]));
                return;
            }
            var a = actionOf(row);
            var sub = a === 'submenu';
            var inherited = trailTone();
            var tone = toneOf(row, inherited);

            // Header: the option as it looks, and what can be done to it.
            var len = ringItems(st.trail).length;
            var tools = ro() ? null : h('div.ph-rb-ed__tools', [
                h('button.ph-iconbtn', { type: 'button', title: 'Earlier (anticlockwise)', disabled: st.sel <= 1, onclick: function () { moveSel(-1); } }, icon('left')),
                h('button.ph-iconbtn', { type: 'button', title: 'Later (clockwise)', disabled: st.sel >= len, onclick: function () { moveSel(1); } }, icon('right')),
                h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function (e) { moveElsewhere(e.currentTarget); } }, [icon('move'), 'Move to…']),
                h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: duplicateSel }, [icon('copy'), 'Duplicate']),
                h('button.ph-btn.ph-btn--ghost.ph-btn--sm.ph-btn--danger-text', { type: 'button', onclick: removeSel }, [icon('trash'), 'Delete']),
            ]);
            var edIcon = h('span.ph-rb-ed__icon' + (tone ? '.tone-' + tone : '') + (row.iconColor ? '.has-color' : ''), iconEl(row.icon || (sub ? 'more' : 'dot')));
            if (row.iconColor) edIcon.style.setProperty('--rb-ico', String(row.iconColor));
            editor.appendChild(h('div.ph-rb-ed__head', [
                edIcon,
                h('div.ph-rb-ed__title', [
                    h('b', String(row.label || row.id || 'Untitled')),
                    h('span', [
                        (sub ? 'Submenu' : 'Option') + ' ' + st.sel + ' of ' + len,
                        row.id !== undefined ? h('code', String(row.id)) : null,
                    ]),
                ]),
                sub ? h('button.ph-btn.ph-btn--primary.ph-btn--sm', { type: 'button', onclick: function () { go(st.trail.concat([st.sel])); } }, [icon('open'), 'Open submenu']) : null,
                tools,
            ]));

            // Problems with this one option.
            var probs = [];
            if (row.id === undefined || row.id === '') probs.push('It has no ID. Give it one so other scripts and cooldowns can find it.');
            if (st.lib && row.icon && !st.lib.has(row.icon)) probs.push('“' + row.icon + '” is not in the icon library: it shows a dot.');
            if (row.minGrade && !list(row.jobs).length) probs.push('A lowest rank without jobs does nothing.');
            if (sub && ['command', 'event', 'serverEvent', 'export', 'builtin'].some(function (k) { return row[k] !== undefined; })) probs.push('A submenu with options inside ignores its own action.');
            if (probs.length) editor.appendChild(h('div.ph-rb-probs', probs.map(function (p) { return h('div.ph-rb-note.is-bad', [icon('alert'), p]); })));

            var grid = h('div.ph-rb-ed__grid');
            editor.appendChild(grid);

            // ── How it looks
            var toneSeg = h('div.ph-rb-tones');
            [''].concat(options('tone').map(function (o) { return o.value; })).forEach(function (t) {
                var on = (row.tone || '') === t;
                toneSeg.appendChild(h('button.ph-rb-tone' + (t ? '.tone-' + t : '') + (on ? '.is-on' : ''), {
                    type: 'button', disabled: ro(), title: t ? ((options('tone').filter(function (o) { return o.value === t; })[0] || {}).label || t) : 'The icon\'s own colour' + (inherited ? ', or the submenu\'s' : ''),
                    onclick: function () { commitSel('tone', t || undefined); drawEditor(); },
                }, [h('i'), TONE_LABELS[t] || PH.readable(t)]));
            });
            var iconBtn = PH.Radial.iconCombo({
                value: row.icon, readonly: ro(), lib: libReady,
                onPick: function (name) {
                    commitSel('icon', name);
                    var hi = editor.querySelector('.ph-rb-ed__icon');
                    if (hi) { PH.clear(hi); hi.appendChild(iconEl(name || 'dot')); }
                },
            });
            var effectSeg = segIn('effect', [{ value: undefined, label: 'None' }].concat(options('effect').length ? options('effect') : [
                { value: 'glow', label: 'Glow' }, { value: 'pulse', label: 'Pulse' }, { value: 'bounce', label: 'Bounce' }]), undefined);
            var iconColorBox = PH.Radial.colorBox({ value: row.iconColor, readonly: ro(), emptyLabel: 'None', onPick: function (v) { commitSel('iconColor', v); } });
            var wedgeColorBox = PH.Radial.colorBox({ value: row.color, readonly: ro(), emptyLabel: 'Theme', onPick: function (v) { commitSel('color', v); } });
            var labelIn = textIn('label', { keep: true });
            labelIn.dataset.rbKey = 'label';
            var idIn = textIn('id', { mono: true, placeholder: 'call_doctor' });
            idIn.dataset.rbKey = 'id';
            idIn.addEventListener('change', function () {
                var v = slug(idIn.value);
                if (idIn.value && v !== idIn.value) { idIn.value = v; commitSel('id', v); }
            });
            var descIn = textIn('description');
            descIn.dataset.rbKey = 'description';
            var lookSum = [row.icon || 'no icon', row.tone || row.iconColor ? (row.iconColor || row.tone) : null, row.effect || null].filter(Boolean).join(' · ');
            grid.appendChild(card('Looks', 'palette', [
                field(flabel('label', 'Name'), ftip('label'), labelIn),
                field(flabel('id', 'ID'), ftip('id'), idIn),
                field(flabel('description', 'Description'), ftip('description'), descIn, { wide: true }),
                field(flabel('icon', 'Icon'), ftip('icon'), iconBtn),
                field(flabel('badge', 'Badge'), ftip('badge'), textIn('badge', { placeholder: 'NEW' })),
                field(flabel('tone', 'Icon colour'), ftip('tone'), toneSeg, { wide: true }),
                field(flabel('iconColor', 'Own icon colour'), ftip('iconColor'), iconColorBox),
                field(flabel('color', 'Wedge colour'), ftip('color'), wedgeColorBox),
                field(flabel('effect', 'Effect'), ftip('effect'), effectSeg, { wide: true }),
            ], lookSum));

            // ── What it does
            var kind = h('div.ph-seg.ph-rb-kind');
            ACTIONS.forEach(function (x) {
                var b = h('button.ph-seg__btn' + (a === x.id ? '.is-on' : ''), { type: 'button', disabled: ro(), onclick: function () { setAction(x.id); } }, x.label);
                PH.tip(b, x.tip);
                kind.appendChild(b);
            });
            var does = [field('Type', null, kind, { wide: true })];
            if (a === 'command') {
                var cmdIn = textIn('command', { mono: true, keep: true, placeholder: 'emotes' });
                cmdIn.dataset.rbKey = 'command';
                var cmdPick = ro() ? null : h('button.ph-btn.ph-btn--ghost.ph-btn--sm', {
                    type: 'button',
                    onclick: function (e) {
                        pickCommand(e.currentTarget, function (c) {
                            var v = PH.clone(selected());
                            v.command = String(c.command).replace(/^\//, '').split(/\s+/)[0];
                            if (!v.resource) v.resource = c.folder;
                            if (staffOnly(c.who) && v.admin !== true && !list(v.ace).length && !list(v.jobs).length) v.admin = true;
                            if (!v.description && c.description) v.description = fromCommand(c).description;
                            setRow(st.trail, st.sel, v);
                            drawAll();
                        });
                    },
                }, [icon('list'), 'Pick…']);
                does.push(field(flabel('command', 'Command'), ftip('command'),
                    h('div.ph-rb-cmd', [h('span.ph-rb-cmd__slash', '/'), cmdIn, cmdPick]), { wide: true }));
            } else if (a === 'export') {
                var ex = isObj(row.export) ? row.export : {};
                var setEx = function (k, v) {
                    var cur = isObj((selected() || {}).export) ? PH.clone(selected().export) : {};
                    if (v === undefined || v === '') delete cur[k]; else cur[k] = v;
                    if (!cur.resource) cur.resource = '';
                    if (!cur.name) cur.name = '';
                    commitSel('export', cur);
                };
                var resIn = textIn('export.resource', { mono: true, get: function () { return ex.resource; }, set: function (v) { setEx('resource', v.trim()); }, placeholder: 'myscript' });
                resIn.dataset.rbKey = 'export.resource';
                var nameIn = textIn('export.name', { mono: true, get: function () { return ex.name; }, set: function (v) { setEx('name', v.trim()); }, placeholder: 'OpenShop' });
                nameIn.dataset.rbKey = 'export.name';
                var exArgs = textIn('export.args', { mono: true, get: function () { return argsText(ex.args !== undefined ? ex.args : row.args); }, set: function (v) { setEx('args', parseArgs(v)); }, placeholder: 'valentine, 2' });
                exArgs.dataset.rbKey = 'export.args';
                does.push(field('Script (resource)', 'The resource that has the export. It must be running, or the option says it is unavailable.', resIn));
                does.push(field('Export', 'The client export\'s name, as the script declares it.', nameIn));
                does.push(field('Arguments', 'Passed to the export, separated by commas. Numbers and true/false are read as such; the rest is text.', exArgs, { wide: true }));
            } else if (a === 'event' || a === 'serverEvent') {
                var evIn = textIn(a, { mono: true, keep: true, placeholder: a === 'event' ? 'myscript:openShop' : 'myscript:server:ring' });
                evIn.dataset.rbKey = a;
                var argIn = textIn('args', { mono: true, get: function (r) { return argsText(r.args); }, set: function (v) { commitSel('args', parseArgs(v)); }, placeholder: 'valentine, 2' });
                argIn.dataset.rbKey = 'args';
                does.push(field(flabel(a, a === 'event' ? 'Client event' : 'Server event'), ftip(a), evIn, { wide: true }));
                does.push(field('Arguments', 'Passed with the event, separated by commas. Numbers and true/false are read as such; the rest is text.', argIn, { wide: true }));
            } else if (a === 'builtin') {
                does.push(field(flabel('builtin', 'Built-in'), ftip('builtin'), segIn('builtin', options('builtin'), undefined), { wide: true }));
            }
            var cl = codeLine(row);
            var codeBox = h('div.ph-rb-code' + (cl.bad ? '.is-bad' : ''), [
                h('div.ph-rb-code__say', [icon(cl.bad ? 'alert' : 'play'), h('span', cl.say)]),
                cl.code ? h('div.ph-rb-code__line', [
                    h('code', cl.code),
                    h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Copy', onclick: function () { PH.copyText(cl.code); PH.toast({ kind: 'success', title: 'Copied', text: cl.code }); } }, icon('copy')),
                ]) : null,
            ]);
            does.push(h('div.ph-rb-f.is-wide', [h('div.ph-rb-f__top', h('label.ph-rb-f__label', 'When a player picks it')), codeBox]));
            var doesSum = { command: '/' + String(row.command || '').replace(/^\//, ''), export: 'Export', event: 'Client event', serverEvent: 'Server event',
                builtin: 'Built-in', submenu: 'Submenu', none: 'Nothing yet' }[a] || '';
            grid.appendChild(card('What it does', 'bolt', does, doesSum));

            // ── Who sees it
            var who = whoLine(row, sub);
            var jobsHost = h('div.ph-chips__list.ph-rb-jobs');
            list(row.jobs).forEach(function (j, i) {
                jobsHost.appendChild(h('span.ph-chip', [h('span', String(j)), ro() ? null : h('button.ph-chip__x', {
                    type: 'button', title: 'Remove',
                    onclick: function () { var js = list(selected().jobs).slice(); js.splice(i, 1); commitSel('jobs', js.length ? js : undefined); drawAll(); },
                }, icon('x'))]));
            });
            if (!ro()) jobsHost.appendChild(h('button.ph-chip.ph-chip--add', {
                type: 'button',
                onclick: function (e) {
                    F.pick({
                        anchor: e.currentTarget, fetch: F.fetchers.job, allowFree: true, freeLabel: 'Add job', placeholder: 'Search jobs…',
                        onPick: function (v) {
                            var js = list(selected().jobs).slice();
                            if (v && js.indexOf(v) < 0) js.push(v);
                            commitSel('jobs', js);
                            drawAll();
                        },
                    });
                },
            }, [icon('plus'), 'Job']));
            if (!list(row.jobs).length) jobsHost.insertBefore(h('span.ph-chips__empty', 'Every job'), jobsHost.firstChild);
            var resIn2 = textIn('resource', { mono: true, get: function (r) { return list(r.resource).join(', '); }, set: function (v) {
                var names = v.split(',').map(function (x) { return x.trim(); }).filter(Boolean);
                commitSel('resource', names.length > 1 ? names : names[0]);
            }, placeholder: 'poggy_emotes' });
            resIn2.dataset.rbKey = 'resource';
            resIn2.addEventListener('change', function () { loadRemote(); });
            var resNote = list(row.resource).map(function (r) {
                var s = st.res[r];
                return s === undefined ? null : h('span.ph-rb-res' + (s === 'started' ? '.is-on' : ''), [h('i'), r + (s === 'started' ? ' is running' : ' is ' + s)]);
            }).filter(Boolean);
            var adminAce = rm.adminAce ? S.get(rm.adminAce) : null;
            var whoFields = [
                h('div.ph-rb-who' + (who.bad ? '.is-bad' : ''), [icon(who.bad ? 'alert' : 'users'), h('span', [h('b', 'Who sees it: '), who.text])]),
                field(flabel('admin', 'Staff only'), (ftip('admin') || '') + (adminAce ? ' The ace here is ' + adminAce + '.' : ''), toggleIn('admin')),
                field(flabel('when', 'When'), ftip('when'), segIn('when', options('when').length ? options('when') : Object.keys(WHEN_LABELS).map(function (k) { return { value: k, label: WHEN_LABELS[k] }; }), sub ? 'any' : 'alive')),
                field(flabel('jobs', 'Jobs'), ftip('jobs'), jobsHost, { wide: true }),
            ];
            if (list(row.jobs).length) whoFields.push(field(flabel('minGrade', 'Lowest rank'), ftip('minGrade'), numIn('minGrade')));
            whoFields.push(field(flabel('onDuty', 'On duty only'), ftip('onDuty'), toggleIn('onDuty')));
            var aceIn = textIn('ace', { mono: true, placeholder: 'poggy_menu.law' });
            aceIn.dataset.rbKey = 'ace';
            whoFields.push(field(flabel('ace', 'Ace'), ftip('ace'), aceIn));
            whoFields.push(field(flabel('resource', 'Needs resource'), ftip('resource'), h('div.ph-rb-resbox', [resIn2].concat(resNote))));
            whoFields.push(field(flabel('showLocked', 'Show when locked'), ftip('showLocked'), toggleIn('showLocked')));
            if (row.showLocked === true) {
                var lrIn = textIn('lockedReason', { placeholder: 'Only for the law.' });
                lrIn.dataset.rbKey = 'lockedReason';
                whoFields.push(field(flabel('lockedReason', 'Locked reason'), ftip('lockedReason'), lrIn));
            }
            whoFields.push(h('p.ph-rb-f__note.is-wide', 'Hiding an option is not security: whatever it runs must check its own permission.'));
            grid.appendChild(card('Who sees it', 'users', whoFields, who.bad ? 'Nobody right now' : who.text.split(/[.;]/)[0]));

            // ── Behaviour
            var confirmOn = row.confirm !== undefined && row.confirm !== false;
            var confirmTg = h('button.ph-toggle' + (confirmOn ? '.is-on' : ''), { type: 'button', role: 'switch', disabled: ro() },
                [h('span.ph-toggle__track', h('span.ph-toggle__knob')), h('span.ph-toggle__text', confirmOn ? 'On' : 'Off')]);
            confirmTg.addEventListener('click', function () { commitSel('confirm', confirmOn ? undefined : true); drawEditor(); });
            var beh = [field(flabel('confirm', 'Confirm'), ftip('confirm'), confirmTg)];
            if (confirmOn) {
                var cq = textIn('confirm', { get: function (r) { return typeof r.confirm === 'string' ? r.confirm : ''; }, set: function (v) { commitSel('confirm', v.trim() ? v : true); }, placeholder: 'Send for a doctor?' });
                cq.dataset.rbKey = 'confirm';
                beh.push(field('Question', 'Shown in the middle while the player is asked to pick again. Empty: "Pick again to confirm".', cq));
            }
            beh.push(field(flabel('cooldown', 'Cooldown'), ftip('cooldown'), numIn('cooldown', { unit: 'seconds' })));
            beh.push(field(flabel('keepOpen', 'Keep menu open'), ftip('keepOpen'), toggleIn('keepOpen')));
            beh.push(field(flabel('disabled', 'Disabled'), ftip('disabled'), toggleIn('disabled')));
            if (row.disabled === true) {
                var drIn = textIn('disabledReason', { placeholder: 'Closed for the winter.' });
                drIn.dataset.rbKey = 'disabledReason';
                beh.push(field(flabel('disabledReason', 'Disabled reason'), ftip('disabledReason'), drIn));
            }
            beh.push(field(flabel('hidden', 'Hidden'), ftip('hidden'), toggleIn('hidden')));
            var behSum = [row.confirm ? 'confirm' : null, row.cooldown ? row.cooldown + ' s cooldown' : null, row.keepOpen ? 'stays open' : null,
                row.disabled ? 'disabled' : null, row.hidden ? 'hidden' : null].filter(Boolean).join(' · ') || 'Normal';
            grid.appendChild(card('Behaviour', 'sliders', beh, behSum));

            if (focusKey) {
                var back = editor.querySelector('[data-rb-key="' + focusKey + '"]');
                if (back) { back.focus(); try { var l = back.value.length; back.setSelectionRange(l, l); } catch (e) { /* number inputs */ } }
            }
        }

        /** An editor section: a header that folds it open or shut (shut at first,
         *  remembered per section), with a line saying what is in it while shut. */
        function card(title, ic, children, summary) {
            var open = !!st.cards[title];
            var body = h('div.ph-rb-card__body', children);
            body.hidden = !open;
            var sum = h('span.ph-rb-card__sum', open ? '' : (summary || ''));
            var headBtn = h('button.ph-rb-card__head', { type: 'button', 'aria-expanded': open ? 'true' : 'false' },
                [icon(ic), h('span.ph-rb-card__title', title), sum, icon('chevdown', 'ph-rb-card__chev')]);
            var el2 = h('div.ph-rb-card' + (open ? '.is-open' : ''), [headBtn, body]);
            headBtn.addEventListener('click', function () {
                open = !open;
                st.cards[title] = open;
                PH.store.set(storeKey + ':cards', st.cards);
                body.hidden = !open;
                sum.textContent = open ? '' : (summary || '');
                el2.classList.toggle('is-open', open);
                headBtn.setAttribute('aria-expanded', open ? 'true' : 'false');
            });
            return el2;
        }

        /** After a keystroke: the wheel and the notes follow; the editor keeps its focus. */
        function refreshLight() {
            drawWheel();
            var row = selected();
            if (!row) return;
            var head = editor.querySelector('.ph-rb-ed__title b');
            if (head) head.textContent = String(row.label || row.id || 'Untitled');
            var box = editor.querySelector('.ph-rb-code');
            if (box) {
                var cl = codeLine(row);
                box.className = 'ph-rb-code' + (cl.bad ? ' is-bad' : '');
                var say = box.querySelector('.ph-rb-code__say span');
                if (say) say.textContent = cl.say;
                var code = box.querySelector('.ph-rb-code__line code');
                if (code && cl.code) code.textContent = cl.code;
                else if (cl.code && !code) drawEditor();
            }
            var whoEl = editor.querySelector('.ph-rb-who span');
            if (whoEl) { var w = whoLine(row, actionOf(row) === 'submenu'); whoEl.lastChild.textContent = w.text; }
        }

        function drawAll() {
            drawWheel();
            if (!lookEl.contains(document.activeElement)) drawLook();
            drawEditor();
            if (opts.onChange) opts.onChange();
        }

        // ------------------------------------------------------ pointer --

        function slotAt(e) {
            var r = wheel.getBoundingClientRect();
            var x = e.clientX - (r.left + r.width / 2), y = e.clientY - (r.top + r.height / 2);
            var d = Math.sqrt(x * x + y * y) / (r.width / 2) * 500;
            if (d < R_IN - 8) return -1;                // the hub
            if (d > R_OUT + 30) return null;            // outside
            var n = st.slots.length;
            if (!n) return null;
            if (n === 1) return 0;
            var deg = Math.atan2(y, x) * 180 / Math.PI + 90 + 180 / n;
            deg = ((deg % 360) + 360) % 360;
            return Math.floor(deg / (360 / n)) % n;
        }

        wheel.addEventListener('mousemove', function (e) {
            if (st.drag) {
                var dx = e.clientX - st.drag.x, dy = e.clientY - st.drag.y;
                if (!st.drag.moving && st.drag.canDrag && dx * dx + dy * dy > 64) st.drag.moving = true;
                if (st.drag.moving) {
                    var over = slotAt(e);
                    over = over !== null && over >= 0 && st.slots[over] && !st.slots[over].more ? over : null;
                    if (over !== st.drag.over) { st.drag.over = over; drawWheel(); }
                }
                return;
            }
            var i = slotAt(e);
            var hov = i !== null && i >= 0 ? i : null;
            hub.classList.toggle('is-hot', i === -1 && st.trail.length > 0);
            if (hov !== st.hover) { st.hover = hov; drawWheel(); }
        });
        wheel.addEventListener('mouseleave', function () {
            hub.classList.remove('is-hot');
            if (st.hover !== null && !st.drag) { st.hover = null; drawWheel(); }
        });
        // Picks happen on mouse-up: the hover redraw replaces the wedge under the
        // pointer, so the browser's own click would often never arrive.
        wheel.addEventListener('mousedown', function (e) {
            if (e.button !== 0) return;
            var i = slotAt(e);
            if (i === null) return;
            var s = i >= 0 ? st.slots[i] : null;
            var canDrag = s && !s.more && !ro() && st.mode === 'all';
            st.drag = { slot: i, from: s && !s.more ? s.n : null, x: e.clientX, y: e.clientY, moving: false, over: null, canDrag: canDrag };
            document.addEventListener('mouseup', onUp);
            e.preventDefault();
            wheel.focus({ preventScroll: true });
        });
        function onUp(e) {
            document.removeEventListener('mouseup', onUp);
            var d = st.drag;
            st.drag = null;
            if (!d) return;
            if (!d.moving) { pick(d.slot); return; }
            var target = d.over !== null ? st.slots[d.over] : null;
            if (target && !target.more && target.n !== d.from) moveTo(d.from, target.n);
            else drawWheel();
            e.preventDefault();
        }
        function pick(i) {
            if (i === -1) { back(); return; }
            var s = st.slots[i];
            if (!s) return;
            if (s.more) { st.page = (st.page + 1) % Math.max(1, st.pages || 1); st.keepPage = true; st.sel = null; drawAll(); return; }
            if (st.sel === s.n && s.sub && st.trail.length < MAX_DEPTH) { go(st.trail.concat([s.n])); return; }
            st.sel = s.n;
            drawAll();
        }
        wheel.addEventListener('keydown', function (e) {
            var items = st.slots.filter(function (s) { return !s.more; });
            if (!items.length && e.key !== 'Backspace') return;
            var at = items.findIndex(function (s) { return s.n === st.sel; });
            if (e.key === 'ArrowRight' || e.key === 'ArrowDown') { e.preventDefault(); st.sel = items[(at + 1) % items.length].n; drawAll(); }
            else if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') { e.preventDefault(); st.sel = items[(at - 1 + items.length) % items.length].n; drawAll(); }
            else if (e.key === 'Enter' && at >= 0 && items[at].sub) { e.preventDefault(); go(st.trail.concat([items[at].n])); }
            else if (e.key === 'Backspace' && st.trail.length) { e.preventDefault(); back(); }
            else if (e.key === 'Delete' && st.sel) { e.preventDefault(); removeSel(); }
        });
        function back() {
            if (!st.trail.length) { st.sel = null; drawAll(); return; }
            var t = st.trail.slice(), from = t.pop();
            go(t, from);
        }

        // ------------------------------------------------------- start --

        var soon = PH.debounce(function () {
            if (!el.isConnected) return;
            if (editor.contains(document.activeElement) || PH.Radial.busy) refreshLight();
            else { drawWheel(); drawEditor(); }
        }, 60);
        el.refresh = function () { drawAll(); };
        el.changed = soon;
        live = { el: el, changed: soon };

        if (window.ResizeObserver) new ResizeObserver(PH.debounce(function () { if (el.isConnected) drawWheel(); }, 60)).observe(wheel);

        libReady.then(function (lib) { st.lib = lib; if (el.isConnected) drawAll(); });
        loadRemote();
        drawAll();
        return el;
    };
})();
