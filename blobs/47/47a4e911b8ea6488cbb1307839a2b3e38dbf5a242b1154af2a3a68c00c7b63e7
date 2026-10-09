/* Built by tools/ui-vue from redm/poggy_trashbins/ui-src. Edit the source there, not this file. */
(function(ui, vue) {
  "use strict";
  const _hoisted_1 = {
    id: "label",
    class: "pg-label tb-label"
  };
  const _sfc_main = {
    __name: "App",
    setup(__props) {
      const visible = vue.ref(false);
      const text = vue.ref("Searching...");
      const bar = vue.ref(null);
      let hideTimer = null;
      function hide() {
        clearTimeout(hideTimer);
        visible.value = false;
      }
      ui.onMessage({
        async showProgress(d) {
          clearTimeout(hideTimer);
          const ms = Number(d.duration) || 5e3;
          text.value = d.label || "Searching...";
          visible.value = true;
          await vue.nextTick();
          bar.value.run(ms);
          hideTimer = setTimeout(hide, ms + 250);
        },
        hideProgress: hide
      });
      return (_ctx, _cache) => {
        const _component_PgBar = vue.resolveComponent("PgBar");
        const _component_PgHud = vue.resolveComponent("PgHud");
        return vue.openBlock(), vue.createBlock(_component_PgHud, {
          id: "progress",
          class: "tb-progress",
          place: "bottom",
          show: visible.value
        }, {
          default: vue.withCtx(() => [
            vue.createElementVNode(
              "div",
              _hoisted_1,
              vue.toDisplayString(text.value),
              1
              /* TEXT */
            ),
            vue.createVNode(
              _component_PgBar,
              {
                ref_key: "bar",
                ref: bar
              },
              null,
              512
              /* NEED_PATCH */
            )
          ]),
          _: 1
          /* STABLE */
        }, 8, ["show"]);
      };
    }
  };
  ui.mount(_sfc_main);
})(PoggyUI, Vue);
