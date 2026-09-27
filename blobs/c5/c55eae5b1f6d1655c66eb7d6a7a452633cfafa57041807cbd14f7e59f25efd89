// ============================================================================
// screens.js — the panel beside the table: choosing a game, waiting, an
// invitation, the end of a game; and the rules card every one of them shows
// ============================================================================
"use strict";

const Screens = {
    setup: null,      // what the host has picked so far
    overOpen: false,
    resumable: [],
};

// ─── The rules card ──────────────────────────────────────────────────────────

/** Every rule of a checkers rule set, as sentences. */
function checkersRules(r) {
    const out = [];
    if (!r) return out;
    out.push(t(r.firstMove === "black" ? "rule_first_black" : "rule_first_red"));
    out.push(t(r.menCaptureBack ? "rule_men_back" : "rule_men_forward"));
    out.push(t(r.flyingKings ? "rule_kings_fly" : "rule_kings_short"));
    if (r.mustCapture) out.push(t(r.maxCapture ? "rule_max_capture" : "rule_must_capture"));
    else out.push(t("rule_optional_capture"));
    out.push(t("rule_multi"));
    out.push(t(`rule_crown_${r.crown || "stop"}`));
    out.push(t(r.blockedLoses ? "rule_blocked_loses" : "rule_blocked_draws"));
    const d = (App.cfg && App.cfg.draws) || {};
    if (d.Repetition > 0 && d.QuietMoves > 0) out.push(t("rule_draws_both", d.Repetition, d.QuietMoves));
    else if (d.Repetition > 0) out.push(t("rule_draws_rep", d.Repetition));
    else if (d.QuietMoves > 0) out.push(t("rule_draws_quiet", d.QuietMoves));
    return out;
}

function chessRules() { return [t("rule_chess_1"), t("rule_chess_2")]; }

function rulesCard(gameType, rules) {
    const list = el("div", { cls: "rules" });
    for (const s of gameType === "checkers" ? checkersRules(rules) : chessRules()) {
        list.appendChild(el("div", { cls: "rule", text: s }));
    }
    return list;
}
Screens.rulesCard = rulesCard;

/** Short labels for the rules that set a variant apart (the HUD pills). */
Screens.rulePills = function (r) {
    const pills = [];
    pills.push({ text: t(r.mustCapture ? (r.maxCapture ? "pill_max_capture" : "pill_must_capture") : "pill_optional_capture"),
        tip: t(r.mustCapture ? (r.maxCapture ? "rule_max_capture" : "rule_must_capture") : "rule_optional_capture") });
    if (r.flyingKings) pills.push({ text: t("pill_flying"), tip: t("rule_kings_fly") });
    if (r.menCaptureBack) pills.push({ text: t("pill_back"), tip: t("rule_men_back") });
    pills.push({ text: t(`pill_crown_${r.crown || "stop"}`), tip: t(`rule_crown_${r.crown || "stop"}`) });
    return pills;
};

function showRulesModal(gameType, variant, rules) {
    openModal(box => {
        box.appendChild(el("div", { cls: "pg-title", text: gameName(gameType, variant) }));
        if (gameType === "checkers") box.appendChild(el("div", { cls: "pg-text", text: t(`variant_${variant}_desc`) }));
        box.appendChild(rulesCard(gameType, rules));
        box.appendChild(el("div", { cls: "btn-row" }, button(t("close"), closeModal)));
    });
}
Screens.showRulesModal = showRulesModal;

// ─── The panel ───────────────────────────────────────────────────────────────

function panel(kicker, title) {
    $("tp-kicker").textContent = kicker || "";
    $("tp-title").textContent = title || "";
    const body = $("tp-body");
    clear(body);
    show("table-panel");
    return body;
}

$("tp-close").addEventListener("click", () => act("stand"));
$("tp-close").dataset.tip = "";

function seatName(seat) {
    const s = App.view && App.view.seats && App.view.seats[seat];
    return s ? s.name : "";
}

// ─── Setting up (the host) ───────────────────────────────────────────────────

