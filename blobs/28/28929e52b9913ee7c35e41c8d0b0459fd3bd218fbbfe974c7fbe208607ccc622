/* ═══════════════════════════════════════════════════════════════════
 *  poggy_crafting · ui/app.js
 *  Vue 3 application behind the crafting browser.
 *
 *  It is handed everything it draws in one message from client/ui.lua and
 *  talks back through NUI callbacks of the same names. It decides nothing:
 *  every lock it draws is checked again on the server before anything moves.
 * ═══════════════════════════════════════════════════════════════════ */

const { createApp } = Vue;

/* ─────────────────────────────────────────────────────────────────
 *  ChainNode Component
 *  Recursive component that renders the crafting-chain tree for a
 *  single recipe, showing each ingredient and whether it can itself
 *  be crafted (with expandable sub-chains up to 3 levels deep).
 * ───────────────────────────────────────────────────────────────── */
const ChainNodeComponent = {
    name: "chain-node",

    props: {
        node:                     { type: Object,   required: true },
        inventory:                { type: Object,   default: () => ({}) },
        allCraftablesUnfiltered:  { type: Array,    default: () => [] },
        allCraftables:            { type: Array,    default: () => [] },
        catColorMap:              { type: Object,   default: () => ({}) },
        itemLabels:               { type: Object,   default: () => ({}) },
        itemSources:              { type: Object,   default: () => ({}) },
        onIngHover:               { type: Function, default: null },
        onIngLeave:               { type: Function, default: null },
        job:                      { default: null },
        depth:                    { type: Number,   default: 0 }
    },

    data() {
        return {
            expanded: {}
        };
    },

    created() {
        // Auto-expand ingredients that have a craftable source recipe
        this.ingredients.forEach((item, index) => {
            if (item.sourceRecipe) {
                this.expanded[index] = true;
            }
        });
    },

    computed: {
        catColor() {
            return this.catColorMap[this.node.recipe.Category] || "var(--accent)";
        },

        catBg() {
            const catKeys = Object.keys(this.catColorMap);
            const idx = catKeys.indexOf(this.node.recipe.Category) % 12 + 1;
            return `var(--cat${idx}-bg)`;
        },

        ingredients() {
            const recipe = this.node.recipe;
            if (!recipe || !recipe.Items) return [];

            return recipe.Items.map(ing => {
                // Find a recipe whose Reward produces this ingredient
                const sourceRecipe = this.allCraftablesUnfiltered.find(
                    r => r.Reward && r.Reward.some(rw => rw.name === ing.name)
                );

                // Locked if the recipe exists but isn't in the player's accessible list
                const isLocked = sourceRecipe && !this.allCraftables.find(
                    r => r.Text === sourceRecipe.Text
                );

                const have = (() => {
                    const names = [ing.name, ...(ing.AltNames || [])];
                    return names.reduce((s, n) => s + (this.inventory[n] || 0), 0);
                })();

                return {
                    ing,
                    sourceRecipe,
                    isLocked,
                    have,
                    status: have >= ing.count ? "ok" : (have > 0 ? "warn" : "bad")
                };
            });
        }
    },

    methods: {
        toggleExpand(index) {
            this.expanded[index] = !this.expanded[index];
        },

        nodeColor(recipe) {
            return (recipe && this.catColorMap[recipe.Category]) || "var(--accent)";
        },

        nodeBg(recipe) {
            if (!recipe) return "var(--bg-card)";
            const catKeys = Object.keys(this.catColorMap);
            const idx = catKeys.indexOf(recipe.Category) % 12 + 1;
            return `var(--cat${idx}-bg)`;
        },

        imgUrl(name) {
            // The icon folder is whatever poggy_core reports for the running
            // framework, so this page is not tied to any one inventory.
            return name ? `${this.$root.imageBase}${encodeURIComponent(name)}.png` : "";
        },

        ingDisplayName(ing) {
            const names = [ing.name, ...(ing.AltNames || [])];
            return names[this.$root.altRotationIndex % names.length];
        },

        // Returns the source recipe for whichever alt name is currently displayed.
        // Reactive to altRotationIndex via ingDisplayName.
        getIngCurrentSourceRecipe(ing) {
            const displayName = this.ingDisplayName(ing);
            return this.allCraftablesUnfiltered.find(
                r => r.Reward && r.Reward.some(rw => rw.name === displayName)
            ) || null;
        },

        isIngCurrentSourceLocked(ing) {
            const sr = this.getIngCurrentSourceRecipe(ing);
            if (!sr) return false;
            return !this.allCraftables.find(r => r.Text === sr.Text);
        },

        recipeLabel(recipe) {
            if (!recipe) return "";
            const rewardName = (recipe.Reward && recipe.Reward[0]) ? recipe.Reward[0].name : "";
            return (rewardName && this.itemLabels[rewardName]) || recipe.Text || "";
        }
    },

    template: `
    <div class="cn-wrapper" :style="{'--depth': depth}">
      <div class="cn-recipe-bar"
           :style="{'--node-color': catColor, '--node-border': catColor+'55', '--node-bg-color': catBg}">
        <div class="cn-recipe-bar-main">
          <div class="cn-recipe-img-wrap">
            <div class="cn-recipe-img"
                 :style="{backgroundImage: node.recipe.Reward && node.recipe.Reward[0]
                   ? 'url(' + imgUrl(node.recipe.Reward[0].name) + ')'
                   : ''}">
            </div>
          </div>
          <div class="cn-recipe-name">{{ recipeLabel(node.recipe) }}</div>
        </div>
        <div class="cn-recipe-cat">{{ node.recipe.Category }}</div>
      </div>

      <div v-if="ingredients.length" class="cn-flow" :style="{'--node-color': catColor}">
        <div class="cn-flow-track">
          <div class="cn-flow-particle" :style="{background: catColor, boxShadow: '0 0 5px '+catColor}"></div>
        </div>
        <span class="cn-flow-label">requires</span>
      </div>

      <div class="cn-ingredients" v-if="ingredients.length">
        <div class="cn-ing-wrap" v-for="(item, i) in ingredients" :key="i">
          <div class="cn-ing-card"
               :class="{
                 'cn-ing-craftable': !!getIngCurrentSourceRecipe(item.ing) && !isIngCurrentSourceLocked(item.ing),
                 'cn-ing-locked':    isIngCurrentSourceLocked(item.ing),
                 'cn-ing-missing':   item.status === 'bad' && !getIngCurrentSourceRecipe(item.ing),
                 'cn-ing-clickable': !!getIngCurrentSourceRecipe(item.ing)
               }"
               @click.stop="getIngCurrentSourceRecipe(item.ing) ? $root.openDetail(getIngCurrentSourceRecipe(item.ing)) : null"
               @mouseenter="onIngHover && onIngHover(ingDisplayName(item.ing), $event)"
               @mouseleave="onIngLeave && onIngLeave()">
            <div class="cn-ing-img-wrap">
              <div class="cn-ing-img" :style="{backgroundImage: 'url('+imgUrl(ingDisplayName(item.ing))+')'}"></div>
              <div v-if="item.ing.AltNames && item.ing.AltNames.length" class="ing-alt-badge">ALT</div>
              <div class="cn-ing-count-badge" :class="item.status">
                {{ item.have }}/{{ item.ing.count }}
              </div>
            </div>
            <div class="cn-ing-label">{{ itemLabels[ingDisplayName(item.ing)] || item.ing.label || ingDisplayName(item.ing) }}</div>
            <div v-if="!getIngCurrentSourceRecipe(item.ing)" class="cn-raw-badge">Raw Ingredient</div>
            <div v-if="isIngCurrentSourceLocked(item.ing)" class="cn-locked-badge">
              <svg class="cn-lock-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <rect x="3" y="11" width="18" height="11" rx="2" ry="2"/>
                <path d="M7 11V7a5 5 0 0110 0v4"/>
              </svg>
              {{ getIngCurrentSourceRecipe(item.ing) && getIngCurrentSourceRecipe(item.ing).Job
                   ? (Array.isArray(getIngCurrentSourceRecipe(item.ing).Job) ? getIngCurrentSourceRecipe(item.ing).Job[0] : getIngCurrentSourceRecipe(item.ing).Job)
                   : 'Locked' }}
            </div>
            <template v-if="getIngCurrentSourceRecipe(item.ing)">
              <div class="cn-ing-actions">
                <button v-if="depth < 3" class="cn-expand-btn"
                        :style="{'--node-color': nodeColor(getIngCurrentSourceRecipe(item.ing)), '--node-bg-color': nodeBg(getIngCurrentSourceRecipe(item.ing))}"
                        @click.stop="toggleExpand(i)">
                  {{ expanded[i] ? '▲ Collapse' : '▼ See Chain' }}
                </button>
                <div v-else class="cn-depth-note">Max depth</div>
                <button class="cn-view-btn"
                        :style="{'--node-color': nodeColor(getIngCurrentSourceRecipe(item.ing))}"
                        @click.stop="$root.openDetail(getIngCurrentSourceRecipe(item.ing))">
                  View →
                </button>
              </div>
              <transition name="cn-expand">
                <div v-if="expanded[i] && depth < 3" class="cn-sub-chain">
                  <div class="cn-sub-connector"></div>
                  <chain-node
                    :node="{recipe: getIngCurrentSourceRecipe(item.ing)}"
                    :inventory="inventory"
                    :all-craftables-unfiltered="allCraftablesUnfiltered"
                    :all-craftables="allCraftables"
                    :cat-color-map="catColorMap"
                    :item-labels="itemLabels"
                    :item-sources="itemSources"
                    :on-ing-hover="onIngHover"
                    :on-ing-leave="onIngLeave"
                    :job="job"
                    :depth="depth + 1">
                  </chain-node>
                </div>
              </transition>
            </template>
          </div>
        </div>
      </div>
      <div v-else class="cn-no-chain">No ingredients required.</div>
    </div>
  `
};


