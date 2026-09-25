/* ============================================================================
   poggy_markets — text prompt
   ----------------------------------------------------------------------------
   A single-line input modal, used when the game needs a word from the player
   outside the store panels: naming a new store, mostly.  The same box also
   asks yes/no questions (type 'confirm'): another player offering this one
   their shop.

   This replaces the external input resource the old build depended on.  It
   builds its own DOM and styles so it can be dropped in without touching the
   rest of the interface, and it borrows the existing CSS variables so it looks
   like it belongs.
   ============================================================================ */

(function () {
    'use strict';

    var RESOURCE = GetParentResourceName();

    var overlay, titleEl, textEl, inputEl, form, confirmBtn, cancelBtn;
    var mode = 'prompt';   // 'prompt' (text back) or 'confirm' (yes / no back)

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
            '.pm-prompt-text{color:var(--text-secondary,#9898b0);font-family:var(--font-body,sans-serif);',
            'font-size:13px;line-height:1.5;margin:0 0 6px}',
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
                    '<p class="pm-prompt-text"></p>' +
                    '<input class="pm-prompt-input" type="text" maxlength="40" autocomplete="off">' +
                    '<div class="pm-prompt-actions">' +
                        '<button type="button" class="pm-prompt-btn cancel">Cancel</button>' +
                        '<button type="submit" class="pm-prompt-btn confirm">Confirm</button>' +
                    '</div>' +
                '</form>' +
            '</div>';
        document.body.appendChild(overlay);

        titleEl    = overlay.querySelector('.pm-prompt-title');
        textEl     = overlay.querySelector('.pm-prompt-text');
        inputEl    = overlay.querySelector('.pm-prompt-input');
        form       = overlay.querySelector('form');
        confirmBtn = overlay.querySelector('.confirm');
        cancelBtn  = overlay.querySelector('.cancel');

        form.addEventListener('submit', function (e) {
            e.preventDefault();
            respond(mode === 'confirm' ? true : inputEl.value.trim());
        });
        cancelBtn.addEventListener('click', function () {
            respond(mode === 'confirm' ? false : null);
        });
    }

    function respond(value) {
        if (overlay) overlay.classList.remove('show');
        if (mode === 'confirm') {
            fetch('https://' + RESOURCE + '/confirmResult', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ accepted: value === true })
            }).catch(function () { /* the game is gone; nothing to do */ });
            return;
        }
        fetch('https://' + RESOURCE + '/promptResult', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ value: value })
        }).catch(function () { /* the game is gone; nothing to do */ });
    }

    function show(title, placeholder) {
        build();
        mode = 'prompt';
        textEl.style.display  = 'none';
        inputEl.style.display = '';
        confirmBtn.textContent = 'Confirm';
        cancelBtn.textContent  = 'Cancel';
        titleEl.textContent = title || '';
        inputEl.value = '';
        inputEl.placeholder = placeholder || '';
        overlay.classList.add('show');
        // Focus has to wait for the element to actually be laid out.
        setTimeout(function () { inputEl.focus(); }, 30);
    }

    function showConfirm(title, text, yes, no) {
        build();
        mode = 'confirm';
        titleEl.textContent = title || '';
        textEl.textContent  = text || '';
        textEl.style.display  = '';
        inputEl.style.display = 'none';
        confirmBtn.textContent = yes || 'Yes';
        cancelBtn.textContent  = no || 'No';
        overlay.classList.add('show');
        setTimeout(function () { confirmBtn.focus(); }, 30);
    }

    window.addEventListener('message', function (event) {
        if (event.data && event.data.type === 'prompt') {
            show(event.data.title, event.data.placeholder);
        } else if (event.data && event.data.type === 'confirm') {
            showConfirm(event.data.title, event.data.text, event.data.yes, event.data.no);
        }
    });

    // Escape cancels (or declines), matching how the rest of the interface behaves.
    document.addEventListener('keyup', function (e) {
        if (e.key === 'Escape' && overlay && overlay.classList.contains('show')) {
            respond(mode === 'confirm' ? false : null);
        }
    });
}());
