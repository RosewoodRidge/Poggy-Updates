/* ================================================================
   poggy_emotes - the menu page.

   The whole list arrives once when the menu opens. Searching, filtering
   and starring all happen here, so nothing has to be rebuilt in Lua and
   the menu never has to close and reopen to stay in step. That is what
   the old script did, and it is why a star click used to rebuild the
   entire list and spawn another control-blocking thread.
   ================================================================ */

(function () {
  'use strict';

  var state = {
    emotes: [],
    categories: [],
    recents: [],
    nearby: [],
    strings: {},
    options: { favourites: true, preview: true },
    category: null,
    query: '',
    cursor: -1,
    visible: [],
    playing: null,
    propsOnly: false,
    open: false,
    confirming: null,  // the nearby row whose block question is showing
    hover: true        // the mouse is over the panel
  };

  var el = {
    root: document.getElementById('root'),
    title: document.getElementById('title'),
    count: document.getElementById('count'),
    search: document.getElementById('search'),
    clear: document.getElementById('btn-clear'),
    props: document.getElementById('btn-props'),
    keep: document.getElementById('chk-keep'),
    keepText: document.getElementById('keep-text'),
    close: document.getElementById('btn-close'),
    stop: document.getElementById('btn-stop'),
    hint: document.getElementById('hint'),
    categories: document.getElementById('categories'),
    list: document.getElementById('list'),
    empty: document.getElementById('empty')
  };

  // ── Talking to the game ───────────────────────────────────────

  function post(name, data) {
    return fetch('https://' + GetParentResourceName() + '/' + name, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json; charset=UTF-8' },
      body: JSON.stringify(data || {})
    }).catch(function () { /* the menu closed under us; nothing to do */ });
  }

  // ── Which emotes to show ──────────────────────────────────────

  function matches(emote, query) {
    if (!query) return true;
    if (emote.nearby) {
      return emote.label.toLowerCase().indexOf(query) !== -1 ||
             (emote.group || '').toLowerCase().indexOf(query) !== -1;
    }
    return emote.label.toLowerCase().indexOf(query) !== -1 ||
           emote.name.indexOf(query) !== -1;
  }

  function computeVisible() {
    var query = state.query.trim().toLowerCase();
    var rows;

    if (query) {
      // A search looks everywhere. Narrowing it to the open category is the
      // single most annoying thing an emote menu can do.
      rows = state.nearby.filter(function (e) { return matches(e, query); })
        .concat(state.emotes.filter(function (e) { return matches(e, query); }));
    } else if (state.category === 'nearby') {
      rows = state.nearby;
    } else if (state.category === 'favourites') {
      rows = state.emotes.filter(function (e) { return e.favourite; });
    } else if (state.category === 'recent') {
      rows = state.recents.map(function (name) {
        for (var i = 0; i < state.emotes.length; i++) {
          if (state.emotes[i].name === name) return state.emotes[i];
        }
        return null;
      }).filter(Boolean);
    } else {
      rows = state.emotes.filter(function (e) { return e.category === state.category; });
    }

    // The props filter narrows whatever is showing: a category, or a search.
    if (state.propsOnly) rows = rows.filter(function (e) { return e.prop; });

    state.visible = rows;
  }

  // ── Drawing ───────────────────────────────────────────────────

  function highlight(text, query) {
    if (!query) return document.createTextNode(text);

    var at = text.toLowerCase().indexOf(query);
    if (at === -1) return document.createTextNode(text);

    var frag = document.createDocumentFragment();
    frag.appendChild(document.createTextNode(text.slice(0, at)));
    var mark = document.createElement('mark');
    mark.textContent = text.slice(at, at + query.length);
    frag.appendChild(mark);
    frag.appendChild(document.createTextNode(text.slice(at + query.length)));
    return frag;
  }

  function drawCategories() {
    el.categories.textContent = '';

    state.categories.forEach(function (cat) {
      var button = document.createElement('button');
      button.type = 'button';
      button.className = 'cat' + (cat.key === state.category && !state.query ? ' active' : '');
      button.textContent = cat.label;
      button.addEventListener('click', function () {
        state.category = cat.key;
        state.query = '';
        el.search.value = '';
        el.clear.classList.add('hidden');
        state.cursor = -1;
        draw();
        tabChanged();
      });
      el.categories.appendChild(button);
    });
  }

  function drawList() {
    var query = state.query.trim().toLowerCase();
    el.list.textContent = '';

    var lastGroup = null;
    var grouped = state.category === 'nearby' && !query;

    state.visible.forEach(function (emote, index) {
      if (emote.nearby && grouped && emote.groupKey !== lastGroup) {
        // A heading per thing: "Chair · 1.2 m", with its actions under it.
        lastGroup = emote.groupKey;
        var head = document.createElement('li');
        head.className = 'group-head';
        var headName = document.createElement('span');
        headName.className = 'group-name';
        headName.textContent = emote.group || '';
        head.appendChild(headName);
        var headDist = document.createElement('span');
        headDist.className = 'group-dist';
        headDist.textContent = fmtDist(emote.dist);
        head.appendChild(headDist);
        el.list.appendChild(head);
      }

      var row = document.createElement('li');
      row.className = 'emote';
      if (index === state.cursor) row.className += ' cursor';
      if (emote.name === state.playing) row.className += ' playing';
      row.dataset.name = emote.name;

      if (emote.nearby) {
        row.classList.add('nearby-row');
        if (grouped) row.classList.add('indent');
        if (emote.blocked) row.classList.add('blocked');

        var mark = document.createElement('span');
        mark.className = 'walk';
        mark.textContent = emote.blocked ? '×' : '›';
        row.appendChild(mark);

        var nlabel = document.createElement('span');
        nlabel.className = 'emote-label';
        nlabel.appendChild(highlight(emote.label, query));
        row.appendChild(nlabel);

        if (emote.chain) {
          var ctag = document.createElement('span');
          ctag.className = 'tag tag-chain';
          ctag.textContent = state.strings.nearbyChain || 'carry';
          ctag.title = state.strings.nearbyChainTip || '';
          row.appendChild(ctag);
        }

        if (emote.blocked) {
          var btag = document.createElement('span');
          btag.className = 'tag tag-blocked';
          btag.textContent = state.strings.nearbyBlocked || 'Blocked';
          row.appendChild(btag);
        }

        if (state.options.dev && emote.scenario) {
          if (state.confirming === emote.name) {
            // The question sits in the row itself, so there is nothing to lose track of.
            var ask = document.createElement('span');
            ask.className = 'confirm';
            var askText = document.createElement('span');
            askText.className = 'confirm-text';
            askText.textContent = (state.strings.nearbyBlockAsk || "Block '%s' for everyone?").replace('%s', emote.label);
            ask.appendChild(askText);
            if (emote.blocked) {
              var back = document.createElement('button');
              back.type = 'button';
              back.className = 'confirm-yes';
              back.textContent = state.strings.nearbyUnblock || 'Allow again';
              back.addEventListener('click', function (event) {
                event.stopPropagation();
                state.confirming = null;
                post('nearbyBlock', { scenario: emote.scenario, on: false, x: emote.x, y: emote.y, z: emote.z });
              });
              ask.appendChild(back);
            } else {
              // Here only is first: the usual case is one broken spot, not a broken scenario.
              var here = document.createElement('button');
              here.type = 'button';
              here.className = 'confirm-yes';
              here.textContent = state.strings.nearbyBlockHere || 'Here only';
              here.addEventListener('click', function (event) {
                event.stopPropagation();
                state.confirming = null;
                post('nearbyBlock', { scenario: emote.scenario, on: true, everywhere: false, x: emote.x, y: emote.y, z: emote.z });
              });
              ask.appendChild(here);
              var all = document.createElement('button');
              all.type = 'button';
              all.className = 'confirm-all';
              all.textContent = state.strings.nearbyBlockAll || 'Everywhere';
              all.addEventListener('click', function (event) {
                event.stopPropagation();
                state.confirming = null;
                post('nearbyBlock', { scenario: emote.scenario, on: true, everywhere: true });
              });
              ask.appendChild(all);
            }
            var no = document.createElement('button');
            no.type = 'button';
            no.className = 'confirm-no';
            no.textContent = state.strings.nearbyBlockNo || 'Cancel';
            no.addEventListener('click', function (event) {
              event.stopPropagation();
              state.confirming = null;
              drawList();
            });
            ask.appendChild(no);
            row.appendChild(ask);
          } else {
            var x = document.createElement('button');
            x.type = 'button';
            x.className = 'block' + (emote.blocked ? ' on' : '');
            x.textContent = emote.blocked ? '↺' : '×';
            x.title = emote.blocked ? (state.strings.nearbyUnblock || 'Allow again') : (state.strings.nearbyBlock || 'Block everywhere');
            x.addEventListener('click', function (event) {
              event.stopPropagation();
              state.confirming = emote.name;
              drawList();
            });
            row.appendChild(x);
          }
        }

        if (!grouped) {
          var where = document.createElement('span');
          where.className = 'tag tag-nearby';
          where.textContent = emote.group || (state.strings.nearbyTag || 'Nearby');
          row.appendChild(where);
          var d = document.createElement('span');
          d.className = 'emote-cmd';
          d.textContent = fmtDist(emote.dist);
          row.appendChild(d);
        }

        row.addEventListener('click', function () { if (!emote.blocked || state.options.dev) playNearby(emote); });
        row.addEventListener('mouseenter', function () { state.cursor = index; markCursor(); });
        el.list.appendChild(row);
        return;
      }

      if (state.options.preview) {
        // The button says out loud what hovering does quietly: watch it first.
        // It plays on the stand-in, never on the player, and keeps the menu open.
        var watch = document.createElement('button');
        watch.type = 'button';
        watch.className = 'watch';
        watch.textContent = '▶';
        watch.title = state.strings.preview || 'Preview';
        watch.addEventListener('click', function (event) {
          event.stopPropagation();
          state.cursor = index;
          markCursor();
          previewNow(emote);
          flash(watch);
        });
        row.appendChild(watch);
      }

      var label = document.createElement('span');
      label.className = 'emote-label';
      label.appendChild(highlight(emote.label, query));
      row.appendChild(label);

      if (emote.prop) {
        var propTag = document.createElement('span');
        propTag.className = 'tag tag-prop';
        propTag.textContent = state.strings.prop || 'Prop';
        row.appendChild(propTag);
      }

      var cmd = document.createElement('span');
      cmd.className = 'emote-cmd';
      cmd.textContent = '/e ' + emote.name;
      row.appendChild(cmd);

      if (state.options.favourites) {
        var star = document.createElement('button');
        star.type = 'button';
        star.className = 'star' + (emote.favourite ? ' on' : '');
        star.textContent = emote.favourite ? '★' : '☆';
        star.title = emote.favourite
          ? (state.strings.favouriteDrop || 'Remove from favourites')
          : (state.strings.favouriteAdd || 'Add to favourites');
        star.addEventListener('click', function (event) {
          // Without this the click falls through to the row and plays it.
          event.stopPropagation();
          toggleFavourite(emote);
        });
        row.appendChild(star);
      }

      row.addEventListener('click', function () { play(emote); });
      row.addEventListener('mouseenter', function () {
        state.cursor = index;
        markCursor();
        preview(emote);
      });

      el.list.appendChild(row);
    });

    var nothing = state.visible.length === 0;
    el.empty.classList.toggle('hidden', !nothing);
    if (nothing) {
      if (state.category === 'nearby' && !state.query) el.empty.textContent = state.strings.noNearby || 'Nothing to do around here.';
      else if (state.propsOnly && !state.query) el.empty.textContent = state.strings.noProps || 'No prop emotes here.';
      else if (state.query) el.empty.textContent = state.strings.noResults || 'No emote matches that.';
      else if (state.category === 'favourites') el.empty.textContent = state.strings.noFavourites || '';
      else if (state.category === 'recent') el.empty.textContent = state.strings.noRecent || '';
      else el.empty.textContent = state.strings.noResults || '';
    }

    var template = state.strings.count || '%s emotes';
    el.count.textContent = template.replace('%s', String(state.visible.length));
    el.hint.textContent = (state.category === 'nearby' && !query)
      ? (state.strings.hintNearby || '')
      : (state.strings.hintKeys || '');
  }

  function fmtDist(d) {
    if (d === undefined || d === null || d < 0.05) return '';
    return d.toFixed(1) + ' m';
  }

  // Nearby rows come in flat, in display order, each with the thing it belongs to.
  function takeNearby(list) {
    state.nearby = (list || []).map(function (r) {
      return {
        // The name is the thing plus the action, so the cursor can follow a row
        // through a refresh even when its position in the list has changed.
        name: 'nearby:' + (r.groupKey || '') + ':' + (r.label || ''), id: r.id, label: r.label || '',
        group: r.group || '', groupKey: r.groupKey || '', dist: r.dist, scenario: r.scenario || null,
        x: r.x, y: r.y, z: r.z, chain: r.chain === true,
        blocked: r.blocked === true, category: 'nearby', nearby: true, favourite: false
      };
    });
  }

  // A fresh Nearby list while it is showing: keep the cursor on the same row
  // and the list where it was scrolled to, so walking about does not jolt it.
  function refreshNearby() {
    var keep = state.cursor >= 0 && state.visible[state.cursor] ? state.visible[state.cursor].name : null;
    var pane = el.list.parentNode;
    var top = pane ? pane.scrollTop : 0;
    computeVisible();
    state.cursor = -1;
    if (keep) {
      for (var i = 0; i < state.visible.length; i++) {
        if (state.visible[i].name === keep) { state.cursor = i; break; }
      }
    }
    if (state.confirming) {
      var still = false;
      for (var j = 0; j < state.visible.length; j++) if (state.visible[j].name === state.confirming) still = true;
      if (!still) state.confirming = null;
    }
    drawList();
    if (pane) pane.scrollTop = top;
    lastHighlight = undefined;   // the ids were renumbered by the scan
    highlightThing();
  }

  // Moves the highlight without redrawing every row: a full redraw on each
  // arrow key press loses the scroll position and flickers.
  function markCursor() {
    var rows = el.list.querySelectorAll('.emote');
    for (var i = 0; i < rows.length; i++) {
      rows[i].classList.toggle('cursor', i === state.cursor);
    }
    if (state.cursor >= 0 && rows[state.cursor]) {
      rows[state.cursor].scrollIntoView({ block: 'nearest' });
    }
    highlightThing();
  }

  // Tells the game which Nearby row is under the cursor, so it can mark the
  // chair or the barrel that row belongs to.
  var lastHighlight = null;
  function highlightThing() {
    var row = onNearby() && state.cursor >= 0 ? state.visible[state.cursor] : null;
    var id = row && row.nearby ? row.id : null;
    if (id === lastHighlight) return;
    lastHighlight = id;
    post('highlight', { id: id });
  }

  function draw() {
    computeVisible();
    drawCategories();
    drawList();
  }

  // ── Actions ───────────────────────────────────────────────────

  var previewTimer = null;

  function preview(emote) {
    if (!state.options.preview || !emote || emote.nearby) return;
    // Waiting a moment stops a fast scroll down the list from firing twenty
    // animations at the preview character in a row.
    if (previewTimer) clearTimeout(previewTimer);
    previewTimer = setTimeout(function () {
      post('preview', { name: emote.name });
    }, 90);
  }

  // A click on the preview button should not wait like a hover does.
  function previewNow(emote) {
    if (!state.options.preview) return;
    if (previewTimer) clearTimeout(previewTimer);
    post('preview', { name: emote.name });
  }

  function flash(button) {
    button.classList.add('on');
    setTimeout(function () { button.classList.remove('on'); }, 600);
  }

  function play(emote) {
    if (emote.nearby) { playNearby(emote); return; }
    state.playing = emote.name;
    post('play', { name: emote.name });
    if (state.options.keepOpen) drawList();
  }

  function playNearby(row) {
    state.playing = row.name;
    post('playNearby', { id: row.id });
    if (state.options.keepOpen) drawList();
  }

  function toggleFavourite(emote) {
    if (!state.options.favourites || emote.nearby) return;

    emote.favourite = !emote.favourite;
    post('favourite', { name: emote.name });

    // Redraw only when the change would move the row out of view, which is
    // just the favourites category.
    if (state.category === 'favourites' && !state.query) draw();
    else drawList();
  }

  function close() {
    state.open = false;
    el.root.classList.add('hidden');
    post('close');
  }

  // ── Keyboard ──────────────────────────────────────────────────

  function moveCursor(step) {
    if (state.visible.length === 0) return;

    state.cursor += step;
    if (state.cursor < 0) state.cursor = state.visible.length - 1;
    if (state.cursor >= state.visible.length) state.cursor = 0;

    markCursor();
    preview(state.visible[state.cursor]);
  }

  function cycleCategory(step) {
    if (state.categories.length === 0) return;

    var at = 0;
    for (var i = 0; i < state.categories.length; i++) {
      if (state.categories[i].key === state.category) { at = i; break; }
    }
    at = (at + step + state.categories.length) % state.categories.length;

    state.category = state.categories[at].key;
    state.query = '';
    el.search.value = '';
    el.clear.classList.add('hidden');
    state.cursor = -1;
    draw();
    tabChanged();
  }

  function onNearby() {
    return state.category === 'nearby' && !state.query;
  }

  // Tells the game which tab is showing. On Nearby the player keeps their
  // controls and walks about while the list follows, so the search box gives
  // up the keyboard there; anywhere else it takes it back.
  function tabChanged() {
    state.confirming = null;
    highlightThing();
    if (onNearby()) {
      el.search.blur();
      post('nearby');
    } else {
      el.search.focus();
    }
    post('tab', { key: state.category });
  }

  // Tab: jump to the first row of the next thing in the Nearby list.
  function nextGroup(step) {
    var rows = state.visible;
    if (rows.length === 0) return;
    var cur = state.cursor >= 0 ? rows[state.cursor] : null;
    var key = cur ? cur.groupKey : null;
    var i = state.cursor;
    for (var n = 0; n < rows.length; n++) {
      i = (i + step + rows.length) % rows.length;
      if (rows[i].groupKey !== key) {
        if (step < 0) {
          // Back to the first row of that group, not its last.
          while (i > 0 && rows[i - 1].groupKey === rows[i].groupKey) i--;
        }
        state.cursor = i;
        markCursor();
        return;
      }
    }
  }

  document.addEventListener('keydown', function (event) {
    if (!state.open) return;

    if (state.options.closeKey && event.key === state.options.closeKey &&
        (event.key.length > 1 || document.activeElement !== el.search)) {
      // A letter key only closes the menu when the player is not typing it.
      event.preventDefault();
      close();
      return;
    }

    if (onNearby() && event.key === 'Tab') {
      event.preventDefault();
      nextGroup(event.shiftKey ? -1 : 1);
      return;
    }

    switch (event.key) {
      case 'Escape':
        event.preventDefault();
        if (state.confirming) { state.confirming = null; drawList(); break; }
        close();
        break;

      case 'ArrowDown':
        event.preventDefault();
        moveCursor(1);
        break;

      case 'ArrowUp':
        event.preventDefault();
        moveCursor(-1);
        break;

      case 'ArrowRight':
        // The arrows only change category when the player is not mid-word in
        // the search box, or they cannot move the text caret.
        if (el.search.value) return;
        event.preventDefault();
        cycleCategory(1);
        break;

      case 'ArrowLeft':
        if (el.search.value) return;
        event.preventDefault();
        cycleCategory(-1);
        break;

      case 'Enter':
        event.preventDefault();
        if (state.cursor >= 0 && state.visible[state.cursor]) {
          play(state.visible[state.cursor]);
        } else if (state.visible.length === 1) {
          // One search result and a press of Enter is the fastest way to
          // play an emote you already know the name of.
          play(state.visible[0]);
        }
        break;

      case 'Backspace':
        // Backspace is the cancel key in game. In the menu it belongs to the
        // search box, so it is only a cancel when the box is empty.
        if (el.search.value) return;
        event.preventDefault();
        post('cancel');
        break;

      default:
        // With the mouse off the menu the keys are the player's: W walks, it
        // does not search. The same on the Nearby tab whatever the mouse does.
        if (onNearby() || !state.hover) return;
        // Any other typing goes to the search box, wherever the focus is.
        if (event.key.length === 1 && document.activeElement !== el.search) {
          el.search.focus();
        }
    }
  });

  // ── Wiring ────────────────────────────────────────────────────

  el.search.addEventListener('input', function () {
    state.query = el.search.value;
    state.cursor = state.query ? 0 : -1;
    el.clear.classList.toggle('hidden', !state.query);
    draw();
    if (state.cursor === 0 && state.visible[0]) preview(state.visible[0]);
  });

  el.clear.addEventListener('click', function () {
    state.query = '';
    el.search.value = '';
    el.clear.classList.add('hidden');
    state.cursor = -1;
    el.search.focus();
    draw();
  });

  el.props.addEventListener('click', function () {
    state.propsOnly = !state.propsOnly;
    el.props.classList.toggle('on', state.propsOnly);
    el.list.classList.toggle('props-only', state.propsOnly);
    state.cursor = -1;
    draw();
  });

  el.keep.addEventListener('change', function () {
    state.options.keepOpen = el.keep.checked;
    post('keepOpen', { on: el.keep.checked });
  });

  // The game is told when the mouse is over the menu, so a click here is only
  // a click here, and off the menu the player can walk and look about.
  var panelEl = document.getElementById('panel');
  panelEl.addEventListener('mouseenter', function () {
    state.hover = true;
    post('hover', { on: true });
    if (!onNearby()) el.search.focus();
  });
  panelEl.addEventListener('mouseleave', function () {
    state.hover = false;
    el.search.blur();
    post('hover', { on: false });
  });

  el.close.addEventListener('click', close);
  el.stop.addEventListener('click', function () { post('cancel'); state.playing = null; drawList(); });

  // ── From the game ─────────────────────────────────────────────

  window.addEventListener('message', function (event) {
    var msg = event.data || {};

    if (msg.action === 'open') {
      state.emotes = msg.emotes || [];
      state.categories = msg.categories || [];
      state.recents = msg.recents || [];
      takeNearby(msg.nearby);
      state.strings = msg.strings || {};
      state.options = msg.options || state.options;
      state.playing = msg.playing || null;
      state.query = '';
      state.cursor = -1;
      state.open = true;
      state.hover = true;
      state.confirming = null;

      // Open on the first category that has something in it, so a player who
      // has never starred anything does not land on an empty Favourites tab.
      state.category = null;
      for (var i = 0; i < state.categories.length; i++) {
        state.category = state.categories[i].key;
        computeVisible();
        if (state.visible.length > 0) break;
      }

      el.title.textContent = state.strings.title || 'Emotes';
      el.search.placeholder = state.strings.search || 'Search emotes';
      el.stop.textContent = state.strings.cancelEmote || 'Stop emote';
      el.props.textContent = state.strings.propsOnly || 'Props only';
      el.props.classList.toggle('on', state.propsOnly);
      el.list.classList.toggle('props-only', state.propsOnly);
      el.keepText.textContent = state.strings.keepOpen || 'Keep menu open';
      el.keep.checked = state.options.keepOpen === true;
      el.hint.textContent = state.strings.hintKeys || '';
      el.search.value = '';
      el.clear.classList.add('hidden');

      draw();
      el.root.classList.remove('hidden');
      el.search.focus();
      tabChanged();
      return;
    }

    if (msg.action === 'close') {
      state.open = false;
      el.root.classList.add('hidden');
      return;
    }

    if (msg.action === 'favourite') {
      for (var j = 0; j < state.emotes.length; j++) {
        if (state.emotes[j].name === msg.name) {
          state.emotes[j].favourite = msg.favourite === true;
          break;
        }
      }
      if (state.open) drawList();
      return;
    }

    if (msg.action === 'nearby') {
      takeNearby(msg.nearby);
      if (state.open && onNearby()) refreshNearby();
      return;
    }

    if (msg.action === 'recents') {
      state.recents = msg.recents || [];
      // Only the Recent category changes shape, so nothing else needs redrawing.
      if (state.open && state.category === 'recent' && !state.query) draw();
      return;
    }

    if (msg.action === 'sync') {
      var starred = {};
      (msg.favourites || []).forEach(function (name) { starred[name] = true; });
      state.emotes.forEach(function (emote) { emote.favourite = starred[emote.name] === true; });
      state.recents = msg.recents || [];
      if (msg.nearby) takeNearby(msg.nearby);
      if (typeof msg.dev === 'boolean') state.options.dev = msg.dev;
      if (state.open) draw();
    }
  });
})();
