// ============================================================================
// play.js — the game on screen: the HUD, and picking a move
//
// The server sends every legal move with each position. Picking works from
// that list only:
//   chess     a piece, then its square (then the piece for a promotion)
//   checkers  a piece, then each landing square in turn; the move is sent when
//             it is complete, so a capture that can go two ways is the
//             player's choice, hop by hop. F takes the last hop back.
// ============================================================================
"use strict";

const Play = {
    sel: null, path: [], hover: null, hint: null, hintTimer: null,
    pending: false, drawBanner: false, lastMeta: null, visible: false,
};

Play.reset = function () {
    Play.sel = null; Play.path = []; Play.hover = null; Play.pending = false;
    Play.clearHint();
    Play.lastMeta = null;
};

Play.clearHint = function () {
    Play.hint = null;
    if (Play.hintTimer) { clearTimeout(Play.hintTimer); Play.hintTimer = null; }
};

// ─── What the position allows ────────────────────────────────────────────────

const st = () => App.state;
const myTurn = () => { const s = st(); return !!(s && s.toMove === s.mySeat && s.legal && !Play.pending); };

function chessTargets(from) { const s = st(); return (s.legal && s.legal[from]) || []; }
function checkersMoves() { const s = st(); return Array.isArray(s.legal) ? s.legal : []; }

/** Squares holding a piece that can move now. */
function movable() {
    const s = st();
    if (!s || !s.legal) return [];
    if (s.gameType === "chess") return Object.keys(s.legal);
    return [...new Set(checkersMoves().map(m => m.from))];
}

function mustCapture() {
    const list = checkersMoves();
    return list.length > 0 && list.every(m => m.capture);
}

/** Checkers moves still possible from the selected piece and hops so far. */
function candidates() {
    return checkersMoves().filter(m => m.from === Play.sel && Play.path.every((sq, i) => m.path[i] === sq));
}

/** Where the selected piece can go next: [{ sq, capture }]. */
function targets() {
    const s = st();
    if (!Play.sel || !s) return [];
    if (s.gameType === "chess") return chessTargets(Play.sel);
    const seen = {}, out = [];
    for (const m of candidates()) {
        const sq = m.path[Play.path.length];
        if (sq && !seen[sq]) { seen[sq] = true; out.push({ sq, capture: m.capture }); }
    }
    return out;
}

// ─── Picking ─────────────────────────────────────────────────────────────────

function send(data) {
    Play.pending = true;
    Play.sel = null; Play.path = [];
    act("move", data);
    Play.draw();
}

function choosePromotion(from, to) {
    const white = st().mySeat === "white";
    openModal(box => {
        box.appendChild(el("div", { cls: "pg-title", text: t("promote_title") }));
        const row = el("div", { cls: "promo" });
        for (const p of ["Q", "R", "B", "N"]) {
            const img = el("img", { attrs: { src: `img/${PIECE_FILE[p.toLowerCase()]}_${white ? "w" : "b"}.png`, alt: "" } });
            const b = el("button", { cls: "pg-btn", attrs: { type: "button" }, on: { click: () => { closeModal(); send({ from, to, promo: p }); } } },
                img, el("span", { text: t(`piece_${p}`) }));
            row.appendChild(b);
        }
        box.appendChild(row);
        box.appendChild(el("div", { cls: "btn-row" }, button(t("cancel"), () => { closeModal(); Play.sel = null; Play.draw(); })));
    });
}

Play.click = function (sq) {
    const s = st();
    if (!s || !sq) return;
    if (!myTurn()) { Play.sel = null; Play.path = []; Play.draw(); return; }
    const tg = targets().find(x => x.sq === sq);

    if (s.gameType === "chess") {
        if (Play.sel && tg) {
            if (tg.promo) return choosePromotion(Play.sel, sq);
            return send({ from: Play.sel, to: sq });
        }
        Play.sel = movable().includes(sq) && Play.sel !== sq ? sq : null;
        return Play.draw();
    }

    // checkers
    if (Play.sel && tg) {
        Play.path.push(sq);
        const left = candidates();
        if (left.length === 1 && left[0].path.length === Play.path.length) {
            return send({ from: Play.sel, path: Play.path.slice() });
        }
        playSound("move");
        return Play.draw();
    }
    if (Play.path.length > 0) {
        // part-way through a capture: the piece is committed until it is undone
        if (sq === Play.sel) { Play.path = []; Play.sel = null; }
        return Play.draw();
    }
    if (movable().includes(sq) && Play.sel !== sq) {
        Play.sel = sq;
    } else {
        if (!Play.sel && mustCapture() && s.pieces && Board.fromCheckers(s.pieces)[sq]) {
            // clicked a piece that cannot take while a capture is compulsory
            playSound("illegal");
        }
        Play.sel = null;
    }
    Play.draw();
};

