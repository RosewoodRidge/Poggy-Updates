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
        wrap.appendChild(roleBar);
        wrap.appendChild(chipsEl);

        function role() { return spec.roleable ? S.roleOf(spec.path) : null; }
        // A list of keys of another list (§8.5): each chip is checked, and Add picks from the keys.
        var refT = spec.meta.ref, refF = spec.meta.refField;
        function refFetch(q) {
            return Promise.resolve(R.keys(refT, refF).filter(function (k) { return arr.indexOf(k) === -1 && PH.matches(q, [k, R.titleOf(refT, k)]); }).map(function (k) {
                var t = R.titleOf(refT, k);
                return { value: k, label: t || k, sub: t ? k : '', icon: 'link' };
            }));
        }

        function draw() {
            PH.clear(chipsEl);
            PH.clear(roleBar);
            var linked = role();
            wrap.classList.toggle('is-linked', !!linked);
            arr.forEach(function (x, i) {
                var bad = refT && !R.known(refT, x, refF);
                var chip = h('span.ph-chip' + (bad ? '.is-bad' : ''), { title: bad ? 'No such ' + R.itemLabel(refT) + ' in ' + R.label(refT) : null },
                    [bad ? icon('alert') : null, h('span.ph-chip__text', String(x))]);
                if (!linked && !disabled) {
                    chip.appendChild(h('button.ph-chip__x', { type: 'button', title: 'Remove ' + x, onclick: function () {
                        arr.splice(i, 1); draw(); commit(PH.clone(arr));
                    } }, icon('x')));
                }
                chipsEl.appendChild(chip);
            });
            if (!arr.length && (linked || disabled)) chipsEl.appendChild(h('span.ph-chips__empty', linked ? 'The role is empty.' : 'Empty'));

            if (!linked && !disabled) {
                if (refT || (picker && F.fetchers[picker])) {
                    var addBtn = h('button.ph-chip.ph-chip--add', { type: 'button' }, [icon('plus'), 'Add']);
                    addBtn.addEventListener('click', function () {
                        F.pick({ anchor: addBtn, fetch: refT ? refFetch : F.fetchers[picker], allowFree: true, freeLabel: 'Add',
                            onPick: function (v) { if (arr.indexOf(v) === -1) { arr.push(v); draw(); commit(PH.clone(arr)); } } });
                    });
                    chipsEl.appendChild(addBtn);
                } else {
                    var input = h('input.ph-chips__input', { type: 'text', placeholder: numeric ? 'Add a number…' : 'Add…', spellcheck: 'false' });
                    input.addEventListener('keydown', function (e) {
                        if (e.key === 'Enter' || e.key === ',') {
                            e.preventDefault();
                            var t = input.value.trim();
                            if (!t) return;
                            var val = t;
                            if (numeric) {
                                val = numberParse(t);
                                if (val === null) { err('Numbers only in this list.'); return; }
                            }
                            err('');
                            if (arr.indexOf(val) === -1) arr.push(val);
                            input.value = '';
                            draw();
                            commit(PH.clone(arr));
                            chipsEl.querySelector('.ph-chips__input').focus();
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
        return {
            el: wrap,
            set: function (v) { arr = PH.clone(v) || []; draw(); },
            disable: function (d) { disabled = d; draw(); },
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
