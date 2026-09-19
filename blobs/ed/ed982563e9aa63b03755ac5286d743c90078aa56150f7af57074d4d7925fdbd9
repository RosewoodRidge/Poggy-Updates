/* ============================================================================
   poggy_markets — Store Manager NUI Script (Vanilla JS)
   ============================================================================ */

(function () {
    'use strict';

    /* ========================================================================
       WEAPON NICE NAMES
       ======================================================================== */
    var WEAPON_LABELS = {
        'WEAPON_LASSO': 'Lasso', 'WEAPON_LASSO_REINFORCED': 'Reinforced Lasso',
        'WEAPON_MELEE_KNIFE': 'Knife', 'WEAPON_MELEE_KNIFE_RUSTIC': 'Knife Rustic',
        'WEAPON_MELEE_KNIFE_HORROR': 'Knife Horror', 'WEAPON_MELEE_KNIFE_CIVIL_WAR': 'Knife Civil War',
        'WEAPON_MELEE_KNIFE_JAWBONE': 'Knife Jawbone', 'WEAPON_MELEE_KNIFE_MINER': 'Knife Miner',
        'WEAPON_MELEE_KNIFE_VAMPIRE': 'Knife Vampire', 'WEAPON_MELEE_CLEAVER': 'Cleaver',
        'WEAPON_MELEE_HATCHET': 'Hatchet', 'WEAPON_MELEE_HATCHET_DOUBLE_BIT': 'Hatchet Double Bit',
        'WEAPON_MELEE_HATCHET_HEWING': 'Hatchet Hewing', 'WEAPON_MELEE_HATCHET_HUNTER': 'Hatchet Hunter',
        'WEAPON_MELEE_HATCHET_VIKING': 'Hatchet Viking', 'WEAPON_THROWN_TOMAHAWK': 'Tomahawk',
        'WEAPON_THROWN_TOMAHAWK_ANCIENT': 'Tomahawk Ancient', 'WEAPON_THROWN_THROWING_KNIVES': 'Throwing Knives',
        'WEAPON_MELEE_MACHETE': 'Machete', 'WEAPON_BOW': 'Bow',
        'WEAPON_PISTOL_SEMIAUTO': 'Pistol Semi-Auto', 'WEAPON_PISTOL_MAUSER': 'Pistol Mauser',
        'WEAPON_PISTOL_VOLCANIC': 'Pistol Volcanic', 'WEAPON_PISTOL_M1899': 'Pistol M1899',
        'WEAPON_REVOLVER_SCHOFIELD': 'Revolver Schofield', 'WEAPON_REVOLVER_NAVY': 'Revolver Navy',
        'WEAPON_REVOLVER_NAVY_CROSSOVER': 'Revolver Navy Crossover', 'WEAPON_REVOLVER_LEMAT': 'Revolver Lemat',
        'WEAPON_REVOLVER_DOUBLEACTION': 'Revolver Double Action', 'WEAPON_REVOLVER_CATTLEMAN': 'Revolver Cattleman',
        'WEAPON_REVOLVER_CATTLEMAN_MEXICAN': 'Revolver Cattleman Mexican',
        'WEAPON_RIFLE_VARMINT': 'Varmint Rifle', 'WEAPON_REPEATER_WINCHESTER': 'Winchester Repeater',
        'WEAPON_REPEATER_HENRY': 'Henry Repeater', 'WEAPON_REPEATER_EVANS': 'Evans Repeater',
        'WEAPON_REPEATER_CARBINE': 'Carbine Repeater', 'WEAPON_SNIPERRIFLE_ROLLINGBLOCK': 'Rolling Block Rifle',
        'WEAPON_SNIPERRIFLE_CARCANO': 'Carcano Rifle', 'WEAPON_RIFLE_SPRINGFIELD': 'Springfield Rifle',
        'WEAPON_RIFLE_ELEPHANT': 'Elephant Rifle', 'WEAPON_RIFLE_BOLTACTION': 'Bolt Action Rifle',
        'WEAPON_SHOTGUN_SEMIAUTO': 'Semi-Auto Shotgun', 'WEAPON_SHOTGUN_SAWEDOFF': 'Sawed-Off Shotgun',
        'WEAPON_SHOTGUN_REPEATING': 'Repeating Shotgun',
        'WEAPON_SHOTGUN_DOUBLEBARREL_EXOTIC': 'Double Barrel Exotic Shotgun',
        'WEAPON_SHOTGUN_PUMP': 'Pump Shotgun', 'WEAPON_SHOTGUN_DOUBLEBARREL': 'Double Barrel Shotgun',
        'WEAPON_KIT_CAMERA': 'Camera', 'WEAPON_KIT_BINOCULARS_IMPROVED': 'Improved Binoculars',
        'WEAPON_MELEE_KNIFE_TRADER': 'Knife Trader', 'WEAPON_KIT_BINOCULARS': 'Binoculars',
        'WEAPON_KIT_CAMERA_ADVANCED': 'Advanced Camera', 'WEAPON_MELEE_LANTERN': 'Lantern',
        'WEAPON_MELEE_DAVY_LANTERN': 'Davy Lantern', 'WEAPON_MELEE_LANTERN_HALLOWEEN': 'Halloween Lantern',
        'WEAPON_THROWN_POISONBOTTLE': 'Poison Bottle', 'WEAPON_KIT_METAL_DETECTOR': 'Metal Detector',
        'WEAPON_THROWN_DYNAMITE': 'Dynamite', 'WEAPON_THROWN_MOLOTOV': 'Molotov',
        'WEAPON_BOW_IMPROVED': 'Improved Bow', 'WEAPON_MELEE_MACHETE_COLLECTOR': 'Machete Collector',
        'WEAPON_MELEE_LANTERN_ELECTRIC': 'Electric Lantern', 'WEAPON_MELEE_TORCH': 'Torch',
        'WEAPON_MOONSHINEJUG_MP': 'Moonshine Jug', 'WEAPON_THROWN_BOLAS': 'Bolas',
        'WEAPON_THROWN_BOLAS_HAWKMOTH': 'Bolas Hawkmoth', 'WEAPON_THROWN_BOLAS_IRONSPIKED': 'Bolas Ironspiked',
        'WEAPON_THROWN_BOLAS_INTERTWINED': 'Bolas Intertwined', 'WEAPON_FISHINGROD': 'Fishing Rod',
        'WEAPON_MACHETE_HORROR': 'Machete Horror', 'WEAPON_MELEE_HAMMER': 'Hammer',
        'WEAPON_REVOLVER_DOUBLEACTION_GAMBLER': 'High Roller Double-Action Revolver'
    };

    function niceLabel(itemname, itemlabel) {
        if (itemname && WEAPON_LABELS[itemname.toUpperCase()]) return WEAPON_LABELS[itemname.toUpperCase()];
        return itemlabel || itemname || '';
    }

    /* ========================================================================
       STATE
       ======================================================================== */

    var State = {
        visible: false,
        shopId: 0,
        shopName: '',
        ledger: 0,
        slots: 0,
        blip: 1,
        blipSprite: '',
        isSociety: false,
        isAdmin: false,
        role: 'owner',
        isDbOwner: false,
        upgradeCost: 0.75,
        moveCost: 750,
        allowWebhooks: true,

        // Inventory (current stock)
        inventory: [],
        invSearch: '',

        // Personal inventory (player's own items)
        personalItems: [],
        personalInvSearch: '',

        // Buy list
        buyList: [],
        allItems: {},       // all server items (name → label)
        buylistSearch: '',
        buylistSelected: null,

        // Sold items / storage
        soldItems: [],
        storageInvSearch: '',

        // Analytics / transactions
        transactions: [],
        topSellers: [],
        topRevenue: [],
        revenueChart: [],
        dashStats: {},

        // Ledger filters
        ledgerFilterType: 'all',
        ledgerFilterRange: 'all',

        // Employee management
        employeeData: {},      // { directEmployees, jobEmployees, onlinePlayers }
        empSearch: '',
        empSelectedPlayer: null,
    };

    /* ========================================================================
       ROLE PERMISSIONS (mirrors server-side Config.ROLE_PERMISSIONS)
       ======================================================================== */
    var ROLE_PERMISSIONS = {
        employee: {
            viewDashboard: true, viewInventory: true, depositItems: true,
            setPrices: false, removeStock: false, manageBuyList: false,
            viewLedger: true, depositLedger: true, withdrawLedger: false,
            viewAnalytics: true, viewTransactions: true, viewSoldItems: true,
            changeSettings: false,
        },
        manager: {
            viewDashboard: true, viewInventory: true, depositItems: true,
            setPrices: true, removeStock: true, manageBuyList: true,
            viewLedger: true, depositLedger: true, withdrawLedger: true,
            viewAnalytics: true, viewTransactions: true, viewSoldItems: true,
            changeSettings: false,
        },
        owner: {
            viewDashboard: true, viewInventory: true, depositItems: true,
            setPrices: true, removeStock: true, manageBuyList: true,
            viewLedger: true, depositLedger: true, withdrawLedger: true,
            viewAnalytics: true, viewTransactions: true, viewSoldItems: true,
            changeSettings: true,
        },
    };

    function hasPermission(perm) {
        var rp = ROLE_PERMISSIONS[State.role];
        return rp ? (rp[perm] === true) : false;
    }

    /* ========================================================================
       HELPERS
       ======================================================================== */

    function sendNUI(event, data) {
        return fetch(`https://${GetParentResourceName()}/${event}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data || {})
        })
        .then(function (resp) { return resp.json(); })
        .catch(function () { return {}; });
    }

    function fmt(amount) {
        if (amount === null || amount === undefined) return '$0.00';
        return '$' + parseFloat(amount).toFixed(2);
    }

    /* Item icon URL prefix, sent by the client from poggy_core's inv.imageBase
       with open / custOpen.  The default only covers a message without it. */
    var IMG_BASE = 'nui://vorp_inventory/html/img/items/';

    function itemImgTag(itemName) {
        if (!itemName) return '';
        var src = IMG_BASE + encodeURIComponent(itemName) + '.png';
        return '<img class="item-img-sm" src="' + src + '" onerror="this.style.display=\'none\'" alt="">';
    }

    function escapeHtml(str) {
        if (!str) return '';
        var div = document.createElement('div');
        div.textContent = str;
        return div.innerHTML;
    }

    function $(sel) { return document.querySelector(sel); }
    function $$(sel) { return document.querySelectorAll(sel); }

    function parseTimestamp(v) {
        var n = parseFloat(v);
        if (!isNaN(n) && isFinite(n) && n > 1e8) {
            return new Date(n >= 1e12 ? n : n * 1000);
        }
        var s = String(v);
        return new Date(s.replace(' ', 'T') + (s.indexOf('Z') === -1 ? 'Z' : ''));
    }

    function formatDate(dateStr) {
        if (!dateStr) return '—';
        var d = parseTimestamp(dateStr);
        if (isNaN(d.getTime())) return String(dateStr);
        var months = ['January','February','March','April','May','June','July','August','September','October','November','December'];
        var day = d.getDate();
        var suffix = 'th';
        if (day % 10 === 1 && day !== 11) suffix = 'st';
        else if (day % 10 === 2 && day !== 12) suffix = 'nd';
        else if (day % 10 === 3 && day !== 13) suffix = 'rd';
        var rpYear = d.getFullYear() - 125;
        return months[d.getMonth()] + ' ' + day + suffix + ', ' + rpYear;
    }

    function formatDateShort(dateStr) {
        if (!dateStr) return '—';
        var d = parseTimestamp(dateStr);
        if (isNaN(d.getTime())) return String(dateStr);
        var months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
        var day = d.getDate();
        var suffix = 'th';
        if (day % 10 === 1 && day !== 11) suffix = 'st';
        else if (day % 10 === 2 && day !== 12) suffix = 'nd';
        else if (day % 10 === 3 && day !== 13) suffix = 'rd';
        var rpYear = d.getFullYear() - 125;
        return months[d.getMonth()] + ' ' + day + suffix + ', ' + rpYear;
    }

    function pad(n) { return n < 10 ? '0' + n : '' + n; }

    function relativeTime(dateStr) {
        if (!dateStr) return '';
        var d = parseTimestamp(dateStr);
        var diff = Math.floor((Date.now() - d.getTime()) / 1000);
        if (diff < 60) return 'just now';
        if (diff < 3600) return Math.floor(diff / 60) + 'm ago';
        if (diff < 86400) return Math.floor(diff / 3600) + 'h ago';
        if (diff < 604800) return Math.floor(diff / 86400) + 'd ago';
        return formatDate(dateStr);
    }

    /* ========================================================================
       NUI MESSAGE HANDLER
       ======================================================================== */

    window.addEventListener('message', function (event) {
        var data = event.data;
        switch (data.type) {
            case 'open':
                if (data.imageBase) IMG_BASE = data.imageBase;
                openUI(data);
                break;
            case 'close':
                closeUI();
                break;
            case 'updateLedger':
                State.ledger = data.ledger || 0;
                renderLedgerBalance();
                renderDashboard();
                break;
            case 'refreshInventory':
                if (data.inventory) {
                    State.inventory = data.inventory;
                    renderInventory();
                }
                break;
            case 'refreshBuyList':
                if (data.buyList) {
                    State.buyList = data.buyList;
                    renderBuyListCurrent();
                }
                break;
            case 'refreshTransactions':
                if (data.transactions) {
                    State.transactions = data.transactions;
                    renderTransactionHistory();
                }
                break;
            case 'refreshDashboard':
                if (data.dashStats) State.dashStats = data.dashStats;
                if (data.topSellers) State.topSellers = data.topSellers;
                if (data.inventory) State.inventory = data.inventory;
                if (data.soldItems) State.soldItems = data.soldItems;
                renderDashboard();
                break;
            case 'refreshAnalytics':
                if (data.topSellers) State.topSellers = data.topSellers;
                if (data.topRevenue) State.topRevenue = data.topRevenue;
                if (data.revenueChart) State.revenueChart = data.revenueChart;
                if (data.chartDays) State.chartDays = data.chartDays;
                State.lastSaleAt = data.lastSaleAt || State.lastSaleAt;
                renderAnalytics();
                break;
            case 'refreshSoldItems':
                if (data.soldItems) {
                    State.soldItems = data.soldItems;
                    renderSoldItems();
                }
                break;
            case 'refreshPersonalInventory':
                if (data.personalItems) {
                    State.personalItems = data.personalItems;
                    renderPersonalInventory();
                    renderStoragePersonalInventory();
                }
                break;
            case 'refreshEmployees':
                if (data.employees) {
                    State.employeeData = data.employees;
                    renderEmployees();
                }
                break;
        }
    });

    /* ========================================================================
       OPEN / CLOSE
       ======================================================================== */

    function openUI(data) {
        State.visible      = true;
        State.shopId        = data.shopId || 0;
        State.shopName      = data.shopName || 'Store';
        State.ledger        = data.ledger || 0;
        State.slots         = data.slots || 0;
        State.blip          = data.blip || 0;
        State.blipSprite    = data.blipSprite || '';
        State.isSociety     = data.isSociety || false;
        State.isAdmin       = data.isAdmin || false;
        State.role          = data.role || 'owner';
        State.isDbOwner     = data.isDbOwner || false;
        State.upgradeCost   = data.upgradeCost || 0.75;
        State.moveCost      = data.moveCost || 750;
        State.allowWebhooks = data.allowWebhooks !== false;
        State.inventory     = data.inventory || [];
        State.buyList       = data.buyList || [];
        State.allItems      = data.allItems || {};
        State.soldItems     = data.soldItems || [];
        State.transactions  = data.transactions || [];
        State.topSellers    = data.topSellers || [];
        State.topRevenue    = data.topRevenue || [];
        State.revenueChart  = data.revenueChart || [];
        State.dashStats     = data.dashStats || {};
        State.chartDays     = data.chartDays || 14;
        State.lastSaleAt    = data.lastSaleAt || null;
        State.employeeData  = {};
        State.empSearch     = '';
        State.empSelectedPlayer = null;

        // Set header
        // Capitals come from the stylesheet, so a skin can show the name as typed.
        $('#shop-title').textContent = State.shopName;
        $('#shop-id-badge').textContent = 'ID: ' + State.shopId;

        // Populate settings
        $('#settings-name').value = State.shopName;
        $('#settings-blip').checked = State.blip === 1;
        $('#settings-blip-label').textContent = State.blip === 1 ? 'Visible' : 'Hidden';
        if ($('#settings-blipsprite')) $('#settings-blipsprite').value = State.blipSprite || '';
        $('#settings-slots').textContent = State.slots;
        $('#settings-slot-cost').textContent = fmt(State.upgradeCost);
        updateUpgradeTotal();
        updateStorageUpgradeTotal();

        // Show/hide webhook card
        if (!State.allowWebhooks) {
            var wc = $('#settings-webhook-card');
            if (wc) wc.classList.add('hidden');
        }

        // Hide society-only features for non-society shops
        // (currently no society-specific tabs)

        // Role-based tab visibility
        $$('.tab-btn').forEach(function (btn) {
            var tab = btn.getAttribute('data-tab');
            if (tab === 'settings') {
                btn.style.display = hasPermission('changeSettings') ? '' : 'none';
            } else if (tab === 'buylist') {
                btn.style.display = hasPermission('manageBuyList') ? '' : 'none';
            } else if (tab === 'employees') {
                btn.style.display = State.isDbOwner ? '' : 'none';
            }
        });

        // Hide withdraw button if no permission
        var withdrawBtn = $('#btn-withdraw');
        if (withdrawBtn) withdrawBtn.style.display = hasPermission('withdrawLedger') ? '' : 'none';

        // Switch to dashboard tab
        switchTab('dashboard');

        // Render all panels
        renderDashboard();
        renderInventory();
        renderBuyListCurrent();
        renderBuyListItems();
        renderLedgerBalance();
        renderTransactionHistory();
        renderAnalytics();
        renderSoldItems();

        $('#manager-container').classList.remove('hidden');
    }

    function closeUI() {
        State.visible = false;
        $('#manager-container').classList.add('hidden');
    }

    /* ========================================================================
       TAB SWITCHING
       ======================================================================== */

    function switchTab(tabName) {
        $$('.tab-btn').forEach(function (btn) {
            btn.classList.toggle('active', btn.getAttribute('data-tab') === tabName);
        });
        $$('.tab-content').forEach(function (tc) {
            tc.classList.toggle('active', tc.id === 'tab-' + tabName);
        });

        // Fetch fresh data when switching to certain tabs
        if (tabName === 'analytics') {
            sendNUI('getAnalytics', { shopId: State.shopId });
        } else if (tabName === 'ledger') {
            sendNUI('getTransactions', {
                shopId: State.shopId,
                filterType: State.ledgerFilterType,
                filterRange: State.ledgerFilterRange
            });
        } else if (tabName === 'dashboard') {
            sendNUI('getDashboard', { shopId: State.shopId });
        } else if (tabName === 'inventory') {
            sendNUI('getPersonalInventory', { shopId: State.shopId });
        } else if (tabName === 'storage') {
            sendNUI('getPersonalInventory', { shopId: State.shopId });
            sendNUI('getSoldItems', { shopId: State.shopId });
        } else if (tabName === 'employees') {
            sendNUI('getEmployees', { shopId: State.shopId });
        }
    }

    /* ========================================================================
       DASHBOARD RENDERING
       ======================================================================== */

    function renderDashboard() {
        var stats = State.dashStats;
        $('#dash-ledger').textContent = fmt(State.ledger);
        $('#dash-today-revenue').textContent = fmt(stats.todayRevenue || 0);
        $('#dash-week-revenue').textContent = fmt(stats.weekRevenue || 0);
        $('#dash-today-items').textContent = stats.todayItems || 0;

        // Stock count
        var totalStock = 0;
        State.inventory.forEach(function (it) { totalStock += (it.itemcount || 0); });
        $('#dash-stock-count').textContent = totalStock;
        $('#dash-storage-used').textContent = totalStock + ' / ' + State.slots;

        // Top sellers
        var topEl = $('#dash-top-sellers');
        if (State.topSellers.length === 0) {
            topEl.innerHTML = '<div class="empty-state">No sales data yet</div>';
        } else {
            var html = '';
            State.topSellers.slice(0, 5).forEach(function (item, idx) {
                html += '<div class="top-seller-row">' +
                    '<span class="top-seller-rank">' + (idx + 1) + '</span>' +
                    itemImgTag(item.label) +
                    '<span class="top-seller-name">' + escapeHtml(item.label) + '</span>' +
                    '<span class="top-seller-qty">' + (item.qty || 0) + ' sold</span>' +
                    '</div>';
            });
            topEl.innerHTML = html;
        }

        // Low stock alerts (items with qty < 5)
        var lowEl = $('#dash-low-stock');
        var lowItems = State.inventory.filter(function (it) {
            return (it.itemcount || 0) > 0 && (it.itemcount || 0) < 5;
        });
        if (lowItems.length === 0) {
            lowEl.innerHTML = '<div class="empty-state">All items well stocked</div>';
        } else {
            var html2 = '';
            lowItems.forEach(function (it) {
                html2 += '<div class="low-stock-row">' +
                    itemImgTag(it.itemname) +
                    '<span class="low-stock-name">' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span>' +
                    '<span class="low-stock-qty">' + (it.itemcount || 0) + ' left</span>' +
                    '</div>';
            });
            lowEl.innerHTML = html2;
        }

        // Recent transactions
        var recentEl = $('#dash-recent-txns');
        var recent = (State.transactions || []).slice(0, 8);
        if (recent.length === 0) {
            recentEl.innerHTML = '<div class="empty-state">No recent transactions</div>';
        } else {
            var html3 = '';
            recent.forEach(function (txn) {
                var typeClass = 'txn-' + (txn.type || 'sale');
                var amtClass = (txn.type === 'sale' || txn.type === 'deposit')
                    ? 'txn-amount-positive' : 'txn-amount-negative';
                var sign = (txn.type === 'sale' || txn.type === 'deposit') ? '+' : '-';
                html3 += '<div class="recent-txn-row">' +
                    '<span class="txn-type-badge ' + typeClass + '">' + (txn.type || '?') + '</span>' +
                    '<span class="txn-item-name">' + escapeHtml(txn.itemLabel || '—') + '</span>' +
                    '<span class="txn-amount ' + amtClass + '">' + sign + fmt(txn.total) + '</span>' +
                    '<span class="txn-time">' + relativeTime(txn.date) + '</span>' +
                    '</div>';
            });
            recentEl.innerHTML = html3;
        }
    }

    /* ========================================================================
       INVENTORY RENDERING
       ======================================================================== */

    function renderInventory() {
        var tbody = $('#inv-tbody');
        var search = State.invSearch.toLowerCase();
        var filtered = State.inventory.filter(function (it) {
            if (!search) return true;
            return (it.itemname || '').toLowerCase().indexOf(search) !== -1 ||
                   (it.itemlabel || '').toLowerCase().indexOf(search) !== -1;
        });

        if (filtered.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="4">No items in stock</td></tr>';
        } else {
            var html = '';
            filtered.forEach(function (it) {
                var qtyCell;
                if (State.isAdmin) {
                    qtyCell = '<div class="inline-edit">' +
                        '<input type="number" class="input-field inv-qty-input" data-item="' + escapeHtml(it.itemname) + '" value="' + (it.itemcount || 0) + '" min="0" style="width:50px;text-align:center">' +
                        '</div>';
                } else {
                    qtyCell = '' + (it.itemcount || 0);
                }

                var priceCell;
                if (hasPermission('setPrices')) {
                    priceCell = '<div class="inline-edit">' +
                        '<input type="number" class="input-field inv-price-input" data-item="' + escapeHtml(it.itemname) + '" value="' + (it.itemprice || 0) + '" min="0" step="0.01">' +
                        '</div>';
                } else {
                    priceCell = fmt(it.itemprice || 0);
                }

                var actionCell = '';
                var _savePart = hasPermission('setPrices') ? '<button class="btn-inline-save" data-item="' + escapeHtml(it.itemname) + '">Save</button>' : '';
                var _delPart  = hasPermission('removeStock') ? '<button class="btn-danger inv-remove-btn" data-item="' + escapeHtml(it.itemname) + '" style="padding:5px 8px">Del</button>' : '';
                if (_savePart || _delPart) {
                    actionCell = '<div style="display:flex;gap:6px;justify-content:center;align-items:center">' + _savePart + _delPart + '</div>';
                }

                html += '<tr>' +
                    '<td><div class="item-cell">' + itemImgTag(it.itemname) + '<span>' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span></div></td>' +
                    '<td style="text-align:center">' + qtyCell + '</td>' +
                    '<td>' + priceCell + '</td>' +
                    '<td style="text-align:center">' + actionCell + '</td>' +
                    '</tr>';
            });
            tbody.innerHTML = html;
        }

        $('#inv-count').textContent = filtered.length + ' items in stock';

        // Bind inline save buttons (also sets qty for admins)
        $$('.btn-inline-save').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var input = $('input.inv-price-input[data-item="' + itemName + '"]');
                var newPrice = parseFloat(input.value) || 0;
                sendNUI('updateItemPrice', { shopId: State.shopId, itemName: itemName, price: newPrice });
                // Also set qty if admin
                var qtyInput = $('input.inv-qty-input[data-item="' + itemName + '"]');
                if (qtyInput) {
                    var newQty = parseInt(qtyInput.value) || 0;
                    sendNUI('adminSetQuantity', { shopId: State.shopId, itemName: itemName, quantity: newQty });
                }
            });
        });

        // Bind remove buttons
        $$('.inv-remove-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                sendNUI('removeStockItem', { shopId: State.shopId, itemName: itemName });
            });
        });
    }

    /* ========================================================================
       BUY LIST RENDERING
       ======================================================================== */

    function renderBuyListCurrent() {
        var tbody = $('#buylist-tbody');
        if (State.buyList.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="4">No items on buy list</td></tr>';
            return;
        }
        var html = '';
        State.buyList.forEach(function (bi) {
            var priceCell, amountCell, actionCell;
            if (hasPermission('manageBuyList')) {
                priceCell = '<div class="inline-edit">' +
                    '<input type="number" class="input-field bl-price-input" data-item="' + escapeHtml(bi.name) + '" value="' + (bi.price || 0) + '" min="0.01" step="0.01">' +
                    '</div>';
                amountCell = '<div class="inline-edit">' +
                    '<input type="number" class="input-field bl-amount-input" data-item="' + escapeHtml(bi.name) + '" value="' + (bi.amount || 1) + '" min="1">' +
                    '</div>';
                actionCell = '<button class="btn-action bl-save-btn" data-item="' + escapeHtml(bi.name) + '" data-label="' + escapeHtml(bi.label || bi.name) + '" style="margin-right:4px;padding:6px 12px;font-size:12px">Save</button>' +
                    '<button class="btn-danger bl-remove-btn" data-item="' + escapeHtml(bi.name) + '" data-label="' + escapeHtml(bi.label || bi.name) + '">Remove</button>';
            } else {
                priceCell = fmt(bi.price || 0);
                amountCell = '' + (bi.amount || 1);
                actionCell = '';
            }
            html += '<tr>' +
                '<td><div class="item-cell">' + itemImgTag(bi.name) + escapeHtml(bi.label || bi.name) + '</div></td>' +
                '<td>' + priceCell + '</td>' +
                '<td style="text-align:center">' + amountCell + '</td>' +
                '<td style="text-align:center">' + actionCell + '</td>' +
                '</tr>';
        });
        tbody.innerHTML = html;

        // Bind save buttons
        $$('.bl-save-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                var priceInput = $('input.bl-price-input[data-item="' + itemName + '"]');
                var amountInput = $('input.bl-amount-input[data-item="' + itemName + '"]');
                sendNUI('editBuyItem', {
                    shopId: State.shopId,
                    item: itemName,
                    label: label,
                    price: parseFloat(priceInput.value) || 0,
                    amount: parseInt(amountInput.value) || 1,
                });
            });
        });

        // Bind remove buttons
        $$('.bl-remove-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                sendNUI('removeBuyItem', {
                    shopId: State.shopId,
                    item: itemName,
                    label: label,
                });
            });
        });
    }

    function renderBuyListItems() {
        var container = $('#buylist-item-list');
        var search = State.buylistSearch.toLowerCase();
        var items = State.allItems;
        var keys = Object.keys(items);

        // Filter by search
        if (search) {
            keys = keys.filter(function (k) {
                return k.toLowerCase().indexOf(search) !== -1 ||
                       (items[k].label || '').toLowerCase().indexOf(search) !== -1;
            });
        }

        // Limit to prevent massive DOM
        keys = keys.slice(0, 200);

        if (keys.length === 0) {
            container.innerHTML = '<div class="empty-state">No items found</div>';
            return;
        }

        container.innerHTML = '<div class="empty-state">Loading items...</div>';

        // Check each item for a valid image, only show items that have one
        var validItems = [];
        var checked = 0;
        var total = keys.length;

        keys.forEach(function (k) {
            var img = new Image();
            img.onload = function () {
                var label = items[k].label || items[k] || k;
                if (typeof label === 'object') label = label.label || k;
                validItems.push({ key: k, label: label });
                checked++;
                if (checked === total) finishRender();
            };
            img.onerror = function () {
                checked++;
                if (checked === total) finishRender();
            };
            img.src = IMG_BASE + encodeURIComponent(k) + '.png';
        });

        function finishRender() {
            if (validItems.length === 0) {
                container.innerHTML = '<div class="empty-state">No items found</div>';
                return;
            }
            // Sort alphabetically by label
            validItems.sort(function (a, b) { return a.label.localeCompare(b.label); });
            // Limit display
            var display = validItems.slice(0, 100);
            var html = '';
            display.forEach(function (vi) {
                html += '<div class="buylist-item" data-item="' + escapeHtml(vi.key) + '" data-label="' + escapeHtml(vi.label) + '">' +
                    itemImgTag(vi.key) +
                    '<span>' + escapeHtml(vi.label) + '</span>' +
                    '</div>';
            });
            container.innerHTML = html;

            // Bind click
            $$('.buylist-item').forEach(function (el) {
                el.addEventListener('click', function () {
                    $$('.buylist-item').forEach(function (x) { x.classList.remove('selected'); });
                    this.classList.add('selected');
                    State.buylistSelected = {
                        name: this.getAttribute('data-item'),
                        label: this.getAttribute('data-label'),
                    };
                    $('#buylist-selected-name').textContent = State.buylistSelected.label;
                    $('#buylist-add-form').classList.remove('hidden');
                    $('#buylist-price').value = '';
                    $('#buylist-amount').value = '10';
                    $('#buylist-price').focus();
                });
            });
        }
    }

    /* ========================================================================
       LEDGER RENDERING
       ======================================================================== */

    function renderLedgerBalance() {
        $('#ledger-balance').textContent = fmt(State.ledger);
        $('#dash-ledger').textContent = fmt(State.ledger);
    }

    function renderTransactionHistory() {
        var tbody = $('#ledger-tbody');
        var txns = State.transactions || [];

        if (txns.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="6">No transactions found</td></tr>';
            return;
        }

        var html = '';
        txns.forEach(function (txn) {
            var typeClass = 'txn-' + (txn.type || 'sale');
            html += '<tr>' +
                '<td>' + formatDate(txn.date) + '</td>' +
                '<td><span class="txn-type-badge ' + typeClass + '">' + escapeHtml(txn.type || '?') + '</span></td>' +
                '<td><div class="item-cell">' + itemImgTag(txn.itemLabel) + escapeHtml(txn.itemLabel || '—') + '</div></td>' +
                '<td style="text-align:center">' + (txn.quantity || '—') + '</td>' +
                '<td>' + fmt(txn.total) + '</td>' +
                '<td>' + escapeHtml(txn.charName || '—') + '</td>' +
                '</tr>';
        });
        tbody.innerHTML = html;
    }

    /* ========================================================================
       ANALYTICS RENDERING
       ======================================================================== */

    function renderAnalytics() {
        // The heading is built from the same number the query used.
        var titleEl = $('#revenue-chart-title');
        if (titleEl) {
            titleEl.textContent = 'Revenue (Last ' + (State.chartDays || 14) + ' Days)';
        }

        // Revenue chart (CSS bar chart)
        var chartEl = $('#revenue-chart');
        var chartData = State.revenueChart || [];

        if (chartData.length === 0) {
            // Distinguish "this shop has never sold anything" from "it has,
            // just not inside the window" -- the second looked like a bug.
            var msg = 'No sales recorded yet';
            if (State.lastSaleAt) {
                msg = 'No sales in the last ' + (State.chartDays || 14) + ' days'
                    + '<br><span style="opacity:.6;font-size:12px">Last sale: '
                    + escapeHtml(formatDate(State.lastSaleAt)) + '</span>';
            }
            chartEl.innerHTML = '<div class="empty-state" style="width:100%;display:flex;'
                + 'align-items:center;justify-content:center;text-align:center">' + msg + '</div>';
        } else {
            var maxRevenue = 0;
            chartData.forEach(function (d) { if (d.revenue > maxRevenue) maxRevenue = d.revenue; });
            if (maxRevenue === 0) maxRevenue = 1;

            var html = '';
            chartData.forEach(function (d) {
                var pct = Math.max(2, (d.revenue / maxRevenue) * 100);
                html += '<div class="chart-bar-wrapper">' +
                    '<span class="chart-bar-value">' + fmt(d.revenue) + '</span>' +
                    '<div class="chart-bar" style="height:' + pct + '%"></div>' +
                    '<span class="chart-bar-label">' + escapeHtml(d.date || '') + '</span>' +
                    '</div>';
            });
            chartEl.innerHTML = html;
        }

        // Top sellers (all time)
        var topEl = $('#analytics-top-sellers');
        if (State.topSellers.length === 0) {
            topEl.innerHTML = '<div class="empty-state">No sales data yet</div>';
        } else {
            var html2 = '';
            State.topSellers.slice(0, 10).forEach(function (item, idx) {
                html2 += '<div class="top-seller-row">' +
                    '<span class="top-seller-rank">' + (idx + 1) + '</span>' +
                    itemImgTag(item.label) +
                    '<span class="top-seller-name">' + escapeHtml(item.label) + '</span>' +
                    '<span class="top-seller-qty">' + (item.qty || 0) + ' sold</span>' +
                    '</div>';
            });
            topEl.innerHTML = html2;
        }

        // Top revenue items
        var revEl = $('#analytics-top-revenue');
        if ((State.topRevenue || []).length === 0) {
            revEl.innerHTML = '<div class="empty-state">No sales data yet</div>';
        } else {
            var html3 = '';
            State.topRevenue.slice(0, 10).forEach(function (item, idx) {
                html3 += '<div class="top-seller-row">' +
                    '<span class="top-seller-rank">' + (idx + 1) + '</span>' +
                    itemImgTag(item.label) +
                    '<span class="top-seller-name">' + escapeHtml(item.label) + '</span>' +
                    '<span class="top-seller-rev">' + fmt(item.revenue || 0) + '</span>' +
                    '</div>';
            });
            revEl.innerHTML = html3;
        }
    }

    /* ========================================================================
       SOLD ITEMS / STORAGE RENDERING
       ======================================================================== */

    function renderSoldItems() {
        var tbody = $('#storage-tbody');

        // Update capacity info
        var totalSoldQty = 0;
        State.soldItems.forEach(function (it) { totalSoldQty += (it.itemcount || 0); });
        var capEl = $('#storage-capacity-info');
        if (capEl) capEl.textContent = totalSoldQty + ' / ' + State.slots + ' slots used';

        // Update upgrade total display
        updateStorageUpgradeTotal();

        if (State.soldItems.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="3">No items in storage</td></tr>';
        } else {
            var html = '';
            State.soldItems.forEach(function (it) {
                var actions = '';
                if (hasPermission('depositItems')) {
                    actions = '<div style="display:flex;gap:4px;justify-content:center;align-items:center">' +
                        '<input type="number" class="input-field storage-take-qty-input" data-item="' + escapeHtml(it.itemname) + '" value="1" min="1" max="' + (it.itemcount || 1) + '" style="width:50px;text-align:center;padding:5px 4px;font-size:12px">' +
                        '<button class="btn-submit storage-take-btn" data-item="' + escapeHtml(it.itemname) + '" data-label="' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '" style="padding:5px 10px;font-size:11px">Take</button>' +
                        '<button class="btn-max storage-max-btn" data-item="' + escapeHtml(it.itemname) + '" style="padding:5px 10px;font-size:11px">Max</button>' +
                        '</div>';
                }
                html += '<tr>' +
                    '<td><div class="item-cell">' + itemImgTag(it.itemname) + '<span>' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span></div></td>' +
                    '<td style="text-align:center">' + (it.itemcount || 0) + '</td>' +
                    '<td>' + actions + '</td>' +
                    '</tr>';
            });
            tbody.innerHTML = html;
        }
        $('#storage-count').textContent = State.soldItems.length + ' items in storage';

        // Bind Take buttons
        $$('.storage-take-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                var qtyInput = $('input.storage-take-qty-input[data-item="' + itemName + '"]');
                var qty = parseInt(qtyInput ? qtyInput.value : 1) || 1;
                sendNUI('takeFromStorage', { shopId: State.shopId, itemName: itemName, label: label, quantity: qty });
            });
        });

        // Bind Max buttons: the server says how many can be taken (what is in
        // storage, capped by how many more the player can carry, which only
        // poggy_core knows).  Without an answer, the whole stored amount.
        $$('.storage-max-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var storageQty = 1;
                State.soldItems.forEach(function (si) {
                    if (si.itemname === itemName) storageQty = si.itemcount || 1;
                });
                sendNUI('storageMaxQty', { itemName: itemName }).then(function (resp) {
                    var n = (resp && typeof resp.max === 'number') ? resp.max : storageQty;
                    var qtyInput = $('input.storage-take-qty-input[data-item="' + itemName + '"]');
                    if (qtyInput) qtyInput.value = Math.max(0, Math.floor(n));
                });
            });
        });
    }

    function renderStoragePersonalInventory() {
        var tbody = $('#storage-player-inv-tbody');
        if (!tbody) return;
        var search = (State.storageInvSearch || '').toLowerCase();
        var filtered = State.personalItems.filter(function (it) {
            if (!search) return true;
            return (it.itemname || '').toLowerCase().indexOf(search) !== -1 ||
                   (it.itemlabel || '').toLowerCase().indexOf(search) !== -1;
        });

        if (filtered.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="4">' +
                (State.personalItems.length === 0 ? 'Click &ldquo;Refresh&rdquo; to load your inventory' : 'No matching items') +
                '</td></tr>';
            return;
        }

        var html = '';
        filtered.forEach(function (it) {
            html += '<tr>' +
                '<td><div class="item-cell">' + itemImgTag(it.itemname) + '<span>' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span></div></td>' +
                '<td style="text-align:center">' + (it.itemcount || 0) + '</td>' +
                '<td style="text-align:center"><input type="number" class="input-field storage-add-qty-input" data-item="' + escapeHtml(it.itemname) + '" value="1" min="1" max="' + (it.itemcount || 1) + '" style="width:100%;text-align:center;padding:5px 4px;font-size:12px"></td>' +
                '<td style="text-align:center"><button class="btn-submit storage-add-btn" data-item="' + escapeHtml(it.itemname) + '" data-label="' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '" style="padding:5px 10px;font-size:11px">Add</button></td>' +
                '</tr>';
        });
        tbody.innerHTML = html;

        $$('.storage-add-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                var qtyInput = $('input.storage-add-qty-input[data-item="' + itemName + '"]');
                var qty = parseInt(qtyInput ? qtyInput.value : 1) || 1;
                sendNUI('addToStorage', { shopId: State.shopId, itemName: itemName, label: label, quantity: qty });
            });
        });
    }

    function updateStorageUpgradeTotal() {
        var el = $('#storage-upgrade-amount');
        var totEl = $('#storage-upgrade-total');
        if (!el || !totEl) return;
        var amount = parseInt(el.value) || 0;
        totEl.textContent = '$' + (amount * State.upgradeCost).toFixed(2);
    }

    /* ========================================================================
       PERSONAL INVENTORY RENDERING
       ======================================================================== */

    function renderPersonalInventory() {
        var tbody = $('#personal-inv-tbody');
        var search = State.personalInvSearch.toLowerCase();
        var filtered = State.personalItems.filter(function (it) {
            if (!search) return true;
            return (it.itemname || '').toLowerCase().indexOf(search) !== -1 ||
                   (it.itemlabel || '').toLowerCase().indexOf(search) !== -1;
        });

        if (filtered.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="5">' + (State.personalItems.length === 0 ? 'No items in your inventory' : 'No matching items') + '</td></tr>';
            return;
        }

        var html = '';
        filtered.forEach(function (it) {
            // Check if already in shop stock and get existing price
            var existingPrice = '';
            State.inventory.forEach(function (inv) {
                if (inv.itemname === it.itemname) existingPrice = inv.itemprice || 0;
            });

            var priceCell;
            if (hasPermission('setPrices')) {
                priceCell = '<input type="number" class="input-field personal-price-input" data-item="' + escapeHtml(it.itemname) + '" value="' + existingPrice + '" min="0" step="0.01" placeholder="0.00" style="width:100%">';
            } else if (existingPrice !== '') {
                // Employee: show existing price read-only, still pass it via hidden input
                priceCell = fmt(existingPrice) + '<input type="hidden" class="personal-price-input" data-item="' + escapeHtml(it.itemname) + '" value="' + existingPrice + '">';
            } else {
                // Employee: no existing price and can't set one — disable add
                priceCell = '<span style="color:var(--text-secondary);font-style:italic">N/A</span><input type="hidden" class="personal-price-input" data-item="' + escapeHtml(it.itemname) + '" value="0">';
            }

            html += '<tr>' +
                    '<td><div class="item-cell">' + itemImgTag(it.itemname) + '<span>' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span></div></td>' +
                '<td style="text-align:center">' + (it.itemcount || 0) + '</td>' +
                '<td>' + priceCell + '</td>' +
                '<td style="text-align:center"><input type="number" class="input-field personal-qty-input" data-item="' + escapeHtml(it.itemname) + '" value="' + (it.itemcount || 1) + '" min="1" max="' + (it.itemcount || 1) + '" style="width:100%;text-align:center"></td>' +
                '<td style="text-align:center"><button class="btn-submit personal-add-btn" data-item="' + escapeHtml(it.itemname) + '" data-label="' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '" style="padding:6px 10px;font-size:11px">Add</button></td>' +
                '</tr>';
        });
        tbody.innerHTML = html;

        // Bind add-to-sell buttons
        $$('.personal-add-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var priceInput = $('input.personal-price-input[data-item="' + itemName + '"]');
                var qtyInput = $('input.personal-qty-input[data-item="' + itemName + '"]');
                var price = parseFloat(priceInput.value) || 0;
                var qty = parseInt(qtyInput.value) || 1;
                if (price <= 0) {
                    priceInput.style.borderColor = 'var(--red)';
                    return;
                }
                priceInput.style.borderColor = '';
                sendNUI('addItemToShop', { shopId: State.shopId, itemName: itemName, quantity: qty, price: price });
            });
        });
    }

    /* ========================================================================
       EVENT BINDINGS
       ======================================================================== */

    document.addEventListener('DOMContentLoaded', function () {

        // ── Close button ──
        $('#btn-close').addEventListener('click', function () {
            sendNUI('close', {});
            closeUI();
        });

        // ── ESC key ──
        document.addEventListener('keydown', function (e) {
            if (e.key === 'Escape' && State.visible) {
                sendNUI('close', {});
                closeUI();
            }
        });

        // ── Tab switching ──
        $$('.tab-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                switchTab(this.getAttribute('data-tab'));
            });
        });

        // ── Inventory search ──
        $('#inv-search').addEventListener('input', function () {
            State.invSearch = this.value;
            renderInventory();
        });

        // ── Personal inventory search ──
        $('#personal-inv-search').addEventListener('input', function () {
            State.personalInvSearch = this.value;
            renderPersonalInventory();
        });

        // ── Refresh personal inventory ──
        $('#btn-refresh-personal-inv').addEventListener('click', function () {
            sendNUI('getPersonalInventory', { shopId: State.shopId });
        });

        // ── Buy list search ──
        $('#buylist-search').addEventListener('input', function () {
            State.buylistSearch = this.value;
            renderBuyListItems();
        });

        // ── Buy list add form — clear selection ──
        $('#buylist-clear-selection').addEventListener('click', function () {
            State.buylistSelected = null;
            $('#buylist-add-form').classList.add('hidden');
            $$('.buylist-item').forEach(function (x) { x.classList.remove('selected'); });
        });

        // ── Buy list — add item ──
        $('#btn-add-buyitem').addEventListener('click', function () {
            if (!State.buylistSelected) return;
            var price = parseFloat($('#buylist-price').value) || 0;
            var amount = parseInt($('#buylist-amount').value) || 1;
            if (price <= 0) return;

            sendNUI('addBuyItem', {
                shopId: State.shopId,
                item: State.buylistSelected.name,
                label: State.buylistSelected.label,
                price: price,
                amount: amount,
            });

            // Clear form
            State.buylistSelected = null;
            $('#buylist-add-form').classList.add('hidden');
            $$('.buylist-item').forEach(function (x) { x.classList.remove('selected'); });
        });

        // ── Ledger — withdraw ──
        $('#btn-withdraw').addEventListener('click', function () {
            var amount = parseFloat($('#ledger-amount').value) || 0;
            if (amount <= 0) return;
            sendNUI('withdraw', { shopId: State.shopId, amount: amount });
            $('#ledger-amount').value = '';
        });

        // ── Ledger — deposit ──
        $('#btn-deposit').addEventListener('click', function () {
            var amount = parseFloat($('#ledger-amount').value) || 0;
            if (amount <= 0) return;
            sendNUI('deposit', { shopId: State.shopId, amount: amount });
            $('#ledger-amount').value = '';
        });

        // ── Ledger filters ──
        $('#ledger-filter-type').addEventListener('change', function () {
            State.ledgerFilterType = this.value;
            sendNUI('getTransactions', {
                shopId: State.shopId,
                filterType: State.ledgerFilterType,
                filterRange: State.ledgerFilterRange,
            });
        });

        $('#ledger-filter-range').addEventListener('change', function () {
            State.ledgerFilterRange = this.value;
            sendNUI('getTransactions', {
                shopId: State.shopId,
                filterType: State.ledgerFilterType,
                filterRange: State.ledgerFilterRange,
            });
        });

        // ── Settings — save name ──
        $('#btn-save-name').addEventListener('click', function () {
            var name = $('#settings-name').value.trim();
            if (!name) return;
            sendNUI('changeName', { shopId: State.shopId, name: name });
            State.shopName = name;
            $('#shop-title').textContent = name;
        });

        // ── Settings — blip toggle ──
        $('#settings-blip').addEventListener('change', function () {
            var val = this.checked ? 1 : 0;
            State.blip = val;
            $('#settings-blip-label').textContent = val === 1 ? 'Visible' : 'Hidden';
            sendNUI('setBlip', { shopId: State.shopId, blip: val });
        });

        // ── Settings — blip sprite ──
        var btnBlipSprite = $('#btn-save-blipsprite');
        if (btnBlipSprite) {
            btnBlipSprite.addEventListener('click', function () {
                var name = $('#settings-blipsprite').value.trim();
                if (!name) return;
                State.blipSprite = name;
                sendNUI('setBlipSprite', { shopId: State.shopId, spriteName: name });
            });
        }

        // ── Settings — upgrade slots ──
        $('#settings-upgrade-amount').addEventListener('input', updateUpgradeTotal);

        $('#btn-upgrade-slots').addEventListener('click', function () {
            var amount = parseInt($('#settings-upgrade-amount').value) || 0;
            if (amount <= 0) return;
            sendNUI('upgradeSlots', { shopId: State.shopId, amount: amount });
        });

        // ── Settings — webhook ──
        var btnWebhook = $('#btn-save-webhook');
        if (btnWebhook) {
            btnWebhook.addEventListener('click', function () {
                var url = $('#settings-webhook').value.trim();
                sendNUI('setWebhook', { shopId: State.shopId, url: url });
            });
        }

        // ── Storage — open sold items inventory ──
        // (Button removed; management is now inline on each row)

        // ── Storage — upgrade storage ──
        var storageUpgradeAmtEl = $('#storage-upgrade-amount');
        if (storageUpgradeAmtEl) {
            storageUpgradeAmtEl.addEventListener('input', updateStorageUpgradeTotal);
        }

        var btnStorageUpgrade = $('#btn-storage-upgrade');
        if (btnStorageUpgrade) {
            btnStorageUpgrade.addEventListener('click', function () {
                var amount = parseInt($('#storage-upgrade-amount').value) || 0;
                if (amount <= 0) return;
                sendNUI('upgradeSlots', { shopId: State.shopId, amount: amount });
            });
        }

        // ── Storage — refresh personal inventory ──
        var btnRefreshStorageInv = $('#btn-refresh-storage-inv');
        if (btnRefreshStorageInv) {
            btnRefreshStorageInv.addEventListener('click', function () {
                sendNUI('getPersonalInventory', { shopId: State.shopId });
            });
        }

        // ── Storage — search personal inventory ──
        var storageInvSearchEl = $('#storage-inv-search');
        if (storageInvSearchEl) {
            storageInvSearchEl.addEventListener('input', function () {
                State.storageInvSearch = this.value;
                renderStoragePersonalInventory();
            });
        }

        // ── Employees — refresh ──
        $('#btn-refresh-emp').addEventListener('click', function () {
            sendNUI('getEmployees', { shopId: State.shopId });
        });

        // ── Employees — search online players ──
        $('#emp-search').addEventListener('input', function () {
            State.empSearch = this.value;
            renderEmployees();
        });

        // ── Employees — clear selection ──
        $('#emp-clear-selection').addEventListener('click', function () {
            State.empSelectedPlayer = null;
            $('#emp-add-form').classList.add('hidden');
            $$('.emp-player-row').forEach(function (r) { r.classList.remove('selected'); });
        });

        // ── Employees — hire selected player ──
        $('#btn-hire-employee').addEventListener('click', function () {
            if (!State.empSelectedPlayer) return;
            var role = $('#emp-role-select').value || 'employee';
            sendNUI('addEmployee', {
                shopId: State.shopId,
                charId: State.empSelectedPlayer.charId,
                name:   State.empSelectedPlayer.name,
                role:   role,
            });
            State.empSelectedPlayer = null;
            $('#emp-add-form').classList.add('hidden');
            $$('.emp-player-row').forEach(function (r) { r.classList.remove('selected'); });
        });
    });

    /* ========================================================================
       EMPLOYEE MANAGEMENT — RENDER
       ======================================================================== */

    function renderEmployees() {
        var tbody = $('#employees-tbody');
        var data = State.employeeData;
        var directEmployees = (data && data.directEmployees) ? data.directEmployees : [];
        var jobEmployees    = (data && data.jobEmployees)    ? data.jobEmployees    : [];
        var onlinePlayers   = (data && data.onlinePlayers)   ? data.onlinePlayers   : [];

        // ── Current employees table ──
        var html = '';
        if (directEmployees.length === 0 && jobEmployees.length === 0) {
            html = '<tr><td class="empty-cell" colspan="4">No employees yet.</td></tr>';
        } else {
            // Job-based employees (read-only)
            jobEmployees.forEach(function (emp) {
                html += '<tr>' +
                    '<td>' + escapeHtml(emp.name || ('ID: ' + emp.charidentifier)) + '</td>' +
                    '<td><span class="emp-role-badge role-' + (emp.role || 'employee') + '">' + capitalize(emp.role || 'employee') + '</span></td>' +
                    '<td><span class="emp-source-badge emp-source-job">Via Job</span></td>' +
                    '<td><em style="color:var(--text-secondary);font-size:11px">Managed via job</em></td>' +
                    '</tr>';
            });
            // Direct employees (editable)
            var sj = (data && data.shopJob) ? data.shopJob : null;
            directEmployees.forEach(function (emp) {
                var cid = escapeHtml(String(emp.charidentifier));
                // With a shop job, the job grade this role is given.
                var gradeNote = '';
                if (sj && sj.job && emp.grade !== undefined && emp.grade !== null && sj.text && sj.text.grade) {
                    gradeNote = ' <span style="color:var(--text-secondary);font-size:11px">' +
                        escapeHtml(sj.text.grade.replace('%s', emp.grade)) + '</span>';
                }
                html += '<tr>' +
                    '<td>' + escapeHtml(emp.name || ('ID: ' + cid)) + '</td>' +
                    '<td>' +
                        '<select class="input-field emp-role-change" data-char="' + cid + '" style="padding:4px 6px;font-size:12px">' +
                            '<option value="employee"' + (emp.role === 'employee' ? ' selected' : '') + '>Employee</option>' +
                            '<option value="manager"' + (emp.role === 'manager'  ? ' selected' : '') + '>Manager</option>' +
                        '</select>' + gradeNote +
                    '</td>' +
                    '<td><span class="emp-source-badge emp-source-direct">Direct</span></td>' +
                    '<td style="display:flex;gap:6px">' +
                        '<button class="btn-submit emp-save-role-btn" data-char="' + cid + '" style="padding:5px 10px;font-size:11px">Save</button>' +
                        '<button class="btn-outline emp-remove-btn" data-char="' + cid + '" style="padding:5px 10px;font-size:11px;color:var(--red);border-color:var(--red)">Remove</button>' +
                    '</td>' +
                    '</tr>';
            });
        }
        tbody.innerHTML = html;

        // Bind save-role buttons.  The character id stays text: RSG and QBR
        // ids are citizenids such as 'ABC12345', which parseInt turned into NaN.
        $$('.emp-save-role-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var charId = this.getAttribute('data-char');
                var sel   = null;
                $$('.emp-role-change').forEach(function (s) {
                    if (s.getAttribute('data-char') === charId) sel = s;
                });
                var role  = sel ? sel.value : 'employee';
                sendNUI('setEmployeeRole', { shopId: State.shopId, charId: charId, role: role });
            });
        });

        // Bind remove buttons
        $$('.emp-remove-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var charId = this.getAttribute('data-char');
                sendNUI('removeEmployee', { shopId: State.shopId, charId: charId });
            });
        });

        renderShopJob(data && data.shopJob);

        // ── Online player list ──
        var playerList = $('#emp-add-player-list');
        var search = State.empSearch.toLowerCase();
        var filtered = onlinePlayers.filter(function (p) {
            if (!search) return true;
            return (p.name || '').toLowerCase().indexOf(search) !== -1 ||
                   String(p.charId).indexOf(search) !== -1;
        });

        if (filtered.length === 0) {
            playerList.innerHTML = '<p class="emp-hint">' + (onlinePlayers.length === 0 ? 'No online players.' : 'No matching players.') + '</p>';
        } else {
            var phtml = '';
            filtered.forEach(function (p) {
                phtml += '<div class="emp-player-row" data-char="' + escapeHtml(String(p.charId)) + '" data-name="' + escapeHtml(p.name) + '">' +
                    '<span class="emp-player-name">' + escapeHtml(p.name) + '</span>' +
                    '<span class="emp-player-id">#' + escapeHtml(String(p.charId)) + '</span>' +
                    '</div>';
            });
            playerList.innerHTML = phtml;
        }

        // Bind player-row click → select for hiring
        $$('.emp-player-row').forEach(function (row) {
            row.addEventListener('click', function () {
                $$('.emp-player-row').forEach(function (r) { r.classList.remove('selected'); });
                this.classList.add('selected');
                State.empSelectedPlayer = {
                    // Text, not parseInt: RSG/QBR ids are citizenids.
                    charId: this.getAttribute('data-char'),
                    name:   this.getAttribute('data-name'),
                };
                $('#emp-selected-name').textContent = escapeHtml(State.empSelectedPlayer.name);
                $('#emp-add-form').classList.remove('hidden');
            });
        });
    }

    /* ========================================================================
       EMPLOYEE MANAGEMENT — SHOP JOB (Config.ShopJobs)
       A locked, read-only title for everyone who sees this tab, admins too.
       Only server staff change a shop's job, with /pmshopjob.
       ======================================================================== */

    function renderShopJob(sj) {
        var box = $('#emp-shopjob');
        if (!box) return;
        if (!sj || !sj.enabled) {
            box.classList.add('hidden');
            return;
        }
        var t = sj.text || {};
        var title = $('#emp-shopjob-title');
        if (sj.job) {
            var name = (sj.label && sj.label !== sj.job) ? (sj.label + ' (' + sj.job + ')') : sj.job;
            title.textContent = (t.locked || 'Shop job: %s').replace('%s', name) + ' \uD83D\uDD12 ' + (t.setBy || '');
        } else {
            title.textContent = t.none || '';
        }
        box.title = t.tooltip || '';
        box.classList.remove('hidden');
    }

    function capitalize(str) {
        return str ? str.charAt(0).toUpperCase() + str.slice(1) : str;
    }

    function updateUpgradeTotal() {
        var amount = parseInt($('#settings-upgrade-amount').value) || 0;
        var total = (amount * State.upgradeCost).toFixed(2);
        $('#settings-upgrade-total').textContent = '$' + total;
    }

    /* ========================================================================
       CUSTOMER SHOP INTERFACE — STATE
       ======================================================================== */

    var CustState = {
        visible: false,
        shopId: 0,
        shopName: '',
        shopType: 0,       // 1=static, 2=player, 3=society
        buyItems: [],      // items available for purchase  { itemname, itemlabel, itemcount, itemprice, category }
        sellBuyList: [],   // items the shop wants to buy   { name, label, price, amount, category }
        playerItems: [],   // player's inventory            { itemname, itemlabel, itemcount }
        categories: [],    // available category names
        buySearch: '',
        sellSearch: '',
        buyCategory: '',
        sellCategory: '',
        showOwnedOnly: true,
        shopLedger: 0,         // shop funds available (player-owned/society only)
        shopMaxSlots: 0,       // max storage slots
        shopStoredCount: 0,    // current items stored in shop
    };

    // Buy tab layout: 'list' (the table) or 'grid' (tiles plus a detail bar).
    // Remembered per player in the NUI's local storage.  Storage can be
    // unavailable, in which case the default is used and nothing breaks.
    var BUY_VIEW_KEY = 'poggy_markets.buyView';
    CustState.buyView = (function () {
        try { return localStorage.getItem(BUY_VIEW_KEY) === 'grid' ? 'grid' : 'list'; }
        catch (e) { return 'list'; }
    })();
    CustState.gridSelected = null;   // item name chosen in the grid
    CustState.gridQty = 1;           // quantity typed in the detail bar

    /* ========================================================================
       CUSTOMER — NUI MESSAGE HANDLER CASES
       ======================================================================== */

    window.addEventListener('message', function (event) {
        var data = event.data;
        switch (data.type) {
            case 'custOpen':
                if (data.imageBase) IMG_BASE = data.imageBase;
                custOpenUI(data);
                break;
            case 'custClose':
                custCloseUI();
                break;
            case 'custRefreshPlayerItems':
                if (data.playerItems) {
                    CustState.playerItems = data.playerItems;
                    renderCustSellList();
                }
                break;
            case 'custRefreshShopData':
                if (data.buyItems) CustState.buyItems = data.buyItems;
                if (data.sellBuyList) CustState.sellBuyList = data.sellBuyList;
                if (data.playerItems) CustState.playerItems = data.playerItems;
                if (data.categories) {
                    CustState.categories = data.categories;
                    populateCategoryDropdowns();
                }
                if (data.shopLedger !== undefined) CustState.shopLedger = data.shopLedger;
                if (data.shopMaxSlots !== undefined) CustState.shopMaxSlots = data.shopMaxSlots;
                if (data.shopStoredCount !== undefined) CustState.shopStoredCount = data.shopStoredCount;
                renderCustBuyList();
                renderCustSellList();
                break;
        }
    });

    /* ========================================================================
       CUSTOMER — OPEN / CLOSE
       ======================================================================== */

    function custOpenUI(data) {
        CustState.visible      = true;
        CustState.shopId       = data.shopId || 0;
        CustState.shopName     = data.shopName || 'Shop';
        CustState.shopType     = data.shopType || 1;
        CustState.buyItems     = data.buyItems || [];
        CustState.sellBuyList  = data.sellBuyList || [];
        CustState.playerItems  = data.playerItems || [];
        CustState.categories   = data.categories || [];
        CustState.buySearch    = '';
        CustState.sellSearch   = '';
        CustState.buyCategory  = '';
        CustState.sellCategory = '';
        CustState.showOwnedOnly = true;
        CustState.gridSelected = null;
        CustState.gridQty = 1;
        CustState.shopLedger     = data.shopLedger || 0;
        CustState.shopMaxSlots   = data.shopMaxSlots || 0;
        CustState.shopStoredCount = data.shopStoredCount || 0;

        // Capitals come from the stylesheet, so a skin can show the name as typed.
        $('#cust-shop-title').textContent = CustState.shopName;
        $('#cust-buy-search').value = '';
        $('#cust-sell-search').value = '';
        $('#cust-sell-owned-toggle').checked = true;

        populateCategoryDropdowns();
        custSwitchTab(CustState.buyItems.length > 0 ? 'buy' : 'sell');
        renderCustBuyList();
        renderCustSellList();

        $('#customer-container').classList.remove('hidden');
    }

    function populateCategoryDropdowns() {
        var cats = CustState.categories || [];
        var buySelect = $('#cust-buy-category');
        var sellSelect = $('#cust-sell-category');
        var optionsHtml = '<option value="">All Categories</option>';
        cats.forEach(function (c) {
            optionsHtml += '<option value="' + escapeHtml(c) + '">' + escapeHtml(c.charAt(0).toUpperCase() + c.slice(1)) + '</option>';
        });
        if (buySelect) { buySelect.innerHTML = optionsHtml; buySelect.value = ''; }
        if (sellSelect) { sellSelect.innerHTML = optionsHtml; sellSelect.value = ''; }
    }

    function custCloseUI() {
        CustState.visible = false;
        $('#customer-container').classList.add('hidden');
    }

    /* ========================================================================
       CUSTOMER — TAB SWITCHING
       ======================================================================== */

    function custSwitchTab(tabName) {
        $$('.cust-tab-btn').forEach(function (btn) {
            btn.classList.toggle('active', btn.getAttribute('data-cust-tab') === tabName);
        });
        $$('.cust-tab-content').forEach(function (tc) {
            tc.classList.toggle('active', tc.id === 'cust-tab-' + tabName);
        });
    }

    /* ========================================================================
       CUSTOMER — RENDER BUY LIST
       ======================================================================== */

    function renderCustBuyList() {
        var tbody = $('#cust-buy-tbody');
        var filtered = custFilteredBuyItems();

        // The grid is a second layout over the same data, so search, the
        // category filter and post-purchase refreshes all drive it for free.
        applyBuyView();
        if (CustState.buyView === 'grid') {
            renderCustBuyGrid(filtered);
            return;
        }

        if (filtered.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="6">No items available for purchase</td></tr>';
            return;
        }

        var html = '';
        filtered.forEach(function (it) {
            var stock = (CustState.shopType === 1) ? '∞' : (it.itemcount || 0);
            var stockClass = '';
            if (CustState.shopType !== 1) {
                if ((it.itemcount || 0) === 0) stockClass = ' class="cust-stock-out"';
                else if ((it.itemcount || 0) < 5) stockClass = ' class="cust-stock-low"';
            }
            var maxQty = custBuyMaxQty(it);
            var unitPrice = it.itemprice || 0;

            html += '<tr>' +
                '<td><div class="item-cell">' + itemImgTag(it.itemname) + '<span>' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span></div></td>' +
                '<td style="text-align:center"' + stockClass + '>' + stock + '</td>' +
                '<td style="text-align:right">' + fmt(unitPrice) + '</td>' +
                '<td style="text-align:center">' +
                    '<div class="cust-qty-group">' +
                        '<input type="number" class="input-field cust-buy-qty" data-item="' + escapeHtml(it.itemname) + '" data-price="' + unitPrice + '" value="1" min="1" max="' + maxQty + '">' +
                        '<button class="btn-max cust-buy-max-btn" data-item="' + escapeHtml(it.itemname) + '" data-max="' + maxQty + '">Max</button>' +
                    '</div>' +
                '</td>' +
                '<td style="text-align:right" class="cust-buy-total-cell" data-item="' + escapeHtml(it.itemname) + '">' + fmt(unitPrice) + '</td>' +
                '<td style="text-align:center">' +
                    '<button class="cust-btn-buy" data-item="' + escapeHtml(it.itemname) + '" data-label="' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '" data-price="' + unitPrice + '">Buy</button>' +
                '</td>' +
                '</tr>';
        });
        tbody.innerHTML = html;

        // Bind qty change → update total cell
        $$('.cust-buy-qty').forEach(function (inp) {
            function updateTotal() {
                var itemName = inp.getAttribute('data-item');
                var price = parseFloat(inp.getAttribute('data-price')) || 0;
                var qty = parseInt(inp.value) || 1;
                var cell = $('td.cust-buy-total-cell[data-item="' + itemName + '"]');
                if (cell) cell.textContent = fmt(price * qty);
            }
            inp.addEventListener('input', updateTotal);
            inp.addEventListener('change', updateTotal);
        });

        // Bind Max buttons: the server says how many can actually be bought.
        $$('.cust-buy-max-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var fallback = parseInt(this.getAttribute('data-max')) || 1;
                custAskMax(itemName, fallback, function (max) {
                    var input = $('input.cust-buy-qty[data-item="' + itemName + '"]');
                    if (!input) return;
                    input.max = Math.max(1, max);
                    input.value = max;
                    input.dispatchEvent(new Event('input'));
                });
            });
        });

        // Bind Buy buttons
        $$('.cust-btn-buy').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                var price = parseFloat(this.getAttribute('data-price')) || 0;
                var input = $('input.cust-buy-qty[data-item="' + itemName + '"]');
                var qty = parseInt(input.value) || 1;
                if (qty < 1) return;
                sendNUI('custBuyItem', {
                    shopId: CustState.shopId,
                    shopType: CustState.shopType,
                    itemName: itemName,
                    label: label,
                    qty: qty,
                    price: price,
                });
            });
        });
    }

    /* ========================================================================
       CUSTOMER — BUY GRID
       A second layout for the Buy tab: a 4-wide grid of tiles with the chosen
       item's details and Buy button pinned underneath.  It reads the same
       filtered list and sends the same custBuyItem call as the table.
       ======================================================================== */

    function custFilteredBuyItems() {
        var search = CustState.buySearch.toLowerCase();
        var catFilter = CustState.buyCategory;
        return CustState.buyItems.filter(function (it) {
            if ((it.itemcount || 0) <= 0 && CustState.shopType !== 1) return false;
            if (catFilter && (it.category || 'default') !== catFilter) return false;
            if (!search) return true;
            return (it.itemname || '').toLowerCase().indexOf(search) !== -1 ||
                   (it.itemlabel || '').toLowerCase().indexOf(search) !== -1;
        });
    }

    /* What Max offers before the server answers, or if it never does: the
       stock, or 999 at a store with unlimited stock.  How many the player can
       carry is only known server-side (poggy_core), so Max asks for it; see
       custAskMax.  Shared by the table and the grid. */
    function custBuyMaxQty(it) {
        return (CustState.shopType === 1) ? 999 : Math.max(1, it.itemcount || 1);
    }

    /* Max: ask the server how many the customer can buy right now (the stock,
       capped by how many more they can carry) and hand that to `apply`.
       Falls back to `fallback` when no answer comes. */
    function custAskMax(itemName, fallback, apply) {
        sendNUI('custMaxQty', { itemName: itemName }).then(function (resp) {
            var n = (resp && typeof resp.max === 'number') ? resp.max : fallback;
            apply(Math.max(0, Math.floor(n)));
        });
    }

    function custFindBuyItem(name) {
        for (var i = 0; i < CustState.buyItems.length; i++) {
            if (CustState.buyItems[i].itemname === name) return CustState.buyItems[i];
        }
        return null;
    }

    function setBuyView(view) {
        CustState.buyView = (view === 'grid') ? 'grid' : 'list';
        try { localStorage.setItem(BUY_VIEW_KEY, CustState.buyView); } catch (e) { /* not persisted */ }
        renderCustBuyList();
    }

    /* Show whichever layout is chosen and light up its toggle button. */
    function applyBuyView() {
        var grid = CustState.buyView === 'grid';
        $$('.cust-view-btn').forEach(function (btn) {
            btn.classList.toggle('active', btn.getAttribute('data-cust-view') === CustState.buyView);
        });
        var table = $('#cust-buy-table-wrapper');
        var gridWrap = $('#cust-buy-grid-wrapper');
        var selection = $('#cust-grid-selection');
        if (table) table.classList.toggle('hidden', grid);
        if (gridWrap) gridWrap.classList.toggle('hidden', !grid);
        if (selection) selection.classList.toggle('hidden', !grid);
    }

    function renderCustBuyGrid(filtered) {
        var gridEl = $('#cust-buy-grid');
        if (!gridEl) return;
        hideGridTip();   // the tiles are about to be rebuilt

        // Keep the chosen item across refreshes (every purchase re-renders),
        // but let it go once it is sold out or filtered away.
        var stillThere = CustState.gridSelected && filtered.some(function (it) {
            return it.itemname === CustState.gridSelected;
        });
        if (!stillThere) {
            CustState.gridSelected = null;
            CustState.gridQty = 1;
        }

        if (filtered.length === 0) {
            gridEl.innerHTML = '<div class="cust-grid-empty">No items available for purchase</div>';
            renderCustGridSelection();
            return;
        }

        var unlimited = CustState.shopType === 1;
        var html = '';
        filtered.forEach(function (it) {
            var count = it.itemcount || 0;
            // Stock badge only when there is more than one.  Unlimited NPC
            // stock would put the same badge on every tile, so it is left off;
            // the detail bar still shows it as infinite.
            var badge = (!unlimited && count > 1)
                ? '<span class="cust-grid-qty">' + count + '</span>'
                : '';
            var selected = (it.itemname === CustState.gridSelected) ? ' selected' : '';
            // Image only.  Name, price and stock live in the detail bar below.
            html += '<button type="button" class="cust-grid-tile' + selected + '" data-item="' + escapeHtml(it.itemname) + '">' +
                itemImgTag(it.itemname) + badge +
                '</button>';
        });
        gridEl.innerHTML = html;
        renderCustGridSelection();
    }

    /* The detail bar under the grid: item, stock, price, qty, total and Buy. */
    function renderCustGridSelection() {
        var panel = $('#cust-grid-selection');
        if (!panel) return;

        var it = CustState.gridSelected ? custFindBuyItem(CustState.gridSelected) : null;
        if (!it) {
            panel.innerHTML = '<div class="cust-sel-empty">Select an item to see its details.</div>';
            return;
        }

        var unlimited = CustState.shopType === 1;
        var count = it.itemcount || 0;
        var stockClass = '';
        if (!unlimited) {
            if (count === 0) stockClass = ' cust-stock-out';
            else if (count < 5) stockClass = ' cust-stock-low';
        }
        var price = it.itemprice || 0;
        var max = custBuyMaxQty(it);
        var qty = Math.min(Math.max(1, CustState.gridQty || 1), max);
        CustState.gridQty = qty;
        var label = niceLabel(it.itemname, it.itemlabel);

        panel.innerHTML =
            '<div class="cust-sel-item">' + itemImgTag(it.itemname) +
                '<span class="cust-sel-name">' + escapeHtml(label) + '</span>' +
            '</div>' +
            '<div class="cust-sel-stat"><span class="cust-sel-key">Stock</span>' +
                '<span class="cust-sel-val' + stockClass + '">' + (unlimited ? '∞' : count) + '</span></div>' +
            '<div class="cust-sel-stat"><span class="cust-sel-key">Price</span>' +
                '<span class="cust-sel-val">' + fmt(price) + '</span></div>' +
            '<div class="cust-sel-stat"><span class="cust-sel-key">Qty</span>' +
                '<div class="cust-qty-group">' +
                    '<input type="number" class="input-field" id="cust-sel-qty" value="' + qty + '" min="1" max="' + max + '">' +
                    '<button type="button" class="btn-max" id="cust-sel-max">Max</button>' +
                '</div></div>' +
            '<div class="cust-sel-stat"><span class="cust-sel-key">Total</span>' +
                '<span class="cust-sel-val cust-sel-total" id="cust-sel-total">' + fmt(price * qty) + '</span></div>' +
            '<button type="button" class="cust-btn-buy cust-sel-buy" id="cust-sel-buy">Buy</button>';

        var qtyInput = $('#cust-sel-qty');
        var totalEl = $('#cust-sel-total');
        function readQty() {
            var n = parseInt(qtyInput.value, 10);
            return isNaN(n) ? 1 : n;
        }
        function updateTotal() {
            CustState.gridQty = readQty();
            totalEl.textContent = fmt(price * Math.max(1, CustState.gridQty));
        }
        qtyInput.addEventListener('input', updateTotal);
        qtyInput.addEventListener('change', function () {
            // Settle the typed value into range once the player is done typing.
            qtyInput.value = Math.min(Math.max(1, readQty()), max);
            updateTotal();
        });
        $('#cust-sel-max').addEventListener('click', function () {
            custAskMax(it.itemname, max, function (n) {
                max = Math.max(1, n);      // typed amounts are held to this from now on
                qtyInput.max = max;
                qtyInput.value = n;
                updateTotal();
            });
        });
        $('#cust-sel-buy').addEventListener('click', function () {
            var n = readQty();
            if (n < 1) return;
            CustState.gridQty = 1;   // the refresh after buying starts from 1
            sendNUI('custBuyItem', {
                shopId: CustState.shopId,
                shopType: CustState.shopType,
                itemName: it.itemname,
                label: label,
                qty: n,
                price: price,
            });
        });
    }

    /* Hover tooltip for the grid: the tile's name and price, since the tiles
       show only the image.  One shared element sits just above the hovered
       tile.  It lives outside the scrolling grid, so the top row's tooltip is
       never cut off. */
    function showGridTip(tile) {
        var tip = $('#cust-grid-tip');
        var it = custFindBuyItem(tile.getAttribute('data-item'));
        // offsetParent is null while the Buy tab is hidden.
        if (!tip || !tip.offsetParent || !it) { hideGridTip(); return; }

        tip.innerHTML =
            '<span class="cust-grid-tip-name">' + escapeHtml(niceLabel(it.itemname, it.itemlabel)) + '</span>' +
            '<span class="cust-grid-tip-price">' + fmt(it.itemprice || 0) + '</span>';

        // Centre it on the tile's top edge; the CSS lifts it clear of the tile.
        // Rects are in screen pixels and left/top in the panel's own pixels,
        // which differ if the panel is ever scaled.
        var host = tip.offsetParent;
        var hostRect = host.getBoundingClientRect();
        var tileRect = tile.getBoundingClientRect();
        var scale = host.offsetWidth ? hostRect.width / host.offsetWidth : 1;
        tip.style.left = ((tileRect.left + tileRect.width / 2 - hostRect.left) / scale - host.clientLeft) + 'px';
        tip.style.top = ((tileRect.top - hostRect.top) / scale - host.clientTop) + 'px';
        tip.classList.add('show');
    }

    function hideGridTip() {
        var tip = $('#cust-grid-tip');
        if (tip) tip.classList.remove('show');
    }

    /* ========================================================================
       CUSTOMER — RENDER SELL LIST
       ======================================================================== */

    /* How many of `name` the player is carrying.
       Keyed lower-case on purpose: catalogs and item tables disagree about
       casing far too often, and a mismatch here silently reads as "you have 0"
       for an item sitting in the player's satchel. */
    function custPlayerHeld(name) {
        var want = String(name || '').toLowerCase();
        var total = 0;
        (CustState.playerItems || []).forEach(function (pi) {
            if (String(pi.itemname || '').toLowerCase() === want) {
                total += (pi.itemcount || 0);
            }
        });
        return total;
    }

    function renderCustSellList() {
        var tbody = $('#cust-sell-tbody');
        var totalBar = $('#cust-sell-total-bar');
        var search = CustState.sellSearch.toLowerCase();
        var catFilter = CustState.sellCategory;

        var playerHeld = custPlayerHeld;

        // Filter the buy list
        var filtered = CustState.sellBuyList.filter(function (bi) {
            var playerHas = playerHeld(bi.name);
            // If showOwnedOnly, only show items player has
            if (CustState.showOwnedOnly && playerHas <= 0) return false;
            if (catFilter && (bi.category || 'default') !== catFilter) return false;
            if (!search) return true;
            return (bi.name || '').toLowerCase().indexOf(search) !== -1 ||
                   (bi.label || '').toLowerCase().indexOf(search) !== -1;
        });

        if (filtered.length === 0) {
            tbody.innerHTML = '<tr><td class="empty-cell" colspan="6">' +
                (CustState.sellBuyList.length === 0 ? 'This shop does not buy items' : 'No matching items') +
                '</td></tr>';
            totalBar.innerHTML = '';
            return;
        }

        var html = '';
        var totalPreview = 0;
        var isPlayerShop = (CustState.shopType === 2 || CustState.shopType === 3);
        var simLedger = CustState.shopLedger || 0;
        var simStored = CustState.shopStoredCount || 0;
        var simMaxSlots = CustState.shopMaxSlots || 999999;
        var ledgerCapped = false;
        var slotsCapped = false;

        filtered.forEach(function (bi) {
            var playerHas = playerHeld(bi.name);
            // amount < 0 means the shop will take as many as you have; only a
            // positive number is a real cap.
            var wanted = (typeof bi.amount === 'number' && bi.amount >= 0)
                ? bi.amount : Infinity;
            var maxSell = Math.max(0, Math.min(playerHas, wanted));
            var defaultQty = Math.max(1, CustState.showOwnedOnly ? maxSell : 1);

            // Simulate server-side constraints for player-owned/society shops
            var previewQty = (playerHas > 0 ? maxSell : 0);
            if (isPlayerShop && previewQty > 0) {
                var lineCost = (bi.price || 0) * previewQty;
                // Check ledger
                if (lineCost > simLedger) {
                    previewQty = Math.floor(simLedger / (bi.price || 1));
                    if (previewQty < maxSell) ledgerCapped = true;
                }
                // Check slot space
                var slotsLeft = simMaxSlots - simStored;
                if (previewQty > slotsLeft) {
                    previewQty = Math.max(0, slotsLeft);
                    slotsCapped = true;
                }
                var actualCost = Math.round((bi.price || 0) * previewQty * 100) / 100;
                totalPreview += actualCost;
                simLedger -= actualCost;
                simStored += previewQty;
            } else {
                totalPreview += Math.round((bi.price || 0) * previewQty * 100) / 100;
            }

            html += '<tr>' +
                '<td><div class="item-cell">' + itemImgTag(bi.name) + '<span>' + escapeHtml(niceLabel(bi.name, bi.label)) + '</span></div></td>' +
                '<td style="text-align:center">' + fmt(bi.price) + '</td>' +
                '<td style="text-align:center">' + playerHas + '</td>' +
                '<td style="text-align:center">' +
                    '<div class="cust-qty-group">' +
                        '<input type="number" class="input-field cust-sell-qty" data-item="' + escapeHtml(bi.name) + '" data-price="' + (bi.price || 0) + '" value="' + defaultQty + '" min="1" max="' + Math.max(1, maxSell) + '"' + (playerHas <= 0 ? ' disabled' : '') + '>' +
                        '<button class="btn-max cust-sell-max-btn" data-item="' + escapeHtml(bi.name) + '" data-max="' + maxSell + '"' + (playerHas <= 0 ? ' disabled' : '') + '>Max</button>' +
                    '</div>' +
                '</td>' +
                '<td style="text-align:right" class="cust-sell-total-cell" data-item="' + escapeHtml(bi.name) + '">' +
                    fmt((bi.price || 0) * (playerHas > 0 ? defaultQty : 0)) +
                '</td>' +
                '<td style="text-align:center">' +
                    '<button class="cust-btn-sell" data-item="' + escapeHtml(bi.name) + '" data-label="' + escapeHtml(niceLabel(bi.name, bi.label)) + '" data-price="' + (bi.price || 0) + '"' + (playerHas <= 0 ? ' disabled' : '') + '>Sell</button>' +
                    (playerHas > 0 ? ' <button class="cust-btn-sell-all-row" data-item="' + escapeHtml(bi.name) + '" data-label="' + escapeHtml(niceLabel(bi.name, bi.label)) + '" data-price="' + (bi.price || 0) + '" data-max="' + maxSell + '">Sell All</button>' : '') +
                '</td>' +
                '</tr>';
        });
        tbody.innerHTML = html;

        // Total bar
        if (totalPreview > 0) {
            var suffix = '';
            if (ledgerCapped) suffix = ' (limited by shop funds)';
            else if (slotsCapped) suffix = ' (limited by shop storage)';
            totalBar.innerHTML = '<span>Potential total if you sell everything: <strong>' + fmt(totalPreview) + '</strong>' + suffix + '</span>';
        } else {
            totalBar.innerHTML = '';
        }

        // Bind sell max buttons
        // Keep the earnings cell in step with the qty box.
        function updateSellTotal(itemName) {
            var input = $('input.cust-sell-qty[data-item="' + itemName + '"]');
            var cell  = $('td.cust-sell-total-cell[data-item="' + itemName + '"]');
            if (!input || !cell) return;
            var qty   = parseInt(input.value) || 0;
            var price = parseFloat(input.getAttribute('data-price')) || 0;
            cell.textContent = fmt(price * Math.max(0, qty));
        }

        $$('.cust-sell-qty').forEach(function (inp) {
            inp.addEventListener('input', function () {
                var max = parseInt(this.getAttribute('max')) || 1;
                var v   = parseInt(this.value) || 0;
                // Clamp here as well as in the markup: typing straight into a
                // number field bypasses min/max entirely.
                if (v > max) { this.value = max; }
                if (v < 1 && this.value !== '') { this.value = 1; }
                updateSellTotal(this.getAttribute('data-item'));
            });
        });

        $$('.cust-sell-max-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var max = parseInt(this.getAttribute('data-max')) || 1;
                var input = $('input.cust-sell-qty[data-item="' + itemName + '"]');
                if (input) input.value = Math.max(1, max);
                updateSellTotal(itemName);
            });
        });

        // Bind sell buttons (single item)
        $$('.cust-btn-sell').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                var price = parseFloat(this.getAttribute('data-price')) || 0;
                var input = $('input.cust-sell-qty[data-item="' + itemName + '"]');
                var qty = parseInt(input.value) || 1;
                if (qty < 1) return;
                sendNUI('custSellItem', {
                    shopId: CustState.shopId,
                    shopType: CustState.shopType,
                    itemName: itemName,
                    label: label,
                    qty: qty,
                    price: price,
                });
            });
        });

        // Bind "Sell All" per-row buttons
        $$('.cust-btn-sell-all-row').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var itemName = this.getAttribute('data-item');
                var label = this.getAttribute('data-label');
                var price = parseFloat(this.getAttribute('data-price')) || 0;
                var max = parseInt(this.getAttribute('data-max')) || 1;
                sendNUI('custSellItem', {
                    shopId: CustState.shopId,
                    shopType: CustState.shopType,
                    itemName: itemName,
                    label: label,
                    qty: max,
                    price: price,
                });
            });
        });
    }

    /* ========================================================================
       CUSTOMER — EVENT BINDINGS (inside DOMContentLoaded)
       ======================================================================== */

    document.addEventListener('DOMContentLoaded', function () {

        // ── Close button ──
        var custClose = $('#cust-btn-close');
        if (custClose) {
            custClose.addEventListener('click', function () {
                sendNUI('custClose', {});
                custCloseUI();
            });
        }

        // ── ESC key (extend existing handler) ──
        document.addEventListener('keydown', function (e) {
            if (e.key === 'Escape' && CustState.visible) {
                sendNUI('custClose', {});
                custCloseUI();
            }
        });

        // ── Tab switching ──
        $$('.cust-tab-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                custSwitchTab(this.getAttribute('data-cust-tab'));
            });
        });

        // ── Buy search ──
        var custBuySearch = $('#cust-buy-search');
        if (custBuySearch) {
            custBuySearch.addEventListener('input', function () {
                CustState.buySearch = this.value;
                renderCustBuyList();
            });
        }

        // ── Buy category filter ──
        var custBuyCat = $('#cust-buy-category');
        if (custBuyCat) {
            custBuyCat.addEventListener('change', function () {
                CustState.buyCategory = this.value;
                renderCustBuyList();
            });
        }

        // ── Buy layout: list or grid ──
        $$('.cust-view-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                setBuyView(this.getAttribute('data-cust-view'));
            });
        });

        // ── Grid: choosing a tile ──
        // One delegated listener, because the tiles are rebuilt on every refresh.
        var custBuyGrid = $('#cust-buy-grid');
        if (custBuyGrid) {
            custBuyGrid.addEventListener('click', function (e) {
                var tile = e.target.closest('.cust-grid-tile');
                if (!tile) return;
                var name = tile.getAttribute('data-item');
                if (name !== CustState.gridSelected) CustState.gridQty = 1;
                CustState.gridSelected = name;
                $$('.cust-grid-tile').forEach(function (t) {
                    t.classList.toggle('selected', t === tile);
                });
                renderCustGridSelection();
            });

            // ── Grid: hover tooltip with the name and price ──
            custBuyGrid.addEventListener('mouseover', function (e) {
                var tile = e.target.closest('.cust-grid-tile');
                if (tile) showGridTip(tile);
                else hideGridTip();     // over a gap between tiles
            });
            custBuyGrid.addEventListener('mouseleave', hideGridTip);
        }
        // Scrolling slides the tiles out from under the tooltip.
        var custBuyGridWrap = $('#cust-buy-grid-wrapper');
        if (custBuyGridWrap) custBuyGridWrap.addEventListener('scroll', hideGridTip);

        // ── Sell search ──
        var custSellSearch = $('#cust-sell-search');
        if (custSellSearch) {
            custSellSearch.addEventListener('input', function () {
                CustState.sellSearch = this.value;
                renderCustSellList();
            });
        }

        // ── Sell category filter ──
        var custSellCat = $('#cust-sell-category');
        if (custSellCat) {
            custSellCat.addEventListener('change', function () {
                CustState.sellCategory = this.value;
                renderCustSellList();
            });
        }

        // ── Toggle "Only my items" ──
        var custToggle = $('#cust-sell-owned-toggle');
        if (custToggle) {
            custToggle.addEventListener('change', function () {
                CustState.showOwnedOnly = this.checked;
                renderCustSellList();
            });
        }

        // ── Global "Sell All" button ──
        var custSellAllBtn = $('#cust-sell-all-btn');
        if (custSellAllBtn) {
            custSellAllBtn.addEventListener('click', function () {
                var sellList = [];
                CustState.sellBuyList.forEach(function (bi) {
                    var playerHas = custPlayerHeld(bi.name);
                    if (playerHas <= 0) return;
                    var qty = Math.min(playerHas, bi.amount || 999);
                    sellList.push({
                        itemName: bi.name,
                        label: bi.label || bi.name,
                        qty: qty,
                        price: bi.price || 0,
                    });
                });

                if (sellList.length === 0) return;

                sendNUI('custSellAll', {
                    shopId: CustState.shopId,
                    shopType: CustState.shopType,
                    items: sellList,
                });
            });
        }
    });

})();
