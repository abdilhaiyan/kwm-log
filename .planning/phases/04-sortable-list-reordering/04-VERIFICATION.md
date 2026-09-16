---
phase: 04-sortable-list-reordering
verified: 2026-09-16T17:30:00Z
status: human_needed
score: 20/35 must-haves verified
behavior_unverified: 15
overrides_applied: 0
re_verification: false
behavior_unverified_items:
  - truth: "SORT-01: Up or Down on an equipment row swaps only adjacent indices i and i-1 or i+1; no other array element changes"
    test: "Open Settings, click Up on a non-first equipment row"
    expected: "Adjacent swap occurs; no array corruption; request form equipment select shows new order immediately"
    why_human: "Requires served app + live Firestore + manager passcode to click the button and observe the swap result"
  - truth: "SORT-01: Equipment order is defined only by the persisted equipmentOptions array in the Firestore options/lists doc"
    test: "Click an equipment reorder arrow, then reload"
    expected: "Order persists via Firestore; reloaded page shows the same order"
    why_human: "Requires live Firestore write + reload to confirm persistence path"
  - truth: "SORT-02: Plat rows use the same adjacent-swap mechanics; the persisted platOptions array order is the source of truth for the plat select"
    test: "Open Settings, move a plat row, check request form plat select (when Car is selected)"
    expected: "Plat select shows swapped order immediately"
    why_human: "Requires served app + live Firestore + selecting Car in form to expose plat select"
  - truth: "SORT-03: Approver rows use the same adjacent-swap mechanics; the persisted approverOptions array order is the source of truth for both approver selects"
    test: "Open Settings, move an approver row, check manager card and detail modal approver selects"
    expected: "Both approver selects show the swapped order"
    why_human: "Requires served app + live Firestore + manager passcode + pending request to see approver selects"
  - truth: "SORT-04: Other is always the final element after any move sequence because the boundary rules can never swap anything below it and addOption inserts before it"
    test: "For each list: confirm Other is last; move items above it; reload; verify Other remains last"
    expected: "Other stays pinned last after every move and after reload"
    why_human: "Requires live app interaction across multiple moves + reload to verify invariant"
  - truth: "SORT-05: All five consuming surfaces read the identical shared state arrays; a persisted swap re-fills them through the options onSnapshot echo"
    test: "Move an equipment row in Settings; check request form select, manager filter select without reloading"
    expected: "All three equipment-consuming selects show the new order immediately via onSnapshot echo"
    why_human: "Requires served app with live Firestore to observe the onSnapshot echo propagation"
  - truth: "SORT-05: saveOptions wholesale-replaces the three arrays in one set(merge:true) call, so element order is preserved end-to-end from state to Firestore and back"
    test: "Perform a move; watch Network tab for one options/lists write; reload and verify order unchanged"
    expected: "Exactly one Firestore write per move; persisted order matches what was set"
    why_human: "Requires live Firestore to observe write behavior and Network tab inspection"
  - truth: "SORT-05: Rapid same-direction clicks produce one step per completed write and a reload converges to the last completed write"
    test: "Click an arrow rapidly 3 times; reload; verify final position"
    expected: "Reload shows last successful write position (Pitfall 4 accepted behavior)"
    why_human: "Requires live app + live Firestore + LAN echo timing to observe the race condition"
  - truth: "SORT-06: On reload the onSnapshot fetches the doc arrays, so every device and tab renders the identical saved order"
    test: "Perform moves, reload page; open second tab/browser; compare all list orders"
    expected: "Both tabs show identical list orders after reload"
    why_human: "Requires live Firestore + second browser/device for cross-tab verification"
  - truth: "SORT-06: Order lives only in the Firestore doc arrays; the kwm_ localStorage keys are dark-mode and seen-requests only"
    test: "After performing a move, open DevTools console; run Object.keys(localStorage).filter(k => k.startsWith('kwm_'))"
    expected: "Only kwm_dark_mode and kwm_seen_requests present; no option-order keys"
    why_human: "Requires running app + DevTools console access"
  - truth: "SORT-06: Order equals the persisted doc array order for every user; there is no per-user ordering"
    test: "Verify on two different devices/tabs that list order matches after reload"
    expected: "Both devices show identical order (no per-user state)"
    why_human: "Requires two devices/browsers on same LAN with live Firestore"
  - truth: "UI E5 populated: Select options render in the persisted order and a swap immediately re-renders every select through the onSnapshot echo"
    test: "Move an item in Settings; check all consuming selects without reload"
    expected: "Every select shows the new order immediately"
    why_human: "Requires served app + live Firestore to observe select re-rendering"
  - truth: "SORT-05: When the doc is missing or an array is absent, the onSnapshot fallback restores the DEFAULT_* arrays"
    test: "If possible, temporarily remove an array from the Firestore doc; observe fallback"
    expected: "List falls back to DEFAULT_* and retains at least Other"
    why_human: "Requires live Firestore manipulation to test the fallback path"
  - truth: "UI E1 error: Save failure reuses handleNotify('Error saving lists', 'error') inside saveOptions"
    test: "Simulate a Firestore write failure (e.g., go offline); click reorder"
    expected: "Error notification appears; last-good state retained"
    why_human: "Requires live app to simulate network failure and observe error handling"
  - truth: "SORT-04: A list containing only Other renders a single row with both arrows disabled"
    test: "Remove all items except Other from a list; observe the single row"
    expected: "One row with all four buttons disabled"
    why_human: "Requires live app + manager passcode to delete items and observe the minimum list state"
