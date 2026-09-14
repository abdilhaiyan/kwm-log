---
phase: 02-config-driven-passcode
plan: 01
subsystem: auth
tags: [passcode, firebase, react, single-file-html]

# Dependency graph
requires:
  - phase: 01-multi-block-decomposition
    provides: 7 ordered `text/babel` blocks in one index.html (global scope via script order) + locked decisions D-02 (one commit per logical change) and D-04 (preserve `data-passcode-unlock` attribute + `lucide.createIcons()` 100ms timeout)
provides:
  - Manager unlock validated directly against the `DEFAULT_PASSCODE` HTML constant (Block 2) — no React state copy, no Firestore mirror
  - Firestore passcode path fully removed: `app_config` fetch effect and rotation write deleted; `onSnapshot` listener is the only remaining Firestore read
  - Rotation control (New passcode input + "Passcode updated" toast) removed; `handleNotify` retained for other toasts
  - D-01 rotation comment above the constant (edit-to-rotate model); `app_config` doc left as harmless dead data (D-02)
affects: [02-config-driven-passcode plan 02, verify phase, any future passcode-flow work]

# Actuals (#2632) — chars/4 over the realized diff (git diff e56c294..5b9bf8b -- index.html = 4371 chars)
actuals:
  tokens: 1093
  tasks: 2
  commits: 2

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Constant-as-config validation source: passcode read from the Block 2 HTML constant; rotation = edit the constant; zero Firestore passcode operations"

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "D-01: rotation comment `// Manager passcode — edit to rotate` placed above `const DEFAULT_PASSCODE = \"1234\";` — rotation is now an HTML edit"
  - "D-02: Firestore `app_config` doc left untouched as dead data — no programmatic deletion"
  - "Validate `passcodeAttempt === DEFAULT_PASSCODE` directly; `currentPasscode`/`newPasscode` state mirrors deleted"

patterns-established:
  - "Constant-as-config validation: unlock handler compares against the module-scope Block 2 constant, shared across all 7 blocks"

requirements-completed: [PASS-01, PASS-02, PASS-03]

# Coverage metadata (#1602)
coverage:
  - id: D1
    description: "Manager unlock validates the typed passcode against the DEFAULT_PASSCODE constant and switches to the manager view on match; wrong passcode clears the input silently with no toast"
    requirement: PASS-01
    verification:
      - kind: other
        ref: "node -e constant-compare + wrong-code-reset checks (index.html contains passcodeAttempt === DEFAULT_PASSCODE, setPasscodeAttempt(\"\"), no 'Passcode updated' toast, no 'New passcode (optional)' input)"
        status: pass
    human_judgment: false
  - id: D2
    description: "Zero Firestore passcode references remain in index.html — app_config, currentPasscode, newPasscode all absent; the borrowing_requests onSnapshot listener is the only Firestore read"
    requirement: PASS-02
    verification:
      - kind: other
        ref: "node -e removed-identifiers check (grep app_config|currentPasscode|newPasscode returns zero matches)"
        status: pass
    human_judgment: false
  - id: D3
    description: "Passcode modal UI preserved byte-identical: exactly one password field, Enter-key submit via data-passcode-unlock click, Cancel button, modal chrome, and the 100ms createIcons timeout; 7 text/babel blocks intact"
    requirement: PASS-03
    verification:
      - kind: manual_procedural
        ref: "Browser UAT at http://10.0.6.12:8080 — 4-point list (1234 unlocks; wrong code clears silently; exactly one password field; Enter submits; app otherwise identical, no console errors) — user verified, responded 'verified'"
        status: pass
    human_judgment: true
    rationale: "End-of-phase browser UAT requires a real browser (Babel standalone cannot run in the preview env — Phase 1 finding). Performed at the Task 1 tracer gate and explicitly verified by the user; verify phase re-runs the full UAT per config human_verify_mode: end-of-phase."

# Metrics
duration: 12min
completed: 2026-09-14
status: complete
---

# Phase 2 Plan 1: Constant-Based Passcode Summary

**Manager unlock now validates against the `DEFAULT_PASSCODE` HTML constant with the Firestore passcode read/write path deleted and a rotate-by-editing comment added — zero Firestore passcode operations remain in the app**

