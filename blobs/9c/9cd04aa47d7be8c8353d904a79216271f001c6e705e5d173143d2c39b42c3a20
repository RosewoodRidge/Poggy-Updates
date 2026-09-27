// ============================================================================
// board.js — drawing boards: the mini board, record cards, the review, and the
// highlights laid over the table's own board
//
// Colours come from the page's tokens (the theme, and the board's content
// colours at the top of the chess styles), read afresh whenever the theme
// changes.
// ============================================================================
"use strict";

const Board = {
    tok: {},
    listeners: [],
};

Board.readTokens = function () {
    const cs = getComputedStyle(document.documentElement);
    const get = n => cs.getPropertyValue(n).trim() || "0,0,0";
    Board.tok = {
        accent: get("--pg-accent-rgb"), text: get("--pg-text-rgb"), shade: get("--pg-shade-rgb"),
        success: get("--pg-success-rgb"), danger: get("--pg-danger-rgb"), warning: get("--pg-warning-rgb"),
        info: get("--pg-info-rgb"), deep: get("--pg-deep-rgb"),
        light: get("--pgc-light-rgb"), dark: get("--pgc-dark-rgb"), red: get("--pgc-red-rgb"),
        black: get("--pgc-black-rgb"), crown: get("--pgc-crown-rgb"), rim: get("--pgc-rim-rgb"),
        font: cs.getPropertyValue("--pg-font-body").trim() || "sans-serif",
    };
};
Board.c = (name, a) => `rgba(${Board.tok[name]}, ${a == null ? 1 : a})`;

Board.onRedraw = fn => Board.listeners.push(fn);
Board.redrawAll = () => { Board.readTokens(); Board.listeners.forEach(fn => { try { fn(); } catch (e) {} }); };
window.addEventListener("poggy:theme", Board.redrawAll);
Board.readTokens();

// ─── Pieces ──────────────────────────────────────────────────────────────────

const PIECE_FILE = { k: "king", q: "queen", r: "rook", b: "bishop", n: "knight", p: "pawn" };
const IMG = {};
for (const name of Object.values(PIECE_FILE)) {
    for (const col of ["w", "b"]) {
        const img = new Image();
        img.onload = () => Board.redrawAll();
        img.src = `img/${name}_${col}.png`;
        IMG[`${name}_${col}`] = img;
    }
}
Board.pieceImage = ch => {
    const name = PIECE_FILE[ch.toLowerCase()];
    return name ? IMG[`${name}_${ch === ch.toUpperCase() ? "w" : "b"}`] : null;
};

// ─── Positions ───────────────────────────────────────────────────────────────

const sqName = (f, r) => String.fromCharCode(96 + f) + r;
const sqCoords = sq => [sq.charCodeAt(0) - 96, parseInt(sq.slice(1), 10)];
Board.sqName = sqName;
Board.sqCoords = sqCoords;
const isDark = (f, r) => (f + r) % 2 === 0;

/** { square: piece } from a chess FEN. */
Board.fromFen = fen => {
    const out = {};
    if (!fen) return out;
    const rows = fen.split(" ")[0].split("/");
    for (let i = 0; i < 8 && i < rows.length; i++) {
        let f = 1;
        for (const ch of rows[i]) {
            if (/\d/.test(ch)) f += parseInt(ch, 10);
            else { out[sqName(f, 8 - i)] = ch; f++; }
        }
    }
    return out;
};

/** { square: piece } from a checkers state string ("rr.b…:red") or a piece list. */
Board.fromCheckers = src => {
    const out = {};
    if (Array.isArray(src)) {
        for (const p of src) out[sqName(p.f, p.r)] = p.piece;
        return out;
    }
    if (typeof src !== "string") return out;
    const cells = src.split(":")[0];
    let i = 0;
    for (let r = 1; r <= 8; r++) for (let f = 1; f <= 8; f++) {
        if (isDark(f, r)) {
            const ch = cells[i++];
            if (ch && ch !== ".") out[sqName(f, r)] = ch;
        }
    }
    return out;
};

/** The standard 1-32 number of a dark square. */
Board.squareNumber = sq => {
    const [f, r] = sqCoords(sq);
    return (8 - r) * 4 + Math.floor((f - 1) / 2) + 1;
};

// ─── Drawing a board ─────────────────────────────────────────────────────────
// o = { kind: "chess" | "checkers", cells: { sq: piece }, flip, coords: "letters" | "numbers" | false,
//       sel, targets: [{ sq, capture }], trail: [sq], must: [sq], last: [sq, sq], check, hint: [sq...], hover }

