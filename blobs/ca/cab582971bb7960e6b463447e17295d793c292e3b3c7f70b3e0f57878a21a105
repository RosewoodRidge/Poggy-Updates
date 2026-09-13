/* ============================================================================
   Poggy Transform — NUI Script (Categorized)
   ============================================================================ */

const app            = document.getElementById('app');
const grid           = document.getElementById('animalGrid');
const revertSec      = document.getElementById('revertSection');
const revertBtn      = document.getElementById('revertBtn');
const tabsEl         = document.getElementById('categoryTabs');
const searchInput    = document.getElementById('searchInput');
const customPedSec   = document.getElementById('customPedSection');
const customPedInput = document.getElementById('customPedInput');
const customPedBtn   = document.getElementById('customPedBtn');

// Emote menu elements
const emoteMenu          = document.getElementById('emote-menu');
const emoteList          = document.getElementById('emote-list');
const emoteAnimalImg     = document.getElementById('emote-animal-img');
const emoteAnimalFallback = document.getElementById('emote-animal-fallback');
const emoteAnimalName    = document.getElementById('emote-animal-name');
const emoteStopBtn       = document.getElementById('emote-stop-btn');

let isOpen          = false;
let allAnimals      = [];
let allPlayerSkins  = [];   // [{ charidentifier, firstname, lastname }]
let categories      = [];
let activeTab       = 'all';
let activeMode      = 'animals';  // 'animals' | 'players'
let isEmoteMenuOpen = false;

// ============================================================================
// Message listener
// ============================================================================
window.addEventListener('message', (event) => {
    const msg = event.data;

    if (msg.type === 'open') {
        openUI(msg.animals || [], msg.isAdmin || false, msg.transformed || false, msg.playerSkins || []);
    } else if (msg.type === 'close') {
        closeUI();
    } else if (msg.type === 'openEmoteMenu') {
        openEmoteMenu(msg.label, msg.image, msg.emotes || []);
    } else if (msg.type === 'closeEmoteMenu') {
        closeEmoteMenuLocal();
    }
});

// ============================================================================
// Open UI
// ============================================================================
function openUI(animals, isAdmin, isTransformed, playerSkins) {
    isOpen         = true;
    allAnimals     = animals;
    allPlayerSkins = playerSkins || [];
    activeMode     = 'animals';

    // Derive categories from the animal data
    categories = buildCategories(animals);
    activeTab  = 'all';

    // Build tabs (includes Players tab if there are player skins)
    buildTabs();

    // Build grid
    buildGrid(animals);

    // Show/hide revert button
    if (isTransformed) {
        revertSec.classList.remove('hidden');
    } else {
        revertSec.classList.add('hidden');
    }

    // Show custom ped input only for admins
    if (isAdmin) {
        customPedSec.classList.remove('hidden');
        customPedInput.value = '';
    } else {
        customPedSec.classList.add('hidden');
    }

    // Reset search
    searchInput.value = '';

    // Show
    app.classList.remove('hidden');
    app.classList.remove('app-closing');

    // Focus search after a beat
    setTimeout(() => searchInput.focus(), 300);
}

// ============================================================================
// Close UI
// ============================================================================
function closeUI() {
    if (!isOpen) return;
    isOpen = false;

    app.classList.add('app-closing');
    setTimeout(() => {
        app.classList.add('hidden');
        app.classList.remove('app-closing');
        grid.innerHTML  = '';
        tabsEl.innerHTML = '';
        allPlayerSkins  = [];
        activeMode      = 'animals';
    }, 250);
}

// ============================================================================
// ESC key handler
// ============================================================================
document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
        if (isEmoteMenuOpen) {
            // Close emote menu first; Lua will release NUI focus
            closeEmoteMenuLocal();
            fetch(`https://${GetParentResourceName()}/closeEmoteMenu`, {
                method: 'POST',
                body: JSON.stringify({}),
            });
        } else if (isOpen) {
            fetch(`https://${GetParentResourceName()}/close`, {
                method: 'POST',
                body: JSON.stringify({}),
            });
        }
    }
});

// ============================================================================
// Revert button
// ============================================================================
revertBtn.addEventListener('click', () => {
    fetch(`https://${GetParentResourceName()}/revert`, {
        method: 'POST',
        body: JSON.stringify({}),
    });
});

// ============================================================================
// Custom ped: trigger on button click or Enter key
// ============================================================================
function triggerCustomPedTransform() {
    const model = (customPedInput.value || '').trim();
    if (!model) return;

    customPedBtn.disabled = true;
    customPedBtn.textContent = 'Loading...';

    fetch(`https://${GetParentResourceName()}/transformCustomPed`, {
        method: 'POST',
        body: JSON.stringify({ model }),
    }).finally(() => {
        customPedBtn.disabled = false;
        customPedBtn.textContent = 'Transform';
    });
}

