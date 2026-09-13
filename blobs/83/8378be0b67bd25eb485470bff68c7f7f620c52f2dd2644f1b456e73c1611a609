/*
    poggy_core — menu and text input, page side. The Lua side is client/cl_menu.lua.

    Messages from Lua (SendNUIMessage):
        { action: 'menu.open',  id, title, subtitle?, items: [{ label, right?, desc?, disabled? }], closeText?,
                                position?, margin? }
        { action: 'input.open', id, title, placeholder?, default?, maxLength?, numeric?, submitText?, closeText?,
                                position?, margin? }
        { action: 'close' }                      hide everything, answer nothing

    `position` is one of center | left | right | top-left | top-right |
    bottom-left | bottom-right (PoggyCoreConfig.Ui.Position; anything else is
    'center') and `margin` the distance in px from the screen edge
    (PoggyCoreConfig.Ui.Margin, 40). Lua sends both on every open.

    Callbacks to Lua (POST https://<resource>/<name>), each carrying the id it
    answers, so a reply to a menu that was replaced can be told apart and dropped:
        ui:select  { id, index }                 1-based index of the chosen item
        ui:submit  { id, value }                 the text typed (a number when numeric)
        ui:close   { id }                        Escape, Backspace, right-click or the close button
        ui:sound   { kind }                      'move' | 'select' | 'close' — played by Lua

    Everything else — which item is highlighted, scrolling, validation — is
    page-local. Values are never sent to the page: Lua keeps the caller's items
    and answers with the original table, so a value can be anything Lua likes.
*/
(function () {
    'use strict';

    var RESOURCE = 'poggy_core';
    try { RESOURCE = GetParentResourceName(); } catch (e) { /* not inside the game */ }

    function post(name, body) {
        try {
            fetch('https://' + RESOURCE + '/' + name, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json; charset=UTF-8' },
                body: JSON.stringify(body || {}),
            }).catch(function () {});
        } catch (e) { /* ignore */ }
    }

    function sound(kind) { post('ui:sound', { kind: kind }); }

    var $ = function (id) { return document.getElementById(id); };

    var menuEl     = $('pg-menu');
    var menuTitle  = $('pg-menu-title');
    var menuSub    = $('pg-menu-subtitle');
    var menuItems  = $('pg-menu-items');
    var menuDesc   = $('pg-menu-desc');          // the clamped text
    var menuDescBox = menuDesc.parentNode;       // the reserved area around it
    var menuCount  = $('pg-menu-count');
    var menuClose  = $('pg-menu-close');

    var inputEl     = $('pg-input');
    var inputTitle  = $('pg-input-title');
    var inputField  = $('pg-input-field');
    var inputNote   = $('pg-input-note');
    var inputSubmit = $('pg-input-submit');
    var inputCancel = $('pg-input-cancel');

    // What is open right now: null, or { kind: 'menu'|'input', id, ... }.
    var current = null;

    // ----------------------------------------------------------- placement --
    //
    // A panel is pinned by one edge, in pixels, never centred with a
    // translateY(-50%): a centred transform shifts the whole panel by half of
    // any change in its height, so the rows would creep up and down as the
    // description under them wrapped to more or fewer lines. Instead:
    //
    //   center / left / right   top edge fixed, at the point that centres the
    //                           panel's steady height (its parts that do not
    //                           change per item) — growth runs downward
    //   top-*                   top edge at the margin — growth runs downward
    //   bottom-*                bottom edge at the margin — growth would run
    //                           upward, into the rows, so the description area
    //                           is held at its full 5 lines (.pg-anchor-bottom)
    //                           and the panel does not change height at all
    //
    // The description area itself is reserved at 3–5 lines by the CSS, so the
    // only growth left is those two lines, and it never touches the rows.

    var POSITIONS = {
        'center': 1, 'left': 1, 'right': 1,
        'top-left': 1, 'top-right': 1, 'bottom-left': 1, 'bottom-right': 1,
    };
    var DEFAULT_MARGIN = 40;

    function readPlacement(msg) {
        var pos = String(msg.position == null ? '' : msg.position).toLowerCase().replace(/_/g, '-');
        var m = Number(msg.margin);
        return {
            position: POSITIONS[pos] ? pos : 'center',
            margin: isFinite(m) && m >= 0 ? Math.round(m) : DEFAULT_MARGIN,
        };
    }

    function placePanel(panel, placement, steadyHeight) {
        var pos = placement.position;
        var m   = placement.margin;
        var vw  = window.innerWidth;
        var vh  = window.innerHeight;
        var s   = panel.style;
        s.top = ''; s.bottom = ''; s.left = ''; s.right = ''; s.maxHeight = '';
        panel.classList.toggle('pg-anchor-bottom', pos.indexOf('bottom-') === 0);

        if (pos.indexOf('bottom-') === 0) {
            s.bottom    = m + 'px';
            s.maxHeight = Math.max(120, vh - 2 * m) + 'px';
        } else {
            var top = pos.indexOf('top-') === 0 ? m : Math.max(m, Math.round((vh - steadyHeight) / 2));
            s.top       = top + 'px';
            s.maxHeight = Math.max(120, vh - top - m) + 'px';
        }

        if (/left$/.test(pos))       s.left  = m + 'px';
        else if (/right$/.test(pos)) s.right = m + 'px';
        else                         s.left  = Math.max(0, Math.round((vw - panel.offsetWidth) / 2)) + 'px';
    }

    // The menu's height with the description area at its reserved minimum:
    // what the panel measures whichever item is highlighted, before the 3→5
    // line allowance. Measured with no height cap so a long list is not
    // mistaken for a short one.
    function menuSteadyHeight() {
        menuEl.style.maxHeight = '';
        var h = menuEl.offsetHeight;
        if (menuEl.classList.contains('has-desc')) {
            var min = parseFloat(window.getComputedStyle(menuDescBox).minHeight) || 0;
            h = h - menuDescBox.offsetHeight + min;
        }
        return h;
    }

    function placeCurrent() {
        if (!current) return;
        if (current.kind === 'menu') {
            current.steadyHeight = menuSteadyHeight();
            placePanel(menuEl, current.placement, current.steadyHeight);
        } else if (current.kind === 'input') {
            inputEl.style.maxHeight = '';
            placePanel(inputEl, current.placement, inputEl.offsetHeight);
        }
    }

    // ---------------------------------------------------------------- menu --

    function clampIndex(i, n) {
        if (n <= 0) return -1;
        if (i < 0) return 0;
        if (i >= n) return n - 1;
        return i;
    }

    // First enabled row at or after `from`, walking in `dir` (+1 / -1), wrapping.
    function nextEnabled(items, from, dir) {
        var n = items.length;
        if (n === 0) return -1;
        var i = from;
        for (var step = 0; step < n; step++) {
            i = (i + dir + n) % n;
            if (!items[i].disabled) return i;
        }
        return -1;
    }

    function renderMenu(state) {
        menuTitle.textContent = state.title;
        menuSub.textContent   = state.subtitle || '';
        menuClose.textContent = state.closeText || 'Close';
        menuItems.innerHTML   = '';

        state.items.forEach(function (item, i) {
            var row = document.createElement('div');
            row.className = 'pg-item' + (item.disabled ? ' disabled' : '');
            row.dataset.index = String(i);

            var label = document.createElement('div');
            label.className = 'pg-label';
            label.textContent = item.label;
            row.appendChild(label);

            var right = document.createElement('div');
            right.className = 'pg-right';
            right.textContent = item.right || '';
            row.appendChild(right);

            row.addEventListener('mouseenter', function () {
                if (item.disabled || state.selected === i) return;
                select(state, i, false);
            });
            row.addEventListener('click', function (e) {
                e.preventDefault();
                if (item.disabled) return;
                select(state, i, false);
                choose(state);
            });
            row.addEventListener('contextmenu', function (e) {
                e.preventDefault();
                closeCurrent();
            });

            menuItems.appendChild(row);
        });

        // Start on the first enabled row.
        var first = state.items.findIndex(function (it) { return !it.disabled; });
        state.selected = -1;
        select(state, first, true, true);
    }

    function select(state, i, _unused, instant) {
        var rows = menuItems.children;
        if (state.selected >= 0 && rows[state.selected]) rows[state.selected].classList.remove('selected');
        state.selected = clampIndex(i, state.items.length);
        var item = state.selected >= 0 ? state.items[state.selected] : null;
        if (state.selected >= 0 && rows[state.selected]) {
            rows[state.selected].classList.add('selected');
            if (instant) {
                rows[state.selected].scrollIntoView({ block: 'nearest' });
            } else {
                rows[state.selected].scrollIntoView({ block: 'nearest', behavior: 'smooth' });
            }
        }
        menuDesc.textContent  = item && item.desc ? item.desc : '';
        menuCount.textContent = state.items.length
            ? ((state.selected + 1) + ' / ' + state.items.length) : '';
        // Keyboard and hover both tick; the first highlight when a menu opens does not.
        if (!instant) sound('move');
    }

    function move(state, dir) {
        var next = nextEnabled(state.items, state.selected, dir);
        if (next < 0 || next === state.selected) return;
        select(state, next, false);
    }

    function choose(state) {
        if (state.selected < 0) return;
        var item = state.items[state.selected];
        if (!item || item.disabled) return;
        sound('select');
        hideAll();
        post('ui:select', { id: state.id, index: state.selected + 1 });
    }

    function openMenu(msg) {
        var items = Array.isArray(msg.items) ? msg.items : [];
        current = {
            kind: 'menu',
            id: msg.id,
            title: msg.title || '',
            subtitle: msg.subtitle || '',
            closeText: msg.closeText || 'Close',
            items: items.map(function (it) {
                it = it || {};
                return {
                    label: it.label == null ? '' : String(it.label),
                    right: it.right == null ? '' : String(it.right),
                    desc:  it.desc  == null ? '' : String(it.desc),
                    disabled: !!it.disabled,
                };
            }),
            selected: -1,
            placement: readPlacement(msg),
            steadyHeight: 0,
        };
        // The description area is reserved (3–5 lines, see style.css) only
        // when some row has one; a menu with none stays compact.
        menuEl.classList.toggle('has-desc', current.items.some(function (it) { return it.desc !== ''; }));
        inputEl.classList.add('hidden');
        menuEl.classList.remove('hidden');
        renderMenu(current);
        // Pin the panel once per open, from its steady height. Moving the
        // highlight afterwards changes only the description, which grows away
        // from the rows.
        placeCurrent();
    }

    // --------------------------------------------------------------- input --

    function isNumber(text) {
        return /^-?(\d+\.?\d*|\.\d+)$/.test(text.trim());
    }

    function updateNote(state) {
        var value = inputField.value;
        var bad = false;
        var note = '';
        if (state.numeric && value.trim() !== '' && !isNumber(value)) {
            bad = true;
            note = 'Numbers only';
        } else if (state.maxLength > 0) {
            note = value.length + ' / ' + state.maxLength;
        }
        inputField.classList.toggle('invalid', bad);
        inputNote.classList.toggle('error', bad);
        inputNote.textContent = note;
        return !bad;
    }

    function submitInput(state) {
        var value = inputField.value;
        if (state.maxLength > 0 && value.length > state.maxLength) value = value.slice(0, state.maxLength);
        if (state.numeric) {
            if (!isNumber(value)) {
                updateNote(state);
                inputField.focus();
                return;
            }
            value = Number(value);
        }
        sound('select');
        hideAll();
        post('ui:submit', { id: state.id, value: value });
    }

    function openInput(msg) {
        current = {
            kind: 'input',
            id: msg.id,
            numeric: !!msg.numeric,
            maxLength: Number(msg.maxLength) > 0 ? Math.floor(Number(msg.maxLength)) : 0,
            placement: readPlacement(msg),
        };
        inputTitle.textContent  = msg.title || '';
        inputSubmit.textContent = msg.submitText || 'Confirm';
        inputCancel.textContent = msg.closeText || 'Cancel';
        inputField.placeholder  = msg.placeholder || '';
        inputField.value        = msg.default == null ? '' : String(msg.default);
        if (current.maxLength > 0) inputField.maxLength = current.maxLength;
        else inputField.removeAttribute('maxlength');
        inputField.setAttribute('inputmode', current.numeric ? 'decimal' : 'text');
        updateNote(current);

        menuEl.classList.add('hidden');
        inputEl.classList.remove('hidden');
        // The note line under the box has a fixed height (style.css), so the
        // dialog's height is steady and the placement holds while typing.
        placeCurrent();
        setTimeout(function () { inputField.focus(); inputField.select(); }, 30);
    }

    // -------------------------------------------------------------- common --

    function hideAll() {
        menuEl.classList.add('hidden');
        inputEl.classList.add('hidden');
        menuItems.innerHTML = '';
        current = null;
    }

    function closeCurrent() {
        if (!current) return;
        var id = current.id;
        sound('close');
        hideAll();
        post('ui:close', { id: id });
    }

    menuClose.addEventListener('click', function () { closeCurrent(); });
    inputCancel.addEventListener('click', function () { closeCurrent(); });
    inputSubmit.addEventListener('click', function () { if (current && current.kind === 'input') submitInput(current); });
    inputField.addEventListener('input', function () { if (current && current.kind === 'input') updateNote(current); });

    document.addEventListener('keydown', function (e) {
        if (!current) return;
        var key = e.key;

        if (current.kind === 'menu') {
            if (key === 'ArrowDown' || key === 's' || key === 'S') { e.preventDefault(); move(current, +1); }
            else if (key === 'ArrowUp' || key === 'w' || key === 'W') { e.preventDefault(); move(current, -1); }
            else if (key === 'Home') { e.preventDefault(); var f = nextEnabled(current.items, -1, +1); if (f >= 0) select(current, f, false); }
            else if (key === 'End')  { e.preventDefault(); var l = nextEnabled(current.items, 0, -1); if (l >= 0) select(current, l, false); }
            else if (key === 'Enter' || key === ' ') { e.preventDefault(); choose(current); }
            else if (key === 'Escape' || key === 'Backspace') { e.preventDefault(); closeCurrent(); }
            return;
        }

        if (current.kind === 'input') {
            if (key === 'Enter')  { e.preventDefault(); submitInput(current); }
            else if (key === 'Escape') { e.preventDefault(); closeCurrent(); }
        }
    });

    // Right-click anywhere outside a row also closes, the way the game's menus do.
    document.addEventListener('contextmenu', function (e) {
        e.preventDefault();
        if (current && current.kind === 'menu') closeCurrent();
    });

    // The game window was resized: re-pin whatever is open.
    window.addEventListener('resize', function () { placeCurrent(); });

    window.addEventListener('message', function (event) {
        var msg = event.data || {};
        switch (msg.action) {
            case 'menu.open':  openMenu(msg);  break;
            case 'input.open': openInput(msg); break;
            case 'close':      hideAll();      break;
            default: break;
        }
    });
})();
