'use strict';
/* ================================================================
   poggy_scene — Scene Placer NUI Script 
   Two-phase UI:
     Phase 1 — text entry  (open_scene_text  message from Lua)
     Phase 2 — positioning (open_scene_position message from Lua)
   ================================================================ */

// ── UTILITIES ─────────────────────────────────────────────────────
const $ = id => document.getElementById(id);

/** Send a named callback to the Lua resource. */
function luaCall(name, data = {}) {
  return fetch(`https://${GetParentResourceName()}/${name}`, {
    method:  'POST',
    headers: { 'Content-Type': 'application/json' },
    body:    JSON.stringify(data),
  }).catch(() => {});
}

// ── STATE ─────────────────────────────────────────────────────────
let currentPhase = null; // null | 'text' | 'position'
let pos = { x: 0, y: 0, z: 0 };
let sceneText = '';

// Debounce for position update callbacks
let posDebounceTimer = null;
function schedulePositionUpdate() {
  clearTimeout(posDebounceTimer);
  posDebounceTimer = setTimeout(() => {
    luaCall('scene_position_update', { x: pos.x, y: pos.y, z: pos.z });
  }, 40);
}

// ── ROOT / PANELS ─────────────────────────────────────────────────
const root           = $('scene-root');
const textPanel      = $('scene-text-panel');
const posPanel       = $('scene-position-panel');
const textInput      = $('scene-text-input');
const previewEl      = $('scene-preview-text');
const statusRoot      = $('status-root');
const statusPanel     = $('status-panel');
const statusTextInput = $('status-text-input');
const statusNameInput = $('status-name-input');
const inputX         = $('input-x');
const inputY         = $('input-y');
const inputZ         = $('input-z');
// ── HELPERS ───────────────────────────────────────────────────────
function fmt(v) {
  return parseFloat(v).toFixed(1);
}

function poggycInputToPos(axis) {
  const el = axisInput[axis];
  const v  = parseFloat(el.value);
  if (!isNaN(v)) {
    pos[axis] = v;
    schedulePositionUpdate();
  }
}

// Write pos → inputs (without firing input events)
function flushPosToInputs() {
  Object.keys(axisInput).forEach(axis => { axisInput[axis].value = fmt(pos[axis]); });
}

// ── PHASE SWITCHING ───────────────────────────────────────────────
function showPhase(phase) {
  currentPhase = phase;

  textPanel.classList.add('hidden');
  posPanel.classList.add('hidden');
  statusRoot.classList.add('hidden');

  if (!phase) {
    root.classList.add('hidden');
    return;
  }

  if (phase === 'status') {
    statusRoot.classList.remove('hidden');
    setTimeout(() => statusTextInput.focus(), 60);
    return;
  }

  root.classList.remove('hidden');

  if (phase === 'text') {
    textPanel.classList.remove('hidden');
    // Reset to defaults and focus the editor
    editorSetDefaults();
    setTimeout(() => textInput.focus(), 60);
  } else if (phase === 'position') {
    posPanel.classList.remove('hidden');
    flushPosToInputs();
    // Show a short preview (strip markup for display)
    previewEl.textContent = sceneText.replace(/<[^>]+>/g, '').replace(/~[^~]+~/g, '');
    // Immediately send starting position so the Lua preview is correct
    luaCall('scene_position_update', { x: pos.x, y: pos.y, z: pos.z });
  }
}

// ── NUI MESSAGE LISTENER (Lua → JS) ──────────────────────────────
window.addEventListener('message', function(event) {
  const msg = event.data;
  if (!msg || !msg.type) return;

  switch (msg.type) {

    case 'open_scene_text':
      textInput.innerHTML = '';
      showPhase('text');
      signReset(msg.signs);
      sceneEditing = !!msg.edit;                  // changing a scene that is already placed
      if (msg.edit && msg.edit.sign && signData) signLoad(msg.edit);
      else if (msg.edit && msg.edit.text) textInput.innerHTML = sceneCodeToHtml(msg.edit.text);
      setSceneMode(sceneMode);                    // the button says Save when editing
      removeBtnReset();
      break;

    case 'open_scene_position':
      pos.x     = parseFloat(msg.x) || 0;
      pos.y     = parseFloat(msg.y) || 0;
      pos.z     = parseFloat(msg.z) || 0;
      sceneText = msg.text || '';
      showPhase('position');
      break;

    case 'close_scene_ui':
      showPhase(null);
      break;

    case 'open_status_ui':
      statusTextInput.innerHTML = '';
      statusNameInput.value = '';
      statusEditorSetDefaults();
      showPhase('status');
      break;

    case 'close_status_ui':
      showPhase(null);
      break;

    case 'status_presets':
      buildPresetList(msg.presets || []);
      break;

    case 'load_colors':
      // Populate COLOR_MAP, CODE_TO_HEX, and COLOR_ORDER from Lua config
      if (msg.historyMax)  meboxHistoryMax  = msg.historyMax;
      if (msg.dissipateMs) meboxDissipateMs = msg.dissipateMs;
      // Apply UI theme
      if (msg.theme) {
        document.documentElement.className =
          document.documentElement.className.replace(/\btheme-\S+/g, '').trim();
        if (msg.theme !== 'amber-frontier') {
          document.documentElement.classList.add('theme-' + msg.theme);
        }
      }
      Object.keys(COLOR_MAP).forEach(k => delete COLOR_MAP[k]);
      Object.keys(CODE_TO_HEX).forEach(k => delete CODE_TO_HEX[k]);
      COLOR_ORDER = [];
      (msg.colors || []).forEach(c => {
        const hex = c.hex.toLowerCase();
        COLOR_MAP[hex] = c.code;
        CODE_TO_HEX[c.code] = hex;
        COLOR_ORDER.push({ hex, code: c.code, title: c.title || c.code });
      });
      // Build swatch elements in both editors
      buildSwatches(editorColorsContainer, (hex) => { editorColor = hex; });
      buildSwatches(statusEditorColorsContainer, (hex) => { statusEditorColor = hex; });
      break;
  }
});

// ── PHASE 1 CONTROLS — Rich-text scene editor ────────────────────

// RDR2 color code mapping (hex → tilde code) — populated from config
const COLOR_MAP = {};
const CODE_TO_HEX = {};
let COLOR_ORDER = []; // [{hex, code, title}, ...] preserves config order

const DEFAULT_COLOR = '#ffffff';
const DEFAULT_SIZE  = 20;

let editorColor = DEFAULT_COLOR;
let editorSize  = DEFAULT_SIZE;

/**
 * A left / right switcher: ◄ value ►. Used instead of dropdowns, which leave marks on the
 * screen in the game's browser. Pressing a button never takes the focus, so a selection in
 * the text box stays selected.
 *   options: [{ value, label }]   onPick(value) is called when the player changes it
 */
function makeSwitcher(el, options, onPick) {
  const label = el.querySelector('.switcher-value');
  let list = options || [], at = 0;
  const show = () => { label.textContent = list.length ? list[at].label : ''; };
  const step = d => {
    if (!list.length) return;
    at = (at + d + list.length) % list.length;
    show();
    if (onPick) onPick(list[at].value);
  };
  [['.step-left', -1], ['.step-right', 1]].forEach(([sel, d]) => {
    const btn = el.querySelector(sel);
    btn.addEventListener('mousedown', e => e.preventDefault());
    btn.addEventListener('click', () => step(d));
  });
  el.addEventListener('wheel', e => {
    e.preventDefault();
    e.stopPropagation();
    step(e.deltaY < 0 ? -1 : 1);
  }, { passive: false });
  show();
  return {
    /** Show this value without calling onPick. */
    set(value) {
      const i = list.findIndex(o => String(o.value) === String(value));
      if (i >= 0) { at = i; show(); }
    },
    get value() { return list.length ? list[at].value : null; },
    setOptions(next) { list = next || []; at = 0; show(); },
  };
}

const FONT_SIZES = [10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 36, 40, 48, 60, 80, 100];
const fontSizeSwitch = makeSwitcher($('editor-font-size'), FONT_SIZES.map(v => ({ value: v, label: String(v) })), v => {
  editorSize = v;
  textInput.focus();
  applyFontSizeToSelection(editorSize);
});
const editorColorsContainer = $('editor-colors');
const statusEditorColorsContainer = $('status-editor-colors');

