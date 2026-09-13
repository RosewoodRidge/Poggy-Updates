/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — nui.js
   NUI bridge. In game, post() talks to client.lua. In a plain browser
   (no GetParentResourceName) a MockGame simulates the Lua side so the
   whole editor can be exercised outside RedM.
   ════════════════════════════════════════════════════════════════════ */
"use strict";

const NUI = (() => {
    const inGame = typeof window.GetParentResourceName === "function";
    const handlers = {};

    function resName() {
        return inGame ? window.GetParentResourceName() : "poggy_animtool";
    }

    function post(event, data) {
        if (!inGame) return Promise.resolve(MockGame.handle(event, data || {}));
        return fetch(`https://${resName()}/${event}`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(data || {}),
        }).then(r => r.text()).catch(() => "");
    }

    function on(action, fn) {
        (handlers[action] = handlers[action] || []).push(fn);
    }

    window.addEventListener("message", (ev) => {
        const msg = ev.data;
        if (!msg || !msg.action) return;
        const list = handlers[msg.action];
        if (!list) return;
        for (const fn of list) {
            try { fn(msg); } catch (e) { console.error("[AnimTool] handler error", msg.action, e); }
        }
    });

    async function fetchText(url) {
        const r = await fetch(url);
        if (!r.ok) throw new Error(`HTTP ${r.status} for ${url}`);
        return r.text();
    }

    async function fetchJSON(url) {
        const txt = await fetchText(url);
        if (!txt) throw new Error(`Empty response for ${url}`);
        return JSON.parse(txt.charCodeAt(0) === 0xFEFF ? txt.slice(1) : txt);
    }

    return { inGame, post, on, fetchText, fetchJSON };
})();

/* ────────────────────────────────────────────────────────────────────
   MockGame — browser stand-in for client.lua + ScenePlayer
   ──────────────────────────────────────────────────────────────────── */
