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
    strings: {},
    options: { favourites: true, preview: true },
    category: null,
    query: '',
    cursor: -1,
    visible: [],
    playing: null,
    propsOnly: false,
    open: false
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
    return emote.label.toLowerCase().indexOf(query) !== -1 ||
           emote.name.indexOf(query) !== -1;
  }

  function computeVisible() {
    var query = state.query.trim().toLowerCase();
    var rows;

    if (query) {
      // A search looks everywhere. Narrowing it to the open category is the
      // single most annoying thing an emote menu can do.
      rows = state.emotes.filter(function (e) { return matches(e, query); });
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
      });
      el.categories.appendChild(button);
    });
  }

  function drawList() {
    var query = state.query.trim().toLowerCase();
    el.list.textContent = '';

    state.visible.forEach(function (emote, index) {
      var row = document.createElement('li');
      row.className = 'emote';
      if (index === state.cursor) row.className += ' cursor';
      if (emote.name === state.playing) row.className += ' playing';
      row.dataset.name = emote.name;

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
      if (state.propsOnly && !state.query) el.empty.textContent = state.strings.noProps || 'No prop emotes here.';
      else if (state.query) el.empty.textContent = state.strings.noResults || 'No emote matches that.';
      else if (state.category === 'favourites') el.empty.textContent = state.strings.noFavourites || '';
      else if (state.category === 'recent') el.empty.textContent = state.strings.noRecent || '';
      else el.empty.textContent = state.strings.noResults || '';
    }

    var template = state.strings.count || '%s emotes';
    el.count.textContent = template.replace('%s', String(state.visible.length));
  }

  // Moves the highlight without redrawing every row: a full redraw on each
  // arrow key press loses the scroll position and flickers.
  function markCursor() {
    var rows = el.list.children;
    for (var i = 0; i < rows.length; i++) {
      rows[i].classList.toggle('cursor', i === state.cursor);
    }
    if (state.cursor >= 0 && rows[state.cursor]) {
      rows[state.cursor].scrollIntoView({ block: 'nearest' });
    }
  }

  function draw() {
    computeVisible();
    drawCategories();
    drawList();
  }

  // ── Actions ───────────────────────────────────────────────────

  var previewTimer = null;

  function preview(emote) {
    if (!state.options.preview) return;
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
    state.playing = emote.name;
    post('play', { name: emote.name });
    if (state.options.keepOpen) drawList();
  }

  function toggleFavourite(emote) {
    if (!state.options.favourites) return;

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

    switch (event.key) {
      case 'Escape':
        event.preventDefault();
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

  el.close.addEventListener('click', close);
  el.stop.addEventListener('click', function () { post('cancel'); state.playing = null; drawList(); });

  // ── From the game ─────────────────────────────────────────────

  window.addEventListener('message', function (event) {
    var msg = event.data || {};

    if (msg.action === 'open') {
      state.emotes = msg.emotes || [];
      state.categories = msg.categories || [];
      state.recents = msg.recents || [];
      state.strings = msg.strings || {};
      state.options = msg.options || state.options;
      state.playing = msg.playing || null;
      state.query = '';
      state.cursor = -1;
      state.open = true;

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
      if (state.open) draw();
    }
  });
})();
