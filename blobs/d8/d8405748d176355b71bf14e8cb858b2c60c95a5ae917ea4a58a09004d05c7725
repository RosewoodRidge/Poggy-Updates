/*
    Poggy Hub — lists and maps: the table, the row drawer, nested rows,
    collections and key tables.

    A `list` node is an array of rows (stores, recipes, zones); a `map` node
    is keyed by data (Config.StoreTypes.generalstore, Config.Lang["en"]) and
    its rows carry a key that can be renamed. Maps of plain values
    ({ ["food"] = "food.png" }) are edited straight in the table.

    A collection (§8.4) is a map or list whose rows hold lists of their own
    (a catalog's "sells" and "buys"). It shows as a two-pane view: an index of
    its rows on the left, the chosen row's fields and its lists as tabs on
    the right. A key table (§8.9) is a map of control names to hashes, shown
    as two columns with the key picker. A data panel (§10) is live data a
    script owns, edited a cell at a time with no save step (L.panel).

    Every edit is a §6.2 change against the working copy (hub-state.js):
        insert  { path: list, index (1-based, append when absent), value }
                maps: { path: map, key, index: key, value }
        remove  { path: list, index } / { path: map, key, index: key }
        move    { path: list, from, to }                  (1-based)
        renameKey { path: map, oldKey, newKey }
        set     { path: Config.Zones[3].loot[2].item, value }
    Paths inside a collection are full paths
    (Config.Catalog.GeneralStore.sell[3].price), so saving is unchanged.

    Field metadata inside a row comes from the list's hub.json `fields`, keyed
    relative to the row; [] stands for any array index or map key
    ("loot[].item", "grades[].label").
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var h = PH.h, icon = PH.icon, S = PH.S, F = PH.F, R = PH.R, IC = PH.IC;

    var L = PH.L = {};
    var PAGE = 50;

    // --------------------------------------------------------------- shapes --

    function isMapNode(node, meta) {
        if (meta && (meta.kind === 'map' || meta.kind === 'list')) return meta.kind === 'map';
        if (node.kind === 'map' || node.kind === 'list') return node.kind === 'map';
        return PH.isPlainObj(S.get(node.path));
    }

    /** Is this nested object keyed by data (a map) rather than by setting names? */
    function mapLike(obj, pattern, listMeta) {
        if (!PH.isPlainObj(obj)) return false;
        if (obj.__int_keys) return true;
        var keys = Object.keys(obj);
        if (!keys.length) return false;
        if (keys.every(function (k) { return /^\d+$/.test(k); })) return true;
        if (keys.some(function (k) { return !PH.IDENT.test(k); })) return true;
        var fields = (listMeta && listMeta.fields) || {};
        var prefix = pattern + '[]';
        return Object.keys(fields).some(function (f) { return f === prefix || f.indexOf(prefix + '.') === 0 || f.indexOf(prefix + '[') === 0; });
    }
    L.mapLike = mapLike;

    function mapKeys(obj) {
        var keys = Object.keys(obj || {}).filter(function (k) { return k !== '__int_keys'; });
        var numeric = keys.length && keys.every(function (k) { return /^-?\d+$/.test(k); });
        keys.sort(numeric ? function (a, b) { return Number(a) - Number(b); } : function (a, b) { return a.localeCompare(b); });
        return keys;
    }
    L.mapKeys = mapKeys;

    /** Segment for a map key: numbers for integer-keyed maps, strings otherwise. */
    function keySeg(obj, key) {
        return obj && obj.__int_keys && /^-?\d+$/.test(String(key)) ? Number(key) : String(key);
    }

    /** Rows of a list or map node, each { key, path, value, n (1-based position) }. */
    function rowsOf(node, meta) {
        var v = S.get(node.path);
        var out = [];
        if (isMapNode(node, meta)) {
            if (Array.isArray(v)) v = {};    // an empty map arrives as []
            mapKeys(v).forEach(function (k, i) {
                var seg = keySeg(v, k);
                out.push({ key: seg, path: PH.joinPath(node.path, seg), value: v[k], n: i + 1 });
            });
        } else if (Array.isArray(v)) {
            v.forEach(function (row, i) { out.push({ key: i + 1, path: node.path + '[' + (i + 1) + ']', value: row, n: i + 1 }); });
        }
        return out;
    }
    L.rowsOf = rowsOf;

    function getField(row, field) {
        var v = row;
        String(field).split('.').forEach(function (p) { v = v && typeof v === 'object' ? v[p] : undefined; });
        return v;
    }

    var TITLE_KEYS = ['label', 'name', 'title', 'id', 'type', 'item', 'model', 'job', 'text'];

    function titleField(rows, meta) {
        if (meta.titleField) return meta.titleField;
        var sample = rows.slice(0, 10).map(function (r) { return r.value; }).filter(PH.isPlainObj);
        // Case-insensitive: configs write `Text`, `Name`, `Label` as often as
        // `text`, `name`, `label`, and without hub.json every row would
        // otherwise be called "Row 1", "Row 2"… (poggy_crafting's recipes).
        for (var i = 0; i < TITLE_KEYS.length; i++) {
            var k = TITLE_KEYS[i];
            for (var j = 0; j < sample.length; j++) {
                var keys = Object.keys(sample[j]);
                for (var n = 0; n < keys.length; n++) {
                    var v = sample[j][keys[n]];
                    if (keys[n].toLowerCase() === k && typeof v === 'string' && v) return keys[n];
                }
            }
        }
        return null;
    }

    function rowTitle(row, meta, tf, isMap) {
        var t = tf && PH.isPlainObj(row.value) ? getField(row.value, tf) : undefined;
        if (t !== undefined && t !== null && t !== '') return PH.fmtValue(t, 70);
        return isMap ? String(row.key) : (meta.itemLabel ? PH.readable(meta.itemLabel) : 'Row') + ' ' + row.n;
    }

    /** The title a row is known by (its name, label, or key), for usage lists and breadcrumbs. */
    L.titleOf = function (node, row) {
        var meta = F.metaFor(node);
        var rows = rowsOf(node, meta);
        return rowTitle(row, meta, titleField(rows, meta), isMapNode(node, meta));
    };

    function defaultColumns(rows, meta, tf) {
        if (Array.isArray(meta.columns) && meta.columns.length) return meta.columns;
        var seen = {}, order = [];
        rows.slice(0, 30).forEach(function (r) {
            if (!PH.isPlainObj(r.value)) return;
            Object.keys(r.value).forEach(function (k) {
                if (k === '__int_keys' || seen[k]) return;
                seen[k] = PH.typeOf(r.value[k]);
                order.push(k);
            });
        });
        function rank(k) {
            var t = seen[k];
            var i = TITLE_KEYS.indexOf(String(k).toLowerCase());
            if (k === tf) return -100;
            if (i !== -1) return -50 + i;
            return { string: 0, number: 1, boolean: 2, vector3: 3, vector4: 3, vector2: 3, hash: 4, array: 5, table: 6 }[t] || 7;
        }
        var cols = order.slice().sort(function (a, b) { return rank(a) - rank(b) || order.indexOf(a) - order.indexOf(b); });
        // Long text (riddles, descriptions) makes a poor column: leave it for the drawer.
        cols = cols.filter(function (k) {
            if (k === tf) return true;
            return !rows.slice(0, 20).some(function (r) { var v = PH.isPlainObj(r.value) ? r.value[k] : null; return typeof v === 'string' && v.length > 70; });
        });
        return cols.slice(0, 5).map(function (k) {
            var fm = (meta.fields || {})[k] || {};
            return { field: k, label: fm.label || PH.readable(k) };
        });
    }

    /** A blank copy of a row: same keys and types, empty values. */
    function blankOf(v) {
        var t = PH.typeOf(v);
        if (t === 'string') return '';
        if (t === 'number') return 0;
        if (t === 'boolean') return false;
        if (PH.isVec(v)) { var o = { __type: v.__type }; PH.vecAxes(v).forEach(function (a) { o[a] = 0; }); return o; }
        if (t === 'hash') return { __type: 'hash', name: v.name };
        if (t === 'array') return [];
        if (t === 'table') { var r = {}; Object.keys(v).forEach(function (k) { r[k] = k === '__int_keys' ? v[k] : blankOf(v[k]); }); return r; }
        return v;
    }
    L.blankOf = blankOf;

    /**
     * Bring a blank row inside its fields' hub.json limits. A blank number is
     * 0, and a field with "min": 1 refuses 0 -- so the insert of a new
     * ingredient was refused on save ("count: below the minimum") before the
     * amount typed after it was ever applied. pattern is the row-relative
     * position of the row ('' for a top-level row, "Items[]" for an entry).
     */
    function fitToLimits(row, listMeta, pattern) {
        if (!PH.isPlainObj(row)) return row;
        var fields = (listMeta && listMeta.fields) || {};
        Object.keys(row).forEach(function (k) {
            var fm = fields[pattern ? pattern + '.' + k : k];
            if (!fm || typeof row[k] !== 'number') return;
            if (typeof fm.min === 'number' && row[k] < fm.min) row[k] = fm.min;
            if (typeof fm.max === 'number' && row[k] > fm.max) row[k] = fm.max;
        });
        return row;
    }

    /** A reference value in a table cell: the target row's name, or red when it does not exist. */
    function refCell(v, colMeta) {
        var target = colMeta.ref;
        if (v === false || v === null || v === undefined || v === '') return h('span.ph-cellnone', v === false ? 'None' : '—');
        var inOptions = (colMeta.options || []).some(function (o) { return String(o && typeof o === 'object' ? o.value : o) === String(v); });
        if (!inOptions && !R.known(target, v, colMeta.refField)) {
            return h('span.ph-cellbad', { title: 'No such ' + R.itemLabel(target) + ' in ' + R.label(target) }, [icon('alert'), h('span', String(v))]);
        }
        var t = R.titleOf(target, v);
        return h('span.ph-cellref', [h('span', t || String(v)), t ? h('span.ph-cellref__key', String(v)) : null]);
    }

    function cellContent(v, colMeta) {
        if (colMeta && colMeta.ref && (v === null || v === undefined || typeof v !== 'object')) return refCell(v, colMeta);
        var t = PH.typeOf(v);
        if (t === 'code') return h('span.ph-cellcode', { title: 'Built from code: edit it in the file' }, [icon('file'), h('span', PH.fmtValue(v, 40))]);
        if (t === 'boolean') return h('span.ph-cellbool' + (v ? '.is-on' : ''), v ? 'On' : 'Off');
        if (/^vector/.test(t)) return h('span.ph-cellmono', PH.fmtValue(v));
        if (t === 'hash') return h('span.ph-cellmono', v.name);
        if (t === 'number') return h('span.ph-cellnum', PH.num(v) + (colMeta && colMeta.unit ? ' ' + colMeta.unit : ''));
        if (t === 'string' && colMeta && colMeta.picker === 'item' && v && IC.enabled()) {
            // §9.3: the name as written, red when it does not exist, yellow when it has no icon.
            var st = IC.status(v);
            if (st && st.state === 'missing') {
                return h('span.ph-cellbad', { title: 'Item does not exist' + (st.suggest ? ': did you mean ' + st.suggest + '?' : '') }, [icon('alert'), h('span', v)]);
            }
            var cimg = h('img.ph-cellimg', { alt: '' });
            cimg.style.visibility = 'hidden';
            var url = st && st.state === 'ok' ? IC.imageUrl(v) : null;
            if (url) { cimg.onload = function () { cimg.style.visibility = ''; }; cimg.src = url; }
            var lab0 = st && st.label && st.label !== v ? st.label : v;
            return h('span.ph-cellitem' + (st && st.state === 'noicon' ? '.is-warn' : ''), { title: st && st.state === 'noicon' ? 'No icon: add ' + IC.iconFile(v) : v },
                [cimg, h('span', lab0)]);
        }
        if (t === 'string' && colMeta && colMeta.picker === 'item' && v) {
            var img = h('img.ph-cellimg', { alt: '' });
            img.style.visibility = 'hidden';
            var lab = h('span', v);
            F.itemInfo(v).then(function (info) {
                if (!info) return;
                if (info.image) { img.src = info.image; img.onload = function () { img.style.visibility = ''; }; }
                if (info.label && info.label !== v) lab.textContent = info.label;
            });
            return h('span.ph-cellitem', [img, lab]);
        }
        if (t === 'nil') return h('span.ph-cellnone', '—');
        return h('span', PH.fmtValue(v, 60));
    }

    function sortValue(v) {
        if (typeof v === 'number') return v;
        if (typeof v === 'boolean') return v ? 1 : 0;
        return PH.fmtValue(v, 200).toLowerCase();
    }

    // ----------------------------------------------------------- references --

    /** Reference declarations whose values live in this list's rows. */
    function refDeclsOn(node) {
        var cur = S.cur;
        return ((cur && cur.refDecls) || []).filter(function (d) { return d.kind === 'field' && d.host === node.path; });
    }

    /** The key a row of a referenced list is known by (a map's key, or its refField). */
    function refKeyOf(node, row) {
        if (isMapNode(node, F.metaFor(node))) return row.key;
        var d = ((S.cur && S.cur.refDecls) || []).filter(function (x) { return x.target === node.path && x.refField; })[0];
        return d && PH.isPlainObj(row.value) ? row.value[d.refField] : null;
    }

    /**
     * Rename a key that other settings may point at. When something does, the
     * owner is asked whether to update those values in the same save (§8.5).
     * Resolves true when the rename was queued.
     */
    L.renameKey = function (mapPath, oldKey, newKey) {
        var usage = (R.usage(mapPath)[String(oldKey)]) || [];
        var go = function (update) {
            var data = S.cur && S.cur.data;
            var serverDoesRefs = !!(data && data.features && data.features.updateRefs);
            var ch = { op: 'renameKey', path: mapPath, oldKey: oldKey, newKey: newKey };
            if (update && serverDoesRefs) { ch.updateRefs = true; ch._refPaths = usage.map(function (u) { return u.path; }); }
            if (!S.queue(ch)) return false;
            if (update && !serverDoesRefs) {
                usage.forEach(function (u) {
                    var cv = S.get(u.path);
                    if (String(cv) === String(oldKey)) S.set(u.path, typeof cv === 'number' ? Number(newKey) : newKey);
                });
            }
            return true;
        };
        if (!usage.length) return Promise.resolve(go(false));
        var what = R.itemLabel(mapPath);
        return PH.modal({
            title: 'Update ' + PH.plural(usage.length, 'reference') + ' too?', icon: 'link',
            body: h('div', [
                h('p', [PH.plural(usage.length, 'setting') + ' ' + PH.verb(usage.length, 'points', 'point') + ' at the ' + what + ' “', h('b', String(oldKey)), '”. Renaming it to “', h('b', String(newKey)), '” without updating ' + PH.verb(usage.length, 'it', 'them') + ' leaves ' + PH.verb(usage.length, 'it', 'them') + ' pointing at nothing.']),
                h('ul.ph-affected', usage.slice(0, 10).map(function (u) { return h('li', [icon('link'), h('span', u.label)]); })),
                usage.length > 10 ? h('p.ph-muted', '…and ' + (usage.length - 10) + ' more.') : null,
            ]),
            actions: [
                { id: 'no', label: 'Cancel', kind: 'ghost' },
                { id: 'only', label: 'Rename only', kind: 'ghost' },
                { id: 'both', label: 'Rename and update ' + usage.length, kind: 'primary', icon: 'link' },
            ],
        }).then(function (id) {
            if (id === 'both') return go(true);
            if (id === 'only') return go(false);
            return false;
        });
    };

    /** Ask for a key for a new or renamed map row. Resolves the key, or null. */
    function askKey(mapPath, title, text, value) {
        var obj = S.get(mapPath);
        var intKeys = obj && obj.__int_keys;
        return PH.prompt({
            title: title, text: text + (intKeys ? ' This map uses whole numbers as keys.' : ''), value: value, ok: 'OK', icon: 'tag',
            validate: function (v) {
                v = v.trim();
                if (!v) return 'Give it a key.';
                if (intKeys && !/^-?\d+$/.test(v)) return 'Whole numbers only.';
                var cur2 = S.get(mapPath);
                if (cur2 && !Array.isArray(cur2) && Object.prototype.hasOwnProperty.call(cur2, v)) return '“' + v + '” is already used.';
                return null;
            },
        }).then(function (v) {
            if (v === null) return null;
            v = v.trim();
            return intKeys ? Number(v) : v;
        });
    }
    L.askKey = askKey;

    // ------------------------------------------------------------ the list --

    /**
     * The whole editor for one list or map node.
     * opts: { label (the navigation's name), bare (no head: inside a collection),
     *         eyebrow (drawer eyebrow), itemLabel, emptyText }
     */
    L.section = function (node, opts) {
        opts = opts || {};
        var meta = F.metaFor(node);
        var isMap = isMapNode(node, meta);
        var itemLabel = opts.itemLabel || meta.itemLabel || (isMap ? 'entry' : 'row');
        var label = opts.label || F.labelFor(node);
        var state = { q: '', sort: null, dir: 1, page: 0, selected: null };
        var el = h('section.ph-list' + (opts.bare ? '.ph-list--bare' : ''), { dataset: { path: PH.canon(node.path) } });
        var body = h('div.ph-list__body');
        var countEl = h('span.ph-list__count');
        var problems = h('div.ph-list__problems');
        var searchInput = h('input.ph-input.ph-list__search', { type: 'text', placeholder: 'Search ' + PH.pluralOf(itemLabel) + '…', spellcheck: 'false' });
        var badges = h('span.ph-field__badges');
        var isTarget = R.isTarget(node.path);
        var decls = refDeclsOn(node);
        // §9.3: item names in the rows, and what the script says about them.
        var icPats = IC.patterns(meta);
        var flagHost = h('div.ph-list__flags');
        state.flag = opts.flag || null;
        var issMemo = { stamp: null, map: {} };

        function checking() { return (IC.enabled() && icPats.length > 0) || IC.diagUnder(node.path); }
        /** The problems of one row (null when this list checks nothing). */
        function issuesOf(r) {
            if (!checking()) return null;
            var st = IC.stamp();
            if (issMemo.stamp !== st) issMemo = { stamp: st, map: {} };
            var k = r.path;
            if (!issMemo.map[k]) issMemo.map[k] = IC.issues(r.value, r.path, icPats, { noDiag: IC.diagStale(node.path) });
            return issMemo.map[k];
        }
        function setFlag(f) {
            state.flag = f || null;
            state.page = 0;
            if (opts.onFlag) opts.onFlag(state.flag);
            draw();
        }

        searchInput.addEventListener('input', PH.debounce(function () { state.q = searchInput.value.trim(); state.page = 0; draw(); }, 120));
        searchInput.addEventListener('keydown', function (e) { if (e.key === 'Escape' && searchInput.value) { e.stopPropagation(); searchInput.value = ''; state.q = ''; draw(); } });

        var addBtn = h('button.ph-btn.ph-btn--primary.ph-btn--sm', { type: 'button' }, [icon('plus'), 'Add ' + itemLabel]);
        var addMore = h('button.ph-btn.ph-btn--primary.ph-btn--sm.ph-btn--split', { type: 'button', title: 'More ways to add' }, icon('chevdown'));
        addBtn.addEventListener('click', function () { addRow('template'); });
        addMore.addEventListener('click', function () {
            PH.menu(addMore, [
                { label: meta.template ? 'New ' + itemLabel + ' from the template' : 'New blank ' + itemLabel, icon: 'plus', onClick: function () { addRow('template'); } },
                { label: 'Copy of the selected ' + itemLabel, icon: 'copy', disabled: state.selected === null, onClick: function () { addRow('copy'); } },
            ], { alignRight: true, minWidth: 280 });
        });

        // A list inside a collection row is not a node of its own, so the server has no reset for it.
        var resetBtn = S.hasDefault(node) && !node.virtual ? h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'Put the whole list back to the shipped default' }, [icon('reset'), 'Reset list']) : null;
        if (resetBtn) resetBtn.addEventListener('click', function () {
            PH.confirm({ title: 'Reset ' + label + '?', danger: true, ok: 'Reset list',
                body: 'Every ' + itemLabel + ' goes back to how the script shipped. ' + PH.pluralOf(PH.readable(itemLabel)) + ' you added are removed. Nothing is written until you save.' })
                .then(function (yes) { if (yes && S.queue({ op: 'reset', path: node.path })) { L.closeDrawer(); draw(); } });
        });

        var info = h('button.ph-info', { type: 'button', 'aria-label': 'About this list' }, icon('info'));
        PH.tip(info, function () {
            var tt = F.tooltipFor(node);
            return h('div', [tt ? h('div.ph-tip__text', tt) : null,
                h('div.ph-tip__row', [h('span', 'Applies'), h('b', meta.live ? 'Straight away' : 'After a restart')]),
                h('div.ph-tip__code', (node.file || '') + (node.line ? ':' + node.line : '') + '  ·  ' + node.path)]);
        });

        if (!opts.bare) {
            el.appendChild(h('div.ph-list__head', [
                h('div.ph-list__titles', [
                    h('h3.ph-list__title', [label, info, badges]),
                    F.tooltipFor(node) ? h('p.ph-list__desc', F.tooltipFor(node)) : null,
                ]),
            ]));
        }
        el.appendChild(h('div.ph-list__bar', [
            h('div.ph-list__searchwrap', [icon('search'), searchInput]),
            countEl,
            opts.bare ? badges : null,
            h('span.ph-grow'),
            resetBtn,
            h('span.ph-split', [addBtn, addMore]),
        ]));
        el.appendChild(flagHost);
        el.appendChild(problems);
        el.appendChild(body);

        function ro() { return S.isReadOnly(); }

        function filtered() {
            var rows = rowsOf(node, meta);
            var tf = titleField(rows, meta);
            var searchFields = Array.isArray(meta.search) && meta.search.length ? meta.search : null;
            if (state.q) {
                rows = rows.filter(function (r) {
                    var hay = [String(r.key)];
                    if (PH.isPlainObj(r.value)) {
                        (searchFields || Object.keys(r.value)).forEach(function (k) {
                            // hub.json search paths may reach into lists: Items[].name.
                            if (/\[\]/.test(k)) { R.expand(r.value, r.path, k).forEach(function (hit) { if (hit.value !== undefined && typeof hit.value !== 'object') hay.push(String(hit.value)); }); return; }
                            var v = getField(r.value, k);
                            if (v !== undefined && (typeof v !== 'object' || PH.isVec(v) || Array.isArray(v))) hay.push(PH.fmtValue(v, 400));
                        });
                    } else hay.push(PH.fmtValue(r.value, 400));
                    return PH.matches(state.q, hay);
                });
            }
            var searched = rows;
            if (state.flag) rows = rows.filter(function (r) { var iss = issuesOf(r); return !!iss && IC.passes(iss, state.flag); });
            if (state.sort) {
                var f = state.sort, d = state.dir;
                rows = rows.slice().sort(function (a, b) {
                    var va = sortValue(f === '__key' ? a.key : getField(a.value, f));
                    var vb = sortValue(f === '__key' ? b.key : getField(b.value, f));
                    if (va < vb) return -d;
                    if (va > vb) return d;
                    return a.n - b.n;
                });
            }
            return { rows: rows, tf: tf, searched: searched };
        }

        /** The filter chips over the table, with a count for each problem. */
        function drawFlags(searched) {
            PH.clear(flagHost);
            if (!checking()) { state.flag = null; return; }
            var counts = { missing: 0, noicon: 0, hidden: 0 };
            searched.forEach(function (r) {
                var iss = issuesOf(r);
                if (!iss) return;
                if (iss.missing.length) counts.missing++;
                if (iss.noicon.length) counts.noicon++;
                if (iss.hidden) counts.hidden++;
            });
            var note = IC.diagUnder(node.path) && IC.diagStale(node.path)
                ? 'Hidden-in-game marks are held back until you save and restart: ' + PH.pluralOf(itemLabel) + ' were added, removed or moved.' : null;
            var bar = IC.flagBar(counts, state.flag, setFlag, note);
            if (bar) flagHost.appendChild(bar);
        }

        /** Rows whose reference fields point at nothing: listed above the table so the owner sees them. */
        function drawProblems(all, tf) {
            PH.clear(problems);
            if (!decls.length) return;
            var bad = [];
            all.forEach(function (r) {
                decls.forEach(function (d) {
                    R.expand(r.value, r.path, d.pattern).forEach(function (hit) {
                        var v0 = d.refField && PH.isPlainObj(hit.value) ? hit.value[d.refField] : hit.value;
                        var fm = (meta.fields || {})[d.pattern] || {};
                        (Array.isArray(v0) ? v0 : [v0]).forEach(function (v) {
                            if (v !== null && typeof v === 'object') return;
                            var inOptions = (fm.options || []).some(function (o) { return String(o && typeof o === 'object' ? o.value : o) === String(v); });
                            if (!inOptions && !R.known(d.target, v, d.refField)) bad.push({ row: r, value: v, target: d.target, path: hit.path });
                        });
                    });
                });
            });
            if (!bad.length) return;
            var first = bad[0];
            problems.appendChild(h('div.ph-list__warn', [
                icon('alert'),
                h('span.ph-list__warn__text', [
                    h('b', PH.plural(bad.length, itemLabel) + ' ' + PH.verb(bad.length, 'points', 'point') + ' at something that does not exist: '),
                    bad.slice(0, 4).map(function (b, i) {
                        return [i ? ', ' : '', h('button.ph-linkbtn', { type: 'button', onclick: function () { openRow(b.row.key); } }, rowTitle(b.row, meta, tf, isMap)),
                            ' (no ' + R.itemLabel(b.target) + ' “' + b.value + '”)'];
                    }),
                    bad.length > 4 ? ', …' : '',
                    '. The script skips ' + PH.verb(bad.length, 'it', 'them') + ' until that is fixed.',
                ]),
                bad.length === 1 ? h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { openRow(first.row.key); } }, [icon('pencil'), 'Fix it']) : null,
            ]));
        }

        function draw() {
            PH.clear(body);
            addBtn.disabled = addMore.disabled = ro();
            if (resetBtn) resetBtn.classList.toggle('is-hidden', ro() || !S.differsFromDefault(node));
            refreshBadges();

            var all = rowsOf(node, meta);
            var res = filtered();
            var rows = res.rows, tf = res.tf;
            countEl.textContent = state.q || state.flag ? rows.length + ' of ' + PH.plural(all.length, itemLabel) : PH.plural(all.length, itemLabel);
            drawProblems(all, tf);
            drawFlags(res.searched);

            if (!all.length) {
                body.appendChild(h('div.ph-empty', [icon(isMap ? 'rows' : 'list', 'ph-empty__ico'),
                    h('p.ph-empty__title', opts.emptyText || ('No ' + PH.pluralOf(itemLabel) + ' yet.')),
                    h('p.ph-empty__text', ro() ? 'Take the edit lock to add one.' : 'Add one here; it is written to ' + (node.file || 'the config file') + ' when you save.'),
                    ro() ? null : h('button.ph-btn.ph-btn--primary', { type: 'button', onclick: function () { addRow('template'); } }, [icon('plus'), 'Add the first ' + itemLabel])]));
                return;
            }
            if (!rows.length && state.flag) {
                body.appendChild(h('div.ph-empty', [icon('check', 'ph-empty__ico'), h('p', 'No ' + itemLabel + (state.q ? ' matching “' + state.q + '”' : '') + ' has this problem.'),
                    h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { setFlag(null); } }, 'Show all')]));
                return;
            }
            if (!rows.length) {
                body.appendChild(h('div.ph-empty', [icon('search', 'ph-empty__ico'), h('p', 'No ' + itemLabel + ' matches “' + state.q + '”.'),
                    h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { searchInput.value = ''; state.q = ''; draw(); } }, 'Clear the search')]));
                return;
            }

            var scalarMap = isMap && all.every(function (r) { return !PH.isPlainObj(r.value) && !(Array.isArray(r.value) && r.value.some(function (x) { return x && typeof x === 'object'; })); });
            var canMove = !isMap && !state.sort && !state.q && !state.flag && !ro();
            var pages = Math.ceil(rows.length / PAGE);
            if (state.page >= pages) state.page = pages - 1;
            var pageRows = rows.slice(state.page * PAGE, state.page * PAGE + PAGE);

            var cols = scalarMap ? [] : defaultColumns(all, meta, tf);
            var table = h('table.ph-table' + (scalarMap ? '.ph-table--kv' : ''));
            var headRow = h('tr');
            if (canMove || !isMap) headRow.appendChild(h('th.ph-table__num', '#'));
            function th(text, field) {
                var active = state.sort === field;
                var cell = h('th.ph-table__sortable' + (active ? '.is-sorted' : ''), { title: 'Sort by ' + text }, [
                    h('span', text), active ? icon(state.dir > 0 ? 'up' : 'down', 'ph-table__sortico') : null]);
                cell.addEventListener('click', function () {
                    if (state.sort !== field) { state.sort = field; state.dir = 1; }
                    else if (state.dir === 1) state.dir = -1;
                    else { state.sort = null; }
                    draw();
                });
                return cell;
            }
            if (isMap) headRow.appendChild(th('Key', '__key'));
            if (scalarMap) headRow.appendChild(h('th', 'Value'));
            cols.forEach(function (c) { headRow.appendChild(th(c.label || PH.readable(c.field), c.field)); });
            if (isTarget) headRow.appendChild(h('th', 'Used by'));
            headRow.appendChild(h('th.ph-table__acts', ''));
            table.appendChild(h('thead', headRow));

            var tbody = h('tbody');
            pageRows.forEach(function (r) {
                var tr = h('tr.ph-row' + (state.selected !== null && String(state.selected) === String(r.key) ? '.is-selected' : '') + (S.isPending(r.path) ? '.is-pending' : ''), { dataset: { key: String(r.key) } });
                var iss = issuesOf(r);
                var flags = iss && iss.level ? IC.flagsEl(iss) : null;
                if (flags) tr.classList.add(iss.level === 'bad' ? 'is-itembad' : 'is-itemwarn');
                if (canMove || !isMap) {
                    var numCell = h('td.ph-table__num', [
                        canMove ? h('span.ph-grip', { title: 'Drag to reorder' }, icon('grip')) : null,
                        h('span.ph-table__n', String(r.n)),
                    ]);
                    tr.appendChild(numCell);
                    if (canMove) bindDrag(numCell.querySelector('.ph-grip'), tr, r, tbody);
                }
                if (isMap) {
                    tr.appendChild(h('td.ph-table__key', [h('span.ph-cellmono', String(r.key)),
                        ro() ? null : h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Rename this key', onclick: function (e) { e.stopPropagation(); renameKey(r); } }, icon('pencil')),
                        flags && !cols.length ? flags : null]));
                }
                if (scalarMap) {
                    var fm = (meta.fields || {})['[]'] || {};
                    var cellField = F.field({ path: r.path, value: r.value, meta: fm, label: String(r.key), onChange: function () { refreshBadges(); } });
                    cellField.classList.add('ph-field--inline');
                    tr.appendChild(h('td.ph-table__val', cellField));
                }
                cols.forEach(function (c, ci) {
                    var fm2 = (meta.fields || {})[c.field] || {};
                    var content = cellContent(PH.isPlainObj(r.value) ? getField(r.value, c.field) : (ci === 0 ? r.value : undefined), fm2);
                    var td = h('td' + (ci === 0 ? '.ph-table__title' : ''), content);
                    if (ci === 0 && meta.subtitleField && PH.isPlainObj(r.value)) {
                        var sub = getField(r.value, meta.subtitleField);
                        if (sub !== undefined && sub !== '') td.appendChild(h('div.ph-table__sub', PH.fmtValue(sub, 80)));
                    }
                    if (ci === 0 && flags) { td.classList.add('has-flags'); td.appendChild(flags); }
                    tr.appendChild(td);
                });
                if (isTarget) tr.appendChild(h('td.ph-table__used', R.usedByChip(node.path, refKeyOf(node, r), { showZero: true })));
                var acts = h('td.ph-table__acts', h('div.ph-rowacts', [
                    canMove ? h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Move up', disabled: r.n === 1, onclick: function (e) { e.stopPropagation(); move(r.n, r.n - 1); } }, icon('up')) : null,
                    canMove ? h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Move down', disabled: r.n === all.length, onclick: function (e) { e.stopPropagation(); move(r.n, r.n + 1); } }, icon('down')) : null,
                    ro() ? null : h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Duplicate', onclick: function (e) { e.stopPropagation(); duplicate(r); } }, icon('copy')),
                    ro() ? null : h('button.ph-iconbtn.ph-iconbtn--sm.ph-iconbtn--danger', { type: 'button', title: 'Delete', onclick: function (e) { e.stopPropagation(); remove(r); } }, icon('trash')),
                    scalarMap ? null : h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Open' }, icon('right')),
                ]));
                tr.appendChild(acts);
                if (!scalarMap) {
                    tr.addEventListener('click', function () { openRow(r.key); });
                    tr.setAttribute('tabindex', '0');
                    tr.addEventListener('keydown', function (e) { if (e.key === 'Enter' && e.target === tr) openRow(r.key); });
                }
                tbody.appendChild(tr);
            });
            table.appendChild(tbody);
            body.appendChild(h('div.ph-tablewrap', table));

            if (!canMove && !isMap && (state.sort || state.q || state.flag) && !ro()) {
                body.appendChild(h('div.ph-list__note', [icon('info'), 'Clear the search, filter and sorting to reorder rows. The order shown here is not saved.']));
            }
            if (pages > 1) body.appendChild(pager(pages));
        }

        function refreshBadges() {
            PH.clear(badges);
            if (S.differsFromDefault(node)) badges.appendChild(h('span.ph-badge.ph-badge--changed', 'Changed'));
            if (S.isPending(node.path)) badges.appendChild(h('span.ph-badge.ph-badge--pending', 'Unsaved'));
        }

        function pager(pages) {
            var wrap = h('div.ph-pager');
            wrap.appendChild(h('button.ph-iconbtn', { type: 'button', title: 'Previous page', disabled: state.page === 0, onclick: function () { state.page--; draw(); } }, icon('left')));
            for (var p = 0; p < pages; p++) {
                if (pages > 9 && p > 1 && p < pages - 2 && Math.abs(p - state.page) > 1) {
                    if (!wrap.lastChild.classList.contains('ph-pager__gap')) wrap.appendChild(h('span.ph-pager__gap', '…'));
                    continue;
                }
                (function (pp) {
                    wrap.appendChild(h('button.ph-pager__page' + (pp === state.page ? '.is-on' : ''), { type: 'button', onclick: function () { state.page = pp; draw(); } }, String(pp + 1)));
                })(p);
            }
            wrap.appendChild(h('button.ph-iconbtn', { type: 'button', title: 'Next page', disabled: state.page >= pages - 1, onclick: function () { state.page++; draw(); } }, icon('right')));
            return wrap;
        }

        // --------------------------------------------------------- actions --

        function newRowValue(mode) {
            var rows = rowsOf(node, meta);
            if (mode === 'copy' && state.selected !== null) {
                var sel = rows.filter(function (r) { return String(r.key) === String(state.selected); })[0];
                if (sel) return PH.clone(sel.value);
            }
            if (meta.template !== undefined) return PH.clone(meta.template);
            if (rows.length) return fitToLimits(blankOf(rows[rows.length - 1].value), meta, '');
            return {};
        }

        function addRow(mode) {
            if (ro()) return;
            var value = newRowValue(mode);
            if (isMap) {
                askKey(node.path, 'Add ' + itemLabel, 'The key for the new ' + itemLabel + '.', '').then(function (key) {
                    if (key === null) return;
                    if (S.queue({ op: 'insert', path: node.path, key: key, index: key, value: value })) {
                        state.q = ''; searchInput.value = '';
                        draw();
                        openRow(key);
                    }
                });
                return;
            }
            var len = rowsOf(node, meta).length;
            if (S.queue({ op: 'insert', path: node.path, value: value })) {
                state.q = ''; searchInput.value = ''; state.sort = null;
                state.page = Math.floor(len / PAGE);
                draw();
                openRow(len + 1);
            }
        }

        function duplicate(r) {
            if (ro()) return;
            if (isMap) {
                askKey(node.path, 'Duplicate “' + r.key + '”', 'The key for the copy.', String(r.key) + '_copy').then(function (key) {
                    if (key === null) return;
                    if (S.queue({ op: 'duplicate', path: node.path, oldKey: r.key, newKey: key })) { draw(); openRow(key); }
                });
                return;
            }
            if (S.queue({ op: 'duplicate', path: node.path, index: r.n })) {
                L.closeDrawer();
                draw();
                openRow(r.n + 1);
            }
        }

        function remove(r) {
            if (ro()) return;
            var rows = rowsOf(node, meta);
            var title = rowTitle(r, meta, titleField(rows, meta), isMap);
            var used = isTarget ? ((R.usage(node.path)[String(refKeyOf(node, r))]) || []) : [];
            PH.confirm({ title: 'Delete ' + itemLabel + ' “' + title + '”?', danger: true, ok: 'Delete', okIcon: 'trash',
                body: h('div', [
                    h('p', 'It is taken out of ' + label + '. Nothing is written until you save, and Discard brings it back.'),
                    used.length ? h('p.ph-warnline', [icon('alert'), PH.plural(used.length, 'setting') + ' still ' + PH.verb(used.length, 'points', 'point') + ' at it and will point at nothing: ' +
                        used.slice(0, 5).map(function (u) { return u.label; }).join(', ') + (used.length > 5 ? ', …' : '') + '.']) : null,
                ]) })
                .then(function (yes) {
                    if (!yes) return;
                    var ch = isMap ? { op: 'remove', path: node.path, key: r.key, index: r.key } : { op: 'remove', path: node.path, index: r.n };
                    if (S.queue(ch)) {
                        if (state.selected !== null && String(state.selected) === String(r.key)) state.selected = null;
                        L.closeDrawer();
                        draw();
                    }
                });
        }

        function move(from, to) {
            if (ro() || from === to) return;
            var len = rowsOf(node, meta).length;
            if (to < 1 || to > len) return;
            if (S.queue({ op: 'move', path: node.path, from: from, to: to })) {
                if (state.selected !== null) {
                    var s = Number(state.selected);
                    if (s === from) state.selected = to;
                    else if (from < s && to >= s) state.selected = s - 1;
                    else if (from > s && to <= s) state.selected = s + 1;
                }
                L.closeDrawer();
                draw();
                var tr = body.querySelector('tr[data-key="' + to + '"]');
                if (tr) tr.classList.add('is-moved');
            }
        }

        function renameKey(r) {
            if (ro()) return;
            askKey(node.path, 'Rename “' + r.key + '”', 'The new key.', String(r.key)).then(function (key) {
                if (key === null || String(key) === String(r.key)) return;
                L.renameKey(node.path, r.key, key).then(function (done) {
                    if (!done) return;
                    if (state.selected !== null && String(state.selected) === String(r.key)) state.selected = key;
                    L.closeDrawer();
                    draw();
                });
            });
        }

        function bindDrag(grip, tr, r, tbody) {
            if (!grip) return;
            grip.addEventListener('mousedown', function (e) {
                if (e.button !== 0) return;
                e.preventDefault();
                e.stopPropagation();
                var trs = PH.$$('tr.ph-row', tbody);
                var rects = trs.map(function (t) { return t.getBoundingClientRect(); });
                var line = h('div.ph-dropline');
                PH.els.overlays.appendChild(line);
                tr.classList.add('is-dragging');
                var target = r.n;
                var firstN = Number(trs[0].dataset.key);
                function onMove(ev) {
                    var y = ev.clientY, idx = trs.length;
                    for (var i = 0; i < rects.length; i++) { if (y < rects[i].top + rects[i].height / 2) { idx = i; break; } }
                    var ref = idx < rects.length ? rects[idx].top : rects[rects.length - 1].bottom;
                    line.style.top = (ref - 1) + 'px';
                    line.style.left = rects[0].left + 'px';
                    line.style.width = rects[0].width + 'px';
                    var to = firstN + idx;          // insertion slot, 1-based over the list
                    target = to > r.n ? to - 1 : to;
                }
                function onUp() {
                    document.removeEventListener('mousemove', onMove);
                    document.removeEventListener('mouseup', onUp);
                    tr.classList.remove('is-dragging');
                    if (line.parentNode) line.parentNode.removeChild(line);
                    if (target !== r.n) move(r.n, target);
                }
                document.addEventListener('mousemove', onMove);
                document.addEventListener('mouseup', onUp);
                onMove(e);
            });
        }

        function openRow(key) {
            state.selected = key;
            PH.$$('tr.ph-row', body).forEach(function (t) { t.classList.toggle('is-selected', t.dataset.key === String(key)); });
            // The row may be on another page of the table.
            var res0 = filtered().rows;
            for (var i = 0; i < res0.length; i++) {
                if (String(res0[i].key) === String(key)) {
                    var pg = Math.floor(i / PAGE);
                    if (pg !== state.page) { state.page = pg; draw(); }
                    break;
                }
            }
            L.openDrawer({
                node: node, meta: meta, isMap: isMap, key: key, itemLabel: itemLabel,
                eyebrow: opts.eyebrow || label, sublists: opts.sublists || null,
                usedBy: isTarget ? function (row) { return R.usedByChip(node.path, refKeyOf(node, row)); } : null,
                issues: checking() ? function (row) { return issuesOf(row); } : null,
                onChange: redrawSoon,
                duplicate: duplicate, remove: remove, renameKey: renameKey,
                navigate: function (dir) {
                    var res2 = filtered().rows;
                    var j = -1;
                    res2.forEach(function (rr, k) { if (String(rr.key) === String(state.selected)) j = k; });
                    var next = res2[j + dir];
                    if (next) {
                        var pageOf = Math.floor((j + dir) / PAGE);
                        if (pageOf !== state.page) { state.page = pageOf; draw(); }
                        openRow(next.key);
                    }
                },
                onClose: function () {
                    state.selected = null;
                    PH.$$('tr.ph-row', body).forEach(function (t) { t.classList.remove('is-selected'); });
                },
            });
        }

        var redrawSoon = PH.debounce(function () {
            if (!el.isConnected) return;
            // Keep focus if it is inside this table (an inline map value).
            if (body.contains(document.activeElement)) { refreshBadges(); return; }
            draw();
            if (opts.onChange) opts.onChange();
        }, 160);

        el.refresh = draw;
        el.openRow = openRow;
        el.findRow = function (key) { openRow(key); };
        el.search = function (q) { searchInput.value = q; state.q = q; state.page = 0; draw(); };
        el.setFlag = setFlag;
        // The item list or an icon answer arrived: redraw, unless someone is typing in the table.
        IC.watch(el, function () { if (!body.contains(document.activeElement)) draw(); });
        if (IC.enabled() && icPats.length) IC.loadList();
        draw();
        return el;
    };

    // -------------------------------------------------------------- drawer --

    var drawer = null;   // { el, layer, opts, tab }

    L.closeDrawer = function () {
        if (!drawer) return;
        var d = drawer;
        drawer = null;
        PH.popLayer(d.layer);
        d.el.classList.remove('is-in');
        setTimeout(function () { if (d.el.parentNode) d.el.parentNode.removeChild(d.el); }, 220);
        if (d.opts.onClose) d.opts.onClose();
    };

    L.drawerOpen = function () { return !!drawer; };

    /** Keys of a row that are lists of their own: shown as tabs in the drawer (§8.7). */
    function rowSublists(row, meta, declared) {
        var out = [], seen = {};
        function add(field, label) {
            if (seen[field]) return;
            seen[field] = true;
            var fm = (meta.fields || {})[field] || {};
            out.push({ field: field, label: label || fm.label || PH.readable(field) });
        }
        (declared || []).forEach(function (s) { add(s.field, s.label); });
        var v = row.value;
        if (PH.isPlainObj(v)) {
            Object.keys(v).forEach(function (k) {
                var x = v[k];
                if (Array.isArray(x) && x.some(function (y) { return PH.isPlainObj(y); })) add(k);
            });
        }
        // Declared in hub.json as a list of tables (loot[].item), even when this row has none yet.
        Object.keys(meta.fields || {}).forEach(function (f) {
            var m = /^([A-Za-z_][A-Za-z0-9_]*)\[\]\./.exec(f);
            if (m && (!PH.isPlainObj(v) || v[m[1]] === undefined || Array.isArray(v[m[1]]))) add(m[1]);
        });
        return out;
    }

    /** The row editor, sliding in from the right. */
    L.openDrawer = function (opts) {
        var reuse = drawer && drawer.opts.node === opts.node;
        var keepTab = reuse ? drawer.tab : null;
        if (drawer && !reuse) L.closeDrawer();
        var node = opts.node, meta = opts.meta, isMap = opts.isMap;
        var rows = rowsOf(node, meta);
        var row = rows.filter(function (r) { return String(r.key) === String(opts.key); })[0];
        if (!row) { L.closeDrawer(); return; }
        var tf = titleField(rows, meta);
        var ro = S.isReadOnly();

        var el = reuse ? drawer.el : h('aside.ph-drawer', { role: 'dialog', 'aria-label': 'Edit ' + opts.itemLabel });
        PH.clear(el);

        var subs = rowSublists(row, meta, opts.sublists);
        var leaves = PH.isPlainObj(row.value) ? countLeaves(row.value, subs) : 1;
        el.classList.toggle('ph-drawer--wide', leaves > 12 || subs.length > 0);

        var titleEl = h('h2.ph-drawer__title', rowTitle(row, meta, tf, isMap));
        var used = opts.usedBy ? opts.usedBy(row) : null;
        // §9.3: the row's problems in full, kept up to date as its fields change.
        var flagsHost = h('div.ph-drawer__flags');
        function drawRowFlags() {
            PH.clear(flagsHost);
            if (!opts.issues) return;
            var fresh = rowsOf(node, meta).filter(function (r) { return String(r.key) === String(opts.key); })[0];
            var iss = fresh ? opts.issues(fresh) : null;
            var f = iss && iss.level ? IC.flagsEl(iss, { full: true }) : null;
            if (f) flagsHost.appendChild(f);
        }
        drawRowFlags();
        IC.watch(flagsHost, function () { drawRowFlags(); drawTabs(); });
        var head = h('div.ph-drawer__head', [
            h('div.ph-drawer__titles', [
                h('div.ph-eyebrow', [opts.eyebrow || F.labelFor(node), h('span.ph-drawer__pos', isMap ? 'key ' + row.key : '#' + row.n + ' of ' + rows.length)]),
                titleEl,
                used ? h('div.ph-drawer__used', used) : null,
                flagsHost,
            ]),
            h('div.ph-drawer__nav', [
                h('button.ph-iconbtn', { type: 'button', title: 'Previous (Alt+↑)', onclick: function () { opts.navigate(-1); } }, icon('up')),
                h('button.ph-iconbtn', { type: 'button', title: 'Next (Alt+↓)', onclick: function () { opts.navigate(1); } }, icon('down')),
                h('button.ph-iconbtn', { type: 'button', title: 'Close (Esc)', onclick: function () { L.closeDrawer(); } }, icon('x')),
            ]),
        ]);

        var body = h('div.ph-drawer__body');

        function onFieldChange() {
            // The title may have changed with the field it is taken from.
            var fresh = rowsOf(node, meta).filter(function (r) { return String(r.key) === String(opts.key); })[0];
            if (fresh) titleEl.textContent = rowTitle(fresh, meta, tf, isMap);
            drawRowFlags();
            if (opts.onChange) opts.onChange();
        }

        if (isMap) {
            body.appendChild(h('div.ph-drawer__key', [
                h('span.ph-drawer__keylabel', 'Key'),
                h('code.ph-drawer__keyval', String(row.key)),
                ro ? null : h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { opts.renameKey(row); } }, [icon('pencil'), 'Rename']),
            ]));
        }

        // Rows that hold lists: "Fields · Sells (18) · Buys (19)".
        var tab = keepTab && (keepTab === 'fields' || subs.some(function (s) { return s.field === keepTab; })) ? keepTab : 'fields';
        var tabBar = null;
        var pane = h('div.ph-drawer__pane');
        function drawPane() {
            PH.clear(pane);
            if (tabBar) PH.$$('.ph-dtab', tabBar).forEach(function (b) { b.classList.toggle('is-on', b.dataset.tab === tab); });
            if (tab === 'fields') {
                var fieldsHost = h('div.ph-drawer__fields');
                if (leaves > 14) {
                    var q = h('input.ph-input', { type: 'text', placeholder: 'Find a field in this ' + opts.itemLabel + '…', spellcheck: 'false' });
                    q.addEventListener('input', PH.debounce(function () { filterFields(fieldsHost, q.value.trim()); }, 100));
                    pane.appendChild(h('div.ph-drawer__search', [icon('search'), q, h('span.ph-drawer__searchcount', PH.plural(leaves, 'field'))]));
                }
                pane.appendChild(fieldsHost);
                L.renderValue(fieldsHost, {
                    path: row.path, value: row.value, pattern: '', listMeta: meta, file: node.file, depth: 0,
                    label: 'Value', onChange: onFieldChange, big: leaves > 24,
                    skipKeys: subs.map(function (s) { return s.field; }),
                });
                return;
            }
            var sub = subs.filter(function (s) { return s.field === tab; })[0];
            var subPath = PH.IDENT.test(sub.field) ? row.path + '.' + sub.field : PH.joinPath(row.path, sub.field);
            renderCollection(pane, {
                path: subPath, value: S.get(subPath), pattern: sub.field, listMeta: meta, file: node.file, depth: 1,
                label: sub.label, onChange: function () { onFieldChange(); drawTabs(); }, checks: !!opts.issues,
            }, false);
        }
        function drawTabs() {
            if (!tabBar) return;
            PH.clear(tabBar);
            tabBar.appendChild(h('button.ph-dtab' + (tab === 'fields' ? '.is-on' : ''), { type: 'button', dataset: { tab: 'fields' }, onclick: function () { tab = 'fields'; if (drawer) drawer.tab = tab; drawPane(); } }, 'Fields'));
            subs.forEach(function (s) {
                var subPath = PH.IDENT.test(s.field) ? row.path + '.' + s.field : PH.joinPath(row.path, s.field);
                var v = S.get(subPath);
                var n = Array.isArray(v) ? v.length : PH.isPlainObj(v) ? mapKeys(v).length : 0;
                var iss = opts.issues ? entriesIssues(v, subPath, IC.subPatterns(meta, s.field)) : null;
                var lvl = iss && iss.level;
                var tabBtn = h('button.ph-dtab' + (tab === s.field ? '.is-on' : '') + (lvl ? '.has-' + lvl : ''), { type: 'button', dataset: { tab: s.field }, onclick: function () { tab = s.field; if (drawer) drawer.tab = tab; drawPane(); } },
                    [s.label, h('span.ph-dtab__n', String(n)), lvl ? icon('alert', 'ph-dtab__flag') : null]);
                if (lvl) tabBtn.title = IC.labels(iss).map(function (l) { return l.text; }).join('\n');
                tabBar.appendChild(tabBtn);
            });
        }
        if (subs.length) {
            tabBar = h('div.ph-dtabs', { role: 'tablist' });
            body.appendChild(tabBar);
            drawTabs();
        }
        body.appendChild(pane);
        drawPane();

        var foot = h('div.ph-drawer__foot', [
            ro ? h('span.ph-drawer__ro', [icon('lock'), 'Read only']) : h('button.ph-btn.ph-btn--ghost.ph-btn--danger-text', { type: 'button', onclick: function () { opts.remove(row); } }, [icon('trash'), 'Delete']),
            ro ? null : h('button.ph-btn.ph-btn--ghost', { type: 'button', onclick: function () { opts.duplicate(row); } }, [icon('copy'), 'Duplicate']),
            h('span.ph-grow'),
            h('button.ph-btn.ph-btn--primary', { type: 'button', onclick: function () { L.closeDrawer(); } }, [icon('check'), 'Done']),
        ]);

        el.appendChild(head);
        el.appendChild(body);
        el.appendChild(foot);

        if (!reuse) {
            var layer = { kind: 'drawer', close: function () { L.closeDrawer(); } };
            PH.pushLayer(layer);
            drawer = { el: el, layer: layer, opts: opts, tab: tab };
            PH.els.overlays.appendChild(el);
            requestAnimationFrame(function () { el.classList.add('is-in'); });
            el.addEventListener('keydown', function (e) {
                if (e.altKey && e.key === 'ArrowUp') { e.preventDefault(); drawer.opts.navigate(-1); }
                if (e.altKey && e.key === 'ArrowDown') { e.preventDefault(); drawer.opts.navigate(1); }
            });
        } else {
            drawer.opts = opts;
            drawer.tab = tab;
            body.scrollTop = 0;
        }
    };

    /** The problems summed over every entry of a list inside a row (an array or a map). */
    function entriesIssues(v, path, pats) {
        var agg = IC.empty();
        if (Array.isArray(v)) v.forEach(function (x, i) { IC.merge(agg, IC.issues(x, path + '[' + (i + 1) + ']', pats)); });
        else if (PH.isPlainObj(v)) mapKeys(v).forEach(function (k) { IC.merge(agg, IC.issues(v[k], PH.joinPath(path, keySeg(v, k)), pats)); });
        return agg;
    }
    L.entriesIssues = entriesIssues;

    /** Switch the open drawer to one of its list tabs (used by "jump to" a nested value). */
    L.drawerTab = function (field) {
        if (!drawer) return;
        var btn = drawer.el.querySelector('.ph-dtab[data-tab="' + String(field).replace(/(["\\])/g, '\\$1') + '"]');
        if (btn) btn.click();
    };

    function countLeaves(v, subs) {
        var skip = {};
        (subs || []).forEach(function (s) { skip[s.field] = true; });
        var n = 0;
        Object.keys(v).forEach(function (k) {
            if (k === '__int_keys' || skip[k]) return;
            var x = v[k];
            if (PH.isPlainObj(x)) n += countLeaves(x);
            else n += 1;
        });
        return n;
    }

    function filterFields(host, q) {
        PH.$$('.ph-field', host).forEach(function (f) {
            var text = (f.dataset.path || '') + ' ' + (f.textContent || '');
            var inp = f.querySelector('input, textarea');
            if (inp) text += ' ' + inp.value;
            f.classList.toggle('is-filtered', !!q && !PH.matches(q, [text]));
        });
        // A group with nothing left showing is hidden; one with matches opens.
        PH.$$('.ph-nest', host).forEach(function (g) {
            var any = PH.$$('.ph-field', g).some(function (f) { return !f.classList.contains('is-filtered'); });
            g.classList.toggle('is-filtered', !!q && !any);
            if (q && any) g.classList.add('is-open');
        });
    }

    // ------------------------------------------------------- nested values --

    function fieldMeta(listMeta, pattern) {
        var fields = (listMeta && listMeta.fields) || {};
        return fields[pattern] || {};
    }

    var ORDER_FIRST = ['name', 'label', 'title', 'type', 'id', 'item', 'model', 'job', 'coords', 'location', 'heading'];

    function orderedKeys(obj, pattern, listMeta) {
        var keys = Object.keys(obj).filter(function (k) { return k !== '__int_keys'; });
        // Key order in hub.json `fields` does not survive Lua (tables have no
        // order), but `columns` is an array: the author's column order leads.
        var colOrder = !pattern && listMeta && Array.isArray(listMeta.columns)
            ? listMeta.columns.map(function (c) { return String(c.field || '').split('.')[0]; }) : [];
        function rank(k) {
            var i = colOrder.indexOf(k);
            if (i !== -1) return i;
            var j = ORDER_FIRST.indexOf(k);
            if (j !== -1) return 100 + j;
            var v = obj[k];
            var t = PH.typeOf(v);
            var tr = { boolean: 0, number: 1, string: 2, hash: 3, vector2: 4, vector3: 4, vector4: 4, array: 5, table: 6 }[t];
            return 200 + (tr === undefined ? 7 : tr) * 10;
        }
        return keys.sort(function (a, b) { return rank(a) - rank(b) || a.localeCompare(b); });
    }

    /** Group a row's many plain fields by the first word of their key (translations and the like). */
    function groupsOf(keys) {
        var buckets = {}, order = [];
        keys.forEach(function (k) {
            var w = String(k).split(/[_.\-]|(?=[A-Z])/)[0].toLowerCase() || k;
            if (!buckets[w]) { buckets[w] = []; order.push(w); }
            buckets[w].push(k);
        });
        var groups = [], other = [];
        order.forEach(function (w) {
            if (buckets[w].length >= 3) groups.push({ name: PH.readable(w), keys: buckets[w] });
            else other = other.concat(buckets[w]);
        });
        if (other.length) groups.push({ name: groups.length ? 'Other' : '', keys: other });
        return groups;
    }

    /**
     * Render the editor for any value inside a row, recursively.
     * ctx: { path, value, pattern, listMeta, file, depth, label, onChange, big, skipKeys }
     */
    L.renderValue = function (host, ctx) {
        var v = ctx.value;
        if (PH.isPlainObj(v) && !mapLike(v, ctx.pattern, ctx.listMeta)) {
            renderObject(host, ctx);
            return;
        }
        if (PH.isPlainObj(v)) { renderCollection(host, ctx, true); return; }
        if (Array.isArray(v) && (v.some(function (x) { return x && typeof x === 'object' && !PH.isVec(x) && !PH.isHash(x); }) ||
            (!v.length && Object.keys((ctx.listMeta && ctx.listMeta.fields) || {}).some(function (f) { return f.indexOf(ctx.pattern + '[].') === 0; })))) {
            renderCollection(host, ctx, false);
            return;
        }
        host.appendChild(leaf(ctx));
    };

    function leaf(ctx) {
        var fm = fieldMeta(ctx.listMeta, ctx.pattern);
        // A list of names whose entries have a picker (AltNames[] = item, Job[] = job): the chips use it.
        // Also for a field that is 0 now but becomes a list with the "any" switch.
        if ((Array.isArray(ctx.value) || fm.any) && !fm.picker) {
            var em = fieldMeta(ctx.listMeta, ctx.pattern + '[]');
            if (em.picker) fm = Object.assign({}, fm, { picker: em.picker });
        }
        return F.field({
            path: ctx.path, value: ctx.value, meta: fm,
            label: fm.label || ctx.label, tooltip: fm.tooltip || '',
            stack: true, onChange: ctx.onChange,
        });
    }

    function childPattern(pattern, key, isCollection) {
        if (isCollection) return pattern + '[]';
        return pattern ? pattern + '.' + key : String(key);
    }

    function childPathOf(base, k) {
        return PH.IDENT.test(k) ? base + '.' + k : PH.joinPath(base, k);
    }

    function renderObject(host, ctx) {
        var v = ctx.value;
        var skip = {};
        (ctx.skipKeys || []).forEach(function (k) { skip[k] = true; });
        var keys = orderedKeys(v, ctx.pattern, ctx.listMeta).filter(function (k) { return !skip[k]; });
        var plain = keys.filter(function (k) { var x = v[k]; return !PH.isPlainObj(x) && !(Array.isArray(x) && x.some(function (y) { return y && typeof y === 'object' && !PH.isVec(y) && !PH.isHash(y); })); });
        var nested = keys.filter(function (k) { return plain.indexOf(k) === -1; });

        function renderKey(target, k) {
            L.renderValue(target, {
                path: childPathOf(ctx.path, k), value: v[k], pattern: childPattern(ctx.pattern, k, false), listMeta: ctx.listMeta,
                file: ctx.file, depth: ctx.depth + 1, label: PH.readable(k), onChange: ctx.onChange,
            });
        }

        if (ctx.big && plain.length > 24) {
            groupsOf(plain).forEach(function (g, i) {
                var sec = nest(g.name || 'Fields', PH.plural(g.keys.length, 'field'), i === 0);
                g.keys.forEach(function (k) { renderKey(sec.body, k); });
                host.appendChild(sec.el);
            });
        } else {
            var grid = h('div.ph-rowgrid');
            plain.forEach(function (k) { renderKey(grid, k); });
            if (plain.length) host.appendChild(grid);
        }
        nested.forEach(function (k) {
            var fm = fieldMeta(ctx.listMeta, childPattern(ctx.pattern, k, false));
            var x = v[k];
            var sec = nest(fm.label || PH.readable(k), Array.isArray(x) ? PH.plural(x.length, 'entry', 'entries') : '', true);
            L.renderValue(sec.body, {
                path: childPathOf(ctx.path, k), value: x, pattern: childPattern(ctx.pattern, k, false), listMeta: ctx.listMeta,
                file: ctx.file, depth: ctx.depth + 1, label: PH.readable(k), onChange: ctx.onChange,
            });
            host.appendChild(sec.el);
        });
        if (!plain.length && !nested.length) host.appendChild(h('div.ph-coll__empty', 'No other fields.'));
    }

    function nest(title, sub, open) {
        var body = h('div.ph-nest__body');
        var el = h('div.ph-nest' + (open ? '.is-open' : ''));
        var head = h('button.ph-nest__head', { type: 'button' }, [icon('right', 'ph-nest__chev'), h('span.ph-nest__title', title), sub ? h('span.ph-nest__sub', sub) : null]);
        head.addEventListener('click', function () { el.classList.toggle('is-open'); });
        el.appendChild(head);
        el.appendChild(body);
        return { el: el, body: body };
    }

    /**
     * An array of tables, or a map, inside a row (loot[], grades[]): one card
     * per entry, with add / remove / reorder. A declared list the row does not
     * have yet shows empty, and Add creates it (the server does the same).
     */
    function renderCollection(host, ctx, isMap) {
        var wrap = h('div.ph-coll');
        var ro = S.isReadOnly();
        var pattern = ctx.pattern;
        // §9.3: each entry's item names are checked (Items[].name, Items[].AltNames[]).
        var entryPats = IC.subPatterns(ctx.listMeta, pattern);
        var checks = (ctx.checks !== false) && ((IC.enabled() && entryPats.length > 0) || IC.diagUnder(ctx.path));

        function draw() {
            PH.clear(wrap);
            var v = S.get(ctx.path);
            var entries = [];
            // "Grades" -> "Grade 3": an entry reads better named than numbered.
            var one = String(ctx.label || '').replace(/ies$/, 'y').replace(/s$/, '');
            if (isMap || (PH.isPlainObj(v))) {
                mapKeys(v).forEach(function (k) {
                    var seg = keySeg(v, k);
                    entries.push({ key: seg, label: (typeof seg === 'number' && one ? one + ' ' : '') + String(k), path: PH.joinPath(ctx.path, seg), value: v[k] });
                });
            } else if (Array.isArray(v)) {
                v.forEach(function (x, i) { entries.push({ key: i + 1, label: (one ? one + ' ' : '#') + (i + 1), path: ctx.path + '[' + (i + 1) + ']', value: x }); });
            }
            var tf = null;
            entries.forEach(function (en) {
                if (!tf && PH.isPlainObj(en.value)) TITLE_KEYS.some(function (k) {
                    // Same case-insensitive match as titleField(): `Name` counts as `name`.
                    return Object.keys(en.value).some(function (key) {
                        if (key.toLowerCase() === k && typeof en.value[key] === 'string' && en.value[key]) { tf = key; return true; }
                        return false;
                    });
                });
            });
            entries.forEach(function (en, i) {
                var title = tf && PH.isPlainObj(en.value) && en.value[tf] ? String(en.value[tf]) : (PH.isPlainObj(en.value) ? '' : PH.fmtValue(en.value, 60));
                var card = h('div.ph-coll__item');
                var bodyEl = h('div.ph-coll__body');
                var flagsEl = h('div.ph-coll__flags');
                function drawCardFlags() {
                    PH.clear(flagsEl);
                    card.classList.remove('is-itembad', 'is-itemwarn');
                    if (!checks) return;
                    var iss = IC.issues(S.get(en.path), en.path, entryPats);
                    var f = iss.level ? IC.flagsEl(iss) : null;
                    if (!f) return;
                    card.classList.add(iss.level === 'bad' ? 'is-itembad' : 'is-itemwarn');
                    flagsEl.appendChild(f);
                }
                var head = h('div.ph-coll__head', [
                    h('span.ph-coll__idx', en.label),
                    h('span.ph-coll__title', title),
                    h('span.ph-grow'),
                    (!isMap && !ro) ? h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Move up', disabled: i === 0, onclick: function () { if (S.queue({ op: 'move', path: ctx.path, from: i + 1, to: i })) { draw(); if (ctx.onChange) ctx.onChange(); } } }, icon('up')) : null,
                    (!isMap && !ro) ? h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Move down', disabled: i === entries.length - 1, onclick: function () { if (S.queue({ op: 'move', path: ctx.path, from: i + 1, to: i + 2 })) { draw(); if (ctx.onChange) ctx.onChange(); } } }, icon('down')) : null,
                    ro ? null : h('button.ph-iconbtn.ph-iconbtn--sm.ph-iconbtn--danger', { type: 'button', title: 'Remove', onclick: function () {
                        var ch = isMap || !Array.isArray(v) ? { op: 'remove', path: ctx.path, key: en.key, index: en.key } : { op: 'remove', path: ctx.path, index: i + 1 };
                        if (S.queue(ch)) { draw(); if (ctx.onChange) ctx.onChange(); }
                    } }, icon('trash')),
                ]);
                card.appendChild(head);
                card.appendChild(flagsEl);
                drawCardFlags();
                if (checks) IC.watch(flagsEl, drawCardFlags);
                L.renderValue(bodyEl, {
                    path: en.path, value: en.value, pattern: pattern + '[]', listMeta: ctx.listMeta,
                    file: ctx.file, depth: ctx.depth + 1, label: 'Value', onChange: function () {
                        var fresh = S.get(en.path);
                        if (tf && PH.isPlainObj(fresh)) head.querySelector('.ph-coll__title').textContent = fresh[tf] || '';
                        drawCardFlags();
                        if (ctx.onChange) ctx.onChange();
                    },
                });
                card.appendChild(bodyEl);
                wrap.appendChild(card);
            });
            if (!entries.length) wrap.appendChild(h('div.ph-coll__empty', v === undefined ? 'This ' + (one ? one.toLowerCase() : 'list') + ' list is not in the file for this row yet. Adding the first entry creates it.' : 'Nothing here yet.'));
            if (!ro) {
                var add = h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button' }, [icon('plus'), entries.length ? 'Add' : 'Add the first entry']);
                add.addEventListener('click', function () {
                    var fm = fieldMeta(ctx.listMeta, pattern + '[]');
                    var tmpl = fm.template !== undefined ? PH.clone(fm.template) : entries.length ? fitToLimits(blankOf(entries[entries.length - 1].value), ctx.listMeta, pattern + '[]') : templateFromFields(ctx.listMeta, pattern);
                    if (isMap || PH.isPlainObj(v)) {
                        PH.prompt({ title: 'Add to ' + ctx.label, text: 'The key for the new entry.', icon: 'tag',
                            validate: function (k) { k = k.trim(); if (!k) return 'Give it a key.'; if (v && Object.prototype.hasOwnProperty.call(v, k)) return 'Already used.'; if (v && v.__int_keys && !/^-?\d+$/.test(k)) return 'Whole numbers only.'; return null; } })
                            .then(function (k) {
                                if (k === null) return;
                                k = k.trim();
                                var key = v && v.__int_keys ? Number(k) : k;
                                if (S.queue({ op: 'insert', path: ctx.path, key: key, index: key, value: tmpl })) { draw(); if (ctx.onChange) ctx.onChange(); }
                            });
                        return;
                    }
                    if (S.queue({ op: 'insert', path: ctx.path, value: tmpl })) { draw(); if (ctx.onChange) ctx.onChange(); }
                });
                wrap.appendChild(h('div.ph-coll__foot', add));
            }
        }
        draw();
        host.appendChild(wrap);
    }

    /** A first entry for an empty list, from the fields hub.json names for it (loot[].item, loot[].count). */
    function templateFromFields(listMeta, pattern) {
        var out = {}, any = false;
        var prefix = pattern + '[].';
        Object.keys((listMeta && listMeta.fields) || {}).forEach(function (f) {
            if (f.indexOf(prefix) !== 0) return;
            var name = f.slice(prefix.length);
            if (!PH.IDENT.test(name)) return;
            var fm = listMeta.fields[f] || {};
            out[name] = fm.picker === 'coords' ? { __type: 'vec3', x: 0, y: 0, z: 0 } : (typeof fm.min === 'number' || typeof fm.max === 'number') ? (fm.min || 0) : '';
            any = true;
        });
        return any ? out : {};
    }

    // ----------------------------------------------------------- collections --

    /**
     * What the page knows about a collection (§8.4), built once per load.
     * `node` is the map or list node; `groupRows` are the child group nodes
     * when the rows are separate settings (an older server's parse of
     * identifier-keyed tables), in which case rows cannot be added here.
     */
    L.collInfo = function (cur, node, groupRows) {
        var meta = F.metaFor(node);
        var server = node.collection || {};
        var hubSubs = meta.sublists || {};
        var info = {
            path: node.path, node: node, virtual: !!(groupRows && groupRows.length && node.value === undefined),
            groupRows: groupRows || [], sublists: [], scalarFields: server.scalarFields || null,
        };

        function addSub(field, label, m, kind) {
            if (!field) return;
            for (var i = 0; i < info.sublists.length; i++) if (info.sublists[i].field === field) return;
            var hm = hubSubs[field] || {};
            var merged = Object.assign({}, hm, m || {});
            info.sublists.push({ field: field, label: label || hm.label || merged.label || PH.readable(field), meta: merged, kind: kind || merged.kind || null });
        }
        (server.sublists || []).forEach(function (s) { if (s) addSub(s.field, s.label, s.meta, s.kind); });
        Object.keys(hubSubs).forEach(function (f) { addSub(f, hubSubs[f].label); });

        info.rowKeys = function () {
            if (info.virtual) return info.groupRows.map(function (g) { return g.key; });
            var v = S.get(node.path);
            if (Array.isArray(v)) {
                if (!v.length) return [];
                return v.map(function (x, i) { return i + 1; });
            }
            if (PH.isPlainObj(v)) return mapKeys(v).map(function (k) { return keySeg(v, k); });
            return [];
        };
        info.isList = function () { return !info.virtual && Array.isArray(S.get(node.path)) && S.get(node.path).length > 0 && node.kind !== 'map'; };
        info.rowPath = function (key) {
            if (info.virtual) {
                for (var i = 0; i < info.groupRows.length; i++) if (String(info.groupRows[i].key) === String(key)) return info.groupRows[i].path;
            }
            return typeof key === 'number' && info.isList() ? node.path + '[' + key + ']' : PH.joinPath(node.path, key);
        };
        info.subPath = function (key, field) {
            var rp = info.rowPath(key);
            return PH.IDENT.test(field) ? rp + '.' + field : PH.joinPath(rp, field);
        };
        info.subLabel = function (field) {
            for (var i = 0; i < info.sublists.length; i++) if (info.sublists[i].field === field) return info.sublists[i].label;
            return PH.readable(field);
        };
        info.rowTitle = function (key) {
            if (info.virtual) {
                var g = info.groupRows.filter(function (x) { return String(x.key) === String(key); })[0];
                var gl = g ? F.metaFor(g).label : null;
                return gl && gl !== PH.readable(g.key) ? gl : String(key);
            }
            var row = S.get(info.rowPath(key));
            var tf = meta.titleField;
            var t = PH.isPlainObj(row) ? ((tf && row[tf]) || row.label || row.name || row.title) : null;
            if (typeof t === 'string' && t) return t;
            return info.isList() ? (meta.itemLabel ? PH.readable(meta.itemLabel) : 'Row') + ' ' + key : String(key);
        };
        // What the server allows on whole rows (rows written as separate
        // statements cannot be added here, for one), and why not.
        var ops = server.rowOps || null;
        info.can = function (op) { return !info.virtual && (!ops || ops[op] !== false); };
        info.editable = function () { return info.can('add'); };
        info.opsReason = server.rowOpsReason || null;
        info.template = server.template;
        var serverTitles = {};
        (server.rows || []).forEach(function (r) { if (r && r.title) serverTitles[String(r.key)] = r.title; });
        var rowTitle0 = info.rowTitle;
        info.rowTitle = function (key) {
            var t = rowTitle0(key);
            return t === String(key) && serverTitles[String(key)] ? serverTitles[String(key)] : t;
        };

        // Lists found in the rows themselves (and, for grouped rows, their list nodes).
        if (info.virtual) {
            info.groupRows.forEach(function (g) {
                Object.keys(cur.nodes).forEach(function (p) {
                    var n = cur.nodes[p];
                    if (n.parent === g.path && (n.kind === 'list' || n.kind === 'map')) addSub(n.key, null, null);
                });
            });
        } else {
            var v0 = S.get(node.path);
            var rowsV = Array.isArray(v0) ? v0 : PH.isPlainObj(v0) ? mapKeys(v0).map(function (k) { return v0[k]; }) : [];
            rowsV.forEach(function (row) {
                if (!PH.isPlainObj(row)) return;
                Object.keys(row).forEach(function (k) {
                    var x = row[k];
                    if (Array.isArray(x) && (x.some(PH.isPlainObj) || (!x.length && rowsV.some(function (r2) { return PH.isPlainObj(r2) && Array.isArray(r2[k]) && r2[k].some(PH.isPlainObj); })))) addSub(k, null, null);
                });
            });
        }
        return info;
    };

    /**
     * The node the page edits a sublist through: the real list node when the
     * rows are separate settings, else a stand-in that reads by path.
     */
    function subNodeOf(info, key, sub) {
        var path = info.subPath(key, sub.field);
        var real = S.node(path);
        if (real && (real.kind === 'list' || real.kind === 'map')) return real;
        var cur = S.cur;
        cur._virt = cur._virt || {};
        var ck = PH.canon(path);
        var v = S.get(path);
        var existing = cur._virt[ck];
        if (existing) return existing;
        var def;
        if (info.node.default !== undefined && info.node.default !== null) {
            def = info.node.default;
            var segs = [key, sub.field];
            for (var i = 0; i < segs.length && def !== undefined && def !== null; i++) {
                def = Array.isArray(def) ? def[Number(segs[i]) - 1] : (typeof def === 'object' ? def[String(segs[i])] : undefined);
            }
        }
        var vn = {
            path: path, key: sub.field, parent: info.rowPath(key), file: info.node.file, line: info.node.line,
            kind: PH.isPlainObj(v) || (v === undefined && sub.kind === 'map') ? 'map' : 'list', virtual: true, editable: true,
            meta: Object.assign({ itemLabel: 'item' }, sub.meta || {}, { label: sub.label }),
            default: def === undefined ? null : def,
        };
        delete vn.meta.kind;
        cur._virt[ck] = vn;
        return vn;
    }
    L.subNodeOf = subNodeOf;

    function rowChanged(info, key) {
        var rp = info.rowPath(key);
        if (S.isPending(rp)) return true;
        // A row kept as settings: any of its settings changed from the default.
        var rowNode = S.node(rp);
        if (rowNode && rowNode.kind === 'group') return anyChangedUnder(rowNode, 0);
        var d = info.node.default;
        if (d === undefined || d === null) return false;
        var dv = Array.isArray(d) ? d[Number(key) - 1] : d[String(key)];
        return !PH.deepEqual(S.get(rp), dv);
    }
    function anyChangedUnder(node, depth) {
        if (depth > 12) return false;
        return S.kidsOf(node.path).some(function (c) { return c.kind === 'group' ? anyChangedUnder(c, depth + 1) : S.differsFromDefault(c); });
    }
    L.rowChanged = rowChanged;

    function countOf(v) { return Array.isArray(v) ? v.length : PH.isPlainObj(v) ? mapKeys(v).length : 0; }

    /**
     * The two-pane collection editor.
     * view: { row, sub, onState(row, sub) } — the page keeps the selection so
     * the breadcrumbs and deep links follow it.
     */
    L.collection = function (info, view) {
        var node = info.node;
        var meta = F.metaFor(node);
        var label = F.labelFor(node);
        var itemLabel = meta.itemLabel || PH.singular(label) || 'entry';
        var isTarget = R.isTarget(node.path);
        var st = { q: '', flag: view.flag || null };
        var el = h('section.ph-collv', { dataset: { path: PH.canon(node.path) } });
        var subFieldsAll = info.sublists.map(function (s) { return s.field; });
        // §9.3: the row's own item fields (not those of its lists), then each list's.
        var rowPats = IC.patterns(meta).filter(function (p) {
            return !subFieldsAll.some(function (f) { return p === f || p.indexOf(f + '[') === 0 || p.indexOf(f + '.') === 0; });
        });
        var cissMemo = { stamp: null, map: {} };
        function collChecking() {
            if (IC.diagUnder(node.path)) return true;
            if (!IC.enabled()) return false;
            return rowPats.length > 0 || info.sublists.some(function (s) { return IC.patterns(s.meta).length > 0; });
        }
        function subIssues(k, s) {
            var sn = subNodeOf(info, k, s);
            var sm = F.metaFor(sn);
            var pats = IC.patterns(sm);
            var agg = IC.empty();
            var stale = IC.diagStale(sn.path);
            rowsOf(sn, sm).forEach(function (r) { IC.merge(agg, IC.issues(r.value, r.path, pats, { noDiag: stale })); });
            return agg;
        }
        /** Everything wrong in one row of the collection, its lists included. */
        function rowIssues(k) {
            if (!collChecking()) return null;
            var stamp = IC.stamp();
            if (cissMemo.stamp !== stamp) cissMemo = { stamp: stamp, map: {} };
            var ck = String(k);
            if (cissMemo.map[ck]) return cissMemo.map[ck];
            var rp = info.rowPath(k);
            var agg = IC.issues(S.get(rp), rp, rowPats, { noDiag: IC.diagStale(node.path) });
            info.sublists.forEach(function (s) { IC.merge(agg, subIssues(k, s)); });
            cissMemo.map[ck] = agg;
            return agg;
        }
        var indexFlags = h('div.ph-cindex__flags');
        var badges = h('span.ph-field__badges');

        var info2 = h('button.ph-info', { type: 'button', 'aria-label': 'About this collection' }, icon('info'));
        PH.tip(info2, function () {
            var tt = F.tooltipFor(node);
            return h('div', [tt ? h('div.ph-tip__text', tt) : null,
                h('div.ph-tip__row', [h('span', 'Each ' + itemLabel + ' holds'), h('b', info.sublists.map(function (s) { return s.label; }).join(', ') || 'fields')]),
                h('div.ph-tip__row', [h('span', 'Applies'), h('b', meta.live ? 'Straight away' : 'After a restart')]),
                h('div.ph-tip__code', (node.file || '') + (node.line ? ':' + node.line : '') + '  ·  ' + node.path)]);
        });
        el.appendChild(h('div.ph-list__head', [h('div.ph-list__titles', [
            h('h3.ph-list__title', [label, info2, badges]),
            F.tooltipFor(node) ? h('p.ph-list__desc', F.tooltipFor(node)) : null,
        ])]));

        var indexList = h('div.ph-cindex__list', { role: 'listbox' });
        var indexCount = h('span.ph-cindex__count');
        var search = h('input.ph-input.ph-list__search', { type: 'text', placeholder: 'Search ' + PH.pluralOf(itemLabel) + '…', spellcheck: 'false' });
        search.addEventListener('input', PH.debounce(function () { st.q = search.value.trim(); drawIndex(); }, 120));
        search.addEventListener('keydown', function (e) { if (e.key === 'Escape' && search.value) { e.stopPropagation(); search.value = ''; st.q = ''; drawIndex(); } });
        var addBtn = h('button.ph-btn.ph-btn--primary.ph-btn--sm.ph-cindex__add', { type: 'button', title: 'Add a new ' + itemLabel }, [icon('plus'), 'Add', h('span.ph-wide-only', ' ' + itemLabel)]);
        addBtn.addEventListener('click', function () { addRow(null); });
        var index = h('div.ph-cindex', [
            h('div.ph-cindex__bar', [h('div.ph-list__searchwrap', [icon('search'), search])]),
            h('div.ph-cindex__meta', [indexCount, h('span.ph-grow'), info.editable() ? addBtn : null]),
            indexFlags,
            indexList,
            info.editable() ? null : h('div.ph-cindex__note', [icon('file'), h('span', info.opsReason ? info.opsReason : ['Each ' + itemLabel + ' is a block of its own in ', h('b', node.file || 'the config file'), '. Add or remove ' + PH.pluralOf(itemLabel) + ' there; everything inside them can be edited here.'])]),
        ]);
        var detail = h('div.ph-cdetail');
        el.appendChild(h('div.ph-collv__panes', [index, detail]));

        function ro() { return S.isReadOnly(); }
        function keys() { return info.rowKeys(); }

        function selected() {
            var ks = keys();
            if (view.row !== null && view.row !== undefined && ks.some(function (k) { return String(k) === String(view.row); })) {
                return ks.filter(function (k) { return String(k) === String(view.row); })[0];
            }
            return ks.length ? ks[0] : null;
        }

        function counts(key) {
            return info.sublists.map(function (s) { return { s: s, n: countOf(S.get(info.subPath(key, s.field))) }; });
        }

        function drawBadges() {
            PH.clear(badges);
            if (S.differsFromDefault(node)) badges.appendChild(h('span.ph-badge.ph-badge--changed', 'Changed'));
            if (S.isPending(node.path) || info.groupRows.some(function (g) { return S.isPending(g.path); })) badges.appendChild(h('span.ph-badge.ph-badge--pending', 'Unsaved'));
        }

        function drawIndex() {
            PH.clear(indexList);
            var ks = keys();
            var sel = selected();
            var shown = ks.filter(function (k) {
                if (!st.q) return true;
                var hay = [String(k), info.rowTitle(k)];
                // A catalog matches when anything in its lists does: "whiskey" finds the saloon.
                info.sublists.forEach(function (s) { hay.push(JSON.stringify(S.get(info.subPath(k, s.field)) || '')); });
                return PH.matches(st.q, hay);
            });
            PH.clear(indexFlags);
            if (collChecking()) {
                var fcounts = { missing: 0, noicon: 0, hidden: 0 };
                shown.forEach(function (k) {
                    var iss = rowIssues(k);
                    if (iss.missing.length) fcounts.missing++;
                    if (iss.noicon.length) fcounts.noicon++;
                    if (iss.hidden) fcounts.hidden++;
                });
                var fb = IC.flagBar(fcounts, st.flag, function (f) { st.flag = f; if (view.onFlag) view.onFlag(f); drawIndex(); }, null);
                if (fb) indexFlags.appendChild(fb);
                if (st.flag) shown = shown.filter(function (k) { return IC.passes(rowIssues(k), st.flag); });
            } else st.flag = null;
            indexCount.textContent = st.q || st.flag ? shown.length + ' of ' + PH.plural(ks.length, itemLabel) : PH.plural(ks.length, itemLabel);
            if (!ks.length) {
                indexList.appendChild(h('div.ph-empty.ph-empty--sm', [h('p.ph-empty__title', 'No ' + PH.pluralOf(itemLabel) + ' yet.'),
                    h('p.ph-empty__text', info.sublists.length ? 'Each ' + itemLabel + ' holds ' + info.sublists.map(function (s) { return s.label.toLowerCase(); }).join(' and ') + '.' : '')]));
                return;
            }
            if (!shown.length) {
                indexList.appendChild(h('div.ph-empty.ph-empty--sm', st.flag ? [h('p', 'No ' + itemLabel + ' has this problem.'),
                    h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { st.flag = null; if (view.onFlag) view.onFlag(null); drawIndex(); } }, 'Show all')]
                    : h('p', 'Nothing matches “' + st.q + '”.')));
                return;
            }
            shown.forEach(function (k) {
                var title = info.rowTitle(k);
                var cs = counts(k);
                var changed = rowChanged(info, k);
                var riss = rowIssues(k);
                var rflags = riss && riss.level ? IC.flagsEl(riss) : null;
                // A div, not a button: the "Used by" chip inside it is a button of its own.
                var btn = h('div.ph-crow' + (String(k) === String(sel) ? '.is-on' : '') + (S.isPending(info.rowPath(k)) ? '.is-pending' : '') +
                    (rflags ? (riss.level === 'bad' ? '.is-itembad' : '.is-itemwarn') : ''), {
                    role: 'option', tabindex: '0', dataset: { key: String(k) }, 'aria-selected': String(k) === String(sel) ? 'true' : 'false',
                }, [
                    h('span.ph-crow__main', [
                        h('span.ph-crow__title', [h('span.ph-crow__name', title), changed ? h('span.ph-rail__dot', { title: 'Changed or unsaved' }) : null]),
                        title !== String(k) && !info.isList() ? h('span.ph-crow__key', String(k)) : null,
                        cs.length ? h('span.ph-crow__counts', cs.map(function (c, i) {
                            return [i ? h('span.ph-crow__sep', '·') : null, h('span' + (c.n ? '' : '.is-zero'), shortLabel(c.s.label) + ' ' + c.n)];
                        })) : null,
                        rflags,
                    ]),
                    isTarget ? R.usedByChip(node.path, k) : null,
                ]);
                btn.addEventListener('click', function () { select(k, null); });
                btn.addEventListener('keydown', function (e) {
                    if (e.target !== btn) return;
                    if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); select(k, null); }
                    if (e.key === 'ArrowDown' && btn.nextSibling) { e.preventDefault(); btn.nextSibling.focus(); }
                    if (e.key === 'ArrowUp' && btn.previousSibling) { e.preventDefault(); btn.previousSibling.focus(); }
                });
                indexList.appendChild(btn);
            });
        }

        function select(k, sub) {
            view.onState(k, sub);
            view.row = k;
            if (sub !== undefined) view.sub = sub;
            PH.$$('.ph-crow', indexList).forEach(function (b) { b.classList.toggle('is-on', b.dataset.key === String(k)); });
            drawDetail();
        }

        function drawDetail() {
            L.closeDrawer();
            PH.clear(detail);
            var k = selected();
            if (k === null) {
                detail.appendChild(h('div.ph-empty.ph-cdetail__empty', [icon('grid', 'ph-empty__ico'),
                    h('p.ph-empty__title', 'No ' + PH.pluralOf(itemLabel) + ' yet'),
                    h('p.ph-empty__text', F.tooltipFor(node) || ('Add a ' + itemLabel + ' to start.')),
                    info.editable() && !ro() ? h('button.ph-btn.ph-btn--primary', { type: 'button', onclick: function () { addRow(null); } }, [icon('plus'), 'Add the first ' + itemLabel]) : null]));
                return;
            }
            var rowPath = info.rowPath(k);
            var title = info.rowTitle(k);
            var acts = h('div.ph-cdetail__acts');
            if (!ro()) {
                if (!info.isList() && info.can('rename')) acts.appendChild(h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'Rename this ' + itemLabel, onclick: function () { renameRow(k); } }, [icon('pencil'), 'Rename']));
                if (info.can('duplicate')) acts.appendChild(h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'Make a copy of this ' + itemLabel, onclick: function () { addRow(k); } }, [icon('copy'), 'Duplicate']));
                if (info.can('remove')) acts.appendChild(h('button.ph-btn.ph-btn--ghost.ph-btn--sm.ph-btn--danger-text', { type: 'button', title: 'Delete this ' + itemLabel, onclick: function () { removeRow(k); } }, [icon('trash'), 'Delete']));
            }
            detail.appendChild(h('div.ph-cdetail__head', [
                h('div.ph-cdetail__titles', [
                    h('div.ph-eyebrow', [PH.readable(itemLabel), !info.isList() && title !== String(k) ? h('span.ph-mono-inline', String(k)) : null]),
                    h('h2.ph-cdetail__title', title),
                    isTarget ? h('div.ph-cdetail__used', R.usedByChip(node.path, k, { showZero: true })) : null,
                ]),
                acts,
            ]));

            // The row's own settings (a label, a blip), above its lists.
            var scalarHost = h('div.ph-cdetail__fields');
            var subFields = info.sublists.map(function (s) { return s.field; });
            var rowNode = S.node(rowPath);
            if (rowNode && rowNode.kind === 'group') {
                // A row kept as settings: its own nodes, with their defaults and Reset.
                S.kidsOf(rowNode.path).forEach(function (n) {
                    if (subFields.indexOf(String(n.key)) !== -1 || F.metaFor(n).hidden) return;
                    if (n.kind === 'group' || n.kind === 'list' || n.kind === 'map') {
                        L.renderValue(scalarHost, { path: n.path, value: S.get(n.path), pattern: String(n.key), listMeta: meta, file: n.file, depth: 1,
                            label: F.labelFor(n), onChange: function () { drawIndexSoon(); } });
                        return;
                    }
                    scalarHost.appendChild(F.nodeField(n, { onChange: function () { drawIndexSoon(); } }));
                });
            } else {
                var rv = S.get(rowPath);
                if (PH.isPlainObj(rv) && Object.keys(rv).some(function (x) { return x !== '__int_keys' && subFields.indexOf(x) === -1; })) {
                    L.renderValue(scalarHost, { path: rowPath, value: rv, pattern: '', listMeta: meta, file: node.file, depth: 0, label: title,
                        onChange: function () { drawIndexSoon(); }, skipKeys: subFields });
                }
            }
            if (scalarHost.childNodes.length) detail.appendChild(h('div.ph-cdetail__panel', [h('div.ph-cdetail__label', PH.readable(itemLabel) + ' settings'), scalarHost]));

            if (!info.sublists.length) return;
            var sub = info.sublists.filter(function (s) { return s.field === view.sub; })[0] || info.sublists[0];
            var tabs = h('div.ph-subtabs', { role: 'tablist' }, info.sublists.map(function (s) {
                var n = countOf(S.get(info.subPath(k, s.field)));
                var sn = subNodeOf(info, k, s);
                var dirty = S.isPending(info.subPath(k, s.field)) || S.differsFromDefault(sn);
                return h('button.ph-subtab' + (s === sub ? '.is-on' : ''), { type: 'button', role: 'tab', onclick: function () { select(k, s.field); } },
                    [h('span', s.label), h('span.ph-subtab__n', String(n)), h('span.ph-subtab__flag'), dirty ? h('span.ph-rail__dot') : null]);
            }));
            detail.appendChild(tabs);
            markSubtabs();
            var subNode = subNodeOf(info, k, sub);
            var subItem = (sub.meta && sub.meta.itemLabel) || 'item';
            detail.appendChild(L.section(subNode, {
                bare: true, label: sub.label, itemLabel: subItem,
                eyebrow: title + ' › ' + sub.label,
                emptyText: title + ' has no ' + sub.label.toLowerCase() + ' yet.',
                onChange: function () { drawIndexSoon(); redrawTabsSoon(); },
            }));
        }

        /** A warning mark on each list tab of the selected row that has a problem in it. */
        function markSubtabs() {
            var k = selected();
            PH.$$('.ph-subtab', detail).forEach(function (b, i) {
                var s = info.sublists[i];
                var slot = b.querySelector('.ph-subtab__flag');
                if (!s || !slot) return;
                var siss = collChecking() && k !== null ? subIssues(k, s) : null;
                var lvl = siss && siss.level;
                b.classList.remove('has-bad', 'has-warn');
                PH.clear(slot);
                b.removeAttribute('title');
                if (!lvl) return;
                b.classList.add('has-' + lvl);
                slot.appendChild(icon('alert', 'ph-dtab__flag'));
                b.title = IC.labels(siss).map(function (l) { return l.text; }).join('\n');
            });
        }

        var drawIndexSoon = PH.debounce(function () { if (el.isConnected) { drawIndex(); drawBadges(); markSubtabs(); } }, 200);
        var redrawTabsSoon = PH.debounce(function () {
            if (!el.isConnected) return;
            var k = selected();
            PH.$$('.ph-subtab', detail).forEach(function (b, i) {
                var s = info.sublists[i];
                if (!s) return;
                var nEl = b.querySelector('.ph-subtab__n');
                if (nEl) nEl.textContent = String(countOf(S.get(info.subPath(k, s.field))));
            });
        }, 200);

        // ---------------------------------------------------------- actions --

        function addRow(copyOf) {
            if (ro()) return;
            var isCopy = copyOf !== null && copyOf !== undefined;
            if (!info.can(isCopy ? 'duplicate' : 'add')) return;
            if (isCopy && (info.isList() || (Array.isArray(S.get(node.path)) && node.kind === 'list'))) {
                if (S.queue({ op: 'duplicate', path: node.path, index: Number(copyOf) })) { select(Number(copyOf) + 1, null); drawIndex(); }
                return;
            }
            if (isCopy) {
                askKey(node.path, 'Duplicate “' + copyOf + '”', 'The key for the copy. It is how ' + PH.pluralOf(itemLabel) + ' are referred to elsewhere, so keep it short and without spaces.', String(copyOf) + 'Copy').then(function (key) {
                    if (key === null) return;
                    if (S.queue({ op: 'duplicate', path: node.path, oldKey: copyOf, newKey: key })) {
                        st.q = ''; search.value = '';
                        select(key, null);
                        drawIndex();
                        drawBadges();
                    }
                });
                return;
            }
            var value;
            if (copyOf !== null && copyOf !== undefined) value = PH.clone(S.get(info.rowPath(copyOf)));
            else if (meta.template !== undefined) value = PH.clone(meta.template);
            else if (info.template !== undefined && info.template !== null) value = PH.clone(info.template);
            else {
                value = {};
                info.sublists.forEach(function (s) { value[s.field] = []; });
                var ks0 = keys();
                var last = ks0.length ? S.get(info.rowPath(ks0[ks0.length - 1])) : null;
                if (PH.isPlainObj(last)) Object.keys(last).forEach(function (f) { if (value[f] === undefined) value[f] = blankOf(last[f]); });
            }
            if (info.isList() || (Array.isArray(S.get(node.path)) && node.kind === 'list')) {
                if (S.queue({ op: 'insert', path: node.path, value: value })) {
                    var n = keys().length;
                    select(n, null);
                    drawIndex();
                }
                return;
            }
            askKey(node.path, copyOf !== null && copyOf !== undefined ? 'Duplicate “' + copyOf + '”' : 'Add ' + itemLabel,
                'The key for the new ' + itemLabel + '. It is how ' + PH.pluralOf(itemLabel) + ' are referred to elsewhere, so keep it short and without spaces.',
                copyOf !== null && copyOf !== undefined ? String(copyOf) + 'Copy' : '').then(function (key) {
                if (key === null) return;
                if (S.queue({ op: 'insert', path: node.path, key: key, index: key, value: value })) {
                    st.q = ''; search.value = '';
                    select(key, null);
                    drawIndex();
                    drawBadges();
                }
            });
        }

        function renameRow(k) {
            if (ro()) return;
            askKey(node.path, 'Rename “' + k + '”', 'The new key for this ' + itemLabel + '.', String(k)).then(function (key) {
                if (key === null || String(key) === String(k)) return;
                L.renameKey(node.path, k, key).then(function (done) {
                    if (!done) return;
                    select(key, view.sub);
                    drawIndex();
                    drawBadges();
                });
            });
        }

        function removeRow(k) {
            if (ro()) return;
            var used = isTarget ? ((R.usage(node.path)[String(k)]) || []) : [];
            var title = info.rowTitle(k);
            var cs = counts(k).filter(function (c) { return c.n; });
            PH.confirm({ title: 'Delete ' + itemLabel + ' “' + title + '”?', danger: true, ok: 'Delete', okIcon: 'trash',
                body: h('div', [
                    h('p', 'It is taken out of ' + label + (cs.length ? ', with everything in it (' + cs.map(function (c) { return c.n + ' in ' + c.s.label.toLowerCase(); }).join(', ') + ')' : '') + '. Nothing is written until you save, and Discard brings it back.'),
                    used.length ? h('p.ph-warnline', [icon('alert'), PH.plural(used.length, 'setting') + ' still ' + PH.verb(used.length, 'points', 'point') + ' at it and will point at nothing: ' +
                        used.slice(0, 5).map(function (u) { return u.label; }).join(', ') + (used.length > 5 ? ', …' : '') + '.']) : null,
                ]) }).then(function (yes) {
                if (!yes) return;
                var ch = info.isList() ? { op: 'remove', path: node.path, index: Number(k) } : { op: 'remove', path: node.path, key: k, index: k };
                if (S.queue(ch)) {
                    var ks = keys();
                    select(ks.length ? ks[0] : null, null);
                    drawIndex();
                    drawBadges();
                }
            });
        }

        el.refresh = function () { drawIndex(); drawDetail(); drawBadges(); };
        el.select = select;
        el.setFlag = function (f) { st.flag = f || null; drawIndex(); };
        if (collChecking()) {
            if (IC.enabled()) IC.loadList();
            IC.watch(el, function () { if (!indexList.contains(document.activeElement)) drawIndex(); markSubtabs(); });
        }
        el.openEntry = function (k, field, entryKey) {
            select(k, field);
            if (entryKey === undefined || entryKey === null) return null;
            var listEl = detail.querySelector('.ph-list');
            if (listEl && listEl.openRow) listEl.openRow(entryKey);
            return listEl;
        };
        drawBadges();
        drawIndex();
        drawDetail();
        return el;
    };

    /** "Sells to players" -> "Sells": short enough for the index counts. */
    function shortLabel(label) {
        var w = String(label || '').split(/\s+/)[0] || label;
        return w.replace(/[:,]$/, '');
    }

    // ------------------------------------------------------------ key tables --

    function hex(n) { return '0x' + ('00000000' + (n >>> 0).toString(16).toUpperCase()).slice(-8); }

    /**
     * §8.9: a map of key names to control hashes, as two columns. The control
     * is chosen with the key picker (a backtick hash) or typed as a number.
     */
    L.keytable = function (node) {
        var meta = F.metaFor(node);
        var label = F.labelFor(node);
        var itemLabel = meta.itemLabel || 'key';
        var st = { q: '' };
        var el = h('section.ph-list.ph-keytable', { dataset: { path: PH.canon(node.path) } });
        var body = h('div.ph-list__body');
        var countEl = h('span.ph-list__count');
        var badges = h('span.ph-field__badges');
        var search = h('input.ph-input.ph-list__search', { type: 'text', placeholder: 'Search keys and controls…', spellcheck: 'false' });
        search.addEventListener('input', PH.debounce(function () { st.q = search.value.trim(); draw(); }, 120));
        var addBtn = h('button.ph-btn.ph-btn--primary.ph-btn--sm', { type: 'button' }, [icon('plus'), 'Add ' + itemLabel]);
        addBtn.addEventListener('click', add);
        var info = h('button.ph-info', { type: 'button', 'aria-label': 'About this table' }, icon('info'));
        PH.tip(info, function () {
            return h('div', [F.tooltipFor(node) ? h('div.ph-tip__text', F.tooltipFor(node)) : null,
                h('div.ph-tip__row', [h('span', 'Applies'), h('b', meta.live ? 'Straight away' : 'After a restart')]),
                h('div.ph-tip__code', (node.file || '') + (node.line ? ':' + node.line : '') + '  ·  ' + node.path)]);
        });
        el.appendChild(h('div.ph-list__head', [h('div.ph-list__titles', [
            h('h3.ph-list__title', [label, info, badges]),
            h('p.ph-list__desc', F.tooltipFor(node) || 'Names for game controls. Settings elsewhere use the name on the left; the control on the right is the key the game listens for.'),
        ])]));
        el.appendChild(h('div.ph-list__bar', [h('div.ph-list__searchwrap', [icon('search'), search]), countEl, h('span.ph-grow'), addBtn]));
        el.appendChild(body);

        function ro() { return S.isReadOnly(); }

        function controlCell(r) {
            var v = r.value;
            var isHashV = PH.isHash(v);
            var face = h('span.ph-picked__face', [icon('keyboard'), h('span.ph-picked__label.is-mono', isHashV ? v.name : typeof v === 'number' ? hex(v) : PH.fmtValue(v))]);
            var btn = h('button.ph-picked.ph-keytable__ctl', { type: 'button', disabled: ro(), title: 'Choose the game control' }, [face, icon('chevdown')]);
            btn.addEventListener('click', function () {
                F.pick({
                    anchor: btn, fetch: F.fetchers.key, allowFree: true, freeLabel: 'Use', width: 380,
                    placeholder: 'Search controls, or type a hash like 0x760A9C6F…',
                    onPick: function (picked) {
                        var s = String(picked).trim();
                        var nv;
                        if (/^0x[0-9a-f]{1,8}$/i.test(s)) nv = parseInt(s, 16);
                        else if (/^\d+$/.test(s)) nv = Number(s);
                        else if (/^[A-Za-z0-9_]+$/.test(s)) nv = { __type: 'hash', name: s };
                        else { PH.toast({ kind: 'error', title: 'Not a control', text: 'Use a control name (INPUT_…) or a hash such as 0x760A9C6F.' }); return; }
                        if (S.set(r.path, nv)) draw();
                    },
                });
            });
            return btn;
        }

        function draw() {
            PH.clear(body);
            PH.clear(badges);
            if (S.differsFromDefault(node)) badges.appendChild(h('span.ph-badge.ph-badge--changed', 'Changed'));
            if (S.isPending(node.path)) badges.appendChild(h('span.ph-badge.ph-badge--pending', 'Unsaved'));
            addBtn.disabled = ro();
            var rows = rowsOf(node, Object.assign({}, meta, { kind: 'map' }));
            var shown = rows.filter(function (r) { return PH.matches(st.q, [String(r.key), PH.isHash(r.value) ? r.value.name : typeof r.value === 'number' ? hex(r.value) : '']); });
            countEl.textContent = st.q ? shown.length + ' of ' + PH.plural(rows.length, itemLabel) : PH.plural(rows.length, itemLabel);
            if (!rows.length) {
                body.appendChild(h('div.ph-empty', [icon('keyboard', 'ph-empty__ico'), h('p.ph-empty__title', 'No keys yet.')]));
                return;
            }
            var isTarget = R.isTarget(node.path);
            var table = h('table.ph-table.ph-table--keys', [h('thead', h('tr', [h('th', 'Key name'), h('th', 'Control'), isTarget ? h('th', 'Used by') : null, h('th.ph-table__acts', '')]))]);
            var tb = h('tbody');
            shown.forEach(function (r) {
                tb.appendChild(h('tr.ph-row' + (S.isPending(r.path) ? '.is-pending' : ''), { dataset: { key: String(r.key) } }, [
                    h('td.ph-table__key', h('span.ph-keycap', String(r.key))),
                    h('td.ph-keytable__cell', controlCell(r)),
                    isTarget ? h('td.ph-table__used', R.usedByChip(node.path, r.key, { showZero: true })) : null,
                    h('td.ph-table__acts', ro() ? null : h('div.ph-rowacts', [
                        h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Rename', onclick: function () { rename(r); } }, icon('pencil')),
                        h('button.ph-iconbtn.ph-iconbtn--sm.ph-iconbtn--danger', { type: 'button', title: 'Delete', onclick: function () { remove(r); } }, icon('trash')),
                    ])),
                ]));
            });
            table.appendChild(tb);
            body.appendChild(h('div.ph-tablewrap', table));
        }

        function add() {
            if (ro()) return;
            askKey(node.path, 'Add a key', 'The name settings will use for it, such as G or ENTER.', '').then(function (key) {
                if (key === null) return;
                var rows = rowsOf(node, Object.assign({}, meta, { kind: 'map' }));
                var sample = rows.length ? rows[0].value : 0;
                var value = PH.isHash(sample) ? { __type: 'hash', name: 'INPUT_CONTEXT' } : 0;
                if (S.queue({ op: 'insert', path: node.path, key: key, index: key, value: value })) draw();
            });
        }
        function rename(r) {
            askKey(node.path, 'Rename “' + r.key + '”', 'The new name.', String(r.key)).then(function (key) {
                if (key === null || String(key) === String(r.key)) return;
                L.renameKey(node.path, r.key, key).then(function (done) { if (done) draw(); });
            });
        }
        function remove(r) {
            PH.confirm({ title: 'Delete the key “' + r.key + '”?', danger: true, ok: 'Delete', okIcon: 'trash',
                body: 'A setting that uses this name will no longer find a control. Nothing is written until you save.' })
                .then(function (yes) { if (yes && S.queue({ op: 'remove', path: node.path, key: r.key, index: r.key })) draw(); });
        }

        el.refresh = draw;
        draw();
        return el;
    };

    // ---------------------------------------------------------- data panels --
    // §10: live data a script owns (its database), not a config file. Read
    // through the server (`panel`), edited a cell at a time (`panelWrite`):
    // every change applies at once, so there is no queue, no save bar and no
    // lock here. §10.5 adds actions (row buttons and toolbar buttons, with
    // input fields and a confirmation for dangerous ones), drill-down (a row's
    // `detail` opens another panel for that row, with a breadcrumb back) and
    // Contents (a row's `container`, read and changed by poggy_core itself).
    // The view keeps a stack of levels per panel in S.cur.panelState, so a
    // re-render after a write keeps the owner where they were:
    //     { kind: 'panel', panelId, parent?, title, data, ... }
    //     { kind: 'contents', ownerId, ownerCtx, rowKey, title, data, ... }
    // Arguments never carry a null in the middle (the client unpacks them by
    // length), so a missing row key is sent as false and a missing input as {}.

    /**
     * A list from the server. CFX encodes an empty Lua table as {} or [], so
     * anything that is not an array is an empty list (no buttons, no rows).
     */
    function arr(x) { return Array.isArray(x) ? x : []; }
    L.arr = arr;

    function panelLevel(kind, extra) {
        return Object.assign({ kind: kind, data: null, loading: false, err: null, q: '', sort: null, dir: 1, page: 0, flag: null, flash: {} }, extra);
    }

    function panelRowTitle(data, row) {
        var cols = arr(data && data.columns);
        var cells = row.cells && typeof row.cells === 'object' ? row.cells : {};
        var i, v;
        for (i = 0; i < cols.length; i++) {
            v = cells[cols[i].field];
            if (typeof v === 'string' && v.trim() !== '' && !/^[#\d\s.:-]+$/.test(v) && v !== '—') return v;
        }
        for (i = 0; i < cols.length; i++) {
            v = cells[cols[i].field];
            if ((typeof v === 'string' && v !== '') || typeof v === 'number') return String(v);
        }
        return String(row.key);
    }

    function flashKey(key, field) { return String(key) + '' + String(field); }

    /** The control an action's input field (or a Contents input) is asked with. Returns { el, get() }. */
    function inputControl(f) {
        var value = f['default'];
        var err = h('div.ph-field__error');
        var errFn = function (t) { err.textContent = t || ''; };
        var ctl, get;
        if (f.picker === 'coords') {
            value = { __type: 'vec4', x: 0, y: 0, z: 0, w: 0 };
            ctl = F.control({ value: value, meta: {} }, function (v) { value = v; }, errFn);
            get = function () { return value && (value.x || value.y || value.z) ? { x: value.x, y: value.y, z: value.z, heading: value.w } : null; };
        } else if (arr(f.options).length) {
            value = value !== undefined && value !== null ? value : f.options[0].value;
            ctl = F.control({ value: value, meta: { options: f.options } }, function (v) { value = v; }, errFn);
            get = function () { return value; };
        } else if (f.picker && F.fetchers[f.picker]) {
            value = value == null ? '' : String(value);
            ctl = F.control({ value: value, meta: { picker: f.picker } }, function (v) { value = v; }, errFn);
            get = function () { return value === '' ? null : value; };
        } else if (f.type === 'boolean') {
            value = !!value;
            ctl = F.control({ value: value, meta: {} }, function (v) { value = v; }, errFn);
            get = function () { return value; };
        } else {
            var input = h('input.ph-input' + (f.type === 'number' ? '.ph-input--num' : ''), {
                type: 'text', inputmode: f.type === 'number' ? 'decimal' : null, spellcheck: 'false',
                placeholder: f.placeholder || (f.type === 'number' && (f.min !== undefined || f.max !== undefined)
                    ? [f.min !== undefined ? 'at least ' + f.min : null, f.max !== undefined ? 'at most ' + f.max : null].filter(Boolean).join(', ') : ''),
            });
            input.value = value == null ? '' : String(value);
            ctl = { el: input };
            get = function () {
                var t = input.value.trim();
                if (t === '') return null;
                if (f.type === 'number') { var n = Number(t); return isFinite(n) ? n : t; }
                return input.value;
            };
        }
        var el = h('label.ph-pinput', [
            h('span.ph-pinput__label', [f.label || PH.readable(f.field), f.required ? h('span.ph-pinput__req', ' *') : null]),
            f.tooltip ? h('span.ph-pinput__hint', f.tooltip) : null,
            ctl.el, err,
        ]);
        function check() {
            var v = get();
            if (v === null || v === undefined) return f.required ? (f.label || f.field) + ' is needed.' : null;
            if (f.type === 'number') {
                if (typeof v !== 'number') return (f.label || f.field) + ' must be a number.';
                if (typeof f.min === 'number' && v < f.min) return (f.label || f.field) + ' must be at least ' + f.min + '.';
                if (typeof f.max === 'number' && v > f.max) return (f.label || f.field) + ' must be at most ' + f.max + '.';
            }
            return null;
        }
        return { el: el, get: get, check: check, setErr: errFn, field: f.field };
    }

    /**
     * Ask for an action's inputs and confirmation, then run it. run(input,
     * confirm) returns the api promise. Resolves true when it ran.
     */
    function askAction(action, rowLabel, key, run) {
        var inputs = arr(action.input).map(inputControl);
        var typed = action.confirm === 'typed';
        var want = action.scope === 'panel' ? 'CONFIRM' : String(key);
        var needsYes = action.confirm === 'simple' || (action.danger && !typed);
        if (!inputs.length && !typed && !needsYes) return run({}, undefined);
        var typedInput = typed ? h('input.ph-input.ph-pinput__typed', { type: 'text', spellcheck: 'false', placeholder: want, autocomplete: 'off' }) : null;
        var failEl = h('div.ph-field__error.ph-paction__fail');
        var body = h('div.ph-paction', [
            h('p.ph-paction__what', [action.tooltip || null, rowLabel ? h('span.ph-paction__row', [action.scope === 'row' ? 'Row: ' : '', h('b', rowLabel)]) : null]),
            inputs.length ? h('div.ph-paction__inputs', inputs.map(function (c) { return c.el; })) : null,
            action.danger ? h('div.ph-banner.ph-banner--warn', [icon('alert', 'ph-banner__ico'), h('div.ph-banner__text', 'This cannot be undone from the hub. It applies at once.')]) : null,
            typed ? h('label.ph-pinput', [h('span.ph-pinput__label', ['Type ', h('b.ph-cellmono', want), ' to confirm']), typedInput]) : null,
            failEl,
        ]);
        var ran = false;
        var m = PH.modal({
            title: action.label, icon: action.icon || (action.danger ? 'alert' : 'bolt'), tone: action.danger ? 'danger' : null, body: body,
            actions: [
                { id: 'no', label: 'Cancel', kind: 'ghost' },
                { id: 'yes', label: action.label, kind: action.danger ? 'danger' : 'primary', icon: action.icon, onClick: function () {
                    failEl.textContent = '';
                    var input = {}, bad = null;
                    inputs.forEach(function (c) {
                        var e = c.check();
                        c.setErr(e || '');
                        if (e && !bad) bad = e;
                        var v = c.get();
                        if (v !== null && v !== undefined) input[c.field] = v;
                    });
                    if (bad) return false;
                    if (typed && typedInput.value.trim() !== want) { failEl.textContent = 'Type ' + want + ' exactly to confirm.'; typedInput.focus(); return false; }
                    return run(input, typed ? typedInput.value.trim() : needsYes ? true : undefined).then(function (ok) {
                        if (ok === true) { ran = true; return true; }
                        failEl.textContent = typeof ok === 'string' ? ok : 'It did not run.';
                        return false;
                    });
                } },
            ],
        });
        if (typedInput) {
            var okBtn = m.box.querySelector('.ph-modal__foot .ph-btn:last-child');
            var sync = function () { if (okBtn) okBtn.disabled = typedInput.value.trim() !== want; };
            typedInput.addEventListener('input', sync);
            typedInput.addEventListener('keydown', function (e) { if (e.key === 'Enter' && okBtn && !okBtn.disabled) { e.preventDefault(); okBtn.click(); } });
            sync();
            setTimeout(function () { if (!inputs.length) typedInput.focus(); }, 60);
        }
        return m.then(function () { return ran; });
    }

    /** A plain text cell that saves on Enter or when it loses focus; Escape puts the value back. */
    function panelText(value, commit, col) {
        var isNum = typeof value === 'number' || !!(col && col.type === 'number');
        var input = h('input.ph-input.ph-pcell__input' + (isNum ? '.ph-input--num' : ''), { type: 'text', spellcheck: 'false', inputmode: isNum ? 'decimal' : null });
        var cur = value;
        input.value = value == null ? '' : String(value);
        function send() {
            var t = input.value;
            var v = t;
            if (isNum) {
                v = Number(t.trim());
                if (t.trim() === '' || !isFinite(v)) { input.classList.add('is-bad'); input.title = 'This needs a number.'; return; }
                if (col && typeof col.min === 'number' && v < col.min) { input.classList.add('is-bad'); input.title = 'At least ' + col.min + '.'; return; }
                if (col && typeof col.max === 'number' && v > col.max) { input.classList.add('is-bad'); input.title = 'At most ' + col.max + '.'; return; }
            }
            input.classList.remove('is-bad'); input.title = '';
            if (String(v) === String(cur)) return;
            commit(v);
        }
        input.addEventListener('keydown', function (e) {
            if (e.key === 'Enter') { e.preventDefault(); input.blur(); }
            else if (e.key === 'Escape') { e.stopPropagation(); input.value = cur == null ? '' : String(cur); input.classList.remove('is-bad'); input.blur(); }
        });
        input.addEventListener('change', send);
        return {
            el: input,
            set: function (v) { cur = v; input.value = v == null ? '' : String(v); },
            disable: function (d) { input.readOnly = d; input.classList.toggle('is-disabled', d); },
        };
    }

    L.panel = function (it) {
        var cur = S.cur;
        var sid = cur.id;
        var info = it.info || {};
        cur.panelState = cur.panelState || {};
        var ps = cur.panelState[it.panel];
        if (!ps) ps = cur.panelState[it.panel] = { stack: [panelLevel('panel', { panelId: it.panel, title: it.label })] };

        var el = h('section.ph-list.ph-panelv', { dataset: { panel: it.panel } });
        var searchInput = h('input.ph-input.ph-list__search', { type: 'text', spellcheck: 'false' });
        var parts = { head: h('div.ph-list__head'), bar: h('div.ph-list__bar'), banners: h('div.ph-panelv__banners'), flags: h('div.ph-list__flags'), body: h('div.ph-list__body') };
        el.appendChild(parts.head); el.appendChild(parts.bar); el.appendChild(parts.banners); el.appendChild(parts.flags); el.appendChild(parts.body);

        function top() { return ps.stack[ps.stack.length - 1]; }
        function isRoot() { return ps.stack.length === 1; }
        /** ctx for the server: a drill-down level's row and root panel. */
        function ctxOf(lv) { return lv.parent !== undefined ? { parent: lv.parent, root: it.panel } : null; }
        function args(list, ctx) { if (ctx) list.push(ctx); return list; }
        function itemWord(lv) {
            if (lv.kind === 'contents') return 'item';
            return (lv.data && lv.data.itemLabel) || (isRoot() && info.itemLabel) || 'row';
        }

        searchInput.addEventListener('input', PH.debounce(function () { var lv = top(); lv.q = searchInput.value.trim(); lv.page = 0; drawBody(); }, 120));
        searchInput.addEventListener('keydown', function (e) { if (e.key === 'Escape' && searchInput.value) { e.stopPropagation(); searchInput.value = ''; top().q = ''; drawBody(); } });

        // ------------------------------------------------------------- read --
        function load(lv) {
            lv = lv || top();
            lv.loading = true;
            if (lv === top()) drawBar();
            var p = lv.kind === 'contents'
                ? PH.api.apply(null, args(['container', sid, lv.ownerId, lv.rowKey], lv.ownerCtx))
                : PH.api.apply(null, args(['panel', sid, lv.panelId], ctxOf(lv)));
            return p.then(function (r) {
                if (S.cur !== cur) return;
                lv.loading = false;
                if (!r.ok) { lv.err = PH.errMsg(r); }
                else {
                    lv.err = null;
                    lv.data = r.value || {};
                    lv.readAt = Date.now();
                    if (lv.kind === 'panel' && lv.parent !== undefined && lv.data.label && lv.rowTitle) lv.title = lv.rowTitle + ' · ' + lv.data.label;
                    if (lv === ps.stack[0]) {
                        ps.data = lv.data;
                        if (typeof lv.data.available === 'boolean') { info.available = lv.data.available; info.reason = lv.data.reason; }
                        if (PH.renderRail) PH.renderRail();
                    }
                }
                if (el.isConnected && lv === top()) draw();
            });
        }

        // ------------------------------------------------------------ write --
        function write(lv, row, col, value, ctl, status) {
            var old = row.cells ? row.cells[col.field] : undefined;
            if (PH.deepEqual(value, old)) return;
            status.className = 'ph-pcell__status is-busy';
            PH.clear(status); status.appendChild(h('span.ph-spinner.ph-spinner--sm'));
            status.title = 'Saving…';
            if (ctl.disable) ctl.disable(true);
            var k = flashKey(row.key, col.field);
            PH.api.apply(null, args(['panelWrite', sid, lv.panelId, row.key, col.field, value === null || value === undefined ? '' : value], ctxOf(lv))).then(function (r) {
                if (S.cur !== cur) return;
                var okay = r.ok && r.value && r.value.ok !== false;
                var text = okay ? (r.value.message || 'Saved.') : PH.errMsg(r);
                lv.flash[k] = { kind: okay ? 'ok' : 'bad', text: text, until: Date.now() + (okay ? 4000 : 8000) };
                if (okay) {
                    row.cells[col.field] = value;
                    PH.toast({ kind: 'success', title: 'Saved', text: text });
                    if (PH.refreshHistory) PH.refreshHistory();
                } else {
                    if (ctl.set) ctl.set(old);
                    PH.toast({ kind: 'error', title: 'Not saved', text: panelRowTitle(lv.data, row) + ' · ' + col.label + ': ' + text });
                }
                if (ctl.disable) ctl.disable(false);
                showStatus(status, lv.flash[k]);
                load(lv);   // the script's own view of it, whatever happened
            });
        }

        function showStatus(status, fl) {
            PH.clear(status);
            if (!fl || fl.until < Date.now()) { status.className = 'ph-pcell__status'; status.title = ''; return; }
            status.className = 'ph-pcell__status is-' + fl.kind;
            status.title = fl.text;
            status.appendChild(icon(fl.kind === 'ok' ? 'check' : 'alert'));
            var left = fl.until - Date.now();
            setTimeout(function () { if (status.classList.contains('is-' + fl.kind)) { status.className = 'ph-pcell__status is-fading'; } }, Math.max(0, left));
        }

        function cellEditor(lv, row, col, disabled) {
            var v = row.cells ? row.cells[col.field] : undefined;
            var status = h('span.ph-pcell__status');
            var ctl;
            var commit = function (nv) { write(lv, row, col, nv, ctl, status); };
            var noop = function () {};
            if (arr(col.options).length) {
                ctl = F.control({ value: v === undefined ? null : v, meta: { options: col.options } }, commit, noop);
            } else if (col.picker === 'coords') {
                var face = h('span.ph-cellmono', v && typeof v === 'object' ? [v.x, v.y, v.z].map(function (n) { return PH.num(Math.round(Number(n) * 100) / 100); }).join(', ') : '—');
                var use = h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'Set it to where your character stands' }, [icon('target'), 'My position']);
                use.addEventListener('click', function () {
                    F.myPosition().then(function (p) { if (p) commit({ x: p.x, y: p.y, z: p.z, heading: p.heading }); });
                });
                ctl = { el: h('span.ph-pcell__coords', [face, use]), disable: function (d) { use.disabled = d; } };
            } else if (col.picker && F.fetchers[col.picker]) {
                ctl = F.control({ value: v == null || v === false ? '' : String(v), meta: { picker: col.picker } }, commit, noop);
            } else if (typeof v === 'boolean') {
                ctl = F.control({ value: v, meta: {} }, commit, noop);
            } else {
                ctl = panelText(v, commit, col);
            }
            var clear = null;
            if (col.picker && F.fetchers[col.picker] && v !== '' && v != null && v !== false) {
                clear = h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Clear it (none)' }, icon('x'));
                clear.addEventListener('click', function (e) { e.stopPropagation(); commit(''); });
            }
            if (disabled && ctl.disable) ctl.disable(true);
            if (clear && disabled) clear.disabled = true;
            showStatus(status, lv.flash[flashKey(row.key, col.field)]);
            return h('div.ph-pcell', [ctl.el, clear, status]);
        }

        // ---------------------------------------------------------- actions --
        function runAction(lv, action, row, btn) {
            var key = row ? row.key : false;
            var rowLabel = row ? panelRowTitle(lv.data, row) : null;
            return askAction(action, rowLabel, key, function (input, confirm) {
                if (btn) btn.classList.add('is-busy');
                var ctx = lv.kind === 'contents' ? Object.assign({}, lv.ownerCtx || {}) : Object.assign({}, ctxOf(lv) || {});
                if (confirm !== undefined) ctx.confirm = confirm;
                var call = lv.kind === 'contents'
                    ? ['containerAction', sid, lv.ownerId, lv.rowKey, action.id, row ? row.key : false, input || {}, ctx]
                    : ['panelAction', sid, lv.panelId, action.id, action.scope === 'row' ? key : false, input || {}, ctx];
                return PH.api.apply(null, call).then(function (r) {
                    if (btn) btn.classList.remove('is-busy');
                    if (S.cur !== cur) return false;
                    var okay = r.ok && r.value && r.value.ok !== false;
                    if (okay) {
                        PH.toast({ kind: 'success', title: action.label, text: r.value.message || 'Done.' });
                        if (PH.refreshHistory) PH.refreshHistory();
                        load(lv);
                        return true;
                    }
                    var why = PH.errMsg(r);
                    PH.toast({ kind: 'error', title: action.label + ' did not run', text: why });
                    load(lv);
                    return why;
                });
            });
        }

        function rowActions(lv, row) {
            var acts = arr(lv.data && lv.data.actions).filter(function (a) { return a && a.scope !== 'panel'; });
            // A row's own list, when it has one (present but empty, [] or {}, means none).
            if (row.actions !== undefined && row.actions !== null) {
                var own = arr(row.actions);
                acts = acts.filter(function (a) { return own.indexOf(a.id) !== -1; });
            }
            return acts;
        }

        // ------------------------------------------------------- navigating --
        /**
         * A drill-down's name, from its id: the part that repeats the parent
         * panel or its item word is dropped ("shopstaff" under shops reads
         * "Staff", "pricehistory" under prices reads "History").
         */
        function subLabel(lv, id) {
            var s0 = String(id || '');
            var low = s0.toLowerCase();
            var stems = [lv.panelId, lv.data && lv.data.itemLabel, isRoot() ? info.itemLabel : null]
                .filter(Boolean).map(function (x) { return String(x).toLowerCase(); });
            stems = stems.concat(stems.map(function (x) { return x.replace(/s$/, ''); }));
            for (var i = 0; i < stems.length; i++) {
                var st = stems[i];
                if (st.length >= 3 && low.indexOf(st) === 0 && low.length > st.length + 1) { s0 = s0.slice(st.length).replace(/^[_\-]+/, ''); break; }
            }
            return PH.readable(s0);
        }
        function detailsOf(row) {
            var list = arr(row.details).slice();
            if (row.detail && list.indexOf(row.detail) === -1) list.unshift(row.detail);
            return list;
        }
        function openDetail(lv, row, detailId) {
            detailId = detailId || row.detail;
            var sub = panelLevel('panel', { panelId: detailId, parent: row.key, rowTitle: panelRowTitle(lv.data, row),
                title: panelRowTitle(lv.data, row) + ' · ' + subLabel(lv, detailId) });
            ps.stack.push(sub);
            searchInput.value = '';
            draw();
            load(sub);
        }
        function openContents(lv, row) {
            var sub = panelLevel('contents', { ownerId: lv.panelId, ownerCtx: ctxOf(lv), rowKey: row.key, title: 'Contents of ' + panelRowTitle(lv.data, row) });
            ps.stack.push(sub);
            searchInput.value = '';
            draw();
            load(sub);
        }
        function backTo(i) {
            ps.stack.length = i + 1;
            searchInput.value = top().q || '';
            draw();
            load(top());
        }

        // ------------------------------------------------------------- draw --
        function drawHead() {
            var lv = top();
            PH.clear(parts.head);
            var about = h('button.ph-info', { type: 'button', 'aria-label': 'About this panel' }, icon('info'));
            PH.tip(about, function () {
                return h('div', [info.tooltip ? h('div.ph-tip__text', info.tooltip) : null,
                    h('div.ph-tip__row', [h('span', 'Applies'), h('b', 'Straight away')]),
                    h('div.ph-tip__code', 'Data from ' + (info.resource || sid) + '  ·  panel ' + it.panel)]);
            });
            var crumbs = null;
            if (!isRoot()) {
                crumbs = h('nav.ph-pcrumbs', { 'aria-label': 'Back up' }, ps.stack.map(function (l, i) {
                    var last = i === ps.stack.length - 1;
                    var label = i === 0 ? it.label : l.title;
                    return [i ? h('span.ph-crumbs__sep', '›') : null,
                        last ? h('span.ph-pcrumbs__item.is-current', label) : h('button.ph-pcrumbs__item', { type: 'button', onclick: function () { backTo(i); } }, label)];
                }));
            }
            parts.head.appendChild(h('div.ph-list__titles', [
                crumbs,
                h('h3.ph-list__title', [
                    !isRoot() ? h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Back', onclick: function () { backTo(ps.stack.length - 2); } }, icon('back')) : null,
                    lv.kind === 'contents' ? icon('box') : null,
                    isRoot() ? it.label : lv.title, about, h('span.ph-livetag', 'Live'),
                ]),
                isRoot() && info.tooltip ? h('p.ph-list__desc', info.tooltip) : null,
                h('p.ph-panelv__live', [icon('bolt'), h('b', 'Changes here apply immediately. No restart needed.'),
                    h('span.ph-panelv__src', ' Data from ' + (info.resource || sid) + '.')]),
            ]));
        }

        function drawBar() {
            var lv = top();
            PH.clear(parts.bar);
            searchInput.placeholder = 'Search ' + PH.pluralOf(itemWord(lv)) + '…';
            var off = lv.data && lv.data.available === false;
            var tools = arr(lv.data && lv.data.actions).filter(function (a) { return a && a.scope === 'panel'; }).map(function (a) {
                var b = h('button.ph-btn.ph-btn--sm' + (a.danger ? '.ph-btn--danger-text.ph-btn--ghost' : '.ph-btn--primary'), { type: 'button', title: a.tooltip || a.label, disabled: off || lv.loading }, [icon(a.icon || (a.danger ? 'alert' : 'bolt')), a.label]);
                b.addEventListener('click', function () { runAction(lv, a, null, b); });
                return b;
            });
            var when = lv.readAt ? PH.when(Math.floor(lv.readAt / 1000)) : null;
            var refresh = h('button.ph-btn.ph-btn--ghost.ph-btn--sm' + (lv.loading ? '.is-busy' : ''), { type: 'button', title: 'Read it again from the script', disabled: lv.loading },
                [lv.loading ? h('span.ph-spinner.ph-spinner--sm') : icon('restart'), 'Refresh']);
            refresh.addEventListener('click', function () { load(lv); });
            parts.bar.appendChild(h('div.ph-list__searchwrap', [icon('search'), searchInput]));
            parts.bar.appendChild(h('span.ph-list__count', countText(lv)));
            parts.bar.appendChild(h('span.ph-grow'));
            if (when) parts.bar.appendChild(h('span.ph-panelv__when', { title: when.abs || '' }, 'Read ' + when.rel));
            tools.forEach(function (b) { parts.bar.appendChild(b); });
            parts.bar.appendChild(refresh);
        }

        function countText(lv) {
            if (!lv.data) return '';
            var all = lv.data.total || arr(lv.data.rows).length;
            var shown = filtered(lv).length;
            return lv.q || lv.flag ? shown + ' of ' + PH.plural(all, itemWord(lv)) : PH.plural(all, itemWord(lv));
        }

        function drawBanners() {
            var lv = top();
            PH.clear(parts.banners);
            var d = lv.data;
            function banner(kind, ic, text) { parts.banners.appendChild(h('div.ph-banner.ph-banner--' + kind, [icon(ic, 'ph-banner__ico'), h('div.ph-banner__text', text)])); }
            if (lv.kind !== 'contents' && isRoot() && (info.available === false || (d && d.available === false))) {
                banner('warn', 'stop', [h('b', 'Read only. '), (d && d.reason) || info.reason || 'The script that owns this data is not available.']);
            }
            if (lv.err) banner('warn', 'alert', [h('b', 'Could not read this. '), lv.err]);
            if (d && d.note) banner('info', 'info', d.note);
            if (d && d.truncated) banner('info', 'info', 'Showing the first ' + arr(d.rows).length + ' of ' + PH.plural(d.total, itemWord(lv)) + '. Search narrows it down.');
        }

        function filtered(lv) {
            var d = lv.data;
            var rows = arr(d && d.rows);
            if (lv.q) {
                rows = rows.filter(function (r) {
                    var hay = [String(r.key), r.note || ''];
                    var cells0 = r.cells && typeof r.cells === 'object' ? r.cells : {};
                    Object.keys(cells0).forEach(function (k) { hay.push(PH.fmtValue(cells0[k], 200)); });
                    return PH.matches(lv.q, hay);
                });
            }
            if (lv.flag) rows = rows.filter(function (r) { return r.level === 'warning' || r.level === 'error'; });
            if (lv.sort) {
                var f = lv.sort, dir = lv.dir;
                rows = rows.map(function (r, i) { return { r: r, i: i }; }).sort(function (a, b) {
                    var va = sortValue(f === '__key' ? a.r.key : (a.r.cells || {})[f]);
                    var vb = sortValue(f === '__key' ? b.r.key : (b.r.cells || {})[f]);
                    if (va < vb) return -dir;
                    if (va > vb) return dir;
                    return a.i - b.i;
                }).map(function (x) { return x.r; });
            }
            return rows;
        }

        function drawFlags() {
            var lv = top();
            PH.clear(parts.flags);
            var rows = arr(lv.data && lv.data.rows);
            var n = rows.filter(function (r) { return r.level === 'warning' || r.level === 'error'; }).length;
            if (!n && !lv.flag) return;
            var bar = h('div.ph-flagbar', { role: 'group', 'aria-label': 'Show only rows that need attention' });
            function chip(flag, label, count, tone) {
                var on = (lv.flag || null) === flag;
                bar.appendChild(h('button.ph-flagchip' + (tone ? '.is-' + tone : '') + (on ? '.is-on' : ''), { type: 'button', 'aria-pressed': on ? 'true' : 'false',
                    onclick: function () { lv.flag = on && flag ? null : flag; lv.page = 0; drawBody(); drawFlags(); drawBar(); } },
                    [tone ? h('span.ph-flagchip__dot') : null, label, flag ? h('span.ph-flagchip__n', String(count)) : null]));
            }
            chip(null, 'All', 0, null);
            chip('attention', 'Needs attention', n, 'warn');
            parts.flags.appendChild(bar);
        }

        function drawBody() {
            var lv = top();
            var body = parts.body;
            PH.clear(body);
            var d = lv.data;
            var cnt = parts.bar.querySelector('.ph-list__count');
            if (cnt) cnt.textContent = countText(lv);
            if (!d) {
                if (lv.loading) body.appendChild(h('div.ph-empty', [h('span.ph-spinner'), h('p.ph-empty__title', 'Reading from ' + (info.resource || 'the script') + '…')]));
                else if (!lv.err) body.appendChild(h('div.ph-empty', [icon('database', 'ph-empty__ico'), h('p.ph-empty__title', 'Nothing read yet.')]));
                return;
            }
            var off = d.available === false;
            var cols = arr(d.columns);
            var all = arr(d.rows);
            if (off && !all.length) {
                body.appendChild(h('div.ph-empty', [icon('stop', 'ph-empty__ico'), h('p.ph-empty__title', 'Nothing to show while the script is not available.'),
                    h('p.ph-empty__text', d.reason || info.reason || '')]));
                return;
            }
            if (!all.length) {
                body.appendChild(h('div.ph-empty', [icon(lv.kind === 'contents' ? 'box' : 'database', 'ph-empty__ico'),
                    h('p.ph-empty__title', lv.kind === 'contents' ? 'This container is empty.' : 'No ' + PH.pluralOf(itemWord(lv)) + ' yet.'),
                    h('p.ph-empty__text', lv.kind === 'contents' ? 'Add item puts something in it.' : 'The script has nothing to list here right now.')]));
                return;
            }
            var rows = filtered(lv);
            if (!rows.length) {
                body.appendChild(h('div.ph-empty', [icon('search', 'ph-empty__ico'), h('p', lv.q ? 'No ' + itemWord(lv) + ' matches “' + lv.q + '”.' : 'No ' + itemWord(lv) + ' needs attention.'),
                    h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { lv.q = ''; lv.flag = null; searchInput.value = ''; draw(); } }, 'Show all')]));
                return;
            }
            var pages = Math.ceil(rows.length / PAGE);
            if (lv.page >= pages) lv.page = pages - 1;
            var pageRows = rows.slice(lv.page * PAGE, lv.page * PAGE + PAGE);
            var anyActs = all.some(function (r) { return r.detail || arr(r.details).length || r.container || rowActions(lv, r).length; });

            var table = h('table.ph-table.ph-table--panel');
            var headRow = h('tr');
            cols.forEach(function (c) {
                var active = lv.sort === c.field;
                var th = h('th.ph-table__sortable' + (active ? '.is-sorted' : '') + (c.editable ? '.is-editable' : ''), {
                    title: (c.tooltip ? c.tooltip + '\n' : '') + 'Sort by ' + c.label, style: c.width ? { width: typeof c.width === 'number' ? c.width + 'px' : c.width } : null,
                }, [h('span', c.label), c.editable ? h('span.ph-table__edithint', { title: 'You can change this; it applies at once' }, icon('pencil')) : null,
                    active ? icon(lv.dir > 0 ? 'up' : 'down', 'ph-table__sortico') : null]);
                th.addEventListener('click', function () {
                    if (lv.sort !== c.field) { lv.sort = c.field; lv.dir = 1; }
                    else if (lv.dir === 1) lv.dir = -1;
                    else lv.sort = null;
                    drawBody();
                });
                headRow.appendChild(th);
            });
            if (anyActs) headRow.appendChild(h('th.ph-table__acts', ''));
            table.appendChild(h('thead', headRow));
            var tbody = h('tbody');
            pageRows.forEach(function (row) {
                var tone = row.level === 'error' ? '.is-itembad' : row.level === 'warning' ? '.is-itemwarn' : '';
                var tr = h('tr.ph-row.ph-prow' + tone + (row.detail && !off ? '.is-clickable' : ''), { dataset: { key: String(row.key) } });
                if (row.detail && !off) {
                    // A click on the row (not on a field or a button) opens its drill-down.
                    tr.title = 'Open ' + subLabel(lv, row.detail).toLowerCase() + ' for this ' + itemWord(lv);
                    tr.addEventListener('click', function (e) {
                        var t = e.target;
                        while (t && t !== tr) {
                            if (/^(BUTTON|INPUT|SELECT|TEXTAREA|A|LABEL)$/.test(t.tagName) || (t.classList && t.classList.contains('ph-pcell'))) return;
                            t = t.parentNode;
                        }
                        openDetail(lv, row, row.detail);
                    });
                }
                cols.forEach(function (c, ci) {
                    var cv = row.cells ? row.cells[c.field] : undefined;
                    var content = c.editable && lv.kind !== 'contents' ? cellEditor(lv, row, c, off)
                        : (cv === '' || cv === null || cv === undefined) ? h('span.ph-cellnone', '—') : cellContent(cv, c);
                    var td = h('td' + (ci === 0 ? '.ph-table__title' : '') + (c.editable ? '.ph-table__edit' : '')
                        + (c.editable && (c.type === 'number' || typeof cv === 'number') ? '.is-num' : ''), content);
                    tr.appendChild(td);
                });
                if (anyActs) {
                    var btns = [];
                    var racts = rowActions(lv, row);
                    if (racts.length === 1) {
                        var a1 = racts[0];
                        var b1 = h('button.ph-btn.ph-btn--ghost.ph-btn--sm' + (a1.danger ? '.ph-btn--danger-text' : ''), { type: 'button', title: a1.tooltip || a1.label, disabled: off }, [icon(a1.icon || (a1.danger ? 'alert' : 'bolt')), a1.label]);
                        b1.addEventListener('click', function (e) { e.stopPropagation(); runAction(lv, a1, row, b1); });
                        btns.push(b1);
                    } else if (racts.length > 1) {
                        // Several actions: one button, a menu (danger ones last, in red).
                        var mb = h('button.ph-btn.ph-btn--ghost.ph-btn--sm.ph-prow__menu', { type: 'button', title: 'Actions: ' + racts.map(function (a) { return a.label; }).join(', '), 'aria-label': 'Actions', disabled: off }, [icon('dots'), icon('chevdown')]);
                        mb.addEventListener('click', function (e) {
                            e.stopPropagation();
                            var safe = racts.filter(function (a) { return !a.danger; }), bad = racts.filter(function (a) { return a.danger; });
                            var items = safe.map(function (a) { return { label: a.label, icon: a.icon || 'bolt', title: a.tooltip || null, onClick: function () { runAction(lv, a, row, mb); } }; });
                            if (safe.length && bad.length) items.push('-');
                            bad.forEach(function (a) { items.push({ label: a.label, icon: a.icon || 'alert', danger: true, title: a.tooltip || null, onClick: function () { runAction(lv, a, row, mb); } }); });
                            PH.menu(mb, items, { alignRight: true, minWidth: 240 });
                        });
                        btns.push(mb);
                    }
                    if (row.container) btns.push(h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'What is inside ' + row.container, disabled: off, onclick: function (e) { e.stopPropagation(); openContents(lv, row); } }, [icon('box'), 'Contents']));
                    detailsOf(row).forEach(function (d) {
                        var main = d === row.detail;
                        btns.push(h('button.ph-btn.ph-btn--sm.ph-prow__open' + (main ? '.ph-btn--ghost.is-main' : '.ph-btn--ghost'), {
                            type: 'button', title: 'Open ' + subLabel(lv, d).toLowerCase() + ' for this ' + itemWord(lv), disabled: off,
                            onclick: function (e) { e.stopPropagation(); openDetail(lv, row, d); },
                        }, [subLabel(lv, d), icon('right')]));
                    });
                    tr.appendChild(h('td.ph-table__acts', h('div.ph-rowacts.ph-prow__acts', btns)));
                }
                tbody.appendChild(tr);
                if (row.note) {
                    // The note goes on its own line under the row, across the whole
                    // table, so it never widens the first column for every row.
                    tr.classList.add('has-note');
                    var span = cols.length + (anyActs ? 1 : 0);
                    tbody.appendChild(h('tr.ph-prow__noterow' + tone, h('td', { colSpan: span },
                        h('div.ph-rowflag.is-' + (row.level === 'error' ? 'bad' : row.level === 'info' ? 'info' : 'warn'),
                            [icon(row.level === 'info' ? 'info' : 'alert'), h('span.ph-rowflag__text', row.note)]))));
                }
            });
            table.appendChild(tbody);
            body.appendChild(h('div.ph-tablewrap', table));
            if (pages > 1) {
                var pager = h('div.ph-pager');
                pager.appendChild(h('button.ph-iconbtn', { type: 'button', title: 'Previous page', disabled: lv.page === 0, onclick: function () { lv.page--; drawBody(); } }, icon('left')));
                pager.appendChild(h('span.ph-pager__gap', (lv.page + 1) + ' / ' + pages));
                pager.appendChild(h('button.ph-iconbtn', { type: 'button', title: 'Next page', disabled: lv.page >= pages - 1, onclick: function () { lv.page++; drawBody(); } }, icon('right')));
                body.appendChild(pager);
            }
        }

        function draw() {
            drawHead();
            drawBar();
            drawBanners();
            drawFlags();
            drawBody();
        }

        el.refresh = function () { load(top()); };
        searchInput.value = top().q || '';
        draw();
        // Read on open when never read, or when it was read more than a few seconds ago.
        var lv0 = top();
        if (!lv0.loading && (!lv0.data || !lv0.readAt || Date.now() - lv0.readAt > 3000)) load(lv0);
        return el;
    };
})();