const MockGame = (() => {
    const BONES = ["PH_R_Hand", "PH_L_Hand", "IK_R_Hand", "IK_L_Hand", "skel_r_hand", "skel_l_hand",
        "SKEL_ROOT", "SKEL_Pelvis", "SKEL_Spine0", "SKEL_Spine1", "SKEL_Spine2", "SKEL_Spine3",
        "SKEL_L_Clavicle", "SKEL_L_UpperArm", "SKEL_L_Forearm", "SKEL_R_Clavicle", "SKEL_R_UpperArm",
        "SKEL_R_Forearm", "SKEL_Neck0", "SKEL_Neck1", "SKEL_Head", "SKEL_L_Thigh", "SKEL_L_Calf",
        "SKEL_L_Foot", "SKEL_R_Thigh", "SKEL_R_Calf", "SKEL_R_Foot"];

    const st = {
        scene: null, layout: [], total: 0,
        time: 0, playing: false, speed: 1, loop: false, armed: false,
        preview: null, timer: null, open: false,
    };

    function emit(msg) { window.postMessage(msg, "*"); }

    function hash(s) {
        let h = 2166136261;
        for (let i = 0; i < s.length; i++) { h ^= s.charCodeAt(i); h = Math.imul(h, 16777619); }
        return Math.abs(h >>> 0);
    }

    function layout(scene) {
        const out = []; let t = 0;
        for (const c of (scene.clips || [])) {
            const dur = +c.duration || 0;
            const tin = Math.min(Math.max(+c.trimIn || 0, 0), dur);
            const tout = Math.min(Math.max(c.trimOut == null ? dur : +c.trimOut, tin), dur);
            const hold = Math.max(0, +c.hold || 0);
            const len = (tout - tin) + hold;
            if (c.dict && c.clip && c.enabled !== false && dur > 0.0005 && len > 0.0005) {
                out.push({ start: t, length: len }); t += len;
            }
        }
        st.layout = out; st.total = t;
    }

    function tick() {
        if (!st.open) return;
        const dt = 0.033;
        if (st.playing) {
            st.time += dt * st.speed;
            if (st.time >= st.total) {
                if (st.loop && st.total > 0) st.time -= st.total;
                else { st.time = st.total; st.playing = false; emit({ action: "finished" }); }
            }
        }
        let clip = null;
        if (st.armed && st.layout.length) {
            for (let i = st.layout.length - 1; i >= 0; i--) if (st.time >= st.layout[i].start) { clip = i + 1; break; }
        }
        emit({ action: "progress", time: st.time, playing: st.playing, clip });
    }

    function handle(event, data) {
        switch (event) {
            case "ready":
                st.open = true;
                setTimeout(() => emit({ action: "open", bones: BONES }), 50);
                if (!st.timer) st.timer = setInterval(tick, 33);
                return "ok";
            case "close":
                st.open = false; st.playing = false; st.time = 0; st.armed = false;
                emit({ action: "close" });
                // Re-open shortly after so the browser session keeps working
                setTimeout(() => { st.open = true; emit({ action: "open", bones: BONES }); }, 800);
                return "ok";
            case "syncScene":
                st.scene = data.scene; layout(st.scene);
                if (st.scene && st.scene.loop != null) st.loop = !!st.scene.loop;
                if (st.time > st.total) st.time = st.total;
                for (const p of (st.scene.props || [])) {
                    if (p.enabled !== false && /bad/i.test(p.model)) {
                        setTimeout(() => emit({ action: "propError", id: p.id, model: p.model, message: "Model failed to load: " + p.model }), 120);
                    }
                }
                return "ok";
            case "transport":
                if (data.action !== "pause") st.preview = null;
                if (data.action === "play") { if (st.total > 0) { if (st.time >= st.total - 0.0005) st.time = 0; st.playing = true; st.armed = true; } }
                else if (data.action === "pause") st.playing = false;
                else if (data.action === "stop") { st.playing = false; st.time = 0; st.armed = false; }
                else if (data.action === "seek") { st.time = Math.min(Math.max(+data.time || 0, 0), st.total); st.armed = true; }
                else if (data.action === "toggle") { if (st.playing) st.playing = false; else if (st.total > 0) { st.playing = true; st.armed = true; } }
                return "ok";
            case "setSpeed": st.speed = +data.speed || 1; return "ok";
            case "setLoop": st.loop = !!data.loop; return "ok";
            case "resolveClip": {
                const { id, dict, clip } = data;
                setTimeout(() => {
                    if (/err/i.test(clip)) emit({ action: "clipResolved", id, ok: false, error: "Dict not found: " + dict });
                    else {
                        const h = hash(dict + "/" + clip);
                        emit({ action: "clipResolved", id, ok: true, duration: 1 + (h % 60) / 10, estimated: (h % 7) === 0 });
                    }
                }, 120 + (hash(clip) % 200));
                return "ok";
            }
            case "previewAnim":
                st.playing = false; st.preview = { dict: data.dict, clip: data.clip };
                emit({ action: "previewStarted", dict: data.dict, clip: data.clip, duration: 1 + (hash(data.clip) % 60) / 10 });
                return "ok";
            case "stopPreview":
                st.preview = null; emit({ action: "previewStopped" }); return "ok";
            case "selectProp": case "cameraHold": return "ok";
            case "clickWorld": {
                const props = (st.scene && st.scene.props) || [];
                if (props.length) { const p = props[hash(String(Date.now())) % props.length]; emit({ action: "propClicked", id: p.id }); return String(p.id); }
                return "none";
            }
            case "saveProject":
                localStorage.setItem("mock_proj_" + data.name, data.data || "");
                emit({ action: "projectSaved", name: data.name, silent: data.silent }); return "ok";
            case "loadProject": {
                const raw = localStorage.getItem("mock_proj_" + data.name);
                if (raw != null) emit({ action: "projectLoaded", name: data.name, data: raw });
                else emit({ action: "notify", kind: "error", message: "Project not found: " + data.name });
                return "ok";
            }
            case "listProjects": {
                const names = Object.keys(localStorage).filter(k => k.startsWith("mock_proj_")).map(k => k.slice(10)).sort();
                emit({ action: "projectList", names }); return "ok";
            }
            case "deleteProject":
                localStorage.removeItem("mock_proj_" + data.name);
                emit({ action: "projectDeleted", name: data.name }); return "ok";
        }
        return "ok";
    }

    return { handle, state: st };
})();
