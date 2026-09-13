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
  const el = axis === 'x' ? inputX : axis === 'y' ? inputY : inputZ;
  const v  = parseFloat(el.value);
  if (!isNaN(v)) {
    pos[axis] = v;
    schedulePositionUpdate();
  }
}

// Write pos → inputs (without firing input events)
function flushPosToInputs() {
  inputX.value = fmt(pos.x);
  inputY.value = fmt(pos.y);
  inputZ.value = fmt(pos.z);
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

const fontSizeSelect = $('editor-font-size');
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
  fontSizeSelect.value = String(DEFAULT_SIZE);
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

// Font size select — save selection before dropdown steals focus
fontSizeSelect.addEventListener('mousedown', () => {
  saveEditorSelection();
});

fontSizeSelect.addEventListener('change', () => {
  editorSize = parseInt(fontSizeSelect.value) || DEFAULT_SIZE;
  restoreEditorSelection();
  applyFontSizeToSelection(editorSize);
  textInput.focus();
});

// Stop font size select from leaking keys
fontSizeSelect.addEventListener('keydown', e => e.stopPropagation());
fontSizeSelect.addEventListener('keyup', e => e.stopPropagation());

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

$('btn-text-cancel').addEventListener('click',  cancelText);
$('btn-text-confirm').addEventListener('click', confirmText);

// ── PHASE 2 CONTROLS ─────────────────────────────────────────────

const AXES = ['x', 'y', 'z'];
const axisInput = { x: inputX, y: inputY, z: inputZ };

AXES.forEach(axis => {
  const stepper = document.querySelector(`.cc-stepper[data-axis="${axis}"]`);
  const el      = axisInput[axis];

  // Arrow buttons (step 0.1)
  stepper.querySelector('.step-left').addEventListener('click', () => {
    pos[axis] = parseFloat((pos[axis] - 0.1).toFixed(2));
    el.value  = fmt(pos[axis]);
    schedulePositionUpdate();
  });
  stepper.querySelector('.step-right').addEventListener('click', () => {
    pos[axis] = parseFloat((pos[axis] + 0.1).toFixed(2));
    el.value  = fmt(pos[axis]);
    schedulePositionUpdate();
  });

  // Scroll wheel on the stepper
  stepper.addEventListener('wheel', e => {
    e.preventDefault();   // stop game from receiving scroll (weapon wheel)
    e.stopPropagation();
    const delta = e.deltaY < 0 ? 0.1 : -0.1;
    pos[axis] = parseFloat((pos[axis] + delta).toFixed(2));
    el.value  = fmt(pos[axis]);
    schedulePositionUpdate();
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
  if (currentPhase === 'position' && (e.button === 0 || e.button === 2) && !posPanel.contains(e.target)) {
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
