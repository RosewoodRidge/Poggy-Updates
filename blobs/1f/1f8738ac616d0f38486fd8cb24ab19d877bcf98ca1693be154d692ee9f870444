/*
    Poggy Hub — the pages: home (the store), a script, and its tabs.

    PH.go.home() / PH.go.script(id, { tab, item, row, sub, entry, reveal }) /
    PH.go.roles(name) switch the view; PH.render() draws whichever is current.
    Each page draws into PH.els.top (the bar) and PH.els.main (the scrolling area).

    A script page is organised as script › tab › section or list › row (§8):
    the rail lists the tabs, each opening to its settings sections and the
    lists, collections and key tables under it; Commands, Help and History sit
    apart as reference. The top bar carries the breadcrumbs. PH.navTo() is the
    one way to move inside a script; search results, "Open" on a reference,
    history rows and the palette all go through it (via PH.reveal for paths).
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var h = PH.h, icon = PH.icon, S = PH.S, F = PH.F, L = PH.L;

    PH.view = 'home';
    PH.home = { cat: 'all', run: 'all', changed: false, editing: false, sort: 'name', q: '' };
    PH.go = {};

    // ------------------------------------------------------------ helpers --

    function scripts() { return (PH.boot && PH.boot.scripts) || []; }
    PH.cardById = function (id) { return scripts().filter(function (c) { return c.id === id; })[0] || null; };

    /** The card art: the script's drawn icon, or a monogram tile in its category colour. */
    PH.art = function (card, size) {
        var color = PH.categoryColor(card.category);
        var mono = h('div.ph-mono', { style: { '--mono': color } }, [
            h('span.ph-mono__letters', monogram(card.label || card.id)),
        ]);
        if (!card.icon) return mono;
        var img = h('img.ph-art__img', { src: card.icon, alt: '', draggable: 'false' });
        var wrap = h('div.ph-art' + (size ? '.ph-art--' + size : ''), [img]);
        img.addEventListener('error', function () {
            if (wrap.parentNode) wrap.parentNode.replaceChild(mono, wrap);
        });
        return wrap;
    };

    function monogram(label) {
        var words = String(label).replace(/^poggy[_\s]*/i, '').split(/[\s_\-]+/).filter(Boolean);
        if (!words.length) return 'P';
        if (words.length === 1) return words[0].slice(0, 2).replace(/^./, function (c) { return c.toUpperCase(); });
        return (words[0].charAt(0) + words[1].charAt(0)).toUpperCase();
    }

    function statusPill(card) {
        return h('span.ph-status' + (card.running ? '.is-on' : '.is-off'), [h('span.ph-status__dot'), card.running ? 'Running' : 'Stopped']);
    }

    function lockPill(card, short) {
        var holder = card.lock && card.lock.holder;
        if (!holder) return null;
        var mine = PH.boot && PH.boot.me && holder.name === PH.boot.me.name && S.cur && S.cur.id === card.id && S.cur.lock.mine;
        return h('span.ph-lockpill' + (mine ? '.is-mine' : ''), { title: (mine ? 'You are' : holder.name + ' is') + ' editing this script' },
            [icon('lock'), short ? (mine ? 'You' : holder.name) : (mine ? 'You are editing' : holder.name + ' is editing')]);
    }

    function topbar(opts) {
        var top = PH.els.top;
        PH.clear(top);
        var left = h('div.ph-top__left');
        if (opts.back) {
            left.appendChild(h('button.ph-btn.ph-btn--ghost.ph-top__back', { type: 'button', title: 'Back to all scripts', onclick: opts.back }, [icon('back'), h('span.ph-top__backlabel', 'All scripts')]));
            // Breadcrumbs: Markets › Catalog › Catalogs › GeneralStore › Sells, each a way back up.
            if (opts.crumbs) left.appendChild(h('nav.ph-crumbs', { id: 'ph-crumbs', 'aria-label': 'Where you are' }, h('span.ph-crumbs__item.is-current.is-script', opts.crumb || '')));
            else if (opts.crumb) left.appendChild(h('span.ph-top__crumb', opts.crumb));
        } else {
            left.appendChild(h('div.ph-brand', [
                h('span.ph-brand__mark', icon('star')),
                h('span.ph-brand__text', [h('span.ph-brand__name', 'Poggy Hub'), h('span.ph-brand__by', 'Settings for every Poggy script on this server')]),
            ]));
        }
        var search = h('input.ph-top__search', { type: 'text', placeholder: opts.searchPlaceholder || 'Search…', spellcheck: 'false', value: opts.searchValue || '' });
        search.addEventListener('input', PH.debounce(function () { opts.onSearch(search.value); }, 140));
        search.addEventListener('keydown', function (e) {
            if (e.key === 'Escape' && search.value) { e.stopPropagation(); search.value = ''; opts.onSearch(''); }
            if (e.key === 'Enter' && opts.onEnter) opts.onEnter(search.value);
        });
        var center = h('div.ph-top__center', h('label.ph-searchbox', [icon('search'), search,
            h('button.ph-kbd', { type: 'button', title: 'Search everything (Ctrl+K)', onclick: function () { PH.palette.open(); } }, 'Ctrl K')]));
        var right = h('div.ph-top__right', [
            h('button.ph-btn.ph-btn--ghost', { type: 'button', title: 'Master job and group lists', onclick: function () { PH.go.roles(); } }, [icon('users'), 'Roles']),
            h('button.ph-iconbtn.ph-iconbtn--lg', { type: 'button', title: 'Close (Esc)', onclick: function () { PH.requestClose(); } }, icon('x')),
        ]);
        top.appendChild(left);
        top.appendChild(center);
        top.appendChild(right);
        PH.els.search = search;
        return search;
    }

    function skeleton(kind) {
        if (kind === 'script') {
            return h('div.ph-skel', [
                h('div.ph-skel__head', [h('div.ph-skel__block.ph-skel__art'), h('div.ph-skel__lines', [h('div.ph-skel__line.w40'), h('div.ph-skel__line.w70.big'), h('div.ph-skel__line.w90'), h('div.ph-skel__line.w60')])]),
                h('div.ph-skel__cols', [h('div.ph-skel__rail', [1, 2, 3, 4, 5, 6].map(function () { return h('div.ph-skel__line.w80'); })),
                    h('div.ph-skel__panel', [1, 2, 3, 4, 5, 6, 7].map(function () { return h('div.ph-skel__row', [h('div.ph-skel__line.w40'), h('div.ph-skel__line.w30')]); }))]),
            ]);
        }
        return h('div.ph-skel', [1, 2, 3, 4, 5, 6, 7, 8].map(function () { return h('div.ph-skel__row', [h('div.ph-skel__line.w40'), h('div.ph-skel__line.w30')]); }));
    }
    PH.skeleton = skeleton;

    // --------------------------------------------------------------- home --

    PH.go.home = function () {
        return PH.leaveScript().then(function (ok) {
            if (!ok) return;
            PH.view = 'home';
            PH.render();
        });
    };

    function renderHome() {
        var st = PH.home;
        var boot = PH.boot || {};
        topbar({
            searchPlaceholder: 'Search scripts, settings and commands…', searchValue: st.q,
            onSearch: function (q) { st.q = q.trim(); drawGrid(); },
            onEnter: function () { var first = PH.$('.ph-card', PH.els.main); if (first) first.click(); },
        });

        var main = PH.els.main;
        PH.clear(main);
        var all = scripts();
        var running = all.filter(function (c) { return c.running; }).length;
        var changed = all.filter(function (c) { return c.changedCount > 0; }).length;
        var editing = all.filter(function (c) { return c.lock && c.lock.holder; }).length;

        var page = h('div.ph-home.ph-page');
        page.appendChild(h('section.ph-masthead', [
            h('p.ph-eyebrow', ['Poggy Scripts', boot.coreVersion ? ' · poggy_core ' + boot.coreVersion : '']),
            h('h1.ph-masthead__title', 'Poggy Hub'),
            h('p.ph-masthead__lede', ['Every Poggy script on this server, with its settings, lists, commands and help. ',
                'Changes are written straight into each script’s own config file, and a restart applies them.']),
            h('ul.ph-stats', [
                stat(all.length, 'scripts'), stat(running, 'running'), stat(changed, 'changed from default'), stat(editing, 'being edited'),
            ]),
        ]));

        // Category chips.
        var cats = ['all'].concat(boot.categories && boot.categories.length ? boot.categories : uniq(all.map(function (c) { return c.category || 'Utility'; })));
        var chips = h('div.ph-chipsbar', cats.map(function (cat) {
            var n = cat === 'all' ? all.length : all.filter(function (c) { return (c.category || 'Utility') === cat; }).length;
            if (!n && cat !== 'all') return null;
            var b = h('button.ph-catchip' + (st.cat === cat ? '.is-active' : ''), { type: 'button', style: { '--cat': cat === 'all' ? '' : PH.categoryColor(cat) } },
                [cat === 'all' ? 'All' : cat, h('span.n', String(n))]);
            b.addEventListener('click', function () { st.cat = cat; PH.$$('.ph-catchip', chips).forEach(function (x) { x.classList.remove('is-active'); }); b.classList.add('is-active'); drawGrid(); });
            return b;
        }));

        // Filter bar.
        function seg(value, label) {
            var b = h('button.ph-seg__btn' + (st.run === value ? '.is-on' : ''), { type: 'button' }, label);
            b.addEventListener('click', function () { st.run = value; PH.$$('.ph-seg__btn', segEl).forEach(function (x) { x.classList.remove('is-on'); }); b.classList.add('is-on'); drawGrid(); });
            return b;
        }
        var segEl = h('div.ph-seg', [seg('all', 'All'), seg('running', 'Running'), seg('stopped', 'Stopped')]);
        function toggleChip(key, label, ic) {
            var b = h('button.ph-fchip' + (st[key] ? '.is-on' : ''), { type: 'button' }, [icon(ic), label]);
            b.addEventListener('click', function () { st[key] = !st[key]; b.classList.toggle('is-on', st[key]); drawGrid(); });
            return b;
        }
        var SORTS = { name: 'Name', category: 'Category', changed: 'Recently changed' };
        var sortBtn = h('button.ph-sortbtn', { type: 'button' }, [icon('sort'), h('span', 'Sort: '), h('b', SORTS[st.sort])]);
        sortBtn.addEventListener('click', function () {
            PH.menu(sortBtn, Object.keys(SORTS).map(function (k) {
                return { label: SORTS[k], icon: st.sort === k ? 'check' : null, onClick: function () { st.sort = k; sortBtn.querySelector('b').textContent = SORTS[k]; drawGrid(); } };
            }), { alignRight: true, minWidth: 220 });
        });

        page.appendChild(h('div.ph-toolbar', [
            chips,
            h('div.ph-filters', [segEl, toggleChip('changed', 'Has changes', 'pencil'), toggleChip('editing', 'Being edited', 'lock'), h('span.ph-grow'), sortBtn]),
        ]));

        var grid = h('div.ph-grid');
        var results = h('div.ph-homeresults');
        var none = h('div.ph-empty.ph-empty--big');
        page.appendChild(grid);
        page.appendChild(none);
        page.appendChild(results);
        main.appendChild(page);

        function drawGrid() {
            PH.clear(grid); PH.clear(results); PH.clear(none);
            var list = all.filter(function (c) {
                if (st.cat !== 'all' && (c.category || 'Utility') !== st.cat) return false;
                if (st.run === 'running' && !c.running) return false;
                if (st.run === 'stopped' && c.running) return false;
                if (st.changed && !(c.changedCount > 0)) return false;
                if (st.editing && !(c.lock && c.lock.holder)) return false;
                if (st.q && !PH.matches(st.q, [c.label, c.id, c.folder, c.tagline, c.category, c.description])) return false;
                return true;
            });
            list.sort(function (a, b) {
                if (st.sort === 'category') return String(a.category).localeCompare(String(b.category)) || String(a.label).localeCompare(String(b.label));
                if (st.sort === 'changed') {
                    var ta = a.changedAt || a.lastChanged || 0, tb = b.changedAt || b.lastChanged || 0;
                    if (ta !== tb) return String(tb) > String(ta) ? 1 : -1;
                    return (b.changedCount || 0) - (a.changedCount || 0) || String(a.label).localeCompare(String(b.label));
                }
                return String(a.label).localeCompare(String(b.label));
            });
            list.forEach(function (c, i) { grid.appendChild(card(c, i)); });
            if (!list.length) {
                none.appendChild(icon('search', 'ph-empty__ico'));
                none.appendChild(h('p', st.q ? 'No script matches “' + st.q + '”.' : 'No script matches these filters.'));
                none.appendChild(h('button.ph-btn.ph-btn--ghost', { type: 'button', onclick: function () {
                    st.cat = 'all'; st.run = 'all'; st.changed = false; st.editing = false; st.q = ''; PH.render();
                } }, 'Clear filters'));
            }
            if (st.q) {
                var hits = PH.palette.search(st.q, { noScripts: true }).slice(0, 40);
                if (hits.length) {
                    results.appendChild(h('div.ph-sechead', [h('h2', 'Settings and commands'), h('span.ph-sechead__count', PH.plural(hits.length, 'match', 'matches'))]));
                    results.appendChild(h('div.ph-hits', hits.map(function (hit) { return PH.palette.row(hit, function () { PH.palette.choose(hit); }); })));
                }
            }
        }
        drawGrid();
    }

    function stat(n, label) { return h('li', [h('b', String(n)), h('span', label)]); }
    function uniq(a) { return a.filter(function (x, i) { return a.indexOf(x) === i; }); }

    function card(c, i) {
        var el = h('article.ph-card', { tabindex: '0', role: 'button', 'aria-label': 'Open ' + c.label, style: { animationDelay: Math.min(i, 16) * 25 + 'ms' } });
        var media = h('div.ph-card__media', { style: { '--cat': PH.categoryColor(c.category) } });
        if (c.icon) media.appendChild(h('div.ph-card__blur', { style: { backgroundImage: 'url("' + c.icon + '")' } }));
        media.appendChild(PH.art(c));
        media.appendChild(h('div.ph-card__badges', [statusPill(c), lockPill(c, true)]));
        if (c.changedCount > 0) media.appendChild(h('span.ph-card__changed', { title: 'Settings that differ from the shipped default' }, [icon('pencil'), c.changedCount + ' changed']));
        var menuBtn = h('button.ph-iconbtn.ph-card__menu', { type: 'button', title: 'More' }, icon('dots'));
        menuBtn.addEventListener('click', function (e) {
            e.stopPropagation();
            PH.menu(menuBtn, [
                { label: 'Open settings', icon: 'sliders', onClick: function () { PH.go.script(c.id); } },
                { label: 'Commands', icon: 'terminal', onClick: function () { PH.go.script(c.id, { tab: 'commands' }); } },
                { label: 'Help', icon: 'book', onClick: function () { PH.go.script(c.id, { tab: 'help' }); } },
                { label: 'History', icon: 'history', onClick: function () { PH.go.script(c.id, { tab: 'history' }); } },
                '-',
                c.running
                    ? { label: 'Restart', icon: 'restart', disabled: PH.isCore(c.id), title: PH.isCore(c.id) ? 'Restart poggy_core by hand; it restarts every Poggy script.' : null, onClick: function () { PH.restartScript(c.id); } }
                    : { label: 'Start', icon: 'play', onClick: function () { PH.startScript(c.id); } },
                PH.isCore(c.id) ? { label: 'Roles', icon: 'users', onClick: function () { PH.go.roles(); } } : null,
            ], { alignRight: true });
        });
        media.appendChild(menuBtn);
        el.appendChild(media);

        el.appendChild(h('div.ph-card__body', [
            h('div.ph-card__tags', [h('span.ph-tag.ph-tag--cat', { style: { '--cat': PH.categoryColor(c.category) } }, c.category || 'Utility'), h('span.ph-tag', 'v' + (c.version || '?'))]),
            h('h3.ph-card__title', c.label || c.id),
            h('p.ph-card__tagline', c.tagline || c.description || c.folder),
        ]));
        el.appendChild(h('div.ph-card__foot', [
            h('span.ph-card__meta', [c.settingsCount != null ? PH.plural(c.settingsCount, 'setting') : c.folder, c.folder !== c.id ? h('span.ph-card__folder', ' · ' + c.folder) : null]),
            h('span.ph-card__open', ['Open', icon('right')]),
        ]));
        el.addEventListener('click', function () { PH.go.script(c.id); });
        el.addEventListener('keydown', function (e) { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); PH.go.script(c.id); } });
        return el;
    }

    // ------------------------------------------------------------- script --

    /*
        Where the owner is inside a script:
            tab      a navigation tab id, or 'commands' / 'help' / 'history'
            item     the path of the list, collection or key table open on that tab (null: the tab's own page)
            row      the selected row of a collection
            sub      the selected list inside that row
            section  a settings section to scroll to once
    */
    PH.scriptView = { tab: null, item: null, row: null, sub: null, section: null, q: '', changedOnly: false, advanced: false, loading: false, helpPage: 0 };

    /**
     * Open a script. opts: { tab, item, row, sub, entry, reveal: path }.
     * Leaving another script first asks about its unsaved changes.
     */
    PH.go.script = function (id, opts) {
        opts = opts || {};
        if (S.cur && S.cur.id === id && PH.view === 'script') {
            if (opts.reveal) PH.reveal(opts.reveal);
            else if (opts.tab || opts.item) PH.navTo(opts);
            return Promise.resolve(true);
        }
        return PH.leaveScript().then(function (ok) {
            if (!ok) return false;
            PH.view = 'script';
            var v = PH.scriptView;
            v.tab = opts.tab || null; v.item = opts.item || null; v.row = opts.row !== undefined ? opts.row : null; v.sub = opts.sub || null;
            v.section = null; v.q = ''; v.changedOnly = false; v.advanced = false; v.loading = true; v.helpPage = 0;
            v.history = null;
            v.id = id;
            S.cur = null;
            PH.render();
            return PH.loadScript(id).then(function (loaded) {
                if (!loaded || PH.scriptView.id !== id) return false;
                if (opts.reveal) setTimeout(function () { PH.reveal(opts.reveal); }, 60);
                else if (opts.entry !== undefined && opts.entry !== null) setTimeout(function () { PH.navTo(opts); }, 60);
                return true;
            });
        });
    };

    /** Fetch a script, take its lock when free, and draw it. */
    PH.loadScript = function (id) {
        return PH.api('script', id).then(function (r) {
            if (PH.scriptView.id !== id || PH.view !== 'script') return false;
            if (!r.ok) {
                PH.scriptView.loading = false;
                PH.toast({ kind: 'error', title: 'Could not open the script', text: PH.errMsg(r) });
                PH.view = 'home';
                PH.render();
                return false;
            }
            S.cur = S.build(r.value);
            S.cur.id = id;
            PH.scriptView.loading = false;
            computeTabs();
            var card0 = PH.cardById(id);
            if (card0 && r.value.card) Object.assign(card0, r.value.card);
            return PH.lock.acquire(id).then(function () {
                PH.render();
                return true;
            });
        });
    };

    // ------------------------------------------------------ the navigation --
    // §8.1: the server sends `nav` (tabs, their setting sections, and the
    // lists and collections under each). An older server sends none; then the
    // page works it out the same way (§8.2, §8.3). Either way the result is
    // one shape:
    //     tab  = { id, label, icon, sections: [{ id, label, nodes }], items: [item] }
    //     item = { path, label, kind: list|map|collection|keytable, node, tab }

    // Kinds that hold their own contents: nothing inside them is a setting of its own.
    var CONTAINER = { list: 1, map: 1, strings: 1, readonly: 1 };
    var GENERIC = { sell: 1, buy: 1, items: 1, item: 1, list: 1, entries: 1, data: 1, rows: 1, options: 1, values: 1, loot: 1, rewards: 1, config: 1, settings: 1 };
    var KIND_ICON = { list: 'list', map: 'rows', collection: 'grid', keytable: 'keyboard' };
    PH.KIND_ICON = KIND_ICON;

    function nodeList() { var cur = S.cur; return cur.order.map(function (p) { return cur.nodes[p]; }).filter(Boolean); }
    function parentOf(node) { return node && node.parent ? S.cur.nodes[node.parent] || S.node(node.parent) : null; }

    function ancestorOf(node, fn) {
        var p = parentOf(node);
        while (p) {
            if (fn(p)) return p;
            p = parentOf(p);
        }
        return null;
    }

    function isHidden(node) {
        if (F.metaFor(node).hidden) return true;
        return !!ancestorOf(node, function (a) { return F.metaFor(a).hidden; });
    }

    function isData(node) { return node.kind === 'list' || node.kind === 'map'; }

    function childGroups(node) {
        return nodeList().filter(function (n) { return n.parent === node.path && n.kind === 'group'; });
    }

    /** A map of control names to hashes (§8.9). */
    function isKeyTable(node) {
        var m = F.metaFor(node);
        if (node.display === 'keytable' || m.display === 'keytable') return true;
        if (m.display) return false;
        if (node.kind !== 'map') return false;
        var v = S.get(node.path);
        if (!PH.isPlainObj(v)) return false;
        var ks = Object.keys(v).filter(function (k) { return k !== '__int_keys'; });
        if (ks.length < 3 || !/key|control|bind|input/i.test(String(node.key))) return false;
        return ks.every(function (k) { var x = v[k]; return PH.isHash(x) || (typeof x === 'number' && x >= 65536); });
    }

    /** §8.4 detection, used when the server has not said: at least half the rows hold a list of tables. */
    function looksLikeCollection(node) {
        var raw = ((S.cur.meta && S.cur.meta.lists) || {})[node.path] || {};
        if (raw.kind === 'map' || raw.kind === 'list') return false;
        var v = S.get(node.path);
        var rows = Array.isArray(v) ? v : PH.isPlainObj(v) ? Object.keys(v).filter(function (k) { return k !== '__int_keys'; }).map(function (k) { return v[k]; }) : [];
        if (rows.length < 1) return false;
        var hits = rows.filter(function (r) {
            return PH.isPlainObj(r) && Object.keys(r).some(function (k) { return Array.isArray(r[k]) && r[k].some(PH.isPlainObj); });
        }).length;
        return hits * 2 >= rows.length && hits > 0;
    }

    /** Identifier-keyed tables parsed as groups: a group whose child groups mostly hold lists. */
    function groupCollection(node) {
        if (node.kind !== 'group' || !node.parent) return null;
        var kids = childGroups(node);
        if (kids.length < 2) return null;
        var withLists = kids.filter(function (g) { return nodeList().some(function (n) { return n.parent === g.path && isData(n); }); });
        return withLists.length * 2 >= kids.length ? kids : null;
    }

    function isCollectionNode(node) {
        var m = F.metaFor(node);
        return !!(node.collection || m.kind === 'collection' || node.kind === 'collection');
    }

    function makeItem(node, label, kind, tab) {
        var cur = S.cur;
        var it = { path: node.path, label: label, kind: kind, node: node, tab: tab };
        cur.navLabel[node.path] = label;
        cur.nav.itemByCanon[PH.canon(node.path)] = it;
        return it;
    }

    function registerCollection(node) {
        var cur = S.cur;
        if (cur.coll[node.path]) return cur.coll[node.path];
        var groups = node.value === undefined ? childGroups(node) : null;
        cur.coll[node.path] = L.collInfo(cur, node, groups);
        return cur.coll[node.path];
    }

    /** Anything inside a list, map, collection or key table is edited as part of it. */
    function insideItem(node) {
        var cur = S.cur;
        return !!ancestorOf(node, function (a) { return CONTAINER[a.kind] || cur.coll[a.path] || cur.nav.itemByCanon[PH.canon(a.path)]; });
    }

    function navFromServer(cur) {
        var tabs = [], byId = {};
        cur.data.nav.forEach(function (t) {
            if (!t || !t.id || byId[t.id]) return;
            var tab = { id: t.id, label: t.label || PH.readable(t.id), icon: t.icon || t.id, sections: [], secById: {}, items: [], server: t };
            (t.sections || []).forEach(function (s) {
                if (!s || s.id === undefined || tab.secById[s.id]) return;
                var sec = { id: String(s.id), label: String(s.id) === String(t.id) ? '' : (s.label || ''), nodes: [] };
                tab.secById[sec.id] = sec;
                tab.sections.push(sec);
            });
            byId[t.id] = tab;
            tabs.push(tab);
        });
        cur.nav.tabs = tabs; cur.nav.byId = byId;
        cur.data.nav.forEach(function (t) {
            var tab = byId[t.id];
            (t.items || []).forEach(function (it) {
                var node = it && it.path ? S.node(it.path) : null;
                if (!node || cur.nav.itemByCanon[PH.canon(node.path)]) return;
                var kind = it.kind === 'collection' || isCollectionNode(node) ? 'collection'
                    : (it.kind === 'keytable' || it.display === 'keytable' || isKeyTable(node)) ? 'keytable'
                    : (it.kind === 'map' || it.kind === 'list') ? it.kind : node.kind === 'map' ? 'map' : 'list';
                if (kind === 'collection') registerCollection(node);
                tab.items.push(makeItem(node, it.label || F.labelFor(node), kind, tab));
            });
        });
        function tabFor(id) {
            if (byId[id]) return byId[id];
            if (!id) return tabs[0];
            var t2 = { id: id, label: id === 'general' ? 'General' : PH.readable(id), icon: id, sections: [], secById: {}, items: [] };
            byId[id] = t2;
            tabs.push(t2);
            return t2;
        }
        // Settings go to their tab and section; lists the server left out still show.
        nodeList().forEach(function (node) {
            if (isHidden(node) || node.kind === 'group' || insideItem(node)) return;
            if (cur.nav.itemByCanon[PH.canon(node.path)]) return;
            var m = F.metaFor(node);
            var tab = tabFor(node.tab || m.tab);
            if (!tab) return;
            if (isData(node)) {
                var kind = isCollectionNode(node) ? 'collection' : isKeyTable(node) ? 'keytable' : node.kind;
                if (kind === 'collection') registerCollection(node);
                tab.items.push(makeItem(node, F.labelFor(node), kind, tab));
                return;
            }
            var sid = node.section === undefined || node.section === null ? '' : String(node.section);
            var sec = tab.secById[sid];
            if (!sec) {
                var sn = sid ? S.node(sid) : null;
                sec = { id: sid, label: sid ? (sn ? F.labelFor(sn) : PH.readable(sid.split('.').pop())) : '', nodes: [] };
                tab.secById[sid] = sec;
                if (sid) tab.sections.push(sec); else tab.sections.unshift(sec);
            }
            sec.nodes.push(node);
            cur.nav.tabOfNode[node.path] = tab.id;
        });
    }

    /** §8.2 and §8.3 in the page, for a server that sends no `nav`. */
    function navDerived(cur) {
        var meta = cur.meta || {};
        var authored = Array.isArray(meta.tabs) && meta.tabs.length > 0;
        var declared = Array.isArray(cur.data.tabs) && cur.data.tabs.length ? cur.data.tabs : (meta.tabs || []);
        var files = (cur.data.files || []).map(function (f) { return f.file; });
        nodeList().forEach(function (n) { if (n.file && files.indexOf(n.file) === -1) files.push(n.file); });
        var multi = files.length > 1;
        var tabs = [], byId = {};
        function tab(id, label, ic) {
            if (byId[id]) return byId[id];
            var t = { id: id, label: label, icon: ic || id, sections: [], secById: {}, items: [] };
            byId[id] = t;
            tabs.push(t);
            return t;
        }
        if (authored) declared.forEach(function (t) { if (t && t.id) tab(t.id, t.label || PH.readable(t.id), t.icon || t.id); });

        function isText(node) {
            var root = String(node.path).split(/[.\[]/)[0];
            return /(^|\/)(translations?|locales?|lang(uage)?s?)[^\/]*\.lua$/i.test(node.file || '') || /^(Translations?|Locales?|Lang|Language)$/.test(root);
        }
        function fileTab(file) {
            if (/(^|\/)config\.lua$/i.test(file)) return tab('general', 'General', 'sliders');
            var base = String(file).split('/').pop().replace(/\.lua$/i, '');
            return tab('file:' + file, PH.readable(base), 'file');
        }
        function top(node) {
            var n = node;
            while (n && n.parent && parentOf(n) && parentOf(n).parent) n = parentOf(n);
            return n;
        }
        var bigMemo = {};
        function bigGroup(g) {
            if (bigMemo[g.path] !== undefined) return bigMemo[g.path];
            var count = 0, hasList = false;
            nodeList().forEach(function (n) {
                if (!S.under(n.path, g.path) || n.path === g.path) return;
                if (isData(n)) hasList = true;
                else if (n.kind !== 'group') count++;
            });
            return (bigMemo[g.path] = hasList || count >= 8);
        }
        function tabOf(node) {
            var m = F.metaFor(node);
            if (m.tab && (authored || m.tab !== 'general')) return byId[m.tab] || tab(m.tab, m.tab === 'general' ? 'General' : PH.readable(m.tab), m.tab);
            if (isText(node)) return tab('text', 'Text & language', 'globe');
            if (multi) return fileTab(node.file);
            var t = top(node);
            if (t && t !== node && t.kind === 'group' && bigGroup(t)) return tab('grp:' + t.path, F.labelFor(t), 'sliders');
            if (t === node && node.kind === 'group' && bigGroup(node)) return tab('grp:' + node.path, F.labelFor(node), 'sliders');
            return tab('general', 'General', 'sliders');
        }

        // Collections and key tables first, so what is inside them is left alone.
        var found = [];
        nodeList().forEach(function (node) {
            if (isHidden(node) || ancestorOf(node, function (a) { return CONTAINER[a.kind] || cur.coll[a.path]; })) return;
            if (isData(node)) {
                var kind = isCollectionNode(node) || looksLikeCollection(node) ? 'collection' : isKeyTable(node) ? 'keytable' : node.kind;
                if (kind === 'collection') registerCollection(node);
                found.push({ node: node, kind: kind });
            } else if (node.kind === 'group' && groupCollection(node)) {
                registerCollection(node);
                found.push({ node: node, kind: 'collection' });
            }
        });

        // §8.3: a generic or repeated name gets its parent's name in front.
        var rawLists = meta.lists || {}, rawSettings = meta.settings || {};
        function authoredLabel(node) {
            var a = rawLists[node.path] || rawSettings[node.path];
            return a && a.label ? a.label : null;
        }
        var labels = found.map(function (f) { return authoredLabel(f.node) || F.metaFor(f.node).label || PH.readable(f.node.key); });
        var seen = {};
        labels.forEach(function (l) { seen[l] = (seen[l] || 0) + 1; });
        found.forEach(function (f, i) {
            var l = labels[i];
            if (!authoredLabel(f.node) && (GENERIC[String(f.node.key).toLowerCase()] || seen[l] > 1)) {
                var p = parentOf(f.node);
                if (p && p.parent) l = (F.metaFor(p).label || PH.readable(p.key)) + ' · ' + l;
            }
            var t = tabOf(f.node);
            t.items.push(makeItem(f.node, l, f.kind, t));
        });

        // Settings, under the nearest group's heading.
        nodeList().forEach(function (node) {
            if (isHidden(node) || node.kind === 'group' || isData(node) || insideItem(node)) return;
            var t = tabOf(node);
            var title = groupTitle(node);
            if (title === t.label) title = '';
            var sid = title ? 'sec:' + title : '';
            var sec = t.secById[sid];
            if (!sec) {
                sec = { id: sid, label: title, nodes: [] };
                t.secById[sid] = sec;
                if (sid) t.sections.push(sec); else t.sections.unshift(sec);
            }
            sec.nodes.push(node);
            cur.nav.tabOfNode[node.path] = t.id;
        });

        // General first, then file order, text and advanced last.
        function rank(t) { return t.id === 'general' ? 0 : t.id === 'text' ? 2 : t.id === 'advanced' ? 3 : 1; }
        var order = tabs.slice();
        tabs.sort(function (a, b) { return rank(a) - rank(b) || order.indexOf(a) - order.indexOf(b); });
        cur.nav.tabs = tabs; cur.nav.byId = byId;
    }

    /**
     * The heading a setting sits under. The server fills meta.group with the
     * nearest group's label; a setting straight under the root (Config) gets
     * the root's label, which is no heading at all.
     */
    function groupTitle(node) {
        var m = F.metaFor(node);
        var parent = parentOf(node);
        var rawSettings = (S.cur.meta && S.cur.meta.settings) || {};
        var explicit = rawSettings[node.path] && rawSettings[node.path].group;
        if (explicit) return explicit;
        if (parent && !parent.parent && (!m.group || m.group === F.metaFor(parent).label || m.group === parent.key)) return '';
        if (m.group) return m.group;
        var parts = [];
        var p = parent;
        while (p && p.kind === 'group' && p.parent) {
            parts.unshift(F.metaFor(p).label || PH.readable(p.key));
            p = parentOf(p);
        }
        return parts.join(' › ');
    }

    /** Work out the tabs, their sections and items, once per load. */
    function computeTabs() {
        var cur = S.cur;
        cur.coll = {};
        cur.navLabel = {};
        cur._virt = {};
        cur._meta = {};
        cur.nav = { tabs: [], byId: {}, itemByCanon: {}, tabOfNode: {} };
        if (Array.isArray(cur.data.nav) && cur.data.nav.length) navFromServer(cur);
        else navDerived(cur);
        cur.nav.tabs = cur.nav.tabs.filter(function (t) {
            return t.items.length || t.sections.some(function (s) { return s.nodes.length; });
        });
        cur.nav.byId = {};
        cur.nav.tabs.forEach(function (t) { cur.nav.byId[t.id] = t; t.kind = 'settings'; });
        PH.R.prepare(cur);

        var ref = [];
        if (PH.commandsOf().length) ref.push({ id: 'commands', label: 'Commands', icon: 'terminal', kind: 'commands' });
        var d = cur.data;
        if (d.readme || (d.help && d.help.length)) ref.push({ id: 'help', label: 'Help', icon: 'book', kind: 'help' });
        ref.push({ id: 'history', label: 'History', icon: 'history', kind: 'history' });
        cur.nav.ref = ref;
        cur.tabs = cur.nav.tabs.concat(ref);
        cur.tabById = {};
        cur.tabs.forEach(function (t) { cur.tabById[t.id] = t; });
        var v = PH.scriptView;
        if (!v.tab || !cur.tabById[v.tab]) { v.tab = cur.tabs[0] ? cur.tabs[0].id : 'history'; v.item = null; }
        if (v.item && !itemAt(v.item)) v.item = null;
    }
    PH.computeTabs = computeTabs;

    function itemAt(path) { return S.cur && S.cur.nav ? S.cur.nav.itemByCanon[PH.canon(path)] || null : null; }
    PH.itemAt = itemAt;

    PH.commandsOf = function () {
        var d = S.cur && S.cur.data;
        if (!d) return [];
        return d.commands || (d.meta && d.meta.commands) || [];
    };

    // --------------------------------------------------------------- counts --

    /** Is this setting shown under the current "changed only" / "advanced" switches? */
    function fieldVisible(node) {
        var v = PH.scriptView;
        if (F.metaFor(node).advanced && !v.advanced && !v.q) return false;
        if (v.changedOnly && !(S.differsFromDefault(node) || S.isPending(node.path) || node.changed)) return false;
        return true;
    }

    function itemVisible(it) {
        var v = PH.scriptView;
        if (F.metaFor(it.node).advanced && !v.advanced) return false;
        if (v.changedOnly && !itemStats(it).changed) return false;
        return true;
    }

    function itemStats(it) {
        var cur = S.cur;
        if (it.kind === 'collection') {
            var info = cur.coll[it.path];
            var ks = info.rowKeys();
            var changed = S.isPending(it.path) || S.differsFromDefault(it.node) ||
                (info.virtual && ks.some(function (k) { return L.rowChanged(info, k); }));
            return { n: ks.length, changed: !!changed };
        }
        var meta = Object.assign({}, F.metaFor(it.node), it.kind === 'keytable' ? { kind: 'map' } : {});
        return { n: L.rowsOf(it.node, meta).length, changed: S.isPending(it.path) || S.differsFromDefault(it.node) };
    }
    PH.itemStats = itemStats;

    function tabStats(t) {
        var n = 0, ch = false;
        t.sections.forEach(function (s) {
            s.nodes.forEach(function (node) {
                if (F.metaFor(node).advanced && !PH.scriptView.advanced) return;
                n++;
                if (!ch && (S.differsFromDefault(node) || S.isPending(node.path))) ch = true;
            });
        });
        t.items.forEach(function (it) {
            if (F.metaFor(it.node).advanced && !PH.scriptView.advanced) return;
            var st = itemStats(it);
            n += st.n;
            if (st.changed) ch = true;
        });
        return { n: n, changed: ch };
    }

    function sectionsShown(t) { return t.sections.filter(function (s) { return s.nodes.some(fieldVisible); }); }

    // --------------------------------------------------------- navigating --

    function railState() { return PH.store.get('rail:' + (S.cur ? S.cur.id : ''), {}); }
    function railOpen(tabId) {
        var st = railState();
        if (st[tabId] !== undefined) return !!st[tabId];
        return PH.scriptView.tab === tabId;
    }
    function setRailOpen(tabId, open) {
        var st = railState();
        st[tabId] = !!open;
        PH.store.set('rail:' + S.cur.id, st);
    }

    function clearSearch() {
        var v = PH.scriptView;
        if (v.q) { v.q = ''; if (PH.els.search) PH.els.search.value = ''; }
    }

    /**
     * Go somewhere inside the open script and draw it.
     * t: { tab, item, row, sub, section, entry (a row of a list, or of the sublist), field (a path to flash) }
     */
    PH.navTo = function (t) {
        var cur = S.cur;
        if (!cur) return;
        var v = PH.scriptView;
        clearSearch();
        var item = t.item ? itemAt(t.item) : null;
        v.tab = item ? item.tab.id : (t.tab && cur.tabById[t.tab] ? t.tab : v.tab);
        v.item = item ? item.path : null;
        v.row = t.row !== undefined ? t.row : null;
        v.sub = t.sub || null;
        v.section = t.section !== undefined ? t.section : null;
        if (t.advanced) v.advanced = true;
        if (t.field) v.changedOnly = false;
        if (cur.tabById[v.tab] && cur.tabById[v.tab].kind === 'settings') setRailOpen(v.tab, true);
        PH.renderScriptBody();
        var layout = document.querySelector('.ph-layout');
        if (!v.section && !t.field && layout) PH.els.main.scrollTop = Math.min(PH.els.main.scrollTop, layout.offsetTop || 0);
        setTimeout(function () { settle(t, item); }, 40);
    };

    /** After a navigation has drawn: open the row, flash the field. */
    function settle(t, item) {
        var content = document.getElementById('ph-content');
        if (!content) return;
        var v = PH.scriptView;
        if (v.section !== null && v.section !== undefined && !item) {
            var secEl = PH.$$('[data-sec]', content).filter(function (e) { return e.getAttribute('data-sec') === String(v.section); })[0];
            if (secEl) { PH.md.scrollToEl(secEl, PH.els.main); secEl.classList.add('is-flash'); setTimeout(function () { secEl.classList.remove('is-flash'); }, 1600); }
            v.section = null;
        }
        var drawerOpened = false;
        if (item && t.entry !== undefined && t.entry !== null) {
            if (item.kind === 'collection') {
                var cv = content.querySelector('.ph-collv');
                if (cv && cv.openEntry) { cv.openEntry(t.row, t.sub, t.entry); drawerOpened = true; }
            } else {
                var le = content.querySelector('.ph-list');
                if (le && le.openRow) { le.openRow(t.entry); drawerOpened = true; }
            }
        }
        if (!t.field) return;
        setTimeout(function () {
            var sel = '.ph-field[data-path="' + PH.pathSel(t.field) + '"]';
            var f = document.querySelector('.ph-drawer ' + sel);
            if (!f && drawerOpened && t.drawerTabs) {
                // A value inside a list of the row: switch the drawer to that list first.
                t.drawerTabs.some(function (name) { L.drawerTab(name); f = document.querySelector('.ph-drawer ' + sel); return !!f; });
            }
            if (!f) f = content.querySelector(sel);
            if (f) PH.flash(f);
            else if (drawerOpened) { var dr = document.querySelector('.ph-drawer'); if (dr) PH.flash(dr.querySelector('.ph-drawer__title')); }
            if (t.onFound) t.onFound(f);
        }, drawerOpened ? 280 : 40);
    }

    /** Where a path is shown: the navigation target for PH.navTo, or null. */
    function locate(path) {
        var cur = S.cur;
        var segs = PH.parsePath(path);
        for (var i = segs.length; i > 0; i--) {
            var p = String(segs[0]);
            for (var j = 1; j < i; j++) p = PH.joinPath(p, segs[j]);
            var item = cur.nav.itemByCanon[PH.canon(p)];
            if (!item) continue;
            var rest = segs.slice(i);
            var t = { tab: item.tab.id, item: item.path, advanced: F.metaFor(item.node).advanced };
            if (item.kind === 'collection') {
                var info = cur.coll[item.path];
                if (rest.length) t.row = rest[0];
                var isSub = rest.length > 1 && info.sublists.some(function (s) { return s.field === rest[1]; });
                if (isSub) { t.sub = rest[1]; if (rest.length > 2) t.entry = rest[2]; }
                if (rest.length > (isSub ? 3 : 1)) t.field = PH.canon(path);
                if (isSub && rest.length > 3) t.drawerTabs = rest.slice(3).filter(function (x) { return typeof x === 'string'; });
            } else {
                if (rest.length) t.entry = rest[0];
                if (rest.length > 1) { t.field = PH.canon(path); t.drawerTabs = rest.slice(1).filter(function (x) { return typeof x === 'string'; }); }
            }
            return t;
        }
        var r = S.resolve(path);
        if (!r) return null;
        var tabId = cur.nav.tabOfNode[r.node.path];
        if (!tabId) return null;
        var tab = cur.nav.byId[tabId];
        var sec = null;
        tab.sections.forEach(function (s) { if (s.nodes.indexOf(r.node) !== -1) sec = s; });
        return { tab: tabId, field: r.node.path, advanced: F.metaFor(r.node).advanced, sectionHint: sec ? sec.id : null };
    }
    PH.locate = locate;

    /** Go to a setting (or a list, a collection row, or a value deep inside one), scroll to it and flash it. */
    PH.reveal = function (path) {
        if (!S.cur) return;
        var t = locate(path);
        if (!t) {
            var r = S.resolve(path);
            PH.toast(r ? { kind: 'info', title: 'Hidden setting', text: path + ' is not shown in the hub.' } : { kind: 'warning', title: 'Not found', text: path + ' is not in this script.' });
            return;
        }
        PH.navTo(t);
    };

    /** "Open" on a reference: the row of the target list or collection with that key (§8.5). */
    PH.openRef = function (target, key, refField) {
        var item = itemAt(target);
        if (!item) { PH.reveal(target); return; }
        if (item.kind === 'collection') { PH.navTo({ item: item.path, row: key }); return; }
        var entry = key;
        if (refField) {
            L.rowsOf(item.node, F.metaFor(item.node)).forEach(function (r) { if (PH.isPlainObj(r.value) && String(r.value[refField]) === String(key)) entry = r.key; });
        }
        PH.navTo({ item: item.path, entry: entry });
    };

    // ------------------------------------------------------------ the page --

    function renderScript() {
        var v = PH.scriptView;
        var cur = S.cur;
        var card0 = PH.cardById(v.id) || { id: v.id, label: v.id };
        topbar({
            back: function () { PH.go.home(); },
            crumbs: true,
            crumb: cur ? (cur.card.label || card0.label) : card0.label,
            searchPlaceholder: 'Search this script’s settings and lists…',
            searchValue: v.q,
            onSearch: function (q) { v.q = q.trim(); PH.renderScriptBody(); },
        });
        var main = PH.els.main;
        PH.clear(main);
        if (v.loading || !cur) {
            main.appendChild(h('div.ph-page', skeleton('script')));
            return;
        }
        var page = h('div.ph-script.ph-page');
        page.appendChild(scriptHead());
        page.appendChild(h('div.ph-banners', { id: 'ph-banners' }));
        var rail = h('nav.ph-rail', { id: 'ph-rail', 'aria-label': 'Sections of this script' });
        var content = h('div.ph-content', { id: 'ph-content' });
        page.appendChild(h('div.ph-layout', [rail, content]));
        main.appendChild(page);
        PH.renderBanners();
        PH.renderScriptBody();
    }

    function scriptHead() {
        var cur = S.cur;
        var c = Object.assign({}, PH.cardById(cur.id) || {}, cur.card || {});
        var isCore = PH.isCore(c.id);
        var actions = h('div.ph-shead__actions', { id: 'ph-shead-actions' });
        function drawActions() {
            PH.clear(actions);
            var c2 = PH.cardById(cur.id) || c;
            if (c2.running) {
                var rb = h('button.ph-btn.ph-btn--ghost' + (cur.restartNeeded ? '.is-attention' : ''), {
                    type: 'button', disabled: isCore,
                    title: isCore ? 'Restart poggy_core by hand; it restarts every Poggy script.' : 'Restart ' + c2.label + ' to apply saved changes',
                    onclick: function () { PH.restartScript(cur.id); },
                }, [icon('restart'), 'Restart']);
                actions.appendChild(rb);
                if (cur.restartNeeded) actions.appendChild(h('span.ph-shead__hint', 'Saved changes wait for a restart'));
            } else {
                actions.appendChild(h('button.ph-btn.ph-btn--go', { type: 'button', onclick: function () { PH.startScript(cur.id); } }, [icon('play'), 'Start']));
            }
            if (isCore) actions.appendChild(h('span.ph-shead__hint', cur.data.restartNote || 'Restart poggy_core by hand: it restarts every Poggy script.'));
        }
        PH.drawHeadActions = drawActions;
        drawActions();

        var lockState = cur.lock.mine
            ? h('span.ph-lockpill.is-mine', [icon('pencil'), 'You are editing'])
            : cur.lock.holder ? h('span.ph-lockpill', [icon('lock'), cur.lock.holder.name + ' is editing']) : h('span.ph-lockpill.is-free', [icon('eye'), 'Read only']);

        return h('section.ph-shead', { style: { '--cat': PH.categoryColor(c.category) } }, [
            c.icon ? h('div.ph-shead__glow', { style: { backgroundImage: 'url("' + c.icon + '")' } }) : null,
            h('div.ph-shead__art', PH.art(c, 'big')),
            h('div.ph-shead__text', [
                h('p.ph-eyebrow', [c.category || 'Utility', h('span.ph-eyebrow__sep', '·'), h('span.ph-mono-inline', c.folder || c.id)]),
                h('h1.ph-shead__title', c.label || c.id),
                (c.description || c.tagline) ? h('p.ph-shead__desc', c.description || c.tagline) : null,
                h('div.ph-shead__tags', [
                    h('span.ph-tag', 'v' + (c.version || '?')),
                    statusPill(PH.cardById(cur.id) || c),
                    lockState,
                    (cur.data.files || []).length ? h('span.ph-tag.ph-tag--files', { title: (cur.data.files || []).map(function (f) { return f.file; }).join(', ') },
                        [icon('file'), PH.plural(cur.data.files.length, 'config file')]) : null,
                    cur.data.defaultsKnown === false ? h('span.ph-tag.ph-tag--muted', { title: 'The shipped config for this version could not be fetched, so changed badges and Reset are hidden.' }, 'Defaults unknown') : null,
                ]),
            ]),
            actions,
        ]);
    }

    PH.renderBanners = function () {
        var host = document.getElementById('ph-banners');
        var cur = S.cur;
        if (!host || !cur) return;
        PH.clear(host);
        var c = PH.cardById(cur.id) || cur.card || {};
        if (cur.lock.mine) {
            host.appendChild(banner('edit', 'pencil', [h('b', 'You are editing this script.'), ' Don’t edit this script’s config files by hand while you’re editing here.'], null));
        } else if (cur.lock.holder) {
            var since = PH.when(cur.lock.holder.since);
            host.appendChild(banner('lock', 'lock', [h('b', cur.lock.holder.name + ' is editing ' + (c.label || cur.id) + '.'),
                ' Since ' + (since.abs || since.rel) + (since.abs ? ' (' + since.rel + ')' : '') + '. You can look around, but nothing can be changed until they finish.'], [
                PH.boot && PH.boot.canTakeover ? h('button.ph-btn.ph-btn--danger.ph-btn--sm', { type: 'button', onclick: function () { PH.lock.takeover(cur.id); } }, [icon('unlock'), 'Take over']) : null,
                h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { PH.lock.acquire(cur.id, true); } }, [icon('restart'), 'Check again']),
            ]));
        } else {
            host.appendChild(banner('info', 'eye', [h('b', 'Read only.'), ' You are not holding this script’s edit lock.'], [
                h('button.ph-btn.ph-btn--primary.ph-btn--sm', { type: 'button', onclick: function () { PH.lock.acquire(cur.id, true); } }, [icon('pencil'), 'Start editing']),
            ]));
        }
        if (!c.running) {
            host.appendChild(banner('warn', 'stop', [h('b', 'This script is stopped.'), ' You can change its settings, but saving needs it running: a script writes its own files.'], [
                h('button.ph-btn.ph-btn--go.ph-btn--sm', { type: 'button', onclick: function () { PH.startScript(cur.id); } }, [icon('play'), 'Start']),
            ]));
        }
        if (cur.restartNeeded && c.running) {
            host.appendChild(banner('restart', 'restart', [h('b', 'Saved.'), ' Restart the script to apply the changes you saved.'], [
                h('button.ph-btn.ph-btn--primary.ph-btn--sm', { type: 'button', disabled: PH.isCore(cur.id), onclick: function () { PH.restartScript(cur.id); } }, [icon('restart'), 'Restart now']),
            ]));
        }
    };

    function banner(kind, ic, text, actions) {
        return h('div.ph-banner.ph-banner--' + kind, [icon(ic, 'ph-banner__ico'), h('div.ph-banner__text', text), actions ? h('div.ph-banner__actions', actions) : null]);
    }

    // --------------------------------------------------------- breadcrumbs --

    /** Script › Tab › List › Row › Sublist, each a way back up. */
    function crumbTrail() {
        var cur = S.cur, v = PH.scriptView;
        var out = [];
        if (!cur) return out;
        var c = PH.cardById(cur.id) || cur.card || {};
        var first = cur.tabs[0];
        out.push({ label: c.label || cur.id, go: first ? function () { PH.navTo({ tab: first.id }); } : null });
        if (v.q) { out.push({ label: 'Search: “' + v.q + '”' }); return out; }
        var t = cur.tabById[v.tab];
        if (!t) return out;
        out.push({ label: t.label, go: function () { PH.navTo({ tab: t.id }); } });
        var item = v.item ? itemAt(v.item) : null;
        if (!item) return out;
        if (item.label !== t.label || item.kind === 'collection') out.push({ label: item.label, go: function () { PH.navTo({ item: item.path }); } });
        if (item.kind === 'collection') {
            var info = cur.coll[item.path];
            var ks = info.rowKeys();
            var row = v.row !== null && v.row !== undefined && ks.some(function (k) { return String(k) === String(v.row); }) ? v.row : ks[0];
            if (row !== undefined && row !== null) {
                out.push({ label: info.isList() ? info.rowTitle(row) : String(row), go: function () { PH.navTo({ item: item.path, row: row }); } });
                var sub = info.sublists.filter(function (s) { return s.field === v.sub; })[0] || info.sublists[0];
                if (sub) out.push({ label: sub.label });
            }
        }
        return out;
    }

    function drawCrumbs() {
        var host = document.getElementById('ph-crumbs');
        if (!host) return;
        PH.clear(host);
        var trail = crumbTrail();
        trail.forEach(function (c, i) {
            var last = i === trail.length - 1;
            if (i) host.appendChild(h('span.ph-crumbs__sep', { 'aria-hidden': 'true' }, '›'));
            var el = last || !c.go ? h('span.ph-crumbs__item.is-current', { title: c.label }, c.label)
                : h('button.ph-crumbs__item', { type: 'button', title: c.label, onclick: c.go }, c.label);
            if (i === 0) el.classList.add('is-script');
            host.appendChild(el);
        });
    }
    PH.updateCrumbs = drawCrumbs;

    // --------------------------------------------------------------- rail --

    function renderRail() {
        var rail = document.getElementById('ph-rail');
        var cur = S.cur;
        if (!rail || !cur) return;
        var keepScroll = rail.scrollTop;
        PH.clear(rail);
        var v = PH.scriptView;

        function sw(key, label, title) {
            var b = h('button.ph-railswitch' + (v[key] ? '.is-on' : ''), { type: 'button', role: 'switch', 'aria-checked': v[key] ? 'true' : 'false', title: title },
                [h('span.ph-railswitch__track', h('span.ph-railswitch__knob')), label]);
            b.addEventListener('click', function () { v[key] = !v[key]; PH.renderScriptBody(); });
            return b;
        }
        rail.appendChild(h('div.ph-rail__switches', [
            sw('changedOnly', 'Changed only', 'Only what differs from the default or is not saved yet'),
            sw('advanced', 'Show advanced', 'Settings most servers never need to touch'),
        ]));

        var sec = h('div.ph-rail__sec', [h('div.ph-rail__title', 'Settings')]);
        cur.nav.tabs.forEach(function (t) {
            var st = tabStats(t);
            var items = t.items.filter(itemVisible);
            var secs = sectionsShown(t);
            var showSecs = secs.length > 1 || (secs.length && items.length);
            var hasKids = items.length > 0 || showSecs;
            if (v.changedOnly && !st.changed && v.tab !== t.id) return;
            var open = hasKids && railOpen(t.id);
            var active = !v.q && v.tab === t.id;
            var chev = hasKids ? h('span.ph-rail__chev' + (open ? '.is-open' : ''), { role: 'button', title: open ? 'Collapse' : 'Expand' }, icon('right')) : h('span.ph-rail__chev.is-empty');
            var btn = h('button.ph-rail__tab' + (active && !v.item ? '.is-active' : active ? '.is-within' : ''), { type: 'button', dataset: { tab: t.id }, title: t.label }, [
                chev,
                icon(t.icon || 'sliders'),
                h('span.ph-rail__label', t.label),
                st.changed ? h('span.ph-rail__dot', { title: 'Something here is changed or unsaved' }) : null,
                h('span.ph-rail__count', String(st.n)),
            ]);
            chev.addEventListener('click', function (e) {
                e.stopPropagation();
                setRailOpen(t.id, !open);
                renderRail();
            });
            btn.addEventListener('click', function () { PH.navTo({ tab: t.id }); });
            sec.appendChild(btn);
            if (!open) return;
            var sub = h('div.ph-rail__subs');
            if (showSecs) {
                secs.forEach(function (s) {
                    var n = s.nodes.filter(fieldVisible).length;
                    var dirty = s.nodes.some(function (node) { return S.isPending(node.path) || S.differsFromDefault(node); });
                    sub.appendChild(h('button.ph-rail__sub', { type: 'button', title: s.label || 'General settings', onclick: function () { PH.navTo({ tab: t.id, section: s.id }); } }, [
                        h('span.ph-rail__subico', icon('sliders')),
                        h('span.ph-rail__sublabel', s.label || 'General settings'),
                        dirty ? h('span.ph-rail__dot') : null,
                        h('span.ph-rail__count', String(n)),
                    ]));
                });
            }
            items.forEach(function (it) {
                var ist = itemStats(it);
                var on = !v.q && v.item && PH.canon(v.item) === PH.canon(it.path);
                sub.appendChild(h('button.ph-rail__sub.is-item' + (on ? '.is-active' : ''), { type: 'button', title: it.label + ' (' + kindWord(it.kind) + ')', onclick: function () { PH.navTo({ item: it.path }); } }, [
                    h('span.ph-rail__subico.is-' + it.kind, icon(KIND_ICON[it.kind] || 'list')),
                    h('span.ph-rail__sublabel', it.label),
                    ist.changed ? h('span.ph-rail__dot') : null,
                    h('span.ph-rail__count', String(ist.n)),
                ]));
            });
            sec.appendChild(sub);
        });
        rail.appendChild(sec);

        var refs = h('div.ph-rail__refs', cur.nav.ref.map(function (t) {
            var active = !v.q && v.tab === t.id;
            var n = t.kind === 'commands' ? PH.commandsOf().length : null;
            return h('button.ph-rail__ref' + (active ? '.is-active' : ''), { type: 'button', title: t.label, onclick: function () { PH.navTo({ tab: t.id }); } },
                [icon(t.icon), h('span.ph-rail__reflabel', t.label), n ? h('span.ph-rail__count', String(n)) : null]);
        }));
        rail.appendChild(h('div.ph-rail__sec.ph-rail__sec--ref', [h('div.ph-rail__title', 'Reference'), refs]));
        rail.scrollTop = keepScroll;
    }
    PH.renderRail = renderRail;

    function kindWord(kind) {
        return { list: 'list', map: 'keyed list', collection: 'collection', keytable: 'key table' }[kind] || 'list';
    }

    // ------------------------------------------------------------ content --

    PH.renderScriptBody = function () {
        var content = document.getElementById('ph-content');
        var cur = S.cur;
        if (!content || !cur) return;
        L.closeDrawer();
        var v = PH.scriptView;
        var t = cur.tabById[v.tab];
        // A tab with nothing of its own to show opens its first list (§8.1).
        if (t && t.kind === 'settings' && !v.item && !v.q && !sectionsShown(t).length) {
            var first = t.items.filter(itemVisible)[0] || t.items[0];
            if (first) v.item = first.path;
        }
        renderRail();
        drawCrumbs();
        PH.clear(content);
        if (v.q) { content.appendChild(searchResults(v.q)); return; }
        if (!t) return;
        var view;
        if (t.kind === 'settings') view = v.item && itemAt(v.item) ? itemView(itemAt(v.item)) : tabPage(t);
        else if (t.kind === 'commands') view = PH.commandsTab();
        else if (t.kind === 'help') view = PH.helpTab();
        else view = PH.historyTab();
        view.classList.add('ph-fadein');
        content.appendChild(view);
    };

    function sectionPanel(s, fields) {
        var resettable = s.nodes.filter(function (n) { return S.hasDefault(n) && S.differsFromDefault(n) && fieldVisible(n); });
        var title = s.label || 'General settings';
        var head = h('div.ph-panel__head', [
            h('h3.ph-panel__title', title),
            h('span.ph-panel__count', PH.plural(fields.length, 'setting')),
            h('span.ph-grow'),
            resettable.length && !S.isReadOnly() ? h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', title: 'Put every setting in this group back to its default', onclick: function () {
                PH.confirm({ title: 'Reset “' + title + '”?', ok: 'Reset ' + resettable.length, icon: 'reset',
                    body: resettable.length + ' setting' + (resettable.length === 1 ? '' : 's') + ' go back to the shipped default: ' + resettable.map(function (n) { return F.labelFor(n); }).join(', ') + '. Nothing is written until you save.' })
                    .then(function (yes) {
                        if (!yes) return;
                        resettable.forEach(function (n) { S.queue({ op: 'reset', path: n.path }); });
                        PH.renderScriptBody();
                    });
            } }, [icon('reset'), 'Reset all']) : null,
        ]);
        var el = h('section.ph-panel', [head, h('div.ph-panel__body', fields)]);
        el.setAttribute('data-sec', s.id);
        return el;
    }

    /** A card for a list or collection on its tab's page: what it is, how many, and a way in. */
    function itemCard(it) {
        var st = itemStats(it);
        var meta = F.metaFor(it.node);
        var what = it.kind === 'collection' ? (meta.itemLabel || PH.singular(it.label) || 'entry') : it.kind === 'keytable' ? 'key' : (meta.itemLabel || (it.kind === 'map' ? 'entry' : 'row'));
        var desc = F.tooltipFor(it.node);
        var info = it.kind === 'collection' ? S.cur.coll[it.path] : null;
        return h('button.ph-itemcard', { type: 'button', onclick: function () { PH.navTo({ item: it.path }); } }, [
            h('span.ph-itemcard__ico.is-' + it.kind, icon(KIND_ICON[it.kind] || 'list')),
            h('span.ph-itemcard__main', [
                h('span.ph-itemcard__title', [it.label, st.changed ? h('span.ph-rail__dot', { title: 'Changed or unsaved' }) : null]),
                h('span.ph-itemcard__meta', [PH.plural(st.n, what), info && info.sublists.length ? ' · each with ' + info.sublists.map(function (s) { return s.label.toLowerCase(); }).join(' and ') : '']),
                desc ? h('span.ph-itemcard__desc', desc.split('\n')[0]) : null,
            ]),
            icon('right', 'ph-itemcard__go'),
        ]);
    }

    function tabPage(t) {
        var wrap = h('div.ph-tab');
        var v = PH.scriptView;
        var hiddenAdv = 0, shown = 0;
        var items = t.items.filter(itemVisible);
        var nSettings = 0;
        t.sections.forEach(function (s) { nSettings += s.nodes.filter(fieldVisible).length; });
        wrap.appendChild(h('div.ph-tabhead', [
            h('h2.ph-tabhead__title', [icon(t.icon || 'sliders'), t.label]),
            h('span.ph-tabhead__meta', [nSettings ? PH.plural(nSettings, 'setting') : null, nSettings && items.length ? ' · ' : null, items.length ? PH.plural(items.length, 'list') : null]),
        ]));
        if (items.length) {
            wrap.appendChild(h('div.ph-itemgrid', items.map(itemCard)));
        }
        t.sections.forEach(function (s) {
            var fields = [];
            s.nodes.forEach(function (node) {
                if (!fieldVisible(node)) { if (F.metaFor(node).advanced && !v.advanced) hiddenAdv++; return; }
                fields.push(F.nodeField(node, { onChange: onFieldChange }));
            });
            shown += fields.length;
            if (fields.length) wrap.appendChild(sectionPanel(s, fields));
        });
        var advItems = t.items.filter(function (it) { return F.metaFor(it.node).advanced && !v.advanced; }).length;
        if (!shown && !items.length) {
            wrap.appendChild(h('div.ph-empty', [icon(v.changedOnly ? 'check' : 'sliders', 'ph-empty__ico'),
                h('p.ph-empty__title', v.changedOnly ? 'Nothing on this tab differs from the default.' : 'Nothing to show here.'),
                v.changedOnly ? h('button.ph-btn.ph-btn--ghost.ph-btn--sm', { type: 'button', onclick: function () { v.changedOnly = false; PH.renderScriptBody(); } }, 'Show everything') : null]));
        }
        if (hiddenAdv || advItems) {
            wrap.appendChild(h('button.ph-advnote', { type: 'button', onclick: function () { v.advanced = true; PH.renderScriptBody(); } },
                [icon('cog'), [hiddenAdv ? PH.plural(hiddenAdv, 'advanced setting') : null, hiddenAdv && advItems ? ' and ' : null, advItems ? PH.plural(advItems, 'advanced list') : null].filter(Boolean).join('') + ' hidden. Show them.']));
        }
        return wrap;
    }

    function itemView(it) {
        var v = PH.scriptView;
        var wrap = h('div.ph-tab');
        if (it.kind === 'collection') {
            var info = S.cur.coll[it.path];
            wrap.appendChild(L.collection(info, {
                row: v.row, sub: v.sub,
                onState: function (row, sub) {
                    v.row = row;
                    if (sub !== undefined) v.sub = sub;
                    drawCrumbs();
                },
            }));
        } else if (it.kind === 'keytable') {
            wrap.appendChild(L.keytable(it.node));
        } else {
            wrap.appendChild(L.section(it.node, { label: it.label, onChange: function () { onFieldChange(); } }));
        }
        return wrap;
    }

    var onFieldChange = PH.debounce(function () { renderRail(); }, 200);

    // ------------------------------------------------------------- search --

    function searchResults(q) {
        var cur = S.cur;
        var wrap = h('div.ph-tab.ph-searchres');
        var total = 0;
        var ROW_CAP = 8;

        function hit(ic, trail, title, sub, go) {
            return h('button.ph-listhit', { type: 'button', onclick: go }, [
                icon(ic),
                h('span.ph-listhit__main', [
                    h('span.ph-listhit__trail', trail.join('  ›  ')),
                    h('b', title),
                    sub ? h('span', sub) : null,
                ]),
                icon('right'),
            ]);
        }

        cur.nav.tabs.forEach(function (t) {
            var blocks = [];
            t.sections.forEach(function (s) {
                var fields = [];
                s.nodes.forEach(function (node) {
                    if (PH.scriptView.changedOnly && !fieldVisible(node)) return;
                    if (PH.matches(q, [F.labelFor(node), node.path, F.tooltipFor(node), PH.fmtValue(S.get(node.path), 200), s.label])) {
                        fields.push(F.nodeField(node, { onChange: onFieldChange }));
                    }
                });
                if (!fields.length) return;
                total += fields.length;
                blocks.push(h('div.ph-searchres__group', [
                    h('div.ph-searchres__trail', [icon(t.icon || 'sliders'), t.label, s.label ? h('span.ph-crumbs__sep', '›') : null, s.label || null]),
                    h('section.ph-panel', h('div.ph-panel__body', fields)),
                ]));
            });
            var hits = [];
            t.items.forEach(function (it) {
                var base = [t.label];
                var nameHit = PH.matches(q, [it.label, it.path, F.tooltipFor(it.node)]);
                var st = itemStats(it);
                if (it.kind === 'collection') {
                    var info = cur.coll[it.path];
                    var n = 0;
                    info.rowKeys().forEach(function (k) {
                        if (n >= ROW_CAP * 2) return;
                        var title = info.rowTitle(k);
                        if (PH.matches(q, [String(k), title])) {
                            n++;
                            hits.push(hit('grid', base.concat(it.label), title, title !== String(k) ? String(k) : '', function () { PH.navTo({ item: it.path, row: k }); }));
                        }
                        info.sublists.forEach(function (s) {
                            var sn = L.subNodeOf(info, k, s);
                            L.rowsOf(sn, F.metaFor(sn)).forEach(function (r) {
                                if (n >= ROW_CAP * 2) return;
                                if (!PH.matches(q, [String(r.key), JSON.stringify(r.value)])) return;
                                n++;
                                hits.push(hit('list', base.concat(it.label, title, s.label), L.titleOf(sn, r), '', function () { PH.navTo({ item: it.path, row: k, sub: s.field, entry: r.key }); }));
                            });
                        });
                    });
                    if (nameHit && !n) hits.push(hit('grid', base, it.label, PH.plural(st.n, 'entry', 'entries'), function () { PH.navTo({ item: it.path }); }));
                    return;
                }
                var meta = Object.assign({}, F.metaFor(it.node), it.kind === 'keytable' ? { kind: 'map' } : {});
                var rows = L.rowsOf(it.node, meta).filter(function (r) { return PH.matches(q, [String(r.key), JSON.stringify(r.value)]); });
                if (nameHit || rows.length) {
                    if (nameHit || rows.length > ROW_CAP) {
                        hits.push(hit(KIND_ICON[it.kind] || 'list', base, it.label, rows.length ? PH.plural(rows.length, 'row') + ' match' : PH.plural(st.n, 'row'), function () {
                            PH.navTo({ item: it.path });
                            if (rows.length) setTimeout(function () { var le = document.querySelector('.ph-content .ph-list'); if (le && le.search) le.search(q); }, 60);
                        }));
                    }
                    rows.slice(0, ROW_CAP).forEach(function (r) {
                        hits.push(hit('right', base.concat(it.label), L.titleOf(it.node, r), it.kind === 'map' || it.kind === 'keytable' ? String(r.key) : '', function () { PH.navTo({ item: it.path, entry: r.key }); }));
                    });
                }
            });
            if (hits.length) {
                total += hits.length;
                blocks.push(h('div.ph-searchres__group', [
                    h('div.ph-searchres__trail', [icon(t.icon || 'sliders'), t.label, h('span.ph-crumbs__sep', '›'), 'Lists']),
                    h('section.ph-panel', h('div.ph-panel__body', hits)),
                ]));
            }
            blocks.forEach(function (b) { wrap.appendChild(b); });
        });
        var cmdHits = PH.commandsOf().filter(function (c) { return PH.matches(q, [c.command, c.description, c.usage, c.who]); });
        if (cmdHits.length) {
            total += cmdHits.length;
            wrap.appendChild(h('div.ph-searchres__group', [
                h('div.ph-searchres__trail', [icon('terminal'), 'Commands']),
                h('section.ph-panel', h('div.ph-panel__body', cmdHits.map(function (c) {
                    return hit('terminal', ['Commands'], liveCommand(c), c.description || '', function () { PH.navTo({ tab: 'commands' }); });
                }))),
            ]));
        }
        if (!total) {
            wrap.appendChild(h('div.ph-empty', [icon('search', 'ph-empty__ico'), h('p.ph-empty__title', 'Nothing in this script matches “' + q + '”.'),
                h('p.ph-empty__text', 'Search looks at setting names, descriptions, values and every row of every list. Try another word, or search every script with Ctrl+K.')]));
        } else {
            wrap.insertBefore(h('p.ph-searchres__sum', PH.plural(total, 'match', 'matches') + ' for “' + q + '”. Settings can be edited right here; lists open where the match is.'), wrap.firstChild);
        }
        return wrap;
    }

    // ----------------------------------------------------------- commands --

    function liveCommand(c) {
        var name = c.command || '';
        // The server sends the live name in `command` and the hub.json one in
        // `declared`; a pending edit of the setting shows straight away.
        if (c.configPath && S.cur) {
            var v = S.get(c.configPath);
            if (typeof v === 'string' && v) {
                var live = v.charAt(0) === '/' ? v : '/' + v;
                var rest = name.replace(/^\/?\S+/, '');
                return live + rest;
            }
        }
        return name;
    }
    PH.liveCommand = liveCommand;

    PH.commandsTab = function () {
        var wrap = h('div.ph-tab.ph-cmds');
        var list = PH.commandsOf();
        wrap.appendChild(h('div.ph-sechead', [icon('terminal'), h('h2', 'Commands'), h('span.ph-sechead__count', String(list.length))]));
        list.forEach(function (c) {
            var name = liveCommand(c);
            var shipped = c.declared || c.command;
            var changedName = c.configPath && name.split(' ')[0] !== String(shipped).split(' ')[0];
            function copyBtn(text) {
                return h('button.ph-iconbtn.ph-iconbtn--sm', { type: 'button', title: 'Copy', onclick: function () { PH.copyText(text); PH.toast({ kind: 'info', title: 'Copied', text: text }); } }, icon('copy'));
            }
            var who = String(c.who || 'Everyone');
            var whoKind = /^admin/i.test(who) ? 'admin' : /^console/i.test(who) ? 'console' : /^job/i.test(who) ? 'job' : /^ace/i.test(who) ? 'ace' : 'all';
            wrap.appendChild(h('article.ph-cmd', [
                h('div.ph-cmd__head', [
                    h('code.ph-cmd__name', name), copyBtn(name.split(' ')[0]),
                    h('span.ph-grow'),
                    h('span.ph-who.ph-who--' + whoKind, [icon(whoKind === 'admin' ? 'shield' : whoKind === 'console' ? 'terminal' : whoKind === 'job' ? 'badge' : whoKind === 'ace' ? 'lock' : 'users'), who]),
                ]),
                c.description ? h('p.ph-cmd__desc', c.description) : null,
                changedName ? h('p.ph-cmd__live', [icon('link'), 'The name comes from ', h('code', c.configPath), ' (shipped as ', h('code', String(shipped).split(' ')[0]), ').',
                    h('button.ph-linkbtn', { type: 'button', onclick: function () { PH.reveal(c.configPath); } }, 'Change it')]) :
                    c.configPath ? h('p.ph-cmd__live', [icon('link'), 'The name is set by ', h('code', c.configPath), '.', h('button.ph-linkbtn', { type: 'button', onclick: function () { PH.reveal(c.configPath); } }, 'Change it')]) : null,
                c.usage ? h('div.ph-cmd__row', [h('span.ph-cmd__label', 'Usage'), h('code', c.usage), copyBtn(c.usage)]) : null,
                (c.examples || []).length ? h('div.ph-cmd__row', [h('span.ph-cmd__label', 'Examples'), h('div.ph-cmd__examples', c.examples.map(function (ex) {
                    return h('span.ph-cmd__ex', [h('code', ex), copyBtn(ex)]);
                }))]) : null,
            ]));
        });
        return wrap;
    };

    // --------------------------------------------------------------- help --

    PH.helpTab = function () {
        var d = S.cur.data;
        var cur = S.cur;
        var pages = [];
        if (d.readme) pages.push({ title: 'Read me', markdown: d.readme });
        (d.help || []).forEach(function (p) { if (p && p.markdown) pages.push(p); });
        var v = PH.scriptView;
        if (v.helpPage >= pages.length) v.helpPage = 0;
        var wrap = h('div.ph-tab.ph-help');
        if (pages.length > 1) {
            wrap.appendChild(h('div.ph-helpnav', pages.map(function (p, i) {
                return h('button.ph-helpnav__btn' + (i === v.helpPage ? '.is-on' : ''), { type: 'button', onclick: function () { v.helpPage = i; PH.renderScriptBody(); } },
                    [icon(i === 0 && d.readme ? 'book' : 'file'), p.title || 'Page ' + (i + 1)]);
            })));
        }
        var page = pages[v.helpPage];
        if (!page) { wrap.appendChild(h('div.ph-empty', [icon('book', 'ph-empty__ico'), h('p', 'This script has no help pages.')])); return wrap; }
        var folder = (cur.card && cur.card.folder) || (PH.cardById(cur.id) || {}).folder;
        var res = PH.md.render(page.markdown, { folder: folder });
        var doc = h('article.ph-doc', { html: res.html });
        PH.md.bind(doc, PH.els.main);
        var toc = res.headings.filter(function (x) { return x.level === 2 || x.level === 3; });
        var layout = h('div.ph-doclayout', [doc]);
        if (toc.length > 2) {
            layout.appendChild(h('nav.ph-toc', [h('div.ph-toc__title', 'On this page')].concat(toc.map(function (x) {
                return h('button.ph-toc__item.lvl' + x.level, { type: 'button', onclick: function () {
                    var target = doc.querySelector('#md-' + x.id);
                    if (target) PH.md.scrollToEl(target, PH.els.main);
                } }, x.text);
            }))));
        }
        wrap.appendChild(layout);
        return wrap;
    };

    // ------------------------------------------------------------ history --

    function decodeLogged(v) {
        if (typeof v !== 'string') return v;
        var t = v.trim();
        if (/^[\[{"]/.test(t) || /^(true|false|null|-?\d+(\.\d+)?)$/.test(t)) {
            try { return JSON.parse(t); } catch (e) { return v; }
        }
        return v;
    }

    PH.historyTab = function () {
        var v = PH.scriptView;
        var wrap = h('div.ph-tab.ph-history');
        wrap.appendChild(h('div.ph-sechead', [icon('history'), h('h2', 'Change history'), h('span.ph-sechead__note', 'Every change saved from the hub or the console, newest first.')]));
        var body = h('div.ph-history__body');
        wrap.appendChild(body);
        if (!v.history || v.history.id !== S.cur.id) {
            v.history = { id: S.cur.id, rows: [], page: 0, more: true, loading: false };
            loadMore();
        }
        draw();

        function loadMore() {
            var hs = v.history;
            if (hs.loading || !hs.more) return;
            hs.loading = true;
            draw();
            PH.api('history', S.cur.id, hs.page + 1).then(function (r) {
                if (v.history !== hs) return;
                hs.loading = false;
                if (!r.ok) { hs.more = false; hs.error = PH.errMsg(r); draw(); return; }
                hs.page += 1;
                if (r.value && r.value.unavailable) hs.error = 'History needs the database (oxmysql). Changes are still saved to the files.';
                hs.rows = hs.rows.concat((r.value && r.value.rows) || []);
                hs.more = !!(r.value && r.value.more);
                draw();
            });
        }

        function draw() {
            if (!body.isConnected && body.parentNode === null && !wrap.parentNode) { /* first draw happens before attach */ }
            PH.clear(body);
            var hs = v.history;
            if (!hs.rows.length && hs.loading) { body.appendChild(skeleton('rows')); return; }
            if (hs.error) body.appendChild(h('div.ph-banner.ph-banner--warn', [icon('alert', 'ph-banner__ico'), h('div.ph-banner__text', hs.error)]));
            if (!hs.rows.length && !hs.loading) {
                body.appendChild(h('div.ph-empty', [icon('history', 'ph-empty__ico'), h('p', 'No changes have been saved from the hub yet.')]));
                return;
            }
            var pendingBlock = S.pendingCount() > 0;
            var table = h('table.ph-table.ph-table--history');
            table.appendChild(h('thead', h('tr', [h('th', 'When'), h('th', 'Who'), h('th', 'Setting'), h('th', 'Change'), h('th.ph-table__acts', '')])));
            var tb = h('tbody');
            hs.rows.forEach(function (row) {
                var when = PH.when(row.at);
                var node = S.resolve(row.path);
                var label = node && node.node.path === row.path ? F.labelFor(node.node) : row.path;
                var oldV = decodeLogged(row.old), newV = decodeLogged(row['new']);
                var change;
                if (row.op === 'set' || row.op === 'reset' || !row.op) {
                    change = h('span.ph-diffline', [h('span.ph-diff__old', PH.fmtValue(oldV, 60)), icon('right'), h('span.ph-diff__new', PH.fmtValue(newV, 60))]);
                } else {
                    change = h('span.ph-diffline', [h('span.ph-opbadge', opLabel(row.op)), h('span.ph-diff__new', PH.fmtValue(newV !== undefined && newV !== null ? newV : oldV, 60))]);
                }
                // A row that added something has no earlier value to go back to.
                var noOld = row.old === undefined || row.old === null;
                var undo = noOld ? null : h('button.ph-btn.ph-btn--ghost.ph-btn--sm', {
                    type: 'button', disabled: S.isReadOnly() || pendingBlock,
                    title: S.isReadOnly() ? 'You need the edit lock to undo' : pendingBlock ? 'Save or discard your unsaved changes first' : 'Put the old value back (saved at once)',
                }, [icon('reset'), 'Undo']);
                if (undo) undo.addEventListener('click', function () { PH.undoLog(row, label); });
                tb.appendChild(h('tr', [
                    h('td', [h('div', when.rel), h('div.ph-table__sub', when.abs)]),
                    h('td', [icon('user'), ' ', row.by || 'console']),
                    h('td', [h('button.ph-linkbtn', { type: 'button', onclick: function () { PH.reveal(row.path); } }, label),
                        h('div.ph-table__sub.ph-cellmono', (row.file ? row.file + '  ·  ' : '') + row.path)]),
                    h('td', change),
                    h('td.ph-table__acts', undo),
                ]));
            });
            table.appendChild(tb);
            body.appendChild(h('div.ph-tablewrap', table));
            if (hs.more) {
                body.appendChild(h('div.ph-center', h('button.ph-btn.ph-btn--ghost', { type: 'button', disabled: hs.loading, onclick: loadMore },
                    hs.loading ? 'Loading…' : 'Load older changes')));
            }
        }
        PH.refreshHistory = function () { if (v.history) { v.history = null; } };
        return wrap;
    };

    function opLabel(op) {
        return { insert: 'Added', remove: 'Removed', move: 'Moved', renameKey: 'Renamed', duplicate: 'Duplicated', linkRole: 'Linked to role', unlinkRole: 'Unlinked', reset: 'Reset' }[op] || op;
    }
    PH.opLabel = opLabel;

    // --------------------------------------------------------------- render --

    PH.render = function () {
        L.closeDrawer();
        PH.hideTip();
        PH.els.root.dataset.view = PH.view;
        if (PH.view === 'home') renderHome();
        else if (PH.view === 'script') renderScript();
        else if (PH.view === 'roles') PH.renderRoles();
        PH.updateSaveBar();
        PH.els.main.scrollTop = 0;
    };
})();
