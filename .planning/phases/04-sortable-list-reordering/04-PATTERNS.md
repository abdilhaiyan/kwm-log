# Phase 4: Sortable List Reordering - Pattern Map

**Mapped:** 2026-09-16
**Files analyzed:** 1 modified (index.html) — all 7 change sites inside it
**Analogs found:** 7 / 7 (every change site has an exact analog — this phase copies the file's own established idioms)

## File Classification

| Change Site (in `index.html`) | Role | Data Flow | Closest Analog | Match Quality |
|---|---|---|---|---|
| `moveOption` function (NEW, inserted after line 260) | service (mutation handler) | CRUD + event-driven echo | `addOption`/`editOption`/`removeOption` (lines 240-259) | exact |
| ListEditor signature destructure (line 100) | component | request-response | Same line — add `onMoveUp`/`onMoveDown` to existing prop list | exact |
| Up/Down row buttons (NEW, inserted between line 116 and 117) | component | event-driven | Edit/Delete buttons (lines 117-118) | exact |
| ListEditor call sites (lines 442-444) | component wiring | request-response | Same lines — add two props per call | exact |
| Legend "Reorder" entry (NEW, inserted before line 435) | component (presentational) | n/a | Legend entries at lines 435-438 | exact |
| `lucide.createIcons()` refresh (ADD to line 207 + line 414) | utility side-effect | event-driven | Line 206 (requests onSnapshot) | exact |
| `addOption` insert-before-`'Other'` change (line 244) | service (mutation handler) | CRUD | Same function (self-amendment) | exact |

**Context:** The repo is a single-file React app (694 lines, `index.html`, 7 `text/babel` blocks). ALL of this phase's work lands inside the main App block (starts line 97) of that one file. There is no other source file, no test framework, no package.json — verification is browser/DevTools manual (see RESEARCH §Validation Architecture). The phase is a pure re-skin of existing mutation/list idioms; no new dependency, no schema change.

---

## Pattern Assignments

### 1. `moveOption` — the Firestore-Echo Mutation pattern (the phase's core pattern)

**Analog:** `addOption` / `editOption` / `removeOption` (index.html:240-259) — same role (option-array mutation), same data flow (mutate → `saveOptions()` → `onSnapshot` echo). `moveOption` MUST be byte-identical in structure: build `next` map from shared state → mutate copy → call `saveOptions(next.plat, next.approver, next.equipment)`. **NEVER call `setState` on the option arrays** (optimistic local state fights the echo and can be overwritten — RESEARCH Anti-Patterns).

**The template the new function copies — verbatim from index.html:**

```javascript
// lines 240-246 — addOption (append; will be AMENDED at line 244, see §7)
const addOption = (list, value) => {
    const v = (value || '').trim(); if (!v) return;
    const next = { equipment: equipmentOptions, plat: platOptions, approver: approverOptions };
    if (next[list].includes(v)) { handleNotify("Already in list", "error"); return; }
    next[list] = [...next[list], v];
    saveOptions(next.plat, next.approver, next.equipment);
    setNewItem(p => ({ ...p, [list]: '' }));
};

// lines 248-253 — editOption (classic map-replace — NO .at(), NO filter-with-index tricks)
const editOption = (list, idx, value) => {
    const v = (value || '').trim(); if (!v) return;
    const next = { equipment: equipmentOptions, plat: platOptions, approver: approverOptions };
    next[list] = next[list].map((o, i) => i === idx ? v : o);
    saveOptions(next.plat, next.approver, next.equipment);
    setEditingList(null);
};

// lines 255-259 — removeOption (contains the 'Other'-immutability guard precedent, line 257)
const removeOption = (list, idx) => {
    const next = { equipment: equipmentOptions, plat: platOptions, approver: approverOptions };
    if (next[list][idx] === 'Other') return;
    next[list] = next[list].filter((_, i) => i !== idx);
    saveOptions(next.plat, next.approver, next.equipment);
};
```

**The destination — approved signature (UI-SPEC §Reorder Function Pattern) + researched bounds guard:**

```javascript
// Source: 04-UI-SPEC.md §Reorder Function Pattern (approved) + 04-RESEARCH.md bounds amendment
// Placement: App scope, between removeOption's closing `};` (line 260) and confirmDelete (line 261)
const moveOption = (list, fromIdx, toIdx) => {
    const next = { equipment: equipmentOptions, plat: platOptions, approver: approverOptions };
    const arr = [...next[list]];
    if (toIdx < 0 || toIdx >= arr.length) return;   // researched amendment — see RESEARCH Pitfall 2
    [arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]];
    next[list] = arr;
    saveOptions(next.plat, next.approver, next.equipment);
};
```