---

# Phase 4: Sortable List Reordering — Verification Report

**Phase Goal:** Managers can reorder the equipment, vehicle plate, and approver lists from Settings, and the new order persists to Firestore and appears consistently on every surface that renders those lists for every user.
**Verified:** 2026-09-16T17:30:00Z
**Status:** human_needed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | SORT-01: Up/Down swaps only adjacent indices; no duplicates via includes(v) dedup; moveOption has no merge/collide path | ✓ VERIFIED | moveOption body: spread copy + destructuring swap + saveOptions (lines 264-271); addOption dedup at line 245 |
| 2 | SORT-01: Minimum equipment list is [Other]; single row renders both arrows disabled (i===0 && i>=items.length-1) | ✓ VERIFIED | DEFAULT_EQUIPMENT_OPTIONS=['DJI Osmo Action 6','Car','Other'] (line 65); Up disabled `i===0||o==='Other'` (line 117); Down disabled `o==='Other'||i>=items.length-1||items[i+1]==='Other'` (line 118) |
| 3 | SORT-01: Equipment order defined only by persisted equipmentOptions array; each move is explicit index swap | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | moveOption reads from state, calls saveOptions, onSnapshot echoes back (lines 264-271, 209); persistence path verified in code, but live Firestore round-trip not exercised |
| 4 | SORT-02: Plat rows use same adjacent-swap mechanics with same dedup; only listKey closure differs (plat) | ✓ VERIFIED | ListEditor call site line 455: `onMoveUp={(i) => moveOption('plat', i, i - 1)} onMoveDown={(i) => moveOption('plat', i, i + 1)}` |
| 5 | SORT-02: Minimum plat list is [Other]; single row renders both arrows disabled | ✓ VERIFIED | DEFAULT_PLAT_OPTIONS=['WRD 5900','Other'] (line 63); same boundary rules apply |
| 6 | SORT-02: Persisted platOptions array order is source of truth for plat select and all consumers | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | State array flows to plat select via .map() (line 361); live propagation unverified at runtime |
| 7 | SORT-03: Approver rows use same adjacent-swap mechanics with same dedup; only listKey closure differs (approver) | ✓ VERIFIED | ListEditor call site line 456: `onMoveUp={(i) => moveOption('approver', i, i - 1)} onMoveDown={(i) => moveOption('approver', i, i + 1)}` |
| 8 | SORT-03: Minimum approver list is [Other]; single row renders both arrows disabled | ✓ VERIFIED | DEFAULT_APPROVER_OPTIONS=['Shafiq','Other'] (line 64); same boundary rules apply |
| 9 | SORT-03: Persisted approverOptions array order is source of truth for both approver selects (manager card + detail modal) | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | State array flows to both approver select surfaces; live propagation unverified at runtime |
| 10 | SORT-04: Row whose next element is Other gets Down disabled; Other row has both arrows disabled; addOption inserts before Other when Other is last (Q2) | ✓ VERIFIED | Down disabled condition: `o==='Other'||i>=items.length-1||items[i+1]==='Other'` (line 118); Up disabled: `i===0||o==='Other'` (line 117); addOption insert-before-Other ternary (lines 246-247) |
| 11 | SORT-04: List containing only Other renders single row with both arrows disabled | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Boundary rules would disable both for a single-Other list (i===0 && o==='Other'); requires live minimum-list state to confirm |
| 12 | SORT-04: Other is always final element after any move sequence | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Boundary rules prevent swapping below Other; addOption inserts before it; but live multi-move sequence unverified |
| 13 | SORT-05: All five consuming surfaces read identical shared state arrays; persisted swap re-fills through onSnapshot echo | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Code shows all selects reading from equipmentOptions/platOptions/approverOptions; onSnapshot sets them (line 209); but live echo propagation unverified |
| 14 | SORT-05: Fallback restores DEFAULT_* arrays when doc missing or array absent; minimum list retains Other | ✓ VERIFIED | onSnapshot: `d.platOptions \|\| DEFAULT_PLAT_OPTIONS` etc. (line 209); every DEFAULT_* includes 'Other' (lines 63-65) |
| 15 | SORT-05: saveOptions wholesale-replaces arrays in one set(merge:true); element order preserved end-to-end | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | saveOptions does `.set({platOptions, approverOptions, equipmentOptions}, {merge:true})` (line 239); code verified, live Firestore round-trip unverified |
| 16 | SORT-05: Rapid same-direction clicks produce one step per write; reload converges to last write (Pitfall 4 accepted) | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Acceptable behavior documented; requires live LAN echo timing to observe |
| 17 | SORT-06: On reload, onSnapshot fetches doc arrays; every device/tab renders identical saved order | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | onSnapshot sets state arrays on load (line 209); reload behavior requires live Firestore |
| 18 | SORT-06: Order in Firestore doc only; kwm_ localStorage keys are dark-mode and seen-requests only | ✓ VERIFIED | localStorage reads: `kwm_dark_mode` (line 143) and `kwm_seen_requests` (line 150); no option arrays in localStorage |
| 19 | SORT-06: Order equals persisted doc array order for every user; no per-user ordering | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | State arrays are component-level singletons shared by all renders; cross-device verification requires live Firestore |
| 20 | UI E1 empty: Lists can never be empty; every DEFAULT_* seeds Other; saveOptions persists non-empty lists | ✓ VERIFIED | DEFAULT_* all contain 'Other' (lines 63-65); addOption and moveOption never produce empty arrays (addOption requires non-empty value; moveOption swaps within existing array) |
| 21 | UI E1 populated: Settings rows render as label → Move Up → Move Down → Edit → Delete (row DOM order per UI-SPEC) | ✓ VERIFIED | ListEditor non-editing fragment: span(label), button(ChevronUp), button(ChevronDown), button(Pencil/Edit), button(Trash2/Delete) at lines 116-120 |
| 22 | UI E1 overflow: Existing truncate class on flex-1 label span clips long names; fixed p-2 icon buttons independent of label width | ✓ VERIFIED | `truncate` on label span (line 116); `p-2` fixed sizing on all buttons (lines 117-120) |
| 23 | UI E1 zero-one-many: One-item list renders one row with all four buttons disabled; multi-item rows enable per amended boundary rules | ✓ VERIFIED | Single-item: i===0 disables Up, o==='Other' disables both, i>=items.length-1 disables Down → all four disabled (line 117-118 conditions) |
| 24 | UI E1 long-text: Long labels truncate via existing truncate class; icon-only reorder buttons width-independent | ✓ VERIFIED | Same as E1 overflow — `truncate` class present on label span |
| 25 | UI E2 empty: Arrows render only inside existing list row; no row-free button path | ✓ VERIFIED | Arrow buttons are inside the items.map() loop (lines 117-118), within the same fragment as Edit/Delete |
| 26 | UI E2 populated: Enabled arrows use bg-white/20 text-white/80 hover:bg-white/30 border border-white/30 with ChevronUp/ChevronDown at size 14; boundary items get exactly one disabled arrow | ✓ VERIFIED | Enabled classes: `bg-white/20 text-white/80 hover:bg-white/30 border border-white/30` (lines 117-118); disabled: `bg-white/5 text-white/20 border-white/10 cursor-not-allowed`; Icon size={14} |
| 27 | UI E2 zero-one-many: Single-item renders both arrows disabled; multi-item enables per i===0 and last/next boundary clauses | ✓ VERIFIED | Same boundary logic as E1 zero-one-many; conditions on lines 117-118 |
| 28 | UI E3 empty: Other is always last because seeded by default, never removable, never movable, addOption inserts before it (Q2) | ✓ VERIFIED | DEFAULT_* all end with 'Other' (lines 63-65); removeOption blocks Other (line 260); moveOption boundary rules prevent moving Other below; addOption inserts before Other (lines 246-247) |
| 29 | UI E3 populated: All four buttons on Other row permanently disabled with correct classes and tooltip Cannot reorder Other | ✓ VERIFIED | Up: `o==='Other'` disables (line 117); Down: `o==='Other'` disables (line 118); Edit: `o==='Other'` disables (line 119); Delete: `o==='Other'` disables (line 120); Tooltip: `Cannot reorder Other` for both arrows |
| 30 | UI E3 zero-one-many: Minimum list never zero; Other-only row shows all four buttons disabled | ✓ VERIFIED | Same as E1 zero-one-many and E3 populated combined |
| 31 | UI E4 overflow: Legend container is flex flex-wrap gap-4; fifth Reorder entry wraps | ✓ VERIFIED | Legend at line 445: `flex flex-wrap gap-4`; Reorder entry is the first of six legend items (lines 446-451) |
| 32 | UI E5 empty: Request form select, manager filter select, detail-modal select can never be empty because every list contains at least Other | ✓ VERIFIED | All three selects map over their respective options arrays; all arrays include Other per DEFAULT_* |
| 33 | UI E5 populated: Select options render in persisted order; swap immediately re-renders every select through onSnapshot echo | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Core deliverable; code path exists but live echo propagation requires served app + Firestore |
| 34 | UI E5 zero-one-many: One-option list renders just Other; many-option lists render in saved order with native scrolling | ✓ VERIFIED | Selects use .map() over options arrays; native scrolling is browser default behavior |
| 35 | Reorder legend entry (span Reorder with neutral glass dot) exists before Edit legend entry | ✓ VERIFIED | `<span>Reorder</span>` at line 446; `<span>Edit</span>` at line 447; position verified: Reorder strictly before Edit |