// Build swatch elements inside a container and bind mousedown handlers
function buildSwatches(container, onSelect) {
  container.innerHTML = '';
  COLOR_ORDER.forEach((entry, i) => {
    const span = document.createElement('span');
    span.className = 'color-swatch' + (i === 0 ? ' active' : '');
    span.dataset.color = entry.hex;
    span.dataset.code = entry.code;
    span.style.background = entry.hex;
    span.title = entry.title;
    span.addEventListener('mousedown', (e) => {
      e.preventDefault();
      container.querySelectorAll('.color-swatch').forEach(s => s.classList.remove('active'));
      span.classList.add('active');
      onSelect(entry.hex);
      document.execCommand('foreColor', false, entry.hex);
    });
    container.appendChild(span);
  });
}

function editorSetDefaults() {
  editorColor = DEFAULT_COLOR;
  editorSize  = DEFAULT_SIZE;
  fontSizeSwitch.set(DEFAULT_SIZE);
  editorColorsContainer.querySelectorAll('.color-swatch').forEach(s =>
    s.classList.toggle('active', s.dataset.color === DEFAULT_COLOR)
  );
}

// Selection save/restore for toolbar controls that steal focus
let savedRange = null;

function saveEditorSelection() {
  const sel = window.getSelection();
  if (sel.rangeCount > 0 && textInput.contains(sel.anchorNode)) {
    savedRange = sel.getRangeAt(0).cloneRange();
  }
}

function restoreEditorSelection() {
  if (savedRange) {
    const sel = window.getSelection();
    sel.removeAllRanges();
    sel.addRange(savedRange);
    savedRange = null;
  }
}

// (Scene color swatch handlers are bound dynamically via buildSwatches)

function applyFontSizeToSelection(sizePx) {
  const sel = window.getSelection();
  if (!sel || sel.rangeCount === 0) return;

  if (sel.isCollapsed) {
    // No text selected — insert a wrapper span so future typing inherits the size
    const span = document.createElement('span');
    span.style.fontSize = sizePx + 'px';
    span.style.color = editorColor; // preserve active color
    span.textContent = '\u200B'; // zero-width space keeps the span alive
    const range = sel.getRangeAt(0);
    range.insertNode(span);
    // Place cursor after the zero-width space, inside the span
    range.setStart(span.firstChild, 1);
    range.setEnd(span.firstChild, 1);
    sel.removeAllRanges();
    sel.addRange(range);
    return;
  }

  // Text is selected — use fontSize 7 as a marker, then replace
  document.execCommand('fontSize', false, '7');
  const bigFonts = textInput.querySelectorAll('font[size="7"]');
  bigFonts.forEach(el => {
    const span = document.createElement('span');
    span.style.fontSize = sizePx + 'px';
    // Preserve color if the browser merged it into the <font> tag (avoids white-on-size-change bug)
    const colorAttr = el.getAttribute('color');
    if (colorAttr) span.style.color = colorAttr;
    span.innerHTML = el.innerHTML;
    el.parentNode.replaceChild(span, el);
  });
}

// Block key events from leaking to the game during text entry
textInput.addEventListener('keydown', e => {
  e.stopPropagation();
  if (e.key === 'Escape') {
    e.preventDefault();
    cancelText();
    return;
  }
  // Enter = newline (default contenteditable behavior)
  // Confirm only via the button
});
textInput.addEventListener('keyup',    e => e.stopPropagation());
textInput.addEventListener('keypress', e => e.stopPropagation());

// On each input, ensure new unformatted text inherits the active color/size.
// Also apply uppercase via CSS (text-transform handles display).
textInput.addEventListener('input', () => {
  // If the user started typing with no formatting, the browser may
  // insert plain text nodes. We leave them — generateSceneCode()
  // handles them using the default color/size.
});

function cancelText() {
  showPhase(null);
  luaCall('scene_text_cancel');
}

function confirmText() {
  if (sceneMode === 'sign') {
    // a sign is written and placed in one step: it is already standing in the world
    const res = signRefresh();
    if (!res || !res.ok) return;
    showPhase(null);
    luaCall('sign_place', { sign: signOf(res), slide });
    return;
  }
  const html = textInput.innerHTML.trim();
  if (!html || textInput.textContent.trim() === '') return;
  savedEditorHtml = textInput.innerHTML; // preserve for cancel-revert
  sceneText = generateSceneCode(textInput);
  luaCall('scene_text_confirm', { text: sceneText });
  // Phase switch happens when Lua sends back open_scene_position
}

// ── CODE GENERATION ─────────────────────────────────────────────
// Walks the contenteditable DOM and produces RDR2 markup like:
//   <font size="20">~q~HELLO WORLD</font>~n~<font size="14">~t6~LINE 2</font>

function rgbToHex(rgb) {
  if (!rgb) return DEFAULT_COLOR;
  if (rgb.startsWith('#')) return rgb.toLowerCase();
  const m = rgb.match(/rgba?\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)/);
  if (!m) return DEFAULT_COLOR;
  const hex = '#' + [m[1], m[2], m[3]].map(x => parseInt(x).toString(16).padStart(2, '0')).join('');
  return hex.toLowerCase();
}

function closestColorCode(hex) {
  if (COLOR_MAP[hex]) return COLOR_MAP[hex];
  // Euclidean distance match
  const r = parseInt(hex.slice(1,3), 16);
  const g = parseInt(hex.slice(3,5), 16);
  const b = parseInt(hex.slice(5,7), 16);
  let best = '~q~', bestDist = Infinity;
  for (const [h, code] of Object.entries(COLOR_MAP)) {
    const cr = parseInt(h.slice(1,3), 16);
    const cg = parseInt(h.slice(3,5), 16);
    const cb = parseInt(h.slice(5,7), 16);
    const d = (cr-r)**2 + (cg-g)**2 + (cb-b)**2;
    if (d < bestDist) { bestDist = d; best = code; }
  }
  return best;
}

function generateSceneCode(rootEl) {
  const segments = [];

  function walk(node, parentSize, parentColor) {
    if (node.nodeType === Node.TEXT_NODE) {
      const t = node.textContent.replace(/\u200B/g, ''); // strip zero-width spaces
      if (t) segments.push({ text: t, size: parentSize, color: parentColor });
      return;
    }
    if (node.nodeType !== Node.ELEMENT_NODE) return;

    let size  = parentSize;
    let color = parentColor;

    // Check for font-size in style
    if (node.style && node.style.fontSize) {
      const m = node.style.fontSize.match(/(\d+)/);
      if (m) size = parseInt(m[1]);
    }
    // Check for color in style
    if (node.style && node.style.color) {
      color = rgbToHex(node.style.color);
    }
    // Legacy <font color="...">
    if (node.tagName === 'FONT' && node.getAttribute('color')) {
      color = rgbToHex(node.getAttribute('color'));
    }

    // Handle <br> and <div>/<p> as line breaks
    if (node.tagName === 'BR') {
      segments.push({ text: '\n', size, color });
      return;
    }

    if (node.childNodes.length === 0) {
      const t = node.textContent;
      if (t) segments.push({ text: t, size, color });
      return;
    }

    // DIV/P children act as new lines (contenteditable wraps lines in divs)
    if ((node.tagName === 'DIV' || node.tagName === 'P') && node !== rootEl) {
      // Insert newline before this block (unless it's the first child)
      if (node.previousSibling) {
        segments.push({ text: '\n', size, color });
      }
    }

    node.childNodes.forEach(child => walk(child, size, color));
  }

  walk(rootEl, DEFAULT_SIZE, DEFAULT_COLOR);

  // Merge adjacent segments with same formatting, split on newlines
  let result = '';
  let curSize = null, curColor = null, curText = '';

  function flush() {
    if (!curText) return;
    const code = closestColorCode(curColor || DEFAULT_COLOR);
    result += '<font size="' + (curSize || DEFAULT_SIZE) + '">' + code + curText.toUpperCase() + '</font>';
    curText = '';
  }

  for (const seg of segments) {
    if (seg.text === '\n') {
      flush();
      result += '~n~';
      curSize = null;
      curColor = null;
      continue;
    }
    if (curSize !== null && (curSize !== seg.size || curColor !== seg.color)) {
      flush();
    }
    curSize  = seg.size;
    curColor = seg.color;
    curText += seg.text;
  }
  flush();

  return result;
}

// Reverse lookup CODE_TO_HEX is populated alongside COLOR_MAP from config