customPedBtn.addEventListener('click', triggerCustomPedTransform);

customPedInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') triggerCustomPedTransform();
});

// ============================================================================
// Search handler
// ============================================================================
searchInput.addEventListener('input', () => {
    applyFilters();
});

// ============================================================================
// Build categories from animal data
// ============================================================================
function buildCategories(animals) {
    // Known category order & meta (matches Config.Categories in Lua)
    const meta = {
        predator:  { label: 'Predators',        icon: '🐺' },
        prey:      { label: 'Prey & Wildlife',  icon: '🦌' },
        bird:      { label: 'Birds',            icon: '🦅' },
        aquatic:   { label: 'Aquatic & Reptile', icon: '🐊' },
        legendary: { label: 'Legendary',        icon: '⭐' },
        people:    { label: 'People',           icon: '🤠' },
    };
    const order = ['predator', 'prey', 'bird', 'aquatic', 'legendary', 'people'];

    // Collect categories that actually have animals
    const found = new Set(animals.map(a => a.category));
    const cats  = [];

    for (const id of order) {
        if (found.has(id)) {
            const count = animals.filter(a => a.category === id).length;
            cats.push({ id, label: meta[id]?.label || id, icon: meta[id]?.icon || '', count });
        }
    }

    // Catch any unknown categories
    for (const id of found) {
        if (!order.includes(id)) {
            const count = animals.filter(a => a.category === id).length;
            cats.push({ id, label: id, icon: '❓', count });
        }
    }

    return cats;
}

// ============================================================================
// Build Tabs
// ============================================================================
function buildTabs() {
    tabsEl.innerHTML = '';

    // "All" tab
    const allTab = createTabEl('all', '🌀', 'All', allAnimals.length);
    allTab.classList.add('active');
    tabsEl.appendChild(allTab);

    // Category tabs
    for (const cat of categories) {
        tabsEl.appendChild(createTabEl(cat.id, cat.icon, cat.label, cat.count));
    }

    // "Players" tab — only shown when there is player skin data
    if (allPlayerSkins.length > 0) {
        const playersTab = createTabEl('__players__', '👤', 'Players', allPlayerSkins.length);
        tabsEl.appendChild(playersTab);
    }
}

function createTabEl(id, icon, label, count) {
    const tab = document.createElement('button');
    tab.className = 'tab';
    tab.dataset.category = id;
    tab.innerHTML = `
        <span class="tab-icon">${icon}</span>
        <span>${label}</span>
        <span class="tab-count">${count}</span>
    `;
    tab.addEventListener('click', () => {
        // Update active class
        tabsEl.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        tab.classList.add('active');

        if (id === '__players__') {
            activeMode = 'players';
            activeTab  = '__players__';
        } else {
            activeMode = 'animals';
            activeTab  = id;
        }
        applyFilters();
    });
    return tab;
}

// ============================================================================
// Apply tab + search filter
// ============================================================================
function applyFilters() {
    const query = (searchInput.value || '').toLowerCase().trim();

    // --- Player Skins mode ---
    if (activeMode === 'players') {
        let filtered = allPlayerSkins;
        if (query) {
            filtered = filtered.filter(p => {
                const full = (p.firstname + ' ' + p.lastname).toLowerCase();
                return full.includes(query) ||
                    p.firstname.toLowerCase().includes(query) ||
                    p.lastname.toLowerCase().includes(query);
            });
        }
        buildPlayerSkinGrid(filtered);
        return;
    }

    // --- Animals mode ---
    let filtered = allAnimals;

    // Category filter
    if (activeTab !== 'all') {
        filtered = filtered.filter(a => a.category === activeTab);
    }

    // Search filter
    if (query) {
        filtered = filtered.filter(a =>
            a.label.toLowerCase().includes(query) ||
            a.model.toLowerCase().includes(query) ||
            (a.id && a.id.toLowerCase().includes(query))
        );
    }

    buildGrid(filtered);
}

// ============================================================================
// Build Player Skin Grid
// ============================================================================
function buildPlayerSkinGrid(skins) {
    grid.innerHTML = '';

    if (!skins || skins.length === 0) {
        grid.innerHTML = '<div class="no-animals">No characters match your search.</div>';
        return;
    }

    skins.forEach((skin, index) => {
        const initials = (skin.firstname[0] + skin.lastname[0]).toUpperCase();

        const card = document.createElement('div');
        card.className = 'animal-card people-card player-skin-card';

        card.innerHTML = `
            <div class="card-image">
                <span class="badge player-badge">👤 Player</span>
                <div class="player-skin-avatar">${initials}</div>
            </div>
            <div class="card-body">
                <div class="card-name">${skin.firstname} ${skin.lastname}</div>
                <div class="card-model" style="font-size:10px;opacity:0.5">#${skin.charidentifier}</div>
            </div>
        `;

        card.addEventListener('click', () => {
            fetch(`https://${GetParentResourceName()}/transformPlayerSkin`, {
                method : 'POST',
                body   : JSON.stringify({ charidentifier: skin.charidentifier }),
            });
        });

        grid.appendChild(card);
    });
}

