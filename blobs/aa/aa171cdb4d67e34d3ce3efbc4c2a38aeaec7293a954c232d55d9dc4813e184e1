/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — app.js
   Glue: transport, sync to the game, projects (save/load/import),
   keyboard shortcuts, toasts, NUI message handling, boot.
   ════════════════════════════════════════════════════════════════════ */
"use strict";

/* ────────────────────────────────────────────────────────────────────
   Transport — playback commands (UI is optimistic, Lua clock is truth)
   ──────────────────────────────────────────────────────────────────── */
const Transport = (() => {
    const postSeek = throttle((t) => NUI.post("transport", { action: "seek", time: t }), 33);
    let scrubEndTimer = null;

    function clearPreview() { if (State.ui.preview) Browser.onPreviewStopped(); }

    function play() {
        if (State.totalDuration() <= 0) { App.toast("Add a clip to the timeline first", "warn"); return; }
        clearPreview();
        if (State.ui.time >= State.totalDuration() - 0.0005) State.setTime(0, "play");
        State.setPlaying(true);
        NUI.post("transport", { action: "play" });
    }
    function pause() { State.setPlaying(false); NUI.post("transport", { action: "pause" }); }
    function toggle() { State.ui.playing ? pause() : play(); }
    function stop() { clearPreview(); State.setPlaying(false); State.setTime(0, "stop"); NUI.post("transport", { action: "stop" }); }
    function seek(t) { clearPreview(); State.setTime(t, "seek"); postSeek(State.ui.time); }

    function beginScrub() {
        State.ui.scrubbing = true; clearTimeout(scrubEndTimer);
        if (State.ui.playing) { State.setPlaying(false); NUI.post("transport", { action: "pause" }); }
        clearPreview();
    }
    function scrub(t) { State.setTime(t, "scrub"); postSeek(State.ui.time); }
    function endScrub() { NUI.post("transport", { action: "seek", time: State.ui.time }); scrubEndTimer = setTimeout(() => { State.ui.scrubbing = false; }, 150); }

    function setSpeed(s) { s = clampNum(+s || 1, 0.1, 2); State.setUI({ speed: s }); NUI.post("setSpeed", { speed: s }); }
    function setLoop(b) { State.update(p => { p.loop = !!b; }, { reason: "loop" }); NUI.post("setLoop", { loop: !!b }); }

    function stepFrame(dir, big) { seek(State.ui.time + dir * (big ? 1 : 1 / 30)); }

    /** Jump to previous/next clip boundary or keyframe of the selected prop */
    function jump(dir) {
        const t = State.ui.time, eps = 0.002;
        const cands = State.boundaries();
        const p = State.selectedProp();
        if (p) for (const k of p.keyframes) cands.push(k.t);
        let best = null;
        for (const c of cands) {
            if (dir > 0 && c > t + eps && (best == null || c < best)) best = c;
            if (dir < 0 && c < t - eps && (best == null || c > best)) best = c;
        }
        if (best != null) seek(best);
    }

    return { play, pause, toggle, stop, seek, beginScrub, scrub, endScrub, setSpeed, setLoop, stepFrame, jump };
})();

/* ────────────────────────────────────────────────────────────────────
   App
   ──────────────────────────────────────────────────────────────────── */
