/* ════════════════════════════════════════════════════════════════════
   poggy_animtool — browser.js
   Animation library ("media pool"): folder tree built from the @-split
   dictionary names, multi-keyword search, favourites, preview + add.
   ════════════════════════════════════════════════════════════════════ */
"use strict";

const Browser = (() => {
    const ROW_H = 26;
    const FAV_KEY = "poggy_animtool_favorites";

    let animations = {};        // dict -> clips[]
    let objects = [];           // prop model names
    let flat = [];              // [{dict, clips, lc, clipsLc}]
    let tree = null;            // { folders:{}, entries:[] }
    let navPath = [];
    let expanded = null;        // expanded dict name
    let term = "";
    let keywords = [];
    let results = [];           // search results [{dict, clips, score, clipScores}]
    let favorites = [];
    let favOpen = false;

    let elSearch, elCount, elCrumb, elList, elFavHeader, elFavList, elFavCount, elPreview;

    // ═══════════════════════════════════════════════════════════════
    //  INIT / DATA
    // ═══════════════════════════════════════════════════════════════
    function init(animData, objectData) {
        animations = (animData && animData.animations) || {};
        objects = objectData || [];

        elSearch = $("#animSearch"); elCount = $("#resultCount"); elCrumb = $("#breadcrumb");
        elList = $("#animList"); elFavHeader = $("#favHeader"); elFavList = $("#favList");
        elFavCount = $("#favCount"); elPreview = $("#previewBar");

        flat = Object.entries(animations).map(([dict, clips]) => ({ dict, clips, lc: dict.toLowerCase(), clipsLc: clips.map(c => c.toLowerCase()) }));
        flat.sort((a, b) => a.dict.localeCompare(b.dict));
        tree = buildTree(flat);

        elSearch.addEventListener("input", debounce(() => { term = elSearch.value; applySearch(); }, 180));
        elSearch.addEventListener("keydown", (e) => { if (e.key === "Escape") { elSearch.value = ""; term = ""; applySearch(); elSearch.blur(); } });
        elList.addEventListener("scroll", () => { if (term) renderSearch(); });
        elFavHeader.addEventListener("click", () => { favOpen = !favOpen; elFavList.classList.toggle("open", favOpen); elFavHeader.classList.toggle("open", favOpen); });
        $("#previewStop").addEventListener("click", stopPreview);
        $("#previewAdd").addEventListener("click", () => { const p = State.ui.preview; if (p) App.addClip(p.dict, p.clip); });

        loadFavorites();
        renderFavorites();
        renderPreviewBar();
        applySearch();
    }

    function buildTree(list) {
        const root = { folders: {}, entries: [] };
        for (const e of list) {
            const parts = e.dict.split("@");
            let node = root;
            for (let i = 0; i < parts.length - 1; i++) {
                node = node.folders[parts[i]] || (node.folders[parts[i]] = { folders: {}, entries: [] });
            }
            node.entries.push({ leaf: parts[parts.length - 1], dict: e.dict, clips: e.clips });
        }
        return root;
    }

    function countDesc(node) { let n = node.entries.length; for (const k in node.folders) n += countDesc(node.folders[k]); return n; }
    function nodeAt(path) { let node = tree; for (const seg of path) { node = node.folders[seg]; if (!node) return null; } return node; }

    function getObjects() { return objects; }
    function dictCount() { return flat.length; }

    // ═══════════════════════════════════════════════════════════════
    //  SEARCH
    // ═══════════════════════════════════════════════════════════════
    function applySearch() {
        const t = term.trim().toLowerCase();
        keywords = t ? t.split(/\s+/).filter(Boolean) : [];
        results = [];
        if (keywords.length) {
            for (const e of flat) {
                let dictScore = 0;
                for (const kw of keywords) if (e.lc.includes(kw)) dictScore++;
                let best = 0; const clipScores = new Array(e.clips.length).fill(0);
                for (let i = 0; i < e.clipsLc.length; i++) {
                    let cs = 0;
                    for (const kw of keywords) if (e.clipsLc[i].includes(kw) || e.lc.includes(kw)) cs++;
                    clipScores[i] = cs; if (cs > best) best = cs;
                }
                if (dictScore > 0 || best > 0) results.push({ dict: e.dict, clips: e.clips, score: Math.max(best, dictScore), dictScore, best, clipScores });
            }
            results.sort((a, b) => (b.best - a.best) || (b.dictScore - a.dictScore) || a.dict.localeCompare(b.dict));
            elCount.textContent = results.length.toLocaleString();
            // auto-expand a single hit
            expanded = results.length === 1 ? results[0].dict : null;
        } else {
            elCount.textContent = flat.length.toLocaleString();
            expanded = null;
        }
        elList.scrollTop = 0;
        render();
    }

    // ═══════════════════════════════════════════════════════════════
    //  RENDER
    // ═══════════════════════════════════════════════════════════════
    function render() {
        renderCrumb();
        if (keywords.length) renderSearch(); else renderTree();
    }

    function renderCrumb() {
        elCrumb.innerHTML = "";
        if (keywords.length) {
            elCrumb.appendChild(el("span", "bc-segment bc-current", `Search: ${esc(term.trim())}`));
            const clear = el("span", "bc-clear", "✕ clear");
            clear.addEventListener("click", () => { elSearch.value = ""; term = ""; applySearch(); });
            elCrumb.appendChild(clear);
            return;
        }
        const root = el("span", "bc-segment" + (navPath.length === 0 ? " bc-current" : ""), "All");
        root.addEventListener("click", () => { navPath = []; expanded = null; render(); });
        elCrumb.appendChild(root);
        navPath.forEach((seg, i) => {
            elCrumb.appendChild(el("span", "bc-sep", "›"));
            const s = el("span", "bc-segment" + (i === navPath.length - 1 ? " bc-current" : ""), esc(seg));
            s.addEventListener("click", () => { navPath = navPath.slice(0, i + 1); expanded = null; render(); });
            elCrumb.appendChild(s);
        });
    }

    function renderTree() {
        elList.innerHTML = "";
        elList.classList.remove("virtual");
        const node = nodeAt(navPath);
        if (!node) { elList.appendChild(el("div", "list-empty", "Nothing here")); return; }
        const frag = document.createDocumentFragment();
        if (navPath.length) {
            const up = el("div", "anim-folder anim-up", `<span class="anim-folder-icon">↩</span><span class="anim-folder-name">..</span>`);
            up.addEventListener("click", () => { navPath.pop(); expanded = null; render(); });
            frag.appendChild(up);
        }
        for (const name of Object.keys(node.folders).sort()) {
            const child = node.folders[name];
            const row = el("div", "anim-folder", `<span class="anim-folder-icon">▸</span><span class="anim-folder-name">${esc(name)}</span><span class="anim-folder-count">${countDesc(child)}</span>`);
            row.addEventListener("click", () => { navPath.push(name); expanded = null; render(); elList.scrollTop = 0; });
            frag.appendChild(row);
        }
        for (const e of node.entries.slice().sort((a, b) => a.leaf.localeCompare(b.leaf))) {
            frag.appendChild(dictRow(e.dict, e.leaf, e.clips.length, null));
            if (expanded === e.dict) for (const clip of e.clips) frag.appendChild(clipRow(e.dict, clip, 0));
        }
        if (!Object.keys(node.folders).length && !node.entries.length) frag.appendChild(el("div", "list-empty", "Empty folder"));
        elList.appendChild(frag);
    }

    function renderSearch() {
        // flat virtualised list of dict headers + clips of the expanded dict
        const rows = [];
        for (const r of results) {
            rows.push({ type: "dict", r });
            if (expanded === r.dict) {
                const order = r.clips.map((c, i) => i).sort((a, b) => (r.clipScores[b] - r.clipScores[a]) || r.clips[a].localeCompare(r.clips[b]));
                for (const i of order) rows.push({ type: "clip", r, clip: r.clips[i], score: r.clipScores[i] });
            }
        }
        elList.classList.add("virtual");
        let inner = elList.querySelector(".anim-list-inner");
        if (!inner) { elList.innerHTML = ""; inner = el("div", "anim-list-inner"); elList.appendChild(inner); }
        inner.style.height = (rows.length * ROW_H) + "px";
        if (!rows.length) { inner.innerHTML = '<div class="list-empty">No matches</div>'; return; }
        const top = elList.scrollTop, h = elList.clientHeight;
        const start = Math.max(0, Math.floor(top / ROW_H) - 6);
        const end = Math.min(rows.length, Math.ceil((top + h) / ROW_H) + 6);
        inner.innerHTML = "";
        const frag = document.createDocumentFragment();
        for (let i = start; i < end; i++) {
            const row = rows[i];
            const node = row.type === "dict"
                ? dictRow(row.r.dict, row.r.dict, row.r.clips.length, `${row.r.score}/${keywords.length}`)
                : clipRow(row.r.dict, row.clip, row.score);
            node.style.position = "absolute"; node.style.top = (i * ROW_H) + "px"; node.style.left = "0"; node.style.right = "0"; node.style.height = ROW_H + "px";
            frag.appendChild(node);
        }
        inner.appendChild(frag);
    }

    function dictRow(dict, label, count, badge) {
        const isOpen = expanded === dict;
        const row = el("div", "anim-dict" + (isOpen ? " open" : ""),
            `<span class="anim-dict-arrow">${isOpen ? "▾" : "▸"}</span><span class="anim-dict-name" title="${esc(dict)}">${esc(label)}</span>` +
            (badge ? `<span class="anim-score">${esc(badge)}</span>` : "") + `<span class="anim-dict-count">${count}</span>`);
        row.addEventListener("click", () => { expanded = isOpen ? null : dict; render(); });
        return row;
    }

    function clipRow(dict, clip, score) {
        const pv = State.ui.preview;
        const isPreview = pv && pv.dict === dict && pv.clip === clip;
        const isFav = favorites.some(f => f.dict === dict && f.clip === clip);
        const row = el("div", "anim-clip" + (isPreview ? " previewing" : "") + (isFav ? " has-fav" : ""),
            `<span class="anim-clip-icon">${isPreview ? "▶" : "·"}</span>` +
            `<span class="anim-clip-text" title="${esc(dict)} › ${esc(clip)}">${esc(clip)}</span>` +
            (score > 0 && keywords.length ? `<span class="anim-score clip-score">${score}/${keywords.length}</span>` : "") +
            `<span class="anim-clip-actions">` +
            `<button class="mini fav${isFav ? " on" : ""}" title="Favourite">★</button>` +
            `<button class="mini add" title="Add to timeline (double-click row / drag onto timeline)">+</button></span>`);
        row.draggable = true; row.dataset.dict = dict; row.dataset.clip = clip;
        row.addEventListener("click", (e) => { if (e.target.closest("button")) return; preview(dict, clip); });
        row.addEventListener("dblclick", (e) => { if (e.target.closest("button")) return; App.addClip(dict, clip); });
        row.querySelector(".add").addEventListener("click", (e) => { e.stopPropagation(); App.addClip(dict, clip); });
        row.querySelector(".fav").addEventListener("click", (e) => { e.stopPropagation(); toggleFavorite(dict, clip); });
        row.addEventListener("contextmenu", (e) => { e.preventDefault(); toggleFavorite(dict, clip); });
        row.addEventListener("dragstart", (e) => {
            e.dataTransfer.setData("text/animtool-clip", JSON.stringify({ dict, clip }));
            e.dataTransfer.setData("text/plain", `${dict} ${clip}`);
            e.dataTransfer.effectAllowed = "copy";
            row.classList.add("dragging");
        });
        row.addEventListener("dragend", () => row.classList.remove("dragging"));
        return row;
    }

    // ═══════════════════════════════════════════════════════════════
    //  PREVIEW
    // ═══════════════════════════════════════════════════════════════
    function preview(dict, clip) {
        const pv = State.ui.preview;
        if (pv && pv.dict === dict && pv.clip === clip) { stopPreview(); return; }
        State.setUI({ preview: { dict, clip, duration: 0 } });
        NUI.post("previewAnim", { dict, clip });
        renderPreviewBar(); updatePreviewRows();
    }

    function stopPreview() {
        if (!State.ui.preview) return;
        State.setUI({ preview: null });
        NUI.post("stopPreview");
        renderPreviewBar(); updatePreviewRows();
    }

    function onPreviewStarted(msg) {
        State.setUI({ preview: { dict: msg.dict, clip: msg.clip, duration: msg.duration || 0 } });
        renderPreviewBar(); updatePreviewRows();
    }
    function onPreviewStopped() { if (State.ui.preview) { State.setUI({ preview: null }); renderPreviewBar(); updatePreviewRows(); } }

    /** Toggle the .previewing class without rebuilding rows (keeps dblclick working) */
    function updatePreviewRows() {
        const pv = State.ui.preview;
        for (const row of elList.querySelectorAll(".anim-clip")) {
            const on = !!pv && row.dataset.dict === pv.dict && row.dataset.clip === pv.clip;
            row.classList.toggle("previewing", on);
            const ic = row.querySelector(".anim-clip-icon"); if (ic) ic.textContent = on ? "▶" : "·";
        }
    }

    function renderPreviewBar() {
        const pv = State.ui.preview;
        elPreview.classList.toggle("idle", !pv);
        const name = $("#previewName", elPreview);
        if (!pv) { name.textContent = "Click a clip to preview it on your ped"; name.title = ""; $("#previewDur", elPreview).textContent = ""; return; }
        name.textContent = `${pv.dict} › ${pv.clip}`;
        name.title = `${pv.dict} › ${pv.clip}`;
        $("#previewDur", elPreview).textContent = pv.duration > 0 ? fmtSec(pv.duration) : "";
    }

    // ═══════════════════════════════════════════════════════════════
    //  FAVOURITES (localStorage)
    // ═══════════════════════════════════════════════════════════════
    function loadFavorites() { try { favorites = JSON.parse(localStorage.getItem(FAV_KEY) || "[]"); } catch { favorites = []; } }
    function saveFavorites() { try { localStorage.setItem(FAV_KEY, JSON.stringify(favorites)); } catch {} }

    function toggleFavorite(dict, clip) {
        const i = favorites.findIndex(f => f.dict === dict && f.clip === clip);
        if (i >= 0) favorites.splice(i, 1); else { favorites.push({ dict, clip }); if (!favOpen) { favOpen = true; elFavList.classList.add("open"); elFavHeader.classList.add("open"); } }
        saveFavorites(); renderFavorites(); render();
    }

    function renderFavorites() {
        elFavCount.textContent = favorites.length;
        elFavList.innerHTML = "";
        if (!favorites.length) { elFavList.appendChild(el("div", "list-empty", "Right-click or ★ a clip to save it here")); return; }
        favorites.forEach((f, idx) => {
            const row = el("div", "fav-item",
                `<span class="fav-name" title="${esc(f.dict)} › ${esc(f.clip)}"><b>${esc(f.clip)}</b><small>${esc(f.dict)}</small></span>` +
                `<span class="anim-clip-actions"><button class="mini add" title="Add to timeline">+</button><button class="mini del" title="Remove">✕</button></span>`);
            row.draggable = true;
            row.addEventListener("click", (e) => { if (e.target.closest("button")) return; preview(f.dict, f.clip); });
            row.addEventListener("dblclick", (e) => { if (e.target.closest("button")) return; App.addClip(f.dict, f.clip); });
            row.querySelector(".add").addEventListener("click", (e) => { e.stopPropagation(); App.addClip(f.dict, f.clip); });
            row.querySelector(".del").addEventListener("click", (e) => { e.stopPropagation(); favorites.splice(idx, 1); saveFavorites(); renderFavorites(); render(); });
            row.addEventListener("dragstart", (e) => { e.dataTransfer.setData("text/animtool-clip", JSON.stringify({ dict: f.dict, clip: f.clip })); e.dataTransfer.effectAllowed = "copy"; });
            elFavList.appendChild(row);
        });
    }

    return { init, getObjects, dictCount, preview, stopPreview, onPreviewStarted, onPreviewStopped, render, focusSearch: () => elSearch && elSearch.focus() };
})();