/** Scene text as saved (<font size="20">~q~HELLO</font>~n~...) back into the editor, sizes and colours kept. */
function sceneCodeToHtml(code) {
  let html = '', size = DEFAULT_SIZE, color = DEFAULT_COLOR;
  const esc = t => t.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
  String(code || '').split(/(<font size="\d+">|<\/font>|~n~|~[^~]+~)/gi).forEach(part => {
    if (!part) return;
    const m = part.match(/^<font size="(\d+)">$/i);
    if (m) { size = parseInt(m[1]) || DEFAULT_SIZE; return; }
    if (/^<\/font>$/i.test(part)) { size = DEFAULT_SIZE; color = DEFAULT_COLOR; return; }
    if (/^~n~$/i.test(part)) { html += '<br>'; return; }
    if (/^~[^~]+~$/.test(part)) { if (CODE_TO_HEX[part]) color = CODE_TO_HEX[part]; return; }
    html += '<span style="font-size:' + size + 'px;color:' + color + '">' + esc(part) + '</span>';
  });
  return html;
}

function presetCodeToHtml(code) {
  if (!code) return '';
  let html = '';
  // Strip outer <font size="N"> wrappers, keep color codes and text
  // Format: <font size="20">~q~HELLO</font>~n~<font size="14">~t6~LINE 2</font>
  const stripped = code.replace(/<\/?font[^>]*>/gi, '');
  // Split on ~n~ for line breaks
  const lines = stripped.split(/~n~/gi);
  lines.forEach((line, i) => {
    // Parse color-coded segments: ~code~text~code~text...
    const parts = line.split(/(~[^~]+~)/);
    let currentColor = '#ffffff';
    for (const part of parts) {
      if (!part) continue;
      const colorMatch = part.match(/^~[^~]+~$/);
      if (colorMatch && CODE_TO_HEX[part]) {
        currentColor = CODE_TO_HEX[part];
      } else if (!colorMatch) {
        html += '<span style="color:' + currentColor + '">' + part + '</span>';
      }
    }
    if (i < lines.length - 1) html += '<br>';
  });
  return html;
}


// ── SIGN BOARD EDITOR ─────────────────────────────────────────────
// A scene can be a wooden sign with painted letters instead of floating text. The words are
// written in one box, like scene text: select part of them and pick a size or a paint to change
// just that part. Each letter is a prop in the game, so the words are capitals from a fixed set
// of characters and must fit the board. What there is (boards, letterings, paints, widths) comes
// from Lua (client/sign_metrics.lua); the sums below are the same ones it does, and the server
// does them again.
const SIGN_PX = { 1: 30, 2: 20, 3: 13 };     // how big each letter size is drawn in the editor

let signData  = null;          // { available, fonts, boards, paints, tones, layout, maxLetters } from Lua
let sceneMode = 'text';        // 'text' | 'sign'
let sceneEditing = false;      // the editor was opened on a scene that is already placed
let sign      = { board: 1, tone: 'l', font: 'r', vertical: false, border: null, mount: null, light: null, gap: 3.5, dz: 0,
                  double: false };
                                     // gap and dz are in centimetres here, as the sliders show them
let signSize  = 1;             // the size new typing takes
let signPaint = null;          // the paint new typing takes (null = the board's default)

const sceneModeEl    = $('scene-mode');
const sceneTextMode  = $('scene-text-mode');
const sceneSignMode  = $('scene-sign-mode');
const signInput      = $('sign-text-input');
const signStatusEl   = $('sign-status');
const signPaintsEl   = $('sign-paints');
const textConfirmBtn = $('btn-text-confirm');

const signFontOf   = key => signData.fonts.find(f => f.key === key);
const signPaintHex = key => (signData.paints.find(p => p.key === key) || signData.paints[0]).hex;
/** The paint plain typing takes: dark on a light board, light on a dark one. */
const signDefaultPaint = () => (sign.tone === 'l' ? 'd' : 'l');

// ── the words: a small editor of its own ─────────────────────────
// The browser's own rich-text commands are not used: in the game's browser they leave stray
// styles behind and cannot select a symbol. Instead the words are kept here as plain data, the
// box is redrawn from that data after every change, and every keystroke is applied to the data,
// never to the page. The caret and the selection are still the browser's.
//   signDoc: lines of items. An item is a letter { c: 'A' } or a symbol { y: 'star' }, with its
//   size s (1..3) and paint p (a paint key, or null for the board's default).
let signDoc  = [[]];
let signUndo = [];
let signLast = { from: { l: 0, i: 0 }, to: { l: 0, i: 0 } };     // the last caret or selection in the box

const signItemPaint = it => it.p || signDefaultPaint();
const signSame = (a, b) => a.l === b.l && a.i === b.i;

function signRender() {
  signInput.innerHTML = '';
  signDoc.forEach(line => {
    const row = document.createElement('div');
    row.className = 'sl';
    line.forEach(it => {
      const el = document.createElement('span');
      el.className = 'si';
      el.style.fontSize = SIGN_PX[it.s] + 'px';
      el.style.color = signPaintHex(signItemPaint(it));
      if (it.y) {
        el.classList.add('si-sym');
        el.contentEditable = 'false';
        el.appendChild(signSymbolEl(signData.symbols.find(y => y.key === it.y)));
      } else {
        el.textContent = it.c === ' ' ? '\u00A0' : it.c;
      }
      row.appendChild(el);
    });
    if (!line.length) row.appendChild(document.createElement('br'));
    signInput.appendChild(row);
  });
  signInput.classList.toggle('empty', signDoc.length === 1 && !signDoc[0].length);
}

/** A place in the page (node, offset) as a place in the words (line, item), or null when outside the box. */
function signPoint(node, offset) {
  if (!node || !signInput.contains(node)) return null;
  const last = signDoc.length - 1;
  if (node === signInput) {
    return offset > last ? { l: last, i: signDoc[last].length } : { l: offset, i: 0 };
  }
  const el = node.nodeType === Node.TEXT_NODE ? node.parentNode : node;
  const row = el.closest('.sl');
  if (!row) return null;
  const l = Array.prototype.indexOf.call(signInput.children, row);
  if (l < 0 || l > last) return null;
  const item = el.closest('.si');
  if (item) {
    const i = Array.prototype.indexOf.call(row.children, item);
    return { l, i: Math.min(i + (offset > 0 ? 1 : 0), signDoc[l].length) };
  }
  return { l, i: Math.min(offset, signDoc[l].length) };
}

/** The caret or selection in the box as { from, to } in the words, or null when it is elsewhere. */
function signSel() {
  const sel = window.getSelection();
  if (!sel || !sel.rangeCount) return null;
  const a = signPoint(sel.anchorNode, sel.anchorOffset), b = signPoint(sel.focusNode, sel.focusOffset);
  if (!a || !b) return null;
  return (a.l < b.l || (a.l === b.l && a.i <= b.i)) ? { from: a, to: b } : { from: b, to: a };
}

function signSetSel(from, to) {
  const place = p => {
    const l = Math.min(p.l, signDoc.length - 1);
    return [signInput.children[l], Math.min(p.i, signDoc[l].length)];
  };
  const range = document.createRange();
  range.setStart(...place(from));
  range.setEnd(...place(to || from));
  const sel = window.getSelection();
  sel.removeAllRanges();
  sel.addRange(range);
  signLast = { from, to: to || from };
  signShowSelection(signLast);
}

/** Show what is selected: our own tint, so a selected symbol shows as plainly as a selected letter. */
function signShowSelection(s) {
  Array.prototype.forEach.call(signInput.children, (row, l) => {
    Array.prototype.forEach.call(row.children, (el, i) => {
      const inside = (l > s.from.l || (l === s.from.l && i >= s.from.i)) && (l < s.to.l || (l === s.to.l && i < s.to.i));
      if (el.classList.contains('si')) el.classList.toggle('sel', inside);
    });
  });
}

/** The size switcher and the paint swatches show what the caret is in, so typing carries on in it. */
function signToolbarFollow(s) {
  const line = signDoc[s.from.l];
  const it = signSame(s.from, s.to) ? (line[s.from.i - 1] || line[s.from.i]) : line[s.from.i] || signDoc[s.to.l][s.to.i - 1];
  if (!it) return;
  signSize = it.s;
  signPaint = it.p;
  signSizeSwitch.set(signSize);
  signShowPaint();
}