// ============================================================================
// Build Grid
// ============================================================================
function buildGrid(animals) {
    grid.innerHTML = '';

    if (!animals || animals.length === 0) {
        grid.innerHTML = '<div class="no-animals">No transformations match your filter.</div>';
        return;
    }

    animals.forEach((animal, index) => {
        const isLegendary = animal.category === 'legendary';
        const isPeople    = animal.category === 'people';

        // Card classes
        let cardClasses = 'animal-card';
        if (isLegendary)       cardClasses += ' legendary';
        else if (isPeople)     cardClasses += ' people-card';
        else if (animal.aggressive) cardClasses += ' aggressive';
        else                   cardClasses += ' passive';

        const card = document.createElement('div');
        card.className = cardClasses;

        // Badge
        let badgeHTML = '';
        if (isLegendary) {
            badgeHTML = `<span class="badge legendary-badge"><span class="star">⭐</span> Legendary</span>`;
        } else if (isPeople) {
            badgeHTML = `<span class="badge people-badge">NPC</span>`;
        } else if (animal.aggressive) {
            badgeHTML = `<span class="badge aggressive">Aggressive</span>`;
        } else {
            badgeHTML = `<span class="badge passive">Passive</span>`;
        }

        // Scale badge (for legendaries with scale)
        let scaleHTML = '';
        if (animal.scale && animal.scale > 1.0) {
            scaleHTML = `<span class="badge passive" style="top:auto;bottom:10px;right:10px;font-size:9px;">×${animal.scale}</span>`;
        }

        // Image with fallback
        const fallbackIcon = isPeople ? '🤠' : '🐾';
        const imageHTML = animal.image
            ? `<img src="images/${animal.image}" alt="${animal.label}" onerror="this.style.display='none'; this.nextElementSibling.style.display='flex';">
               <div class="fallback-icon" style="display:none;">${fallbackIcon}</div>`
            : `<div class="fallback-icon" style="display:flex;">${fallbackIcon}</div>`;

        card.innerHTML = `
            <div class="card-image">
                ${badgeHTML}
                ${scaleHTML}
                ${imageHTML}
            </div>
            <div class="card-body">
                <div class="card-name">${animal.label}</div>
                <div class="card-model">${animal.model}</div>
            </div>
        `;

        // Click → transform
        card.addEventListener('click', () => {
            fetch(`https://${GetParentResourceName()}/transform`, {
                method: 'POST',
                body: JSON.stringify({ id: animal.id }),
            });
        });

        grid.appendChild(card);
    });
}


// ============================================================================
// Emote Menu — open
// ============================================================================
function openEmoteMenu(label, image, emotes) {
    isEmoteMenuOpen = true;

    // Set animal image / fallback
    if (image) {
        emoteAnimalImg.src = 'images/' + image;
        emoteAnimalImg.style.display = 'block';
        emoteAnimalFallback.style.display = 'none';
    } else {
        emoteAnimalImg.style.display = 'none';
        emoteAnimalFallback.style.display = 'flex';
    }

    emoteAnimalName.textContent = label || 'Animal';

    // Build emote buttons
    emoteList.innerHTML = '';
    emotes.forEach(emote => {
        const btn = document.createElement('button');
        btn.className = 'emote-btn';
        btn.innerHTML = `
            <span>${emote.label}</span>
            ${emote.loop ? '<span class="emote-loop-tag">loop</span>' : ''}
        `;
        btn.addEventListener('click', () => {
            fetch(`https://${GetParentResourceName()}/playEmote`, {
                method: 'POST',
                body: JSON.stringify({ cmd: emote.cmd }),
            });
            closeEmoteMenuLocal();
        });
        emoteList.appendChild(btn);
    });

    emoteMenu.classList.remove('hidden');
    emoteMenu.classList.remove('emote-closing');
}

// ============================================================================
// Emote Menu — close (local DOM only — Lua handles NUI focus separately)
// ============================================================================
function closeEmoteMenuLocal() {
    if (!isEmoteMenuOpen) return;
    isEmoteMenuOpen = false;
    emoteMenu.classList.add('emote-closing');
    setTimeout(() => {
        emoteMenu.classList.add('hidden');
        emoteMenu.classList.remove('emote-closing');
        emoteList.innerHTML = '';
    }, 200);
}

// ============================================================================
// Stop emote button
// ============================================================================
emoteStopBtn.addEventListener('click', () => {
    fetch(`https://${GetParentResourceName()}/stopEmote`, {
        method: 'POST',
        body: JSON.stringify({}),
    });
    closeEmoteMenuLocal();
});