function defaultSetup() {
    const c = App.cfg;
    const last = (App.view && App.view.last) || {};
    const gameType = (last.gameType === "checkers" && c.games.checkers) || !c.games.chess ? "checkers" : "chess";
    return {
        gameType,
        opponent: last.opponent === "ai" ? "ai" : "player",
        level: last.level || (gameType === "checkers" ? c.checkersDefault : c.chessDefault),
        variant: last.variant || c.defaultVariant || "american",
        custom: Object.assign({}, c.presets.american, last.variant === "custom" ? last.rules : {}),
        wager: 0,
    };
}

function currentRules(s) {
    return s.variant === "custom" ? s.custom : App.cfg.presets[s.variant];
}

function seg(options, value, onPick, quiet) {
    const row = el("div", { cls: "seg" });
    for (const o of options) {
        const b = button(o.label, () => onPick(o.value), (value === o.value ? "is-on" : "") + (quiet ? " pg-btn--quiet" : ""));
        if (o.disabled) { b.disabled = true; if (o.tip) b.dataset.tip = o.tip; }
        row.appendChild(b);
    }
    return row;
}

function field(label, control) {
    return el("div", { cls: "field" }, el("div", { cls: "pg-label", text: label }), control);
}

function toggle(label, checked, onChange, tip) {
    const input = el("input", { attrs: { type: "checkbox" } });
    input.checked = !!checked;
    input.addEventListener("change", () => onChange(input.checked));
    const lab = el("label", { cls: "toggle", title: tip }, input, el("span", { cls: "track" }), el("span", { cls: "toggle-text", text: label }));
    return lab;
}

