/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — inspector.js
   Right panel: prop list (+ add dialog) and a context inspector for
   the current selection (project / clip / prop+keyframes).
   ════════════════════════════════════════════════════════════════════ */
"use strict";

const Inspector = (() => {
    const STEP_LEVELS = [
        { label: "X-Fine", pos: 0.001, rot: 0.1 },
        { label: "Fine",   pos: 0.005, rot: 0.5 },
        { label: "Medium", pos: 0.01,  rot: 1 },
        { label: "Coarse", pos: 0.05,  rot: 5 },
        { label: "Max",    pos: 0.1,   rot: 10 },
    ];
    const AXES = ["px", "py", "pz", "rx", "ry", "rz"];

    let elPropList, elPropCount, elAddDialog, elModelInput, elSuggest, elContext;
    let propEd = null;            // prop editor DOM refs
    let clipEd = null;
    let stepIdx = 2;
    let activeSlider = null;      // axis being dragged (don't overwrite its value)
    let hintTimer = null;
    let sugIdx = -1;

    // ═══════════════════════════════════════════════════════════════
    //  INIT
    // ═══════════════════════════════════════════════════════════════
    function init() {
        elPropList = $("#propList"); elPropCount = $("#propCount");
        elAddDialog = $("#addPropDialog"); elModelInput = $("#propModelInput"); elSuggest = $("#propSuggestions");
        elContext = $("#inspectorContext");

        $("#addPropBtn").addEventListener("click", () => openAddDialog());
        $("#cancelAddProp").addEventListener("click", closeAddDialog);
        $("#confirmAddProp").addEventListener("click", confirmAdd);
        elModelInput.addEventListener("input", debounce(renderSuggestions, 120));
        elModelInput.addEventListener("keydown", (e) => {
            const items = $$(".prop-sug-item", elSuggest);
            if (e.key === "ArrowDown") { e.preventDefault(); sugIdx = Math.min(items.length - 1, sugIdx + 1); highlightSug(items); }
            else if (e.key === "ArrowUp") { e.preventDefault(); sugIdx = Math.max(-1, sugIdx - 1); highlightSug(items); }
            else if (e.key === "Enter") { e.preventDefault(); if (sugIdx >= 0 && items[sugIdx]) elModelInput.value = items[sugIdx].dataset.model; confirmAdd(); }
            else if (e.key === "Escape") { e.stopPropagation(); closeAddDialog(); }
        });

        buildPropEditor();
        buildClipEditor();

        State.subscribe((reason) => {
            if (reason === "project" || reason === "selection") render();
            else if (reason === "time") onTime();
            else if (reason === "ui") { updateKfStatus(); }
        });
        render();
    }

    // ═══════════════════════════════════════════════════════════════
    //  ADD PROP DIALOG
    // ═══════════════════════════════════════════════════════════════
    function openAddDialog() {
        elAddDialog.classList.remove("hidden");
        elModelInput.value = ""; elSuggest.innerHTML = ""; sugIdx = -1;
        elModelInput.focus();
    }
    function closeAddDialog() { elAddDialog.classList.add("hidden"); }
    function confirmAdd() {
        const model = elModelInput.value.trim();
        if (!model) return;
        const id = State.addProp(model);
        State.select({ type: "prop", propId: id });
        closeAddDialog();
        App.toast(`Prop added: ${model}`, "ok");
    }
    function renderSuggestions() {
        const term = elModelInput.value.trim().toLowerCase();
        elSuggest.innerHTML = ""; sugIdx = -1;
        if (!term) return;
        const words = term.split(/\s+/).filter(Boolean);
        const objs = Browser.getObjects();
        const out = [];
        for (const o of objs) {
            const lc = o.toLowerCase();
            if (words.every(w => lc.includes(w))) { out.push(o); if (out.length >= 60) break; }
        }
        out.sort((a, b) => (a.toLowerCase().startsWith(term) ? 0 : 1) - (b.toLowerCase().startsWith(term) ? 0 : 1) || a.length - b.length);
        if (!out.length) { elSuggest.appendChild(el("div", "list-empty", "No matching models — you can still spawn the typed name")); return; }
        for (const o of out) {
            const row = el("div", "prop-sug-item", esc(o)); row.dataset.model = o;
            row.addEventListener("click", () => { elModelInput.value = o; confirmAdd(); });
            elSuggest.appendChild(row);
        }
    }
    function highlightSug(items) { items.forEach((it, i) => it.classList.toggle("active", i === sugIdx)); if (items[sugIdx]) items[sugIdx].scrollIntoView({ block: "nearest" }); }

    // ═══════════════════════════════════════════════════════════════
    //  PROP LIST
    // ═══════════════════════════════════════════════════════════════
    function renderPropList() {
        const props = State.project.props;
        elPropCount.textContent = props.length;
        elPropList.innerHTML = "";
        if (!props.length) { elPropList.appendChild(el("div", "list-empty", "No props yet. Press + Add to spawn one on the player.")); return; }
        const sel = State.ui.sel;
        for (const p of props) {
            const parent = p.parentId != null ? State.getProp(p.parentId) : null;
            const row = el("div", "prop-item" + (sel.propId === p.id ? " selected" : "") + (p.enabled === false ? " disabled" : "") + (p.error ? " error" : ""),
                `<span class="prop-dot" style="background:${p.color}"></span>` +
                `<span class="prop-item-name" title="${esc(p.model)}">${esc(p.model)}</span>` +
                `<span class="prop-item-sub">${p.error ? "⚠ failed" : parent ? "→ " + esc(parent.model) : esc(p.bone)}${p.keyframes.length ? ` · ${p.keyframes.length}◆` : ""}</span>`);
            row.addEventListener("click", () => State.select({ type: "prop", propId: p.id }));
            elPropList.appendChild(row);
        }
    }

    // ═══════════════════════════════════════════════════════════════
    //  CONTEXT
    // ═══════════════════════════════════════════════════════════════
    function render() {
        renderPropList();
        const sel = State.ui.sel;
        propEd.root.classList.add("hidden"); clipEd.root.classList.add("hidden"); $("#projectInfo").classList.add("hidden");
        if ((sel.type === "prop" || sel.type === "kf") && State.selectedProp()) renderPropEditor();
        else if (sel.type === "clip" && State.selectedClip()) renderClipEditor();
        else renderProjectInfo();
    }

    function renderProjectInfo() {
        const box = $("#projectInfo"); box.classList.remove("hidden");
        const L = State.layout(); const p = State.project;
        const okClips = L.clips.filter(c => c.valid).length;
        $("#piDuration").textContent = fmtTime(L.total);
        $("#piClips").textContent = `${okClips}${okClips !== p.clips.length ? ` (${p.clips.length - okClips} pending/failed)` : ""}`;
        $("#piProps").textContent = String(p.props.length);
        $("#piKfs").textContent = String(p.props.reduce((n, x) => n + x.keyframes.length, 0));
    }

    // ── Clip editor ──────────────────────────────────────────────
    function buildClipEditor() {
        const root = $("#clipEditor");
        root.innerHTML = `
            <div class="insp-title"><span class="insp-kind">CLIP</span><span id="ceName" class="insp-name"></span></div>
            <div class="insp-dict" id="ceDict"></div>
            <div class="insp-row"><label>Native</label><span id="ceNative" class="insp-val"></span></div>
            <div class="insp-row"><label>Trim in</label><input id="ceIn" type="number" step="0.01" min="0"><span class="unit">s</span></div>
            <div class="insp-row"><label>Trim out</label><input id="ceOut" type="number" step="0.01" min="0"><span class="unit">s</span></div>
            <div class="insp-row"><label>Hold</label><input id="ceHold" type="number" step="0.05" min="0"><span class="unit">s</span><span class="insp-hint">keep last frame</span></div>
            <div class="insp-row"><label>Length</label><span id="ceLen" class="insp-val"></span></div>
            <div class="insp-row"><label>Starts at</label><span id="ceStart" class="insp-val"></span></div>
            <div class="insp-actions">
                <button id="cePreview" class="btn btn-ghost" title="Preview this clip on its own (looping)">▶ Preview</button>
                <button id="ceReset" class="btn btn-ghost" title="Reset trim and hold">Reset trim</button>
                <button id="ceDup" class="btn btn-ghost" title="Duplicate (Ctrl+D)">Duplicate</button>
                <button id="ceUp" class="btn btn-ghost" title="Move earlier">◀</button>
                <button id="ceDown" class="btn btn-ghost" title="Move later">▶</button>
                <button id="ceToggle" class="btn btn-ghost" title="Enable / disable (skipped in playback and export)">Disable</button>
                <button id="ceDel" class="btn btn-danger" title="Remove from timeline (Delete)">Remove</button>
            </div>`;
        const ref = (id) => $("#" + id, root);
        clipEd = { root, name: ref("ceName"), dict: ref("ceDict"), native: ref("ceNative"), tin: ref("ceIn"), tout: ref("ceOut"), hold: ref("ceHold"), len: ref("ceLen"), start: ref("ceStart"), toggle: ref("ceToggle") };
        const commit = (field, elx, min, maxFn) => elx.addEventListener("change", () => {
            const c = State.selectedClip(); if (!c) return;
            let v = parseFloat(elx.value); if (isNaN(v)) v = 0;
            v = clampNum(v, min, maxFn(c));
            State.setClip(c.id, { [field]: v });
        });
        commit("trimIn", clipEd.tin, 0, c => c.trimOut - State.MIN_CLIP_LEN);
        commit("trimOut", clipEd.tout, 0, c => c.duration);
        commit("hold", clipEd.hold, 0, () => 600);
        ref("cePreview").addEventListener("click", () => { const c = State.selectedClip(); if (c) Browser.preview(c.dict, c.clip); });
        ref("ceReset").addEventListener("click", () => { const c = State.selectedClip(); if (c) State.setClip(c.id, { trimIn: 0, trimOut: c.duration, hold: 0 }); });
        ref("ceDup").addEventListener("click", () => { const c = State.selectedClip(); if (c) { const id = State.duplicateClip(c.id); State.select({ type: "clip", clipId: id }); } });
        ref("ceUp").addEventListener("click", () => { const c = State.selectedClip(); if (c) State.moveClip(c.id, State.clipIndex(c.id) - 1); });
        ref("ceDown").addEventListener("click", () => { const c = State.selectedClip(); if (c) State.moveClip(c.id, State.clipIndex(c.id) + 1); });
        clipEd.toggle.addEventListener("click", () => { const c = State.selectedClip(); if (c) State.setClip(c.id, { enabled: c.enabled === false }); });
        ref("ceDel").addEventListener("click", () => { const c = State.selectedClip(); if (c) App.deleteClip(c.id); });
    }

    function renderClipEditor() {
        const c = State.selectedClip(); const e = clipEd;
        e.root.classList.remove("hidden");
        e.name.textContent = c.clip; e.dict.textContent = c.dict; e.dict.title = c.dict;
        e.native.textContent = c.status === "ok" ? fmtSec(c.duration, 3) + (c.estimated ? " (estimated)" : "") : c.status === "pending" ? "resolving…" : "⚠ " + (c.error || "failed");
        const disabled = c.status !== "ok";
        [e.tin, e.tout, e.hold].forEach(i => i.disabled = disabled);
        if (document.activeElement !== e.tin) e.tin.value = c.trimIn.toFixed(2);
        if (document.activeElement !== e.tout) e.tout.value = c.trimOut.toFixed(2);
        if (document.activeElement !== e.hold) e.hold.value = c.hold.toFixed(2);
        e.tin.max = Math.max(0, c.trimOut - State.MIN_CLIP_LEN).toFixed(2); e.tout.max = c.duration.toFixed(2);
        const L = State.layout().clips.find(x => x.clip.id === c.id);
        e.len.textContent = L && L.valid ? fmtSec(L.length, 3) : "—";
        e.start.textContent = L && L.valid ? fmtTime(L.start) : "—";
        e.toggle.textContent = c.enabled === false ? "Enable" : "Disable";
        e.root.classList.toggle("is-disabled", c.enabled === false);
    }

    // ── Prop editor ──────────────────────────────────────────────
    function buildPropEditor() {
        const root = $("#propEditor");
        const slider = (axis, label, min, max, step) => `
            <div class="slider-row" data-axis="${axis}">
                <label>${label}</label>
                <input type="range" min="${min}" max="${max}" step="${step}" value="0">
                <input type="number" step="${step}" value="0">
            </div>`;
        root.innerHTML = `
            <div class="insp-title"><span class="prop-dot" id="peDot"></span><span id="peName" class="insp-name"></span>
                <span class="insp-title-btns"><button id="peToggle" class="mini" title="Enable / disable">◉</button><button id="peDup" class="mini" title="Duplicate prop">⧉</button><button id="peDel" class="mini del" title="Delete prop (Delete)">✕</button></span></div>
            <div id="peError" class="insp-error hidden"></div>
            <div class="insp-row"><label>Attach to</label><select id="peParent"></select></div>
            <div class="insp-row" id="peBoneRow"><label>Bone</label><select id="peBone"></select></div>
            <div class="insp-row"><label>Scale</label><input id="peScaleRange" type="range" min="0.05" max="3" step="0.01" value="1"><input id="peScale" type="number" step="0.01" min="0.01" max="10" value="1"></div>
            <div class="insp-row"><label>Range</label>
                <input id="peIn" type="number" step="0.01" min="0" title="In point (s)"><span class="unit">→</span>
                <input id="peOut" type="number" step="0.01" min="0" title="Out point (s)" placeholder="end"><button id="peOutEnd" class="mini" title="Out point follows the end of the timeline">end</button></div>
            <div class="insp-row"><label>Step</label><input id="peStep" type="range" min="0" max="4" step="1" value="2"><b id="peStepLabel">Medium</b></div>

            <div class="slider-section"><div class="section-label">Position <span class="insp-hint">metres, relative to bone</span></div>
                ${slider("px", "X", -2, 2, 0.01)}${slider("py", "Y", -2, 2, 0.01)}${slider("pz", "Z", -2, 2, 0.01)}</div>
            <div class="slider-section"><div class="section-label">Rotation <span class="insp-hint">degrees</span></div>
                ${slider("rx", "X", -180, 180, 1)}${slider("ry", "Y", -180, 180, 1)}${slider("rz", "Z", -180, 180, 1)}</div>

            <div class="kf-section">
                <div class="section-label">Keyframes <span id="peKfCount" class="badge">0</span></div>
                <div id="peKfStatus" class="kf-status"></div>
                <div class="kf-actions">
                    <button id="peKfAdd" class="btn btn-accent" title="Add / update keyframe at playhead (K)">◆ Keyframe</button>
                    <button id="peKfPrev" class="mini" title="Previous keyframe">◀</button>
                    <button id="peKfNext" class="mini" title="Next keyframe">▶</button>
                    <button id="peKfDel" class="btn btn-ghost" title="Delete keyframe at playhead">Delete</button>
                    <select id="peKfEase" title="Easing from this keyframe to the next">
                        <option value="linear">linear</option><option value="smooth">smooth</option><option value="in">ease in</option><option value="out">ease out</option><option value="hold">hold</option>
                    </select>
                </div>
                <div class="kf-actions">
                    <button id="peKfCopy" class="btn btn-ghost" title="Copy transform (Ctrl+C)">Copy</button>
                    <button id="peKfPaste" class="btn btn-ghost" title="Paste as keyframe at playhead (Ctrl+V)">Paste</button>
                    <button id="peKfClear" class="btn btn-ghost" title="Remove all keyframes, keeping the current pose">Clear all</button>
                </div>
                <div id="peKfList" class="kf-list"></div>
            </div>`;

        const ref = (id) => $("#" + id, root);
        propEd = {
            root, dot: ref("peDot"), name: ref("peName"), error: ref("peError"), parent: ref("peParent"), bone: ref("peBone"), boneRow: ref("peBoneRow"),
            scaleRange: ref("peScaleRange"), scale: ref("peScale"), tin: ref("peIn"), tout: ref("peOut"), outEnd: ref("peOutEnd"),
            step: ref("peStep"), stepLabel: ref("peStepLabel"), kfCount: ref("peKfCount"), kfStatus: ref("peKfStatus"),
            kfAdd: ref("peKfAdd"), kfDel: ref("peKfDel"), kfEase: ref("peKfEase"), kfList: ref("peKfList"), toggle: ref("peToggle"),
            sliders: {},
        };
        for (const axis of AXES) {
            const row = $(`.slider-row[data-axis="${axis}"]`, root);
            propEd.sliders[axis] = { range: $("input[type=range]", row), num: $("input[type=number]", row) };
        }

        ref("peDel").addEventListener("click", () => { const p = State.selectedProp(); if (p) App.deleteProp(p.id); });
        ref("peDup").addEventListener("click", () => { const p = State.selectedProp(); if (p) { const id = State.duplicateProp(p.id); State.select({ type: "prop", propId: id }); } });
        propEd.toggle.addEventListener("click", () => { const p = State.selectedProp(); if (p) State.setProp(p.id, { enabled: p.enabled === false }); });
        propEd.parent.addEventListener("change", () => { const p = State.selectedProp(); if (!p) return; const v = parseInt(propEd.parent.value); State.setProp(p.id, { parentId: v > 0 ? v : null }); });
        propEd.bone.addEventListener("change", () => { const p = State.selectedProp(); if (p) State.setProp(p.id, { bone: propEd.bone.value }); });
        const scaleIn = (src) => { const p = State.selectedProp(); if (!p) return; const v = clampNum(parseFloat(src.value) || 1, 0.01, 10); propEd.scaleRange.value = v; propEd.scale.value = v; State.setProp(p.id, { scale: v }, { coalesce: "scale-" + p.id }); };
        propEd.scaleRange.addEventListener("input", () => scaleIn(propEd.scaleRange));
        propEd.scale.addEventListener("input", () => scaleIn(propEd.scale));
        propEd.tin.addEventListener("change", () => { const p = State.selectedProp(); if (!p) return; const v = Math.max(0, parseFloat(propEd.tin.value) || 0); State.setProp(p.id, { inTime: v }); });
        propEd.tout.addEventListener("change", () => { const p = State.selectedProp(); if (!p) return; const raw = propEd.tout.value.trim(); if (raw === "") { State.setProp(p.id, { outTime: null }); return; } const v = parseFloat(raw); State.setProp(p.id, { outTime: isNaN(v) ? null : Math.max(p.inTime + State.MIN_CLIP_LEN, v) }); });
        propEd.outEnd.addEventListener("click", () => { const p = State.selectedProp(); if (p) State.setProp(p.id, { outTime: null }); });
        propEd.step.addEventListener("input", () => { stepIdx = parseInt(propEd.step.value); applyStep(); });

        for (const axis of AXES) {
            const s = propEd.sliders[axis];
            const onInput = (src) => {
                const p = State.selectedProp(); if (!p) return;
                let v = parseFloat(src.value); if (isNaN(v)) return;
                if (src === s.num) s.range.value = clampNum(v, +s.range.min, +s.range.max); else s.num.value = v;
                const tr = readTransform();
                const res = State.setTransformAt(p.id, State.ui.time, tr, { coalesce: "slider-" + axis + "-" + p.id, reason: "transform" });
                if (res === "blocked") flashHint("Auto-key is off and there is no keyframe here — press ◆ Keyframe first, or enable Auto-key.");
            };
            s.range.addEventListener("mousedown", () => { activeSlider = axis; });
            s.range.addEventListener("input", () => onInput(s.range));
            s.num.addEventListener("focus", () => { activeSlider = axis; });
            s.num.addEventListener("blur", () => { if (activeSlider === axis) activeSlider = null; onTime(); });
            s.num.addEventListener("input", () => onInput(s.num));
        }
        document.addEventListener("mouseup", () => { if (activeSlider && document.activeElement !== propEd.sliders[activeSlider]?.num) { activeSlider = null; onTime(); } });

        propEd.kfAdd.addEventListener("click", () => addKeyframeHere());
        propEd.kfDel.addEventListener("click", () => deleteKeyframeHere());
        ref("peKfPrev").addEventListener("click", () => jumpKf(-1));
        ref("peKfNext").addEventListener("click", () => jumpKf(1));
        propEd.kfEase.addEventListener("change", () => { const p = State.selectedProp(); const k = p && State.kfAt(p, State.ui.time); if (k) State.setKeyframe(p.id, k.id, { ease: propEd.kfEase.value }); });
        ref("peKfCopy").addEventListener("click", () => App.copyTransform());
        ref("peKfPaste").addEventListener("click", () => App.pasteTransform());
        ref("peKfClear").addEventListener("click", () => { const p = State.selectedProp(); if (p && p.keyframes.length) { State.clearKeyframes(p.id); App.toast("Keyframes cleared (pose kept)", "info"); } });
        applyStep();
    }

    function applyStep() {
        const lvl = STEP_LEVELS[stepIdx];
        propEd.stepLabel.textContent = lvl.label;
        for (const axis of AXES) { const s = propEd.sliders[axis]; const st = axis[0] === "p" ? lvl.pos : lvl.rot; s.range.step = st; s.num.step = st; }
    }

    function readTransform() {
        const v = (a) => parseFloat(propEd.sliders[a].num.value) || 0;
        return { pos: { x: v("px"), y: v("py"), z: v("pz") }, rot: { x: v("rx"), y: v("ry"), z: v("rz") } };
    }

    function writeTransform(tr) {
        const set = (axis, val) => {
            if (activeSlider === axis) return;
            const s = propEd.sliders[axis];
            const r = Math.round(val * 10000) / 10000;
            s.num.value = r; s.range.value = clampNum(r, +s.range.min, +s.range.max);
        };
        set("px", tr.pos.x); set("py", tr.pos.y); set("pz", tr.pos.z); set("rx", tr.rot.x); set("ry", tr.rot.y); set("rz", tr.rot.z);
    }

    function renderPropEditor() {
        const p = State.selectedProp(); const e = propEd;
        e.root.classList.remove("hidden");
        e.dot.style.background = p.color; e.name.textContent = p.model; e.name.title = p.model;
        e.root.classList.toggle("is-disabled", p.enabled === false);
        e.toggle.textContent = p.enabled === false ? "◌" : "◉";
        e.error.classList.toggle("hidden", !p.error); e.error.textContent = p.error ? "⚠ " + p.error : "";

        // parent options
        e.parent.innerHTML = '<option value="0">Player (bone)</option>';
        for (const o of State.project.props) {
            if (o.id === p.id || State.wouldCycle(p.id, o.id)) continue;
            const opt = document.createElement("option"); opt.value = o.id; opt.textContent = `Prop: ${o.model}`; e.parent.appendChild(opt);
        }
        e.parent.value = p.parentId != null ? String(p.parentId) : "0";
        e.boneRow.classList.toggle("hidden", p.parentId != null);
        if (e.bone.options.length && document.activeElement !== e.bone) e.bone.value = p.bone;
        if (document.activeElement !== e.scale) { e.scale.value = p.scale; e.scaleRange.value = p.scale; }
        if (document.activeElement !== e.tin) e.tin.value = p.inTime.toFixed(2);
        if (document.activeElement !== e.tout) e.tout.value = p.outTime == null ? "" : p.outTime.toFixed(2);
        e.outEnd.classList.toggle("on", p.outTime == null);

        writeTransform(State.evalProp(p, State.ui.time));
        updateKfStatus();
        renderKfList(p);
    }

    function updateKfStatus() {
        const p = State.selectedProp(); if (!p || propEd.root.classList.contains("hidden")) return;
        const e = propEd; const t = State.ui.time;
        const here = State.kfAt(p, t);
        e.kfCount.textContent = p.keyframes.length;
        e.kfDel.disabled = !here;
        e.kfEase.disabled = !here;
        if (here) e.kfEase.value = here.ease || "linear";
        e.kfAdd.textContent = here ? "◆ Update" : "◆ Keyframe";
        let msg, cls = "";
        if (!p.keyframes.length) { msg = "No keyframes — sliders set the prop's fixed pose."; }
        else if (here) { msg = `◆ On keyframe at ${fmtSec(here.t)} — sliders edit it.`; cls = "on-kf"; }
        else if (State.ui.autoKey) { msg = "Between keyframes — moving a slider adds a keyframe here (auto-key)."; cls = "auto"; }
        else { msg = "Between keyframes — auto-key is off, press ◆ to key this pose."; cls = "warn"; }
        e.kfStatus.textContent = msg; e.kfStatus.className = "kf-status " + cls;
        $$(".kf-row", e.kfList).forEach(r => r.classList.toggle("current", here && +r.dataset.kfId === here.id));
    }

    function renderKfList(p) {
        const e = propEd; e.kfList.innerHTML = "";
        const sel = State.ui.sel;
        p.keyframes.forEach((k, i) => {
            const row = el("div", "kf-row" + (sel.type === "kf" && sel.kfId === k.id ? " selected" : ""),
                `<span class="kf-idx">${i + 1}</span><span class="kf-time">${fmtSec(k.t, 2)}</span>` +
                `<span class="kf-info">${k.pos.x.toFixed(3)}, ${k.pos.y.toFixed(3)}, ${k.pos.z.toFixed(3)} · ${k.ease || "linear"}</span>` +
                `<button class="mini del" title="Delete keyframe">✕</button>`);
            row.dataset.kfId = k.id;
            row.addEventListener("click", (ev) => { if (ev.target.closest("button")) return; State.select({ type: "kf", propId: p.id, kfId: k.id }); Transport.seek(k.t, true); });
            row.querySelector(".del").addEventListener("click", (ev) => { ev.stopPropagation(); State.removeKeyframe(p.id, k.id); });
            e.kfList.appendChild(row);
        });
    }

    let timeRaf = null;
    function onTime() {
        if (timeRaf) return;
        timeRaf = requestAnimationFrame(() => {
            timeRaf = null;
            const p = State.selectedProp();
            if (!p || propEd.root.classList.contains("hidden")) return;
            if (p.keyframes.length) writeTransform(State.evalProp(p, State.ui.time));
            updateKfStatus();
        });
    }

    function addKeyframeHere() {
        const p = State.selectedProp(); if (!p) return;
        const tr = readTransform();
        const id = State.addKeyframe(p.id, State.ui.time, tr.pos, tr.rot);
        State.select({ type: "kf", propId: p.id, kfId: id });
    }
    function deleteKeyframeHere() {
        const p = State.selectedProp(); if (!p) return;
        const k = State.kfAt(p, State.ui.time); if (k) State.removeKeyframe(p.id, k.id);
    }
    function jumpKf(dir) {
        const p = State.selectedProp(); if (!p || !p.keyframes.length) return;
        const t = State.ui.time;
        const next = dir > 0 ? p.keyframes.find(k => k.t > t + State.KF_EPS) : [...p.keyframes].reverse().find(k => k.t < t - State.KF_EPS);
        if (next) { State.select({ type: "kf", propId: p.id, kfId: next.id }); Transport.seek(next.t, true); }
    }

    function flashHint(msg) { App.toast(msg, "warn"); }

    function setBones(bones) {
        propEd.bone.innerHTML = "";
        for (const b of bones) { const o = document.createElement("option"); o.value = b; o.textContent = b; propEd.bone.appendChild(o); }
        const p = State.selectedProp(); if (p) propEd.bone.value = p.bone;
    }

    function setStep(delta) { stepIdx = clampNum(stepIdx + delta, 0, STEP_LEVELS.length - 1); propEd.step.value = stepIdx; applyStep(); App.toast("Step: " + STEP_LEVELS[stepIdx].label, "info"); }

    return { init, render, setBones, addKeyframeHere, deleteKeyframeHere, jumpKf, readTransform, openAddDialog, closeAddDialog, setStep, isAddDialogOpen: () => !elAddDialog.classList.contains("hidden") };
})();