Play.back = function () {
    if (Play.path.length > 0) Play.path.pop();
    else Play.sel = null;
    Play.draw();
};

// ─── Drawing ─────────────────────────────────────────────────────────────────

function cells() {
    const s = st();
    if (!s) return {};
    return s.gameType === "chess" ? Board.fromFen(s.fen) : Board.fromCheckers(s.pieces);
}

function highlights() {
    const s = st();
    if (!s) return {};
    const o = {
        sel: Play.sel, targets: targets(), trail: Play.path.slice(), hover: Play.hover,
        check: s.check || null,
        hint: Play.hint ? [Play.hint.from, ...(Play.hint.path || [])] : [],
        last: [],
        must: [],
    };
    const m = Play.lastMeta;
    if (m) o.last = s.gameType === "chess" ? [m.fromSq, m.toSq] : [m.fromSq, ...(m.path || [])];
    if (s.gameType === "checkers" && myTurn() && !Play.sel && mustCapture()) o.must = movable();
    return o;
}

Play.draw = function () {
    const s = st();
    if (!s || !Play.visible) return;
    const h = highlights();
    const flip = s.mySeat === "black";
    const notation = s.gameType === "checkers" && s.rules && s.rules.notation === "numeric" ? "numbers" : "letters";
    Board.draw($("mini"), Object.assign({ kind: s.gameType, cells: cells(), flip, coords: notation }, h));
    Overlay.fill = App.cfg.squareFill || 0.85;
    Overlay.draw(h);
    renderStatus();
};
Board.onRedraw(() => Play.draw());

// ─── The HUD ─────────────────────────────────────────────────────────────────

function renderPlayers() {
    const s = st(), v = App.view;
    for (const seat of ["white", "black"]) {
        const box = $(seat === "white" ? "p-white" : "p-black");
        const info = (v.seats && v.seats[seat]) || {};
        box.querySelector(".chip").className = "chip c-" + colourOf(s.gameType, seat);
        box.querySelector(".player-name").textContent = info.name || colourName(s.gameType, seat);
        box.querySelector(".player-you").textContent = seat === s.mySeat ? t("you") : (info.ai ? t("ai_tag") : "");
        const ratings = App.view.game && App.view.game.ratings;
        box.querySelector(".player-rating").textContent = ratings && ratings[seat] != null ? String(ratings[seat]) : "";
        box.classList.toggle("is-turn", s.toMove === seat);
    }
    $("turn-mark").textContent = s.toMove === s.mySeat ? t("your_move_short") : t("their_move_short");
}

function renderVariant() {
    const s = st(), g = App.view.game || {};
    const bar = $("hud-variant");
    clear(bar);
    bar.appendChild(el("span", { cls: "pg-pill pg-pill--accent", text: gameName(s.gameType, s.variant) }));
    bar.appendChild(el("span", { cls: "pg-pill", text: t("you_play", colourName(s.gameType, s.mySeat)) }));
    if (s.gameType === "checkers" && s.rules) {
        for (const p of Screens.rulePills(s.rules)) bar.appendChild(el("span", { cls: "pg-pill", text: p.text, title: p.tip }));
    }
    if (g.isAI) bar.appendChild(el("span", { cls: "pg-pill pg-pill--info", text: t("vs_ai", levelName(s.gameType, g.aiLevel)) }));
    if (g.wager) bar.appendChild(el("span", { cls: "pg-pill pg-pill--warning", text: t("stake_pill", money(g.wager.amount, g.wager.currency)) }));
    bar.appendChild(button(t("rules_button"), () => Screens.showRulesModal(s.gameType, s.variant, s.rules), "pg-btn--quiet"));
}

function renderStatus() {
    const s = st(), g = App.view && App.view.game;
    const box = $("hud-status");
    box.className = "pg-hud";
    let text;
    if (!s) { box.textContent = ""; return; }
    if (s.toMove !== s.mySeat) {
        const who = (App.view.seats[s.toMove] || {}).name || colourName(s.gameType, s.toMove);
        text = g && g.isAI && g.aiSeat === s.toMove ? t("status_ai_thinking", who) : t("status_waiting", who);
        if (s.myDrawPending) text += " · " + t("status_draw_pending");
        if (s.idleClaimIn != null) {
            const left = Math.max(0, Math.ceil((Play.claimAt - Date.now()) / 1000));
            if (left <= 0) { text = t("status_idle_claim", who); box.classList.add("is-urgent"); }
        }
    } else if (Play.pending) {
        text = t("status_sending");
    } else if (s.gameType === "chess") {
        if (s.check) { text = t("status_in_check"); box.classList.add("is-bad"); }
        else text = Play.sel ? t("status_pick_square") : t("status_your_move");
    } else if (Play.path.length > 0) {
        text = t("status_keep_jumping"); box.classList.add("is-urgent");
    } else if (Play.sel) {
        text = t("status_pick_square");
    } else if (mustCapture()) {
        text = t("status_must_capture"); box.classList.add("is-urgent");
    } else {
        text = Play.sel ? t("status_pick_square") : t("status_your_move");
    }
    box.textContent = text;
}

