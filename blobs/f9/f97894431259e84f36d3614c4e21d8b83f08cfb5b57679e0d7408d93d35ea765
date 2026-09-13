/* =========================================================================
   poggy_fishing_journal  —  NUI APP  v2
   Redesigned: water body modal, right lore sidebar, fixed-size pins,
   achievement toast at bottom, fish icons from inventory
   ========================================================================= */

(() => {
    "use strict";

    // ---- STATE ----
    let waterBodies    = [];
    let fishConfig     = {};
    let zoneFish       = {};
    let inSeason       = {};
    let discoveries    = {};
    let gameTime       = { hour: 12, minute: 0 };
    let mapConfig      = {};
    let typeColours    = {};
    let sizeLabels     = {};
    let speciesLore    = {};
    let waterBodyLore  = {};
    let catchCounts    = {};
    let weightData     = {};   // { fishKey: { best, total } }
    let selectedBody   = null;
    let loreOpen       = false;
    let modalOpen      = false;
    let currentTab     = "journal";
    let recordsSort    = "count";
    let pbsSort        = "weight";

    // Map pan/zoom
    let scale     = 1;
    let panX      = 0;
    let panY      = 0;
    let dragging  = false;
    let dragStart = { x: 0, y: 0 };
    const MIN_SCALE = 0.3;
    const MAX_SCALE = 4;

    // ---- HELPERS ----
    let ICON_BASE = "";   // item icon URL prefix, sent by the client (poggy_core inv.imageBase)
    function fishIconUrl(cfg) {
        return cfg && cfg.item ? `${ICON_BASE}${cfg.item}.png` : "";
    }

    function formatHour(h) {
        if (h === 0 || h === 24) return "12AM";
        if (h === 12) return "12PM";
        return h < 12 ? `${h}AM` : `${h - 12}PM`;
    }

    function formatWeight(cfg) {
        if (!cfg) return "";
        const lo = cfg.weightMin != null ? cfg.weightMin : "?";
        const hi = cfg.weightMax != null ? cfg.weightMax : "?";
        return `${lo} – ${hi} lbs`;
    }

    function isFishActiveNow(fishKey) {
        const cfg = fishConfig[fishKey];
        if (!cfg || !cfg.hours) return true;
        const [start, end] = cfg.hours;
        const h = gameTime.hour;
        if (start < end) return h >= start && h < end;
        return h >= start || h < end;
    }

    // ---- DOM REFS ----
    const $ = id => document.getElementById(id);
    const container      = $("journalContainer");
    const closeBtn       = $("closeBtn");
    const gameTimeEl     = $("gameTime");
    const inSeasonEl     = $("inSeasonCount");
    const outSeasonEl    = $("outOfSeasonCount");
    const activeFishEl   = $("activeFishList");
    const progressBar    = $("progressBar");
    const progressText   = $("progressText");
    const mapArea        = $("mapArea");
    const mapContainer   = $("mapContainer");
    const mapImage       = $("mapImage");
    const mapPinsEl      = $("mapPins");

    // Lore sidebar
    const lorePanel      = $("fishLorePanel");
    const loreCloseBtn   = $("loreCloseBtn");
    const loreFishIcon   = $("loreFishIcon");
    const loreName       = $("loreFishName");
    const loreSci        = $("loreScientific");
    const loreFamilyEl   = $("loreFamily");
    const loreSizeBadge  = $("loreSizeBadge");
    const loreWeight     = $("loreWeight");
    const lorePrice      = $("lorePrice");
    const loreHours      = $("loreHours");
    const loreSummary    = $("loreSummary");
    const loreHistory    = $("loreHistory");
    const loreHabitat    = $("loreHabitat");
    const loreBehavior   = $("loreBehavior");
    const loreFunFact    = $("loreFunFact");
    const loreLocations  = $("loreLocations");

    // Modal
    const modalOverlay   = $("waterBodyModal");
    const modalCloseBtn  = $("modalCloseBtn");
    const modalWaterName = $("modalWaterName");
    const modalWaterType = $("modalWaterType");
    const modalWaterLore = $("modalWaterLore");
    const modalFishList  = $("modalFishList");

    // Records
    const journalTabEl    = $("journalTabContent");
    const recordsTabEl    = $("recordsTabContent");
    const discoveriesTabEl = $("discoveriesTabContent");
    const pbsTabEl         = $("pbsTabContent");
    const pbsListEl        = $("pbsList");
    const totalCatchesEl  = $("totalCatches");
    const uniqueSpeciesEl = $("uniqueSpecies");
    const totalWeightEl   = $("totalWeight");
    const recordsListEl   = $("recordsList");

    // Discoveries tab
    const discoveryRingFill = $("discoveryRingFill");
    const discoveryPctEl    = $("discoveryPct");
    const discoveryCountEl  = $("discoveryCount");
    const discoveryGridEl   = $("discoveryGrid");

    // Discovery toast (outside journal container — always visible)
    const discoveryToast     = $("discoveryToast");
    const discoveryIcon      = $("discoveryIcon");
    const discoveryFishLabel = $("discoveryFishLabel");
    const discoveryLocation  = $("discoveryLocation");
    let discoveryTimer = null;

    const recordToast     = $("recordToast");
    const recordFishLabel = $("recordFishLabel");
    const recordWeight    = $("recordWeight");
    let recordTimer = null;

    // Tab content map for switching
    const tabEls = {
        journal: journalTabEl,
        discoveries: discoveriesTabEl,
        records: recordsTabEl,
        pbs: pbsTabEl,
    };

    // ==================== NUI MESSAGE HANDLER ====================
    window.addEventListener("message", (e) => {
        const d = e.data;
        switch (d.action) {
            case "openJournal":
                if (d.imageBase) ICON_BASE = d.imageBase;
                waterBodies   = d.waterBodies   || [];
                fishConfig    = d.fishConfig     || {};
                zoneFish      = d.zoneFish       || {};
                inSeason      = d.inSeason       || {};
                discoveries   = d.discoveries    || {};
                gameTime      = d.gameTime       || { hour: 12, minute: 0 };
                mapConfig     = d.mapConfig       || {};
                typeColours   = d.typeColours     || {};
                sizeLabels    = d.sizeLabels      || {};
                speciesLore   = d.speciesLore     || {};
                waterBodyLore = d.waterBodyLore   || {};
                catchCounts   = d.catchCounts     || {};
                weightData    = d.weightData      || {};
                selectedBody  = null;
                loreOpen = false;
                modalOpen = false;
                currentTab = "journal";
                openUI();
                break;

            case "closeJournal":
                closeUI();
                break;

            case "addDiscovery":
                addDiscovery(d.fishKey, d.zoneHash);
                break;

            case "showDiscoveryToast":
                showDiscoveryToast(d.fishLabel, d.location, d.icon);
                break;

            case "updateSeasons":
                inSeason = d.inSeason || {};
                renderSeasonStats();
                renderActiveFish();
                if (modalOpen && selectedBody) renderModalFish(selectedBody);
                break;

            case "updateCatchCount":
                catchCounts[d.fishKey] = d.count;
                if (d.fishWeightData) weightData[d.fishKey] = d.fishWeightData;
                if (currentTab === "records") renderRecords();
                break;

            case "showRecordToast":
                showRecordToast(d.fishLabel, d.newBest, d.oldBest);
                break;

            case "updateTime":
                gameTime = d.gameTime || gameTime;
                renderTime();
                renderActiveFish();
                break;
        }
    });

    // ==================== KEYBOARD ====================
    window.addEventListener("keydown", (e) => {
        if (e.key === "Escape") {
            if (modalOpen) {
                closeModal();
            } else if (loreOpen) {
                closeLorePanel();
            } else {
                fetch(`https://${GetParentResourceName()}/closeJournal`, {
                    method: "POST",
                    body: JSON.stringify({}),
                });
            }
        }
    });

    // ==================== CLOSE BUTTON ====================
    closeBtn.addEventListener("click", () => {
        fetch(`https://${GetParentResourceName()}/closeJournal`, {
            method: "POST",
            body: JSON.stringify({}),
        });
    });

    // ==================== OPEN / CLOSE ====================
    function openUI() {
        container.classList.remove("hidden");
        mapImage.src = mapConfig.imageUrl || "";

        // Reset tabs
        document.querySelectorAll(".tab-btn").forEach(b => b.classList.remove("active"));
        document.querySelector('.tab-btn[data-tab="journal"]').classList.add("active");
        Object.values(tabEls).forEach(el => el.classList.add("hidden"));
        journalTabEl.classList.remove("hidden");
        currentTab = "journal";

        renderTime();
        renderSeasonStats();
        renderActiveFish();
        renderMapPins();
        renderProgress();
        centerMap();
        closeLorePanel();
        closeModal();
        startBgMusic();
    }

    function closeUI() {
        container.classList.add("hidden");
        selectedBody = null;
        loreOpen = false;
        modalOpen = false;
        stopSpeciesAudio();
        stopBgMusic();
    }

    // ==================== TAB SWITCHING ====================
    function switchTab(tab) {
        if (tab === currentTab) return;
        currentTab = tab;
        document.querySelectorAll(".tab-btn").forEach(b => b.classList.remove("active"));
        const btn = document.querySelector(`.tab-btn[data-tab="${tab}"]`);
        if (btn) btn.classList.add("active");

        Object.entries(tabEls).forEach(([key, el]) => {
            if (key === tab) el.classList.remove("hidden");
            else el.classList.add("hidden");
        });

        if (tab === "records") renderRecords();
        if (tab === "discoveries") renderDiscoveries();
        if (tab === "pbs") renderPBs();
    }

    document.querySelectorAll(".tab-btn").forEach(btn => {
        btn.addEventListener("click", () => switchTab(btn.dataset.tab));
    });

    // Click discovery progress bar to open Discoveries tab
    document.querySelector(".progress-section").addEventListener("click", () => switchTab("discoveries"));

    // ==================== SORT BUTTONS ====================
    document.querySelectorAll(".sort-btn").forEach(btn => {
        btn.addEventListener("click", () => {
            recordsSort = btn.dataset.sort;
            document.querySelectorAll(".sort-btn").forEach(b => b.classList.remove("active"));
            btn.classList.add("active");
            renderRecords();
        });
    });

    document.querySelectorAll(".pb-sort-btn").forEach(btn => {
        btn.addEventListener("click", () => {
            pbsSort = btn.dataset.pbsort;
            document.querySelectorAll(".pb-sort-btn").forEach(b => b.classList.remove("active"));
            btn.classList.add("active");
            renderPBs();
        });
    });

    // ==================== TIME ====================
    function renderTime() {
        const h = String(gameTime.hour).padStart(2, "0");
        const m = String(gameTime.minute).padStart(2, "0");
        gameTimeEl.textContent = `${h}:${m}`;
    }

    // ==================== SEASON STATS ====================
    function renderSeasonStats() {
        const allKeys = Object.keys(fishConfig);
        let inC = 0, outC = 0;
        allKeys.forEach(k => { if (inSeason[k]) inC++; else outC++; });
        inSeasonEl.textContent  = inC;
        outSeasonEl.textContent = outC;
    }

    // ==================== ACTIVE FISH (sidebar) ====================
    let inactiveCollapsed = true; // collapsed by default

    function buildFishRow(key, cfg, active, season) {
        const row = document.createElement("div");
        row.className = `fish-row${active ? "" : " inactive"}`;

        const icon = document.createElement("img");
        icon.className = "fish-row-icon";
        icon.src = fishIconUrl(cfg);
        icon.alt = "";
        icon.onerror = function(){ this.style.display = "none"; };

        const name = document.createElement("span");
        name.className = "fish-row-name";
        name.textContent = cfg.label;

        const badge = document.createElement("span");
        badge.className = `fish-size-badge ${cfg.size}`;
        badge.textContent = cfg.size.toUpperCase();

        const dot = document.createElement("span");
        dot.className = `fish-season-dot ${season ? "in" : "out"}`;

        row.appendChild(icon);
        row.appendChild(name);
        row.appendChild(badge);
        row.appendChild(dot);

        row.addEventListener("click", () => openLorePanel(key));
        return row;
    }

    function renderActiveFish() {
        activeFishEl.innerHTML = "";
        const allKeys = Object.keys(fishConfig).sort((a, b) =>
            fishConfig[a].label.localeCompare(fishConfig[b].label)
        );

        const activeRows = [];
        const inactiveRows = [];

        allKeys.forEach(key => {
            const cfg    = fishConfig[key];
            const active = isFishActiveNow(key);
            const season = !!inSeason[key];
            if (!active && !season) return;

            const row = buildFishRow(key, cfg, active, season);
            if (active) activeRows.push(row);
            else inactiveRows.push(row);
        });

        // Active fish
        if (activeRows.length === 0 && inactiveRows.length === 0) {
            activeFishEl.innerHTML = '<div class="fish-row inactive"><span class="fish-row-name">No fish active right now</span></div>';
            return;
        }

        activeRows.forEach(r => activeFishEl.appendChild(r));

        // Inactive fish — collapsible group
        if (inactiveRows.length > 0) {
            const header = document.createElement("div");
            header.className = `fish-group-header${inactiveCollapsed ? " collapsed" : ""}`;
            header.innerHTML = `<span class="fish-group-chevron">▼</span><span class="section-title" style="margin-bottom:0">Inactive</span><span class="fish-group-count">${inactiveRows.length}</span>`;

            const body = document.createElement("div");
            body.className = `fish-group-body${inactiveCollapsed ? " collapsed" : ""}`;
            inactiveRows.forEach(r => body.appendChild(r));

            // Set natural max-height for animation when expanded
            if (!inactiveCollapsed) {
                requestAnimationFrame(() => { body.style.maxHeight = body.scrollHeight + "px"; });
            }

            header.addEventListener("click", () => {
                inactiveCollapsed = !inactiveCollapsed;
                header.classList.toggle("collapsed", inactiveCollapsed);
                body.classList.toggle("collapsed", inactiveCollapsed);
                if (!inactiveCollapsed) {
                    body.style.maxHeight = body.scrollHeight + "px";
                } else {
                    body.style.maxHeight = "";
                }
            });

            activeFishEl.appendChild(header);
            activeFishEl.appendChild(body);
        }
    }

    // ==================== DISCOVERY PROGRESS ====================
    function renderProgress() {
        let total = 0, found = 0;
        waterBodies.forEach(body => {
            const fishKeys = zoneFish[body.zoneHash] || [];
            const zoneDisc = discoveries[body.zoneHash] || {};
            fishKeys.forEach(key => { total++; if (zoneDisc[key]) found++; });
        });
        const pct = total > 0 ? (found / total) * 100 : 0;
        progressBar.style.width = `${pct}%`;
        progressText.textContent = `${found} / ${total}`;
    }

    // ==================== DISCOVERIES TAB ====================
    function renderDiscoveries() {
        // Collect unique fish across all zones, track discovered status
        let total = 0, found = 0;
        const fishMap = {}; // fishKey → { cfg, discoveredAt: [bodyLabel...] }

        waterBodies.forEach(body => {
            const fishKeys = zoneFish[body.zoneHash] || [];
            const zoneDisc = discoveries[body.zoneHash] || {};
            fishKeys.forEach(key => {
                total++;
                const disc = !!zoneDisc[key];
                if (disc) found++;
                if (!fishMap[key]) {
                    fishMap[key] = { cfg: fishConfig[key], discoveredAt: [] };
                }
                if (disc) fishMap[key].discoveredAt.push(body.label);
            });
        });

        // Update ring
        const pct = total > 0 ? Math.round((found / total) * 100) : 0;
        const circumference = 2 * Math.PI * 20; // r=20
        discoveryRingFill.style.strokeDashoffset = circumference - (circumference * pct / 100);
        discoveryPctEl.textContent = `${pct}%`;
        discoveryCountEl.textContent = `${found} / ${total}`;

        // Build sorted list: discovered first (by name), then undiscovered
        const entries = Object.entries(fishMap)
            .map(([key, data]) => ({ key, ...data, isDiscovered: data.discoveredAt.length > 0 }));
        entries.sort((a, b) => {
            if (a.isDiscovered !== b.isDiscovered) return a.isDiscovered ? -1 : 1;
            if (!a.cfg || !b.cfg) return 0;
            return a.cfg.label.localeCompare(b.cfg.label);
        });

        discoveryGridEl.innerHTML = "";

        if (entries.length === 0) {
            discoveryGridEl.innerHTML = '<div class="records-empty">No fish species found yet.<br>Explore the waters!</div>';
            return;
        }

        entries.forEach(({ key, cfg, discoveredAt, isDiscovered }) => {
            if (!cfg) return;
            const el = document.createElement("div");
            el.className = `discovery-item${isDiscovered ? "" : " undiscovered"}`;

            const icon = document.createElement("img");
            icon.className = "discovery-item-icon";
            if (isDiscovered) {
                icon.src = fishIconUrl(cfg);
                icon.alt = cfg.label;
            }
            icon.onerror = function(){ this.style.display = "none"; };

            const info = document.createElement("div");
            info.className = "discovery-item-info";

            const nameEl = document.createElement("span");
            nameEl.className = "discovery-item-name";
            nameEl.textContent = isDiscovered ? cfg.label : "???";

            const locEl = document.createElement("span");
            locEl.className = "discovery-item-locations";
            if (isDiscovered && discoveredAt.length > 0) {
                locEl.textContent = discoveredAt.join(", ");
            }

            info.appendChild(nameEl);
            info.appendChild(locEl);

            const check = document.createElement("span");
            check.className = `discovery-item-check ${isDiscovered ? "found" : "missing"}`;
            check.textContent = isDiscovered ? "✓" : "—";

            el.appendChild(icon);
            el.appendChild(info);
            el.appendChild(check);

            if (isDiscovered) {
                el.addEventListener("click", () => openLorePanel(key));
            }

            discoveryGridEl.appendChild(el);
        });
    }

    // ==================== MAP PINS (fixed size) ====================
    function gameToMapPercent(coords) {
        if (!mapConfig.gameBounds) return { x: 50, y: 50 };
        const b = mapConfig.gameBounds;
        const x = ((coords[0] - b.minX) / (b.maxX - b.minX)) * 100;
        const y = (1 - (coords[1] - b.minY) / (b.maxY - b.minY)) * 100;
        return { x, y };
    }

    function renderMapPins() {
        mapPinsEl.innerHTML = "";
        waterBodies.forEach(body => {
            const pos = gameToMapPercent(body.coords);
            const pin = document.createElement("div");
            pin.className = "map-pin";
            pin.style.left = `${pos.x}%`;
            pin.style.top  = `${pos.y}%`;
            pin.style.background = typeColours[body.type] || "#4a9eff";
            pin.dataset.bodyId = body.id;
            pin.innerHTML = `<span class="pin-label">${body.label}</span>`;
            pin.addEventListener("click", (e) => {
                e.stopPropagation();
                selectWaterBody(body);
            });
            mapPinsEl.appendChild(pin);
        });
        updatePinScale();
    }

    // Counter-scale pins so they stay fixed size regardless of zoom
    function updatePinScale() {
        const invScale = 1 / scale;
        document.querySelectorAll(".map-pin").forEach(pin => {
            pin.style.transform = `translate(-50%, -50%) scale(${invScale})`;
        });
    }

    function selectWaterBody(body) {
        document.querySelectorAll(".map-pin.selected").forEach(p => p.classList.remove("selected"));
        const pin = document.querySelector(`.map-pin[data-body-id="${body.id}"]`);
        if (pin) pin.classList.add("selected");
        selectedBody = body;
        openModal(body);
    }

    // ==================== WATER BODY MODAL ====================
    function openModal(body) {
        modalWaterName.textContent = body.label;

        const colour = typeColours[body.type] || "#4a9eff";
        modalWaterType.textContent = body.type;
        modalWaterType.style.background = colour + "20";
        modalWaterType.style.color = colour;
        modalWaterType.style.borderColor = colour + "40";

        const lore = waterBodyLore[body.id];
        modalWaterLore.textContent = lore || "";
        modalWaterLore.style.display = lore ? "block" : "none";

        renderModalFish(body);
        modalOverlay.classList.remove("hidden");
        modalOpen = true;
    }

    function renderModalFish(body) {
        const fishKeys = zoneFish[body.zoneHash] || [];
        const zoneDisc = discoveries[body.zoneHash] || {};

        // Group by size
        const groups = { sm: [], md: [], lg: [], xl: [] };
        fishKeys.forEach(key => {
            const cfg = fishConfig[key];
            if (!cfg) return;
            const discovered = !!zoneDisc[key];
            groups[cfg.size] = groups[cfg.size] || [];
            groups[cfg.size].push({ key, cfg, discovered });
        });

        modalFishList.innerHTML = "";
        const sizeOrder = ["sm", "md", "lg", "xl"];
        sizeOrder.forEach(sz => {
            const items = groups[sz];
            if (!items || items.length === 0) return;

            // Size header
            const header = document.createElement("div");
            header.style.cssText = "font-size:9px;font-weight:600;text-transform:uppercase;letter-spacing:1.5px;color:#5a7089;padding:8px 0 4px;";
            header.textContent = sizeLabels[sz] || sz.toUpperCase();
            modalFishList.appendChild(header);

            items.sort((a, b) => {
                if (a.discovered !== b.discovered) return a.discovered ? -1 : 1;
                return a.cfg.label.localeCompare(b.cfg.label);
            });

            items.forEach(({ key, cfg, discovered }) => {
                const row = document.createElement("div");
                row.className = `modal-fish-row${discovered ? "" : " undiscovered"}`;

                const icon = document.createElement("img");
                icon.className = "modal-fish-icon";
                icon.src = discovered ? fishIconUrl(cfg) : "";
                icon.alt = "";
                icon.onerror = function(){ this.style.display = "none"; };
                if (!discovered) icon.style.visibility = "hidden";

                const info = document.createElement("div");
                info.className = "modal-fish-info";

                const nameEl = document.createElement("span");
                nameEl.className = "modal-fish-name";
                nameEl.textContent = discovered ? cfg.label : "???";

                const meta = document.createElement("span");
                meta.className = "modal-fish-meta";
                if (discovered) {
                    let hoursText = "All day";
                    if (cfg.hours) hoursText = `${formatHour(cfg.hours[0])} – ${formatHour(cfg.hours[1])}`;
                    meta.textContent = `${formatWeight(cfg)} · ${hoursText}`;
                }

                info.appendChild(nameEl);
                info.appendChild(meta);

                const right = document.createElement("div");
                right.className = "modal-fish-right";

                if (discovered) {
                    const badge = document.createElement("span");
                    badge.className = `fish-size-badge ${cfg.size}`;
                    badge.textContent = cfg.size.toUpperCase();

                    const season = !!inSeason[key];
                    const dot = document.createElement("span");
                    dot.className = `fish-season-dot ${season ? "in" : "out"}`;

                    right.appendChild(badge);
                    right.appendChild(dot);
                }

                row.appendChild(icon);
                row.appendChild(info);
                row.appendChild(right);

                if (discovered) {
                    row.addEventListener("click", () => {
                        closeModal();
                        openLorePanel(key);
                    });
                }

                modalFishList.appendChild(row);
            });
        });

        if (fishKeys.length === 0) {
            modalFishList.innerHTML = '<div style="text-align:center;padding:20px;color:#5a7089;font-style:italic;">No fish data for this location</div>';
        }
    }

    function closeModal() {
        modalOverlay.classList.add("hidden");
        modalOpen = false;
    }

    modalCloseBtn.addEventListener("click", closeModal);
    modalOverlay.addEventListener("click", (e) => {
        if (e.target === modalOverlay) closeModal();
    });

    // ==================== AUDIO ENGINE ====================
    const speciesAudioMap = {
        bluegill:        "sfx/Bluegill.mp3",
        bullhead:        "sfx/BrownBullhead.mp3",
        chain_pickerel:  "sfx/ChainPickerel.mp3",
        channel_catfish: "sfx/ChannelCatfish.mp3",
        lake_sturgeon:   "sfx/LakeSturgeon.mp3",
        largemouth_bass: "sfx/LargenmouthBass.mp3",
        longnose_gar:    "sfx/LongnoseGar.mp3",
        muskellunge:     "sfx/Muskellunge.mp3",
        northern_pike:   "sfx/NorthernPike.mp3",
        rainbow_trout:   "sfx/RainbowTrout.mp3",
        redfin_pickerel: "sfx/RedfinPickerel.mp3",
        rock_bass:       "sfx/RockBass.mp3",
        smallmouth_bass: "sfx/Smallmouth.mp3",
        sockeye_salmon:  "sfx/SockeyeSalmon.mp3",
        steelhead_trout: "sfx/SteelheadTrout.mp3",
        perch:           "sfx/YellowPerch.mp3",
    };

    // --- Background music ---
    let bgMusic = null;
    let audioMuted = false;

    function startBgMusic() {
        if (bgMusic) return;
        bgMusic = new Audio("sfx/fishingMusic_1.mp3");
        bgMusic.loop = true;
        bgMusic.volume = 0.1;  // 10%
        if (!audioMuted) bgMusic.play().catch(() => {});
    }

    function stopBgMusic() {
        if (bgMusic) {
            bgMusic.pause();
            bgMusic.currentTime = 0;
            bgMusic = null;
        }
    }

    // --- Species voice (boosted to 300% via Web Audio API GainNode) ---
    let speciesAudio = null;
    let speciesSource = null;
    let speciesGain = null;
    let audioCtx = null;

    function getAudioCtx() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        return audioCtx;
    }

    function playSpeciesAudio(speciesId) {
        stopSpeciesAudio();
        const file = speciesId ? speciesAudioMap[speciesId] : null;
        if (!file || audioMuted) return;
        const ctx = getAudioCtx();
        speciesAudio = new Audio(file);
        speciesSource = ctx.createMediaElementSource(speciesAudio);
        speciesGain = ctx.createGain();
        speciesGain.gain.value = 3.0;  // 300%
        speciesSource.connect(speciesGain);
        speciesGain.connect(ctx.destination);
        speciesAudio.play().catch(() => {});
    }

    function stopSpeciesAudio() {
        if (speciesAudio) {
            speciesAudio.pause();
            speciesAudio.currentTime = 0;
            if (speciesSource) { speciesSource.disconnect(); speciesSource = null; }
            if (speciesGain) { speciesGain.disconnect(); speciesGain = null; }
            speciesAudio = null;
        }
    }

    // --- Mute toggle ---
    const muteBtn  = document.getElementById("muteBtn");
    const muteIcon = document.getElementById("muteIcon");

    function toggleMute() {
        audioMuted = !audioMuted;
        muteBtn.classList.toggle("muted", audioMuted);
        muteIcon.textContent = audioMuted ? "\u2716" : "\u266B";  // ✖ or ♫
        if (audioMuted) {
            if (bgMusic) bgMusic.pause();
            stopSpeciesAudio();
        } else {
            if (bgMusic) bgMusic.play().catch(() => {});
        }
    }

    if (muteBtn) muteBtn.addEventListener("click", toggleMute);

    // ==================== FISH LORE SIDEBAR ====================
    function openLorePanel(fishKey) {
        const cfg = fishConfig[fishKey];
        if (!cfg) return;

        const speciesId = cfg.species;
        const lore = speciesId ? speciesLore[speciesId] : null;

        loreFishIcon.src = fishIconUrl(cfg);
        loreFishIcon.onerror = function(){ this.style.visibility = "hidden"; };
        loreFishIcon.style.visibility = "visible";

        loreName.textContent = cfg.label;
        loreSci.textContent = lore ? lore.scientific : "";
        loreFamilyEl.textContent = lore ? lore.family : "";

        loreSizeBadge.textContent = (sizeLabels[cfg.size] || cfg.size).toUpperCase();
        loreSizeBadge.className = `fish-size-badge ${cfg.size}`;
        loreWeight.textContent = formatWeight(cfg);
        const wd = weightData[fishKey];
        if (wd && wd.best) {
            loreWeight.textContent += ` (PB: ${wd.best} lbs)`;
            if (wd.bestDate) {
                loreWeight.textContent += ` on ${wd.bestDate}`;
            }
        }
        lorePrice.textContent = `$${cfg.price.toFixed(2)}`;
        loreHours.textContent = cfg.hours ? `${formatHour(cfg.hours[0])} – ${formatHour(cfg.hours[1])}` : "All day";

        loreSummary.textContent = lore ? lore.summary : "No lore data available for this species yet.";
        loreHistory.textContent = lore ? lore.history : "—";
        loreHabitat.textContent = lore ? lore.habitat : "—";
        loreBehavior.textContent = lore ? lore.behavior : "—";
        loreFunFact.textContent = lore ? lore.funFact : "—";

        // Known locations
        loreLocations.innerHTML = "";
        waterBodies.forEach(body => {
            const fishKeys = zoneFish[body.zoneHash] || [];
            if (!fishKeys.includes(fishKey)) return;

            const zoneDisc = discoveries[body.zoneHash] || {};
            const foundHere = !!zoneDisc[fishKey];

            const tag = document.createElement("span");
            tag.className = `lore-location-tag ${foundHere ? "discovered" : "undiscovered"}`;
            tag.innerHTML = `<span class="lore-location-dot" style="background:${typeColours[body.type] || '#4a9eff'}"></span> ${foundHere ? body.label : "???"}`;

            if (foundHere) {
                tag.addEventListener("click", () => {
                    closeLorePanel();
                    selectWaterBody(body);
                });
            }
            loreLocations.appendChild(tag);
        });

        lorePanel.classList.remove("hidden");
        lorePanel.querySelector(".lore-scroll").scrollTop = 0;
        loreOpen = true;

        playSpeciesAudio(speciesId);
    }

    function closeLorePanel() {
        lorePanel.classList.add("hidden");
        loreOpen = false;
        stopSpeciesAudio();
    }

    loreCloseBtn.addEventListener("click", closeLorePanel);

    // ==================== ADD DISCOVERY ====================
    function addDiscovery(fishKey, zoneHash) {
        if (!discoveries[zoneHash]) discoveries[zoneHash] = {};
        discoveries[zoneHash][fishKey] = true;

        if (modalOpen && selectedBody && selectedBody.zoneHash === zoneHash) {
            renderModalFish(selectedBody);
        }
        renderProgress();
        if (currentTab === "discoveries") renderDiscoveries();

        // Trigger the toast for in-journal discoveries too
        const cfg2 = fishConfig[fishKey];
        const label = cfg2 ? cfg2.label : fishKey;
        const icon  = cfg2 ? fishIconUrl(cfg2) : "";
        // Find water body name
        let locName = "";
        for (const wb of waterBodies) {
            if (wb.zoneHash === zoneHash) { locName = wb.label; break; }
        }
        showDiscoveryToast(label, locName, icon);
    }

    function showDiscoveryToast(fishLabel, location, icon) {
        discoveryFishLabel.textContent = fishLabel || "Unknown Fish";
        discoveryLocation.textContent  = location ? "at " + location : "";
        if (icon) {
            discoveryIcon.src = icon;
            discoveryIcon.style.display = "";
        } else {
            discoveryIcon.style.display = "none";
        }
        discoveryToast.classList.remove("hidden", "fade-out");

        if (discoveryTimer) clearTimeout(discoveryTimer);
        discoveryTimer = setTimeout(() => {
            discoveryToast.classList.add("fade-out");
            setTimeout(() => discoveryToast.classList.add("hidden"), 500);
        }, 4000);
    }

    function showRecordToast(fishLabel, newBest, oldBest) {
        recordFishLabel.textContent = fishLabel || "Unknown Fish";
        const newW = typeof newBest === "number" ? newBest.toFixed(1) : newBest;
        const oldW = typeof oldBest === "number" ? oldBest.toFixed(1) : oldBest;
        recordWeight.textContent = newW + " lbs  (prev: " + oldW + " lbs)";
        recordToast.classList.remove("hidden", "fade-out");

        if (recordTimer) clearTimeout(recordTimer);
        recordTimer = setTimeout(() => {
            recordToast.classList.add("fade-out");
            setTimeout(() => recordToast.classList.add("hidden"), 500);
        }, 5000);
    }

    // ==================== PERSONAL BESTS ====================
    function renderPBs() {
        const pbs = [];
        Object.keys(weightData).forEach(key => {
            const wd = weightData[key];
            const cfg = fishConfig[key];
            if (!cfg || !wd || !wd.best || wd.best <= 0) return;
            pbs.push({ key, cfg, wd });
        });

        if (pbsSort === "weight") {
            pbs.sort((a, b) => b.wd.best - a.wd.best);
        } else if (pbsSort === "name") {
            pbs.sort((a, b) => a.cfg.label.localeCompare(b.cfg.label));
        } else if (pbsSort === "date") {
            pbs.sort((a, b) => (b.wd.bestDate || "").localeCompare(a.wd.bestDate || ""));
        }

        pbsListEl.innerHTML = "";
        if (pbs.length === 0) {
            pbsListEl.innerHTML = '<div class="pb-empty">No personal bests yet.<br>Catch some fish to start tracking!</div>';
            return;
        }

        pbs.forEach(rec => {
            const el = document.createElement("div");
            el.className = "pb-item";

            const icon = document.createElement("img");
            icon.className = "pb-icon";
            icon.src = fishIconUrl(rec.cfg);
            icon.alt = "";
            icon.onerror = function(){ this.style.display = "none"; };

            const info = document.createElement("div");
            info.className = "pb-info";
            const dateParts = [];
            if (rec.wd.bestDate) dateParts.push("PB set: " + rec.wd.bestDate);
            if (rec.wd.firstCaught) dateParts.push("First: " + rec.wd.firstCaught);
            if (rec.wd.lastCaught) dateParts.push("Last: " + rec.wd.lastCaught);
            info.innerHTML = `<span class="pb-name">${rec.cfg.label}</span>`
                + `<span class="pb-meta">${(sizeLabels[rec.cfg.size] || rec.cfg.size).toUpperCase()}`
                + ` · ${catchCounts[rec.key] || 0} caught · $${rec.cfg.price.toFixed(2)}</span>`
                + (dateParts.length ? `<span class="pb-meta pb-meta-date">${dateParts.join(" · ")}</span>` : "");

            const wt = document.createElement("span");
            wt.className = "pb-weight";
            wt.textContent = rec.wd.best + " lbs";

            el.appendChild(icon);
            el.appendChild(info);
            el.appendChild(wt);
            el.addEventListener("click", () => openLorePanel(rec.key));
            pbsListEl.appendChild(el);
        });
    }

    // ==================== RECORDS ====================
    function renderRecords() {
        const records = [];
        let totalCaught = 0, totalLbs = 0;

        Object.keys(catchCounts).forEach(key => {
            const count = catchCounts[key];
            const cfg   = fishConfig[key];
            if (!cfg || count <= 0) return;
            totalCaught += count;
            const wd = weightData[key];
            const totalW = wd ? (wd.total || 0) : 0;
            totalLbs += totalW;
            records.push({ key, count, cfg, wd: wd || {} });
        });

        totalCatchesEl.textContent  = totalCaught;
        uniqueSpeciesEl.textContent = records.length;
        totalWeightEl.textContent   = totalLbs % 1 === 0 ? totalLbs : totalLbs.toFixed(1);

        if (recordsSort === "count") {
            records.sort((a, b) => b.count - a.count || a.cfg.label.localeCompare(b.cfg.label));
        } else if (recordsSort === "name") {
            records.sort((a, b) => a.cfg.label.localeCompare(b.cfg.label));
        } else if (recordsSort === "weight") {
            records.sort((a, b) => (b.wd.total || 0) - (a.wd.total || 0) || b.count - a.count);
        }

        recordsListEl.innerHTML = "";
        if (records.length === 0) {
            recordsListEl.innerHTML = '<div class="records-empty">No fish caught yet.<br>Start fishing to build your record!</div>';
            return;
        }

        records.forEach((rec, i) => {
            const rank = i + 1;
            const el = document.createElement("div");
            el.className = "record-item";

            const icon = document.createElement("img");
            icon.className = "record-icon";
            icon.src = fishIconUrl(rec.cfg);
            icon.alt = "";
            icon.onerror = function(){ this.style.display = "none"; };

            const rankEl = document.createElement("span");
            rankEl.className = `record-rank${rank <= 3 ? " top-3" : ""}`;
            rankEl.textContent = rank;

            const info = document.createElement("div");
            info.className = "record-info";
            const bestStr = rec.wd.best ? ` · PB: ${rec.wd.best} lbs` : "";
            const dateStr = rec.wd.lastCaught ? ` · Last: ${rec.wd.lastCaught}` : "";
            info.innerHTML = `<span class="record-name">${rec.cfg.label}</span><span class="record-meta">${formatWeight(rec.cfg)}${bestStr}${dateStr} · <span class="fish-size-badge ${rec.cfg.size}">${rec.cfg.size.toUpperCase()}</span></span>`;

            const countEl = document.createElement("span");
            countEl.className = "record-count";
            countEl.textContent = `×${rec.count}`;

            const valEl = document.createElement("span");
            valEl.className = "record-value";
            valEl.textContent = `$${(rec.count * rec.cfg.price).toFixed(2)}`;

            el.appendChild(icon);
            el.appendChild(rankEl);
            el.appendChild(info);
            el.appendChild(countEl);
            el.appendChild(valEl);

            el.addEventListener("click", () => openLorePanel(rec.key));
            recordsListEl.appendChild(el);
        });
    }

    // ==================== MAP PAN & ZOOM ====================
    function centerMap() {
        const areaW = mapArea.clientWidth;
        const areaH = mapArea.clientHeight;
        const imgW  = mapConfig.imageWidth  || 3988;
        const imgH  = mapConfig.imageHeight || 2637;
        scale = Math.min(areaW / imgW, areaH / imgH) * 0.95;
        panX  = (areaW - imgW * scale) / 2;
        panY  = (areaH - imgH * scale) / 2;
        applyTransform();
    }

    function applyTransform() {
        mapContainer.style.transform = `translate(${panX}px, ${panY}px) scale(${scale})`;
        updatePinScale();
    }

    mapArea.addEventListener("mousedown", (e) => {
        if (e.button !== 0) return;
        dragging  = true;
        dragStart = { x: e.clientX - panX, y: e.clientY - panY };
    });

    window.addEventListener("mousemove", (e) => {
        if (!dragging) return;
        panX = e.clientX - dragStart.x;
        panY = e.clientY - dragStart.y;
        applyTransform();
    });

    window.addEventListener("mouseup", () => { dragging = false; });

    mapArea.addEventListener("wheel", (e) => {
        e.preventDefault();
        const rect   = mapArea.getBoundingClientRect();
        const mouseX = e.clientX - rect.left;
        const mouseY = e.clientY - rect.top;
        const prevScale = scale;
        const delta = e.deltaY > 0 ? 0.9 : 1.1;
        scale = Math.min(MAX_SCALE, Math.max(MIN_SCALE, scale * delta));
        panX = mouseX - (mouseX - panX) * (scale / prevScale);
        panY = mouseY - (mouseY - panY) * (scale / prevScale);
        applyTransform();
    }, { passive: false });

})();
