/* ============================================================================
   poggy_markets — shop wages
   ----------------------------------------------------------------------------
   Two small things, fed by the same status message from server/wages.lua:

     * the clock strip in the manager's header, for hired staff with a back
       office;
     * the clock card, shown instead of the manager to staff without one.

   Every word and amount arrives finished from the server (translations.lua),
   and every action is only a request: the server decides.  The card borrows
   the text prompt's box (js/prompt.js) so skins and themes reach it.
   ============================================================================ */

(function () {
    'use strict';

    var RESOURCE = GetParentResourceName();

    var card = null, parts = {};
    var cardOpen = false;
    var status = null;

    function post(name) {
        return fetch('https://' + RESOURCE + '/' + name, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: '{}'
        }).catch(function () { /* the game is gone; nothing to do */ });
    }

    function clockAction() {
        if (status && status.onDuty) post('clockOut');
        else if (status && status.canClockIn) post('clockIn');
    }

    function closeCard() {
        if (cardOpen) post('clerkClose');
    }

    function buildCard() {
        if (card) return;
        if (window.PMPromptStyle) window.PMPromptStyle();

        card = document.createElement('div');
        card.className = 'pm-prompt-overlay';
        card.innerHTML =
            '<div class="pm-prompt-box wage-card">' +
                '<h3 class="pm-prompt-title"></h3>' +
                '<p class="pm-prompt-text" data-part="state"></p>' +
                '<p class="pm-prompt-text" data-part="rate"></p>' +
                '<p class="pm-prompt-text" data-part="today"></p>' +
                '<p class="pm-prompt-text" data-part="shift"></p>' +
                '<div class="pm-prompt-actions wage-card-actions">' +
                    '<button type="button" class="pm-prompt-btn cancel" data-part="close"></button>' +
                    '<button type="button" class="pm-prompt-btn cancel" data-part="browse"></button>' +
                    '<button type="button" class="pm-prompt-btn confirm" data-part="clock"></button>' +
                '</div>' +
            '</div>';
        document.body.appendChild(card);

        parts.title = card.querySelector('.pm-prompt-title');
        ['state', 'rate', 'today', 'shift', 'close', 'browse', 'clock'].forEach(function (name) {
            parts[name] = card.querySelector('[data-part="' + name + '"]');
        });

        parts.clock.addEventListener('click', clockAction);
        parts.browse.addEventListener('click', function () { post('clerkBrowse'); });
        parts.close.addEventListener('click', closeCard);
    }

    function setLine(el, text) {
        el.textContent = text || '';
        el.style.display = text ? '' : 'none';
    }

    function renderCard() {
        if (!card || !status) return;
        var t = status.text || {};
        parts.title.textContent = status.shopName || t.title || '';
        setLine(parts.state, t.state);
        setLine(parts.rate, t.rate);
        setLine(parts.today, t.today);
        setLine(parts.shift, t.shift);
        parts.close.textContent = t.close || '';
        parts.browse.textContent = t.browse || '';
        parts.clock.textContent = status.onDuty ? t.clockOut : t.clockIn;
        parts.clock.disabled = !status.onDuty && !status.canClockIn;
    }

    function renderStrip() {
        var strip = document.getElementById('wage-strip');
        if (!strip) return;
        if (!status || !status.enabled || !status.employed) {
            strip.classList.add('hidden');
            return;
        }
        var t = status.text || {};
        document.getElementById('wage-strip-text').textContent =
            [t.state, t.rate, t.today].filter(Boolean).join(' · ');
        var btn = document.getElementById('wage-strip-btn');
        btn.textContent = status.onDuty ? t.clockOut : t.clockIn;
        btn.disabled = !status.onDuty && !status.canClockIn;
        strip.classList.remove('hidden');
    }

    window.addEventListener('message', function (event) {
        var d = event.data;
        if (!d) return;
        switch (d.type) {
            case 'open':          // a manager opening: its answer follows
                status = null;
                renderStrip();
                break;
            case 'clerkOpen':
                buildCard();
                status = d.status;
                cardOpen = true;
                renderCard();
                card.classList.add('show');
                break;
            case 'clerkClose':
                cardOpen = false;
                if (card) card.classList.remove('show');
                break;
            case 'wageStatus':
                status = d.status;
                renderCard();
                renderStrip();
                break;
        }
    });

    // This script loads at the end of the body, so the button already exists.
    var stripBtn = document.getElementById('wage-strip-btn');
    if (stripBtn) stripBtn.addEventListener('click', clockAction);

    document.addEventListener('keyup', function (e) {
        if (e.key === 'Escape' && cardOpen) closeCard();
    });
}());
