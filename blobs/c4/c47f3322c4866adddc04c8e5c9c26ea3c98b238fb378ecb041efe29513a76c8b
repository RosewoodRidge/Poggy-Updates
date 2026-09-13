/* ───────────────────────────────────────────────────────────
   Poggy Badge – NUI Script
   Communicates with client/main.lua via NUI callbacks / messages
   ─────────────────────────────────────────────────────────── */

const RESOURCE = GetParentResourceName();
const editorRoot = document.getElementById('editor-root');
const badgeDisplay = document.getElementById('badge-display');
const badgeImage = document.getElementById('badge-image');
const badgeName = document.getElementById('badge-name');
const badgeServer = document.getElementById('badge-server');
const presetList = document.getElementById('preset-list');
const presetNameInput = document.getElementById('preset-name');
const badgePreviewImg = document.getElementById('badge-preview-img');
const editorPanel = document.querySelector('.editor-panel');
const toggleAttachBtn = document.getElementById('btn-toggle-attach');
const boneSelect = document.getElementById('bone-select');

/* ── State ─────────────────────────────────────────────────── */
let currentValues = { ox: 0, oy: 0, oz: 0, rx: 0, ry: 0, rz: 0 };
let currentBone = 'SKEL_Spine5';
let presets = [];
let sendThrottle = null;
let isAttached = false;

/* ── Slider binding ────────────────────────────────────────── */
const sliderRows = document.querySelectorAll('.slider-row');
sliderRows.forEach(row => {
  const key   = row.dataset.key;
  const range = row.querySelector('.slider');
  const num   = row.querySelector('.slider-val');

  // Range → number sync + send
  range.addEventListener('input', () => {
    const v = parseFloat(range.value);
    num.value = v;
    currentValues[key] = v;
    throttleSend();
  });

  // Number box → range sync + send
  num.addEventListener('input', () => {
    let v = parseFloat(num.value);
    if (isNaN(v)) return;
    v = clamp(v, parseFloat(range.min), parseFloat(range.max));
    range.value = v;
    num.value = v;
    currentValues[key] = v;
    throttleSend();
  });

  // Mouse wheel on both slider and number box
  const step = parseFloat(row.dataset.step) || 0.001;
  const wheelHandler = (e) => {
    e.preventDefault();
    const dir = e.deltaY < 0 ? 1 : -1;
    let v = currentValues[key] + dir * step;
    v = clamp(v, parseFloat(range.min), parseFloat(range.max));
    v = round(v, step);
    range.value = v;
    num.value = v;
    currentValues[key] = v;
    throttleSend();
  };
  range.addEventListener('wheel', wheelHandler, { passive: false });
  num.addEventListener('wheel', wheelHandler, { passive: false });
});

/* ── Utility ───────────────────────────────────────────────── */
function clamp(v, mn, mx) { return Math.min(mx, Math.max(mn, v)); }
function round(v, step) {
  const d = step < 1 ? (step.toString().split('.')[1] || '').length : 0;
  return parseFloat(v.toFixed(d));
}

// Nudge rotation values off whole numbers — the engine resets rotation
// axes that land exactly on an integer (pitch snaps to default, roll/yaw vanish).
function safeValues() {
  const out = { ...currentValues };
  ['rx', 'ry', 'rz'].forEach(k => {
    if (Number.isInteger(out[k]) || out[k] === Math.floor(out[k])) {
      out[k] += 0.1;
    }
  });
  return out;
}

function throttleSend() {
  if (sendThrottle) return;
  sendThrottle = setTimeout(() => {
    sendThrottle = null;
    postNUI('badge_update', safeValues());
  }, 50);            // ~20 fps update rate – gives engine breathing room
}

