/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — export.js
   Generates the Lua scene table (+ usage snippet), serves the runtime
   (scene_player.lua) and the raw project JSON in a tabbed modal.
   ════════════════════════════════════════════════════════════════════ */
"use strict";

const Exporter = (() => {
    let modal, tabs, codeEl, copyBtn, runtimeSrc = null, current = "scene";

    function init() {
        modal = $("#exportModal"); codeEl = $("#exportCode"); copyBtn = $("#copyBtn");
        tabs = $$(".export-tab", modal);
        for (const t of tabs) t.addEventListener("click", () => show(t.dataset.tab));
        $("#closeModal").addEventListener("click", close);
        $(".modal-backdrop", modal).addEventListener("click", close);
        copyBtn.addEventListener("click", copy);
        $("#exportBtn").addEventListener("click", open);
    }

    function open() {
        modal.classList.remove("hidden");
        show(current);
    }
    function close() { modal.classList.add("hidden"); }
    function isOpen() { return !modal.classList.contains("hidden"); }

    async function show(tab) {
        current = tab;
        for (const t of tabs) t.classList.toggle("active", t.dataset.tab === tab);
        if (tab === "scene") codeEl.value = sceneLua();
        else if (tab === "json") codeEl.value = JSON.stringify(JSON.parse(State.serialize()), null, 2);
        else if (tab === "runtime") {
            codeEl.value = "-- loading scene_player.lua …";
            if (!runtimeSrc) { try { runtimeSrc = await NUI.fetchText("../client/scene_player.lua"); } catch (e) { runtimeSrc = "-- Could not load client/scene_player.lua from the resource: " + e.message; } }
            if (current === "runtime") codeEl.value = runtimeSrc;
        }
        codeEl.scrollTop = 0;
    }

    function copy() {
        codeEl.select();
        let ok = false;
        try { ok = document.execCommand("copy"); } catch {}
        if (!ok && navigator.clipboard) navigator.clipboard.writeText(codeEl.value).catch(() => {});
        copyBtn.textContent = "Copied!";
        setTimeout(() => { copyBtn.textContent = "Copy to clipboard"; }, 1400);
    }

    // ── Lua generation ────────────────────────────────────────────
    const n3 = (v) => (Math.round((+v || 0) * 1000) / 1000).toFixed(3);
    const n4 = (v) => (Math.round((+v || 0) * 10000) / 10000).toFixed(4);
    const str = (s) => '"' + String(s).replace(/\\/g, "\\\\").replace(/"/g, '\\"') + '"';
    const v3 = (v, f) => `{ x = ${f(v.x)}, y = ${f(v.y)}, z = ${f(v.z)} }`;

    function sceneLua() {
        const p = State.project;
        const scene = State.toScene();
        const L = State.layout();
        const lines = [];
        lines.push(`-- ═══════════════════════════════════════════════════════════════`);
        lines.push(`--  Scene exported from Poggy AnimTool${p.name ? "  ·  " + p.name : ""}`);
        lines.push(`--  ${new Date().toISOString().slice(0, 16).replace("T", " ")}  ·  ${fmtTime(scene.duration)} · ${scene.clips.filter(c => c.enabled).length} clip(s) · ${scene.props.filter(x => x.enabled).length} prop(s)`);
        lines.push(`--  Needs client/scene_player.lua loaded first (see the "Runtime" tab).`);
        lines.push(`-- ═══════════════════════════════════════════════════════════════`);
        lines.push(`local scene = {`);
        lines.push(`    duration = ${n3(scene.duration)},`);
        lines.push(`    loop     = ${scene.loop ? "true" : "false"},`);
        lines.push(`    clips = {`);
        L.clips.forEach((e, i) => {
            const c = e.clip;
            if (!e.valid) { lines.push(`        -- skipped (${c.status === "error" ? "failed" : c.enabled === false ? "disabled" : "unresolved"}): ${c.dict} / ${c.clip}`); return; }
            lines.push(`        { -- ${i + 1}: ${fmtTime(e.start)} → ${fmtTime(e.start + e.length)}`);
            lines.push(`            dict = ${str(c.dict)}, clip = ${str(c.clip)},`);
            lines.push(`            duration = ${n3(c.duration)}, trimIn = ${n3(c.trimIn)}, trimOut = ${n3(c.trimOut)}, hold = ${n3(c.hold)},`);
            lines.push(`        },`);
        });
        lines.push(`    },`);
        lines.push(`    props = {`);
        for (const x of p.props) {
            if (x.enabled === false) { lines.push(`        -- skipped (disabled): ${x.model}`); continue; }
            lines.push(`        {`);
            lines.push(`            id = ${x.id}, model = ${str(x.model)},`);
            lines.push(`            bone = ${str(x.bone)},${x.parentId != null ? ` parent = ${x.parentId},` : ""} scale = ${n3(x.scale)},`);
            lines.push(`            inTime = ${n3(x.inTime)},${x.outTime != null ? ` outTime = ${n3(x.outTime)},` : " -- outTime omitted = until the end"}`);
            lines.push(`            pos = ${v3(x.pos, n4)}, rot = ${v3(x.rot, n3)},`);
            if (x.keyframes.length) {
                lines.push(`            keyframes = {`);
                for (const k of x.keyframes) lines.push(`                { t = ${n3(k.t)}, pos = ${v3(k.pos, n4)}, rot = ${v3(k.rot, n3)}, ease = ${str(k.ease || "linear")} },`);
                lines.push(`            },`);
            }
            lines.push(`        },`);
        }
        lines.push(`    },`);
        lines.push(`}`);
        lines.push(``);
        lines.push(`-- ── Usage ─────────────────────────────────────────────────────`);
        lines.push(`-- One-shot (auto cleans up when finished):`);
        lines.push(`--     ScenePlayer.playOnce(PlayerPedId(), scene)`);
        lines.push(`--`);
        lines.push(`-- Full control:`);
        lines.push(`--     local player = ScenePlayer.new(PlayerPedId())`);
        lines.push(`--     player:load(scene)`);
        lines.push(`--     player:play()            -- :pause() :seek(t) :stop() :setSpeed(s) :setLoop(b)`);
        lines.push(`--     player.onFinish = function() player:destroy() end`);
        lines.push(``);
        lines.push(`return scene`);
        return lines.join("\n");
    }

    return { init, open, close, isOpen, sceneLua };
})();
