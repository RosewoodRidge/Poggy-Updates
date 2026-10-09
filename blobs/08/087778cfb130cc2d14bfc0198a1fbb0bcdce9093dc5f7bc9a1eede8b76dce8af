/* Built by tools/ui-vue from redm/poggy_supplydrops/ui-src. Edit the source there, not this file. */
(function(ui, vue) {
  "use strict";
  const _hoisted_1 = { class: "pg-stack" };
  const _hoisted_2 = { class: "pg-text" };
  const _hoisted_3 = {
    key: 0,
    class: "sd-reveal"
  };
  const _hoisted_4 = ["innerHTML"];
  const _hoisted_5 = { class: "pg-box sd-reward" };
  const _hoisted_6 = {
    id: "reward-value",
    class: "pg-price sd-amount"
  };
  const _sfc_main = {
    __name: "App",
    setup(__props) {
      const open = vue.ref(false);
      const big = vue.ref(false);
      const revealed = vue.ref(false);
      const hunt = vue.ref({});
      const penalty = vue.computed(() => Number(hunt.value.cluePenalty) || 0);
      const image = vue.computed(() => "images/" + (hunt.value.imagePath || "bardscrossingbridge.jpg"));
      const penaltyText = vue.computed(() => hunt.value.penaltyText || `Warning: Revealing the clue will reduce your reward by ${Math.round(penalty.value * 100)}%`);
      function money(amount, currency) {
        return currency === "gold" ? `${amount.toFixed(2)} Gold` : `$${amount.toFixed(2)}`;
      }
      const reward = vue.computed(() => {
        const r = hunt.value.reward;
        if (!r) return "";
        if (!revealed.value || penalty.value <= 0) return r.formatted;
        if (r.type === "money") return money(Math.max(0, parseFloat((r.originalAmount * (1 - penalty.value)).toFixed(2))), r.currencyType);
        return `${Math.max(0, Math.floor(r.originalAmount * (1 - penalty.value)))}x ${r.label}`;
      });
      function close() {
        open.value = false;
        ui.post("closeUI");
      }
      function reveal() {
        if (revealed.value) return;
        revealed.value = true;
        ui.post("revealClue");
      }
      function showBig(on) {
        big.value = on;
        ui.post("toggleImageFocus", { isModalOpen: on });
      }
      ui.onMessage({
        showHunt(d) {
          hunt.value = d;
          revealed.value = !!d.clueRevealed;
          open.value = true;
        },
        hideHunt() {
          open.value = false;
          big.value = false;
          revealed.value = false;
        }
      });
      ui.onKey("Escape", () => {
        if (big.value) showBig(false);
        else if (open.value) close();
      }, "keyup");
      return (_ctx, _cache) => {
        const _component_PgPicture = vue.resolveComponent("PgPicture");
        const _component_PgButton = vue.resolveComponent("PgButton");
        const _component_PgWindow = vue.resolveComponent("PgWindow");
        const _component_PgScreen = vue.resolveComponent("PgScreen");
        const _component_PgViewer = vue.resolveComponent("PgViewer");
        return vue.openBlock(), vue.createElementBlock(
          vue.Fragment,
          null,
          [
            vue.createVNode(_component_PgScreen, {
              show: open.value,
              veiled: ""
            }, {
              default: vue.withCtx(() => [
                vue.createVNode(_component_PgWindow, {
                  id: "hunt",
                  class: "sd-card",
                  title: hunt.value.huntName || "Scavenger Hunt",
                  width: "420px",
                  onClose: close
                }, {
                  default: vue.withCtx(() => [
                    vue.createElementVNode("div", _hoisted_1, [
                      vue.createVNode(_component_PgPicture, {
                        id: "hunt-image",
                        src: image.value,
                        alt: "Where the hunt is",
                        onOpen: _cache[0] || (_cache[0] = ($event) => showBig(true))
                      }, null, 8, ["src"]),
                      vue.createElementVNode(
                        "div",
                        {
                          id: "clue",
                          class: vue.normalizeClass(["pg-box sd-clue", { "is-hidden": !revealed.value }])
                        },
                        [
                          vue.createElementVNode(
                            "p",
                            _hoisted_2,
                            vue.toDisplayString(hunt.value.clue || "No clue available..."),
                            1
                            /* TEXT */
                          )
                        ],
                        2
                        /* CLASS */
                      ),
                      !revealed.value ? (vue.openBlock(), vue.createElementBlock("div", _hoisted_3, [
                        vue.createCommentVNode(" the owner's text from config, which may carry simple formatting "),
                        vue.createElementVNode("p", {
                          id: "penalty-text",
                          class: "pg-text pg-muted",
                          innerHTML: penaltyText.value
                        }, null, 8, _hoisted_4),
                        vue.createVNode(_component_PgButton, {
                          id: "reveal-clue",
                          variant: "primary",
                          onClick: reveal
                        }, {
                          default: vue.withCtx(() => [
                            vue.createTextVNode(
                              vue.toDisplayString(hunt.value.showClueText || "SHOW CLUE"),
                              1
                              /* TEXT */
                            )
                          ]),
                          _: 1
                          /* STABLE */
                        })
                      ])) : vue.createCommentVNode("v-if", true),
                      vue.createElementVNode("div", _hoisted_5, [
                        _cache[2] || (_cache[2] = vue.createElementVNode(
                          "div",
                          { class: "pg-label" },
                          "Reward",
                          -1
                          /* HOISTED */
                        )),
                        vue.createElementVNode(
                          "div",
                          _hoisted_6,
                          vue.toDisplayString(reward.value),
                          1
                          /* TEXT */
                        )
                      ])
                    ])
                  ]),
                  _: 1
                  /* STABLE */
                }, 8, ["title"])
              ]),
              _: 1
              /* STABLE */
            }, 8, ["show"]),
            open.value && big.value ? (vue.openBlock(), vue.createBlock(_component_PgViewer, {
              key: 0,
              id: "viewer",
              src: image.value,
              onClose: _cache[1] || (_cache[1] = ($event) => showBig(false))
            }, null, 8, ["src"])) : vue.createCommentVNode("v-if", true)
          ],
          64
          /* STABLE_FRAGMENT */
        );
      };
    }
  };
  ui.mount(_sfc_main);
})(PoggyUI, Vue);
