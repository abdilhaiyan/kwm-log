---
phase: 01-multi-block-decomposition
plan: 02
subsystem: "index.html decomposition"
tags: [refactor, decomposition, firebase, utilities]
provides: [Block 3: Firebase Init, Block 4: Utilities]
affects: [index.html]
tech-stack:
  added: []
  patterns: [multi-block text/babel decomposition, pure-move extraction]
key-files:
  created: []
  modified: [index.html]
key-decisions:
  - "Pure-move extraction: byte-identical copy of Firebase init and utility functions into new blocks, zero behavior change"
patterns-established:
  - "Firebase init (app/auth/db globals) placed before Utilities in script order (dependency: utilities don't use db, but App does)"
duration: "15min"
completed: 2026-09-13
---

# Phase 1: multi-block-decomposition Summary

Plan 02 extracted the Firebase initialization and the five pure utility functions from the App component into two new `<script type="text/babel">` blocks.

## Performance

- **Duration:** 15min
- **Tasks:** 2 completed
- **Files modified:** 1

## Accomplishments

- Extracted Block 3 (Firebase Init): `let app, auth, db;` + `try { initializeApp/auth()/firestore() } catch` — byte-identical move into a new block after Config.
- Extracted Block 4 (Utilities): `isOverdue`, `getDaysInfo`, `getRequestDuration`, `getEquipmentLabel`, `getEquipmentIcon` — byte-identical move out of the App component body into a new block after Firebase Init.
- Verified: exactly 5 text/babel blocks in dependency order (Constants → Config → Firebase Init → Utilities → original); init code and utility declarations absent from the original block; diff confirmed pure move (no other lines touched).

## Task Commits

1. **Task 1: Extract Block 3: Firebase Init** - `ba8174c`
2. **Task 2: Extract Block 4: Utilities** - `1411324`

## Files Created/Modified

- `index.html` - Firebase Init and Utilities extracted into their own text/babel blocks (5 blocks total now)

## Decisions & Deviations

- None - followed plan as specified. Pure moves only; all 5 useEffect hooks (with cleanups) and all 14 useState hooks remain in App.

## Next Phase Readiness

- Ready for Plan 03: extract Block 5 (Shared UI: `Icon`), Block 6 (App Shell: `App` + React destructure), and Block 7 (Render: `ReactDOM.createRoot(...).render(<App />)` + remove empty inline block).