function drawChecker(ctx, cx, cy, rad, piece) {
    const red = piece === "r" || piece === "R";
    const king = piece === "R" || piece === "B";
    ctx.beginPath(); ctx.arc(cx, cy + rad * 0.08, rad, 0, Math.PI * 2);
    ctx.fillStyle = Board.c("shade", 0.45); ctx.fill();
    ctx.beginPath(); ctx.arc(cx, cy, rad, 0, Math.PI * 2);
    ctx.fillStyle = Board.c(red ? "red" : "black"); ctx.fill();
    ctx.lineWidth = Math.max(1, rad * 0.08);
    ctx.strokeStyle = Board.c("rim", red ? 0.55 : 0.35); ctx.stroke();
    ctx.beginPath(); ctx.arc(cx, cy, rad * 0.68, 0, Math.PI * 2);
    ctx.strokeStyle = Board.c("rim", red ? 0.35 : 0.2); ctx.stroke();
    if (king) {
        // a crown: three points on a band
        const w = rad * 0.9, h = rad * 0.55, x = cx - w / 2, y = cy - h / 2;
        ctx.beginPath();
        ctx.moveTo(x, y + h); ctx.lineTo(x, y + h * 0.35); ctx.lineTo(x + w * 0.25, y + h * 0.65);
        ctx.lineTo(x + w * 0.5, y); ctx.lineTo(x + w * 0.75, y + h * 0.65); ctx.lineTo(x + w, y + h * 0.35);
        ctx.lineTo(x + w, y + h); ctx.closePath();
        ctx.fillStyle = Board.c("crown"); ctx.fill();
        ctx.lineWidth = Math.max(1, rad * 0.05); ctx.strokeStyle = Board.c("shade", 0.5); ctx.stroke();
    }
}

Board.draw = function (canvas, o) {
    const ctx = canvas.getContext("2d");
    const size = canvas.width, s = size / 8;
    const flip = !!o.flip;
    const xy = sq => {
        const [f, r] = sqCoords(sq);
        return [(flip ? 8 - f : f - 1) * s, (flip ? r - 1 : 8 - r) * s];
    };
    ctx.clearRect(0, 0, size, size);

    for (let r = 1; r <= 8; r++) for (let f = 1; f <= 8; f++) {
        const [x, y] = xy(sqName(f, r));
        ctx.fillStyle = Board.c(isDark(f, r) ? "dark" : "light");
        ctx.fillRect(x, y, s, s);
    }

    const fill = (sq, color, a) => { if (!sq) return; const [x, y] = xy(sq); ctx.fillStyle = Board.c(color, a); ctx.fillRect(x, y, s, s); };
    const ring = (sq, color, a, w) => {
        if (!sq) return; const [x, y] = xy(sq);
        ctx.lineWidth = w || Math.max(2, s * 0.07); ctx.strokeStyle = Board.c(color, a);
        ctx.strokeRect(x + ctx.lineWidth / 2, y + ctx.lineWidth / 2, s - ctx.lineWidth, s - ctx.lineWidth);
    };

    (o.last || []).forEach(sq => fill(sq, "accent", 0.22));
    if (o.hover && o.hover !== o.sel) fill(o.hover, "text", 0.14);
    if (o.check) fill(o.check, "danger", 0.6);
    (o.hint || []).forEach(sq => fill(sq, "info", 0.45));
    (o.trail || []).forEach(sq => fill(sq, "accent", 0.35));
    if (o.sel) fill(o.sel, "accent", 0.55);

    // pieces
    for (const [sq, p] of Object.entries(o.cells || {})) {
        const [x, y] = xy(sq);
        if (o.kind === "checkers") {
            drawChecker(ctx, x + s / 2, y + s / 2, s * 0.36, p);
        } else {
            const img = Board.pieceImage(p);
            if (img && img.complete && img.naturalWidth) ctx.drawImage(img, x, y, s, s);
        }
    }

    (o.must || []).forEach(sq => ring(sq, "warning", 0.95));
    if (o.sel) ring(o.sel, "accent", 1);
    for (const tg of o.targets || []) {
        const [x, y] = xy(tg.sq);
        if (tg.capture) {
            ring(tg.sq, "danger", 0.9);
        } else {
            ctx.beginPath(); ctx.arc(x + s / 2, y + s / 2, s * 0.16, 0, Math.PI * 2);
            ctx.fillStyle = Board.c("success", 0.85); ctx.fill();
        }
    }

    if (o.coords) {
        ctx.font = `600 ${Math.max(9, Math.round(s * 0.2))}px ${Board.tok.font}`;
        if (o.coords === "numbers") {
            ctx.textAlign = "left"; ctx.textBaseline = "top";
            for (let r = 1; r <= 8; r++) for (let f = 1; f <= 8; f++) {
                if (!isDark(f, r)) continue;
                const sq = sqName(f, r);
                const [x, y] = xy(sq);
                ctx.fillStyle = Board.c("light", 0.85);
                ctx.fillText(String(Board.squareNumber(sq)), x + s * 0.06, y + s * 0.04);
            }
        } else {
            const files = flip ? "hgfedcba" : "abcdefgh";
            ctx.textAlign = "right"; ctx.textBaseline = "alphabetic";
            for (let c = 0; c < 8; c++) {
                ctx.fillStyle = Board.c(c % 2 === 0 ? "light" : "dark", 0.95);
                ctx.fillText(files[c], c * s + s - s * 0.06, 8 * s - s * 0.06);
            }
            ctx.textAlign = "left"; ctx.textBaseline = "top";
            for (let r = 0; r < 8; r++) {
                ctx.fillStyle = Board.c(r % 2 === 0 ? "dark" : "light", 0.95);
                ctx.fillText(String(flip ? r + 1 : 8 - r), s * 0.06, r * s + s * 0.04);
            }
        }
    }
};