function renderMoves() {
    const s = st();
    const list = $("moves");
    clear(list);
    const moves = s.moves || [];
    $("moves-title").textContent = t("moves_title");
    if (moves.length === 0) {
        list.appendChild(el("li", { cls: "moves-empty", text: t("moves_none") }));
        return;
    }
    // numbered in pairs from whoever moved first
    const first = s.firstMover || (moves[0] && moves[0].c) || "white";
    let row = null, n = 0;
    moves.forEach((m, i) => {
        const cls = "mv" + (i === moves.length - 1 ? " is-last" : "");
        if (m.c === first || !row) {
            n++;
            row = el("li", null, el("span", { cls: "num", text: n + "." }));
            if (m.c !== first) row.appendChild(el("span", { cls: "mv", text: "…" }));
            list.appendChild(row);
        }
        row.appendChild(el("span", { cls, text: m.n }));
        if (m.c !== first) row = null;
    });
    list.scrollTop = list.scrollHeight;
}

function renderMaterial() {
    const s = st();
    const box = $("hud-material");
    clear(box);
    if (s.gameType === "chess") {
        for (const seat of ["white", "black"]) {
            const caps = el("div", { cls: "caps" });
            for (const p of (seat === "white" ? s.whiteCaptures : s.blackCaptures) || []) {
                caps.appendChild(el("img", { attrs: { src: `img/${PIECE_FILE[p.toLowerCase()]}_${p === p.toUpperCase() ? "w" : "b"}.png`, alt: "" } }));
            }
            box.appendChild(el("div", null, el("span", { cls: "row-sub", text: t("captured_by", colourName("chess", seat)) }), caps));
        }
    } else if (s.counts) {
        for (const seat of ["white", "black"]) {
            const n = s.counts[seat] || 0, k = s.counts[seat + "Kings"] || 0;
            box.appendChild(el("div", { cls: "pg-num", text: t("pieces_left", colourName("checkers", seat), n, k) }));
        }
    }
    // the evaluation bar, chess against the AI
    const bar = $("evalbar");
    if (s.gameType === "chess" && s.evalCp != null) {
        show(bar);
        const cp = Math.max(-2000, Math.min(2000, s.evalCp));
        const whitePct = 50 + 50 * (2 / (1 + Math.exp(-0.00368208 * cp)) - 1);
        bar.querySelector(".bar > i").style.width = whitePct.toFixed(1) + "%";
        $("eval-label").textContent = t("eval_label");
        const pawns = s.evalCp / 100;
        $("eval-num").textContent = Math.abs(s.evalCp) > 5000 ? t("eval_mate") : (pawns >= 0 ? "+" : "") + pawns.toFixed(1);
    } else hide(bar);
}

function renderActions() {
    const s = st(), g = App.view.game || {};
    const box = $("hud-actions");
    clear(box);
    const mine = s.toMove === s.mySeat;
    // hints exist only against the AI
    if (App.cfg.hint && g.isAI) {
        const hint = button(t("btn_hint"), () => act("hint"));
        hint.disabled = !mine;
        box.appendChild(hint);
    }
    const draw = button(t("btn_draw"), () => act("draw"));
    if (s.myDrawPending) draw.disabled = true;
    box.appendChild(draw);
    box.appendChild(button(t("btn_camera"), () => act("camera")));
    box.appendChild(button(t("btn_resign"), () => openModal(b => {
        b.appendChild(el("div", { cls: "pg-title", text: t("resign_title") }));
        b.appendChild(el("p", { cls: "pg-text", text: g.wager ? t("resign_text_wager", money(g.wager.amount, g.wager.currency)) : t("resign_text") }));
        b.appendChild(el("div", { cls: "btn-row" },
            button(t("btn_resign"), () => { closeModal(); act("resign"); }, "pg-btn--danger-fill"),
            button(t("cancel"), closeModal)));
    }), "pg-btn--danger"));
    if (s.idleClaimIn != null && Date.now() >= Play.claimAt) {
        const claim = button(t("btn_claim"), () => act("claim"), "pg-btn--success-fill");
        claim.style.gridColumn = "1 / -1";
        box.appendChild(claim);
    }
    $("keys").textContent = s.gameType === "checkers" ? t("keys_checkers") : t("keys_chess");
}

