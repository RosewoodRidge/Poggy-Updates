/* Built by tools/ui-vue from poggy_core/ui-src. Edit the source there, not this file. */
var PoggyUI = function(exports, vue) {
  "use strict";
  const _sfc_main$e = {
    __name: "PgScreen",
    props: {
      show: { type: Boolean, default: true },
      veiled: { type: Boolean, default: false }
      // darken the game behind
    },
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createBlock(vue.Transition, { name: "pg-fade" }, {
          default: vue.withCtx(() => [
            __props.show ? (vue.openBlock(), vue.createElementBlock(
              "div",
              {
                key: 0,
                class: vue.normalizeClass(["pg-screen", { "is-veiled": __props.veiled }])
              },
              [
                vue.renderSlot(_ctx.$slots, "default")
              ],
              2
              /* CLASS */
            )) : vue.createCommentVNode("v-if", true)
          ]),
          _: 3
          /* FORWARDED */
        });
      };
    }
  };
  const _hoisted_1$a = { class: "pg-panel-head" };
  const _hoisted_2$5 = { class: "pg-title" };
  const _hoisted_3$3 = { class: "pg-window-head-extra" };
  const _hoisted_4$2 = { class: "pg-window-body" };
  const _sfc_main$d = {
    __name: "PgWindow",
    props: {
      title: { type: String, default: "" },
      width: { type: String, default: "" },
      height: { type: String, default: "" },
      closable: { type: Boolean, default: true }
    },
    emits: ["close"],
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          "section",
          {
            class: "pg-panel pg-window",
            style: vue.normalizeStyle({ width: __props.width, height: __props.height })
          },
          [
            vue.createElementVNode("header", _hoisted_1$a, [
              vue.createElementVNode(
                "h1",
                _hoisted_2$5,
                vue.toDisplayString(__props.title),
                1
                /* TEXT */
              ),
              vue.createElementVNode("div", _hoisted_3$3, [
                vue.renderSlot(_ctx.$slots, "head"),
                __props.closable ? (vue.openBlock(), vue.createElementBlock("button", {
                  key: 0,
                  type: "button",
                  class: "pg-close",
                  "aria-label": "Close",
                  onClick: _cache[0] || (_cache[0] = ($event) => _ctx.$emit("close"))
                }, "×")) : vue.createCommentVNode("v-if", true)
              ])
            ]),
            vue.createElementVNode("div", _hoisted_4$2, [
              vue.renderSlot(_ctx.$slots, "default")
            ])
          ],
          4
          /* STYLE */
        );
      };
    }
  };
  const _hoisted_1$9 = { class: "pg-tabs" };
  const _hoisted_2$4 = ["data-tab", "onClick"];
  const _sfc_main$c = {
    __name: "PgTabs",
    props: {
      modelValue: { type: [String, Number], default: null },
      tabs: { type: Array, required: true }
    },
    emits: ["update:modelValue"],
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("nav", _hoisted_1$9, [
          (vue.openBlock(true), vue.createElementBlock(
            vue.Fragment,
            null,
            vue.renderList(__props.tabs, (t) => {
              return vue.openBlock(), vue.createElementBlock("button", {
                key: t.value,
                type: "button",
                class: vue.normalizeClass(["pg-tab", { "is-active": t.value === __props.modelValue }]),
                "data-tab": t.value,
                onClick: ($event) => _ctx.$emit("update:modelValue", t.value)
              }, vue.toDisplayString(t.label), 11, _hoisted_2$4);
            }),
            128
            /* KEYED_FRAGMENT */
          ))
        ]);
      };
    }
  };
  const _hoisted_1$8 = ["disabled"];
  const _sfc_main$b = {
    __name: "PgButton",
    props: {
      variant: { type: String, default: "" },
      busy: { type: Boolean, default: false },
      busyText: { type: String, default: "" },
      disabled: { type: Boolean, default: false }
    },
    setup(__props) {
      const props = __props;
      const classes = vue.computed(() => ["pg-btn"].concat(
        props.variant.split(/\s+/).filter(Boolean).map((v) => "pg-btn--" + v)
      ));
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("button", {
          type: "button",
          class: vue.normalizeClass(classes.value),
          disabled: __props.disabled || __props.busy
        }, [
          __props.busy ? (vue.openBlock(), vue.createElementBlock(
            vue.Fragment,
            { key: 0 },
            [
              vue.createTextVNode(
                vue.toDisplayString(__props.busyText || "…"),
                1
                /* TEXT */
              )
            ],
            64
            /* STABLE_FRAGMENT */
          )) : vue.renderSlot(_ctx.$slots, "default", { key: 1 })
        ], 10, _hoisted_1$8);
      };
    }
  };
  const _hoisted_1$7 = { class: "pg-field" };
  const _hoisted_2$3 = {
    key: 0,
    class: "pg-label"
  };
  const _hoisted_3$2 = ["type", "value", "placeholder", "disabled", "min", "max"];
  const _sfc_main$a = {
    __name: "PgInput",
    props: {
      modelValue: { type: [String, Number], default: "" },
      label: { type: String, default: "" },
      type: { type: String, default: "text" },
      placeholder: { type: String, default: "" },
      disabled: { type: Boolean, default: false },
      min: { type: [String, Number], default: void 0 },
      max: { type: [String, Number], default: void 0 }
    },
    emits: ["update:modelValue"],
    setup(__props, { emit: __emit }) {
      const emit = __emit;
      function input(e) {
        const v = e.target.value;
        emit("update:modelValue", e.target.type === "number" && v !== "" ? Number(v) : v);
      }
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("label", _hoisted_1$7, [
          __props.label ? (vue.openBlock(), vue.createElementBlock(
            "span",
            _hoisted_2$3,
            vue.toDisplayString(__props.label),
            1
            /* TEXT */
          )) : vue.createCommentVNode("v-if", true),
          vue.createElementVNode("input", {
            class: "pg-input",
            type: __props.type,
            value: __props.modelValue,
            placeholder: __props.placeholder,
            disabled: __props.disabled,
            min: __props.min,
            max: __props.max,
            onInput: input
          }, null, 40, _hoisted_3$2)
        ]);
      };
    }
  };
  const _hoisted_1$6 = { class: "pg-switch-value" };
  const _sfc_main$9 = {
    __name: "PgSwitcher",
    props: {
      modelValue: { type: [String, Number], default: null },
      options: { type: Array, required: true }
      // { value, label } or plain values
    },
    emits: ["update:modelValue"],
    setup(__props, { emit: __emit }) {
      const props = __props;
      const emit = __emit;
      const list = vue.computed(() => props.options.map((o) => o !== null && typeof o === "object" ? { value: o.value, label: o.label != null ? String(o.label) : String(o.value) } : { value: o, label: String(o) }));
      const index = vue.computed(() => Math.max(0, list.value.findIndex((o) => o.value === props.modelValue)));
      const label = vue.computed(() => (list.value[index.value] || { label: "" }).label);
      function step(n) {
        const l = list.value;
        if (!l.length) return;
        emit("update:modelValue", l[(index.value + n + l.length) % l.length].value);
      }
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          "div",
          {
            class: "pg-switch",
            onWheel: _cache[4] || (_cache[4] = vue.withModifiers(($event) => step($event.deltaY > 0 ? 1 : -1), ["prevent"]))
          },
          [
            vue.createElementVNode(
              "button",
              {
                type: "button",
                class: "pg-switch-btn",
                "aria-label": "Previous",
                onMousedown: _cache[0] || (_cache[0] = vue.withModifiers(() => {
                }, ["prevent"])),
                onClick: _cache[1] || (_cache[1] = ($event) => step(-1))
              },
              "◀",
              32
              /* NEED_HYDRATION */
            ),
            vue.createElementVNode(
              "span",
              _hoisted_1$6,
              vue.toDisplayString(label.value),
              1
              /* TEXT */
            ),
            vue.createElementVNode(
              "button",
              {
                type: "button",
                class: "pg-switch-btn",
                "aria-label": "Next",
                onMousedown: _cache[2] || (_cache[2] = vue.withModifiers(() => {
                }, ["prevent"])),
                onClick: _cache[3] || (_cache[3] = ($event) => step(1))
              },
              "▶",
              32
              /* NEED_HYDRATION */
            )
          ],
          32
          /* NEED_HYDRATION */
        );
      };
    }
  };
  const _hoisted_1$5 = { class: "pg-row-main" };
  const _hoisted_2$2 = { class: "pg-row-title" };
  const _hoisted_3$1 = {
    key: 0,
    class: "pg-row-sub"
  };
  const _hoisted_4$1 = {
    key: 0,
    class: "pg-row-actions"
  };
  const _sfc_main$8 = {
    __name: "PgRow",
    props: {
      title: { type: String, default: "" },
      sub: { type: String, default: "" },
      selected: { type: Boolean, default: false },
      current: { type: Boolean, default: false },
      // marked on the left: "this is the active one"
      clickable: { type: Boolean, default: false },
      dim: { type: Boolean, default: false }
    },
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          "div",
          {
            class: vue.normalizeClass(["pg-row", { "is-selected": __props.selected, "is-current": __props.current, "is-clickable": __props.clickable, "is-dim": __props.dim }])
          },
          [
            vue.createElementVNode("div", _hoisted_1$5, [
              vue.renderSlot(_ctx.$slots, "default", {}, () => [
                vue.createElementVNode(
                  "div",
                  _hoisted_2$2,
                  vue.toDisplayString(__props.title),
                  1
                  /* TEXT */
                ),
                __props.sub ? (vue.openBlock(), vue.createElementBlock(
                  "div",
                  _hoisted_3$1,
                  vue.toDisplayString(__props.sub),
                  1
                  /* TEXT */
                )) : vue.createCommentVNode("v-if", true)
              ])
            ]),
            _ctx.$slots.actions ? (vue.openBlock(), vue.createElementBlock("div", _hoisted_4$1, [
              vue.renderSlot(_ctx.$slots, "actions")
            ])) : vue.createCommentVNode("v-if", true)
          ],
          2
          /* CLASS */
        );
      };
    }
  };
  const _hoisted_1$4 = { class: "pg-group" };
  const _hoisted_2$1 = { class: "pg-chevron" };
  const _sfc_main$7 = {
    __name: "PgGroup",
    props: {
      title: { type: String, default: "" },
      open: { type: Boolean, default: false },
      sub: { type: Boolean, default: false }
      // a group inside a group
    },
    emits: ["update:open"],
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("div", _hoisted_1$4, [
          vue.createElementVNode(
            "div",
            {
              class: vue.normalizeClass(["pg-group-head", { "is-sub": __props.sub }]),
              onClick: _cache[0] || (_cache[0] = ($event) => _ctx.$emit("update:open", !__props.open))
            },
            [
              vue.createElementVNode(
                "span",
                null,
                vue.toDisplayString(__props.title),
                1
                /* TEXT */
              ),
              vue.createElementVNode(
                "span",
                _hoisted_2$1,
                vue.toDisplayString(__props.open ? "▲" : "▼"),
                1
                /* TEXT */
              )
            ],
            2
            /* CLASS */
          ),
          __props.open ? (vue.openBlock(), vue.createElementBlock(
            "div",
            {
              key: 0,
              class: vue.normalizeClass(["pg-group-body", { "is-sub": __props.sub }])
            },
            [
              vue.renderSlot(_ctx.$slots, "default")
            ],
            2
            /* CLASS */
          )) : vue.createCommentVNode("v-if", true)
        ]);
      };
    }
  };
  const _hoisted_1$3 = { class: "pg-table-wrap" };
  const _hoisted_2 = { class: "pg-table" };
  const _hoisted_3 = ["data-sort", "onClick"];
  const _hoisted_4 = {
    key: 0,
    class: "pg-sort"
  };
  const _hoisted_5 = { key: 0 };
  const _hoisted_6 = ["colspan"];
  const _sfc_main$6 = {
    __name: "PgTable",
    props: {
      columns: { type: Array, required: true },
      rows: { type: Array, required: true },
      rowKey: { type: Function, default: (r, i) => i },
      sort: { type: Object, default: null },
      empty: { type: String, default: "Nothing to show" }
    },
    emits: ["update:sort"],
    setup(__props, { emit: __emit }) {
      const props = __props;
      const emit = __emit;
      const valueOf = (col, row) => col.value ? col.value(row) : row[col.key];
      const sortOf = (col, row) => col.sortValue ? col.sortValue(row) : valueOf(col, row);
      const sorted = vue.computed(() => {
        const s = props.sort;
        const col = s && props.columns.find((c) => c.key === s.key);
        if (!col) return props.rows;
        const dir = s.dir === "desc" ? -1 : 1;
        return props.rows.slice().sort((a, b) => {
          const x = sortOf(col, a), y = sortOf(col, b);
          if (typeof x === "number" && typeof y === "number") return dir * (x - y);
          return dir * String(x == null ? "" : x).localeCompare(String(y == null ? "" : y));
        });
      });
      function sortBy(col) {
        if (!col.sortable) return;
        const s = props.sort;
        emit("update:sort", s && s.key === col.key ? { key: col.key, dir: s.dir === "asc" ? "desc" : "asc" } : { key: col.key, dir: "asc" });
      }
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("div", _hoisted_1$3, [
          vue.createElementVNode("table", _hoisted_2, [
            vue.createElementVNode("thead", null, [
              vue.createElementVNode("tr", null, [
                (vue.openBlock(true), vue.createElementBlock(
                  vue.Fragment,
                  null,
                  vue.renderList(__props.columns, (c) => {
                    return vue.openBlock(), vue.createElementBlock("th", {
                      key: c.key,
                      "data-sort": c.key,
                      class: vue.normalizeClass({ "is-sortable": c.sortable, "is-sorted": __props.sort && __props.sort.key === c.key, "is-num": c.num }),
                      onClick: ($event) => sortBy(c)
                    }, [
                      vue.createTextVNode(
                        vue.toDisplayString(c.label),
                        1
                        /* TEXT */
                      ),
                      c.sortable ? (vue.openBlock(), vue.createElementBlock(
                        "span",
                        _hoisted_4,
                        vue.toDisplayString(__props.sort && __props.sort.key === c.key ? __props.sort.dir === "desc" ? "▼" : "▲" : "⇅"),
                        1
                        /* TEXT */
                      )) : vue.createCommentVNode("v-if", true)
                    ], 10, _hoisted_3);
                  }),
                  128
                  /* KEYED_FRAGMENT */
                ))
              ])
            ]),
            vue.createElementVNode("tbody", null, [
              (vue.openBlock(true), vue.createElementBlock(
                vue.Fragment,
                null,
                vue.renderList(sorted.value, (row, i) => {
                  return vue.openBlock(), vue.createElementBlock("tr", {
                    key: __props.rowKey(row, i)
                  }, [
                    (vue.openBlock(true), vue.createElementBlock(
                      vue.Fragment,
                      null,
                      vue.renderList(__props.columns, (c) => {
                        return vue.openBlock(), vue.createElementBlock(
                          "td",
                          {
                            key: c.key,
                            class: vue.normalizeClass([c.class, { "is-num": c.num }])
                          },
                          [
                            vue.renderSlot(_ctx.$slots, "cell-" + c.key, {
                              row,
                              value: valueOf(c, row)
                            }, () => [
                              vue.createTextVNode(
                                vue.toDisplayString(valueOf(c, row)),
                                1
                                /* TEXT */
                              )
                            ])
                          ],
                          2
                          /* CLASS */
                        );
                      }),
                      128
                      /* KEYED_FRAGMENT */
                    ))
                  ]);
                }),
                128
                /* KEYED_FRAGMENT */
              )),
              !sorted.value.length ? (vue.openBlock(), vue.createElementBlock("tr", _hoisted_5, [
                vue.createElementVNode("td", {
                  colspan: __props.columns.length,
                  class: "pg-empty"
                }, vue.toDisplayString(__props.empty), 9, _hoisted_6)
              ])) : vue.createCommentVNode("v-if", true)
            ])
          ])
        ]);
      };
    }
  };
  const _sfc_main$5 = {
    __name: "PgBar",
    props: {
      value: { type: Number, default: 0 },
      variant: { type: String, default: "" }
      // danger
    },
    setup(__props, { expose: __expose }) {
      const props = __props;
      const width = vue.ref(props.value);
      const ms = vue.ref(0);
      vue.watch(() => props.value, (v) => {
        ms.value = 0;
        width.value = v;
      });
      let run_id = 0;
      async function run(duration) {
        const id = ++run_id;
        ms.value = 0;
        width.value = 0;
        await vue.nextTick();
        await new Promise((r) => requestAnimationFrame(() => requestAnimationFrame(r)));
        if (id !== run_id) return;
        ms.value = duration;
        width.value = 100;
      }
      function reset() {
        run_id++;
        ms.value = 0;
        width.value = 0;
      }
      __expose({ run, reset });
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          "div",
          {
            class: vue.normalizeClass(["pg-bar", [__props.variant && "pg-bar--" + __props.variant, { "is-running": ms.value > 0 }]])
          },
          [
            vue.createElementVNode(
              "i",
              {
                style: vue.normalizeStyle({ width: width.value + "%", transitionDuration: ms.value + "ms" })
              },
              null,
              4
              /* STYLE */
            )
          ],
          2
          /* CLASS */
        );
      };
    }
  };
  const _hoisted_1$2 = ["src", "alt"];
  const _sfc_main$4 = {
    __name: "PgPicture",
    props: {
      src: { type: String, required: true },
      height: { type: String, default: "250px" },
      alt: { type: String, default: "" }
    },
    emits: ["open"],
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          "div",
          {
            class: "pg-picture",
            style: vue.normalizeStyle({ height: __props.height }),
            onClick: _cache[0] || (_cache[0] = ($event) => _ctx.$emit("open"))
          },
          [
            vue.createElementVNode("img", {
              src: __props.src,
              alt: __props.alt
            }, null, 8, _hoisted_1$2)
          ],
          4
          /* STYLE */
        );
      };
    }
  };
  const _hoisted_1$1 = ["src"];
  const SIZE = 150;
  const _sfc_main$3 = {
    __name: "PgViewer",
    props: {
      src: { type: String, required: true },
      zoom: { type: Number, default: 2 },
      lens: { type: Boolean, default: true }
    },
    emits: ["close"],
    setup(__props) {
      const props = __props;
      const at = vue.reactive({ show: false, x: 0, y: 0, w: 0, h: 0 });
      function move(e) {
        const r = e.currentTarget.getBoundingClientRect();
        const x = e.clientX - r.left, y = e.clientY - r.top;
        Object.assign(at, { x, y, w: r.width, h: r.height });
        at.show = props.lens && x >= 0 && y >= 0 && x <= r.width && y <= r.height;
      }
      const lensStyle = vue.computed(() => ({
        left: at.x + "px",
        top: at.y + "px",
        backgroundImage: `url(${props.src})`,
        backgroundSize: `${at.w * props.zoom}px ${at.h * props.zoom}px`,
        backgroundPosition: `-${at.x * props.zoom - SIZE / 2}px -${at.y * props.zoom - SIZE / 2}px`
      }));
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock("div", {
          class: "pg-veil",
          onClick: _cache[2] || (_cache[2] = vue.withModifiers(($event) => _ctx.$emit("close"), ["self"]))
        }, [
          vue.createElementVNode(
            "div",
            {
              class: "pg-viewer",
              onMousemove: move,
              onMouseleave: _cache[1] || (_cache[1] = ($event) => at.show = false)
            },
            [
              vue.createElementVNode("button", {
                type: "button",
                class: "pg-close",
                "aria-label": "Close",
                onClick: _cache[0] || (_cache[0] = ($event) => _ctx.$emit("close"))
              }, "×"),
              vue.createElementVNode("img", {
                src: __props.src,
                alt: ""
              }, null, 8, _hoisted_1$1),
              at.show ? (vue.openBlock(), vue.createElementBlock(
                "div",
                {
                  key: 0,
                  class: "pg-lens",
                  style: vue.normalizeStyle(lensStyle.value)
                },
                null,
                4
                /* STYLE */
              )) : vue.createCommentVNode("v-if", true)
            ],
            32
            /* NEED_HYDRATION */
          )
        ]);
      };
    }
  };
  const _sfc_main$2 = {
    __name: "PgHud",
    props: {
      place: { type: String, default: "bottom" },
      show: { type: Boolean, default: true }
    },
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createBlock(vue.Transition, {
          name: "pg-fade",
          persisted: ""
        }, {
          default: vue.withCtx(() => [
            vue.withDirectives(vue.createElementVNode(
              "div",
              {
                class: vue.normalizeClass(["pg-hud pg-hud-placed", "is-" + __props.place])
              },
              [
                vue.renderSlot(_ctx.$slots, "default")
              ],
              2
              /* CLASS */
            ), [
              [vue.vShow, __props.show]
            ])
          ]),
          _: 3
          /* FORWARDED */
        });
      };
    }
  };
  const _export_sfc = (sfc, props) => {
    const target = sfc.__vccOpts || sfc;
    for (const [key, val] of props) {
      target[key] = val;
    }
    return target;
  };
  const _sfc_main$1 = {};
  const _hoisted_1 = { class: "pg-empty" };
  function _sfc_render(_ctx, _cache) {
    return vue.openBlock(), vue.createElementBlock("div", _hoisted_1, [
      vue.renderSlot(_ctx.$slots, "default")
    ]);
  }
  const PgEmpty = /* @__PURE__ */ _export_sfc(_sfc_main$1, [["render", _sfc_render]]);
  const _sfc_main = {
    __name: "PgPill",
    props: { variant: { type: String, default: "" } },
    setup(__props) {
      return (_ctx, _cache) => {
        return vue.openBlock(), vue.createElementBlock(
          "span",
          {
            class: vue.normalizeClass(["pg-pill", __props.variant && "pg-pill--" + __props.variant])
          },
          [
            vue.renderSlot(_ctx.$slots, "default")
          ],
          2
          /* CLASS */
        );
      };
    }
  };
  function resource() {
    return typeof GetParentResourceName === "function" ? GetParentResourceName() : "unknown";
  }
  function post(name, data) {
    return fetch(`https://${resource()}/${name}`, {
      method: "POST",
      headers: { "Content-Type": "application/json; charset=UTF-8" },
      body: JSON.stringify(data === void 0 ? {} : data)
    }).then((r) => r.text()).then((t) => {
      try {
        return t ? JSON.parse(t) : null;
      } catch (e) {
        return t;
      }
    }).catch(() => null);
  }
  function onMessage(a, b) {
    let fn;
    if (typeof a === "function") fn = a;
    else if (typeof a === "string") fn = (d) => {
      if (d && d.type === a) b(d);
    };
    else {
      const key = typeof b === "string" ? b : "type";
      fn = (d) => {
        const h = d && a[d[key]];
        if (h) h(d);
      };
    }
    const listener = (e) => fn(e.data);
    vue.onMounted(() => window.addEventListener("message", listener));
    vue.onBeforeUnmount(() => window.removeEventListener("message", listener));
  }
  function onKey(key, fn, type = "keydown") {
    const listener = (e) => {
      if (e.key === key) fn(e);
    };
    vue.onMounted(() => document.addEventListener(type, listener));
    vue.onBeforeUnmount(() => document.removeEventListener(type, listener));
  }
  const version = "0.28.0";
  const components = {
    PgScreen: _sfc_main$e,
    PgWindow: _sfc_main$d,
    PgTabs: _sfc_main$c,
    PgButton: _sfc_main$b,
    PgInput: _sfc_main$a,
    PgSwitcher: _sfc_main$9,
    PgRow: _sfc_main$8,
    PgGroup: _sfc_main$7,
    PgTable: _sfc_main$6,
    PgBar: _sfc_main$5,
    PgPicture: _sfc_main$4,
    PgViewer: _sfc_main$3,
    PgHud: _sfc_main$2,
    PgEmpty,
    PgPill: _sfc_main
  };
  function mount(App, el = "#app") {
    const app = vue.createApp(App);
    for (const [name, c] of Object.entries(components)) app.component(name, c);
    app.mount(el);
    return app;
  }
  exports.PgBar = _sfc_main$5;
  exports.PgButton = _sfc_main$b;
  exports.PgEmpty = PgEmpty;
  exports.PgGroup = _sfc_main$7;
  exports.PgHud = _sfc_main$2;
  exports.PgInput = _sfc_main$a;
  exports.PgPicture = _sfc_main$4;
  exports.PgPill = _sfc_main;
  exports.PgRow = _sfc_main$8;
  exports.PgScreen = _sfc_main$e;
  exports.PgSwitcher = _sfc_main$9;
  exports.PgTable = _sfc_main$6;
  exports.PgTabs = _sfc_main$c;
  exports.PgViewer = _sfc_main$3;
  exports.PgWindow = _sfc_main$d;
  exports.components = components;
  exports.mount = mount;
  exports.onKey = onKey;
  exports.onMessage = onMessage;
  exports.post = post;
  exports.resource = resource;
  exports.version = version;
  Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
  return exports;
}({}, Vue);
