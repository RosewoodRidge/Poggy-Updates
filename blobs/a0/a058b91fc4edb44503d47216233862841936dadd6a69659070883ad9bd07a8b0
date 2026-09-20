/*
    Poggy Hub — field components.

    One field per setting: a label, an ⓘ tooltip (text, default, file:line),
    badges for "changed from default" and "unsaved", a reset button, and the
    control that fits the value and its hub.json metadata:

        boolean                          toggle
        number                           number box (+ slider when min and max are set, unit label)
        number + picker heading          heading dial with "use my heading"
        number/string + picker color     colour swatch and hex box
        string                           text; multiline, webhook, item, job, group, role, key pickers
        any + options                    select (values keep their type: false stays false)
        vector2/3/4                      x/y/z(/w) boxes + "use my position"
        hash                             text (the control name)
        array of strings / numbers       chips; a top-level list can follow a role
        any + "any" hint                 a switch: one fixed value ("Anyone") or a list of names
        readonly                         the source text and "edit in the file"

    The same components are used inside list rows (hub-lists.js), where a
    field is a path inside a row (Config.Zones[3].loot[2].item) and its
    metadata comes from the list's `fields` with [] for any index or map key.

    Every control commits through PH.S (hub-state.js): a change is queued and
    applied to the working copy at once.
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var h = PH.h, icon = PH.icon, S = PH.S;

    var F = PH.F = {};

    // ----------------------------------------------------------------- meta --

    /** hub.json metadata for a node: prefix rules, then the exact entry, then the server's merge. */
    F.metaFor = function (node) {
        var cur = S.cur;
        if (!cur || !node) return {};
        cur._meta = cur._meta || {};
        if (cur._meta[node.path]) return cur._meta[node.path];
        var hub = cur.meta || {};
        var settings = hub.settings || {};
        var lists = hub.lists || {};
        var out = {};
        Object.keys(settings)
            .filter(function (k) {
                if (!/\.\*$/.test(k)) return false;
                var base = k.slice(0, -2);
                return node.path !== base && S.under(node.path, base);
            })
            .sort(function (a, b) { return a.length - b.length; })
            .forEach(function (k) { Object.assign(out, settings[k]); });
        if (settings[node.path]) Object.assign(out, settings[node.path]);
        if (lists[node.path]) Object.assign(out, lists[node.path]);
        if (node.meta && typeof node.meta === 'object') Object.assign(out, node.meta);
        cur._meta[node.path] = out;
        return out;
    };

    F.labelFor = function (node) {
        // The navigation's name wins: it never repeats a bare "Sell" (§8.3).
        var nl = S.cur && S.cur.navLabel && S.cur.navLabel[node.path];
        if (nl) return nl;
        var m = F.metaFor(node);
        return m.label || PH.readable(node.key != null ? node.key : node.path.split('.').pop());
    };

    F.tooltipFor = function (node) {
        var m = F.metaFor(node);
        return m.tooltip || node.comment || '';
    };

    // ------------------------------------------------------------- pickers --

    var cache = { jobs: null, roles: null, items: {} };
    F.cache = cache;

    F.loadJobs = function () {
        if (!cache.jobs) {
            cache.jobs = PH.api('jobs').then(function (r) {
                return r.ok && Array.isArray(r.value) ? r.value : [];
            });
        }
        return cache.jobs;
    };

    F.loadRoles = function (force) {
        if (!cache.roles || force) {
            cache.roles = PH.api('roles').then(function (r) {
                var v = r.ok && r.value ? r.value : { roles: {}, usage: {} };
                v.roles = v.roles || {};
                v.usage = v.usage || {};
                S.roles = v;
                return v;
            });
        }
        return cache.roles;
    };

    F.searchItems = function (q) {
        q = String(q || '').trim();
        if (cache.items[q]) return cache.items[q];
        var p = PH.api('items', q).then(function (r) {
            var list = r.ok && Array.isArray(r.value) ? r.value : [];
            list.forEach(function (it) { if (it && it.name) cache.itemInfo[it.name] = it; });
            return list;
        });
        cache.items[q] = p;
        return p;
    };
    cache.itemInfo = {};

    /** Label and image for one item name, from the same search the picker uses. */
    F.itemInfo = function (name) {
        if (!name) return Promise.resolve(null);
        if (cache.itemInfo[name]) return Promise.resolve(cache.itemInfo[name]);
        return F.searchItems(name).then(function () { return cache.itemInfo[name] || null; });
    };

    var DEFAULT_GROUPS = ['admin', 'superadmin', 'god', 'moderator', 'mod', 'user'];

    var CONTROL_NAMES = [
        'INPUT_CONTEXT', 'INPUT_CONTEXT_X', 'INPUT_CONTEXT_Y', 'INPUT_CONTEXT_A', 'INPUT_CONTEXT_B', 'INPUT_CONTEXT_LT', 'INPUT_CONTEXT_RT',
        'INPUT_INTERACT_OPTION1', 'INPUT_INTERACT_OPTION2', 'INPUT_INTERACT_LOCKON_POS', 'INPUT_INTERACT_LOCKON_NEG',
        'INPUT_FRONTEND_ACCEPT', 'INPUT_FRONTEND_CANCEL', 'INPUT_FRONTEND_UP', 'INPUT_FRONTEND_DOWN', 'INPUT_FRONTEND_LEFT', 'INPUT_FRONTEND_RIGHT',
        'INPUT_FRONTEND_RB', 'INPUT_FRONTEND_LB', 'INPUT_FRONTEND_X', 'INPUT_FRONTEND_Y',
        'INPUT_GAME_MENU_ACCEPT', 'INPUT_GAME_MENU_CANCEL', 'INPUT_GAME_MENU_OPTION',
        'INPUT_JUMP', 'INPUT_SPRINT', 'INPUT_DUCK', 'INPUT_RELOAD', 'INPUT_ENTER', 'INPUT_LOOT', 'INPUT_LOOT2', 'INPUT_LOOT3',
        'INPUT_OPEN_JOURNAL', 'INPUT_OPEN_SATCHEL_MENU', 'INPUT_PLAYER_MENU', 'INPUT_QUICK_USE_ITEM', 'INPUT_WHISTLE',
        'INPUT_AIM', 'INPUT_ATTACK', 'INPUT_COVER', 'INPUT_HORSE_JUMP', 'INPUT_SELECT_RADAR_MODE', 'INPUT_MAP',
        'INPUT_CREATOR_LT', 'INPUT_CREATOR_RT', 'INPUT_CREATOR_ACCEPT', 'INPUT_DYNAMIC_SCENARIO', 'INPUT_SPECIAL_ABILITY',
    ];

    /**
     * A search-as-you-type picker in a popover.
     * opts: { anchor, fetch(q) -> Promise<[{ value, label, sub, image }]>, allowFree, freeLabel, placeholder, onPick(value) }
     */
    F.pick = function (opts) {
        var input = h('input.ph-input.ph-pick__search', { type: 'text', placeholder: opts.placeholder || 'Search…', spellcheck: 'false' });
        var list = h('div.ph-pick__list', { role: 'listbox' });
        var status = h('div.ph-pick__status');
        var pop = PH.popover(opts.anchor, h('div.ph-pick', [h('div.ph-pick__bar', [icon('search'), input]), list, status]),
            { minWidth: opts.width || 360, maxHeight: 440, cls: 'ph-pop--pick' });
        var results = [], sel = 0, seq = 0;

        function row(r, i) {
            var img = r.image ? h('img.ph-pick__img', { src: r.image, alt: '' }) : null;
            if (img) img.addEventListener('error', function () { img.style.visibility = 'hidden'; });
            return h('button.ph-pick__row' + (i === sel ? '.is-sel' : ''), {
                type: 'button', role: 'option',
                onmouseenter: function () { select(i); },
                onclick: function () { choose(i); },
            }, [
                r.image !== undefined ? h('span.ph-pick__imgwrap', img) : (r.icon ? icon(r.icon) : null),
                h('span.ph-pick__main', [h('span.ph-pick__label', r.label), r.sub ? h('span.ph-pick__sub', r.sub) : null]),
                r.badge ? h('span.ph-tag', r.badge) : null,
            ]);
        }
        function draw() {
            PH.clear(list);
            results.forEach(function (r, i) { list.appendChild(row(r, i)); });
        }
        function select(i) {
            sel = i;
            PH.$$('.ph-pick__row', list).forEach(function (el, k) { el.classList.toggle('is-sel', k === i); });
            var el = list.children[i];
            if (el) el.scrollIntoView({ block: 'nearest' });
        }
        function choose(i) {
            var r = results[i];
            if (!r) return;
            pop.close();
            opts.onPick(r.value, r);
        }
        var run = PH.debounce(function () {
            var q = input.value.trim();
            var mine = ++seq;
            status.textContent = 'Searching…';
            Promise.resolve(opts.fetch(q)).then(function (res) {
                if (mine !== seq) return;
                results = (res || []).slice(0, 60);
                if (opts.allowFree && q && !results.some(function (r) { return String(r.value).toLowerCase() === q.toLowerCase(); })) {
                    results.push({ value: q, label: (opts.freeLabel || 'Use') + ' “' + q + '”', sub: 'Typed by hand', icon: 'pencil' });
                }
                sel = 0;
                draw();
                status.textContent = results.length ? '' : (q ? 'Nothing matches “' + q + '”.' : 'Type to search.');
            });
        }, 180);
        input.addEventListener('input', run);
        input.addEventListener('keydown', function (e) {
            if (e.key === 'ArrowDown') { e.preventDefault(); select(Math.min(results.length - 1, sel + 1)); }
            else if (e.key === 'ArrowUp') { e.preventDefault(); select(Math.max(0, sel - 1)); }
            else if (e.key === 'Enter') { e.preventDefault(); choose(sel); }
        });
        run();
        setTimeout(function () { input.focus(); }, 20);
        return pop;
    };

    function jobFetch(q) {
        return F.loadJobs().then(function (jobs) {
            return jobs.filter(function (j) { return PH.matches(q, [j.name, j.label]); }).map(function (j) {
                return { value: j.name, label: j.label || j.name, sub: j.name + (j.grades && j.grades.length ? ' · ' + PH.plural(j.grades.length, 'grade') : ''), icon: 'badge' };
            });
        });
    }

    function groupFetch(q) {
        return F.loadRoles().then(function (rv) {
            var names = DEFAULT_GROUPS.slice();
            Object.keys(rv.roles).forEach(function (k) {
                var role = rv.roles[k];
                if (role && role.kind === 'groups' && Array.isArray(role.list)) role.list.forEach(function (g) { if (names.indexOf(g) === -1) names.push(g); });
            });
            return names.filter(function (n) { return PH.matches(q, [n]); }).map(function (n) { return { value: n, label: n, icon: 'shield' }; });
        });
    }

    function roleFetch(q) {
        return F.loadRoles().then(function (rv) {
            return Object.keys(rv.roles).filter(function (k) { return PH.matches(q, [k, rv.roles[k].label]); }).map(function (k) {
                var r = rv.roles[k];
                return { value: k, label: r.label || k, sub: k + ' · ' + (r.kind === 'groups' ? 'admin groups' : 'jobs') + ' · ' + (r.list || []).join(', '), icon: 'users' };
            });
        });
    }

    function itemFetch(q) {
        return F.searchItems(q).then(function (items) {
            return items.map(function (it) { return { value: it.name, label: it.label || it.name, sub: it.name, image: it.image || null }; });
        });
    }

    function keyFetch(q) {
        return Promise.resolve(CONTROL_NAMES.filter(function (n) { return PH.matches(q, [n]); }).map(function (n) {
            return { value: n, label: n, icon: 'keyboard' };
        }));
    }

    F.fetchers = { job: jobFetch, group: groupFetch, role: roleFetch, item: itemFetch, key: keyFetch };

    // ---------------------------------------------------------- item checks --
    // §9: a script whose hub.json turns on `itemCheck` gets every item-picker
    // value checked against the server's item registry. The `script` answer
    // carries `itemCheck = { iconResource, iconPattern, values: { name: {
    // exists, icon, label, suggest } } }` for the values in the file; the
    // whole registry comes once per hub session from `itemList`, so a name
    // typed since is checked here, exactly and case-sensitively; whether it
    // has an icon is asked of the server with `itemCheck(names)` (batched,
    // cached per script). `diagnostics` (§9.2) explains rows the script hides.

    var IC = PH.IC = { list: null, byName: null, byLower: null, subs: [], version: 0, editSeq: 0 };
    var STRUCT_OPS = { insert: 1, remove: 1, move: 1, renameKey: 1, duplicate: 1 };
    S.on(function () { IC.editSeq++; });

    /** A stamp that changes whenever a check could give a different answer (for memos). */
    IC.stamp = function () { return (S.cur ? S.cur.id : '') + ':' + IC.editSeq + ':' + IC.version; };

    /** Is the open script checking its item names? */
    IC.enabled = function () {
        var cur = S.cur;
        return !!(cur && ((cur.data && cur.data.itemCheck) || (cur.meta && cur.meta.itemCheck)));
    };

    function icData() { return (S.cur && S.cur.data && PH.isPlainObj(S.cur.data.itemCheck)) ? S.cur.data.itemCheck : {}; }

    /** The whole registry, once per session. Resolves the array, or null when the server has none. */
    IC.loadList = function () {
        if (IC._listP) return IC._listP;
        IC._listP = PH.api('itemList').then(function (r) {
            var v = r.ok ? r.value : null;
            var items = v && Array.isArray(v.items) ? v.items : Array.isArray(v) ? v : null;
            if (v && v.available === false) items = null;
            if (v && typeof v.imageBase === 'string') IC.imageBase = v.imageBase;
            if (!items) {
                // An older server, or the call failed: try again in a while, and
                // meanwhile check with the answers the script came with.
                setTimeout(function () { IC._listP = null; }, 30000);
                return null;
            }
            var byName = Object.create(null), byLower = Object.create(null), list = [];
            items.forEach(function (it) {
                if (!it || typeof it.name !== 'string' || !it.name || byName[it.name]) return;
                var e = { name: it.name, label: typeof it.label === 'string' && it.label ? it.label : it.name, image: it.image || null };
                e.lname = e.name.toLowerCase();
                e.llabel = e.label.toLowerCase();
                byName[e.name] = e;
                (byLower[e.lname] = byLower[e.lname] || []).push(e.name);
                list.push(e);
            });
            list.sort(function (a, b) { return a.lname < b.lname ? -1 : a.lname > b.lname ? 1 : 0; });
            IC.list = list; IC.byName = byName; IC.byLower = byLower;
            IC.bump();
            return list;
        });
        return IC._listP;
    };

    /** Something the checks depend on arrived: tell whoever is showing them. */
    IC.bump = function () {
        IC.version++;
        var now = Date.now();
        IC.subs = IC.subs.filter(function (s) {
            if (s.el.isConnected) { s.seen = true; return true; }
            return !s.seen && now - s.at < 4000;   // not attached yet: keep it a moment
        });
        IC.subs.slice().forEach(function (s) { if (s.el.isConnected) { try { s.fn(); } catch (e) { console.error(e); } } });
    };

    /** Re-run fn when the item list or an icon answer arrives, for as long as el is on the page. */
    IC.watch = function (el, fn) { IC.subs.push({ el: el, fn: fn, at: Date.now(), seen: false }); };

    function extraOf(cur) { cur._icExtra = cur._icExtra || {}; return cur._icExtra; }

    function answerFor(name) {
        var cur = S.cur;
        if (!cur) return null;
        var ex = extraOf(cur)[name];
        if (ex) return ex;
        var vals = icData().values;
        return vals && Object.prototype.hasOwnProperty.call(vals, name) ? vals[name] : null;
    }

    // Names typed since the script loaded: asked of the server in one batch.
    var askQueue = {};
    var askFlush = PH.debounce(function () {
        var cur = S.cur;
        if (!cur) { askQueue = {}; return; }
        var names = Object.keys(askQueue);
        askQueue = {};
        if (!names.length) return;
        cur._icAsked = cur._icAsked || {};
        names.forEach(function (n) { cur._icAsked[n] = true; });
        // The server answers up to 500 names a call; it checks icons the way this script does (its id).
        for (var i = 0; i < names.length; i += 500) {
            PH.api('itemCheck', names.slice(i, i + 500), cur.id).then(function (r) {
                if (S.cur !== cur) return;
                var vals = r.ok && r.value && PH.isPlainObj(r.value.values) ? r.value.values : null;
                if (!vals) return;
                var ex = extraOf(cur);
                Object.keys(vals).forEach(function (k) { if (PH.isPlainObj(vals[k])) ex[k] = vals[k]; });
                IC.bump();
            });
        }
    }, 350);

    function ask(name) {
        var cur = S.cur;
        if (!cur || (cur._icAsked && cur._icAsked[name])) return;
        askQueue[name] = true;
        askFlush();
    }

    /**
     * What is known about one item name:
     *   null for an empty value (never flagged), else
     *   { name, state: 'ok' | 'missing' | 'noicon' | 'unknown', suggest, label }
     */
    IC.status = function (name) {
        if (typeof name !== 'string' || name === '') return null;
        var a = answerFor(name);
        var exists, suggest = null, label = null;
        // The server could not read its item registry: nothing can be said (never red).
        if (icData().available === false && !IC.byName) return { name: name, state: 'unknown' };
        if (IC.byName) {
            var it = IC.byName[name];
            // Weapons are not in the item list; the server says they exist (weapon: true).
            exists = !!it || !!(a && a.exists === true);
            if (it) label = it.label;
            else if (!exists && !a && /^weapon_/i.test(name)) { ask(name); return { name: name, state: 'unknown' }; }
            else if (!exists) {
                var alts = IC.byLower[name.toLowerCase()];
                if (alts && alts.length) suggest = alts[0];
            }
        } else if (a) {
            if (typeof a.exists !== 'boolean') return { name: name, state: 'unknown' };
            exists = a.exists;
        } else {
            ask(name);
            return { name: name, state: 'unknown' };
        }
        if (!exists) {
            if (!suggest && a && typeof a.suggest === 'string' && a.suggest && a.suggest !== name) suggest = a.suggest;
            return { name: name, state: 'missing', suggest: suggest };
        }
        if (a && typeof a.label === 'string' && a.label) label = a.label;
        var iconKnown = !!a && typeof a.icon === 'boolean';
        if (!iconKnown) ask(name);
        return { name: name, state: iconKnown && a.icon === false ? 'noicon' : 'ok', label: label, iconKnown: iconKnown };
    };

    /** The file an item's icon should be, as the game looks for it: vorp_inventory/html/img/items/steel.png. */
    IC.iconFile = function (name) {
        var a = answerFor(name);
        if (a && typeof a.file === 'string' && a.file) return a.file;
        var d = icData();
        var pat = typeof d.iconPattern === 'string' && d.iconPattern ? d.iconPattern : '%s.png';
        var file = pat.indexOf('%s') !== -1 ? pat.replace(/%s/g, name) : pat.replace(/\/?$/, '/') + name + '.png';
        return (d.iconResource ? d.iconResource + '/' : '') + file;
    };

    /** An image URL for the item, or null. */
    IC.imageUrl = function (name) {
        var it = IC.byName && IC.byName[name];
        if (it && it.image) return it.image;
        var d = icData();
        if (!d.iconResource) {
            if (IC.imageBase) return IC.imageBase + name + '.png';
            var c = F.cache.itemInfo[name];
            return c && c.image ? c.image : null;
        }
        return 'nui://' + IC.iconFile(name);
    };

    /**
     * Suggestions for what was typed: names or labels that start with it
     * (case-insensitive) first, then ones that contain it. [{ name, label, image, rank }]
     */
    IC.suggest = function (q, max) {
        var list = IC.list || [];
        q = String(q || '').trim().toLowerCase();
        if (!q) return [];
        var out = [];
        for (var i = 0; i < list.length; i++) {
            var it = list[i], rank;
            if (it.lname === q) rank = 0;
            else if (it.lname.indexOf(q) === 0) rank = 1;
            else if (it.llabel.indexOf(q) === 0) rank = 2;
            else if (it.lname.indexOf(q) !== -1 || it.llabel.indexOf(q) !== -1) rank = 3;
            else continue;
            out.push({ it: it, rank: rank });
        }
        out.sort(function (a, b) {
            return a.rank - b.rank || a.it.name.length - b.it.name.length || (a.it.lname < b.it.lname ? -1 : a.it.lname > b.it.lname ? 1 : 0);
        });
        return out.slice(0, max || 40).map(function (x) { return { name: x.it.name, label: x.it.label, image: x.it.image, rank: x.rank }; });
    };

    // ------------------------------------------------ item checks: rows --

    /** Row-relative patterns of a list's item-picker fields ("Items[].name", "Items[].AltNames[]"). '' = the row itself. */
    IC.patterns = function (meta) {
        var f = (meta && meta.fields) || {};
        return Object.keys(f).filter(function (k) { return f[k] && f[k].picker === 'item'; }).map(function (k) { return k === '[]' ? '' : k; });
    };

    /** The patterns under one list inside a row (sub "Items" of "Items[].name" -> "name"), relative to each entry. */
    IC.subPatterns = function (meta, prefix) {
        var out = [];
        var p = prefix + '[]';
        IC.patterns(meta).forEach(function (k) {
            if (k === p) out.push('');
            else if (k.indexOf(p + '.') === 0) out.push(k.slice(p.length + 1));
            else if (k.indexOf(p + '[') === 0) out.push(k.slice(p.length));
        });
        return out;
    };

    /** Every item name (string) the patterns reach inside a value: [{ path, name }]. */
    IC.valuesIn = function (value, basePath, patterns) {
        var out = [];
        (patterns || []).forEach(function (pat) {
            var hits = pat === '' ? [{ path: basePath, value: value }] : R.expand(value, basePath, pat);
            hits.forEach(function (hit) {
                if (typeof hit.value === 'string') out.push({ path: hit.path, name: hit.value });
                else if (Array.isArray(hit.value)) hit.value.forEach(function (x, i) { if (typeof x === 'string') out.push({ path: hit.path + '[' + (i + 1) + ']', name: x }); });
            });
        });
        return out;
    };

    function diagIndex(cur) {
        if (cur._diagIdx) return cur._diagIdx;
        var idx = {};
        var d = cur.data && cur.data.diagnostics;
        var rows = d && PH.isPlainObj(d.rows) ? d.rows : {};
        Object.keys(rows).forEach(function (k) {
            var list = Array.isArray(rows[k]) ? rows[k].filter(PH.isPlainObj) : [];
            if (list.length) idx[PH.canon(k)] = (idx[PH.canon(k)] || []).concat(list);
        });
        cur._diagIdx = idx;
        return idx;
    }
    IC.hasDiagnostics = function () { return !!(S.cur && Object.keys(diagIndex(S.cur)).length); };

    /**
     * Diagnostics describe the config the script is running. After a row is
     * added, removed or moved under `listPath`, row numbers no longer line up
     * with them, so they are held back until the next save and restart.
     */
    IC.diagStale = function (listPath) {
        var cur = S.cur;
        if (!cur || !listPath) return false;
        return cur.pending.some(function (c) { return STRUCT_OPS[c.op] && (S.under(c.path, listPath) || S.under(listPath, c.path)); });
    };

    /** Does the script say anything about rows under this list? */
    IC.diagUnder = function (listPath) {
        var cur = S.cur;
        if (!cur) return false;
        return Object.keys(diagIndex(cur)).some(function (k) { return S.under(k, listPath); });
    };

    IC.diagFor = function (rowPath) {
        var cur = S.cur;
        if (!cur) return [];
        return diagIndex(cur)[PH.canon(rowPath)] || [];
    };

    function isHiddenDiag(d) { return d.code === 'missing_icon' || d.code === 'chain' || d.code === 'hidden' || /hidden/i.test(String(d.message || '')); }

    /** The problems of one row: item names that do not exist or have no icon, and what the script says about it. */
    IC.issues = function (value, rowPath, patterns, opts) {
        opts = opts || {};
        var out = { missing: [], noicon: [], diag: [], hidden: false, level: null };
        var seenM = {}, seenN = {};
        if (IC.enabled()) {
            IC.valuesIn(value, rowPath, patterns).forEach(function (v) {
                var st = IC.status(v.name);
                if (!st) return;
                if (st.state === 'missing' && !seenM[v.name]) { seenM[v.name] = true; out.missing.push({ name: v.name, path: v.path, suggest: st.suggest }); }
                if (st.state === 'noicon' && !seenN[v.name]) { seenN[v.name] = true; out.noicon.push({ name: v.name, path: v.path }); }
            });
        }
        if (!opts.noDiag) {
            (opts.diagPaths || [rowPath]).forEach(function (p) {
                IC.diagFor(p).forEach(function (d) { if (d.level !== 'info') out.diag.push(d); });
            });
        }
        out.hidden = out.diag.some(isHiddenDiag);
        out.level = out.missing.length ? 'bad' : (out.noicon.length || out.diag.length) ? 'warn' : null;
        return out;
    };

    /** Add one set of issues into another (a collection row sums its lists). */
    IC.merge = function (into, add) {
        add.missing.forEach(function (m) { if (!into.missing.some(function (x) { return x.name === m.name; })) into.missing.push(m); });
        add.noicon.forEach(function (m) { if (!into.noicon.some(function (x) { return x.name === m.name; })) into.noicon.push(m); });
        add.diag.forEach(function (d) { if (into.diag.indexOf(d) === -1) into.diag.push(d); });
        into.hidden = into.hidden || add.hidden;
        into.level = into.missing.length ? 'bad' : (into.noicon.length || into.diag.length) ? 'warn' : null;
        return into;
    };
    IC.empty = function () { return { missing: [], noicon: [], diag: [], hidden: false, level: null }; };

    /** Does a row pass a filter chip? flag: 'missing' | 'noicon' | 'hidden'. */
    IC.passes = function (iss, flag) {
        if (!flag) return true;
        if (flag === 'missing') return iss.missing.length > 0;
        if (flag === 'noicon') return iss.noicon.length > 0;
        if (flag === 'hidden') return iss.hidden;
        return true;
    };

    /** The labels for a row: [{ level: 'bad'|'warn', text }]. */
    IC.labels = function (iss) {
        var out = [];
        if (iss.missing.length) out.push({ level: 'bad', text: 'Item does not exist: ' + iss.missing.map(function (m) { return m.name; }).join(', ') });
        var covered = {};
        iss.diag.forEach(function (d) {
            var t = String(d.message || d.code || 'Warning');
            var files = Array.isArray(d.files) ? d.files.filter(function (f) { return typeof f === 'string' && f; }) : [];
            if (files.length) t += ' (add ' + files.join(', ') + ')';
            (Array.isArray(d.items) ? d.items : []).forEach(function (n) { covered[n] = true; });
            out.push({ level: d.level === 'error' ? 'bad' : 'warn', text: t, diag: true });
        });
        var loose = iss.noicon.filter(function (m) { return !covered[m.name]; });
        if (loose.length) {
            out.push({ level: 'warn', text: 'No icon: ' + loose.map(function (m) { return m.name; }).join(', ') +
                ' (add ' + loose.map(function (m) { return IC.iconFile(m.name); }).join(', ') + ')' });
        }
        return out;
    };

    /** The labels as elements; full text on hover. */
    IC.flagsEl = function (iss, opts) {
        opts = opts || {};
        var labels = IC.labels(iss);
        if (!labels.length) return null;
        var wrap = h('div.ph-rowflags' + (opts.full ? '.is-full' : ''));
        labels.forEach(function (l) {
            var el = h('div.ph-rowflag.is-' + l.level, [icon('alert'), h('span.ph-rowflag__text', l.text)]);
            PH.tip(el, function () {
                return h('div', [h('div.ph-tip__text', l.text), l.diag ? h('div.ph-tip__code', 'As the script is running now: save and restart to check again.') : null]);
            });
            wrap.appendChild(el);
        });
        return wrap;
    };

    /**
     * The filter chips for a list toolbar: All · Item does not exist (n) ·
     * Missing icon (n) · Hidden in game (n). counts: { missing, noicon, hidden }.
     * Chips with nothing to show are left out (the active one stays).
     */
    IC.flagBar = function (counts, active, onPick, note) {
        var any = counts.missing || counts.noicon || counts.hidden || active;
        if (!any && !note) return null;
        var bar = h('div.ph-flagbar', { role: 'group', 'aria-label': 'Show only rows with a problem' });
        function chip(flag, label, n, tone) {
            if (flag && !n && active !== flag) return;
            var on = (active || null) === flag;
            bar.appendChild(h('button.ph-flagchip' + (tone ? '.is-' + tone : '') + (on ? '.is-on' : ''), {
                type: 'button', 'aria-pressed': on ? 'true' : 'false',
                onclick: function () { onPick(on && flag ? null : flag); },
            }, [tone ? h('span.ph-flagchip__dot') : null, label, flag ? h('span.ph-flagchip__n', String(n)) : null]));
        }
        if (any) {
            chip(null, 'All', 0, null);
            chip('missing', 'Item does not exist', counts.missing, 'bad');
            chip('noicon', 'Missing icon', counts.noicon, 'warn');
            chip('hidden', 'Hidden in game', counts.hidden, 'warn');
        }
        if (note) bar.appendChild(h('span.ph-flagbar__note', [icon('info'), note]));
        return bar;
    };

    // --------------------------------------------- item checks: type-ahead --

    function hl(text, q) {
        var s = String(text), i = q ? s.toLowerCase().indexOf(q.toLowerCase()) : -1;
        if (i === -1) return s;
        return [s.slice(0, i), h('b.ph-ta__hl', s.slice(i, i + q.length)), s.slice(i + q.length)];
    }

    function itemImg(name, cls) {
        var url = IC.imageUrl(name);
        var img = h('img' + (cls || '.ph-pick__img'), { alt: '' });
        img.style.visibility = 'hidden';
        if (url) {
            img.addEventListener('load', function () { img.style.visibility = ''; });
            img.addEventListener('error', function () { img.style.visibility = 'hidden'; });
            img.src = url;
        }
        return img;
    }

    /**
     * Type-ahead on a text box: typing lists registry items (starts-with
     * first, then contains); ↑/↓ move, Enter / Tab / click take the exact
     * name, Esc cancels.
     * opts: { anchor, onPick(name), onEnter(text), onCancel(), exclude(name) }
     */
    IC.typeahead = function (input, opts) {
        var pop = null, listEl = null, statusEl = null, results = [], sel = -1;
        function close() { if (pop) { var p = pop; pop = null; p.close(); } }
        function open() {
            if (pop) return;
            listEl = h('div.ph-pick__list.ph-ta__list', { role: 'listbox' });
            // Scrolling the list with the mouse must not take the focus out of the box.
            listEl.addEventListener('mousedown', function (e) { e.preventDefault(); });
            statusEl = h('div.ph-pick__status');
            pop = PH.popover(opts.anchor || input, h('div.ph-pick.ph-ta', [listEl, statusEl]),
                { minWidth: 360, maxHeight: 380, cls: 'ph-pop--pick.ph-pop--ta', onClose: function () { pop = null; } });
        }
        function choose(i) {
            var r = results[i];
            if (!r) return;
            close();
            input.value = r.name;
            opts.onPick(r.name);
        }
        function select(i) {
            sel = i;
            if (!listEl) return;
            PH.$$('.ph-pick__row', listEl).forEach(function (el, k) { el.classList.toggle('is-sel', k === i); });
            var el = listEl.children[i];
            if (el) el.scrollIntoView({ block: 'nearest' });
        }
        function draw() {
            var q = input.value.trim();
            if (!q || input.readOnly || input.disabled) { close(); return; }
            open();
            PH.clear(listEl);
            if (!IC.list) {
                statusEl.textContent = 'Loading the server’s item list…';
                IC.loadList().then(function (l) {
                    if (document.activeElement !== input) return;
                    if (l) draw();
                    else if (statusEl) statusEl.textContent = 'This server did not send its item list, so there is nothing to suggest.';
                });
                return;
            }
            results = IC.suggest(q, 60).filter(function (r) { return !opts.exclude || !opts.exclude(r.name); });
            sel = -1;
            // Preselect only a real match: Enter on a name that is not in the list keeps what was typed.
            for (var i = 0; i < results.length; i++) {
                if (results[i].name === q) { sel = i; break; }
            }
            if (sel === -1 && results.length && results[0].rank <= 2) sel = 0;
            results.forEach(function (r, i) {
                var st = IC.status(r.name);
                var row = h('button.ph-pick__row' + (i === sel ? '.is-sel' : ''), { type: 'button', role: 'option', tabindex: '-1' }, [
                    h('span.ph-pick__imgwrap', itemImg(r.name)),
                    h('span.ph-pick__main', [h('span.ph-pick__label', hl(r.label, q)), h('span.ph-pick__sub', hl(r.name, q))]),
                    st && st.state === 'noicon' ? h('span.ph-tag.ph-tag--warn', { title: 'No icon: add ' + IC.iconFile(r.name) }, 'No icon') : null,
                ]);
                // Keep the focus in the box: the click picks, the box never blurs.
                row.addEventListener('mousedown', function (e) { e.preventDefault(); });
                row.addEventListener('mouseenter', function () { select(i); });
                row.addEventListener('click', function () { choose(i); });
                listEl.appendChild(row);
            });
            statusEl.textContent = results.length ? (IC.list.length > 60 && results.length === 60 ? 'Keep typing to narrow it down.' : '')
                : 'No item is called “' + q + '”. Names must match exactly, capitals included.';
            if (pop && pop.place) pop.place();
        }
        input.addEventListener('input', draw);
        input.addEventListener('focus', function () { if (!IC.list) IC.loadList(); });
        input.addEventListener('keydown', function (e) {
            if (e.key === 'ArrowDown' || e.key === 'ArrowUp') {
                e.preventDefault();
                if (!pop) { draw(); return; }
                if (!results.length) return;
                select(e.key === 'ArrowDown' ? Math.min(results.length - 1, sel + 1) : Math.max(0, sel - 1));
            } else if (e.key === 'Enter') {
                e.preventDefault();
                if (pop && sel >= 0) choose(sel);
                else { close(); opts.onEnter(input.value); }
            } else if (e.key === 'Tab') {
                if (pop && sel >= 0) choose(sel);
                else close();
            } else if (e.key === 'Escape') {
                var had = !!pop;
                close();
                var changed = opts.onCancel ? opts.onCancel() : false;
                if (had || changed) { e.preventDefault(); e.stopPropagation(); }
            }
        });
        input.addEventListener('blur', function () { setTimeout(function () { if (document.activeElement !== input) close(); }, 0); });
        return { close: close };
    };

    /** The red / yellow note under an item box: "Item does not exist · Did you mean …?" / "No icon: add …". */
    function itemNote(note, st, onFix) {
        PH.clear(note);
        if (!st) return null;
        if (st.state === 'missing') {
            note.appendChild(h('span.ph-icchip.is-bad', [icon('alert'), 'Item does not exist']));
            if (st.suggest) {
                note.appendChild(h('span.ph-itemnote__text', ['Did you mean ',
                    h('button.ph-itemfix', { type: 'button', title: 'Use ' + st.suggest + ' (the exact name)', onclick: function () { onFix(st.suggest); } }, st.suggest), '?']));
            } else {
                note.appendChild(h('span.ph-itemnote__text', '“' + st.name + '” is not in this server’s item list. Names must match exactly, capitals included.'));
            }
            return 'bad';
        }
        if (st.state === 'noicon') {
            var file = IC.iconFile(st.name);
            note.appendChild(h('span.ph-icchip.is-warn', { title: 'Without an icon the item is hidden in some menus' }, [icon('alert'), 'No icon: add ' + file]));
            note.appendChild(h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Copy the file path', onclick: function () {
                PH.copyText(file); PH.toast({ kind: 'info', title: 'Copied', text: file });
            } }, icon('copy')));
            return 'warn';
        }
        return null;
    }

    /** §9.3: an item name typed with type-ahead, checked exactly against the registry. */
    function ctlItemText(spec, commit) {
        var val = spec.value == null ? '' : String(spec.value);
        var input = h('input.ph-input.ph-itemta__input', { type: 'text', spellcheck: 'false', autocomplete: 'off', placeholder: 'Type an item name…', value: val });
        var imgWrap = h('span.ph-pick__imgwrap.ph-itemta__img');
        var labelEl = h('span.ph-itemta__label');
        var note = h('div.ph-itemnote');
        var wrap = h('div.ph-itemta', [h('div.ph-itemta__box', [imgWrap, input, labelEl]), note]);
        var shownFor = null;

        function show() {
            var st = IC.status(val);
            var level = itemNote(note, st, function (fix) { take(fix); });
            wrap.classList.toggle('is-bad', level === 'bad');
            wrap.classList.toggle('is-warn', level === 'warn');
            input.classList.toggle('is-bad', level === 'bad');
            labelEl.style.visibility = '';
            labelEl.textContent = st && st.label && st.label !== val ? st.label : '';
            if (shownFor !== val + '|' + (st ? st.state : '')) {
                shownFor = val + '|' + (st ? st.state : '');
                PH.clear(imgWrap);
                if (st && st.state === 'ok') imgWrap.appendChild(itemImg(val));
                else imgWrap.appendChild(icon(st && st.state === 'missing' ? 'alert' : 'bag'));
            }
        }
        function take(text) {
            text = String(text == null ? '' : text).trim();
            input.value = text;
            if (text !== val) { val = text; commit(text); }
            show();
        }
        // What was typed is not the value yet: its label and picture wait for the commit.
        input.addEventListener('input', function () { labelEl.style.visibility = input.value === val ? '' : 'hidden'; });
        IC.typeahead(input, {
            anchor: wrap.firstChild,
            onPick: take,
            onEnter: take,
            onCancel: function () { if (input.value === val) return false; input.value = val; return true; },
        });
        input.addEventListener('change', function () { take(input.value); });
        IC.watch(wrap, show);
        IC.loadList();
        show();
        return {
            el: wrap,
            set: function (v) { val = v == null ? '' : String(v); input.value = val; show(); },
            disable: function (d) { input.readOnly = d; input.classList.toggle('is-disabled', d); },
        };
    }

    // ---------------------------------------------------------- references --
    // §8.5: a setting or list field can hold the key of a row elsewhere in the
    // script (a store's type is a key of Store types). The declarations come
    // from hub.json (`ref` in a field's metadata) and, for anything the server
    // found on its own, from the paths in its `refs[target].usedBy`. Keys and
    // usage are worked out from the working copy, so a row added or renamed a
    // moment ago counts before it is saved.

    var R = PH.R = {};

    function tokens(pattern) {
        var out = [];
        String(pattern || '').replace(/\[\]|[^.\[\]]+/g, function (t) { out.push(t); return t; });
        return out;
    }

    /** Every { path, value } a row-relative pattern ("loot[].item") reaches inside a value. */
    function expand(value, basePath, toks, out) {
        if (!toks.length) { out.push({ path: basePath, value: value }); return out; }
        var t = toks[0], rest = toks.slice(1);
        if (t === '[]') {
            if (Array.isArray(value)) value.forEach(function (x, i) { expand(x, basePath + '[' + (i + 1) + ']', rest, out); });
            else if (PH.isPlainObj(value)) Object.keys(value).forEach(function (k) { if (k !== '__int_keys') expand(value[k], PH.joinPath(basePath, k, true), rest, out); });
            return out;
        }
        if (!PH.isPlainObj(value) || !Object.prototype.hasOwnProperty.call(value, t)) return out;
        expand(value[t], PH.IDENT.test(t) ? basePath + '.' + t : PH.joinPath(basePath, t, true), rest, out);
        return out;
    }
    R.expand = function (value, basePath, pattern) { return expand(value, basePath, tokens(pattern), []); };

    function isData(node) { return node && (node.kind === 'list' || node.kind === 'map'); }

    /** A declaration from one concrete path the server says points at `target`. */
    function declFromPath(cur, path, target) {
        var r = S.resolve(path, cur);
        if (!r) return null;
        var n = r.node, rest = r.rest;
        var coll = cur.coll && cur.coll[n.path];
        if (!rest.length) return isData(n) ? null : { kind: 'setting', path: n.path, target: target };
        if (coll && !coll.virtual && rest.length >= 2) {
            var isSub = coll.sublists.some(function (s) { return s.field === rest[1]; });
            if (isSub && rest.length >= 3) return { kind: 'sub', host: n.path, sub: rest[1], pattern: PH.fieldPattern(rest.slice(3)), target: target };
            if (!isSub) return { kind: 'field', host: n.path, pattern: PH.fieldPattern(rest.slice(1)), target: target };
            return null;
        }
        if (isData(n) && rest.length >= 2) return { kind: 'field', host: n.path, pattern: PH.fieldPattern(rest.slice(1)), target: target };
        return null;
    }

    function sameDecl(a, b) {
        return a.kind === b.kind && a.target === b.target && (a.path || '') === (b.path || '') &&
            (a.host || '') === (b.host || '') && (a.sub || '') === (b.sub || '') && (a.pattern || '') === (b.pattern || '');
    }

    /** Collect every reference declaration once per load; fills missing `ref` metadata from the server's usage. */
    R.prepare = function (cur) {
        var decls = [];
        var list = Object.keys(cur.nodes).map(function (p) { return cur.nodes[p]; });
        list.forEach(function (n) {
            var m = n.meta || {};
            if (m.ref && !isData(n)) decls.push({ kind: 'setting', path: n.path, target: m.ref, refField: m.refField });
            if (isData(n) && m.fields) {
                Object.keys(m.fields).forEach(function (p) {
                    var f = m.fields[p];
                    if (f && f.ref) decls.push({ kind: 'field', host: n.path, pattern: p, target: f.ref, refField: f.refField });
                });
            }
        });
        Object.keys(cur.coll || {}).forEach(function (cp) {
            cur.coll[cp].sublists.forEach(function (s) {
                var fields = (s.meta && s.meta.fields) || {};
                Object.keys(fields).forEach(function (p) {
                    var f = fields[p];
                    if (f && f.ref) decls.push({ kind: 'sub', host: cp, sub: s.field, pattern: p, target: f.ref, refField: f.refField });
                });
            });
        });
        var refs = (cur.data && cur.data.refs) || {};
        Object.keys(refs).forEach(function (target) {
            var used = (refs[target] && refs[target].usedBy) || {};
            Object.keys(used).forEach(function (key) {
                (used[key] || []).forEach(function (u) {
                    var d = u && u.path ? declFromPath(cur, u.path, target) : null;
                    if (!d || decls.some(function (x) { return sameDecl(x, d); })) return;
                    decls.push(d);
                    // So the field shows as a reference dropdown like a declared one.
                    if (d.kind === 'setting') { var sn = cur.nodes[d.path]; sn.meta = sn.meta || {}; sn.meta.ref = target; }
                    else if (d.kind === 'field') {
                        var hn = cur.nodes[d.host]; hn.meta = hn.meta || {}; hn.meta.fields = hn.meta.fields || {};
                        hn.meta.fields[d.pattern] = Object.assign({}, hn.meta.fields[d.pattern] || {}, { ref: target });
                    } else if (d.kind === 'sub') {
                        cur.coll[d.host].sublists.forEach(function (s) {
                            if (s.field !== d.sub) return;
                            s.meta = s.meta || {}; s.meta.fields = s.meta.fields || {};
                            s.meta.fields[d.pattern] = Object.assign({}, s.meta.fields[d.pattern] || {}, { ref: target });
                        });
                    }
                });
            });
        });
        cur.refDecls = decls;
        cur.refTargets = {};
        decls.forEach(function (d) { cur.refTargets[d.target] = true; });
        Object.keys(refs).forEach(function (t) { cur.refTargets[t] = true; });
        cur._meta = {};
    };

    R.isTarget = function (path) { return !!(S.cur && S.cur.refTargets && S.cur.refTargets[path]); };

    /** The keys a reference to `target` may hold, from the working copy when the target is in this script. */
    R.keys = function (target, refField) {
        var cur = S.cur;
        if (!cur) return [];
        var coll = cur.coll && cur.coll[target];
        if (coll && coll.virtual) return coll.rowKeys().map(String);
        var r = S.resolve(target);
        var v = r ? S.get(target) : undefined;
        if (v !== undefined) {
            if (refField) {
                var rows = Array.isArray(v) ? v : PH.isPlainObj(v) ? Object.keys(v).filter(function (k) { return k !== '__int_keys'; }).map(function (k) { return v[k]; }) : [];
                return rows.map(function (row) { return PH.isPlainObj(row) ? row[refField] : undefined; })
                    .filter(function (x) { return x !== undefined && x !== null && x !== ''; }).map(String);
            }
            if (PH.isPlainObj(v)) return Object.keys(v).filter(function (k) { return k !== '__int_keys'; });
            if (Array.isArray(v) && !v.length) return [];
            // A list of names (a role-like list) is its own set of keys.
            if (Array.isArray(v) && v.every(function (x) { return typeof x === 'string' || typeof x === 'number'; })) return v.map(String);
        }
        var sr = cur.data && cur.data.refs && cur.data.refs[target];
        return sr && Array.isArray(sr.keys) ? sr.keys.map(String) : [];
    };

    /** A readable name for a target's key: its row's label, when it has one. */
    R.titleOf = function (target, key) {
        var cur = S.cur;
        var coll = cur && cur.coll && cur.coll[target];
        if (coll && coll.virtual) return '';
        var v = S.get(target);
        var row = null;
        if (PH.isPlainObj(v) && Object.prototype.hasOwnProperty.call(v, key)) row = v[key];
        if (!PH.isPlainObj(row)) return '';
        var tn = cur.nodes[target];
        var tf = tn ? F.metaFor(tn).titleField : null;
        var t = (tf && row[tf]) || row.label || row.name || row.title;
        return typeof t === 'string' && t !== String(key) ? t : '';
    };

    /** "catalog", "store type": what one row of the target is called. */
    R.itemLabel = function (target) {
        var cur = S.cur;
        var tn = cur && cur.nodes[target];
        var m = tn ? F.metaFor(tn) : {};
        if (m.itemLabel) return m.itemLabel;
        if (tn) return PH.singular(F.labelFor(tn));
        return PH.singular(PH.readable(String(target).split('.').pop()));
    };

    R.label = function (target) {
        var tn = S.cur && S.cur.nodes[target];
        return tn ? F.labelFor(tn) : PH.readable(String(target).split('.').pop());
    };

    /** Is this value a key of the target? Empty and false mean "none" and are never flagged. */
    R.known = function (target, value, refField) {
        if (value === false || value === null || value === undefined || value === '') return true;
        // A target this page cannot see (another script, an older server) flags nothing.
        var cur = S.cur;
        var seen = cur && (S.node(target) || (cur.coll && cur.coll[target]) || (cur.data && cur.data.refs && cur.data.refs[target]));
        if (!seen) return true;
        return R.keys(target, refField).indexOf(String(value)) !== -1;
    };

    function rowName(listNode, row) {
        return PH.L && PH.L.titleOf ? PH.L.titleOf(listNode, row) : String(row.key);
    }

    var usageMemo = { stamp: null, byTarget: {} };
    S.on(function () { usageMemo.stamp = null; });

    /**
     * Where each key of `target` is used: { key: [{ path, label }] }, from the
     * working copy. Falls back to the server's list when nothing in this
     * script declares the reference.
     */
    R.usage = function (target) {
        var cur = S.cur;
        if (!cur) return {};
        var stamp = cur.id + ':' + cur.pending.length + ':' + (cur._stamp || 0);
        if (usageMemo.stamp !== stamp) { usageMemo = { stamp: stamp, byTarget: {} }; }
        if (usageMemo.byTarget[target]) return usageMemo.byTarget[target];
        var out = {};
        var seenPath = {};
        function add(key, path, label) {
            // A list of keys (chips) counts once per entry.
            if (Array.isArray(key)) { key.forEach(function (k, i) { add(k, path + '[' + (i + 1) + ']', label); }); return; }
            if (key === false || key === null || key === undefined || key === '' || typeof key === 'object') return;
            // The server declares a sublist's field both on the sublist and on the collection (sell[].item): count it once.
            var cp = PH.canon(path);
            if (seenPath[cp]) return;
            seenPath[cp] = true;
            var k = String(key);
            (out[k] = out[k] || []).push({ path: path, label: label });
        }
        var decls = (cur.refDecls || []).filter(function (d) { return d.target === target; });
        decls.forEach(function (d) {
            if (d.kind === 'setting') {
                var sn = cur.nodes[d.path];
                add(S.get(d.path), d.path, sn ? F.labelFor(sn) : d.path);
                return;
            }
            var host = cur.nodes[d.host];
            if (!host || !PH.L) return;
            if (d.kind === 'field') {
                PH.L.rowsOf(host, F.metaFor(host)).forEach(function (row) {
                    R.expand(row.value, row.path, d.pattern).forEach(function (hit) {
                        add(d.refField && PH.isPlainObj(hit.value) ? hit.value[d.refField] : hit.value, hit.path, F.labelFor(host) + ' › ' + rowName(host, row));
                    });
                });
                return;
            }
            // A list inside each row of a collection.
            var coll = cur.coll[d.host];
            if (!coll) return;
            coll.rowKeys().forEach(function (rk) {
                var rowPath = coll.rowPath(rk);
                var arr = S.get(rowPath + '.' + d.sub);
                if (!arr || typeof arr !== 'object') return;
                var entries = Array.isArray(arr) ? arr.map(function (x, i) { return { p: rowPath + '.' + d.sub + '[' + (i + 1) + ']', v: x }; })
                    : Object.keys(arr).map(function (k) { return { p: PH.joinPath(rowPath + '.' + d.sub, k, true), v: arr[k] }; });
                entries.forEach(function (en) {
                    R.expand(en.v, en.p, d.pattern).forEach(function (hit) {
                        add(hit.value, hit.path, F.labelFor(host) + ' › ' + rk + ' › ' + coll.subLabel(d.sub));
                    });
                });
            });
        });
        if (!decls.length) {
            var sr = cur.data && cur.data.refs && cur.data.refs[target];
            if (sr && sr.usedBy) Object.keys(sr.usedBy).forEach(function (k) { (sr.usedBy[k] || []).forEach(function (u) { add(k, u.path, u.label || u.path); }); });
        }
        usageMemo.byTarget[target] = out;
        return out;
    };

    /** The "Used by N" chip for a row of a referenced target, with the list on hover and a menu to jump. */
    R.usedByChip = function (target, key, opts) {
        var list = (R.usage(target)[String(key)]) || [];
        opts = opts || {};
        if (!list.length) return opts.showZero ? h('span.ph-usedby.is-none', { title: 'Nothing refers to this ' + R.itemLabel(target) + ' yet' }, 'Not used') : null;
        var chip = h('button.ph-usedby', { type: 'button' }, [icon('link'), 'Used by ' + list.length]);
        PH.tip(chip, function () {
            return h('div', [h('div.ph-tip__text', 'Referred to by:'), h('ul.ph-tip__list', list.slice(0, 12).map(function (u) { return h('li', u.label); })),
                list.length > 12 ? h('div.ph-tip__code', '…and ' + (list.length - 12) + ' more') : null]);
        });
        chip.addEventListener('click', function (e) {
            e.stopPropagation();
            PH.menu(chip, list.slice(0, 30).map(function (u) {
                return { label: u.label, icon: 'open', onClick: function () { PH.reveal(u.path); } };
            }), { minWidth: 300 });
        });
        return chip;
    };

    // ------------------------------------------------------------ controls --
    // Each control: build(spec, commit) -> { el, set(value), disable(bool) }.
    // commit(value) queues the change; it returns false when the value was refused.

    function numberParse(text) {
        var t = String(text).trim().replace(',', '.');
        if (!/^-?(\d+\.?\d*|\.\d+)(e-?\d+)?$/i.test(t)) return null;
        return Number(t);
    }

    function ctlToggle(spec, commit) {
        var btn = h('button.ph-toggle', { type: 'button', role: 'switch' }, [h('span.ph-toggle__track', h('span.ph-toggle__knob')), h('span.ph-toggle__text')]);
        var val = !!spec.value;
        function show() {
            btn.setAttribute('aria-checked', val ? 'true' : 'false');
            btn.classList.toggle('is-on', val);
            btn.querySelector('.ph-toggle__text').textContent = val ? 'On' : 'Off';
        }
        btn.addEventListener('click', function () { if (btn.disabled) return; val = !val; show(); commit(val); });
        show();
        return { el: btn, set: function (v) { val = !!v; show(); }, disable: function (d) { btn.disabled = d; } };
    }

    function ctlNumber(spec, commit, err) {
        var m = spec.meta || {};
        var min = typeof m.min === 'number' ? m.min : null, max = typeof m.max === 'number' ? m.max : null;
        var isInt = Number.isInteger(spec.value) && !(typeof m.step === 'number' && !Number.isInteger(m.step));
        var step = typeof m.step === 'number' ? m.step : (isInt ? 1 : 0.1);
        var input = h('input.ph-input.ph-input--num', { type: 'text', inputmode: 'decimal', spellcheck: 'false', value: PH.num(spec.value) });
        var slider = null;
        if (min !== null && max !== null && max > min) {
            slider = h('input.ph-range', { type: 'range', min: String(min), max: String(max), step: String(step), value: String(spec.value) });
            slider.addEventListener('input', function () {
                var v = Number(slider.value);
                input.value = PH.num(v);
                paint();
                commit(v);
                err('');
            });
        }
        function paint() {
            if (!slider) return;
            var pct = (Number(slider.value) - min) / (max - min) * 100;
            slider.style.setProperty('--pct', Math.max(0, Math.min(100, pct)) + '%');
        }
        function check(text) {
            var v = numberParse(text);
            if (v === null) return { bad: 'Enter a number.' };
            if (min !== null && v < min) return { bad: 'The lowest allowed is ' + PH.num(min) + '.' };
            if (max !== null && v > max) return { bad: 'The highest allowed is ' + PH.num(max) + '.' };
            return { v: v };
        }
        input.addEventListener('input', function () {
            var r = check(input.value);
            if (r.bad) { err(r.bad); input.classList.add('is-bad'); return; }
            err(''); input.classList.remove('is-bad');
            if (slider) { slider.value = String(r.v); paint(); }
            commit(r.v);
        });
        input.addEventListener('keydown', function (e) {
            if (e.key !== 'ArrowUp' && e.key !== 'ArrowDown') return;
            var r = check(input.value);
            if (r.bad) return;
            e.preventDefault();
            var mult = e.shiftKey ? 10 : 1;
            var v = r.v + (e.key === 'ArrowUp' ? step : -step) * mult;
            v = Math.round(v * 1e6) / 1e6;
            if (min !== null) v = Math.max(min, v);
            if (max !== null) v = Math.min(max, v);
            input.value = PH.num(v);
            if (slider) { slider.value = String(v); paint(); }
            commit(v);
        });
        input.addEventListener('blur', function () {
            // Leave a box that was refused showing what is actually queued.
            if (check(input.value).bad) { input.value = PH.num(spec.get()); input.classList.remove('is-bad'); err(''); }
        });
        paint();
        var wrap = h('div.ph-num' + (slider ? '.has-slider' : ''), [
            slider,
            h('div.ph-num__box', [input, m.unit ? h('span.ph-num__unit', m.unit) : null]),
        ]);
        return {
            el: wrap,
            set: function (v) { input.value = PH.num(v); if (slider) { slider.value = String(v); paint(); } },
            disable: function (d) { input.disabled = d; if (slider) slider.disabled = d; },
        };
    }

    function ctlHeading(spec, commit, err) {
        var num = ctlNumber(Object.assign({}, spec, { meta: Object.assign({ min: 0, max: 360, step: 1, unit: '°' }, spec.meta || {}) }), function (v) {
            dial(v); return commit(v);
        }, err);
        var needle = h('span.ph-dial__needle');
        var dialEl = h('span.ph-dial', { title: 'Which way it faces' }, [h('span.ph-dial__n', 'N'), needle]);
        function dial(v) { needle.style.transform = 'rotate(' + (-Number(v || 0)) + 'deg)'; }
        dial(spec.value);
        var btn = h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'Use the way your character is facing now' }, [icon('compass'), 'Use my heading']);
        btn.addEventListener('click', function () {
            F.myPosition().then(function (p) {
                if (!p) return;
                var v = Math.round(p.heading * 100) / 100;
                num.set(v); dial(v); commit(v);
            });
        });
        return {
            el: h('div.ph-heading', [dialEl, num.el, btn]),
            set: function (v) { num.set(v); dial(v); },
            disable: function (d) { num.disable(d); btn.disabled = d; },
        };
    }

    function ctlText(spec, commit, err, multiline) {
        var m = spec.meta || {};
        var input = multiline
            ? h('textarea.ph-input.ph-textarea', { rows: '3', spellcheck: 'false', placeholder: m.placeholder || '' })
            : h('input.ph-input', { type: 'text', spellcheck: 'false', placeholder: m.placeholder || '' });
        input.value = spec.value == null ? '' : String(spec.value);
        if (typeof m.maxLength === 'number') input.maxLength = m.maxLength;
        function grow() {
            if (!multiline) return;
            input.style.height = 'auto';
            input.style.height = Math.min(320, input.scrollHeight + 2) + 'px';
        }
        input.addEventListener('input', function () {
            grow();
            var bad = spec.validate ? spec.validate(input.value) : null;
            input.classList.toggle('is-bad', !!bad);
            err(bad || '');
            if (!bad) commit(input.value);
        });
        setTimeout(grow, 0);
        return {
            el: input,
            set: function (v) { input.value = v == null ? '' : String(v); grow(); },
            disable: function (d) { input.readOnly = d; input.classList.toggle('is-disabled', d); },
        };
    }

    var WEBHOOK = /^https:\/\/(?:(?:ptb|canary)\.)?discord(?:app)?\.com\/api\/webhooks\/\d+\/[\w-]+\/?$/;

    function ctlWebhook(spec, commit, err) {
        var state = h('span.ph-webhook__state');
        function mark(v) {
            var ok = WEBHOOK.test(v);
            state.className = 'ph-webhook__state' + (v === '' ? '' : ok ? ' is-ok' : ' is-bad');
            PH.clear(state);
            if (v === '') state.appendChild(document.createTextNode('Off (empty)'));
            else if (ok) { state.appendChild(icon('check')); state.appendChild(document.createTextNode('Discord webhook')); }
        }
        var text = ctlText(Object.assign({}, spec, {
            meta: Object.assign({ placeholder: 'https://discord.com/api/webhooks/…' }, spec.meta || {}),
            validate: function (v) {
                mark(v);
                if (v === '' || WEBHOOK.test(v)) return null;
                return 'Not a Discord webhook URL. It should start with https://discord.com/api/webhooks/ (or leave it empty to turn it off).';
            },
        }), commit, err);
        mark(String(spec.value || ''));
        var v0 = String(spec.value || '');
        if (v0 !== '' && !WEBHOOK.test(v0)) {
            PH.clear(state);
            state.className = 'ph-webhook__state is-warn';
            state.appendChild(document.createTextNode('Not set up yet'));
        }
        return { el: h('div.ph-webhook', [text.el, state]), set: function (v) { text.set(v); mark(String(v || '')); }, disable: text.disable };
    }

    var SWATCHES = ['#f3e7d8', '#f2cda0', '#c2a15c', '#cc7459', '#b4514e', '#8b2e2e', '#7fa66a', '#3f6b3a',
        '#5d8fa6', '#2f4f6f', '#8c73b3', '#5b4a7a', '#9a8c74', '#5a4b3c', '#2b2522', '#ffffff'];

    function ctlColor(spec, commit, err) {
        var asNumber = typeof spec.value === 'number';
        var hashPrefix = !asNumber && String(spec.value || '').charAt(0) === '#';
        function toHex(v) {
            if (typeof v === 'number') return '#' + ('000000' + Math.max(0, Math.floor(v)).toString(16)).slice(-6);
            var s = String(v || '').trim();
            if (/^#?[0-9a-f]{6}$/i.test(s)) return (s.charAt(0) === '#' ? s : '#' + s).toLowerCase();
            if (/^#?[0-9a-f]{3}$/i.test(s)) { s = s.replace('#', ''); return ('#' + s[0] + s[0] + s[1] + s[1] + s[2] + s[2]).toLowerCase(); }
            return null;
        }
        function fromHex(hex) {
            if (asNumber) return parseInt(hex.slice(1), 16);
            return hashPrefix ? hex : hex.slice(1);
        }
        var sw = h('button.ph-swatch', { type: 'button', title: 'Pick a colour' });
        var input = h('input.ph-input.ph-input--mono', { type: 'text', spellcheck: 'false' });
        function show(v) {
            var hex = toHex(v);
            sw.style.background = hex || 'transparent';
            sw.classList.toggle('is-empty', !hex);
            input.value = asNumber ? (hex || '') : String(v == null ? '' : v);
        }
        input.addEventListener('input', function () {
            var hex = toHex(input.value);
            if (!hex) { err('Use a hex colour like #c9a84c.'); input.classList.add('is-bad'); return; }
            err(''); input.classList.remove('is-bad');
            sw.style.background = hex;
            commit(fromHex(hex));
        });
        sw.addEventListener('click', function () {
            if (sw.disabled) return;
            var grid = h('div.ph-swatches', SWATCHES.map(function (c) {
                return h('button.ph-swatch', { type: 'button', title: c, style: { background: c }, onclick: function () { pop.close(); show(fromHex(c)); commit(fromHex(c)); } });
            }));
            var pop = PH.popover(sw, h('div.ph-colorpop', [h('div.ph-colorpop__title', 'Colours'), grid, h('div.ph-colorpop__hint', 'Or type any hex colour in the box.')]), { width: 250 });
        });
        show(spec.value);
        return { el: h('div.ph-color', [sw, input]), set: show, disable: function (d) { sw.disabled = d; input.readOnly = d; } };
    }

    /** Select from hub.json options. Values keep their JSON type (false, 3, "en"). */
    function ctlSelect(spec, commit) {
        var opts = (spec.meta.options || []).map(function (o) {
            return (o && typeof o === 'object' && 'value' in o) ? o : { value: o, label: PH.fmtValue(o) };
        });
        var btn = h('button.ph-select', { type: 'button', 'aria-haspopup': 'listbox' }, [h('span.ph-select__label'), icon('chevdown')]);
        var val = spec.value;
        function labelOf(v) {
            for (var i = 0; i < opts.length; i++) if (PH.deepEqual(opts[i].value, v)) return opts[i].label;
            return PH.fmtValue(v) + ' (not in the list)';
        }
        function show() { btn.querySelector('.ph-select__label').textContent = labelOf(val); }
        btn.addEventListener('click', function () {
            if (btn.disabled) return;
            PH.menu(btn, opts.map(function (o) {
                var on = PH.deepEqual(o.value, val);
                return { label: o.label, icon: on ? 'check' : null, active: on, hint: typeof o.value === 'string' ? '' : PH.fmtValue(o.value),
                    onClick: function () { val = o.value; show(); commit(PH.clone(o.value)); } };
            }), { minWidth: Math.max(240, btn.offsetWidth) });
        });
        show();
        return { el: btn, set: function (v) { val = v; show(); }, disable: function (d) { btn.disabled = d; } };
    }

    /** A single string chosen with a picker (item, job, group, role, key), free text allowed. */
    function ctlPicked(spec, commit, kind) {
        var val = spec.value;
        var face = h('span.ph-picked__face');
        var btn = h('button.ph-picked', { type: 'button' }, [face, icon('chevdown')]);
        function show() {
            PH.clear(face);
            var name = PH.isHash(val) ? val.name : (val == null ? '' : String(val));
            if (kind === 'item') {
                var img = h('img.ph-picked__img', { alt: '' });
                img.style.visibility = 'hidden';
                var lab = h('span.ph-picked__label', name || 'Choose an item');
                var sub = h('span.ph-picked__sub', name ? name : '');
                face.appendChild(h('span.ph-pick__imgwrap', img));
                face.appendChild(h('span.ph-picked__main', [lab, sub]));
                if (name) F.itemInfo(name).then(function (info) {
                    if (!info) { sub.textContent = name + ' · not found in the item list'; sub.classList.add('is-warn'); return; }
                    lab.textContent = info.label || name;
                    if (info.image) {
                        img.src = info.image;
                        img.onload = function () { img.style.visibility = ''; };
                    }
                });
            } else {
                face.appendChild(icon(kind === 'job' ? 'badge' : kind === 'group' ? 'shield' : kind === 'role' ? 'users' : 'keyboard'));
                face.appendChild(h('span.ph-picked__label' + (kind === 'key' ? '.is-mono' : ''), name || 'Choose…'));
            }
        }
        btn.addEventListener('click', function () {
            if (btn.disabled) return;
            F.pick({
                anchor: btn, fetch: F.fetchers[kind], allowFree: true,
                placeholder: kind === 'item' ? 'Search items by name or label…' : kind === 'job' ? 'Search jobs…' : kind === 'key' ? 'Search controls…' : 'Search…',
                onPick: function (v) {
                    val = PH.isHash(spec.value) ? { __type: 'hash', name: String(v) } : v;
                    show();
                    commit(PH.clone(val));
                },
            });
        });
        show();
        return { el: btn, set: function (v) { val = v; show(); }, disable: function (d) { btn.disabled = d; } };
    }

    /**
     * A reference (§8.5): a searchable dropdown of the target's keys, the
     * value in red with "No such …" when it is not one of them, and Open to
     * jump to that row. Free text is allowed: the owner may be about to add it.
     */
    function ctlRef(spec, commit) {
        var m = spec.meta || {};
        var target = m.ref, refField = m.refField;
        var val = spec.value;
        var what = R.itemLabel(target);
        // A field may carry both a ref and hub.json options (fish sizes): one
        // dropdown, the target's keys plus the option values, option labels where given.
        var opts = (Array.isArray(m.options) ? m.options : []).map(function (o) {
            return (o && typeof o === 'object' && 'value' in o) ? o : { value: o, label: PH.fmtValue(o) };
        });
        function optLabel(v) {
            for (var i = 0; i < opts.length; i++) if (String(opts[i].value) === String(v)) return opts[i].label;
            return '';
        }
        function allKeys() {
            var keys = R.keys(target, refField).slice();
            opts.forEach(function (o) { if (o.value !== false && keys.indexOf(String(o.value)) === -1) keys.push(String(o.value)); });
            return keys;
        }
        function titleFor(k) { return optLabel(k) || R.titleOf(target, k); }
        var face = h('span.ph-picked__face');
        var btn = h('button.ph-picked.ph-ref', { type: 'button', title: 'Choose a ' + what + ' from ' + R.label(target) }, [face, icon('chevdown')]);
        var openBtn = h('button.ph-btn.ph-btn--ghost.ph-btn--sm.ph-ref__open', { type: 'button', title: 'Go to this ' + what }, [icon('open'), 'Open']);
        var note = h('div.ph-ref__note');
        function isNone(v) { return v === false || v === null || v === undefined || v === ''; }
        function show() {
            PH.clear(face);
            PH.clear(note);
            var known = isNone(val) || allKeys().indexOf(String(val)) !== -1;
            btn.classList.toggle('is-unknown', !known);
            if (isNone(val)) {
                face.appendChild(icon('link'));
                face.appendChild(h('span.ph-picked__label.is-none', val === false ? 'None (off)' : 'None'));
            } else {
                face.appendChild(icon(known ? 'link' : 'alert'));
                var title = known ? titleFor(val) : '';
                face.appendChild(h('span.ph-picked__main', [
                    h('span.ph-picked__label', title || String(val)),
                    title ? h('span.ph-picked__sub', String(val)) : null,
                ]));
                if (!known) {
                    note.appendChild(icon('alert'));
                    note.appendChild(h('span', ['No such ' + what + ' “' + val + '” in ', h('b', R.label(target)), '. The script will not find it: pick one from the list, or add it there first.']));
                }
            }
            openBtn.classList.toggle('is-hidden', isNone(val) || !known);
        }
        btn.addEventListener('click', function () {
            if (btn.disabled) return;
            F.pick({
                anchor: btn, allowFree: true, freeLabel: 'Use', width: 380,
                placeholder: 'Search ' + PH.pluralOf(what) + '…',
                fetch: function (q) {
                    var usage = R.usage(target);
                    var out = allKeys().filter(function (k) { return PH.matches(q, [k, titleFor(k)]); }).map(function (k) {
                        var t = titleFor(k);
                        var n = (usage[k] || []).length;
                        return { value: k, label: t || k, sub: (t ? k + ' · ' : '') + (n ? 'used by ' + n : 'not used yet'), icon: 'link' };
                    });
                    // A setting that is off (false) can be turned off again from here.
                    // false means "none" (a store that does not buy): offer it when the value, the file or the default uses it.
                    var dflt = spec.node && spec.node.default;
                    if (spec.value === false || S.getBase(spec.path) === false || dflt === false || m.allowNone) {
                        if (PH.matches(q, ['none', 'off'])) out.unshift({ value: false, label: 'None (off)', sub: 'Leaves this empty', icon: 'x' });
                    }
                    return Promise.resolve(out);
                },
                onPick: function (v) {
                    if (typeof v === 'string' && typeof spec.value === 'number' && /^-?\d+$/.test(v)) v = Number(v);
                    // An option keeps its own JSON type (a number stays a number).
                    opts.forEach(function (o) { if (String(o.value) === String(v)) v = o.value; });
                    val = v;
                    show();
                    commit(PH.clone(v));
                },
            });
        });
        openBtn.addEventListener('click', function () { PH.openRef(target, val, refField); });
        show();
        return {
            el: h('div.ph-refctl', [h('div.ph-ref__row', [btn, openBtn]), note]),
            set: function (v) { val = v; show(); },
            disable: function (d) { btn.disabled = d; },
        };
    }

    function ctlHash(spec, commit, err) {
        var t = ctlText(Object.assign({}, spec, {
            value: spec.value ? spec.value.name : '',
            validate: function (v) { return /^[A-Za-z0-9_]+$/.test(v) ? null : 'A control or hash name: letters, digits and _ only.'; },
        }), function (v) { return commit({ __type: 'hash', name: v }); }, err);
        t.el.classList.add('ph-input--mono');
        return {
            el: h('div.ph-hash', [h('span.ph-hash__tick', '`'), t.el, h('span.ph-hash__tick', '`')]),
            set: function (v) { t.set(v ? v.name : ''); },
            disable: t.disable,
        };
    }

    F.myPosition = function () {
        return PH.post('hubPosition', {}).then(function (r) {
            if (r && r.ok && r.value) return r.value;
            PH.toast({ kind: 'error', title: 'No position', text: 'Could not read your character’s position.' });
            return null;
        }, function () {
            PH.toast({ kind: 'error', title: 'No position', text: 'Could not read your character’s position.' });
            return null;
        });
    };

    function ctlVector(spec, commit, err) {
        var v = PH.clone(spec.value);
        var axes = PH.vecAxes(v);
        var inputs = {};
        var boxes = axes.map(function (a) {
            var inp = h('input.ph-input.ph-input--num', { type: 'text', inputmode: 'decimal', spellcheck: 'false', value: PH.num(v[a]) });
            inp.addEventListener('input', function () {
                var n = numberParse(inp.value);
                if (n === null) { inp.classList.add('is-bad'); err('Each axis needs a number.'); return; }
                inp.classList.remove('is-bad'); err('');
                v[a] = n;
                commit(PH.clone(v));
            });
            inputs[a] = inp;
            return h('label.ph-vec__axis', [h('span.ph-vec__name', a === 'w' ? 'h' : a), inp]);
        });
        var use = h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: axes.length === 4 ? 'Your position, and the way you face as the fourth value' : 'Where your character stands now' },
            [icon('target'), 'Use my position']);
        use.addEventListener('click', function () {
            F.myPosition().then(function (p) {
                if (!p) return;
                v.x = p.x; v.y = p.y;
                if (axes.indexOf('z') !== -1) v.z = p.z;
                if (axes.indexOf('w') !== -1) v.w = p.heading;
                axes.forEach(function (a) { inputs[a].value = PH.num(v[a]); inputs[a].classList.remove('is-bad'); });
                err('');
                commit(PH.clone(v));
                PH.toast({ kind: 'success', title: 'Position set', text: axes.map(function (a) { return a + ' ' + PH.num(v[a]); }).join('  ') });
            });
        });
        var copy = h('button.ph-iconbtn', { type: 'button', title: 'Copy as Lua' }, icon('copy'));
        copy.addEventListener('click', function () {
            var fn = v.__type === 'vec2' ? 'vector2' : v.__type === 'vec3' ? 'vector3' : 'vector4';
            PH.copyText(fn + '(' + axes.map(function (a) { return PH.num(v[a]); }).join(', ') + ')');
            PH.toast({ kind: 'info', title: 'Copied', text: 'The coordinates are on the clipboard.' });
        });
        return {
            el: h('div.ph-vec', [h('div.ph-vec__axes.ph-vec__axes--' + axes.length, boxes), h('div.ph-vec__tools', [use, copy])]),
            set: function (nv) { v = PH.clone(nv); axes.forEach(function (a) { inputs[a].value = PH.num(v[a]); }); },
            disable: function (d) { axes.forEach(function (a) { inputs[a].readOnly = d; }); use.disabled = d; },
        };
    }

    /**
     * Chips for an array of strings (or numbers). Top-level string lists can
     * follow a role (spec.roleable): then the chips are the role's list and
     * cannot be edited here.
     */
    function ctlChips(spec, commit, err) {
        var arr = PH.clone(spec.value) || [];
        var numeric = arr.length > 0 && arr.every(function (x) { return typeof x === 'number'; });
        var picker = spec.meta.picker;
        var wrap = h('div.ph-chips');
        var chipsEl = h('div.ph-chips__list');
        var roleBar = h('div.ph-chips__role');
        var disabled = false;
        // §9.3: item names are checked one by one, and added with the type-ahead.
        var itemMode = picker === 'item' && IC.enabled();
        var notes = itemMode ? h('div.ph-chips__notes') : null;
        var deferred = false;
        wrap.appendChild(roleBar);
        wrap.appendChild(chipsEl);
        if (notes) wrap.appendChild(notes);

        function role() { return spec.roleable ? S.roleOf(spec.path) : null; }
        // A list of keys of another list (§8.5): each chip is checked, and Add picks from the keys.
        var refT = spec.meta.ref, refF = spec.meta.refField;
        function refFetch(q) {
            return Promise.resolve(R.keys(refT, refF).filter(function (k) { return arr.indexOf(k) === -1 && PH.matches(q, [k, R.titleOf(refT, k)]); }).map(function (k) {
                var t = R.titleOf(refT, k);
                return { value: k, label: t || k, sub: t ? k : '', icon: 'link' };
            }));
        }
        // Names offered from other lists of this script, several at once and
        // without being a strict reference (hub.json `suggest`): crafting's
        // Where takes bench ids and prop titles. Typing by hand stays possible.
        var sugg = Array.isArray(spec.meta.suggest) && spec.meta.suggest.length ? spec.meta.suggest : null;
        function suggestions() {
            var out = [], seen = {};
            sugg.forEach(function (src) {
                if (!src || !src.from) return;
                var rows = S.get(src.from);
                rows = Array.isArray(rows) ? rows : PH.isPlainObj(rows) ? Object.keys(rows).filter(function (k) { return k !== '__int_keys'; }).map(function (k) { return rows[k]; }) : [];
                rows.forEach(function (row) {
                    var val = src.field ? (PH.isPlainObj(row) ? row[src.field] : undefined) : row;
                    if (val === undefined || val === null || val === '' || typeof val === 'object') return;
                    val = String(val);
                    if (src.lower) val = val.toLowerCase();
                    if (seen[val]) return;
                    seen[val] = true;
                    var name = src.label && PH.isPlainObj(row) && typeof row[src.label] === 'string' ? row[src.label] : '';
                    out.push({ value: val, label: name && name !== val ? name : val, sub: [src.group, name && name !== val ? val : ''].filter(Boolean).join(' · '), icon: src.icon || 'link' });
                });
            });
            return out;
        }
        function suggestFetch(q) {
            return Promise.resolve(suggestions().filter(function (o) { return arr.indexOf(o.value) === -1 && PH.matches(q, [o.value, o.label]); }));
        }

        function draw() {
            PH.clear(chipsEl);
            PH.clear(roleBar);
            var linked = role();
            wrap.classList.toggle('is-linked', !!linked);
            if (notes) PH.clear(notes);
            arr.forEach(function (x, i) {
                var bad = refT && !R.known(refT, x, refF);
                var st = itemMode ? IC.status(typeof x === 'string' ? x : String(x)) : null;
                var warn = false;
                var title = bad ? 'No such ' + R.itemLabel(refT) + ' in ' + R.label(refT) : null;
                if (sugg && !suggestions().some(function (o) { return o.value === String(x); })) { warn = true; title = spec.meta.suggestNote || 'Not one of the known names. Check the spelling.'; }
                if (st && st.state === 'missing') { bad = true; title = 'Item does not exist' + (st.suggest ? ': did you mean ' + st.suggest + '?' : ''); }
                else if (st && st.state === 'noicon') { warn = true; title = 'No icon: add ' + IC.iconFile(st.name); }
                var chip = h('span.ph-chip' + (bad ? '.is-bad' : warn ? '.is-warn' : ''), { title: title },
                    [bad || warn ? icon('alert') : null, h('span.ph-chip__text', String(x))]);
                if (notes && st && (st.state === 'missing' || st.state === 'noicon')) {
                    var line = h('div.ph-itemnote.ph-itemnote--chip', [h('code.ph-itemnote__name', String(x))]);
                    var body = h('span.ph-itemnote__body');
                    line.appendChild(body);
                    itemNote(body, st, function (fix) {
                        if (disabled || role()) return;
                        if (arr.indexOf(fix) !== -1) arr.splice(i, 1); else arr[i] = fix;
                        draw(); commit(PH.clone(arr));
                    });
                    notes.appendChild(line);
                }
                if (!linked && !disabled) {
                    chip.appendChild(h('button.ph-chip__x', { type: 'button', title: 'Remove ' + x, onclick: function () {
                        arr.splice(i, 1); draw(); commit(PH.clone(arr));
                    } }, icon('x')));
                }
                chipsEl.appendChild(chip);
            });
            if (!arr.length && (linked || disabled)) chipsEl.appendChild(h('span.ph-chips__empty', linked ? 'The role is empty.' : 'Empty'));

            if (!linked && !disabled) {
                if (itemMode) {
                    var tin = h('input.ph-chips__input.ph-chips__input--item', { type: 'text', placeholder: arr.length ? 'Add an item…' : 'Type an item name…', spellcheck: 'false', autocomplete: 'off' });
                    var addName = function (v) {
                        v = String(v == null ? '' : v).trim();
                        if (!v) return;
                        if (arr.indexOf(v) === -1) { arr.push(v); commit(PH.clone(arr)); }
                        draw();
                        var again = chipsEl.querySelector('.ph-chips__input');
                        if (again) again.focus();
                    };
                    IC.typeahead(tin, {
                        anchor: chipsEl, onPick: addName, onEnter: addName,
                        onCancel: function () { if (!tin.value) return false; tin.value = ''; return true; },
                        exclude: function (n) { return arr.indexOf(n) !== -1; },
                    });
                    tin.addEventListener('keydown', function (e) {
                        if (e.key === 'Backspace' && !tin.value && arr.length) {
                            arr.pop(); draw(); commit(PH.clone(arr));
                            var again = chipsEl.querySelector('.ph-chips__input');
                            if (again) again.focus();
                        }
                    });
                    // A redraw that waited for the box to lose focus.
                    tin.addEventListener('blur', function () { setTimeout(function () { if (deferred && !chipsEl.contains(document.activeElement)) { deferred = false; draw(); } }, 150); });
                    chipsEl.appendChild(tin);
                } else if (refT || sugg || (picker && F.fetchers[picker])) {
                    var addBtn = h('button.ph-chip.ph-chip--add', { type: 'button' }, [icon('plus'), 'Add']);
                    addBtn.addEventListener('click', function () {
                        F.pick({ anchor: addBtn, fetch: refT ? refFetch : sugg ? suggestFetch : F.fetchers[picker], allowFree: true, freeLabel: 'Add',
                            onPick: function (v) { if (arr.indexOf(v) === -1) { arr.push(v); draw(); commit(PH.clone(arr)); } } });
                    });
                    chipsEl.appendChild(addBtn);
                } else {
                    var input = h('input.ph-chips__input', { type: 'text', placeholder: numeric ? 'Add a number…' : 'Add…', spellcheck: 'false' });
                    // Returns true when the typed text became a chip.
                    var addTyped = function () {
                        var t = input.value.trim();
                        if (!t) return false;
                        var val = t;
                        if (numeric) {
                            val = numberParse(t);
                            if (val === null) { err('Numbers only in this list.'); return false; }
                        }
                        err('');
                        if (arr.indexOf(val) === -1) arr.push(val);
                        input.value = '';
                        draw();
                        commit(PH.clone(arr));
                        return true;
                    };
                    // Text left in the box is an entry too: without this, typing a
                    // name and pressing Done saved an empty list.
                    input.addEventListener('blur', function () { addTyped(); });
                    input.addEventListener('keydown', function (e) {
                        if (e.key === 'Enter' || e.key === ',') {
                            e.preventDefault();
                            if (addTyped()) chipsEl.querySelector('.ph-chips__input').focus();
                        } else if (e.key === 'Backspace' && !input.value && arr.length) {
                            arr.pop(); draw(); commit(PH.clone(arr));
                            chipsEl.querySelector('.ph-chips__input').focus();
                        }
                    });
                    chipsEl.appendChild(input);
                }
            }

            if (spec.roleable && !numeric) {
                if (linked) {
                    var rl = S.roles && S.roles.roles && S.roles.roles[linked];
                    roleBar.appendChild(h('span.ph-rolepill', [icon('link'), 'Follows the role ', h('b', rl ? (rl.label || linked) : linked),
                        h('span.ph-rolepill__note', ' — edit the names on the Roles page')]));
                    if (!disabled) roleBar.appendChild(h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () {
                        S.queue({ op: 'unlinkRole', path: spec.path });
                        arr = PH.clone(S.get(spec.path)) || [];
                        draw();
                        if (spec.onRefresh) spec.onRefresh();
                    } }, [icon('unlink'), 'Unlink']));
                    roleBar.appendChild(h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { PH.go.roles(linked); } }, [icon('users'), 'Open role']));
                    if (!rl) F.loadRoles().then(function () { if (role() === linked) draw(); });
                } else if (!disabled) {
                    var linkBtn = h('button.ph-btn.ph-btn--ghost.ph-btn--sm.ph-chips__link', { type: 'button', title: 'Make this list follow one of the master job or group lists' }, [icon('link'), 'Link to role']);
                    linkBtn.addEventListener('click', function () {
                        F.loadRoles().then(function (rv) {
                            var want = picker === 'group' ? 'groups' : picker === 'job' ? 'jobs' : null;
                            var names = Object.keys(rv.roles).filter(function (k) { return !want || rv.roles[k].kind === want; });
                            if (!names.length) {
                                PH.toast({ kind: 'info', title: 'No roles yet', text: 'Create one on the Roles page first.', actions: [{ label: 'Open Roles', primary: true, onClick: function () { PH.go.roles(); } }] });
                                return;
                            }
                            PH.menu(linkBtn, names.map(function (k) {
                                var r = rv.roles[k];
                                return { label: r.label || k, icon: 'users', hint: (r.list || []).length + ' names', onClick: function () {
                                    S.queue({ op: 'linkRole', path: spec.path, role: k });
                                    arr = PH.clone(S.get(spec.path)) || [];
                                    draw();
                                    if (spec.onRefresh) spec.onRefresh();
                                } };
                            }).concat(['-', { label: 'Manage roles…', icon: 'users', onClick: function () { PH.go.roles(); } }]), { minWidth: 280 });
                        });
                    });
                    roleBar.appendChild(linkBtn);
                }
            }
        }
        draw();
        if (itemMode) {
            IC.loadList();
            // An answer about an item arrived: redraw, but never under the cursor of someone typing.
            IC.watch(wrap, function () { if (chipsEl.contains(document.activeElement)) deferred = true; else draw(); });
        }
        return {
            el: wrap,
            set: function (v) { arr = PH.clone(v) || []; draw(); },
            disable: function (d) { disabled = d; draw(); },
        };
    }

    /**
     * A value that is either "no restriction" (one fixed value, nearly always
     * 0) or a list of names: Job = 0, or Job = { 'blacksmith' }. hub.json says
     * which fields work this way:
     *
     *     "Job": { "any": { "value": 0, "label": "Anyone" } }
     *
     * Without it, a field holding 0 is a number box, and the only thing a
     * person can type into a number box is another number -- which matches
     * no job and locks the recipe for everyone. That is how Britannia ended
     * up with Job = 8. Here a switch picks the mode, and the list mode is the
     * ordinary chips control, so the [] picker (job, item) still applies.
     */
    function ctlAnyOr(spec, commit, err) {
        var any = spec.meta.any || {};
        var anyValue = any.value === undefined ? 0 : any.value;
        var anyLabel = any.label || 'Any';
        var someLabel = any.listLabel || 'Only these';
        var v = PH.clone(spec.value);
        var disabled = false;
        var chips = null;
        var wrap = h('div.ph-anyor');
        var seg = h('div.ph-seg');
        var body = h('div.ph-anyor__body');
        var note = h('div.ph-anyor__note');

        function isAny(x) { return !Array.isArray(x) && (x === anyValue || x === null || x === undefined); }
        function mode() { return Array.isArray(v) ? 'some' : isAny(v) ? 'any' : 'bad'; }

        var anyBtn = h('button.ph-seg__btn', { type: 'button' }, anyLabel);
        var someBtn = h('button.ph-seg__btn', { type: 'button' }, someLabel);
        anyBtn.addEventListener('click', function () {
            if (disabled || mode() === 'any') return;
            v = anyValue; commit(anyValue); draw();
        });
        someBtn.addEventListener('click', function () {
            if (disabled || mode() === 'some') return;
            v = []; commit([]); draw();
        });
        seg.appendChild(anyBtn); seg.appendChild(someBtn);
        wrap.appendChild(seg); wrap.appendChild(body); wrap.appendChild(note);

        function warn(text) { note.appendChild(h('span.ph-anyor__warn', [icon('alert'), h('span', text)])); }

        function draw() {
            var m = mode();
            anyBtn.classList.toggle('is-on', m === 'any');
            someBtn.classList.toggle('is-on', m === 'some');
            PH.clear(note);
            if (m === 'some') {
                // Keep the chips control between draws: recreating it would
                // close the picker under a person who is still adding names.
                if (!chips) {
                    chips = ctlChips(Object.assign({}, spec, { value: v }), function (nv) {
                        v = PH.clone(nv); commit(nv); draw();
                    }, err);
                    body.appendChild(chips.el);
                }
                chips.disable(disabled);
                if (!v.length) warn(any.emptyNote || 'Empty list: nobody matches until you add a name.');
            } else {
                if (chips) { PH.clear(body); chips = null; }
                if (m === 'bad') warn(PH.fmtValue(v) + ' is neither "' + anyLabel + '" nor a list of names. Choose one.');
            }
        }
        draw();
        return {
            el: wrap,
            set: function (nv) { v = PH.clone(nv); if (chips && Array.isArray(v)) chips.set(v); draw(); },
            disable: function (d) { disabled = d; anyBtn.disabled = d; someBtn.disabled = d; draw(); },
        };
    }

    /** A short array of numbers ({5, 40}): one box per number. */
    function ctlTuple(spec, commit, err) {
        var arr = PH.clone(spec.value) || [];
        var wrap = h('div.ph-tuple');
        var disabled = false;
        function draw() {
            PH.clear(wrap);
            arr.forEach(function (x, i) {
                var inp = h('input.ph-input.ph-input--num', { type: 'text', inputmode: 'decimal', value: PH.num(x), readOnly: disabled });
                inp.addEventListener('input', function () {
                    var n = numberParse(inp.value);
                    if (n === null) { inp.classList.add('is-bad'); err('Numbers only.'); return; }
                    inp.classList.remove('is-bad'); err('');
                    arr[i] = n;
                    commit(PH.clone(arr));
                });
                if (i > 0) wrap.appendChild(h('span.ph-tuple__sep', arr.length === 2 ? 'to' : ','));
                wrap.appendChild(inp);
            });
        }
        draw();
        return { el: wrap, set: function (v) { arr = PH.clone(v) || []; draw(); }, disable: function (d) { disabled = d; draw(); } };
    }

    function ctlReadonly(spec) {
        var n = spec.node || {};
        var src = n.source || n.text || n.raw || (spec.value !== undefined ? JSON.stringify(spec.value, null, 2) : '');
        return {
            el: h('div.ph-readonly', [
                h('pre.ph-readonly__src', String(src)),
                h('div.ph-readonly__note', [icon('file'), h('span', [
                    'Edit this in ', h('b', (n.file || 'the config file') + (n.line ? ' line ' + n.line : '')), '. ',
                    n.reason ? n.reason : 'The hub cannot rewrite it safely (it is built from code, not a plain value).',
                ])]),
            ]),
            set: function () {}, disable: function () {},
        };
    }

    /** Choose the control for a value and its metadata. */
    F.control = function (spec, commit, err) {
        var m = spec.meta || {};
        var v = spec.value;
        if (spec.readonly) return ctlReadonly(spec);
        if (PH.isCode(v)) return ctlReadonly(Object.assign({}, spec, { node: Object.assign({}, spec.node || {}, { source: v.source, reason: v.reason || 'It is built from code, so it can only be changed in the file.' }) }));
        if (m.any && typeof m.any === 'object') return ctlAnyOr(spec, commit, err);
        if (m.ref && (v === null || v === undefined || typeof v !== 'object')) return ctlRef(spec, commit);
        if (Array.isArray(m.options) && m.options.length && (v === null || typeof v !== 'object')) return ctlSelect(spec, commit);
        var t = PH.typeOf(v);
        var picker = m.picker;
        if (t === 'boolean') return ctlToggle(spec, commit);
        if (t === 'number') {
            if (picker === 'heading') return ctlHeading(spec, commit, err);
            if (picker === 'color') return ctlColor(spec, commit, err);
            return ctlNumber(spec, commit, err);
        }
        if (t === 'string') {
            if (picker === 'webhook') return ctlWebhook(spec, commit, err);
            if (picker === 'color') return ctlColor(spec, commit, err);
            if (picker === 'multiline') return ctlText(spec, commit, err, true);
            if (picker === 'item' && IC.enabled()) return ctlItemText(spec, commit);
            if (picker && F.fetchers[picker]) return ctlPicked(spec, commit, picker);
            if (String(v).length > 90 || String(v).indexOf('\n') !== -1) return ctlText(spec, commit, err, true);
            return ctlText(spec, commit, err, false);
        }
        if (t === 'hash') return picker === 'key' ? ctlPicked(spec, commit, 'key') : ctlHash(spec, commit, err);
        if (/^vector/.test(t)) return ctlVector(spec, commit, err);
        if (t === 'array') {
            var allNum = v.length > 0 && v.every(function (x) { return typeof x === 'number'; });
            if (allNum && v.length <= 4 && !spec.roleable) return ctlTuple(spec, commit, err);
            if (v.every(function (x) { return typeof x === 'string' || typeof x === 'number'; })) return ctlChips(spec, commit, err);
        }
        return ctlReadonly(Object.assign({}, spec, { node: Object.assign({}, spec.node || {}, { reason: 'This value has a shape the hub does not edit.' }) }));
    };

    // --------------------------------------------------------------- shell --

    /**
     * A whole field row.
     * spec: { path, value, label, tooltip, meta, node (top-level node, optional),
     *         stack (label above control), roleable, readonly, onChange() }
     * Returns the element; el.refresh() re-reads the working value.
     */
    F.field = function (spec) {
        var node = spec.node || null;
        var meta = spec.meta || {};
        var label = spec.label || PH.readable(String(spec.path).split(/[.\[]/).pop().replace(/[\]"]/g, ''));
        var tooltip = spec.tooltip || '';
        var el = h('div.ph-field' + (spec.stack ? '.ph-field--stack' : '') + (meta.advanced ? '.is-advanced' : ''), { dataset: { path: PH.canon(spec.path) } });
        var errEl = h('div.ph-field__error');
        var badges = h('span.ph-field__badges');
        var resetBtn = null;
        var ro = spec.readonly || S.isReadOnly();

        function err(text) { errEl.textContent = text || ''; el.classList.toggle('has-error', !!text); }
        el.showError = err;

        spec.get = function () { return S.get(spec.path); };
        var commit = function (v) {
            var ok = S.set(spec.path, v);
            updateBadges();
            if (spec.onChange) spec.onChange(v);
            return ok;
        };
        spec.onRefresh = function () { updateBadges(); if (spec.onChange) spec.onChange(); };
        var ctl = F.control(spec, commit, err);
        if (ro && ctl.disable) ctl.disable(true);

        var info = h('button.ph-info', { type: 'button', 'aria-label': 'About this setting' }, icon('info'));
        PH.tip(info, function () {
            var parts = [];
            if (tooltip) parts.push(h('div.ph-tip__text', tooltip));
            if (node && S.hasDefault(node)) parts.push(h('div.ph-tip__row', [h('span', 'Default'), h('b', PH.fmtValue(node.default, 60))]));
            if (meta.min !== undefined || meta.max !== undefined) {
                parts.push(h('div.ph-tip__row', [h('span', 'Allowed'), h('b', (meta.min !== undefined ? PH.num(meta.min) : '…') + ' to ' + (meta.max !== undefined ? PH.num(meta.max) : '…') + (meta.unit ? ' ' + meta.unit : ''))]));
            }
            if (node) parts.push(h('div.ph-tip__row', [h('span', 'Applies'), h('b', meta.live ? 'Straight away' : 'After a restart')]));
            var where = (node && node.file ? node.file + (node.line ? ':' + node.line : '') + '  ·  ' : '') + spec.path;
            parts.push(h('div.ph-tip__code', where));
            return h('div', parts);
        });

        if (node && S.hasDefault(node) && !spec.readonly) {
            resetBtn = h('button.ph-iconbtn.ph-field__reset', { type: 'button', title: 'Reset to default: ' + PH.fmtValue(node.default, 60) }, icon('reset'));
            resetBtn.addEventListener('click', function () {
                if (S.isReadOnly()) return;
                if (S.queue({ op: 'reset', path: node.path })) {
                    ctl.set(S.get(node.path));
                    err('');
                    updateBadges();
                    if (spec.onChange) spec.onChange();
                }
            });
        }

        function updateBadges() {
            PH.clear(badges);
            var pending = S.isPending(spec.path);
            var differs = node ? S.differsFromDefault(node) : false;
            if (differs) badges.appendChild(h('span.ph-badge.ph-badge--changed', { title: 'Different from the shipped default' }, 'Changed'));
            if (pending) badges.appendChild(h('span.ph-badge.ph-badge--pending', { title: 'Not saved yet' }, 'Unsaved'));
            if (meta.advanced) badges.appendChild(h('span.ph-badge.ph-badge--adv', 'Advanced'));
            el.classList.toggle('is-pending', pending);
            el.classList.toggle('is-changed', differs);
            if (resetBtn) resetBtn.classList.toggle('is-hidden', !differs || S.isReadOnly());
        }

        el.appendChild(h('div.ph-field__info', [
            h('div.ph-field__top', [h('label.ph-field__label', label), info, badges]),
            tooltip && !spec.stack ? h('div.ph-field__desc', tooltip) : null,
        ]));
        el.appendChild(h('div.ph-field__ctl', [ctl.el, errEl]));
        // Coordinates, lists and long text need the full width of a row grid.
        if (/\bph-(vec|chips|textarea|readonly|webhook|heading|refctl)\b/.test(ctl.el.className)) el.classList.add('is-wide');
        el.appendChild(h('div.ph-field__side', resetBtn));
        updateBadges();

        el.refresh = function () {
            ctl.set(S.get(spec.path));
            err('');
            if (ctl.disable) ctl.disable(spec.readonly || S.isReadOnly());
            updateBadges();
        };
        return el;
    };

    /** The field for a top-level node, with all its metadata. */
    F.nodeField = function (node, opts) {
        opts = opts || {};
        var meta = F.metaFor(node);
        return F.field({
            path: node.path,
            value: S.get(node.path),
            node: node,
            meta: meta,
            label: F.labelFor(node),
            tooltip: F.tooltipFor(node),
            readonly: node.kind === 'readonly' || node.editable === false || !!meta.readonly,
            roleable: node.kind === 'strings',
            onChange: opts.onChange,
        });
    };
})();