function signShowPaint() {
  signPaintsEl.querySelectorAll('.color-swatch').forEach(el =>
    el.classList.toggle('active', el.dataset.paint === (signPaint || signDefaultPaint())));
}

document.addEventListener('selectionchange', () => {
  if (sceneMode !== 'sign' || !signData) return;
  const s = signSel();
  if (!s) return;
  signLast = s;
  signShowSelection(s);
  signToolbarFollow(s);
});

// ── changing the words ──
function signSnapshot() {
  signUndo.push(JSON.stringify(signDoc));
  if (signUndo.length > 80) signUndo.shift();
}

/** Take out a selection; returns where the caret is left. */
function signDelete(s) {
  if (s.from.l === s.to.l) {
    signDoc[s.from.l].splice(s.from.i, s.to.i - s.from.i);
  } else {
    const head = signDoc[s.from.l].slice(0, s.from.i), tail = signDoc[s.to.l].slice(s.to.i);
    signDoc.splice(s.from.l, s.to.l - s.from.l + 1, head.concat(tail));
  }
  return { l: s.from.l, i: s.from.i };
}

function signBreak(at) {
  const line = signDoc[at.l];
  signDoc.splice(at.l, 1, line.slice(0, at.i), line.slice(at.i));
  return { l: at.l + 1, i: 0 };
}

/** Write text at the caret in the size and paint in use. Only the characters a sign can show are kept. */
function signType(at, text) {
  const widths = signData.fonts[0].width;
  for (const raw of String(text || '').replace(/\r/g, '')) {
    if (raw === '\n') { at = signBreak(at); continue; }
    const ch = raw === '\u00A0' ? ' ' : raw.toUpperCase();
    if (ch !== ' ' && widths[ch] === undefined) continue;
    signDoc[at.l].splice(at.i, 0, { c: ch, s: signSize, p: signPaint });
    at = { l: at.l, i: at.i + 1 };
  }
  return at;
}

/** Redraw the box from the words, put the caret back, and show the sign. */
function signCommit(from, to) {
  signRender();
  signInput.focus();
  signSetSel(from, to);
  signRefresh();
}

/** Change every selected letter and symbol. False when nothing is selected. */
function signApply(change) {
  const s = signSel() || signLast;
  if (signSame(s.from, s.to)) return false;
  signSnapshot();
  for (let l = s.from.l; l <= s.to.l; l++) {
    const a = l === s.from.l ? s.from.i : 0, b = l === s.to.l ? s.to.i : signDoc[l].length;
    for (let i = a; i < b; i++) change(signDoc[l][i]);
  }
  signCommit(s.from, s.to);
  return true;
}

function signUndoLast() {
  if (!signUndo.length) return;
  signDoc = JSON.parse(signUndo.pop());
  const l = signDoc.length - 1;
  signCommit({ l, i: signDoc[l].length });
}

signInput.addEventListener('beforeinput', e => {
  e.preventDefault();                                   // nothing is typed into the page itself
  const s = signSel() || signLast, t = e.inputType || '';
  if (t === 'historyUndo') { signUndoLast(); return; }
  let caret;
  if (t.startsWith('insert')) {
    signSnapshot();
    caret = signDelete(s);
    if (t === 'insertParagraph' || t === 'insertLineBreak') {
      caret = signBreak(caret);
    } else {
      const text = e.data != null ? e.data : (e.dataTransfer ? e.dataTransfer.getData('text/plain') : '');
      caret = signType(caret, text);
    }
  } else if (t.startsWith('delete')) {
    signSnapshot();
    if (!signSame(s.from, s.to)) {
      caret = signDelete(s);
    } else if (t.indexOf('Forward') >= 0) {
      const line = signDoc[s.from.l];
      if (s.from.i < line.length) caret = signDelete({ from: s.from, to: { l: s.from.l, i: s.from.i + 1 } });
      else if (s.from.l < signDoc.length - 1) caret = signDelete({ from: s.from, to: { l: s.from.l + 1, i: 0 } });
      else caret = s.from;
    } else {
      if (s.from.i > 0) caret = signDelete({ from: { l: s.from.l, i: s.from.i - 1 }, to: s.from });
      else if (s.from.l > 0) caret = signDelete({ from: { l: s.from.l - 1, i: signDoc[s.from.l - 1].length }, to: s.from });
      else caret = s.from;
    }
  } else {
    return;                                             // bold, italic and the like: a sign has none
  }
  signCommit(caret);
});
// A click on a symbol selects it (it cannot hold a caret itself), ready for a new size or paint.
signInput.addEventListener('mousedown', e => {
  const item = e.target.closest ? e.target.closest('.si-sym') : null;
  if (!item) return;
  e.preventDefault();
  const row = item.parentNode;
  const l = Array.prototype.indexOf.call(signInput.children, row), i = Array.prototype.indexOf.call(row.children, item);
  signInput.focus();
  signSetSel({ l, i }, { l, i: i + 1 });
  signToolbarFollow(signLast);
});
// Should the page ever be changed behind our back, draw it again from the words.
signInput.addEventListener('input', () => { signRender(); signSetSel(signLast.from, signLast.to); });

/** The words as lines of runs, as Lua takes them: [[{ s, p, t } or { s, p, y }, ...], ...]. */
function signCollect() {
  return signDoc.map(line => {
    const runs = [];
    line.forEach(it => {
      const p = signItemPaint(it);
      if (it.y) { runs.push({ s: it.s, p, y: it.y }); return; }
      const last = runs[runs.length - 1];
      if (last && last.t !== undefined && last.s === it.s && last.p === p) last.t += it.c;
      else runs.push({ s: it.s, p, t: it.c });
    });
    return runs;
  });
}

/** The same sums as SignMeasure in Lua. Returns { ok, why, lines, letters, note }. */
function signMeasure() {
  const f = signFontOf(sign.font), b = signData.boards[sign.board - 1], lay = signData.layout;
  const res = { ok: false, why: '', lines: [], letters: 0, note: '' };
  const allowed = ch => ch === ' ' || f.width[ch] !== undefined;

  signCollect().forEach(raw => {
    const runs = [];
    raw.forEach(r => {
      if (r.y !== undefined) { runs.push({ s: r.s, p: r.p, y: r.y }); return; }
      let text = '';
      for (const ch of r.t.toUpperCase()) if (allowed(ch)) text += ch;
      if (!runs.length) text = text.replace(/^ +/, '');
      if (!text) return;
      const last = runs[runs.length - 1];
      if (last && last.t !== undefined && last.s === r.s && last.p === r.p) last.t += text;
      else runs.push({ s: r.s, p: r.p, t: text });
    });
    while (runs.length && runs[runs.length - 1].t !== undefined) {
      const t = runs[runs.length - 1].t.replace(/ +$/, '');
      if (!t) runs.pop(); else { runs[runs.length - 1].t = t; break; }
    }
    if (runs.length) res.lines.push(runs);
  });
  if (!res.lines.length) { res.why = 'Type the words for the sign.'; return res; }
  if (res.lines.length > lay.maxLines) { res.why = 'Too many lines: the most is ' + lay.maxLines + '.'; return res; }

  // every line as a row of items: a letter, a space, or a symbol
  const rows = res.lines.map(runs => {
    const items = [];
    runs.forEach(run => {
      if (run.y !== undefined) {
        const sym = signData.symbols.find(y => y.key === run.y);
        const h = (sym.flourish ? signData.symbolSizes.flourish : signData.symbolSizes.emblem)[run.s - 1];
        items.push({ w: sym.w * h, h, pad: lay.symbolPad * h, symbol: true });
      } else {
        const rc = f.sizes[run.s - 1];
        for (const ch of run.t) {
          if (ch === ' ') items.push({ w: f.space * rc, h: rc, space: true });
          else items.push({ w: f.width[ch] * rc, h: rc });
        }
      }
    });
    return items;
  });

  // Nothing typed is refused for its size: words that are too long simply run past the frame.
  // The line under the box says how much room is used, and says so when they overhang.
  const m = v => v.toFixed(2) + ' m';
  const gap = sign.gap / 100;
  let wide = 0, tall = 0;
  if (!sign.vertical) {
    rows.forEach((items, i) => {
      let height = 0, width = 0;
      items.forEach((it, k) => {
        if (!it.space) { height = Math.max(height, it.h); res.letters++; }
        width += it.w;
        if (it.symbol) {
          if (k > 0) width += it.pad;
          if (k < items.length - 1) width += it.pad;
        }
      });
      wide = Math.max(wide, width);
      tall += height + (i > 0 ? gap : 0);
    });
  } else {
    rows.forEach((items, i) => {
      let width = 0, height = 0, first = true;
      items.forEach(it => {
        if (it.space) { height += lay.stackSpace * it.h; return; }
        if (!first) height += lay.stackGap * it.h;
        height += it.h;
        first = false;
        width = Math.max(width, it.w);
        res.letters++;
      });
      tall = Math.max(tall, height);
      wide += width + (i > 0 ? gap : 0);
    });
  }
  res.note = 'Width ' + m(wide) + ' of ' + m(b.w) + '  ·  height ' + m(tall) + ' of ' + m(b.h);
  res.over = wide > b.w + 0.0005 || tall > b.h + 0.0005;
  if (res.letters > signData.maxLetters) {
    res.why = 'Too many letters and symbols: ' + res.letters + ' of ' + signData.maxLetters + '.';
    return res;
  }
  res.ok = true;
  return res;
}

