// ============================================================================
// record.js — "My Record" (games, totals, opponents) and the game review
// ============================================================================
"use strict";

const Record = {
    tab: "games", page: 1, gameType: "all", hideAI: true, data: null, loading: false, opened: false,
    lb: null, lbPage: 1, lbLoading: false,
    rv: null, at: 0,
};

function outcomePill(outcome) {
    const cls = { won: "pg-pill--success", lost: "pg-pill--danger", draw: "pg-pill--accent", unfinished: "pg-pill--info" }[outcome] || "";
    return el("span", { cls: "pg-pill " + cls, text: t(`outcome_${outcome}`) });
}

function opponentText(g) {
    return g.opponent ? t("vs_name", g.opponent) : t("vs_ai", levelName(g.gameType, g.aiLevel));
}

function reasonShort(reason) {
    const k = `reason_short_${reason}`;
    return reason && App.strings[k] != null ? t(k) : "";
}

// ─── Opening ─────────────────────────────────────────────────────────────────

Record.open = function () {
    if (!Record.opened) { Record.hideAI = !!(App.cfg && App.cfg.hideAI); Record.opened = true; }
    Record.page = 1;
    $("rec-title").textContent = t("record_title");
    show("record-panel");
    Record.renderChrome();
    Record.load();
};

Record.close = function () { hide("record-panel"); };
$("rec-close").addEventListener("click", Record.close);

Record.load = function () {
    Record.loading = true;
    Record.renderBody();
    ask("record", { opts: { gameType: Record.gameType, hideAI: Record.hideAI }, page: Record.page }).then(data => {
        Record.loading = false;
        Record.data = data && data.games ? data : { games: [], total: 0, page: 1, pageSize: 6, stats: {}, opponents: [] };
        Record.renderBody();
    });
};

// ─── Filters and tabs ────────────────────────────────────────────────────────

Record.renderChrome = function () {
    const f = $("rec-filters");
    clear(f);
    f.appendChild(seg([
        { label: t("filter_all"), value: "all" },
        { label: t("game_chess"), value: "chess" },
        { label: t("game_checkers"), value: "checkers" },
    ], Record.gameType, v => {
        Record.gameType = v; Record.page = 1; Record.renderChrome();
        if (Record.tab === "leaderboard") { Record.lbPage = 1; Record.loadLeaderboard(); } else Record.load();
    }));
    f.appendChild(toggle(t("filter_hide_ai"), Record.hideAI, v => { Record.hideAI = v; Record.page = 1; Record.load(); }));

    const tabs = $("rec-tabs");
    clear(tabs);
    for (const id of ["games", "summary", "opponents", "leaderboard"]) {
        tabs.appendChild(el("button", { cls: "pg-tab" + (Record.tab === id ? " is-active" : ""), text: t(`tab_${id}`),
            attrs: { type: "button" }, on: { click: () => {
                Record.tab = id; Record.renderChrome();
                if (id === "leaderboard") { Record.lbPage = 1; Record.loadLeaderboard(); } else Record.renderBody();
            } } }));
    }
};

// ─── The tabs ────────────────────────────────────────────────────────────────

