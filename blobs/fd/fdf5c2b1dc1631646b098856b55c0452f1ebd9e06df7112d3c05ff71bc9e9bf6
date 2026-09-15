/* =========================================================================
   poggy_fishing - app.js
   NUI bridge. Handles all state transitions, the fish window, the tug
   bars, the Pro Rod pull bar, the progress bar, the reel overlay, audio
   and result flashes.
   ========================================================================= */
(() => {
    "use strict";

    const hud          = document.getElementById("fishing-hud");
    const zoneLabel    = document.getElementById("zone-label");
    const approachView = document.getElementById("approach-view");
    const approachScene = document.getElementById("approach-scene");
    const fishMarker   = document.getElementById("fish-marker");
    const fishBody     = document.getElementById("fish-body-el");
    const alertBadge   = document.getElementById("alert-badge");
    const approachLbl  = document.getElementById("approach-label");
    const statusIcon   = document.getElementById("status-icon");
    const statusText   = document.getElementById("status-text");
    const fishInfo     = document.getElementById("fish-info");
    const fishName     = document.getElementById("fish-name");
    const fishWeight   = document.getElementById("fish-weight");
    const progContainer = document.getElementById("progress-container");
    const progFill     = document.getElementById("progress-fill");
    const progPct      = document.getElementById("progress-pct");
    const tugArea      = document.getElementById("tug-area");
    const tensionFill  = document.getElementById("tension-fill");
    const interestFill = document.getElementById("interest-fill");
    const interestPrompt = document.getElementById("interest-prompt");
    const resultFlash  = document.getElementById("result-flash");
    const ctrlCast     = document.getElementById("ctrl-cast");
    const ctrlBait     = document.getElementById("ctrl-bait");
    const ctrlStow     = document.getElementById("ctrl-stow");
    const ctrlMusic    = document.getElementById("ctrl-music");
    const baitIndicator = document.getElementById("bait-indicator");
    const baitIndImg    = document.getElementById("bait-indicator-img");
    const baitIndName   = document.getElementById("bait-indicator-name");
    const baitIndQty    = document.getElementById("bait-indicator-qty");
    const baitMenu      = document.getElementById("bait-menu");
    const baitList      = document.getElementById("bait-list");
    const baitRemove    = document.getElementById("bait-remove");
    const btnRemoveBait = document.getElementById("btn-remove-bait");
    const baitMenuClose = document.getElementById("bait-menu-close");
    const nowPlaying    = document.getElementById("now-playing");
    const npTrackName   = document.getElementById("np-track-name");

    // Fish window: hook rig and glows
    const hookRig       = document.getElementById("hook-rig");
    const hookGlow      = document.getElementById("hook-glow");
    const hookTipEl     = document.getElementById("hook-tip");
    const fishGlow      = document.getElementById("fish-glow");

    // Pull direction bar (Pro Rod)
    const pullDirection  = document.getElementById("pull-direction");
    const pullTrack      = document.getElementById("pull-track");
    const pullNeedle     = document.getElementById("pull-needle");
    const pullZoneLeft   = document.getElementById("pull-zone-left");
    const pullZoneCenter = document.getElementById("pull-zone-center");
    const pullZoneRight  = document.getElementById("pull-zone-right");
    const pullTensionFill = document.getElementById("pull-tension-fill");
    const pullDirLabels  = {
        left:    document.querySelector(".pull-dir-left"),
        forward: document.querySelector(".pull-dir-fwd"),
        right:   document.querySelector(".pull-dir-right"),
    };
    const interestBars   = document.querySelector(".interest-bars");

    // Fish flash state
    let fishFlashActive = false;
    let fishFlashStartTime = 0;

    const zoneFish     = document.getElementById("zone-fish");

    // Reel overlay
    const reelOverlay   = document.getElementById("reel-overlay");
    const reelHandle    = document.getElementById("reel-handle");
    const reelGear      = document.getElementById("reel-gear");
    let reelAngle       = 0;
    let reelSpeed       = 0;       // 0 = slow (1 rot/3s), 1 = fast (1 rot/1s)
    let reelSpinning    = false;
    let reelRAF         = null;
    let reelLastFrame   = 0;
    const REEL_SLOW_DPS = 120;     // degrees/sec at speed 0  (360/3)
    const REEL_FAST_DPS = 360;     // degrees/sec at speed 1  (360/1)
    const REEL_CLICKS   = 8;       // click sounds per full rotation

    // NUI callbacks go to this resource whatever its folder is called
    const RESOURCE = (typeof GetParentResourceName === "function") ? GetParentResourceName() : "poggy_fishing";
    function nuiPost(name, body) {
        fetch("https://" + RESOURCE + "/" + name, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(body || {})
        });
    }

    let currentBaitKey = null;
    let flashTimeout = null;
    let currentZoneFishKeys = [];   // fish keys in current zone
    let currentBaitTargets  = {};   // baitKey -> [fishKey,...] lookup

    // Footer time-of-day
    const footerTimeIcon  = document.getElementById("footer-time-icon");
    const footerTimeLabel = document.getElementById("footer-time-label");
    const footerTimeClock = document.getElementById("footer-time-clock");
    let lastZoneFishRaw   = [];     // raw fishList from Lua (with available flag)

    // Item icon URL prefix from poggy_core (inv.imageBase), set by index.html
    function baitImgUrl(itemName) {
        return (window.POGGY_IMG_BASE || "") + encodeURIComponent(itemName) + ".png";
    }

    // ---- ZONE FISH GRID ----
    function buildZoneFish(fishList) {
        // fishList = [ {key, label, item, available, discovered}, ... ]
        if (!zoneFish) return;
        zoneFish.innerHTML = "";
        currentZoneFishKeys = [];
        lastZoneFishRaw = fishList || [];
        if (!fishList || fishList.length === 0) { hide(zoneFish); return; }
        // De-duplicate by item; skip unavailable fish entirely
        const seen = new Set();
        fishList.forEach(f => {
            if (f.available === false) return;
            if (seen.has(f.item)) return;
            seen.add(f.item);
            currentZoneFishKeys.push(f.key);
            const el = document.createElement("div");
            const isHidden = f.discovered === false;
            el.className = "zone-fish-icon" + (isHidden ? " undiscovered" : "");
            el.dataset.fishKey = f.key;
            if (isHidden) {
                el.innerHTML =
                    '<span class="zf-unknown">?</span>' +
                    '<span class="zf-tooltip">???</span>';
            } else {
                el.innerHTML =
                    '<img src="' + baitImgUrl(f.item) + '" alt="">' +
                    '<span class="zf-tooltip">' + f.label + '</span>';
            }
            zoneFish.appendChild(el);
        });
        // Skins with a fixed row size the slots to the count (the brass skin)
        zoneFish.style.setProperty("--n", Math.max(1, currentZoneFishKeys.length));
        show(zoneFish);
    }

    function revealFish(fishKey, item, label) {
        if (!zoneFish) return;
        const el = zoneFish.querySelector('.zone-fish-icon[data-fish-key="' + fishKey + '"]');
        if (!el) return;
        el.classList.remove("undiscovered");
        el.innerHTML =
            '<img src="' + baitImgUrl(item) + '" alt="">' +
            '<span class="zf-tooltip">' + label + '</span>';
    }

    function highlightTargetFish(fishKeys) {
        if (!zoneFish) return;
        const set = new Set(fishKeys || []);
        zoneFish.querySelectorAll(".zone-fish-icon").forEach(el => {
            el.classList.toggle("bait-highlight", set.has(el.dataset.fishKey));
        });
    }

    function clearFishHighlights() {
        if (!zoneFish) return;
        zoneFish.querySelectorAll(".zone-fish-icon.bait-highlight").forEach(el => {
            el.classList.remove("bait-highlight");
        });
    }

    // ---- PRO FIGHT BAR LABELS ----
    const interestLabel = document.querySelector(".ibar-interest .ibar-label");
    const interestIcon  = document.querySelector(".ibar-interest .ibar-icon");
    const origInterestLabel = interestLabel ? interestLabel.textContent : "Interest";
    const origInterestIcon  = interestIcon  ? interestIcon.innerHTML    : "\u{1F41F}";

    function setProFightLabels(on) {
        if (!interestLabel || !interestIcon) return;
        if (on) {
            interestLabel.textContent = "Reeling";
            interestIcon.innerHTML = '\u{1F3A3}';
        } else {
            interestLabel.textContent = origInterestLabel;
            interestIcon.innerHTML = origInterestIcon;
        }
    }

    // ---- WATER FILLS ----
    // Every bar's fill is the full track width, clipped to the value, so the
    // wave texture spans the whole bar (see .water-fill in style.css).
    function setFill(el, pct) {
        if (!el) return;
        const v = Math.max(0, Math.min(100, pct));
        el.style.setProperty("--cut", (100 - v).toFixed(1) + "%");
    }

    // ---- DIRECTION PULL BAR (Pro Rod) ----
    let pullModeActive = false;
    let pullLastRgb = "";
    let pullLastW   = -1;
    let pullLastDir = "";

    function showPullMode(on) {
        pullModeActive = !!on;
        if (on) {
            if (interestBars) interestBars.classList.add("hidden");
            if (pullDirection) pullDirection.classList.remove("hidden");
            resetPullBar();
        } else {
            if (pullDirection) pullDirection.classList.add("hidden");
            if (interestBars) interestBars.classList.remove("hidden");
        }
    }

    function resetPullBar() {
        pullLastRgb = ""; pullLastW = -1; pullLastDir = "";
        if (pullNeedle) pullNeedle.style.setProperty("--needle-x", "0px");
        if (pullDirection) {
            pullDirection.style.setProperty("--pull-rgb", "80, 220, 160");
            pullDirection.style.setProperty("--pull-w", "1");
        }
        setFill(pullTensionFill, 0);
        if (pullTensionFill) pullTensionFill.classList.remove("danger");
        [pullZoneLeft, pullZoneCenter, pullZoneRight].forEach(z => {
            if (z) z.classList.remove("target");
        });
        Object.values(pullDirLabels).forEach(l => {
            if (l) l.classList.remove("active");
        });
    }

    function updatePullBar(position, direction, correct, tension, wrongness) {
        // position: -100 (full left) to 100 (full right)
        // direction: "left", "right", "forward"
        // tension: 0-100
        // wrongness: 0 (perfect) to 1 (max wrong); drives the green → red colour
        //
        // Colour and zone alpha are CSS variables on #pull-direction, written
        // only when the quantised value changes.  The labels change by class
        // alone, so nothing about the text animates while the needle moves.
        const w = Math.max(0, Math.min(1, wrongness));
        const wq = Math.round(w * 20) / 20;
        if (wq !== pullLastW) {
            pullLastW = wq;
            const r = Math.round(80 + (255 - 80) * wq);
            const g = Math.round(220 + (80 - 220) * wq);
            const b = Math.round(160 + (50 - 160) * wq);
            const rgb = r + ", " + g + ", " + b;
            if (rgb !== pullLastRgb && pullDirection) {
                pullLastRgb = rgb;
                pullDirection.style.setProperty("--pull-rgb", rgb);
            }
            if (pullDirection) pullDirection.style.setProperty("--pull-w", wq.toFixed(2));
        }

        // Needle: -100..100 → 0..track width, moved with a transform
        if (pullNeedle && pullTrack) {
            const pct = 50 + (position / 2);
            const x = (pct / 100) * pullTrack.clientWidth;
            pullNeedle.style.setProperty("--needle-x", x.toFixed(1) + "px");
        }

        // Target zone and label follow the fish's direction
        if (direction !== pullLastDir) {
            pullLastDir = direction;
            const zones = { left: pullZoneLeft, center: pullZoneCenter, right: pullZoneRight };
            const zoneKey = direction === "forward" ? "center" : direction;
            [pullZoneLeft, pullZoneCenter, pullZoneRight].forEach(z => {
                if (z) z.classList.remove("target");
            });
            if (zones[zoneKey]) zones[zoneKey].classList.add("target");
            Object.entries(pullDirLabels).forEach(([key, el]) => {
                if (el) el.classList.toggle("active", key === direction);
            });
        }

        // Tension bar
        const t = Math.max(0, Math.min(100, tension));
        setFill(pullTensionFill, t);
        if (pullTensionFill) pullTensionFill.classList.toggle("danger", t >= 75);
    }

    function updateBaitIndicator(baitKey, label, item, qty) {
        if (baitKey) {
            currentBaitKey = baitKey;
            baitIndImg.src = baitImgUrl(item);
            baitIndName.textContent = label;
            baitIndQty.textContent = qty != null ? ("x" + qty) : "";
            show(baitIndicator);
        } else {
            currentBaitKey = null;
            hide(baitIndicator);
        }
    }

    function buildBaitMenu(baits, activeBaitKey) {
        baitList.innerHTML = "";
        baitList.scrollTop = 0;   // every open starts at the top of the list
        currentBaitTargets = {};
        if (!baits || baits.length === 0) {
            baitList.innerHTML = '<div class="bait-empty">No bait in inventory</div>';
            hide(baitRemove);
            clearFishHighlights();
            return;
        }
        baits.forEach(b => {
            if (b.targetFish && b.targetFish.length > 0) {
                currentBaitTargets[b.key] = b.targetFish;
            }
            const row = document.createElement("div");
            row.className = "bait-item" + (b.key === activeBaitKey ? " active" : "");
            row.innerHTML =
                '<img class="bait-item-img" src="' + baitImgUrl(b.item) + '" alt="">' +
                '<div class="bait-item-info">' +
                    '<span class="bait-item-name">' + b.label + '</span>' +
                    (b.key === activeBaitKey ? '<span class="bait-item-active-tag">Equipped</span>' :
                        '<span class="bait-item-desc">' + (b.loseChance > 0 ? b.loseChance + '% lose chance' : 'Durable') + '</span>') +
                '</div>' +
                '<span class="bait-item-qty">x' + b.qty + '</span>';
            row.addEventListener("mouseenter", () => {
                if (currentBaitTargets[b.key]) {
                    highlightTargetFish(currentBaitTargets[b.key]);
                } else {
                    clearFishHighlights();
                }
            });
            row.addEventListener("mouseleave", clearFishHighlights);
            row.addEventListener("click", () => {
                clearFishHighlights();
                if (zoneFish) zoneFish.classList.remove("selecting");
                nuiPost("selectBait", { key: b.key, qty: b.qty });
                hide(baitMenu);
            });
            baitList.appendChild(row);
        });
        if (activeBaitKey) {
            show(baitRemove);
        } else {
            hide(baitRemove);
        }
    }

    // A long bait list fades out at whichever edge has more to scroll to
    // (skins style .fade-top / .fade-bottom); a list that fits gets neither.
    function updateBaitFade() {
        if (!baitList) return;
        const room = baitList.scrollHeight - baitList.clientHeight;
        baitList.classList.toggle("fade-top", room > 1 && baitList.scrollTop > 1);
        baitList.classList.toggle("fade-bottom", room > 1 && baitList.scrollTop < room - 1);
    }

    function closeBaitMenu() {
        hide(baitMenu);
        clearFishHighlights();
        if (zoneFish) zoneFish.classList.remove("selecting");
        nuiPost("closeBaitMenu");
    }

    // ---- HELPERS ----
    function show(el) { if (el) el.classList.remove("hidden"); }
    function hide(el) { if (el) el.classList.add("hidden"); }

    function setStatus(icon, text, cls) {
        statusIcon.textContent = icon;
        statusText.textContent = text;
        statusText.className = cls || "";
    }

    function setProgress(pct) {
        const v = Math.max(0, Math.min(100, pct));
        setFill(progFill, v);
        progPct.textContent = Math.round(v) + "%";
        progFill.classList.toggle("danger", v < 25);
        progFill.classList.toggle("complete", v >= 90);
    }

    // ---- REEL CONTINUOUS SPIN ----
    let reelClickSource = null;

    function playReelClick() {
        try {
            const ctx = getAudioCtx();
            const buffer = audioBufferCache["sfx/reelclick.mp3"];
            if (!buffer) {
                loadAudioBuffer("sfx/reelclick.mp3");
                return;
            }
            if (reelClickSource) {
                try { reelClickSource.stop(); } catch (_) {}
            }
            const source = ctx.createBufferSource();
            source.buffer = buffer;
            const gain = ctx.createGain();
            gain.gain.value = 0.3;
            source.connect(gain);
            gain.connect(ctx.destination);
            source.start(0);
            reelClickSource = source;
            source.addEventListener("ended", () => {
                if (reelClickSource === source) reelClickSource = null;
            });
        } catch (_) {}
    }

    function stopReelClick() {
        if (reelClickSource) {
            try { reelClickSource.stop(); } catch (_) {}
            reelClickSource = null;
        }
    }

    function reelFrame(ts) {
        if (!reelSpinning) return;
        if (!reelLastFrame) { reelLastFrame = ts; reelRAF = requestAnimationFrame(reelFrame); return; }
        const dt = (ts - reelLastFrame) / 1000;
        reelLastFrame = ts;
        const dps = REEL_SLOW_DPS + reelSpeed * (REEL_FAST_DPS - REEL_SLOW_DPS);
        reelAngle += dps * dt;
        if (reelHandle) reelHandle.style.transform = "rotate(" + reelAngle + "deg)";
        if (reelGear)   reelGear.style.transform   = "rotate(" + (-reelAngle) + "deg)";
        const clickEvery = 360 / REEL_CLICKS;
        const prevClick = Math.floor((reelAngle - dps * dt) / clickEvery);
        const curClick  = Math.floor(reelAngle / clickEvery);
        if (curClick > prevClick) playReelClick();
        reelRAF = requestAnimationFrame(reelFrame);
    }

    function startReelSpin() {
        if (reelSpinning) return;
        reelSpinning = true;
        reelLastFrame = 0;
        if (!audioBufferCache["sfx/reelclick.mp3"]) loadAudioBuffer("sfx/reelclick.mp3");
        reelRAF = requestAnimationFrame(reelFrame);
    }

    function stopReelSpin() {
        reelSpinning = false;
        if (reelRAF) { cancelAnimationFrame(reelRAF); reelRAF = null; }
        stopReelClick();
    }

    function setReelSpeed(s) {
        reelSpeed = Math.max(0, Math.min(1, s));
    }

    function showReel(visible) {
        if (!reelOverlay) return;
        if (visible) {
            reelOverlay.classList.remove("hidden");
            requestAnimationFrame(() => reelOverlay.classList.add("visible"));
        } else {
            reelOverlay.classList.remove("visible");
            setTimeout(() => { if (!reelOverlay.classList.contains("visible")) reelOverlay.classList.add("hidden"); }, 400);
        }
    }

    function resetReel() {
        stopReelSpin();
        reelAngle = 0;
        reelSpeed = 0;
        if (reelHandle) reelHandle.style.transform = "rotate(0deg)";
        if (reelGear)   reelGear.style.transform   = "rotate(0deg)";
    }

    // ---- REELING SPLASH AMBIENCE ----
    // Two tracks that crossfade randomly during the fight
    const REELING_SPLASH_FILES = ["sfx/reeling_splash_1.mp3", "sfx/reeling_splash_2.mp3"];
    const REELING_CROSSFADE    = 2.0;  // seconds
    const REELING_VOLUME       = 0.22;
    let reelingSplashActive    = false;
    let reelingSplashSlot      = 0;
    let reelingSplashNodes     = [null, null];
    let reelingSplashTimer     = null;

    async function playReelingSplashSlot(slot, fadeIn) {
        const ctx = getAudioCtx();
        const file = REELING_SPLASH_FILES[slot];
        const buffer = await loadAudioBuffer(file);
        if (!buffer || !reelingSplashActive) return;
        const source = ctx.createBufferSource();
        source.buffer = buffer;
        source.loop = true;
        source.playbackRate.value = 0.9 + Math.random() * 0.2;
        const gain = ctx.createGain();
        gain.gain.setValueAtTime(fadeIn ? 0.001 : REELING_VOLUME, ctx.currentTime);
        if (fadeIn) {
            gain.gain.linearRampToValueAtTime(REELING_VOLUME, ctx.currentTime + REELING_CROSSFADE);
        }
        source.connect(gain);
        gain.connect(ctx.destination);
        source.start(0);
        reelingSplashNodes[slot] = { source, gain };
    }

    function crossfadeReelingSplash() {
        if (!reelingSplashActive) return;
        const ctx = getAudioCtx();
        const now = ctx.currentTime;
        const cur = reelingSplashNodes[reelingSplashSlot];
        if (cur) {
            cur.gain.gain.linearRampToValueAtTime(0.001, now + REELING_CROSSFADE);
            const oldSource = cur.source;
            setTimeout(() => { try { oldSource.stop(); } catch(_){} }, REELING_CROSSFADE * 1000 + 100);
            reelingSplashNodes[reelingSplashSlot] = null;
        }
        reelingSplashSlot = reelingSplashSlot === 0 ? 1 : 0;
        playReelingSplashSlot(reelingSplashSlot, true);
        const next = 4000 + Math.random() * 4000;
        reelingSplashTimer = setTimeout(() => crossfadeReelingSplash(), next);
    }

    function startReelingSplash() {
        if (reelingSplashActive) return;
        reelingSplashActive = true;
        reelingSplashSlot = Math.random() < 0.5 ? 0 : 1;
        playReelingSplashSlot(reelingSplashSlot, false);
        const first = 4000 + Math.random() * 4000;
        reelingSplashTimer = setTimeout(() => crossfadeReelingSplash(), first);
    }

    function stopReelingSplash() {
        reelingSplashActive = false;
        if (reelingSplashTimer) { clearTimeout(reelingSplashTimer); reelingSplashTimer = null; }
        const ctx = audioCtx;
        for (let i = 0; i < 2; i++) {
            const node = reelingSplashNodes[i];
            if (node) {
                if (ctx) {
                    node.gain.gain.linearRampToValueAtTime(0.001, ctx.currentTime + 0.5);
                    const s = node.source;
                    setTimeout(() => { try { s.stop(); } catch(_){} }, 600);
                } else {
                    try { node.source.stop(); } catch(_){}
                }
                reelingSplashNodes[i] = null;
            }
        }
    }

    // =====================================================================
    //  FISH WINDOW
    //  The hook hangs from the surface at HOOK_X.  The fish's anchor is its
    //  mouth.  There is no fish until the line is in the water: on "waiting"
    //  or "proWaiting" the client sends biteIn (ms), and the fish swims in
    //  from off the right edge over exactly that time.  With the Pro Rod it
    //  reaches the hook as the bite lands; with the normal rod it reaches
    //  the spot where the interest game picks it up, and interest moves it
    //  from there.  On the bite it lunges onto the hook.
    // =====================================================================
    const HOOK_X       = 34;    // % from the left of the scene
    const HOOK_DEPTH   = 30;    // px below the surface
    const FISH_ENTER_X = 112;   // % : off the right edge, where it swims in from
    const FISH_FAR_X   = 96;    // % : far end of the interest range
    const FISH_BITE_X  = HOOK_X + 1.5;  // mouth on the hook's bend
    const FISH_FAR_Y   = 62;    // % : deep and far
    const FISH_HOOK_Y  = 70;    // % : level with the hook's bend

    let swimRAF = null;

    function setFishPos(xPct, yPct) {
        if (!fishMarker) return;
        fishMarker.style.setProperty("--fish-x", xPct.toFixed(1) + "%");
        fishMarker.style.setProperty("--fish-y", yPct.toFixed(1) + "%");
    }

    function setHookRig() {
        if (!hookRig) return;
        hookRig.style.setProperty("--hook-x", HOOK_X + "%");
        hookRig.style.setProperty("--hook-depth", HOOK_DEPTH + "px");
        const splash = document.getElementById("bite-splash");
        if (splash) {
            splash.style.setProperty("--hook-x", HOOK_X + "%");
            splash.style.setProperty("--hook-depth", HOOK_DEPTH + "px");
        }
    }

    // Where interest i (0-100) puts the fish: closer to the hook as it rises
    function interestPos(i) {
        const k = Math.max(0, Math.min(100, i)) / 100;
        const eased = k * k * (3 - 2 * k);   // smoothstep: hesitant at first, committed at the end
        return [FISH_FAR_X + (FISH_BITE_X - FISH_FAR_X) * eased,
                FISH_FAR_Y + (FISH_HOOK_Y - FISH_FAR_Y) * eased];
    }

    function showFish() { if (fishMarker) fishMarker.classList.remove("away"); }

    function hideFish() {
        stopSwim();
        if (fishMarker) fishMarker.classList.add("away");
    }

    function stopSwim() {
        if (swimRAF) { cancelAnimationFrame(swimRAF); swimRAF = null; }
        if (fishMarker) fishMarker.classList.remove("swimming");
    }

    // Fish closes on the hook as interest rises, backs away as it falls
    function setFishInterest(i) {
        stopSwim();
        showFish();
        const [x, y] = interestPos(i);
        setFishPos(x, y);
    }

    // Smooth 0..1 over [a, b] with zero speed at both ends
    function ease(t, a, b) {
        const k = Math.max(0, Math.min(1, (t - a) / (b - a)));
        return k * k * (3 - 2 * k);
    }

    // The fish swims in from off the right edge to (toX, toY) over ms.
    // It comes into view quickly, noses about in the middle of the time,
    // then commits for the last stretch, pausing briefly between each, so
    // it arrives exactly when the time is up.  toHook: the Pro Rod, where
    // arriving IS the bite, so the label and glow build as it closes.
    function swimTo(toX, toY, ms, toHook) {
        stopSwim();
        if (!fishMarker) return;
        const fromX = FISH_ENTER_X, fromY = FISH_FAR_Y;
        const dur   = Math.max(1500, ms || 0);
        const phase = Math.random() * Math.PI * 2;
        fishMarker.classList.add("swimming");
        setFishPos(fromX, fromY);
        showFish();
        const start = performance.now();
        const tick = (now) => {
            const el = now - start;
            const t  = Math.min(1, el / dur);
            const p  = 0.32 * ease(t, 0, 0.18)        // swims into view
                     + 0.23 * ease(t, 0.22, 0.72)     // noses about
                     + 0.45 * ease(t, 0.76, 1);       // commits
            const bob = Math.sin(el / 900 + phase) * 6 * (1 - p);
            setFishPos(fromX + (toX - fromX) * p, fromY + (toY - fromY) * p + bob);
            if (fishBody) fishBody.classList.toggle("excited", t > 0.76);
            if (toHook) {
                setGlowClass(fishGlow, "interest", Math.round(p * 70));
                approachLbl.textContent = t < 0.22 ? "Watching the water..."
                                        : t < 0.76 ? "Something stirs nearby..."
                                        : "It's circling the bait!";
            }
            if (t < 1) {
                swimRAF = requestAnimationFrame(tick);
            } else {
                swimRAF = null;
                fishMarker.classList.remove("swimming");
            }
        };
        swimRAF = requestAnimationFrame(tick);
    }

    // Bite: fish snaps onto the hook, line jerks, water bursts
    function playBite() {
        stopSwim();
        showFish();
        if (approachScene) {
            approachScene.classList.remove("bite");
            void approachScene.offsetWidth;   // restart the CSS animations
            approachScene.classList.add("bite");
        }
        setFishPos(FISH_BITE_X, FISH_HOOK_Y);
        if (fishBody) {
            fishBody.classList.remove("excited");
            fishBody.classList.add("frenzied");
        }
        setGlowClass(fishGlow, "interest", 100);
        show(alertBadge);
        approachLbl.textContent = "HOOKED!";
    }

    // ---- GLOW HELPERS ----
    function setGlowClass(el, prefix, value) {
        if (!el) return;
        el.classList.remove(prefix + "-low", prefix + "-mid", prefix + "-high");
        if (value >= 65) el.classList.add(prefix + "-high");
        else if (value >= 30) el.classList.add(prefix + "-mid");
        else if (value >= 10) el.classList.add(prefix + "-low");
    }

    function showFlash(text, cls) {
        resultFlash.textContent = text;
        resultFlash.className = "show " + cls;
        if (flashTimeout) clearTimeout(flashTimeout);
        flashTimeout = setTimeout(() => {
            resultFlash.className = "hidden";
        }, 1000);
    }

    function updateInterestBars(interest, tension) {
        const i = Math.max(0, Math.min(100, interest));
        const t = Math.max(0, Math.min(100, tension));

        setFill(interestFill, i);
        setFill(tensionFill, t);
        interestFill.classList.toggle("high", i >= 70);
        tensionFill.classList.toggle("danger", t >= 75);

        // Pulse the E prompt when bars are active
        interestPrompt.classList.toggle("pulse", i > 0 || t > 0);

        // ---- FISH: closes on the hook with interest ----
        setFishInterest(i);

        // ---- LINE AND HOOK: tension shows on the rig ----
        setGlowClass(hookRig, "tension", t);
        setGlowClass(hookGlow, "tension", t);
        if (hookTipEl) hookTipEl.classList.toggle("tension-glow", t >= 30);

        // ---- FISH GLOW (interest-based) ----
        setGlowClass(fishGlow, "interest", i);

        // ---- FISH ANIMATION STATE ----
        if (fishBody) {
            fishBody.classList.remove("excited", "frenzied");
            if (i >= 70) {
                fishBody.classList.add("frenzied");
            } else if (i >= 35) {
                fishBody.classList.add("excited");
            }

            // Blue flash once interest is high, quickening as it stays there
            if (i >= 60) {
                if (!fishFlashActive) {
                    fishFlashActive = true;
                    fishFlashStartTime = Date.now();
                }
                const elapsed = (Date.now() - fishFlashStartTime) / 1000;
                const speed = Math.max(0.18, 0.6 - elapsed * 0.08);
                fishBody.style.setProperty("--flash-speed", speed.toFixed(2) + "s");
                fishBody.classList.add("flash-blue");
            } else {
                fishBody.classList.remove("flash-blue");
                fishFlashActive = false;
            }
        }

        // ---- LABEL ----
        if (i < 15) {
            approachLbl.textContent = "Watching the water...";
        } else if (i < 35) {
            approachLbl.textContent = "Something stirs nearby...";
        } else if (i < 55) {
            approachLbl.textContent = "A fish is approaching!";
        } else if (i < 75) {
            approachLbl.textContent = "It's circling the bait!";
        } else if (i < 90) {
            approachLbl.textContent = "Almost hooked!";
        } else {
            approachLbl.textContent = "NOW!";
        }

        if (i >= 80) show(alertBadge); else hide(alertBadge);

        // Ripples quicken with interest
        const ripples = fishMarker.querySelectorAll(".fish-ripple");
        ripples.forEach(r => {
            r.style.animationDuration = (i > 50 ? "1.2s" : i > 25 ? "1.8s" : "2.5s");
        });
    }

    function resetInterestBars() {
        setFill(tensionFill, 0);
        setFill(interestFill, 0);
        if (tensionFill)  tensionFill.classList.remove("danger");
        if (interestFill) interestFill.classList.remove("high");
        if (interestPrompt) interestPrompt.classList.remove("pulse");
        if (fishBody) {
            fishBody.classList.remove("flash-blue", "excited", "frenzied");
            fishFlashActive = false;
        }
        if (hookRig)   hookRig.className = "hook-rig";
        if (hookGlow)  hookGlow.className = "hook-glow";
        if (hookTipEl) hookTipEl.classList.remove("tension-glow");
        if (fishGlow)  fishGlow.className = "fish-glow";
    }

    // Kept for the optional "approach" message (a plain 0-100 approach value)
    function setFishApproach(pct) {
        setFishInterest(pct);
        if (pct < 20) {
            approachLbl.textContent = "Watching the water...";
        } else if (pct < 50) {
            approachLbl.textContent = "Something stirs nearby...";
        } else if (pct < 75) {
            approachLbl.textContent = "A fish is approaching!";
        } else if (pct < 90) {
            approachLbl.textContent = "It's circling the bait!";
        } else {
            approachLbl.textContent = "Almost there...";
        }
        if (pct >= 85) show(alertBadge); else hide(alertBadge);
        if (fishBody) {
            fishBody.classList.remove("excited", "frenzied");
            if (pct >= 70) fishBody.classList.add("frenzied");
            else if (pct >= 35) fishBody.classList.add("excited");
        }
        setGlowClass(fishGlow, "interest", pct);
        const ripples = fishMarker.querySelectorAll(".fish-ripple");
        ripples.forEach(r => {
            r.style.animationDuration = (pct > 50 ? "1.5s" : "2.5s");
        });
    }

    function resetApproach() {
        hideFish();                       // no fish until the line is in the water
        if (approachScene) approachScene.classList.remove("bite");
        setHookRig();
        setFishPos(FISH_ENTER_X, FISH_FAR_Y);
        hide(alertBadge);
        approachLbl.textContent = "Watching the water...";
    }

    function resetAll() {
        hide(approachView);
        hide(fishInfo);
        hide(progContainer);
        hide(tugArea);
        hide(alertBadge);
        hide(resultFlash);
        hide(baitMenu);
        clearFishHighlights();
        setProFightLabels(false);
        showPullMode(false);
        setProgress(0);
        resetApproach();
        resetInterestBars();
        resetReel();
        statusText.className = "";
        show(ctrlCast);
        show(ctrlBait);
        show(ctrlStow);
        show(ctrlMusic);
    }

    // ---- FOOTER TIME-OF-DAY ----
    function updateFooterTime(hour, minute) {
        if (!footerTimeIcon || !footerTimeLabel || !footerTimeClock) return;
        var h12 = hour % 12 || 12;
        var ampm = hour < 12 ? "AM" : "PM";
        var mm = (minute < 10 ? "0" : "") + minute;
        footerTimeClock.textContent = h12 + ":" + mm + " " + ampm;
        if (hour >= 5 && hour < 7)        { footerTimeIcon.textContent = "🌅"; footerTimeLabel.textContent = "DAWN"; }
        else if (hour >= 7 && hour < 12)  { footerTimeIcon.textContent = "☀️"; footerTimeLabel.textContent = "MORNING"; }
        else if (hour >= 12 && hour < 17) { footerTimeIcon.textContent = "☀️"; footerTimeLabel.textContent = "AFTERNOON"; }
        else if (hour >= 17 && hour < 20) { footerTimeIcon.textContent = "🌇"; footerTimeLabel.textContent = "DUSK"; }
        else                              { footerTimeIcon.textContent = "🌙"; footerTimeLabel.textContent = "NIGHTFALL"; }
    }

    // ---- NUI HANDLER ----
    window.addEventListener("message", (event) => {
        const d = event.data;
        if (!d || !d.action) return;

        switch (d.action) {

            case "show":
                show(hud);
                document.documentElement.dataset.state = "idle";
                zoneLabel.textContent = d.zone || "";
                buildZoneFish(d.fish || []);
                resetAll();
                setStatus("🎣", "Ready to fish", "");
                show(nowPlaying);
                showReel(true);
                if (d.hour != null) updateFooterTime(d.hour, d.minute || 0);
                break;

            case "close":
                hide(hud);
                hide(resultFlash);
                hide(nowPlaying);
                stopSwim();
                stopReelSpin();
                showReel(false);
                stopAllSounds();
                break;

            case "setState":
                handleState(d);
                break;

            case "approach":
                setFishApproach(d.progress || 0);
                break;

            case "interestUpdate":
            case "tug":
                updateInterestBars(d.interest || 0, d.tension || 0);
                break;

            case "pullUpdate":
                updatePullBar(d.position || 0, d.direction || "forward", !!d.correct, d.tension || 0, d.wrongness != null ? d.wrongness : 1);
                setReelSpeed(1 - (d.wrongness != null ? d.wrongness : 1));
                break;

            case "updateProgress":
                setProgress(d.progress);
                if (d.result === "fail") {
                    hud.classList.add("shake");
                    setTimeout(() => hud.classList.remove("shake"), 300);
                }
                break;

            case "showBaitMenu":
                buildBaitMenu(d.baits || [], d.activeBait || null);
                show(baitMenu);
                updateBaitFade();   // reading the list's size lays it out, so no frame is shown stale
                if (zoneFish) zoneFish.classList.add("selecting");
                break;

            case "closeBaitMenu":
                hide(baitMenu);
                clearFishHighlights();
                if (zoneFish) zoneFish.classList.remove("selecting");
                break;

            case "updateBait":
                updateBaitIndicator(d.baitKey, d.label, d.item, d.qty);
                break;

            case "clearBait":
                updateBaitIndicator(null);
                break;

            case "timeUpdate":
                if (d.hour != null) updateFooterTime(d.hour, d.minute || 0);
                if (d.fish) buildZoneFish(d.fish);
                break;

            case "revealFish":
                revealFish(d.fishKey, d.item, d.label);
                break;
        }
    });

    function handleState(d) {
        // Skins can style by state (the brass skin keeps the window on show while idle)
        document.documentElement.dataset.state = d.state || "";
        switch (d.state) {

            case "idle":
                resetAll();
                setStatus("🎣", "Ready to fish", "");
                zoneLabel.textContent = d.zone || zoneLabel.textContent;
                break;

            case "casting":
                resetAll();
                setStatus("🌊", "Casting...", "");
                hide(ctrlCast);
                break;

            case "waiting":
                if (d.keepFish) {
                    // interest game starting: the fish has already swum in, leave it
                    stopSwim();
                    show(approachView);
                    show(tugArea);
                    hide(ctrlCast);
                    setStatus("🌊", "Line in the water...", "");
                    break;
                }
                resetAll();
                show(approachView);
                show(tugArea);
                hide(ctrlCast);
                setStatus("🌊", "Line in the water...", "");
                resetApproach();
                resetInterestBars();
                if (d.biteIn != null) {
                    const [x, y] = interestPos(d.approachTo != null ? d.approachTo : 20);
                    swimTo(x, y, d.biteIn, false);
                }
                break;

            case "proWaiting":
                resetAll();
                show(approachView);
                hide(tugArea);
                hide(ctrlCast);
                setStatus("🌊", "Line in the water...", "");
                resetApproach();
                if (d.biteIn != null) swimTo(FISH_BITE_X, FISH_HOOK_Y, d.biteIn, true);
                break;

            case "scared":
                hide(approachView);
                hide(tugArea);
                stopSwim();
                resetInterestBars();
                setStatus("💨", "Fish spooked!", "scared");
                break;

            case "bite":
                hide(tugArea);
                setProFightLabels(false);
                playBite();
                setStatus("❗", "Something's on the line!", "bite");
                statusText.classList.add("status-pulse");
                break;

            case "fighting":
                hide(approachView);
                hide(tugArea);
                stopSwim();
                statusText.classList.remove("status-pulse");
                setStatus("💪", "Fight!", "fighting");
                show(progContainer);
                hide(ctrlCast);
                setReelSpeed(0.5);
                startReelSpin();
                if (d.fishName) {
                    fishName.textContent = d.fishName;
                    fishWeight.textContent = d.weight ? (d.weight + " lbs") : "";
                    show(fishInfo);
                }
                setProgress(d.progress || 0);
                break;

            case "proFighting":
                hide(approachView);
                stopSwim();
                statusText.classList.remove("status-pulse");
                setStatus("💪", "Counter the pull!", "fighting");
                show(progContainer);
                show(tugArea);
                hide(ctrlCast);
                showPullMode(true);
                setReelSpeed(0);
                startReelSpin();
                setProgress(d.progress || 0);
                if (d.fishName) {
                    fishName.textContent = d.fishName;
                    fishWeight.textContent = d.weight ? (d.weight + " lbs") : "";
                    show(fishInfo);
                }
                break;

            case "caught":
                stopReelSpin();
                setStatus("⭐", "Fish Caught!", "caught");
                show(fishInfo);
                if (d.fishName) {
                    fishName.textContent = d.fishName;
                    fishWeight.textContent = d.weight ? (d.weight + " lbs") : "";
                }
                setProgress(100);
                hide(ctrlCast);
                break;

            case "keepChoice":
                hide(progContainer);
                hide(tugArea);
                hide(approachView);
                show(fishInfo);
                if (d.fishName) {
                    fishName.textContent = d.fishName;
                    fishWeight.textContent = d.weight ? (d.weight + " lbs") : "";
                }
                statusIcon.textContent = "⭐";
                statusText.innerHTML = "<kbd>E</kbd> Keep &nbsp;│&nbsp; <kbd>Bksp</kbd> Throw Back";
                statusText.className = "caught";
                hide(ctrlCast);
                hide(ctrlBait);
                hide(ctrlStow);
                hide(ctrlMusic);
                break;

            case "escaped":
                stopReelSpin();
                setStatus("💨", "The fish escaped!", "escaped");
                hide(progContainer);
                break;

            case "lineBreak":
                stopReelSpin();
                setStatus("💥", "Line snapped!", "lineBreak");
                setProgress(0);
                break;
        }
    }

    // NUI callback: escape key
    document.addEventListener("keydown", (e) => {
        if (e.key === "Escape") {
            if (!baitMenu.classList.contains("hidden")) {
                closeBaitMenu();
                return;
            }
            nuiPost("close");
        }
    });

    baitMenuClose.addEventListener("click", closeBaitMenu);
    if (baitList) baitList.addEventListener("scroll", updateBaitFade, { passive: true });

    btnRemoveBait.addEventListener("click", () => {
        nuiPost("removeBait");
        hide(baitMenu);
    });

    // ---- AUDIO ENGINE (Web Audio API — bypasses CEF autoplay policy) ----
    const audioInstances = {};  // id -> { source, gainNode, buffer, loop }
    let audioCtx = null;
    const audioBufferCache = {};  // file -> AudioBuffer

    function getAudioCtx() {
        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
        if (audioCtx.state === "suspended") {
            audioCtx.resume();
        }
        return audioCtx;
    }

    async function loadAudioBuffer(file) {
        if (audioBufferCache[file]) return audioBufferCache[file];
        if (audioBufferCache[file] === false) return null;  // previously failed
        try {
            const resp = await fetch(file);
            if (!resp.ok) {
                audioBufferCache[file] = false;
                return null;
            }
            const arrayBuf = await resp.arrayBuffer();
            const buffer = await getAudioCtx().decodeAudioData(arrayBuf);
            audioBufferCache[file] = buffer;
            return buffer;
        } catch (e) {
            console.warn("[FishAudio] load error:", file, e);
            audioBufferCache[file] = false;
            return null;
        }
    }

    const soundGeneration = {};  // id -> generation counter for cancellation
    const soundLastPlay = {};    // id -> timestamp of last play (for throttling)

    async function playSound(id, file, volume, loop) {
        const now = performance.now();
        if (!loop && soundLastPlay[id] && (now - soundLastPlay[id]) < 400) return;
        soundLastPlay[id] = now;

        const gen = (soundGeneration[id] || 0) + 1;
        soundGeneration[id] = gen;
        if (audioInstances[id]) {
            try { audioInstances[id].source.stop(); } catch (_) {}
            delete audioInstances[id];
        }
        try {
            const ctx = getAudioCtx();
            const buffer = await loadAudioBuffer(file);
            if (soundGeneration[id] !== gen) return;
            if (!buffer) return;
            const source = ctx.createBufferSource();
            source.buffer = buffer;
            source.loop = !!loop;

            // Pitch variation for splash sounds
            if (id === "fishSplash" || id === "splash") {
                source.playbackRate.value = 0.85 + Math.random() * 0.30;
            }

            const gainNode = ctx.createGain();
            let vol = Math.max(0, Math.min(1, volume != null ? volume : 0.4));
            gainNode.gain.value = vol;
            source.connect(gainNode);
            gainNode.connect(ctx.destination);
            source.start(0);
            audioInstances[id] = { source, gainNode, buffer, loop: !!loop };
            if (!loop) {
                source.addEventListener("ended", () => { delete audioInstances[id]; });
            }
        } catch (e) {
            console.warn("[FishAudio] play error:", id, e);
        }
    }

    function stopSound(id) {
        soundGeneration[id] = (soundGeneration[id] || 0) + 1;
        if (audioInstances[id]) {
            try { audioInstances[id].source.stop(); } catch (_) {}
            delete audioInstances[id];
        }
    }

    function stopAllSounds() {
        for (const id in audioInstances) {
            try { audioInstances[id].source.stop(); } catch (_) {}
            delete audioInstances[id];
        }
    }

    function setSoundVolume(id, volume) {
        if (audioInstances[id]) {
            audioInstances[id].gainNode.gain.value = Math.max(0, Math.min(1, volume));
        }
    }

    // ---- MUSIC TRACK DISPLAY ----
    function updateNowPlaying(trackName) {
        if (!nowPlaying || !npTrackName) return;
        if (trackName && trackName !== "Off") {
            npTrackName.textContent = trackName;
            nowPlaying.classList.add("playing");
        } else {
            npTrackName.textContent = "Off";
            nowPlaying.classList.remove("playing");
        }
        show(nowPlaying);
    }

    // Audio messages
    window.addEventListener("message", (event) => {
        const d = event.data;
        if (!d) return;

        if (d.action === "playSound")        playSound(d.id, d.file, d.volume, d.loop);
        if (d.action === "stopSound")        stopSound(d.id);
        if (d.action === "stopAllSounds")  { stopAllSounds(); stopReelingSplash(); }
        if (d.action === "startReelSplash")  startReelingSplash();
        if (d.action === "stopReelSplash")   stopReelingSplash();
        if (d.action === "setSoundVolume")   setSoundVolume(d.id, d.volume);
        if (d.action === "updateNowPlaying") updateNowPlaying(d.trackName || null);
    });

    setHookRig();
})();