function signRefresh() {
  if (!signData) return null;
  const res = signMeasure();
  const empty = !res.lines.length;
  signStatusEl.classList.toggle('bad', !res.ok && !empty);
  signStatusEl.classList.toggle('over', res.ok && res.over);
  signStatusEl.textContent = res.ok
    ? res.note + '  ·  ' + res.letters + ' of ' + signData.maxLetters + ' letters'
      + (res.over ? '  ·  runs past the board' : '')
    : res.why;
  if (sceneMode === 'sign') {
    textConfirmBtn.disabled = !res.ok;
    signLive(res);
  }
  return res;
}

/** The sign as Lua takes it. */
function signOf(res) {
  const out = { board: sign.board, tone: sign.tone, font: sign.font, vertical: sign.vertical, lines: res.lines };
  if (sign.border) out.border = sign.border;      // all three optional: left out when not chosen
  if (sign.mount) out.mount = sign.mount;
  if (sign.light) out.light = sign.light;
  out.gap = sign.gap / 100;                       // metres
  out.dz = sign.dz / 100;
  if (sign.double) out.double = true;
  return out;
}

// The sign stands in the world while it is written. A change of words is sent a moment after the
// last keystroke (the game rebuilds it from props); while the words do not fit, the last sign
// that did stays up.
let signLiveTimer = null;
function signLive(res) {
  clearTimeout(signLiveTimer);
  if (!res.ok) return;
  signLiveTimer = setTimeout(() => luaCall('sign_live', { sign: signOf(res), slide }), 140);
}

function segSelect(container, attr, value) {
  container.querySelectorAll('.seg-btn').forEach(b => b.classList.toggle('active', b.dataset[attr] === String(value)));
}

/** The box wears the board's colour and the lettering, so it reads like the sign will. */
function signDressInput() {
  signInput.classList.remove('tone-l', 'tone-d', 'sign-font-r', 'sign-font-s', 'sign-down');
  signInput.classList.add('tone-' + sign.tone, 'sign-font-' + sign.font);
  if (sign.vertical) signInput.classList.add('sign-down');
  signInput.style.color = signPaintHex(signDefaultPaint());
  signRender();                       // the default paint follows the board's colour
  signShowPaint();
}

function setSceneMode(mode) {
  sceneMode = (mode === 'sign' && signData) ? 'sign' : 'text';
  segSelect(sceneModeEl, 'mode', sceneMode);
  const was = !sceneSignMode.classList.contains('hidden');
  sceneTextMode.classList.toggle('hidden', sceneMode !== 'text');
  sceneSignMode.classList.toggle('hidden', sceneMode !== 'sign');
  textConfirmBtn.disabled = false;
  textConfirmBtn.textContent = sceneMode === 'sign' ? (sceneEditing ? '✓ Save' : '✓ Place') : 'Confirm →';
  if (sceneMode === 'sign') {
    signRefresh();
    setTimeout(() => signInput.focus(), 60);
  } else {
    clearTimeout(signLiveTimer);
    if (was) luaCall('sign_live_stop');        // back to floating text: the sign comes down
    setTimeout(() => textInput.focus(), 60);
  }
}

/** A fresh editor: called each time /scene opens. */
function signReset(data) {
  signData = data && data.available ? data : null;
  sceneModeEl.classList.toggle('hidden', !signData);
  if (signData) {
    sign = { board: 1, tone: 'l', font: signData.fonts[0].key, vertical: false, border: null, mount: null, light: null,
             gap: Math.round(signData.layout.lineGap * 1000) / 10, dz: 0, double: false };
    signWordSliders();
    segSelect($('sign-sides'), 'sides', '1');
    signSize = 1;
    signPaint = null;
    slideReset();
    signDoc = [[]];
    signUndo = [];
    signLast = { from: { l: 0, i: 0 }, to: { l: 0, i: 0 } };
    signRender();
    signSizeSwitch.set(1);
    signBoardSwitch.setOptions(signData.boards.map(b => ({ value: b.id, label: b.label })));
    const none = [{ value: null, label: 'None' }];
    signMountSwitch.setOptions(none.concat(signData.mounts.map(o => ({ value: o.key, label: o.label }))));
    signLightSwitch.setOptions(none.concat(signData.lights.map(o => ({ value: o.key, label: o.label }))));

    // the painted border: none, or any paint
    const borderEl = $('sign-border');
    borderEl.innerHTML = '';
    [{ key: null, label: 'None' }].concat(signData.paints).forEach(p => {
      const span = document.createElement('span');
      span.className = 'color-swatch' + (p.key ? '' : ' swatch-none active');
      span.title = p.label;
      if (p.key) span.style.background = p.hex;
      span.addEventListener('mousedown', e => {
        e.preventDefault();
        sign.border = p.key;
        borderEl.querySelectorAll('.color-swatch').forEach(el => el.classList.toggle('active', el === span));
        signRefresh();
      });
      borderEl.appendChild(span);
    });

    // the symbols: a click writes one in where the caret is, in the size and paint in use
    const symbolsEl = $('sign-symbols');
    symbolsEl.innerHTML = '';
    signData.symbols.forEach(y => {
      const btn = document.createElement('button');
      btn.type = 'button';
      btn.className = 'sym-btn' + (y.flourish ? ' sym-wide' : '');
      btn.title = y.label;
      btn.appendChild(signSymbolEl(y));
      btn.addEventListener('mousedown', e => e.preventDefault());      // the words keep the focus and the caret
      btn.addEventListener('click', () => {
        signSnapshot();
        const at = signDelete(signSel() || signLast);
        signDoc[at.l].splice(at.i, 0, { y: y.key, s: signSize, p: signPaint });
        signCommit({ l: at.l, i: at.i + 1 });
      });
      symbolsEl.appendChild(btn);
    });

    const seg = (el, items, attr, pick) => {
      el.innerHTML = '';
      items.forEach((it, i) => {
        const btn = document.createElement('button');
        btn.type = 'button';
        btn.className = 'seg-btn' + (i === 0 ? ' active' : '');
        btn.dataset[attr] = it.key;
        btn.textContent = it.label;
        btn.addEventListener('click', () => { pick(it.key); segSelect(el, attr, it.key); signDressInput(); signRefresh(); });
        el.appendChild(btn);
      });
    };
    seg($('sign-tone'), signData.tones, 'tone', key => { sign.tone = key; });
    seg($('sign-font'), signData.fonts, 'font', key => { sign.font = key; });
    segSelect($('sign-dir'), 'dir', 'h');

    signPaintsEl.innerHTML = '';
    signData.paints.forEach(p => {
      const span = document.createElement('span');
      span.className = 'color-swatch';
      span.dataset.paint = p.key;
      span.style.background = p.hex;
      span.title = p.label;
      span.addEventListener('mousedown', e => {
        e.preventDefault();                       // the words keep the focus and their selection
        signPaint = p.key;                        // what is selected takes it; so does what is typed next
        signApply(it => { it.p = p.key; });
        signShowPaint();
      });
      signPaintsEl.appendChild(span);
    });
    signDressInput();
  }
  setSceneMode('text');
}

sceneModeEl.querySelectorAll('.seg-btn').forEach(b =>
  b.addEventListener('click', () => setSceneMode(b.dataset.mode)));

$('sign-dir').querySelectorAll('.seg-btn').forEach(b => b.addEventListener('click', () => {
  sign.vertical = b.dataset.dir === 'v';
  segSelect($('sign-dir'), 'dir', b.dataset.dir);
  signDressInput();
  signRefresh();
}));