**Score:** 20/35 must-haves verified (15 present, behavior-unverified)

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `index.html: moveOption function` | New App-scope function with (list, fromIdx, toIdx) signature, spread copy, bounds guard, destructuring swap, saveOptions call | ✓ VERIFIED | Lines 264-271; exact plan form confirmed by node -e sweep `{sig:1,guard:1,ib:1,last:1}` + `{bad:0,fn:true}` |
| `index.html: ListEditor signature` | onMoveUp, onMoveDown props added after onRemove | ✓ VERIFIED | Line 100: `onMoveUp, onMoveDown` present in destructure |
| `index.html: Up/Down buttons` | Two buttons between label span and Edit button in non-editing fragment; ChevronUp/ChevronDown at size 14 | ✓ VERIFIED | Lines 117-118; node -e confirms `name="ChevronUp"` x1, `name="ChevronDown"` x1 |
| `index.html: Three call-site wirings` | Each ListEditor call gets onMoveUp/onMoveDown arrow closures closing over listKey | ✓ VERIFIED | Lines 454-456; node -e confirms onMoveUp x5, onMoveDown x5 |
| `index.html: Reorder legend entry` | `<span>Reorder</span>` with neutral glass dot before Edit entry | ✓ VERIFIED | Line 446; node -e confirms leg:1, pos:true |
| `index.html: Two createIcons call sites` | setTimeout in options onSnapshot (line 209) and Settings nav onClick (line 425) | ✓ VERIFIED | node -e confirms ci:3 (existing + 2 new); lines 209 and 425 |
| `index.html: addOption insert-before-Other` | `const last = next[list][next[list].length - 1]` + ternary | ✓ VERIFIED | Lines 246-247; node -e confirms ib:1, last:1 |

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| Arrow button click → moveOption → Firestore | onMoveUp/onMoveDown prop → moveOption(list, i, i±1) → saveOptions → Firestore doc | Arrow click → prop closure → function → saveOptions set(merge:true) | ✓ VERIFIED | Up button: `onClick={() => onMoveUp(i)}` (line 117); Down: `onClick={() => onMoveDown(i)}` (line 118); call sites pass `(i) => moveOption(key, i, i-1)` / `(i) => moveOption(key, i, i+1)` (lines 454-456); moveOption calls `saveOptions(next.plat, next.approver, next.equipment)` (line 270) |
| options onSnapshot → all five select surfaces | onSnapshot sets state → selects re-render from shared arrays | State arrays feed .map() in each select | ✓ VERIFIED | onSnapshot sets all three arrays (line 209); equipment select (line 357); plat select (line 361); filter select (line 482); manager card approver (line 534); detail modal approver (line 681) |
| createIcons refresh → materialized SVGs | createIcons called after Settings mount and after options echo | setTimeout in Settings nav + options onSnapshot | ✓ VERIFIED | Settings nav: `setTimeout(() => lucide.createIcons(), 100)` after `setView('settings')` (line 425); options onSnapshot: same call (line 209) |
| Other-last invariant → boundary rules + Q2 add | Down disabled clauses + addOption insert-before-Other | Three-condition Down disable + addOption ternary | ✓ VERIFIED | Down: `o==='Other'||i>=items.length-1||items[i+1]==='Other'` (line 118); addOption: `last==='Other' ? [..slice..,v,'Other'] : [...,v]` (lines 246-247) |

