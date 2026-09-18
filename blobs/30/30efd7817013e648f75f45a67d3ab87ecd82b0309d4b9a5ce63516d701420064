/*
    Poggy Hub — the open script's state and its pending changes.

    The server sends a script's settings as nodes (§3 of the spec): one per
    setting, group, list, map or string list, each with its value. The page
    never edits those. It keeps:

        base    { nodePath: value }  what the server sent (the file on disk)
        work    { nodePath: value }  base with every pending change applied
        pending [change, ...]        the §6.2 change queue, in the order made

    Every edit becomes a change, is applied to `work` at once so the page shows
    it, and is sent to the server in order on Save. Because `work` can always
    be rebuilt from `base` + `pending`, undoing one change, discarding, or
    re-applying the queue to a freshly loaded script (after a "stale" save) all
    use the same code: rebuild().

    Paths are Lua-shaped and 1-based. A path deeper than a node
    (Config.Zones[3].loot[2].count) is resolved by finding the longest node
    path it starts with and walking the rest inside that node's value.
*/
(function () {
    'use strict';

    var PH = window.PoggyHub;
    var clone = PH.clone, deepEqual = PH.deepEqual, parsePath = PH.parsePath, segKey = PH.segKey;

    var STRUCTURAL = { insert: 1, remove: 1, move: 1, renameKey: 1, duplicate: 1 };

    var S = PH.S = {
        cur: null,
        listeners: [],
        roles: null,           // cached roles answer, for linkRole
    };

    S.on = function (fn) { S.listeners.push(fn); };
    function emit(what, detail) {
        S.listeners.forEach(function (fn) { try { fn(what, detail); } catch (e) { console.error(e); } });
    }
    S.emit = emit;

    // ----------------------------------------------------------------- load --

    /** Build the state for a `script` answer. Pending changes start empty. */
    S.build = function (data) {
        var nodesIn = data.nodes || [];
        var list = Array.isArray(nodesIn) ? nodesIn : Object.keys(nodesIn).map(function (k) { return nodesIn[k]; });
        var cur = {
            id: data.card ? data.card.id : data.id,
            data: data,
            card: data.card || {},
            meta: data.meta || {},
            nodes: {},
            byKey: {},
            byCanon: {},
            order: [],
            base: {},
            work: {},
            roleBase: {},
            roleWork: {},
            pending: [],
            fingerprints: {},
            lock: data.lock || { mine: false },
            restartNeeded: false,
        };
        list.forEach(function (n) {
            if (!n || !n.path) return;
            cur.nodes[n.path] = n;
            cur.byKey[segKey(parsePath(n.path))] = n;
            cur.byCanon[PH.canon(n.path)] = n;
            cur.base[n.path] = clone(n.value);
            cur.roleBase[n.path] = n.role || null;
        });
        var order = Array.isArray(data.order) && data.order.length ? data.order : list.map(function (n) { return n.path; });
        cur.order = order.filter(function (p) { return cur.nodes[p]; });
        // Nodes missing from `order` still show, after the rest.
        list.forEach(function (n) { if (n && n.path && cur.order.indexOf(n.path) === -1) cur.order.push(n.path); });
        cur.kids = {};
        cur.order.forEach(function (p) {
            var n = cur.nodes[p];
            if (n.parent) (cur.kids[n.parent] = cur.kids[n.parent] || []).push(n);
        });
        cur.orderIndex = {};
        cur.order.forEach(function (p, i) { cur.orderIndex[p] = i; });

        (data.files || []).forEach(function (f) {
            if (f && f.file) cur.fingerprints[f.file] = f.fingerprint;
        });
        resetWork(cur);
        return cur;
    };

    function resetWork(cur) {
        cur.work = {};
        cur.roleWork = {};
        Object.keys(cur.base).forEach(function (p) { cur.work[p] = clone(cur.base[p]); });
        Object.keys(cur.roleBase).forEach(function (p) { cur.roleWork[p] = cur.roleBase[p]; });
    }

    // -------------------------------------------------------------- resolve --

    /** The node a path lives in, and the rest of the path inside its value. */
    S.resolve = function (path, cur) {
        cur = cur || S.cur;
        if (!cur) return null;
        var segs = typeof path === 'string' ? parsePath(path) : path;
        for (var i = segs.length; i > 0; i--) {
            var node = cur.byKey[segKey(segs.slice(0, i))];
            if (node) return { node: node, rest: segs.slice(i) };
        }
        return null;
    };

    function step(v, seg) {
        if (v === null || v === undefined || typeof v !== 'object') return undefined;
        if (Array.isArray(v)) return typeof seg === 'number' ? v[seg - 1] : undefined;
        if (typeof seg === 'number') return v[String(seg)];
        return v[seg];
    }

    function walk(v, rest) {
        for (var i = 0; i < rest.length; i++) {
            v = step(v, rest[i]);
            if (v === undefined) return undefined;
        }
        return v;
    }

    /** Working value at any path (with pending changes applied). */
    S.get = function (path) {
        var r = S.resolve(path);
        if (!r) return undefined;
        var v = S.cur.work[r.node.path];
        // A group has no value of its own (its settings are nodes); put one
        // together from its children, so a collection row kept as settings
        // reads like any other row.
        if (v === undefined && r.node.kind === 'group') v = assemble(S.cur, r.node, 0);
        return walk(v, r.rest);
    };

    function assemble(cur, node, depth) {
        if (depth > 12) return undefined;
        var out = {};
        (cur.kids[node.path] || []).forEach(function (c) {
            var cv = cur.work[c.path];
            if (cv === undefined && c.kind === 'group') cv = assemble(cur, c, depth + 1);
            if (cv !== undefined) out[String(c.key)] = cv;
        });
        return out;
    }
    S.kidsOf = function (path) { return (S.cur && S.cur.kids[path]) || []; };

    /** The value on disk at a path, when nothing structural is pending for its node. */
    S.getBase = function (path) {
        var r = S.resolve(path);
        if (!r) return undefined;
        return walk(S.cur.base[r.node.path], r.rest);
    };

    S.node = function (path) { return S.cur ? (S.cur.nodes[path] || S.cur.byCanon[PH.canon(path)] || null) : null; };

    S.fileFor = function (path) {
        var r = S.resolve(path);
        return r ? r.node.file : null;
    };

    S.roleOf = function (path) { return S.cur ? (S.cur.roleWork[path] || null) : null; };

    // ---------------------------------------------------------------- apply --

    function setIn(root, rest, value) {
        var parent = walk(root, rest.slice(0, -1));
        if (!parent || typeof parent !== 'object') return false;
        var last = rest[rest.length - 1];
        if (Array.isArray(parent)) {
            if (typeof last !== 'number' || last < 1 || last > parent.length) return false;
            parent[last - 1] = value;
            return true;
        }
        var key = typeof last === 'number' ? String(last) : last;
        if (!(key in parent)) return false;
        parent[key] = value;
        return true;
    }

    /** Apply one change to cur.work. Returns null, or why it could not apply. */
    function apply(cur, ch) {
        var r = S.resolve(ch.path, cur);
        if (!r) return 'The setting ' + ch.path + ' is not in the file any more.';
        var np = r.node.path, rest = r.rest;

        switch (ch.op) {
        case 'set':
            if (!rest.length) { cur.work[np] = clone(ch.value); return null; }
            return setIn(cur.work[np], rest, clone(ch.value)) ? null : 'Nothing at ' + ch.path + ' any more.';

        case 'reset':
            if (rest.length || r.node.default === null || r.node.default === undefined) return 'No default is known for ' + ch.path + '.';
            cur.work[np] = clone(r.node.default);
            return null;

        case 'insert': {
            var holder = rest.length ? walk(cur.work[np], rest) : cur.work[np];
            var key = ch.key !== undefined && ch.key !== null ? ch.key : ch.index;
            if (holder === undefined && rest.length) {
                // A declared list the row does not have yet (a collection row
                // without "buys"): the server creates it on insert, and so does the page.
                var owner = walk(cur.work[np], rest.slice(0, -1));
                var last0 = rest[rest.length - 1];
                if (PH.isPlainObj(owner) && typeof last0 === 'string') { owner[last0] = []; holder = owner[last0]; }
            }
            if (Array.isArray(holder) && (typeof key !== 'string' || /^\d+$/.test(key) && holder.length)) {
                var idx = ch.index === undefined || ch.index === null ? holder.length + 1 : Number(ch.index);
                if (!(idx >= 1 && idx <= holder.length + 1)) return 'Position ' + ch.index + ' is outside ' + ch.path + '.';
                holder.splice(idx - 1, 0, clone(ch.value));
                return null;
            }
            if (Array.isArray(holder) && holder.length === 0) {
                // An empty map arrives as [] (Lua cannot tell them apart); the first key makes it a map.
                var obj = {};
                obj[key] = clone(ch.value);
                if (!rest.length) cur.work[np] = obj;
                else if (!setIn(cur.work[np], rest, obj)) return 'Nothing at ' + ch.path + ' any more.';
                return null;
            }
            if (!holder || typeof holder !== 'object') return 'Nothing at ' + ch.path + ' any more.';
            if (key === undefined || key === null || String(key) in holder) return 'The key "' + key + '" is already used in ' + ch.path + '.';
            holder[String(key)] = clone(ch.value);
            return null;
        }

        case 'remove': {
            var h2 = rest.length ? walk(cur.work[np], rest) : cur.work[np];
            var k2 = ch.key !== undefined && ch.key !== null ? ch.key : ch.index;
            if (Array.isArray(h2)) {
                var i2 = Number(k2);
                if (!(i2 >= 1 && i2 <= h2.length)) return 'Row ' + k2 + ' is not in ' + ch.path + ' any more.';
                h2.splice(i2 - 1, 1);
                return null;
            }
            if (!h2 || typeof h2 !== 'object' || !(String(k2) in h2)) return '"' + k2 + '" is not in ' + ch.path + ' any more.';
            delete h2[String(k2)];
            return null;
        }

        case 'duplicate': {
            // The server copies the row in the file, code and all; the page copies the value.
            var hd = rest.length ? walk(cur.work[np], rest) : cur.work[np];
            if (Array.isArray(hd) && (ch.oldKey === undefined || ch.oldKey === null)) {
                var di = Number(ch.index);
                if (!(di >= 1 && di <= hd.length)) return 'Row ' + ch.index + ' is not in ' + ch.path + ' any more.';
                hd.splice(di, 0, clone(hd[di - 1]));
                return null;
            }
            if (!hd || typeof hd !== 'object' || Array.isArray(hd)) return ch.path + ' is not a map.';
            var dk = String(ch.oldKey), nk2 = String(ch.newKey);
            if (!(dk in hd)) return '"' + dk + '" is not in ' + ch.path + ' any more.';
            if (nk2 in hd) return 'The key "' + nk2 + '" is already used.';
            hd[nk2] = clone(hd[dk]);
            return null;
        }

        case 'move': {
            var h3 = rest.length ? walk(cur.work[np], rest) : cur.work[np];
            if (!Array.isArray(h3)) return ch.path + ' is not a list.';
            var from = Number(ch.from), to = Number(ch.to);
            if (!(from >= 1 && from <= h3.length && to >= 1 && to <= h3.length)) return 'Row ' + ch.from + ' is not in ' + ch.path + ' any more.';
            var item = h3.splice(from - 1, 1)[0];
            h3.splice(to - 1, 0, item);
            return null;
        }

        case 'renameKey': {
            var h4 = rest.length ? walk(cur.work[np], rest) : cur.work[np];
            if (!h4 || typeof h4 !== 'object' || Array.isArray(h4)) return ch.path + ' is not a map.';
            var ok = String(ch.oldKey), nk = String(ch.newKey);
            if (!(ok in h4)) return '"' + ok + '" is not in ' + ch.path + ' any more.';
            if (nk in h4) return 'The key "' + nk + '" is already used.';
            // Rebuild so the renamed key keeps its place.
            var rebuilt = {};
            Object.keys(h4).forEach(function (k) { rebuilt[k === ok ? nk : k] = h4[k]; });
            Object.keys(h4).forEach(function (k) { delete h4[k]; });
            Object.keys(rebuilt).forEach(function (k) { h4[k] = rebuilt[k]; });
            // A server that updates references itself (updateRefs) is sent one
            // change; the page mirrors what it will do so the values shown match.
            if (ch.updateRefs && Array.isArray(ch._refPaths)) {
                ch._refPaths.forEach(function (p) {
                    var rr = S.resolve(p, cur);
                    if (!rr) return;
                    var curVal = rr.rest.length ? walk(cur.work[rr.node.path], rr.rest) : cur.work[rr.node.path];
                    if (String(curVal) !== ok) return;
                    var nv = typeof curVal === 'number' ? Number(nk) : nk;
                    if (!rr.rest.length) cur.work[rr.node.path] = nv;
                    else setIn(cur.work[rr.node.path], rr.rest, nv);
                });
            }
            return null;
        }

        case 'linkRole': {
            if (rest.length) return 'Only a whole list can follow a role.';
            cur.roleWork[np] = ch.role;
            var roles = S.roles && S.roles.roles;
            if (roles && roles[ch.role] && Array.isArray(roles[ch.role].list)) cur.work[np] = clone(roles[ch.role].list);
            return null;
        }

        case 'unlinkRole':
            if (rest.length) return 'Only a whole list can follow a role.';
            cur.roleWork[np] = null;
            return null;

        default:
            return 'Unknown change ' + ch.op + '.';
        }
    }

    /**
     * Rebuild work from base + pending. Changes that no longer apply are taken
     * out of the queue and returned, each with its reason.
     */
    S.rebuild = function (cur) {
        cur = cur || S.cur;
        resetWork(cur);
        var failed = [];
        var kept = [];
        cur.pending.forEach(function (ch) {
            var why = apply(cur, ch);
            if (why) { ch._why = why; failed.push(ch); } else kept.push(ch);
        });
        cur.pending = kept;
        return failed;
    };

    // ---------------------------------------------------------------- queue --

    function under(path, prefix) {
        path = PH.canon(path); prefix = PH.canon(prefix);
        if (path === prefix) return true;
        if (path.indexOf(prefix) !== 0) return false;
        var c = path.charAt(prefix.length);
        return c === '.' || c === '[';
    }
    S.under = under;

    function structuralOn(cur, nodePath) {
        return cur.pending.some(function (c) { return STRUCTURAL[c.op] && under(c.path, nodePath); });
    }

    /**
     * Queue one change (§6.2 shape; the file is filled in). Applies it to
     * work at once. Returns true when it was queued or folded into an
     * earlier one.
     */
    S.queue = function (change) {
        var cur = S.cur;
        if (!cur || S.isReadOnly()) return false;
        var r = S.resolve(change.path);
        if (!r) { PH.toast({ kind: 'error', title: 'Cannot change that', text: change.path + ' is not in this script.' }); return false; }
        var ch = Object.assign({}, change);
        ch.path = PH.canon(ch.path);
        if (!ch.file) ch.file = r.node.file;
        ch._node = r.node.path;
        if (ch.op === 'set' || ch.op === 'reset' || ch.op === 'linkRole' || ch.op === 'unlinkRole') {
            ch._old = clone(S.get(ch.path));
            if (ch.op === 'linkRole' || ch.op === 'unlinkRole') ch._oldRole = S.roleOf(r.node.path);
        }

        // Fold a set into the last set of the same path, as long as nothing
        // structural came after it (which could have moved what the path
        // points at). Typing into a box makes one change, not one per key.
        if (ch.op === 'set' || ch.op === 'reset') {
            for (var j = cur.pending.length - 1; j >= 0; j--) {
                var c = cur.pending[j];
                if (STRUCTURAL[c.op]) break;
                if ((c.op === 'set' || c.op === 'reset') && c.path === ch.path) {
                    ch._old = c._old;
                    cur.pending.splice(j, 1);
                    break;
                }
            }
        }
        if (ch.op === 'linkRole' || ch.op === 'unlinkRole') {
            for (var k = cur.pending.length - 1; k >= 0; k--) {
                var c2 = cur.pending[k];
                if ((c2.op === 'linkRole' || c2.op === 'unlinkRole') && c2.path === ch.path) {
                    ch._old = c2._old;
                    ch._oldRole = c2._oldRole;
                    cur.pending.splice(k, 1);
                    break;
                }
            }
        }

        var why = apply(cur, ch);
        if (why) {
            S.rebuild(cur);   // put work back exactly as the queue says
            PH.toast({ kind: 'error', title: 'That change could not be made', text: why });
            emit('change', null);
            return false;
        }

        // An edit that puts a value back to what is on disk is no change at all.
        var dropped = false;
        if ((ch.op === 'set' || ch.op === 'reset') && !structuralOn(cur, r.node.path)) {
            var baseVal = S.getBase(ch.path);
            if (baseVal !== undefined && deepEqual(S.get(ch.path), baseVal)) dropped = true;
        }
        if ((ch.op === 'linkRole' || ch.op === 'unlinkRole') && cur.roleWork[r.node.path] === cur.roleBase[r.node.path] &&
            deepEqual(cur.work[r.node.path], cur.base[r.node.path])) dropped = true;
        if (!dropped) cur.pending.push(ch);

        if (PH.lock && PH.lock.activity) PH.lock.activity();
        emit('change', ch);
        return true;
    };

    /** Shorthand for the commonest change. */
    S.set = function (path, value) { return S.queue({ op: 'set', path: path, value: value }); };

    /** Take one queued change back out, and rebuild. Returns changes that fell out with it. */
    S.unqueue = function (ch) {
        var cur = S.cur;
        var i = cur.pending.indexOf(ch);
        if (i === -1) return [];
        cur.pending.splice(i, 1);
        var failed = S.rebuild(cur);
        emit('change', null);
        return failed;
    };

    S.discard = function () {
        if (!S.cur) return;
        S.cur.pending = [];
        resetWork(S.cur);
        emit('change', null);
    };

    /** The queue as the server wants it: the §6.2 fields only. */
    S.wire = function () {
        return S.cur.pending.map(function (c) {
            var out = {};
            Object.keys(c).forEach(function (k) { if (k.charAt(0) !== '_') out[k] = c[k]; });
            return out;
        });
    };

    // --------------------------------------------------------------- status --

    S.isReadOnly = function () {
        var cur = S.cur;
        return !cur || !cur.lock || !cur.lock.mine;
    };

    /** Is anything pending at this path, inside it, or (structurally) around it? */
    S.isPending = function (path) {
        var cur = S.cur;
        if (!cur) return false;
        return cur.pending.some(function (c) { return under(c.path, path) || (under(path, c.path) && c.path !== path && STRUCTURAL[c.op]); });
    };

    /** Does this node's working value differ from the shipped default? */
    S.differsFromDefault = function (node) {
        if (!node || node.default === null || node.default === undefined) return false;
        // A list inside a collection row is not a node of its own; read it by path.
        var v = Object.prototype.hasOwnProperty.call(S.cur.work, node.path) ? S.cur.work[node.path] : S.get(node.path);
        return !deepEqual(v, node.default);
    };

    S.hasDefault = function (node) { return !!node && node.default !== null && node.default !== undefined; };

    S.pendingCount = function () { return S.cur ? S.cur.pending.length : 0; };
})();
