/*
    Poggy Hub — boot, messages from Lua, saving, locks, pushes, keys.

    Messages from Lua (client/cl_hub.lua), all others are ignored here and
    ui/script.js ignores these:
        { action: 'hub:open', data: <open value> }        show the hub
        { action: 'hub:event', type, data }                a server push (§6.3)
        { action: 'hub:requestClose' }                     /poggy typed again: close, asking first if unsaved
        { action: 'hub:close' }                            hide at once (nothing is asked)

    Callbacks to Lua: hub { call, args } (every server call), hubClose,
    hubPosition. See hub-core.js PH.api.
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var h = PH.h, icon = PH.icon, S = PH.S, F = PH.F;

    PH.isOpen = false;

    // ----------------------------------------------------------------- DOM --

    function build() {
        var root = document.getElementById('poggy-hub');
        if (!root) {
            root = h('div');
            root.id = 'poggy-hub';
            document.body.appendChild(root);
        }
        root.className = 'ph';
        root.setAttribute('hidden', '');
        PH.clear(root);
        var top = h('header.ph-top');
        var main = h('main.ph-main');
        var savebar = h('div.ph-savebar', { role: 'region', 'aria-label': 'Unsaved changes' });
        var toasts = h('div.ph-toasts', { 'aria-live': 'polite' });
        var overlays = h('div.ph-overlays');
        var busy = h('div.ph-busy', [h('div.ph-busy__box', [h('span.ph-spinner'), h('span.ph-busy__text', 'Working…')])]);
        root.appendChild(h('div.ph-bg', [h('div.ph-bg__art'), h('div.ph-bg__shade')]));
        root.appendChild(h('div.ph-app', [top, main]));
        root.appendChild(savebar);
        root.appendChild(toasts);
        root.appendChild(overlays);
        root.appendChild(busy);
        PH.els = { root: root, top: top, main: main, savebar: savebar, toasts: toasts, overlays: overlays, busy: busy };
    }

    PH.busy = function (on, text) {
        var b = PH.els.busy;
        if (text) b.querySelector('.ph-busy__text').textContent = text;
        b.classList.toggle('is-on', !!on);
    };

    // -------------------------------------------------------- open / close --

    PH.open = function (data) {
        PH.boot = data || {};
        PH.boot.scripts = PH.boot.scripts || [];
        PH.isOpen = true;
        S.cur = null;
        S.roles = null;
        F.cache.jobs = null;
        F.cache.roles = null;
        F.cache.items = {};
        PH.rolesView.data = null;
        PH.rolesView.edit = null;
        PH.rolesView.returnTo = null;
        PH.view = 'home';
        PH.els.root.removeAttribute('hidden');
        PH.els.root.classList.remove('is-closing');
        requestAnimationFrame(function () { PH.els.root.classList.add('is-open'); });
        PH.render();
        setTimeout(function () { if (PH.els.search) PH.els.search.focus(); }, 60);
    };

    /** Hide the page and hand the game its input back. */
    PH.close = function (silent) {
        if (!PH.isOpen) return;
        PH.isOpen = false;
        PH.lock.stopHeartbeat();
        PH.palette.close();
        PH.L.closeDrawer();
        while (PH.layers.length) { var l = PH.layers.pop(); try { l.close(); } catch (e) { /* already gone */ } }
        PH.clear(PH.els.overlays);
        PH.clear(PH.els.toasts);
        PH.hideTip();
        S.cur = null;
        PH.els.root.classList.remove('is-open');
        PH.els.root.classList.add('is-closing');
        setTimeout(function () { if (!PH.isOpen) PH.els.root.setAttribute('hidden', ''); }, 180);
        // The server releases every lock this player holds when told the hub closed.
        if (!silent) PH.post('hubClose', {}).catch(function () {});
    };

    /** Close, but ask first when there is anything unsaved. */
    PH.requestClose = function () {
        if (!PH.isOpen) return;
        var n = PH.view === 'script' ? S.pendingCount() : 0;
        var rolesDirty = PH.view === 'roles' && PH.rolesDirty();
        if (!n && !rolesDirty) { PH.close(); return; }
        PH.modal({
            title: 'Close with unsaved changes?', icon: 'alert',
            body: n ? 'You have ' + PH.plural(n, 'unsaved change') + ' in ' + ((PH.cardById(S.cur.id) || {}).label || S.cur.id) + '. Closing drops them.'
                    : 'You changed roles without saving. Closing drops those changes.',
            actions: [
                { id: 'stay', label: 'Keep editing', kind: 'ghost' },
                { id: 'discard', label: 'Discard and close', kind: 'danger' },
                { id: 'save', label: 'Save and close', kind: 'primary', icon: 'save' },
            ],
        }).then(function (id) {
            if (id === 'discard') { if (n) S.discard(); PH.close(); }
            if (id === 'save') {
                (n ? PH.save(false) : PH.saveRoles()).then(function (ok) { if (ok) PH.close(); });
            }
        });
    };

    /**
     * Before leaving the current script (or the Roles page): ask about unsaved
     * changes, then release the lock. Resolves true when it is fine to go.
     */
    PH.leaveScript = function () {
        if (PH.view === 'roles') return PH.leaveRoles();
        if (PH.view !== 'script' || !S.cur) return Promise.resolve(true);
        var cur = S.cur;
        var go = function () {
            if (cur.lock && cur.lock.mine) PH.api('release', cur.id);
            PH.lock.stopHeartbeat();
            S.cur = null;
            return true;
        };
        var n = S.pendingCount();
        if (!n) return Promise.resolve(go());
        return PH.modal({
            title: 'Unsaved changes', icon: 'alert',
            body: 'You have ' + PH.plural(n, 'unsaved change') + ' in ' + ((PH.cardById(cur.id) || {}).label || cur.id) + '.',
            actions: [
                { id: 'stay', label: 'Keep editing', kind: 'ghost' },
                { id: 'discard', label: 'Discard', kind: 'danger' },
                { id: 'save', label: 'Save', kind: 'primary', icon: 'save' },
            ],
        }).then(function (id) {
            if (id === 'discard') { S.discard(); return go(); }
            if (id === 'save') return PH.save(false).then(function (ok) { return ok ? go() : false; });
            return false;
        });
    };

    // ------------------------------------------------------------ save bar --

    PH.updateSaveBar = function () {
        var bar = PH.els.savebar;
        PH.clear(bar);
        var show = false;
        if (PH.view === 'script' && S.cur && S.pendingCount() > 0) {
            show = true;
            var n = S.pendingCount();
            var files = S.cur.pending.map(function (c) { return c.file; }).filter(function (f, i, a) { return f && a.indexOf(f) === i; });
            var ro = S.isReadOnly();
            var isCore = PH.isCore(S.cur.id);
            bar.appendChild(h('div.ph-savebar__inner', [
                h('span.ph-savebar__dot'),
                h('div.ph-savebar__text', [h('b', PH.plural(n, 'unsaved change')), h('span', files.length ? 'in ' + files.join(', ') : '')]),
                h('span.ph-grow'),
                h('button.ph-btn.ph-btn--ghost', { type: 'button', onclick: function () { PH.review(); } }, [icon('eye'), 'Review']),
                h('button.ph-btn.ph-btn--ghost', { type: 'button', onclick: function () { PH.discard(); } }, [icon('reset'), 'Discard']),
                h('button.ph-btn.ph-btn--primary', { type: 'button', disabled: ro || PH.saving, title: 'Save (Ctrl+S)', onclick: function () { PH.save(false); } }, [icon('save'), PH.saving ? 'Saving…' : 'Save']),
                isCore ? null : h('button.ph-btn.ph-btn--go', { type: 'button', disabled: ro || PH.saving, title: 'Save, then restart the script so the changes apply', onclick: function () { PH.save(true); } }, [icon('restart'), 'Save & Restart']),
            ]));
            if (ro) bar.firstChild.appendChild(h('span.ph-savebar__ro', [icon('lock'), 'Read only: take the lock to save']));
        } else if (PH.view === 'roles' && PH.rolesDirty()) {
            show = true;
            bar.appendChild(h('div.ph-savebar__inner', [
                h('span.ph-savebar__dot'),
                h('div.ph-savebar__text', [h('b', 'Roles changed'), h('span', 'Saving rewrites every linked list and restarts those scripts')]),
                h('span.ph-grow'),
                h('button.ph-btn.ph-btn--ghost', { type: 'button', onclick: function () { PH.discardRoles(); } }, [icon('reset'), 'Discard']),
                h('button.ph-btn.ph-btn--primary', { type: 'button', onclick: function () { PH.saveRoles(); } }, [icon('save'), 'Save roles']),
            ]));
        }
        bar.classList.toggle('is-on', show);
        PH.els.root.classList.toggle('has-savebar', show);
    };

    S.on(function () {
        PH.updateSaveBar();
        if (PH.renderRail && PH.view === 'script') railSoon();
    });
    var railSoon = PH.debounce(function () { if (PH.view === 'script') PH.renderRail(); }, 180);

    PH.discard = function () {
        var n = S.pendingCount();
        if (!n) return;
        PH.confirm({ title: 'Discard ' + PH.plural(n, 'change') + '?', danger: true, ok: 'Discard', okIcon: 'reset',
            body: 'Every change since the last save is dropped. The files are not touched.' })
            .then(function (yes) { if (yes) { S.discard(); PH.renderScriptBody(); } });
    };

    /** The diff of every pending change, with a way to take any one back out. */
    PH.review = function () {
        var cur = S.cur;
        if (!cur || !cur.pending.length) return;
        var listEl = h('div.ph-review');
        var m;
        function labelFor(path) {
            var r = S.resolve(path);
            if (!r) return path;
            var base = F.labelFor(r.node);
            if (!r.rest.length) return base;
            return base + ' › ' + r.rest.map(function (s) { return typeof s === 'number' ? '#' + s : PH.readable(s); }).join(' › ');
        }
        function draw() {
            PH.clear(listEl);
            if (!cur.pending.length) { listEl.appendChild(h('div.ph-empty', [icon('check', 'ph-empty__ico'), h('p', 'Nothing left to save.')])); return; }
            cur.pending.forEach(function (c, i) {
                var what;
                if (c.op === 'set' || c.op === 'reset') {
                    what = h('div.ph-diff', [h('span.ph-diff__old', PH.fmtValue(c._old, 120)), icon('right'),
                        h('span.ph-diff__new', c.op === 'reset' ? 'default (' + PH.fmtValue(S.get(c.path), 80) + ')' : PH.fmtValue(c.value, 120))]);
                } else if (c.op === 'insert') {
                    what = h('div.ph-diff', [h('span.ph-diff__new', 'Adds ' + (c.key !== undefined ? '“' + c.key + '”' : c.index ? 'at #' + c.index : 'at the end') + ': ' + PH.fmtValue(c.value, 100))]);
                } else if (c.op === 'remove') {
                    what = h('div.ph-diff', [h('span.ph-diff__old', 'Removes ' + (c.key !== undefined ? '“' + c.key + '”' : '#' + c.index))]);
                } else if (c.op === 'move') {
                    what = h('div.ph-diff', [h('span.ph-diff__new', 'Moves #' + c.from + ' to #' + c.to)]);
                } else if (c.op === 'duplicate') {
                    what = h('div.ph-diff', [h('span.ph-diff__new', 'Copies ' + (c.oldKey !== undefined ? '“' + c.oldKey + '” as “' + c.newKey + '”' : '#' + c.index + ' below itself'))]);
                } else if (c.op === 'renameKey') {
                    what = h('div.ph-diff', [h('span.ph-diff__old', c.oldKey), icon('right'), h('span.ph-diff__new', c.newKey)]);
                } else if (c.op === 'linkRole') {
                    what = h('div.ph-diff', [h('span.ph-diff__new', 'Follows the role “' + c.role + '”')]);
                } else {
                    what = h('div.ph-diff', [h('span.ph-diff__old', 'Stops following ' + (c._oldRole ? '“' + c._oldRole + '”' : 'its role'))]);
                }
                listEl.appendChild(h('div.ph-review__row', [
                    h('span.ph-review__n', String(i + 1)),
                    h('span.ph-opbadge.ph-opbadge--' + c.op, c.op === 'set' ? 'Changed' : PH.opLabel(c.op)),
                    h('div.ph-review__main', [
                        h('button.ph-linkbtn.ph-review__label', { type: 'button', onclick: function () { m.close(); PH.reveal(c.path); } }, labelFor(c.path)),
                        h('div.ph-review__path', (c.file ? c.file + '  ·  ' : '') + c.path),
                        what,
                    ]),
                    h('button.ph-iconbtn', { type: 'button', title: 'Take this change back', onclick: function () {
                        var fell = S.unqueue(c);
                        if (fell.length) PH.toast({ kind: 'warning', title: PH.plural(fell.length, 'later change') + ' no longer fit and were dropped', text: fell.map(function (f) { return f.path; }).join(', ') });
                        PH.renderScriptBody();
                        draw();
                    } }, icon('reset')),
                ]));
            });
        }
        draw();
        m = PH.modal({
            title: 'Review changes', icon: 'eye', wide: true, body: listEl,
            actions: [
                { id: 'close', label: 'Close', kind: 'ghost' },
                { id: 'save', label: 'Save', kind: 'primary', icon: 'save', onClick: function () { setTimeout(function () { PH.save(false); }, 0); } },
            ],
        });
    };

    // --------------------------------------------------------------- saving --

    PH.saving = false;

    /** Save the queue. Resolves true when it was written. */
    PH.save = function (restartAfter) {
        var cur = S.cur;
        if (!cur || !cur.pending.length) return Promise.resolve(true);
        if (PH.saving) return Promise.resolve(false);
        if (S.isReadOnly()) {
            PH.toast({ kind: 'warning', title: 'Read only', text: 'You need this script’s edit lock to save. Press “Start editing” first.' });
            return Promise.resolve(false);
        }
        PH.saving = true;
        PH.updateSaveBar();
        var id = cur.id;
        var count = cur.pending.length;
        return PH.api('save', id, { fingerprints: cur.fingerprints, changes: S.wire() }).then(function (r) {
            PH.saving = false;
            if (!S.cur || S.cur.id !== id) return false;
            if (!r.ok) { PH.updateSaveBar(); return handleSaveError(r, restartAfter); }
            var v = r.value || {};
            if (v.fingerprints) cur.fingerprints = v.fingerprints;
            var needRestart = v.restart !== false;
            return reloadAfterWrite(id).then(function () {
                PH.toast({ kind: 'success', title: 'Saved', text: PH.plural(v.applied || count, 'change') + ' written to the config file.' });
                if (restartAfter) {
                    PH.restartScript(id, true);
                } else if (needRestart) {
                    S.cur.restartNeeded = true;
                    PH.renderBanners();
                    if (PH.drawHeadActions) PH.drawHeadActions();
                    if (!PH.isCore(id)) {
                        PH.toast({ kind: 'info', title: 'Restart to apply', text: 'These settings are read when the script starts.', timeout: 12000,
                            actions: [{ label: 'Restart now', primary: true, onClick: function () { PH.restartScript(id); } }] });
                    }
                }
                return true;
            });
        });
    };

    /** Re-read a script after a write (its nodes, defaults and fingerprints), keeping the tab and the lock. */
    function reloadAfterWrite(id) {
        return PH.api('script', id).then(function (r) {
            if (!r.ok || !S.cur || S.cur.id !== id) return;
            var old = S.cur;
            var fresh = S.build(r.value);
            fresh.id = id;
            fresh.lock = old.lock;
            fresh.restartNeeded = old.restartNeeded;
            S.cur = fresh;
            PH.computeTabs();
            var c = PH.cardById(id);
            if (c && r.value.card) Object.assign(c, r.value.card);
            if (PH.scriptView.history) PH.scriptView.history = null;
            var scroll = PH.els.main.scrollTop;
            PH.renderScriptBody();
            PH.renderBanners();
            PH.updateSaveBar();
            PH.els.main.scrollTop = scroll;
            S.emit('change', null);
        });
    }
    PH.reloadAfterWrite = reloadAfterWrite;

    function fileList(files) {
        var names = (files || []).map(function (f) { return typeof f === 'string' ? f : (f && (f.file || f.name)) || '?'; });
        return names.length ? names : ['a config file'];
    }

    function handleSaveError(r, restartAfter) {
        var cur = S.cur;
        var label = (PH.cardById(cur.id) || {}).label || cur.id;
        if (r.err === 'stale') {
            var names = fileList(r.files);
            return PH.modal({
                title: 'The files changed on disk', icon: 'alert', tone: 'warn',
                body: h('div', [
                    h('p', ['Something changed ', h('b', names.join(', ')), ' since you opened ' + label + ' (a text editor, an update, or the console). Nothing was saved.']),
                    h('p', 'Reload reads the files again and puts your ' + PH.plural(cur.pending.length, 'change') + ' back on top wherever they still fit. Anything that no longer fits is listed.'),
                ]),
                actions: [
                    { id: 'keep', label: 'Not now', kind: 'ghost' },
                    { id: 'reload', label: 'Reload and re-apply', kind: 'primary', icon: 'restart' },
                ],
            }).then(function (id) {
                if (id === 'reload') return reloadAndReapply();
                return false;
            });
        }
        if (r.err === 'stopped') {
            return PH.modal({
                title: label + ' is stopped', icon: 'stop',
                body: 'A script writes its own config files, so it has to be running to save. Start it, and your changes are saved straight after.',
                actions: [
                    { id: 'no', label: 'Cancel', kind: 'ghost' },
                    { id: 'start', label: 'Start and save', kind: 'go', icon: 'play' },
                ],
            }).then(function (id) {
                if (id !== 'start') return false;
                return PH.startScript(cur.id).then(function (ok) { return ok ? PH.save(restartAfter) : false; });
            });
        }
        if (r.err === 'invalid') {
            var path = r.path;
            PH.toast({ kind: 'error', title: 'Not saved: a value is not valid', text: (path ? path + ': ' : '') + (r.reason || 'Check the highlighted field.'), timeout: 12000 });
            if (path) {
                PH.reveal(path);
                setTimeout(function () {
                    var sel = '.ph-field[data-path="' + PH.pathSel(path) + '"]';
                    var f = document.querySelector('.ph-drawer ' + sel) || document.querySelector(sel);
                    if (f && f.showError) f.showError(r.reason || 'This value is not allowed.');
                }, 420);
            }
            return Promise.resolve(false);
        }
        if (r.err === 'locked' || r.err === 'not_locked') {
            cur.lock = { mine: false, holder: r.holder || cur.lock.holder || null };
            PH.lock.stopHeartbeat();
            PH.render();
            PH.toast({ kind: 'error', title: 'Not saved', text: 'You no longer hold the edit lock on ' + label + '. Your changes are still here; take the lock again to save them.' });
            return Promise.resolve(false);
        }
        PH.toast({ kind: 'error', title: 'Not saved', text: PH.errMsg(r) });
        return Promise.resolve(false);
    }

    /** After a stale save: fetch the script again and put the pending changes back on top. */
    function reloadAndReapply() {
        var old = S.cur;
        var id = old.id;
        PH.busy(true, 'Reloading…');
        return PH.api('script', id).then(function (r) {
            PH.busy(false);
            if (!r.ok || !S.cur || S.cur.id !== id) {
                PH.toast({ kind: 'error', title: 'Could not reload', text: PH.errMsg(r) });
                return false;
            }
            var fresh = S.build(r.value);
            fresh.id = id;
            fresh.lock = old.lock;
            fresh.pending = old.pending;
            S.cur = fresh;
            var failed = S.rebuild(fresh);
            PH.computeTabs();
            PH.render();
            S.emit('change', null);
            if (failed.length) {
                PH.modal({
                    title: PH.plural(failed.length, 'change') + ' could not be re-applied', icon: 'alert', wide: true,
                    body: h('div', [
                        h('p', 'The files changed in a way that ' + PH.verb(failed.length, 'this no longer fits. It was', 'these no longer fit. They were') + ' dropped; everything else is back and ready to save.'),
                        h('ul.ph-affected', failed.map(function (c) { return h('li', [h('code', c.path), h('div.ph-muted', c._why || '')]); })),
                    ]),
                    actions: [{ id: 'ok', label: 'OK', kind: 'primary' }],
                });
            } else {
                PH.toast({ kind: 'success', title: 'Reloaded', text: 'Your ' + PH.plural(fresh.pending.length, 'change') + ' ' + PH.verb(fresh.pending.length, 'is', 'are') + ' back on top of the new files. Save when ready.' });
            }
            return false;
        });
    }

    /** History → Undo: the server applies the old value as a new, saved change. */
    PH.undoLog = function (row, label) {
        var old = row.old;
        if (row.panel || /^(panel|container):/.test(String(row.file || ''))) { undoPanelLog(row, label); return; }
        PH.confirm({ title: 'Undo this change?', icon: 'reset', ok: 'Undo and save', okIcon: 'reset',
            body: label + ' goes back to ' + PH.fmtValue(typeof old === 'string' ? safeJson(old) : old, 80) + '. This is saved straight away.' })
            .then(function (yes) {
                if (!yes) return;
                var id = S.cur.id;
                PH.api('undo', row.id).then(function (r) {
                    if (!S.cur || S.cur.id !== id) return;
                    if (!r.ok) { handleSaveError(r, false); return; }
                    if (r.value && r.value.fingerprints) S.cur.fingerprints = r.value.fingerprints;
                    if (!r.value || r.value.restart !== false) S.cur.restartNeeded = true;
                    reloadAfterWrite(id).then(function () {
                        PH.toast({ kind: 'success', title: 'Undone', text: label + ' is back to its old value.' });
                    });
                });
            });
    };

    function safeJson(s) { try { return JSON.parse(s); } catch (e) { return s; } }

    /** §10: a data panel write is undone through the panel: applies at once, no lock, no reload. */
    function undoPanelLog(row, label) {
        var old = typeof row.old === 'string' ? safeJson(row.old) : row.old;
        PH.confirm({ title: 'Undo this change?', icon: 'reset', ok: 'Undo', okIcon: 'reset',
            body: label + ' goes back to ' + PH.fmtValue(old, 80) + '. The script applies it at once.' })
            .then(function (yes) {
                if (!yes) return;
                var id = S.cur && S.cur.id;
                PH.api('undo', row.id).then(function (r) {
                    if (!S.cur || S.cur.id !== id) return;
                    var okay = r.ok && r.value && r.value.ok !== false;
                    PH.toast(okay ? { kind: 'success', title: 'Undone', text: (r.value && r.value.message) || label + ' is back to its old value.' }
                        : { kind: 'error', title: 'Not undone', text: PH.errMsg(r) });
                    if (!okay) return;
                    if (S.cur.panelState && row.panel && S.cur.panelState[row.panel]) S.cur.panelState[row.panel].stack.forEach(function (lv) { lv.readAt = 0; });
                    if (PH.refreshHistory) PH.refreshHistory();
                    PH.renderScriptBody();
                });
            });
    }

    // ------------------------------------------------------ restart / start --

    function setRunning(id, running) {
        var c = PH.cardById(id);
        if (c) c.running = running;
        if (S.cur && S.cur.id === id) {
            if (S.cur.card) S.cur.card.running = running;
            if (running) S.cur.restartNeeded = false;
        }
    }

    function refreshAfterState() {
        if (PH.view === 'script' && S.cur) {
            var head = document.querySelector('.ph-shead .ph-status');
            var c = PH.cardById(S.cur.id);
            if (head && c) {
                head.className = 'ph-status' + (c.running ? ' is-on' : ' is-off');
                head.lastChild.textContent = c.running ? 'Running' : 'Stopped';
            }
            if (PH.drawHeadActions) PH.drawHeadActions();
            PH.renderBanners();
        } else if (PH.view === 'home') {
            var scroll = PH.els.main.scrollTop;
            PH.render();
            PH.els.main.scrollTop = scroll;
        }
    }

    PH.restartScript = function (id, skipAsk) {
        var c = PH.cardById(id) || { label: id };
        if (PH.isCore(id)) {
            PH.toast({ kind: 'info', title: 'Restart poggy_core by hand', text: 'It restarts every Poggy script with it, so the hub never does it for you.' });
            return Promise.resolve(false);
        }
        var ask = (!skipAsk && S.cur && S.cur.id === id && S.pendingCount())
            ? PH.confirm({ title: 'Restart without saving?', icon: 'alert', ok: 'Restart anyway',
                body: 'You have ' + PH.plural(S.pendingCount(), 'unsaved change') + '. A restart only applies what is saved in the files.' })
            : Promise.resolve(true);
        return ask.then(function (yes) {
            if (!yes) return false;
            var t = PH.toast({ kind: 'info', title: 'Restarting ' + c.label + '…', timeout: 0 });
            return PH.api('restart', id).then(function (r) {
                t.close();
                if (!r.ok) {
                    PH.toast({ kind: 'error', title: c.label + ' did not restart', text: PH.errMsg(r) });
                    return false;
                }
                setRunning(id, !r.value || r.value.running !== false);
                refreshAfterState();
                PH.toast({ kind: 'success', title: c.label + ' restarted', text: 'The saved settings are live.' });
                return true;
            });
        });
    };

    PH.startScript = function (id) {
        var c = PH.cardById(id) || { label: id };
        var t = PH.toast({ kind: 'info', title: 'Starting ' + c.label + '…', timeout: 0 });
        return PH.api('start', id).then(function (r) {
            t.close();
            if (!r.ok) { PH.toast({ kind: 'error', title: c.label + ' did not start', text: PH.errMsg(r) }); return false; }
            setRunning(id, !r.value || r.value.running !== false);
            refreshAfterState();
            PH.toast({ kind: 'success', title: c.label + ' started' });
            return true;
        });
    };

    // ---------------------------------------------------------------- locks --

    var hbTimer = null, lastBeat = 0;
    var HEARTBEAT_MS = 60000, ACTIVITY_MIN_MS = 20000;

    PH.lock = {
        /** Ask for the lock (opening a script, or "Start editing" / "Check again"). */
        acquire: function (id, redraw) {
            return PH.api('lock', id).then(function (r) {
                if (!S.cur || S.cur.id !== id) return;
                if (r.ok && r.value) {
                    S.cur.lock = { mine: !!r.value.mine, holder: r.value.holder || null };
                } else if (!r.ok) {
                    S.cur.lock = { mine: false, holder: r.holder || (S.cur.lock && S.cur.lock.holder) || null };
                    if (redraw) PH.toast({ kind: 'warning', title: 'Could not take the lock', text: PH.errMsg(r) });
                }
                var c = PH.cardById(id);
                if (c) c.lock = S.cur.lock.mine ? { holder: { name: (PH.boot.me || {}).name || 'You', since: Date.now() / 1000 } } : (S.cur.lock.holder ? { holder: S.cur.lock.holder } : null);
                if (S.cur.lock.mine) PH.lock.startHeartbeat(id); else PH.lock.stopHeartbeat();
                if (redraw) {
                    var scroll = PH.els.main.scrollTop;
                    PH.render();
                    PH.els.main.scrollTop = scroll;
                    if (S.cur.lock.mine) PH.toast({ kind: 'success', title: 'You are editing ' + ((c && c.label) || id) });
                    else if (S.cur.lock.holder) PH.toast({ kind: 'info', title: S.cur.lock.holder.name + ' is still editing' });
                }
            });
        },

        takeover: function (id) {
            var holder = S.cur && S.cur.lock.holder;
            PH.confirm({
                title: 'Take over from ' + (holder ? holder.name : 'the current editor') + '?', danger: true, ok: 'Take over', okIcon: 'unlock',
                body: 'Their page turns read only and any changes they have not saved are dropped. They are told it was you.',
            }).then(function (yes) {
                if (!yes) return;
                PH.api('takeover', id).then(function (r) {
                    if (!S.cur || S.cur.id !== id) return;
                    if (!r.ok) { PH.toast({ kind: 'error', title: 'Could not take over', text: PH.errMsg(r) }); return; }
                    S.cur.lock = { mine: true, holder: null };
                    var c = PH.cardById(id);
                    if (c) c.lock = { holder: { name: (PH.boot.me || {}).name || 'You', since: Date.now() / 1000 } };
                    PH.lock.startHeartbeat(id);
                    PH.render();
                    PH.toast({ kind: 'success', title: 'You are editing ' + ((c && c.label) || id) });
                });
            });
        },

        startHeartbeat: function (id) {
            PH.lock.stopHeartbeat();
            lastBeat = Date.now();
            hbTimer = setInterval(function () { PH.lock.beat(id); }, HEARTBEAT_MS);
        },

        stopHeartbeat: function () {
            if (hbTimer) clearInterval(hbTimer);
            hbTimer = null;
        },

        beat: function (id) {
            if (!S.cur || S.cur.id !== id || !S.cur.lock.mine) return;
            lastBeat = Date.now();
            PH.api('heartbeat', id);
        },

        /** Any edit counts as activity; tell the server, but not on every keystroke. */
        activity: function () {
            if (!S.cur || !S.cur.lock.mine) return;
            if (Date.now() - lastBeat > ACTIVITY_MIN_MS) PH.lock.beat(S.cur.id);
        },
    };

    // Moving around the page also counts as being there.
    var lastMove = 0;
    document.addEventListener('mousemove', function () {
        var now = Date.now();
        if (now - lastMove < 5000) return;
        lastMove = now;
        if (PH.isOpen) PH.lock.activity();
    }, true);

    // ---------------------------------------------------------------- pushes --

    var idleToast = null;

    PH.onEvent = function (type, data) {
        data = data || {};
        var cur = S.cur;
        var isCur = cur && cur.id === data.id;
        var c = PH.cardById(data.id);
        var label = (c && c.label) || data.id;

        if (type === 'lock') {
            if (c) c.lock = data.holder ? { holder: data.holder } : null;
            if (isCur && !cur.lock.mine) {
                var was = cur.lock.holder;
                cur.lock.holder = data.holder || null;
                PH.renderBanners();
                if (was && !data.holder) {
                    PH.toast({ kind: 'info', title: was.name + ' finished editing ' + label, timeout: 12000,
                        actions: [{ label: 'Start editing', primary: true, onClick: function () { PH.lock.acquire(data.id, true); } }] });
                }
            }
            if (PH.view === 'home') refreshAfterState();
            return;
        }

        if (type === 'kicked') {
            if (!isCur) return;
            var n = S.pendingCount();
            S.discard();
            cur.lock = { mine: false, holder: { name: data.by || 'Someone', since: Date.now() / 1000 } };
            if (c) c.lock = { holder: cur.lock.holder };
            PH.lock.stopHeartbeat();
            if (idleToast) { idleToast.close(); idleToast = null; }
            PH.render();
            PH.modal({
                title: (data.by || 'Someone') + ' took over ' + label, icon: 'lock', tone: 'warn',
                body: n ? 'They are editing it now, so your page is read only. Your ' + PH.plural(n, 'unsaved change') + ' ' + PH.verb(n, 'was', 'were') + ' dropped.'
                        : 'They are editing it now, so your page is read only. You had nothing unsaved.',
                actions: [{ id: 'ok', label: 'OK', kind: 'primary' }],
            });
            return;
        }

        if (type === 'idleWarning') {
            if (!isCur || !cur.lock.mine) return;
            var left = Math.max(0, Math.round(Number(data.secondsLeft) || 0));
            if (idleToast) idleToast.close();
            var t = PH.toast({
                kind: 'warning', title: 'Still editing ' + label + '?', timeout: 0,
                text: 'The edit lock is released in ' + fmtSecs(left) + ' unless you do something.',
                actions: [{ label: 'I’m still here', primary: true, onClick: function () { PH.lock.beat(data.id); idleToast = null; clearInterval(timer); } }],
                onClose: function () { clearInterval(timer); },
            });
            idleToast = t;
            var timer = setInterval(function () {
                left -= 1;
                if (left <= 0) { clearInterval(timer); t.set('Releasing the lock…'); return; }
                t.set('The edit lock is released in ' + fmtSecs(left) + ' unless you do something.');
            }, 1000);
            return;
        }

        if (type === 'released') {
            if (idleToast) { idleToast.close(); idleToast = null; }
            if (c) c.lock = null;
            if (!isCur || !cur.lock.mine) return;
            cur.lock = { mine: false, holder: null };
            PH.lock.stopHeartbeat();
            var np = S.pendingCount();
            PH.render();
            PH.modal({
                title: 'Your edit lock on ' + label + ' was released', icon: 'unlock', tone: 'warn',
                body: (data.reason === 'idle' ? 'Nothing happened for a while, so someone else may now edit it. ' : 'The server released it. ') +
                    (np ? 'Your ' + PH.plural(np, 'unsaved change') + ' ' + PH.verb(np, 'is', 'are') + ' still here. Take the lock again to save ' + PH.verb(np, 'it', 'them') + ' (the save checks that nobody changed the files meanwhile).'
                        : 'You had nothing unsaved.'),
                actions: [
                    { id: 'ok', label: 'Stay read only', kind: 'ghost' },
                    { id: 'take', label: 'Start editing again', kind: 'primary', icon: 'pencil', onClick: function () { PH.lock.acquire(data.id, true); } },
                ],
            });
            return;
        }

        if (type === 'scriptState') {
            setRunning(data.id, !!data.running);
            refreshAfterState();
            return;
        }
    };

    function fmtSecs(s) {
        var m = Math.floor(s / 60), r = s % 60;
        return m ? m + ':' + (r < 10 ? '0' : '') + r + ' min' : r + ' s';
    }

    // ------------------------------------------------------------------ keys --

    function typing(el) {
        if (!el) return false;
        var tag = el.tagName;
        return tag === 'INPUT' || tag === 'TEXTAREA' || el.isContentEditable;
    }

    document.addEventListener('keydown', function (e) {
        if (!PH.isOpen) return;
        var key = e.key;
        var ctrl = e.ctrlKey || e.metaKey;

        if (ctrl && (key === 's' || key === 'S')) {
            e.preventDefault();
            if (PH.view === 'script') PH.save(false);
            else if (PH.view === 'roles' && PH.rolesDirty()) PH.saveRoles();
            return;
        }
        if (ctrl && (key === 'k' || key === 'K')) { e.preventDefault(); PH.palette.open(); return; }
        if (ctrl && (key === 'f' || key === 'F')) {
            e.preventDefault();
            if (PH.els.search) { PH.els.search.focus(); PH.els.search.select(); }
            return;
        }
        if (key === '/' && !typing(document.activeElement) && !PH.topLayer()) { e.preventDefault(); PH.palette.open(); return; }

        if (key === 'Escape') {
            e.preventDefault();
            var top = PH.topLayer();
            if (top) { top.close(); return; }
            if (typing(document.activeElement) && document.activeElement !== PH.els.search) { document.activeElement.blur(); return; }
            PH.requestClose();
        }
    });

    // Keep the game's right-click from closing things; the hub has no context menus.
    document.addEventListener('contextmenu', function (e) { if (PH.isOpen) e.preventDefault(); });

    // -------------------------------------------------------------- messages --

    window.addEventListener('message', function (event) {
        var msg = event.data;
        if (!msg || typeof msg.action !== 'string' || msg.action.indexOf('hub:') !== 0) return;
        switch (msg.action) {
        case 'hub:open': PH.open(msg.data); break;
        case 'hub:event': if (PH.isOpen) PH.onEvent(msg.type, msg.data); break;
        case 'hub:requestClose': PH.requestClose(); break;
        case 'hub:close': PH.close(true); break;
        default: break;
        }
    });

    // The scripts sit at the end of <body>, so the body exists now. Build at
    // once: a hub:open message can arrive before DOMContentLoaded.
    if (document.body) build();
    else document.addEventListener('DOMContentLoaded', build);
})();