$('sign-sides').querySelectorAll('.seg-btn').forEach(b => b.addEventListener('click', () => {
  sign.double = b.dataset.sides === '2';
  segSelect($('sign-sides'), 'sides', b.dataset.sides);
  signRefresh();
}));

/** Open the editor on a sign that is already placed: its board, its words, how it is turned. */
function signLoad(edit) {
  const s = edit.sign;
  sign = { board: s.board, tone: s.tone, font: s.font, vertical: s.vertical === true, border: s.border || null,
           mount: s.mount || null, light: s.light || null,
           gap: Math.round((s.gap !== undefined ? s.gap : signData.layout.lineGap) * 1000) / 10,
           dz: Math.round((s.dz || 0) * 1000) / 10, double: s.double === true };
  signDoc = (s.lines || []).map(runs => {
    const line = [];
    (runs || []).forEach(run => {
      if (run.y) line.push({ y: run.y, s: run.s, p: run.p });
      else for (const ch of String(run.t || '')) line.push({ c: ch, s: run.s, p: run.p });
    });
    return line;
  });
  if (!signDoc.length) signDoc = [[]];
  signBoardSwitch.set(sign.board);
  signMountSwitch.set(sign.mount);
  signLightSwitch.set(sign.light);
  segSelect($('sign-tone'), 'tone', sign.tone);
  segSelect($('sign-font'), 'font', sign.font);
  segSelect($('sign-dir'), 'dir', sign.vertical ? 'v' : 'h');
  segSelect($('sign-sides'), 'sides', sign.double ? '2' : '1');
  const paints = [null].concat(signData.paints.map(p => p.key));
  $('sign-border').querySelectorAll('.color-swatch').forEach((el, i) => el.classList.toggle('active', paints[i] === sign.border));
  signWordSliders();
  // the sliders start from where the sign already is: nothing moved, tilted and leaning as it was
  slideReset();
  slide.rx = Math.round((edit.rx || 0) * 2) / 2;
  slide.ry = Math.round((edit.ry || 0) * 2) / 2;
  ['rx', 'ry'].forEach(key => {
    const row = signSlidersEl.querySelector('.slider-row[data-key="' + key + '"]');
    row.querySelector('.slider').value = slide[key];
    row.querySelector('.slider-val').value = slide[key];
  });
  sceneMode = 'sign';
  signDressInput();
  const last = signDoc.length - 1;
  signLast = { from: { l: last, i: signDoc[last].length }, to: { l: last, i: signDoc[last].length } };
}

const signBoardSwitch = makeSwitcher($('sign-board'), [], v => { sign.board = v; signRefresh(); });
const signMountSwitch = makeSwitcher($('sign-mount'), [], v => { sign.mount = v; signRefresh(); });
const signLightSwitch = makeSwitcher($('sign-light'), [], v => { sign.light = v; signRefresh(); });

/** A symbol as it sits in the words: a picture cut out of the paint's colour, sized with the text. */
function signSymbolEl(y) {
  const el = document.createElement('span');
  el.className = 'sym' + (y.flourish ? ' sym-flourish' : '');
  el.dataset.sym = y.key;
  el.contentEditable = 'false';
  el.style.setProperty('--w', y.w);
  el.style.webkitMaskImage = 'url(sym_' + y.key + '.png)';
  return el;
}
const signSizeSwitch = makeSwitcher($('sign-size'),
  [{ value: 1, label: 'Large' }, { value: 2, label: 'Medium' }, { value: 3, label: 'Small' }], v => {
    signSize = v;                                 // what is selected takes it; so does what is typed next
    signApply(it => { it.s = v; });
  });

[signInput].forEach(el => {
  el.addEventListener('keydown', e => {
    e.stopPropagation();
    if (e.key === 'Escape') { e.preventDefault(); cancelText(); }
    if ((e.ctrlKey || e.metaKey) && (e.key === 'z' || e.key === 'Z')) { e.preventDefault(); signUndoLast(); }
  });
  el.addEventListener('keyup',    e => e.stopPropagation());
  el.addEventListener('keypress', e => e.stopPropagation());
});

// ── sections that fold away ──
// The sign editor has a lot in it. Each section opens and closes on its heading; which are open is
// remembered for as long as the game runs, so the editor comes back the way it was left.
document.querySelectorAll('#scene-sign-mode .sec').forEach(sec => {
  sec.querySelector('.sec-head').addEventListener('mousedown', e => e.preventDefault());   // the words keep the focus
  sec.querySelector('.sec-head').addEventListener('click', () => sec.classList.toggle('open'));
});

// ── how the words sit on the board: line spacing, and up or down ──
const signWordsEl = $('sign-words');
function signWordSliders() {
  signWordsEl.querySelectorAll('.slider-row').forEach(row => {
    row.querySelector('.slider').value = sign[row.dataset.key];
    row.querySelector('.slider-val').value = sign[row.dataset.key];
  });
}
signWordsEl.querySelectorAll('.slider-row').forEach(row => {
  const key = row.dataset.key, range = row.querySelector('.slider'), num = row.querySelector('.slider-val');
  const min = parseFloat(range.min), max = parseFloat(range.max), step = parseFloat(range.step);
  const set = v => {
    if (isNaN(v)) return;
    sign[key] = Math.round(Math.min(max, Math.max(min, Math.round(v / step) * step)) * 10) / 10;
    range.value = sign[key];
    num.value = sign[key];
    signRefresh();
  };
  range.addEventListener('input', () => set(parseFloat(range.value)));
  num.addEventListener('change', () => set(parseFloat(num.value)));
  row.addEventListener('wheel', e => {
    e.preventDefault();
    e.stopPropagation();
    set(sign[key] + (e.deltaY < 0 ? step : -step) * (e.shiftKey ? 10 : 1));
  }, { passive: false });
  num.addEventListener('keydown', e => {
    e.stopPropagation();
    if (e.key === 'Escape') { e.preventDefault(); cancelText(); }
    if (e.key === 'Enter') { set(parseFloat(num.value)); num.blur(); }
  });
  num.addEventListener('keyup',    e => e.stopPropagation());
  num.addEventListener('keypress', e => e.stopPropagation());
});

// ── SIGN PLACEMENT SLIDERS ────────────────────────────────────────
// As in Poggy Badges: a slider and a number box per value, the wheel works on both.
// Position is measured from where the sign first appeared, as the player stood: right, away, up.
const signSlidersEl = $('sign-sliders');
const slide = { dx: 0, dy: 0, dz: 0, rx: 0, ry: 0, rz: 0 };
let slideTimer = null;

function slideSend() {
  clearTimeout(slideTimer);
  slideTimer = setTimeout(() => luaCall('sign_live', { slide }), 30);
}

signSlidersEl.querySelectorAll('.slider-row').forEach(row => {
  const key = row.dataset.key, range = row.querySelector('.slider'), num = row.querySelector('.slider-val');
  const min = parseFloat(range.min), max = parseFloat(range.max), step = parseFloat(range.step);
  const set = v => {
    if (isNaN(v)) return;
    v = Math.min(max, Math.max(min, Math.round(v / step) * step));
    slide[key] = parseFloat(v.toFixed(2));
    range.value = slide[key];
    num.value = slide[key];
    slideSend();
  };
  range.addEventListener('input', () => set(parseFloat(range.value)));
  num.addEventListener('change', () => set(parseFloat(num.value)));
  row.addEventListener('wheel', e => {
    e.preventDefault();
    e.stopPropagation();
    set(slide[key] + (e.deltaY < 0 ? step : -step) * (e.shiftKey ? 10 : 1));
  }, { passive: false });
  num.addEventListener('keydown', e => {
    e.stopPropagation();
    if (e.key === 'Escape') { e.preventDefault(); cancelText(); }
    if (e.key === 'Enter') { set(parseFloat(num.value)); num.blur(); }
  });
  num.addEventListener('keyup',    e => e.stopPropagation());
  num.addEventListener('keypress', e => e.stopPropagation());
});

function slideReset() {
  Object.keys(slide).forEach(k => { slide[k] = 0; });
  signSlidersEl.querySelectorAll('.slider-row').forEach(row => {
    row.querySelector('.slider').value = 0;
    row.querySelector('.slider-val').value = 0;
  });
}

