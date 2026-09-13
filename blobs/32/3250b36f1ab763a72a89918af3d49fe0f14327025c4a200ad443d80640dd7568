/* ==========================================================================
   Poggy Storage — menu + prompt controller

   The Lua side keeps the vorp_menu call shape it always had, so none of the
   menu-building code in menu.lua had to change: it still passes
   { title, subtext, align, elements[] } and gets a value back.
   ========================================================================== */

const RESOURCE = typeof GetParentResourceName === 'function' ? GetParentResourceName() : '';

function post(name, data) {
    return fetch(`https://${RESOURCE}/${name}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json; charset=UTF-8' },
        body: JSON.stringify(data || {})
    }).catch(() => {});
}

/* ------------------------------------------------------------------ menu */

const menuEl     = document.getElementById('menu');
const titleEl    = document.getElementById('menu-title');
const subtextEl  = document.getElementById('menu-subtext');
const listEl     = document.getElementById('menu-list');

let elements = [];
let index    = 0;
let menuOpen = false;

function renderMenu() {
    listEl.textContent = '';

    elements.forEach((el, i) => {
        const li = document.createElement('li');
        li.className = 'item' + (i === index ? ' item--active' : '');
        li.setAttribute('role', 'option');
        li.setAttribute('aria-selected', String(i === index));

        const label = document.createElement('div');
        label.className = 'item__label';
        label.textContent = el.label ?? '';
        li.appendChild(label);

        if (el.desc) {
            const desc = document.createElement('div');
            desc.className = 'item__desc';
            desc.textContent = el.desc;
            li.appendChild(desc);
        }

        li.addEventListener('mouseenter', () => { index = i; paint(); });
        li.addEventListener('click', choose);

        listEl.appendChild(li);
    });
}

/* Repaint selection without rebuilding the list, so the scroll position and
   any in-flight hover state survive arrow-key navigation. */
function paint() {
    [...listEl.children].forEach((li, i) => {
        const active = i === index;
        li.classList.toggle('item--active', active);
        li.setAttribute('aria-selected', String(active));
        if (active) li.scrollIntoView({ block: 'nearest' });
    });
}

function move(step) {
    if (!elements.length) return;
    index = (index + step + elements.length) % elements.length;
    paint();
}

function choose() {
    const el = elements[index];
    if (!el) return;
    post('menuSelect', { value: el.value, index: index });
}

function openMenu(data) {
    elements = Array.isArray(data.elements) ? data.elements : [];
    index    = 0;

    titleEl.textContent   = data.title || '';
    subtextEl.textContent = data.subtext || '';

    menuEl.className = 'menu';
    const align = String(data.align || 'top-right');
    if (align.includes('left')) menuEl.classList.add('menu--left');
    if (align.includes('top'))  menuEl.classList.add('menu--top');

    renderMenu();
    menuEl.hidden = false;
    menuOpen = true;
}

function closeMenu() {
    menuEl.hidden = true;
    menuOpen = false;
    elements = [];
}

/* ---------------------------------------------------------------- prompt */

const promptEl      = document.getElementById('prompt');
const promptForm    = document.getElementById('prompt-form');
const promptHeader  = document.getElementById('prompt-header');
const promptHint    = document.getElementById('prompt-hint');
const promptInput   = document.getElementById('prompt-input');
const promptError   = document.getElementById('prompt-error');
const promptConfirm = document.getElementById('prompt-confirm');
const promptCancel  = document.getElementById('prompt-cancel');

let promptOpen    = false;
let promptPattern = null;
let promptMessage = '';

function openPrompt(data) {
    promptHeader.textContent = data.header || 'Enter a value';
    promptHint.textContent   = data.hint || '';
    promptInput.value        = data.value || '';
    promptInput.placeholder  = data.placeholder || '';
    promptInput.type         = data.type === 'number' ? 'number' : 'text';
    promptConfirm.textContent = data.button || 'Confirm';

    promptPattern = data.pattern ? new RegExp('^(?:' + data.pattern + ')$') : null;
    promptMessage = data.message || 'That value is not allowed.';

    promptError.hidden = true;
    promptEl.hidden    = false;
    promptOpen         = true;

    // Focus after paint or the input silently refuses the caret.
    requestAnimationFrame(() => promptInput.focus());
}

function closePrompt() {
    promptEl.hidden = true;
    promptOpen = false;
    promptError.hidden = true;
}

function submitPrompt(ev) {
    if (ev) ev.preventDefault();

    const value = promptInput.value.trim();

    if (promptPattern && !promptPattern.test(value)) {
        promptError.textContent = promptMessage;
        promptError.hidden = false;
        return;
    }

    post('promptSubmit', { value: value });
}

promptForm.addEventListener('submit', submitPrompt);
promptCancel.addEventListener('click', () => post('promptCancel', {}));

/* -------------------------------------------------------------- keyboard */

window.addEventListener('keydown', (e) => {
    if (promptOpen) {
        // Enter is handled by the form's submit; only Escape needs catching.
        if (e.key === 'Escape') {
            e.preventDefault();
            post('promptCancel', {});
        }
        return;
    }

    if (!menuOpen) return;

    switch (e.key) {
        case 'ArrowUp':    e.preventDefault(); move(-1); break;
        case 'ArrowDown':  e.preventDefault(); move(1);  break;
        case 'Enter':      e.preventDefault(); choose(); break;
        case 'Escape':
        case 'Backspace':  e.preventDefault(); post('menuBack', {}); break;
    }
});

/* ------------------------------------------------------------- from Lua */

window.addEventListener('message', (event) => {
    const data = event.data || {};

    switch (data.action) {
        case 'openMenu':    openMenu(data);   break;
        case 'closeMenu':   closeMenu();      break;
        case 'openPrompt':  openPrompt(data); break;
        case 'closePrompt': closePrompt();    break;
        case 'closeAll':    closeMenu(); closePrompt(); break;
    }
});