Key structural elements to preserve (borrowed from lines 240-259):
- **`next` map idiom**: `{ equipment: equipmentOptions, plat: platOptions, approver: approverOptions }` — reads the three shared state arrays inside the render closure.
- **Classic-syntax array copy**: `[...next[list]]` (ES2015 spread — used at lines 244, 251, 258; object spread `{ ...p }` used throughout, e.g. line 246). Allowed. NOT allowed: `.at(-1)` (use `arr[arr.length - 1]`), ES-module `import`/`export`.
- **Destructuring swap**: `[arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]]` (ES2015 destructuring assignment — fine for the target browsers).
- **No `setState`**: the echo (options onSnapshot, line 207) re-fills state. This is how SORT-05/06 propagate order to all 5 consuming surfaces (lines 346, 350, 470, 524, 667).
- **Error handling**: none added — `saveOptions` (235-238) already wraps the write in try/catch with `handleNotify("Error saving lists", "error")`. UI-SPEC §Copywriting explicitly: no new error states.

**saveOptions — the persistence half of the echo (lines 235-238, verbatim):**

```javascript
const saveOptions = async (plat, approver, equip) => {
    if (!db) return;
    try { await db.collection('artifacts/' + appId + '/public/data/options').doc('lists').set({ platOptions: plat, approverOptions: approver, equipmentOptions: equip }, { merge: true }); }
    catch (err) { console.error("Options save error:", err); handleNotify("Error saving lists", "error"); }
};
```

**State + hydration context (do not change):**
- State arrays: line 157 `useState(DEFAULT_PLAT_OPTIONS)`, 158 `approverOptions`, 159 `equipmentOptions`.
- Defaults: lines 63-65 (`['WRD 5900', 'Other']` / `['Shafiq', 'Other']` / `['DJI Osmo Action 6', 'Car', 'Other']`) — fallback only when the Firestore doc is missing.
- Echo: line 207 onSnapshot → `setPlatOptions`/`setApproverOptions`/`setEquipmentOptions`.
- **Verification warning (RESEARCH A1):** live doc order differs from defaults (`['DJI Osmo Action 6', 'Car', 'Other', 'Laptop']` etc.) — never hard-code the defaults in acceptance criteria.

---

### 2. ListEditor row layout — the editing-vs-display branch and button disabled-state pattern

**Analog:** The Edit/Delete buttons (lines 117-118) inside the non-editing fragment (lines 114-120). The new Up/Down buttons copy this exact pattern: `p-2 rounded-lg transition-all border` sizing, conditional-template-literal class swap, `o === 'Other'` disabled guard, `title` tooltip with ternary.

**Component signature (line 100) — `onMoveUp`/`onMoveDown` must be added to this destructure:**

```javascript
const ListEditor = ({ title, listKey, items, newVal, setNewVal, editing, setEditing, onAdd, onEdit, onRemove, onRequestDelete, placeholder }) => {
```

→ becomes (add the two new props; keep order readable, e.g. after `onRemove`):

```javascript
const ListEditor = ({ title, listKey, items, newVal, setNewVal, editing, setEditing, onAdd, onEdit, onRemove, onMoveUp, onMoveDown, onRequestDelete, placeholder }) => {
```

**The row branch (lines 107-122, verbatim) — Up/Down buttons go INSIDE the non-editing fragment (114-120), between the `<span>` (116) and the Edit button (117). NOT in the editing branch (108-113) — RESEARCH Pitfall 5:**