// Remove: only when the editor was opened on a scene that is already placed. Two clicks, so a slip
// of the mouse does not take a scene away.
const removeBtn = $('btn-text-remove');
let removeArmed = null;
function removeBtnReset() {
  clearTimeout(removeArmed);
  removeArmed = null;
  removeBtn.textContent = 'Remove';
  removeBtn.classList.remove('armed');
  removeBtn.classList.toggle('hidden', !sceneEditing);
}
removeBtn.addEventListener('click', () => {
  if (!removeArmed) {
    removeBtn.textContent = 'Click again to remove';
    removeBtn.classList.add('armed');
    removeArmed = setTimeout(removeBtnReset, 3500);
    return;
  }
  removeBtnReset();
  showPhase(null);
  luaCall('scene_remove');
});

$('btn-text-cancel').addEventListener('click',  cancelText);
$('btn-text-confirm').addEventListener('click', confirmText);

// ── PHASE 2 CONTROLS ─────────────────────────────────────────────

const AXES = ['x', 'y', 'z'];
const axisInput = { x: inputX, y: inputY, z: inputZ };

AXES.forEach(axis => {
  const stepper = document.querySelector(`.cc-stepper[data-axis="${axis}"]`);
  const el      = axisInput[axis];

  function nudge(direction) {
    pos[axis] = parseFloat((pos[axis] + direction * 0.1).toFixed(2));
    el.value  = fmt(pos[axis]);
    schedulePositionUpdate();
  }

  // Arrow buttons (step 0.1)
  stepper.querySelector('.step-left').addEventListener('click',  () => nudge(-1));
  stepper.querySelector('.step-right').addEventListener('click', () => nudge(1));

  // Scroll wheel on the stepper
  stepper.addEventListener('wheel', e => {
    e.preventDefault();   // stop game from receiving scroll (weapon wheel)
    e.stopPropagation();
    nudge(e.deltaY < 0 ? 1 : -1);
  }, { passive: false });

  // Manual input
  el.addEventListener('input', () => poggycInputToPos(axis));

  // Clamp/format on blur
  el.addEventListener('blur', () => {
    el.value = fmt(pos[axis]);
  });

  // Block key events from reaching game while typing in inputs
  el.addEventListener('keydown', e => {
    e.stopPropagation();
    if (e.key === 'Escape') cancelPosition();  // reverts to editor
    if (e.key === 'Enter') { poggycInputToPos(axis); el.blur(); }
  });
  el.addEventListener('keyup',    e => e.stopPropagation());
  el.addEventListener('keypress', e => e.stopPropagation());
});

function cancelPosition() {
  // Revert to the text editor so the user doesn't lose their work
  luaCall('scene_place_revert');
  textInput.innerHTML = savedEditorHtml || '';
  showPhase('text');
  setSceneMode('text');
}

let savedEditorHtml = '';

function placeScene() {
  showPhase(null);
  luaCall('scene_place_confirm', { x: pos.x, y: pos.y, z: pos.z });
}

$('btn-position-cancel').addEventListener('click', cancelPosition);
$('btn-position-place').addEventListener('click',  placeScene);

// Keyboard escape on the panel
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') {
    e.stopPropagation();
    if (meboxMoveActive) { exitMeboxMoveMode(true); return; }
    if (currentPhase === 'text')     cancelText();
    if (currentPhase === 'position') cancelPosition();
    if (currentPhase === 'status')   { luaCall('status_cancel'); showPhase(null); }
  }
});

// ── HOVER STATE (tells Lua when mouse is over any panel) ──────────
// This lets Lua disable game controls that would conflict.
[textPanel, posPanel, statusPanel].forEach(panel => {
  panel.addEventListener('mouseenter', () => luaCall('scene_hover', { hovered: true  }));
  panel.addEventListener('mouseleave', () => luaCall('scene_hover', { hovered: false }));
});

// Prevent right-click context menu
document.addEventListener('contextmenu', e => e.preventDefault());

// ── Click-hold to orbit camera (position phase only) ──────────────
let cameraHeld = false;
document.addEventListener('mousedown', (e) => {
  // Only during position phase, clicking outside the panel orbits camera
  const looking = currentPhase === 'position' ? !posPanel.contains(e.target)
    : (currentPhase === 'text' && sceneMode === 'sign' && !textPanel.contains(e.target));
  if (looking && (e.button === 0 || e.button === 2)) {
    cameraHeld = true;
    luaCall('scene_camera_hold', { holding: true });
  }
});
document.addEventListener('mouseup', (e) => {
  if ((e.button === 0 || e.button === 2) && cameraHeld) {
    cameraHeld = false;
    luaCall('scene_camera_hold', { holding: false });
  }
});

// ── STATUS CONTROLS ────────────────────────────────────────────────
let statusEditorColor = '#ffffff';

function statusEditorSetDefaults() {
  statusEditorColor = '#ffffff';
  statusEditorColorsContainer.querySelectorAll('.color-swatch').forEach(s =>
    s.classList.toggle('active', s.dataset.color === '#ffffff')
  );
}

// (Status color swatch handlers are bound dynamically via buildSwatches)

function applyStatus() {
  const html = statusTextInput.innerHTML.trim();
  if (!html || statusTextInput.textContent.trim() === '') return;
  const code = generateSceneCode(statusTextInput);
  luaCall('status_apply', { text: code });
  showPhase(null);
}

function clearStatus() {
  luaCall('status_clear');
  showPhase(null);
}

function savePreset() {
  const html = statusTextInput.innerHTML.trim();
  if (!html || statusTextInput.textContent.trim() === '') return;
  const text = generateSceneCode(statusTextInput);
  const name = statusNameInput.value.trim();
  luaCall('status_save', { text, name });
}

function buildPresetList(presets) {
  const list = $('status-presets-list');
  list.innerHTML = '';
  if (!presets || presets.length === 0) {
    list.innerHTML = '<div class="presets-empty">No saved presets</div>';
    return;
  }
  presets.forEach(p => {
    const item    = document.createElement('div');
    item.className = 'preset-item';

    const nameEl        = document.createElement('span');
    nameEl.className    = 'preset-item-name';
    nameEl.textContent  = p.name || 'Unnamed';
    nameEl.title        = p.text;
    nameEl.addEventListener('click', () => {
      // Render the preset's RDR2 markup into colored HTML for preview
      statusTextInput.innerHTML = presetCodeToHtml(p.text);
    });

    const applyBtn      = document.createElement('button');
    applyBtn.className  = 'preset-btn preset-btn-apply';
    applyBtn.textContent = '▶';
    applyBtn.title      = 'Apply';
    applyBtn.addEventListener('click', () => { luaCall('status_apply', { text: p.text }); showPhase(null); });

    const delBtn        = document.createElement('button');
    delBtn.className    = 'preset-btn preset-btn-delete';
    delBtn.textContent  = '✕';
    delBtn.title        = 'Delete';
    delBtn.addEventListener('click', () => { luaCall('status_delete', { id: p.id }); });

    item.appendChild(nameEl);
    item.appendChild(applyBtn);
    item.appendChild(delBtn);
    list.appendChild(item);
  });
}

[statusTextInput, statusNameInput].forEach(el => {
  el.addEventListener('keydown', e => {
    e.stopPropagation();
    if (e.key === 'Escape') { e.preventDefault(); luaCall('status_cancel'); showPhase(null); }
  });
  el.addEventListener('keyup',    e => e.stopPropagation());
  el.addEventListener('keypress', e => e.stopPropagation());
});

$('btn-status-cancel').addEventListener('click', () => { luaCall('status_cancel'); showPhase(null); });
$('btn-status-clear').addEventListener('click',  clearStatus);
$('btn-status-apply').addEventListener('click',  applyStatus);
$('btn-status-save').addEventListener('click',   savePreset);

// ================================================================
//  ME Display Box — NUI-side logic
// ================================================================

const meboxRoot      = $('mebox-root');
const meboxContainer = $('mebox-container');
const meboxMessages  = $('mebox-messages');
const meboxMoveOverlay = $('mebox-move-overlay');
let   meboxMode        = 'auto';    // 'auto' | 'persist' | 'off'
let   meboxFadeTimer   = null;
let   meboxMaxLines    = 7;
let   meboxHistoryMax  = 50;        // max scrollback messages (set from Config.meboxhistory)
let   meboxDissipateMs = 300000;    // ms before a message is removed (set from Config.meboxdissipateafter)
let   meboxMoveActive  = false;
let   meboxDragging   = false;
let   meboxDragOffX   = 0;
let   meboxDragOffY   = 0;
let   meboxHasCustomPos = false;