### Data-Flow Trace (Level 4)

| Artifact | Data Variable | Source | Produces Real Data | Status |
|----------|---------------|--------|-------------------|--------|
| Equipment ListEditor items | equipmentOptions | onSnapshot reads Firestore options/lists doc → setEquipmentOptions (line 209) | Yes — Firestore doc is the runtime data source, not DEFAULT_* | ✓ FLOWING |
| Plat ListEditor items | platOptions | onSnapshot reads Firestore options/lists doc → setPlatOptions (line 209) | Yes — Firestore doc is the runtime data source | ✓ FLOWING |
| Approver ListEditor items | approverOptions | onSnapshot reads Firestore options/lists doc → setApproverOptions (line 209) | Yes — Firestore doc is the runtime data source | ✓ FLOWING |
| Request form equipment select | equipmentOptions.map() | Shared state array fed by onSnapshot | Yes — same array as ListEditor | ✓ FLOWING |
| Manager filter equipment select | equipmentOptions.map() | Shared state array fed by onSnapshot | Yes — same array as ListEditor | ✓ FLOWING |
| Approver selects (card + modal) | approverOptions.map() | Shared state array fed by onSnapshot | Yes — same array as ListEditor | ✓ FLOWING |

### Behavioral Spot-Checks

Step 7b: SKIPPED — no runnable entry points without the served app on localhost:8080 with live Firestore. All automated verification was done via node -e regex sweeps against index.html (which passed all 4 plan commands). Behavioral verification requires the live served app.

