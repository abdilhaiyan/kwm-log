# Requirements: KWM Logistics Equipment Log

**Defined:** 2026-09-15
**Core Value:** Accountable equipment handover — track who borrowed what, when, and whether it was returned, with an approval trail (approver name, passed status) so nothing gets lost or unaccounted.

## v1 Requirements

Requirements for milestone v1.1. Each maps to a roadmap phase.

### Sortable Lists

- [x] **SORT-01**: Manager can reorder equipment list items (up/down) in the Settings Equipment List card
- [x] **SORT-02**: Manager can reorder vehicle plates list items (up/down) in the Settings Vehicle Plates List card
- [x] **SORT-03**: Manager can reorder approvers list items (up/down) in the Settings Approvers List card
- [x] **SORT-04**: "Other" stays pinned as the last item in every list and cannot be moved
- [x] **SORT-05**: Reordered lists persist to Firestore and sync to the request form select, manager filter select, and detail views on all devices
- [x] **SORT-06**: Reordered lists survive a page reload and stay in the same order for every user

### Manager Styling

- [x] **UI-01**: Approve button on manager request cards uses the translucent glassmorphic theme
- [x] **UI-02**: Reject button on manager request cards uses the translucent glassmorphic theme
- [x] **UI-03**: Approve button in the request detail modal uses the translucent glassmorphic theme
- [x] **UI-04**: Reject button in the request detail modal uses the translucent glassmorphic theme

### Form Tweaks & Borrower Return Flow

- [x] **FORM-01**: Placeholders for 'Requested By' and 'Purpose of use' form inputs are in ALL CAPS
- [x] **FORM-02**: Vehicles equipment option renders the fuel level status bar selector
- [x] **FORM-03**: Mark Returned feature is moved/added to borrower status view so borrowers can mark items returned to company with auto date & time
- [x] **FORM-04**: Manual request time input is omitted from the vehicle borrow form section (captured automatically by system)

### Vehicle Fuel-Bar Mode

- [x] **VEH-01**: Vehicle plates removed from the borrow form — plate is embedded in the equipment name (e.g., "WRD 5900 (Myvi)")
- [x] **VEH-02**: Email input hidden when a vehicle equipment option is selected
- [x] **VEH-03**: Fuel bar labels are exactly `1 Bar / 2 Bar / 3 Bar / 4 Bar / Full`
- [x] **VEH-04**: Vehicle equipment items are locked in Settings (no reorder/edit); deletable only via passcode modal with a red warning
- [x] **VEH-05**: Return fuel bar must be actively chosen — CONFIRM RETURN disabled until a bar is selected

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
| SORT-01 | Phase 4 | Complete |
| SORT-02 | Phase 4 | Complete |
| SORT-03 | Phase 4 | Complete |
| SORT-04 | Phase 4 | Complete |
| SORT-05 | Phase 4 | Complete |
| SORT-06 | Phase 4 | Complete |
| UI-01 | Phase 5 | Complete |
| UI-02 | Phase 5 | Complete |
| UI-03 | Phase 5 | Complete |
| UI-04 | Phase 5 | Complete |
| FORM-01 | Phase 6 | Complete |
| FORM-02 | Phase 6 | Complete |
| FORM-03 | Phase 6 | Complete |
| FORM-04 | Phase 6 | Complete |
| VEH-01 | Phase 6 | Complete |
| VEH-02 | Phase 6 | Complete |
| VEH-03 | Phase 6 | Complete |
| VEH-04 | Phase 6 | Complete |
| VEH-05 | Phase 6 | Complete |

**Coverage:**

- v1 requirements: 19 total
- Mapped to phases: 19
- Unmapped: 0 ✓

---
*Requirements defined: 2026-09-15*
*Last updated: 2026-09-15 after initial definition*
