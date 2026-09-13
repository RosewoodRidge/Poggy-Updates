/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — timeline.js
   NLE-style timeline: transport bar, zoomable/scrollable ruler, an
   animation track with back-to-back clip blocks (trim / reorder / drop
   from browser), and one lane per prop with in/out range + keyframes.
   All edits go through State; playback goes through Transport.
   ════════════════════════════════════════════════════════════════════ */
"use strict";

const Timeline = (() => {
    const LABEL_W = 176, PAD_RIGHT = 240, MIN_PPS = 6, MAX_PPS = 1500, SNAP_PX = 7;
    const DEFAULT_H = 236, MIN_H = 120;

    let root, elBody, elContent, elRulerInner, elLanes, elPlayhead, elPlayheadTime, elInsert;
    let pps = 120, userZoomed = false, panelH = DEFAULT_H;
    let drag = null;
    let lastTotal = -1;
    let followPlayhead = true;

    // transport refs
    let btnPlay, btnLoop, btnAutoKey, btnSnap, elCur, elTot, speedSlider, speedLabel, zoomSlider;

    // ── coordinate helpers ──────────────────────────────────────────
    const total = () => State.totalDuration();
    const timeToX = (t) => LABEL_W + t * pps;                       // content px
    function mouseToTime(clientX) {
        const left = elRulerInner.getBoundingClientRect().left;
        return (clientX - left) / pps;
    }
    function trackWidth() { return Math.max(total() * pps, 40); }

    // ═══════════════════════════════════════════════════════════════
    //  INIT
    // ═══════════════════════════════════════════════════════════════
    function init(container) {
        root = container;
        root.style.height = panelH + "px";
        root.innerHTML = `
            <div class="tl-resize" title="Drag to resize"></div>
            <div class="tl-transport">
                <div class="tl-group">
                    <button id="tpStart" class="tbtn" title="Go to start (Home)">⏮</button>
                    <button id="tpPrev" class="tbtn" title="Previous clip / keyframe (↑)">◀◆</button>
                    <button id="tpPlay" class="tbtn play" title="Play / Pause (Space)">▶</button>
                    <button id="tpNext" class="tbtn" title="Next clip / keyframe (↓)">◆▶</button>
                    <button id="tpEnd" class="tbtn" title="Go to end (End)">⏭</button>
                    <button id="tpStop" class="tbtn" title="Stop (rewind, clear anim)">■</button>
                    <span class="tl-sep"></span>
                    <button id="tpLoop" class="tbtn toggle" title="Loop playback (L)">↻</button>
                </div>
                <div class="tl-group tl-timecode">
                    <span id="tlCur" class="tc">0:00.00</span><span class="tc-sep">/</span><span id="tlTot" class="tc dim">0:00.00</span>
                </div>
                <div class="tl-group tl-right">
                    <button id="tpAutoKey" class="tbtn toggle" title="Auto-key: slider edits create keyframes at the playhead (A)">◆ Auto-key</button>
                    <button id="tpSnap" class="tbtn toggle" title="Snap to clip edges, keyframes and seconds (S)">⌖ Snap</button>
                    <span class="tl-sep"></span>
                    <label class="tl-slider"><span>Speed</span><input id="tpSpeed" type="range" min="0.1" max="2" step="0.1" value="1"><b id="tpSpeedLabel">1.0×</b></label>
                    <span class="tl-sep"></span>
                    <label class="tl-slider"><span>Zoom</span><input id="tpZoom" type="range" min="0" max="1000" step="1" value="500"></label>
                    <button id="tpFit" class="tbtn" title="Fit timeline to view (F)">Fit</button>
                </div>
            </div>
            <div class="tl-body">
                <div class="tl-content">
                    <div class="tl-ruler">
                        <div class="tl-ruler-corner"><span id="tlRulerInfo">0 clips</span></div>
                        <div class="tl-ruler-inner"></div>
                    </div>
                    <div class="tl-lanes"></div>
                    <div class="tl-insert hidden"></div>
                    <div class="tl-playhead"><div class="tl-playhead-grab"></div><div class="tl-playhead-time">0:00.00</div></div>
                </div>
            </div>`;

        elBody = $(".tl-body", root); elContent = $(".tl-content", root);
        elRulerInner = $(".tl-ruler-inner", root); elLanes = $(".tl-lanes", root);
        elPlayhead = $(".tl-playhead", root); elPlayheadTime = $(".tl-playhead-time", root); elInsert = $(".tl-insert", root);

        btnPlay = $("#tpPlay", root); btnLoop = $("#tpLoop", root); btnAutoKey = $("#tpAutoKey", root); btnSnap = $("#tpSnap", root);
        elCur = $("#tlCur", root); elTot = $("#tlTot", root); speedSlider = $("#tpSpeed", root); speedLabel = $("#tpSpeedLabel", root); zoomSlider = $("#tpZoom", root);

        $("#tpStart", root).addEventListener("click", () => Transport.seek(0, true));
        $("#tpEnd", root).addEventListener("click", () => Transport.seek(total(), true));
        $("#tpPrev", root).addEventListener("click", () => Transport.jump(-1));
        $("#tpNext", root).addEventListener("click", () => Transport.jump(1));
        btnPlay.addEventListener("click", () => Transport.toggle());
        $("#tpStop", root).addEventListener("click", () => Transport.stop());
        btnLoop.addEventListener("click", () => Transport.setLoop(!State.project.loop));
        btnAutoKey.addEventListener("click", () => State.setUI({ autoKey: !State.ui.autoKey }));
        btnSnap.addEventListener("click", () => State.setUI({ snap: !State.ui.snap }));
        speedSlider.addEventListener("input", () => Transport.setSpeed(parseFloat(speedSlider.value)));
        zoomSlider.addEventListener("input", () => { userZoomed = true; setPps(sliderToPps(parseFloat(zoomSlider.value)), null); });
        $("#tpFit", root).addEventListener("click", zoomFit);

        $(".tl-resize", root).addEventListener("mousedown", (e) => { e.preventDefault(); drag = { type: "resize", startY: e.clientY, startH: panelH }; });
        elContent.addEventListener("mousedown", onMouseDown);
        elBody.addEventListener("wheel", onWheel, { passive: false });
        elBody.addEventListener("scroll", () => { updatePlayhead(); });
        document.addEventListener("mousemove", onMouseMove);
        document.addEventListener("mouseup", onMouseUp);

        // drop from browser
        elContent.addEventListener("dragover", onDragOver);
        elContent.addEventListener("dragleave", () => elInsert.classList.add("hidden"));
        elContent.addEventListener("drop", onDrop);

        if (typeof ResizeObserver !== "undefined") new ResizeObserver(() => { if (!userZoomed) zoomFit(false); else updatePlayhead(); }).observe(elBody);

        State.subscribe((reason) => {
            if (reason === "project") { render(); }
            else if (reason === "selection") { syncSelection(); }
            else if (reason === "time") { updatePlayhead(); updateTimecode(); }
            else if (reason === "ui") { updateTransport(); }
        });
        render();
    }

    // ═══════════════════════════════════════════════════════════════
    //  ZOOM
    // ═══════════════════════════════════════════════════════════════
    function sliderToPps(v) { return MIN_PPS * Math.pow(MAX_PPS / MIN_PPS, v / 1000); }
    function ppsToSlider(p) { return 1000 * Math.log(p / MIN_PPS) / Math.log(MAX_PPS / MIN_PPS); }

    /** Set zoom, keeping `anchorTime` under the same screen x when given */
    function setPps(next, anchorClientX) {
        next = clampNum(next, MIN_PPS, MAX_PPS);
        let anchorT = null, anchorOff = 0;
        if (anchorClientX != null) { anchorT = mouseToTime(anchorClientX); anchorOff = anchorClientX - elBody.getBoundingClientRect().left; }
        pps = next;
        zoomSlider.value = ppsToSlider(pps);
        render();
        if (anchorT != null) elBody.scrollLeft = timeToX(anchorT) - anchorOff;
        updatePlayhead();
    }

    function zoomFit(markUser = false) {
        const avail = Math.max(120, elBody.clientWidth - LABEL_W - 40);
        const t = total();
        const p = t > 0 ? clampNum(avail / t, MIN_PPS, MAX_PPS) : 120;
        userZoomed = !!markUser && false;
        pps = p; zoomSlider.value = ppsToSlider(pps);
        render(); elBody.scrollLeft = 0; updatePlayhead();
    }

    function onWheel(e) {
        if (e.ctrlKey || e.metaKey) {
            e.preventDefault();
            userZoomed = true;
            setPps(pps * (e.deltaY < 0 ? 1.15 : 1 / 1.15), e.clientX);
        } else if (e.shiftKey || Math.abs(e.deltaX) > Math.abs(e.deltaY)) {
            // horizontal scroll: let the browser handle it
        }
    }

    // ═══════════════════════════════════════════════════════════════
    //  RENDER
    // ═══════════════════════════════════════════════════════════════
    function render() {
        const L = State.layout();
        const t = L.total;
        if (t !== lastTotal) {
            if (!userZoomed) { const avail = Math.max(120, elBody.clientWidth - LABEL_W - 40); pps = t > 0 ? clampNum(avail / t, MIN_PPS, MAX_PPS) : 120; zoomSlider.value = ppsToSlider(pps); }
            lastTotal = t;
            if (State.ui.time > t) State.ui.time = t;
        }
        elContent.style.width = (LABEL_W + trackWidth() + PAD_RIGHT) + "px";
        elRulerInner.style.width = trackWidth() + "px";
        renderRuler(t);
        renderLanes(L);
        updatePlayhead();
        updateTimecode();
        updateTransport();
        $("#tlRulerInfo", root).textContent = `${L.clips.length} clip${L.clips.length === 1 ? "" : "s"} · ${State.project.props.length} prop${State.project.props.length === 1 ? "" : "s"}`;
    }

    function renderRuler(t) {
        elRulerInner.innerHTML = "";
        if (t <= 0) return;
        const steps = [0.1, 0.25, 0.5, 1, 2, 5, 10, 15, 30, 60];
        let major = steps.find(s => s * pps >= 64) || 60;
        const minor = major / (major === 0.25 || major === 0.5 ? 5 : (major >= 5 ? 5 : 4));
        const frag = document.createDocumentFragment();
        const end = t + 1e-6;
        for (let x = 0; x <= end; x = +(x + minor).toFixed(6)) {
            const isMajor = Math.abs(x / major - Math.round(x / major)) < 1e-6;
            const tick = el("div", "tl-tick" + (isMajor ? " major" : ""));
            tick.style.left = (x * pps) + "px";
            frag.appendChild(tick);
            if (isMajor) { const lab = el("span", "tl-tick-label", fmtTime(x)); lab.style.left = (x * pps) + "px"; frag.appendChild(lab); }
        }
        const endTick = el("div", "tl-tick end"); endTick.style.left = (t * pps) + "px"; frag.appendChild(endTick);
        elRulerInner.appendChild(frag);
    }

    function renderLanes(L) {
        elLanes.innerHTML = "";
        const sel = State.ui.sel;
        const frag = document.createDocumentFragment();
        const trackW = trackWidth();

        // ── Animation track ───────────────────────────────────────
        const animLane = el("div", "tl-lane anim-lane");
        animLane.appendChild(el("div", "tl-lane-label", `<span class="tl-lane-icon">🎬</span><span class="tl-lane-name">Animation</span><span class="tl-lane-sub">${L.clips.length ? fmtSec(L.total) : ""}</span>`));
        const animTrack = el("div", "tl-lane-track anim-track"); animTrack.style.width = trackW + "px";
        if (!L.clips.length) animTrack.appendChild(el("div", "tl-track-hint", "Double-click a clip in the library, press its + button, or drag it here"));
        for (const e of L.clips) {
            const c = e.clip;
            const block = el("div", "tl-clip" + (c.status === "pending" ? " pending" : "") + (c.status === "error" ? " error" : "") + (c.enabled === false ? " disabled" : "") + (sel.type === "clip" && sel.clipId === c.id ? " selected" : ""));
            block.dataset.clipId = c.id;
            const w = e.valid ? e.length * pps : 60;
            block.style.left = (e.start * pps) + "px"; block.style.width = Math.max(w, 10) + "px";
            if (!e.valid) { block.style.position = "absolute"; block.classList.add("floating"); block.style.left = (L.total * pps + 8 + (L.clips.indexOf(e) * 4)) + "px"; }
            const holdW = e.valid ? c.hold * pps : 0;
            block.innerHTML =
                `<div class="tl-clip-handle l" title="Trim in"></div>` +
                `<div class="tl-clip-body"><span class="tl-clip-name">${esc(c.clip)}</span><span class="tl-clip-dict">${esc(c.dict)}</span>` +
                `<span class="tl-clip-meta">${c.status === "pending" ? "resolving…" : c.status === "error" ? "⚠ " + esc(c.error || "error") : fmtSec(e.length) + (c.estimated ? " ≈" : "")}</span></div>` +
                (holdW > 0 ? `<div class="tl-clip-hold" style="width:${holdW}px" title="Hold last frame ${fmtSec(c.hold)}"><span>hold</span></div>` : "") +
                `<div class="tl-clip-handle r" title="Trim out / extend hold"></div>`;
            animTrack.appendChild(block);
        }
        animLane.appendChild(animTrack);
        frag.appendChild(animLane);

        // ── Prop lanes ────────────────────────────────────────────
        for (const p of State.project.props) {
            const isSel = sel.propId === p.id;
            const lane = el("div", "tl-lane prop-lane" + (isSel ? " selected" : "") + (p.enabled === false ? " disabled" : ""));
            lane.dataset.propId = p.id;
            const label = el("div", "tl-lane-label",
                `<span class="tl-lane-dot" style="background:${p.color}"></span><span class="tl-lane-name" title="${esc(p.model)}">${esc(p.model)}</span>` +
                `<span class="tl-lane-btns"><button class="mini eye${p.enabled === false ? " off" : ""}" title="Enable / disable prop">${p.enabled === false ? "◌" : "◉"}</button><button class="mini del" title="Delete prop">✕</button></span>`);
            lane.appendChild(label);
            const track = el("div", "tl-lane-track prop-track"); track.style.width = trackW + "px";
            const inT = p.inTime, outT = p.outTime == null ? L.total : Math.min(p.outTime, Math.max(L.total, p.outTime));
            const bar = el("div", "tl-prop-bar" + (isSel ? " selected" : ""));
            bar.dataset.propId = p.id;
            bar.style.left = (inT * pps) + "px";
            bar.style.width = Math.max(4, (outT - inT) * pps) + "px";
            bar.style.setProperty("--c", p.color);
            bar.style.setProperty("--c-bg", p.color + "66"); bar.style.setProperty("--c-hover", p.color + "8c"); bar.style.setProperty("--c-line", p.color + "bf");
            bar.innerHTML = `<div class="tl-prop-handle in" title="In point"></div><div class="tl-prop-handle out${p.outTime == null ? " follow" : ""}" title="${p.outTime == null ? "Out point (follows end)" : "Out point"}"></div>`;
            if (p.error) bar.appendChild(el("span", "tl-prop-err", "⚠ " + esc(p.error)));
            track.appendChild(bar);
            for (const k of p.keyframes) {
                const d = el("div", "tl-kf" + (sel.type === "kf" && sel.kfId === k.id ? " selected" : "") + (k.ease && k.ease !== "linear" ? " ease-" + k.ease : ""));
                d.dataset.propId = p.id; d.dataset.kfId = k.id;
                d.style.left = (k.t * pps) + "px";
                d.title = `${fmtSec(k.t, 3)} · ${k.ease || "linear"}`;
                track.appendChild(d);
            }
            lane.appendChild(track);
            frag.appendChild(lane);
        }
        if (!State.project.props.length) {
            const hint = el("div", "tl-lane hint-lane");
            hint.appendChild(el("div", "tl-lane-label", `<span class="tl-lane-icon dim">◆</span><span class="tl-lane-name dim">Props</span>`));
            const tr = el("div", "tl-lane-track"); tr.style.width = trackW + "px";
            tr.appendChild(el("div", "tl-track-hint", "Add a prop from the Inspector to get a keyframe lane"));
            hint.appendChild(tr); frag.appendChild(hint);
        }
        elLanes.appendChild(frag);
    }

    function syncSelection() {
        const sel = State.ui.sel;
        $$(".tl-clip", elLanes).forEach(b => b.classList.toggle("selected", sel.type === "clip" && +b.dataset.clipId === sel.clipId));
        $$(".prop-lane", elLanes).forEach(l => l.classList.toggle("selected", +l.dataset.propId === sel.propId));
        $$(".tl-prop-bar", elLanes).forEach(b => b.classList.toggle("selected", +b.dataset.propId === sel.propId));
        $$(".tl-kf", elLanes).forEach(k => k.classList.toggle("selected", sel.type === "kf" && +k.dataset.kfId === sel.kfId));
    }

    function updatePlayhead() {
        const t = State.ui.time;
        const x = timeToX(t);
        elPlayhead.style.left = x + "px";
        elPlayheadTime.textContent = fmtTime(t);
        const visible = total() > 0 && (x - elBody.scrollLeft) >= LABEL_W - 1;
        elPlayhead.style.visibility = visible ? "visible" : "hidden";
        // follow during playback
        if (State.ui.playing && followPlayhead && !drag) {
            const viewR = elBody.scrollLeft + elBody.clientWidth;
            if (x > viewR - 24 || x < elBody.scrollLeft + LABEL_W) elBody.scrollLeft = Math.max(0, x - LABEL_W - 24);
        }
    }

    function updateTimecode() { elCur.textContent = fmtTime(State.ui.time); elTot.textContent = fmtTime(total()); }

    function updateTransport() {
        const u = State.ui;
        btnPlay.textContent = u.playing ? "⏸" : "▶";
        btnPlay.classList.toggle("on", u.playing);
        btnLoop.classList.toggle("on", !!State.project.loop);
        btnAutoKey.classList.toggle("on", !!u.autoKey);
        btnSnap.classList.toggle("on", !!u.snap);
        if (document.activeElement !== speedSlider) speedSlider.value = u.speed;
        speedLabel.textContent = u.speed.toFixed(1) + "×";
        root.classList.toggle("playing", !!u.playing);
    }

    // ═══════════════════════════════════════════════════════════════
    //  SNAPPING
    // ═══════════════════════════════════════════════════════════════
    function snapTime(t, opts = {}) {
        t = clampNum(t, 0, opts.max != null ? opts.max : total());
        if (!State.ui.snap || opts.noSnap) return t;
        const tol = SNAP_PX / pps;
        const cands = State.boundaries();
        if (!opts.ignorePlayhead) cands.push(State.ui.time);
        for (const p of State.project.props) {
            if (p.outTime != null) cands.push(p.outTime);
            if (p.inTime > 0) cands.push(p.inTime);
            for (const k of p.keyframes) if (!(opts.excludeKf && k.id === opts.excludeKf)) cands.push(k.t);
        }
        for (let s = Math.floor(t) - 1; s <= Math.ceil(t) + 1; s++) if (s >= 0) cands.push(s);
        let best = null, bestD = tol;
        for (const c of cands) { const d = Math.abs(c - t); if (d < bestD) { bestD = d; best = c; } }
        return best != null ? best : t;
    }

    // ═══════════════════════════════════════════════════════════════
    //  MOUSE INTERACTION
    // ═══════════════════════════════════════════════════════════════
    function onMouseDown(e) {
        if (e.button !== 0) return;
        const tgt = e.target;

        // lane label buttons
        const laneLabel = tgt.closest(".tl-lane-label");
        if (laneLabel) {
            const lane = laneLabel.closest(".tl-lane");
            if (lane.classList.contains("prop-lane")) {
                const id = +lane.dataset.propId;
                if (tgt.closest(".eye")) { const p = State.getProp(id); State.setProp(id, { enabled: p.enabled === false }); return; }
                if (tgt.closest(".del")) { App.deleteProp(id); return; }
                State.select({ type: "prop", propId: id });
            }
            return;
        }

        // keyframe
        const kf = tgt.closest(".tl-kf");
        if (kf) {
            e.preventDefault();
            const propId = +kf.dataset.propId, kfId = +kf.dataset.kfId;
            const p = State.getProp(propId); const k = p && p.keyframes.find(x => x.id === kfId);
            if (!k) return;
            State.select({ type: "kf", propId, kfId });
            drag = { type: "kf", propId, kfId, startX: e.clientX, startT: k.t, moved: false, ghost: makeGhost(e) };
            return;
        }

        // prop range handles / bar
        const ph = tgt.closest(".tl-prop-handle");
        if (ph) {
            e.preventDefault();
            const bar = ph.closest(".tl-prop-bar"); const propId = +bar.dataset.propId;
            const p = State.getProp(propId);
            State.select({ type: "prop", propId });
            drag = { type: ph.classList.contains("in") ? "prop-in" : "prop-out", propId, startX: e.clientX, startIn: p.inTime, startOut: p.outTime == null ? total() : p.outTime, ghost: makeGhost(e) };
            return;
        }
        const pbar = tgt.closest(".tl-prop-bar");
        if (pbar) {
            e.preventDefault();
            const propId = +pbar.dataset.propId; const p = State.getProp(propId);
            State.select({ type: "prop", propId });
            drag = { type: "prop-move", propId, startX: e.clientX, startIn: p.inTime, startOut: p.outTime == null ? null : p.outTime, moved: false, ghost: makeGhost(e) };
            return;
        }

        // clip trim handles / block
        const ch = tgt.closest(".tl-clip-handle");
        if (ch) {
            e.preventDefault();
            const block = ch.closest(".tl-clip"); const clipId = +block.dataset.clipId;
            const c = State.getClip(clipId); if (!c || c.status !== "ok") return;
            State.select({ type: "clip", clipId });
            drag = { type: ch.classList.contains("l") ? "trim-in" : "trim-out", clipId, startX: e.clientX, startIn: c.trimIn, startOut: c.trimOut, startHold: c.hold, ghost: makeGhost(e) };
            return;
        }
        const block = tgt.closest(".tl-clip");
        if (block) {
            e.preventDefault();
            const clipId = +block.dataset.clipId;
            State.select({ type: "clip", clipId });
            drag = { type: "clip-move", clipId, startX: e.clientX, moved: false, el: block, toIndex: null };
            return;
        }

        // playhead grab / ruler / empty track → scrub
        if (tgt.closest(".tl-playhead-grab") || tgt.closest(".tl-ruler-inner") || tgt.closest(".tl-lane-track")) {
            if (total() <= 0) return;
            e.preventDefault();
            if (tgt.closest(".tl-lane-track") && !tgt.closest(".tl-ruler-inner")) State.select(null);
            drag = { type: "scrub" };
            followPlayhead = false;
            Transport.beginScrub();
            Transport.scrub(snapTime(mouseToTime(e.clientX), { ignorePlayhead: true, noSnap: !e.shiftKey && !State.ui.snap }));
            return;
        }
        if (tgt.closest(".tl-ruler-corner")) State.select(null);
    }

    function onMouseMove(e) {
        if (!drag) return;
        const dx = e.clientX - drag.startX;
        switch (drag.type) {
            case "resize": {
                panelH = clampNum(drag.startH + (drag.startY - e.clientY), MIN_H, window.innerHeight * 0.7);
                root.style.height = panelH + "px";
                break;
            }
            case "scrub":
                Transport.scrub(snapTime(mouseToTime(e.clientX), { ignorePlayhead: true }));
                break;
            case "kf": {
                if (!drag.moved && Math.abs(dx) < 3) break;
                drag.moved = true;
                const t = snapTime(drag.startT + dx / pps, { excludeKf: drag.kfId });
                State.setKeyframe(drag.propId, drag.kfId, { t }, { coalesce: "kf-move-" + drag.kfId, reason: "moveKf" });
                ghost(drag.ghost, e, fmtSec(t, 3));
                break;
            }
            case "prop-in": {
                const maxIn = drag.startOut - State.MIN_CLIP_LEN;
                const t = clampNum(snapTime(drag.startIn + dx / pps), 0, maxIn);
                State.setProp(drag.propId, { inTime: t }, { coalesce: "prop-in-" + drag.propId, reason: "propRange" });
                ghost(drag.ghost, e, "in " + fmtSec(t));
                break;
            }
            case "prop-out": {
                let t = clampNum(snapTime(drag.startOut + dx / pps), drag.startIn + State.MIN_CLIP_LEN, total());
                const follow = t >= total() - SNAP_PX / pps;
                State.setProp(drag.propId, { outTime: follow ? null : t }, { coalesce: "prop-out-" + drag.propId, reason: "propRange" });
                ghost(drag.ghost, e, follow ? "out → end" : "out " + fmtSec(t));
                break;
            }
            case "prop-move": {
                if (!drag.moved && Math.abs(dx) < 3) break;
                drag.moved = true;
                const len = (drag.startOut == null ? total() : drag.startOut) - drag.startIn;
                let inT = snapTime(drag.startIn + dx / pps, { max: Math.max(0, total() - len) });
                inT = clampNum(inT, 0, Math.max(0, total() - len));
                const outT = drag.startOut == null ? null : inT + len;
                State.setProp(drag.propId, { inTime: inT, outTime: outT }, { coalesce: "prop-move-" + drag.propId, reason: "propRange" });
                ghost(drag.ghost, e, `${fmtSec(inT)} – ${outT == null ? "end" : fmtSec(outT)}`);
                break;
            }
            case "trim-in": {
                const c = State.getClip(drag.clipId); if (!c) break;
                const t = clampNum(drag.startIn + dx / pps, 0, drag.startOut - State.MIN_CLIP_LEN);
                State.setClip(drag.clipId, { trimIn: t }, { coalesce: "trim-in-" + drag.clipId, reason: "trim" });
                ghost(drag.ghost, e, `in ${fmtSec(t)} · len ${fmtSec(drag.startOut - t + c.hold)}`);
                break;
            }
            case "trim-out": {
                const c = State.getClip(drag.clipId); if (!c) break;
                const desired = Math.max(drag.startIn + State.MIN_CLIP_LEN, drag.startOut + drag.startHold + dx / pps);
                let trimOut, hold;
                if (desired <= c.duration) { trimOut = desired; hold = 0; } else { trimOut = c.duration; hold = desired - c.duration; }
                State.setClip(drag.clipId, { trimOut, hold }, { coalesce: "trim-out-" + drag.clipId, reason: "trim" });
                ghost(drag.ghost, e, hold > 0 ? `out ${fmtSec(trimOut)} + hold ${fmtSec(hold)}` : `out ${fmtSec(trimOut)}`);
                break;
            }
            case "clip-move": {
                if (!drag.moved && Math.abs(dx) < 4) break;
                drag.moved = true;
                drag.el.style.transform = `translateX(${dx}px)`; drag.el.classList.add("dragging");
                const L = State.layout();
                const me = L.clips.find(x => x.clip.id === drag.clipId);
                const center = (me.start + me.length / 2) * pps + dx;
                let idx = 0, x = 0;
                const others = L.clips.filter(x => x.clip.id !== drag.clipId);
                for (const o of others) { const oc = (o.start + o.length / 2) * pps; if (center > oc) { idx++; x = (o.start + o.length) * pps; } }
                if (idx === 0) x = 0; else if (!others[idx - 1].valid) x = (others[idx - 1].start) * pps;
                drag.toIndex = State.clipIndex(others[idx] ? others[idx].clip.id : -1);
                if (drag.toIndex < 0) drag.toIndex = State.project.clips.length;
                showInsert(x);
                break;
            }
        }
    }

    function onMouseUp(e) {
        if (!drag) return;
        const d = drag; drag = null;
        if (d.ghost) d.ghost.remove();
        elInsert.classList.add("hidden");
        switch (d.type) {
            case "scrub": Transport.endScrub(); followPlayhead = true; break;
            case "kf":
                if (!d.moved) { const p = State.getProp(d.propId); const k = p && p.keyframes.find(x => x.id === d.kfId); if (k) Transport.seek(k.t, true); }
                break;
            case "clip-move": {
                if (d.el) { d.el.style.transform = ""; d.el.classList.remove("dragging"); }
                if (d.moved && d.toIndex != null) {
                    const from = State.clipIndex(d.clipId);
                    let to = d.toIndex; if (to > from) to--;   // account for removal
                    if (to !== from) State.moveClip(d.clipId, to);
                }
                break;
            }
        }
    }

    function showInsert(x) { elInsert.classList.remove("hidden"); elInsert.style.left = (LABEL_W + x) + "px"; }

    // ── drop from browser ─────────────────────────────────────────
    function dropIndexFromX(clientX) {
        const t = mouseToTime(clientX);
        const L = State.layout();
        let idx = 0, x = 0;
        for (const e of L.clips) { if (!e.valid) continue; if (t > e.start + e.length / 2) { idx = State.clipIndex(e.clip.id) + 1; x = (e.start + e.length) * pps; } }
        return { idx, x };
    }
    function onDragOver(e) {
        if (![...e.dataTransfer.types].includes("text/animtool-clip")) return;
        e.preventDefault(); e.dataTransfer.dropEffect = "copy";
        showInsert(dropIndexFromX(e.clientX).x);
    }
    function onDrop(e) {
        const raw = e.dataTransfer.getData("text/animtool-clip"); if (!raw) return;
        e.preventDefault(); elInsert.classList.add("hidden");
        try { const { dict, clip } = JSON.parse(raw); App.addClip(dict, clip, dropIndexFromX(e.clientX).idx); } catch {}
    }

    // ── ghost tooltip ─────────────────────────────────────────────
    function makeGhost(e) { const g = el("div", "tl-ghost"); document.body.appendChild(g); g.style.left = (e.clientX + 14) + "px"; g.style.top = (e.clientY - 26) + "px"; return g; }
    function ghost(g, e, text) { if (!g) return; g.textContent = text; g.style.left = (e.clientX + 14) + "px"; g.style.top = (e.clientY - 26) + "px"; }

    return { init, render, zoomFit, zoomBy: (f) => { userZoomed = true; setPps(pps * f, null); }, getPps: () => pps };
})();