### Probe Execution

No probes exist for this phase (no-build repo, no test framework, no probe scripts).

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| SORT-01 | 04-01-PLAN.md | Manager can reorder equipment list items (up/down) in Settings Equipment List card | ✓ SATISFIED (code); ⚠️ runtime unverified | moveOption + Up/Down buttons + call-site wiring all present and wired; live click-through needed |
| SORT-02 | 04-01-PLAN.md | Manager can reorder vehicle plates list items (up/down) in Settings Vehicle Plates List card | ✓ SATISFIED (code); ⚠️ runtime unverified | Same mechanism, listKey='plat' (line 455); live click-through needed |
| SORT-03 | 04-01-PLAN.md | Manager can reorder approvers list items (up/down) in Settings Approvers List card | ✓ SATISFIED (code); ⚠️ runtime unverified | Same mechanism, listKey='approver' (line 456); live click-through needed |
| SORT-04 | 04-01-PLAN.md | "Other" stays pinned as the last item in every list and cannot be moved | ✓ SATISFIED (code); ⚠️ runtime unverified | Boundary rules + addOption insert-before-Other; live multi-move verification needed |
| SORT-05 | 04-01-PLAN.md | Reordered lists persist to Firestore and sync to all surfaces on all devices | ✓ SATISFIED (code path); ⚠️ runtime unverified | saveOptions + onSnapshot + 5 select surfaces wired; live persistence + cross-tab sync needed |
| SORT-06 | 04-01-PLAN.md | Reordered lists survive a page reload and stay in the same order for every user | ✓ SATISFIED (code path); ⚠️ runtime unverified | onSnapshot reload + no localStorage order state; live reload + cross-device verification needed |

