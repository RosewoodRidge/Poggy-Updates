/* Built by tools/ui-vue from redm/poggy_multijob/ui-src. Edit the source there, not this file. */
(function(ui, vue) {
  "use strict";
  const _hoisted_1 = { class: "pg-split" };
  const _hoisted_2 = { class: "mj-side pg-stack" };
  const _hoisted_3 = { class: "pg-scroll pg-stack" };
  const _hoisted_4 = { class: "mj-head" };
  const _hoisted_5 = {
    id: "online-list",
    class: "pg-list"
  };
  const _hoisted_6 = { class: "mj-head" };
  const _hoisted_7 = {
    id: "offline-list",
    class: "pg-list"
  };
  const _hoisted_8 = { class: "pg-grow pg-stack" };
  const _hoisted_9 = {
    key: 0,
    id: "tab-edit",
    class: "pg-grow pg-stack"
  };
  const _hoisted_10 = {
    id: "player-jobs",
    class: "mj-jobs pg-scroll pg-list"
  };
  const _hoisted_11 = { class: "mj-form" };
  const _hoisted_12 = { class: "pg-actions" };
  const _hoisted_13 = {
    key: 1,
    id: "tab-presets",
    class: "pg-grow pg-stack"
  };
  const _hoisted_14 = { class: "pg-line" };
  const _hoisted_15 = {
    id: "presets",
    class: "pg-grow pg-scroll pg-list mj-well"
  };
  const _hoisted_16 = { class: "pg-actions" };
  const _hoisted_17 = {
    key: 2,
    id: "tab-alljobs",
    class: "pg-grow pg-stack"
  };
  const _hoisted_18 = { class: "pg-line" };
  const _hoisted_19 = { class: "mj-name" };
  const _hoisted_20 = { class: "mj-code" };
  const _hoisted_21 = { class: "pg-num mj-grade" };
  const _hoisted_22 = { class: "pg-actions mj-foot" };
  const _hoisted_23 = {
    id: "alljobs-count",
    class: "pg-muted pg-num"
  };
  const _sfc_main = {
    __name: "App",
    setup(__props) {
      const open = vue.ref(false);
      const tab = vue.ref("edit");
      const TABS = [
        { value: "edit", label: "Edit Job" },
        { value: "presets", label: "Job Presets" },
        { value: "alljobs", label: "All Players" }
      ];
      const players = vue.ref([]);
      const selected = vue.ref(null);
      const playerJobs = vue.ref([]);
      const search = vue.ref("");
      const byName = (a, b) => a.charName.localeCompare(b.charName);
      const shown = (p) => !search.value || p.charName.toLowerCase().includes(search.value.toLowerCase());
      const online = vue.computed(() => players.value.filter((p) => p.isOnline && shown(p)).sort(byName));
      const offline = vue.computed(() => players.value.filter((p) => !p.isOnline && shown(p)).sort(byName));
      const isSelected = (p) => !!selected.value && selected.value.cid === p.cid;
      const playerName = vue.computed(() => selected.value ? selected.value.charName + (selected.value.isOnline ? " (Online)" : " (Offline)") : "");
      const form = vue.reactive({ job: "", label: "", grade: 0 });
      const busy = vue.reactive({});
      const later = (fn) => setTimeout(fn, 300);
      function fill(job, label, grade) {
        form.job = job;
        form.label = label;
        form.grade = grade;
      }
      function selectPlayer(p) {
        selected.value = p;
        fill(p.job, p.jobLabel, p.grade);
        ui.post("getPlayerJobs", { cid: p.cid });
        playerJobs.value = [{ job: p.job, joblabel: p.jobLabel, jobgrade: p.grade }];
      }
      function switchJob(job) {
        const p = selected.value;
        if (!p) return;
        p.job = job;
        busy["switch:" + job] = true;
        ui.post("switchPlayerJob", { cid: p.cid, job }).then(() => later(() => {
          ui.post("getPlayerJobs", { cid: p.cid });
          ui.post("refreshPlayers");
        }));
      }
      function removeJob(job) {
        const p = selected.value;
        if (!p) return;
        busy["remove:" + job] = true;
        ui.post("removePlayerJob", { cid: p.cid, job }).then(() => later(() => ui.post("getPlayerJobs", { cid: p.cid })));
      }
      function saveJob() {
        const p = selected.value;
        if (!p) return;
        const data = { cid: p.cid, job: form.job, jobLabel: form.label, grade: form.grade, oldJob: p.job };
        busy.save = true;
        ui.post("updateJob", data).then(() => {
          p.job = data.job;
          p.jobLabel = data.jobLabel;
          p.grade = data.grade;
          ui.post("getPlayerJobs", { cid: p.cid });
          ui.post("refreshPlayers");
        }).finally(() => {
          busy.save = false;
        });
      }
      function addJob() {
        const p = selected.value;
        const job = String(form.job || "").trim();
        if (!p || !job) return;
        busy.add = true;
        ui.post("addJob", { cid: p.cid, job, jobLabel: String(form.label || "").trim() || job, grade: form.grade || 0 }).then(() => later(() => ui.post("getPlayerJobs", { cid: p.cid }))).finally(() => {
          busy.add = false;
        });
      }
      const presets = vue.ref({});
      const presetSearch = vue.ref("");
      const category = vue.ref("all");
      const expanded = vue.reactive({});
      const preset = vue.ref(null);
      const categories = vue.computed(() => [{ value: "all", label: "All Categories" }].concat(Object.keys(presets.value).map((k) => ({ value: k, label: presets.value[k].name || k }))));
      const groups = vue.computed(() => {
        const q = presetSearch.value.toLowerCase();
        const out = [];
        for (const key of Object.keys(presets.value)) {
          if (category.value !== "all" && category.value !== key) continue;
          const g = presets.value[key];
          const jobs = (g.jobs || []).filter((j) => !q || String(j.job).toLowerCase().includes(q) || String(j.label).toLowerCase().includes(q) || j.category && String(j.category).toLowerCase().includes(q));
          if (!jobs.length) continue;
          const subs = [];
          const at = {};
          for (const j of jobs) {
            const name = j.category || "Other";
            if (!at[name]) subs.push(at[name] = { name, jobs: [] });
            at[name].jobs.push(j);
          }
          out.push({ key, name: g.name || key, subs });
        }
        return out;
      });
      function pickPreset(p) {
        if (preset.value === p) {
          preset.value = null;
          fill("", "", 0);
        } else {
          preset.value = p;
          fill(p.job, p.label, p.grade);
        }
      }
      function applyPreset() {
        const p = selected.value, pr = preset.value;
        if (!p || !pr) return;
        busy.apply = true;
        ui.post("addJob", { cid: p.cid, job: pr.job, jobLabel: pr.label, grade: pr.grade }).then(() => {
          tab.value = "edit";
          ui.post("getPlayerJobs", { cid: p.cid });
        }).finally(() => {
          busy.apply = false;
        });
      }
      const allJobs = vue.ref([]);
      const allSearch = vue.ref("");
      const jobFilter = vue.ref("all");
      const sort = vue.ref({ key: "name", dir: "asc" });
      const fullName = (r) => (r.firstname || "Unknown") + " " + (r.lastname || "Unknown");
      function daysSince(v) {
        if (!v || v === "Unknown") return null;
        const d = new Date(v);
        return isNaN(d.getTime()) ? null : Math.floor((/* @__PURE__ */ new Date() - d) / 864e5);
      }
      function lastLogin(v) {
        const d = daysSince(v);
        if (d === null) return v || "Unknown";
        if (d === 0) return "Today";
        if (d === 1) return "Yesterday";
        if (d < 7) return d + " days ago";
        if (d < 30) return Math.floor(d / 7) + " weeks ago";
        if (d < 365) return Math.floor(d / 30) + " months ago";
        return Math.floor(d / 365) + " years ago";
      }
      function loginClass(v) {
        const d = daysSince(v);
        return d === null ? "" : d <= 7 ? "mj-recent" : d > 30 ? "mj-old" : "";
      }
      const COLUMNS = [
        { key: "name", label: "Character", sortable: true, value: fullName, sortValue: (r) => fullName(r).toLowerCase() },
        { key: "job", label: "Job", sortable: true, sortValue: (r) => String(r.job).toLowerCase() },
        { key: "label", label: "Label", sortable: true, value: (r) => r.joblabel || r.job, sortValue: (r) => String(r.joblabel || r.job).toLowerCase() },
        { key: "grade", label: "Grade", sortable: true, num: true, value: (r) => r.jobgrade, sortValue: (r) => Number(r.jobgrade) || 0 },
        { key: "lastlogin", label: "Last Login", sortable: true, value: (r) => r.lastonline, sortValue: (r) => new Date(r.lastonline || 0).getTime() },
        { key: "actions", label: "Actions", num: true }
      ];
      const jobNames = vue.computed(() => [...new Set(allJobs.value.map((r) => r.job))].sort());
      const jobOptions = vue.computed(() => [{ value: "all", label: "All Jobs" }].concat(jobNames.value));
      const rows = vue.computed(() => {
        const q = allSearch.value.toLowerCase();
        return allJobs.value.filter((r) => (!q || fullName(r).toLowerCase().includes(q) || String(r.job).toLowerCase().includes(q) || r.joblabel && String(r.joblabel).toLowerCase().includes(q)) && (jobFilter.value === "all" || r.job === jobFilter.value));
      });
      function loadAllJobs() {
        ui.post("getAllJobs");
      }
      function removeEntry(r) {
        const key = "entry:" + r.cid + ":" + r.job;
        busy[key] = true;
        ui.post("removeJobEntry", { cid: r.cid, job: r.job });
      }
      function setTab(t) {
        tab.value = t;
        if (t === "alljobs") loadAllJobs();
      }
      function close() {
        open.value = false;
        ui.post("close");
      }
      function clearBusy(prefix) {
        for (const k of Object.keys(busy)) if (k.startsWith(prefix)) delete busy[k];
      }
      ui.onMessage({
        show(d) {
          if (!d.show) {
            open.value = false;
            return;
          }
          if (d.presets) presets.value = d.presets;
          for (const k of Object.keys(expanded)) delete expanded[k];
          preset.value = null;
          if (!categories.value.some((c) => c.value === category.value)) category.value = "all";
          open.value = true;
        },
        updatePlayers(d) {
          players.value = d.players || [];
          if (selected.value) {
            const again = players.value.find((p) => p.cid === selected.value.cid);
            if (again) selected.value = again;
          }
        },
        updatePlayerJobs(d) {
          playerJobs.value = d.jobs || [];
          clearBusy("switch:");
          clearBusy("remove:");
        },
        updateAllJobs(d) {
          allJobs.value = d.jobs || [];
          if (jobFilter.value !== "all" && !jobNames.value.includes(jobFilter.value)) jobFilter.value = "all";
          clearBusy("entry:");
        }
      });
      ui.onKey("Escape", close);
      return (_ctx, _cache) => {
        const _component_PgInput = vue.resolveComponent("PgInput");
        const _component_PgPill = vue.resolveComponent("PgPill");
        const _component_PgRow = vue.resolveComponent("PgRow");
        const _component_PgEmpty = vue.resolveComponent("PgEmpty");
        const _component_PgTabs = vue.resolveComponent("PgTabs");
        const _component_PgButton = vue.resolveComponent("PgButton");
        const _component_PgSwitcher = vue.resolveComponent("PgSwitcher");
        const _component_PgGroup = vue.resolveComponent("PgGroup");
        const _component_PgTable = vue.resolveComponent("PgTable");
        const _component_PgWindow = vue.resolveComponent("PgWindow");
        const _component_PgScreen = vue.resolveComponent("PgScreen");
        return vue.openBlock(), vue.createBlock(_component_PgScreen, { show: open.value }, {
          default: vue.withCtx(() => [
            vue.createVNode(_component_PgWindow, {
              class: "mj-window",
              title: "Multijob",
              width: "1100px",
              height: "700px",
              onClose: close
            }, {
              default: vue.withCtx(() => [
                vue.createElementVNode("div", _hoisted_1, [
                  vue.createCommentVNode(" players "),
                  vue.createElementVNode("aside", _hoisted_2, [
                    vue.createVNode(_component_PgInput, {
                      modelValue: search.value,
                      "onUpdate:modelValue": _cache[0] || (_cache[0] = ($event) => search.value = $event),
                      placeholder: "Search character..."
                    }, null, 8, ["modelValue"]),
                    vue.createElementVNode("div", _hoisted_3, [
                      vue.createElementVNode("div", _hoisted_4, [
                        _cache[9] || (_cache[9] = vue.createElementVNode(
                          "span",
                          { class: "pg-label" },
                          "Online",
                          -1
                          /* HOISTED */
                        )),
                        vue.createVNode(_component_PgPill, { variant: "success" }, {
                          default: vue.withCtx(() => [
                            vue.createTextVNode(
                              vue.toDisplayString(online.value.length),
                              1
                              /* TEXT */
                            )
                          ]),
                          _: 1
                          /* STABLE */
                        })
                      ]),
                      vue.createElementVNode("div", _hoisted_5, [
                        (vue.openBlock(true), vue.createElementBlock(
                          vue.Fragment,
                          null,
                          vue.renderList(online.value, (p) => {
                            return vue.openBlock(), vue.createBlock(_component_PgRow, {
                              key: p.cid,
                              title: p.charName,
                              clickable: "",
                              selected: isSelected(p),
                              onClick: ($event) => selectPlayer(p)
                            }, null, 8, ["title", "selected", "onClick"]);
                          }),
                          128
                          /* KEYED_FRAGMENT */
                        )),
                        !online.value.length ? (vue.openBlock(), vue.createBlock(_component_PgEmpty, { key: 0 }, {
                          default: vue.withCtx(() => _cache[10] || (_cache[10] = [
                            vue.createTextVNode("No online players")
                          ])),
                          _: 1
                          /* STABLE */
                        })) : vue.createCommentVNode("v-if", true)
                      ]),
                      vue.createElementVNode("div", _hoisted_6, [
                        _cache[11] || (_cache[11] = vue.createElementVNode(
                          "span",
                          { class: "pg-label" },
                          "Offline",
                          -1
                          /* HOISTED */
                        )),
                        vue.createVNode(_component_PgPill, null, {
                          default: vue.withCtx(() => [
                            vue.createTextVNode(
                              vue.toDisplayString(offline.value.length),
                              1
                              /* TEXT */
                            )
                          ]),
                          _: 1
                          /* STABLE */
                        })
                      ]),
                      vue.createElementVNode("div", _hoisted_7, [
                        (vue.openBlock(true), vue.createElementBlock(
                          vue.Fragment,
                          null,
                          vue.renderList(offline.value, (p) => {
                            return vue.openBlock(), vue.createBlock(_component_PgRow, {
                              key: p.cid,
                              title: p.charName,
                              clickable: "",
                              dim: "",
                              selected: isSelected(p),
                              onClick: ($event) => selectPlayer(p)
                            }, null, 8, ["title", "selected", "onClick"]);
                          }),
                          128
                          /* KEYED_FRAGMENT */
                        )),
                        !offline.value.length ? (vue.openBlock(), vue.createBlock(_component_PgEmpty, { key: 0 }, {
                          default: vue.withCtx(() => _cache[12] || (_cache[12] = [
                            vue.createTextVNode("No offline players")
                          ])),
                          _: 1
                          /* STABLE */
                        })) : vue.createCommentVNode("v-if", true)
                      ])
                    ])
                  ]),
                  vue.createElementVNode("main", _hoisted_8, [
                    vue.createVNode(_component_PgTabs, {
                      "model-value": tab.value,
                      tabs: TABS,
                      "onUpdate:modelValue": setTab
                    }, null, 8, ["model-value"]),
                    vue.createCommentVNode(" edit one player's jobs "),
                    tab.value === "edit" ? (vue.openBlock(), vue.createElementBlock("section", _hoisted_9, [
                      vue.createVNode(_component_PgInput, {
                        id: "player-name",
                        label: "Player",
                        "model-value": playerName.value,
                        disabled: "",
                        placeholder: "Pick a character on the left"
                      }, null, 8, ["model-value"]),
                      _cache[21] || (_cache[21] = vue.createElementVNode(
                        "span",
                        { class: "pg-label" },
                        "Player's jobs",
                        -1
                        /* HOISTED */
                      )),
                      vue.createElementVNode("div", _hoisted_10, [
                        !selected.value ? (vue.openBlock(), vue.createBlock(_component_PgEmpty, { key: 0 }, {
                          default: vue.withCtx(() => _cache[13] || (_cache[13] = [
                            vue.createTextVNode("Select a player to view their jobs")
                          ])),
                          _: 1
                          /* STABLE */
                        })) : !playerJobs.value.length ? (vue.openBlock(), vue.createBlock(_component_PgEmpty, { key: 1 }, {
                          default: vue.withCtx(() => _cache[14] || (_cache[14] = [
                            vue.createTextVNode("No jobs found for this player")
                          ])),
                          _: 1
                          /* STABLE */
                        })) : (vue.openBlock(true), vue.createElementBlock(
                          vue.Fragment,
                          { key: 2 },
                          vue.renderList(playerJobs.value, (j, i) => {
                            return vue.openBlock(), vue.createBlock(_component_PgRow, {
                              key: j.job + ":" + i,
                              current: j.job === selected.value.job,
                              title: (j.joblabel || j.job) + (j.job === selected.value.job ? " (Active)" : ""),
                              sub: "Job: " + j.job + "  ·  Grade: " + j.jobgrade
                            }, {
                              actions: vue.withCtx(() => [
                                j.job !== selected.value.job ? (vue.openBlock(), vue.createBlock(_component_PgButton, {
                                  key: 0,
                                  class: "mj-switch",
                                  variant: "primary",
                                  busy: !!busy["switch:" + j.job],
                                  "busy-text": "Switching...",
                                  onClick: ($event) => switchJob(j.job)
                                }, {
                                  default: vue.withCtx(() => _cache[15] || (_cache[15] = [
                                    vue.createTextVNode("Set Active")
                                  ])),
                                  _: 2
                                  /* DYNAMIC */
                                }, 1032, ["busy", "onClick"])) : vue.createCommentVNode("v-if", true),
                                vue.createVNode(_component_PgButton, {
                                  class: "mj-edit",
                                  onClick: ($event) => fill(j.job, j.joblabel, j.jobgrade)
                                }, {
                                  default: vue.withCtx(() => _cache[16] || (_cache[16] = [
                                    vue.createTextVNode("Edit")
                                  ])),
                                  _: 2
                                  /* DYNAMIC */
                                }, 1032, ["onClick"]),
                                vue.createVNode(_component_PgButton, {
                                  class: "mj-remove",
                                  variant: "danger-fill",
                                  busy: !!busy["remove:" + j.job],
                                  "busy-text": "Removing...",
                                  onClick: ($event) => removeJob(j.job)
                                }, {
                                  default: vue.withCtx(() => _cache[17] || (_cache[17] = [
                                    vue.createTextVNode("Remove")
                                  ])),
                                  _: 2
                                  /* DYNAMIC */
                                }, 1032, ["busy", "onClick"])
                              ]),
                              _: 2
                              /* DYNAMIC */
                            }, 1032, ["current", "title", "sub"]);
                          }),
                          128
                          /* KEYED_FRAGMENT */
                        ))
                      ]),
                      _cache[22] || (_cache[22] = vue.createElementVNode(
                        "hr",
                        { class: "pg-rule" },
                        null,
                        -1
                        /* HOISTED */
                      )),
                      _cache[23] || (_cache[23] = vue.createElementVNode(
                        "span",
                        { class: "pg-label" },
                        "Edit or add a job",
                        -1
                        /* HOISTED */
                      )),
                      vue.createElementVNode("div", _hoisted_11, [
                        vue.createVNode(_component_PgInput, {
                          id: "job-id",
                          modelValue: form.job,
                          "onUpdate:modelValue": _cache[1] || (_cache[1] = ($event) => form.job = $event),
                          label: "Job (ID)",
                          placeholder: "e.g. sheriff"
                        }, null, 8, ["modelValue"]),
                        vue.createVNode(_component_PgInput, {
                          id: "job-grade",
                          modelValue: form.grade,
                          "onUpdate:modelValue": _cache[2] || (_cache[2] = ($event) => form.grade = $event),
                          label: "Grade",
                          type: "number",
                          min: "0"
                        }, null, 8, ["modelValue"]),
                        vue.createVNode(_component_PgInput, {
                          id: "job-label",
                          modelValue: form.label,
                          "onUpdate:modelValue": _cache[3] || (_cache[3] = ($event) => form.label = $event),
                          class: "mj-wide",
                          label: "Job label",
                          placeholder: "e.g. DEPUTY"
                        }, null, 8, ["modelValue"])
                      ]),
                      vue.createElementVNode("div", _hoisted_12, [
                        vue.createVNode(_component_PgButton, {
                          id: "save-btn",
                          variant: "primary",
                          disabled: !selected.value,
                          busy: !!busy.save,
                          "busy-text": "Saving...",
                          onClick: saveJob
                        }, {
                          default: vue.withCtx(() => _cache[18] || (_cache[18] = [
                            vue.createTextVNode("Save Job")
                          ])),
                          _: 1
                          /* STABLE */
                        }, 8, ["disabled", "busy"]),
                        vue.createVNode(_component_PgButton, {
                          id: "add-job-btn",
                          variant: "success-fill",
                          disabled: !selected.value,
                          busy: !!busy.add,
                          "busy-text": "Adding...",
                          onClick: addJob
                        }, {
                          default: vue.withCtx(() => _cache[19] || (_cache[19] = [
                            vue.createTextVNode("Add as New Job")
                          ])),
                          _: 1
                          /* STABLE */
                        }, 8, ["disabled", "busy"]),
                        vue.createVNode(_component_PgButton, { onClick: close }, {
                          default: vue.withCtx(() => _cache[20] || (_cache[20] = [
                            vue.createTextVNode("Close")
                          ])),
                          _: 1
                          /* STABLE */
                        })
                      ])
                    ])) : vue.createCommentVNode("v-if", true),
                    vue.createCommentVNode(" presets "),
                    tab.value === "presets" ? (vue.openBlock(), vue.createElementBlock("section", _hoisted_13, [
                      vue.createElementVNode("div", _hoisted_14, [
                        vue.createVNode(_component_PgInput, {
                          id: "preset-search",
                          modelValue: presetSearch.value,
                          "onUpdate:modelValue": _cache[4] || (_cache[4] = ($event) => presetSearch.value = $event),
                          class: "pg-grow",
                          placeholder: "Search presets..."
                        }, null, 8, ["modelValue"]),
                        vue.createVNode(_component_PgSwitcher, {
                          id: "category",
                          modelValue: category.value,
                          "onUpdate:modelValue": _cache[5] || (_cache[5] = ($event) => category.value = $event),
                          options: categories.value
                        }, null, 8, ["modelValue", "options"])
                      ]),
                      vue.createElementVNode("div", _hoisted_15, [
                        (vue.openBlock(true), vue.createElementBlock(
                          vue.Fragment,
                          null,
                          vue.renderList(groups.value, (g) => {
                            return vue.openBlock(), vue.createBlock(_component_PgGroup, {
                              key: g.key,
                              title: g.name,
                              open: expanded[g.key],
                              "onUpdate:open": ($event) => expanded[g.key] = $event
                            }, {
                              default: vue.withCtx(() => [
                                (vue.openBlock(true), vue.createElementBlock(
                                  vue.Fragment,
                                  null,
                                  vue.renderList(g.subs, (s) => {
                                    return vue.openBlock(), vue.createBlock(_component_PgGroup, {
                                      key: s.name,
                                      sub: "",
                                      title: s.name,
                                      open: expanded[g.key + "/" + s.name],
                                      "onUpdate:open": ($event) => expanded[g.key + "/" + s.name] = $event
                                    }, {
                                      default: vue.withCtx(() => [
                                        (vue.openBlock(true), vue.createElementBlock(
                                          vue.Fragment,
                                          null,
                                          vue.renderList(s.jobs, (p, i) => {
                                            return vue.openBlock(), vue.createBlock(_component_PgRow, {
                                              key: i,
                                              class: "mj-preset",
                                              clickable: "",
                                              selected: preset.value === p,
                                              title: p.label,
                                              sub: "Job: " + p.job + "  ·  Grade: " + p.grade,
                                              onClick: ($event) => pickPreset(p)
                                            }, null, 8, ["selected", "title", "sub", "onClick"]);
                                          }),
                                          128
                                          /* KEYED_FRAGMENT */
                                        ))
                                      ]),
                                      _: 2
                                      /* DYNAMIC */
                                    }, 1032, ["title", "open", "onUpdate:open"]);
                                  }),
                                  128
                                  /* KEYED_FRAGMENT */
                                ))
                              ]),
                              _: 2
                              /* DYNAMIC */
                            }, 1032, ["title", "open", "onUpdate:open"]);
                          }),
                          128
                          /* KEYED_FRAGMENT */
                        )),
                        !groups.value.length ? (vue.openBlock(), vue.createBlock(_component_PgEmpty, { key: 0 }, {
                          default: vue.withCtx(() => _cache[24] || (_cache[24] = [
                            vue.createTextVNode("No presets match")
                          ])),
                          _: 1
                          /* STABLE */
                        })) : vue.createCommentVNode("v-if", true)
                      ]),
                      vue.createElementVNode("div", _hoisted_16, [
                        vue.createVNode(_component_PgButton, {
                          id: "apply-preset-btn",
                          variant: "primary",
                          disabled: !preset.value || !selected.value,
                          busy: !!busy.apply,
                          "busy-text": "Applying...",
                          onClick: applyPreset
                        }, {
                          default: vue.withCtx(() => _cache[25] || (_cache[25] = [
                            vue.createTextVNode("Apply Selected Preset")
                          ])),
                          _: 1
                          /* STABLE */
                        }, 8, ["disabled", "busy"]),
                        vue.createVNode(_component_PgButton, { onClick: close }, {
                          default: vue.withCtx(() => _cache[26] || (_cache[26] = [
                            vue.createTextVNode("Close")
                          ])),
                          _: 1
                          /* STABLE */
                        })
                      ])
                    ])) : vue.createCommentVNode("v-if", true),
                    vue.createCommentVNode(" every saved job "),
                    tab.value === "alljobs" ? (vue.openBlock(), vue.createElementBlock("section", _hoisted_17, [
                      vue.createElementVNode("div", _hoisted_18, [
                        vue.createVNode(_component_PgInput, {
                          id: "alljobs-search",
                          modelValue: allSearch.value,
                          "onUpdate:modelValue": _cache[6] || (_cache[6] = ($event) => allSearch.value = $event),
                          class: "pg-grow",
                          placeholder: "Search by name or job..."
                        }, null, 8, ["modelValue"]),
                        vue.createVNode(_component_PgSwitcher, {
                          id: "job-filter",
                          modelValue: jobFilter.value,
                          "onUpdate:modelValue": _cache[7] || (_cache[7] = ($event) => jobFilter.value = $event),
                          options: jobOptions.value
                        }, null, 8, ["modelValue", "options"]),
                        vue.createVNode(_component_PgButton, {
                          id: "refresh-btn",
                          variant: "primary",
                          onClick: loadAllJobs
                        }, {
                          default: vue.withCtx(() => _cache[27] || (_cache[27] = [
                            vue.createTextVNode("Refresh")
                          ])),
                          _: 1
                          /* STABLE */
                        })
                      ]),
                      vue.createVNode(_component_PgTable, {
                        class: "pg-grow",
                        columns: COLUMNS,
                        rows: rows.value,
                        "row-key": (r) => r.cid + ":" + r.job,
                        sort: sort.value,
                        "onUpdate:sort": _cache[8] || (_cache[8] = ($event) => sort.value = $event),
                        empty: "No jobs found"
                      }, {
                        "cell-name": vue.withCtx(({ value }) => [
                          vue.createElementVNode(
                            "span",
                            _hoisted_19,
                            vue.toDisplayString(value),
                            1
                            /* TEXT */
                          )
                        ]),
                        "cell-job": vue.withCtx(({ value }) => [
                          vue.createElementVNode(
                            "span",
                            _hoisted_20,
                            vue.toDisplayString(value),
                            1
                            /* TEXT */
                          )
                        ]),
                        "cell-grade": vue.withCtx(({ value }) => [
                          vue.createElementVNode(
                            "span",
                            _hoisted_21,
                            vue.toDisplayString(value),
                            1
                            /* TEXT */
                          )
                        ]),
                        "cell-lastlogin": vue.withCtx(({ value }) => [
                          vue.createElementVNode(
                            "span",
                            {
                              class: vue.normalizeClass(["mj-login", loginClass(value)])
                            },
                            vue.toDisplayString(lastLogin(value)),
                            3
                            /* TEXT, CLASS */
                          )
                        ]),
                        "cell-actions": vue.withCtx(({ row }) => [
                          vue.createVNode(_component_PgButton, {
                            class: "mj-entry-remove",
                            variant: "danger-fill",
                            busy: !!busy["entry:" + row.cid + ":" + row.job],
                            "busy-text": "Removing...",
                            onClick: vue.withModifiers(($event) => removeEntry(row), ["stop"])
                          }, {
                            default: vue.withCtx(() => _cache[28] || (_cache[28] = [
                              vue.createTextVNode("Remove")
                            ])),
                            _: 2
                            /* DYNAMIC */
                          }, 1032, ["busy", "onClick"])
                        ]),
                        _: 1
                        /* STABLE */
                      }, 8, ["rows", "row-key", "sort"]),
                      vue.createElementVNode("div", _hoisted_22, [
                        vue.createElementVNode(
                          "span",
                          _hoisted_23,
                          vue.toDisplayString(rows.value.length) + " " + vue.toDisplayString(rows.value.length === 1 ? "job" : "jobs") + " found",
                          1
                          /* TEXT */
                        ),
                        vue.createVNode(_component_PgButton, { onClick: close }, {
                          default: vue.withCtx(() => _cache[29] || (_cache[29] = [
                            vue.createTextVNode("Close")
                          ])),
                          _: 1
                          /* STABLE */
                        })
                      ])
                    ])) : vue.createCommentVNode("v-if", true)
                  ])
                ])
              ]),
              _: 1
              /* STABLE */
            })
          ]),
          _: 1
          /* STABLE */
        }, 8, ["show"]);
      };
    }
  };
  ui.mount(_sfc_main);
})(PoggyUI, Vue);
