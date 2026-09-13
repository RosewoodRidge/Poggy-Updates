/* ============================================================================
   Commodities Exchange — NUI Script (Vanilla JS)
   TradingView-style trading interface
   ============================================================================ */
(function () {
    'use strict';

    /* ── State ─────────────────────────────────────────────────────────── */
    var EX = {
        open: false,
        tab: 'market',        // market | portfolio | account | chart
        filter: 'all',        // all | pelt | metal
        search: '',
        sortCol: 'name',
        sortDir: 'asc',
        balance: 0,
        items: [],             // {item_name, label, price, base_price, market_type, category}
        positions: [],         // {item_name, label, position_type, quantity, avg_price, current_price, pnl, market_type}
        tradeLog: [],
        /* Chart state */
        chartInstance: null,
        chartItem: '',         // currently viewed item_name
        chartRange: '7d',
        chartData: null,       // last received chart payload
    };

    /* ── Helpers ───────────────────────────────────────────────────────── */
    /* The fee shown in trade previews.  Read from the server rather than
       hardcoded: a preview that disagrees with what the server charges is
       worse than showing no preview at all. */
    function tradeFee() {
        return (EX.tradeFee != null) ? EX.tradeFee : 0.015;
    }

    function $(sel) { return document.querySelector(sel); }
    function $$(sel) { return document.querySelectorAll(sel); }

    function fmt(v) {
        if (v === null || v === undefined) return '$0.00';
        return '$' + parseFloat(v).toFixed(2);
    }

    function fmtPct(v) {
        if (!v && v !== 0) return '0.0%';
        var p = parseFloat(v);
        return (p >= 0 ? '+' : '') + p.toFixed(1) + '%';
    }

    function niceLabel(itemName) {
        if (!itemName) return '';
        return itemName
            .replace(/_/g, ' ')
            .replace(/\b\w/g, function (c) { return c.toUpperCase(); });
    }

    /* Item icon URL prefix, sent by the client from poggy_core's inv.imageBase
       with exchange:open.  The default only covers a message without it. */
    var IMG_BASE = 'nui://vorp_inventory/html/img/items/';

    function itemImg(itemName) {
        if (!itemName) return '';
        var src = IMG_BASE + encodeURIComponent(itemName) + '.png';
        return '<img src="' + src + '" onerror="this.style.display=\'none\'" alt="">';
    }

    function escHtml(s) {
        if (!s) return '';
        var d = document.createElement('div');
        d.textContent = s;
        return d.innerHTML;
    }

    function sendNUI(event, data) {
        return fetch(`https://${GetParentResourceName()}/${event}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data || {})
        }).then(function (r) { return r.json(); }).catch(function () { return {}; });
    }

    /* ── Toast notifications ───────────────────────────────────────────── */
    function showToast(msg, level) {
        var c = $('#ex-toast-container');
        if (!c) return;
        var t = document.createElement('div');
        t.className = 'ex-toast ' + (level || 'success');
        t.textContent = msg;
        c.appendChild(t);
        setTimeout(function () {
            t.classList.add('fade-out');
            setTimeout(function () { if (t.parentNode) t.parentNode.removeChild(t); }, 300);
        }, 3500);
    }

    /* ── Tab switching ─────────────────────────────────────────────────── */
    function switchTab(tab) {
        EX.tab = tab;
        $$('.ex-nav-item').forEach(function (el) {
            el.classList.toggle('active', el.dataset.tab === tab);
        });
        $$('.ex-panel').forEach(function (el) {
            el.classList.toggle('active', el.id === 'ex-panel-' + tab);
        });

        // Update header title
        var titles = { market: 'Market Overview', portfolio: 'My Portfolio', account: 'Account', chart: 'Price Chart' };
        var h = $('#ex-header-title');
        if (h) h.textContent = titles[tab] || 'Market Overview';

        // Show/hide filter bar and search
        var fb = $('#ex-filter-bar');
        var sb = $('#ex-search-box');
        if (fb) fb.style.display = (tab === 'market') ? 'flex' : 'none';
        if (sb) sb.style.display = (tab === 'market') ? 'block' : 'none';

        // If switching to chart, update search + sidebar
        if (tab === 'chart') {
            updateChartSearch();
            updateTradeSidebar();
        }

        render();
    }

    function switchFilter(f) {
        EX.filter = f;
        $$('.ex-filter-btn').forEach(function (el) {
            el.classList.toggle('active', el.dataset.filter === f);
        });
        renderMarket();
    }

    /* Rebuild the market filter tabs from whatever markets the server has
       configured.  They used to be hardcoded, which meant renaming a market in
       the config silently broke its tab. */
    function buildFilterBar(markets) {
        var bar = $('#ex-filter-bar');
        if (!bar || !markets) return;

        var html = '<button class="ex-filter-btn' + (EX.filter === 'all' ? ' active' : '')
                 + '" data-filter="all">All</button>';

        var stillValid = (EX.filter === 'all');
        markets.forEach(function (m) {
            if (m.key === EX.filter) stillValid = true;
            html += '<button class="ex-filter-btn' + (m.key === EX.filter ? ' active' : '')
                 +  '" data-filter="' + escHtml(m.key) + '">' + escHtml(m.label || m.key)
                 +  '</button>';
        });

        bar.innerHTML = html;

        // The market this filter pointed at may no longer exist.
        if (!stillValid) EX.filter = 'all';
    }

    /* ── Sorting ───────────────────────────────────────────────────────── */
    function toggleSort(col) {
        if (EX.sortCol === col) {
            EX.sortDir = EX.sortDir === 'asc' ? 'desc' : 'asc';
        } else {
            EX.sortCol = col;
            EX.sortDir = col === 'name' ? 'asc' : 'desc';
        }
        renderMarket();
    }

    function sortItems(items) {
        var col = EX.sortCol;
        var dir = EX.sortDir === 'asc' ? 1 : -1;
        return items.slice().sort(function (a, b) {
            var va, vb;
            if (col === 'name') {
                va = (a.label || a.item_name || '').toLowerCase();
                vb = (b.label || b.item_name || '').toLowerCase();
                return va < vb ? -dir : va > vb ? dir : 0;
            }
            if (col === 'price')   { va = a.price; vb = b.price; }
            if (col === 'change')  { va = a.change_pct || 0; vb = b.change_pct || 0; }
            if (col === 'base')    { va = a.base_price; vb = b.base_price; }
            if (col === 'spread')  { va = Math.abs((a.price - a.base_price) / a.base_price); vb = Math.abs((b.price - b.base_price) / b.base_price); }
            return (va - vb) * dir;
        });
    }

    /* ── Market rendering ──────────────────────────────────────────────── */
    function getFilteredItems() {
        var items = EX.items;
        // Filter by market type
        if (EX.filter !== 'all') {
            items = items.filter(function (it) { return it.market_type === EX.filter; });
        }
        // Filter by search
        if (EX.search) {
            var q = EX.search.toLowerCase();
            items = items.filter(function (it) {
                return (it.item_name && it.item_name.toLowerCase().indexOf(q) !== -1) ||
                       (it.label && it.label.toLowerCase().indexOf(q) !== -1) ||
                       (it.category && it.category.toLowerCase().indexOf(q) !== -1);
            });
        }
        return sortItems(items);
    }

    function renderMarket() {
        var tbody = $('#ex-market-tbody');
        if (!tbody) return;

        var items = getFilteredItems();
        if (items.length === 0) {
            tbody.innerHTML = '<tr><td colspan="7" class="ex-empty"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/></svg>No items found</td></tr>';
            return;
        }

        var sortCl = function (col) {
            if (EX.sortCol !== col) return '';
            return EX.sortDir === 'asc' ? ' sort-asc' : ' sort-desc';
        };

        // Update header classes
        $$('#ex-market-table th[data-sort]').forEach(function (th) {
            th.className = th.dataset.sort === EX.sortCol ? (EX.sortDir === 'asc' ? 'sort-asc' : 'sort-desc') : '';
        });

        var html = '';
        items.forEach(function (it) {
            var changePct = it.change_pct || 0;
            var changeClass = changePct > 0.05 ? 'up' : (changePct < -0.05 ? 'down' : 'flat');
            var spreadPct = it.base_price > 0 ? ((it.price - it.base_price) / it.base_price * 100) : 0;

            html += '<tr data-item="' + escHtml(it.item_name) + '">'
                + '<td><div class="ex-item-cell">'
                + itemImg(it.item_name)
                + '<div><div class="ex-item-name">' + escHtml(it.label || niceLabel(it.item_name)) + '</div>'
                + '<div class="ex-item-category">' + escHtml(it.category || '') + ' <span class="ex-market-badge ' + it.market_type + '">' + it.market_type + '</span></div>'
                + '</div></div></td>'
                + '<td class="ex-price">' + fmt(it.price) + '</td>'
                + '<td><span class="ex-change ' + changeClass + '">' + fmtPct(changePct) + '</span></td>'
                + '<td>' + fmt(it.base_price) + '</td>'
                + '<td><span class="ex-change ' + (spreadPct > 0.05 ? 'up' : spreadPct < -0.05 ? 'down' : 'flat') + '">' + spreadPct.toFixed(1) + '%</span></td>'
                + '<td><div class="ex-action-btns">'
                + '<button class="ex-btn ex-btn-buy" data-action="buy" data-item="' + escHtml(it.item_name) + '">Buy</button>'
                + (EX.allowShorts
                    ? '<button class="ex-btn ex-btn-short" data-action="short" data-item="' + escHtml(it.item_name) + '">Short</button>'
                    : '')
                + '</div></td>'
                + '</tr>';
        });
        tbody.innerHTML = html;
    }

    /* ── Portfolio rendering (card layout) ─────────────────────────────── */
    function renderPortfolio() {
        var container = $('#ex-port-cards');
        if (!container) return;

        var positions = EX.positions;

        // Summary stats
        var totalValue = 0, totalPnl = 0, longs = 0, shorts = 0;
        positions.forEach(function (p) {
            var val = p.quantity * p.current_price;
            totalValue += val;
            totalPnl += (p.pnl || 0);
            if (p.position_type === 'long') longs++; else shorts++;
        });

        var tv = $('#ex-stat-total-value');
        var tp = $('#ex-stat-unrealized-pnl');
        var tl = $('#ex-stat-long-count');
        var ts = $('#ex-stat-short-count');
        if (tv) { tv.textContent = fmt(totalValue); tv.className = 'ex-stat-value neutral'; }
        if (tp) {
            tp.textContent = fmt(totalPnl);
            tp.className = 'ex-stat-value ' + (totalPnl > 0 ? 'positive' : totalPnl < 0 ? 'negative' : 'neutral');
        }
        if (tl) tl.textContent = longs;
        if (ts) ts.textContent = shorts;

        if (positions.length === 0) {
            container.innerHTML = '<div class="ex-port-empty"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="18" height="18" rx="2"/><path d="M3 9h18M9 21V9"/></svg><span>No open positions</span></div>';
            return;
        }

        var html = '';
        positions.forEach(function (p) {
            var pnlClass = p.pnl > 0 ? 'up' : p.pnl < 0 ? 'down' : 'flat';
            var pnlPct = p.avg_price > 0 ? (p.pnl / (p.avg_price * p.quantity) * 100) : 0;
            var isLong = p.position_type === 'long';
            var typeBadge = isLong
                ? '<span class="ex-type-badge buy">LONG</span>'
                : '<span class="ex-type-badge short">SHORT</span>';
            var tpVal = p.take_profit || '';
            var tpStatus = '';
            if (tpVal) {
                if (isLong && p.current_price >= tpVal) tpStatus = ' tp-hit';
                else if (!isLong && p.current_price <= tpVal) tpStatus = ' tp-hit';
            }

            html += '<div class="ex-port-card' + (pnlClass === 'up' ? ' card-profit' : pnlClass === 'down' ? ' card-loss' : '') + '">'
                + '<div class="ex-pc-header">'
                + '<div class="ex-pc-item">'
                + '<div class="ex-pc-img">' + itemImg(p.item_name) + '</div>'
                + '<div>'
                + '<div class="ex-pc-name">' + escHtml(p.label || niceLabel(p.item_name)) + '</div>'
                + '<div class="ex-pc-meta">' + typeBadge + ' <span class="ex-market-badge ' + (p.market_type || 'pelt') + '">' + (p.market_type || 'pelt') + '</span></div>'
                + '</div>'
                + '</div>'
                + '<div class="ex-pc-pnl ' + pnlClass + '">'
                + '<div class="ex-pc-pnl-value">' + fmt(p.pnl) + '</div>'
                + '<div class="ex-pc-pnl-pct">' + (pnlPct >= 0 ? '+' : '') + pnlPct.toFixed(1) + '%</div>'
                + '</div>'
                + '</div>'
                + '<div class="ex-pc-details">'
                + '<div class="ex-pc-row"><span>Quantity</span><span>' + p.quantity + '</span></div>'
                + '<div class="ex-pc-row"><span>Avg Entry</span><span>' + fmt(p.avg_price) + '</span></div>'
                + '<div class="ex-pc-row"><span>Current Price</span><span class="ex-pc-current">' + fmt(p.current_price) + '</span></div>'
                + '<div class="ex-pc-row"><span>Position Value</span><span>' + fmt(p.quantity * p.current_price) + '</span></div>'
                + '<div class="ex-pc-row ex-pc-tp-row' + tpStatus + '">'
                + '<span>Take Profit</span>'
                + '<div class="ex-pc-tp-input-wrap">'
                + '<span class="ex-pc-tp-prefix">$</span>'
                + '<input type="number" class="ex-pc-tp-input" data-item="' + escHtml(p.item_name) + '" data-postype="' + p.position_type + '" value="' + tpVal + '" placeholder="—" min="0" step="0.01">'
                + '<button class="ex-pc-tp-set" data-item="' + escHtml(p.item_name) + '" data-postype="' + p.position_type + '">Set</button>'
                + '</div>'
                + '</div>'
                + '</div>'
                + '<div class="ex-pc-actions">';

            if (isLong) {
                html += '<button class="ex-btn ex-btn-sell" data-action="sell" data-item="' + escHtml(p.item_name) + '">Sell</button>';
            } else {
                html += '<button class="ex-btn ex-btn-cover" data-action="cover" data-item="' + escHtml(p.item_name) + '">Cover</button>';
            }

            html += '</div></div>';
        });
        container.innerHTML = html;
    }

    function setTakeProfit(itemName, posType) {
        var input = document.querySelector('.ex-pc-tp-input[data-item="' + itemName + '"][data-postype="' + posType + '"]');
        if (!input) return;
        var val = parseFloat(input.value);
        if (!val || val <= 0) {
            // Clear take profit
            sendNUI('exchange:setTakeProfit', { item: itemName, posType: posType, price: 0 });
            return;
        }
        sendNUI('exchange:setTakeProfit', { item: itemName, posType: posType, price: val });
    }

    /* ── Account rendering ─────────────────────────────────────────────── */
    /* Turn a millisecond interval into something a player can read. */
    function everyText(ms) {
        var mins = Math.round((ms || 0) / 60000);
        if (mins < 1)  return 'continuously';
        if (mins < 60) return 'Every ' + mins + ' min';
        var hrs = Math.round(mins / 60);
        return hrs === 1 ? 'Hourly' : 'Every ' + hrs + ' hours';
    }

    function renderAccount() {
        var bal = $('#ex-acct-balance');
        if (bal) bal.textContent = fmt(EX.balance);

        // Every figure here comes from the server's config.  It used to be
        // hardcoded in the page, which meant it quietly lied whenever the
        // config said something else.
        var info = $('#ex-trading-info');
        if (!info) return;

        function row(label, value) {
            return '<div class="ex-modal-row">'
                 + '<span class="ex-modal-row-label">' + escHtml(label) + '</span>'
                 + '<span class="ex-modal-row-value">' + escHtml(value) + '</span></div>';
        }

        var html = row('Trade Fee',
            ((EX.tradeFee != null ? EX.tradeFee : 0.015) * 100).toFixed(2)
                .replace(/\.?0+$/, '') + '% per side');

        // Only meaningful when the server actually allows shorting.
        if (EX.allowShorts) {
            html += row('Short Collateral',
                Math.round((EX.shortMargin != null ? EX.shortMargin : 0.5) * 100) + '%');
        }

        if (EX.maxPosition) {
            html += row('Position Limit', EX.maxPosition + ' per commodity');
        }

        // Two distinct clocks.  Prices move on the first; the chart records a
        // point on the second, so the chart is a coarser sampling of the same
        // movement.  Showing only one of them invites the wrong conclusion.
        html += row('Price Updates', everyText(EX.priceUpdateMs || 300000));
        html += row('Chart Points',  everyText(EX.snapshotMs || 3600000));

        info.innerHTML = html;
    }

    /* ── Main render ───────────────────────────────────────────────────── */
    function render() {
        // Sidebar balance
        var sb = $('#ex-sidebar-balance');
        if (sb) sb.textContent = fmt(EX.balance);

        if (EX.tab === 'market')    renderMarket();
        if (EX.tab === 'portfolio') renderPortfolio();
        if (EX.tab === 'account')   renderAccount();
        if (EX.tab === 'chart')     { updateChartSearch(); updateTradeSidebar(); }
    }

    /* ── Trade modal ───────────────────────────────────────────────────── */
    var currentTrade = { action: '', item: '', price: 0, label: '', maxQty: 0 };

    function openTradeModal(action, itemName) {
        var item = null;

        // Find item in market items
        for (var i = 0; i < EX.items.length; i++) {
            if (EX.items[i].item_name === itemName) { item = EX.items[i]; break; }
        }

        // For sell/cover, look in positions
        var pos = null;
        if (action === 'sell' || action === 'cover') {
            for (var j = 0; j < EX.positions.length; j++) {
                if (EX.positions[j].item_name === itemName) { pos = EX.positions[j]; break; }
            }
        }

        if (!item && !pos) return;

        var price = item ? item.price : (pos ? pos.current_price : 0);
        var label = item ? (item.label || niceLabel(item.item_name)) : (pos ? (pos.label || niceLabel(pos.item_name)) : itemName);
        var maxQty = 100; // server-enforced max per trade

        if (action === 'sell') {
            maxQty = pos ? Math.min(pos.quantity, 100) : 0;
        } else if (action === 'cover') {
            maxQty = pos ? Math.min(pos.quantity, 100) : 0;
        }

        currentTrade = { action: action, item: itemName, price: price, label: label, maxQty: maxQty };

        // Update modal
        var titles = { buy: 'Buy Long', sell: 'Sell Position', short: 'Open Short', cover: 'Cover Short' };
        $('#ex-modal-title').textContent = titles[action] || action;
        $('#ex-modal-item-name').textContent = label;
        $('#ex-modal-item-price').textContent = fmt(price);
        $('#ex-modal-qty').value = 1;
        $('#ex-modal-qty').max = maxQty;

        updateModalSummary();

        // Style confirm button
        var confirmBtn = $('#ex-modal-confirm');
        confirmBtn.className = 'ex-modal-confirm-' + action;
        var btnLabels = { buy: 'Confirm Buy', sell: 'Confirm Sell', short: 'Confirm Short', cover: 'Confirm Cover' };
        confirmBtn.textContent = btnLabels[action] || 'Confirm';

        $('#ex-modal-overlay').classList.remove('hidden');
        $('#ex-modal-qty').focus();
    }

    function updateModalSummary() {
        var qty = parseInt($('#ex-modal-qty').value) || 0;
        if (qty < 0) qty = 0;
        var item = findItem(currentTrade.item);
        var spotPrice = currentTrade.price;
        var fee = 0;
        var collateral = 0;
        var total = 0;
        var subtotal = 0;
        var vwapNote = '';

        if (currentTrade.action === 'buy') {
            var vwap = calcVWAP(item, qty, 'buy');
            subtotal = qty * vwap;
            fee = subtotal * tradeFee();
            total = subtotal + fee;
            if (qty > 1 && vwap > spotPrice * 1.01) vwapNote = '<div class="ex-modal-summary-row note"><span>Avg Price (slippage)</span><span>' + fmt(vwap) + '/ea</span></div>';
        } else if (currentTrade.action === 'sell') {
            var vwap = calcVWAP(item, qty, 'sell');
            subtotal = qty * vwap;
            fee = subtotal * tradeFee();
            total = subtotal - fee; // receive
            if (qty > 1 && vwap < spotPrice * 0.99) vwapNote = '<div class="ex-modal-summary-row note"><span>Avg Price (slippage)</span><span>' + fmt(vwap) + '/ea</span></div>';
        } else if (currentTrade.action === 'short') {
            var vwap = calcVWAP(item, qty, 'sell');
            subtotal = qty * vwap;
            collateral = subtotal * 2; // 200%
            fee = subtotal * tradeFee();
            total = collateral + fee;
            if (qty > 1 && vwap < spotPrice * 0.99) vwapNote = '<div class="ex-modal-summary-row note"><span>Avg Price (slippage)</span><span>' + fmt(vwap) + '/ea</span></div>';
        } else if (currentTrade.action === 'cover') {
            var vwap = calcVWAP(item, qty, 'buy');
            subtotal = qty * vwap;
            fee = subtotal * tradeFee();
            total = subtotal + fee; // cost to cover
            if (qty > 1 && vwap > spotPrice * 1.01) vwapNote = '<div class="ex-modal-summary-row note"><span>Avg Price (slippage)</span><span>' + fmt(vwap) + '/ea</span></div>';
        }

        var summaryHtml = vwapNote;
        summaryHtml += '<div class="ex-modal-summary-row"><span>Subtotal</span><span>' + fmt(subtotal) + '</span></div>';
        if (fee > 0) summaryHtml += '<div class="ex-modal-summary-row"><span>Fee (1.5%)</span><span>' + fmt(fee) + '</span></div>';
        if (collateral > 0) summaryHtml += '<div class="ex-modal-summary-row"><span>Collateral (200%)</span><span>' + fmt(collateral) + '</span></div>';

        var totalLabel = (currentTrade.action === 'sell') ? 'You Receive' : 'Total Cost';
        var totalDisplay = fmt(total);
        summaryHtml += '<div class="ex-modal-summary-row total"><span>' + totalLabel + '</span><span>' + totalDisplay + '</span></div>';

        $('#ex-modal-summary').innerHTML = summaryHtml;
    }

    function closeModal() {
        $('#ex-modal-overlay').classList.add('hidden');
    }

    function confirmTrade() {
        var qty = parseInt($('#ex-modal-qty').value) || 0;
        if (qty <= 0) { showToast('Quantity must be at least 1', 'error'); return; }
        if (qty > currentTrade.maxQty && currentTrade.maxQty < 9999) {
            showToast('Max quantity is ' + currentTrade.maxQty, 'error');
            return;
        }

        sendNUI('exchange:' + currentTrade.action, { item: currentTrade.item, qty: qty });
        closeModal();
    }

    /* ── Deposit / Withdraw modal (inline) ─────────────────────────────── */
    function doDeposit() {
        var inp = $('#ex-deposit-input');
        var val = parseFloat(inp ? inp.value : 0);
        if (!val || val <= 0) { showToast('Enter a valid amount', 'error'); return; }
        sendNUI('exchange:deposit', { amount: val });
        if (inp) inp.value = '';
    }

    function doWithdraw() {
        var inp = $('#ex-withdraw-input');
        var val = parseFloat(inp ? inp.value : 0);
        if (!val || val <= 0) { showToast('Enter a valid amount', 'error'); return; }
        sendNUI('exchange:withdraw', { amount: val });
        if (inp) inp.value = '';
    }

    function doSidebarDeposit() {
        var inp = $('#ex-sidebar-dep-input');
        var val = parseFloat(inp ? inp.value : 0);
        if (!val || val <= 0) { showToast('Enter a valid amount', 'error'); return; }
        sendNUI('exchange:deposit', { amount: val });
        if (inp) inp.value = '';
    }

    function doSidebarWithdraw() {
        var inp = $('#ex-sidebar-dep-input');
        var val = parseFloat(inp ? inp.value : 0);
        if (!val || val <= 0) { showToast('Enter a valid amount', 'error'); return; }
        sendNUI('exchange:withdraw', { amount: val });
        if (inp) inp.value = '';
    }

    /* ── NUI message handler ───────────────────────────────────────────── */
    window.addEventListener('message', function (event) {
        var d = event.data;
        if (!d || !d.type) return;

        switch (d.type) {
            case 'exchange:open':
                if (d.imageBase) IMG_BASE = d.imageBase;
                EX.open = true;
                $('#exchange-container').classList.remove('hidden');
                sendNUI('exchange:refresh');
                break;

            case 'exchange:close':
                EX.open = false;
                $('#exchange-container').classList.add('hidden');
                closeModal();
                break;

            case 'exchange:state':
                if (d.data) {
                    EX.balance = d.data.balance || 0;
                    EX.positions = d.data.positions || [];
                    EX.tradeLog = d.data.tradeLog || [];
                    if (d.data.tradeFee      != null) EX.tradeFee      = d.data.tradeFee;
                    if (d.data.shortMargin   != null) EX.shortMargin   = d.data.shortMargin;
                    if (d.data.allowShorts   != null) EX.allowShorts   = d.data.allowShorts;
                    if (d.data.maxPosition   != null) EX.maxPosition   = d.data.maxPosition;
                    if (d.data.priceUpdateMs != null) EX.priceUpdateMs = d.data.priceUpdateMs;
                    if (d.data.snapshotMs    != null) EX.snapshotMs    = d.data.snapshotMs;
                    // Filter tabs come from the server so they always match the
                    // markets that are actually configured.
                    if (d.data.markets) buildFilterBar(d.data.markets);
                    if (d.data.items) {
                        // Merge with existing change_pct
                        var oldMap = {};
                        EX.items.forEach(function (it) { oldMap[it.item_name] = it; });
                        EX.items = d.data.items.map(function (it) {
                            var old = oldMap[it.item_name];
                            if (old && !it.change_pct) {
                                it.change_pct = old.price > 0 ? ((it.price - old.price) / old.price * 100) : 0;
                            }
                            if (!it.label) it.label = niceLabel(it.item_name);
                            return it;
                        });
                    }
                }
                render();
                // If on chart tab viewing an item, refresh chart to update position annotations
                if (EX.tab === 'chart' && EX.chartItem) {
                    requestChartData(EX.chartItem, EX.chartRange);
                }
                break;

            case 'exchange:balanceUpdate':
                EX.balance = d.balance || 0;
                render();
                break;

            case 'exchange:priceUpdate':
                if (d.items && Array.isArray(d.items)) {
                    var priceMap = {};
                    d.items.forEach(function (it) { priceMap[it.item_name] = it.price; });

                    EX.items.forEach(function (it) {
                        if (priceMap[it.item_name] !== undefined) {
                            var oldPrice = it.price;
                            it.price = priceMap[it.item_name];
                            it.change_pct = oldPrice > 0 ? ((it.price - oldPrice) / oldPrice * 100) : 0;
                        }
                    });

                    // Update position current prices
                    EX.positions.forEach(function (p) {
                        if (priceMap[p.item_name] !== undefined) {
                            p.current_price = priceMap[p.item_name];
                            if (p.position_type === 'long') {
                                p.pnl = (p.current_price - p.avg_price) * p.quantity;
                            } else {
                                p.pnl = (p.avg_price - p.current_price) * p.quantity;
                            }
                        }
                    });

                    render();
                }
                break;

            case 'exchange:notification':
                showToast(d.message || '', d.level || 'success');
                if (d.refresh) sendNUI('exchange:refresh');
                break;

            case 'exchange:takeProfitSet':
                showToast('Take-profit set at ' + fmt(d.price) + ' for ' + escHtml(d.item), 'success');
                // Update local position data
                EX.positions.forEach(function (p) {
                    if (p.item_name === d.item && p.position_type === d.posType) {
                        p.take_profit = d.price || 0;
                    }
                });
                break;

            case 'exchange:chartData':
                if (d.data) {
                    EX.chartData = d.data;
                    renderChart(d.data);
                }
                break;
        }
    });

    /* ── Event delegation ──────────────────────────────────────────────── */
    document.addEventListener('click', function (e) {
        var t = e.target;

        // Nav items
        if (t.closest('.ex-nav-item')) {
            var tab = t.closest('.ex-nav-item').dataset.tab;
            if (tab) switchTab(tab);
            return;
        }

        // Filter buttons
        if (t.closest('.ex-filter-btn')) {
            var f = t.closest('.ex-filter-btn').dataset.filter;
            if (f) switchFilter(f);
            return;
        }

        // Table header sort
        if (t.closest('#ex-market-table th[data-sort]')) {
            var col = t.closest('th').dataset.sort;
            if (col) toggleSort(col);
            return;
        }

        // Chart range buttons
        if (t.closest('.ex-chart-range-btn')) {
            var range = t.closest('.ex-chart-range-btn').dataset.range;
            if (range) {
                EX.chartRange = range;
                $$('.ex-chart-range-btn').forEach(function (b) {
                    b.classList.toggle('active', b.dataset.range === range);
                });
                if (EX.chartItem) requestChartData(EX.chartItem, range);
            }
            return;
        }

        // Autocomplete item click
        if (t.closest('.ex-chart-ac-item')) {
            var acItem = t.closest('.ex-chart-ac-item');
            if (acItem.dataset.item) selectAutocompleteItem(acItem.dataset.item);
            return;
        }

        // Trade sidebar tabs (buy/short)
        if (t.closest('.ex-ct-tab')) {
            var action = t.closest('.ex-ct-tab').dataset.ctAction;
            if (action) {
                ctAction = action;
                $$('.ex-ct-tab').forEach(function (el) {
                    el.classList.toggle('active', el.dataset.ctAction === action);
                });
                updateTradeSidebar();
            }
            return;
        }

        // Trade sidebar confirm
        if (t.closest('#ex-ct-confirm')) {
            confirmChartTrade();
            return;
        }

        // Trade sidebar position action buttons (sell/cover)
        if (t.closest('.ex-ct-pos-btn')) {
            var posBtn = t.closest('.ex-ct-pos-btn');
            if (posBtn.dataset.action && posBtn.dataset.item) {
                openTradeModal(posBtn.dataset.action, posBtn.dataset.item);
            }
            return;
        }

        // Portfolio card take-profit set button
        if (t.closest('.ex-pc-tp-set')) {
            var tpBtn = t.closest('.ex-pc-tp-set');
            if (tpBtn.dataset.item) {
                setTakeProfit(tpBtn.dataset.item, tpBtn.dataset.postype);
            }
            return;
        }

        // Click item name in market table to open chart
        if (t.closest('#ex-market-table .ex-item-name')) {
            var row = t.closest('tr[data-item]');
            if (row && row.dataset.item) {
                openChartForItem(row.dataset.item);
            }
            return;
        }

        // Action buttons (buy/sell/short/cover)
        if (t.closest('.ex-btn[data-action]')) {
            var btn = t.closest('.ex-btn[data-action]');
            openTradeModal(btn.dataset.action, btn.dataset.item);
            return;
        }

        // Close button
        if (t.closest('.ex-btn-close')) {
            sendNUI('exchange:close');
            return;
        }

        // Modal close
        if (t.closest('.ex-modal-close') || t.classList.contains('ex-modal-overlay')) {
            closeModal();
            return;
        }

        // Modal cancel
        if (t.closest('.ex-modal-cancel')) {
            closeModal();
            return;
        }

        // Modal confirm
        if (t.closest('#ex-modal-confirm')) {
            confirmTrade();
            return;
        }

        // Sidebar deposit/withdraw
        if (t.closest('#ex-sidebar-dep-btn'))  { doSidebarDeposit(); return; }
        if (t.closest('#ex-sidebar-wd-btn'))   { doSidebarWithdraw(); return; }

        // Account deposit/withdraw
        if (t.closest('#ex-acct-dep-btn'))  { doDeposit(); return; }
        if (t.closest('#ex-acct-wd-btn'))   { doWithdraw(); return; }
    });

    // Search input + chart search + trade qty
    document.addEventListener('input', function (e) {
        if (e.target.id === 'ex-search-input') {
            EX.search = e.target.value;
            renderMarket();
        }
        if (e.target.id === 'ex-modal-qty') {
            updateModalSummary();
        }
        if (e.target.id === 'ex-chart-search') {
            showAutocomplete(e.target.value);
        }
        if (e.target.id === 'ex-ct-qty') {
            updateCtSummary();
        }
    });

    // Chart search keyboard navigation
    document.addEventListener('keydown', function (e) {
        if (e.target.id === 'ex-chart-search') {
            var ac = $('#ex-chart-ac');
            if (!ac || !ac.classList.contains('open')) return;
            var items = ac.querySelectorAll('.ex-chart-ac-item');
            if (items.length === 0) return;

            if (e.key === 'ArrowDown') {
                e.preventDefault();
                acIndex = Math.min(acIndex + 1, items.length - 1);
                items.forEach(function (el, i) { el.classList.toggle('highlighted', i === acIndex); });
            } else if (e.key === 'ArrowUp') {
                e.preventDefault();
                acIndex = Math.max(acIndex - 1, 0);
                items.forEach(function (el, i) { el.classList.toggle('highlighted', i === acIndex); });
            } else if (e.key === 'Enter') {
                e.preventDefault();
                if (acIndex >= 0 && items[acIndex]) {
                    selectAutocompleteItem(items[acIndex].dataset.item);
                } else if (items.length === 1) {
                    selectAutocompleteItem(items[0].dataset.item);
                }
            } else if (e.key === 'Escape') {
                hideAutocomplete();
            }
            return;
        }

        // ESC key (global)
        if (e.key === 'Escape') {
            if (!$('#ex-modal-overlay').classList.contains('hidden')) {
                closeModal();
            } else if (EX.open) {
                sendNUI('exchange:close');
            }
        }
    });

    // Hide autocomplete on blur (slight delay for click to register)
    document.addEventListener('focusout', function (e) {
        if (e.target.id === 'ex-chart-search') {
            setTimeout(hideAutocomplete, 200);
        }
    });

    /* ── Chart search autocomplete ────────────────────────────────────── */
    var acIndex = -1; // highlighted autocomplete index

    function updateChartSearch() {
        var inp = $('#ex-chart-search');
        if (!inp) return;
        // Set value to current item label if one is selected
        if (EX.chartItem) {
            var it = findItem(EX.chartItem);
            if (it && !inp.matches(':focus')) {
                inp.value = it.label || niceLabel(it.item_name);
            }
        }
    }

    function findItem(name) {
        for (var i = 0; i < EX.items.length; i++) {
            if (EX.items[i].item_name === name) return EX.items[i];
        }
        return null;
    }

    /* ── Client-side VWAP estimate ─────────────────────────────────────
       Mirrors the server calculateVWAP so previews show realistic costs.
       direction: "buy" or "sell"
       Returns the average execution price, or spot if params missing. */
    function calcVWAP(item, qty, direction) {
        if (!item || !qty || qty <= 0) return item ? item.price : 0;
        var bp   = item.base_price  || 0;
        var mult = item.multiplier  || 1;
        var dpu  = (direction === 'buy') ? (item.demand_pu || 0) : (item.decay_pu || 0);
        var fl   = item.floor   || 0.20;
        var ceil = item.ceiling || 1.80;
        if (!bp || !dpu) return item.price;  // fallback to spot

        var startMult = mult;
        var endMult;
        if (direction === 'buy') {
            endMult = Math.min(startMult + dpu * qty, ceil);
        } else {
            endMult = Math.max(startMult - dpu * qty, fl);
        }
        var avgMult = (startMult + endMult) / 2;
        return Math.round(bp * avgMult * 100) / 100;
    }

    function showAutocomplete(query) {
        var ac = $('#ex-chart-ac');
        if (!ac) return;
        acIndex = -1;

        if (!query) { ac.classList.remove('open'); ac.innerHTML = ''; return; }

        var q = query.toLowerCase();
        var matches = EX.items.filter(function (it) {
            return (it.item_name && it.item_name.toLowerCase().indexOf(q) !== -1) ||
                   (it.label && it.label.toLowerCase().indexOf(q) !== -1) ||
                   (it.category && it.category.toLowerCase().indexOf(q) !== -1);
        }).slice(0, 15);

        if (matches.length === 0) { ac.classList.remove('open'); ac.innerHTML = ''; return; }

        var html = '';
        matches.forEach(function (it, idx) {
            var label = escHtml(it.label || niceLabel(it.item_name));
            var imgSrc = IMG_BASE + encodeURIComponent(it.item_name) + '.png';
            html += '<div class="ex-chart-ac-item" data-item="' + escHtml(it.item_name) + '" data-idx="' + idx + '">'
                + '<img src="' + imgSrc + '" onerror="this.style.display=\'none\'" alt="">'
                + '<span class="ex-chart-ac-name">' + label + '</span>'
                + '<span class="ex-chart-ac-badge ' + it.market_type + '">' + it.market_type + '</span>'
                + '<span class="ex-chart-ac-price">' + fmt(it.price) + '</span>'
                + '</div>';
        });
        ac.innerHTML = html;
        ac.classList.add('open');
    }

    function hideAutocomplete() {
        var ac = $('#ex-chart-ac');
        if (ac) { ac.classList.remove('open'); ac.innerHTML = ''; }
        acIndex = -1;
    }

    function selectAutocompleteItem(itemName) {
        var it = findItem(itemName);
        if (!it) return;
        var inp = $('#ex-chart-search');
        if (inp) inp.value = it.label || niceLabel(it.item_name);
        hideAutocomplete();
        requestChartData(itemName, EX.chartRange);
        updateTradeSidebar();
    }

    function requestChartData(itemName, range) {
        if (!itemName) return;
        EX.chartItem = itemName;
        EX.chartRange = range || EX.chartRange || '7d';
        sendNUI('exchange:getChartData', { item: itemName, range: EX.chartRange });
    }

    function renderChart(data) {
        var canvas = $('#ex-chart-canvas');
        var emptyEl = $('#ex-chart-empty');
        var infoBar = $('#ex-chart-info-bar');
        if (!canvas) return;

        var snaps = data.snapshots || [];

        // Show/hide empty state
        if (snaps.length === 0 && !data.current_price) {
            if (emptyEl) emptyEl.classList.remove('hidden');
            if (infoBar) infoBar.style.display = 'none';
            if (EX.chartInstance) { EX.chartInstance.destroy(); EX.chartInstance = null; }
            return;
        }
        if (emptyEl) emptyEl.classList.add('hidden');
        if (infoBar) infoBar.style.display = 'flex';

        // Build data arrays
        var labels = [];
        var prices = [];
        snaps.forEach(function (s) {
            labels.push(s.snapshot_at);
            prices.push(parseFloat(s.price));
        });

        // Append current price as the latest point
        if (data.current_price) {
            labels.push('Now');
            prices.push(parseFloat(data.current_price));
        }

        // Compute min/max for Y axis padding
        var allVals = prices.slice();
        (data.markers || []).forEach(function (m) { allVals.push(m.price); });
        if (data.base_price) allVals.push(parseFloat(data.base_price));
        var minP = Math.min.apply(null, allVals);
        var maxP = Math.max.apply(null, allVals);
        var pad = (maxP - minP) * 0.15 || 1;

        // Build annotation lines for positions
        var annotations = {};

        // Base price line (dashed grey)
        if (data.base_price) {
            annotations.baseLine = {
                type: 'line',
                yMin: parseFloat(data.base_price),
                yMax: parseFloat(data.base_price),
                borderColor: 'rgba(150,150,150,0.4)',
                borderWidth: 1,
                borderDash: [6, 4],
                label: {
                    display: true,
                    content: 'Base $' + parseFloat(data.base_price).toFixed(2),
                    position: 'start',
                    backgroundColor: 'rgba(80,80,80,0.7)',
                    color: '#ccc',
                    font: { size: 10, weight: '500' },
                    padding: 3,
                }
            };
        }

        // Position markers
        (data.markers || []).forEach(function (m, i) {
            var isLong = m.type === 'long';
            annotations['pos_' + i] = {
                type: 'line',
                yMin: m.price,
                yMax: m.price,
                borderColor: isLong ? 'rgba(34,197,94,0.8)' : 'rgba(239,68,68,0.8)',
                borderWidth: 2,
                borderDash: [4, 3],
                label: {
                    display: true,
                    content: m.label + ' x' + m.qty,
                    position: 'end',
                    backgroundColor: isLong ? 'rgba(34,197,94,0.15)' : 'rgba(239,68,68,0.15)',
                    color: isLong ? '#22c55e' : '#ef4444',
                    font: { size: 11, weight: '600' },
                    padding: { top: 3, bottom: 3, left: 6, right: 6 },
                    borderRadius: 3,
                }
            };
        });

        // Determine line color based on price direction
        var lineColor = '#638cff';  // default accent
        if (prices.length >= 2) {
            var first = prices[0];
            var last = prices[prices.length - 1];
            lineColor = last >= first ? '#22c55e' : '#ef4444';
        }

        // Create gradient fill
        var ctx = canvas.getContext('2d');
        var gradient = ctx.createLinearGradient(0, 0, 0, canvas.clientHeight || 400);
        if (lineColor === '#22c55e') {
            gradient.addColorStop(0, 'rgba(34,197,94,0.25)');
            gradient.addColorStop(1, 'rgba(34,197,94,0.01)');
        } else if (lineColor === '#ef4444') {
            gradient.addColorStop(0, 'rgba(239,68,68,0.25)');
            gradient.addColorStop(1, 'rgba(239,68,68,0.01)');
        } else {
            gradient.addColorStop(0, 'rgba(99,140,255,0.25)');
            gradient.addColorStop(1, 'rgba(99,140,255,0.01)');
        }

        // Destroy existing chart
        if (EX.chartInstance) {
            EX.chartInstance.destroy();
            EX.chartInstance = null;
        }

        // Format time labels
        var displayLabels = labels.map(function (l) {
            if (l === 'Now') return 'Now';
            var d = new Date(l);
            if (isNaN(d.getTime())) return l;
            if (EX.chartRange === '24h') {
                return d.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            }
            return (d.getMonth() + 1) + '/' + d.getDate() + ' ' + d.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        });

        EX.chartInstance = new Chart(ctx, {
            type: 'line',
            data: {
                labels: displayLabels,
                datasets: [{
                    label: data.label || data.item_name || 'Price',
                    data: prices,
                    borderColor: lineColor,
                    backgroundColor: gradient,
                    borderWidth: 2,
                    pointRadius: prices.length > 50 ? 0 : 2,
                    pointHoverRadius: 4,
                    pointBackgroundColor: lineColor,
                    pointBorderColor: lineColor,
                    fill: true,
                    tension: 0.3,
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                interaction: {
                    mode: 'index',
                    intersect: false,
                },
                plugins: {
                    legend: { display: false },
                    tooltip: {
                        backgroundColor: 'rgba(20,20,25,0.95)',
                        borderColor: 'rgba(99,140,255,0.3)',
                        borderWidth: 1,
                        titleColor: '#fff',
                        bodyColor: '#ddd',
                        titleFont: { size: 12, weight: '600' },
                        bodyFont: { size: 12 },
                        padding: 10,
                        cornerRadius: 6,
                        displayColors: false,
                        callbacks: {
                            label: function (ctx) {
                                return '$' + ctx.parsed.y.toFixed(2);
                            }
                        }
                    },
                    annotation: {
                        annotations: annotations
                    }
                },
                scales: {
                    x: {
                        grid: {
                            color: 'rgba(255,255,255,0.04)',
                            drawTicks: false,
                        },
                        ticks: {
                            color: 'rgba(255,255,255,0.3)',
                            font: { size: 10 },
                            maxTicksLimit: 10,
                            maxRotation: 0,
                        },
                        border: { color: 'rgba(255,255,255,0.08)' },
                    },
                    y: {
                        position: 'right',
                        min: minP - pad,
                        max: maxP + pad,
                        grid: {
                            color: 'rgba(255,255,255,0.04)',
                            drawTicks: false,
                        },
                        ticks: {
                            color: 'rgba(255,255,255,0.3)',
                            font: { size: 10 },
                            callback: function (val) { return '$' + val.toFixed(2); }
                        },
                        border: { display: false },
                    }
                },
                layout: {
                    padding: { top: 10, right: 10, bottom: 0, left: 0 }
                }
            }
        });

        // Update current price display
        var priceEl = $('#ex-chart-current-price');
        if (priceEl && data.current_price) {
            priceEl.textContent = '$' + parseFloat(data.current_price).toFixed(2);
            priceEl.style.color = lineColor;
        }

        // Update info bar
        var bp = $('#ex-chart-base-price');
        var lp = $('#ex-chart-live-price');
        var sp = $('#ex-chart-spread');
        var dp = $('#ex-chart-points');
        if (bp) bp.textContent = '$' + parseFloat(data.base_price || 0).toFixed(2);
        if (lp) lp.textContent = '$' + parseFloat(data.current_price || 0).toFixed(2);
        if (sp) {
            var spread = data.base_price > 0
                ? ((data.current_price - data.base_price) / data.base_price * 100)
                : 0;
            sp.textContent = (spread >= 0 ? '+' : '') + spread.toFixed(1) + '%';
            sp.style.color = spread >= 0 ? '#22c55e' : '#ef4444';
        }
        if (dp) dp.textContent = snaps.length;

        // Setup chart click interaction + update trade sidebar
        setupChartClick();
        updateTradeSidebar();
    }

    function openChartForItem(itemName) {
        switchTab('chart');
        var inp = $('#ex-chart-search');
        var it = findItem(itemName);
        if (inp && it) inp.value = it.label || niceLabel(it.item_name);
        requestChartData(itemName, EX.chartRange);
        updateTradeSidebar();
    }

    /* ── Trade sidebar (right of chart) ────────────────────────────────── */
    var ctAction = 'buy'; // 'buy' | 'short'

    function updateTradeSidebar() {
        var titleEl = $('#ex-ct-title');
        var labelEl = $('#ex-ct-item-label');
        var priceEl = $('#ex-ct-price');
        var qtyEl = $('#ex-ct-qty');
        var confirmEl = $('#ex-ct-confirm');
        var posBlock = $('#ex-ct-position');

        if (!EX.chartItem) {
            if (labelEl) labelEl.textContent = 'No item selected';
            if (priceEl) priceEl.textContent = '$0.00';
            if (posBlock) posBlock.style.display = 'none';
            updateCtSummary();
            return;
        }

        var it = findItem(EX.chartItem);
        var price = it ? it.price : (EX.chartData ? EX.chartData.current_price : 0);
        var label = it ? (it.label || niceLabel(it.item_name)) : niceLabel(EX.chartItem);

        if (titleEl) titleEl.textContent = ctAction === 'buy' ? 'Buy Long' : 'Open Short';
        if (labelEl) labelEl.textContent = label;
        if (priceEl) priceEl.textContent = fmt(price);

        // Style confirm button
        if (confirmEl) {
            confirmEl.className = 'ex-ct-confirm ' + ctAction;
            confirmEl.textContent = ctAction === 'buy' ? 'Confirm Buy' : 'Confirm Short';
        }

        updateCtSummary();

        // Show position if player has one for this item
        renderCtPosition();
    }

    function updateCtSummary() {
        var summaryEl = $('#ex-ct-summary');
        if (!summaryEl) return;

        var it = findItem(EX.chartItem);
        var spotPrice = it ? it.price : 0;
        var qty = parseInt($('#ex-ct-qty') ? $('#ex-ct-qty').value : 0) || 0;
        if (qty < 0) qty = 0;
        var html = '';

        if (ctAction === 'buy') {
            var vwap = calcVWAP(it, qty, 'buy');
            var subtotal = qty * vwap;
            var fee = subtotal * tradeFee();
            var total = subtotal + fee;
            if (qty > 1 && vwap > spotPrice * 1.01) html += '<div class="ex-ct-summary-row note"><span>Avg Price</span><span>' + fmt(vwap) + '/ea</span></div>';
            html += '<div class="ex-ct-summary-row"><span>Subtotal</span><span>' + fmt(subtotal) + '</span></div>';
            html += '<div class="ex-ct-summary-row"><span>Fee (1.5%)</span><span>' + fmt(fee) + '</span></div>';
            html += '<div class="ex-ct-summary-row total"><span>Total Cost</span><span>' + fmt(total) + '</span></div>';
        } else {
            var vwap = calcVWAP(it, qty, 'sell');
            var subtotal = qty * vwap;
            var fee = subtotal * tradeFee();
            var collateral = subtotal * 2;
            var total = collateral + fee;
            if (qty > 1 && vwap < spotPrice * 0.99) html += '<div class="ex-ct-summary-row note"><span>Avg Price</span><span>' + fmt(vwap) + '/ea</span></div>';
            html += '<div class="ex-ct-summary-row"><span>Subtotal</span><span>' + fmt(subtotal) + '</span></div>';
            html += '<div class="ex-ct-summary-row"><span>Fee (1.5%)</span><span>' + fmt(fee) + '</span></div>';
            html += '<div class="ex-ct-summary-row"><span>Collateral (200%)</span><span>' + fmt(collateral) + '</span></div>';
            html += '<div class="ex-ct-summary-row total"><span>Total Cost</span><span>' + fmt(total) + '</span></div>';
        }
        summaryEl.innerHTML = html;
    }

    function renderCtPosition() {
        var posBlock = $('#ex-ct-position');
        if (!posBlock) return;

        // Find any position for this item
        var pos = null;
        for (var i = 0; i < EX.positions.length; i++) {
            if (EX.positions[i].item_name === EX.chartItem) { pos = EX.positions[i]; break; }
        }

        if (!pos) { posBlock.style.display = 'none'; return; }

        posBlock.style.display = 'block';
        var typeEl = $('#ex-ct-pos-type');
        var qtyEl = $('#ex-ct-pos-qty');
        var avgEl = $('#ex-ct-pos-avg');
        var pnlEl = $('#ex-ct-pos-pnl');
        var actionsEl = $('#ex-ct-pos-actions');

        if (typeEl) {
            typeEl.textContent = pos.position_type.toUpperCase();
            typeEl.style.color = pos.position_type === 'long' ? '#22c55e' : '#ef4444';
        }
        if (qtyEl) qtyEl.textContent = pos.quantity;
        if (avgEl) avgEl.textContent = fmt(pos.avg_price);
        if (pnlEl) {
            pnlEl.textContent = fmt(pos.pnl);
            pnlEl.style.color = pos.pnl >= 0 ? '#22c55e' : '#ef4444';
        }

        if (actionsEl) {
            var html = '';
            if (pos.position_type === 'long') {
                html += '<button class="ex-ct-pos-btn sell" data-action="sell" data-item="' + escHtml(pos.item_name) + '">Sell</button>';
            } else {
                html += '<button class="ex-ct-pos-btn cover" data-action="cover" data-item="' + escHtml(pos.item_name) + '">Cover</button>';
            }
            actionsEl.innerHTML = html;
        }
    }

    function confirmChartTrade() {
        if (!EX.chartItem) { showToast('Select an item first', 'error'); return; }
        var qty = parseInt($('#ex-ct-qty') ? $('#ex-ct-qty').value : 0) || 0;
        if (qty <= 0) { showToast('Quantity must be at least 1', 'error'); return; }
        if (qty > 100) { showToast('Max 100 units per trade', 'error'); return; }
        sendNUI('exchange:' + ctAction, { item: EX.chartItem, qty: qty });
    }

    /* ── Chart click → set price context ───────────────────────────────── */
    function setupChartClick() {
        var canvas = $('#ex-chart-canvas');
        if (!canvas || !EX.chartInstance) return;

        canvas.onclick = function (evt) {
            if (!EX.chartItem || !EX.chartInstance) return;
            var points = EX.chartInstance.getElementsAtEventForMode(evt, 'index', { intersect: false }, false);
            if (points.length > 0) {
                var idx = points[0].index;
                var price = EX.chartInstance.data.datasets[0].data[idx];
                if (price !== undefined && price !== null) {
                    // Flash the price in the sidebar
                    var priceEl = $('#ex-ct-price');
                    if (priceEl) {
                        priceEl.textContent = fmt(price) + ' (chart)';
                        priceEl.style.color = '#638cff';
                        setTimeout(function () {
                            // Revert to live price
                            var it = findItem(EX.chartItem);
                            if (it && priceEl) {
                                priceEl.textContent = fmt(it.price);
                                priceEl.style.color = '';
                            }
                        }, 2000);
                    }
                    // Focus the qty input to prompt trade
                    var qtyEl = $('#ex-ct-qty');
                    if (qtyEl) qtyEl.focus();
                }
            }
        };
    }

    console.log('[Exchange] exchange.js loaded');
})();