function renderSetup() {
    const c = App.cfg, v = App.view;
    if (!Screens.setup) Screens.setup = defaultSetup();
    const s = Screens.setup;
    const body = panel(v.label, t("setup_title"));

    // saved games this seat can pick up
    if (Screens.resumable.length > 0) {
        body.appendChild(el("div", { cls: "pg-label", text: t("setup_resume") }));
        const rows = el("div", { cls: "rows" });
        for (const g of Screens.resumable) {
            const who = g.opponent ? t("vs_name", g.opponent) : t("vs_ai", levelName(g.gameType, g.aiLevel));
            const sub = [gameName(g.gameType, g.variant), movesText(Math.ceil(g.moves / 2)), g.myTurn ? t("your_turn") : t("their_turn")].join(" · ");
            rows.appendChild(el("div", { cls: "pg-row" },
                el("div", { cls: "row-main" }, el("div", { cls: "row-title", text: who }), el("div", { cls: "row-sub", text: sub })),
                el("div", { cls: "row-actions" }, button(t("resume"), () => act("resume", { gameId: g.id }), "pg-btn--quiet"))));
        }
        body.appendChild(rows);
        body.appendChild(el("hr", { cls: "pg-rule" }));
    }

    // which game
    const games = [];
    if (c.games.chess) games.push({ label: t("game_chess"), value: "chess" });
    if (c.games.checkers) games.push({ label: t("game_checkers"), value: "checkers" });
    if (games.length > 1) {
        body.appendChild(field(t("setup_game"), seg(games, s.gameType, val => {
            s.gameType = val;
            s.level = val === "checkers" ? c.checkersDefault : c.chessDefault;
            renderSetup();
        })));
    }

    // against whom
    const aiOn = s.gameType === "checkers" ? c.checkersAI : c.chessAI;
    const otherTaken = !!v.otherSeated;
    if (!aiOn || otherTaken) s.opponent = "player";
    body.appendChild(field(t("setup_opponent"), seg([
        { label: t("opponent_player"), value: "player" },
        { label: t("opponent_ai"), value: "ai", disabled: !aiOn || otherTaken, tip: otherTaken ? t("ai_seat_taken_tip") : t("ai_off_tip") },
    ], s.opponent, val => { s.opponent = val; renderSetup(); })));

    if (s.opponent === "ai") {
        const sel = el("select", { cls: "pg-select" });
        const levels = s.gameType === "checkers" ? c.checkersLevels : c.chessLevels;
        for (const id of levels) {
            const o = el("option", { text: levelName(s.gameType, id), attrs: { value: id } });
            if (id === s.level) o.selected = true;
            sel.appendChild(o);
        }
        if (!levels.includes(s.level)) s.level = levels[0];
        sel.addEventListener("change", () => { s.level = sel.value; });
        body.appendChild(field(t("setup_level"), sel));
    }

    // the checkers variant and its rules
    if (s.gameType === "checkers") {
        body.appendChild(el("div", { cls: "pg-label", text: t("setup_variant") }));
        const list = el("div");
        for (const id of c.variants) {
            const b = el("button", { cls: "variant" + (s.variant === id ? " is-on" : ""), attrs: { type: "button" },
                on: { click: () => { s.variant = id; renderSetup(); } } },
                el("div", { cls: "variant-name", text: variantName(id) }),
                el("div", { cls: "variant-desc", text: t(`variant_${id}_short`) }));
            list.appendChild(b);
        }
        body.appendChild(list);

        if (s.variant === "custom") {
            const r = s.custom;
            const box = el("div", { cls: "pg-box stack" });
            box.appendChild(toggle(t("custom_must"), r.mustCapture, val => { r.mustCapture = val; if (!val) r.maxCapture = false; renderSetup(); }, t("rule_must_capture")));
            if (r.mustCapture) box.appendChild(toggle(t("custom_max"), r.maxCapture, val => { r.maxCapture = val; renderSetup(); }, t("rule_max_capture")));
            box.appendChild(toggle(t("custom_flying"), r.flyingKings, val => { r.flyingKings = val; renderSetup(); }, t("rule_kings_fly")));
            box.appendChild(toggle(t("custom_back"), r.menCaptureBack, val => { r.menCaptureBack = val; renderSetup(); }, t("rule_men_back")));
            box.appendChild(toggle(t("custom_blocked"), r.blockedLoses, val => { r.blockedLoses = val; renderSetup(); }, t("rule_blocked_loses")));
            box.appendChild(field(t("custom_crown"), seg([
                { label: t("crown_stop"), value: "stop" }, { label: t("crown_continue"), value: "continue" }, { label: t("crown_pass"), value: "pass" },
            ], r.crown, val => { r.crown = val; renderSetup(); }, true)));
            box.appendChild(field(t("custom_first"), seg([
                { label: t("colour_red"), value: "red" }, { label: t("colour_black"), value: "black" },
            ], r.firstMove, val => { r.firstMove = val; renderSetup(); }, true)));
            box.appendChild(field(t("custom_notation"), seg([
                { label: t("notation_numeric"), value: "numeric" }, { label: t("notation_algebraic"), value: "algebraic" },
            ], r.notation || "algebraic", val => { r.notation = val; renderSetup(); }, true)));
            body.appendChild(box);
        }
        body.appendChild(el("div", { cls: "pg-label", text: t("setup_rules") }));
        body.appendChild(rulesCard("checkers", currentRules(s)));
    }

    // a wager, against a player
    const w = c.wagers;
    if (s.opponent === "player" && w.enabled) {
        const input = el("input", { cls: "pg-input pg-num", attrs: { type: "number", min: "0", max: String(w.max), step: "1", placeholder: "0" } });
        input.value = s.wager > 0 ? String(s.wager) : "";
        input.addEventListener("input", () => {
            const n = Math.floor(Number(input.value) || 0);
            s.wager = Math.max(0, Math.min(w.max, n));
        });
        body.appendChild(field(t("setup_wager"), input));
        body.appendChild(el("div", { cls: "hint-line", text: t("wager_explain", money(w.min, w.currency), money(w.max, w.currency)) }));
    }

    // go
    const startText = s.opponent === "ai" ? t("start_ai")
        : (otherTaken ? t("start_invite", seatName(v.mySeat === "white" ? "black" : "white")) : t("start_wait"));
    const actions = el("div", { cls: "btn-row" },
        button(startText, () => {
            const opts = { gameType: s.gameType, opponent: s.opponent, level: s.level };
            if (s.gameType === "checkers") { opts.variant = s.variant; if (s.variant === "custom") opts.rules = s.custom; }
            if (s.opponent === "player" && s.wager > 0) opts.wager = s.wager;
            act("create", { opts });
        }, "pg-btn--primary"));
    body.appendChild(el("div", { cls: "sticky-actions stack" }, actions, el("div", { cls: "btn-row" },
        button(t("my_record"), () => Record.open()),
        button(t("stand_up"), () => act("stand"), "pg-btn--danger"))));
}