const App = (() => {
    const ANIM_CHUNKS = 14, OBJ_CHUNKS = 5;
    const RECOVER_KEY = "poggy_animtool_recover";
    const AUTOSAVE_MS = 60000;

    let clipboard = null;      // { pos, rot }
    let bones = [];
    let uiOpen = false;
    let elApp, elToasts, elProjectName, elDropdown, elDropItems, elStatus;
    let cameraHold = null;

    // ── sync to game ────────────────────────────────────────────
    const scheduleSync = debounce(() => NUI.post("syncScene", { scene: State.toScene() }), 30);
    function syncNow() { NUI.post("syncScene", { scene: State.toScene() }); }

    // ── recovery snapshot ───────────────────────────────────────
    const snapshotRecovery = debounce(() => {
        try {
            const p = State.project;
            if (p.clips.length || p.props.length) localStorage.setItem(RECOVER_KEY, JSON.stringify({ at: Date.now(), name: p.name, data: State.serialize() }));
            else localStorage.removeItem(RECOVER_KEY);
        } catch {}
    }, 1500);

    // ═══════════════════════════════════════════════════════════
    //  CLIPS / PROPS
    // ═══════════════════════════════════════════════════════════
    function addClip(dict, clip, index) {
        const id = State.addClip(dict, clip, index);
        NUI.post("resolveClip", { id, dict, clip });
        State.select({ type: "clip", clipId: id });
        toast(`Added ${clip}`, "ok");
        return id;
    }

    function resolvePending() {
        for (const c of State.project.clips) if (c.status !== "ok") NUI.post("resolveClip", { id: c.id, dict: c.dict, clip: c.clip });
    }

    function deleteClip(id) {
        const c = State.getClip(id); if (!c) return;
        State.removeClip(id); State.select(null);
        toast(`Removed ${c.clip}`, "info", { label: "Undo", fn: () => State.undo() });
    }

    function deleteProp(id) {
        const p = State.getProp(id); if (!p) return;
        State.removeProp(id); State.select(null);
        toast(`Deleted prop ${p.model}`, "info", { label: "Undo", fn: () => State.undo() });
    }

    function deleteSelection() {
        const s = State.ui.sel;
        if (s.type === "kf") { const p = State.getProp(s.propId); if (p) { State.removeKeyframe(s.propId, s.kfId); State.select({ type: "prop", propId: p.id }); } }
        else if (s.type === "clip") deleteClip(s.clipId);
        else if (s.type === "prop") deleteProp(s.propId);
    }

    function duplicateSelection() {
        const s = State.ui.sel;
        if (s.type === "clip") { const id = State.duplicateClip(s.clipId); if (id != null) State.select({ type: "clip", clipId: id }); }
        else if (s.propId != null) { const id = State.duplicateProp(s.propId); if (id != null) State.select({ type: "prop", propId: id }); }
    }

    function copyTransform() {
        const p = State.selectedProp(); if (!p) return;
        clipboard = State.evalProp(p, State.ui.time);
        toast("Pose copied", "info");
    }
    function pasteTransform() {
        const p = State.selectedProp(); if (!p || !clipboard) { if (!clipboard) toast("Nothing copied yet", "warn"); return; }
        const id = State.addKeyframe(p.id, State.ui.time, clipboard.pos, clipboard.rot);
        State.select({ type: "kf", propId: p.id, kfId: id });
        toast("Pose pasted as keyframe", "ok");
    }

    // ═══════════════════════════════════════════════════════════
    //  PROJECTS
    // ═══════════════════════════════════════════════════════════
    function saveProject(silent) {
        const name = elProjectName.value.trim();
        if (!name) { elProjectName.focus(); if (!silent) toast("Give the project a name first", "warn"); return; }
        State.project.name = name;
        NUI.post("saveProject", { name, data: State.serialize(), silent: !!silent });
    }

    function openProjectList() { NUI.post("listProjects"); }

    function renderProjectList(names) {
        elDropItems.innerHTML = "";
        if (!names.length) elDropItems.appendChild(el("div", "list-empty", "No saved projects"));
        for (const name of names) {
            const row = el("div", "dd-item", `<span class="dd-name">${esc(name)}</span><button class="mini del" title="Delete project">✕</button>`);
            row.addEventListener("click", async (e) => {
                if (e.target.closest(".del")) {
                    e.stopPropagation();
                    if (await confirm(`Delete project "${name}"?`)) { NUI.post("deleteProject", { name }); }
                    return;
                }
                elDropdown.classList.add("hidden");
                if (State.ui.dirty && !(await confirm("Discard unsaved changes and open this project?"))) return;
                NUI.post("loadProject", { name });
            });
            elDropItems.appendChild(row);
        }
        elDropdown.classList.remove("hidden");
    }

    async function newProject() {
        if (State.ui.dirty && !(await confirm("Discard unsaved changes and start a new project?"))) return;
        State.newProject(); elProjectName.value = ""; syncNow(); Transport.stop();
        toast("New project", "info");
    }

    function loadProjectData(obj, name) {
        if (!State.loadProject(obj, name)) { toast("That is not a valid AnimTool project", "error"); return false; }
        elProjectName.value = State.project.name || "";
        Transport.stop();
        syncNow(); resolvePending();
        Timeline.zoomFit();
        return true;
    }

    function openImport() { $("#importModal").classList.remove("hidden"); $("#importText").value = ""; $("#importText").focus(); }
    function doImport() {
        const raw = $("#importText").value.trim(); if (!raw) return;
        let obj; try { obj = JSON.parse(raw); } catch { toast("Import failed: not valid JSON", "error"); return; }
        if (loadProjectData(obj, obj.name || "")) { $("#importModal").classList.add("hidden"); toast("Project imported", "ok"); }
    }

    function checkRecovery() {
        let rec = null;
        try { rec = JSON.parse(localStorage.getItem(RECOVER_KEY) || "null"); } catch {}
        if (!rec || !rec.data || rec.saved) return;
        const age = Math.round((Date.now() - rec.at) / 60000);
        toast(`Unsaved work${rec.name ? ` ("${rec.name}")` : ""} from ${age < 1 ? "moments" : age + " min"} ago`, "info", {
            label: "Restore", fn: () => { try { if (loadProjectData(JSON.parse(rec.data), rec.name)) { State.ui.dirty = true; updateDirty(); } } catch { toast("Recovery data was corrupt", "error"); } },
        }, 20000);
    }

    function updateDirty() {
        $("#dirtyDot").classList.toggle("on", !!State.ui.dirty);
        $("#undoBtn").disabled = !State.canUndo(); $("#redoBtn").disabled = !State.canRedo();
    }

    // ═══════════════════════════════════════════════════════════
    //  TOASTS / CONFIRM
    // ═══════════════════════════════════════════════════════════
    function toast(msg, kind = "info", action = null, ms = 3200) {
        const t = el("div", "toast " + kind, `<span>${esc(msg)}</span>`);
        if (action) { const b = el("button", "toast-btn", esc(action.label)); b.addEventListener("click", () => { action.fn(); t.remove(); }); t.appendChild(b); }
        elToasts.appendChild(t);
        while (elToasts.children.length > 4) elToasts.firstChild.remove();
        setTimeout(() => { t.classList.add("out"); setTimeout(() => t.remove(), 250); }, ms);
    }

    function confirm(msg) {
        return new Promise((resolve) => {
            const m = $("#confirmModal"); $("#confirmText").textContent = msg; m.classList.remove("hidden");
            const done = (v) => { m.classList.add("hidden"); ok.onclick = no.onclick = null; resolve(v); };
            const ok = $("#confirmOk"), no = $("#confirmCancel");
            ok.onclick = () => done(true); no.onclick = () => done(false);
            ok.focus();
        });
    }

    // ═══════════════════════════════════════════════════════════
    //  KEYBOARD
    // ═══════════════════════════════════════════════════════════
    function inField() { const a = document.activeElement; return a && (a.tagName === "INPUT" || a.tagName === "TEXTAREA" || a.tagName === "SELECT"); }

    function onKey(e) {
        const ctrl = e.ctrlKey || e.metaKey;
        if (e.key === "Escape") {
            if (Exporter.isOpen()) return Exporter.close();
            if (!$("#importModal").classList.contains("hidden")) return $("#importModal").classList.add("hidden");
            if (!$("#helpModal").classList.contains("hidden")) return $("#helpModal").classList.add("hidden");
            if (!$("#confirmModal").classList.contains("hidden")) return $("#confirmCancel").click();
            if (!elDropdown.classList.contains("hidden")) return elDropdown.classList.add("hidden");
            if (Inspector.isAddDialogOpen()) return Inspector.closeAddDialog();
            if (inField()) return document.activeElement.blur();
            if (State.ui.preview) return Browser.stopPreview();
            if (State.ui.sel.type) return State.select(null);
            return NUI.post("close");
        }
        if (ctrl && e.key.toLowerCase() === "s") { e.preventDefault(); saveProject(false); return; }
        if (ctrl && e.key.toLowerCase() === "f") { e.preventDefault(); Browser.focusSearch(); return; }
        if (inField()) return;
        if (Exporter.isOpen() || !$("#confirmModal").classList.contains("hidden")) return;

        if (ctrl && e.key.toLowerCase() === "z") { e.preventDefault(); if (e.shiftKey) State.redo(); else State.undo(); return; }
        if (ctrl && e.key.toLowerCase() === "y") { e.preventDefault(); State.redo(); return; }
        if (ctrl && e.key.toLowerCase() === "c") { e.preventDefault(); copyTransform(); return; }
        if (ctrl && e.key.toLowerCase() === "v") { e.preventDefault(); pasteTransform(); return; }
        if (ctrl && e.key.toLowerCase() === "d") { e.preventDefault(); duplicateSelection(); return; }
        if (ctrl) return;

        switch (e.key) {
            case " ": e.preventDefault(); Transport.toggle(); break;
            case "Home": e.preventDefault(); Transport.seek(0); break;
            case "End": e.preventDefault(); Transport.seek(State.totalDuration()); break;
            case "ArrowLeft": e.preventDefault(); Transport.stepFrame(-1, e.shiftKey); break;
            case "ArrowRight": e.preventDefault(); Transport.stepFrame(1, e.shiftKey); break;
            case "ArrowUp": e.preventDefault(); Transport.jump(-1); break;
            case "ArrowDown": e.preventDefault(); Transport.jump(1); break;
            case "Delete": case "Backspace": e.preventDefault(); deleteSelection(); break;
            case "k": case "K": if (State.selectedProp()) Inspector.addKeyframeHere(); else toast("Select a prop to keyframe", "warn"); break;
            case "l": case "L": Transport.setLoop(!State.project.loop); break;
            case "a": case "A": State.setUI({ autoKey: !State.ui.autoKey }); toast("Auto-key " + (State.ui.autoKey ? "on" : "off"), "info"); break;
            case "s": case "S": State.setUI({ snap: !State.ui.snap }); toast("Snap " + (State.ui.snap ? "on" : "off"), "info"); break;
            case "f": case "F": Timeline.zoomFit(); break;
            case "+": case "=": Timeline.zoomBy(1.25); break;
            case "-": case "_": Timeline.zoomBy(1 / 1.25); break;
            case "[": Inspector.setStep(-1); break;
            case "]": Inspector.setStep(1); break;
            case "?": $("#helpModal").classList.remove("hidden"); break;
        }
    }

    // ═══════════════════════════════════════════════════════════
    //  NUI MESSAGES
    // ═══════════════════════════════════════════════════════════
    function bindMessages() {
        NUI.on("open", (msg) => {
            uiOpen = true; elApp.classList.remove("hidden");
            if (msg.bones && msg.bones.length) { bones = msg.bones; Inspector.setBones(bones); }
            syncNow(); resolvePending();
            State.setPlaying(false);
            Timeline.zoomFit();
        });
        NUI.on("close", () => {
            uiOpen = false; elApp.classList.add("hidden");
            State.setPlaying(false); Browser.onPreviewStopped();
            elDropdown.classList.add("hidden");
        });
        NUI.on("progress", (msg) => {
            if (State.ui.scrubbing) return;
            State.setPlaying(!!msg.playing);
            if (typeof msg.time === "number") State.setTime(msg.time, "progress");
        });
        NUI.on("finished", () => State.setPlaying(false));
        NUI.on("clipResolved", (msg) => {
            const c = State.getClip(msg.id); if (!c) return;
            State.resolveClip(msg.id, !!msg.ok, msg.duration, msg.error, msg.estimated);
            if (!msg.ok) toast(`Clip failed: ${msg.error || c.clip}`, "error");
            else if (msg.estimated) toast(`${c.clip}: duration estimated (${fmtSec(msg.duration)})`, "info");
        });
        NUI.on("previewStarted", (msg) => Browser.onPreviewStarted(msg));
        NUI.on("previewStopped", () => Browser.onPreviewStopped());
        NUI.on("propError", (msg) => { State.setPropError(msg.id, msg.message || "failed"); toast(msg.message || "Prop failed", "error"); });
        NUI.on("propClicked", (msg) => { if (State.getProp(msg.id)) State.select({ type: "prop", propId: msg.id }); });
        NUI.on("notify", (msg) => toast(msg.message, msg.kind || "info"));
        NUI.on("projectSaved", (msg) => {
            State.markSaved(msg.name); updateDirty();
            try { const rec = JSON.parse(localStorage.getItem(RECOVER_KEY) || "null"); if (rec) { rec.saved = true; localStorage.setItem(RECOVER_KEY, JSON.stringify(rec)); } } catch {}
            if (!msg.silent) toast(`Saved "${msg.name}"`, "ok");
        });
        NUI.on("projectLoaded", (msg) => {
            let obj; try { obj = typeof msg.data === "string" ? JSON.parse(msg.data) : msg.data; } catch { toast("Project data is corrupt", "error"); return; }
            if (loadProjectData(obj, msg.name)) toast(`Opened "${msg.name}"`, "ok");
        });
        NUI.on("projectList", (msg) => renderProjectList(msg.names || []));
        NUI.on("projectDeleted", (msg) => { toast(`Deleted "${msg.name}"`, "info"); openProjectList(); });
    }

    // ═══════════════════════════════════════════════════════════
    //  DATA LOADING
    // ═══════════════════════════════════════════════════════════
    async function loadData() {
        const status = $("#loadingText");
        const anim = { categories: {}, animations: {} };
        let objects = [];
        let done = 0; const totalN = ANIM_CHUNKS + OBJ_CHUNKS;
        const tick = () => { done++; status.textContent = `Loading library… ${Math.round(done / totalN * 100)}%`; };
        const animJobs = Array.from({ length: ANIM_CHUNKS }, (_, i) => NUI.fetchJSON(`data/anim_chunk_${i}.json`).then(c => { if (c.animations) Object.assign(anim.animations, c.animations); tick(); }).catch(e => { console.error("[AnimTool] anim chunk", i, e); tick(); }));
        const objJobs = Array.from({ length: OBJ_CHUNKS }, (_, i) => NUI.fetchJSON(`data/obj_chunk_${i}.json`).then(a => { objects = objects.concat(a); tick(); }).catch(e => { console.error("[AnimTool] obj chunk", i, e); tick(); }));
        await Promise.all([...animJobs, ...objJobs]);
        objects.sort();
        return { anim, objects };
    }

    // ═══════════════════════════════════════════════════════════
    //  BOOT
    // ═══════════════════════════════════════════════════════════
    async function init() {
        elApp = $("#app"); elToasts = $("#toasts"); elProjectName = $("#projectName");
        elDropdown = $("#projectListDropdown"); elDropItems = $("#projectListItems"); elStatus = $("#statusText");

        Timeline.init($("#timeline"));
        Inspector.init();
        Exporter.init();
        bindMessages();

        // topbar
        $("#saveProjectBtn").addEventListener("click", () => saveProject(false));
        $("#loadProjectBtn").addEventListener("click", () => { if (elDropdown.classList.contains("hidden")) openProjectList(); else elDropdown.classList.add("hidden"); });
        $("#newProjectBtn").addEventListener("click", newProject);
        $("#importBtn").addEventListener("click", openImport);
        $("#importDo").addEventListener("click", doImport);
        $("#importCancel").addEventListener("click", () => $("#importModal").classList.add("hidden"));
        $$("#importModal .modal-backdrop, #helpModal .modal-backdrop").forEach(b => b.addEventListener("click", () => b.closest(".modal").classList.add("hidden")));
        $("#helpBtn").addEventListener("click", () => $("#helpModal").classList.toggle("hidden"));
        $("#helpClose").addEventListener("click", () => $("#helpModal").classList.add("hidden"));
        $("#undoBtn").addEventListener("click", () => State.undo());
        $("#redoBtn").addEventListener("click", () => State.redo());
        $("#closeBtn").addEventListener("click", () => NUI.post("close"));
        elProjectName.addEventListener("change", () => { State.project.name = elProjectName.value.trim(); });
        elProjectName.addEventListener("keydown", (e) => { if (e.key === "Enter") { saveProject(false); elProjectName.blur(); } });
        document.addEventListener("mousedown", (e) => { if (!e.target.closest("#projectListDropdown") && !e.target.closest("#loadProjectBtn")) elDropdown.classList.add("hidden"); });

        // panel collapse handles
        $$(".panel-handle").forEach(h => h.addEventListener("click", () => { const p = h.closest(".panel"); p.classList.toggle("collapsed"); }));

        // camera orbit / world click on the transparent centre
        document.addEventListener("mousedown", (e) => {
            if (e.button !== 0) return;
            if (e.target === document.body || e.target === document.documentElement || e.target === elApp) {
                cameraHold = { x: e.clientX, y: e.clientY };
                NUI.post("cameraHold", { holding: true });
            }
        });
        document.addEventListener("mouseup", (e) => {
            if (!cameraHold) return;
            const moved = Math.hypot(e.clientX - cameraHold.x, e.clientY - cameraHold.y) > 4;
            cameraHold = null;
            NUI.post("cameraHold", { holding: false });
            if (!moved) NUI.post("clickWorld");
        });
        document.addEventListener("contextmenu", (e) => { if (!e.target.closest("input, textarea")) e.preventDefault(); });
        document.addEventListener("keydown", onKey);

        // state → sync / dirty / status
        State.subscribe((reason) => {
            if (reason === "project") { scheduleSync(); snapshotRecovery(); updateDirty(); updateStatus(); }
            else if (reason === "ui") updateDirty();
            else if (reason === "selection") updateStatus();
        });

        // autosave
        setInterval(() => { if (uiOpen && State.ui.dirty && State.ui.savedName && elProjectName.value.trim() === State.ui.savedName) saveProject(true); }, AUTOSAVE_MS);

        // library
        const { anim, objects } = await loadData();
        Browser.init(anim, objects);
        $("#loading").classList.add("hidden");
        updateStatus();
        updateDirty();
        checkRecovery();
        NUI.post("ready");
    }

    function updateStatus() {
        const s = State.ui.sel;
        let txt = `${Browser.dictCount ? Browser.dictCount().toLocaleString() : "…"} dictionaries`;
        if (s.type === "clip") { const c = State.getClip(s.clipId); if (c) txt = `Clip · ${c.dict} › ${c.clip}`; }
        else if (s.propId != null) { const p = State.getProp(s.propId); if (p) txt = `Prop · ${p.model}${s.type === "kf" ? " · keyframe" : ""}`; }
        elStatus.textContent = txt;
    }

    document.addEventListener("DOMContentLoaded", init);

    return { addClip, deleteClip, deleteProp, deleteSelection, copyTransform, pasteTransform, toast, confirm, saveProject, syncNow };
})();