function renderGames(body, d) {
    if (!d.games.length) { body.appendChild(el("div", { cls: "empty", text: t("record_none") })); return; }
    const rows = el("div", { cls: "rows" });
    for (const g of d.games) {
        const cv = el("canvas", { attrs: { width: "160", height: "160" } });
        const kind = g.gameType === "checkers" ? "checkers" : "chess";
        Board.draw(cv, { kind, cells: kind === "chess" ? Board.fromFen(g.state) : Board.fromCheckers(g.state), flip: g.seat === "black" });
        const pills = el("div", { cls: "pills" },
            outcomePill(g.outcome),
            el("span", { cls: "pg-pill", text: gameName(g.gameType, g.variant) }),
            el("span", { cls: "pg-pill", text: t("you_played", colourName(g.gameType, g.seat)) }));
        if (g.wager) pills.appendChild(el("span", { cls: "pg-pill pg-pill--warning", text: t("stake_pill", money(g.wager.amount, g.wager.currency)) }));
        if (g.ratingChange != null) {
            const c = g.ratingChange;
            pills.appendChild(el("span", { cls: "pg-pill " + (c > 0 ? "pg-pill--success" : c < 0 ? "pg-pill--danger" : ""),
                text: t("rating_change", c > 0 ? "+" + c : String(c)) }));
        }
        const sub = [g.date, movesText(Math.ceil(g.moves / 2)), reasonShort(g.reason)].filter(Boolean).join(" · ");
        const card = el("div", { cls: "pg-row card" },
            cv,
            el("div", { cls: "row-main" }, el("div", { cls: "row-title", text: opponentText(g) }), el("div", { cls: "row-sub pg-num", text: sub }), pills),
            button(t("review"), () => Record.review(g.id), "pg-btn--quiet"));
        rows.appendChild(card);
    }
    body.appendChild(rows);

    const pages = Math.max(1, Math.ceil((d.total || 0) / (d.pageSize || 6)));
    const prev = button(t("page_prev"), () => { Record.page--; Record.load(); });
    const next = button(t("page_next"), () => { Record.page++; Record.load(); });
    prev.disabled = Record.page <= 1;
    next.disabled = Record.page >= pages;
    body.appendChild(el("div", { cls: "pager" }, prev, el("span", { cls: "pg-num", text: t("page_of", Record.page, pages) }), next));
}

function tile(label, value, cls) {
    return el("div", { cls: "tile " + (cls || "") }, el("div", { cls: "v", text: String(value) }), el("div", { cls: "l", text: label }));
}

function renderSummary(body, d) {
    const s = d.stats || {};
    if (d.ratings) {
        const tiles = el("div", { cls: "tiles", attrs: { style: "grid-template-columns: repeat(2, 1fr); margin-bottom: .5rem" } });
        for (const kind of ["chess", "checkers"]) {
            const r = d.ratings[kind];
            const tl = tile(t("stat_rating", t(`game_${kind}`)), r.rating);
            tl.appendChild(el("div", { cls: "row-sub pg-num", text: t("lb_peak", r.peak) + (r.games < 30 ? " · " + t("provisional") : "") }));
            tiles.appendChild(tl);
        }
        body.appendChild(tiles);
    }
    if (!s.total) { body.appendChild(el("div", { cls: "empty", text: t("record_none") })); return; }
    const done = (s.wins || 0) + (s.draws || 0) + (s.losses || 0);
    body.appendChild(el("div", { cls: "tiles" },
        tile(t("stat_wins"), s.wins || 0, "win"), tile(t("stat_draws"), s.draws || 0), tile(t("stat_losses"), s.losses || 0, "loss"),
        tile(t("stat_played"), s.total || 0), tile(t("stat_vs_ai"), s.vs_ai || 0), tile(t("stat_unfinished"), (s.unfinished || 0) + (s.abandoned || 0))));
    if (done > 0) {
        const pct = n => (100 * n / done).toFixed(1) + "%";
        const bar = el("div", { cls: "split" }, el("i"), el("i"), el("i"));
        bar.children[0].style.width = pct(s.wins || 0);
        bar.children[1].style.width = pct(s.draws || 0);
        bar.children[2].style.width = pct(s.losses || 0);
        body.appendChild(el("div", { cls: "stack", attrs: { style: "margin-top:.8rem" } },
            el("div", { cls: "pg-text pg-num", text: t("win_rate", Math.round(100 * (s.wins || 0) / done), done) }), bar));
    }
    body.appendChild(el("div", { cls: "hint-line", attrs: { style: "margin-top:.6rem" }, text: t("summary_note") }));
}

function renderOpponents(body, d) {
    const list = d.opponents || [];
    if (!list.length) { body.appendChild(el("div", { cls: "empty", text: t("record_none") })); return; }
    const table = el("table", { cls: "opp" });
    const head = el("tr", null, el("th", { text: t("col_opponent") }), el("th", { cls: "n", text: t("col_games") }),
        el("th", { cls: "n", text: t("col_w") }), el("th", { cls: "n", text: t("col_d") }), el("th", { cls: "n", text: t("col_l") }));
    table.appendChild(el("thead", null, head));
    const tb = el("tbody");
    for (const o of list) {
        tb.appendChild(el("tr", null,
            el("td", { text: o.name || t("vs_ai", levelName(null, o.aiLevel)) }),
            el("td", { cls: "n", text: String(o.games) }), el("td", { cls: "n pg-gain", text: String(o.wins) }),
            el("td", { cls: "n", text: String(o.draws) }), el("td", { cls: "n pg-loss", text: String(o.losses) })));
    }
    table.appendChild(tb);
    body.appendChild(table);
}

