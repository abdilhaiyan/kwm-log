# Roadmap: KWM Logistics Equipment Log

## Milestones

- ✅ **v1.0 MVP** — Phases 1-3 (shipped 2026-09-15)
- 🚧 **v1.1 Sortable Lists & Manager Styling** — Phases 4-5 (in progress)

## Phases

<details>
<summary>✅ v1.0 MVP (Phases 1-3) — SHIPPED 2026-09-15</summary>

- [x] Phase 1: Multi-Block Decomposition (3/3 plans) — completed 2026-09-13
- [x] Phase 2: Config-Driven Passcode (2/2 plans) — completed 2026-09-14
- [x] Phase 3: Configurable Plat & Approver Dropdowns (1/1 plan) — completed 2026-09-14

</details>

### 🚧 v1.1 Sortable Lists & Manager Styling (In Progress)

**Milestone Goal:** Managers can reorder the three dropdown lists (equipment, vehicle plates, approvers) via up/down controls in the Settings list cards — order persists to Firestore and syncs live to every surface that renders the list — and the Approve/Reject buttons match the translucent glassmorphic theme in both manager request cards and the detail modal.

- [ ] **Phase 4: Sortable List Reordering** - Up/down reorder controls in all three Settings list cards; "Other" pinned last; order persists via Firestore and syncs to every list surface
- [ ] **Phase 5: Manager Glassmorphic Styling** - Approve/Reject buttons restyled to the translucent glass theme in manager request cards and the detail modal

## Phase Details

### Phase 4: Sortable List Reordering
**Goal**: Managers can reorder the equipment, vehicle plate, and approver lists from Settings, and the new order persists to Firestore and appears consistently on every surface that renders those lists for every user.
**Depends on**: Phase 3 (existing ListEditor cards + shared `equipmentOptions`/`platOptions`/`approverOptions` state + `saveOptions()` Firestore write path)
**Requirements**: SORT-01, SORT-02, SORT-03, SORT-04, SORT-05, SORT-06
**Success Criteria** (what must be TRUE):
  1. Manager can move any equipment item up or down with new controls in the Settings Equipment List card, and the request form's equipment select shows the new order immediately.
  2. Manager can reorder vehicle plates and approvers the same way; the plat select and the approver select on the request form and in the detail modal reflect the new order.
  3. "Other" stays pinned as the last item in every list after any sequence of moves — its up/down controls are disabled, matching the existing lock on edit/delete.
  4. After a page reload — and on a second phone or computer — every list renders in the saved order for all users (order persisted via Firestore).
  5. Reordered order applies live to every surface rendering the list (request form select, manager filter select, detail view selects) without an app restart.
**Plans**: TBD
**UI hint**: yes

### Phase 5: Manager Glassmorphic Styling
**Goal**: The manager Approve/Reject buttons match the app's translucent glassmorphic theme in both the request cards and the detail modal, with zero change to approve/reject behavior.
**Depends on**: Phase 4 (execution order only — styling work is independent of sorting)
**Requirements**: UI-01, UI-02, UI-03, UI-04
**Success Criteria** (what must be TRUE):
  1. Approve button on manager request cards uses the translucent glassmorphic treatment (translucent emerald fill, emerald text, subtle emerald border) instead of the solid emerald fill.
  2. Reject button on manager request cards uses the matching translucent red glassmorphic treatment instead of the solid red fill.
  3. Approve and Reject buttons in the request detail modal use the same translucent glassmorphic treatment as the card buttons.
  4. With the new styling, the manager can still approve (recording approver name/date) and reject requests from both the cards and the modal exactly as before — only the button appearance changed.
**Plans**: TBD
**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 4 → 5

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Multi-Block Decomposition | v1.0 | 3/3 | Complete | 2026-09-13 |
| 2. Config-Driven Passcode | v1.0 | 2/2 | Complete | 2026-09-14 |
| 3. Configurable Plat & Approver Dropdowns | v1.0 | 1/1 | Complete | 2026-09-14 |
| 4. Sortable List Reordering | v1.1 | 0/TBD | Not started | - |
| 5. Manager Glassmorphic Styling | v1.1 | 0/TBD | Not started | - |