Play.renderAll = function () {
    if (!st() || !App.view) return;
    renderPlayers();
    renderVariant();
    renderMoves();
    renderMaterial();
    renderActions();
    Play.draw();
};

Play.show = function () {
    Play.visible = true;
    show("hud");
    show("overlay");
    Play.renderAll();
};

Play.hide = function () {
    Play.visible = false;
    hide("hud");
    hide("overlay");
    hide("draw-banner");
    Play.reset();
};

Play.onOver = function () {
    Play.clearHint();
    hide("draw-banner");
    closeModal();
};

// ─── From the game ───────────────────────────────────────────────────────────

Handlers.state = m => {
    const prev = App.state;
    App.state = m.state;
    const s = App.state;
    Play.pending = false;
    if (s.meta) {
        Play.lastMeta = s.meta;
        Play.clearHint();
        const delay = App.cfg.moveMs || 600;
        if (s.gameType === "chess") playSound(s.meta.sound || "move", delay);
        else {
            const hops = (s.meta.path || []).length || 1;
            for (let i = 0; i < hops; i++) playSound((s.meta.captures || []).length ? "capture" : "move", delay * (i + 1));
            if (s.meta.crowned) playSound("crown", delay * hops + 100);
        }
    }
    // a selection only survives while it is still the same turn
    if (!prev || prev.toMove !== s.toMove || s.meta) { Play.sel = null; Play.path = []; }
    Play.claimAt = s.idleClaimIn != null ? Date.now() + s.idleClaimIn * 1000 : null;
    if (!s.drawOffered) hide("draw-banner");
    if (Play.visible) Play.renderAll();
};

Handlers.squares = m => { Overlay.setPositions(m.positions); if (Play.visible) Play.draw(); };

Handlers.key = m => {
    if (!Play.visible) return;
    if (m.key === "select") Play.click(Play.hover);
    else if (m.key === "back") Play.back();
};

Handlers.hint = m => {
    if (!m.hint) return;
    Play.hint = m.hint;
    // a hint also picks up the piece, so the player can follow it
    Play.sel = null; Play.path = [];
    if (Play.hintTimer) clearTimeout(Play.hintTimer);
    Play.hintTimer = setTimeout(() => { Play.hint = null; Play.draw(); }, App.cfg.hintMs || 8000);
    Play.draw();
};

Handlers.drawOffer = m => {
    $("draw-text").textContent = t("draw_offer_text", m.name || "");
    $("draw-yes").textContent = t("draw_accept");
    $("draw-no").textContent = t("draw_decline");
    show("draw-banner");
};
$("draw-yes").addEventListener("click", () => { hide("draw-banner"); act("drawAnswer", { accept: true }); });
$("draw-no").addEventListener("click", () => { hide("draw-banner"); act("drawAnswer", { accept: false }); });

// the idle claim opens while nothing else happens
setInterval(() => {
    const s = st();
    if (Play.visible && s && s.idleClaimIn != null && Play.claimAt && Date.now() >= Play.claimAt && !Play.claimShown) {
        Play.claimShown = true;
        Play.renderAll();
    }
    if (s && (!Play.claimAt || Date.now() < Play.claimAt)) Play.claimShown = false;
}, 1000);

// ─── Mouse ───────────────────────────────────────────────────────────────────

const overlay = $("overlay");
overlay.addEventListener("mousemove", e => {
    const sq = Overlay.hit(e.clientX, e.clientY);
    if (sq !== Play.hover) { Play.hover = sq; Play.draw(); }
});
overlay.addEventListener("click", e => Play.click(Overlay.hit(e.clientX, e.clientY)));
overlay.addEventListener("contextmenu", e => { e.preventDefault(); Play.back(); });

const mini = $("mini");
mini.addEventListener("mousemove", e => {
    const s = st();
    const sq = s ? Board.hit(mini, e.clientX, e.clientY, s.mySeat === "black") : null;
    if (sq !== Play.hover) { Play.hover = sq; Play.draw(); }
});
mini.addEventListener("mouseleave", () => { Play.hover = null; Play.draw(); });
mini.addEventListener("click", e => {
    const s = st();
    if (s) Play.click(Board.hit(mini, e.clientX, e.clientY, s.mySeat === "black"));
});
mini.addEventListener("contextmenu", e => { e.preventDefault(); Play.back(); });

window.addEventListener("resize", () => Play.draw());
