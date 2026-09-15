# Requirements: KWM Logistics Equipment Log

**Defined:** 2026-09-15
**Core Value:** Accountable equipment handover — track who borrowed what, when, and whether it was returned, with an approval trail (approver name, passed status) so nothing gets lost or unaccounted.

## v1 Requirements

Requirements for milestone v1.1. Each maps to a roadmap phase.

### Sortable Lists

- [ ] **SORT-01**: Manager can reorder equipment list items (up/down) in the Settings Equipment List card
- [ ] **SORT-02**: Manager can reorder vehicle plates list items (up/down) in the Settings Vehicle Plates List card
- [ ] **SORT-03**: Manager can reorder approvers list items (up/down) in the Settings Approvers List card
- [ ] **SORT-04**: "Other" stays pinned as the last item in every list and cannot be moved
- [ ] **SORT-05**: Reordered lists persist to Firestore and sync to the request form select, manager filter select, and detail views on all devices
- [ ] **SORT-06**: Reordered lists survive a page reload and stay in the same order for every user

### Manager Styling

- [ ] **UI-01**: Approve button on manager request cards uses the translucent glassmorphic theme
- [ ] **UI-02**: Reject button on manager request cards uses the translucent glassmorphic theme
- [ ] **UI-03**: Approve button in the request detail modal uses the translucent glassmorphic theme
- [ ] **UI-04**: Reject button in the request detail modal uses the translucent glassmorphic theme

## v2 Requirements

Deferred to a future release. Tracked but not in the current roadmap.

### Sortable Lists

- **SORT-07**: Drag-and-drop reordering of lists (up/down buttons are touch-safe for the LAN/mobile context; revisit when a desktop-first use case appears)

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Renaming internal `'Car'` value | Existing Firestore documents use `'Car'`; renaming breaks data |
| Adding new default equipment options | Kept list as-is at user request; add via Settings when needed |
| Reordering settings beyond the three list cards | Passcode section order is fixed |
| Migrating the option storage schema | Reorder writes the existing `options/lists` doc — no schema change |
| Drag-and-drop reordering | Desktop-first gesture; deferred to v2 |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| SORT-01 | Phase 4 | Pending |
| SORT-02 | Phase 4 | Pending |
| SORT-03 | Phase 4 | Pending |
| SORT-04 | Phase 4 | Pending |
| SORT-05 | Phase 4 | Pending |
| SORT-06 | Phase 4 | Pending |
| UI-01 | Phase 5 | Pending |
| UI-02 | Phase 5 | Pending |
| UI-03 | Phase 5 | Pending |
| UI-04 | Phase 5 | Pending |

**Coverage:**
- v1 requirements: 10 total
- Mapped to phases: 10
- Unmapped: 0 ✓

---
*Requirements defined: 2026-09-15*
*Last updated: 2026-09-15 after initial definition*