function loadResumable() {
    ask("resumable").then(list => {
        Screens.resumable = Array.isArray(list) ? list : [];
        if (App.view && App.view.phase === "setup" && App.view.host === App.view.mySeat && !Screens.overOpen) renderSetup();
    });
}

// ─── Waiting and invitations ─────────────────────────────────────────────────

function renderWaiting() {
    const v = App.view;
    const host = seatName(v.host);
    const body = panel(v.label, t("waiting_title"));
    body.appendChild(el("p", { cls: "pg-text", text: t("waiting_host", host) }));
    body.appendChild(el("div", { cls: "btn-row panel-actions" },
        button(t("my_record"), () => Record.open()),
        button(t("stand_up"), () => act("stand"), "pg-btn--danger")));
}

function wagerLine(wager) {
    return t("wager_line", money(wager.amount, wager.currency), money(wager.amount * 2, wager.currency));
}

function renderInvite() {
    const v = App.view, inv = v.invite;
    const body = panel(v.label, t("invite_title"));
    body.appendChild(el("p", { cls: "pg-text", text: t("invite_text", inv.hostName, gameName(inv.gameType, inv.variant)) }));
    const myColour = colourName(inv.gameType, v.mySeat);
    body.appendChild(el("div", { cls: "pills" },
        el("span", { cls: "pg-pill pg-pill--accent", text: gameName(inv.gameType, inv.variant) }),
        el("span", { cls: "pg-pill", text: t("you_play", myColour) })));
    if (inv.resumeId) body.appendChild(el("p", { cls: "pg-text", text: t("invite_resume") }));
    if (inv.gameType === "checkers") body.appendChild(el("div", { cls: "pg-text", text: t(`variant_${inv.variant}_desc`) }));
    body.appendChild(rulesCard(inv.gameType, inv.rules));
    if (inv.wager) body.appendChild(el("div", { cls: "pg-box pg-text", text: wagerLine(inv.wager) }));
    body.appendChild(el("div", { cls: "btn-row panel-actions" },
        button(inv.wager ? t("accept_stake", money(inv.wager.amount, inv.wager.currency)) : t("accept"),
            () => act("answer", { accept: true }), "pg-btn--primary" + (inv.wager ? " btn-num" : "")),
        button(t("decline"), () => act("answer", { accept: false }))));
    body.appendChild(el("div", { cls: "btn-row" }, button(t("stand_up"), () => act("stand"), "pg-btn--danger")));
}

function renderInviteWait() {
    const v = App.view, inv = v.invite;
    const other = v.mySeat === "white" ? "black" : "white";
    const guest = v.seats[other];
    const body = panel(v.label, gameName(inv.gameType, inv.variant));
    let text;
    if (guest) text = t("invite_wait_answer", guest.name);
    else if (inv.opponentName) text = t("invite_wait_named", inv.opponentName);
    else text = t("invite_wait_seat");
    body.appendChild(el("p", { cls: "pg-text", text }));
    body.appendChild(el("div", { cls: "pills" }, el("span", { cls: "pg-pill", text: t("you_play", colourName(inv.gameType, v.mySeat)) })));
    body.appendChild(rulesCard(inv.gameType, inv.rules));
    if (inv.wager) body.appendChild(el("div", { cls: "pg-box pg-text", text: wagerLine(inv.wager) }));
    body.appendChild(el("div", { cls: "btn-row panel-actions" },
        button(t("cancel"), () => act("cancelInvite")),
        button(t("stand_up"), () => act("stand"), "pg-btn--danger")));
}

// ─── The end of a game ───────────────────────────────────────────────────────