## Performance

- **Duration:** 12 min (continuation session: Task 1 committed earlier + human-verified at tracer gate; Task 2 + summary in this session)
- **Started:** 2026-09-14T03:41:00Z (approx — continuation session start)
- **Completed:** 2026-09-14T03:53:29Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Manager unlock validates against the Block 2 constant: `passcodeAttempt === DEFAULT_PASSCODE` (PASS-01, PASS-03) — no React state copy of the passcode remains.
- Firestore passcode path deleted: `app_config` fetch `useEffect`, rotation `.set()` write, and `currentPasscode`/`newPasscode` state all removed (PASS-02); the borrowing_requests `onSnapshot` listener is the only Firestore read left.
- Rotation control gone: the "New passcode (optional)" input and the "Passcode updated" toast are removed; `handleNotify` kept for other toasts (wrong-passcode branch stays byte-identical — input clears, no toast).
- D-01 rotation comment `// Manager passcode — edit to rotate` added above the constant; the `app_config` Firestore doc is left untouched as dead data (D-02).
- Preserved byte-identical: passcode input + Enter-key handler, `data-passcode-unlock` attribute, Cancel button, modal chrome, the other four effects, `handleNotify`, and the 100ms `createIcons` timeout (Phase 1 locked decision D-04). Exactly 7 `text/babel` blocks intact.

## Task Commits

Each task was committed atomically:

1. **Task 1: End-to-end "constant-based passcode" — validate against DEFAULT_PASSCODE; delete the Firestore passcode path and the rotation control** - `238cc36` (feat)
2. **Task 2: Annotate DEFAULT_PASSCODE with a rotation comment (D-01)** - `5b9bf8b` (docs)

**Plan metadata:** (final docs commit — see below)

## Files Created/Modified
- `index.html` — Block 2: D-01 rotation comment above the constant. Block 6: removed `newPasscode`/`currentPasscode` state, the `app_config` fetch effect, the New passcode input, and the Firestore rotation write block; unlock handler now compares `passcodeAttempt === DEFAULT_PASSCODE`.

## Decisions Made
- Followed plan as specified — decisions D-01 (rotation comment) and D-02 (leave `app_config` doc untouched) from 2-CONTEXT.md implemented exactly; wrong-passcode silent-clear and the D-04 preservation rules honored.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- PowerShell quoting mangled `node -e` one-liners containing escaped double quotes (e.g. the constant-value check) — resolved using the plan's documented fallback pattern: dot-wildcard regex (`/const DEFAULT_PASSCODE = .1234.;/`) avoids embedded quote literals; diff-chars measured via `git diff | Measure-Object -Character`. All plan-specified checks (removed identifiers, handler shape, block integrity, comment-before-constant) ran unmodified and passed.

## User Setup Required

None - no external service configuration required. Passcode rotation is a plain HTML edit: change `"1234"` on the `DEFAULT_PASSCODE` line in index.html Block 2.

## Next Phase Readiness
- Plan 02-02 (Configurable Plat & Approver) is untouched by this plan's removal — the constants pattern this phase establishes (module-scope config constants in Block 2) is the template for the plat/approver option lists.
- D-03 doc updates (PROJECT.md Firebase section, FEATURES.md passcode row, ARCHITECTURE.md passcode flow) are scoped OUT of this plan (files_modified: index.html only) and remain pending — 02-PATTERNS.md carries the exact edit targets for a later plan in this phase.
- End-of-phase browser UAT (config `human_verify_mode: end-of-phase`) will be re-run in the verify phase: serve `python -m http.server 8080`, open http://10.0.6.12:8080, and exercise submit form, manager passcode, approve, filter, CSV export, copy details, dark-mode toggle, return flow.

---
*Phase: 02-config-driven-passcode, Plan 01*
*Completed: 2026-09-14*

## Self-Check: PASSED

- SUMMARY file exists: `.planning/phases/02-config-driven-passcode/02-01-SUMMARY.md` — FOUND
- Task 1 commit `238cc36` — FOUND in git log
- Task 2 commit `5b9bf8b` — FOUND in git log