/** The square at a point on a drawn board (canvas pixels, CSS size), or null. */
Board.hit = function (canvas, clientX, clientY, flip) {
    const rect = canvas.getBoundingClientRect();
    const col = Math.floor((clientX - rect.left) / (rect.width / 8));
    const row = Math.floor((clientY - rect.top) / (rect.height / 8));
    if (col < 0 || col > 7 || row < 0 || row > 7) return null;
    const f = flip ? 8 - col : col + 1;
    const r = flip ? row + 1 : 8 - row;
    return sqName(f, r);
};

// ─── The table's own board ───────────────────────────────────────────────────
// Square centres on screen come from the game (0-1); each highlight is drawn as
// the square's own four-sided shape, from its neighbours, so perspective holds.

const Overlay = { pos: {}, fill: 0.85 };

Overlay.setPositions = function (raw) {
    const W = innerWidth, H = innerHeight;
    Overlay.pos = {};
    for (const [sq, p] of Object.entries(raw || {})) Overlay.pos[sq] = { x: p.x * W, y: p.y * H };
};

function steps(sq) {
    const P = Overlay.pos, p = P[sq];
    const [f, r] = sqCoords(sq);
    let fx = 0, fy = 0, rx = 0, ry = 0;
    const nf = P[sqName(f < 8 ? f + 1 : f - 1, r)];
    if (nf) { const d = f < 8 ? 1 : -1; fx = (nf.x - p.x) * d; fy = (nf.y - p.y) * d; }
    const nr = P[sqName(f, r < 8 ? r + 1 : r - 1)];
    if (nr) { const d = r < 8 ? 1 : -1; rx = (nr.x - p.x) * d; ry = (nr.y - p.y) * d; }
    return { fx, fy, rx, ry };
}

Overlay.draw = function (o) {
    const cv = $("overlay");
    if (cv.width !== innerWidth || cv.height !== innerHeight) { cv.width = innerWidth; cv.height = innerHeight; }
    const ctx = cv.getContext("2d");
    ctx.clearRect(0, 0, cv.width, cv.height);
    const quad = (sq, color, a, stroke) => {
        const p = Overlay.pos[sq];
        if (!p) return;
        const { fx, fy, rx, ry } = steps(sq);
        const k = Overlay.fill * 0.5;
        ctx.beginPath();
        ctx.moveTo(p.x - fx * k - rx * k, p.y - fy * k - ry * k);
        ctx.lineTo(p.x + fx * k - rx * k, p.y + fy * k - ry * k);
        ctx.lineTo(p.x + fx * k + rx * k, p.y + fy * k + ry * k);
        ctx.lineTo(p.x - fx * k + rx * k, p.y - fy * k + ry * k);
        ctx.closePath();
        if (stroke) { ctx.lineWidth = 3; ctx.strokeStyle = Board.c(color, a); ctx.stroke(); }
        else { ctx.fillStyle = Board.c(color, a); ctx.fill(); }
    };
    (o.last || []).forEach(sq => quad(sq, "accent", 0.18));
    if (o.hover && o.hover !== o.sel) quad(o.hover, "text", 0.18);
    if (o.check) quad(o.check, "danger", 0.55);
    (o.hint || []).forEach(sq => quad(sq, "info", 0.5));
    (o.trail || []).forEach(sq => quad(sq, "accent", 0.4));
    if (o.sel) { quad(o.sel, "accent", 0.45); quad(o.sel, "accent", 1, true); }
    (o.must || []).forEach(sq => quad(sq, "warning", 0.95, true));
    for (const tg of o.targets || []) quad(tg.sq, tg.capture ? "danger" : "success", tg.capture ? 0.5 : 0.45);
};

/** The square of the table's board under a screen point, or null. */
Overlay.hit = function (x, y) {
    let best = null, bestD = Infinity;
    for (const [sq, p] of Object.entries(Overlay.pos)) {
        const d = Math.hypot(x - p.x, y - p.y);
        if (d < bestD) { bestD = d; best = sq; }
    }
    if (!best) return null;
    const { fx, fy, rx, ry } = steps(best);
    const size = Math.max(Math.hypot(fx, fy), Math.hypot(rx, ry)) || 40;
    return bestD < size * 0.72 ? best : null;
};
