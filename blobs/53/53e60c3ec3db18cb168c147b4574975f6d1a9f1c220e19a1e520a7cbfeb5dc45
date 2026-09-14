/* ============================================
   POGGY HELP UI — Futuristic Guide System
   State machine: grid → category → search
   ============================================ */
(function () {
    'use strict';

    /* ── SVG Icon Library ──────────────────────── */
    const ICONS = {
        compass: '<svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="9.5"/><polygon points="12,5 13.5,11 12,12.5 10.5,11" fill="currentColor" opacity="0.5"/><polygon points="12,19 10.5,13 12,11.5 13.5,13" fill="currentColor" opacity="0.8"/><line x1="12" y1="2" x2="12" y2="4"/><line x1="12" y1="20" x2="12" y2="22"/><line x1="2" y1="12" x2="4" y2="12"/><line x1="20" y1="12" x2="22" y2="12"/></svg>',

        book: '<svg viewBox="0 0 24 24"><path d="M4 4.5C4 3.12 5.12 2 6.5 2H20v20H6.5A2.5 2.5 0 014 19.5z"/><path d="M4 19.5A2.5 2.5 0 016.5 17H20"/><line x1="8" y1="7" x2="16" y2="7"/><line x1="8" y1="10.5" x2="14" y2="10.5"/></svg>',

        terminal: '<svg viewBox="0 0 24 24"><rect x="2" y="3" width="20" height="18" rx="2"/><polyline points="7,8 11,12 7,16"/><line x1="13" y1="16" x2="17" y2="16"/></svg>',

        envelope: '<svg viewBox="0 0 24 24"><rect x="2" y="4" width="20" height="16" rx="2"/><polyline points="2,4 12,13 22,4"/></svg>',

        crosshair: '<svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="8"/><line x1="12" y1="2" x2="12" y2="6"/><line x1="12" y1="18" x2="12" y2="22"/><line x1="2" y1="12" x2="6" y2="12"/><line x1="18" y1="12" x2="22" y2="12"/><circle cx="12" cy="12" r="1.5" fill="currentColor"/></svg>',

        fish: '<svg viewBox="0 0 24 24"><path d="M18 12c0 0-3-6-10-6C5 6 2 9 2 12s3 6 6 6c7 0 10-6 10-6z"/><circle cx="7" cy="11.5" r="1" fill="currentColor"/><path d="M18 12l4-4v8z" fill="currentColor" opacity="0.6"/></svg>',

        tent: '<svg viewBox="0 0 24 24"><path d="M12 3L2 20h20L12 3z"/><path d="M12 3v17"/><path d="M9 20l3-8 3 8"/></svg>',

        pickaxe: '<svg viewBox="0 0 24 24"><path d="M14.5 3L21 9.5 19 11.5 12.5 5z"/><line x1="5" y1="19" x2="12.5" y2="11.5"/><line x1="3" y1="21" x2="5" y2="19"/><circle cx="3.5" cy="20.5" r="1" fill="currentColor"/></svg>',

        plant: '<svg viewBox="0 0 24 24"><path d="M12 22V10"/><path d="M12 10C12 7 9 4 5 4c0 4 2 6 7 6"/><path d="M12 10c0-3 3-6 7-6-1 4-3 6-7 6"/><path d="M12 15c-2-1-5-1-6 1"/><path d="M12 15c2-1 5-1 6 1"/><line x1="8" y1="22" x2="16" y2="22"/></svg>',

        horse: '<svg viewBox="0 0 24 24"><path d="M17 3l2 2-2 4h3l-3 4-1 4.5"/><path d="M7 9c0 0-2 1-3 4s0 7 4 7h1l1-3"/><path d="M16 13.5c0 0 1 3 1 6.5h-2l-1-3"/><path d="M7 9l3-2 4 1 3-1"/><circle cx="8" cy="8" r="0.5" fill="currentColor"/></svg>',

        wheel: '<svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="9"/><circle cx="12" cy="12" r="2.5"/><line x1="12" y1="3" x2="12" y2="9.5"/><line x1="12" y1="14.5" x2="12" y2="21"/><line x1="3" y1="12" x2="9.5" y2="12"/><line x1="14.5" y1="12" x2="21" y2="12"/><line x1="5.6" y1="5.6" x2="10.2" y2="10.2"/><line x1="13.8" y1="13.8" x2="18.4" y2="18.4"/><line x1="18.4" y1="5.6" x2="13.8" y2="10.2"/><line x1="10.2" y1="13.8" x2="5.6" y2="18.4"/></svg>',

        hammer: '<svg viewBox="0 0 24 24"><path d="M15 3l6 6-2 2-6-6z"/><line x1="10" y1="10" x2="3" y2="21"/><line x1="5" y1="19" x2="7" y2="21"/></svg>',

        mask: '<svg viewBox="0 0 24 24"><path d="M4 10c0-4 3.5-6 8-6s8 2 8 6c0 5-3 8-8 8s-8-3-8-8z"/><path d="M7 10.5c0-1 1-2 2.5-2s2.5 1 2.5 2"/><path d="M11.5 10.5c0-1 1-2 2.5-2s2.5 1 2.5 2"/><line x1="12" y1="14" x2="12" y2="16"/></svg>',

        badge: '<svg viewBox="0 0 24 24"><polygon points="12,2 14.5,8 21,9 16.5,13.5 17.5,20 12,17 6.5,20 7.5,13.5 3,9 9.5,8"/></svg>',

        medcross: '<svg viewBox="0 0 24 24"><rect x="3" y="3" width="18" height="18" rx="3"/><line x1="12" y1="7" x2="12" y2="17"/><line x1="7" y1="12" x2="17" y2="12"/></svg>',

        coins: '<svg viewBox="0 0 24 24"><ellipse cx="10" cy="8" rx="7" ry="4"/><path d="M3 8v4c0 2.2 3.1 4 7 4"/><path d="M17 8v4c0 2.2-3.1 4-7 4"/><ellipse cx="15" cy="14" rx="6" ry="3.5"/><path d="M9 14v3c0 1.9 2.7 3.5 6 3.5s6-1.6 6-3.5v-3"/></svg>'
    };

    /* ── State ─────────────────────────────────── */
    let state = 'grid';       // 'grid' | 'category' | 'search'
    let categories = [];
    let activeCategory = null;
    let itemImageBase = '';   // inventory icon URL prefix, sent by client/help.lua

    /* ── DOM refs ──────────────────────────────── */
    const $overlay    = () => document.getElementById('help-overlay');
    const $hero       = () => document.getElementById('help-hero');
    const $grid       = () => document.getElementById('help-grid');
    const $detail     = () => document.getElementById('help-detail');
    const $search     = () => document.getElementById('help-search');
    const $results    = () => document.getElementById('help-search-results');
    const $crumb      = () => document.getElementById('help-breadcrumb');
    const $crumbTitle = () => document.getElementById('help-breadcrumb-title');
    const $unstuck    = () => document.getElementById('help-unstuck');

    /* ── Render Grid ──────────────────────────── */
    function renderGrid() {
        const hero = $hero();
        const grid = $grid();
        hero.innerHTML = '';
        grid.innerHTML = '';

        // Separate Getting Started from the rest
        var featured = null;
        var rest = [];
        categories.forEach(function (cat) {
            if (cat.id === 'getting_started') {
                featured = cat;
            } else {
                rest.push(cat);
            }
        });

        // Alphabetize the rest by title
        rest.sort(function (a, b) {
            return a.title.localeCompare(b.title);
        });

        // Render featured hero node
        if (featured) {
            var heroNode = document.createElement('div');
            heroNode.className = 'help-node help-node-hero';
            heroNode.style.setProperty('--i', 0);
            heroNode.style.setProperty('--node-color', featured.color);
            heroNode.style.setProperty('--node-glow', featured.color + '33');
            heroNode.style.setProperty('--node-bg', featured.color + '14');
            heroNode.setAttribute('data-id', featured.id);
            heroNode.innerHTML =
                '<div class="help-node-icon" style="color:' + featured.color + ';background:' + featured.color + '14">' +
                    (ICONS[featured.icon] || ICONS.compass) +
                '</div>' +
                '<div class="help-node-label">' + escHtml(featured.title) + '</div>' +
                '<div class="help-node-subtitle">New here? Start with the basics</div>';
            heroNode.addEventListener('click', function () { openCategory(featured); });
            hero.appendChild(heroNode);
        }

        // Render remaining nodes alphabetized
        rest.forEach(function (cat, i) {
            const node = document.createElement('div');
            node.className = 'help-node';
            node.style.setProperty('--i', i + 1);
            node.style.setProperty('--node-color', cat.color);
            node.style.setProperty('--node-glow', cat.color + '33');
            node.style.setProperty('--node-bg', cat.color + '14');
            node.setAttribute('data-id', cat.id);
            node.innerHTML =
                '<div class="help-node-icon" style="color:' + cat.color + ';background:' + cat.color + '14">' +
                    (ICONS[cat.icon] || ICONS.compass) +
                '</div>' +
                '<div class="help-node-label">' + escHtml(cat.title) + '</div>';
            node.addEventListener('click', function () { openCategory(cat); });
            grid.appendChild(node);
        });
    }

    /* ── Open Category ────────────────────────── */
    function openCategory(cat) {
        if (state === 'category' && activeCategory === cat.id) return;
        activeCategory = cat.id;
        state = 'category';

        // Clear search
        $search().value = '';
        $results().classList.remove('visible');
        $results().innerHTML = '';

        // Animate grid nodes out
        const hero = $hero();
        const grid = $grid();
        var heroNodes = hero.querySelectorAll('.help-node');
        var nodes = grid.querySelectorAll('.help-node');
        heroNodes.forEach(function (n) { n.classList.add('leaving'); });
        nodes.forEach(function (n) { n.classList.add('leaving'); });

        setTimeout(function () {
            hero.classList.add('hidden');
            grid.classList.add('hidden');
            $unstuck().classList.add('hidden');
            showDetail(cat);
        }, 300);

        // Show breadcrumb
        $crumb().classList.add('visible');
        $crumbTitle().textContent = cat.title;
    }

    /* ── Show Detail ──────────────────────────── */
    function showDetail(cat) {
        const detail = $detail();
        detail.innerHTML = '';

        // Header
        var hdr = document.createElement('div');
        hdr.id = 'help-detail-header';
        hdr.innerHTML =
            '<div id="help-detail-icon" style="color:' + cat.color + ';background:' + cat.color + '14">' +
                (ICONS[cat.icon] || ICONS.compass) +
            '</div>' +
            '<div id="help-detail-title">' + escHtml(cat.title) + '</div>';
        detail.appendChild(hdr);

        // Sections
        (cat.sections || []).forEach(function (sec, si) {
            var card = document.createElement('div');
            card.className = 'help-section';
            card.setAttribute('data-type', sec.type || 'info');
            card.style.setProperty('--si', si);

            var title = document.createElement('div');
            title.className = 'help-section-title';
            title.textContent = sec.title;
            card.appendChild(title);

            var items = document.createElement('div');
            items.className = 'help-section-items';

            if (sec.type === 'drug-crafting') {
                items.appendChild(renderDrugCraftingHelp(sec, cat.color));
            } else if (sec.type === 'medicine-grid') {
                items.appendChild(renderMedicineHelp(sec, cat.color));
            } else {
                (sec.items || []).forEach(function (txt, ii) {
                    var item = document.createElement('div');
                    item.className = 'help-item';
                    if (sec.type === 'steps') {
                        item.innerHTML = '<div class="help-item-bullet">' + (ii + 1) + '</div><div>' + escHtml(txt) + '</div>';
                    } else if (sec.type === 'tips') {
                        item.innerHTML = '<div class="help-item-bullet">💡</div><div>' + escHtml(txt) + '</div>';
                    } else {
                        item.innerHTML = '<div class="help-item-bullet"></div><div>' + escHtml(txt) + '</div>';
                    }
                    items.appendChild(item);
                });
            }

            card.appendChild(items);
            detail.appendChild(card);
        });

        detail.classList.add('visible');

        // scroll to top
        var body = document.getElementById('help-body');
        if (body) body.scrollTop = 0;
    }

    /* ── Back to Grid ─────────────────────────── */
    function backToGrid() {
        if (state === 'grid') return;
        state = 'grid';
        activeCategory = null;

        $detail().classList.remove('visible');
        $detail().innerHTML = '';
        $crumb().classList.remove('visible');

        var hero = $hero();
        var grid = $grid();
        hero.classList.remove('hidden');
        grid.classList.remove('hidden');
        $unstuck().classList.remove('hidden');

        // Re-animate nodes in
        var heroNodes = hero.querySelectorAll('.help-node');
        heroNodes.forEach(function (n, i) {
            n.classList.remove('leaving');
            n.style.setProperty('--i', 0);
            n.style.animation = 'none';
            n.offsetHeight;
            n.style.animation = '';
        });
        var nodes = grid.querySelectorAll('.help-node');
        nodes.forEach(function (n, i) {
            n.classList.remove('leaving');
            n.style.setProperty('--i', i + 1);
            n.style.animation = 'none';
            n.offsetHeight; // force reflow
            n.style.animation = '';
        });

        $results().classList.remove('visible');
        $results().innerHTML = '';
    }

    /* ── Search ───────────────────────────────── */
    var searchTimeout = null;
    function onSearch(e) {
        var q = (e.target.value || '').trim().toLowerCase();
        clearTimeout(searchTimeout);
        if (!q) {
            $results().classList.remove('visible');
            $results().innerHTML = '';
            if (state === 'search') { backToGrid(); }
            return;
        }
        searchTimeout = setTimeout(function () { doSearch(q); }, 180);
    }

    function doSearch(q) {
        // If in category view, go back first
        if (state === 'category') {
            $detail().classList.remove('visible');
            $detail().innerHTML = '';
            $crumb().classList.remove('visible');
        }
        state = 'search';

        // Hide grid, hero, and unstuck button
        $hero().classList.add('hidden');
        $grid().classList.add('hidden');
        $unstuck().classList.add('hidden');

        // Also filter category nodes visually
        var filtered = [];
        categories.forEach(function (cat) {
            var matches = [];
            // Check category title
            var catMatch = cat.title.toLowerCase().indexOf(q) !== -1;

            (cat.sections || []).forEach(function (sec) {
                // Section title match
                var secMatch = sec.title.toLowerCase().indexOf(q) !== -1;
                (sec.items || []).forEach(function (txt) {
                    if (txt.toLowerCase().indexOf(q) !== -1 || secMatch || catMatch) {
                        matches.push({ section: sec.title, text: txt, type: sec.type || 'info' });
                    }
                });
            });

            if (matches.length > 0 || catMatch) {
                filtered.push({ cat: cat, matches: matches });
            }
        });

        renderSearchResults(filtered, q);
    }

    function renderSearchResults(filtered, q) {
        var container = $results();
        container.innerHTML = '';

        if (filtered.length === 0) {
            container.innerHTML = '<div class="help-empty">No results for "' + escHtml(q) + '"</div>';
            container.classList.add('visible');
            return;
        }

        filtered.forEach(function (group, gi) {
            var gDiv = document.createElement('div');
            gDiv.className = 'help-search-group';
            gDiv.style.setProperty('--si', gi);

            // Group title (clickable → go to that category)
            var gTitle = document.createElement('div');
            gTitle.className = 'help-search-group-title';
            gTitle.innerHTML =
                '<div class="help-search-group-icon" style="color:' + group.cat.color + ';background:' + group.cat.color + '14">' +
                    (ICONS[group.cat.icon] || ICONS.compass) +
                '</div>' +
                highlightMatch(group.cat.title, q);
            gTitle.addEventListener('click', function () {
                $search().value = '';
                openCategory(group.cat);
            });
            gDiv.appendChild(gTitle);

            // Result items
            group.matches.forEach(function (m) {
                var iDiv = document.createElement('div');
                iDiv.className = 'help-search-item';
                iDiv.innerHTML =
                    '<div class="help-search-item-dot"></div>' +
                    '<div>' + highlightMatch(m.text, q) + '</div>';
                gDiv.appendChild(iDiv);
            });

            container.appendChild(gDiv);
        });

        container.classList.add('visible');
    }

    /* ── Highlight matched text ────────────────── */
    function highlightMatch(text, q) {
        if (!q) return escHtml(text);
        var escaped = escHtml(text);
        var qEsc = escRegex(q);
        var re = new RegExp('(' + qEsc + ')', 'gi');
        return escaped.replace(re, '<mark>$1</mark>');
    }

    /* ── Utilities ─────────────────────────────── */
    function escHtml(s) {
        var d = document.createElement('div');
        d.appendChild(document.createTextNode(s));
        return d.innerHTML;
    }
    function escRegex(s) { return s.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'); }

    /* ── Header branding (Config.Help.ServerName / Config.Help.Logo) ── */
    function applyBranding(serverName, logo) {
        var brand = document.getElementById('help-brand');
        if (brand) brand.textContent = serverName || '';
        var img = document.getElementById('help-logo');
        if (!img) return;
        if (logo) {
            img.hidden = false;
            if (img.getAttribute('src') !== logo) img.src = logo; // onerror hides it if the file is absent
        } else {
            img.hidden = true;
            img.removeAttribute('src');
        }
    }

    /* ── Close help ───────────────────────────── */
    function closeHelp() {
        var ov = $overlay();
        if (ov) ov.classList.remove('visible');
        state = 'grid';
        activeCategory = null;
        $.post(`https://${GetParentResourceName()}/closeHelp`, JSON.stringify({}));
    }

    /* ── NUI Message Handler ──────────────────── */
    window.addEventListener('message', function (event) {
        var data = event.data;
        if (!data || !data.action) return;

        if (data.action === 'openHelp') {
            if (data.categories && data.categories.length > 0) {
                categories = data.categories;
            }
            if (data.imageBase) {
                itemImageBase = data.imageBase;
            }
            applyBranding(data.serverName, data.logo);
            state = 'grid';
            activeCategory = null;

            var ov = $overlay();
            ov.classList.add('visible');

            // Reset views
            $hero().classList.remove('hidden');
            $grid().classList.remove('hidden');
            $unstuck().classList.remove('hidden');
            $detail().classList.remove('visible');
            $detail().innerHTML = '';
            $crumb().classList.remove('visible');
            $search().value = '';
            $results().classList.remove('visible');
            $results().innerHTML = '';

            renderGrid();
        }

        if (data.action === 'closeHelp') {
            closeHelp();
        }
    });

    /* ── Drug Crafting Pipeline Renderer ──────── */
    function renderDrugCraftingHelp(sec, catColor) {
        var IMG = itemImageBase;
        var wrapper = document.createElement('div');
        wrapper.className = 'dc-wrapper';

        if (sec.note) {
            var note = document.createElement('div');
            note.className = 'dc-note';
            note.textContent = sec.note;
            wrapper.appendChild(note);
        }

        var drugs = sec.drugs || [];
        if (!drugs.length) return wrapper;

        // Selector
        var selector = document.createElement('div');
        selector.className = 'dc-selector';
        wrapper.appendChild(selector);

        // Main area
        var mainArea = document.createElement('div');
        mainArea.className = 'dc-main';
        wrapper.appendChild(mainArea);

        function showDrugHelp(drug) {
            // Update active tab
            selector.querySelectorAll('.dc-drug-tab').forEach(function (t) {
                t.classList.toggle('active', t.dataset.id === drug.id);
            });

            mainArea.innerHTML = '';
            var dc = drug.color || '#c8a870';
            mainArea.style.setProperty('--dc', dc);

            // Profile (left)
            var profile = document.createElement('div');
            profile.className = 'dc-profile';
            profile.innerHTML =
                '<div class="dc-profile-glow"></div>' +
                '<div class="dc-profile-img-wrap">' +
                    '<div class="dc-profile-glow-ring"></div>' +
                    '<img class="dc-profile-img" src="' + IMG + escHtml(drug.image) + '.png" onerror="this.style.opacity=\'0.3\'">' +
                '</div>' +
                '<div class="dc-profile-name">' + escHtml(drug.name) + '</div>' +
                '<div class="dc-profile-desc">' + escHtml(drug.description || '') + '</div>';

            if (drug.stats) {
                var stats = document.createElement('div');
                stats.className = 'dc-profile-stats';
                Object.entries(drug.stats).forEach(function (kv) {
                    stats.innerHTML += '<div class="dc-stat"><div class="dc-stat-label">' + escHtml(kv[0]) + '</div><div class="dc-stat-value">' + escHtml(kv[1]) + '</div></div>';
                });
                profile.appendChild(stats);
            }

            function addSec(label, text) {
                if (!text) return;
                var s = document.createElement('div');
                s.className = 'dc-profile-section';
                s.innerHTML = '<div class="dc-profile-section-label">' + escHtml(label) + '</div><div class="dc-profile-section-text">' + escHtml(text) + '</div>';
                profile.appendChild(s);
            }
            addSec('How to Obtain', drug.acquisition);
            addSec('How to Use', drug.use);
            mainArea.appendChild(profile);

            // Pipeline (right)
            var pipeline = document.createElement('div');
            pipeline.className = 'dc-pipeline';
            pipeline.style.setProperty('--dc', dc);
            (drug.steps || []).forEach(function (step, si) {
                if (si > 0) {
                    var con = document.createElement('div');
                    con.className = 'dc-connector';
                    con.innerHTML = '<div class="dc-connector-track"><div class="dc-connector-dot"></div></div>';
                    pipeline.appendChild(con);
                }
                var stepEl = document.createElement('div');
                stepEl.className = 'dc-step';
                stepEl.style.setProperty('--si', si);

                var hdr = '<div class="dc-step-header">' +
                    '<div class="dc-step-badge">' + escHtml(step.stage) + '</div>' +
                    '<div class="dc-step-name">' + escHtml(step.name) + '</div>' +
                    '<div class="dc-step-location">📍 ' + escHtml(step.location) + '</div>' +
                    '</div>';

                var ingHtml = '<div class="dc-ingredients">';
                (step.inputs || []).forEach(function (inp, ii) {
                    if (ii > 0) ingHtml += '<div class="dc-plus">+</div>';
                    ingHtml += '<div class="dc-ingredient">' +
                        '<div class="dc-ingredient-img-wrap">' +
                            '<img src="' + IMG + escHtml(inp.item) + '.png" onerror="this.style.opacity=\'0.3\'" title="' + escHtml(inp.name) + '">' +
                            '<div class="dc-ingredient-count">×' + inp.count + '</div>' +
                        '</div>' +
                        '<div class="dc-ingredient-name">' + escHtml(inp.name) + '</div>' +
                        '</div>';
                });
                ingHtml += '</div>';

                var yieldHtml = '<div class="dc-yield">' +
                    '<div class="dc-yield-line"><div class="dc-yield-flow"></div></div>' +
                    '<div class="dc-yield-label">Yields</div>' +
                    '<svg class="dc-yield-arrow" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="6 9 12 15 18 9"/></svg>' +
                    '</div>';

                var out = step.output || {};
                var outHtml = '<div class="dc-output">' +
                    '<div class="dc-output-img-wrap">' +
                        '<div class="dc-output-glow"></div>' +
                        '<img src="' + IMG + escHtml(out.item || '') + '.png" onerror="this.style.opacity=\'0.3\'" title="' + escHtml(out.name || '') + '">' +
                        (out.count ? '<div class="dc-output-count">×' + out.count + '</div>' : '') +
                    '</div>' +
                    '<div class="dc-output-name">' + escHtml(out.name || '') + '</div>' +
                    '</div>';

                stepEl.innerHTML = hdr + ingHtml + yieldHtml + outHtml;
                pipeline.appendChild(stepEl);
            });
            mainArea.appendChild(pipeline);
        }

        // Build tabs and show first drug
        drugs.forEach(function (drug) {
            var tab = document.createElement('div');
            tab.className = 'dc-drug-tab';
            tab.dataset.id = drug.id;
            tab.style.setProperty('--dc', drug.color || '#c8a870');
            tab.innerHTML =
                '<div class="dc-tab-img-wrap">' +
                    '<img src="' + IMG + escHtml(drug.image) + '.png" onerror="this.style.opacity=\'0.3\'">' +
                '</div>' +
                '<div class="dc-tab-name">' + escHtml(drug.name) + '</div>';
            tab.addEventListener('click', function () { showDrugHelp(drug); });
            selector.appendChild(tab);
        });
        showDrugHelp(drugs[0]);

        return wrapper;
    }

    /* ── Medicine Grid Renderer ────────────────── */
    function renderMedicineHelp(sec, catColor) {
        var IMG = itemImageBase;
        var wrapper = document.createElement('div');
        wrapper.className = 'med-grid';

        (sec.medicines || []).forEach(function (med) {
            var card = document.createElement('div');
            card.className = 'med-card' + (med.doctorOnly ? ' med-card-doctor' : '');

            // Glowing image
            var imgWrap = document.createElement('div');
            imgWrap.className = 'med-card-img-wrap';
            var glow = document.createElement('div');
            glow.className = 'med-card-img-glow';
            var img = document.createElement('img');
            img.src = IMG + escHtml(med.item) + '.png';
            img.onerror = function () { this.style.opacity = '0.2'; };
            imgWrap.appendChild(glow);
            imgWrap.appendChild(img);
            card.appendChild(imgWrap);

            // Doctor-only badge
            if (med.doctorOnly) {
                var badge = document.createElement('div');
                badge.className = 'med-doctor-badge';
                badge.textContent = '\u2695 DOCTOR ONLY';
                card.appendChild(badge);
            }

            // Name
            var nameEl = document.createElement('div');
            nameEl.className = 'med-card-name';
            nameEl.textContent = med.name;
            card.appendChild(nameEl);

            // Description
            if (med.desc) {
                var descEl = document.createElement('div');
                descEl.className = 'med-card-desc';
                descEl.textContent = med.desc;
                card.appendChild(descEl);
            }

            // Condition tags
            if (med.conditions && med.conditions.length > 0) {
                var condRow = document.createElement('div');
                condRow.className = 'med-conditions';
                med.conditions.forEach(function (cond) {
                    var tag = document.createElement('div');
                    tag.className = 'med-condition';
                    tag.style.setProperty('--cc', cond.color || catColor);
                    if (cond.pct < 100) {
                        tag.innerHTML = escHtml(cond.label) + ' <span class="med-cond-pct">' + cond.pct + '%</span>';
                    } else {
                        tag.textContent = cond.label;
                    }
                    condRow.appendChild(tag);
                });
                card.appendChild(condRow);
            }

            // Stats row
            var statsRow = document.createElement('div');
            statsRow.className = 'med-stats';
            if (med.health > 0) {
                var hpStat = document.createElement('div');
                hpStat.className = 'med-stat';
                hpStat.innerHTML = '<span class="med-stat-label">HP</span><span class="med-stat-value">+' + med.health + '</span>';
                statsRow.appendChild(hpStat);
            }
            if (med.healAmount) {
                var haStat = document.createElement('div');
                haStat.className = 'med-stat';
                haStat.innerHTML = '<span class="med-stat-label">BODY HEAL</span><span class="med-stat-value med-stat-body">+' + med.healAmount + '</span>';
                statsRow.appendChild(haStat);
            }
            if (med.permanent) {
                var permStat = document.createElement('div');
                permStat.className = 'med-stat';
                permStat.innerHTML = '<span class="med-stat-label">DURATION</span><span class="med-stat-value med-stat-perm">Permanent</span>';
                statsRow.appendChild(permStat);
            }
            if (med.note) {
                var noteStat = document.createElement('div');
                noteStat.className = 'med-stat med-stat-warn';
                noteStat.innerHTML = '<span class="med-stat-label">\u26a0</span><span class="med-stat-value">' + escHtml(med.note) + '</span>';
                statsRow.appendChild(noteStat);
            }
            if (statsRow.children.length > 0) {
                card.appendChild(statsRow);
            }

            wrapper.appendChild(card);
        });

        return wrapper;
    }

    /* ── DOM ready: bind events ────────────────── */
    $(document).ready(function () {
        // Close button
        $(document).on('click', '#help-close', function () { closeHelp(); });

        // Unstuck button
        $(document).on('click', '#help-unstuck', function () {
            $.post(`https://${GetParentResourceName()}/unstuckFromHelp`, JSON.stringify({}));
            closeHelp();
        });

        // Back button
        $(document).on('click', '#help-back', function () {
            $search().value = '';
            backToGrid();
        });

        // Search
        $(document).on('input', '#help-search', onSearch);

        // ESC key — only close if help is visible
        $(document).on('keydown', function (e) {
            if (e.key === 'Escape') {
                var ov = $overlay();
                if (ov && ov.classList.contains('visible')) {
                    closeHelp();
                }
            }
        });
    });
})();
