/* ============================================================================
   poggy_markets — shop admin panel (/pmadmin) and owner handovers
   ----------------------------------------------------------------------------
   Two things that move shops between people:

     * The admin panel: every shop in one table, searched and filtered here,
       and the picked shop's actions beside it.  The server re-checks the
       admin and every action (server/shopadmin.lua).

     * The "Hand Over Shop" card in the manager's Settings tab, shown only to
       the shop's real owner.  It listens to the manager's own 'open' message
       rather than reaching into manager.js.

   Markup is in index.html and reuses the manager's classes, so every skin
   and the Poggy theme style it with nothing extra.
   ============================================================================ */

(function () {
    'use strict';

    var RESOURCE = GetParentResourceName();

    function $(sel) { return document.querySelector(sel); }

    function post(event, data) {
        return fetch('https://' + RESOURCE + '/' + event, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data || {})
        }).catch(function () { /* the game is gone; nothing to do */ });
    }

    function esc(v) {
        return String(v == null ? '' : v)
            .replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;').replace(/'/g, '&#39;');
    }

    function money(n) {
        return '$' + (parseFloat(n) || 0).toFixed(2);
    }

    /* ========================================================================
       THE ADMIN PANEL
       ======================================================================== */

    var Admin = {
        visible: false,
        shops: [],
        jobsOn: false,
        picked: null,      // shop id
        armedRepo: 0,      // time the Repossess button was first pressed
        newOwner: null,    // { charId, name } picked in the transfer search
        searchTimer: null
    };

    function shopById(id) {
        for (var i = 0; i < Admin.shops.length; i++) {
            if (Admin.shops[i].id === id) return Admin.shops[i];
        }
        return null;
    }

    function statusOf(s) {
        if (s.repo) return 'Repossessed';
        if (!s.ownerId) return 'No owner';
        return 'Open';
    }

    function renderTable() {
        var q = ($('#admin-search').value || '').trim().toLowerCase();
        var filter = $('#admin-filter').value;
        var rows = Admin.shops.filter(function (s) {
            if (filter === 'owned' && (!s.ownerId || s.repo)) return false;
            if (filter === 'unowned' && s.ownerId) return false;
            if (filter === 'repo' && !s.repo) return false;
            if (!q) return true;
            return String(s.id) === q ||
                (s.name || '').toLowerCase().indexOf(q) !== -1 ||
                (s.owner || '').toLowerCase().indexOf(q) !== -1 ||
                (s.job || '').toLowerCase().indexOf(q) !== -1;
        });

        $('#admin-count').textContent = Admin.shops.length + ' shops';
        var tbody = $('#admin-tbody');
        if (rows.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="8">' +
                (Admin.shops.length === 0 ? 'No shops yet.' : 'No shop matches that search.') + '</td></tr>';
            return;
        }
        var html = '';
        rows.forEach(function (s) {
            html += '<tr class="admin-row' + (s.id === Admin.picked ? ' selected' : '') + '" data-id="' + s.id + '">' +
                '<td>' + s.id + '</td>' +
                '<td>' + esc(s.name) + '</td>' +
                '<td>' + (s.ownerId ? esc(s.owner) : '<span class="emp-hint">&mdash;</span>') + '</td>' +
                '<td>' + money(s.ledger) + '</td>' +
                '<td>' + (s.stock || 0) + '</td>' +
                '<td>' + ((s.staff && s.staff.length) || 0) + '</td>' +
                '<td>' + esc(s.job || '') + '</td>' +
                '<td>' + statusOf(s) + '</td>' +
                '</tr>';
        });
        tbody.innerHTML = html;
        Array.prototype.forEach.call(tbody.querySelectorAll('.admin-row'), function (tr) {
            tr.addEventListener('click', function () {
                pick(parseInt(this.getAttribute('data-id'), 10));
            });
        });
    }

    /* ── Transfer: search every character ─────────────────────────────── */

    function resetTransfer() {
        Admin.newOwner = null;
        $('#admin-transfer-search').value = '';
        $('#admin-transfer-results').classList.add('hidden');
        $('#admin-transfer-results').innerHTML = '';
        $('#admin-transfer-form').classList.add('hidden');
        $('#admin-transfer-confirm').value = '';
    }

    function renderCharResults(results) {
        var list = $('#admin-transfer-results');
        list.classList.remove('hidden');
        if (!results || results.length === 0) {
            list.innerHTML = '<p class="emp-hint">No character matches.</p>';
            return;
        }
        var html = '';
        results.forEach(function (c) {
            html += '<div class="emp-player-row" data-char="' + esc(c.charId) + '" data-name="' + esc(c.name) + '">' +
                '<span class="emp-player-name">' + esc(c.name) + (c.online ? ' <span class="admin-online">online</span>' : '') + '</span>' +
                '<span class="emp-player-id">#' + esc(c.charId) + '</span>' +
                '</div>';
        });
        list.innerHTML = html;
        Array.prototype.forEach.call(list.querySelectorAll('.emp-player-row'), function (row) {
            row.addEventListener('click', function () {
                Admin.newOwner = { charId: this.getAttribute('data-char'), name: this.getAttribute('data-name') };
                list.classList.add('hidden');
                $('#admin-transfer-selected').textContent = 'Give to ' + Admin.newOwner.name + ' (#' + Admin.newOwner.charId + ')';
                $('#admin-transfer-form').classList.remove('hidden');
                $('#admin-transfer-confirm').focus();
            });
        });
    }

    /* ── Ledger: the balance, edited in place ───────────────────────────── */

    function updateLedgerDiff() {
        var s = shopById(Admin.picked);
        var el = $('#admin-ledger-diff');
        var v = parseFloat($('#admin-ledger-balance').value);
        if (!s || isNaN(v)) { el.textContent = ''; return; }
        var d = Math.round((v - (s.ledger || 0)) * 100) / 100;
        el.textContent = d === 0 ? 'No change' : (d > 0 ? '+' : '\u2212') + money(Math.abs(d));
        el.classList.toggle('admin-message-bad', d < 0);
    }

    function renderDetail() {
        var s = Admin.picked != null ? shopById(Admin.picked) : null;
        if (!s) {
            $('#admin-detail-title').textContent = 'Pick a shop';
            $('#admin-detail-sub').textContent = 'Select a shop in the table to act on it.';
            $('#admin-detail-body').classList.add('hidden');
            return;
        }
        $('#admin-detail-title').textContent = s.name + '  #' + s.id;
        $('#admin-detail-sub').textContent =
            (s.ownerId ? 'Owner: ' + s.owner + ' (#' + s.ownerId + ')' : 'No owner') +
            '  ·  ' + statusOf(s) + '  ·  Ledger ' + money(s.ledger);
        $('#admin-detail-body').classList.remove('hidden');

        var repoBtn = $('#admin-btn-repo');
        repoBtn.textContent = s.repo ? 'Restore' : 'Repossess';
        repoBtn.className = s.repo ? 'btn-action' : 'btn-danger';
        Admin.armedRepo = 0;

        $('#admin-btn-route').disabled = !(s.coords && s.coords.x !== undefined);
        // Keep an edit in progress; otherwise show the current balance.
        var balEl = $('#admin-ledger-balance');
        if (document.activeElement !== balEl) balEl.value = (parseFloat(s.ledger) || 0).toFixed(2);
        updateLedgerDiff();
        $('#admin-job').value = s.job || '';
        $('#admin-job-desc').textContent = Admin.jobsOn
            ? 'The job the owner and staff are given. Empty = no job, "reset" = the job in config.'
            : 'Shop jobs are off on this server: a job set here is saved but nobody is given it.';

        var staff = s.staff || [];
        var tb = $('#admin-staff-tbody');
        if (staff.length === 0) {
            tb.innerHTML = '<tr><td class="empty-cell" colspan="3">No staff.</td></tr>';
        } else {
            var html = '';
            staff.forEach(function (m) {
                html += '<tr><td>' + esc(m.name) + ' <span class="emp-hint">#' + esc(m.charId) + '</span></td>' +
                    '<td>' + esc(m.role) + '</td>' +
                    '<td><button class="btn-danger admin-remove" data-char="' + esc(m.charId) + '">Remove</button></td></tr>';
            });
            tb.innerHTML = html;
            Array.prototype.forEach.call(tb.querySelectorAll('.admin-remove'), function (b) {
                b.addEventListener('click', function () {
                    act('removeStaff', { charId: this.getAttribute('data-char') });
                });
            });
        }
    }

    function pick(id) {
        Admin.picked = id;
        resetTransfer();
        $('#admin-ledger-balance').value = '';
        $('#admin-ledger-reason').value = '';
        showMessage(null);
        renderTable();
        renderDetail();
    }

    function showMessage(text, ok) {
        var el = $('#admin-message');
        if (!text) { el.classList.add('hidden'); return; }
        el.textContent = text;
        el.classList.remove('hidden');
        el.classList.toggle('admin-message-bad', ok === false);
    }

    function act(action, input) {
        if (Admin.picked == null) return;
        showMessage('Working...');
        post('adminAction', { shopId: Admin.picked, action: action, input: input || {} });
    }

    function openAdmin(data) {
        Admin.visible = true;
        Admin.picked = null;
        applyData(data);
        $('#admin-search').value = '';
        $('#admin-filter').value = 'all';
        $('#admin-container').classList.remove('hidden');
        renderTable();
        renderDetail();
        setTimeout(function () { $('#admin-search').focus(); }, 30);
    }

    function applyData(data) {
        Admin.shops = (data && data.shops) || [];
        Admin.jobsOn = !!(data && data.jobsOn);
    }

    function closeAdmin(tellGame) {
        if (!Admin.visible) return;
        Admin.visible = false;
        $('#admin-container').classList.add('hidden');
        if (tellGame) post('adminClose');
    }

    function bindAdmin() {
        $('#admin-btn-close').addEventListener('click', function () { closeAdmin(true); });
        $('#admin-btn-refresh').addEventListener('click', function () { post('adminRefresh'); });
        $('#admin-search').addEventListener('input', renderTable);
        $('#admin-filter').addEventListener('change', renderTable);

        $('#admin-btn-route').addEventListener('click', function () {
            var s = shopById(Admin.picked);
            if (s && s.coords) post('adminRoute', { coords: s.coords, name: s.name });
        });
        $('#admin-btn-manage').addEventListener('click', function () {
            var id = Admin.picked;
            if (id == null) return;
            closeAdmin(false);
            post('adminManage', { shopId: id });
        });

        // Repossessing hides a shop from its owner and staff: ask twice.
        $('#admin-btn-repo').addEventListener('click', function () {
            var s = shopById(Admin.picked);
            if (!s) return;
            if (s.repo) return act('restore');
            var now = Date.now();
            if (now - Admin.armedRepo < 4000) {
                Admin.armedRepo = 0;
                return act('repossess');
            }
            Admin.armedRepo = now;
            this.textContent = 'Click again to repossess';
        });

        // Search as the admin types, a moment after they stop.
        $('#admin-transfer-search').addEventListener('input', function () {
            var q = this.value.trim();
            clearTimeout(Admin.searchTimer);
            if (q.length < 2) {
                $('#admin-transfer-results').classList.add('hidden');
                return;
            }
            Admin.searchTimer = setTimeout(function () { post('adminCharSearch', { query: q }); }, 300);
        });
        $('#admin-transfer-clear').addEventListener('click', resetTransfer);
        $('#admin-btn-transfer').addEventListener('click', function () {
            if (!Admin.newOwner) return showMessage('Search for the new owner and pick them.', false);
            if ($('#admin-transfer-confirm').value.trim() !== String(Admin.picked)) {
                return showMessage('Type the shop ID (' + Admin.picked + ') to confirm the transfer.', false);
            }
            act('transfer', { charId: Admin.newOwner.charId });
            resetTransfer();
        });

        $('#admin-ledger-balance').addEventListener('input', updateLedgerDiff);
        $('#admin-btn-ledger').addEventListener('click', function () {
            var s = shopById(Admin.picked);
            var balance = parseFloat($('#admin-ledger-balance').value);
            var reason = $('#admin-ledger-reason').value.trim();
            if (isNaN(balance) || balance < 0) return showMessage('Enter a balance of 0 or more.', false);
            if (s && Math.round(balance * 100) === Math.round((s.ledger || 0) * 100)) {
                return showMessage('Change the balance first.', false);
            }
            if (!reason) return showMessage('Say why: it shows in the shop\u2019s ledger.', false);
            act('setLedger', { balance: balance, reason: reason });
            $('#admin-ledger-reason').value = '';
            $('#admin-ledger-balance').blur();
        });

        $('#admin-btn-job').addEventListener('click', function () {
            act('setJob', { job: $('#admin-job').value.trim() });
        });

        document.addEventListener('keydown', function (e) {
            if (e.key === 'Escape' && Admin.visible) closeAdmin(true);
        });
    }

    /* ========================================================================
       HAND OVER SHOP (the manager's Settings tab, owner only)
       ======================================================================== */

    var Hand = { shopName: '', picked: null };

    function resetHandover() {
        Hand.picked = null;
        $('#transfer-player-list').classList.add('hidden');
        $('#transfer-player-list').innerHTML = '';
        $('#transfer-form').classList.add('hidden');
        $('#transfer-confirm-name').value = '';
    }

    function renderCandidates(players, error) {
        var list = $('#transfer-player-list');
        list.classList.remove('hidden');
        if (error) {
            list.innerHTML = '<p class="emp-hint">' + esc(error) + '</p>';
            return;
        }
        if (!players || players.length === 0) {
            list.innerHTML = '<p class="emp-hint">Nobody is standing near you.</p>';
            return;
        }
        var html = '';
        players.forEach(function (p) {
            html += '<div class="emp-player-row' + (p.ok ? '' : ' admin-disabled') + '" data-id="' + p.serverId +
                '" data-name="' + esc(p.name) + '" data-ok="' + (p.ok ? 1 : 0) + '">' +
                '<span class="emp-player-name">' + esc(p.name) + '</span>' +
                '<span class="emp-player-id">' + (p.ok ? '' : esc(p.reason || '')) + '</span>' +
                '</div>';
        });
        list.innerHTML = html;
        Array.prototype.forEach.call(list.querySelectorAll('.emp-player-row'), function (row) {
            row.addEventListener('click', function () {
                if (this.getAttribute('data-ok') !== '1') return;
                Array.prototype.forEach.call(list.querySelectorAll('.emp-player-row'), function (r) {
                    r.classList.remove('selected');
                });
                this.classList.add('selected');
                Hand.picked = { serverId: parseInt(this.getAttribute('data-id'), 10), name: this.getAttribute('data-name') };
                $('#transfer-selected-name').textContent = 'Give to ' + Hand.picked.name;
                $('#transfer-form').classList.remove('hidden');
            });
        });
    }

    function bindHandover() {
        $('#btn-transfer-find').addEventListener('click', function () {
            resetHandover();
            post('transferCandidates');
        });
        $('#transfer-clear').addEventListener('click', resetHandover);
        $('#btn-transfer-offer').addEventListener('click', function () {
            if (!Hand.picked) return;
            var typed = $('#transfer-confirm-name').value.trim();
            if (typed.toLowerCase() !== Hand.shopName.trim().toLowerCase()) {
                $('#transfer-confirm-name').focus();
                return;
            }
            post('transferOffer', { serverId: Hand.picked.serverId, confirmName: typed });
            resetHandover();
        });
    }

    /* ========================================================================
       MESSAGES FROM THE GAME
       ======================================================================== */

    window.addEventListener('message', function (event) {
        var data = event.data || {};
        switch (data.type) {
            case 'adminOpen':
                openAdmin(data);
                break;
            case 'adminData':
                if (!Admin.visible) break;
                applyData(data);
                renderTable();
                renderDetail();
                showMessage(data.message || null, data.ok);
                break;
            case 'adminClose':
                closeAdmin(false);
                break;
            case 'adminCharResults':
                // Only the answer to what is in the box now; an older one is stale.
                if (Admin.visible && data.query === $('#admin-transfer-search').value.trim()) {
                    renderCharResults(data.results);
                }
                break;
            case 'open':
                // The manager opened: the handover card is for the real owner only.
                Hand.shopName = data.shopName || '';
                resetHandover();
                $('#settings-transfer-card').classList.toggle('hidden', !(data.ownsShop && data.transfers));
                break;
            case 'transferCandidates':
                renderCandidates(data.players, data.error);
                break;
        }
    });

    var bound = false;
    function bindAll() {
        if (bound) return;
        bound = true;
        bindAdmin();
        bindHandover();
    }
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', bindAll);
    } else {
        bindAll();
    }
}());
