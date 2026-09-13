/* poggy_animtool — util.js : tiny shared helpers (loaded first) */
"use strict";

const $  = (s, p) => (p || document).querySelector(s);
const $$ = (s, p) => [...(p || document).querySelectorAll(s)];

/** 7.256 → "0:07.26" */
function fmtTime(t) {
    t = Math.max(0, +t || 0);
    const m = Math.floor(t / 60);
    const s = t - m * 60;
    return `${m}:${s < 10 ? "0" : ""}${s.toFixed(2)}`;
}

function fmtSec(t, digits = 2) { return (+t || 0).toFixed(digits) + "s"; }

function esc(s) {
    return String(s ?? "").replace(/[&<>"']/g, c => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c]));
}

function el(tag, cls, html) {
    const e = document.createElement(tag);
    if (cls) e.className = cls;
    if (html != null) e.innerHTML = html;
    return e;
}

function clampNum(v, lo, hi) { return Math.max(lo, Math.min(hi, v)); }

function debounce(fn, ms) {
    let t = null;
    return (...a) => { clearTimeout(t); t = setTimeout(() => fn(...a), ms); };
}

function throttle(fn, ms) {
    let last = 0, pending = null, timer = null;
    return (...a) => {
        const now = Date.now();
        if (now - last >= ms) { last = now; fn(...a); }
        else { pending = a; clearTimeout(timer); timer = setTimeout(() => { last = Date.now(); fn(...pending); pending = null; }, ms - (now - last)); }
    };
}
