---
phase: 04-sortable-list-reordering
plan: 01
subsystem: ui
tags: [react, firestore, list-reorder, lucide, single-file-app]

requires:
  - phase: 03
    provides: Settings list management (ListEditor add/edit/delete, saveOptions Firestore persistence, options onSnapshot echo)
provides:
  - Up/Down reorder controls on all three Settings list cards (Equipment, Vehicle Plates, Approvers)
  - moveOption Firestore-echo mutation with out-of-bounds guard and destructuring adjacent swap
  - addOption insert-before-Other (Q2 amendment) keeping Other pinned last
  - Lucide icon materialization fix for Settings surface (2 new createIcons call sites)
affects: [verify-work UAT, future phases touching Settings lists or select surfaces]

actuals:
  tokens: 1901    # chars/4 over the realized diff (7604 chars: 4924 added + 2680 deleted in index.html)
  tasks: 2
  commits: 3

tech-stack:
  added: []
  patterns:
    - "Firestore-echo mutation (no optimistic setState; onSnapshot echo refills state)"
    - "Boundary-guarded array swap (if (toIdx < 0 || toIdx >= arr.length) return; before destructuring swap)"
    - "scoped lucide.createIcons() refresh at each new materialization trigger"

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "moveOption follows the plan/research (list, fromIdx, toIdx) signature with a destructuring swap on a copied array and no setState on option arrays"

patterns-established:
  - "Reorder buttons disabled-state matrix: i===0 (Up), o==='Other' (both), i>=items.length-1 (Down), items[i+1]==='Other' (Down)"

requirements-completed: [SORT-01, SORT-02, SORT-03, SORT-04, SORT-05, SORT-06]

coverage:
  - id: D1
    description: "Up/Down reorder buttons on Equipment, Vehicle Plates, and Approvers list cards; click swaps adjacent row, saveOptions persists, all consuming selects re-render in new order"
    requirement: SORT-01
    verification: []
    human_judgment: true
    rationale: "No test framework in this no-build repo (no package.json). Live click-through requires the served app (localhost:8080), manager passcode to open Settings, and the live Firestore options/lists doc — browser/DevTools interaction is end-of-phase human UAT."
  - id: D2
    description: "Other is immovable (both arrows disabled with 'Cannot reorder Other' tooltip) and pinned last after any move sequence and after an add (Q2 insert-before-Other); legacy Equipment Laptop-after-Other pin restored by one-time up-move"
    requirement: SORT-04
    verification: []
    human_judgment: true
    rationale: "Depends on live data per RESEARCH A1 (Equipment currently ['DJI Osmo Action 6','Car','Other','Laptop']). Disabled-state matrix was DevTools-asserted only against live state; requires manager passcode + served page."
  - id: D3
    description: "Reordered order persists to the Firestore options/lists doc and renders identically after reload and on any other device/tab; localStorage holds only kwm_dark_mode / kwm_seen_requests"
    requirement: SORT-06
    verification: []
    human_judgment: true
    rationale: "Requires second-tab/second-device reload verification against live Firestore — manual UAT per plan S5/S6."
  - id: D4
    description: "Source-level acceptance criteria from PLAN.md tasks 1-2 all hold on edited index.html (moveOption signature/guard/insert-before ternary, 5+5 prop refs, single ChevronUp/ChevronDown, 3 createIcons call sites, legend order, Down-button Q1 clauses, editing branch untouched)"
    verification:
      - kind: other
        ref: "node -e regex sweep (Task 1 verify #1 sig/guard/ib/last = 1,1,1,1; #2 bad=0 fn=true)"
        status: pass
      - kind: other
        ref: "node -e regex sweep (Task 2 verify #1 up=5 down=5 cu=1 cd=1 ci=3 leg=1 pos=true; #2 lastIdx=3 nextOther=2 firstUp=1)"
        status: pass
      - kind: other
        ref: "node -e delimiter-balance sweep — CUR block5 +2/+2 identical to baseline (no new imbalance)"
        status: pass
      - kind: other
        ref: "browser smoke (localhost:8080): app renders live data, 0 unmaterialized i[data-lucide], 2 materialized svg.lucide"
        status: pass
    human_judgment: false

metrics:
  duration: 21min
  completed: 2026-09-16
  status: complete
---

# Phase 04 Plan 01: Sortable List Reordering Summary

**Up/Down adjacent-swap reorder controls on all three Settings list cards (Equipment, Vehicle Plates, Approvers) persisted via the Firestore-echo saveOptions pattern, with Other pinned last (Q2 insert-before-Other) and the Settings-surface Lucide icon materialization gap closed by two scoped createIcons call sites**

## Performance

- **Duration:** 21 min
- **Started:** 2026-09-16T15:53:18Z (approx)
- **Completed:** 2026-09-16T16:54:02Z
- **Tasks:** 2
- **Files modified:** 1 (index.html)