// ─── Leaderboard ─────────────────────────────────────────────────────────────

Record.loadLeaderboard = function () {
    const kind = Record.gameType === "checkers" ? "checkers" : "chess";
    Record.lbLoading = true;
    Record.renderBody();
    ask("leaderboard", { gameType: kind, page: Record.lbPage }).then(d => {
        Record.lbLoading = false;
        Record.lb = d && d.rows ? d : { rows: [], total: 0, page: 1, pages: 1, minGames: 5, enabled: true };
        Record.renderBody();
    });
};

function renderLeaderboard(body) {
    const d = Record.lb;
    if (!d.enabled) { body.appendChild(el("div", { cls: "empty", text: t("lb_off") })); return; }
    const kind = t(`game_${d.gameType || "chess"}`);
    if (d.me) {
        body.appendChild(el("div", { cls: "pg-box pg-num lb-me", text: d.me.rank
            ? t("lb_mine", kind, d.me.rating, d.me.rank, d.total)
            : t("lb_mine_unranked", kind, d.me.rating, d.me.needed) }));
    }
    if (!d.rows.length) {
        body.appendChild(el("div", { cls: "empty", text: t("lb_none") }));
    } else {
        const table = el("table", { cls: "opp" });
        table.appendChild(el("thead", null, el("tr", null,
            el("th", { cls: "n", text: t("lb_rank") }), el("th", { text: t("lb_player") }), el("th", { cls: "n", text: t("lb_rating") }),
            el("th", { cls: "n", text: t("lb_games") }), el("th", { cls: "n", text: t("col_w") }),
            el("th", { cls: "n", text: t("col_d") }), el("th", { cls: "n", text: t("col_l") }))));
        const tb = el("tbody");
        for (const r of d.rows) {
            tb.appendChild(el("tr", { cls: r.me ? "is-me" : "" },
                el("td", { cls: "n", text: String(r.rank) }), el("td", { text: r.name || "?" }),
                el("td", { cls: "n", text: String(r.rating) }), el("td", { cls: "n", text: String(r.games) }),
                el("td", { cls: "n", text: String(r.wins) }), el("td", { cls: "n", text: String(r.draws) }),
                el("td", { cls: "n", text: String(r.losses) })));
        }
        table.appendChild(tb);
        body.appendChild(table);
        const prev = button(t("page_prev_lb"), () => { Record.lbPage--; Record.loadLeaderboard(); });
        const next = button(t("page_next_lb"), () => { Record.lbPage++; Record.loadLeaderboard(); });
        prev.disabled = Record.lbPage <= 1;
        next.disabled = Record.lbPage >= (d.pages || 1);
        body.appendChild(el("div", { cls: "pager" }, prev, el("span", { cls: "pg-num", text: t("page_of", Record.lbPage, d.pages || 1) }), next));
    }
    body.appendChild(el("div", { cls: "hint-line", attrs: { style: "margin-top:.6rem" }, text: t("lb_note", d.minGames) }));
}

Record.renderBody = function () {
    const body = $("rec-body");
    clear(body);
    if (Record.tab === "leaderboard") {
        if (Record.lbLoading || !Record.lb) { body.appendChild(el("div", { cls: "empty", text: t("loading") })); return; }
        return renderLeaderboard(body);
    }
    if (Record.loading || !Record.data) { body.appendChild(el("div", { cls: "empty", text: t("loading") })); return; }
    const d = Record.data;
    if (Record.tab === "games") renderGames(body, d);
    else if (Record.tab === "summary") renderSummary(body, d);
    else renderOpponents(body, d);
};

// ─── Review ──────────────────────────────────────────────────────────────────