function postNUI(name, data) {
  fetch(`https://${RESOURCE}/${name}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data || {})
  }).catch(() => {});
}

/* ── Update the toggle attach/detach button ────────────────── */
function updateToggleBtn() {
  if (isAttached) {
    toggleAttachBtn.textContent = 'Detach';
    toggleAttachBtn.className = 'btn btn-detach';
  } else {
    toggleAttachBtn.textContent = 'Attach';
    toggleAttachBtn.className = 'btn btn-attach';
  }
}

/* ── Set slider UI from values object ──────────────────────── */
function setSliders(vals) {
  for (const [key, v] of Object.entries(vals)) {
    currentValues[key] = v;
    const row = document.querySelector(`.slider-row[data-key="${key}"]`);
    if (!row) continue;
    row.querySelector('.slider').value = v;
    row.querySelector('.slider-val').value = v;
  }
}

/* ── Preset rendering ──────────────────────────────────────── */
function renderPresets() {
  presetList.innerHTML = '';
  if (presets.length === 0) {
    presetList.innerHTML = '<div class="presets-empty">No saved presets</div>';
    return;
  }
  presets.forEach(p => {
    const item = document.createElement('div');
    item.className = 'preset-item';

    const name = document.createElement('span');
    name.className = 'preset-item-name';
    name.textContent = p.name;
    name.title = 'Click to load';
    name.addEventListener('click', () => {
      postNUI('badge_load_preset', { id: p.id, bone: p.bone || 'SKEL_Spine5', offset_x: p.offset_x, offset_y: p.offset_y, offset_z: p.offset_z, rot_x: p.rot_x, rot_y: p.rot_y, rot_z: p.rot_z });
    });

    const del = document.createElement('button');
    del.className = 'preset-item-del';
    del.textContent = '✕';
    del.title = 'Delete preset';
    del.addEventListener('click', () => {
      postNUI('badge_delete_preset', { id: p.id });
    });

    item.appendChild(name);
    item.appendChild(del);
    presetList.appendChild(item);
  });
}

/* ── Action buttons ────────────────────────────────────────── */
document.getElementById('btn-close').addEventListener('click', () => postNUI('badge_close'));
document.getElementById('btn-show').addEventListener('click', () => postNUI('badge_show'));

/* ── Click-hold to orbit camera ─────────────────────────────── */
// Prevent context menu on right-click
document.addEventListener('contextmenu', (e) => e.preventDefault());

let cameraHeld = false;
document.addEventListener('mousedown', (e) => {
  // Only trigger if clicking outside the editor panel (on the game area)
  if ((e.button === 0 || e.button === 2) && !editorPanel.contains(e.target)) {
    cameraHeld = true;
    postNUI('badge_camera_hold', { holding: true });
  }
});
document.addEventListener('mouseup', (e) => {
  if ((e.button === 0 || e.button === 2) && cameraHeld) {
    cameraHeld = false;
    postNUI('badge_camera_hold', { holding: false });
  }
});

/* Toggle attach / detach */
toggleAttachBtn.addEventListener('click', () => {
  postNUI('badge_toggle_attach', {});
});

document.getElementById('btn-save-preset').addEventListener('click', () => {
  const name = presetNameInput.value.trim();
  if (!name) return;
  postNUI('badge_save_preset', { name, bone: currentBone, ...currentValues });
  presetNameInput.value = '';
});

/* Escape key closes */
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    postNUI('badge_close');
  }
});

/* Bone selector */
boneSelect.addEventListener('change', () => {
  currentBone = boneSelect.value;
  postNUI('badge_bone_change', { bone: currentBone });
});

/* ── Messages from Lua ─────────────────────────────────────── */
window.addEventListener('message', (ev) => {
  const d = ev.data;
  switch (d.type) {
    case 'open':
      setSliders(d.values || currentValues);
      presets = d.presets || [];
      isAttached = !!d.attached;
      currentBone = d.bone || 'SKEL_Spine5';
      boneSelect.value = currentBone;
      updateToggleBtn();
      if (d.badgeImage) {
        badgePreviewImg.src = 'images/' + d.badgeImage + '.png';
        badgePreviewImg.style.display = '';
      } else {
        badgePreviewImg.style.display = 'none';
      }
      renderPresets();
      editorRoot.classList.remove('hidden');
      badgeDisplay.classList.add('hidden');
      badgeDisplay.classList.remove('active');
      break;

    case 'close':
      editorRoot.classList.add('hidden');
      break;

    case 'load_values':
      setSliders(d.values);
      if (d.bone) {
        currentBone = d.bone;
        boneSelect.value = currentBone;
      }
      break;

    case 'update_presets':
      presets = d.presets || [];
      renderPresets();
      break;

    case 'update_attached':
      isAttached = !!d.attached;
      updateToggleBtn();
      break;

    case 'displayBadge':
      showBadgeFlash(d.badgeImage, d.displayName, d.serverName);
      break;
  }
});

/* ── Badge flash (nearby players see this) ─────────────────── */
let badgeTimeout = null;
function showBadgeFlash(img, name, server) {
  if (badgeTimeout) { clearTimeout(badgeTimeout); badgeTimeout = null; }

  // Remove any lingering animation
  badgeDisplay.classList.remove('active');
  badgeDisplay.classList.add('hidden');

  // Set content
  badgeImage.style.backgroundImage = img ? `url('images/${img}.png')` : 'none';
  badgeName.textContent = name || '';
  badgeServer.textContent = server || '';

  // Trigger animation
  void badgeDisplay.offsetWidth;           // reflow
  badgeDisplay.classList.remove('hidden');
  badgeDisplay.classList.add('active');

  // Auto-hide after 5s
  badgeTimeout = setTimeout(() => {
    badgeDisplay.classList.remove('active');
    badgeDisplay.classList.add('hidden');
    badgeTimeout = null;
  }, 5000);
}