No orphaned requirements — all 6 SORT IDs from the plan are accounted for in REQUIREMENTS.md and trace to Phase 4.

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| index.html | 14 | `lucide@latest` unpinned CDN URL (WR-02 from review) | ⚠️ Warning | Pre-existing; deepened by this phase adding new icon names. No code change in this phase caused it, but future Lucide drift could blank new icons. |
| index.html | 267 | `fromIdx` not validated in moveOption guard (WR-01 from review) | ℹ️ Info | Defense-in-depth gap; no UI-reachable path passes bad fromIdx. Not a FAILED truth. |
| index.html | 118 | Disabled Down tooltip says "Move down" when `items[i+1]==='Other'` (IN-01 from review) | ℹ️ Info | Cosmetic: button is disabled (correct behavior), but tooltip text is misleading. Does not affect functionality. |
| index.html | 246-247 | addOption insert-before-Other bypassed while legacy Laptop-after-Other data unrepaired (IN-02 from review) | ℹ️ Info | Documented transitional behavior (Q4); fixed by one-time manual up-move during UAT. |

No debt markers (TBD/FIXME/XXX) found in phase-changed code. No stub patterns in the phase's code additions.

### Human Verification Required

### 1. Mobile Visual/UX Confirmation

**Test:** Open Settings on a phone viewport (or responsive preview in DevTools). Check the four-button row layout.
**Expected:** The four-button row fits mobile width; the truncating label clips long names; disabled arrows read as faded (bg-white/5) vs enabled glass arrows; hover/tap targets feel consistent (p-2, ~30x30px); the Settings legend reads Reorder, Edit, Delete, Save, Cancel, ADD with the neutral dot for Reorder.
**Why human:** Visual/UX judgment on mobile layout, spacing, and visual hierarchy cannot be verified programmatically.

### 2. S1 — Equipment Reorder Live (SORT-01)

**Test:** Settings → Equipment List → click Up on a non-first row → observe the request form's equipment select.
**Expected:** The equipment select shows the swapped order immediately (no reload).
**Why human:** Requires served app (localhost:8080), live Firestore, and manager passcode to interact with Settings and observe the request form select.

### 3. S2 — Plat Reorder Live (SORT-02)

**Test:** Vehicle Plates card → move a row → check request form plat select (form with cameraModel = Car).
**Expected:** The plat select reflects the same order as the Settings card.
**Why human:** Requires served app, live Firestore, and selecting Car in the form to expose the plat select.

### 4. S3 — Approver Reorder Live (SORT-03)

**Test:** Approvers card → move a row → check manager card approver select AND detail-modal approver select.
**Expected:** Both approver selects reflect the new order.
**Why human:** Requires served app, live Firestore, and a pending request to view both approver selects.

