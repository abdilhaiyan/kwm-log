# Requirements: KWM Logistics Equipment Log

**Defined:** 2026-09-11
**Core Value:** Accountable equipment handover — managers know who borrowed what, when it's due, and that it came back in good condition.

## v1 Requirements

### Refactor (Decomposition)

- [ ] **REF-01**: Decompose the monolithic `App` component (~14 useState, ~532 lines) into smaller, maintainable components without changing any user-visible behavior
- [ ] **REF-02**: Extract utility functions (isOverdue, getDaysInfo, getRequestDuration, getEquipmentLabel, getEquipmentIcon, copyDetails, exportCSV) out of the component into shared scope
- [ ] **REF-03**: Extract constants (STATUS_COLORS, INPUT_CLS, SELECT_CLS, equipment options, fuel levels, condition options) to a config section at the top
- [ ] **REF-04**: Preserve all 17 existing behaviors identically (see Validation Checklist in FEATURES.md)
- [ ] **REF-05**: Keep single-file deployment — no build step, no npm, no bundler, served via `python -m http.server`

### Passcode Configuration

- [ ] **PASS-01**: Manager passcode stored as a configurable constant in the HTML code (e.g. `const DEFAULT_PASSCODE = "1234"`), not integrated with Firebase
- [ ] **PASS-02**: Remove Firestore passcode fetch and update logic (no `app_config` doc reads/writes for passcode)
- [ ] **PASS-03**: Keep the passcode modal UI (input + Unlock button + Enter key submit) — just validate against the local constant

### Plat Number

- [ ] **PLAT-01**: Plat Number field is a `<select>` choose box (not a text placeholder)
- [ ] **PLAT-02**: Options include "WRD 5900" (first) and "Other" (second), with a placeholder "Select vehicle"
- [ ] **PLAT-03**: Plat number list defined as a configurable constant/array at the top of the code, easy to add more options later

### Approver

- [ ] **APPR-01**: Approver dropdown includes "Shafiq" (first) and "Other" (second), with a placeholder "Select approver"
- [ ] **APPR-02**: Approver list defined as a configurable constant/array at the top of the code, easy to add more names later

## v2 Requirements

### Features

- **FEAT-01**: ES modules + import maps migration (needs htm syntax conversion, separate `.js` files, static server)
- **FEAT-02**: Firebase modular SDK migration (v9+ API)
- **FEAT-03**: React 19 upgrade (requires ESM loading)
- **FEAT-04**: New features, error boundaries, performance optimization, TypeScript, tests

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| New user-facing features | This milestone is refactor + config adjustments only |
| UI styling or layout changes | Users notice visual changes — copy-paste exact same CSS |
| Firebase SDK migration | Highest behavior-change risk; separate milestone |
| ES modules / build step | Violates single-file CDN deployment constraint |
| State management library (Redux/Zustand) | 14 useState, 5 levels — prop drilling is fine at this scale |
| Renaming internal `'Car'` value | Existing Firestore documents use `'Car'`; renaming breaks data |
| Performance optimization | Internal tool, small list — premature optimization adds complexity |
| Error boundaries | Refactor milestone, not reliability milestone |
| Tests (unit/integration/E2E) | Can be a follow-up milestone |
| TypeScript or type annotations | Out of scope for this milestone |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| REF-01 | Phase 1 | Pending |
| REF-02 | Phase 1 | Pending |
| REF-03 | Phase 1 | Pending |
| REF-04 | Phase 1 | Pending |
| REF-05 | Phase 1 | Pending |
| PASS-01 | Phase 2 | Pending |
| PASS-02 | Phase 2 | Pending |
| PASS-03 | Phase 2 | Pending |
| PLAT-01 | Phase 3 | Pending |
| PLAT-02 | Phase 3 | Pending |
| PLAT-03 | Phase 3 | Pending |
| APPR-01 | Phase 3 | Pending |
| APPR-02 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 13 total
- Mapped to phases: 13
- Unmapped: 0 ✓

---
*Requirements defined: 2026-09-11*
*Last updated: 2026-09-11 after roadmap creation*
