// ============================================================================
// app.js — the page's core: messages from the game, words, sounds, helpers
// ============================================================================
"use strict";

const App = {
    cfg: null,        // from "init"
    strings: {},
    view: null,       // the table view (who sits where, what phase)
    state: null,      // the game state
    over: null,       // the last game-over summary
};

const RES = (typeof GetParentResourceName === "function") ? GetParentResourceName() : "poggy_chess";

// ─── Talking to the game ─────────────────────────────────────────────────────

function act(action, data) {
    return fetch(`https://${RES}/act`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(Object.assign({ action }, data || {})),
    }).catch(() => {});
}

function ask(what, data) {
    return fetch(`https://${RES}/ask`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(Object.assign({ what }, data || {})),
    }).then(r => r.json()).catch(() => ({}));
}

// ─── Words ───────────────────────────────────────────────────────────────────

/** A string from translations.lua, with %s / %d filled in order. */
function t(key, ...args) {
    let s = App.strings[key];
    if (s == null) return key;
    let i = 0;
    return String(s).replace(/%%|%[sd]/g, m => (m === "%%" ? "%" : (i < args.length ? String(args[i++]) : "")));
}

function levelName(gameType, id) {
    if (!id) return "";
    const k = `level_${id}`;
    return App.strings[k] != null ? t(k) : id.charAt(0).toUpperCase() + id.slice(1);
}

function variantName(v) { return v ? t(`variant_${v}`) : ""; }
function gameName(gameType, variant) {
    return gameType === "checkers" ? t("game_checkers_variant", variantName(variant || "american")) : t("game_chess");
}

/** "1 move" / "12 moves" from a count of full moves. */
function movesText(n) {
    n = Math.max(0, Math.floor(n || 0));
    return n === 1 ? t("moves_count_one", n) : t("moves_count", n);
}

function money(amount, currency) {
    return t(currency === "gold" ? "money_gold" : "money_cash", Math.floor(amount || 0));
}

/** "5+3" for a time control { minutes, increment }. */
function clockText(tc) { return tc ? t("clock_short", tc.minutes, tc.increment) : ""; }

/** Time on a clock: 1:05:00, 4:59, and 0:09.4 when it is low. */
function clockTime(ms, low) {
    ms = Math.max(0, ms);
    if (low && ms < 10000) return "0:0" + (Math.floor(ms / 100) / 10).toFixed(1);
    const total = Math.ceil(ms / 1000);
    const h = Math.floor(total / 3600), m = Math.floor((total % 3600) / 60), s = total % 60;
    const ss = String(s).padStart(2, "0");
    return h > 0 ? `${h}:${String(m).padStart(2, "0")}:${ss}` : `${m}:${ss}`;
}

/** A seat's colour in this game: white / black, or red / black in checkers. */
function colourOf(gameType, seat) {
    if (gameType === "checkers") return seat === "white" ? "red" : "black";
    return seat;
}
function colourName(gameType, seat) { return t(`colour_${colourOf(gameType, seat)}`); }

// ─── DOM helpers ─────────────────────────────────────────────────────────────

const $ = id => document.getElementById(id);
const show = el => (typeof el === "string" ? $(el) : el).classList.remove("hidden");
const hide = el => (typeof el === "string" ? $(el) : el).classList.add("hidden");

/** Makes an element. Text is always set as text, never as HTML. */
function el(tag, opts, ...children) {
    const e = document.createElement(tag);
    if (opts) {
        if (opts.cls) e.className = opts.cls;
        if (opts.text != null) e.textContent = opts.text;
        if (opts.title) e.dataset.tip = opts.title;
        if (opts.on) for (const [k, fn] of Object.entries(opts.on)) e.addEventListener(k, fn);
        if (opts.attrs) for (const [k, v] of Object.entries(opts.attrs)) e.setAttribute(k, v);
    }
    for (const c of children) if (c) e.appendChild(c);
    return e;
}

function button(text, onClick, cls) {
    return el("button", { cls: "pg-btn" + (cls ? " " + cls : ""), text, attrs: { type: "button" }, on: { click: onClick } });
}

function clear(e) { while (e.firstChild) e.removeChild(e.firstChild); }

// Tooltips: anything with data-tip.
document.addEventListener("mouseover", e => {
    const target = e.target.closest && e.target.closest("[data-tip]");
    const tip = $("tip");
    if (!target) { hide(tip); return; }
    tip.textContent = target.dataset.tip;
    show(tip);
    const r = target.getBoundingClientRect();
    const w = tip.offsetWidth, h = tip.offsetHeight;
    let x = r.left + r.width / 2 - w / 2, y = r.bottom + 8;
    if (y + h > innerHeight - 8) y = r.top - h - 8;
    tip.style.left = Math.max(8, Math.min(innerWidth - w - 8, x)) + "px";
    tip.style.top = Math.max(8, y) + "px";
});

// ─── Sounds ──────────────────────────────────────────────────────────────────

const SOUNDS = { move: "move", capture: "capture", check: "check", castle: "castle", checkmate: "checkmate", start: "start", illegal: "illegal", crown: "start" };

function playSound(name, delay) {
    const file = SOUNDS[name];
    if (!file || !App.cfg) return;
    setTimeout(() => {
        const a = new Audio(`sfx/chess/${file}.mp3`);
        a.volume = Math.max(0, Math.min(1, App.cfg.volume));
        a.play().catch(() => {});
    }, delay || 0);
}

// ─── Modal ───────────────────────────────────────────────────────────────────

function openModal(build) {
    const box = $("modal-box");
    clear(box);
    build(box);
    show("modal");
}
function closeModal() { hide("modal"); }

// ─── Messages from the game ──────────────────────────────────────────────────

const Handlers = {};

window.addEventListener("message", e => {
    const m = e.data;
    if (!m || !m.type) return;
    const fn = Handlers[m.type];
    if (fn) {
        try { fn(m); } catch (err) { console.error(`[poggy_chess] ${m.type}:`, err); }
    }
});

Handlers.init = m => {
    App.cfg = m;
    App.strings = m.strings || {};
};

Handlers.close = () => {
    App.view = null; App.state = null; App.over = null;
    ["table-panel", "hud", "record-panel", "review-panel", "modal", "overlay", "draw-banner", "tip"].forEach(hide);
    if (window.Play) Play.reset();
};

// Escape closes whatever sits on top; it never stands the player up.
document.addEventListener("keydown", e => {
    if (e.key !== "Escape") return;
    if (!$("modal").classList.contains("hidden")) return closeModal();
    if (!$("review-panel").classList.contains("hidden")) return Record.closeReview();
    if (!$("record-panel").classList.contains("hidden")) return Record.close();
});
