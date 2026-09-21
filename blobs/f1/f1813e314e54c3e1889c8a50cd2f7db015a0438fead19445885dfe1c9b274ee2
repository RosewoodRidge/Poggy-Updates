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
        me: { staff: false, roles: [], powers: {}, duty: true, mayHelp: false, canDelete: false },
        roleDefs: [], powerList: [], answers: true, full: false, canned: [], web: null,
        view: null,
        player: { tab: 'new', step: 'choose', players: [], cooldown: 0, tickets: [], cur: null, form: null, msg: null,
                  waits: {}, similar: [], answers: [], answersQ: '', answer: null },
        staff: { tab: 'tickets', status: 'open', category: null, priority: null, online: false, q: '', sort: 'age',
                 tickets: [], closed: [], archived: [], helps: [], curId: null, detail: null, canReturn: false, draft: '',
                 note: false, sub: 'roster' }
    };

    // Roles are data (an owner can add "Staff Manager"), so their names and
    // colours come from the server, not from the translations.
    function roleOf(id) { for (var i = 0; i < S.roleDefs.length; i++) if (S.roleDefs[i].id === id) return S.roleDefs[i]; return null; }
    function roleLabel(id) { var r = roleOf(id); return r ? r.label : id; }
    function roleColor(id) { var r = roleOf(id); return (r && r.color) || '#8fb8f0'; }
    function roleOpts() { return S.roleDefs.map(function (r) { return { value: r.id, label: r.label, color: r.color }; }); }

    // Full screen is each staff member's own choice, remembered on their machine.
    function loadFull(fallback) { try { var v = window.localStorage.getItem('pt-full'); return v === null ? !!fallback : v === '1'; } catch (e) { return !!fallback; } }
    function saveFull(on) { try { window.localStorage.setItem('pt-full', on ? '1' : '0'); } catch (e) { /* private mode */ } }

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
            // A bubble: how many tickets are waiting behind this chip.
            if (o.count) chip.appendChild(h('span.pt-chip__n', { text: String(o.count) }));
            row.appendChild(chip);
        });
        return row;
    }
    /** Tick any number. picked is an object used as a set; onChange runs after each tick. */
    function multiChips(opts, picked, onChange, small) {
        var holder = h('div');
        var draw = function () {
            clear(holder);
            holder.appendChild(chips(opts.map(function (o) { return { value: o.value, label: o.label, color: o.color, on: !!picked[o.value] }; }), null, function (v) {
                picked[v] = !picked[v]; draw(); if (onChange) onChange();
            }, small));
        };
        draw();
        return holder;
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
        win.className = 'pt-window pt-window--' + kind + (kind === 'staff' && S.full ? ' pt-window--full' : '');
        var tabRow = h('div.pt-tabs');
        tabs.forEach(function (tb) {
            tabRow.appendChild(h('button.pt-tab', { type: 'button', 'class': tb.id === currentTab ? 'is-on' : '', onclick: function () { onTab(tb.id); } },
                [tb.label, tb.dot ? h('span.pt-tab__dot') : null, tb.count ? h('span.pt-chip__n', { text: String(tb.count) }) : null]));
        });
        // The community's one web ID, for everyone: in the staff window's header, and
        // as a line of its own under the player window's (which is too narrow for it).
        win.appendChild(h('div.pt-head', null, [
            h('div.pt-head__title', { text: title }), tabRow, h('div.pt-head__spacer'), kind === 'staff' ? webId(false) : null, extra || null,
            h('button.pt-x', { type: 'button', text: '×', onclick: closeWindow })
        ]));
        if (kind !== 'staff' && S.web) win.appendChild(webId(true));
        $('shade').classList.remove('hidden');
        win.classList.remove('hidden');
    }

    function webId(asLine) {
        if (!S.web) return null;
        var code = h('span.pt-webid__code', { text: S.web.code });
        var said = h('span.pt-webid__copy', { text: t('sp_copy') });
        var el = h('button.pt-webid' + (asLine ? '.pt-webid--line' : ''), { type: 'button', title: t('web_id_tip', S.web.site), onclick: function () {
            copyText(S.web.code); said.textContent = t('sp_copied');
        } }, [h('span.pt-webid__k', { text: t('web_id_short') }), code, asLine ? h('span.pt-webid__site', { text: S.web.site }) : null, said]);
        return el;
    }

    function dialog(title, build, confirmLabel, onConfirm, danger) {
        var msg = h('div.pt-msg.is-bad');
        var okBtn = h('button.pt-btn' + (danger ? '.pt-btn--danger' : '.pt-btn--go'), { type: 'button', text: confirmLabel || t('sp_confirm') });
        var body = h('div.pt-dialog__body');
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

    /** The answer staff marked: pinned above the chat, for staff and player alike. */
    function resolutionBlock(ticket) {
        if (!ticket.resolution) return null;
        return h('div.pt-answer', null, [
            h('div.pt-answer__k', null, [t('ui_answer'), ticket.resolutionBy ? h('small', { text: ' · ' + ticket.resolutionBy }) : null]),
            h('div.pt-answer__v', { text: ticket.resolution })
        ]);
    }

    /** A chat. The reader's own side is on the right, the other side on the left.
        onMark(index, isMarked): staff who may mark the answer get a link on each message. */
    function conversation(ticket, readerIsStaff, onMark) {
        var box = h('div.pt-convo');
        var list = ticket.conversation || [];
        if (!list.length) box.appendChild(h('div.pt-convo__empty', { text: t('ui_no_messages') }));
        list.forEach(function (m, i) {
            var me = !!m.staff === !!readerIsStaff;
            var who = m.staff ? t('ui_staff') + ' · ' + m.name : m.name;
            if (m.system) who = m.name;
            else if (me && !readerIsStaff) who = t('ui_you');
            if (m.internal) who = t('sp_internal') + ' · ' + m.name;
            var head = h('div.pt-bub__who', null, [who, h('time', { text: clock(m.at) })]);
            if (onMark && !m.internal && !m.system) {
                head.appendChild(h('button.pt-bub__mark', { type: 'button', text: t(m.resolution ? 'sp_unmark_answer' : 'sp_mark_answer'),
                    onclick: function () { onMark(i + 1, !!m.resolution); } }));
            }
            box.appendChild(h('div.pt-bub' + (me ? '.is-me' : '') + (m.staff ? '.is-staff' : '') + (m.internal ? '.is-note' : '') +
                (m.system ? '.is-system' : '') + (m.resolution ? '.is-answer' : ''), null, [head, h('div.pt-bub__text', { text: m.text })]));
        });
        setTimeout(function () { box.scrollTop = box.scrollHeight; }, 0);
        return box;
    }

    // ------------------------------------------------------- player: the form

    var similarTimer = null;
    function formKinds() { return categoryOpts().filter(function (o) { return !(categoryOf(o.value) || {}).webOnly; }); }
    function freshForm() {
        var first = S.categories.filter(function (c) { return !c.webOnly; })[0] || {};
        return { category: first.id, priority: first.priority || 'medium', description: '', reported: null, clip: '', presence: true };
    }

    function renderPlayer() {
        var P = S.player;
        var unseen = P.tickets.some(function (x) { return x.unseen; });
        var tabs = [{ id: 'new', label: t('ui_tab_new') }, { id: 'mine', label: t('ui_tab_mine'), dot: unseen }];
        if (S.answers) tabs.push({ id: 'answers', label: t('ui_tab_answers') });
        if (S.web) tabs.push({ id: 'web', label: t('web_tab'), dot: !S.web.linked });
        if (P.tab === 'web' && !S.web) P.tab = 'new';
        shell('player', t('ui_title'), tabs, P.tab, function (id) {
            P.tab = id; P.cur = null; P.step = 'choose'; P.answer = null;
            if (id === 'mine') loadMine(); else if (id === 'answers') loadAnswers(); else renderPlayer();
            if (id === 'web') watchWeb();
        });
        var body = h('div.pt-body');
        win.appendChild(body);
        if (P.tab === 'new') { if (P.step === 'form') renderForm(body); else renderChoice(body); }
        else if (P.tab === 'answers') renderAnswers(body);
        else if (P.tab === 'web') renderWebMine(body);
        else if (P.cur) renderMineDetail(body); else renderMineList(body);
    }

    // Solved questions: tickets staff chose to share, every name hidden.
    function loadAnswers() {
        ask('player', 'answers', { search: S.player.answersQ }).then(function (r) { S.player.answers = r.rows || []; if (S.view === 'player' && S.player.tab === 'answers') renderPlayer(); });
    }
    function openAnswer(id) {
        ask('player', 'answer', { id: id }).then(function (r) {
            if (!r.ok) { toast(r.message); return; }
            S.player.tab = 'answers'; S.player.answer = r.ticket; renderPlayer();
        });
    }
    function answerCard(a) {
        var card = h('div.pt-mine', { onclick: function () { openAnswer(a.id); } }, [
            h('div.pt-mine__top', null, [h('span.pt-mine__cat', null, [pill(a.categoryLabel, catColor(a.category))])]),
            h('div.pt-mine__text', { text: a.description }),
            a.resolution ? h('div.pt-mine__answer', { text: a.resolution }) : null
        ]);
        card.style.borderLeftColor = catColor(a.category);
        return card;
    }
    function renderAnswers(body) {
        var P = S.player, a = P.answer;
        if (a) {
            var wrap = h('div.pt-minedetail'); body.appendChild(wrap);
            var back = h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: '‹ ' + t('ui_back'), onclick: function () { P.answer = null; renderPlayer(); } });
            back.style.alignSelf = 'flex-start'; back.style.marginBottom = '0.8rem';
            wrap.appendChild(back);
            wrap.appendChild(h('div.pt-d-head', null, [h('span.pt-d-head__cat', { text: a.categoryLabel })]));
            var desc = h('div.pt-desc', { text: a.description }); desc.style.borderLeftColor = catColor(a.category);
            wrap.appendChild(desc);
            var pinned = resolutionBlock({ resolution: a.resolution }); if (pinned) wrap.appendChild(pinned);
            wrap.appendChild(h('div.pt-label', { text: t('sp_conversation') }));
            wrap.appendChild(conversation(a, false));
            return;
        }
        body.appendChild(h('div.pt-hint', { text: t('ui_answers_hint') }));
        var search = h('input.pt-input', { type: 'text', placeholder: t('ui_answers_search'), value: P.answersQ });
        search.addEventListener('keydown', function (e) { if (e.key === 'Enter') { P.answersQ = search.value; loadAnswers(); } });
        body.appendChild(search);
        if (!P.answers.length) { body.appendChild(h('div.pt-empty', { text: t('ui_answers_none') })); return; }
        P.answers.forEach(function (x) { body.appendChild(answerCard(x)); });
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
        body.appendChild(chips(formKinds(), F.category, function (v) {
            F.category = v; F.priority = (categoryOf(v) || {}).priority || F.priority; F.reported = null; renderPlayer();
        }));

        if (P.waits && P.waits[F.category]) body.appendChild(h('div.pt-hint.pt-wait', { text: t('ui_wait', P.waits[F.category]) }));

        body.appendChild(h('div.pt-label', { text: t('ui_priority') }));
        body.appendChild(chips(priorityOpts(), F.priority, function (v) { F.priority = v; renderPlayer(); }));

        body.appendChild(h('div.pt-label', { text: t('ui_description') }));
        body.appendChild(h('div.pt-hint', { text: t('ui_description_hint') }));
        var max = S.limits.description || 1000;
        var count = h('div.pt-count', { text: F.description.length + ' / ' + max });
        var similarBox = h('div.pt-similar');
        // Drawn on its own, so the player keeps typing while answers come and go.
        var drawSimilar = function () {
            clear(similarBox);
            if (!P.similar.length) return;
            similarBox.appendChild(h('div.pt-label', { text: t('ui_similar_title') }));
            similarBox.appendChild(h('div.pt-hint', { text: t('ui_similar_hint') }));
            P.similar.forEach(function (a) { similarBox.appendChild(answerCard(a)); });
        };
        var ta = h('textarea.pt-text', { maxlength: max, value: F.description, oninput: function () {
            F.description = ta.value; count.textContent = ta.value.length + ' / ' + max;
            if (!S.answers) return;
            clearTimeout(similarTimer);
            similarTimer = setTimeout(function () {
                if (ta.value.trim().length < 12) { P.similar = []; drawSimilar(); return; }
                ask('player', 'similar', { text: ta.value, category: F.category }).then(function (r) { P.similar = r.rows || []; drawSimilar(); });
            }, 700);
        } });
        body.appendChild(ta); body.appendChild(count); body.appendChild(similarBox); drawSimilar();

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
        var pinned = resolutionBlock(x); if (pinned) wrap.appendChild(pinned);
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
        var waiting = S.staff.tickets.filter(function (x) { return x.status === 'open'; }).length;
        var tabs = [{ id: 'tickets', label: t('sp_tab_tickets'), count: waiting }];
        tabs.push({ id: 'chat', label: t('sp_tab_chat'), count: chatUnreadTotal() });
        if (S.me.powers.warn || S.me.powers.kick || S.me.powers.ban) tabs.push({ id: 'players', label: t('sp_tab_players') });
        if (S.me.powers.unban) tabs.push({ id: 'bans', label: t('sp_tab_bans') });
        if (S.me.powers.hire || S.me.powers.manage) tabs.push({ id: 'staff', label: t('sp_tab_staff') });
        if (S.me.powers.audit) tabs.push({ id: 'audit', label: t('sp_tab_audit') });
        if (S.web) tabs.push({ id: 'web', label: t('web_tab'), dot: !S.web.linked });
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
        var full = h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: t(S.full ? 'sp_windowed' : 'sp_fullscreen'), onclick: function () { S.full = !S.full; saveFull(S.full); renderStaff(); } });
        full.style.margin = '0 0.8rem 0 0';
        var extra = h('div', null, [back, full, duty]); extra.style.display = 'flex'; extra.style.alignItems = 'center';

        shell('staff', t('sp_title'), staffTabs(), A.tab, function (id) {
            A.tab = id;
            renderStaff();
            if (id === 'bans') loadBans(); else if (id === 'staff') loadStaffTab(); else if (id === 'audit') loadAudit(); else if (id === 'players') loadPlayers(); else if (id === 'chat') loadChat(); else if (id === 'web') watchWeb();
        }, extra);

        if (A.tab === 'tickets') renderTickets();
        else if (A.tab === 'chat') renderChat();
        else if (A.tab === 'players') renderPlayers();
        else if (A.tab === 'bans') renderBans();
        else if (A.tab === 'staff') renderStaffTab();
        else if (A.tab === 'web' && S.web) { var wp = h('div.pt-page'); renderWebMine(wp); win.appendChild(wp); }
        else renderAudit();
    }

    function loadTickets() {
        return ask('staff', 'list', { closed: false }).then(function (r) {
            S.staff.tickets = (r.tickets || []).map(markMine); S.staff.helps = r.helps || [];
            if (S.view === 'staff' && S.staff.tab === 'tickets') renderStaff();
        });
    }

    function visibleTickets() {
        var A = S.staff, src = A.status === 'closed' ? A.closed : (A.status === 'archived' ? A.archived : A.tickets), q = A.q.trim().toLowerCase();
        var rank = {}; S.priorities.forEach(function (p, i) { rank[p] = i; });
        var out = src.filter(function (x) {
            if (A.status === 'open' && x.status !== 'open') return false;
            if (A.status === 'claimed' && x.status !== 'claimed') return false;
            if (A.status === 'mine' && !(x.status === 'claimed' && x.mine)) return false;
            if (A.status === 'closed' && x.status !== 'closed') return false;
            if (A.status === 'archived' && !x.archivedAt) return false;
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
            if (A.status === 'archived') return (b.archivedAt || 0) - (a.archivedAt || 0);
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
        // The bubbles: what is still waiting behind each chip. Closed tickets never count.
        function tally(test) { return A.tickets.filter(test).length; }
        var states = [
            { value: 'open', label: t('sp_filter_open'), color: STATUS_COLORS.open, count: tally(function (x) { return x.status === 'open'; }) },
            { value: 'claimed', label: t('sp_filter_claimed'), color: STATUS_COLORS.claimed, count: tally(function (x) { return x.status === 'claimed'; }) },
            { value: 'mine', label: t('sp_filter_mine'), color: '#b9a8f0', count: tally(function (x) { return x.status === 'claimed' && x.mine; }) },
            { value: 'closed', label: t('sp_filter_closed'), color: STATUS_COLORS.closed }];
        if (S.me.powers.archive) states.push({ value: 'archived', label: t('sp_filter_archived'), color: '#9aa8ba' });
        filters.appendChild(chips(states, A.status, function (v) {
            A.status = v;
            if (v === 'closed') ask('staff', 'list', { mode: 'closed' }).then(function (r) { A.closed = (r.tickets || []).map(markMine); renderStaff(); });
            else if (v === 'archived') ask('staff', 'list', { mode: 'archived' }).then(function (r) { A.archived = (r.tickets || []).map(markMine); renderStaff(); });
            else renderStaff();
        }, true));
        filters.appendChild(h('div.pt-filters__label', { text: t('sp_filter_kind') }));
        filters.appendChild(chips(categoryOpts().map(function (o) { o.count = tally(function (x) { return x.category === o.value && x.status === 'open'; }); return o; }),
            A.category, function (v) { A.category = A.category === v ? null : v; renderStaff(); }, true));
        filters.appendChild(h('div.pt-filters__label', { text: t('sp_filter_priority') }));
        filters.appendChild(chips(priorityOpts().map(function (o) { o.count = tally(function (x) { return x.priority === o.value && x.status === 'open'; }); return o; }),
            A.priority, function (v) { A.priority = A.priority === v ? null : v; renderStaff(); }, true));
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
                        h('div.pt-row__top', null, [h('span.pt-row__id', { text: '#' + x.id }), pill(x.categoryLabel, catColor(x.category)), pill(t('priority_' + x.priority), PRI_COLORS[x.priority]),
                            (x.escalatedTo && x.escalatedTo.length) ? pill('↑ ' + x.escalatedTo.map(roleLabel).join(', '), roleColor(x.escalatedTo[0])) : null]),
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
        A.curId = id; A.detail = null; A.draft = ''; A.note = false;
        renderStaff();
        if (!S.canned.length && S.me.powers.reply) ask('staff', 'canned').then(function (r) { S.canned = r.rows || []; if (A.curId === id && S.view === 'staff') renderStaff(); });
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
        (x.escalatedTo || []).forEach(function (rid) {
            var el = pill('↑ ' + roleLabel(rid) + (open && P.escalate ? '  ×' : ''), roleColor(rid));
            if (open && P.escalate) { el.style.cursor = 'pointer'; el.addEventListener('click', function () { act('unescalate', { id: x.id, role: rid }); }); }
            pills.appendChild(el);
        });
        if (x.isPublic) pills.appendChild(pill(t('sp_public'), '#8fd3a8'));
        if (x.archivedAt) pills.appendChild(pill(t('sp_filter_archived') + (x.archivedByName ? ' · ' + x.archivedByName : ''), '#9aa8ba'));
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
        var onMark = P.resolve ? function (index, marked) { act('resolve', { id: x.id, index: marked ? 0 : index }).then(function (r) { if (!r.ok) toast(r.message); }); } : null;
        var chat = h('div.pt-d-chat', null, [h('div.pt-label', { text: t('sp_conversation') }), resolutionBlock(x), conversation(x, true, onMark)]);
        if (open && P.reply) {
            var input = h('input.pt-input' + (A.note ? '.is-note' : ''), { type: 'text', maxlength: S.limits.message || 500, placeholder: A.note ? t('sp_internal_hint') : t('ui_reply'), value: A.draft, oninput: function () { A.draft = input.value; } });
            var go = function () {
                if (!input.value.trim()) return;
                var text = input.value; input.value = ''; A.draft = '';
                act(A.note ? 'note' : 'reply', { id: x.id, text: text }).then(function (r) { if (!r.ok) toast(r.message); });
            };
            input.addEventListener('keydown', function (e) { if (e.key === 'Enter') go(); });

            // Ready-made replies: the ones whose keywords the player used come first.
            var fill = function (c) {
                A.draft = c.body.replace(/\{player\}/g, (x.reporterName || '').split(' ')[0]).replace(/\{staff\}/g, S.me.name || '').replace(/\{id\}/g, String(x.id));
                A.note = false; renderStaff();
            };
            var offered = suggestCanned(x);
            if (S.canned.length && !A.note) {
                var cannedRow = h('div.pt-canned');
                offered.forEach(function (c) { cannedRow.appendChild(h('button.pt-chip.pt-chip--small', { type: 'button', text: c.label, title: c.body, onclick: function () { fill(c); } })); });
                cannedRow.appendChild(h('button.pt-chip.pt-chip--small.pt-chip--more', { type: 'button', text: t('sp_replies') + ' …', onclick: function () {
                    dialog(t('sp_replies_all'), function (body) {
                        S.canned.forEach(function (c) {
                            body.appendChild(h('div.pt-result', { onclick: function () { var open2 = win.querySelector('.pt-dialog-shade'); if (open2) open2.parentNode.removeChild(open2); fill(c); } },
                                [h('b', { text: c.label }), h('div.pt-hint', { text: c.body })]));
                        });
                    }, t('sp_cancel'), function () { return true; });
                } }));
                chat.appendChild(cannedRow);
            }
            var noteToggle = h('div.pt-toggle.pt-toggle--note' + (A.note ? '.is-on' : ''), { onclick: function () { A.note = !A.note; renderStaff(); } }, [h('div.pt-toggle__box'), h('div', { text: t('sp_internal') })]);
            noteToggle.style.marginTop = '0';
            chat.appendChild(h('div.pt-replyrow', null, [noteToggle, input, h('button.pt-btn' + (A.note ? '.pt-btn--note' : '.pt-btn--act'), { type: 'button', text: t('ui_send'), onclick: go })]));
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
            if (P.escalate) row1.appendChild(btn(t('sp_escalate'), '.pt-btn--note', function () {
                var pick = null, why;
                var options = roleOpts().filter(function (o) { return (x.escalatedTo || []).indexOf(o.value) === -1; });
                dialog(t('sp_escalate') + ' · #' + x.id, function (body) {
                    var box = h('div');
                    var draw = function () { clear(box); box.appendChild(chips(options, pick, function (v) { pick = v; draw(); })); };
                    body.appendChild(h('div.pt-label', { text: t('sp_escalate_to') })); body.appendChild(box); draw();
                    body.appendChild(h('div.pt-label', { text: t('sp_escalate_note') }));
                    why = h('textarea.pt-text.pt-text--short.is-note', { maxlength: S.limits.message || 500 }); body.appendChild(why);
                }, t('sp_escalate'), function () { return pick ? act('escalate', { id: x.id, role: pick, note: why.value }).then(function (r) { if (r.ok && r.message) toast(r.message); return r; }) : false; });
            }));
        }
        // Public, archive and delete work on a closed ticket too.
        if (P.resolve && x.publishable && x.status === 'closed' && x.resolution && !x.archivedAt) {
            if (x.isPublic) row1.appendChild(btn(t('sp_unpublish'), '', simple('setPublic', { id: x.id, on: false })));
            else row1.appendChild(btn(t('sp_publish'), '', function () {
                dialog(t('sp_publish') + ' · #' + x.id, function (body) { body.appendChild(h('div.pt-hint', { text: t('sp_publish_warn') })); },
                    t('sp_publish'), function () { return act('setPublic', { id: x.id, on: true }).then(function (r) { if (r.message) toast(r.message); return r; }); });
            }));
        }
        if (P.archive) {
            if (x.archivedAt) row1.appendChild(btn(t('sp_unarchive'), '', function () { ask('staff', 'unarchive', { id: x.id }).then(function (r) { toast(r.message); }); }));
            else row1.appendChild(btn(t('sp_archive'), '.pt-btn--ghost', function () {
                dialog(t('sp_archive') + ' · #' + x.id, function (body) { body.appendChild(h('div.pt-hint', { text: t('sp_archive_warn') })); },
                    t('sp_archive'), function () { return ask('staff', 'archive', { id: x.id }).then(function (r) { if (r.message) toast(r.message); return r; }); });
            }));
        }
        if (S.me.canDelete && x.archivedAt) row1.appendChild(btn(t('sp_delete'), '.pt-btn--danger', function () {
            dialog(t('sp_delete') + ' · #' + x.id, function (body) { body.appendChild(h('div.pt-hint', { text: t('sp_delete_warn') })); },
                t('sp_delete'), function () { return ask('staff', 'delete', { id: x.id }).then(function (r) { if (r.message) toast(r.message); return r; }); }, true);
        }));
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
    /** "ban:7d" -> "a ban (7 days)". */
    function ladderText(action) {
        var m = /^ban:(.+)$/.exec(action || '');
        if (m) return t('lad_ban', t('sp_len_' + m[1]));
        return t(action === 'kick' ? 'lad_kick' : 'lad_warn');
    }

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
                    h('div.pt-row__top', null, [p.src ? h('span.pt-row__id', { text: String(p.src) }) : null, h('span.pt-name' + (p.online === false ? '.is-off' : ''), { text: p.name }), p.staff ? pill(t('pl_staff'), '#b9a8f0') : null,
                        p.watched ? pill(t('pl_watching'), '#e6a8d7') : null]),
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
                pill(t('pl_warnings') + ' ' + c.warnings.length, c.warnings.length ? '#f2b880' : null), pill(t('pl_bans') + ' ' + c.bans.length, c.bans.length ? '#f08a8a' : null),
                c.watch ? pill(t('pl_watching') + (c.watch.reason ? ' · ' + c.watch.reason : ''), '#e6a8d7') : null])]));
        if (c.suggest) top.appendChild(h('div.pt-hint.pt-ladder', { text: t('pl_suggest', c.suggest.offence, ladderText(c.suggest.action)) }));
        right.appendChild(top);

        var rec = h('div.pt-record');
        function section(title, rows, draw) {
            rec.appendChild(h('div.pt-label', { text: title + ' · ' + rows.length }));
            if (!rows.length) rec.appendChild(h('div.pt-hint', { text: t('pl_clean') }));
            rows.forEach(function (r) { rec.appendChild(draw(r)); });
        }
        section(t('pl_warnings'), c.warnings, function (w) {
            return h('div.pt-rec' + (w.decayed ? '.is-dim' : ''), null, [h('div.pt-rec__when', { text: clock(w.created_at) }), h('div.pt-rec__text', null, [w.reason,
                h('span', { text: '  —  ' + (w.warned_by_name || '?') + (w.acknowledged_at ? '' : ' · ' + t('pl_unread')) + (w.decayed ? ' · ' + t('pl_decayed') : '') })])]);
        });
        if (c.canNote) {
            rec.appendChild(h('div.pt-label', { text: t('pl_notes') + ' · ' + (c.notes || []).length }));
            (c.notes || []).forEach(function (n) {
                rec.appendChild(h('div.pt-rec.is-note', null, [h('div.pt-rec__when', { text: clock(n.created_at) }), h('div.pt-rec__text', null, [n.note, h('span', { text: '  —  ' + (n.by_name || '?') })]),
                    h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: '×', onclick: function () { ask('staff', 'noteDelete', { id: n.id }).then(function () { openPlayer(who); }); } })]));
            });
            var noteIn = h('input.pt-input.is-note', { type: 'text', maxlength: S.limits.note || 500, placeholder: t('pl_note_hint') });
            var addNote = function () { if (!noteIn.value.trim()) return; ask('staff', 'noteAdd', { player: who, note: noteIn.value }).then(function (r) { if (!r.ok) toast(r.message); openPlayer(who); }); };
            noteIn.addEventListener('keydown', function (e) { if (e.key === 'Enter') addNote(); });
            rec.appendChild(h('div.pt-replyrow', null, [noteIn, h('button.pt-btn.pt-btn--note', { type: 'button', text: t('pl_note_add'), onclick: addNote })]));
        }
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
        if (c.canNote) {
            if (c.watch) row.appendChild(btn(t('pl_unwatch'), '.pt-btn--ghost', function () { ask('staff', 'watch', { player: who, on: false }).then(function () { loadPlayers(); openPlayer(who); }); }));
            else row.appendChild(btn(t('pl_watch'), '.pt-btn--ghost', function () {
                var why;
                dialog(t('pl_watch') + ' · ' + (c.name || c.account), function (body) { body.appendChild(h('div.pt-label', { text: t('pl_watch_reason') })); why = h('textarea.pt-text.pt-text--short.is-note', { maxlength: 300 }); body.appendChild(why); },
                    t('pl_watch'), function () { return ask('staff', 'watch', { player: who, on: true, reason: why.value }).then(function (r) { if (r.ok) { toast(r.message); loadPlayers(); openPlayer(who); } return r; }); });
            }));
        }
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
    function hireRoles() { return S.roleDefs.map(function (r) { return r.id; }); }
    function loadStaffTab() {
        var A = S.staff;
        if (A.sub === 'roster' && !S.me.powers.hire) A.sub = 'roles';
        if (A.sub !== 'roster' && !S.me.powers.manage) A.sub = 'roster';
        if (A.sub === 'roster') loadStaff(); else if (A.sub === 'roles') loadRoles(); else if (A.sub === 'web') loadWeb(); else loadCannedAdmin();
    }
    function renderStaffTab() {
        var A = S.staff;
        var page = h('div.pt-page');
        var subs = [];
        if (S.me.powers.hire) subs.push({ value: 'roster', label: t('sp_sub_roster') });
        if (S.me.powers.manage) { subs.push({ value: 'roles', label: t('sp_sub_roles') }); subs.push({ value: 'replies', label: t('sp_sub_replies') }); subs.push({ value: 'web', label: t('sp_sub_web') }); }
        if (!subs.some(function (o) { return o.value === A.sub; })) A.sub = subs.length ? subs[0].value : 'roster';
        if (subs.length > 1) {
            var sub = chips(subs, A.sub, function (v) { A.sub = v; renderStaff(); loadStaffTab(); });
            sub.style.marginBottom = '1rem';
            page.appendChild(sub);
        }
        if (A.sub === 'roles') { renderRoles(page); win.appendChild(page); return; }
        if (A.sub === 'replies') { renderCannedAdmin(page); win.appendChild(page); return; }
        if (A.sub === 'web') { renderWeb(page); win.appendChild(page); return; }
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
                row.appendChild(h('button.pt-chip' + (roster.roles[role] ? '.is-on' : ''), { type: 'button', text: roleLabel(role), onclick: function () { roster.roles[role] = !roster.roles[role]; renderStaff(); } }));
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
            if (m.framework) rolesCell.appendChild(h('span', { text: roleLabel('admin') + ' (' + t('sp_framework_admin') + ')' }));
            else {
                var chipsRow = h('div.pt-chips');
                hireRoles().forEach(function (role) {
                    var on = m.roles.indexOf(role) !== -1;
                    chipsRow.appendChild(h('button.pt-chip.pt-chip--small' + (on ? '.is-on' : ''), { type: 'button', text: roleLabel(role), onclick: function () {
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
        note: 'tickets', escalate: 'tickets', unescalate: 'tickets', resolve: 'tickets', publish: 'tickets', unpublish: 'tickets', autoclose: 'tickets',
        archive: 'tickets', unarchive: 'tickets', 'delete': 'tickets',
        warn: 'moderation', kick: 'moderation', ban: 'moderation', unban: 'moderation', pnote: 'moderation', pnote_delete: 'moderation', watch: 'moderation', unwatch: 'moderation',
        role_save: 'staff', role_delete: 'staff', chat_save: 'staff', chat_delete: 'staff', canned_save: 'staff', canned_delete: 'staff',
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

    // ------------------------------------------------- ready-made replies (1.1.0)

    /** Up to three replies whose keywords appear in what the player wrote. Plain word matching. */
    function suggestCanned(x) {
        var said = (x.description || '');
        (x.conversation || []).forEach(function (m) { if (!m.staff) said += ' ' + m.text; });
        said = ' ' + said.toLowerCase() + ' ';
        var scored = [];
        S.canned.forEach(function (c) {
            var n = 0;
            (c.keywords || '').split(',').forEach(function (k) { k = k.trim().toLowerCase(); if (k && said.indexOf(k) !== -1) n++; });
            if (n > 0) scored.push({ n: n, c: c });
        });
        scored.sort(function (a, b) { return b.n - a.n; });
        return scored.slice(0, 3).map(function (k) { return k.c; });
    }

    function field(body, label, el, hint) {
        body.appendChild(h('div.pt-label', { text: label }));
        if (hint) body.appendChild(h('div.pt-hint', { text: hint }));
        body.appendChild(el);
        return el;
    }

    function loadCannedAdmin() { ask('staff', 'canned').then(function (r) { S.canned = r.rows || []; if (S.view === 'staff' && S.staff.tab === 'staff') renderStaff(); }); }
    function cannedDialog(c) {
        var label, text, keys;
        dialog(t(c ? 'ca_edit' : 'ca_new'), function (body) {
            label = field(body, t('ca_label'), h('input.pt-input', { type: 'text', maxlength: 64, value: c ? c.label : '' }));
            text = field(body, t('ca_body'), h('textarea.pt-text.pt-text--short', { maxlength: 500, value: c ? c.body : '' }), t('ca_body_hint'));
            keys = field(body, t('ca_keywords'), h('input.pt-input', { type: 'text', maxlength: 300, value: c ? c.keywords : '' }));
        }, t('sp_save'), function () {
            return ask('staff', 'cannedSave', { id: c ? c.id : null, label: label.value, body: text.value, keywords: keys.value }).then(function (r) { if (r.ok) loadCannedAdmin(); return r; });
        });
    }
    function renderCannedAdmin(page) {
        page.appendChild(h('div.pt-page__bar', null, [h('div.pt-hint', { text: t('ca_body_hint') }), h('button.pt-btn.pt-btn--go', { type: 'button', text: t('ca_new'), onclick: function () { cannedDialog(null); } })]));
        if (!S.canned.length) { page.appendChild(h('div.pt-empty', { text: t('ca_none') })); return; }
        var table = h('table.pt-table', null, [h('tr', null, [t('ca_label'), t('ca_body'), t('ca_keywords'), ''].map(function (x) { return h('th', { text: x }); }))]);
        S.canned.forEach(function (c) {
            table.appendChild(h('tr', null, [h('td', null, [h('b', { text: c.label })]), h('td', { text: c.body }), h('td.pt-mono', { text: c.keywords || '' }),
                h('td', null, [h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_edit'), onclick: function () { cannedDialog(c); } }),
                    h('button.pt-btn.pt-btn--small.pt-btn--danger', { type: 'button', text: t('sp_delete_short'), onclick: function () { ask('staff', 'cannedDelete', { id: c.id }).then(loadCannedAdmin); } })])]));
        });
        page.appendChild(table);
    }

    // --------------------------------------------------------- the web panel (1.1.0)
    // Only its status: it is switched on in /poggy, because turning it on sends ticket text off the server.

    var webInfo = null;
    function loadWeb() { ask('staff', 'web').then(function (r) { webInfo = r.web || null; if (S.view === 'staff' && S.staff.tab === 'staff') renderStaff(); }); }
    // ------------------------------------------------ the Website tab (everyone)
    //
    // One community ID for the whole server. A player whose game named a Cfx.re
    // account is linked already; anyone else gets a link code here and confirms it
    // on the website while signed in with Cfx.re.
    var webMine = { code: null, until: 0, msg: null, busy: false };
    function onWebTab() { return (S.view === 'player' && S.player.tab === 'web') || (S.view === 'staff' && S.staff.tab === 'web'); }
    function redrawWeb() { if (S.view === 'player') renderPlayer(); else if (S.view === 'staff') renderStaff(); }
    function takeWeb(w) {
        var was = S.web && S.web.linked;
        S.web = (w && w.code) ? w : null;
        if (S.web && S.web.linked && !was) { webMine.code = null; webMine.msg = null; }
    }
    var webTimer = null;
    function watchWeb() {
        if (webTimer) return;
        webTimer = setInterval(function () {
            if (!onWebTab()) { clearInterval(webTimer); webTimer = null; return; }
            if (webMine.code && Date.now() > webMine.until) { webMine.code = null; redrawWeb(); return; }
            if (S.web && S.web.linked) return;
            ask('player', 'web').then(function (r) {
                var was = S.web && S.web.linked;
                if (r && r.ok) takeWeb(r.web);
                if (onWebTab() && S.web && S.web.linked !== was) redrawWeb();
            });
        }, 3000);
    }
    function renderWebMine(page) {
        var w = S.web;
        if (!w) { page.appendChild(h('div.pt-hint', { text: t('web_off_short') })); return; }
        page.appendChild(h('div.pt-web__intro', { text: t('web_intro') }));
        if (w.linked) {
            page.appendChild(h('div.pt-web__state.is-on', { text: '\u2713 ' + t('web_you_linked', w.cfx || '') }));
            page.appendChild(h('div.pt-hint', { text: t('web_you_linked_how', w.site) }));
        } else {
            page.appendChild(h('div.pt-web__state', { text: t('web_not_linked') + (w.why && w.why !== 'unlinked' ? ' ' + t('web_why_' + w.why) : '') }));
        }
        var copy = h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_copy'), onclick: function () { copyText(w.code); copy.textContent = t('sp_copied'); } });
        var steps = h('div.pt-web__steps');
        // A game window cannot open a browser, so the address is there to be copied.
        function copyBtn(text) { var b = h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_copy'), onclick: function () { copyText(text); b.textContent = t('sp_copied'); } }); return b; }
        var words = t('web_step1', '').split('');
        steps.appendChild(h('div.pt-web__step', null, [h('span.pt-web__n', { text: '1' }), h('div', null, [
            h('div', null, [words[0] || '', h('span.pt-web__url', { text: w.site, title: t('sp_copy'), onclick: function () { copyText('https://' + w.site); } }), words[1] || '']),
            h('div.pt-page__bar', null, [h('div.pt-web__urlbig', { text: 'https://' + w.site }), copyBtn('https://' + w.site)])])]));
        steps.appendChild(h('div.pt-web__step', null, [h('span.pt-web__n', { text: '2' }), h('div', null, [h('div', { text: t('web_step2') }),
            h('div.pt-page__bar', null, [h('div.pt-webcode', { text: w.code }), copy])])]));
        // Not linked: the code links them. Linked STAFF: the same code confirms a new browser for the staff desk.
        if (!w.linked || w.devices) {
            var three = h('div', null, [h('div', { text: t(w.linked ? 'web_step3_device' : 'web_step3') })]);
            if (webMine.code) {
                var left = Math.max(1, Math.ceil((webMine.until - Date.now()) / 60000));
                var newCode = h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: t('web_new_code'), onclick: getWebCode });
                newCode.style.marginLeft = '0.5rem';
                three.appendChild(h('div.pt-page__bar', null, [h('div.pt-webcode.pt-webcode--pair', { text: webMine.code }), copyBtn(webMine.code), newCode]));
                three.appendChild(h('div.pt-hint', { text: t('web_code_for', left) }));
                three.appendChild(h('div.pt-msg.is-bad', { text: t('web_code_warn') }));
            } else {
                var get = h('button.pt-btn.pt-btn--go', { type: 'button', text: t('web_get_code'), onclick: getWebCode });
                get.style.marginTop = '0.5rem';
                three.appendChild(get);
            }
            steps.appendChild(h('div.pt-web__step', null, [h('span.pt-web__n', { text: '3' }), three]));
        }
        page.appendChild(steps);
        if (webMine.msg) page.appendChild(h('div.pt-msg.is-bad', { text: webMine.msg }));
        if (w.linked) {
            var un = h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: t('web_unlink'), onclick: function () {
                dialog(t('web_unlink'), function (body) { body.appendChild(h('div.pt-hint', { text: t('web_unlink_ask') })); }, t('web_unlink'), function () {
                    return ask('player', 'webUnlink').then(function (r) { if (r && r.ok) { takeWeb(r.web); redrawWeb(); } return r; });
                }, true);
            } });
            un.style.marginTop = '1.2rem';
            page.appendChild(un);
            if (w.devices) {
                var out = h('button.pt-btn.pt-btn--small.pt-btn--ghost', { type: 'button', text: t('web_signout'), onclick: function () {
                    dialog(t('web_signout'), function (body) { body.appendChild(h('div.pt-hint', { text: t('web_signout_ask') })); }, t('web_signout'), function () {
                        return ask('player', 'webSignOut').then(function (r) { if (r && r.ok) toast(t('web_signout_done')); return r; });
                    }, true);
                } });
                out.style.margin = '1.2rem 0 0 0.6rem';
                page.appendChild(out);
            }
        }
    }
    function getWebCode() {
        if (webMine.busy) return;
        webMine.busy = true;
        ask('player', 'webCode').then(function (r) {
            webMine.busy = false;
            if (r && r.ok) { webMine.code = r.code; webMine.until = Date.now() + (r.seconds || 600) * 1000; webMine.msg = null; takeWeb(r.web); watchWeb(); }
            else webMine.msg = (r && r.message) || '…';
            redrawWeb();
        });
    }

    function renderWeb(page) {
        var w = webInfo;
        if (!w) { page.appendChild(h('div.pt-empty', { text: '…' })); return; }
        if (!w.enabled) { page.appendChild(h('div.pt-hint', { text: t('web_off') })); page.appendChild(h('div.pt-hint', { text: t('web_cannot') })); return; }
        page.appendChild(h('div.pt-label', { text: t('web_id') }));
        var code = h('div.pt-webcode', { text: w.code || '…' });
        var copy = h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_copy'), onclick: function () { copyText(w.code || ''); copy.textContent = t('sp_copied'); } });
        page.appendChild(h('div.pt-page__bar', null, [code, copy]));
        page.appendChild(h('div.pt-hint', { text: t('web_site') + ' ' + w.site }));
        page.appendChild(h('div.pt-hint', { text: t('web_how') }));
        page.appendChild(h('div.pt-hint', { text: t('web_cannot') }));
        var facts = h('div.pt-facts');
        facts.appendChild(h('div.pt-fact', null, [h('div.pt-fact__k', { text: t('sp_tab_staff') }), h('div.pt-fact__v', { text: t('web_linked', w.linked || 0, w.players || 0) })]));
        facts.appendChild(h('div.pt-fact', null, [h('div.pt-fact__k', { text: 'Sync' }), h('div.pt-fact__v', { text: w.lastSync ? t('web_synced', age(w.lastSync)) : t('web_never') })]));
        facts.appendChild(h('div.pt-fact.pt-fact--wide', null, [h('div.pt-fact__k', { text: t('kind_appeal') }), h('div.pt-fact__v', { text: w.appeals ? t('web_appeals_on', w.publicName || '…') : t('web_appeals_off') })]));
        page.appendChild(facts);
        if (w.error) page.appendChild(h('div.pt-msg.is-bad', { text: w.error }));
    }

    // ------------------------------------------------------------- roles (1.1.0)

    var ROLE_COLORS = ['#f08a8a', '#f2b880', '#e3c98a', '#8fd3a8', '#7fd1c7', '#8fb8f0', '#b9a8f0', '#e6a8d7', '#a9b4c4'];
    var roleAdmin = { rows: [] };
    function loadRoles() { ask('staff', 'roles').then(function (r) { roleAdmin.rows = r.roles || []; if (S.view === 'staff' && S.staff.tab === 'staff') renderStaff(); }); }

    /** A webhook is only ever written: the page is told that one is set, never what it is. */
    function webhookField(body, has) {
        var state = { remove: false };
        state.input = field(body, t('ch_webhook'), h('input.pt-input', { type: 'text', maxlength: 300, placeholder: 'https://discord.com/api/webhooks/…' }), t('ch_webhook_hint') + (has ? ' ' + t('ch_webhook_set') : ''));
        if (has) {
            var tg = h('div.pt-toggle', { onclick: function () { state.remove = !state.remove; tg.classList.toggle('is-on', state.remove); } }, [h('div.pt-toggle__box'), h('div', { text: t('ch_webhook_remove') })]);
            body.appendChild(tg);
        }
        state.value = function () { if (state.remove) return ''; return state.input.value.trim() ? state.input.value.trim() : undefined; };
        return state;
    }
    function setOf(list) { var o = {}; (list || []).forEach(function (k) { o[k] = true; }); return o; }
    function listOf(set) { return Object.keys(set).filter(function (k) { return set[k]; }); }

    function roleDialog(r) {
        var name, rank, hook, color = r ? r.color : ROLE_COLORS[5];
        var powers = setOf(r ? r.powers : ['claim', 'reply', 'close', 'teleport']), kinds = setOf(r ? r.kinds : []), chats = setOf(r ? r.chats : []);
        var helps = r ? !!r.helps : false, locked = !!(r && r.locked);
        dialog(t(r ? 'ro_edit' : 'ro_new') + (r ? ' · ' + r.label : ''), function (body) {
            name = field(body, t('ro_name'), h('input.pt-input', { type: 'text', maxlength: 64, value: r ? r.label : '' }));
            var colorBox = h('div');
            var drawColor = function () { clear(colorBox); colorBox.appendChild(chips(ROLE_COLORS.map(function (c) { return { value: c, label: ' ', color: c }; }), color, function (v) { color = v; drawColor(); }, true)); };
            body.appendChild(h('div.pt-label', { text: t('ro_color') })); body.appendChild(colorBox); drawColor();
            if (locked) { body.appendChild(h('div.pt-hint', { text: t('ro_locked') })); hook = webhookField(body, r.hasWebhook); return; }
            rank = field(body, t('ro_rank'), h('input.pt-input', { type: 'text', maxlength: 2, value: String(r ? r.rank : 10) }));
            body.appendChild(h('div.pt-label', { text: t('ro_powers') }));
            body.appendChild(multiChips(S.powerList.map(function (pw) { return { value: pw, label: t('pw_' + pw) }; }), powers, null, true));
            body.appendChild(h('div.pt-label', { text: t('ro_kinds') }));
            body.appendChild(multiChips(categoryOpts(), kinds, null, true));
            body.appendChild(h('div.pt-label', { text: t('ro_chats') }));
            body.appendChild(multiChips(roleOpts().filter(function (o) { return !r || o.value !== r.id; }), chats, null, true));
            var ht = h('div.pt-toggle' + (helps ? '.is-on' : ''), { onclick: function () { helps = !helps; ht.classList.toggle('is-on', helps); } }, [h('div.pt-toggle__box'), h('div', { text: t('ro_helps') })]);
            body.appendChild(ht);
            hook = webhookField(body, r && r.hasWebhook);
        }, t('sp_save'), function () {
            var payload = { id: r ? r.id : null, label: name.value, color: color, webhook: hook.value() };
            if (!locked) { payload.rank = rank.value; payload.powers = listOf(powers); payload.kinds = listOf(kinds); payload.chats = listOf(chats); payload.helps = helps; }
            return ask('staff', 'roleSave', payload).then(function (res) { if (res.ok) { loadRoles(); post('ask', { channel: 'player', action: 'hello' }); } return res; });
        });
        widenDialog();
    }
    function widenDialog() { var d = win.querySelector('.pt-dialog'); if (d) d.classList.add('pt-dialog--wide'); }
    function renderRoles(page) {
        page.appendChild(h('div.pt-page__bar', null, [h('div.pt-hint', { text: t('ro_locked') }), h('button.pt-btn.pt-btn--go', { type: 'button', text: t('ro_new'), onclick: function () { roleDialog(null); } })]));
        var table = h('table.pt-table', null, [h('tr', null, [t('ro_name'), t('ro_powers'), t('ro_kinds'), ''].map(function (x) { return h('th', { text: x }); }))]);
        roleAdmin.rows.forEach(function (r) {
            var every = r.powers.indexOf('*') !== -1;
            var does = h('td'), sees = h('td');
            if (every) does.appendChild(pill(t('ro_everything'), r.color));
            else r.powers.forEach(function (pw) { does.appendChild(pill(t('pw_' + pw).split(',')[0], null)); });
            if (r.kinds.indexOf('*') !== -1) sees.appendChild(pill(t('ro_everything'), r.color));
            else r.kinds.forEach(function (k) { var c = categoryOf(k); sees.appendChild(pill(c ? c.label : k, catColor(k))); });
            table.appendChild(h('tr', null, [h('td', null, [pill(r.label, r.color), h('small.pt-mono', { text: '  ' + r.rank })]), does, sees,
                h('td', null, [h('button.pt-btn.pt-btn--small', { type: 'button', text: t('sp_edit'), onclick: function () { roleDialog(r); } }),
                    r.locked ? null : h('button.pt-btn.pt-btn--small.pt-btn--danger', { type: 'button', text: t('sp_delete_short'), onclick: function () {
                        dialog(t('ro_delete') + ' · ' + r.label, function (body) { body.appendChild(h('div.pt-hint', { text: t('ro_delete_warn') })); }, t('ro_delete'),
                            function () { return ask('staff', 'roleDelete', { id: r.id }).then(function (res) { if (res.ok) loadRoles(); return res; }); }, true);
                    } })])]));
        });
        page.appendChild(table);
    }

    // -------------------------------------------------------- staff chat (1.1.0)
    // Rooms by role, plus the ones an admin made. Nothing here is ever saved.

    var chat = { rooms: [], cur: null, logs: {}, unread: {}, draft: '' };
    function chatUnreadTotal() { var n = 0; Object.keys(chat.unread).forEach(function (k) { n += chat.unread[k] || 0; }); return n; }
    function loadChat() {
        return ask('staff', 'chatRooms').then(function (r) {
            chat.rooms = r.rooms || [];
            if (chat.cur && !chat.rooms.some(function (x) { return x.key === chat.cur; })) chat.cur = null;
            // Only open a room when the tab is showing: opening one marks it read.
            if (S.staff.tab === 'chat' && !chat.cur && chat.rooms.length) openRoom(chat.rooms[0].key); else if (S.view === 'staff') renderStaff();
        });
    }
    function openRoom(key) {
        chat.cur = key; chat.unread[key] = 0;
        if (S.view === 'staff' && S.staff.tab === 'chat') renderStaff();
        ask('staff', 'chatHistory', { key: key }).then(function (r) { if (r.ok) { chat.logs[key] = r.messages || []; if (chat.cur === key && S.view === 'staff' && S.staff.tab === 'chat') renderStaff(); } });
    }
    function roomDialog(room) {
        var name, hook, roles = setOf(room ? room.roles : []);
        dialog(t(room ? 'ch_edit_room' : 'ch_new_room'), function (body) {
            name = field(body, t('ch_room_name'), h('input.pt-input', { type: 'text', maxlength: 64, value: room ? room.label : '' }));
            body.appendChild(h('div.pt-label', { text: t('ch_room_roles') }));
            body.appendChild(multiChips(roleOpts(), roles, null, true));
            hook = webhookField(body, room && room.hasWebhook);
        }, t('sp_save'), function () {
            return ask('staff', 'chatSaveRoom', { id: room ? Number(room.key.split(':')[1]) : null, label: name.value, roles: listOf(roles), webhook: hook.value() }).then(function (r) { if (r.ok) loadChat(); return r; });
        });
    }
    function renderChat() {
        var left = h('div.pt-left'), right = h('div.pt-right');
        win.appendChild(h('div.pt-staff', null, [left, right]));
        var listEl = h('div.pt-list');
        if (S.me.powers.manage) {
            var bar = h('div.pt-filters'); bar.appendChild(h('button.pt-btn.pt-btn--go', { type: 'button', text: t('ch_new_room'), onclick: function () { roomDialog(null); } }));
            left.appendChild(bar);
        }
        left.appendChild(listEl);
        chat.rooms.forEach(function (room) {
            var n = chat.unread[room.key] || 0;
            var row = h('div.pt-row' + (room.key === chat.cur ? '.is-cur' : ''), { onclick: function () { openRoom(room.key); } }, [
                h('div.pt-row__main', null, [h('div.pt-row__top', null, [h('span.pt-room', { text: room.label }), room.hasWebhook ? pill('Discord', '#8fb8f0') : null])]),
                h('div.pt-row__side', null, [n ? h('span.pt-chip__n', { text: String(n) }) : null])]);
            row.style.borderLeftColor = room.color || '#a9b4c4';
            listEl.appendChild(row);
        });

        var room = chat.rooms.filter(function (x) { return x.key === chat.cur; })[0];
        if (!room) { right.appendChild(h('div.pt-empty', { text: t('ch_pick') })); return; }
        var head = h('div.pt-d-top', null, [h('div.pt-d-head', null, [h('span.pt-d-head__cat', { text: room.label }), h('span.pt-d-head__pills')])]);
        head.appendChild(h('div.pt-hint', { text: t('ch_note') }));
        if (S.me.powers.manage) {
            if (room.kind === 'room') {
                var tools = h('div.pt-page__bar', null, [
                    h('button.pt-btn.pt-btn--small', { type: 'button', text: t('ch_edit_room'), onclick: function () { roomDialog(room); } }),
                    h('button.pt-btn.pt-btn--small.pt-btn--danger', { type: 'button', text: t('ch_delete_room'), onclick: function () { ask('staff', 'chatDeleteRoom', { id: Number(room.key.split(':')[1]) }).then(loadChat); } })]);
                head.appendChild(tools);
            } else head.appendChild(h('div.pt-hint', { text: t('ch_role_room') }));
        }
        right.appendChild(head);

        var log = chat.logs[room.key] || [];
        var box = h('div.pt-convo');
        if (!log.length) box.appendChild(h('div.pt-convo__empty', { text: t('ch_empty') }));
        log.forEach(function (m) {
            var me = m.account === S.me.account;
            box.appendChild(h('div.pt-bub.is-staff' + (me ? '.is-me' : ''), null, [h('div.pt-bub__who', null, [me ? t('ui_you') : m.name, h('time', { text: clock(m.at) })]), h('div.pt-bub__text', { text: m.text })]));
        });
        setTimeout(function () { box.scrollTop = box.scrollHeight; }, 0);
        var wrap = h('div.pt-d-chat', null, [box]);
        var input = h('input.pt-input', { type: 'text', maxlength: S.limits.chat || 500, placeholder: t('ch_placeholder', room.label), value: chat.draft, oninput: function () { chat.draft = input.value; } });
        var go = function () {
            if (!input.value.trim()) return;
            var text = input.value; input.value = ''; chat.draft = '';
            ask('staff', 'chatSend', { key: room.key, text: text }).then(function (r) { if (!r.ok) toast(r.message); });
        };
        input.addEventListener('keydown', function (e) { if (e.key === 'Enter') go(); });
        wrap.appendChild(h('div.pt-replyrow', null, [input, h('button.pt-btn.pt-btn--act', { type: 'button', text: t('ui_send'), onclick: go })]));
        right.appendChild(wrap);
        setTimeout(function () { input.focus(); }, 30);
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
        if (kind === 'chat') {
            // A room you are looking at needs no pop-up and no bubble.
            var watching = S.view === 'staff' && A.tab === 'chat' && chat.cur === d.key;
            if (chat.logs[d.key]) chat.logs[d.key].push(d.message);
            var mine = d.message && d.message.account === S.me.account;
            if (!watching && !mine) { chat.unread[d.key] = (chat.unread[d.key] || 0) + 1; if (d.toast) toast(d.toast, false); }
            if (S.view === 'staff') renderStaff();
            return;
        }
        if (kind === 'chatRooms') { if (S.view === 'staff') loadChat(); return; }
        if (kind === 'mineGone') {
            S.player.tickets = S.player.tickets.filter(function (k) { return k.id !== d.id; });
            if (S.player.cur === d.id) S.player.cur = null;
            if (S.view === 'player' && S.player.tab === 'mine') renderPlayer();
            return;
        }
        if (d.toast) toast(d.toast, d.sound);

        if (kind === 'ticket' && d.ticket) {
            var x = markMine(d.ticket);
            if (x.archivedAt) { upsert(A.archived, x); }
            else if (x.status === 'closed') { A.tickets = A.tickets.filter(function (k) { return k.id !== x.id; }); upsert(A.closed, x); }
            else upsert(A.tickets, x);
            if (A.detail && A.curId === x.id) A.detail.ticket = x;
            if (S.view === 'staff' && A.tab === 'tickets') renderStaff();
        } else if (kind === 'gone') {
            A.tickets = A.tickets.filter(function (k) { return k.id !== d.id; });
            A.closed = A.closed.filter(function (k) { return k.id !== d.id; });
            A.archived = A.archived.filter(function (k) { return k.id !== d.id; });
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
            S.powerList = m.powerList || []; S.answers = m.answers !== false; S.full = loadFull(m.fullscreen);
        } else if (m.type === 'me') {
            S.me = { staff: !!m.staff, roles: m.roles || [], powers: m.powers || {}, duty: m.duty !== false, mayHelp: !!m.mayHelp, account: m.account,
                     name: m.name || '', canDelete: !!m.canDelete };
            var wasLinked = S.web && S.web.linked;
            takeWeb(m.web);
            if (onWebTab() && S.web && S.web.linked !== wasLinked) redrawWeb();
            // An older server (or the mock) sends no role list: fall back to the four everyone had.
            S.roleDefs = (m.roleDefs && m.roleDefs.length) ? m.roleDefs : (S.roleDefs.length ? S.roleDefs : S.allRoles.map(function (id) { return { id: id, label: t('role_' + id) }; }));
            setBadge(m.badge);
        } else if (m.type === 'open') {
            S.view = m.view;
            if (m.view === 'player') {
                S.player.players = m.players || []; S.player.cooldown = m.cooldown || 0; S.player.msg = null; S.player.cur = null; S.player.tab = 'new'; S.player.step = 'choose';
                S.player.waits = m.waits || {}; S.player.similar = []; S.player.answer = null;
                renderPlayer(); loadMine();
            } else {
                S.staff.canReturn = !!m.canReturn;
                S.staff.tab = (m.tab === 'players' || m.tab === 'staff' || m.tab === 'chat' || (m.tab === 'web' && S.web)) ? m.tab : 'tickets';
                if (S.staff.tab === 'web') watchWeb();
                renderStaff(); loadTickets();
                // Know the rooms from the start, so the Staff chat tab can show its bubble.
                loadChat().then(function () { if (S.staff.tab === 'staff') loadStaffTab(); });
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
