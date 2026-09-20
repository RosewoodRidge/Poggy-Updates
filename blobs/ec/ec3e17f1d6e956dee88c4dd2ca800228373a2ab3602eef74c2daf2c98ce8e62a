/* ==========================================================================
   poggy_tickets — the page: the player's form, the staff panel, pop-ups, the
   staff badge and the warning screen.

   Every piece of text players typed is written with textContent, never
   innerHTML. No <select>: every choice is a row of radio chips. A video link
   is only ever shown and copied, never opened.
   ========================================================================== */
(function () {
    'use strict';

    var RES = (typeof GetParentResourceName === 'function') ? GetParentResourceName() : 'poggy_tickets';

    // The development mock (tools\tickets-mock) replaces this; the game never does.
    var post = window.__ptPost || function (name, body) {
        return fetch('https://' + RES + '/' + name, {
            method: 'POST', headers: { 'Content-Type': 'application/json; charset=UTF-8' }, body: JSON.stringify(body || {})
        }).then(function (r) { return r.json(); }).catch(function () { return { ok: false, message: '…' }; });
    };
    function ask(channel, action, payload) { return post('ask', { channel: channel, action: action, payload: payload || {} }); }

    var S = {
        strings: {}, categories: [], priorities: [], closeReasons: [], allRoles: [], limits: {}, volume: 0.4, command: 'ticket',
        me: { staff: false, roles: [], powers: {}, duty: true, mayHelp: false },
        view: null,
        player: { tab: 'new', step: 'choose', players: [], cooldown: 0, tickets: [], cur: null, form: null, msg: null },
        staff: { tab: 'tickets', status: 'open', category: null, priority: null, online: false, q: '', sort: 'age',
                 tickets: [], closed: [], helps: [], curId: null, detail: null, canReturn: false, draft: '' }
    };

    // ---------------------------------------------------------------- helpers

    function t(key) {
        var text = S.strings[key] || key;
        var args = Array.prototype.slice.call(arguments, 1), i = 0;
        return text.replace(/%[sd]/g, function () { return i < args.length ? String(args[i++]) : ''; });
    }

    function h(spec, attrs, kids) {
        var parts = spec.split('.'), el = document.createElement(parts[0] || 'div');
        if (parts.length > 1) el.className = parts.slice(1).join(' ');
        if (attrs) Object.keys(attrs).forEach(function (k) {
            var v = attrs[k];
            if (v === undefined || v === null || v === false) return;
            if (k === 'text') el.textContent = v;
            else if (k.slice(0, 2) === 'on') el.addEventListener(k.slice(2), v);
            else if (k === 'class') el.className += ' ' + v;
            else if (k === 'disabled' || k === 'value') el[k] = v;
            else el.setAttribute(k, v);
        });
        (kids || []).forEach(function (c) { if (c) el.appendChild(typeof c === 'string' ? document.createTextNode(c) : c); });
        return el;
    }
    function clear(el) { while (el.firstChild) el.removeChild(el.firstChild); }
    function $(id) { return document.getElementById(id); }

    function age(since, until) {
        var s = Math.max(0, Math.floor((until || Date.now() / 1000) - since));
        if (s >= 86400) return Math.floor(s / 86400) + 'd ' + Math.floor((s % 86400) / 3600) + 'h';
        if (s >= 3600) return Math.floor(s / 3600) + 'h ' + Math.floor((s % 3600) / 60) + 'm';
        if (s >= 60) return Math.floor(s / 60) + 'm ' + (s % 60) + 's';
        return s + 's';
    }
    function clock(at) {
        var d = new Date(at * 1000);
        function p(n) { return (n < 10 ? '0' : '') + n; }
        return p(d.getDate()) + '/' + p(d.getMonth() + 1) + ' ' + p(d.getHours()) + ':' + p(d.getMinutes());
    }

    // Colour carries meaning: each kind of ticket has its own pastel, priorities
    // run red -> peach -> blue -> grey, states are blue / gold / green.
    var CAT_COLORS = ['#b9a8f0', '#f08a8a', '#f2b880', '#8fb8f0', '#7fd1c7', '#a9b4c4', '#e6a8d7', '#c3e08a'];
    var PRI_COLORS = { urgent: '#f08a8a', high: '#f2b880', medium: '#8fb8f0', low: '#9aa8ba' };
    var STATUS_COLORS = { open: '#8fb8f0', claimed: '#e3c98a', closed: '#8fd3a8' };

    function tint(hex, alpha) {
        var n = parseInt(hex.slice(1), 16);
        return 'rgba(' + ((n >> 16) & 255) + ', ' + ((n >> 8) & 255) + ', ' + (n & 255) + ', ' + alpha + ')';
    }
    function catColor(id) {
        for (var i = 0; i < S.categories.length; i++) if (S.categories[i].id === id) return CAT_COLORS[i % CAT_COLORS.length];
        return '#a9b4c4';
    }
    function pill(text, color) {
        var el = h('span.pt-pill', { text: text });
        if (color) { el.style.color = color; el.style.borderColor = tint(color, 0.5); el.style.background = tint(color, 0.1); }
        return el;
    }

    /** A row of radio chips. opts: [{ value, label, color }]. */
    function chips(opts, current, onPick, small) {
        var row = h('div.pt-chips');
        opts.forEach(function (o) {
            var on = o.value === current || o.on === true;
            var chip = h('button.pt-chip' + (small ? '.pt-chip--small' : ''), { type: 'button', 'class': on ? 'is-on' : '', onclick: function () { onPick(o.value); } });
            if (o.color) {
                var dot = h('span.pt-chip__dot'); dot.style.background = o.color; chip.appendChild(dot);
                if (on) { chip.style.background = tint(o.color, 0.18); chip.style.borderColor = o.color; chip.style.color = o.color; }
            }
            chip.appendChild(document.createTextNode(o.label));
            row.appendChild(chip);
        });
        return row;
    }
    function priorityOpts() { return S.priorities.map(function (p) { return { value: p, label: t('priority_' + p), color: PRI_COLORS[p] }; }); }
    function categoryOpts() { return S.categories.map(function (c) { return { value: c.id, label: c.label, color: catColor(c.id) }; }); }
    function categoryOf(id) { return S.categories.filter(function (c) { return c.id === id; })[0]; }

    function copyText(text) {
        var ta = document.createElement('textarea');
        ta.value = text; ta.style.position = 'fixed'; ta.style.opacity = '0';
        document.body.appendChild(ta); ta.select();
        try { document.execCommand('copy'); } catch (e) { /* nothing to do */ }
        document.body.removeChild(ta);
    }

    // ---------------------------------------------------------------- pop-ups

    var sound = new Audio('sfx/incoming.mp3');
    function ding() {
        try { sound.volume = Math.max(0, Math.min(1, S.volume)); sound.currentTime = 0; sound.play(); } catch (e) { /* silent */ }
    }
    function toast(text, withSound) {
        if (!text) return;
        var el = h('div.pt-toast', null, [h('div.pt-toast__title', { text: t('ui_title') }), h('div', { text: text })]);
        $('toasts').appendChild(el);
        setTimeout(function () { el.classList.add('is-in'); }, 20);
        setTimeout(function () { el.classList.remove('is-in'); setTimeout(function () { if (el.parentNode) el.parentNode.removeChild(el); }, 350); }, 7000);
        if (withSound) ding();
    }
    function setBadge(n) {
        var b = $('badge');
        if (!n || n < 1) { b.classList.add('hidden'); return; }
        clear(b); b.appendChild(h('b', { text: String(n) })); b.appendChild(document.createTextNode(t('status_open').toLowerCase()));
        b.classList.remove('hidden');
    }

    // ------------------------------------------------------------- the window

    var win = $('window');

    function closeWindow() { post('close'); hideWindow(); }
    function hideWindow() { S.view = null; win.classList.add('hidden'); $('shade').classList.add('hidden'); clear(win); }

    function shell(kind, title, tabs, currentTab, onTab, extra) {
        clear(win);
        win.className = 'pt-window pt-window--' + kind;
        var tabRow = h('div.pt-tabs');
        tabs.forEach(function (tb) {
            tabRow.appendChild(h('button.pt-tab', { type: 'button', 'class': tb.id === currentTab ? 'is-on' : '', onclick: function () { onTab(tb.id); } },
                [tb.label, tb.dot ? h('span.pt-tab__dot') : null]));
        });
        win.appendChild(h('div.pt-head', null, [
            h('div.pt-head__title', { text: title }), tabRow, h('div.pt-head__spacer'), extra || null,
            h('button.pt-x', { type: 'button', text: '×', onclick: closeWindow })
        ]));
        $('shade').classList.remove('hidden');
        win.classList.remove('hidden');
    }

    function dialog(title, build, confirmLabel, onConfirm, danger) {
        var msg = h('div.pt-msg.is-bad');
        var body = h('div');
        var okBtn = h('button.pt-btn' + (danger ? '.pt-btn--danger' : '.pt-btn--go'), { type: 'button', text: confirmLabel || t('sp_confirm') });
        var shade = h('div.pt-dialog-shade', null, [h('div.pt-dialog', null, [
            h('div.pt-dialog__title', { text: title }), body, msg,
            h('div.pt-dialog__foot', null, [h('button.pt-btn', { type: 'button', text: t('sp_cancel'), onclick: close }), okBtn])
        ])]);
        function close() { if (shade.parentNode) shade.parentNode.removeChild(shade); }
        build(body);
        okBtn.addEventListener('click', function () {
            okBtn.disabled = true;
            Promise.resolve(onConfirm()).then(function (r) {
                okBtn.disabled = false;
                if (r === false) return;
                if (r && r.ok === false) { msg.textContent = r.message || '…'; return; }
                close();
            });
        });
        win.appendChild(shade);
    }

    /** A chat. The reader's own side is on the right, the other side on the left. */
    function conversation(ticket, readerIsStaff) {
        var box = h('div.pt-convo');
        var list = ticket.conversation || [];
        if (!list.length) box.appendChild(h('div.pt-convo__empty', { text: t('ui_no_messages') }));
        list.forEach(function (m) {
            var me = !!m.staff === !!readerIsStaff;
            var who = m.staff ? t('ui_staff') + ' · ' + m.name : m.name;
            if (me && !readerIsStaff) who = t('ui_you');
            box.appendChild(h('div.pt-bub' + (me ? '.is-me' : '') + (m.staff ? '.is-staff' : ''), null, [
                h('div.pt-bub__who', null, [who, h('time', { text: clock(m.at) })]),
                h('div.pt-bub__text', { text: m.text })
            ]));
        });
        setTimeout(function () { box.scrollTop = box.scrollHeight; }, 0);
        return box;
    }

    // ------------------------------------------------------- player: the form

    function freshForm() {
        var first = S.categories[0] || {};
        return { category: first.id, priority: first.priority || 'medium', description: '', reported: null, clip: '', presence: true };
    }

    function renderPlayer() {
        var P = S.player;
        var unseen = P.tickets.some(function (x) { return x.unseen; });
        shell('player', t('ui_title'), [{ id: 'new', label: t('ui_tab_new') }, { id: 'mine', label: t('ui_tab_mine'), dot: unseen }], P.tab, function (id) {
            P.tab = id; P.cur = null; P.step = 'choose';
            if (id === 'mine') loadMine(); else renderPlayer();
        });
        var body = h('div.pt-body');
        win.appendChild(body);
        if (P.tab === 'new') { if (P.step === 'form') renderForm(body); else renderChoice(body); }
        else if (P.cur) renderMineDetail(body); else renderMineList(body);
    }

    /** First thing a player sees: two big buttons, so the choice is obvious. */
    function renderChoice(body) {
        var P = S.player;
        function card(cls, title, text, onclick) {
            return h('button.pt-choice.' + cls, { type: 'button', onclick: onclick }, [h('div.pt-choice__title', { text: title }), h('div.pt-choice__text', { text: text })]);
        }
        body.appendChild(h('div.pt-choose__title', { text: t('ui_choose_title') }));
        var help = card('pt-choice--help', t('ui_help_button'), t('ui_help_hint'), function () {
            help.disabled = true;
            ask('player', 'help', {}).then(function (r) { toast(r.message, false); if (r.ok) closeWindow(); else { P.msg = { ok: false, text: r.message }; renderPlayer(); } });
        });
        body.appendChild(h('div.pt-choose', null, [help, card('pt-choice--ticket', t('ui_choice_ticket_title'), t('ui_choice_ticket_text'), function () { P.step = 'form'; P.msg = null; renderPlayer(); })]));
        if (P.msg && !P.msg.ok) body.appendChild(h('div.pt-msg.is-bad', { text: P.msg.text }));
    }

    function renderForm(body) {
        var P = S.player, F = P.form || (P.form = freshForm());
        var cat = categoryOf(F.category) || {};

        body.appendChild(h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: '‹ ' + t('ui_back'), onclick: function () { P.step = 'choose'; renderPlayer(); } }));
        body.appendChild(h('div.pt-label', { text: t('ui_category') }));
        body.appendChild(chips(categoryOpts(), F.category, function (v) {
            F.category = v; F.priority = (categoryOf(v) || {}).priority || F.priority; F.reported = null; renderPlayer();
        }));

        body.appendChild(h('div.pt-label', { text: t('ui_priority') }));
        body.appendChild(chips(priorityOpts(), F.priority, function (v) { F.priority = v; renderPlayer(); }));

        body.appendChild(h('div.pt-label', { text: t('ui_description') }));
        body.appendChild(h('div.pt-hint', { text: t('ui_description_hint') }));
        var max = S.limits.description || 1000;
        var count = h('div.pt-count', { text: F.description.length + ' / ' + max });
        var ta = h('textarea.pt-text', { maxlength: max, value: F.description, oninput: function () { F.description = ta.value; count.textContent = ta.value.length + ' / ' + max; } });
        body.appendChild(ta); body.appendChild(count);

        if (cat.reportsPlayer) {
            body.appendChild(h('div.pt-label', { text: t('ui_reported') }));
            if (!P.players.length) body.appendChild(h('div.pt-hint', { text: t('ui_reported_none') }));
            var opts = P.players.map(function (p) { return { value: p.id, label: p.label }; });
            opts.push({ value: null, label: t('ui_reported_skip') });
            body.appendChild(chips(opts, F.reported, function (v) { F.reported = v; renderPlayer(); }, true));
        }
        if (cat.wantsClip) {
            body.appendChild(h('div.pt-label', { text: t('ui_clip') }));
            body.appendChild(h('div.pt-hint', { text: t('ui_clip_hint') }));
            var clip = h('input.pt-input', { type: 'text', maxlength: S.limits.clip || 300, value: F.clip, placeholder: 'https://', oninput: function () { F.clip = clip.value; } });
            body.appendChild(clip);
        }

        var toggle = h('div.pt-toggle' + (F.presence ? '.is-on' : ''), { onclick: function () { F.presence = !F.presence; toggle.classList.toggle('is-on', F.presence); } },
            [h('div.pt-toggle__box'), h('div', { text: t('ui_presence') })]);
        body.appendChild(toggle);

        var msg = h('div.pt-msg' + (P.msg ? (P.msg.ok ? '.is-good' : '.is-bad') : ''), { text: P.msg ? P.msg.text : (P.cooldown > 0 ? t('msg_cooldown', P.cooldown) : '') });
        body.appendChild(msg);

        var send = h('button.pt-btn.pt-btn--act', { type: 'button', text: t('ui_submit'), disabled: P.cooldown > 0, onclick: function () {
            send.disabled = true;
            ask('player', 'submit', F).then(function (r) {
                P.msg = { ok: !!r.ok, text: r.message || '' };
                if (r.ok) { P.form = freshForm(); P.cooldown = 1; toast(r.message, false); closeWindow(); return; }
                renderPlayer();
            });
        } });
        body.appendChild(h('div.pt-foot', null, [h('div.pt-foot__note', { text: t('ui_position_note') }), send]));

    }

    function loadMine() {
        ask('player', 'mine').then(function (r) { S.player.tickets = r.tickets || []; if (S.view === 'player') renderPlayer(); });
    }

    function stateLine(x) {
        if (x.status === 'closed') return t('ui_closed_as', t('close_' + (x.closeReason || 'resolved')));
        if (x.status === 'claimed') return t('ui_claimed_by', x.claimedByName || '?');
        return t('ui_waiting');
    }

    function renderMineList(body) {
        var list = S.player.tickets;
        if (!list.length) { body.appendChild(h('div.pt-empty', { text: t('ui_no_tickets') })); return; }
        list.forEach(function (x) {
            var row = h('div.pt-mine' + (x.unseen ? '.is-unseen' : ''), { onclick: function () {
                S.player.cur = x.id;
                if (x.unseen) { x.unseen = false; ask('player', 'seen', { id: x.id }); }
                renderPlayer();
            } }, [
                h('div.pt-mine__top', null, [h('span.pt-mine__id', { text: '#' + x.id }), h('span.pt-mine__cat', null, [pill(x.categoryLabel, catColor(x.category))]), pill(stateLine(x), STATUS_COLORS[x.status])]),
                h('div.pt-mine__text', { text: x.description })
            ]);
            if (!x.unseen) row.style.borderLeftColor = catColor(x.category);
            body.appendChild(row);
        });
    }

    function renderMineDetail(body) {
        var P = S.player, x = P.tickets.filter(function (k) { return k.id === P.cur; })[0];
        if (!x) { P.cur = null; renderMineList(body); return; }
        var wrap = h('div.pt-minedetail');
        body.appendChild(wrap);
        var back = h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: '‹ ' + t('ui_back'), onclick: function () { P.cur = null; renderPlayer(); } });
        back.style.alignSelf = 'flex-start'; back.style.marginBottom = '0.8rem';
        wrap.appendChild(back);
        wrap.appendChild(h('div.pt-d-head', null, [h('span.pt-d-head__id', { text: '#' + x.id }), h('span.pt-d-head__cat', { text: x.categoryLabel }),
            h('span.pt-d-head__pills', null, [pill(stateLine(x), STATUS_COLORS[x.status])]), h('span.pt-d-head__age', { text: clock(x.createdAt) })]));
        if (x.closeNote) wrap.appendChild(h('div.pt-hint', { text: x.closeNote }));
        var desc = h('div.pt-desc', { text: x.description }); desc.style.borderLeftColor = catColor(x.category);
        wrap.appendChild(desc);
        wrap.appendChild(h('div.pt-label', { text: t('sp_conversation') }));
        wrap.appendChild(conversation(x, false));
        if (x.status !== 'closed') {
            var input = h('input.pt-input', { type: 'text', maxlength: S.limits.message || 500, placeholder: t('ui_reply') });
            var go = function () {
                if (!input.value.trim()) return;
                ask('player', 'reply', { id: x.id, text: input.value }).then(function (r) { if (!r.ok) toast(r.message, false); });
                input.value = '';
            };
            input.addEventListener('keydown', function (e) { if (e.key === 'Enter') go(); });
            wrap.appendChild(h('div.pt-replyrow', null, [input, h('button.pt-btn.pt-btn--act', { type: 'button', text: t('ui_send'), onclick: go })]));
            setTimeout(function () { input.focus(); }, 30);
        }
    }

    // ------------------------------------------------------------ staff panel

    function staffTabs() {
        var tabs = [{ id: 'tickets', label: t('sp_tab_tickets') }];
        if (S.me.powers.warn || S.me.powers.kick || S.me.powers.ban) tabs.push({ id: 'players', label: t('sp_tab_players') });
        if (S.me.powers.unban) tabs.push({ id: 'bans', label: t('sp_tab_bans') });
        if (S.me.powers.hire) tabs.push({ id: 'staff', label: t('sp_tab_staff') });
        if (S.me.powers.audit) tabs.push({ id: 'audit', label: t('sp_tab_audit') });
        return tabs;
    }

    function renderStaff() {
        var A = S.staff;
        var duty = h('div.pt-toggle.pt-duty' + (S.me.duty ? '.is-on' : ''), { onclick: function () {
            ask('staff', 'duty').then(function (r) { if (r.ok) { S.me.duty = r.duty; renderStaff(); } });
        } }, [h('div.pt-toggle__box'), h('div', { text: t(S.me.duty ? 'sp_on_duty' : 'sp_off_duty') })]);
        duty.style.marginTop = '0';
        var back = A.canReturn ? h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_return'), onclick: function () { ask('staff', 'return'); } }) : null;
        if (back) back.style.margin = '0 0.8rem 0 0';
        var extra = h('div', null, [back, duty]); extra.style.display = 'flex'; extra.style.alignItems = 'center';

        shell('staff', t('sp_title'), staffTabs(), A.tab, function (id) {
            A.tab = id;
            renderStaff();
            if (id === 'bans') loadBans(); else if (id === 'staff') loadStaff(); else if (id === 'audit') loadAudit(); else if (id === 'players') loadPlayers();
        }, extra);

        if (A.tab === 'tickets') renderTickets();
        else if (A.tab === 'players') renderPlayers();
        else if (A.tab === 'bans') renderBans();
        else if (A.tab === 'staff') renderStaffTab();
        else renderAudit();
    }

    function loadTickets() {
        return ask('staff', 'list', { closed: false }).then(function (r) {
            S.staff.tickets = (r.tickets || []).map(markMine); S.staff.helps = r.helps || [];
            if (S.view === 'staff' && S.staff.tab === 'tickets') renderStaff();
        });
    }

    function visibleTickets() {
        var A = S.staff, src = A.status === 'closed' ? A.closed : A.tickets, q = A.q.trim().toLowerCase();
        var rank = {}; S.priorities.forEach(function (p, i) { rank[p] = i; });
        var out = src.filter(function (x) {
            if (A.status === 'open' && x.status !== 'open') return false;
            if (A.status === 'claimed' && x.status !== 'claimed') return false;
            if (A.status === 'mine' && !(x.status === 'claimed' && x.mine)) return false;
            if (A.status === 'closed' && x.status !== 'closed') return false;
            if (A.category && x.category !== A.category) return false;
            if (A.priority && x.priority !== A.priority) return false;
            if (A.online && !x.reporterOnline) return false;
            if (q) {
                var hay = ('#' + x.id + ' ' + (x.reporterName || '') + ' ' + (x.reportedName || '') + ' ' + x.description + ' ' + (x.town || '') + ' ' + (x.claimedByName || '')).toLowerCase();
                if (hay.indexOf(q) === -1) return false;
            }
            return true;
        });
        out.sort(function (a, b) {
            if (A.status === 'closed') return (b.closedAt || 0) - (a.closedAt || 0);
            if (A.sort === 'priority' && rank[a.priority] !== rank[b.priority]) return rank[a.priority] - rank[b.priority];
            return a.createdAt - b.createdAt;
        });
        return out;
    }

    function renderTickets() {
        var A = S.staff;
        var left = h('div.pt-left'), right = h('div.pt-right');
        win.appendChild(h('div.pt-staff', null, [left, right]));

        if (A.helps.length) {
            var strip = h('div.pt-helps', null, [h('div.pt-helps__title', { text: t('sp_help_strip') })]);
            A.helps.forEach(function (hp) {
                var row = h('div.pt-helprow', null, [h('div.pt-helprow__who', null, [hp.name, h('small', { text: (hp.town || '') + ' · ' }), h('small', { 'data-since': hp.at, text: age(hp.at) })])]);
                if (hp.responder) row.appendChild(h('span.pt-helprow__by', { text: t('sp_help_by', hp.responder) }));
                else row.appendChild(h('button.pt-btn.pt-btn--small.pt-btn--act', { type: 'button', text: t('sp_help_respond'), onclick: function () { ask('staff', 'helpRespond', { id: hp.id }).then(function (r) { if (!r.ok) toast(r.message); }); } }));
                row.appendChild(h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_goto_short'), onclick: function () { ask('staff', 'goTo', { help: hp.id }).then(function (r) { if (!r.ok) toast(r.message); }); } }));
                row.appendChild(h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: t('sp_help_done'), onclick: function () { ask('staff', 'helpDone', { id: hp.id }); } }));
                strip.appendChild(row);
            });
            left.appendChild(strip);
        }

        var filters = h('div.pt-filters');
        var search = h('input.pt-input', { type: 'text', placeholder: t('sp_search'), value: A.q, oninput: function () { A.q = search.value; drawList(); } });
        filters.appendChild(search);
        filters.appendChild(chips([
            { value: 'open', label: t('sp_filter_open'), color: STATUS_COLORS.open }, { value: 'claimed', label: t('sp_filter_claimed'), color: STATUS_COLORS.claimed },
            { value: 'mine', label: t('sp_filter_mine'), color: '#b9a8f0' }, { value: 'closed', label: t('sp_filter_closed'), color: STATUS_COLORS.closed }
        ], A.status, function (v) {
            A.status = v;
            if (v === 'closed') ask('staff', 'list', { closed: true }).then(function (r) { A.closed = (r.tickets || []).map(markMine); renderStaff(); });
            else renderStaff();
        }, true));
        filters.appendChild(h('div.pt-filters__label', { text: t('sp_filter_kind') }));
        filters.appendChild(chips(categoryOpts(), A.category, function (v) { A.category = A.category === v ? null : v; renderStaff(); }, true));
        filters.appendChild(h('div.pt-filters__label', { text: t('sp_filter_priority') }));
        filters.appendChild(chips(priorityOpts(), A.priority, function (v) { A.priority = A.priority === v ? null : v; renderStaff(); }, true));
        var count = h('div.pt-filters__count');
        filters.appendChild(h('div.pt-filters__foot', null, [count,
            chips([{ value: 'online', label: t('sp_filter_online'), on: A.online }, { value: 'sort', label: t(A.sort === 'age' ? 'sp_sort_age' : 'sp_sort_priority') }], null, function (v) {
                if (v === 'online') A.online = !A.online; else A.sort = A.sort === 'age' ? 'priority' : 'age';
                renderStaff();
            }, true)]));
        left.appendChild(filters);

        var listEl = h('div.pt-list');
        left.appendChild(listEl);

        function drawList() {
            clear(listEl);
            var rows = visibleTickets();
            count.textContent = t('sp_count', rows.length);
            if (!rows.length) { listEl.appendChild(h('div.pt-empty', { text: t('sp_none') })); return; }
            rows.forEach(function (x) {
                var state = x.status === 'claimed' ? (x.claimedByName || '') + (x.claimerOnline ? '' : ' · ' + t('sp_staff_offline')) : t('status_' + x.status);
                var row = h('div.pt-row' + (x.id === A.curId ? '.is-cur' : '') + (x.reporterOnline ? '' : '.is-off'), { onclick: function () { openDetail(x.id); } }, [
                    h('div.pt-row__main', null, [
                        h('div.pt-row__top', null, [h('span.pt-row__id', { text: '#' + x.id }), pill(x.categoryLabel, catColor(x.category)), pill(t('priority_' + x.priority), PRI_COLORS[x.priority])]),
                        h('div.pt-row__who', null, [h('span.pt-name' + (x.reporterOnline ? '' : '.is-off'), { text: x.reporterName || '?' })])
                    ]),
                    h('div.pt-row__side', null, [
                        x.status === 'closed' ? h('div.pt-row__age', { text: age(x.createdAt, x.closedAt) }) : h('div.pt-row__age', { 'data-since': x.createdAt, text: age(x.createdAt) }),
                        h('div.pt-row__state' + (x.status === 'claimed' && !x.claimerOnline ? '.is-stale' : ''), { text: state })
                    ])
                ]);
                row.style.borderLeftColor = PRI_COLORS[x.priority] || '';
                listEl.appendChild(row);
            });
        }
        drawList();
        renderDetail(right);
    }

    function openDetail(id) {
        var A = S.staff;
        A.curId = id; A.detail = null; A.draft = '';
        renderStaff();
        ask('staff', 'detail', { id: id }).then(function (r) {
            if (A.curId !== id) return;
            if (!r.ok) { A.curId = null; toast(r.message); renderStaff(); return; }
            markMine(r.ticket); A.detail = r; renderStaff();
        });
    }

    function act(action, payload) {
        return ask('staff', action, payload).then(function (r) {
            if (r.ok && S.staff.curId) ask('staff', 'detail', { id: S.staff.curId }).then(function (d) { if (d.ok) { markMine(d.ticket); S.staff.detail = d; if (S.view === 'staff') renderStaff(); } });
            return r;
        });
    }

    function reasonDialog(title, label, action, id, danger, more) {
        var ta, extra = {};
        dialog(title, function (body) {
            if (more) more(body, extra);
            body.appendChild(h('div.pt-label', { text: label }));
            ta = h('textarea.pt-text.pt-text--short', { maxlength: S.limits.reason || 300 });
            body.appendChild(ta);
            setTimeout(function () { ta.focus(); }, 30);
        }, title, function () {
            if (!ta.value.trim()) return { ok: false, message: t('msg_need_reason') };
            var p = { id: id, reason: ta.value };
            Object.keys(extra).forEach(function (k) { p[k] = extra[k]; });
            return act(action, p).then(function (r) { if (r.ok && r.message) toast(r.message); return r; });
        }, danger);
    }

    function renderDetail(right) {
        var A = S.staff, D = A.detail;
        if (!A.curId || !D) { right.appendChild(h('div.pt-empty', { text: A.curId ? '…' : t('sp_pick') })); return; }
        var x = D.ticket, P = S.me.powers, open = x.status !== 'closed';

        // --- top: what, who, where (never scrolls)
        var top = h('div.pt-d-top');
        var statusText = x.status === 'claimed' ? t('ui_claimed_by', x.claimedByName || '?') : (x.status === 'closed' ? t('ui_closed_as', t('close_' + (x.closeReason || 'resolved'))) : t('status_open'));
        var pills = h('span.pt-d-head__pills', null, [pill(t('priority_' + x.priority), PRI_COLORS[x.priority]), pill(statusText, STATUS_COLORS[x.status])]);
        if (x.wantPresence) pills.appendChild(pill(t('sp_wants_presence'), '#e6a8d7'));
        var catEl = h('span.pt-d-head__cat', { text: x.categoryLabel }); catEl.style.color = catColor(x.category);
        top.appendChild(h('div.pt-d-head', null, [h('span.pt-d-head__id', { text: '#' + x.id }), catEl, pills,
            open ? h('span.pt-d-head__age', { 'data-since': x.createdAt, text: age(x.createdAt) }) : h('span.pt-d-head__age', { text: age(x.createdAt, x.closedAt) })]));

        var canMod = P.warn || P.kick || P.ban;
        function person(label, name, online, hist, half, account) {
            var nameEl = h('span.pt-name' + (online ? '' : '.is-off') + (canMod && account ? '.is-link' : ''), { text: name });
            if (canMod && account) nameEl.addEventListener('click', function () { openPlayer({ account: account, name: name }); });
            return h('div.pt-fact' + (half ? '.pt-fact--half' : ''), null, [h('div.pt-fact__k', { text: label }), h('div.pt-fact__v', null, [
                nameEl,
                hist ? h('small', { text: t('sp_history', hist.tickets, hist.warnings) }) : null
            ])]);
        }
        var facts = h('div.pt-facts');
        facts.appendChild(person(t('sp_player'), x.reporterName || '?', x.reporterOnline, D.history && D.history.reporter, !x.hasReported, x.reporterAccount));
        if (x.hasReported) facts.appendChild(person(t('sp_reported'), x.reportedName, x.reportedOnline, D.history && D.history.reported, false, x.reportedAccount));
        facts.appendChild(h('div.pt-fact', null, [h('div.pt-fact__k', { text: t('sp_where') }), h('div.pt-fact__v', null, [x.town || '—', h('small.pt-mono', { text: Math.round(x.pos.x) + ', ' + Math.round(x.pos.y) })])]));
        facts.appendChild(h('div.pt-fact', null, [h('div.pt-fact__k', { text: t('sp_sent') }), h('div.pt-fact__v', { text: clock(x.createdAt) })]));
        if (x.clip) {
            var copyBtn = h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_copy'), onclick: function () { copyText(x.clip); copyBtn.textContent = t('sp_copied'); } });
            facts.appendChild(h('div.pt-fact.pt-fact--wide', null, [h('div.pt-fact__k', { text: t('sp_clip') }), h('div.pt-clip', null, [h('div.pt-clip__url', { text: x.clip }), copyBtn])]));
        }
        top.appendChild(facts);
        var desc = h('div.pt-desc', { text: x.description }); desc.style.borderLeftColor = catColor(x.category);
        top.appendChild(desc);
        if (x.closeNote) top.appendChild(h('div.pt-hint', { text: t('sp_note') + ': ' + x.closeNote }));
        right.appendChild(top);

        // --- middle: the chat takes whatever height is left and scrolls inside
        var chat = h('div.pt-d-chat', null, [h('div.pt-label', { text: t('sp_conversation') }), conversation(x, true)]);
        if (open && P.reply) {
            var input = h('input.pt-input', { type: 'text', maxlength: S.limits.message || 500, placeholder: t('ui_reply'), value: A.draft, oninput: function () { A.draft = input.value; } });
            var go = function () {
                if (!input.value.trim()) return;
                var text = input.value; input.value = ''; A.draft = '';
                act('reply', { id: x.id, text: text }).then(function (r) { if (!r.ok) toast(r.message); });
            };
            input.addEventListener('keydown', function (e) { if (e.key === 'Enter') go(); });
            chat.appendChild(h('div.pt-replyrow', null, [input, h('button.pt-btn.pt-btn--act', { type: 'button', text: t('ui_send'), onclick: go })]));
        }
        right.appendChild(chat);

        // --- bottom: the action bar, always in the same place
        function btn(label, cls, fn, disabled) { return h('button.pt-btn' + (cls || ''), { type: 'button', text: label, onclick: fn, disabled: disabled }); }
        function simple(action, payload) { return function () { act(action, payload).then(function (r) { if (!r.ok) toast(r.message); }); }; }
        function pickDialog(title, options, start, confirm, send) {
            var pick = start;
            dialog(title, function (body) {
                var draw = function () { clear(body); body.appendChild(chips(options, pick, function (v) { pick = v; draw(); })); if (!options.length) body.appendChild(h('div.pt-hint', { text: t('sp_none') })); };
                draw();
            }, confirm, function () { return pick ? send(pick) : false; });
        }
        var bar = h('div.pt-d-bar');

        var row1 = h('div.pt-d-bar__row', null, [h('div.pt-d-bar__label', { text: t('sp_bar_ticket') })]);
        if (open) {
            if (x.status === 'open' || !x.mine) row1.appendChild(btn(t('sp_claim'), '.pt-btn--act', simple('claim', { id: x.id }), x.status === 'claimed' && x.claimerOnline && !x.mine));
            if (x.status === 'claimed' && (x.mine || P.assign)) row1.appendChild(btn(t('sp_unclaim'), '', simple('unclaim', { id: x.id })));
            if (P.assign) row1.appendChild(btn(t('sp_assign'), '', function () {
                pickDialog(t('sp_assign_to'), (D.assignable || []).map(function (m) { return { value: m.account, label: m.name + (m.duty ? '' : ' (' + t('sp_off_duty').toLowerCase() + ')') }; }),
                    null, t('sp_assign'), function (v) { return act('assign', { id: x.id, account: v }); });
            }));
            if (P.category) row1.appendChild(btn(t('sp_category'), '', function () { pickDialog(t('sp_category'), categoryOpts(), x.category, t('sp_confirm'), function (v) { return act('category', { id: x.id, category: v }); }); }));
            if (P.priority) row1.appendChild(btn(t('sp_priority'), '', function () { pickDialog(t('sp_priority'), priorityOpts(), x.priority, t('sp_confirm'), function (v) { return act('priority', { id: x.id, priority: v }); }); }));
        }
        row1.appendChild(h('div.pt-d-bar__spacer'));
        if (P.audit) row1.appendChild(btn(t('sp_history_btn'), '.pt-btn--ghost', function () { audit.ticket = x.id; audit.account = ''; audit.group = ''; audit.q = ''; A.tab = 'audit'; renderStaff(); loadAudit(); }));
        if (open && P.close) row1.appendChild(btn(t('sp_close_ticket'), '.pt-btn--go', function () {
            var pick = 'resolved', note;
            dialog(t('sp_close_ticket'), function (body) {
                var box = h('div');
                var draw = function () { clear(box); box.appendChild(chips(S.closeReasons.map(function (v) { return { value: v, label: t('close_' + v) }; }), pick, function (v) { pick = v; draw(); })); };
                body.appendChild(h('div.pt-label', { text: t('sp_close_reason') })); body.appendChild(box); draw();
                body.appendChild(h('div.pt-label', { text: t('sp_note') }));
                note = h('textarea.pt-text.pt-text--short', { maxlength: 500 }); body.appendChild(note);
            }, t('sp_close_ticket'), function () { return act('close', { id: x.id, reason: pick, note: note.value }); });
        }));
        bar.appendChild(row1);

        var row2 = h('div.pt-d-bar__row', null, [h('div.pt-d-bar__label', { text: t('sp_bar_go') })]);
        if (P.teleport) {
            var go2 = function (where) { return function () { ask('staff', 'goTo', { id: x.id, where: where }).then(function (r) { if (!r.ok) toast(r.message); }); }; };
            row2.appendChild(btn(t('sp_goto_player'), '', go2('player'), !x.reporterOnline));
            row2.appendChild(btn(t('sp_goto_place'), '', go2('place')));
            if (A.canReturn) row2.appendChild(btn(t('sp_return'), '.pt-btn--ghost', function () { ask('staff', 'return'); }));
        }
        row2.appendChild(h('div.pt-d-bar__spacer'));
        if (x.hasReported && (P.warn || P.kick || P.ban)) {
            var who = ' · ' + x.reportedName;
            row2.appendChild(h('div.pt-d-bar__label', { text: x.reportedName }));
            if (x.reportedAccount) row2.appendChild(btn(t('pl_record'), '.pt-btn--ghost', function () { openPlayer({ account: x.reportedAccount, name: x.reportedName }); }));
            if (P.warn) row2.appendChild(btn(t('sp_warn'), '.pt-btn--danger', function () { reasonDialog(t('sp_warn') + who, t('sp_reason'), 'warn', x.id, true); }));
            if (P.kick) row2.appendChild(btn(t('sp_kick'), '.pt-btn--danger', function () { reasonDialog(t('sp_kick') + who, t('sp_reason'), 'kick', x.id, true); }, !x.reportedOnline));
            if (P.ban) row2.appendChild(btn(t('sp_ban'), '.pt-btn--danger', function () {
                reasonDialog(t('sp_ban') + who, t('sp_reason'), 'ban', x.id, true, function (body, extra) {
                    extra.length = 'perm'; extra.cheat = x.category === 'cheater';
                    var lenBox = h('div'), cheat;
                    var draw = function () { clear(lenBox); lenBox.appendChild(chips(['perm', '1d', '3d', '7d', '30d'].map(function (v) { return { value: v, label: t('sp_len_' + v) }; }), extra.length, function (v) { extra.length = v; draw(); })); };
                    body.appendChild(h('div.pt-label', { text: t('sp_ban_length') })); body.appendChild(lenBox); draw();
                    cheat = h('div.pt-toggle' + (extra.cheat ? '.is-on' : ''), { onclick: function () { extra.cheat = !extra.cheat; cheat.classList.toggle('is-on', extra.cheat); } }, [h('div.pt-toggle__box'), h('div', { text: t('sp_ban_cheat') })]);
                    body.appendChild(cheat);
                });
            }));
        }
        if (row2.childNodes.length > 2) bar.appendChild(row2);
        right.appendChild(bar);
    }

    // ---------------------------------------------------------------- players
    // Warn, kick and ban with no ticket: /mod, this tab, or the button in /poggy.

    var people = { online: [], found: [], q: '', cur: null, card: null };

    function loadPlayers() {
        return ask('staff', 'players').then(function (r) {
            people.online = r.players || [];
            if (S.view === 'staff' && S.staff.tab === 'players') renderStaff();
        });
    }
    /** who: { src } or { account, name }. Opens the Players tab on that person. */
    function openPlayer(who) {
        people.cur = who; people.card = null;
        S.staff.tab = 'players';
        renderStaff();
        if (!people.online.length) loadPlayers();
        ask('staff', 'playerCard', who).then(function (r) {
            if (people.cur !== who) return;
            if (!r.ok) { toast(r.message); people.cur = null; } else people.card = r.card;
            if (S.view === 'staff' && S.staff.tab === 'players') renderStaff();
        });
    }
    function sameAsCur(p) { return !!(people.card && people.card.account === p.account); }

    function modDialog(title, action, who, withLength) {
        var ta, extra = { length: 'perm', cheat: false };
        dialog(title, function (body) {
            if (withLength) {
                var lenBox = h('div'), cheat;
                var draw = function () { clear(lenBox); lenBox.appendChild(chips(['perm', '1d', '3d', '7d', '30d'].map(function (v) { return { value: v, label: t('sp_len_' + v) }; }), extra.length, function (v) { extra.length = v; draw(); })); };
                body.appendChild(h('div.pt-label', { text: t('sp_ban_length') })); body.appendChild(lenBox); draw();
                cheat = h('div.pt-toggle', { onclick: function () { extra.cheat = !extra.cheat; cheat.classList.toggle('is-on', extra.cheat); } }, [h('div.pt-toggle__box'), h('div', { text: t('sp_ban_cheat') })]);
                body.appendChild(cheat);
            }
            body.appendChild(h('div.pt-label', { text: t('sp_reason') }));
            ta = h('textarea.pt-text.pt-text--short', { maxlength: S.limits.reason || 300 });
            body.appendChild(ta);
            setTimeout(function () { ta.focus(); }, 30);
        }, title, function () {
            if (!ta.value.trim()) return { ok: false, message: t('msg_need_reason') };
            return ask('staff', action, { player: who, reason: ta.value, length: extra.length, cheat: extra.cheat }).then(function (r) {
                if (r.ok) { if (r.message) toast(r.message); loadPlayers(); openPlayer(who); }
                return r;
            });
        }, true);
    }

    function renderPlayers() {
        var left = h('div.pt-left'), right = h('div.pt-right');
        win.appendChild(h('div.pt-staff', null, [left, right]));

        var filters = h('div.pt-filters');
        var search = h('input.pt-input', { type: 'text', placeholder: t('pl_search'), value: people.q, oninput: function () { people.q = search.value; drawList(); } });
        search.addEventListener('keydown', function (e) {
            if (e.key !== 'Enter' || !search.value.trim()) return;
            ask('staff', 'findPlayers', { text: search.value }).then(function (r) { people.found = r.rows || []; drawList(); });
        });
        search.style.marginBottom = '0';
        filters.appendChild(search);
        left.appendChild(filters);
        var listEl = h('div.pt-list');
        left.appendChild(listEl);

        function rowOf(p) {
            var who = p.src ? { src: p.src } : { account: p.account, name: p.name };
            var row = h('div.pt-row' + (sameAsCur(p) ? '.is-cur' : '') + (p.online === false ? '.is-off' : ''), { onclick: function () { openPlayer(who); } }, [
                h('div.pt-row__main', null, [
                    h('div.pt-row__top', null, [p.src ? h('span.pt-row__id', { text: String(p.src) }) : null, h('span.pt-name' + (p.online === false ? '.is-off' : ''), { text: p.name }), p.staff ? pill(t('pl_staff'), '#b9a8f0') : null]),
                    (p.warnings || p.reports) ? h('div.pt-row__who', null, [h('small', { text: t('pl_counts', p.warnings || 0, p.reports || 0) })]) : null
                ])
            ]);
            var heat = (p.warnings || 0) + (p.reports || 0);
            row.style.borderLeftColor = heat >= 3 ? '#f08a8a' : (heat > 0 ? '#f2b880' : '');
            return row;
        }
        function drawList() {
            clear(listEl);
            var q = people.q.trim().toLowerCase();
            var online = people.online.filter(function (p) { return !q || (p.name + ' ' + p.src).toLowerCase().indexOf(q) !== -1; });
            var found = people.found.filter(function (p) { return !p.online; });
            if (found.length) { listEl.appendChild(h('div.pt-filters__label', { text: t('pl_found') })); found.forEach(function (p) { listEl.appendChild(rowOf(p)); }); }
            listEl.appendChild(h('div.pt-filters__label', { text: t('pl_online') + ' · ' + online.length }));
            online.forEach(function (p) { listEl.appendChild(rowOf(p)); });
        }
        drawList();

        var c = people.card;
        if (!people.cur || !c) { right.appendChild(h('div.pt-empty', { text: people.cur ? '…' : t('pl_pick') })); return; }
        var who = c.src ? { src: c.src } : { account: c.account, name: c.name };
        var P = S.me.powers;

        var top = h('div.pt-d-top');
        top.appendChild(h('div.pt-d-head', null, [c.src ? h('span.pt-d-head__id', { text: String(c.src) }) : null, h('span.pt-d-head__cat', { text: c.name || c.account }),
            h('span.pt-d-head__pills', null, [pill(t(c.online ? 'sp_online' : 'sp_offline'), c.online ? '#8fd3a8' : '#9aa8ba'),
                pill(t('pl_warnings') + ' ' + c.warnings.length, c.warnings.length ? '#f2b880' : null), pill(t('pl_bans') + ' ' + c.bans.length, c.bans.length ? '#f08a8a' : null)])]));
        right.appendChild(top);

        var rec = h('div.pt-record');
        function section(title, rows, draw) {
            rec.appendChild(h('div.pt-label', { text: title + ' · ' + rows.length }));
            if (!rows.length) rec.appendChild(h('div.pt-hint', { text: t('pl_clean') }));
            rows.forEach(function (r) { rec.appendChild(draw(r)); });
        }
        section(t('pl_warnings'), c.warnings, function (w) {
            return h('div.pt-rec', null, [h('div.pt-rec__when', { text: clock(w.created_at) }), h('div.pt-rec__text', null, [w.reason, h('span', { text: '  —  ' + (w.warned_by_name || '?') + (w.acknowledged_at ? '' : ' · ' + t('pl_unread')) })])]);
        });
        section(t('pl_bans'), c.bans, function (b) {
            var state = b.active ? t('sp_ban_active') + ' · ' + b.remaining : (b.revokedAt ? t('sp_ban_lifted') : t('sp_ban_expired'));
            return h('div.pt-rec', null, [h('div.pt-rec__when', { text: clock(b.createdAt) }), h('div.pt-rec__text', null, [pill(state, b.active ? '#f08a8a' : '#8fd3a8'), b.reason, h('span', { text: '  —  ' + (b.by || '?') })])]);
        });
        section(t('pl_tickets'), c.tickets, function (k) {
            var row = h('div.pt-rec', null, [h('div.pt-rec__when', { text: clock(k.createdAt) }), h('div.pt-rec__text', null, [
                pill('#' + k.id + ' ' + k.categoryLabel, catColor(k.category)), pill(t(k.reportedThem ? 'pl_reported' : 'pl_sent'), k.reportedThem ? '#f08a8a' : null), k.description])]);
            if (k.status !== 'closed') row.appendChild(h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: t('pl_open_ticket'), onclick: function () { S.staff.tab = 'tickets'; openDetail(k.id); } }));
            return row;
        });
        right.appendChild(rec);

        var bar = h('div.pt-d-bar');
        var row = h('div.pt-d-bar__row', null, [h('div.pt-d-bar__label', { text: t('sp_bar_player') })]);
        function btn(label, cls, fn, disabled) { return h('button.pt-btn' + (cls || ''), { type: 'button', text: label, onclick: fn, disabled: disabled }); }
        if (P.teleport) row.appendChild(btn(t('sp_goto'), '', function () { ask('staff', 'goTo', { player: who }).then(function (r) { if (!r.ok) toast(r.message); }); }, !c.online));
        if (S.staff.canReturn) row.appendChild(btn(t('sp_return'), '.pt-btn--ghost', function () { ask('staff', 'return'); }));
        row.appendChild(h('div.pt-d-bar__spacer'));
        var title = ' · ' + (c.name || c.account);
        if (P.warn) row.appendChild(btn(t('sp_warn'), '.pt-btn--danger', function () { modDialog(t('sp_warn') + title, 'modWarn', who, false); }));
        if (P.kick) row.appendChild(btn(t('sp_kick'), '.pt-btn--danger', function () { modDialog(t('sp_kick') + title, 'modKick', who, false); }, !c.online));
        if (P.ban) row.appendChild(btn(t('sp_ban'), '.pt-btn--danger', function () { modDialog(t('sp_ban') + title, 'modBan', who, true); }));
        bar.appendChild(row);
        right.appendChild(bar);
    }

    // ------------------------------------------------------------------- bans

    var bans = { rows: [], q: '' };
    function loadBans() { ask('staff', 'bans', { search: bans.q }).then(function (r) { bans.rows = r.bans || []; if (S.view === 'staff' && S.staff.tab === 'bans') renderStaff(); }); }
    function renderBans() {
        var page = h('div.pt-page');
        var search = h('input.pt-input', { type: 'text', placeholder: t('sp_search'), value: bans.q });
        search.addEventListener('keydown', function (e) { if (e.key === 'Enter') { bans.q = search.value; loadBans(); } });
        page.appendChild(h('div.pt-page__bar', null, [search, h('button.pt-btn', { type: 'button', text: t('sp_search'), onclick: function () { bans.q = search.value; loadBans(); } })]));
        var table = h('table.pt-table', null, [h('tr', null, ['#', t('sp_player'), t('sp_reason'), t('sp_audit_who'), '', ''].map(function (c) { return h('th', { text: c }); }))]);
        bans.rows.forEach(function (b) {
            var state = b.active ? t('sp_ban_active') + ' · ' + b.remaining : (b.revokedAt ? t('sp_ban_lifted') + ' · ' + (b.revokedBy || '') : t('sp_ban_expired'));
            table.appendChild(h('tr' + (b.active ? '' : '.is-dim'), null, [
                h('td.pt-mono', { text: String(b.id) }), h('td', { text: (b.name || '?') + (b.category === 'cheat' ? ' ⚑' : '') }),
                h('td', { text: b.reason + (b.revokeReason ? ' — ' + b.revokeReason : '') }), h('td', { text: (b.by || '?') + ' · ' + clock(b.createdAt) }), h('td', null, [pill(state, b.active ? '#f08a8a' : '#8fd3a8')]),
                h('td', null, [b.active ? h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_unban'), onclick: function () {
                    var ta;
                    dialog(t('sp_unban') + ' #' + b.id, function (body) { body.appendChild(h('div.pt-label', { text: t('sp_unban_reason') })); ta = h('textarea.pt-text.pt-text--short', { maxlength: 300 }); body.appendChild(ta); },
                        t('sp_unban'), function () { if (!ta.value.trim()) return { ok: false, message: t('msg_need_reason') }; return ask('staff', 'unban', { id: b.id, reason: ta.value }).then(function (r) { if (r.ok) loadBans(); return r; }); });
                } }) : null])
            ]));
        });
        page.appendChild(table);
        if (!bans.rows.length) page.appendChild(h('div.pt-empty', { text: t('sp_none') }));
        win.appendChild(page);
    }

    // ------------------------------------------------------------------ staff

    var roster = { rows: [], results: [], q: '', pick: null, roles: {} };
    function loadStaff() { ask('staff', 'staffList').then(function (r) { roster.rows = r.staff || []; if (S.view === 'staff' && S.staff.tab === 'staff') renderStaff(); }); }
    function hireRoles() { return S.allRoles; }
    function renderStaffTab() {
        var page = h('div.pt-page');
        page.appendChild(h('div.pt-label', { text: t('sp_hire') }));
        var search = h('input.pt-input', { type: 'text', placeholder: t('sp_hire_search'), value: roster.q });
        var find = function () { roster.q = search.value; ask('staff', 'search', { text: roster.q }).then(function (r) { roster.results = r.rows || []; roster.pick = null; renderStaff(); }); };
        search.addEventListener('keydown', function (e) { if (e.key === 'Enter') find(); });
        page.appendChild(h('div.pt-page__bar', null, [search, h('button.pt-btn', { type: 'button', text: t('sp_search'), onclick: find })]));
        if (roster.results.length) {
            var box = h('div.pt-results');
            roster.results.forEach(function (c, i) {
                box.appendChild(h('div.pt-result' + (roster.pick === i ? '.is-on' : ''), { onclick: function () { roster.pick = i; roster.roles = {}; renderStaff(); } },
                    [h('span.pt-name' + (c.online ? '' : '.is-off'), { text: c.name })]));
            });
            page.appendChild(box);
        }
        if (roster.pick !== null && roster.results[roster.pick]) {
            var c = roster.results[roster.pick];
            page.appendChild(h('div.pt-label', { text: t('sp_hire_roles') + ' · ' + c.name }));
            var row = h('div.pt-chips');
            hireRoles().forEach(function (role) {
                row.appendChild(h('button.pt-chip' + (roster.roles[role] ? '.is-on' : ''), { type: 'button', text: t('role_' + role), onclick: function () { roster.roles[role] = !roster.roles[role]; renderStaff(); } }));
            });
            page.appendChild(row);
            page.appendChild(h('button.pt-btn.pt-btn--go', { type: 'button', text: t('sp_save'), onclick: function () {
                var roles = Object.keys(roster.roles).filter(function (k) { return roster.roles[k]; });
                if (!roles.length) return;
                ask('staff', 'setRoles', { account: c.account, name: c.name, roles: roles }).then(function (r) { toast(r.message); roster.pick = null; roster.results = []; loadStaff(); });
            } }));
        }

        page.appendChild(h('div.pt-label', { text: t('sp_tab_staff') }));
        var table = h('table.pt-table', null, [h('tr', null, [t('sp_player'), t('sp_hire_roles'), ''].map(function (x) { return h('th', { text: x }); }))]);
        roster.rows.forEach(function (m) {
            var rolesCell = h('td');
            if (m.framework) rolesCell.appendChild(h('span', { text: t('role_admin') + ' (' + t('sp_framework_admin') + ')' }));
            else {
                var chipsRow = h('div.pt-chips');
                hireRoles().forEach(function (role) {
                    var on = m.roles.indexOf(role) !== -1;
                    chipsRow.appendChild(h('button.pt-chip.pt-chip--small' + (on ? '.is-on' : ''), { type: 'button', text: t('role_' + role), onclick: function () {
                        var next = m.roles.filter(function (r) { return r !== role; }); if (!on) next.push(role);
                        ask('staff', 'setRoles', { account: m.account, name: m.name, roles: next }).then(loadStaff);
                    } }));
                });
                rolesCell.appendChild(chipsRow);
            }
            table.appendChild(h('tr', null, [
                h('td', null, [h('span.pt-name' + (m.online ? '' : '.is-off'), { text: m.name })]), rolesCell,
                h('td', null, [m.framework ? null : h('button.pt-btn.pt-btn--small.pt-btn--danger', { type: 'button', text: t('sp_remove'), onclick: function () { ask('staff', 'setRoles', { account: m.account, name: m.name, roles: [] }).then(loadStaff); } })])
            ]));
        });
        page.appendChild(table);
        win.appendChild(page);
    }

    // ------------------------------------------------------------------ audit

    var audit = { rows: [], account: '', group: '', ticket: null, q: '' };
    var AUDIT_GROUPS = { tickets: '#8fb8f0', moderation: '#f08a8a', movement: '#7fd1c7', staff: '#b9a8f0' };
    var AUDIT_GROUP_OF = { claim: 'tickets', release: 'tickets', assign: 'tickets', reply: 'tickets', category: 'tickets', priority: 'tickets', close: 'tickets',
        warn: 'moderation', kick: 'moderation', ban: 'moderation', unban: 'moderation',
        teleport: 'movement', 'return': 'movement', invis_on: 'movement', invis_off: 'movement',
        duty: 'staff', staff_hire: 'staff', staff_roles: 'staff', staff_remove: 'staff', help_respond: 'staff' };

    function loadAudit() {
        ask('staff', 'audit', { account: audit.account, group: audit.group, ticket: audit.ticket, search: audit.q }).then(function (r) {
            audit.rows = r.rows || []; audit.people = r.people || [];
            if (S.view === 'staff' && S.staff.tab === 'audit') renderStaff();
        });
    }

    function dayLabel(at) {
        var d = new Date(at * 1000), now = new Date();
        var days = Math.round((new Date(now.getFullYear(), now.getMonth(), now.getDate()) - new Date(d.getFullYear(), d.getMonth(), d.getDate())) / 86400000);
        if (days === 0) return t('sp_today');
        if (days === 1) return t('sp_yesterday');
        return d.getDate() + ' ' + ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'][d.getMonth()] + ' ' + d.getFullYear();
    }
    function hhmm(at) { var d = new Date(at * 1000); function p(n) { return (n < 10 ? '0' : '') + n; } return p(d.getHours()) + ':' + p(d.getMinutes()); }

    function renderAudit() {
        var side = h('div.pt-audit__side'), main = h('div.pt-audit__main');
        win.appendChild(h('div.pt-audit', null, [side, main]));

        var search = h('input.pt-input', { type: 'text', placeholder: t('sp_search'), value: audit.q });
        search.addEventListener('keydown', function (e) { if (e.key === 'Enter') { audit.q = search.value; loadAudit(); } });
        side.appendChild(search);

        if (audit.ticket) {
            side.appendChild(h('div.pt-label', { text: t('sp_audit_ticket') }));
            side.appendChild(chips([{ value: 'x', label: '#' + audit.ticket + '  ×', on: true }], null, function () { audit.ticket = null; loadAudit(); }, true));
        }

        side.appendChild(h('div.pt-label', { text: t('sp_audit_action') }));
        var groups = [{ value: '', label: t('sp_audit_all') }].concat(Object.keys(AUDIT_GROUPS).map(function (g) { return { value: g, label: t('au_g_' + g), color: AUDIT_GROUPS[g] }; }));
        side.appendChild(chips(groups, audit.group, function (v) { audit.group = v; loadAudit(); }, true));

        side.appendChild(h('div.pt-label', { text: t('sp_audit_who') }));
        var people = [{ account: '', name: t('sp_audit_all') }].concat(audit.people || []);
        people.forEach(function (p) {
            side.appendChild(h('div.pt-person' + (audit.account === p.account ? '.is-on' : ''), { onclick: function () { audit.account = p.account; loadAudit(); } },
                [h('span', { text: p.name || p.account }), p.n ? h('span.pt-person__n', { text: String(p.n) }) : null]));
        });

        if (!audit.rows.length) { main.appendChild(h('div.pt-empty', { text: t('sp_none') })); return; }
        var lastDay = null;
        audit.rows.forEach(function (r) {
            var day = dayLabel(r.created_at);
            if (day !== lastDay) { lastDay = day; main.appendChild(h('div.pt-audit__day', { text: day })); }
            var group = AUDIT_GROUP_OF[r.action] || 'staff';
            var text = h('div.pt-aud__text', null, [h('b', { text: r.name || r.account }), ' ' + t('au_' + r.action)]);
            if (r.target) text.appendChild(document.createTextNode(' ' + r.target));
            if (r.detail) text.appendChild(h('span', { text: '  —  ' + r.detail }));
            var row = h('div.pt-aud', null, [h('div.pt-aud__time', { text: hhmm(r.created_at) }), h('div.pt-aud__pill', null, [pill(r.action.replace(/_/g, ' '), AUDIT_GROUPS[group])]), text]);
            if (r.ticket_id) row.appendChild(h('button.pt-aud__ticket', { type: 'button', text: '#' + r.ticket_id, onclick: function () { audit.ticket = r.ticket_id; loadAudit(); } }));
            main.appendChild(row);
        });
    }

    // ------------------------------------------------------ the server tells us

    function upsert(list, x) {
        for (var i = 0; i < list.length; i++) if (list[i].id === x.id) { list[i] = x; return; }
        list.push(x);
    }
    function markMine(x) { x.mine = !!(x.claimedBy && S.me.account && x.claimedBy === S.me.account); return x; }

    function onPush(kind, d) {
        var A = S.staff;
        if (kind === 'toast') { toast(d.toast || d.text, d.sound); return; }
        if (kind === 'badge') { setBadge(d.count); return; }
        if (d.toast) toast(d.toast, d.sound);

        if (kind === 'ticket' && d.ticket) {
            var x = markMine(d.ticket);
            if (x.status === 'closed') { A.tickets = A.tickets.filter(function (k) { return k.id !== x.id; }); upsert(A.closed, x); }
            else upsert(A.tickets, x);
            if (A.detail && A.curId === x.id) A.detail.ticket = x;
            if (S.view === 'staff' && A.tab === 'tickets') renderStaff();
        } else if (kind === 'gone') {
            A.tickets = A.tickets.filter(function (k) { return k.id !== d.id; });
            if (A.curId === d.id) { A.curId = null; A.detail = null; }
            if (S.view === 'staff' && A.tab === 'tickets') renderStaff();
        } else if (kind === 'mine' && d.ticket) {
            upsert(S.player.tickets, d.ticket);
            S.player.tickets.sort(function (a, b) { return b.id - a.id; });
            if (S.view === 'player' && S.player.tab === 'mine') {
                if (S.player.cur === d.ticket.id) { d.ticket.unseen = false; ask('player', 'seen', { id: d.ticket.id }); }
                renderPlayer();
            }
        } else if (kind === 'help' && d.help) {
            upsert(A.helps, d.help);
            if (S.view === 'staff' && A.tab === 'tickets') renderStaff();
        } else if (kind === 'helpGone') {
            A.helps = A.helps.filter(function (k) { return k.id !== d.id; });
            if (S.view === 'staff' && A.tab === 'tickets') renderStaff();
        }
    }

    // ------------------------------------------------------------ the warning

    var warnTimer = null;
    function showWarning(id, reason) {
        $('warning-title').textContent = t('warn_title');
        $('warning-reason').textContent = reason || '';
        var ok = $('warning-ok'), left = 3;
        ok.disabled = true; ok.textContent = t('warn_wait');
        ok.onclick = function () { $('warning').classList.add('hidden'); post('ackWarning', { id: id }); };
        $('warning').classList.remove('hidden');
        ding();
        clearInterval(warnTimer);
        warnTimer = setInterval(function () { left--; if (left <= 0) { clearInterval(warnTimer); ok.disabled = false; ok.textContent = t('warn_button'); } }, 1000);
    }

    // ------------------------------------------------------------------ wiring

    window.addEventListener('message', function (e) {
        var m = e.data || {};
        if (m.type === 'init') {
            S.strings = m.strings || {}; S.categories = m.categories || []; S.priorities = m.priorities || [];
            S.closeReasons = m.closeReasons || []; S.allRoles = m.allRoles || []; S.limits = m.limits || {};
            S.volume = typeof m.volume === 'number' ? m.volume : 0.4; S.command = m.command || 'ticket';
        } else if (m.type === 'me') {
            S.me = { staff: !!m.staff, roles: m.roles || [], powers: m.powers || {}, duty: m.duty !== false, mayHelp: !!m.mayHelp, account: m.account };
            setBadge(m.badge);
        } else if (m.type === 'open') {
            S.view = m.view;
            if (m.view === 'player') {
                S.player.players = m.players || []; S.player.cooldown = m.cooldown || 0; S.player.msg = null; S.player.cur = null; S.player.tab = 'new'; S.player.step = 'choose';
                renderPlayer(); loadMine();
            } else {
                S.staff.canReturn = !!m.canReturn;
                S.staff.tab = m.tab === 'players' ? 'players' : 'tickets';
                renderStaff(); loadTickets();
                if (m.tab === 'players') { people.cur = null; people.card = null; people.found = []; people.q = ''; loadPlayers().then(function () { if (m.focus) openPlayer({ src: m.focus }); }); }
            }
        } else if (m.type === 'close') {
            hideWindow();
        } else if (m.type === 'push') {
            onPush(m.kind, m.data || {});
        } else if (m.type === 'warn') {
            showWarning(m.id, m.reason);
        }
    });

    document.addEventListener('keydown', function (e) {
        if (e.key === 'Escape' && S.view) {
            var open = win.querySelector('.pt-dialog-shade');
            if (open) open.parentNode.removeChild(open); else closeWindow();
        }
    });

    // Live ages: every element with data-since counts up.
    setInterval(function () {
        var els = document.querySelectorAll('[data-since]');
        for (var i = 0; i < els.length; i++) els[i].textContent = age(Number(els[i].getAttribute('data-since')));
    }, 1000);
})();