/* ─────────────────────────────────────────────────────────────────
 *  Main Application
 * ───────────────────────────────────────────────────────────────── */
const app = createApp({

    components: {
        "chain-node": ChainNodeComponent
    },

    data() {
        return {
            // ── UI state ──
            isVisible: false,
            devMode: false,
            view: "grid",                // "grid" | "detail" | "shopping"
            selectedRecipe: null,
            recipeChain: null,
            searchText: "",
            filterOwned: false,
            filterAccessible: true,
            selectedCategory: "all",

            // ── Craft dialog ──
            showQtyDialog: false,
            craftItem: null,
            craftQty: 1,

            // ── Server data ──
            job: null,
            // Where item icons live. poggy_core reports this per framework;
            // the fallback only matters in devMode, outside the game.
            imageBase: "nui://vorp_inventory/html/img/items/",
            shoppingListEnabled: true,
            language: {},
            location: {},
            categories: [],
            allCraftables: [],           // accessible recipes only
            allCraftablesUnfiltered: [],  // every recipe (locked + unlocked)
            inventory: {},
            itemLabels: {},
            itemLimits: {},
            itemSources: {},
            categoryColorMap: {},

            // ── Image cache (persists across menu opens) ──
            imageCache: {},
            imageCacheKey: "poggy_crafting_icons",

            // ── Progress bar (drawn here, so there is no bar resource) ──
            progress: { shown: false, label: "", duration: 0, position: "bottom", fill: 0 },
            progressTimers: [],

            // ── Tooltip ──
            hoveredIng: null,
            tooltipPos: { x: 0, y: 0 },

            // ── Style / timing ──
            style: { fontSize: "m" },
            crafttime: 15000,
            max: 999,
            min: 1,

            // ── Shopping list ──
            shoppingList: [],
            recipeRawMap: {},

            // ── Gathering tracker ──
            trackedRecipeNames: [],
            trackerPos: { x: 20, y: 80 },
            trackerDragging: false,
            trackerDragOffset: { x: 0, y: 0 },
            trackerMinimized: false,
            trackerHidden: false,
            trackerFocusMode: false,

            // ── Alt-ingredient rotation ──
            altRotationIndex: 0,
            altRotationTimer: null,
            hoveredAltIng: null,
            hoveredAltIndex: 0,

            // ── Dev-mode test fixtures ──
            testData: [
                {
                    Text: "Dried Beef", Category: "food", Job: 0, Location: 0,
                    Items: [{ name: "raw_beef", count: 2, label: "Raw Beef" }],
                    Reward: [{ name: "dried_beef", count: 1 }]
                },
                {
                    Text: "Rope", Category: "tools", Job: 0, Location: 0,
                    Items: [
                        { name: "fiber", count: 4, label: "Fiber" },
                        { name: "twine", count: 2, label: "Twine" }
                    ],
                    Reward: [{ name: "rope", count: 1 }]
                }
            ],
            testCategory: [
                { ident: "food",  text: "Food",  Job: 0, Location: 0 },
                { ident: "tools", text: "Tools", Job: 0, Location: 0 }
            ]
        };
    },

    /* ── Lifecycle ──────────────────────────────────────────────── */

    mounted() {
        window.addEventListener("message", this.onMessage);
        window.addEventListener("keydown", this.onKeypress);

        this.altRotationTimer = setInterval(() => { this.altRotationIndex++; }, 1500);

        if (this.devMode) {
            this.setData({
                craftables: this.testData,
                categories: this.testCategory,
                crafttime: 15000,
                style: { fontSize: "m" },
                language: {},
                inventory: {}
            });
            this.isVisible = true;
        }
    },

    beforeUnmount() {
        window.removeEventListener("message", this.onMessage);
        window.removeEventListener("keydown", this.onKeypress);
        window.removeEventListener("mousemove", this.onTrackerDrag);
        window.removeEventListener("mouseup", this.stopTrackerDrag);
        if (this.altRotationTimer) clearInterval(this.altRotationTimer);
    },

    /* ── Computed ───────────────────────────────────────────────── */

    computed: {
        filteredCraftables() {
            let list = this.allCraftablesUnfiltered;

            // Category filter
            if (this.selectedCategory !== "all") {
                list = list.filter(r => r.Category === this.selectedCategory);
            }

            // Search filter (matches recipe name, ingredient names, reward names)
            if (this.searchText) {
                const term = this.searchText.toLowerCase();
                list = list.filter(r =>
                    (r.Text && r.Text.toLowerCase().includes(term)) ||
                    (r.Items && r.Items.some(ing =>
                        (this.itemLabels[ing.name] || ing.label || ing.name || "").toLowerCase().includes(term)
                    )) ||
                    (r.Reward && r.Reward.some(rw =>
                        (this.itemLabels[rw.name] || rw.name || "").toLowerCase().includes(term)
                    ))
                );
            }

            // "Materials only" — player has all ingredients
            if (this.filterOwned) {
                list = list.filter(r => !r._locked && this.playerHasAllIngredients(r));
            }

            // "Only Accessible" — hide locked recipes
            if (this.filterAccessible) {
                list = list.filter(r => !r._locked);
            }

            // Sort: unlocked first, then alphabetical
            list = [...list].sort((a, b) => {
                if (!!a._locked !== !!b._locked) return a._locked ? 1 : -1;
                return (a.Text || "").localeCompare(b.Text || "");
            });

            return list;
        },

        usedInRecipes() {
            if (!this.selectedRecipe || !this.selectedRecipe.Reward) return [];
            const rewardNames = new Set(this.selectedRecipe.Reward.map(rw => rw.name));
            return this.allCraftablesUnfiltered.filter(
                r => r.Items && r.Items.some(ing =>
                    rewardNames.has(ing.name) ||
                    (ing.AltNames && ing.AltNames.some(alt => rewardNames.has(alt)))
                )
            );
        },

        usedInAccessible() {
            return this.usedInRecipes.filter(r => !r._locked);
        },

        usedInLocked() {
            return this.usedInRecipes.filter(r => r._locked);
        },

        shoppingByRecipe() {
            const groups = {};
            const order = [];

            for (const item of this.shoppingList) {
                const key = item.recipe_name || "Other";
                if (!groups[key]) {
                    groups[key] = { recipeName: key, items: [] };
                    order.push(key);
                }
                groups[key].items.push(item);
            }

            return order.map(key => groups[key]);
        },

        trackedGroups() {
            if (!this.trackedRecipeNames.length) return [];
            return this.shoppingByRecipe.filter(
                g => this.trackedRecipeNames.includes(g.recipeName)
            );
        },

        trackerVisible() {
            return this.trackedGroups.length > 0;
        }
    },

    /* ── Methods ───────────────────────────────────────────────── */

    methods: {

        /* ·· NUI message handler ···························· */

        onMessage(event) {
            switch (event.data.type) {
                case "open":
                    this.setData(event.data);
                    this.isVisible = true;
                    this.view = "grid";
                    if (this.shoppingListEnabled) this.loadShoppingList();
                    break;

                case "close":
                    this.hoveredIng = null;
                    this.isVisible = false;
                    this.view = "grid";
                    this.selectedRecipe = null;
                    this.selectedCategory = "all";
                    this.searchText = "";
                    fetch(`https://${GetParentResourceName()}/closed`, {
                        method: "POST"
                    }).catch(() => {});
                    break;

                case "busy":
                    this.animationPlaying(event.data.duration);
                    break;

                case "progress":
                    this.showProgress(event.data);
                    break;

                case "progressEnd":
                    this.hideProgress();
                    break;

                case "inventory":
                    if (event.data.inventory) {
                        this.inventory = event.data.inventory;
                    }
                    break;

                case "trackerFocus":
                    this.trackerFocusMode = true;
                    break;

                case "trackerToggle":
                    this.trackerHidden = !this.trackerHidden;
                    break;
            }
        },

        /* ·· Keyboard handler ······························ */

        onKeypress(event) {
            const tag = event.target && event.target.tagName;

            // Let inputs handle their own keys; only intercept Escape to blur
            if (tag === "INPUT" || tag === "TEXTAREA" || event.target.isContentEditable) {
                if (event.key === "Escape") event.target.blur();
                return;
            }

            // Escape: close dialogs → back to grid → close menu
            if (event.key === "Escape") {
                if (this.trackerFocusMode) {
                    this.releaseTrackerFocus();
                    return;
                }
                if (this.showQtyDialog) {
                    this.showQtyDialog = false;
                } else if (this.view === "detail") {
                    this.view = "grid";
                } else {
                    this.closeMenu();
                }
            }
        },

        /* ·· Category helpers ······························ */

        getCatColor(ident) {
            return this.categoryColorMap[ident] || "var(--accent)";
        },

        getCatBg(ident) {
            const idx = Object.keys(this.categoryColorMap).indexOf(ident) % 12 + 1;
            return `var(--cat${idx}-bg)`;
        },

        getCatCount(ident) {
            return this.allCraftablesUnfiltered.filter(r => r.Category === ident).length;
        },

        getRecipeJobLabel(recipe) {
            if (!recipe) return "";
            if (recipe.Job && recipe.Job !== 0) {
                return this.t("ui_requires",
                    Array.isArray(recipe.Job) ? recipe.Job.join(", ") : String(recipe.Job));
            }
            if (recipe.Location && recipe.Location !== 0) {
                return this.t("ui_requires_place");
            }
            return this.t("ui_unavailable");
        },

        selectCategory(ident) {
            this.selectedCategory = ident;
            if (this.view === "shopping" || this.view === "detail") {
                this.view = "grid";
            }
        },

        /* ·· Inventory & crafting math ····················· */

        getInventoryCount(name) {
            return this.inventory[name] || 0;
        },

        // Sums inventory count across primary ingredient name + all its AltNames
        getIngHaveCount(ing) {
            const names = [ing.name, ...(ing.AltNames || [])];
            return names.reduce((sum, n) => sum + (this.inventory[n] || 0), 0);
        },

        // Returns the currently visible name (rotates through primary + alts)
        // If this exact ingredient is being hovered, returns the frozen/cycled index instead
        getIngDisplayName(ing) {
            const names = [ing.name, ...(ing.AltNames || [])];
            if (this.hoveredAltIng === ing) {
                return names[this.hoveredAltIndex % names.length];
            }
            return names[this.altRotationIndex % names.length];
        },

        // Returns the CSS background-image URL for the currently visible alt
        getIngDisplayImageUrl(ing) {
            return this.getItemImageUrl(this.getIngDisplayName(ing));
        },

        // Called on mouseenter for a grid ingredient pill; freezes rotation if it has alts
        enterAltHover(ing, event) {
            if (ing.AltNames && ing.AltNames.length) {
                const names = [ing.name, ...(ing.AltNames || [])];
                this.hoveredAltIndex = this.altRotationIndex % names.length;
                this.hoveredAltIng = ing;
                if (this.altRotationTimer) {
                    clearInterval(this.altRotationTimer);
                    this.altRotationTimer = null;
                }
            }
            this.showIngTooltip(this.getIngDisplayName(ing), event);
        },

        // Called on mouseleave; resumes rotation
        leaveAltHover() {
            this.hoveredAltIng = null;
            if (!this.altRotationTimer) {
                this.altRotationTimer = setInterval(() => { this.altRotationIndex++; }, 1500);
            }
            this.hideIngTooltip();
        },

        // Step through alts manually while hovering; also syncs the tooltip
        cycleAlt(delta) {
            if (!this.hoveredAltIng) return;
            const names = [this.hoveredAltIng.name, ...(this.hoveredAltIng.AltNames || [])];
            this.hoveredAltIndex = ((this.hoveredAltIndex + delta) % names.length + names.length) % names.length;
            if (this.hoveredIng) {
                const name = names[this.hoveredAltIndex];
                this.hoveredIng.name = name;
                this.hoveredIng.source = this.getItemSource(name);
            }
        },

        getItemLimit(name) {
            return this.itemLimits[name] || 0;
        },

        getMaxCraftable(recipe) {
            if (!recipe || !recipe.Items || !recipe.Items.length) return 999;

            let maxCrafts = 999;

            // Limit by available ingredients (primary + all AltNames)
            for (const ing of recipe.Items) {
                const have = this.getIngHaveCount(ing);
                const possible = Math.floor(have / ing.count);
                if (possible < maxCrafts) maxCrafts = possible;
            }

            // Limit by inventory caps on reward items
            if (recipe.Reward) {
                for (const rw of recipe.Reward) {
                    if (!rw.name || !rw.count) continue;
                    const limit = this.getItemLimit(rw.name);
                    if (limit && limit > 0) {
                        const current = this.getInventoryCount(rw.name);
                        const room = Math.max(0, limit - current);
                        const possible = Math.floor(room / rw.count);
                        if (possible < maxCrafts) maxCrafts = possible;
                    }
                }
            }

            return Math.max(0, maxCrafts);
        },

        playerHasAllIngredients(recipe) {
            return this.getMaxCraftable(recipe) > 0;
        },

        hasSubChain(recipe) {
            if (!recipe || !recipe.Items) return false;
            return recipe.Items.some(ing =>
                this.allCraftablesUnfiltered.some(
                    r => r.Reward && r.Reward.some(rw => rw.name === ing.name)
                )
            );
        },

        /* ·· Detail view ··································· */

        openDetail(recipe) {
            this.hoveredIng = null;
            this.selectedRecipe = recipe;
            this.recipeChain = { recipe };
            this.view = "detail";
        },

        /* ·· Craft dialog ·································· */

        startCraft(recipe) {
            if (this.getMaxCraftable(recipe) < 1) return;
            this.craftItem = recipe;
            this.craftQty = 1;
            this.showQtyDialog = true;
        },

        adjustQty(delta) {
            const cap = Math.min(this.max, this.getMaxCraftable(this.craftItem));
            this.craftQty = Math.max(1, Math.min(cap, this.craftQty + delta));
        },

        confirmCraft() {
            if (!this.craftItem) return;

            const recipe = this.craftItem;
            const qty = this.craftQty;

            this.showQtyDialog = false;
            this.craftItem = null;
            this.craftQty = 1;

            fetch(`https://${GetParentResourceName()}/craft`, {
                method: "POST",
                body: JSON.stringify({
                    // Only the name travels. The server looks the recipe up
                    // again from its own config, so nothing here can change
                    // what an ingredient costs or a reward gives.
                    recipe: recipe.Text,
                    quantity: qty,
                    location: this.location
                })
            }).catch(console.warn);
        },

        updateInventoryAfterCrafting(recipe, qty) {
            if (recipe.Items) {
                recipe.Items.forEach(ing => {
                    this.inventory[ing.name] = Math.max(0, (this.inventory[ing.name] || 0) - ing.count * qty);
                });
            }
            if (recipe.Reward) {
                recipe.Reward.forEach(rw => {
                    this.inventory[rw.name] = (this.inventory[rw.name] || 0) + rw.count * qty;
                });
            }
        },

        /* ·· Shopping list helpers ························· */

        shoppingItemHave(item) {
            return this.inventory[item.item_name] || 0;
        },

        shoppingItemFill(item) {
            return Math.min(1, this.shoppingItemHave(item) / Math.max(1, item.quantity));
        },

        shoppingItemStatus(item) {
            const have = this.shoppingItemHave(item);
            return have >= item.quantity ? "ok" : (have > 0 ? "warn" : "bad");
        },

        /* ·· Tracker ······································· */

        isRecipeTracked(recipeName) {
            return this.trackedRecipeNames.includes(recipeName);
        },

        toggleTrackRecipe(recipeName) {
            const idx = this.trackedRecipeNames.indexOf(recipeName);
            if (idx >= 0) {
                this.trackedRecipeNames.splice(idx, 1);
            } else {
                this.trackedRecipeNames.push(recipeName);
            }
        },

        openDetailByName(recipeName) {
            const recipe = this.allCraftablesUnfiltered.find(r => r.Text === recipeName);
            if (recipe) {
                if (!this.isVisible) this.isVisible = true;
                this.openDetail(recipe);
            }
        },

        startTrackerDrag(event) {
            this.trackerDragging = true;
            this.trackerDragOffset = {
                x: event.clientX - this.trackerPos.x,
                y: event.clientY - this.trackerPos.y
            };
            window.addEventListener("mousemove", this.onTrackerDrag);
            window.addEventListener("mouseup", this.stopTrackerDrag);
        },

        onTrackerDrag(event) {
            if (!this.trackerDragging) return;
            this.trackerPos = {
                x: Math.max(0, Math.min(window.innerWidth - 340, event.clientX - this.trackerDragOffset.x)),
                y: Math.max(0, Math.min(window.innerHeight - 60, event.clientY - this.trackerDragOffset.y))
            };
        },

        stopTrackerDrag() {
            this.trackerDragging = false;
            window.removeEventListener("mousemove", this.onTrackerDrag);
            window.removeEventListener("mouseup", this.stopTrackerDrag);
            if (this.trackerFocusMode) this.releaseTrackerFocus();
        },

        releaseTrackerFocus() {
            this.trackerFocusMode = false;
            fetch(`https://${GetParentResourceName()}/trackerRelease`, {
                method: "POST"
            }).catch(() => {});
        },

        /* ·· Shopping list CRUD ···························· */

        openShopping() {
            this.view = "shopping";
        },

        loadShoppingList() {
            fetch(`https://${GetParentResourceName()}/list:get`, {
                method: "POST"
            })
            .then(res => res.json())
            .then(data => {
                if (Array.isArray(data)) this.shoppingList = data;
            })
            .catch(() => {});
        },

        _gatherRawIngredients(recipe, multiplier, result, visited) {
            if (!recipe || !recipe.Items) return;
            if (visited.has(recipe.Text)) return;

            const newVisited = new Set(visited);
            newVisited.add(recipe.Text);

            for (const ing of recipe.Items) {
                const needed = ing.count * multiplier;

                // Check if this ingredient is itself craftable
                const subRecipe = this.allCraftablesUnfiltered.find(
                    r => r.Reward && r.Reward.some(rw => rw.name === ing.name)
                );

                if (subRecipe && !visited.has(subRecipe.Text)) {
                    const yieldPerCraft = (subRecipe.Reward.find(rw => rw.name === ing.name) || {}).count || 1;
                    const craftsNeeded = Math.ceil(needed / yieldPerCraft);
                    this._gatherRawIngredients(subRecipe, craftsNeeded, result, newVisited);
                } else {
                    result[ing.name] = (result[ing.name] || 0) + needed;
                }
            }
        },

        addToShoppingList(recipe) {
            if (!recipe || !recipe.Items) return;

            const rawMap = {};
            this._gatherRawIngredients(recipe, 1, rawMap, new Set());
            this.recipeRawMap[recipe.Text] = rawMap;

            const catColorNum = Object.keys(this.categoryColorMap).indexOf(recipe.Category) % 12 + 1;

            Object.entries(rawMap).forEach(([itemName, qty]) => {
                fetch(`https://${GetParentResourceName()}/list:add`, {
                    method: "POST",
                    body: JSON.stringify({
                        item_name: itemName,
                        item_label: this.getItemDisplayLabel(itemName),
                        quantity: Math.ceil(qty),
                        recipe_name: recipe.Text,
                        cat_color_num: catColorNum
                    })
                })
                .then(res => res.json())
                .then(data => {
                    if (data && data.id) this.shoppingList.push(data);
                })
                .catch(() => {});
            });
        },

        removeShoppingItem(id) {
            fetch(`https://${GetParentResourceName()}/list:remove`, {
                method: "POST",
                body: JSON.stringify({ id })
            })
            .then(() => {
                this.shoppingList = this.shoppingList.filter(i => i.id !== id);
            })
            .catch(() => {
                this.shoppingList = this.shoppingList.filter(i => i.id !== id);
            });
        },

        toggleShoppingItem(id) {
            fetch(`https://${GetParentResourceName()}/list:toggle`, {
                method: "POST",
                body: JSON.stringify({ id })
            })
            .then(res => res.json())
            .then(data => {
                const item = this.shoppingList.find(i => i.id === id);
                if (item) item.checked = data.checked;
            })
            .catch(() => {
                const item = this.shoppingList.find(i => i.id === id);
                if (item) item.checked = !item.checked;
            });
        },

        updateShoppingQty(item, delta) {
            const newQty = Math.max(1, item.quantity + delta);
            item.quantity = newQty;
            fetch(`https://${GetParentResourceName()}/list:qty`, {
                method: "POST",
                body: JSON.stringify({ id: item.id, quantity: newQty })
            }).catch(() => {});
        },

        clearCheckedItems() {
            fetch(`https://${GetParentResourceName()}/list:clear`, {
                method: "POST"
            })
            .then(() => {
                this.shoppingList = this.shoppingList.filter(i => !i.checked);
            })
            .catch(() => {});
        },

        getGroupRecipeQty(group) {
            const rawMap = this.recipeRawMap[group.recipeName];
            if (!rawMap) return 1;

            let minQty = Infinity;
            for (const item of group.items) {
                const perOne = rawMap[item.item_name];
                if (perOne && perOne > 0) {
                    minQty = Math.min(minQty, Math.round(item.quantity / perOne));
                }
            }
            return (isFinite(minQty) && minQty > 0) ? minQty : 1;
        },

        adjustRecipeQty(group, delta) {
            let rawMap = this.recipeRawMap[group.recipeName];

            if (!rawMap) {
                const recipe = this.allCraftablesUnfiltered.find(r => r.Text === group.recipeName);
                if (!recipe) return;
                rawMap = {};
                this._gatherRawIngredients(recipe, 1, rawMap, new Set());
                this.recipeRawMap[group.recipeName] = rawMap;
            }

            const currentQty = this.getGroupRecipeQty(group);
            const newQty = Math.max(1, currentQty + delta);

            if (newQty !== currentQty) {
                for (const item of group.items) {
                    const perOne = rawMap[item.item_name] || 0;
                    if (perOne > 0) {
                        this.setShoppingItemQty(item, Math.ceil(newQty * perOne));
                    }
                }
            }
        },

        setShoppingItemQty(item, qty) {
            const newQty = Math.max(1, qty);
            item.quantity = newQty;
            fetch(`https://${GetParentResourceName()}/list:qty`, {
                method: "POST",
                body: JSON.stringify({ id: item.id, quantity: newQty })
            }).catch(() => {});
        },

        /* ·· Menu open / close ····························· */

        closeMenu() {
            this.hoveredIng = null;
            this.isVisible = false;
            this.view = "grid";
            this.selectedRecipe = null;
            this.selectedCategory = "all";
            this.searchText = "";

            fetch(`https://${GetParentResourceName()}/close`, {
                method: "POST"
            }).catch(() => {});
        },

        // The recipe list steps aside while a craft plays out, then comes back.
        animationPlaying(duration) {
            const ms = Number(duration) || this.crafttime;
            this.isVisible = false;
            this.progressTimers.push(setTimeout(() => {
                this.isVisible = true;
            }, ms));
        },

        /* ·· Progress bar ·································· */

        showProgress(data) {
            this.clearProgressTimers();
            const ms = Number(data.duration) || 5000;
            this.progress = {
                shown: true,
                label: data.label || this.t("crafting"),
                duration: ms,
                position: data.position === "top" ? "top" : "bottom",
                fill: 0
            };
            // Two frames: one to paint at zero width, one to start the
            // transition. Setting both in the same tick gives no animation.
            requestAnimationFrame(() => requestAnimationFrame(() => {
                this.progress.fill = 100;
            }));
            // Self-hiding, so a dropped progressEnd cannot strand the bar.
            this.progressTimers.push(setTimeout(() => this.hideProgress(), ms + 400));
        },

        hideProgress() {
            this.clearProgressTimers();
            this.progress.shown = false;
            this.progress.fill = 0;
        },

        clearProgressTimers() {
            this.progressTimers.forEach(clearTimeout);
            this.progressTimers = [];
        },

        /* ·· Translation ··································· */

        // Falls back to the key, which reads badly on screen but says exactly
        // which line of translations.lua is missing.
        t(key, ...args) {
            let text = this.language[key];
            if (text === undefined) return key;
            args.forEach(a => { text = text.replace("%s", a); });
            return text;
        },

        /* ·· Image preloading & caching ····················
         *  Uses localStorage to remember which item images exist
         *  vs. need the placeholder. Survives server restarts so
         *  subsequent opens skip the Image() probe entirely.
         * ·················································· */

        _loadLocalImageMap() {
            try {
                // Only confirmed-found (1) entries are stored; a missing image is
                // never persisted, so it re-probes each session and picks up an
                // icon the owner added since, with no cache to clear.
                const raw = localStorage.getItem(this.imageCacheKey);
                return raw ? JSON.parse(raw) : {};
            } catch { return {}; }
        },

        _saveLocalImageMap(map) {
            try { localStorage.setItem(this.imageCacheKey, JSON.stringify(map)); }
            catch { /* storage full — not worth caring about */ }
        },

        _buildUrl(name, isPlaceholder) {
            return isPlaceholder
                ? "url('img/placeholder.png')"
                : `url('${this.imageBase}${encodeURIComponent(name)}.png')`;
        },

        _doPreloadItemImage(name, retries = 1) {
            return new Promise(resolve => {
                const imgPath = `${this.imageBase}${encodeURIComponent(name)}.png`;
                const img = new Image();
                img.onload = () => {
                    this.imageCache[name] = this._buildUrl(name, false);
                    const map = this._loadLocalImageMap();
                    map[name] = 1;
                    this._saveLocalImageMap(map);
                    resolve();
                };
                img.onerror = () => {
                    if (retries > 0) {
                        // Retry once — nui:// probes can fail transiently for images
                        // that are new to the browser cache even though they exist.
                        setTimeout(() => {
                            this._doPreloadItemImage(name, 0).then(resolve);
                        }, 500);
                        return;
                    }
                    // Still missing after retry — use placeholder in-memory only.
                    // Do NOT persist to localStorage; re-probe next session so
                    // newly added item images are picked up automatically.
                    this.imageCache[name] = this._buildUrl(name, true);
                    resolve();
                };
                img.src = imgPath;
            });
        },

        preloadItemImage(name) {
            if (!name || this.imageCache[name]) return Promise.resolve();

            // Check localStorage for confirmed-found images only (v2 never stores missing).
            const map = this._loadLocalImageMap();
            if (map[name] === 1) {
                this.imageCache[name] = this._buildUrl(name, false);
                return Promise.resolve();
            }

            const promise = this._doPreloadItemImage(name);
            this.imageCache[name] = promise;  // store promise as placeholder until resolved
            return promise;
        },

        async preloadAllItemImages(craftables) {
            const names = new Set();
            craftables.forEach(recipe => {
                (recipe.Reward || []).forEach(rw => names.add(rw.name));
                (recipe.Items  || []).forEach(ing => {
                    names.add(ing.name);
                    (ing.AltNames || []).forEach(n => names.add(n));
                });
            });
            await Promise.allSettled([...names].map(name => this.preloadItemImage(name)));
        },

        getItemImageUrl(name) {
            const cached = this.imageCache[name];
            if (typeof cached === "string") return cached;
            // Probe in-flight (Promise) or not yet started: return direct URL so the
            // browser loads it immediately — same approach the chain-node uses.
            // This prevents showing the TMP placeholder during the probe window.
            return name
                ? `url('${this.imageBase}${encodeURIComponent(name)}.png')`
                : "url('img/placeholder.png')";
        },

        /** Call this to force all images to re-probe (e.g. after adding new item images) */
        clearImageCache() {
            this.imageCache = {};
            try { localStorage.removeItem(this.imageCacheKey); } catch {}
        },

        /* ·· Label / display helpers ······················· */

        getItemDisplayLabel(name) {
            return this.itemLabels[name] || name;
        },

        recipeLabel(recipe) {
            if (!recipe) return "";
            const rewardName = (recipe.Reward && recipe.Reward[0]) ? recipe.Reward[0].name : "";
            return (rewardName && this.itemLabels[rewardName]) || recipe.Text || "";
        },

        recipeNameToLabel(recipeName) {
            const recipe = this.allCraftablesUnfiltered.find(r => r.Text === recipeName);
            return recipe ? this.recipeLabel(recipe) : recipeName;
        },

        /* ·· Item source tooltip ··························· */

        getItemSource(name) {
            if (this.itemSources && this.itemSources[name]) {
                return this.itemSources[name];
            }
            return { source: this.t("ui_source_unknown"), description: this.t("ui_source_none") };
        },

        showIngTooltip(name, event) {
            const rect = event.currentTarget.getBoundingClientRect();
            this.hoveredIng = {
                name,
                source: this.getItemSource(name)
            };
            this.tooltipPos = {
                x: Math.min(rect.left + rect.width / 2, window.innerWidth - 240),
                y: rect.top - 8
            };
        },

        hideIngTooltip() {
            this.hoveredIng = null;
        },

        /* ·· Data initialization ··························· */

        async setData(payload) {
            const {
                craftables  = [],
                categories  = [],
                crafttime   = 15000,
                style       = { fontSize: "m" },
                language    = {},
                location    = {},
                job         = null,
                itemLabels  = {},
                itemLimits  = {},
                inventory   = {},
                itemSources = {},
                imageBase   = "",
                shoppingList = true
            } = payload;

            if (imageBase) this.imageBase = imageBase;
            this.shoppingListEnabled = shoppingList !== false;

            // Determine which categories are accessible to this player
            const accessibleCats = [];
            const accessibleRecipes = [];

            categories.forEach(cat => {
                const jobOk = cat.Job === 0 || (Array.isArray(cat.Job) && cat.Job.includes(job));
                const locOk = cat.Location == 0 || (Array.isArray(cat.Location) && cat.Location.includes(location?.id));
                if (jobOk && locOk) accessibleCats.push(cat);
            });

            const accessibleIdents = new Set(accessibleCats.map(c => c.ident));

            // Mark each recipe as locked or unlocked. This only decides what
            // the browser greys out -- the server asks the same questions again
            // before it takes anything, so a locked recipe cannot be crafted by
            // editing this page.
            craftables.forEach(recipe => {
                const hasJobSkillcheck = recipe.jobSkillcheck > 0;
                const jobOk = recipe.Job === 0 || (Array.isArray(recipe.Job) && recipe.Job.includes(job)) || hasJobSkillcheck;
                const locOk = recipe.Location == 0 || (Array.isArray(recipe.Location) && recipe.Location.includes(location?.id));
                const accessible = jobOk && locOk && accessibleIdents.has(recipe.Category);
                recipe._locked = !accessible;
                if (accessible) accessibleRecipes.push(recipe);
            });

            // Build category colour map
            const colorMap = {};
            const palette = [
                "#ffe5b4", "#b4e1ff", "#e0b4ff", "#ffb4b4",
                "#ffd6b4", "#b4ffd6", "#b4fff7", "#e6e6e6",
                "#f7e6b4", "#e6b4f7", "#f7b4e6", "#b4f7e6"
            ];
            categories.forEach((cat, idx) => {
                const color = palette[idx % 12];
                colorMap[cat.ident] = color;
                if (cat.text && cat.text !== cat.ident) {
                    colorMap[cat.text] = color;
                }
            });

            // Apply all state
            this.job = job;
            this.language = language;
            this.location = location;
            this.categories = categories
                .map(cat => ({
                    ...cat,
                    text: cat.text || cat.ident,
                    _accessible: accessibleIdents.has(cat.ident)
                }))
                .sort((a, b) => {
                    if (a._accessible !== b._accessible) return a._accessible ? -1 : 1;
                    return (a.text || "").localeCompare(b.text || "");
                });
            this.allCraftables = accessibleRecipes;
            this.allCraftablesUnfiltered = craftables;
            this.categoryColorMap = colorMap;
            this.itemLabels = itemLabels;
            this.itemLimits = itemLimits;
            this.itemSources = itemSources;
            this.inventory = inventory;
            this.style = style;
            this.crafttime = crafttime;

            // Preload images (cache persists across menu opens)
            await this.preloadAllItemImages(craftables);
        }
    }
});

app.component("chain-node", ChainNodeComponent);
app.mount("#app");