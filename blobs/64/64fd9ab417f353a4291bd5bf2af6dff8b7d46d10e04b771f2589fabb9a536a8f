/* ============================================================================
   poggy_markets — text prompt
   ----------------------------------------------------------------------------
   A single-line input modal, used when the game needs a word from the player
   outside the store panels: naming a new store, mostly.

   This replaces the external input resource the old build depended on.  It
   builds its own DOM and styles so it can be dropped in without touching the
   rest of the interface, and it borrows the existing CSS variables so it looks
   like it belongs.
   ============================================================================ */

(function () {
    'use strict';

    var RESOURCE = GetParentResourceName();

    var overlay, titleEl, inputEl, form;

    function build() {
        if (overlay) return;

        var style = document.createElement('style');
        style.textContent = [
            '.pm-prompt-overlay{position:fixed;inset:0;display:none;align-items:center;',
            'justify-content:center;background:rgba(0,0,0,.55);backdrop-filter:blur(3px);z-index:9999}',
            '.pm-prompt-overlay.show{display:flex}',
            '.pm-prompt-box{background:var(--bg-elevated,#1c1c28);border:1px solid var(--border-secondary,rgba(255,255,255,.1));',
            'border-radius:var(--radius-lg,12px);padding:24px;width:min(420px,90vw);',
            'box-shadow:0 24px 60px rgba(0,0,0,.55)}',
            '.pm-prompt-title{font-family:var(--font-heading,sans-serif);font-size:16px;font-weight:600;',
            'color:var(--text-primary,#f0f0f5);margin:0 0 14px}',
            '.pm-prompt-input{width:100%;box-sizing:border-box;background:var(--bg-tertiary,#16161f);',
            'border:1px solid var(--border-secondary,rgba(255,255,255,.1));border-radius:var(--radius-md,8px);',
            'padding:10px 12px;color:var(--text-primary,#f0f0f5);font-family:var(--font-body,sans-serif);font-size:14px}',
            '.pm-prompt-input:focus{outline:none;border-color:var(--accent,#638cff)}',
            '.pm-prompt-actions{display:flex;gap:10px;justify-content:flex-end;margin-top:16px}',
            '.pm-prompt-btn{padding:8px 18px;border-radius:var(--radius-md,8px);font-size:13px;',
            'font-weight:500;cursor:pointer;border:1px solid transparent;font-family:var(--font-body,sans-serif)}',
            '.pm-prompt-btn.confirm{background:var(--accent,#638cff);color:#fff}',
            '.pm-prompt-btn.confirm:hover{background:var(--accent-hover,#7da0ff)}',
            '.pm-prompt-btn.cancel{background:transparent;color:var(--text-secondary,#9898b0);',
            'border-color:var(--border-secondary,rgba(255,255,255,.1))}',
            '.pm-prompt-btn.cancel:hover{color:var(--text-primary,#f0f0f5)}'
        ].join('');
        document.head.appendChild(style);

        overlay = document.createElement('div');
        overlay.className = 'pm-prompt-overlay';
        overlay.innerHTML =
            '<div class="pm-prompt-box">' +
                '<form>' +
                    '<h3 class="pm-prompt-title"></h3>' +
                    '<input class="pm-prompt-input" type="text" maxlength="40" autocomplete="off">' +
                    '<div class="pm-prompt-actions">' +
                        '<button type="button" class="pm-prompt-btn cancel">Cancel</button>' +
                        '<button type="submit" class="pm-prompt-btn confirm">Confirm</button>' +
                    '</div>' +
                '</form>' +
            '</div>';
        document.body.appendChild(overlay);

        titleEl = overlay.querySelector('.pm-prompt-title');
        inputEl = overlay.querySelector('.pm-prompt-input');
        form    = overlay.querySelector('form');

        form.addEventListener('submit', function (e) {
            e.preventDefault();
            respond(inputEl.value.trim());
        });
        overlay.querySelector('.cancel').addEventListener('click', function () {
            respond(null);
        });
    }

    function respond(value) {
        if (overlay) overlay.classList.remove('show');
        fetch('https://' + RESOURCE + '/promptResult', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ value: value })
        }).catch(function () { /* the game is gone; nothing to do */ });
    }

    function show(title, placeholder) {
        build();
        titleEl.textContent = title || '';
        inputEl.value = '';
        inputEl.placeholder = placeholder || '';
        overlay.classList.add('show');
        // Focus has to wait for the element to actually be laid out.
        setTimeout(function () { inputEl.focus(); }, 30);
    }

    window.addEventListener('message', function (event) {
        if (event.data && event.data.type === 'prompt') {
            show(event.data.title, event.data.placeholder);
        }
    });

    // Escape cancels, matching how the rest of the interface behaves.
    document.addEventListener('keyup', function (e) {
        if (e.key === 'Escape' && overlay && overlay.classList.contains('show')) {
            respond(null);
        }
    });
}());