function reasonText(sm) {
    if (sm.reason === "saved") {
        const other = sm.mySeat === "white" ? "black" : "white";
        return t("reason_saved", sm.names[other] || colourName(sm.gameType, other));
    }
    const winnerSeat = sm.result === "white" || sm.result === "black" ? sm.result : null;
    const loserSeat = winnerSeat ? (winnerSeat === "white" ? "black" : "white") : null;
    const loser = loserSeat ? (sm.names[loserSeat] || colourName(sm.gameType, loserSeat)) : "";
    const winner = winnerSeat ? (sm.names[winnerSeat] || colourName(sm.gameType, winnerSeat)) : "";
    const key = `reason_${sm.reason}`;
    return App.strings[key] != null ? t(key, loser, winner) : "";
}

function renderOver() {
    const sm = App.over, v = App.view;
    const body = panel(v ? v.label : "", gameName(sm.gameType, sm.variant));
    let cls = "draw", title;
    if (sm.status === "abandoned") title = t("over_abandoned");
    else if (sm.status === "active") title = t("over_saved");
    else if (sm.result === "draw") title = t("over_draw");
    else if (sm.result === sm.mySeat) { title = t("over_won"); cls = "win"; }
    else { title = t("over_lost"); cls = "loss"; }
    body.appendChild(el("h2", { cls: "result " + cls, text: title }));
    const why = reasonText(sm);
    if (why) body.appendChild(el("p", { cls: "pg-text", text: why }));
    body.appendChild(el("div", { cls: "row-sub", text: movesText(Math.ceil((sm.moves || 0) / 2)) }));
    const r = App.overRating;
    if (r) {
        const sign = r.change > 0 ? "+" + r.change : String(r.change);
        body.appendChild(el("div", { cls: "pg-box pg-num " + (r.change > 0 ? "pg-gain" : r.change < 0 ? "pg-loss" : ""),
            text: t("over_rating", r.before, r.after, sign) }));
    }
    if (sm.wager && sm.status === "completed") {
        let line;
        if (sm.result === "draw") line = t("over_wager_back", money(sm.wager.amount, sm.wager.currency));
        else if (sm.result === sm.mySeat) line = t("over_wager_won", money(sm.wager.amount * 2, sm.wager.currency));
        else line = t("over_wager_lost", money(sm.wager.amount, sm.wager.currency));
        body.appendChild(el("div", { cls: "pg-box pg-text", text: line }));
    } else if (sm.wager) {
        body.appendChild(el("div", { cls: "pg-box pg-text", text: t("over_wager_back", money(sm.wager.amount, sm.wager.currency)) }));
    }
    const row = el("div", { cls: "btn-row panel-actions" });
    if (sm.gameId && sm.moves > 0) row.appendChild(button(t("review"), () => Record.review(sm.gameId)));
    row.appendChild(button(t("continue"), () => {
        Screens.overOpen = false;
        App.over = null;
        act("overDone");
        Screens.render();
    }, "pg-btn--primary"));
    body.appendChild(row);
    body.appendChild(el("div", { cls: "btn-row" }, button(t("stand_up"), () => act("stand"), "pg-btn--danger")));
}

// ─── Which one ───────────────────────────────────────────────────────────────

Screens.render = function () {
    const v = App.view;
    if (!v) return;
    if (Screens.overOpen && App.over) {
        Play.hide();
        return renderOver();
    }
    if (v.phase === "playing") {
        hide("table-panel");
        return Play.show();
    }
    Play.hide();
    const host = v.host === v.mySeat;
    if (v.phase === "invite") return host ? renderInviteWait() : renderInvite();
    if (host) return renderSetup();
    return renderWaiting();
};

Handlers.table = m => {
    const before = App.view;
    App.view = m.view;
    // a new game started: whatever was on screen from the last one goes
    if (m.view.phase === "playing" && (!before || before.phase !== "playing")) {
        Screens.overOpen = false; App.over = null; Screens.setup = null;
        playSound("start");
    }
    if (m.view.phase === "setup" && m.view.host === m.view.mySeat && (!before || before.phase !== "setup")) {
        Screens.setup = null;
        Screens.resumable = [];
        loadResumable();
    }
    Screens.render();
};

Handlers.rated = m => {
    App.overRating = m.rated;
    if (Screens.overOpen && App.over) Screens.render();
};

Handlers.over = m => {
    App.overRating = null;
    App.over = m.summary;
    Screens.overOpen = true;
    Play.onOver(m.summary);
    Screens.render();
};