### 5. S4 — Other Pin + Q4 Legacy Fix + Q2 Add Observation (SORT-04)

**Test:** For EVERY list: verify the Other row has both arrows disabled with tooltip "Cannot reorder Other". Then in Equipment: click Laptop Up once → order becomes ['DJI Osmo Action 6','Car','Laptop','Other']. Then ADD a new equipment item → confirm it appears ABOVE Other.
**Expected:** Other is immovable everywhere; after the Laptop up-move, Down on Laptop shows "Already at bottom" (Q1 amendment observable); new item lands above Other (Q2 observable); every select re-renders with Other last.
**Why human:** Requires live app interaction with live Firestore data; visual tooltip and position verification.

### 6. S5 — Persistence + Cross-Tab (SORT-05)

**Test:** After the moves in S1-S4, reload the page → order persists. Open a second tab/browser → the same saved order. Watch Network tab → exactly one options/lists write per move.
**Expected:** Order persists through Firestore round-trip; both tabs show identical order; Network tab shows one write per move.
**Why human:** Requires live Firestore, cross-tab verification, and Network tab inspection.

### 7. S6 — Reload Consistency + No localStorage Order (SORT-06)

**Test:** After S5 reload, confirm order is identical on both surfaces. In DevTools console: `Object.keys(localStorage).filter(k => k.startsWith('kwm_'))` — should only show kwm_dark_mode and kwm_seen_requests.
**Expected:** Order matches across tabs/devices after reload; localStorage contains no option-order keys.
**Why human:** Requires running app + DevTools console access for localStorage inspection.

### 8. Regression UAT (STACK.md)

**Test:** Submit a request form, manager passcode unlock, approve with approver select, reject, filter, CSV export, copy details, dark-mode toggle, return flow (mark returned + condition), item-passed flow.
**Expected:** All flows behave exactly as before the phase — zero new error states, zero layout regressions.
**Why human:** Full behavior-preservation check across the entire app; requires live Firestore interactions.

### 9. DevTools Icon Assertion (Pitfall 1 Reproduction)

**Test:** Open Settings → DevTools console → run `[...document.querySelectorAll('i[data-lucide]')]`. Then navigate back to Form, open Settings again, re-run.
**Expected:** Both runs return `[]` (was 18 unmaterialized before this phase). All icons materialized to `svg.lucide-*`.
**Why human:** Requires served app with live DOM to inspect icon materialization state.

### 10. DevTools Disabled-Matrix Assertion

**Test:** Open Settings → DevTools console → evaluate the matrix assertion from PLAN Task 2 verify.
**Expected:** `icons === 0` (no unmaterialized), `otherUp >= 3`, `otherDown >= 3`, `otherUpDisabled === true`, `otherDownDisabled === true`, `firstUpDisabled === true`, `bottomDownDisabled === true`.
**Why human:** Requires served app with live DOM and live Firestore data (boundary states depend on actual list content).

### Gaps Summary

No code-level gaps found. All 20 code-verifiable truths are VERIFIED. All artifacts exist, are substantive, and are wired. All key links are confirmed. All 6 requirements (SORT-01 through SORT-06) have satisfied code-level evidence.

The 15 PRESENT_BEHAVIOR_UNVERIFIED truths represent runtime behaviors that cannot be verified without the served app on localhost:8080 with live Firestore and the manager passcode. These are the core deliverables of the phase (click-to-reorder, Firestore persistence, cross-surface sync, reload consistency) — they are present and correctly wired in the code, but their behavior must be confirmed through the human UAT steps S1-S6 listed above.

Code review findings (04-REVIEW.md): WR-01 (fromIdx guard gap — defensive, not UI-reachable), WR-02 (lucide@latest unpinned — pre-existing), IN-01 (disabled-Down tooltip cosmetic), IN-02 (transitional add-below-Other until Q4 repair) — all at info/warning level, none constituting a FAILED truth.

---

_Verified: 2026-09-16T17:30:00Z_
_Verifier: the agent (gsd-verifier)_