function meboxShow() {
  const wasHidden = meboxRoot.classList.contains('mebox-hidden') ||
                   meboxRoot.classList.contains('mebox-fading');
  meboxRoot.classList.remove('mebox-hidden', 'mebox-fading');
  if (wasHidden) {
    // Reset scroll to bottom so only recent messages are visible on reopen;
    // old messages remain accessible by scrolling up manually.
    setTimeout(() => { meboxContainer.scrollTop = meboxContainer.scrollHeight; }, 0);
  }
}

function meboxFade() {
  meboxRoot.classList.add('mebox-fading');
  // After transition completes, fully hide
  setTimeout(() => {
    if (meboxRoot.classList.contains('mebox-fading')) {
      meboxRoot.classList.add('mebox-hidden');
      meboxRoot.classList.remove('mebox-fading');
    }
  }, 1600);
}

function meboxHide() {
  meboxRoot.classList.add('mebox-hidden');
  meboxRoot.classList.remove('mebox-fading');
}

function meboxClearFadeTimer() {
  if (meboxFadeTimer) {
    clearTimeout(meboxFadeTimer);
    meboxFadeTimer = null;
  }
}

function meboxResetFadeTimer(timeout) {
  meboxClearFadeTimer();
  if (meboxMode === 'auto' && timeout > 0) {
    meboxFadeTimer = setTimeout(() => {
      meboxFade();
    }, timeout);
  }
}

function meboxAddLine(name, nameColor, text) {
  const line = document.createElement('div');
  line.className = 'mebox-line';

  const nameEl = document.createElement('span');
  nameEl.className = 'mebox-name';
  nameEl.style.color = nameColor || '#e8dfc8';
  nameEl.textContent = name + ':';

  const textEl = document.createElement('span');
  textEl.className = 'mebox-text';
  textEl.textContent = ' ' + text.trim();

  line.appendChild(nameEl);
  line.appendChild(textEl);
  // Tag with expiry so the cleanup interval can remove stale messages
  line.dataset.expires = String(Date.now() + meboxDissipateMs);
  meboxMessages.appendChild(line);

  // Keep a generous history for scroll-back, trim only the oldest
  while (meboxMessages.children.length > meboxHistoryMax) {
    meboxMessages.removeChild(meboxMessages.firstChild);
  }

  // Auto-scroll to bottom
  meboxContainer.scrollTop = meboxContainer.scrollHeight;
}

// Wheel handler — stop the event from reaching the game (weapon wheel)
// but DON'T preventDefault so the browser's native overflow scroll works.
meboxContainer.addEventListener('wheel', function(e) {
  e.stopPropagation();
}, { passive: false });

// Dissipate messages after their per-message lifetime expires
setInterval(() => {
  const now = Date.now();
  Array.from(meboxMessages.children).forEach(line => {
    const exp = parseInt(line.dataset.expires || '0');
    if (exp && now > exp) line.remove();
  });
}, 30000);


// ── Mebox move/drag support ─────────────────────────────────────
function meboxApplyPosition(top, left) {
  meboxRoot.style.top = top;
  meboxRoot.style.left = left;
  meboxRoot.style.transform = 'none';
  meboxHasCustomPos = true;
}

function exitMeboxMoveMode(doSave) {
  meboxMoveActive = false;
  meboxDragging = false;
  if (meboxMoveOverlay) meboxMoveOverlay.style.display = 'none';
  meboxRoot.style.pointerEvents = 'none';
  meboxRoot.style.cursor = 'default';
  meboxRoot.style.zIndex = '800';
  // Re-enable pointer-events on the container for scrolling
  meboxContainer.style.pointerEvents = 'all';
  if (doSave) {
    luaCall('meboxPositionSave', { top: meboxRoot.style.top, left: meboxRoot.style.left });
  }
  luaCall('meboxMoveExit');
}

meboxRoot.addEventListener('mousedown', function(e) {
  if (!meboxMoveActive) return;
  meboxDragging = true;
  const rect = meboxRoot.getBoundingClientRect();
  meboxDragOffX = e.clientX - rect.left;
  meboxDragOffY = e.clientY - rect.top;
  meboxRoot.style.cursor = 'grabbing';
  e.preventDefault();
  e.stopPropagation();
});

document.addEventListener('mousemove', function(e) {
  if (!meboxDragging) return;
  const newLeft = Math.max(0, Math.min(window.innerWidth - meboxRoot.offsetWidth, e.clientX - meboxDragOffX));
  const newTop  = Math.max(0, Math.min(window.innerHeight - meboxRoot.offsetHeight, e.clientY - meboxDragOffY));
  meboxApplyPosition(newTop + 'px', newLeft + 'px');
});

document.addEventListener('mouseup', function() {
  if (!meboxDragging) return;
  meboxDragging = false;
  meboxRoot.style.cursor = 'grab';
});

if (meboxMoveOverlay) {
  meboxMoveOverlay.addEventListener('click', function(e) {
    if (e.target === meboxMoveOverlay) {
      if (meboxMoveActive) exitMeboxMoveMode(true);
    }
  });
}

// ── Mebox NUI message listener ──────────────────────────────────
window.addEventListener('message', function(event) {
  const msg = event.data;
  if (!msg || !msg.type) return;

  switch (msg.type) {

    case 'mebox_scroll':
      meboxContainer.scrollTop += (msg.delta || 0);
      break;

    case 'mebox_add':
      if (msg.maxLines)    meboxMaxLines    = msg.maxLines;
      if (msg.dissipateMs) meboxDissipateMs = msg.dissipateMs;
      meboxAddLine(msg.name, msg.nameColor, msg.text);
      meboxShow();
      meboxResetFadeTimer(msg.timeout || 60000);
      break;

    case 'mebox_fade':
      meboxFade();
      break;

    case 'mebox_hide':
      meboxClearFadeTimer();
      meboxHide();
      break;

    case 'mebox_mode':
      meboxMode = msg.mode || 'auto';
      if (meboxMode === 'persist') {
        meboxClearFadeTimer();
        if (meboxMessages.children.length > 0) meboxShow();
      } else if (meboxMode === 'auto') {
        if (meboxMessages.children.length > 0) {
          meboxShow();
          meboxResetFadeTimer(msg.timeout || 60000);
        }
      } else if (meboxMode === 'off') {
        meboxClearFadeTimer();
        meboxHide();
      }
      break;

    case 'mebox_move_mode':
      if (msg.enabled) {
        meboxMoveActive = true;
        meboxRoot.classList.remove('mebox-hidden', 'mebox-fading');
        meboxRoot.style.pointerEvents = 'all';
        meboxRoot.style.cursor = 'grab';
        meboxRoot.style.zIndex = '9999';
        meboxContainer.style.pointerEvents = 'none'; // disable scroll during drag
        if (meboxMoveOverlay) meboxMoveOverlay.style.display = 'block';
        // Show placeholder text if no messages
        if (meboxMessages.children.length === 0) {
          const hint = document.createElement('div');
          hint.className = 'mebox-line mebox-move-placeholder';
          hint.textContent = 'Drag to reposition — press ESC to save';
          meboxMessages.appendChild(hint);
        }
      } else {
        exitMeboxMoveMode(false);
      }
      break;

    case 'mebox_position_load':
      if (msg.top && msg.left) {
        meboxApplyPosition(msg.top, msg.left);
      }
      break;

    case 'mebox_size':
      meboxRoot.classList.remove('mebox-small', 'mebox-normal', 'mebox-large');
      if (msg.size === 'small' || msg.size === 'normal' || msg.size === 'large') {
        meboxRoot.classList.add('mebox-' + msg.size);
      }
      break;

    case 'mebox_show_if_messages':
      if (meboxMode !== 'off' && meboxMessages.children.length > 0) {
        meboxShow();
        if (meboxMode === 'auto') {
          meboxResetFadeTimer(msg.timeout || 60000);
        }
      }
      break;

    case 'mebox_move_cleanup':
      // Remove placeholder text added during move mode
      const placeholders = meboxMessages.querySelectorAll('.mebox-move-placeholder');
      placeholders.forEach(el => el.remove());
      // If no real messages left, hide
      if (meboxMessages.children.length === 0 && meboxMode !== 'persist') {
        meboxHide();
      }
      break;
  }
});
