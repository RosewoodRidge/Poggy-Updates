/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — state.js
   Single source of truth for the project (clips + props + keyframes),
   UI state (time, selection), undo/redo and (de)serialisation.
   Every edit goes through State.update() so it is undoable and
   triggers one 'project' change event.
   ════════════════════════════════════════════════════════════════════ */
"use strict";

const State = (() => {
    const PROJECT_VERSION = 2;
    const PROP_COLORS = ["#f0883e", "#a371f7", "#3fb950", "#db61a2", "#38ada9", "#daba37", "#ff7b72", "#79c0ff"];
    const MIN_CLIP_LEN = 0.05;
    const KF_EPS = 0.012;   // keyframes closer than this count as "the same time"

    // ── project ────────────────────────────────────────────────────
    let project = blankProject();

    // ── ui state (not undoable) ─────────────────────────────────────
    const ui = {
        time: 0, playing: false, speed: 1.0, scrubbing: false,
        autoKey: true, snap: true,
        preview: null,                    // { dict, clip, duration } while previewing from the browser
        sel: { type: null, clipId: null, propId: null, kfId: null },
        dirty: false,
        savedName: "",
    };

    // ── undo/redo ───────────────────────────────────────────────────
    const undoStack = [], redoStack = [];
    const UNDO_LIMIT = 80;
    let lastCoalesce = null;   // { key, at }

    const listeners = [];
    function subscribe(fn) { listeners.push(fn); return () => { const i = listeners.indexOf(fn); if (i >= 0) listeners.splice(i, 1); }; }
    function emit(reason, detail) { for (const fn of listeners) { try { fn(reason, detail); } catch (e) { console.error("[AnimTool] listener", reason, e); } } }

    function blankProject() {
        return { version: PROJECT_VERSION, name: "", loop: false, clips: [], props: [], nextId: 1 };
    }

    const clone = (o) => JSON.parse(JSON.stringify(o));
    const clamp = (v, lo, hi) => Math.max(lo, Math.min(hi, v));
    const num = (v, d = 0) => (typeof v === "number" && isFinite(v)) ? v : d;
    const vec = (v) => ({ x: num(v && v.x), y: num(v && v.y), z: num(v && v.z) });

    function nextId() { return project.nextId++; }

    // ═══════════════════════════════════════════════════════════════
    //  UPDATE / UNDO
    // ═══════════════════════════════════════════════════════════════
    /**
     * Apply a mutation. opts.coalesce = key: consecutive updates with the same
     * key within 800ms share one undo step (slider drags, keyframe drags).
     */
    function update(fn, opts = {}) {
        const now = Date.now();
        const coalesce = opts.coalesce && lastCoalesce && lastCoalesce.key === opts.coalesce && (now - lastCoalesce.at) < 800;
        if (!coalesce) {
            undoStack.push(clone(project));
            if (undoStack.length > UNDO_LIMIT) undoStack.shift();
            redoStack.length = 0;
        }
        lastCoalesce = opts.coalesce ? { key: opts.coalesce, at: now } : null;
        const result = fn(project);
        normalise();
        ui.dirty = true;
        emit("project", opts.reason || "edit");
        return result;
    }

    function undo() {
        if (!undoStack.length) return false;
        redoStack.push(clone(project));
        project = undoStack.pop();
        lastCoalesce = null;
        normalise(); pruneSelection(); ui.dirty = true;
        emit("project", "undo"); emit("selection");
        return true;
    }

    function redo() {
        if (!redoStack.length) return false;
        undoStack.push(clone(project));
        project = redoStack.pop();
        lastCoalesce = null;
        normalise(); pruneSelection(); ui.dirty = true;
        emit("project", "redo"); emit("selection");
        return true;
    }

    /** Keep invariants: sorted keyframes, clamped trims, valid parents */
    function normalise() {
        for (const c of project.clips) {
            c.duration = Math.max(0, num(c.duration));
            c.trimIn = clamp(num(c.trimIn), 0, c.duration);
            c.trimOut = c.trimOut == null ? c.duration : clamp(num(c.trimOut, c.duration), c.trimIn, c.duration);
            if (c.duration > 0 && c.trimOut - c.trimIn < MIN_CLIP_LEN) {
                c.trimOut = Math.min(c.duration, c.trimIn + MIN_CLIP_LEN);
                c.trimIn = Math.max(0, c.trimOut - MIN_CLIP_LEN);
            }
            c.hold = Math.max(0, num(c.hold));
        }
        const ids = new Set(project.props.map(p => p.id));
        for (const p of project.props) {
            if (p.parentId != null && (!ids.has(p.parentId) || p.parentId === p.id || wouldCycle(p.id, p.parentId))) p.parentId = null;
            p.keyframes.sort((a, b) => a.t - b.t);
            p.inTime = Math.max(0, num(p.inTime));
            if (p.outTime != null && p.outTime < p.inTime + MIN_CLIP_LEN) p.outTime = p.inTime + MIN_CLIP_LEN;
            p.scale = clamp(num(p.scale, 1), 0.01, 10);
        }
    }

    function pruneSelection() {
        const s = ui.sel;
        if (s.clipId != null && !getClip(s.clipId)) select(null);
        else if (s.propId != null && !getProp(s.propId)) select(null);
        else if (s.kfId != null) { const p = getProp(s.propId); if (!p || !p.keyframes.find(k => k.id === s.kfId)) select({ type: "prop", propId: s.propId }); }
    }

    // ═══════════════════════════════════════════════════════════════
    //  LAYOUT  (clips laid back-to-back; pending/failed clips are skipped)
    // ═══════════════════════════════════════════════════════════════
    function layoutClip(c) {
        const animLen = Math.max(0, c.trimOut - c.trimIn);
        return { animLen, length: animLen + c.hold, valid: c.enabled !== false && c.status === "ok" && c.duration > 0.0005 && (animLen + c.hold) > 0.0005 };
    }

    function layout() {
        let t = 0;
        const clips = project.clips.map(c => {
            const l = layoutClip(c);
            const entry = { clip: c, start: t, length: l.valid ? l.length : 0, animLen: l.animLen, valid: l.valid };
            if (l.valid) t += l.length;
            return entry;
        });
        return { clips, total: t };
    }

    function totalDuration() { return layout().total; }

    function clipAt(t) {
        const L = layout();
        let found = null;
        for (const e of L.clips) if (e.valid && t >= e.start) found = e;
        return found;
    }

    /** Clip boundaries (for snapping / prev-next navigation) */
    function boundaries() {
        const L = layout(); const out = [0];
        for (const e of L.clips) if (e.valid) out.push(e.start + e.length);
        return out;
    }

    // ═══════════════════════════════════════════════════════════════
    //  CLIPS
    // ═══════════════════════════════════════════════════════════════
    function getClip(id) { return project.clips.find(c => c.id === id) || null; }
    function clipIndex(id) { return project.clips.findIndex(c => c.id === id); }

    function addClip(dict, clip, index) {
        return update(p => {
            const c = { id: nextId(), dict, clip, duration: 0, trimIn: 0, trimOut: 0, hold: 0, enabled: true, status: "pending", error: null, estimated: false };
            if (index == null || index < 0 || index > p.clips.length) p.clips.push(c); else p.clips.splice(index, 0, c);
            return c.id;
        }, { reason: "addClip" });
    }

    function resolveClip(id, ok, duration, error, estimated) {
        const c = getClip(id); if (!c) return;
        // Resolution is not a user edit: no undo entry
        if (ok) {
            const trimWasFull = c.trimOut === 0 || c.trimOut >= c.duration;
            c.duration = duration; c.status = "ok"; c.error = null; c.estimated = !!estimated;
            if (trimWasFull) c.trimOut = duration;
            c.trimIn = clamp(c.trimIn, 0, duration); c.trimOut = clamp(c.trimOut, c.trimIn, duration);
        } else { c.status = "error"; c.error = error || "Failed"; }
        // patch undo snapshots so undoing an unrelated edit doesn't resurrect "pending"
        for (const snap of undoStack) { const sc = snap.clips.find(x => x.id === id); if (sc) Object.assign(sc, { duration: c.duration, trimOut: c.trimOut, status: c.status, error: c.error, estimated: c.estimated }); }
        emit("project", "resolve");
    }

    function removeClip(id) { update(p => { p.clips = p.clips.filter(c => c.id !== id); }, { reason: "removeClip" }); }
    function moveClip(id, toIndex) {
        update(p => {
            const from = p.clips.findIndex(c => c.id === id); if (from < 0) return;
            const [c] = p.clips.splice(from, 1);
            p.clips.splice(clamp(toIndex, 0, p.clips.length), 0, c);
        }, { reason: "moveClip" });
    }
    function duplicateClip(id) {
        return update(p => {
            const i = p.clips.findIndex(c => c.id === id); if (i < 0) return null;
            const c = clone(p.clips[i]); c.id = nextId(); p.clips.splice(i + 1, 0, c); return c.id;
        }, { reason: "dupClip" });
    }
    function setClip(id, fields, opts) { update(p => { const c = p.clips.find(x => x.id === id); if (c) Object.assign(c, fields); }, opts); }

    // ═══════════════════════════════════════════════════════════════
    //  PROPS
    // ═══════════════════════════════════════════════════════════════
    function getProp(id) { return project.props.find(p => p.id === id) || null; }

    function addProp(model) {
        return update(p => {
            const prop = {
                id: nextId(), model, bone: "PH_R_Hand", parentId: null, scale: 1, enabled: true,
                inTime: 0, outTime: null, pos: { x: 0, y: 0, z: 0 }, rot: { x: 0, y: 0, z: 0 },
                keyframes: [], color: PROP_COLORS[p.props.length % PROP_COLORS.length], error: null,
            };
            p.props.push(prop); return prop.id;
        }, { reason: "addProp" });
    }
    function removeProp(id) {
        update(p => {
            p.props = p.props.filter(x => x.id !== id);
            for (const x of p.props) if (x.parentId === id) x.parentId = null;
        }, { reason: "removeProp" });
    }
    function duplicateProp(id) {
        return update(p => {
            const i = p.props.findIndex(x => x.id === id); if (i < 0) return null;
            const c = clone(p.props[i]); c.id = nextId(); c.color = PROP_COLORS[p.props.length % PROP_COLORS.length];
            for (const k of c.keyframes) k.id = nextId();
            p.props.splice(i + 1, 0, c); return c.id;
        }, { reason: "dupProp" });
    }
    function setProp(id, fields, opts) { update(p => { const x = p.props.find(y => y.id === id); if (x) Object.assign(x, fields); }, opts); }
    function setPropError(id, message) { const p = getProp(id); if (p) { p.error = message; emit("project", "propError"); } }

    function wouldCycle(childId, newParentId) {
        let cur = newParentId; const seen = new Set();
        while (cur != null) {
            if (cur === childId || seen.has(cur)) return true;
            seen.add(cur);
            const p = getProp(cur); if (!p) break; cur = p.parentId;
        }
        return false;
    }

    // ═══════════════════════════════════════════════════════════════
    //  KEYFRAMES
    // ═══════════════════════════════════════════════════════════════
    function kfAt(prop, t) { return prop.keyframes.find(k => Math.abs(k.t - t) < KF_EPS) || null; }

    /** Transform at time t (mirrors ScenePlayer:_evalTransform) */
    function evalProp(prop, t) {
        const kfs = prop.keyframes;
        if (!kfs.length) return { pos: vec(prop.pos), rot: vec(prop.rot) };
        if (t <= kfs[0].t || kfs.length === 1) return { pos: vec(kfs[0].pos), rot: vec(kfs[0].rot) };
        const last = kfs[kfs.length - 1];
        if (t >= last.t) return { pos: vec(last.pos), rot: vec(last.rot) };
        for (let i = 0; i < kfs.length - 1; i++) {
            const a = kfs[i], b = kfs[i + 1];
            if (t >= a.t && t <= b.t) {
                const span = b.t - a.t; let u = span > 0 ? (t - a.t) / span : 1;
                u = ease(a.ease, u);
                return {
                    pos: { x: lerp(a.pos.x, b.pos.x, u), y: lerp(a.pos.y, b.pos.y, u), z: lerp(a.pos.z, b.pos.z, u) },
                    rot: { x: lerpAngle(a.rot.x, b.rot.x, u), y: lerpAngle(a.rot.y, b.rot.y, u), z: lerpAngle(a.rot.z, b.rot.z, u) },
                };
            }
        }
        return { pos: vec(last.pos), rot: vec(last.rot) };
    }
    const lerp = (a, b, u) => a + (b - a) * u;
    function lerpAngle(a, b, u) { let d = (b - a) % 360; if (d > 180) d -= 360; if (d < -180) d += 360; return a + d * u; }
    function ease(kind, u) {
        switch (kind) {
            case "smooth": return u * u * (3 - 2 * u);
            case "in": return u * u;
            case "out": return 1 - (1 - u) * (1 - u);
            case "hold": return 0;
            default: return u;
        }
    }

    function addKeyframe(propId, t, pos, rot, opts) {
        return update(p => {
            const prop = p.props.find(x => x.id === propId); if (!prop) return null;
            const cur = pos && rot ? { pos: vec(pos), rot: vec(rot) } : evalProp(prop, t);
            const existing = kfAt(prop, t);
            if (existing) { existing.pos = cur.pos; existing.rot = cur.rot; return existing.id; }
            const kf = { id: nextId(), t, pos: cur.pos, rot: cur.rot, ease: "linear" };
            prop.keyframes.push(kf); return kf.id;
        }, Object.assign({ reason: "addKf" }, opts || {}));
    }

    /** Write a transform at time t: updates the keyframe there, or creates one (auto-key), or edits base when no keyframes */
    function setTransformAt(propId, t, tr, opts) {
        const prop = getProp(propId); if (!prop) return "none";
        if (!prop.keyframes.length) { setProp(propId, { pos: vec(tr.pos), rot: vec(tr.rot) }, opts); return "base"; }
        const existing = kfAt(prop, t);
        if (existing) { update(p => { const k = p.props.find(x => x.id === propId).keyframes.find(x => x.id === existing.id); k.pos = vec(tr.pos); k.rot = vec(tr.rot); }, opts); return "kf"; }
        if (!ui.autoKey) return "blocked";
        addKeyframe(propId, t, tr.pos, tr.rot, opts); return "new";
    }

    function removeKeyframe(propId, kfId) { update(p => { const prop = p.props.find(x => x.id === propId); if (prop) prop.keyframes = prop.keyframes.filter(k => k.id !== kfId); }, { reason: "removeKf" }); }
    function setKeyframe(propId, kfId, fields, opts) { update(p => { const prop = p.props.find(x => x.id === propId); const k = prop && prop.keyframes.find(x => x.id === kfId); if (k) Object.assign(k, fields); }, opts); }
    function clearKeyframes(propId) {
        update(p => { const prop = p.props.find(x => x.id === propId); if (!prop) return; const cur = evalProp(prop, ui.time); prop.pos = cur.pos; prop.rot = cur.rot; prop.keyframes = []; }, { reason: "clearKf" });
    }
    function shiftKeyframes(propId, dt) { update(p => { const prop = p.props.find(x => x.id === propId); if (prop) for (const k of prop.keyframes) k.t = Math.max(0, k.t + dt); }, { reason: "shiftKf" }); }

    // ═══════════════════════════════════════════════════════════════
    //  SELECTION / TIME
    // ═══════════════════════════════════════════════════════════════
    function select(sel) {
        const next = { type: null, clipId: null, propId: null, kfId: null };
        if (sel && sel.type === "clip") { next.type = "clip"; next.clipId = sel.clipId; }
        else if (sel && sel.type === "prop") { next.type = "prop"; next.propId = sel.propId; }
        else if (sel && sel.type === "kf") { next.type = "kf"; next.propId = sel.propId; next.kfId = sel.kfId; }
        const changed = JSON.stringify(next) !== JSON.stringify(ui.sel);
        ui.sel = next;
        if (changed) emit("selection", next);
    }
    function selectedProp() { return ui.sel.propId != null ? getProp(ui.sel.propId) : null; }
    function selectedClip() { return ui.sel.clipId != null ? getClip(ui.sel.clipId) : null; }
    function selectedKf() { const p = selectedProp(); return p && ui.sel.kfId != null ? p.keyframes.find(k => k.id === ui.sel.kfId) || null : null; }

    function setTime(t, from) { ui.time = clamp(num(t), 0, totalDuration()); emit("time", from); }
    function setPlaying(b) { if (ui.playing !== !!b) { ui.playing = !!b; emit("ui", "playing"); } }
    function setUI(fields) { Object.assign(ui, fields); emit("ui", Object.keys(fields).join(",")); }

    // ═══════════════════════════════════════════════════════════════
    //  SCENE (what Lua / export consume)
    // ═══════════════════════════════════════════════════════════════
    function toScene() {
        const L = layout();
        return {
            duration: L.total,
            loop: !!project.loop,
            clips: project.clips.map(c => ({
                dict: c.dict, clip: c.clip, duration: c.duration, trimIn: c.trimIn, trimOut: c.trimOut, hold: c.hold,
                enabled: c.enabled !== false && c.status === "ok",
            })),
            props: project.props.map(p => ({
                id: p.id, model: p.model, bone: p.bone, parent: p.parentId, scale: p.scale, enabled: p.enabled !== false,
                inTime: p.inTime, outTime: p.outTime, pos: vec(p.pos), rot: vec(p.rot),
                keyframes: p.keyframes.map(k => ({ t: k.t, pos: vec(k.pos), rot: vec(k.rot), ease: k.ease || "linear" })),
            })),
        };
    }

    // ═══════════════════════════════════════════════════════════════
    //  SERIALISE / LOAD
    // ═══════════════════════════════════════════════════════════════
    function serialize() { return JSON.stringify(project); }

    function loadProject(obj, name) {
        const p = migrate(obj);
        if (!p) return false;
        undoStack.length = 0; redoStack.length = 0; lastCoalesce = null;
        project = p;
        if (name != null) project.name = name;
        ui.savedName = project.name;
        normalise();
        ui.time = 0; ui.dirty = false;
        select(null);
        emit("project", "load"); emit("time", "load");
        return true;
    }

    function newProject() {
        undoStack.length = 0; redoStack.length = 0; lastCoalesce = null;
        project = blankProject(); ui.time = 0; ui.dirty = false; ui.savedName = "";
        select(null);
        emit("project", "new"); emit("time", "new");
    }

    function markSaved(name) { project.name = name; ui.savedName = name; ui.dirty = false; emit("ui", "saved"); }

    /** Accept v2 projects, and convert v1 (sequences + per-sequence prop data) projects */
    function migrate(obj) {
        if (!obj || typeof obj !== "object") return null;
        if (obj.version === PROJECT_VERSION && Array.isArray(obj.clips)) {
            const p = blankProject();
            p.name = obj.name || ""; p.loop = !!obj.loop; p.nextId = num(obj.nextId, 1);
            p.clips = obj.clips.map(c => ({
                id: num(c.id), dict: String(c.dict || ""), clip: String(c.clip || ""), duration: num(c.duration), trimIn: num(c.trimIn), trimOut: c.trimOut == null ? null : num(c.trimOut),
                hold: num(c.hold), enabled: c.enabled !== false, status: c.duration > 0 ? "ok" : "pending", error: null, estimated: !!c.estimated,
            }));
            p.props = obj.props.map((x, i) => ({
                id: num(x.id), model: String(x.model || ""), bone: x.bone || "PH_R_Hand", parentId: x.parentId == null ? null : num(x.parentId), scale: num(x.scale, 1), enabled: x.enabled !== false,
                inTime: num(x.inTime), outTime: x.outTime == null ? null : num(x.outTime), pos: vec(x.pos), rot: vec(x.rot),
                keyframes: (x.keyframes || []).map(k => ({ id: num(k.id), t: num(k.t), pos: vec(k.pos), rot: vec(k.rot), ease: k.ease || "linear" })),
                color: x.color || PROP_COLORS[i % PROP_COLORS.length], error: null,
            }));
            // repair ids
            let max = 0;
            for (const c of p.clips) { if (!c.id) c.id = ++p.nextId + 1000; max = Math.max(max, c.id); }
            for (const x of p.props) { if (!x.id) x.id = ++p.nextId + 1000; max = Math.max(max, x.id); for (const k of x.keyframes) { if (!k.id) k.id = ++p.nextId + 1000; max = Math.max(max, k.id); } }
            p.nextId = Math.max(p.nextId, max + 1);
            return p;
        }
        // ── v1 (sequences) ──
        if (Array.isArray(obj.props) || Array.isArray(obj.sequences)) {
            const p = blankProject();
            const seqs = Array.isArray(obj.sequences) ? obj.sequences : [];
            const starts = []; let t = 0;
            for (const s of seqs) {
                const dur = num(s.duration);
                starts.push(t);
                if (s.dict && s.clip) {
                    p.clips.push({ id: p.nextId++, dict: s.dict, clip: s.clip, duration: dur, trimIn: 0, trimOut: dur, hold: 0, enabled: true, status: dur > 0 ? "ok" : "pending", error: null, estimated: false });
                    t += dur;
                }
            }
            if (!seqs.length && obj.animation && obj.animation.dict) {
                p.clips.push({ id: p.nextId++, dict: obj.animation.dict, clip: obj.animation.clip, duration: 0, trimIn: 0, trimOut: 0, hold: 0, enabled: true, status: "pending", error: null, estimated: false });
            }
            const idMap = {};
            (obj.props || []).forEach((x, i) => {
                const id = p.nextId++; idMap[x.id] = id;
                const prop = {
                    id, model: String(x.model || ""), bone: x.bone || "PH_R_Hand", parentId: null, scale: num(x.scale, 1), enabled: true,
                    inTime: 0, outTime: null, pos: vec(x.pos), rot: vec(x.rot), keyframes: [], color: PROP_COLORS[i % PROP_COLORS.length], error: null, _v1parent: x.parentId,
                };
                // Per-sequence keyframes → absolute times
                let any = false;
                seqs.forEach((s, si) => {
                    const pd = s.propData && (s.propData[x.id] || s.propData[String(x.id)]);
                    if (!pd || !Array.isArray(pd.keyframes)) return;
                    for (const k of pd.keyframes) { prop.keyframes.push({ id: p.nextId++, t: starts[si] + num(k.time), pos: vec(k.pos), rot: vec(k.rot), ease: "linear" }); any = true; }
                });
                if (!any && Array.isArray(x.keyframes)) for (const k of x.keyframes) prop.keyframes.push({ id: p.nextId++, t: num(k.time), pos: vec(k.pos), rot: vec(k.rot), ease: "linear" });
                p.props.push(prop);
            });
            for (const prop of p.props) { if (prop._v1parent != null && idMap[prop._v1parent] != null) prop.parentId = idMap[prop._v1parent]; delete prop._v1parent; }
            return p;
        }
        return null;
    }

    return {
        PROP_COLORS, KF_EPS, MIN_CLIP_LEN,
        get project() { return project; }, ui,
        subscribe, update, undo, redo, canUndo: () => undoStack.length > 0, canRedo: () => redoStack.length > 0,
        layout, totalDuration, clipAt, boundaries,
        getClip, clipIndex, addClip, resolveClip, removeClip, moveClip, duplicateClip, setClip,
        getProp, addProp, removeProp, duplicateProp, setProp, setPropError, wouldCycle,
        kfAt, evalProp, addKeyframe, setTransformAt, removeKeyframe, setKeyframe, clearKeyframes, shiftKeyframes,
        select, selectedProp, selectedClip, selectedKf, setTime, setPlaying, setUI,
        toScene, serialize, loadProject, newProject, markSaved, migrate,
    };
})();