```html
{items.map((o, i) => (
    <div key={i} className="flex items-center gap-2">
        {isEditing && editing.idx === i ? (
            <>
                <input autoFocus className={INPUT_CLS_SM + " flex-1"} value={editing.val} onChange={e => setEditing({ ...editing, val: e.target.value })} onKeyDown={e => { if (e.key === 'Enter') onEdit(i, editing.val); }} />
                <button onClick={() => onEdit(i, editing.val)} title="Save" className="p-2 rounded-lg bg-emerald-500/20 text-emerald-300 hover:bg-emerald-500/30 transition-all border border-emerald-400/30"><Icon name="Check" size={14} /></button>
                <button onClick={() => setEditing(null)} title="Cancel" className="p-2 rounded-lg bg-orange-500/20 text-orange-300 hover:bg-orange-500/30 transition-all border border-orange-400/30"><Icon name="X" size={14} /></button>
            </>
        ) : (
            <>
                <span className="flex-1 text-sm text-white/80 truncate">{getEquipDisplay(o)}</span>
                <button onClick={() => setEditing({ list: listKey, idx: i, val: o })} disabled={o === 'Other'} title={o === 'Other' ? 'Cannot edit Other' : 'Edit'} className={`p-2 rounded-lg transition-all border ${o === 'Other' ? 'bg-white/5 text-white/20 border-white/10 cursor-not-allowed' : 'bg-blue-500/20 text-blue-300 hover:bg-blue-500/30 border-blue-400/30'}`}><Icon name="Pencil" size={14} /></button>
                <button onClick={() => onRequestDelete(i, o)} disabled={o === 'Other'} title={o === 'Other' ? 'Cannot delete Other' : 'Delete'} className={`p-2 rounded-lg transition-all border ${o === 'Other' ? 'bg-white/5 text-white/20 border-white/10 cursor-not-allowed' : 'bg-red-500/20 text-red-300 hover:bg-red-500/30 border-red-400/30'}`}><Icon name="Trash2" size={14} /></button>
            </>
        )}
    </div>
))}
```

**The glass-button class tuple the Up/Down buttons must use (UI-SPEC §Button Color Map):**

| State | Classes (copy exact string) | Source line |
|---|---|---|
| Enabled (Up/Down — neutral white glass) | `bg-white/20 text-white/80 hover:bg-white/30 border border-white/30` | UI-SPEC §Button Color Map (same tuple as ADD button line 126, COPY INFO line 533) |
| Disabled (all cases: `'Other'`, first-up, last-down) | `bg-white/5 text-white/20 border-white/10 cursor-not-allowed` | lines 117-118 ternary |
| Sizing (all 4 buttons share this) | `p-2 rounded-lg transition-all border` | lines 117-118 |

**New row buttons (researched + amended — 04-RESEARCH.md lines 174-175; insert between the `<span>` line 116 and the Edit button line 117):**

```html
<button onClick={() => onMoveUp(i)} disabled={i === 0 || o === 'Other'} title={o === 'Other' ? 'Cannot reorder Other' : i === 0 ? 'Already at top' : 'Move up'} className={`p-2 rounded-lg transition-all border ${i === 0 || o === 'Other' ? 'bg-white/5 text-white/20 border-white/10 cursor-not-allowed' : 'bg-white/20 text-white/80 hover:bg-white/30 border border-white/30'}`}><Icon name="ChevronUp" size={14} /></button>
<button onClick={() => onMoveDown(i)} disabled={o === 'Other' || i >= items.length - 1 || items[i+1] === 'Other'} title={o === 'Other' ? 'Cannot reorder Other' : i >= items.length - 1 ? 'Already at bottom' : 'Move down'} className={`p-2 rounded-lg transition-all border ${o === 'Other' || i >= items.length - 1 || items[i+1] === 'Other' ? 'bg-white/5 text-white/20 border-white/10 cursor-not-allowed' : 'bg-white/20 text-white/80 hover:bg-white/30 border border-white/30'}`}><Icon name="ChevronDown" size={14} /></button>
```

**Amended disabled-state matrix (plan-checker must verify against this — RESEARCH, not UI-SPEC literal):**

| Row state | Up disabled | Down disabled | Tooltip |
|---|---|---|---|
| `o === 'Other'` (always) | ✅ (`o === 'Other'`) | ✅ (`o === 'Other'`) | "Cannot reorder Other" |
| First item (i === 0, non-Other) | ✅ (`i === 0`) | per normal rule | "Already at top" |
| Last item (i === length-1, non-Other) | per normal rule | ✅ (`i >= items.length - 1`) | "Already at bottom" |
| Next item is `'Other'` | per normal rule | ✅ (`items[i+1] === 'Other'`) | "Move down" (hover shows disabled control) |
| Else | — | — | "Move up" / "Move down" |