## Accomplishments
- `moveOption(list, fromIdx, toIdx)` implemented with `[...next[list]]` copy, out-of-bounds guard, destructuring adjacent swap, Firestore-echo persistence (no setState on option arrays)
- ChevronUp/ChevronDown buttons in each Settings list row between label and Edit; disabled-state matrix per amended boundary rules (`i === 0`, `o === 'Other'`, `i >= items.length - 1`, `items[i+1] === 'Other'`)
- addOption changed from append to insert-before-Other (Q2), keeping Other pinned last on every add
- "Reorder" legend entry with neutral glass dot added before "Edit" (Q3, row-order fidelity)
- Lucide icon materialization fixed for Settings: `lucide.createIcons()` refresh added to options onSnapshot (line 207) and Settings nav onClick (line 414)

## Task Commits

Each task was committed atomically:

1. **Task 1: moveOption + reorder list rendering** - `1d24dc3` (feat)
2. **Task 2: call-site wiring + legend + createIcons refresh** - `1e890c3` (feat)
3. **Rule 3 alignment: acceptance-criteria conformance** - `315c34b` (fix)

## Files Created/Modified
- `index.html` - ListEditor gains Up/Down buttons + onMoveUp/onMoveDown props; App gains moveOption; addOption insert-before-Other; Reorder legend; 2 createIcons call sites; 3 ListEditor call sites wired with `(i) => moveOption(listKey, i, i±1)`

## Decisions Made
- moveOption follows the plan/research signature `(list, fromIdx, toIdx)` with destructuring swap on a copied array — no setState on the option arrays anywhere in the function (onSnapshot echo fills them back)
- Finished `moveOption` with ASI (no trailing `;` after closing `}`) to satisfy the plan verify regex `/saveOptions\(...\);\s*\}$/` anchor
- Reorder emits no success toast — matches existing add/edit/remove mutation UX (only saveOptions error path surfaces)

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Initial implementation deviated from PLAN.md literal acceptance-criteria regexes; realigned in `315c34b`**
- **Found during:** Post-commit verification of Tasks 1 and 2 (plan `<verify>` automated sweeps)
- **Issue:** The first pass implemented moveOption as `(list, idx, dir)` with a temp-variable swap and used an `indexOf('Other')`-based insert expression; the first pass also used non-literal button disabled conditions/titles/classes and `moveOption(list, i, 'up'/'down')` call-site closures. These differ from the plan's exact regex-verified forms (`const moveOption = (list, fromIdx, toIdx) =>`, `const last` + ternary insert, `disabled={i === 0 || o === 'Other'}`, `(i) => moveOption(listKey, i, i - 1)`), so the plan's `<automated>` verify commands failed.
- **Fix:** Rewrote moveOption to the plan form (fromIdx/toIdx, `[...next[list]]` copy, `if (toIdx < 0 || toIdx >= arr.length) return;` guard, destructuring swap, saveOptions final statement, ASI closing); replaced the insert expression with `const last = next[list][next[list].length - 1];` + the Q2 ternary; replaced the button JSX with the exact RESEARCH lines 174-175 forms (Q1 amendment included); rewired call sites to `(i, i − 1)` / `(i, i + 1)`.
- **Files modified:** index.html
- **Verification:** ALL plan automated verify commands now pass: Task 1 sweep `{sig:1,guard:1,ib:1,last:1}` + `{bad:0,fn:true}`; Task 2 sweep `{up:5,down:5,cu:1,cd:1,ci:3,leg:1,pos:true}` + `{lastIdx:3,nextOther:2,firstUp:1}`. Delimiter-balance sweep unchanged vs baseline. Browser smoke on localhost:8080 renders live data with 0 unmaterialized icons.
- **Committed in:** `315c34b` (fix commit, separate from the two task commits)

---

**Total deviations:** 1 auto-fixed (Rule 3 - blocking: acceptance-criteria conformance)
**Impact on plan:** The fix brought the working tree to exact plan conformance; no scope creep, no behavioral difference from the researched design. All plan `must_haves` truth predicates satisfied by the final source.

## Issues Encountered
- None beyond the auto-fixed deviation above. PowerShell does not support bash variable assignment for `date -u` (environment note only, no impact on deliverables).

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- All three Settings lists now reorderable with persistence; Other pinned last; Settings icons materialize on every entry path
- Deferred to end-of-phase human UAT (`/gsd-verify-work`): DevTools disabled-matrix assertion on the served app (requires manager passcode), S1-S6 acceptance against live Firestore data, and the regression UAT from STACK.md — pending human interaction, not code
- Threat register: T-04-01 mitigated (Q1 boundary rules + in-function guard), T-04-02/03/04 accepted per plan, T-04-SC n/a (zero packages added)

## TDD Gate Compliance
Not applicable — plan `type: execute`, no `tdd="true"` tasks, and the repo has no test framework (no package.json).

## Self-Check: PASSED

- FOUND: `.planning/phases/04-sortable-list-reordering/04-01-SUMMARY.md`
- FOUND: `index.html`
- FOUND: commit `1d24dc3`
- FOUND: commit `1e890c3`
- FOUND: commit `315c34b`

---
*Phase: 04-sortable-list-reordering*
*Completed: 2026-09-16*