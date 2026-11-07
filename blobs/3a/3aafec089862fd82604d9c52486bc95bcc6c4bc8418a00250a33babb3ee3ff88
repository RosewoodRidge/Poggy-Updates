/* Built by tools/ui-vue from redm/poggy_scenes_2/ui-src. Edit the source there, not this file. */
(function(ui$1, vue) {
  "use strict";
  const ui = vue.reactive({ phase: null, sceneMode: "text" });
  const DEFAULT_COLOR = "#ffffff";
  const DEFAULT_SIZE = 20;
  const colours = vue.reactive({ order: [] });
  const COLOR_MAP = {};
  const CODE_TO_HEX = {};
  function loadColours(list) {
    Object.keys(COLOR_MAP).forEach((k) => delete COLOR_MAP[k]);
    Object.keys(CODE_TO_HEX).forEach((k) => delete CODE_TO_HEX[k]);
    colours.order = (list || []).map((c) => {
      const hex = String(c.hex).toLowerCase();
      COLOR_MAP[hex] = c.code;
      CODE_TO_HEX[c.code] = hex;
      return { key: hex, hex, code: c.code, title: c.title || c.code };
    });
  }
  function rgbToHex(rgb) {
    if (!rgb) return DEFAULT_COLOR;
    if (rgb.startsWith("#")) return rgb.toLowerCase();
    const m = rgb.match(/rgba?\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)/);
    if (!m) return DEFAULT_COLOR;
    return ("#" + [m[1], m[2], m[3]].map((x) => parseInt(x).toString(16).padStart(2, "0")).join("")).toLowerCase();
  }
  function closestColorCode(hex) {
    if (COLOR_MAP[hex]) return COLOR_MAP[hex];
    const r = parseInt(hex.slice(1, 3), 16), g = parseInt(hex.slice(3, 5), 16), b = parseInt(hex.slice(5, 7), 16);
    let best = "~q~", bestDist = Infinity;
    for (const [h, code] of Object.entries(COLOR_MAP)) {
      const d = (parseInt(h.slice(1, 3), 16) - r) ** 2 + (parseInt(h.slice(3, 5), 16) - g) ** 2 + (parseInt(h.slice(5, 7), 16) - b) ** 2;
      if (d < bestDist) {
        bestDist = d;
        best = code;
      }
    }
    return best;
  }
  function generateSceneCode(rootEl) {
    const segments = [];
    function walk(node, parentSize, parentColor) {
      if (node.nodeType === Node.TEXT_NODE) {
        const t = node.textContent.replace(/​/g, "");
        if (t) segments.push({ text: t, size: parentSize, color: parentColor });
        return;
      }
      if (node.nodeType !== Node.ELEMENT_NODE) return;
      let size = parentSize, color = parentColor;
      if (node.style && node.style.fontSize) {
        const m = node.style.fontSize.match(/(\d+)/);
        if (m) size = parseInt(m[1]);
      }
      if (node.style && node.style.color) color = rgbToHex(node.style.color);
      if (node.tagName === "FONT" && node.getAttribute("color")) color = rgbToHex(node.getAttribute("color"));
      if (node.tagName === "BR") {
        segments.push({ text: "\n", size, color });
        return;
      }
      if (node.childNodes.length === 0) {
        if (node.textContent) segments.push({ text: node.textContent, size, color });
        return;
      }
      if ((node.tagName === "DIV" || node.tagName === "P") && node !== rootEl && node.previousSibling) {
        segments.push({ text: "\n", size, color });
      }
      node.childNodes.forEach((child) => walk(child, size, color));
    }
    walk(rootEl, DEFAULT_SIZE, DEFAULT_COLOR);
    let result = "", curSize = null, curColor = null, curText = "";
    const flush = () => {
      if (!curText) return;
      result += '<font size="' + (curSize || DEFAULT_SIZE) + '">' + closestColorCode(curColor || DEFAULT_COLOR) + curText.toUpperCase() + "</font>";
      curText = "";
    };
    for (const seg of segments) {
      if (seg.text === "\n") {
        flush();
        result += "~n~";
        curSize = null;
        curColor = null;
        continue;
      }
      if (curSize !== null && (curSize !== seg.size || curColor !== seg.color)) flush();
      curSize = seg.size;
      curColor = seg.color;
      curText += seg.text;
    }
    flush();
    return result;
  }
  const esc = (t) => t.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
  function sceneCodeToHtml(code) {
    let html = "", size = DEFAULT_SIZE, color = DEFAULT_COLOR;
    String(code || "").split(/(<font size="\d+">|<\/font>|~n~|~[^~]+~)/gi).forEach((part) => {
      if (!part) return;
      const m = part.match(/^<font size="(\d+)">$/i);
      if (m) {
        size = parseInt(m[1]) || DEFAULT_SIZE;
        return;
      }
      if (/^<\/font>$/i.test(part)) {
        size = DEFAULT_SIZE;
        color = DEFAULT_COLOR;
        return;
      }
      if (/^~n~$/i.test(part)) {
        html += "<br>";
        return;
      }
      if (/^~[^~]+~$/.test(part)) {
        if (CODE_TO_HEX[part]) color = CODE_TO_HEX[part];
        return;
      }
      html += '<span style="font-size:' + size + "px;color:" + color + '">' + esc(part) + "</span>";
    });
    return html;
  }
  function presetCodeToHtml(code) {
    if (!code) return "";
    let html = "";
    const lines = code.replace(/<\/?font[^>]*>/gi, "").split(/~n~/gi);
    lines.forEach((line, i) => {
      let current = DEFAULT_COLOR;
      for (const part of line.split(/(~[^~]+~)/)) {
        if (!part) continue;
        const isCode = /^~[^~]+~$/.test(part);
        if (isCode && CODE_TO_HEX[part]) current = CODE_TO_HEX[part];
        else if (!isCode) html += '<span style="color:' + current + '">' + esc(part) + "</span>";
      }
      if (i < lines.length - 1) html += "<br>";
    });
    return html;
  }
  const plainText = (code) => String(code || "").replace(/<[^>]+>/g, "").replace(/~[^~]+~/g, "");
  const _hoisted_1$6 = { class: "sc-swatches" };
  const _hoisted_2$4 = ["data-paint", "data-color", "title", "onMousedown"];
  const _sfc_main$6 = {
    __name: "Swatches",
    props: {
      items: { type: Array, required: true },
      active: { default: null }
    },
    emits: ["pick"],
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("div", _hoisted_1$6, [
          (vue.openBlock(true), vue.createElementBlock(
            vue.Fragment,
            null,
            vue.renderList(__props.items, (it) => {
              return vue.openBlock(), vue.createElementBlock("span", {
                key: String(it.key),
                class: vue.normalizeClass(["sc-swatch", { "is-active": it.key === __props.active, "is-none": it.key === null }]),
                "data-paint": it.key,
                "data-color": it.hex,
                title: it.title,
                style: vue.normalizeStyle(it.key === null ? null : { background: it.hex }),
                onMousedown: vue.withModifiers(($event) => _ctx.$emit("pick", it), ["prevent"])
              }, null, 46, _hoisted_2$4);
            }),
            128
            /* KEYED_FRAGMENT */
          ))
        ]);
      };
    }
  };
  const _hoisted_1$5 = ["data-key"];
  const _hoisted_2$3 = { class: "sc-slider-label" };
  const _hoisted_3$3 = ["min", "max", "step", "value"];
  const _hoisted_4$3 = ["step", "value"];
  const _sfc_main$5 = {
    __name: "SliderRow",
    props: {
      label: { type: String, default: "" },
      min: { type: Number, required: true },
      max: { type: Number, required: true },
      step: { type: Number, required: true },
      modelValue: { type: Number, default: 0 },
      keyName: { type: String, default: "" }
    },
    emits: ["update:modelValue"],
    setup(__props, { emit: __emit }) {
      const props = __props;
      const emit = __emit;
      function set(v) {
        if (isNaN(v)) return;
        v = Math.min(props.max, Math.max(props.min, Math.round(v / props.step) * props.step));
        emit("update:modelValue", parseFloat(v.toFixed(2)));
      }
      function wheel(e) {
        set(props.modelValue + (e.deltaY < 0 ? props.step : -props.step) * (e.shiftKey ? 10 : 1));
      }
      function enter(e) {
        set(parseFloat(e.target.value));
        e.target.blur();
      }
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("div", {
          class: "sc-slider",
          "data-key": __props.keyName,
          onWheel: vue.withModifiers(wheel, ["prevent", "stop"])
        }, [
          vue.createElementVNode(
            "span",
            _hoisted_2$3,
            vue.toDisplayString(__props.label),
            1
            /* TEXT */
          ),
          vue.createElementVNode("input", {
            type: "range",
            class: "sc-range",
            min: __props.min,
            max: __props.max,
            step: __props.step,
            value: __props.modelValue,
            onInput: _cache[0] || (_cache[0] = ($event) => set(parseFloat($event.target.value)))
          }, null, 40, _hoisted_3$3),
          vue.createElementVNode("input", {
            type: "number",
            class: "pg-input sc-slider-num pg-num",
            step: __props.step,
            value: __props.modelValue,
            onChange: _cache[1] || (_cache[1] = ($event) => set(parseFloat($event.target.value))),
            onKeydown: vue.withKeys(enter, ["enter"])
          }, null, 40, _hoisted_4$3)
        ], 40, _hoisted_1$5);
      };
    }
  };
  const _hoisted_1$4 = {
    id: "scene-text-mode",
    class: "pg-stack"
  };
  const _hoisted_2$2 = { class: "pg-line sc-toolbar" };
  const _hoisted_3$2 = {
    id: "scene-sign-mode",
    class: "pg-stack"
  };
  const _hoisted_4$2 = { class: "sc-sec-body" };
  const _hoisted_5$1 = { class: "sc-options" };
  const _hoisted_6$1 = { class: "sc-option" };
  const _hoisted_7$1 = { class: "sc-option" };
  const _hoisted_8$1 = {
    id: "sign-tone",
    class: "sc-seg"
  };
  const _hoisted_9 = { class: "sc-options" };
  const _hoisted_10 = { class: "sc-option" };
  const _hoisted_11 = { class: "sc-option" };
  const _hoisted_12 = { class: "sc-options" };
  const _hoisted_13 = { class: "sc-option" };
  const _hoisted_14 = { class: "sc-option" };
  const _hoisted_15 = {
    id: "sign-sides",
    class: "sc-seg",
    title: "Both: the back says the same as the front"
  };
  const _hoisted_16 = { class: "sc-options" };
  const _hoisted_17 = { class: "sc-option" };
  const _hoisted_18 = {
    id: "sign-font",
    class: "sc-seg"
  };
  const _hoisted_19 = { class: "sc-option" };
  const _hoisted_20 = {
    id: "sign-dir",
    class: "sc-seg"
  };
  const _hoisted_21 = {
    id: "sign-toolbar",
    class: "pg-line sc-toolbar"
  };
  const _hoisted_22 = { class: "sc-sec-body" };
  const _hoisted_23 = { id: "sign-symbols" };
  const _hoisted_24 = ["title", "onClick"];
  const _hoisted_25 = ["data-sym"];
  const _hoisted_26 = {
    id: "sign-words",
    class: "sc-sec-body"
  };
  const _hoisted_27 = {
    id: "sign-sliders",
    class: "sc-sec-body"
  };
  const _hoisted_28 = { class: "pg-actions sc-actions" };
  const _sfc_main$4 = {
    __name: "SceneEditor",
    emits: ["hover"],
    setup(__props, { expose: __expose, emit: __emit }) {
      const emit = __emit;
      const textInput = vue.ref(null);
      const signInput = vue.ref(null);
      const editing = vue.ref(false);
      const confirmDisabled = vue.ref(false);
      let savedEditorHtml = "";
      const MODES = [{ value: "text", label: "Floating text" }, { value: "sign", label: "Sign board" }];
      const confirmLabel = vue.computed(() => ui.sceneMode === "sign" ? editing.value ? "✓ Save" : "✓ Place" : "Confirm →");
      const FONT_SIZES = [10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 36, 40, 48, 60, 80, 100];
      const editorSize = vue.ref(DEFAULT_SIZE);
      const editorColor = vue.ref(DEFAULT_COLOR);
      function editorSetDefaults() {
        editorColor.value = DEFAULT_COLOR;
        editorSize.value = DEFAULT_SIZE;
      }
      function pickColour(it) {
        editorColor.value = it.hex;
        document.execCommand("foreColor", false, it.hex);
      }
      function pickFontSize(v) {
        editorSize.value = v;
        textInput.value.focus();
        const sel = window.getSelection();
        if (!sel || sel.rangeCount === 0) return;
        if (sel.isCollapsed) {
          const span = document.createElement("span");
          span.style.fontSize = v + "px";
          span.style.color = editorColor.value;
          span.textContent = "​";
          const range = sel.getRangeAt(0);
          range.insertNode(span);
          range.setStart(span.firstChild, 1);
          range.setEnd(span.firstChild, 1);
          sel.removeAllRanges();
          sel.addRange(range);
          return;
        }
        document.execCommand("fontSize", false, "7");
        textInput.value.querySelectorAll('font[size="7"]').forEach((el) => {
          const span = document.createElement("span");
          span.style.fontSize = v + "px";
          const colour = el.getAttribute("color");
          if (colour) span.style.color = colour;
          span.innerHTML = el.innerHTML;
          el.parentNode.replaceChild(span, el);
        });
      }
      const SIGN_PX = { 1: 30, 2: 20, 3: 13 };
      const SIZES = [{ value: 1, label: "Large" }, { value: 2, label: "Medium" }, { value: 3, label: "Small" }];
      const signData = vue.ref(null);
      const sign = vue.reactive({
        board: 1,
        tone: "l",
        font: "r",
        vertical: false,
        border: null,
        mount: null,
        light: null,
        gap: 3.5,
        dz: 0,
        double: false
      });
      const signSize = vue.ref(1);
      const signPaint = vue.ref(null);
      const status = vue.reactive({ text: "", bad: false, over: false });
      const slide = vue.reactive({ dx: 0, dy: 0, dz: 0, rx: 0, ry: 0, rz: 0 });
      const open = vue.reactive({ board: true, symbols: false, spacing: false, place: true });
      const signDefaultPaint = () => sign.tone === "l" ? "d" : "l";
      const signFontOf = (key) => signData.value.fonts.find((f) => f.key === key);
      const signPaintHex = (key) => (signData.value.paints.find((p) => p.key === key) || signData.value.paints[0]).hex;
      const boardOptions = vue.computed(() => signData.value ? signData.value.boards.map((b) => ({ value: b.id, label: b.label })) : []);
      const none = [{ value: null, label: "None" }];
      const mountOptions = vue.computed(() => signData.value ? none.concat(signData.value.mounts.map((o) => ({ value: o.key, label: o.label }))) : none);
      const lightOptions = vue.computed(() => signData.value ? none.concat(signData.value.lights.map((o) => ({ value: o.key, label: o.label }))) : none);
      const paintItems = vue.computed(() => signData.value ? signData.value.paints.map((p) => ({ key: p.key, hex: p.hex, title: p.label })) : []);
      const borderItems = vue.computed(() => [{ key: null, hex: "", title: "None" }].concat(paintItems.value));
      const activePaint = vue.computed(() => signPaint.value || signDefaultPaint());
      const boardFace = vue.computed(() => {
        const t = signData.value && signData.value.tones.find((x) => x.key === sign.tone);
        return t && t.hex ? t.hex : null;
      });
      let signDoc = [[]];
      const signEmpty = vue.ref(true);
      let signUndo = [];
      let signLast = { from: { l: 0, i: 0 }, to: { l: 0, i: 0 } };
      const signItemPaint = (it) => it.p || signDefaultPaint();
      const signSame = (a, b) => a.l === b.l && a.i === b.i;
      function signSymbolEl(y) {
        const el = document.createElement("span");
        el.className = "sym" + (y.flourish ? " sym-flourish" : "");
        el.dataset.sym = y.key;
        el.contentEditable = "false";
        el.style.setProperty("--w", y.w);
        el.style.webkitMaskImage = "url(sym_" + y.key + ".png)";
        return el;
      }
      function signRender() {
        const box = signInput.value;
        box.innerHTML = "";
        signDoc.forEach((line) => {
          const row = document.createElement("div");
          row.className = "sl";
          line.forEach((it) => {
            const el = document.createElement("span");
            el.className = "si";
            el.style.fontSize = SIGN_PX[it.s] + "px";
            el.style.color = signPaintHex(signItemPaint(it));
            if (it.y) {
              el.classList.add("si-sym");
              el.contentEditable = "false";
              el.appendChild(signSymbolEl(signData.value.symbols.find((y) => y.key === it.y)));
            } else {
              el.textContent = it.c === " " ? " " : it.c;
            }
            row.appendChild(el);
          });
          if (!line.length) row.appendChild(document.createElement("br"));
          box.appendChild(row);
        });
        signEmpty.value = signDoc.length === 1 && !signDoc[0].length;
      }
      function signPoint(node, offset) {
        const box = signInput.value;
        if (!node || !box.contains(node)) return null;
        const last = signDoc.length - 1;
        if (node === box) return offset > last ? { l: last, i: signDoc[last].length } : { l: offset, i: 0 };
        const el = node.nodeType === Node.TEXT_NODE ? node.parentNode : node;
        const row = el.closest(".sl");
        if (!row) return null;
        const l = Array.prototype.indexOf.call(box.children, row);
        if (l < 0 || l > last) return null;
        const item = el.closest(".si");
        if (item) {
          const i = Array.prototype.indexOf.call(row.children, item);
          return { l, i: Math.min(i + (offset > 0 ? 1 : 0), signDoc[l].length) };
        }
        return { l, i: Math.min(offset, signDoc[l].length) };
      }
      function signSel() {
        const sel = window.getSelection();
        if (!sel || !sel.rangeCount) return null;
        const a = signPoint(sel.anchorNode, sel.anchorOffset), b = signPoint(sel.focusNode, sel.focusOffset);
        if (!a || !b) return null;
        return a.l < b.l || a.l === b.l && a.i <= b.i ? { from: a, to: b } : { from: b, to: a };
      }
      function signSetSel(from, to) {
        const box = signInput.value;
        const place = (p) => {
          const l = Math.min(p.l, signDoc.length - 1);
          return [box.children[l], Math.min(p.i, signDoc[l].length)];
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
      function signShowSelection(s) {
        Array.prototype.forEach.call(signInput.value.children, (row, l) => {
          Array.prototype.forEach.call(row.children, (el, i) => {
            const inside = (l > s.from.l || l === s.from.l && i >= s.from.i) && (l < s.to.l || l === s.to.l && i < s.to.i);
            if (el.classList.contains("si")) el.classList.toggle("sel", inside);
          });
        });
      }
      function signToolbarFollow(s) {
        const line = signDoc[s.from.l];
        const it = signSame(s.from, s.to) ? line[s.from.i - 1] || line[s.from.i] : line[s.from.i] || signDoc[s.to.l][s.to.i - 1];
        if (!it) return;
        signSize.value = it.s;
        signPaint.value = it.p;
      }
      function onSelectionChange() {
        if (ui.phase !== "text" || ui.sceneMode !== "sign" || !signData.value) return;
        const s = signSel();
        if (!s) return;
        signLast = s;
        signShowSelection(s);
        signToolbarFollow(s);
      }
      vue.onMounted(() => document.addEventListener("selectionchange", onSelectionChange));
      vue.onBeforeUnmount(() => document.removeEventListener("selectionchange", onSelectionChange));
      function signSnapshot() {
        signUndo.push(JSON.stringify(signDoc));
        if (signUndo.length > 80) signUndo.shift();
      }
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
      function signType(at, text) {
        const widths = signData.value.fonts[0].width;
        for (const raw of String(text || "").replace(/\r/g, "")) {
          if (raw === "\n") {
            at = signBreak(at);
            continue;
          }
          const ch = raw === " " ? " " : raw.toUpperCase();
          if (ch !== " " && widths[ch] === void 0) continue;
          signDoc[at.l].splice(at.i, 0, { c: ch, s: signSize.value, p: signPaint.value });
          at = { l: at.l, i: at.i + 1 };
        }
        return at;
      }
      function signCommit(from, to) {
        signRender();
        signInput.value.focus();
        signSetSel(from, to);
        signRefresh();
      }
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
      function onSignBeforeInput(e) {
        e.preventDefault();
        const s = signSel() || signLast, t = e.inputType || "";
        if (t === "historyUndo") {
          signUndoLast();
          return;
        }
        let caret;
        if (t.startsWith("insert")) {
          signSnapshot();
          caret = signDelete(s);
          if (t === "insertParagraph" || t === "insertLineBreak") {
            caret = signBreak(caret);
          } else {
            const text = e.data != null ? e.data : e.dataTransfer ? e.dataTransfer.getData("text/plain") : "";
            caret = signType(caret, text);
          }
        } else if (t.startsWith("delete")) {
          signSnapshot();
          if (!signSame(s.from, s.to)) {
            caret = signDelete(s);
          } else if (t.indexOf("Forward") >= 0) {
            const line = signDoc[s.from.l];
            if (s.from.i < line.length) caret = signDelete({ from: s.from, to: { l: s.from.l, i: s.from.i + 1 } });
            else if (s.from.l < signDoc.length - 1) caret = signDelete({ from: s.from, to: { l: s.from.l + 1, i: 0 } });
            else caret = s.from;
          } else if (s.from.i > 0) {
            caret = signDelete({ from: { l: s.from.l, i: s.from.i - 1 }, to: s.from });
          } else if (s.from.l > 0) {
            caret = signDelete({ from: { l: s.from.l - 1, i: signDoc[s.from.l - 1].length }, to: s.from });
          } else {
            caret = s.from;
          }
        } else {
          return;
        }
        signCommit(caret);
      }
      function onSignMouseDown(e) {
        const item = e.target.closest ? e.target.closest(".si-sym") : null;
        if (!item) return;
        e.preventDefault();
        const row = item.parentNode, box = signInput.value;
        const l = Array.prototype.indexOf.call(box.children, row), i = Array.prototype.indexOf.call(row.children, item);
        box.focus();
        signSetSel({ l, i }, { l, i: i + 1 });
        signToolbarFollow(signLast);
      }
      function onSignInput() {
        signRender();
        signSetSel(signLast.from, signLast.to);
      }
      function onSignKeyDown(e) {
        if ((e.ctrlKey || e.metaKey) && (e.key === "z" || e.key === "Z")) {
          e.preventDefault();
          signUndoLast();
        }
      }
      function pickSignSize(v) {
        signSize.value = v;
        signApply((it) => {
          it.s = v;
        });
      }
      function pickSignPaint(it) {
        signPaint.value = it.key;
        signApply((x) => {
          x.p = it.key;
        });
      }
      function pickSymbol(y) {
        signSnapshot();
        const at = signDelete(signSel() || signLast);
        signDoc[at.l].splice(at.i, 0, { y: y.key, s: signSize.value, p: signPaint.value });
        signCommit({ l: at.l, i: at.i + 1 });
      }
      function signCollect() {
        return signDoc.map((line) => {
          const runs = [];
          line.forEach((it) => {
            const p = signItemPaint(it);
            if (it.y) {
              runs.push({ s: it.s, p, y: it.y });
              return;
            }
            const last = runs[runs.length - 1];
            if (last && last.t !== void 0 && last.s === it.s && last.p === p) last.t += it.c;
            else runs.push({ s: it.s, p, t: it.c });
          });
          return runs;
        });
      }
      function signMeasure() {
        const data = signData.value;
        const f = signFontOf(sign.font), b = data.boards[sign.board - 1], lay = data.layout;
        const res = { ok: false, why: "", lines: [], letters: 0, note: "" };
        const allowed = (ch) => ch === " " || f.width[ch] !== void 0;
        signCollect().forEach((raw) => {
          const runs = [];
          raw.forEach((r) => {
            if (r.y !== void 0) {
              runs.push({ s: r.s, p: r.p, y: r.y });
              return;
            }
            let text = "";
            for (const ch of r.t.toUpperCase()) if (allowed(ch)) text += ch;
            if (!runs.length) text = text.replace(/^ +/, "");
            if (!text) return;
            const last = runs[runs.length - 1];
            if (last && last.t !== void 0 && last.s === r.s && last.p === r.p) last.t += text;
            else runs.push({ s: r.s, p: r.p, t: text });
          });
          while (runs.length && runs[runs.length - 1].t !== void 0) {
            const t = runs[runs.length - 1].t.replace(/ +$/, "");
            if (!t) runs.pop();
            else {
              runs[runs.length - 1].t = t;
              break;
            }
          }
          if (runs.length) res.lines.push(runs);
        });
        if (!res.lines.length) {
          res.why = "Type the words for the sign.";
          return res;
        }
        if (res.lines.length > lay.maxLines) {
          res.why = "Too many lines: the most is " + lay.maxLines + ".";
          return res;
        }
        const rows = res.lines.map((runs) => {
          const items = [];
          runs.forEach((run) => {
            if (run.y !== void 0) {
              const sym = data.symbols.find((y) => y.key === run.y);
              const h = (sym.flourish ? data.symbolSizes.flourish : data.symbolSizes.emblem)[run.s - 1];
              items.push({ w: sym.w * h, h, pad: lay.symbolPad * h, symbol: true });
            } else {
              const rc = f.sizes[run.s - 1];
              for (const ch of run.t) {
                if (ch === " ") items.push({ w: f.space * rc, h: rc, space: true });
                else items.push({ w: f.width[ch] * rc, h: rc });
              }
            }
          });
          return items;
        });
        const m = (v) => v.toFixed(2) + " m";
        const gap = sign.gap / 100;
        let wide = 0, tall = 0;
        if (!sign.vertical) {
          rows.forEach((items, i) => {
            let height = 0, width = 0;
            items.forEach((it, k) => {
              if (!it.space) {
                height = Math.max(height, it.h);
                res.letters++;
              }
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
            items.forEach((it) => {
              if (it.space) {
                height += lay.stackSpace * it.h;
                return;
              }
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
        res.note = "Width " + m(wide) + " of " + m(b.w) + "  ·  height " + m(tall) + " of " + m(b.h);
        res.over = wide > b.w + 5e-4 || tall > b.h + 5e-4;
        if (res.letters > data.maxLetters) {
          res.why = "Too many letters and symbols: " + res.letters + " of " + data.maxLetters + ".";
          return res;
        }
        res.ok = true;
        return res;
      }
      function signRefresh() {
        if (!signData.value) return null;
        const res = signMeasure();
        const empty = !res.lines.length;
        status.bad = !res.ok && !empty;
        status.over = res.ok && res.over;
        status.text = res.ok ? res.note + "  ·  " + res.letters + " of " + signData.value.maxLetters + " letters" + (res.over ? "  ·  runs past the board" : "") : res.why;
        if (ui.sceneMode === "sign") {
          confirmDisabled.value = !res.ok;
          signLive(res);
        }
        return res;
      }
      function signOf(res) {
        const out = { board: sign.board, tone: sign.tone, font: sign.font, vertical: sign.vertical, lines: res.lines };
        if (sign.border) out.border = sign.border;
        if (sign.mount) out.mount = sign.mount;
        if (sign.light) out.light = sign.light;
        out.gap = sign.gap / 100;
        out.dz = sign.dz / 100;
        if (sign.double) out.double = true;
        return out;
      }
      let signLiveTimer = null;
      function signLive(res) {
        clearTimeout(signLiveTimer);
        if (!res.ok) return;
        signLiveTimer = setTimeout(() => ui$1.post("sign_live", { sign: signOf(res), slide: { ...slide } }), 140);
      }
      let slideTimer = null;
      function setSlide(key, v) {
        slide[key] = v;
        clearTimeout(slideTimer);
        slideTimer = setTimeout(() => ui$1.post("sign_live", { slide: { ...slide } }), 30);
      }
      function setWords(key, v) {
        sign[key] = Math.round(v * 10) / 10;
        signRefresh();
      }
      function slideReset() {
        Object.keys(slide).forEach((k) => {
          slide[k] = 0;
        });
      }
      function setSign(key, value) {
        sign[key] = value;
        if (key === "tone" || key === "font" || key === "vertical") signRender();
        signRefresh();
      }
      const signBoxClass = vue.computed(() => ["tone-" + sign.tone, "sign-font-" + sign.font, { "sign-down": sign.vertical, empty: signEmpty.value }]);
      const signBoxStyle = vue.computed(() => signData.value ? {
        background: boardFace.value || void 0,
        color: signPaintHex(signDefaultPaint())
      } : null);
      function setSceneMode(mode) {
        const was = ui.sceneMode === "sign";
        ui.sceneMode = mode === "sign" && signData.value ? "sign" : "text";
        confirmDisabled.value = false;
        if (ui.sceneMode === "sign") {
          signRefresh();
          setTimeout(() => signInput.value.focus(), 60);
        } else {
          clearTimeout(signLiveTimer);
          if (was) ui$1.post("sign_live_stop");
          setTimeout(() => textInput.value.focus(), 60);
        }
      }
      function signReset(data) {
        signData.value = data && data.available ? data : null;
        if (signData.value) {
          Object.assign(sign, {
            board: 1,
            tone: "l",
            font: data.fonts[0].key,
            vertical: false,
            border: null,
            mount: null,
            light: null,
            gap: Math.round(data.layout.lineGap * 1e3) / 10,
            dz: 0,
            double: false
          });
          signSize.value = 1;
          signPaint.value = null;
          slideReset();
          signDoc = [[]];
          signUndo = [];
          signLast = { from: { l: 0, i: 0 }, to: { l: 0, i: 0 } };
          signRender();
        }
        setSceneMode("text");
      }
      function signLoad(edit) {
        const s = edit.sign;
        Object.assign(sign, {
          board: s.board,
          tone: s.tone,
          font: s.font,
          vertical: s.vertical === true,
          border: s.border || null,
          mount: s.mount || null,
          light: s.light || null,
          gap: Math.round((s.gap !== void 0 ? s.gap : signData.value.layout.lineGap) * 1e3) / 10,
          dz: Math.round((s.dz || 0) * 1e3) / 10,
          double: s.double === true
        });
        signDoc = (s.lines || []).map((runs) => {
          const line = [];
          (runs || []).forEach((run) => {
            if (run.y) line.push({ y: run.y, s: run.s, p: run.p });
            else for (const ch of String(run.t || "")) line.push({ c: ch, s: run.s, p: run.p });
          });
          return line;
        });
        if (!signDoc.length) signDoc = [[]];
        slideReset();
        slide.rx = Math.round((edit.rx || 0) * 2) / 2;
        slide.ry = Math.round((edit.ry || 0) * 2) / 2;
        ui.sceneMode = "sign";
        signRender();
        const last = signDoc.length - 1;
        signLast = { from: { l: last, i: signDoc[last].length }, to: { l: last, i: signDoc[last].length } };
      }
      const removeArmed = vue.ref(false);
      let removeTimer = null;
      function removeReset() {
        clearTimeout(removeTimer);
        removeArmed.value = false;
      }
      function remove() {
        if (!removeArmed.value) {
          removeArmed.value = true;
          removeTimer = setTimeout(removeReset, 3500);
          return;
        }
        removeReset();
        ui.phase = null;
        ui$1.post("scene_remove");
      }
      function show() {
        ui.phase = "text";
        editorSetDefaults();
        setTimeout(() => textInput.value && textInput.value.focus(), 60);
      }
      ui$1.onMessage("open_scene_text", (msg) => {
        textInput.value.innerHTML = "";
        show();
        signReset(msg.signs);
        editing.value = !!msg.edit;
        if (msg.edit && msg.edit.sign && signData.value) signLoad(msg.edit);
        else if (msg.edit && msg.edit.text) textInput.value.innerHTML = sceneCodeToHtml(msg.edit.text);
        setSceneMode(ui.sceneMode);
        removeReset();
      });
      function cancel() {
        ui.phase = null;
        ui$1.post("scene_text_cancel");
      }
      function confirm() {
        if (ui.sceneMode === "sign") {
          const res = signRefresh();
          if (!res || !res.ok) return;
          ui.phase = null;
          ui$1.post("sign_place", { sign: signOf(res), slide: { ...slide } });
          return;
        }
        const box = textInput.value;
        if (!box.innerHTML.trim() || box.textContent.trim() === "") return;
        savedEditorHtml = box.innerHTML;
        ui$1.post("scene_text_confirm", { text: generateSceneCode(box) });
      }
      function reopen() {
        textInput.value.innerHTML = savedEditorHtml || "";
        show();
        setSceneMode("text");
      }
      function keepFocus(e) {
        if (e.target.closest && e.target.closest(".pg-group-head")) e.preventDefault();
      }
      __expose({ cancel, reopen });
      return (_ctx, _cache) => {
        const _component_PgTabs = vue.resolveComponent("PgTabs");
        const _component_PgSwitcher = vue.resolveComponent("PgSwitcher");
        const _component_PgButton = vue.resolveComponent("PgButton");
        const _component_PgGroup = vue.resolveComponent("PgGroup");
        const _component_PgWindow = vue.resolveComponent("PgWindow");
        return vue.withDirectives((vue.openBlock(), vue.createBlock(
          _component_PgWindow,
          {
            id: "scene-text-panel",
            class: "sc-panel",
            title: "Scene Editor",
            width: "480px",
            onClose: cancel,
            onMouseenter: _cache[21] || (_cache[21] = ($event) => emit("hover", true)),
            onMouseleave: _cache[22] || (_cache[22] = ($event) => emit("hover", false))
          },
          {
            default: vue.withCtx(() => [
              vue.createElementVNode(
                "div",
                {
                  class: "sc-body pg-stack",
                  onMousedown: keepFocus
                },
                [
                  signData.value ? (vue.openBlock(), vue.createBlock(_component_PgTabs, {
                    key: 0,
                    id: "scene-mode",
                    "model-value": vue.unref(ui).sceneMode,
                    tabs: MODES,
                    "onUpdate:modelValue": setSceneMode
                  }, null, 8, ["model-value"])) : vue.createCommentVNode("v-if", true),
                  vue.createCommentVNode(" floating text "),
                  vue.withDirectives(vue.createElementVNode(
                    "div",
                    _hoisted_1$4,
                    [
                      vue.createElementVNode("div", _hoisted_2$2, [
                        vue.createVNode(_component_PgSwitcher, {
                          id: "editor-font-size",
                          class: "sc-switch-small",
                          "model-value": editorSize.value,
                          options: FONT_SIZES,
                          "onUpdate:modelValue": pickFontSize
                        }, null, 8, ["model-value"]),
                        vue.createVNode(_sfc_main$6, {
                          id: "editor-colors",
                          items: vue.unref(colours).order,
                          active: editorColor.value,
                          onPick: pickColour
                        }, null, 8, ["items", "active"])
                      ]),
                      vue.createElementVNode(
                        "div",
                        {
                          id: "scene-text-input",
                          ref_key: "textInput",
                          ref: textInput,
                          class: "sc-words sc-scene-words",
                          contenteditable: "true",
                          spellcheck: "false",
                          "data-placeholder": "Type your scene text…"
                        },
                        null,
                        512
                        /* NEED_PATCH */
                      )
                    ],
                    512
                    /* NEED_PATCH */
                  ), [
                    [vue.vShow, vue.unref(ui).sceneMode === "text"]
                  ]),
                  vue.createCommentVNode(" a sign board "),
                  vue.withDirectives(vue.createElementVNode(
                    "div",
                    _hoisted_3$2,
                    [
                      signData.value ? (vue.openBlock(), vue.createElementBlock(
                        vue.Fragment,
                        { key: 0 },
                        [
                          vue.createVNode(_component_PgGroup, {
                            open: open.board,
                            "onUpdate:open": _cache[6] || (_cache[6] = ($event) => open.board = $event),
                            class: "sc-sec",
                            "data-sec": "board",
                            title: "Board, mount and light"
                          }, {
                            default: vue.withCtx(() => [
                              vue.createElementVNode("div", _hoisted_4$2, [
                                vue.createElementVNode("div", _hoisted_5$1, [
                                  vue.createElementVNode("div", _hoisted_6$1, [
                                    _cache[23] || (_cache[23] = vue.createElementVNode(
                                      "span",
                                      { class: "pg-label" },
                                      "Board",
                                      -1
                                      /* HOISTED */
                                    )),
                                    vue.createVNode(_component_PgSwitcher, {
                                      id: "sign-board",
                                      "model-value": sign.board,
                                      options: boardOptions.value,
                                      "onUpdate:modelValue": _cache[0] || (_cache[0] = (v) => setSign("board", v))
                                    }, null, 8, ["model-value", "options"])
                                  ]),
                                  vue.createElementVNode("div", _hoisted_7$1, [
                                    _cache[24] || (_cache[24] = vue.createElementVNode(
                                      "span",
                                      { class: "pg-label" },
                                      "Board colour",
                                      -1
                                      /* HOISTED */
                                    )),
                                    vue.createElementVNode("div", _hoisted_8$1, [
                                      (vue.openBlock(true), vue.createElementBlock(
                                        vue.Fragment,
                                        null,
                                        vue.renderList(signData.value.tones, (t) => {
                                          return vue.openBlock(), vue.createBlock(_component_PgButton, {
                                            key: t.key,
                                            "data-tone": t.key,
                                            variant: sign.tone === t.key ? "primary" : "",
                                            onClick: ($event) => setSign("tone", t.key)
                                          }, {
                                            default: vue.withCtx(() => [
                                              vue.createTextVNode(
                                                vue.toDisplayString(t.label),
                                                1
                                                /* TEXT */
                                              )
                                            ]),
                                            _: 2
                                            /* DYNAMIC */
                                          }, 1032, ["data-tone", "variant", "onClick"]);
                                        }),
                                        128
                                        /* KEYED_FRAGMENT */
                                      ))
                                    ])
                                  ])
                                ]),
                                vue.createElementVNode("div", _hoisted_9, [
                                  vue.createElementVNode("div", _hoisted_10, [
                                    _cache[25] || (_cache[25] = vue.createElementVNode(
                                      "span",
                                      { class: "pg-label" },
                                      "Mount",
                                      -1
                                      /* HOISTED */
                                    )),
                                    vue.createVNode(_component_PgSwitcher, {
                                      id: "sign-mount",
                                      "model-value": sign.mount,
                                      options: mountOptions.value,
                                      "onUpdate:modelValue": _cache[1] || (_cache[1] = (v) => setSign("mount", v))
                                    }, null, 8, ["model-value", "options"])
                                  ]),
                                  vue.createElementVNode("div", _hoisted_11, [
                                    _cache[26] || (_cache[26] = vue.createElementVNode(
                                      "span",
                                      { class: "pg-label" },
                                      "Light",
                                      -1
                                      /* HOISTED */
                                    )),
                                    vue.createVNode(_component_PgSwitcher, {
                                      id: "sign-light",
                                      "model-value": sign.light,
                                      options: lightOptions.value,
                                      "onUpdate:modelValue": _cache[2] || (_cache[2] = (v) => setSign("light", v))
                                    }, null, 8, ["model-value", "options"])
                                  ])
                                ]),
                                vue.createElementVNode("div", _hoisted_12, [
                                  vue.createElementVNode("div", _hoisted_13, [
                                    _cache[27] || (_cache[27] = vue.createElementVNode(
                                      "span",
                                      { class: "pg-label" },
                                      "Painted border",
                                      -1
                                      /* HOISTED */
                                    )),
                                    vue.createVNode(_sfc_main$6, {
                                      id: "sign-border",
                                      items: borderItems.value,
                                      active: sign.border,
                                      onPick: _cache[3] || (_cache[3] = (it) => setSign("border", it.key))
                                    }, null, 8, ["items", "active"])
                                  ]),
                                  vue.createElementVNode("div", _hoisted_14, [
                                    _cache[30] || (_cache[30] = vue.createElementVNode(
                                      "span",
                                      { class: "pg-label" },
                                      "Sides",
                                      -1
                                      /* HOISTED */
                                    )),
                                    vue.createElementVNode("div", _hoisted_15, [
                                      vue.createVNode(_component_PgButton, {
                                        "data-sides": "1",
                                        variant: sign.double ? "" : "primary",
                                        onClick: _cache[4] || (_cache[4] = ($event) => setSign("double", false))
                                      }, {
                                        default: vue.withCtx(() => _cache[28] || (_cache[28] = [
                                          vue.createTextVNode("One")
                                        ])),
                                        _: 1
                                        /* STABLE */
                                      }, 8, ["variant"]),
                                      vue.createVNode(_component_PgButton, {
                                        "data-sides": "2",
                                        variant: sign.double ? "primary" : "",
                                        onClick: _cache[5] || (_cache[5] = ($event) => setSign("double", true))
                                      }, {
                                        default: vue.withCtx(() => _cache[29] || (_cache[29] = [
                                          vue.createTextVNode("Both")
                                        ])),
                                        _: 1
                                        /* STABLE */
                                      }, 8, ["variant"])
                                    ])
                                  ])
                                ])
                              ])
                            ]),
                            _: 1
                            /* STABLE */
                          }, 8, ["open"]),
                          vue.createElementVNode("div", _hoisted_16, [
                            vue.createElementVNode("div", _hoisted_17, [
                              _cache[31] || (_cache[31] = vue.createElementVNode(
                                "span",
                                { class: "pg-label" },
                                "Lettering",
                                -1
                                /* HOISTED */
                              )),
                              vue.createElementVNode("div", _hoisted_18, [
                                (vue.openBlock(true), vue.createElementBlock(
                                  vue.Fragment,
                                  null,
                                  vue.renderList(signData.value.fonts, (f) => {
                                    return vue.openBlock(), vue.createBlock(_component_PgButton, {
                                      key: f.key,
                                      "data-font": f.key,
                                      variant: sign.font === f.key ? "primary" : "",
                                      onClick: ($event) => setSign("font", f.key)
                                    }, {
                                      default: vue.withCtx(() => [
                                        vue.createTextVNode(
                                          vue.toDisplayString(f.label),
                                          1
                                          /* TEXT */
                                        )
                                      ]),
                                      _: 2
                                      /* DYNAMIC */
                                    }, 1032, ["data-font", "variant", "onClick"]);
                                  }),
                                  128
                                  /* KEYED_FRAGMENT */
                                ))
                              ])
                            ]),
                            vue.createElementVNode("div", _hoisted_19, [
                              _cache[34] || (_cache[34] = vue.createElementVNode(
                                "span",
                                { class: "pg-label" },
                                "Words run",
                                -1
                                /* HOISTED */
                              )),
                              vue.createElementVNode("div", _hoisted_20, [
                                vue.createVNode(_component_PgButton, {
                                  "data-dir": "h",
                                  variant: sign.vertical ? "" : "primary",
                                  onClick: _cache[7] || (_cache[7] = ($event) => setSign("vertical", false))
                                }, {
                                  default: vue.withCtx(() => _cache[32] || (_cache[32] = [
                                    vue.createTextVNode("Across")
                                  ])),
                                  _: 1
                                  /* STABLE */
                                }, 8, ["variant"]),
                                vue.createVNode(_component_PgButton, {
                                  "data-dir": "v",
                                  variant: sign.vertical ? "primary" : "",
                                  onClick: _cache[8] || (_cache[8] = ($event) => setSign("vertical", true))
                                }, {
                                  default: vue.withCtx(() => _cache[33] || (_cache[33] = [
                                    vue.createTextVNode("Down")
                                  ])),
                                  _: 1
                                  /* STABLE */
                                }, 8, ["variant"])
                              ])
                            ])
                          ]),
                          vue.createElementVNode("div", _hoisted_21, [
                            vue.createVNode(_component_PgSwitcher, {
                              id: "sign-size",
                              class: "sc-switch-small",
                              "model-value": signSize.value,
                              options: SIZES,
                              "onUpdate:modelValue": pickSignSize
                            }, null, 8, ["model-value"]),
                            vue.createVNode(_sfc_main$6, {
                              id: "sign-paints",
                              items: paintItems.value,
                              active: activePaint.value,
                              onPick: pickSignPaint
                            }, null, 8, ["items", "active"])
                          ])
                        ],
                        64
                        /* STABLE_FRAGMENT */
                      )) : vue.createCommentVNode("v-if", true),
                      vue.createElementVNode(
                        "div",
                        {
                          id: "sign-text-input",
                          ref_key: "signInput",
                          ref: signInput,
                          class: vue.normalizeClass(["sc-words sc-sign-words", signBoxClass.value]),
                          style: vue.normalizeStyle(signBoxStyle.value),
                          contenteditable: "true",
                          spellcheck: "false",
                          "data-placeholder": "Type the sign's words. Enter starts a new line.",
                          onBeforeinput: onSignBeforeInput,
                          onMousedown: onSignMouseDown,
                          onInput: onSignInput,
                          onKeydown: onSignKeyDown
                        },
                        null,
                        38
                        /* CLASS, STYLE, NEED_HYDRATION */
                      ),
                      vue.createElementVNode(
                        "div",
                        {
                          id: "sign-status",
                          class: vue.normalizeClass(["sc-hint", { "is-bad": status.bad, "is-over": status.over }])
                        },
                        vue.toDisplayString(status.text),
                        3
                        /* TEXT, CLASS */
                      ),
                      signData.value ? (vue.openBlock(), vue.createElementBlock(
                        vue.Fragment,
                        { key: 1 },
                        [
                          vue.createVNode(_component_PgGroup, {
                            open: open.symbols,
                            "onUpdate:open": _cache[10] || (_cache[10] = ($event) => open.symbols = $event),
                            class: "sc-sec",
                            "data-sec": "symbols",
                            title: "Symbols"
                          }, {
                            default: vue.withCtx(() => [
                              vue.createElementVNode("div", _hoisted_22, [
                                _cache[35] || (_cache[35] = vue.createElementVNode(
                                  "div",
                                  { class: "sc-hint sc-hint-left" },
                                  "Click a symbol to write it in where you are typing. It takes the size and paint in use.",
                                  -1
                                  /* HOISTED */
                                )),
                                vue.createElementVNode("div", _hoisted_23, [
                                  (vue.openBlock(true), vue.createElementBlock(
                                    vue.Fragment,
                                    null,
                                    vue.renderList(signData.value.symbols, (y) => {
                                      return vue.openBlock(), vue.createElementBlock("button", {
                                        key: y.key,
                                        type: "button",
                                        class: vue.normalizeClass(["sc-sym-btn", { "is-wide": y.flourish }]),
                                        title: y.label,
                                        onMousedown: _cache[9] || (_cache[9] = vue.withModifiers(() => {
                                        }, ["prevent"])),
                                        onClick: ($event) => pickSymbol(y)
                                      }, [
                                        vue.createElementVNode("span", {
                                          class: vue.normalizeClass(["sym", { "sym-flourish": y.flourish }]),
                                          "data-sym": y.key,
                                          style: vue.normalizeStyle({ "--w": y.w, webkitMaskImage: "url(sym_" + y.key + ".png)" })
                                        }, null, 14, _hoisted_25)
                                      ], 42, _hoisted_24);
                                    }),
                                    128
                                    /* KEYED_FRAGMENT */
                                  ))
                                ])
                              ])
                            ]),
                            _: 1
                            /* STABLE */
                          }, 8, ["open"]),
                          vue.createVNode(_component_PgGroup, {
                            open: open.spacing,
                            "onUpdate:open": _cache[13] || (_cache[13] = ($event) => open.spacing = $event),
                            class: "sc-sec",
                            "data-sec": "spacing",
                            title: "Line spacing and height"
                          }, {
                            default: vue.withCtx(() => [
                              vue.createElementVNode("div", _hoisted_26, [
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "gap",
                                  label: "Line spacing",
                                  min: -6,
                                  max: 30,
                                  step: 0.5,
                                  "model-value": sign.gap,
                                  "onUpdate:modelValue": _cache[11] || (_cache[11] = (v) => setWords("gap", v))
                                }, null, 8, ["model-value"]),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "dz",
                                  label: "Words up / down",
                                  min: -60,
                                  max: 60,
                                  step: 0.5,
                                  "model-value": sign.dz,
                                  "onUpdate:modelValue": _cache[12] || (_cache[12] = (v) => setWords("dz", v))
                                }, null, 8, ["model-value"])
                              ])
                            ]),
                            _: 1
                            /* STABLE */
                          }, 8, ["open"]),
                          vue.createVNode(_component_PgGroup, {
                            open: open.place,
                            "onUpdate:open": _cache[20] || (_cache[20] = ($event) => open.place = $event),
                            class: "sc-sec",
                            "data-sec": "place",
                            title: "Position and rotation"
                          }, {
                            default: vue.withCtx(() => [
                              vue.createElementVNode("div", _hoisted_27, [
                                _cache[36] || (_cache[36] = vue.createElementVNode(
                                  "span",
                                  { class: "pg-label" },
                                  "Position",
                                  -1
                                  /* HOISTED */
                                )),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "dx",
                                  label: "Left / right",
                                  min: -4,
                                  max: 4,
                                  step: 0.01,
                                  "model-value": slide.dx,
                                  "onUpdate:modelValue": _cache[14] || (_cache[14] = (v) => setSlide("dx", v))
                                }, null, 8, ["model-value"]),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "dy",
                                  label: "Near / far",
                                  min: -4,
                                  max: 4,
                                  step: 0.01,
                                  "model-value": slide.dy,
                                  "onUpdate:modelValue": _cache[15] || (_cache[15] = (v) => setSlide("dy", v))
                                }, null, 8, ["model-value"]),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "dz",
                                  label: "Down / up",
                                  min: -3,
                                  max: 5,
                                  step: 0.01,
                                  "model-value": slide.dz,
                                  "onUpdate:modelValue": _cache[16] || (_cache[16] = (v) => setSlide("dz", v))
                                }, null, 8, ["model-value"]),
                                _cache[37] || (_cache[37] = vue.createElementVNode(
                                  "span",
                                  { class: "pg-label" },
                                  "Rotation",
                                  -1
                                  /* HOISTED */
                                )),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "rz",
                                  label: "Turn",
                                  min: -180,
                                  max: 180,
                                  step: 0.5,
                                  "model-value": slide.rz,
                                  "onUpdate:modelValue": _cache[17] || (_cache[17] = (v) => setSlide("rz", v))
                                }, null, 8, ["model-value"]),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "rx",
                                  label: "Tilt",
                                  min: -180,
                                  max: 180,
                                  step: 0.5,
                                  "model-value": slide.rx,
                                  "onUpdate:modelValue": _cache[18] || (_cache[18] = (v) => setSlide("rx", v))
                                }, null, 8, ["model-value"]),
                                vue.createVNode(_sfc_main$5, {
                                  "key-name": "ry",
                                  label: "Lean",
                                  min: -180,
                                  max: 180,
                                  step: 0.5,
                                  "model-value": slide.ry,
                                  "onUpdate:modelValue": _cache[19] || (_cache[19] = (v) => setSlide("ry", v))
                                }, null, 8, ["model-value"])
                              ])
                            ]),
                            _: 1
                            /* STABLE */
                          }, 8, ["open"])
                        ],
                        64
                        /* STABLE_FRAGMENT */
                      )) : vue.createCommentVNode("v-if", true)
                    ],
                    512
                    /* NEED_PATCH */
                  ), [
                    [vue.vShow, vue.unref(ui).sceneMode === "sign"]
                  ]),
                  vue.createElementVNode("div", _hoisted_28, [
                    editing.value ? (vue.openBlock(), vue.createBlock(_component_PgButton, {
                      key: 0,
                      id: "btn-text-remove",
                      variant: removeArmed.value ? "danger-fill" : "danger",
                      onClick: remove
                    }, {
                      default: vue.withCtx(() => [
                        vue.createTextVNode(
                          vue.toDisplayString(removeArmed.value ? "Click again to remove" : "Remove"),
                          1
                          /* TEXT */
                        )
                      ]),
                      _: 1
                      /* STABLE */
                    }, 8, ["variant"])) : vue.createCommentVNode("v-if", true),
                    vue.createVNode(_component_PgButton, {
                      id: "btn-text-cancel",
                      onClick: cancel
                    }, {
                      default: vue.withCtx(() => _cache[38] || (_cache[38] = [
                        vue.createTextVNode("✕ Cancel")
                      ])),
                      _: 1
                      /* STABLE */
                    }),
                    vue.createVNode(_component_PgButton, {
                      id: "btn-text-confirm",
                      variant: "primary",
                      disabled: confirmDisabled.value,
                      onClick: confirm
                    }, {
                      default: vue.withCtx(() => [
                        vue.createTextVNode(
                          vue.toDisplayString(confirmLabel.value),
                          1
                          /* TEXT */
                        )
                      ]),
                      _: 1
                      /* STABLE */
                    }, 8, ["disabled"])
                  ])
                ],
                32
                /* NEED_HYDRATION */
              )
            ]),
            _: 1
            /* STABLE */
          },
          512
          /* NEED_PATCH */
        )), [
          [vue.vShow, vue.unref(ui).phase === "text"]
        ]);
      };
    }
  };
  const _hoisted_1$3 = { class: "pg-stack" };
  const _hoisted_2$1 = {
    id: "scene-preview-text",
    class: "pg-box sc-preview"
  };
  const _hoisted_3$1 = { class: "pg-label sc-axis-label" };
  const _hoisted_4$1 = ["data-axis", "onWheel"];
  const _hoisted_5 = ["onClick"];
  const _hoisted_6 = ["id", "value", "onInput", "onBlur", "onKeydown"];
  const _hoisted_7 = ["onClick"];
  const _hoisted_8 = { class: "pg-actions" };
  const _sfc_main$3 = {
    __name: "PositionPanel",
    emits: ["hover", "back"],
    setup(__props, { expose: __expose, emit: __emit }) {
      const emit = __emit;
      const pos = vue.reactive({ x: 0, y: 0, z: 0 });
      const shown = vue.reactive({ x: "0.0", y: "0.0", z: "0.0" });
      const preview = vue.ref("");
      const fmt = (v) => parseFloat(v).toFixed(1);
      let timer = null;
      function sendSoon() {
        clearTimeout(timer);
        timer = setTimeout(() => ui$1.post("scene_position_update", { x: pos.x, y: pos.y, z: pos.z }), 40);
      }
      function nudge(axis, d) {
        pos[axis] = parseFloat((pos[axis] + d * 0.1).toFixed(2));
        shown[axis] = fmt(pos[axis]);
        sendSoon();
      }
      function typed(axis, value) {
        shown[axis] = value;
        const v = parseFloat(value);
        if (!isNaN(v)) {
          pos[axis] = v;
          sendSoon();
        }
      }
      ui$1.onMessage("open_scene_position", (msg) => {
        pos.x = parseFloat(msg.x) || 0;
        pos.y = parseFloat(msg.y) || 0;
        pos.z = parseFloat(msg.z) || 0;
        ["x", "y", "z"].forEach((a) => {
          shown[a] = fmt(pos[a]);
        });
        preview.value = plainText(msg.text);
        ui.phase = "position";
        ui$1.post("scene_position_update", { x: pos.x, y: pos.y, z: pos.z });
      });
      function place() {
        ui.phase = null;
        ui$1.post("scene_place_confirm", { x: pos.x, y: pos.y, z: pos.z });
      }
      function cancel() {
        ui$1.post("scene_place_revert");
        emit("back");
      }
      __expose({ cancel });
      return (_ctx, _cache) => {
        const _component_PgButton = vue.resolveComponent("PgButton");
        const _component_PgWindow = vue.resolveComponent("PgWindow");
        return vue.withDirectives((vue.openBlock(), vue.createBlock(
          _component_PgWindow,
          {
            id: "scene-position-panel",
            class: "sc-panel",
            title: "Adjust Position",
            width: "360px",
            onClose: cancel,
            onMouseenter: _cache[0] || (_cache[0] = ($event) => emit("hover", true)),
            onMouseleave: _cache[1] || (_cache[1] = ($event) => emit("hover", false))
          },
          {
            default: vue.withCtx(() => [
              vue.createElementVNode("div", _hoisted_1$3, [
                vue.createElementVNode(
                  "div",
                  _hoisted_2$1,
                  vue.toDisplayString(preview.value),
                  1
                  /* TEXT */
                ),
                (vue.openBlock(), vue.createElementBlock(
                  vue.Fragment,
                  null,
                  vue.renderList(["x", "y", "z"], (axis) => {
                    return vue.createElementVNode("div", {
                      key: axis,
                      class: "pg-line sc-axis"
                    }, [
                      vue.createElementVNode(
                        "span",
                        _hoisted_3$1,
                        vue.toDisplayString(axis.toUpperCase()),
                        1
                        /* TEXT */
                      ),
                      vue.createElementVNode("div", {
                        class: "pg-switch sc-stepper",
                        "data-axis": axis,
                        onWheel: vue.withModifiers(($event) => nudge(axis, $event.deltaY < 0 ? 1 : -1), ["prevent", "stop"])
                      }, [
                        vue.createElementVNode("button", {
                          type: "button",
                          class: "pg-switch-btn step-left",
                          "aria-label": "Less",
                          onClick: ($event) => nudge(axis, -1)
                        }, "◀", 8, _hoisted_5),
                        vue.createElementVNode("input", {
                          id: "input-" + axis,
                          class: "sc-stepper-num pg-num",
                          type: "number",
                          step: "0.1",
                          value: shown[axis],
                          onInput: ($event) => typed(axis, $event.target.value),
                          onBlur: ($event) => shown[axis] = fmt(pos[axis]),
                          onKeydown: vue.withKeys(($event) => {
                            typed(axis, $event.target.value);
                            $event.target.blur();
                          }, ["enter"])
                        }, null, 40, _hoisted_6),
                        vue.createElementVNode("button", {
                          type: "button",
                          class: "pg-switch-btn step-right",
                          "aria-label": "More",
                          onClick: ($event) => nudge(axis, 1)
                        }, "▶", 8, _hoisted_7)
                      ], 40, _hoisted_4$1)
                    ]);
                  }),
                  64
                  /* STABLE_FRAGMENT */
                )),
                _cache[4] || (_cache[4] = vue.createElementVNode(
                  "div",
                  { class: "sc-hint" },
                  "Scroll on a value or use ◄ ► to fine-tune",
                  -1
                  /* HOISTED */
                )),
                vue.createElementVNode("div", _hoisted_8, [
                  vue.createVNode(_component_PgButton, {
                    id: "btn-position-cancel",
                    onClick: cancel
                  }, {
                    default: vue.withCtx(() => _cache[2] || (_cache[2] = [
                      vue.createTextVNode("✕ Cancel")
                    ])),
                    _: 1
                    /* STABLE */
                  }),
                  vue.createVNode(_component_PgButton, {
                    id: "btn-position-place",
                    variant: "primary",
                    onClick: place
                  }, {
                    default: vue.withCtx(() => _cache[3] || (_cache[3] = [
                      vue.createTextVNode("✓ Place")
                    ])),
                    _: 1
                    /* STABLE */
                  })
                ])
              ])
            ]),
            _: 1
            /* STABLE */
          },
          512
          /* NEED_PATCH */
        )), [
          [vue.vShow, vue.unref(ui).phase === "position"]
        ]);
      };
    }
  };
  const _hoisted_1$2 = { class: "pg-stack" };
  const _hoisted_2 = { class: "pg-line" };
  const _hoisted_3 = { class: "pg-actions sc-actions" };
  const _hoisted_4 = {
    id: "status-presets-list",
    class: "pg-list pg-scroll sc-presets"
  };
  const _sfc_main$2 = {
    __name: "StatusEditor",
    emits: ["hover"],
    setup(__props, { expose: __expose, emit: __emit }) {
      const emit = __emit;
      const box = vue.ref(null);
      const name = vue.ref("");
      const colour = vue.ref(DEFAULT_COLOR);
      const presets = vue.ref([]);
      ui$1.onMessage("open_status_ui", () => {
        box.value.innerHTML = "";
        name.value = "";
        colour.value = DEFAULT_COLOR;
        ui.phase = "status";
        setTimeout(() => box.value.focus(), 60);
      });
      ui$1.onMessage("status_presets", (msg) => {
        presets.value = msg.presets || [];
      });
      function pick(it) {
        colour.value = it.hex;
        document.execCommand("foreColor", false, it.hex);
      }
      function words() {
        const el = box.value;
        if (!el.innerHTML.trim() || el.textContent.trim() === "") return null;
        return generateSceneCode(el);
      }
      function apply() {
        const text = words();
        if (!text) return;
        ui$1.post("status_apply", { text });
        ui.phase = null;
      }
      function applyPreset(p) {
        ui$1.post("status_apply", { text: p.text });
        ui.phase = null;
      }
      function clear() {
        ui$1.post("status_clear");
        ui.phase = null;
      }
      function save() {
        const text = words();
        if (!text) return;
        ui$1.post("status_save", { text, name: name.value.trim() });
      }
      function cancel() {
        ui$1.post("status_cancel");
        ui.phase = null;
      }
      __expose({ cancel });
      return (_ctx, _cache) => {
        const _component_PgButton = vue.resolveComponent("PgButton");
        const _component_PgEmpty = vue.resolveComponent("PgEmpty");
        const _component_PgRow = vue.resolveComponent("PgRow");
        const _component_PgWindow = vue.resolveComponent("PgWindow");
        return vue.withDirectives((vue.openBlock(), vue.createBlock(
          _component_PgWindow,
          {
            id: "status-panel",
            class: "sc-panel",
            title: "Set Status",
            width: "480px",
            onClose: cancel,
            onMouseenter: _cache[1] || (_cache[1] = ($event) => emit("hover", true)),
            onMouseleave: _cache[2] || (_cache[2] = ($event) => emit("hover", false))
          },
          {
            default: vue.withCtx(() => [
              vue.createElementVNode("div", _hoisted_1$2, [
                _cache[10] || (_cache[10] = vue.createElementVNode(
                  "span",
                  { class: "pg-label" },
                  "Status text",
                  -1
                  /* HOISTED */
                )),
                vue.createVNode(_sfc_main$6, {
                  id: "status-editor-colors",
                  items: vue.unref(colours).order,
                  active: colour.value,
                  onPick: pick
                }, null, 8, ["items", "active"]),
                vue.createElementVNode(
                  "div",
                  {
                    id: "status-text-input",
                    ref_key: "box",
                    ref: box,
                    class: "sc-words sc-status-words",
                    contenteditable: "true",
                    spellcheck: "false",
                    "data-placeholder": "Enter status text…"
                  },
                  null,
                  512
                  /* NEED_PATCH */
                ),
                vue.createElementVNode("div", _hoisted_2, [
                  vue.withDirectives(vue.createElementVNode(
                    "input",
                    {
                      id: "status-name-input",
                      "onUpdate:modelValue": _cache[0] || (_cache[0] = ($event) => name.value = $event),
                      type: "text",
                      class: "pg-input pg-grow",
                      placeholder: "Preset name (optional)…",
                      autocomplete: "off",
                      spellcheck: "false"
                    },
                    null,
                    512
                    /* NEED_PATCH */
                  ), [
                    [vue.vModelText, name.value]
                  ]),
                  vue.createVNode(_component_PgButton, {
                    id: "btn-status-save",
                    title: "Save preset",
                    onClick: save
                  }, {
                    default: vue.withCtx(() => _cache[3] || (_cache[3] = [
                      vue.createTextVNode("Save preset")
                    ])),
                    _: 1
                    /* STABLE */
                  })
                ]),
                vue.createElementVNode("div", _hoisted_3, [
                  vue.createVNode(_component_PgButton, {
                    id: "btn-status-cancel",
                    onClick: cancel
                  }, {
                    default: vue.withCtx(() => _cache[4] || (_cache[4] = [
                      vue.createTextVNode("✕ Cancel")
                    ])),
                    _: 1
                    /* STABLE */
                  }),
                  vue.createVNode(_component_PgButton, {
                    id: "btn-status-clear",
                    onClick: clear
                  }, {
                    default: vue.withCtx(() => _cache[5] || (_cache[5] = [
                      vue.createTextVNode("■ Clear")
                    ])),
                    _: 1
                    /* STABLE */
                  }),
                  vue.createVNode(_component_PgButton, {
                    id: "btn-status-apply",
                    variant: "primary",
                    onClick: apply
                  }, {
                    default: vue.withCtx(() => _cache[6] || (_cache[6] = [
                      vue.createTextVNode("▶ Apply")
                    ])),
                    _: 1
                    /* STABLE */
                  })
                ]),
                _cache[11] || (_cache[11] = vue.createElementVNode(
                  "hr",
                  { class: "pg-rule" },
                  null,
                  -1
                  /* HOISTED */
                )),
                _cache[12] || (_cache[12] = vue.createElementVNode(
                  "span",
                  { class: "pg-label" },
                  "Saved presets",
                  -1
                  /* HOISTED */
                )),
                vue.createElementVNode("div", _hoisted_4, [
                  !presets.value.length ? (vue.openBlock(), vue.createBlock(_component_PgEmpty, { key: 0 }, {
                    default: vue.withCtx(() => _cache[7] || (_cache[7] = [
                      vue.createTextVNode("No saved presets")
                    ])),
                    _: 1
                    /* STABLE */
                  })) : vue.createCommentVNode("v-if", true),
                  (vue.openBlock(true), vue.createElementBlock(
                    vue.Fragment,
                    null,
                    vue.renderList(presets.value, (p) => {
                      return vue.openBlock(), vue.createBlock(_component_PgRow, {
                        key: p.id,
                        class: "preset-item",
                        clickable: "",
                        title: p.name || "Unnamed",
                        onClick: ($event) => box.value.innerHTML = vue.unref(presetCodeToHtml)(p.text)
                      }, {
                        actions: vue.withCtx(() => [
                          vue.createVNode(_component_PgButton, {
                            class: "preset-btn-apply",
                            variant: "primary",
                            title: "Apply",
                            onClick: vue.withModifiers(($event) => applyPreset(p), ["stop"])
                          }, {
                            default: vue.withCtx(() => _cache[8] || (_cache[8] = [
                              vue.createTextVNode("▶")
                            ])),
                            _: 2
                            /* DYNAMIC */
                          }, 1032, ["onClick"]),
                          vue.createVNode(_component_PgButton, {
                            class: "preset-btn-delete",
                            variant: "danger",
                            title: "Delete",
                            onClick: vue.withModifiers(($event) => vue.unref(ui$1.post)("status_delete", { id: p.id }), ["stop"])
                          }, {
                            default: vue.withCtx(() => _cache[9] || (_cache[9] = [
                              vue.createTextVNode("✕")
                            ])),
                            _: 2
                            /* DYNAMIC */
                          }, 1032, ["onClick"])
                        ]),
                        _: 2
                        /* DYNAMIC */
                      }, 1032, ["title", "onClick"]);
                    }),
                    128
                    /* KEYED_FRAGMENT */
                  ))
                ])
              ])
            ]),
            _: 1
            /* STABLE */
          },
          512
          /* NEED_PATCH */
        )), [
          [vue.vShow, vue.unref(ui).phase === "status"]
        ]);
      };
    }
  };
  const _hoisted_1$1 = { class: "mebox-text" };
  const _sfc_main$1 = {
    __name: "MeBox",
    props: {
      historyMax: { type: Number, default: 50 },
      dissipateMs: { type: Number, default: 3e5 }
    },
    setup(__props, { expose: __expose }) {
      const props = __props;
      const root = vue.ref(null);
      const list = vue.ref(null);
      const lines = vue.ref([]);
      const state = vue.ref("hidden");
      const size = vue.ref("");
      const place = vue.reactive({ top: "", left: "" });
      const moving = vue.ref(false);
      let mode = "auto";
      let fadeTimer = null;
      let dissipateMs = props.dissipateMs;
      let nextId = 1;
      const rootStyle = vue.computed(() => place.top && place.left ? { top: place.top, left: place.left, transform: "none" } : null);
      function toBottom() {
        vue.nextTick(() => {
          if (list.value) list.value.scrollTop = list.value.scrollHeight;
        });
      }
      function show() {
        const wasHidden = state.value !== "shown";
        state.value = "shown";
        if (wasHidden) toBottom();
      }
      let fadeEnd = null;
      function fade() {
        state.value = "fading";
        clearTimeout(fadeEnd);
        fadeEnd = setTimeout(() => {
          if (state.value === "fading") state.value = "hidden";
        }, 1600);
      }
      function hide() {
        state.value = "hidden";
      }
      function clearFade() {
        clearTimeout(fadeTimer);
        fadeTimer = null;
      }
      function fadeLater(timeout) {
        clearFade();
        if (mode === "auto" && timeout > 0) fadeTimer = setTimeout(fade, timeout);
      }
      function realLines() {
        return lines.value.filter((l) => !l.placeholder);
      }
      const sweep = setInterval(() => {
        const now = Date.now();
        lines.value = lines.value.filter((l) => !l.expires || now <= l.expires);
      }, 3e4);
      vue.onBeforeUnmount(() => clearInterval(sweep));
      ui$1.onMessage({
        mebox_scroll: (msg) => {
          if (list.value) list.value.scrollTop += msg.delta || 0;
        },
        mebox_add: (msg) => {
          if (msg.dissipateMs) dissipateMs = msg.dissipateMs;
          lines.value.push({
            id: nextId++,
            name: msg.name,
            color: msg.nameColor || null,
            text: String(msg.text || "").trim(),
            expires: Date.now() + dissipateMs
          });
          while (lines.value.length > props.historyMax) lines.value.shift();
          toBottom();
          show();
          fadeLater(msg.timeout || 6e4);
        },
        mebox_fade: () => fade(),
        mebox_hide: () => {
          clearFade();
          hide();
        },
        mebox_mode: (msg) => {
          mode = msg.mode || "auto";
          if (mode === "persist") {
            clearFade();
            if (lines.value.length) show();
          } else if (mode === "auto") {
            if (lines.value.length) {
              show();
              fadeLater(msg.timeout || 6e4);
            }
          } else if (mode === "off") {
            clearFade();
            hide();
          }
        },
        mebox_move_mode: (msg) => {
          if (!msg.enabled) {
            exitMove(false);
            return;
          }
          moving.value = true;
          state.value = "shown";
          if (!lines.value.length) lines.value.push({ id: nextId++, placeholder: true, text: "Drag to reposition — press ESC to save" });
        },
        mebox_position_load: (msg) => {
          if (msg.top && msg.left) {
            place.top = msg.top;
            place.left = msg.left;
          }
        },
        mebox_size: (msg) => {
          size.value = ["small", "normal", "large"].includes(msg.size) ? msg.size : "";
        },
        mebox_show_if_messages: (msg) => {
          if (mode !== "off" && lines.value.length) {
            show();
            if (mode === "auto") fadeLater(msg.timeout || 6e4);
          }
        },
        mebox_move_cleanup: () => {
          lines.value = realLines();
          if (!lines.value.length && mode !== "persist") hide();
        }
      });
      let drag = null;
      function startDrag(e) {
        if (!moving.value) return;
        const rect = root.value.getBoundingClientRect();
        drag = { x: e.clientX - rect.left, y: e.clientY - rect.top };
        e.preventDefault();
        e.stopPropagation();
      }
      function onMove(e) {
        if (!drag) return;
        const el = root.value;
        const left = Math.max(0, Math.min(window.innerWidth - el.offsetWidth, e.clientX - drag.x));
        const top = Math.max(0, Math.min(window.innerHeight - el.offsetHeight, e.clientY - drag.y));
        place.top = top + "px";
        place.left = left + "px";
      }
      function onUp() {
        drag = null;
      }
      vue.onMounted(() => {
        document.addEventListener("mousemove", onMove);
        document.addEventListener("mouseup", onUp);
      });
      vue.onBeforeUnmount(() => {
        document.removeEventListener("mousemove", onMove);
        document.removeEventListener("mouseup", onUp);
      });
      function exitMove(save) {
        moving.value = false;
        drag = null;
        if (save) ui$1.post("meboxPositionSave", { top: place.top, left: place.left });
        ui$1.post("meboxMoveExit");
      }
      __expose({ moving, exitMove });
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          vue.Fragment,
          null,
          [
            vue.createElementVNode(
              "div",
              {
                id: "mebox-root",
                ref_key: "root",
                ref: root,
                class: vue.normalizeClass(["sc-mebox", ["is-" + state.value, size.value ? "is-" + size.value : "", { "is-moving": moving.value }]]),
                style: vue.normalizeStyle(rootStyle.value),
                onMousedown: startDrag
              },
              [
                vue.createElementVNode(
                  "div",
                  {
                    id: "mebox-container",
                    ref_key: "list",
                    ref: list,
                    class: "pg-panel sc-mebox-list",
                    onWheel: _cache[0] || (_cache[0] = vue.withModifiers(() => {
                    }, ["stop"]))
                  },
                  [
                    (vue.openBlock(true), vue.createElementBlock(
                      vue.Fragment,
                      null,
                      vue.renderList(lines.value, (l) => {
                        return vue.openBlock(), vue.createElementBlock(
                          "div",
                          {
                            key: l.id,
                            class: vue.normalizeClass(["mebox-line", { "mebox-move-placeholder": l.placeholder }])
                          },
                          [
                            l.placeholder ? (vue.openBlock(), vue.createElementBlock(
                              vue.Fragment,
                              { key: 0 },
                              [
                                vue.createTextVNode(
                                  vue.toDisplayString(l.text),
                                  1
                                  /* TEXT */
                                )
                              ],
                              64
                              /* STABLE_FRAGMENT */
                            )) : (vue.openBlock(), vue.createElementBlock(
                              vue.Fragment,
                              { key: 1 },
                              [
                                vue.createElementVNode(
                                  "span",
                                  {
                                    class: "mebox-name",
                                    style: vue.normalizeStyle(l.color ? { color: l.color } : null)
                                  },
                                  vue.toDisplayString(l.name) + ":",
                                  5
                                  /* TEXT, STYLE */
                                ),
                                vue.createElementVNode(
                                  "span",
                                  _hoisted_1$1,
                                  vue.toDisplayString(l.text),
                                  1
                                  /* TEXT */
                                )
                              ],
                              64
                              /* STABLE_FRAGMENT */
                            ))
                          ],
                          2
                          /* CLASS */
                        );
                      }),
                      128
                      /* KEYED_FRAGMENT */
                    ))
                  ],
                  544
                  /* NEED_HYDRATION, NEED_PATCH */
                )
              ],
              38
              /* CLASS, STYLE, NEED_HYDRATION */
            ),
            moving.value ? (vue.openBlock(), vue.createElementBlock("div", {
              key: 0,
              id: "mebox-move-overlay",
              class: "sc-move-overlay",
              onClick: _cache[1] || (_cache[1] = vue.withModifiers(($event) => exitMove(true), ["self"]))
            }, _cache[2] || (_cache[2] = [
              vue.createElementVNode(
                "div",
                { class: "pg-hud sc-move-hint" },
                "Drag the /me box • Press Esc to save and finish",
                -1
                /* HOISTED */
              )
            ]))) : vue.createCommentVNode("v-if", true)
          ],
          64
          /* STABLE_FRAGMENT */
        );
      };
    }
  };
  const _hoisted_1 = { class: "sc-dock" };
  const _sfc_main = {
    __name: "App",
    setup(__props) {
      const editor = vue.ref(null);
      const position = vue.ref(null);
      const statusEd = vue.ref(null);
      const mebox = vue.ref(null);
      const historyMax = vue.ref(50);
      const dissipateMs = vue.ref(3e5);
      ui$1.onMessage({
        load_colors: (msg) => {
          if (msg.historyMax) historyMax.value = msg.historyMax;
          if (msg.dissipateMs) dissipateMs.value = msg.dissipateMs;
          loadColours(msg.colors);
        },
        close_scene_ui: () => {
          ui.phase = null;
        },
        close_status_ui: () => {
          ui.phase = null;
        }
      });
      function hover(on) {
        ui$1.post("scene_hover", { hovered: on });
      }
      function onKey(e) {
        if (e.key !== "Escape") return;
        if (mebox.value && mebox.value.moving) {
          mebox.value.exitMove(true);
          return;
        }
        if (ui.phase === "text") editor.value.cancel();
        else if (ui.phase === "position") position.value.cancel();
        else if (ui.phase === "status") statusEd.value.cancel();
      }
      let cameraHeld = false;
      function onDown(e) {
        const offPanel = !(e.target.closest && e.target.closest(".sc-panel"));
        const looking = offPanel && (ui.phase === "position" || ui.phase === "text" && ui.sceneMode === "sign");
        if (looking && (e.button === 0 || e.button === 2)) {
          cameraHeld = true;
          ui$1.post("scene_camera_hold", { holding: true });
        }
      }
      function onUp(e) {
        if ((e.button === 0 || e.button === 2) && cameraHeld) {
          cameraHeld = false;
          ui$1.post("scene_camera_hold", { holding: false });
        }
      }
      const noMenu = (e) => e.preventDefault();
      vue.onMounted(() => {
        document.addEventListener("keydown", onKey);
        document.addEventListener("mousedown", onDown);
        document.addEventListener("mouseup", onUp);
        document.addEventListener("contextmenu", noMenu);
      });
      vue.onBeforeUnmount(() => {
        document.removeEventListener("keydown", onKey);
        document.removeEventListener("mousedown", onDown);
        document.removeEventListener("mouseup", onUp);
        document.removeEventListener("contextmenu", noMenu);
      });
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          vue.Fragment,
          null,
          [
            vue.createElementVNode("div", _hoisted_1, [
              vue.createVNode(
                _sfc_main$4,
                {
                  ref_key: "editor",
                  ref: editor,
                  onHover: hover
                },
                null,
                512
                /* NEED_PATCH */
              ),
              vue.createVNode(
                _sfc_main$3,
                {
                  ref_key: "position",
                  ref: position,
                  onHover: hover,
                  onBack: _cache[0] || (_cache[0] = ($event) => editor.value.reopen())
                },
                null,
                512
                /* NEED_PATCH */
              ),
              vue.createVNode(
                _sfc_main$2,
                {
                  ref_key: "statusEd",
                  ref: statusEd,
                  onHover: hover
                },
                null,
                512
                /* NEED_PATCH */
              )
            ]),
            vue.createVNode(_sfc_main$1, {
              ref_key: "mebox",
              ref: mebox,
              "history-max": historyMax.value,
              "dissipate-ms": dissipateMs.value
            }, null, 8, ["history-max", "dissipate-ms"])
          ],
          64
          /* STABLE_FRAGMENT */
        );
      };
    }
  };
  ui$1.mount(_sfc_main);
})(PoggyUI, Vue);