The `i >= items.length - 1` clauses are the **researched amendment** (RESEARCH Pitfall 2 — live data has `'Laptop'` after `'Other'`; without them a Down click at the last index writes `undefined` → persists `null` into the doc). UI-SPEC literally only lists `i === 0` and `items[i+1] === 'Other'` — the plan must note the amendment.

**Do-not-touch constraints on this block:**
- Rows stay keyed `key={i}` (line 107) — index keys are fine for adjacent swaps; do not "improve" to value keys.
- Do NOT add reorder controls to the editing branch (108-113).
- Keep existing `gap-2` row layout and `p-2` button sizing (~30×30px touch target, locked by UI-SPEC §Spacing exceptions).
- `getEquipDisplay(o)` (line 89: `o === 'Car' ? 'Vehicles' : o`) stays the label renderer.

---

### 3. Lucide icon pattern (`Icon` component + `createIcons()` timing)

**Analog:** The `Icon` component (lines 92-95) and its single existing refresh call site (line 206). The new ChevronUp/ChevronDown buttons render through the same `Icon` component. **Critical (RESEARCH Pitfall 1, empirically verified):** `lucide.createIcons()` currently runs ONLY in the requests onSnapshot (line 206), never for Settings — the new arrows will be invisible unless two scoped `createIcons()` calls are added.

**Icon component (lines 92-95, verbatim — do not change):**

```javascript
const Icon = ({ name, size = 20, className = "" }) => {
    const LucideIcon = lucide.icons[name];
    return LucideIcon ? <i data-lucide={name.toLowerCase()} className={className} style={{width: size, height: size}}></i> : null;
};
```

New buttons use it exactly like the existing ones: `<Icon name="ChevronUp" size={14} />` / `<Icon name="ChevronDown" size={14} />`.

**The refresh pattern to replicate (line 206, verbatim — the ONLY existing call site):**

```javascript
setTimeout(() => lucide.createIcons(), 100);
```

