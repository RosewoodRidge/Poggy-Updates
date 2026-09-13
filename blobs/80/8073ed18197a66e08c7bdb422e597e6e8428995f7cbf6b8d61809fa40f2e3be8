/* ============================================================================
   poggy_auction — NUI Script (Vanilla JS)
   Auction House UI controller
   ============================================================================ */

(function () {
    'use strict';

    /* ========================================================================
       STATE
       ======================================================================== */

    var State = {
        visible: false,
        translations: {},
        categories: [],
        config: {},
        durations: [],
        charId: '',

        // Browse
        listings: [],
        currentCategory: 'all',
        currentSearch: '',
        currentSort: 'time_asc',

        // Sell
        inventory: [],
        inventorySearch: '',
        selectedItem: null,

        // My Auctions
        myAuctions: [],

        // My Bids
        myBids: [],

        // Mailbox
        mailbox: [],

        // Catalogue
        catalogItems: [],
        catalogCompanies: {},
        catalogConfig: {},
        catalogFilter: { company: 'all', category: 'all', search: '' },
        myOrders: [],
        orderModalItem: null,

        // Requests
        requestConfig: {},
        openRequests: [],
        myRequests: [],
        requestsSearch: '',
        requestsSort: 'newest',
        fulfillModalRequest: null
    };

    /* ========================================================================
       HELPERS
       ======================================================================== */

    function T(key) {
        return State.translations[key] || key;
    }

    function sendNUI(event, data) {
        return fetch(`https://${GetParentResourceName()}/` + event, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data || {})
        })
        .then(function (resp) { return resp.json(); })
        .catch(function () { return {}; });
    }

    function formatCurrency(amount) {
        if (amount === null || amount === undefined) return '\u2014';
        return '$' + parseFloat(amount).toFixed(2);
    }

    /**
     * Item icon URL prefix. The open message replaces it with poggy_core's
     * inv.imageBase; this default is only used until then.
     */
    var IMG_BASE = 'nui://vorp_inventory/html/img/items/';

    /**
     * Icons whose file is not <IMG_BASE><name>.png (RSG's `image` field),
     * by item name. Sent with the open message from poggy_core's registry.
     */
    var ITEM_IMAGES = {};

    /**
     * Item image URL. Every icon in the UI goes through here, so the base
     * and the odd file names come from poggy_core, never from a guess.
     */
    function itemImageUrl(itemName) {
        if (!itemName) return '';
        if (ITEM_IMAGES[itemName]) return ITEM_IMAGES[itemName];
        return IMG_BASE +encodeURIComponent(itemName) + '.png';
    }

    function itemImgTag(itemName, cssClass) {
        cssClass = cssClass || 'item-img-sm';
        var src = itemImageUrl(itemName);
        return '<img class="' + cssClass + '" src="' + src + '" onerror="this.style.display=\'none\'" alt="">';
    }

    /**
     * Parse expires_at — handles MySQL DATETIME strings and unix timestamps.
     * MySQL sends "2026-02-18 12:00:00" or "2026-02-18T12:00:00" format.
     * Returns unix seconds.
     */
    function parseExpiresAt(expiresAt) {
        if (!expiresAt) return 0;

        // Already a number (unix timestamp)
        if (typeof expiresAt === 'number') {
            // If it's way too large, it's likely milliseconds
            if (expiresAt > 9999999999) return Math.floor(expiresAt / 1000);
            return expiresAt;
        }

        // String — parse as datetime
        if (typeof expiresAt === 'string') {
            // MySQL DATETIME: "2026-02-18 12:00:00" — add UTC timezone hint
            // Replace space with T for standard ISO parsing, append Z for UTC
            var normalized = expiresAt.replace(' ', 'T');
            // If no timezone info, treat as UTC (server time)
            if (normalized.indexOf('Z') === -1 && normalized.indexOf('+') === -1 && normalized.indexOf('-', 11) === -1) {
                normalized += 'Z';
            }
            var parsed = new Date(normalized);
            if (!isNaN(parsed.getTime())) {
                return Math.floor(parsed.getTime() / 1000);
            }
            // Fallback: try direct parse
            var direct = new Date(expiresAt);
            if (!isNaN(direct.getTime())) {
                return Math.floor(direct.getTime() / 1000);
            }
        }

        return 0;
    }

    function formatTimeLeft(expiresAt) {
        var expiresSec = parseExpiresAt(expiresAt);
        if (!expiresSec) return '\u2014';

        var now = Math.floor(Date.now() / 1000);
        var diff = expiresSec - now;

        if (diff <= 0) return T('ui_expired') || 'Expired';

        var days = Math.floor(diff / 86400);
        var hours = Math.floor((diff % 86400) / 3600);
        var minutes = Math.floor((diff % 3600) / 60);

        if (days > 0) return days + 'd ' + hours + 'h';
        if (hours > 0) return hours + 'h ' + minutes + 'm';
        return minutes + 'm';
    }

    function isTimeUrgent(expiresAt) {
        var expiresSec = parseExpiresAt(expiresAt);
        if (!expiresSec) return false;
        var now = Math.floor(Date.now() / 1000);
        return (expiresSec - now) < 3600;
    }

    function escapeHtml(str) {
        if (!str) return '';
        var div = document.createElement('div');
        div.textContent = str;
        return div.innerHTML;
    }

    function $(selector) {
        return document.querySelector(selector);
    }

    function $$(selector) {
        return document.querySelectorAll(selector);
    }

    /* ========================================================================
       NUI MESSAGE HANDLER
       ======================================================================== */

    window.addEventListener('message', function (event) {
        var data = event.data;

        switch (data.type) {
            case 'open':
                openUI(data);
                break;
            case 'close':
                closeUI();
                break;
            case 'refreshListings':
                fetchListings();
                break;
            case 'refreshMailbox':
                fetchMailbox();
                break;
            case 'refreshMyAuctions':
                fetchMyAuctions();
                break;
            case 'refreshShipments':
                // Only re-render if the orders view is currently visible
                if ($('#orders-view') && !$('#orders-view').classList.contains('hidden')) {
                    fetchMyOrders();
                }
                break;
            case 'refreshRequests':
                fetchOpenRequests();
                fetchMyRequests();
                break;
        }
    });

    /* ========================================================================
       ESC KEY
       ======================================================================== */

    document.addEventListener('keydown', function (e) {
        if (e.key === 'Escape' && State.visible) {
            var bidModal = $('#bid-modal');
            var buyoutModal = $('#buyout-modal');
            var cancelModal = $('#cancel-modal');

            if (bidModal && !bidModal.classList.contains('hidden')) {
                bidModal.classList.add('hidden');
                return;
            }
            if (buyoutModal && !buyoutModal.classList.contains('hidden')) {
                buyoutModal.classList.add('hidden');
                return;
            }
            if (cancelModal && !cancelModal.classList.contains('hidden')) {
                cancelModal.classList.add('hidden');
                return;
            }
            var orderModal = $('#order-modal');
            if (orderModal && !orderModal.classList.contains('hidden')) {
                orderModal.classList.add('hidden');
                return;
            }
            var fulfillModal = $('#fulfill-modal');
            if (fulfillModal && !fulfillModal.classList.contains('hidden')) {
                fulfillModal.classList.add('hidden');
                return;
            }

            sendNUI('close');
            closeUI();
        }
    });

    /* ========================================================================
       OPEN / CLOSE
       ======================================================================== */

    function openUI(data) {
        State.visible = true;
        State.translations = data.translations || {};
        State.categories = data.categories || [];
        if (data.imageBase) IMG_BASE = data.imageBase;
        ITEM_IMAGES = data.itemImages || {};
        State.config = data.auctionConfig || data.config || {};
        State.durations = data.durations || (State.config.durations) || [];
        State.charId = data.charId || '';
        State.listings = data.listings || [];
        State.inventory = [];
        State.inventorySearch = '';
        State.selectedItem = null;
        State.myAuctions = [];
        State.myBids = [];
        State.mailbox = [];
        State.currentCategory = 'all';
        State.currentSearch   = '';
        State.currentSort     = 'time_asc';

        // New features
        State.catalogItems     = data.shipmentCatalog   || [];
        State.catalogCompanies = data.shipmentCompanies || {};
        State.catalogConfig    = data.shipmentConfig    || {};
        State.requestConfig    = data.requestConfig     || {};
        State.shopDelivery     = data.shopDelivery === true;
        State.catalogFilter    = { category: 'all', search: '' };
        State.catalogPrices    = {};
        State.myOrders         = [];
        State.openRequests     = [];
        State.myRequests       = [];
        State.requestsSearch   = '';
        State.requestsSort     = 'newest';
        State.orderModalItem   = null;
        State.selectedRequestItem = null;
        State.fulfillModalRequest = null;

        var container = $('#auction-container');
        if (container) container.classList.remove('hidden');

        buildCategories();
        buildDurations();
        renderListings();
        switchTab('browse');

        var searchInput = $('#search-input');
        if (searchInput) searchInput.value = '';
        var sortSelect = $('#sort-select');
        if (sortSelect) sortSelect.value = 'time_asc';
        var invSearch = $('#inventory-search');
        if (invSearch) invSearch.value = '';
    }

    function closeUI() {
        State.visible = false;
        var container = $('#auction-container');
        if (container) container.classList.add('hidden');

        $$('.modal-overlay').forEach(function (m) { m.classList.add('hidden'); });
    }

    /* ========================================================================
       TAB SWITCHING
       ======================================================================== */

    function switchTab(tabName) {
        $$('.tab-btn').forEach(function (btn) {
            btn.classList.toggle('active', btn.dataset.tab === tabName);
        });

        $$('.tab-content').forEach(function (content) {
            content.classList.toggle('active', content.id === 'tab-' + tabName);
        });

        if (tabName === 'sell') {
            fetchInventory();
        } else if (tabName === 'myauctions') {
            fetchMyAuctions();
        } else if (tabName === 'mybids') {
            fetchMyBids();
        } else if (tabName === 'mailbox') {
            fetchMailbox();
        } else if (tabName === 'catalogue') {
            initCatalogueTab();
        } else if (tabName === 'requests') {
            buildRequestDurations();
            fetchOpenRequests();
        }
    }

    /* ========================================================================
       DURATION SELECT BUILDER
       ======================================================================== */

    function buildDurations() {
        var sel = $('#sell-duration');
        if (!sel) return;

        var durations = State.durations;
        if (!durations || durations.length === 0) {
            sel.innerHTML = '<option value="1">Short (2h)</option>'
                + '<option value="2">Medium (8h)</option>'
                + '<option value="3" selected>Long (24h)</option>'
                + '<option value="4">Very Long (48h)</option>';
            return;
        }

        var html = '';
        for (var i = 0; i < durations.length; i++) {
            var d = durations[i];
            var label = d.label || (d.hours + ' Hours');
            html += '<option value="' + (i + 1) + '">' + escapeHtml(label) + '</option>';
        }
        sel.innerHTML = html;
    }

    /* ========================================================================
       DOM LISTENERS
       ======================================================================== */

    document.addEventListener('DOMContentLoaded', function () {
        // Tab buttons
        $$('.tab-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                switchTab(this.dataset.tab);
            });
        });

        // Close button
        var closeBtn = $('#btn-close');
        if (closeBtn) {
            closeBtn.addEventListener('click', function () {
                sendNUI('close');
                closeUI();
            });
        }

        // Search input (debounced)
        var searchInput = $('#search-input');
        if (searchInput) {
            var searchTimeout = null;
            searchInput.addEventListener('input', function () {
                clearTimeout(searchTimeout);
                searchTimeout = setTimeout(function () {
                    State.currentSearch = searchInput.value;
                    fetchListings();
                }, 350);
            });
        }

        // Sort select
        var sortSelect = $('#sort-select');
        if (sortSelect) {
            sortSelect.addEventListener('change', function () {
                State.currentSort = sortSelect.value;
                fetchListings();
            });
        }

        // Search button
        var searchBtn = $('#btn-search');
        if (searchBtn) {
            searchBtn.addEventListener('click', function () {
                State.currentSearch = ($('#search-input') || {}).value || '';
                fetchListings();
            });
        }

        // Inventory search (debounced)
        var invSearchInput = $('#inventory-search');
        if (invSearchInput) {
            var invSearchTimeout = null;
            invSearchInput.addEventListener('input', function () {
                clearTimeout(invSearchTimeout);
                invSearchTimeout = setTimeout(function () {
                    State.inventorySearch = invSearchInput.value.toLowerCase();
                    renderInventory();
                }, 200);
            });
        }

        // Sell form submit
        var submitBtn = $('#btn-create-listing');
        if (submitBtn) {
            submitBtn.addEventListener('click', createListing);
        }

        // Sell form cancel
        var cancelFormBtn = $('#btn-cancel-form');
        if (cancelFormBtn) {
            cancelFormBtn.addEventListener('click', function () {
                State.selectedItem = null;
                renderSellForm();
            });
        }

        // Modal close buttons
        $$('.modal-close').forEach(function (btn) {
            btn.addEventListener('click', function () {
                this.closest('.modal-overlay').classList.add('hidden');
            });
        });

        // Bid modal confirm
        var confirmBidBtn = $('#btn-confirm-bid');
        if (confirmBidBtn) {
            confirmBidBtn.addEventListener('click', confirmBid);
        }

        // Buyout modal confirm
        var confirmBuyoutBtn = $('#btn-confirm-buyout');
        if (confirmBuyoutBtn) {
            confirmBuyoutBtn.addEventListener('click', confirmBuyout);
        }

        // Cancel modal confirm
        var confirmCancelBtn = $('#btn-confirm-cancel');
        if (confirmCancelBtn) {
            confirmCancelBtn.addEventListener('click', confirmCancel);
        }

        // Collect All button
        var collectAllBtn = $('#btn-collect-all');
        if (collectAllBtn) {
            collectAllBtn.addEventListener('click', collectAll);
        }

        // ---- CATALOGUE TAB ----
        var myOrdersBtn = $('#btn-my-orders');
        if (myOrdersBtn) { myOrdersBtn.addEventListener('click', toggleOrdersView); }

        var backCatalogueBtn = $('#btn-back-catalogue');
        if (backCatalogueBtn) { backCatalogueBtn.addEventListener('click', backToCatalogue); }

        var catalogSearch = $('#catalogue-search');
        if (catalogSearch) {
            var catSearchTimeout = null;
            catalogSearch.addEventListener('input', function () {
                clearTimeout(catSearchTimeout);
                catSearchTimeout = setTimeout(function () {
                    State.catalogFilter.search = catalogSearch.value.toLowerCase();
                    renderCatalogueGrid();
                }, 250);
            });
        }

        // Delegated filter button clicks (source category) + catalogue card click
        document.addEventListener('click', function (e) {
            if (e.target.classList.contains('filter-btn') && e.target.dataset.category !== undefined) {
                var parent2 = e.target.closest('#catalogue-category-list');
                if (parent2) {
                    parent2.querySelectorAll('.filter-btn').forEach(function (b) { b.classList.remove('active'); });
                    e.target.classList.add('active');
                    State.catalogFilter.category = e.target.dataset.category;
                    renderCatalogueGrid();
                }
            }
            var card = e.target.closest('.catalogue-card');
            if (card && card.dataset.catalogIdx !== undefined) {
                var idx = parseInt(card.dataset.catalogIdx);
                if (!isNaN(idx) && State.catalogItems[idx]) {
                    openOrderModal(State.catalogItems[idx]);
                }
            }
        });

        var confirmOrderBtn = $('#btn-confirm-order');
        if (confirmOrderBtn) { confirmOrderBtn.addEventListener('click', confirmOrder); }

        var orderQtyInput = $('#order-qty');
        if (orderQtyInput) {
            orderQtyInput.addEventListener('input', updateOrderBreakdown);
            orderQtyInput.addEventListener('change', updateOrderBreakdown);
        }

        // ---- REQUESTS TAB ----
        var reqSearchBtn = $('#btn-search-requests');
        if (reqSearchBtn) {
            reqSearchBtn.addEventListener('click', function () {
                State.requestsSearch = ($('#requests-search') || {}).value || '';
                State.requestsSort   = ($('#requests-sort')   || {}).value || 'newest';
                fetchOpenRequests();
            });
        }

        var reqSortSelect = $('#requests-sort');
        if (reqSortSelect) {
            reqSortSelect.addEventListener('change', function () {
                State.requestsSort = reqSortSelect.value;
                fetchOpenRequests();
            });
        }

        $$('.subnav-btn').forEach(function (btn) {
            btn.addEventListener('click', function () {
                var panelName = this.dataset.panel;
                $$('.subnav-btn').forEach(function (b) { b.classList.remove('active'); });
                $$('.requests-panel').forEach(function (p) { p.classList.remove('active'); });
                this.classList.add('active');
                var panel = $('#panel-' + panelName);
                if (panel) panel.classList.add('active');
                if (panelName === 'browse-requests')  { fetchOpenRequests(); }
                else if (panelName === 'my-requests') { fetchMyRequests(); }
                else if (panelName === 'post-request') { clearRequestItem(); }
            });
        });

        var submitReqBtn = $('#btn-submit-request');
        if (submitReqBtn) { submitReqBtn.addEventListener('click', submitRequest); }

        var reqSearchBtn2 = $('#btn-req-search');
        if (reqSearchBtn2) { reqSearchBtn2.addEventListener('click', searchItemsForRequest); }

        var reqSearchInput = $('#req-item-search');
        if (reqSearchInput) {
            reqSearchInput.addEventListener('keydown', function (e) {
                if (e.key === 'Enter') searchItemsForRequest();
            });
        }

        var reqChangeItem = $('#btn-req-change-item');
        if (reqChangeItem) { reqChangeItem.addEventListener('click', clearRequestItem); }

        ['req-quantity', 'req-price'].forEach(function (id) {
            var el = $('#' + id);
            if (el) {
                el.addEventListener('input', updateRequestEscrow);
                el.addEventListener('change', updateRequestEscrow);
            }
        });

        var fulfillQtyInput = $('#fulfill-qty');
        if (fulfillQtyInput) {
            fulfillQtyInput.addEventListener('input', updateFulfillBreakdown);
            fulfillQtyInput.addEventListener('change', updateFulfillBreakdown);
        }

        var confirmFulfillBtn = $('#btn-confirm-fulfill');
        if (confirmFulfillBtn) { confirmFulfillBtn.addEventListener('click', confirmFulfill); }

        // Sell form inputs — live fee calculation
        ['sell-quantity', 'sell-starting-price', 'sell-buyout-price', 'sell-duration'].forEach(function (id) {
            var el = $('#' + id);
            if (el) {
                el.addEventListener('input', updateFeeBreakdown);
                el.addEventListener('change', updateFeeBreakdown);
            }
        });
    });

    /* ========================================================================
       CATEGORIES
       ======================================================================== */

    function buildCategories() {
        var list = $('#category-list');
        if (!list) return;

        var html = '<div class="category-item active" data-category="all">';
        html += '<span class="category-icon">\uD83C\uDFF7\uFE0F</span>';
        html += '<span>All Items</span>';
        html += '</div>';

        (State.categories || []).forEach(function (cat) {
            var catKey = cat.key || cat.name;
            html += '<div class="category-item" data-category="' + escapeHtml(catKey) + '">';
            html += '<span class="category-icon">' + escapeHtml(cat.icon || '') + '</span>';
            html += '<span>' + escapeHtml(cat.label || catKey) + '</span>';
            html += '</div>';
        });

        list.innerHTML = html;

        list.querySelectorAll('.category-item').forEach(function (item) {
            item.addEventListener('click', function () {
                list.querySelectorAll('.category-item').forEach(function (c) { c.classList.remove('active'); });
                this.classList.add('active');
                State.currentCategory = this.dataset.category;
                fetchListings();
            });
        });
    }

    /* ========================================================================
       FETCH DATA
       ======================================================================== */

    function fetchListings() {
        sendNUI('getListings', {
            category: State.currentCategory === 'all' ? null : State.currentCategory,
            search: State.currentSearch || null,
            sort: State.currentSort
        }).then(function (resp) {
            if (resp && resp.listings) {
                State.listings = resp.listings;
                renderListings();
            }
        });
    }

    function fetchInventory() {
        sendNUI('getInventory').then(function (resp) {
            if (resp && resp.items) {
                State.inventory = resp.items;
                renderInventory();
            }
        });
    }

    function fetchMyAuctions() {
        sendNUI('getMyAuctions').then(function (resp) {
            if (resp && resp.listings) {
                State.myAuctions = resp.listings;
                renderMyAuctions();
            }
        });
    }

    function fetchMailbox() {
        sendNUI('getMailbox').then(function (resp) {
            if (resp && resp.entries) {
                State.mailbox = resp.entries;
                renderMailbox();
            }
        });
    }

    function fetchMyBids() {
        sendNUI('getMyBids').then(function (resp) {
            if (resp && resp.bids) {
                State.myBids = resp.bids;
                renderMyBids();
            }
        });
    }

    /* ========================================================================
       RENDER — BROWSE LISTINGS
       ======================================================================== */

    function renderListings() {
        var tbody = $('#listings-tbody');
        if (!tbody) return;

        var listings = State.listings || [];

        if (listings.length === 0) {
            tbody.innerHTML = '<tr><td colspan="8" class="empty-cell">No listings found</td></tr>';
            updateListingCount(0);
            return;
        }

        var html = '';
        listings.forEach(function (listing) {
            var timeClass = isTimeUrgent(listing.expires_at) ? 'time-urgent' : 'time-normal';

            html += '<tr data-id="' + listing.id + '">';
            html += '<td class="col-item"><div class="item-cell">';
            html += itemImgTag(listing.item_name, 'item-img-sm');
            html += '<span>' + escapeHtml(listing.item_label || listing.item_name) + '</span>';
            html += '</div></td>';
            html += '<td class="col-qty" style="text-align:center">' + (listing.quantity || 1) + '</td>';
            html += '<td class="col-bid"><span class="price-bid">' + formatCurrency(listing.current_bid || listing.start_price) + '</span></td>';
            html += '<td class="col-buyout">';
            if (listing.buyout_price && listing.buyout_price > 0) {
                html += '<span class="price-buyout">' + formatCurrency(listing.buyout_price) + '</span>';
            } else {
                html += '<span class="price-none">No Buyout</span>';
            }
            html += '</td>';
            html += '<td class="col-time"><span class="' + timeClass + '">' + formatTimeLeft(listing.expires_at) + '</span></td>';
            html += '<td class="col-seller">' + escapeHtml(listing.seller_name || 'Unknown') + '</td>';
            html += '<td class="col-bids" style="text-align:center">' + (listing.bid_count || 0) + '</td>';
            html += '<td class="col-actions" style="text-align:center">';
            if (State.charId && String(listing.seller_id) === String(State.charId)) {
                html += '<button class="btn-cancel-listing" onclick="AuctionUI.openCancelModal(' + listing.id + ')">Cancel</button>';
            } else {
                html += '<button class="btn-bid" onclick="AuctionUI.openBidModal(' + listing.id + ')">Bid</button>';
                if (listing.buyout_price && listing.buyout_price > 0) {
                    html += '<button class="btn-buyout" onclick="AuctionUI.openBuyoutModal(' + listing.id + ')">Buyout</button>';
                }
            }
            html += '</td>';
            html += '</tr>';
        });

        tbody.innerHTML = html;
        updateListingCount(listings.length);
    }

    function updateListingCount(count) {
        var el = $('#listings-count');
        if (el) {
            el.textContent = count + ' listing' + (count !== 1 ? 's' : '');
        }
    }

    /* ========================================================================
       RENDER — SELL INVENTORY (with search filter + item images)
       ======================================================================== */

    function renderInventory() {
        var list = $('#inventory-list');
        if (!list) return;

        var items = State.inventory || [];
        var searchTerm = State.inventorySearch || '';

        // Filter by search
        var filtered = items;
        if (searchTerm) {
            filtered = items.filter(function (item) {
                var label = (item.label || item.name || '').toLowerCase();
                var name = (item.name || '').toLowerCase();
                return label.indexOf(searchTerm) !== -1 || name.indexOf(searchTerm) !== -1;
            });
        }

        if (items.length === 0) {
            list.innerHTML = '<div class="empty-state"><p>Your inventory is empty</p></div>';
            return;
        }

        if (filtered.length === 0) {
            list.innerHTML = '<div class="empty-state"><p>No items match your search</p></div>';
            return;
        }

        var html = '';
        filtered.forEach(function (item) {
            // Find original index for selection
            var origIndex = items.indexOf(item);
            var selectedClass = (State.selectedItem && State.selectedItem.name === item.name) ? ' selected' : '';
            html += '<div class="inventory-item' + selectedClass + '" data-index="' + origIndex + '">';
            html += itemImgTag(item.name, 'item-img');
            html += '<div class="inv-item-name">' + escapeHtml(item.label || item.name) + '</div>';
            html += '<div class="inv-item-count">x' + (item.count || 1) + '</div>';
            html += '</div>';
        });

        list.innerHTML = html;

        list.querySelectorAll('.inventory-item').forEach(function (el) {
            el.addEventListener('click', function () {
                var idx = parseInt(this.dataset.index);
                var item = State.inventory[idx];
                if (item) {
                    State.selectedItem = item;
                    renderInventory();
                    renderSellForm();
                }
            });
        });
    }

    function renderSellForm() {
        var form = $('.listing-form');
        var placeholder = $('.form-placeholder');

        if (!State.selectedItem) {
            if (form) form.classList.add('hidden');
            if (placeholder) placeholder.classList.remove('hidden');
            return;
        }

        if (form) form.classList.remove('hidden');
        if (placeholder) placeholder.classList.add('hidden');

        var previewName = $('.item-preview-name');
        var previewCount = $('.item-preview-count');
        if (previewName) previewName.textContent = State.selectedItem.label || State.selectedItem.name;
        if (previewCount) previewCount.textContent = 'Available: ' + (State.selectedItem.count || 1);

        var qtyInput = $('#sell-quantity');
        if (qtyInput) {
            qtyInput.value = 1;
            qtyInput.max = State.selectedItem.count || 1;
        }
        var priceInput = $('#sell-starting-price');
        if (priceInput) priceInput.value = '';
        var buyoutInput = $('#sell-buyout-price');
        if (buyoutInput) buyoutInput.value = '';

        updateFeeBreakdown();
    }

    function updateFeeBreakdown() {
        var quantity = parseInt(($('#sell-quantity') || {}).value) || 1;
        var pricePerPiece = parseFloat(($('#sell-starting-price') || {}).value) || 0;
        var buyoutPerPiece = parseFloat(($('#sell-buyout-price') || {}).value) || 0;
        var durationIdx = parseInt(($('#sell-duration') || {}).value) || 1;

        // Calculate totals from per-piece inputs
        var totalStartPrice = pricePerPiece * quantity;
        var totalBuyoutPrice = buyoutPerPiece * quantity;

        // Get deposit percent from durations config
        var depositPercent = 5; // default
        if (State.durations && State.durations[durationIdx - 1]) {
            depositPercent = State.durations[durationIdx - 1].depositPercent || 5;
        }

        var baseTotal = totalBuyoutPrice > 0 ? totalBuyoutPrice : totalStartPrice;

        // depositPercent is already a percentage (e.g., 15 means 15%)
        var deposit = Math.max(baseTotal * (depositPercent / 100), 0);

        // salesTaxPercent from config is a percentage (e.g., 5 means 5%)
        var salesTaxPercent = 5; // default
        if (State.config && State.config.salesTaxPercent !== undefined) {
            salesTaxPercent = State.config.salesTaxPercent;
        }
        var estimatedTax = baseTotal * (salesTaxPercent / 100);

        // Total listing price row
        var totalEl = $('#fee-total');
        if (totalEl) {
            var totalLabel = formatCurrency(totalStartPrice);
            if (quantity > 1) totalLabel += '  (' + quantity + ' x ' + formatCurrency(pricePerPiece) + '/ea)';
            totalEl.textContent = totalLabel;
        }

        // Total buyout price row — show only when buyout is set
        var buyoutTotalRow = $('#fee-buyout-total-row');
        var buyoutTotalEl = $('#fee-buyout-total');
        if (buyoutTotalRow && buyoutTotalEl) {
            if (totalBuyoutPrice > 0) {
                var buyLabel = formatCurrency(totalBuyoutPrice);
                if (quantity > 1) buyLabel += '  (' + quantity + ' x ' + formatCurrency(buyoutPerPiece) + '/ea)';
                buyoutTotalEl.textContent = buyLabel;
                buyoutTotalRow.style.display = '';
            } else {
                buyoutTotalRow.style.display = 'none';
            }
        }

        var depositEl = $('#fee-deposit');
        var taxEl = $('#fee-tax');

        if (depositEl) depositEl.textContent = formatCurrency(deposit);
        if (taxEl) taxEl.textContent = '~' + formatCurrency(estimatedTax);
    }

    /* ========================================================================
       RENDER — MY AUCTIONS (with item images)
       ======================================================================== */

    function renderMyAuctions() {
        var tbody = $('#myauctions-tbody');
        if (!tbody) return;

        var auctions = State.myAuctions || [];

        if (auctions.length === 0) {
            tbody.innerHTML = '<tr><td colspan="7" class="empty-cell">You have no auctions</td></tr>';
            return;
        }

        var html = '';
        auctions.forEach(function (listing) {
            var statusClass = 'status-' + (listing.status || 'active');
            var statusLabel = (listing.status || 'active').charAt(0).toUpperCase() + (listing.status || 'active').slice(1);

            html += '<tr data-id="' + listing.id + '">';
            html += '<td><div class="item-cell">';
            html += itemImgTag(listing.item_name, 'item-img-sm');
            html += '<span>' + escapeHtml(listing.item_label || listing.item_name) + '</span>';
            html += '</div></td>';
            html += '<td style="text-align:center">' + (listing.quantity || 1) + '</td>';
            html += '<td><span class="price-bid">' + formatCurrency(listing.current_bid || listing.start_price) + '</span></td>';
            html += '<td>';
            if (listing.buyout_price && listing.buyout_price > 0) {
                html += '<span class="price-buyout">' + formatCurrency(listing.buyout_price) + '</span>';
            } else {
                html += '<span class="price-none">\u2014</span>';
            }
            html += '</td>';
            html += '<td>' + formatTimeLeft(listing.expires_at) + '</td>';
            html += '<td><span class="status-badge ' + statusClass + '">' + statusLabel + '</span></td>';
            html += '<td style="text-align:center">';
            if (listing.status === 'active') {
                html += '<button class="btn-cancel-listing" onclick="AuctionUI.openCancelModal(' + listing.id + ')">Cancel</button>';
            }
            html += '</td>';
            html += '</tr>';
        });

        tbody.innerHTML = html;
    }

    /* ========================================================================
       RENDER — MAILBOX (with item images)
       ======================================================================== */

    function renderMailbox() {
        var list = $('#mailbox-list');
        if (!list) return;

        var entries = State.mailbox || [];
        var collectAllBtn = $('#btn-collect-all');

        if (entries.length === 0) {
            list.innerHTML = '<div class="empty-state"><p>Your mailbox is empty</p></div>';
            if (collectAllBtn) collectAllBtn.classList.add('hidden');
            return;
        }

        if (collectAllBtn) collectAllBtn.classList.remove('hidden');

        var html = '';
        entries.forEach(function (entry) {
            // DB column is "type" not "entry_type", and "amount" not "money_amount"
            var entryType = entry.type || entry.entry_type;
            var isItem = entryType === 'item';
            var iconClass = isItem ? 'type-item' : 'type-money';
            var moneyAmount = entry.amount || entry.money_amount || 0;

            var label = isItem
                ? (entry.item_label || entry.item_name) + ' x' + (entry.quantity || 1)
                : formatCurrency(moneyAmount);

            html += '<div class="mailbox-entry" data-id="' + entry.id + '">';
            if (isItem && entry.item_name) {
                html += '<img class="item-img-lg" src="' + itemImageUrl(entry.item_name) + '" onerror="this.style.display=\'none\'" alt="">';
            } else {
                html += '<div class="mailbox-icon ' + iconClass + '">💰</div>';
            }
            html += '<div class="mailbox-details">';
            html += '<div class="mailbox-item-name">' + escapeHtml(label) + '</div>';
            html += '<div class="mailbox-reason">' + escapeHtml(entry.reason || '') + '</div>';
            html += '</div>';
            html += '<button class="btn-collect" onclick="AuctionUI.collectItem(' + entry.id + ')">Collect</button>';
            html += '</div>';
        });

        list.innerHTML = html;
    }

    /* ========================================================================
       RENDER — MY BIDS (listings the player has bid on)
       ======================================================================== */

    function renderMyBids() {
        var tbody = $('#mybids-tbody');
        if (!tbody) return;

        var bids = State.myBids || [];

        if (bids.length === 0) {
            tbody.innerHTML = '<tr><td colspan="8" class="empty-cell">You have no active bids</td></tr>';
            return;
        }

        var html = '';
        bids.forEach(function (listing) {
            var myBid = parseFloat(listing.my_bid_amount) || 0;
            var currentBid = parseFloat(listing.current_bid) || parseFloat(listing.start_price) || 0;
            var isWinning = myBid >= currentBid;
            var bidStatusClass = isWinning ? 'status-active' : 'status-outbid';
            var bidStatusLabel = isWinning ? 'Winning' : 'Outbid';
            var timeClass = isTimeUrgent(listing.expires_at) ? 'time-urgent' : 'time-normal';

            html += '<tr data-id="' + listing.id + '">';
            html += '<td><div class="item-cell">';
            html += itemImgTag(listing.item_name, 'item-img-sm');
            html += '<span>' + escapeHtml(listing.item_label || listing.item_name) + '</span>';
            html += '</div></td>';
            html += '<td style="text-align:center">' + (listing.quantity || 1) + '</td>';
            html += '<td><span class="price-bid">' + formatCurrency(myBid) + '</span></td>';
            html += '<td><span class="price-bid">' + formatCurrency(currentBid) + '</span></td>';
            html += '<td>';
            if (listing.buyout_price && listing.buyout_price > 0) {
                html += '<span class="price-buyout">' + formatCurrency(listing.buyout_price) + '</span>';
            } else {
                html += '<span class="price-none">&mdash;</span>';
            }
            html += '</td>';
            html += '<td><span class="' + timeClass + '">' + formatTimeLeft(listing.expires_at) + '</span></td>';
            html += '<td>' + escapeHtml(listing.seller_name || 'Unknown') + '</td>';
            html += '<td><span class="status-badge ' + bidStatusClass + '">' + bidStatusLabel + '</span></td>';
            html += '</tr>';
        });

        tbody.innerHTML = html;
    }

    /* ========================================================================
       ACTIONS — CREATE LISTING
       ======================================================================== */

    function createListing() {
        if (!State.selectedItem) return;

        var quantity = parseInt(($('#sell-quantity') || {}).value) || 1;
        var pricePerPiece = parseFloat(($('#sell-starting-price') || {}).value) || 0;
        var buyoutPerPiece = parseFloat(($('#sell-buyout-price') || {}).value) || 0;
        var durationIdx = parseInt(($('#sell-duration') || {}).value) || 1;

        if (pricePerPiece <= 0) return;

        // Convert per-piece prices to totals for the server
        var totalStartPrice = pricePerPiece * quantity;
        var totalBuyoutPrice = buyoutPerPiece * quantity;

        sendNUI('createListing', {
            itemName: State.selectedItem.name,
            quantity: quantity,
            startPrice: totalStartPrice,
            buyoutPrice: totalBuyoutPrice > 0 ? totalBuyoutPrice : null,
            durationIdx: durationIdx
        }).then(function () {
            State.selectedItem = null;
            renderSellForm();
            setTimeout(fetchInventory, 500);
        });
    }

    /* ========================================================================
       ACTIONS — BID MODAL
       ======================================================================== */

    function openBidModal(listingId) {
        var listing = findListing(listingId);
        if (!listing) return;

        var modal = $('#bid-modal');
        if (!modal) return;

        modal.dataset.listingId = listingId;

        var nameEl = modal.querySelector('.modal-item-name');
        if (nameEl) nameEl.textContent = (listing.item_label || listing.item_name) + ' x' + (listing.quantity || 1);

        var currentBid = listing.current_bid || listing.start_price || 0;
        var bidIncrement = (State.config && State.config.minBidIncrement) || 0.50;
        var minBid = parseFloat(currentBid) + parseFloat(bidIncrement);

        var currentBidEl = $('#bid-current');
        if (currentBidEl) currentBidEl.textContent = formatCurrency(currentBid);

        var minBidEl = $('#bid-minimum');
        if (minBidEl) minBidEl.textContent = formatCurrency(minBid);

        var bidInput = $('#bid-amount');
        if (bidInput) {
            bidInput.value = minBid.toFixed(2);
            bidInput.min = minBid.toFixed(2);
        }

        modal.classList.remove('hidden');
    }

    function confirmBid() {
        var modal = $('#bid-modal');
        if (!modal) return;

        var listingId = parseInt(modal.dataset.listingId);
        var bidAmount = parseFloat(($('#bid-amount') || {}).value) || 0;

        if (bidAmount <= 0 || !listingId) return;

        sendNUI('placeBid', {
            listingId: listingId,
            bidAmount: bidAmount
        }).then(function () {
            modal.classList.add('hidden');
            setTimeout(fetchListings, 300);
        });
    }

    /* ========================================================================
       ACTIONS — BUYOUT MODAL
       ======================================================================== */

    function openBuyoutModal(listingId) {
        var listing = findListing(listingId);
        if (!listing) return;

        var modal = $('#buyout-modal');
        if (!modal) return;

        modal.dataset.listingId = listingId;

        var nameEl = modal.querySelector('.modal-item-name');
        if (nameEl) nameEl.textContent = (listing.item_label || listing.item_name) + ' x' + (listing.quantity || 1);

        var priceEl = $('#buyout-price');
        if (priceEl) priceEl.textContent = 'Buyout price: ' + formatCurrency(listing.buyout_price);

        modal.classList.remove('hidden');
    }

    function confirmBuyout() {
        var modal = $('#buyout-modal');
        if (!modal) return;

        var listingId = parseInt(modal.dataset.listingId);
        if (!listingId) return;

        sendNUI('buyout', {
            listingId: listingId
        }).then(function () {
            modal.classList.add('hidden');
            setTimeout(fetchListings, 300);
        });
    }

    /* ========================================================================
       ACTIONS — CANCEL MODAL
       ======================================================================== */

    function openCancelModal(listingId) {
        var listing = findMyAuction(listingId) || findListing(listingId);
        if (!listing) return;

        var modal = $('#cancel-modal');
        if (!modal) return;

        modal.dataset.listingId = listingId;

        var nameEl = modal.querySelector('.modal-item-name');
        if (nameEl) nameEl.textContent = (listing.item_label || listing.item_name) + ' x' + (listing.quantity || 1);

        modal.classList.remove('hidden');
    }

    function confirmCancel() {
        var modal = $('#cancel-modal');
        if (!modal) return;

        var listingId = parseInt(modal.dataset.listingId);
        if (!listingId) return;

        sendNUI('cancelListing', {
            listingId: listingId
        }).then(function () {
            modal.classList.add('hidden');
            setTimeout(function () {
                fetchListings();
                fetchMyAuctions();
            }, 300);
        });
    }

    /* ========================================================================
       ACTIONS — MAILBOX COLLECT
       ======================================================================== */

    function collectItem(entryId) {
        sendNUI('collectItem', {
            entryId: entryId
        }).then(function () {
            setTimeout(fetchMailbox, 300);
        });
    }

    function collectAll() {
        sendNUI('collectAll').then(function () {
            setTimeout(fetchMailbox, 500);
        });
    }

    /* ========================================================================
       UTILITIES
       ======================================================================== */

    function findListing(id) {
        for (var i = 0; i < State.listings.length; i++) {
            if (State.listings[i].id === id) return State.listings[i];
        }
        return null;
    }

    function findMyAuction(id) {
        for (var i = 0; i < State.myAuctions.length; i++) {
            if (State.myAuctions[i].id === id) return State.myAuctions[i];
        }
        return null;
    }

    /* ========================================================================
       TIME UPDATE LOOP — refresh time-left every 30s
       ======================================================================== */

    setInterval(function () {
        if (!State.visible) return;

        var rows = $$('#listings-tbody tr[data-id]');
        rows.forEach(function (row) {
            var id = parseInt(row.dataset.id);
            var listing = findListing(id);
            if (!listing) return;
            var timeCell = row.querySelector('.col-time span');
            if (timeCell) {
                timeCell.textContent = formatTimeLeft(listing.expires_at);
                timeCell.className = isTimeUrgent(listing.expires_at) ? 'time-urgent' : 'time-normal';
            }
        });
    }, 30000);

    /* ========================================================================
       PUBLIC API (for inline onclick handlers)
       ======================================================================== */

    window.AuctionUI = {
        openBidModal: openBidModal,
        openBuyoutModal: openBuyoutModal,
        openCancelModal: openCancelModal,
        collectItem: collectItem,
        openOrderModal: openOrderModal,
        cancelOrder: cancelOrder,
        archiveShipment: archiveShipment,
        reorderShipment: reorderShipment,
        openFulfillModal: openFulfillModal,
        cancelMyRequest: cancelMyRequest,
        backToCatalogue: backToCatalogue,
        selectRequestItem: selectRequestItem,
        clearRequestItem: clearRequestItem
    };

    /* ========================================================================
       CATALOGUE — Order Supply Catalog
       ======================================================================== */

    function initCatalogueTab() {
        // Ensure catalogue view is visible, orders view is hidden
        var catView = $('#catalogue-view');
        var ordView = $('#orders-view');
        if (catView) catView.classList.remove('hidden');
        if (ordView) ordView.classList.add('hidden');

        // Generate fresh price variances for every item when the tab is opened
        var variancePct = (State.catalogConfig && State.catalogConfig.PriceVariancePercent) || 5;
        State.catalogPrices = {};
        (State.catalogItems || []).forEach(function (item) {
            var delta = ((Math.random() * 2 - 1) * variancePct) / 100;
            State.catalogPrices[item.name] = { multiplier: 1 + delta };
        });
        renderCatalogueGrid();
    }

    function buildCompanyFilters() {
        // No-op: company filter removed; sidebar now uses source-category buttons only
    }

    function renderCatalogueGrid() {
        var grid = $('#catalogue-grid');
        if (!grid) return;

        var items  = State.catalogItems  || [];
        var filter = State.catalogFilter;

        var filtered = [];
        var indices  = [];
        items.forEach(function (item, idx) {
            if (filter.category !== 'all' && item.source !== filter.category) return;
            if (filter.search) {
                var s = filter.search;
                var nameMatch  = (item.name  || '').toLowerCase().indexOf(s) !== -1;
                var labelMatch = (item.label || '').toLowerCase().indexOf(s) !== -1;
                if (!nameMatch && !labelMatch) return;
            }
            filtered.push(item);
            indices.push(idx);
        });

        if (filtered.length === 0) {
            grid.innerHTML = '<div class="empty-state">No items found.</div>';
            return;
        }

        var sources = State.catalogCompanies || {};

        var html = filtered.map(function (item, i) {
            var origIdx    = indices[i];
            var priceData  = (State.catalogPrices && State.catalogPrices[item.name]) || { multiplier: 1 };
            var finalPrice = item.price * priceData.multiplier;
            var pctDiff    = (priceData.multiplier - 1) * 100;
            var isUp       = pctDiff > 0.05;
            var isDown     = pctDiff < -0.05;
            var sign       = isUp ? '+' : '';
            var priceColor = isUp ? '#e05252' : (isDown ? '#5cb85c' : 'var(--text-secondary)');
            var arrow      = isUp ? '▲' : (isDown ? '▼' : '');
            var sourceInfo  = sources[item.source] || {};
            var sourceName  = sourceInfo.name  || (item.source || '');
            var sourceColor = sourceInfo.color || '#8b7355';
            var imgSrc = itemImageUrl(item.name);

            return '<div class="catalogue-card" data-catalog-idx="' + origIdx + '">'
                + '<div class="card-img-wrapper">'
                + '<img class="card-img" src="' + imgSrc + '" alt="' + escapeHtml(item.label) + '" onerror="this.style.display=\'none\'">'
                + '</div>'
                + '<div class="card-body">'
                + '<div class="card-name">' + escapeHtml(item.label) + '</div>'
                + '<div class="card-desc">' + escapeHtml(item.desc || '') + '</div>'
                + '<div class="card-footer">'
                + '<span class="card-price">$' + finalPrice.toFixed(2) + '/unit'
                + (arrow ? ' <span class="price-variance" style="color:' + priceColor + ';font-size:11px">'
                    + arrow + sign + pctDiff.toFixed(1) + '%</span>' : '')
                + '</span>'
                + '<span class="company-badge" style="background:' + escapeHtml(sourceColor) + '20;color:' + escapeHtml(sourceColor) + '">'
                + escapeHtml(sourceName) + '</span>'
                + '</div>'
                + '</div>'
                + '</div>';
        }).join('');

        grid.innerHTML = html;
    }

    function openOrderModal(item) {
        State.orderModalItem = item;
        var modal = $('#order-modal');
        if (!modal) return;

        var imgEl = $('#order-modal-img');
        if (imgEl) {
            imgEl.src = itemImageUrl(item.name);
            imgEl.onerror = function () { this.style.display = 'none'; };
        }
        var nameEl = $('#order-modal-name');
        if (nameEl) nameEl.textContent = item.label;

        var companyEl = $('#order-modal-company');
        if (companyEl) {
            var src = State.catalogCompanies && State.catalogCompanies[item.source || item.company];
            companyEl.textContent = src ? src.name : (item.source || item.company || '');
        }
        var descEl = $('#order-modal-desc');
        if (descEl) descEl.textContent = item.desc || '';

        // Apply stored price variance for this item
        var priceData = (State.catalogPrices && State.catalogPrices[item.name]) || { multiplier: 1 };
        var multiplier = priceData.multiplier;
        var effectivePrice = item.price * multiplier;
        State.orderModalItem._effectivePrice = effectivePrice;

        var qtyInput = $('#order-qty');
        if (qtyInput) {
            qtyInput.value = '1';
            qtyInput.max   = (State.catalogConfig && State.catalogConfig.MaxQtyPerOrder) || 50;
        }
        var shopIdInput = $('#order-shop-id');
        if (shopIdInput) { shopIdInput.value = ''; }
        // Shipping to a shop needs Poggy Markets.  Without it every order goes
        // to the mailbox, so the field is hidden rather than left to fail.
        var shopGroup = $('#order-shop-group');
        if (shopGroup) shopGroup.classList.toggle('hidden', !State.shopDelivery);
        var priceEl = $('#order-price-unit');
        if (priceEl) {
            var pctDiff = (multiplier - 1) * 100;
            var isUp    = pctDiff > 0.05;
            var isDown  = pctDiff < -0.05;
            var sign    = isUp ? '+' : '';
            var arrow   = isUp ? '▲' : (isDown ? '▼' : '');
            var color   = isUp ? '#e05252' : (isDown ? '#5cb85c' : '');
            var varianceHtml = arrow
                ? ' <span style="font-size:11px;color:' + color + '">' + arrow + sign + pctDiff.toFixed(1) + '%</span>'
                : '';
            priceEl.innerHTML = '$' + effectivePrice.toFixed(2) + varianceHtml;
        }

        updateOrderBreakdown();
        modal.classList.remove('hidden');
    }

    function updateOrderBreakdown() {
        var item = State.orderModalItem;
        if (!item) return;
        var effectivePrice = item._effectivePrice !== undefined ? item._effectivePrice : item.price;
        var qty         = parseInt(($('#order-qty') || {}).value) || 1;
        var shippingPct = (State.catalogConfig && State.catalogConfig.ShippingFeePercent) || 12;
        var subtotal    = effectivePrice * qty;
        var shipping    = subtotal * (shippingPct / 100);
        var total       = subtotal + shipping;

        var subtotalEl  = $('#order-subtotal');  if (subtotalEl)  subtotalEl.textContent  = '$' + subtotal.toFixed(2);
        var shippingEl  = $('#order-shipping');  if (shippingEl)  shippingEl.textContent  = '$' + shipping.toFixed(2) + ' (' + shippingPct + '%)';
        var totalEl     = $('#order-total');     if (totalEl)     totalEl.textContent     = '$' + total.toFixed(2);
    }

    function confirmOrder() {
        var item = State.orderModalItem;
        if (!item) return;
        var qty = parseInt(($('#order-qty') || {}).value) || 1;
        if (qty < 1) return;
        var effectivePrice = item._effectivePrice !== undefined ? item._effectivePrice : item.price;
        var shopIdVal = parseInt(($('#order-shop-id') || {}).value) || 0;
        var payload = { itemName: item.name, quantity: qty, unitPrice: effectivePrice };
        if (shopIdVal > 0) payload.shopId = shopIdVal;
        sendNUI('placeShipmentOrder', payload);
        $('#order-modal').classList.add('hidden');
    }

    function fetchMyOrders() {
        sendNUI('getMyShipments').then(function (res) {
            State.myOrders = (res && res.orders) || [];
            renderMyOrders();
        });
    }

    function toggleOrdersView() {
        var catView = $('#catalogue-view');
        var ordView = $('#orders-view');
        if (!catView || !ordView) return;
        catView.classList.add('hidden');
        ordView.classList.remove('hidden');
        fetchMyOrders();
    }

    function backToCatalogue() {
        var catView = $('#catalogue-view');
        var ordView = $('#orders-view');
        if (catView) catView.classList.remove('hidden');
        if (ordView) ordView.classList.add('hidden');
    }

    function renderMyOrders() {
        var list = $('#orders-list');
        if (!list) return;

        var orders = State.myOrders || [];

        if (orders.length === 0) {
            list.innerHTML = '<div class="empty-state">No orders yet. Browse the catalogue to place an order.</div>';
            return;
        }

        var stepDefs = [
            { label: 'Order\nPlaced' },
            { label: 'Processing' },
            { label: 'In\nTransit' },
            { label: 'Out for\nDelivery' },
            { label: 'Delivered' }
        ];

        var statusActiveStep = {
            processing:       1,
            in_transit:       2,
            out_for_delivery: 3,
            delivered:        4,
            cancelled:        0
        };

        var statusLabel = {
            processing:       'Processing',
            in_transit:       'In Transit',
            out_for_delivery: 'Out for Delivery',
            delivered:        'Delivered',
            cancelled:        'Cancelled'
        };

        var now = Math.floor(Date.now() / 1000);

        var html = orders.map(function (order) {
            var activeStep = statusActiveStep[order.status] !== undefined ? statusActiveStep[order.status] : 1;
            var isCancelled = order.status === 'cancelled';
            var isDelivered = order.status === 'delivered';

            // Build tracker
            var trackerHtml = '';
            if (!isCancelled) {
                var stepsHtml = '';
                for (var i = 0; i < stepDefs.length; i++) {
                    var stepCls = i < activeStep ? 'step-done' : (i === activeStep ? 'step-active' : 'step-pending');
                    // label with \n converted to <br>
                    var labelText = stepDefs[i].label.replace(/\n/g, '<br>');
                    stepsHtml += '<div class="track-step ' + stepCls + '">'
                        + '<div class="track-dot"></div>'
                        + '<div class="track-label">' + labelText + '</div>'
                        + '</div>';
                    if (i < stepDefs.length - 1) {
                        var connCls = activeStep > i ? 'done' : '';
                        stepsHtml += '<div class="track-conn ' + connCls + '"></div>';
                    }
                }

                // ETA line
                var etaText = '';
                if (order.status === 'processing') {
                    var procEnd = parseExpiresAt(order.processing_ends);
                    var diff = procEnd - now;
                    etaText = diff > 0 ? '⏱ Processing — moves to transit in ~' + Math.ceil(diff / 60) + ' min' : '⏱ Completing processing...';
                } else if (order.status === 'in_transit') {
                    var transEnd = parseExpiresAt(order.transit_ends);
                    var diffT = transEnd - now;
                    etaText = diffT > 0 ? '🚚 En route — out for delivery in ~' + Math.ceil(diffT / 60) + ' min' : '🚚 Arriving soon...';
                } else if (order.status === 'out_for_delivery') {
                    var delivEnd = parseExpiresAt(order.delivery_ends);
                    var diffD = delivEnd - now;
                    etaText = diffD > 0 ? '📦 Out for delivery — arrives in ~' + Math.ceil(diffD / 60) + ' min' : '📦 Arriving any moment!';
                } else if (order.status === 'delivered') {
                    var shopId = order.delivery_shop_id && parseInt(order.delivery_shop_id);
                    etaText = shopId && shopId > 0
                        ? '✓ Delivered to Shop #' + shopId + ' — items added to shop storage'
                        : '✓ Delivered — check the mailbox to collect your items';
                }

                trackerHtml = '<div class="order-tracker">'
                    + '<div class="track-steps">' + stepsHtml + '</div>'
                    + (etaText ? '<div class="order-eta">' + escapeHtml(etaText) + '</div>' : '')
                    + '</div>';
            }

            var cardCls = 'order-card'
                + (isDelivered  ? ' order-delivered'  : '')
                + (isCancelled  ? ' order-cancelled'  : '');
            var badgeCls = 'order-status-badge order-status-' + (order.status || 'processing');
            var imgSrc = itemImageUrl(order.item_name);
            var cancelBtn = (order.status === 'processing')
                ? '<button class="btn-order-cancel" onclick="AuctionUI.cancelOrder(' + order.id + ')">Cancel</button>'
                : '';

            var reorderBtn = (order.status === 'delivered' || order.status === 'cancelled')
                ? '<button class="btn-order-reorder" onclick="AuctionUI.reorderShipment(' + order.id + ')">Reorder</button>'
                : '';

            var archiveBtn = (order.status === 'delivered' || order.status === 'cancelled')
                ? '<button class="btn-order-archive" onclick="AuctionUI.archiveShipment(' + order.id + ')">Archive</button>'
                : '';

            var shopDeliveryHtml = '';
            if (order.delivery_shop_id && parseInt(order.delivery_shop_id) > 0) {
                shopDeliveryHtml = '<div class="order-shop-delivery">🏪 Delivering to Shop #' + order.delivery_shop_id + '</div>';
            }

            return '<div class="' + cardCls + '" data-id="' + order.id + '">'
                + '<div class="order-card-header">'
                + '<div class="order-card-item">'
                + '<img class="order-item-img" src="' + imgSrc + '" onerror="this.style.display=\'none\'" alt="">'
                + '<div class="order-card-info">'
                + '<div class="order-item-name">' + escapeHtml(order.item_label || order.item_name) + ' &times;' + (order.quantity || 1) + '</div>'
                + '<div class="order-item-sub">' + escapeHtml(order.company_name || '') + '</div>'
                + shopDeliveryHtml
                + '</div>'
                + '</div>'
                + '<div class="order-card-right">'
                + '<span class="order-total-paid">$' + parseFloat(order.total_paid || 0).toFixed(2) + '</span>'
                + '<span class="' + badgeCls + '">' + escapeHtml(statusLabel[order.status] || order.status) + '</span>'
                + cancelBtn + reorderBtn + archiveBtn
                + '</div>'
                + '</div>'
                + trackerHtml
                + '</div>';
        }).join('');

        list.innerHTML = html;
    }

    function cancelOrder(orderId) {
        sendNUI('cancelShipmentOrder', { orderId: orderId });
        setTimeout(fetchMyOrders, 700);
    }

    function archiveShipment(orderId) {
        sendNUI('archiveShipmentOrder', { orderId: orderId });
        setTimeout(fetchMyOrders, 700);
    }

    function reorderShipment(orderId) {
        sendNUI('reorderShipment', { orderId: orderId });
        setTimeout(fetchMyOrders, 1000);
    }

    /* ========================================================================
       REQUESTS — Player Want Listings
       ======================================================================== */

    function fetchOpenRequests() {
        sendNUI('getOpenRequests', {
            search: State.requestsSearch || '',
            sort:   State.requestsSort   || 'newest'
        }).then(function (res) {
            State.openRequests = (res && res.requests) || [];
            renderOpenRequests();
        });
    }

    function fetchMyRequests() {
        sendNUI('getMyRequests', {}).then(function (res) {
            State.myRequests = (res && res.requests) || [];
            renderMyRequests();
        });
    }

    function renderOpenRequests() {
        var tbody = $('#requests-tbody');
        if (!tbody) return;

        var reqs = State.openRequests || [];
        if (reqs.length === 0) {
            tbody.innerHTML = '<tr><td colspan="7" class="empty-cell">No open requests found.</td></tr>';
            return;
        }

        var html = reqs.map(function (req) {
            var qtyLeft    = req.quantity - (req.qty_filled || 0);
            var escrowLeft = parseFloat(req.escrow_total || 0) - parseFloat(req.escrow_spent || 0);
            var expiry     = req.is_persistent == '1' || req.is_persistent === 1
                ? '<span class="badge-persistent">Persistent</span>'
                : (req.expires_at ? req.expires_at.replace('T', ' ').substring(0, 16) : '—');
            var imgSrc = itemImageUrl(req.item_name);

            return '<tr>'
                + '<td class="col-req-img"><img src="' + imgSrc + '" style="width:28px;height:28px;object-fit:contain;image-rendering:pixelated;vertical-align:middle" onerror="this.style.display=\'none\'"></td>'
                + '<td class="col-item">' + escapeHtml(req.item_label) + '</td>'
                + '<td class="col-qty">' + qtyLeft + ' / ' + req.quantity + '</td>'
                + '<td class="col-bid">$' + parseFloat(req.price_per_unit).toFixed(2) + '</td>'
                + '<td class="col-bid">$' + escrowLeft.toFixed(2) + '</td>'
                + '<td class="col-seller">' + escapeHtml(req.requester_name) + '</td>'
                + '<td class="col-time">' + expiry + '</td>'
                + '<td class="col-actions"><button class="btn-action btn-sm" onclick="AuctionUI.openFulfillModal(' + req.id + ')">Fulfill</button></td>'
                + '</tr>';
        }).join('');

        tbody.innerHTML = html;
    }

    function renderMyRequests() {
        var tbody = $('#my-requests-tbody');
        if (!tbody) return;

        var reqs = State.myRequests || [];
        if (reqs.length === 0) {
            tbody.innerHTML = '<tr><td colspan="8" class="empty-cell">No requests found.</td></tr>';
            return;
        }

        var statusColors = {
            open: '#5cb85c', partial: '#f0ad4e',
            filled: '#5bc0de', cancelled: '#d9534f', expired: '#888'
        };

        var html = reqs.map(function (req) {
            var escrowLeft = (parseFloat(req.escrow_total || 0) - parseFloat(req.escrow_spent || 0)).toFixed(2);
            var expiry = req.is_persistent == '1' || req.is_persistent === 1
                ? 'Persistent'
                : (req.expires_at ? req.expires_at.replace('T', ' ').substring(0, 16) : '—');
            var color     = statusColors[req.status] || '#888';
            var canCancel = req.status === 'open' || req.status === 'partial';
            var cancelBtn = canCancel
                ? '<button class="btn-action btn-sm btn-danger" onclick="AuctionUI.cancelMyRequest(' + req.id + ')">Cancel</button>'
                : '';

            return '<tr>'
                + '<td class="col-item">' + escapeHtml(req.item_label) + '</td>'
                + '<td class="col-qty">'  + req.quantity + '</td>'
                + '<td class="col-qty">'  + (req.qty_filled || 0) + '</td>'
                + '<td class="col-bid">$' + parseFloat(req.price_per_unit).toFixed(2) + '</td>'
                + '<td class="col-bid">$' + escrowLeft + '</td>'
                + '<td class="col-time">' + expiry + '</td>'
                + '<td class="col-status"><span style="color:' + color + '">' + req.status + '</span></td>'
                + '<td class="col-actions">' + cancelBtn + '</td>'
                + '</tr>';
        }).join('');

        tbody.innerHTML = html;
    }

    function buildRequestDurations() {
        var sel = $('#req-duration');
        if (!sel) return;
        var durations = (State.requestConfig && State.requestConfig.Durations) || [];
        if (!durations.length) {
            sel.innerHTML = '<option value="1">Short (4h)</option><option value="2" selected>Medium (12h)</option><option value="3">Long (24h)</option>';
            return;
        }
        var html = '';
        for (var i = 0; i < durations.length; i++) {
            html += '<option value="' + (i + 1) + '">' + escapeHtml(durations[i].label) + '</option>';
        }
        sel.innerHTML = html;
    }

    function updateRequestEscrow() {
        var qty   = parseInt(($('#req-quantity') || {}).value) || 0;
        var price = parseFloat(($('#req-price')  || {}).value) || 0;
        var escrow = qty * price;
        var escrowEl = $('#req-escrow');
        if (escrowEl) escrowEl.textContent = '$' + escrow.toFixed(2);
        var taxPct = (State.requestConfig && State.requestConfig.SalesTaxPercent) || 8;
        var taxEl  = $('#req-seller-tax');
        if (taxEl) taxEl.textContent = taxPct + '% (deducted from seller)';
    }

    function checkItemImage(itemName) {
        return new Promise(function (resolve) {
            var img = new Image();
            img.onload  = function () { resolve(true); };
            img.onerror = function () { resolve(false); };
            img.src = itemImageUrl(itemName);
        });
    }

    function searchItemsForRequest() {
        var query = (($('#req-item-search') || {}).value || '').trim();
        if (!query) return;
        var resultsEl = $('#req-item-results');
        if (resultsEl) { resultsEl.innerHTML = '<div class="req-item-no-results">Searching…</div>'; resultsEl.classList.remove('hidden'); }
        sendNUI('searchItems', { search: query }).then(function (res) {
            var items = (res && res.items) || [];
            if (!resultsEl) return;
            if (items.length === 0) {
                resultsEl.innerHTML = '<div class="req-item-no-results">No items found for "' + escapeHtml(query) + '".</div>';
                return;
            }
            // Filter to only items that have a valid inventory image
            Promise.all(items.map(function (item) {
                return checkItemImage(item.item).then(function (hasImg) {
                    return hasImg ? item : null;
                });
            })).then(function (filtered) {
                var valid = filtered.filter(Boolean);
                if (!resultsEl) return;
                if (valid.length === 0) {
                    resultsEl.innerHTML = '<div class="req-item-no-results">No items found for "' + escapeHtml(query) + '".</div>';
                    return;
                }
                resultsEl.innerHTML = valid.map(function (item) {
                    var imgSrc = itemImageUrl(item.item);
                    var label  = escapeHtml(item.label || item.item);
                    var desc   = escapeHtml(item.desc  || '');
                    return '<div class="req-item-card" onclick="AuctionUI.selectRequestItem(' + JSON.stringify(item).replace(/"/g, '&quot;') + ')">'
                        + '<img src="' + imgSrc + '">'
                        + '<div class="req-item-card-label">' + label + '</div>'
                        + (desc ? '<div class="req-item-card-desc">' + desc + '</div>' : '')
                        + '</div>';
                }).join('');
            });
        });
    }

    function selectRequestItem(item) {
        State.selectedRequestItem = item;
        var searchStep = $('#req-search-step');
        var formStep   = $('#req-form-step');
        var preview    = $('#req-selected-preview');
        if (searchStep) searchStep.classList.add('hidden');
        if (formStep)   formStep.classList.remove('hidden');
        if (preview) {
            var imgSrc = itemImageUrl(item.item);
            preview.innerHTML = '<img src="' + imgSrc + '" onerror="this.style.display=\'none\'">'
                + '<div class="req-selected-info">'
                + '<div class="req-selected-label">' + escapeHtml(item.label || item.item) + '</div>'
                + '<div class="req-selected-name">' + escapeHtml(item.item) + '</div>'
                + (item.desc ? '<div class="req-selected-desc">' + escapeHtml(item.desc) + '</div>' : '')
                + '</div>';
        }
        buildRequestDurations();
        updateRequestEscrow();
        var qtyEl = $('#req-quantity'); if (qtyEl) qtyEl.value = '1';
        var priceEl = $('#req-price'); if (priceEl) priceEl.value = '';
    }

    function clearRequestItem() {
        State.selectedRequestItem = null;
        var searchStep = $('#req-search-step');
        var formStep   = $('#req-form-step');
        var resultsEl  = $('#req-item-results');
        var searchInput = $('#req-item-search');
        if (searchStep) searchStep.classList.remove('hidden');
        if (formStep)   formStep.classList.add('hidden');
        if (resultsEl)  { resultsEl.innerHTML = ''; resultsEl.classList.add('hidden'); }
        if (searchInput) searchInput.value = '';
    }

    function submitRequest() {
        var item  = State.selectedRequestItem;
        if (!item) { alert('Please select an item first.'); return; }
        var qty    = parseInt(($('#req-quantity') || {}).value) || 0;
        var price  = parseFloat(($('#req-price')  || {}).value) || 0;
        var durIdx = parseInt(($('#req-duration') || {}).value) || 1;

        if (qty < 1)    { alert('Quantity must be at least 1.'); return; }
        if (price <= 0) { alert('Price per unit must be greater than 0.'); return; }

        sendNUI('createRequest', {
            itemName:    item.item,
            itemLabel:   item.label || item.item,
            quantity:    qty,
            pricePerUnit: price,
            durationIdx: durIdx
        });

        clearRequestItem();
        updateRequestEscrow();
    }

    function openFulfillModal(requestId) {
        var req = null;
        for (var i = 0; i < State.openRequests.length; i++) {
            if (State.openRequests[i].id == requestId) { req = State.openRequests[i]; break; }
        }
        if (!req) return;
        State.fulfillModalRequest = req;

        var modal = $('#fulfill-modal');
        if (!modal) return;

        var nameEl = $('#fulfill-item-name');
        if (nameEl) nameEl.textContent = req.item_label;
        var reqEl = $('#fulfill-requester');
        if (reqEl) reqEl.textContent = 'Requested by: ' + req.requester_name;

        var qtyLeft = req.quantity - (req.qty_filled || 0);
        var maxEl = $('#fulfill-max-qty');
        if (maxEl) maxEl.textContent = qtyLeft;

        var qtyInput = $('#fulfill-qty');
        if (qtyInput) { qtyInput.value = '1'; qtyInput.max = qtyLeft; }

        var payEl = $('#fulfill-pay-unit');
        if (payEl) payEl.textContent = '$' + parseFloat(req.price_per_unit).toFixed(2);

        updateFulfillBreakdown();
        modal.classList.remove('hidden');
    }

    function updateFulfillBreakdown() {
        var req = State.fulfillModalRequest;
        if (!req) return;
        var qty    = parseInt(($('#fulfill-qty') || {}).value) || 1;
        var gross  = qty * parseFloat(req.price_per_unit);
        var taxPct = (State.requestConfig && State.requestConfig.SalesTaxPercent) || 8;
        var tax    = gross * (taxPct / 100);
        var net    = gross - tax;

        var grossEl = $('#fulfill-gross'); if (grossEl) grossEl.textContent = '$' + gross.toFixed(2);
        var taxEl   = $('#fulfill-tax');   if (taxEl)   taxEl.textContent   = '-$' + tax.toFixed(2) + ' (' + taxPct + '%)';
        var netEl   = $('#fulfill-net');   if (netEl)   netEl.textContent   = '$' + net.toFixed(2);
    }

    function confirmFulfill() {
        var req = State.fulfillModalRequest;
        if (!req) return;
        var qty = parseInt(($('#fulfill-qty') || {}).value) || 1;
        if (qty < 1) return;
        sendNUI('fulfillRequest', { requestId: req.id, quantity: qty });
        $('#fulfill-modal').classList.add('hidden');
    }

    function cancelMyRequest(requestId) {
        sendNUI('cancelRequest', { requestId: requestId });
        setTimeout(fetchMyRequests, 700);
    }

})();
