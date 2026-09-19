/*
    Poggy Hub — the Roles page and the command palette.

    Roles are poggy_core's master job and group lists (PoggyCoreConfig.Roles).
    A job or group list in any script can follow a role; saving a role
    rewrites every list that follows it and restarts those scripts, so the
    confirm step names every script affected before anything is written.

    The palette (Ctrl+K or /) searches everything the hub knows without
    loading a script: scripts, every setting and list (the `index` sent with
    `open`), every command, and a few actions.
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var h = PH.h, icon = PH.icon, S = PH.S, F = PH.F;

    // ---------------------------------------------------------------- roles --

    var R = PH.rolesView = { data: null, edit: null, selected: null, loading: false, q: '', returnTo: null };

    PH.go.roles = function (name) {
        var from = PH.view === 'script' && S.cur ? S.cur.id : null;
        return PH.leaveScript().then(function (ok) {
            if (!ok) return;
            PH.view = 'roles';
            if (from) R.returnTo = from;
            if (name) R.selected = name;
            R.q = '';
            if (!R.data || !R.edit || !PH.rolesDirty()) load();
            else PH.render();
        });
    };

    function load() {
        R.loading = true;
        PH.render();
        F.loadRoles(true).then(function (v) {
            R.loading = false;
            R.data = PH.clone(v);
            R.edit = PH.clone(v.roles || {});
            var names = Object.keys(R.edit);
            if (!R.selected || !R.edit[R.selected]) R.selected = names[0] || null;
            if (PH.view === 'roles') PH.render();
        });
    }

    PH.rolesDirty = function () {
        return !!(R.data && R.edit && !PH.deepEqual(R.edit, R.data.roles || {}));
    };

    function changedRoles() {
        var before = (R.data && R.data.roles) || {}, after = R.edit || {};
        var names = Object.keys(before).concat(Object.keys(after)).filter(function (n, i, a) { return a.indexOf(n) === i; });
        return names.filter(function (n) { return !PH.deepEqual(before[n], after[n]); });
    }

    PH.renderRoles = function () {
        var back = R.returnTo;
        var backCard = back ? PH.cardById(back) : null;
        var top = PH.els.top;
        PH.clear(top);
        var search = h('input.ph-top__search', { type: 'text', placeholder: 'Search roles, jobs and groups…', spellcheck: 'false', value: R.q });
        search.addEventListener('input', PH.debounce(function () { R.q = search.value.trim(); drawList(); }, 120));
        top.appendChild(h('div.ph-top__left', [
            h('button.ph-btn.ph-btn--ghost.ph-top__back', { type: 'button', onclick: function () {
                leaveRoles().then(function (ok) {
                    if (!ok) return;
                    if (back) { R.returnTo = null; PH.go.script(back); } else { PH.view = 'home'; PH.render(); }
                });
            } }, [icon('back'), backCard ? backCard.label : 'All scripts']),
            h('span.ph-top__crumb', 'Roles'),
        ]));
        top.appendChild(h('div.ph-top__center', h('label.ph-searchbox', [icon('search'), search,
            h('button.ph-kbd', { type: 'button', onclick: function () { PH.palette.open(); } }, 'Ctrl K')])));
        top.appendChild(h('div.ph-top__right', [
            h('button.ph-iconbtn.ph-iconbtn--lg', { type: 'button', title: 'Close (Esc)', onclick: function () { PH.requestClose(); } }, icon('x')),
        ]));
        PH.els.search = search;

        var main = PH.els.main;
        PH.clear(main);
        var page = h('div.ph-roles.ph-page');
        page.appendChild(h('section.ph-masthead.ph-masthead--small', [
            h('p.ph-eyebrow', 'poggy_core'),
            h('h1.ph-masthead__title', 'Roles'),
            h('p.ph-masthead__lede', 'Master job and group lists. Link a job or group list in any script’s settings to a role; saving the role rewrites every linked list and restarts the scripts that use it.'),
        ]));
        if (R.loading || !R.edit) {
            page.appendChild(PH.skeleton('rows'));
            main.appendChild(page);
            return;
        }
        var listEl = h('div.ph-roles__list');
        var editorEl = h('div.ph-roles__editor');
        page.appendChild(h('div.ph-roles__layout', [h('div.ph-roles__col', [listEl,
            h('button.ph-btn.ph-btn--ghost.ph-roles__new', { type: 'button', onclick: newRole }, [icon('plus'), 'New role'])]), editorEl]));
        main.appendChild(page);

        function drawList() {
            PH.clear(listEl);
            var names = Object.keys(R.edit).sort(function (a, b) { return String(R.edit[a].label || a).localeCompare(String(R.edit[b].label || b)); });
            var shown = names.filter(function (n) { var r = R.edit[n]; return PH.matches(R.q, [n, r.label, (r.list || []).join(' ')]); });
            if (!shown.length) listEl.appendChild(h('div.ph-empty', [icon('users', 'ph-empty__ico'), h('p', names.length ? 'No role matches.' : 'No roles yet.')]));
            shown.forEach(function (n) {
                var r = R.edit[n];
                var usage = (R.data.usage && R.data.usage[n]) || [];
                var dirty = !PH.deepEqual(r, (R.data.roles || {})[n]);
                listEl.appendChild(h('button.ph-rolecard' + (R.selected === n ? '.is-on' : ''), { type: 'button', onclick: function () { R.selected = n; drawList(); drawEditor(); } }, [
                    h('span.ph-rolecard__ico', icon(r.kind === 'groups' ? 'shield' : 'badge')),
                    h('span.ph-rolecard__main', [
                        h('span.ph-rolecard__label', [r.label || n, dirty ? h('span.ph-badge.ph-badge--pending', 'Unsaved') : null]),
                        h('span.ph-rolecard__sub', [h('code', n), ' · ', r.kind === 'groups' ? 'admin groups' : 'jobs', ' · ', PH.plural((r.list || []).length, 'name')]),
                    ]),
                    h('span.ph-rolecard__use', { title: 'Settings that follow this role' }, [icon('link'), String(usage.length)]),
                ]));
            });
        }

        function drawEditor() {
            PH.clear(editorEl);
            var n = R.selected;
            var r = n && R.edit[n];
            if (!r) { editorEl.appendChild(h('div.ph-empty', [icon('users', 'ph-empty__ico'), h('p', 'Pick a role, or make a new one.')])); return; }
            var usage = (R.data.usage && R.data.usage[n]) || [];

            var labelIn = h('input.ph-input', { type: 'text', value: r.label || '', spellcheck: 'false' });
            labelIn.addEventListener('input', function () { r.label = labelIn.value; drawListSoon(); PH.updateSaveBar(); });

            function kindBtn(k, label) {
                var b = h('button.ph-seg__btn' + ((r.kind || 'jobs') === k ? '.is-on' : ''), { type: 'button' }, label);
                b.addEventListener('click', function () { r.kind = k; drawEditor(); drawList(); PH.updateSaveBar(); });
                return b;
            }

            var chips = h('div.ph-chips');
            function drawChips() {
                PH.clear(chips);
                var listEl2 = h('div.ph-chips__list');
                (r.list || []).forEach(function (name, i) {
                    listEl2.appendChild(h('span.ph-chip', [h('span.ph-chip__text', name), h('button.ph-chip__x', { type: 'button', title: 'Remove', onclick: function () {
                        r.list.splice(i, 1); drawChips(); drawList(); PH.updateSaveBar();
                    } }, icon('x'))]));
                });
                var add = h('button.ph-chip.ph-chip--add', { type: 'button' }, [icon('plus'), r.kind === 'groups' ? 'Add group' : 'Add job']);
                add.addEventListener('click', function () {
                    F.pick({ anchor: add, fetch: F.fetchers[r.kind === 'groups' ? 'group' : 'job'], allowFree: true, freeLabel: 'Add',
                        placeholder: r.kind === 'groups' ? 'Search admin groups…' : 'Search jobs…',
                        onPick: function (v) {
                            r.list = r.list || [];
                            if (r.list.indexOf(v) === -1) r.list.push(v);
                            drawChips(); drawList(); PH.updateSaveBar();
                        } });
                });
                listEl2.appendChild(add);
                chips.appendChild(listEl2);
            }
            drawChips();

            var usageEl = h('div.ph-usage');
            if (!usage.length) usageEl.appendChild(h('p.ph-muted', 'No setting follows this role yet. Open a script’s job or group list and choose “Link to role”.'));
            usage.forEach(function (u) {
                var c = PH.cardById(u.id);
                usageEl.appendChild(h('button.ph-usage__row', { type: 'button', title: 'Open this setting', onclick: function () {
                    leaveRoles().then(function (ok) { if (ok) PH.go.script(u.id, { reveal: u.path }); });
                } }, [
                    c ? h('span.ph-usage__art', PH.art(c)) : icon('file'),
                    h('span.ph-usage__main', [h('b', c ? c.label : u.id), h('code', u.path)]),
                    c && c.lock && c.lock.holder ? h('span.ph-lockpill', [icon('lock'), c.lock.holder.name]) : null,
                    icon('right'),
                ]));
            });

            editorEl.appendChild(h('section.ph-panel.ph-roleedit', [
                h('div.ph-panel__head', [h('h3.ph-panel__title', r.label || n), h('code.ph-roleedit__key', n), h('span.ph-grow'),
                    h('button.ph-btn.ph-btn--ghost.ph-btn--sm.ph-btn--danger-text', { type: 'button', onclick: function () { deleteRole(n, usage); } }, [icon('trash'), 'Delete role'])]),
                h('div.ph-panel__body', [
                    formRow('Label', 'What the hub shows for this role.', labelIn),
                    formRow('Kind', 'Jobs are character jobs; admin groups are framework permission groups.', h('div.ph-seg', [kindBtn('jobs', 'Jobs'), kindBtn('groups', 'Admin groups')])),
                    formRow(r.kind === 'groups' ? 'Groups' : 'Jobs', 'Exactly as your framework spells them.', chips),
                    formRow('Used by', 'Settings that follow this role. Saving rewrites them and restarts their scripts.', usageEl),
                ]),
            ]));
        }

        var drawListSoon = PH.debounce(drawList, 150);

        function newRole() {
            PH.prompt({
                title: 'New role', icon: 'users', ok: 'Create',
                text: 'A short key for the role, used in the config marker (-- poggy:role <key>). Letters, digits and _ only.',
                validate: function (v) {
                    v = v.trim();
                    if (!/^[A-Za-z_][A-Za-z0-9_]*$/.test(v)) return 'Letters, digits and _ only, not starting with a digit.';
                    if (R.edit[v]) return 'That key is already a role.';
                    return null;
                },
            }).then(function (key) {
                if (!key) return;
                key = key.trim();
                R.edit[key] = { label: PH.readable(key), kind: 'jobs', list: [] };
                R.selected = key;
                drawList(); drawEditor(); PH.updateSaveBar();
            });
        }

        function deleteRole(n, usage) {
            PH.confirm({
                title: 'Delete the role “' + (R.edit[n].label || n) + '”?', danger: true, ok: 'Delete role', okIcon: 'trash',
                body: usage.length
                    ? 'Settings that follow it (' + usage.length + ') keep their current names but stop following any role. Nothing is written until you save.'
                    : 'Nothing follows this role. Nothing is written until you save.',
            }).then(function (yes) {
                if (!yes) return;
                delete R.edit[n];
                R.selected = Object.keys(R.edit)[0] || null;
                drawList(); drawEditor(); PH.updateSaveBar();
            });
        }

        drawList();
        drawEditor();
    };

    function formRow(label, hint, control) {
        return h('div.ph-field.ph-field--form', [
            h('div.ph-field__info', [h('div.ph-field__top', h('label.ph-field__label', label)), hint ? h('div.ph-field__desc', hint) : null]),
            h('div.ph-field__ctl', control),
            h('div.ph-field__side'),
        ]);
    }

    function leaveRoles() {
        if (PH.view !== 'roles' || !PH.rolesDirty()) return Promise.resolve(true);
        return PH.modal({
            title: 'Unsaved role changes', icon: 'alert',
            body: 'You changed ' + PH.plural(changedRoles().length, 'role') + ' without saving.',
            actions: [
                { id: 'stay', label: 'Keep editing', kind: 'ghost' },
                { id: 'discard', label: 'Discard', kind: 'danger' },
                { id: 'save', label: 'Save roles', kind: 'primary' },
            ],
        }).then(function (id) {
            if (id === 'discard') { R.edit = PH.clone(R.data.roles || {}); return true; }
            if (id === 'save') return PH.saveRoles();
            return false;
        });
    }
    PH.leaveRoles = leaveRoles;

    PH.discardRoles = function () {
        if (!R.data) return;
        R.edit = PH.clone(R.data.roles || {});
        PH.render();
    };

    /** Confirm (listing the scripts affected), save, then report. Resolves true when saved. */
    PH.saveRoles = function () {
        var names = changedRoles();
        if (!names.length) return Promise.resolve(true);
        var usage = (R.data && R.data.usage) || {};
        var affected = [];
        names.forEach(function (n) { (usage[n] || []).forEach(function (u) { affected.push({ role: n, id: u.id, path: u.path }); }); });
        var byScript = {};
        affected.forEach(function (a) { (byScript[a.id] = byScript[a.id] || []).push(a); });
        var ids = Object.keys(byScript);

        var body = h('div.ph-confirmlist', [
            h('p', ids.length
                ? 'Saving rewrites these settings in their config files and restarts ' + PH.plural(ids.length, 'script') + '. Scripts someone else is editing are skipped and reported.'
                : 'No setting follows the changed roles, so nothing else is rewritten or restarted.'),
            ids.length ? h('ul.ph-affected', ids.map(function (id) {
                var c = PH.cardById(id);
                var busy = c && c.lock && c.lock.holder;
                return h('li', [h('b', c ? c.label : id), busy ? h('span.ph-lockpill', [icon('lock'), busy.name + ' is editing: will be skipped']) : null,
                    h('div.ph-affected__paths', byScript[id].map(function (a) { return h('code', a.path); }))]);
            })) : null,
            h('p.ph-muted', 'Changed: ' + names.map(function (n) { return (R.edit[n] && R.edit[n].label) || n + ' (deleted)'; }).join(', ') + '.'),
        ]);

        return PH.modal({
            title: 'Save ' + PH.plural(names.length, 'role') + '?', icon: 'users', wide: true, body: body,
            actions: [
                { id: 'no', label: 'Cancel', kind: 'ghost' },
                { id: 'yes', label: ids.length ? 'Save and restart ' + PH.plural(ids.length, 'script') : 'Save roles', kind: 'primary', icon: 'save' },
            ],
        }).then(function (id) {
            if (id !== 'yes') return false;
            PH.busy(true, 'Saving roles…');
            return PH.api('saveRoles', R.edit).then(function (r) {
                PH.busy(false);
                if (!r.ok) {
                    PH.toast({ kind: 'error', title: 'Roles were not saved', text: PH.errMsg(r) });
                    return false;
                }
                report(r.value || {});
                F.cache.roles = null;
                load();
                return true;
            });
        });
    };

    function report(v) {
        var rewritten = v.rewritten || [], skipped = v.skipped || [], restarted = v.restarted || [];
        function label(id) { var c = PH.cardById(id); return c ? c.label : id; }
        PH.modal({
            title: 'Roles saved', icon: 'check',
            body: h('div.ph-report', [
                h('div.ph-report__row', [icon('pencil'), h('b', PH.plural(rewritten.length, 'setting')), ' rewritten']),
                rewritten.length ? h('ul.ph-report__list', rewritten.map(function (x) { return h('li', [label(x.id), ' ', h('code', x.path)]); })) : null,
                h('div.ph-report__row', [icon('restart'), h('b', PH.plural(restarted.length, 'script')), ' restarted', restarted.length ? ': ' + restarted.map(label).join(', ') : '']),
                skipped.length ? h('div.ph-report__row.is-warn', [icon('alert'), h('b', PH.plural(skipped.length, 'script')), ' skipped:']) : null,
                skipped.length ? h('ul.ph-report__list', skipped.map(function (x) {
                    var holder = x.holder && (x.holder.name || x.holder);
                    return h('li', [label(x.id), holder ? ' (' + holder + ' is editing it)' : x.reason ? ' (' + x.reason + ')' : '']);
                })) : null,
                skipped.length ? h('p.ph-muted', 'Save the roles again once they are free (or started), and those scripts will follow.') : null,
            ]),
            actions: [{ id: 'ok', label: 'Done', kind: 'primary' }],
        });
    }

    // -------------------------------------------------------------- palette --

    var P = PH.palette = { el: null };

    var KIND_ICON = { panel: 'database', script: 'box', command: 'terminal', list: 'list', map: 'rows', collection: 'grid', keytable: 'keyboard', strings: 'tag', group: 'sliders', value: 'sliders', readonly: 'file', action: 'bolt', role: 'users' };
    var KIND_LABEL = { panel: 'Data panel', script: 'Script', command: 'Command', list: 'List', map: 'List', collection: 'Collection', keytable: 'Key table', strings: 'Setting', group: 'Group', value: 'Setting', readonly: 'Setting', action: 'Action', role: 'Role' };

    /** Where an index entry lives: "Markets › Stores". The index names tabs by id only. */
    function trailOf(e, card) {
        var parts = [card ? card.label : e.id];
        if (e.tabLabel) parts.push(e.tabLabel);
        else if (e.tab && e.tab !== 'general' && /^[a-z0-9_\-]+$/i.test(e.tab)) parts.push(PH.readable(e.tab));
        return parts.join('  ›  ');
    }

    function actions() {
        var out = [
            { kind: 'action', label: 'All scripts', sub: 'Go to the home page', icon: 'home', run: function () { PH.go.home(); } },
            { kind: 'action', label: 'Roles', sub: 'Master job and group lists', icon: 'users', run: function () { PH.go.roles(); } },
        ];
        if (PH.view === 'script' && S.cur) {
            var c = PH.cardById(S.cur.id) || {};
            if (S.pendingCount()) {
                out.push({ kind: 'action', label: 'Save changes', sub: PH.plural(S.pendingCount(), 'unsaved change') + ' · Ctrl+S', icon: 'save', run: function () { PH.save(false); } });
                out.push({ kind: 'action', label: 'Review changes', sub: 'See every pending change', icon: 'eye', run: function () { PH.review(); } });
            }
            if (c.running && !PH.isCore(c.id)) out.push({ kind: 'action', label: 'Restart ' + (c.label || c.id), sub: 'Apply saved changes', icon: 'restart', run: function () { PH.restartScript(c.id); } });
            if (!c.running) out.push({ kind: 'action', label: 'Start ' + (c.label || c.id), icon: 'play', run: function () { PH.startScript(c.id); } });
            out.push({ kind: 'action', label: PH.scriptView.changedOnly ? 'Show every setting' : 'Show changed settings only', icon: 'filter', run: function () { PH.scriptView.changedOnly = !PH.scriptView.changedOnly; PH.renderScriptBody(); } });
            out.push({ kind: 'action', label: 'History of ' + (c.label || c.id), icon: 'history', run: function () { PH.go.script(c.id, { tab: 'history' }); } });
        }
        out.push({ kind: 'action', label: 'Close the hub', sub: 'Esc', icon: 'x', run: function () { PH.requestClose(); } });
        return out;
    }

    function entries(opts) {
        var list = [];
        if (!opts || !opts.noScripts) {
            (PH.boot && PH.boot.scripts || []).forEach(function (c) {
                list.push({ kind: 'script', id: c.id, label: c.label || c.id, sub: c.tagline || c.folder, card: c });
            });
        }
        (PH.boot && PH.boot.index || []).forEach(function (e) {
            if (!e) return;
            if (e.kind === 'command') list.push({ kind: 'command', id: e.id, label: e.command, sub: e.description || '', command: e.command });
            else if (e.kind === 'panel') list.push({ kind: 'panel', id: e.id, path: e.path || ('panel:' + e.panel), panel: e.panel, label: e.label || PH.readable(String(e.panel)), sub: e.tooltip || 'Live data: changes apply at once', tab: e.tab });
            else list.push({ kind: e.kind || 'value', id: e.id, path: e.path, label: e.label || PH.readable(String(e.path).split('.').pop()), sub: e.tooltip || '', tab: e.tab, tabLabel: e.tabLabel });
        });
        if (!opts || !opts.noActions) actions().forEach(function (a) { list.push(a); });
        return list;
    }

    function score(e, words) {
        var label = String(e.label || '').toLowerCase();
        var card = PH.cardById(e.id);
        var scriptLabel = card ? String(card.label).toLowerCase() : String(e.id || '').toLowerCase();
        var path = String(e.path || e.command || '').toLowerCase();
        var sub = String(e.sub || '').toLowerCase();
        var total = 0;
        for (var i = 0; i < words.length; i++) {
            var w = words[i];
            var s = 0;
            if (label === w) s = 120;
            else if (label.indexOf(w) === 0) s = 90;
            else if (new RegExp('\\b' + w.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')).test(label)) s = 70;
            else if (label.indexOf(w) !== -1) s = 50;
            else if (path.indexOf(w) !== -1) s = 35;
            else if (scriptLabel.indexOf(w) !== -1) s = 25;
            else if (sub.indexOf(w) !== -1) s = 12;
            if (!s) return 0;
            total += s;
        }
        if (e.kind === 'script') total += 15;
        if (e.kind === 'action') total += 5;
        return total;
    }

    P.search = function (q, opts) {
        var words = String(q || '').toLowerCase().split(/\s+/).filter(Boolean);
        var list = entries(opts);
        if (!words.length) return list.filter(function (e) { return e.kind === 'action' || e.kind === 'script'; });
        return list.map(function (e) { return { e: e, s: score(e, words) }; })
            .filter(function (x) { return x.s > 0; })
            .sort(function (a, b) { return b.s - a.s; })
            .slice(0, 80)
            .map(function (x) { return x.e; });
    };

    P.row = function (e, onClick) {
        var card = e.kind !== 'action' ? PH.cardById(e.id) : null;
        var sub = e.kind === 'script' ? e.sub
            : e.kind === 'action' ? (e.sub || '')
            : trailOf(e, card) + (e.sub ? '  —  ' + e.sub : '');
        var lead = e.kind === 'script' && e.card ? h('span.ph-hit__art', PH.art(e.card)) : h('span.ph-hit__ico', icon(e.icon || KIND_ICON[e.kind] || 'dot'));
        return h('button.ph-hit', { type: 'button', onclick: onClick }, [
            lead,
            h('span.ph-hit__main', [h('span.ph-hit__label' + (e.kind === 'command' ? '.is-mono' : ''), e.label), sub ? h('span.ph-hit__sub', sub) : null]),
            h('span.ph-hit__kind', KIND_LABEL[e.kind] || 'Setting'),
        ]);
    };

    P.choose = function (e) {
        P.close();
        if (e.kind === 'action') { e.run(); return; }
        if (e.kind === 'script') { PH.go.script(e.id); return; }
        if (e.kind === 'command') { PH.go.script(e.id, { tab: 'commands' }); return; }
        if (e.kind === 'panel') { PH.go.script(e.id, { item: e.path }); return; }
        PH.go.script(e.id, { reveal: e.path });
    };

    P.open = function () {
        if (P.el) { P.input.focus(); return; }
        var input = h('input.ph-palette__input', { type: 'text', placeholder: 'Search scripts, settings, lists, commands and actions…', spellcheck: 'false' });
        var list = h('div.ph-palette__list', { role: 'listbox' });
        var box = h('div.ph-palette', [
            h('div.ph-palette__bar', [icon('search'), input, h('span.ph-kbd.is-static', 'Esc')]),
            list,
            h('div.ph-palette__foot', [h('span', [h('span.ph-kbd.is-static', '↑'), h('span.ph-kbd.is-static', '↓'), ' move']), h('span', [h('span.ph-kbd.is-static', 'Enter'), ' open']), h('span', [h('span.ph-kbd.is-static', 'Esc'), ' close'])]),
        ]);
        var scrim = h('div.ph-scrim.ph-scrim--palette', { onmousedown: function (e) { if (e.target === scrim) P.close(); } }, box);
        PH.els.overlays.appendChild(scrim);
        requestAnimationFrame(function () { scrim.classList.add('is-in'); });
        var layer = { kind: 'palette', close: function () { P.close(); } };
        PH.pushLayer(layer);
        P.el = scrim; P.input = input; P.layer = layer;
        var results = [], sel = 0;

        function draw() {
            results = P.search(input.value);
            PH.clear(list);
            var lastKind = null;
            results.forEach(function (e, i) {
                var group = e.kind === 'script' ? 'Scripts' : e.kind === 'action' ? 'Actions' : e.kind === 'command' ? 'Commands' : 'Settings and lists';
                if (group !== lastKind && !input.value.trim()) { list.appendChild(h('div.ph-palette__group', group)); }
                lastKind = group;
                var row = P.row(e, function () { P.choose(e); });
                row.addEventListener('mouseenter', function () { select(i, true); });
                if (i === sel) row.classList.add('is-sel');
                row.dataset.i = String(i);
                list.appendChild(row);
            });
            if (!results.length) list.appendChild(h('div.ph-palette__none', 'Nothing matches “' + input.value + '”.'));
        }
        function select(i, noScroll) {
            sel = Math.max(0, Math.min(results.length - 1, i));
            PH.$$('.ph-hit', list).forEach(function (r) { r.classList.toggle('is-sel', Number(r.dataset.i) === sel); });
            if (!noScroll) { var el = list.querySelector('.ph-hit.is-sel'); if (el) el.scrollIntoView({ block: 'nearest' }); }
        }
        input.addEventListener('input', function () { sel = 0; draw(); });
        input.addEventListener('keydown', function (e) {
            if (e.key === 'ArrowDown') { e.preventDefault(); select(sel + 1); }
            else if (e.key === 'ArrowUp') { e.preventDefault(); select(sel - 1); }
            else if (e.key === 'PageDown') { e.preventDefault(); select(sel + 8); }
            else if (e.key === 'PageUp') { e.preventDefault(); select(sel - 8); }
            else if (e.key === 'Enter') { e.preventDefault(); if (results[sel]) P.choose(results[sel]); }
        });
        draw();
        setTimeout(function () { input.focus(); }, 20);
    };

    P.close = function () {
        if (!P.el) return;
        var el = P.el;
        PH.popLayer(P.layer);
        P.el = null;
        el.classList.remove('is-in');
        setTimeout(function () { if (el.parentNode) el.parentNode.removeChild(el); }, 160);
    };
})();