**Why `setTimeout(..., 100)` (PITFALLS.md Pitfall 5 / RESEARCH Anti-Patterns):** the `Icon` component renders `<i data-lucide>` tags; lucide DOM-scans and replaces them with `<svg>` only when `createIcons()` runs after React commits. Calling it synchronously after `setState` is too early (React hasn't committed). An always-running effect causes layout thrashing. The established idiom is a scoped 100 ms refresh at each real trigger point. **Do not "clean this up".**

**The two NEW call sites (both mandatory — one is mount via snapshot, one is re-entry via nav):**

1. **End of the options onSnapshot callback (line 207)** — covers first Settings mount (snapshot fires on subscribe) AND every add/edit/remove/move echo:

```javascript
// line 207 today (verbatim) — ADD the setTimeout line at the end of the callback
useEffect(() => { if (!user || !db) return; const ref = db.collection('artifacts/' + appId + '/public/data/options').doc('lists'); return ref.onSnapshot(doc => { if (doc.exists) { const d = doc.data(); setPlatOptions(d.platOptions || DEFAULT_PLAT_OPTIONS); setApproverOptions(d.approverOptions || DEFAULT_APPROVER_OPTIONS); setEquipmentOptions(d.equipmentOptions || DEFAULT_EQUIPMENT_OPTIONS); if (d.passcode) setAppPasscode(d.passcode); } /* ADD HERE: setTimeout(() => lucide.createIcons(), 100); */ }, err => console.error("Options load error:", err)); }, [user]);
```

2. **Settings nav onClick (line 414)** — covers re-entering Settings when the doc did not change (no snapshot fires, but the `<i>` elements mount fresh):

```javascript
// line 414 today (verbatim) — ADD the setTimeout after setView('settings')
{isManagerAuthenticated && <button onClick={() => setView('settings')} className={`flex-1 md:flex-none px-4 md:px-6 py-2.5 rounded-xl text-[11px] md:text-xs font-bold transition-all ${view === 'settings' ? 'bg-white/80 dark:bg-white/20 text-slate-900 dark:text-white' : 'text-white/60 hover:text-white/80'}`}>Settings</button>}
```

Note for the planner: do NOT add the refresh to the Manager nav (line 410) or the passcode-unlock view switch (line 561) — those paths are covered by the request snapshot (line 206) and by the options snapshot firing on mount (line 207), respectively. Two additions only.

**Verifier console assertion (RESEARCH Pitfall 1 reproduction):** `[...document.querySelectorAll('i[data-lucide]')]` must be `[]` after opening Settings twice (all tags materialized to `<svg class="lucide ...">`).

---

### 4. Call-site wiring pattern (lines 442-444)

**Analog:** The three existing `<ListEditor ...>` call sites — each gains `onMoveUp`/`onMoveDown` props mirroring how `onAdd`/`onEdit`/`onRemove` are wired. Each new prop is an arrow that closes over the list key:

**Pattern (line 442, verbatim today — equipment shown; 443 plat / 444 approver identical shape):**

```html
<ListEditor title="Equipment List" listKey="equipment" items={equipmentOptions} newVal={newItem.equipment} setNewVal={v => setNewItem(p => ({ ...p, equipment: v }))} editing={editingList} setEditing={setEditingList} onAdd={v => addOption('equipment', v)} onEdit={(i, v) => editOption('equipment', i, v)} onRemove={i => removeOption('equipment', i)} onRequestDelete={(i, o) => setListDeleteModal({ list: 'equipment', idx: i, value: o, title: 'Equipment List' })} placeholder="Add equipment (e.g. Tripod)" />
```

**Add to each of lines 442-444 (pattern shown for equipment — RESEARCH §Code Examples):**

```html
onMoveUp={(i) => moveOption('equipment', i, i - 1)} onMoveDown={(i) => moveOption('equipment', i, i + 1)}
```

| Call site | listKey | onMoveUp / onMoveDown closure |
|---|---|---|
| line 442 | `'equipment'` | `moveOption('equipment', i, i - 1)` / `moveOption('equipment', i, i + 1)` |
| line 443 | `'plat'` | `moveOption('plat', i, i - 1)` / `moveOption('plat', i, i + 1)` |
| line 444 | `'approver'` | `moveOption('approver', i, i - 1)` / `moveOption('approver', i, i + 1)` |

---

### 5. Legend entry pattern (lines 434-440)

**Analog:** The four existing legend entries (435-438). The new "Reorder" entry copies the exact structure: `flex flex-col items-center gap-1` wrapper with a `<span>` label and a `w-3 h-3 rounded-full` glass dot. **Placement (RESEARCH Q3):** insert BEFORE the Edit entry (line 435) so the legend reads `[Reorder][Edit][Delete][Save][Cancel]` matching row order `[Up][Down][Edit][Delete]`; either placement is cosmetic-only (E4 wrap verified).

**Legend block (lines 434-440, verbatim):**

```html
<div className="flex flex-wrap gap-4 mt-3 text-[10px] text-white/50">
    <div className="flex flex-col items-center gap-1"><span>Edit</span><div className="w-3 h-3 rounded-full bg-blue-500/20 border border-blue-400/40"></div></div>
    <div className="flex flex-col items-center gap-1"><span>Delete</span><div className="w-3 h-3 rounded-full bg-red-500/20 border border-red-400/40"></div></div>
    <div className="flex flex-col items-center gap-1"><span>Save</span><div className="w-3 h-3 rounded-full bg-emerald-500/20 border border-emerald-400/40"></div></div>
    <div className="flex flex-col items-center gap-1"><span>Cancel</span><div className="w-3 h-3 rounded-full bg-orange-500/20 border border-orange-400/40"></div></div>
    <div className="flex flex-col items-center gap-1"><span className="font-bold">ADD</span><span className="text-[9px] text-white/40">Add new</span></div>
</div>
```

**New entry (neutral glass dot — RESEARCH §Code Examples):**

```html
<div className="flex flex-col items-center gap-1"><span>Reorder</span><div className="w-3 h-3 rounded-full bg-white/20 border border-white/40"></div></div>
```

---

### 6. Classic-syntax constraints (global, all change sites)

The codebase targets older Android stock browsers on staff phones (AGENTS.md compatibility constraint). Every new line must match the existing idiom:

| Constraint | Codebase idiom (source line) | Forbidden |
|---|---|---|
| No `.at()` | `next[list][next[list].length - 1]` (insert-before-`'Other'` amendment, RESEARCH line 241) | `arr.at(-1)` |
| No ES-module syntax | All files are classic-scope `text/babel` scripts sharing globals; `const { useState } = React` (line 98) | `import` / `export` |
| ES2015 spread OK | `[...next[list]]` (lines 244, 251, 258); object spread `{ ...p }` (line 246) | — |
| ES2015 destructuring OK | `[arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]]` (moveOption) | — |
| No template-literal class swapping? | Conditional classes use backtick template literals (lines 117-118) — this IS the idiom | — |

The RESEARCH amendment for `addOption` itself uses the classic idiom explicitly:

```javascript
// RESEARCH Pitfall 3 amendment for line 244 (replace `next[list] = [...next[list], v];`)
const last = next[list][next[list].length - 1];
next[list] = last === 'Other' ? [...next[list].slice(0, -1), v, 'Other'] : [...next[list], v];
```

(`[ASSUMED]` over the approved UI-SPEC — RESEARCH Q2, needs user sign-off; boundary rules alone can never put an item below `'Other'`, so without this the pin breaks on the next add — the live `'Laptop'` record is this bug already in production.)

---

## Shared Patterns

### Authentication / Authorization Gate (inherited, no new code)
**Source:** index.html:414 — `{isManagerAuthenticated && <button onClick={() => setView('settings')} ...>Settings</button>}`; authentication state from the passcode modal unlock (`passcodeAttempt === appPasscode`, line 558) and `isManagerAuthenticated` state (line 136).
**Apply to:** All reorder UI. The Up/Down buttons live inside the Settings view, which is only reachable when `isManagerAuthenticated` is true. **No new gating is needed** — do not add per-button auth checks. ASVS V2/V4 unchanged (RESEARCH §Security Domain).

### Error Handling (inherited, no new code)
**Source:** index.html:235-238 `saveOptions` — try/catch + `console.error` + `handleNotify("Error saving lists", "error")`.
**Apply to:** `moveOption` calls `saveOptions` and inherits the whole path. UI-SPEC §Copywriting: "no new error states". Do NOT wrap `moveOption` in its own try/catch or add new toasts.

### Notification pattern
**Source:** index.html:210 — `const handleNotify = (msg, type = 'success') => { setShowNotification({ msg, type }); setTimeout(() => setShowNotification(null), 4000); };`
**Apply to:** Only existing error strings surface from reorder (via saveOptions). No success toast for moves (matches add/edit/remove which also show none).

### Validation
**Source:** index.html:241 (`trim` + empty guard), 243 (`includes` duplicate guard), 257 (`'Other'` immutability guard).
**Apply to:** `moveOption`'s validation is the single bounds guard `if (toIdx < 0 || toIdx >= arr.length) return;` (RESEARCH amendment). The disabled-state conditions (`i === 0`, `i >= items.length - 1`, `items[i+1] === 'Other'`, `o === 'Other'`) are the UI-level validation — both layers together guarantee no out-of-bounds write (RESEARCH Pitfall 2).

### Mobile-first touch targets
**Source:** index.html:117-118 `p-2` icon buttons; UI-SPEC §Spacing exceptions ("locked, do not change sizing").
**Apply to:** Up/Down buttons use identical `p-2 rounded-lg transition-all border` sizing — roughly 30×30 px, touch-safe on LAN phones. Row `gap-2` (line 107) unchanged.

---

## No Analog Found

None — every change site has an exact in-file analog. The closest thing to a "no existing pattern" case is the `lucide.createIcons()` refresh at line 207/414 (the codebase has only one call site today, line 206) — but that single call site IS the pattern to replicate, so the analog is exact.

---

## Metadata

**Analog search scope:** `index.html` (the only source file; full 694-line read verified against RESEARCH anchors 63-65, 89, 92-95, 100-130, 157-159, 206-208, 235-260, 346/350, 409-414, 429-454, 470, 524, 561, 667 — all match)
**Files scanned:** 1 (index.html) + RESEARCH.md + UI-SPEC.md (both read in full)
**Pattern extraction date:** 2026-09-16
**Important caveat for the planner:** the researched amendments (Down-boundary clause, `moveOption` bounds guard, `addOption` insert-before-`'Other'`, legend placement, two `createIcons` call sites) deviate from the literal APPROVED UI-SPEC text — they are mandated by RESEARCH Pitfalls 1-3 and live-data observation (2026-09-15). The plan must record them as UI-SPEC amendment notes (RESEARCH Q1-Q4) rather than silently implementing or silently dropping them.