Record.review = function (gameId) {
    ask("review", { gameId }).then(rv => {
        if (!rv || !rv.id) return;
        Record.rv = rv;
        Record.at = rv.moves.length;   // open on the final position
        $("rv-kicker").textContent = rv.date || "";
        const other = rv.mySeat === "white" ? "black" : "white";
        const opp = rv.aiSeat ? t("vs_ai", levelName(rv.gameType, rv.aiLevel)) : t("vs_name", (rv.names && rv.names[other]) || "?");
        $("rv-title").textContent = opp;
        const pills = $("rv-pills");
        clear(pills);
        pills.appendChild(outcomePill(rv.outcome));
        pills.appendChild(el("span", { cls: "pg-pill", text: gameName(rv.gameType, rv.variant) }));
        pills.appendChild(el("span", { cls: "pg-pill", text: t("you_played", colourName(rv.gameType, rv.mySeat)) }));
        if (rv.gameType === "checkers" && rv.rules) {
            const b = button(t("rules_button"), () => Screens.showRulesModal("checkers", rv.variant, rv.rules), "pg-btn--quiet");
            pills.appendChild(b);
        }
        const why = reasonShort(rv.reason) || "";
        $("rv-result").textContent = why ? why.charAt(0).toUpperCase() + why.slice(1) + "." : "";
        $("rv-note").textContent = rv.complete ? "" : t("review_partial");
        show("review-panel");
        Record.renderReview();
    });
};

Record.closeReview = function () { hide("review-panel"); Record.rv = null; };
$("rv-close").addEventListener("click", Record.closeReview);

function reviewCells(rv, at) {
    const src = at === 0 ? rv.start : (rv.moves[at - 1] && rv.moves[at - 1].s);
    if (!src && at === rv.moves.length && rv.final) return rv.gameType === "chess" ? Board.fromFen(rv.final) : Board.fromCheckers(rv.final);
    return rv.gameType === "chess" ? Board.fromFen(src) : Board.fromCheckers(src);
}

Record.renderReview = function () {
    const rv = Record.rv;
    if (!rv) return;
    const at = Record.at;
    const notation = rv.gameType === "checkers" && rv.rules && rv.rules.notation === "numeric" ? "numbers" : "letters";
    Board.draw($("rv-board"), { kind: rv.gameType, cells: reviewCells(rv, at), flip: rv.mySeat === "black", coords: notation });

    const list = $("rv-moves");
    clear(list);
    const first = rv.moves[0] ? rv.moves[0].c : "white";
    let row = null, n = 0;
    rv.moves.forEach((m, i) => {
        if (m.c === first || !row) {
            n++;
            row = el("li", null, el("span", { cls: "num", text: n + "." }));
            if (m.c !== first) row.appendChild(el("span", { cls: "mv", text: "…" }));
            list.appendChild(row);
        }
        const mv = el("span", { cls: "mv" + (i + 1 === at ? " is-at" : ""), text: m.n, on: { click: () => { Record.at = i + 1; Record.renderReview(); } } });
        row.appendChild(mv);
        if (m.c !== first) row = null;
    });
    const cur = list.querySelector(".is-at");
    if (cur) cur.scrollIntoView({ block: "nearest" });
    $("rv-first").disabled = at <= 0; $("rv-prev").disabled = at <= 0;
    $("rv-next").disabled = at >= rv.moves.length; $("rv-last").disabled = at >= rv.moves.length;
};

const step = d => () => {
    if (!Record.rv) return;
    Record.at = Math.max(0, Math.min(Record.rv.moves.length, d === "first" ? 0 : d === "last" ? Record.rv.moves.length : Record.at + d));
    Record.renderReview();
};
$("rv-first").addEventListener("click", step("first"));
$("rv-prev").addEventListener("click", step(-1));
$("rv-next").addEventListener("click", step(1));
$("rv-last").addEventListener("click", step("last"));
document.addEventListener("keydown", e => {
    if ($("review-panel").classList.contains("hidden")) return;
    if (e.key === "ArrowLeft") step(-1)();
    else if (e.key === "ArrowRight") step(1)();
    else if (e.key === "Home") step("first")();
    else if (e.key === "End") step("last")();
});
Board.onRedraw(() => {
    if (Record.rv) Record.renderReview();
    if (!$("record-panel").classList.contains("hidden")) Record.renderBody();
});
