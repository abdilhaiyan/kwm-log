---
phase: 04-sortable-list-reordering
plan: 02
subsystem: ui
tags: [react, firestore, list-reorder, lucide, single-file-app, gap-closure]

requires:
  - phase: 04
    provides: Settings ListEditor reorder controls (ChevronUp/ChevronDown at size 14, neutral glass, boundary-guarded moveOption, Other-pinned-last)
provides:
  - Directional ArrowUp/ArrowDown (size 16) glyphs in the Settings reorder column — up/down readable at a glance without tooltips (G-04-2)
  - Byte-identical preservation of reorder semantics: disabled conditions, tooltip ternaries, neutral glass classes, createIcons sites, legend entry
affects: [end-of-phase UAT /gsd-verify-work 04 (tests 2/3/4 for G-04-2), any future phase touching Settings list cards]

actuals:
  tokens: 509    # chars/4 over the realized diff (2038 chars: 1017 added + 1021 deleted, both full reorder-button lines in index.html)
  tasks: 1
  commits: 1

tech-stack:
  added: []
  patterns:
    - "Directional arrow glyphs (ArrowUp/ArrowDown) as the sole affordance for reorder direction — no color accent added (single-neutral-dot legend preserved)"

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "G-04-2 closed as an icon-detail redesign in place: swap ChevronUp/ChevronDown size 14 -> ArrowUp/ArrowDown size 16 on index.html lines 117-118 only, per UAT user's 'little up arrow and down arrow is okay to make it much friendly and easier'"

patterns-established:
  - "Reorder direction affordance = glyph shape/pointing direction only; hover tooltips remain desktop-only aid and colors stay neutral (Edit=blue, Delete=red, Cancel=orange, Save=emerald occupy the palette)"

requirements-completed: [SORT-01, SORT-02, SORT-03, SORT-04, SORT-05, SORT-06]

coverage:
  - id: D1
    description: "Settings reorder buttons render ArrowUp (size 16) above ArrowDown (size 16) in every ListEditor row; zero ChevronUp/ChevronDown tokens remain in index.html"
    requirement: SORT-01
    verification:
      - kind: other
        ref: "node -e Sweep A (au=1, ad=1, cu=0, cd=0, s16=2, up=1, down=1, enCls=2) — exit 0"
        status: pass
    human_judgment: false
  - id: D2
    description: "Byte-identical no-change zones: up/down disabled conditions, all five tooltip strings, onMoveUp/onMoveDown call counts (5 each), three lucide.createIcons() sites, single neutral Reorder legend entry, enabled-class tuple on both reorder buttons and no other element"
    requirement: SORT-04
    verification:
      - kind: other
        ref: "node -e Sweep B (upDis=1, downDis=1, top=1, bottom=1, otherT=2, upd=5, dnd=5, ci=3, legend=1) — exit 0"
        status: pass
    human_judgment: false
  - id: D3
    description: "Arrow glyphs materialize in the live Settings DOM (svg.lucide-arrow-up / -down >= 6 each, bare i[data-lucide] == 0) and read as up/down at a glance on a phone-width viewport"
    requirement: SORT-01
    verification: []
    human_judgment: true
    rationale: "Requires the served app (python -m http.server 8080), an anonymous sign-in, the manager passcode, and live Firestore list data — browser/DevTools interaction belongs to the end-of-phase human verify (/gsd-verify-work 04, resuming UAT tests 2/3/4) per the plan's human_verify_mode: end-of-phase and verification section."

duration: 5min
completed: 2026-09-17
status: complete
---

# Phase 04 Plan 02: Arrow Affordance — Directional Up/Down Glyphs in Settings Reorder Controls

**G-04-2 gap closed in place: the Settings reorder column now renders a directional ArrowUp (size 16) above a directional ArrowDown (size 16) in every ListEditor row — readable as up/down at a glance on touch devices — with the move semantics, boundary conditions, tooltips, neutral glass classes, all three lucide.createIcons() sites, and the legend entry byte-identical to the shipped 04-01 state.**

## Performance

- **Duration:** 5 min
- **Started:** 2026-09-17T07:28Z
- **Completed:** 2026-09-17T07:33Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments
- Two-token edit on index.html lines 117-118: `<Icon name="ChevronUp" size={14} />` → `<Icon name="ArrowUp" size={16} />` (Up button) and `<Icon name="ChevronDown" size={14} />` → `<Icon name="ArrowDown" size={16} />` (Down button). Nothing else on either line changed — onClick, disabled conditions, title ternaries, and className are byte-identical.
- Sweep A (the fix) passes: `ArrowUp` ×1, `ArrowDown` ×1, `ChevronUp` ×0, `ChevronDown` ×0, `size={16}` ×2, `'Move up'` ×1, `'Move down'` ×1, enabled-class tuple ×2.
- Sweep B (no-change zones) passes: up-disabled ×1, down-disabled ×1, `'Already at top'` ×1, `'Already at bottom'` ×1, `'Cannot reorder Other'` ×2, `onMoveUp` ×5, `onMoveDown` ×5, `setTimeout(() => lucide.createIcons(), 100)` ×3, `span>Reorder</span>` ×1.
- Zero CDN pin changes (lucide@1.44.0, React/ReactDOM 18.3.1, Babel 8.0.5, Tailwind Play CDN, Firebase compat 11.6.1); no new or moved createIcons call sites; no helper, call-site, or legend edits.

## Task Commits

1. **Task 1: Swap chevron reorder glyphs for directional ArrowUp/ArrowDown (size 16) on index.html lines 117-118** — `e67a7cc` (fix)

**Plan metadata:** `docs(04-02): complete arrow affordance gap-closure plan` (pending this run's docs commit)

## Files Created/Modified
- `index.html` — Up/Down reorder button icon props on lines 117-118 changed from ChevronUp/ChevronDown size 14 to ArrowUp/ArrowDown size 16; entire lines otherwise byte-identical. Only file modified.

## Decisions Made
- Closed G-04-2 in place as an icon-detail redesign (no new feature): directional arrow glyphs at size 16 keep the neutral white-glass pill styling — no color accent (color-blind-safe, avoids collisions with Edit=blue/Delete=red/Cancel=orange/Save=emerald, and keeps the legend's single neutral Reorder dot valid per UAT test 1).
- No framework/tooling/data changes — single-file, no-build, CDN-loaded React app constraint untouched.

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None. First edit attempt failed only because of a typo in my own `oldString` (dropped the leading `<`); the corrected edit applied cleanly and the diff is exactly two lines.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness
- G-04-2's automated verification (Sweeps A + B) is green; the live-DOM materialization + visual/interaction checks are queued for the end-of-phase human verify (`/gsd-verify-work 04`, resuming UAT tests 2/3/4). Do NOT mark the gap closed in 04-UAT.md — the orchestrator reconciles gap status.
- Phase 4 is otherwise complete; Phase 5 (Manager Glassmorphic Styling, UI-01..UI-04) is ready to plan.

## Self-Check: PASSED

Verified before state updates:
- `04-02-SUMMARY.md` exists at `.planning/phases/04-sortable-list-reordering/`
- Commit `e67a7cc` (`fix(04-02)`) exists in git history and contains exactly the two intended line swaps in `index.html`
- Workspace clean after the task commit (no stray untracked files)

---
*Phase: 04-sortable-list-reordering*
*Completed: 2026-09-17*