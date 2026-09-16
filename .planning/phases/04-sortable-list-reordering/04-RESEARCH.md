# Phase 4: Sortable List Reordering - Research

**Researched:** 2026-09-15
**Domain:** In-place list reordering (Settings list cards) with Firestore persistence and live cross-surface sync, in a no-build single-file CDN React app
**Confidence:** HIGH (every code claim verified by Read of `index.html` this session; one live browser experiment against the deployed app)

## Summary

Phase 4 adds up/down reorder controls to the three Settings list cards (Equipment, Vehicle Plates, Approvers), pins `'Other'` as the immovable last item, and persists order through the existing `saveOptions()` → Firestore → `onSnapshot` round trip. The architecture already supports this: all five list-rendering surfaces read the same shared `equipmentOptions`/`platOptions`/`approverOptions` state arrays (index.html:346, 350, 470, 524, 667), so a persisted swap automatically propagates to the request form, manager filter, and detail modal selects. **No new dependencies, no schema change, no build step** — the phase is a pure addition to the existing ListEditor component plus one new `moveOption` function following the exact mutation pattern of `addOption`/`editOption`/`removeOption` (lines 240-259).

Live-browser experiment against the deployed app (http://localhost:8080, served by the running Python server) surfaced two production realities the plan must handle: **(1)** the Settings view currently renders **18 unmaterialized Lucide icons** (9× `pencil` + 9× `trash2`) because `lucide.createIcons()` is only called from the *requests* snapshot (line 206), never from the *options* snapshot (line 207) or the Settings nav handler (line 414) — the new ChevronUp/ChevronDown buttons will be invisible unless the plan adds the `createIcons()` refresh; **(2)** live Firestore data contains **`'Laptop'` positioned AFTER `'Other'`** in the Equipment list (`['DJI Osmo Action 6', 'Car', 'Other', 'Laptop']`), proving the existing `addOption` append breaks the "`'Other'` pinned last" invariant — and making the UI-SPEC's boundary rule (`down disabled only if `items[i+1] === 'Other'`) unsafe for that state: the last non-`'Other'` item's Down button would be enabled and the approved swap pattern would write `undefined` into the array and persist it.

**Primary recommendation:** Implement the UI-SPEC contract verbatim (row layout `[label][Up][Down][Edit][Delete]`, glass-arrow classes, `moveOption` pattern) **plus three confirmed amendments**: (a) add `i >= items.length - 1` to the Down-disabled condition (prevents `undefined`-element corruption on legacy data), (b) insert `setTimeout(() => lucide.createIcons(), 100)` in the options onSnapshot callback **and** the Settings nav onClick (icons are provably blank today), and (c) change `addOption` to insert before `'Other'` when `'Other'` is last (restores the pin the UI-SPEC E3 already claims, `[ASSUMED]`-flagged pending user sign-off in Q2).

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| SORT-01 | Manager can reorder equipment list items (up/down) in the Settings Equipment List card | `moveOption('equipment', i, i±1)` wired via `onMoveUp`/`onMoveDown` props on the line 442 `<ListEditor listKey="equipment">`; state array `equipmentOptions` (line 159) | 
| SORT-02 | Manager can reorder vehicle plates list items (up/down) in the Settings Vehicle Plates List card | Same, `listKey="plat"`, call site line 443; `platOptions` (line 157) |
| SORT-03 | Manager can reorder approvers list items (up/down) in the Settings Approvers List card | Same, `listKey="approver"`, call site line 444; `approverOptions` (line 158) |
| SORT-04 | "Other" stays pinned as the last item in every list and cannot be moved | `o === 'Other'` disables both arrows (UI-SPEC §Helper-color classes `bg-white/5 text-white/20`); boundary rule needs the last-index amendment (Pitfall 2); `addOption` must insert before `'Other'` (Pitfall 3), legacy live data broken today |
| SORT-05 | Reordered lists persist to Firestore and sync to request form select, manager filter select, and detail views on all devices | `saveOptions` (lines 235-238) wholesale-replaces the three arrays in the single `options/lists` doc (merge confirmed, Firestore docs [CITED]); onSnapshot (line 207) re-fills state → all 5 select surfaces (346, 350, 470, 524, 667) re-render |
| SORT-06 | Reordered lists survive a page reload and stay in the same order for every user | Order lives only in the Firestore doc arrays; `DEFAULT_*` constants (lines 63-65) are the fallback only when the doc is missing; no localStorage involved in order state |

</phase_requirements>

## Project Constraints (from AGENTS.md)

Directives extracted from `./AGENTS.md` (GSD project block) — the planner MUST treat these as locked:

1. **Single-file HTML with CDN React 18 + Tailwind + Babel + Lucide + Firebase compat v11.6.1 — no build step, no npm, no bundler.** No new packages allowed for this phase; reorder code goes into `index.html` (multi-`text/babel`-block layout already in place, main App block starts line 97).
2. **Firebase API style: namespaced compat API** (`db.collection().doc().set()`, `.onSnapshot()`) — not modular v9+.
3. **Mobile-first** — staff phones over LAN HTTP (non-secure context); the reorder buttons are `p-2` icon buttons (~30×30px) per UI-SPEC §Spacing exceptions — locked, do not change sizing.
4. **Security: manager actions gated by passcode (Firestore-backed, changeable).** Settings view (and therefore the new reorder buttons) is only reachable when `isManagerAuthenticated` is true (line 414 nav gate).
5. **Compatibility** — must keep working opened/served anywhere without a build step; avoid ES-module syntax (PITFALLS.md Pitfall 2), keep the `setTimeout(() => lucide.createIcons(), 100)` pattern for icons (PITFALLS.md Pitfall 5).

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Reorder interaction (up/down buttons, disabled states, tooltips) | Browser (Settings view) | — | Pure client-side UI state; ListEditor renders rows from shared state arrays |
| Reorder persistence | Firestore (options/lists doc) | — | `saveOptions()` writes the three arrays wholesale; this is the only durable store of order |
| Order propagation to all list surfaces | Browser (React state + onSnapshot) | — | No server/SSR tier exists; state arrays feed every select via `.map()` |
| Auth/gate (manager-only access) | Browser (passcode gate) + Firestore (passcode field) | — | `isManagerAuthenticated` gates the Settings nav; passcode lives in the same `options/lists` doc (`d.passcode`, line 207) |

**Tier misassignment warning:** do not add a server, worker, or localStorage layer for order state — Firestore is the single source of truth; localStorage is only used for `kwm_dark_mode` and `kwm_seen_requests` (lines 204, 208) and must NOT hold option order.

## Standard Stack

### Core

The phase adds **no new packages** — project constraint (AGENTS.md). The verified in-repo stack the code must use:

| Library | Version (CDN pin) | Purpose | Why Standard |
|---------|-------------------|---------|--------------|
| React | 18.3.1 UMD | UI framework | Locked by AGENTS.md; `const { useState, useEffect, useMemo } = React` (line 98) |
| Babel standalone | 8.0.5 | In-browser JSX transform | Locked; main `text/babel` block line 97 |
| Tailwind Play CDN | 3.4.17 | Styling | Locked; reorder buttons use existing utility classes only |
| Lucide | 1.44.0 UMD | Icons (`ChevronUp`, `ChevronDown`, existing Pencil/Trash2/Check/X) | Locked; Icon component lines 92-95 (`icon name` global, `<i data-lucide={name.toLowerCase()}>`); **must call `lucide.createIcons()` after re-renders — see Pitfall 1** |
| Firebase compat | 11.6.1 namespaced | Firestore options doc + anonymous auth | Locked; `db.collection('artifacts/' + appId + '/public/data/options').doc('lists')` (lines 207, 237, 273) |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None new required | — | — | The constraint is no-build; nothing outside the CDN set is needed for index-swap reordering |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Index-swap buttons (UI-SPEC) | Drag-and-drop (SORT-07) | Deferred to v2 explicitly — drag is a desktop gesture; up/down buttons are touch-safe on LAN phones (REQUIREMENTS.md v2 SORT-07) |
| Existing `saveOptions()` persist | Local optimistic state + batch write | Optimistic local state diverges from the rest of the phase-3 mutation pattern and risks the onSnapshot echo overwriting it; keep the established "mutate → saveOptions → echo" flow |
| `setTimeout` createIcons refresh | React effect on every render | PITFALLS.md warns full-render createIcons causes layout thrashing; the 100 ms snapshot-scoped refresh matches the existing pattern |

**Installation:** none — no `npm install` for this phase.

**Version verification:** React 18.3.1 UMD / Babel 8.0.5 / Tailwind 3.4.17 / Lucide 1.44.0 / Firebase 11.6.1 pins are the locked, STACK.md-verified in-repo decisions (`[CITED: research/STACK.md]`) — head-checks and registry queries were performed against these exact URLs in the v1.0 milestone research (2026-09-11); re-verification was out of scope for a no-new-dependency phase and no CDN URL changes this phase.

## Package Legitimacy Audit

**No external packages are introduced by this phase** (AGENTS.md no-npm constraint). The legacy-CDN package set was audited in the v1.0 milestone against npm registry + unpkg/gstatic HEAD checks (STACK.md `## Sources` — react 18.3.1, @babel/standalone 8.0.5, firebase 11.6.1 compat, lucide 1.44.0, tailwind 3.4.17 all `[VERIFIED: npm registry + HEAD]`). `gsd-tools query package-legitimacy` is not applicable with a zero-add list.

**Packages removed due to [SLOP] verdict:** none
**Packages flagged as suspicious [SUS]:** none

## Architecture Patterns

### System Architecture Diagram

```
Settings (view === 'settings', gated by isManagerAuthenticated)
   │  ListEditor card (×3: equipment / plat / approver)
   │    row: [label][▲Up][▼Down][✎Edit][🗑Delete]      ← NEW buttons
   │       │ onClick → moveOption(list, i, i±1)
   ▼
App scope: moveOption (NEW, after line 260)
   │  reads equipmentOptions/platOptions/approverOptions state (lines 157-159)
   │  builds `next` map, swaps fromIdx↔toIdx, calls saveOptions(next.plat, next.approver, next.equipment)
   ▼
saveOptions (lines 235-238)
   │  db.collection('artifacts/<appId>/public/data/options').doc('lists')
   │  .set({ platOptions, approverOptions, equipmentOptions }, { merge: true })   ← wholesale array replace
   ▼
Firestore (options/lists doc)          ── the single source of truth for order
   ▲
onSnapshot (line 207) → setPlatOptions/setApproverOptions/setEquipmentOptions (no createIcons today ⇒ Pitfall 1)
   ▼
All consuming surfaces re-render from shared state:
   • request form equipment select (line 346)   • request form plat select (line 350, when vehicle)
   • manager filter equipment select (line 470) • manager card approver select (line 524)
   • detail modal approver select (line 667)
```

Traceable path for SORT-05/06: click ▲ → moveOption → saveOptions → Firestore → every browser's onSnapshot → all 5 selects update; reload returns the same arrays from the doc.

### Recommended Project Structure
No new files. All changes land in `index.html` (single-file constraint):
- App scope after line 260: `moveOption`
- ListEditor component (lines 100-130): two new props + two buttons per non-editing row
- Line 244: `addOption` insert-before-`'Other'` amendment (pending sign-off, Q2)
- Line 207: add `setTimeout(() => lucide.createIcons(), 100)` to options snapshot
- Line 414: Settings nav onClick — add `setTimeout(() => lucide.createIcons(), 100)` after `setView('settings')`
- Lines 442-444: pass `onMoveUp`/`onMoveDown` to the three ListEditor call sites
- Lines 434-440: legend — add Reorder entry

### Edit Anchors (verified line ranges — source of truth for the planner's grep-based review)

| Region | Lines | What lives there |
|--------|-------|------------------|
| `DEFAULT_*_OPTIONS` | 63-65 | `['WRD 5900', 'Other']`, `['Shafiq', 'Other']`, `['DJI Osmo Action 6', 'Car', 'Other']` |
| `getEquipDisplay` | 89 | `o === 'Car' ? 'Vehicles' : o` |
| Icon component | 92-95 | `<i data-lucide={name.toLowerCase()}>` — needs createIcons |
| ListEditor definition | 100-130 | rows keyed `key={i}`; edit/delete buttons lines 117-118 with `o === 'Other'` disabled pattern |
| Option state | 157-159 | `useState(DEFAULT_*)` for the three arrays |
| Requests onSnapshot | 206 | **only** createIcons call site today: `setTimeout(() => lucide.createIcons(), 100);` |
| Options onSnapshot | 207 | re-fills the three arrays; **no createIcons** (Pitfall 1) |
| saveOptions | 235-238 | `.set(..., { merge: true })`, signature `(plat, approver, equip)` |
| addOption | 240-246 | line 244 `next[list] = [...next[list], v];` — appends AFTER `'Other'` (Pitfall 3) |
| editOption | 248-253 | map-replace by index |
| removeOption | 255-259 | line 257 `if (next[list][idx] === 'Other') return;` |
| Nav buttons | 409-414 | Form (409), Manager (410), Settings (414) — gates + view switching |
| Settings view | 429-454 | header legend 434-440; three ListEditor calls 442-444; passcode card 445-453 |
| Form selects | 346, 350 | equipment / plat — consume shared state |
| Manager filter select | 470 | equipment — consumes shared state |
| Approver selects | 524, 667 | manager card + detail modal — consume shared state |

### Pattern 1: Firestore-Echo Mutation (the established mutation pattern — reorder MUST follow it)

**What:** Mutations never call `setState` on the option arrays directly. They build a `next` map from current state, mutate it, and call `saveOptions()`; the options `onSnapshot` echo (line 207) then re-fills state and re-renders every surface. This is byte-identical to `addOption`/`editOption`/`removeOption` (lines 240-259).

**When to use:** For `moveOption` — the UI-SPEC approved function (`04-UI-SPEC.md` §Reorder Function Pattern) already follows it:

```javascript
// Source: 04-UI-SPEC.md §Reorder Function Pattern (approved), placed in App scope after line 260
const moveOption = (list, fromIdx, toIdx) => {
    const next = { equipment: equipmentOptions, plat: platOptions, approver: approverOptions };
    const arr = [...next[list]];
    [arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]];
    next[list] = arr;
    saveOptions(next.plat, next.approver, next.equipment);
};
```

**Researched amendment (bounds guard — see Pitfall 2):** add before the swap:
```javascript
    if (toIdx < 0 || toIdx >= arr.length) return;
```
This is defense-in-depth: even if a future caller passes a bad index, the array and the Firestore doc can never gain an `undefined`/`null` element. It is free (no behavior change for correct callers) and costs one line.

**Row button JSX** (replaces lines 116-118 row content — two buttons inserted before the Edit button; only the non-editing branch, lines 114-120, changes):
```javascript
<span className="flex-1 text-sm text-white/80 truncate">{getEquipDisplay(o)}</span>
<button onClick={() => onMoveUp(i)} disabled={i === 0 || o === 'Other'} title={o === 'Other' ? 'Cannot reorder Other' : i === 0 ? 'Already at top' : 'Move up'} className={`p-2 rounded-lg transition-all border ${i === 0 || o === 'Other' ? 'bg-white/5 text-white/20 border-white/10 cursor-not-allowed' : 'bg-white/20 text-white/80 hover:bg-white/30 border border-white/30'}`}><Icon name="ChevronUp" size={14} /></button>
<button onClick={() => onMoveDown(i)} disabled={o === 'Other' || i >= items.length - 1 || items[i+1] === 'Other'} title={o === 'Other' ? 'Cannot reorder Other' : i >= items.length - 1 ? 'Already at bottom' : 'Move down'} className={`p-2 rounded-lg transition-all border ${o === 'Other' || i >= items.length - 1 || items[i+1] === 'Other' ? 'bg-white/5 text-white/20 border-white/10 cursor-not-allowed' : 'bg-white/20 text-white/80 hover:bg-white/30 border border-white/30'}`}><Icon name="ChevronDown" size={14} /></button>
```
(`items[i+1] === 'Other'` clause retained per UI-SPEC; note the `i >= items.length - 1` clause is **the researched amendment** in both disable conditions and tooltips — UI-SPEC text only lists `i === 0` and `items[i+1] === 'Other'`.)

Disabled-state matrix (amended — the plan-checker should verify against this):

| Row state | Up disabled | Down disabled | Tooltip |
|-----------|-------------|---------------|---------|
| `o === 'Other'` (always) | ✅ | ✅ | "Cannot reorder Other" |
| First item (i === 0, non-Other) | ✅ | per normal rule | "Already at top" |
| Last item (i === length-1, non-Other) | per normal rule | ✅ | "Already at bottom" |
| Next item is `'Other'` (non-Other row) | per normal rule | ✅ | "Move down" (hover text — boundary case shows disabled) |
| Else | — | — | "Move up" / "Move down" |

### Anti-Patterns to Avoid

- **Optimistic local `setState` on the option arrays:** would fight the onSnapshot echo and can be overwritten by a slower echo; the project pattern is mutate → persist → echo (PITFALLS.md §Firestore `.set()`).
- **Reordering via sort/filter on the rendered array:** the state arrays are the source of truth; single adjacent swap, then persist — don't re-sort on every render.
- **"Cleaning up" the createIcons timing:** moving `lucide.createIcons()` into an always-running effect causes layout thrashing (PITFALLS.md §Performance Traps); adding the scoped refresh to the two real trigger points is the minimal correct fix.
- **Using `.at(-1)` or spread-in-object ES2022 syntax:** classic-syntax-only for older Android stock browsers on staff phones; the codebase uses `arr[arr.length - 1]` style (see editOption line 251).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Icon materialization after Settings re-renders | A custom "icon system" or CSS sprites | `lucide.createIcons()` refresh (existing pattern) | PITFALLS.md Pitfall 5 — the app's icon path is a DOM-scan; adding the two scoped calls reuses the proven pattern |
| Order persistence / cross-device sync | localStorage mirrors, server round-trips, or local "pending order" state | Existing `saveOptions()` → Firestore → `onSnapshot` echo | Single source of truth; SORT-05/06 require every device to converge on the doc's arrays |
| Array index bookkeeping | Hand-rolled move normalization/re-sorting | Exact `i, i±1` swap with the two disabled-state guards + `if (toIdx < 0 || toIdx >= arr.length) return;` | Boundary mistakes are the highest-corruption risk in this phase (Pitfall 2) |

**Key insight:** this phase's complexity is not in the swap — it is in the *boundaries* (first/last/`'Other'`) and in the *icon side effect* (createIcons). Both are one-line guards; neither needs a library or a hand-built subsystem.

## Live Data State (observed in browser against deployed app, 2026-09-15)

The runtime `options/lists` Firestore doc currently holds (per the deployed app's rendered selects):

| List | Observed runtime order (raw values) | Note |
|------|--------------------------------------|------|
| Equipment | `['DJI Osmo Action 6', 'Car', 'Other', 'Laptop']` | **`'Other'` NOT last — a legacy `'Laptop'` entry was appended after it by the current `addOption`** |
| Plat | `['WRD 5900', 'Other']` | pinned, healthy |
| Approvers | `['Shafiq', 'Other', 'Abdil Haiyan']` | pinned, healthy |

Implications (drive the SORT-04 + SORT-05 acceptance tests):
- The verifier MUST test from this live state, not from clean defaults: Laptop shows Up enabled / Down disabled (it is last); `'Other'` shows both disabled; Car's Down is disabled (`items[1+1] === 'Other'`). Moving Laptop up once yields `['DJI Osmo Action 6', 'Car', 'Laptop', 'Other']` — restoring the pin.
- The plan MUST NOT hard-code `DEFAULT_EQUIPMENT_OPTIONS` in any verification step; use live Fed data (or seed the doc via the UI).

## Common Pitfalls

### Pitfall 1: New reorder icons render as empty/invisible (lucide.createIcons never runs for Settings)
**What goes wrong:** The ChevronUp/ChevronDown buttons (and today's Pencil/Trash2 in Settings) appear as blank icon areas. **Empirically verified live:** Settings currently shows 18 unmaterialized `<i data-lucide>` nodes (9× pencil, 9× trash2) and only 2 materialized SVGs; the Manager view shows 1 unmaterialized `download` icon (Export button).
**Why it happens:** `lucide.createIcons()` is called in exactly one place — inside the *requests* onSnapshot callback (line 206, `setTimeout(..., 100)`). The *options* onSnapshot (line 207) and the Settings nav (line 414) never trigger it, so any `<i data-lucide>` created by the Settings mount or list edits stays a bare tag.
**How to avoid:**
1. Add `setTimeout(() => lucide.createIcons(), 100);` at the end of the options onSnapshot callback (line 207) — covers first Settings mount and every add/edit/remove/move echo.
2. Add `setTimeout(() => lucide.createIcons(), 100);` to the Settings nav onClick after `setView('settings')` (line 414) — covers re-entering Settings when the doc did not change (no snapshot fires, but new `<i>` elements were mounted).
**Warning signs:** `document.querySelectorAll('i[data-lucide]')` non-empty without `svg` children after opening Settings; blank squares where arrows should be.

### Pitfall 2: Down arrow at the true last item enables an out-of-bounds swap → `undefined` element persisted to Firestore
**What goes wrong:** With legacy data (`'Laptop'` after `'Other'` — proven live), the last item's `items[i+1]` is `undefined`, not `'Other'`. The UI-SPEC's literal boundary rule (`down disabled only if `items[i+1] === 'Other'`) leaves Down **enabled** at the last index; clicking it runs `moveOption(list, last, last+1)`, and `[arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]]` writes `arr[length] = value` and `arr[last] = undefined`. `saveOptions` then persists an array with `undefined` → JSON `null` → every select gains a bogus `null` option.
**Why it happens:** The UI-SPEC assumed `'Other'` is always last (its E3 claim), which the live data contradicts; no `i === items.length - 1` clause exists in the approved rule.
**How to avoid:** Apply the amendment — Down disabled when `o === 'Other' || i >= items.length - 1 || items[i+1] === 'Other'`; add the `if (toIdx < 0 || toIdx >= arr.length) return;` guard in `moveOption`. Verify by testing against the LIVE list state (Laptop last → Down disabled, tooltip "Already at bottom"), then fix order via Laptop's Up and re-verify.
**Warning signs:** A `null`/`undefined` `<option>` appears in any select after clicking Down; array length grew by one after a move.

### Pitfall 3: addOption appends after 'Other', silently breaking the SORT-04 pin on every new item
**What goes wrong:** SORT-04 ("Other stays pinned as the last item") and UI-SPEC E3 ("always last") fail the moment a manager adds an item: line 244 `next[list] = [...next[list], v];` appends after `'Other'`. The live `'Laptop'` entry is this bug already burned into production data.
**Why it happens:** The reorder boundary rules can never *move* anything below `'Other'`, but there is no rule stopping *adds* from landing below it.
**How to avoid ([ASSUMED] — needs sign-off, Q2):** replace line 244 with insert-before-`'Other'`:
```javascript
const last = next[list][next[list].length - 1];
next[list] = last === 'Other' ? [...next[list].slice(0, -1), v, 'Other'] : [...next[list], v];
```
(Classic syntax on purpose — no `.at()`.) If the user declines this change, SORT-04 acceptance must be scoped to *moves only* and the Q2 record kept.
**Warning signs:** After adding an item in Settings, the new item renders below the disabled `'Other'` row.

### Pitfall 4: Multi-click / rapid-click races on slow LAN echo
**What goes wrong:** Each click reads the arrays from the current render closure; two rapid clicks before the Firestore echo re-renders compute two swaps from the same snapshot. Both call `saveOptions` — the second write wins; outcome is idempotent for a single back-and-forth pair, but consecutive same-direction clicks appear to "miss" a step.
**Why it happens:** No `enablePersistence()`, no optimistic setState — the echo round-trip is 50-300 ms on LAN (matches the existing add/edit/delete UX exactly).
**How to avoid:** Do NOT add optimistic state or click-debouncing (behavior change vs. existing buttons). Document the echo latency as accepted; verify with a reload after a burst — persisted order equals the last completed write. This is identical to the established mutation pattern; consistency wins over freshness here.
**Warning signs:** Arrow clicks seem "slow"; a double-click moves one position instead of two (expected — matches edit/save behavior).

### Pitfall 5: Reorder buttons appearing during the row's edit mode
**What goes wrong:** The editing branch (lines 108-113) renders `[input][Save][Cancel]` instead of the label row — if the new Up/Down buttons are placed outside the `isEditing` conditional, they'd render next to the input (clutter) or break the Save/Cancel layout.
**Why it happens:** The two-button insertion must live inside the non-editing fragment (lines 114-120), mirroring where Edit/Delete already are.
**How to avoid:** Insert the Up/Down buttons between the `<span>` (line 116) and the Edit button (line 117), keeping them inside the same `<>...</>` fragment. No edit-mode changes.
**Warning signs:** Chevron icons visible while typing in an edit input.

## Code Examples

### Adding moveOption (App scope, after removeOption ~line 260)
```javascript
// Source: 04-UI-SPEC.md §Reorder Function Pattern (approved) + researched bounds guard
const moveOption = (list, fromIdx, toIdx) => {
    const next = { equipment: equipmentOptions, plat: platOptions, approver: approverOptions };
    const arr = [...next[list]];
    if (toIdx < 0 || toIdx >= arr.length) return;   // research amendment — see Pitfall 2
    [arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]];
    next[list] = arr;
    saveOptions(next.plat, next.approver, next.equipment);
};
```

### Wiring the three call sites (lines 442-444 — one line each; pattern shown for equipment)
```javascript
<ListEditor ... listKey="equipment" items={equipmentOptions} ... onMoveUp={(i) => moveOption('equipment', i, i - 1)} onMoveDown={(i) => moveOption('equipment', i, i + 1)} ... />
```

### Legend insertion (Settings header legend, lines 434-440)
Per UI-SPEC §Visual Legend Update, a "Reorder" entry with a neutral glass dot is added. **Placement flag (Q3):** the UI-SPEC text says "between Edit and Delete", but the row order is `[Up][Down][Edit][Delete]` — recommend inserting the Reorder entry **before** the Edit entry (line 435) for row-order fidelity; either placement is visually harmless (E4 wrap verified).
```javascript
<div className="flex flex-col items-center gap-1"><span>Reorder</span><div className="w-3 h-3 rounded-full bg-white/20 border border-white/40"></div></div>
```

### Verified reproduction of Pitfall 1 (for the verifier)
Open Settings → DevTools console:
```javascript
[...document.querySelectorAll('i[data-lucide]')].map(i => i.getAttribute('data-lucide'))
// BEFORE fix on a fresh Settings entry: ["pencil","trash2", ... 18 entries]
// AFTER fix: []  (all converted to <svg class="lucide ...">)
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Static option order (v1.0) | Firestore-persisted order with live sync (Phase 3) | 2026-09-14 | This phase adds manager-driven reordering on top of that layer — no storage-design change |
| No reorder controls | Up/down glass buttons (Phase 4, UI-SPEC approved) | 2026-09-15 | Drag-and-drop is explicitly deferred to v2 (SORT-07) as a desktop-first gesture |

**Deprecated/outdated:**
- **Drag-and-drop reordering**: deliberately out of scope (REQUIREMENTS.md v2) — not touch-safe for LAN staff phones and would fight the current keyed-row reconciliation.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | Live `options/lists` doc content (`['DJI Osmo Action 6','Car','Other','Laptop']`, `['WRD 5900','Other']`, `['Shafiq','Other','Abdil Haiyan']`) — observed from the rendered selects in one browser session | Live Data State | Content can drift; the plan must test against whatever is in the doc at execution time, never hard-code the observed arrays (initial-defaults fallback verified at lines 157-159/63-65) |
| A2 | The `i >= items.length - 1` boundary amendment is required to avoid array corruption (derived from code semantics + live data, not from the UI-SPEC text) | Pitfall 2 | If omitted and UI-SPEC is executed literally, the legacy state corrupts the Equipment array with `null` on a Down click — the phase fails SORT-04/05 acceptance |
| A3 | `addOption` insert-before-`'Other'` change (line 244) is required for SORT-04 to hold across adds | Pitfall 3 | If declined, acceptance must be re-scoped to moves-only, and the legacy-pin violation persists for any future add |
| A4 | Node 24.18.0 / npm 11.16.0 availability (verified earlier in the working session via `node --version` / `npm --version`) | Environment Availability | Only used for `gsd-tools`; the app itself requires only Python 3.12 (verified live) and a browser |
| A5 | Reorder legend entry placement (before Edit vs. between Edit and Delete per UI-SPEC literal text) | Code Examples | Cosmetic only (E4 wrap verified) — no functional impact whichever is chosen |

## Open Questions (RESOLVED)

1. **Accept the Down-boundary amendment?** (`i >= items.length - 1` in the disabled condition + `moveOption` bounds guard)
   - What we know: Proven necessary — live data has a non-`'Other'` tail (`'Laptop'`), and the UI-SPEC's literal rule enables a corruption swap at the last index (Pitfall 2).
   - What's unclear: The UI-SPEC is APPROVED; amending it deviates from the locked visual/interaction contract.
   - Recommendation: **Accept** — it changes no enabled-state UX for pinned-healthy lists; it only disables a button that would corrupt data. Record as a UI-SPEC amendment note in the plan.
2. **Accept the `addOption` insert-before-`'Other'` change?** (line 244)
   - What we know: Current append already produced production data with `'Other'` non-last (`'Laptop'`); SORT-04 textually requires the pin; UI-SPEC E3 claims it.
   - What's unclear: It modifies Phase 3 shipped behavior (items no longer land at the literal end of the stored array).
   - Recommendation: **Accept**, and decide in the plan whether the legacy `'Laptop'` row is normalized via a one-time manual step (Laptop ▲ once) or left as-is (Q4).
3. **Legend placement** — UI-SPEC says "between Edit and Delete"; row order is `[Up][Down][Edit][Delete]`.
   - Recommendation: insert before Edit for order fidelity; whichever chosen is cosmetic-only.
4. **Legacy data normalization (`'Other'` not last in the live Equipment doc):** normalize on load (auto reorder + persist) vs. leave for one-time manual up-move.
   - What we know: Boundary rules prevent worsening; Laptop's enabled Up button restores the pin; no load-time normalization exists anywhere in the app today.
   - Recommendation: **Do not auto-normalize** (silent data mutation on load would surprise managers and is a behavior change beyond this phase). Verify the fix via the manual step during UAT.

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Python 3.12 static server | App serving (LAN) | ✓ (verified: PID 19720 listening on :8080, HTTP 200, serves index.html 71,928 bytes) | 3.12.x (`C:\Users\KWM-ACCOUNT\AppData\Local\Programs\Python\Python312`) | — (already serving the deployment) |
| Browser (Firefox/Chromium preview) | UAT / icon verification | ✓ (collaborative preview used for the Pitfall-1/2 experiments) | current | DevTools console on staff phone |
| Firebase project (`camera-borrow-wiramas`) | All persistence | ✓ (app loaded live data; passcode unlock `1234` verified) | compat v11.6.1 | — |
| Node.js / npm | `gsd-tools` research/commit tooling only | ✓ | 24.18.0 / 11.16.0 `[ASSUMED A4]` | n/a for app runtime |
| Network reachability of CDNs (unpkg, gstatic, cdn.tailwindcss.com) | App boot | ✓ (app rendered; icons script loaded) | pins from STACK.md | — |

**Missing dependencies with no fallback:** none.
**Missing dependencies with fallback:** none.

## Validation Architecture

`workflow.nyquist_validation` is `true` in `.planning/config.json` → this section is required.

### Test Framework
| Property | Value |
|----------|-------|
| Framework | none — no test framework, no `package.json`, no test runner in the repo (verified: zero `*.test.*`/`*.spec.*`/config files) |
| Config file | none |
| Quick run command | Browser preview of `http://localhost:8080` + DevTools assertions (see requirement map) |
| Full suite command | UAT checklist (STACK.md "Verify after each extraction step" list, extended with reorder steps below) |

### Phase Requirements → Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| SORT-01 | Equipment ▲/▼ moves item; form select shows new order immediately | manual + console | Settings → Equipment List → click ▲ on rows 2+; assert form select (line 346) order matches | ❌ manual (no harness) |
| SORT-02 | Plat ▲/▼ reorders; plat select follows | manual | Vehicle Plates card ↔ request form plat select (line 350) | ❌ manual |
| SORT-03 | Approvers ▲/▼ reorders; approver select in cards/modal follows | manual | Approvers card ↔ manager card select (524) + detail modal (667) | ❌ manual |
| SORT-04 | `'Other'` immovable, pinned last after any move sequence + after add (if Q2 accepted) | manual | Live list: Laptop ▲ once → `'Other'` last; assert both arrows of `'Other'` disabled; add item → appears above `'Other'` | ❌ manual |
| SORT-05 | Order persists to Firestore and syncs across surfaces/devices | manual | Reload page; open second browser tab; assert both render saved order; Network tab shows single `options/lists` write per move | ❌ manual |
| SORT-06 | Order survives reload for every user | manual | After SORT-05 reload, order identical; verify no localStorage involvement (`kwm_*` keys are dark-mode/seen-requests only) | ❌ manual |
| (regression) | Lucide icons materialize in Settings incl. new chevrons | console assertion | `[...document.querySelectorAll('i[data-lucide]')]` empty after opening Settings twice (Pitfall 1 reproduction) | ❌ manual |

### Sampling Rate
- **Per task commit:** browser reload + Settings icon check + one move + one surface check (form select / approver select)
- **Per wave merge:** full UAT list — submit form, manager passcode, approve, filter, CSV export, copy details, dark toggle, return flow (STACK.md) **plus** the SORT-01..06 rows above
- **Phase gate:** full suite green before `/gsd-verify-work` (same manual UAT; no automated gate exists)

### Wave 0 Gaps
- [ ] No test framework exists (repo is a no-build single-file CDN app — this is by design). The plan should treat the DevTools-assertion snippets in RESEARCH (Pitfall 1 reproduction; select-order checks) as Wave-0 verification scaffolding.

*(If no gaps: "None — existing test infrastructure covers all phase requirements" — this does NOT apply here; the gap list above is the honest state.)*

## Security Domain

`workflow.security_enforcement` is `true`, `security_asvs_level: 1` → required.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | yes | Unchanged: Firebase anonymous auth + passcode-gated manager view (`isManagerAuthenticated`, line 410/414 gate; unlock flow sets view at line 561). Reorder controls are unreachable until authenticated. |
| V3 Session Management | yes (inherited) | No session mechanism changes; `localStorage` remains limited to dark-mode/seen-requests (lines 204, 208) — order state must NOT be added there (session-independent correctness) |
| V4 Access Control | yes | Settings nav rendered only when `isManagerAuthenticated` (line 414); passcode stored in and compared via the same `options/lists` Firestore doc (line 207 `d.passcode`, save flow lines 267-278) — no new elevation path |
| V5 Input Validation | yes | `moveOption` index validation (`toIdx < 0 || toIdx >= arr.length` guard) prevents out-of-bounds array corruption; `addOption` already trims + duplicate-checks (`const v = (value || '').trim(); if (!v) return;` line 241; `includes(v)` line 243) |
| V6 Cryptography | no | No new crypto; passcode travels over LAN HTTP (documented accepted risk, PITFALLS.md §Security Mistakes) |

### Known Threat Patterns for this stack

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| Out-of-bounds array write via Down on legacy non-`'Other'` tail → persisted `undefined`/`null` option and corrupted list order | Tampering (integrity) | Disabled-state boundary rule incl. `i >= items.length - 1` + `moveOption` bounds guard (both plan-wide, verifiable by grep) |
| Duplicate `'Other'` created via editOption rename (pre-existing, unchanged this phase) | Tampering | Out of scope (pre-existing Phase 3 gap at line 248-253); note in plan as known-risk, do not fix during reorder phase |
| Client-side passcode visible in DevTools / over HTTP | Information Disclosure | Accepted for the internal LAN deployment (PITFALLS.md §Security Mistakes); Firestore security rules remain the real boundary — no rule change this phase |

## Sources

### Primary (HIGH confidence)
- `index.html` — full Read this session (lines 56-95, 97-144, 144-215, 230-284, 424-463) + Grep-verified anchors (206, 207, 235-259, 346, 350, 409-414, 470, 524, 667). All quoted code verbatim from these reads.
- Live browser experiment (2026-09-15, http://localhost:8080, preview browser): unlocked manager view (passcode 1234), read runtime select order, counted unmaterialized `<i data-lucide>` nodes (18 in Settings, 1 in Manager). HIGH — primary observation.
- `04-UI-SPEC.md` (260 lines, APPROVED by checker v2026-09-15) — reorder contract quoted verbatim (§Interaction Contract, §Reorder Function Pattern, §Button Color Map).
- `REQUIREMENTS.md` (SORT-01..06) and `ROADMAP.md` (Phase 4 success criteria ×5) — requirement anchors.
- `.planning/config.json` — `nyquist_validation: true`, `security_enforcement: true`, `commit_docs: true`.
- `.planning/research/PITFALLS.md` — Pitfall 5 (lucide timing) corroborates Pitfall 1 of this research; §Security Mistakes corroborates the HTTP/passcode risk posture.

### Secondary (MEDIUM confidence)
- Firebase official docs, "Add data" page (set-with-merge semantics; `arrayUnion`/`arrayRemove` exist because plain set/update replaces arrays wholesale) — `[CITED: firebase.google.com/docs/firestore/manage-data/add-data]`; confirms `saveOptions` wholesale-replace behavior (line 237 `merge: true`).
- `research/STACK.md` — version pins and CDN verifications from the v1.0 milestone `[CITED]`.

### Tertiary (LOW confidence)
- Node/npm version numbers (24.18.0 / 11.16.0) — recorded earlier in the working session, not re-probed this run `[ASSUMED A4]`.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — locked CDN set, no new packages; pins from STACK.md verified in v1.0 milestone.
- Architecture: HIGH — every anchor and mutation-pattern claim verified by Read/Grep this session; the Firestore-echo propagation is observed in the running app.
- Pitfalls: HIGH — Pitfalls 1 and 2 are empirically demonstrated against the live deployment, not hypothetical; Pitfall 3 is evidenced by the live `'Laptop'` record.
- Amendments to the approved UI-SPEC: MEDIUM (confident they are necessary, but they deviate from a locked contract — flagged for user sign-off in Q1/Q2).

**Research date:** 2026-09-15
**Valid until:** 2026-10-15 (30 days — stable repo, pinned CDNs; the only drift risk is the live `options/lists` doc content, see A1)