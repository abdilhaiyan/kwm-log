# Roadmap: KWM Logistics Equipment Log

## Overview

This milestone refactors the monolithic single-file React app (one ~532-line `App` component with 14 `useState` hooks) into a maintainable multi-block structure — 7 ordered `<script type="text/babel">` blocks (Constants → Utilities → Firebase Service → Shared UI → Modals → View Components → App Shell) — while preserving all 17 existing user behaviors identically and keeping single-file, no-build deployment. After the decomposition, three config-driven adjustments ride on the new constants block: the manager passcode becomes an HTML constant (Firebase decoupled), and the Plat Number + Approver fields become selects populated from editable constants at the top of the file.

**Coverage:** 13/13 v1 requirements mapped. No orphans.

## Phases

- [ ] **Phase 1: Multi-Block Decomposition** - Split the monolithic App into 7 ordered script blocks with no user-visible change
- [ ] **Phase 2: Config-Driven Passcode** - Passcode validated against an HTML constant; Firestore passcode logic removed
- [ ] **Phase 3: Configurable Plat & Approver Dropdowns** - Plat Number and Approver selects driven by editable constant arrays

## Phase Details

### Phase 1: Multi-Block Decomposition
**Goal**: The monolithic App component becomes a 7-block structure (Constants → Utilities → Firebase Service → Shared UI → Modals → View Components → App Shell) with a slim App orchestrator — and every user-visible behavior still works identically
**Mode:** mvp
**Depends on**: Nothing (first phase)
**Requirements**: REF-01, REF-02, REF-03, REF-04, REF-05
**Success Criteria** (what must be TRUE):
  1. Staff and managers can run the complete equipment workflow (borrow → approve/reject → pass → return) on a phone over LAN at http://10.0.6.12:8080 with results identical to before the refactor
  2. All 17 checklist behaviors from research/FEATURES.md pass when manually tested (borrow form, status filters, counts, search, CSV export, clipboard fallback, dark mode, toasts, unseen badge, passcode unlock)
  3. index.html is organized into the 7 ordered script blocks; utilities and constants live outside the component; the App shell holds only cross-cutting state (requests, auth, dark mode, view)
  4. The app still deploys as a single index.html served via `python -m http.server` — no npm, no build step, no new CDN dependencies
  5. Real-time Firestore sync still updates the request list in place after every extraction step (no duplicate listeners), and Lucide icons render in every view
**Plans**: 3 plans
**UI hint**: yes

Plans:
- [ ] 01-01-PLAN.md — Extract Constants + Utilities blocks (tracer: proves multi-block mechanism)
- [ ] 01-02-PLAN.md — Extract Firebase Service + Shared UI blocks
- [ ] 01-03-PLAN.md — Extract View Components + App Shell + Render blocks (completes decomposition)

### Phase 2: Config-Driven Passcode
**Goal**: Manager unlock works from a passcode constant in the HTML (`DEFAULT_PASSCODE = "1234"`); Firestore is no longer involved in passcode validation or rotation
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: PASS-01, PASS-02, PASS-03
**Success Criteria** (what must be TRUE):
  1. Manager unlocks manager actions by entering the passcode from the HTML constant (default "1234") — same modal, same Enter-key submit
  2. Entering a wrong passcode is rejected with the same error behavior and does not unlock
  3. No `app_config` document reads or writes appear in the network panel during unlock, page reload, or a full session — Firestore passcode logic is gone
  4. The change-passcode control is removed; rotating the passcode now means editing the `DEFAULT_PASSCODE` constant at the top of index.html
  5. The passcode modal looks and behaves identically (input, Unlock button, Enter submit, error state)
**Plans**: 2 plans
**UI hint**: yes

### Phase 3: Configurable Plat & Approver Dropdowns
**Goal**: Plat Number and Approver are config-driven selects populated from editable constant arrays at the top of index.html
**Mode:** mvp
**Depends on**: Phase 2
**Requirements**: PLAT-01, PLAT-02, PLAT-03, APPR-01, APPR-02
**Success Criteria** (what must be TRUE):
  1. When Vehicles is selected, the Plat Number field is a `<select>` showing a "Select vehicle" placeholder with "WRD 5900" first and "Other" second
  2. On approval, the Approver field is a dropdown showing a "Select approver" placeholder with "Shafiq" first and "Other" second; the chosen approver is recorded on the request
  3. Both option lists are defined as constants at the top of index.html — adding a new plate or approver is a one-line constant edit with no JSX or logic changes
  4. Plat number and approver still display correctly in the request cards, detail modal, copied details, and CSV export
**Plans**: TBD
**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Multi-Block Decomposition | 0/3 | Planned | - |
| 2. Config-Driven Passcode | TBD | Not started | - |
| 3. Configurable Plat & Approver Dropdowns | TBD | Not started | - |